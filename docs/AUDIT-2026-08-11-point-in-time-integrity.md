# Audit — point-in-time integrity of the flagship's inputs

**Date:** 2026-08-11 · Project `pvqflvqfdlmiyjjetfer` · read-only SQL, nothing written to the database
**Scope:** whether run #14's closed years used data that was publicly knowable at the rebalance date.
**Prompted by:** the discovery that `AnnualValuationQuality` carries `Filing Date`, joinable to
`apex_screening_master` at 100% coverage — which makes the look-ahead question testable for the
first time.

---

## Verdict

Two separate defects, of very different severity.

1. **Filing-date look-ahead is real but structural, narrow, and — measurably — not inflating the
   backtest.** It concentrates almost entirely in companies whose fiscal year ends between January
   and March. Affected picks *underperformed* the clean ones.
2. **The panel's company identity is not point-in-time.** Proven with a named example: the master
   file's own `company_name` for ticker `WBD` in performance year **2008** reads *"Warner Bros.
   Discovery Inc"* — an entity that did not exist until April 2022. This is the blocking
   survivorship condition, and the mechanism is now visible rather than hypothetical.

Defect 2 is the serious one. Defect 1 is fixable with a filter.

---

## 1. Filing-date look-ahead

### Method

`apex_screening_master.reporting_period_id` → `AnnualReportPeriods.annual_report_id` →
`AnnualValuationQuality."Filing Date"` (text, `MM/DD/YY`). A row for rebalance year Y carries
fiscal year Y−1 fundamentals — verified: `performance_year = calendar_year + 1` on all 141,321
rows that have a `performance_year`. The rebalance is 1 April Y. So the test is:

```sql
to_date(q."Filing Date", 'MM/DD/YY') >= make_date(ry, 4, 1)
```

### Result — the 186 stored picks

| | count |
|---|---|
| picks matched to a filing date | 184 of 186 |
| **filed on or after the rebalance date** | **15 (8.2%)** |
| mean lead time, clean rows | 56 days before rebalance |

### Result — the 380-name candidate pool, split by fiscal year end

This is the cut that explains it:

| fiscal year end | pool rows | filed after rebalance | rate | < 180 days late | ≥ 180 days late |
|---|---|---|---|---|---|
| **Jan–Mar** | 21 | **16** | **76.2%** | 16 | 0 |
| Dec | 241 | 11 | 4.6% | 6 | 5 |
| Apr–Nov | 116 | 1 | 0.9% | 0 | 1 |

A company whose fiscal year ends 31 March cannot have filed that year's accounts by 1 April.
The 76.2% rate in that cohort is not bad luck; it is arithmetic. Named cases: **UHAL** (March FYE,
rank 1 in both 2009 and 2010, filed 64 and 69 days after rebalance), **ELF** (March FYE, 2022,
filed 55 days after), **CVLT** (March FYE, 2011), **CATO** (January FYE — a retailer, filed 24 and
21 days after rebalance in 1997 and 1998).

### The five long-lag December cases are metadata, not data

`SHOO` 1998, `MHK` 1996, `WBD` 2008, `TEX` 2005, `FLS` 2005 show filing dates 318–362 days after
the rebalance. My first hypothesis was a year-alignment bug in `reporting_period_id`. **That was
wrong.** Checked directly:

| pick | `calendar_year` | linked fiscal year | `current_eps` | report `EPS without NRI` |
|---|---|---|---|---|
| SHOO 1998 | 1997 | 1997-12 ✓ | 0.0170 | 0.017 ✓ |
| MHK 1996 | 1995 | 1995-12 ✓ | 0.1600 | 0.16 ✓ |
| WBD 2008 | 2007 | 2007-12 ✓ | 0.1470 | 0.147 ✓ |
| TEX 2005 | 2004 | 2004-12 ✓ | 1.4260 | 1.426 ✓ |
| FLS 2005 | 2004 | 2004-12 ✓ | 0.2340 | 0.234 ✓ |

The fiscal year is correctly aligned and the EPS matches the linked report exactly. A FY1997-12
report cannot have an original filing date of 1999-03-29 — so the **`Filing Date` field is
unreliable for these rows** (restatement or amendment date, or simply wrong), while the
fundamentals are sound. Do not treat these five as look-ahead.

### Materiality — the reassuring part

If the screen were exploiting foreknowledge, the affected picks should outperform. They do not:

| group | picks | mean return | median return | times ranked 1 |
|---|---|---|---|---|
| FYE Jan–Mar (structural look-ahead) | 10 | **72.5%** | **47.3%** | 3 |
| Dec-FYE genuine late filers | 4 | 10.3% | −7.1% | 1 |
| clean | 172 | **76.6%** | **52.0%** | 26 |

Affected picks came in **below** the clean cohort on both mean and median. With n = 10 against
n = 172 this is not a precise estimate and the confidence interval is wide — but the direction is
the opposite of the inflation hypothesis, and the exposure is small either way: 14 of 186 picks
(7.5%), 21 of 380 pool name-years (5.5%).

**Conclusion:** a genuine methodology defect that should be fixed, but not a reason to distrust
the headline numbers. Removing the affected picks would, if anything, nudge returns up.

### Fix

Add to the generator, and to the Tier 0 feature view:

```sql
-- exclude any row whose fundamentals were not filed before the rebalance
and to_date(q."Filing Date",'MM/DD/YY') < make_date(a.performance_year, 4, 1)
```

with a carve-out for the five known bad-metadata rows, or better, a rule that ignores filing dates
more than 180 days after the fiscal year end as implausible. Expose the guard as a column rather
than filtering silently, so the excluded set stays visible.

---

## 2. Company identity is not point-in-time — BLOCKING

### The proof

```sql
select performance_year, ticker, company_name
from public.apex_screening_master
where ticker = 'WBD' and performance_year = 2008;
-- 2008 | WBD | Warner Bros. Discovery Inc
```

Warner Bros. Discovery was created in **April 2022** by the merger of WarnerMedia and Discovery.
It has no 2008. The ticker `WBD` on the NYSE in 2008 belonged to a different company entirely.

This is not an artefact of joining to `public.companies` — the name is carried in
`apex_screening_master`'s own `company_name` column, and `companies` agrees with it. The panel is
keyed on **ticker**, and present-day identity has been carried backward through history.

### Why it is blocking

Everything the universe gate does with identity is therefore suspect for any reused ticker:

- **`company_name` is the dedup key.** `row_number() over (partition by performance_year,
  company_name order by market_cap desc)` — if the name is wrong, the dedup is wrong.
- **`sector` and `industry` drive the Energy exclusion and the 14-industry exclusion.** WBD/2008
  carries sector *Communication Services*, which is the 2022 company's classification.
- **`is_adr` drives the ADR exclusion**, and it is resolved through the same ticker join.

And the deeper issue: if the panel is assembled from a present-day ticker list, **companies that
delisted and whose tickers were never reused are simply absent**. That is the classic survivorship
condition, and no amount of feature engineering repairs it.

### Related: the universe gate does not enforce "US"

The generator's only nationality test is `coalesce(c.is_adr, false) = false`. It never checks
`country` or `exchange`. Two picks fail the config's stated universe of *"core US universe
(NYSE/NAS, no ADR)"*:

| pick | company | country in `companies` | `is_adr` |
|---|---|---|---|
| EBR.B (2022, rank 2) | Centrais Eletricas Brasileiras SA | **null** | false |
| STNE (2023, rank 5) | StoneCo Ltd | **"Unknown"** | false |

A Brazilian state utility and a Cayman-incorporated Brazilian payments company both cleared a
"core US, no ADR" filter, because the flag they are tested on is wrong and the country field is
never consulted.

### What must happen next

1. **Run the full survivorship test.** Take several hundred names with `companies.active = false`
   and `universe_last_seen` in 2010–2012, and check whether they appear in
   `apex_screening_master` for `performance_year` 2011. If they do not, the panel is not
   point-in-time and **no model should be trained on it**.
2. **Scan for ticker reuse.** Count tickers whose `company_name` is constant across the panel but
   whose linked `annual_report_id` chain shows a discontinuity — WBD is one instance; the
   population size is unknown.
3. **Source point-in-time identity** — `company_name`, `sector`, `industry`, `country`, `is_adr`
   as at each rebalance date, keyed on a stable company identifier rather than the ticker string.
   This is item 2.3 in the data request and its priority should be raised.

---

## 3. What this means for the ML programme

- **The filing-date defect does not block anything.** Fix it in the generator, note it, move on.
  It is small and it is not paying.
- **The identity defect blocks model training.** Not because it invalidates the existing CAGR —
  that remains an open question pending the survivorship test — but because a model trained on a
  panel carrying present-day identity backward will learn the characteristics of companies that
  are still here in 2026. That failure mode produces excellent backtests and no live performance.
- Sequence is unchanged from the plan: survivorship test → widen the pool → learn to rank. This
  audit moves the first item from "recommended" to "required", and supplies the mechanism to
  look for.
