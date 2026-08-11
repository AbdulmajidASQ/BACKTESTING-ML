# BACKTESTING-ML

Documentation and research notes for the **Mizan** backtesting project — a quantitative
strategy research programme run against the Supabase project `pvqflvqfdlmiyjjetfer`.

This repository holds written artefacts: audits, data-dictionary work, and findings.
The data, views and procedures themselves live in Supabase, not here.

---

## Contents

| Document | What it covers |
|---|---|
| [`docs/AUDIT-2026-08-11-run-14-flagship.md`](docs/AUDIT-2026-08-11-run-14-flagship.md) | Full audit of the flagship US run #14 against the registration checklist and the audit gates. What passes, what fails, and the SQL behind each claim. |
| [`docs/apex_screening_master-column-map.md`](docs/apex_screening_master-column-map.md) | Every one of the 213 columns in `public.apex_screening_master`, classified as used or not used by the flagship, with null rates and the exact gate each column drives. |

---

## The project in one page

**Supabase project:** `pvqflvqfdlmiyjjetfer`. Three schemas, one rule each:

| Schema | Rule |
|---|---|
| `bt` | Everything a backtest produces or holds as strategy state |
| `intl` | Shared market data — inputs, reused by the live screener and other surfaces |
| `archive` | Superseded / backup / scratch. Never dropped, never read by the app |

The iOS app talks **only** to `public.mizan_app_*` RPCs (all `SECURITY DEFINER`). It never
reads `bt` or `intl` directly.

**Registry tables that define a run:** `bt.runs` (one row per backtest), `bt.run_years` (one
row per rebalance year including idle years), `bt.picks` (one row per holding per year),
`bt.catalog` (the subscriber-facing record), `bt.run_events` (provenance log), `bt.glossary`.

**Daily NAV layer:** `bt.btd_books` (holdings *with weights* — the authoritative weight
source, never infer weight from rank) → `bt.btd_px` → `bt.btd_rel` → `bt.btd_daily`.

**The flagship** is run #14: `United States (US) · Conc-7 · LIVE-RULES + Recovery branch
($5B era 2026+)`, 1996-04-01 → live. See the audit for its verified numbers.

---

## House rules

These are owner constraints and engineering standards. They are not negotiable and should
not be worked around.

**Owner constraints**

- Buying options is permitted. **Selling / writing options is not** — no covered calls, no
  cash-secured puts, no spreads with a short leg, no premium collection of any kind.
- **Cash earns 0%.** No interest-bearing instruments, no money-market yield, no margin
  borrowing. Any cash leg in a backtest returns exactly zero.
- Broker is IBKR. Assume US-listed instruments and standard retail fills.

**Engineering standards**

- **Never `DROP` anything.** Supersede by archiving into the `archive` schema; renaming to
  `*_pre_<change>` in `archive` is the house pattern.
- **RLS on every new table, in the same statement batch as the `CREATE`:**
  `alter table X enable row level security; revoke all on X from anon, authenticated;`
- Name markets by country, not by exchange code, in anything user-facing.
- Look-ahead / perfect-foresight variants are never `subscribable`.
- Light theme for any visual.
- Distrust long backtests; weight the live out-of-sample period.

**Reporting standard**

Every claim carries its uncertainty. If a result rests on a handful of observations, say so
in the same sentence as the number. Retract cleanly when wrong — withdrawing a finding after
scrutiny is correct behaviour, not failure.

---

## Methodology notes that are easy to get wrong

Each of these was learned the hard way and has caused at least one wrong published number.

- **Exposure scales the arithmetic return, not the log return.** `x = e * (nav[i]/nav[i-1] - 1)`.
  Using `exp(e * log_return)` erases the variance-drag channel entirely.
- **Calendar spines differ per run — never assume 252 rows/year.** Run #14's `btd_daily`
  carries moving NAV on weekend rows and runs at ~364 rows/year; others run at ~250. Derive
  `ppy = (n-1)/years` per run, and compute CAGR from actual dates, never from a row count.
- **Forward-fill onto a common date spine.** `sum(w*rel)` over available rows silently drops
  names on days lacking a price. This bug has been made twice.
- **Permutation-test every timing or episode-based claim.** Re-place the rule's episodes at
  random N times; report the percentile and p-value.
- **Walk-forward, purged and embargoed, for anything learned.** April→April labels overlap,
  so naive cross-validation leaks.
- **Compare against the incumbent, never against zero.** The benchmark for a new ranker is
  rank-by-discount; for a timing rule it is always-100%.

---

## Reproducing the checks

Every number in these documents was produced by SQL against `pvqflvqfdlmiyjjetfer`. The
queries are quoted inline in each document so a reader can re-run them. Nothing in this
repository writes to the database.

Two gates are worth re-running after any change to a run:

```sql
-- 1. NAV reconciliation: daily NAV at each rebalance-year end vs the registry
with ry_end as (select ry, max(dt) as end_dt from bt.btd_rel where run_id = :run group by ry)
select r.ry, round((d.nav / y.nav_strat - 1) * 100, 4) as gap_pct
from ry_end r
join bt.btd_daily d on d.run_id = :run and d.dt = r.end_dt
join bt.run_years y on y.run_id = :run and y.rebal_year = r.ry
order by 1;

-- 2. Weight audit: returns rows only where weights are broken
select * from bt.v_btd_weight_audit where run_id = :run;
```

Note what gate 1 does and does not prove — see the audit's section on anchoring.
