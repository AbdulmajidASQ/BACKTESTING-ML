# Data request — ML feature layer for the flagship (run #14)

**Date:** 2026-08-11 · Project `pvqflvqfdlmiyjjetfer` · for the data agent
**Requested by:** quant/ML research, in support of selection-alpha capture and sleeve sizing

---

## 0. Purpose in one paragraph

Run #14 selects 7 names a year from a 380-name-year candidate pool by ranking on
`dcf_discount_percent`. That ranking captures **28%** of the available selection alpha; 72% is
open. Five alternative ranking formulas have already been tested on the current pool and all
landed within a point of the incumbent, so the remaining alpha is **not** reachable by
re-weighting the existing signals. This request covers the feature layer needed to reach it,
plus the integrity data needed to know whether any of it is real.

**Everything here is subject to one rule that overrides convenience: no value may enter a row
for rebalance year Y unless it was publicly knowable before 1 April of year Y.** A feature that
violates this is worse than a missing feature, because it will look excellent in backtest.

---

## 1. Headline finding — most of this needs no sourcing

`public.apex_screening_master.reporting_period_id` joins to
`public."AnnualReportPeriods".annual_report_id` at **150,224 / 150,224 rows — 100% coverage**:

```sql
select count(*) from public.apex_screening_master a
join public."AnnualReportPeriods" p on p.annual_report_id = a.reporting_period_id;
-- 150224, equal to the full row count of apex_screening_master
```

That key unlocks five sibling tables the flagship has never read. Column counts, excluding the
join key:

| table | columns | character |
|---|---|---|
| `AnnualValuationQuality` | 37 | quality scores, filing dates, buybacks, alternative fair values |
| `AnnualCashFlowStatements` | 39 | full cash-flow statement incl. SBC, buybacks, working-capital deltas |
| `AnnualCommonSizeRatios` | 38 | returns on capital incl. **ROIC and WACC**, turnover, margins, cash cycle |
| `AnnualValuationRatios` | 21 | EV multiples, earnings yield, cyclically-adjusted ratios |
| `AnnualPerShareMetrics` | 14 | per-share fundamentals incl. owner earnings, tangible book |
| **total** | **~149** | |

The flagship currently reads **21 of the master file's 213 columns**. There are ~149 more one
join away, and they include precisely the axis the current screen is missing (see §3).

Verified coverage on `AnnualValuationQuality`, by era, over 1996–2025:

| field | 1996–99 | 2000–09 | 2010–19 | 2020–25 |
|---|---|---|---|---|
| Sloan Ratio % | 100% | 100% | 100% | 100% |
| Buyback / Shareholder Yield % | 100% | 100% | 100% | 100% |
| Beneish M-Score | 100% | 100% | 99.8% | 98.9% |
| Piotroski F-Score | 95.6% | 96.9% | 96.9% | 91.9% |
| Filing Date | 79.8% | 90.3% | 98.9% | 96.6% |
| Altman Z-Score | 73.4% | 74.9% | 76.0% | 78.7% |
| Beta · Highest/Lowest Stock Price | 77.6% | 79.0% | 79.1% | 76.7% |
| Earnings Release Date | 0.0% | 1.4% | 65.2% | 73.0% |

`Earnings Release Date` is unusable before 2010 — treat it as a 2010+ field only.

**Coverage for the other four tables has not been measured.** Please report null rates by era
for each, using the same four buckets, as part of delivering Tier 0.

---

## 2. Tier 0 — build one view. No new data.

This is the single highest-value item in the request and it is pure plumbing.

**Deliverable:** `bt.v_us14_features` — one row per candidate name-year, at the grain of
`bt.v_us14_cand_all`, carrying every column from the five sibling tables plus the master file's
own fields.

```
grain:        (ry, ticker)
base:         bt.v_us14_cand_all           -- the candidate pool, 1996-2025 + frozen 2026
join path:    apex_screening_master.reporting_period_id
                -> AnnualReportPeriods.annual_report_id
                -> the five Annual* tables
```

Requirements:

1. **Widen the base, don't use the shipped pool.** Build it on the pool with `eps_forward_cagr`
   relaxed to `> 0` (all other gates unchanged), which yields **835 name-years** instead of 380.
   Carry a boolean `in_incumbent_pool` marking the 380 that pass `fwd > 25`, so the incumbent
   remains an exact subset and the control is reproducible.
2. **Snake_case the column names.** The source tables use spaced, capitalised, symbol-bearing
   names (`Sloan Ratio %`, `ROC (Joel Greenblatt) %`, `EV-to-EBITDA`). Normalise to
   `sloan_ratio_pct`, `roc_greenblatt_pct`, `ev_to_ebitda`. Do not drop columns to save effort —
   feature selection is my job, not the pipeline's.
3. **Apply the filing-date guard** (§4) and expose the guard columns rather than silently
   filtering.
4. **RLS in the same statement batch as the create**, per house standard:
   `alter table ... enable row level security; revoke all on ... from anon, authenticated;`
   Add a table comment stating what it is, when built, and its key caveat.

Do not select features for me, do not impute nulls, do not winsorise. Deliver it raw.

---

## 3. Why these columns — the orthogonality problem

Measured on the 380-name pool, the within-year Spearman correlation between `disc` and `fwd` is
**0.494**. The DCF fair value substantially embeds forward growth, so the flagship's two
"independent" gates are roughly half the same gate. That is why re-weighting them goes nowhere.

Any feature that adds information must be **orthogonal to the forward-EPS axis**. Ranked by how
well they meet that test, my priority subset for the first model:

**Earnings quality and accruals** — separates a real discount from a value trap. The incumbent
rank key is positive on sign in only 17 of 29 years; it is strong when right and near-random
otherwise, which is the signature of an unfiltered value screen.
`Sloan Ratio %` · `Scaled Net Operating Assets` · `Beneish M-Score` · `Piotroski F-Score` ·
`Change In Working Capital` · `Change In Receivables` · `Change In Inventory` ·
`Asset Impairment Charge` · `Deferred Tax`

**Capital allocation and dilution** — currently invisible to the screen, and material in a
small/mid-cap growth universe that issues heavily.
`Buyback Yield %` · `Shareholder Yield %` · `Shares Buyback Ratio %` · `Issuance of Stock` ·
`Repurchase of Stock` · `Stock Based Compensation` · `Shares Outstanding (Basic Average)` ·
`Shares Outstanding (EOP)`

SBC deserves emphasis: every one of the 2026 book's seven names is Technology, and SBC-adjusted
earnings quality is a first-order differentiator in that sector. It is not in the master file.

**Return on capital vs cost of capital** — `ROIC %` and `WACC %` are both present, so
`ROIC − WACC` is directly computable. That is the cleanest available measure of whether growth
creates value, and it is conceptually independent of how cheap the stock looks.
`ROIC %` · `WACC %` · `ROC (Joel Greenblatt) %` · `Gross-Profit-to-Asset %` ·
`Return-on-Tangible-Equity` · `1-Year ROIIC %` · `5-Year RORE %`

`Gross-Profit-to-Asset %` is Novy-Marx gross profitability, among the most robust
cross-sectional factors in the published literature, and it is absent from the current screen.

**Operating efficiency and operating leverage** — candidates for explaining why forward growth
does or does not convert.
`Asset Turnover` · `Cash Conversion Cycle` · `Days Inventory` · `Days Sales Outstanding` ·
`Degree of Operating Leverage` · `Degree of Financial Leverage` · `Capex-to-Operating-Cash-Flow` ·
`RD-to-Revenue` · `FCF Margin %`

**Alternative valuation anchors** — partially orthogonal to an equity DCF, and capital-structure
neutral, which the current discount is not.
`EV-to-EBITDA` · `EV-to-EBIT` · `EV-to-FCF` · `Earnings Yield (Joel Greenblatt) %` ·
`FCF Yield %` · `Forward Rate of Return (Yacktman) %` · `Price-to-Owner-Earnings` ·
`Graham Number` · `Earnings Power Value (EPV)` · `Net Cash per Share` ·
`Intrinsic Value: Projected FCF` · `Median PS Value`

**Distress and risk** — for the drawdown and sleeve-sizing track.
`Altman Z-Score` · `Beta` · `Highest Stock Price` · `Lowest Stock Price` ·
`Interest Coverage` · `Cash Ratio`

`Highest`/`Lowest Stock Price` give an intra-year price range at annual grain, which supports a
first-pass drawdown taxonomy without sourcing any daily prices.

---

## 4. Tier 1 — derivable from columns already present. No sourcing.

Please compute these into the Tier 0 view. All inputs already exist in `apex_screening_master`
or the sibling tables.

| new column | definition | why |
|---|---|---|
| `accruals_ratio` | `(net_income − operating_cash_flow) / total_assets` | classic earnings-quality signal; all three inputs in the master |
| `cash_conversion` | `operating_cash_flow / nullif(net_income, 0)` | corroborates reported earnings |
| `dilution_1y` | `shares_outstanding / lag(shares_outstanding) over (partition by company_id order by calendar_year) − 1` | one row per company-year already exists |
| `roic_less_wacc` | `"ROIC %" − "WACC %"` | value creation, directly available |
| `margin_trend_1y` | `operating_margin_percent − lag(operating_margin_percent)` | direction, not level |
| `fcf_yield_calc` | `free_cash_flow / nullif(market_cap, 0)` | cross-check against the supplied `FCF Yield %` |
| `disc_resid_fwd` | residual of `disc` regressed on `fwd` **within each `ry`** | the orthogonalised discount; given ρ = 0.494 this is the single most important derived column in this table |
| `estimate_curvature` | `(eps_f3/eps_f2) − (eps_f2/eps_f1)` | shape of the forward path, not just its slope |

`disc_resid_fwd` must be computed within-year, using only that year's pool. Do not fit it across
the panel — that leaks.

---

## 5. Tier 2 — genuinely must be sourced

Only four items. Ordered by value.

### 2.1 Estimate revision history — highest value

The DCF discount is a *shadow* of forward estimates; revisions are the thing itself. Currently
only a single snapshot of `eps_f1..f4` at rebalance exists, with no history, so no delta can be
computed.

Required, per `(company_id, rebalance_year)`:

| column | type | definition |
|---|---|---|
| `eps_f1_t30`, `eps_f1_t90`, `eps_f1_t180` | numeric | consensus `eps_f1` as it stood 30 / 90 / 180 calendar days before 1 April of the rebalance year |
| `eps_f2_t30`, `eps_f2_t90`, `eps_f2_t180` | numeric | same for `eps_f2` |
| `analyst_count` | integer | number of contributing analysts at rebalance |
| `eps_f1_stddev`, `eps_f2_stddev` | numeric | dispersion of contributing estimates at rebalance |
| `revision_up_count_90d`, `revision_down_count_90d` | integer | analyst revisions in each direction over the prior 90 days |

`intl.estimates` (48,389 rows) may already hold some of this — check there before sourcing
externally. If only a subset is recoverable, deliver the subset; the 90-day `eps_f1` delta and
`analyst_count` alone would be worth having.

### 2.2 Delisted companies and point-in-time status — blocking

No feature fixes a panel that is missing the dead. If `apex_screening_master` was assembled from
today's company list and back-filled, every backtest figure in this project is inflated and no
model should be trained.

| column | type | definition |
|---|---|---|
| `delisted_date` | date | when the listing ceased |
| `delisting_reason` | text | one of: acquired · merged · bankrupt · taken_private · exchange_rule · other |
| `is_listed_at_rebal` | boolean | point-in-time as at 1 April of the rebalance year |
| `terminal_return_pct` | numeric | return to the delisting event, for names that die mid-holding-year |

More important than the columns: **rows must exist for companies that no longer trade.**
`public.companies` carries `active` and `universe_last_seen`, which is enough to identify
candidates for the audit but not enough to repair the panel if it is incomplete.

### 2.3 Point-in-time sector and industry

`sector` and `industry` drive the 14-industry exclusion and the Energy exclusion, and today's
classifications are almost certainly back-filled. A company classified Technology now may have
been Industrials in 1998.

| column | type |
|---|---|
| `sector_at_rebal` | text |
| `industry_at_rebal` | text |
| `classification_as_of` | date |

If a true history is unavailable, say so explicitly rather than approximating — I would rather
carry a documented limitation than an invented one.

### 2.4 DCF assumptions

`dcf_fair_value` is 67.6% null and is the parent of the rank key, but its construction is opaque.
Given the 0.494 correlation with `fwd`, I need to know how much of the discount is an independent
valuation judgement and how much is a restatement of the growth estimate.

| column | type |
|---|---|
| `dcf_discount_rate` | numeric |
| `dcf_terminal_growth` | numeric |
| `dcf_horizon_years` | integer |
| `dcf_method` | text |
| `dcf_base_eps` | numeric |

---

## 6. Point-in-time rules — non-negotiable

1. **The 1 April cutoff.** No value in a row for rebalance year Y may have been publicly unknown
   before 1 April Y. This governs fundamentals, estimates, classifications and prices alike.
2. **Use `Filing Date`, not fiscal year, to decide knowability.** `AnnualValuationQuality` carries
   `Filing Date` and `Restated Filing Date`; `AnnualReportPeriods` carries a `Preliminary` flag.
   A row whose `Filing Date` falls on or after 1 April Y must not contribute features to
   rebalance year Y — carry the prior year's filing instead, and flag it.
3. **Prefer the original filing over the restatement.** Where `Restated Filing Date` differs from
   `Filing Date`, the original figures are what the market saw. Expose both; default to the
   original.
4. **Expose the guard, do not hide it.** Ship `filing_date`, `restated_filing_date`,
   `is_preliminary`, and a computed `knowable_at_rebal` boolean. I want to see what was excluded
   and why, not receive a pre-cleaned table.
5. **No imputation, no winsorising, no scaling.** Nulls are information — the null rate on
   `dcf_discount_percent` is 71.4% and that fact is itself a finding. Any filling destroys it.

---

## 7. Acceptance tests

Tier 0 is done when all of these hold. Please report the actual numbers, not a pass/fail.

1. `bt.v_us14_features` restricted to `in_incumbent_pool = true` returns **exactly 380 rows** for
   `ry` 1996–2025, and its `(ry, ticker)` set is identical to `bt.v_us14_cand`.
2. The widened pool returns **835 rows** for the same window.
3. Every one of run #14's **186 stored closed-year picks** is present in the view.
4. Row count is preserved through the join — no fan-out. `count(*)` equals the pool size exactly.
5. Null rates by era, four buckets, reported for every delivered column.
6. **Look-ahead audit:** count of rows where `Filing Date >= make_date(ry, 4, 1)`. This number is
   the headline output of the whole delivery. If it is materially above zero, the existing
   backtest has been using fundamentals that were not yet public, and that finding takes
   precedence over every feature in this document.

---

## 8. What I will do with it

For context, so the shape of the delivery makes sense:

- **Control first.** Re-run the incumbent `disc desc` ranking on the widened 835-name pool, with
  no new features, to establish whether widening alone helps or hurts. Ranking is measured to add
  +19.8 points/year when the pool is at least 2× the book and +3.9 when it is not; widening on
  `fwd` takes the count of years meeting that condition from 15/30 to 26/30 and removes all four
  years in which no selection decision existed at all.
- **Then a learning-to-rank model** — pairwise objective, year as query group — trained on the
  global 432k company-year panel and deployed on the US pool. One frozen model, not a per-year
  selection among candidates: annual re-selection is exactly what produced the previous
  walk-forward result of 63.66 against an incumbent of 67.67.
- **Validation** is purged and embargoed walk-forward plus leave-one-market-out. April→April
  labels overlap by construction.
- **The baseline to beat is 67.67**, the incumbent's CAGR on the 380-name pool. Not zero.

Nothing gets trained until acceptance test 6 and the survivorship question in §5.2 are settled.
