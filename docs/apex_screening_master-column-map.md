# `public.apex_screening_master` — column map

**What the flagship actually uses, and what it does not.**

Date: 2026-08-11 · Project `pvqflvqfdlmiyjjetfer` · verified by SQL, read-only.

---

## 1. The table

`public.apex_screening_master` is the master screening file — one row per company per fiscal
year, carrying prices, fundamentals, forward estimates, derived valuation metrics and the
realised forward return.

| property | value |
|---|---|
| Columns | **213** (max ordinal 252 — 39 positions are historically dropped columns) |
| Rows | **150,224** |
| Distinct tickers | 10,311 |
| `performance_year` range | 1979 – 2027 (49 distinct values) |
| Rows with `performance_year` null | 6.1% — **invisible to the flagship**, which joins on it |

Note the ordinal gaps (10, 20, 116–119, 146–149, 191, 194–211, 213–222). Those columns were
dropped in earlier revisions; the numbering was never compacted.

---

## 2. How the flagship consumes it

Run #14's closed years (1996–2025) are generated **entirely** by `bt.v_us14_cand`, which
reproduces `bt.picks` 186/186 exactly including rank. That view is the definitive answer to
"what is used": it is the only path from this table into the flagship.

```sql
FROM apex_screening_master a
  JOIN bt.run_years r    ON r.rebal_year = a.performance_year AND r.run_id = 14
  JOIN public.companies c ON c.ticker = a.ticker
```

Everything the view touches is listed in §3. **Everything else in the table — 192 of 213
columns — is not read by the flagship at any point.**

Two columns come from outside this table:

| column | source | role |
|---|---|---|
| `is_adr` | `public.companies` | universe gate — ADRs excluded |
| `regime` | `bt.run_years` | selects the NORMAL or RECOVERY gate set and `top_n` |

---

## 3. The 21 columns the flagship reads directly

| # | column | role in the flagship | gate / use |
|---|---|---|---|
| 29 | `performance_year` | join key; also derives the screening year | `sy = performance_year - 1` drives the market-cap floor |
| 3 | `ticker` | identity; join to `companies` for the ADR test | — |
| 4 | `company_name` | dedup key | one line per company, largest `market_cap` kept |
| 8 | `sector` | universe gate | `is not null and <> 'Energy'` |
| 9 | `industry` | universe gate | not in the 14 excluded industries |
| 30 | `current_eps` | universe gate | `> 0` |
| 98 | `eps_f1` | universe gate | `> 0`, `< eps_f2`, and `current_price >= 2 * eps_f1` |
| 99 | `eps_f2` | universe gate | `> eps_f1`, `> eps_2yr_ago`, `> eps_3yr_ago` |
| 54 | `eps_2yr_ago` | universe gate | `eps_f2 > eps_2yr_ago`, **non-null required** |
| 58 | `eps_3yr_ago` | universe gate | `eps_f2 > eps_3yr_ago`, **non-null required** |
| 35 | `last_year_operating_income` | universe gate | `coalesce(OI,0) > 0 OR coalesce(ebitda,0) > 0` |
| 38 | `ebitda` | universe gate | same clause as above |
| 14 | `current_price` | P/F1 gate and EMA gate | `>= 2 * eps_f1`; `> ema_200w_rebal` |
| 15 | `market_cap` | era floor gate; dedup tiebreak | see floor table below. **Denominated in USD millions** |
| 152 | `dcf_discount_percent` | value gate **and rank key** | NORMAL `> 25`, RECOVERY `> 75`; ranks NORMAL |
| 108 | `eps_growth_f1` | growth gate | `> 25` in both regimes |
| 120 | `eps_forward_cagr` | growth gate **and rank key** | `> 25` in both regimes; ranks RECOVERY |
| 139 | `net_debt_to_ebitda` | leverage gate | NORMAL `< 3`, RECOVERY `< 5` |
| 141 | `current_ratio` | leverage gate | NORMAL `> 1.0`, RECOVERY `> 0.8` |
| 246 | `ema_200w_rebal` | trend gate, point-in-time | NORMAL: non-null and `current_price > ema`. **Excluded in RECOVERY** |
| 28 | `total_return_pct` | the realised April→April return | the label; flows into `picks.fwd_return` |

### The market-cap floor

Keyed on the **screening** year (`performance_year − 1`), not the rebalance year. Getting this
wrong wrongly excludes ESE (2000, mcap 210) and DDD (2010, mcap 309).

| screening year | floor (USD millions) |
|---|---|
| ≤ 1999 | 50 |
| 2000–2009 | 250 |
| 2010–2019 | 500 |
| 2020–2024 | 1,000 |
| 2025+ | 5,000 |

### The 14 excluded industries, verbatim

Drug Manufacturers · Biotechnology · Medical Devices & Instruments · Medical Diagnostics &
Research · Medical Distribution · Healthcare Providers & Services · Healthcare Plans · Banks ·
Insurance · Metals & Mining · Oil & Gas · Other Energy Sources · Transportation · REITs

---

## 4. Indirect inputs — columns that feed the used ones upstream

These are not referenced by the generator, but the derived columns it *does* read are computed
from them elsewhere in the pipeline. Changing them changes the flagship.

| derived column used | computed from | verification |
|---|---|---|
| `dcf_discount_percent` | `dcf_fair_value`, `current_price` | `(dcf_fair_value − current_price) / dcf_fair_value × 100` — **16,848 / 16,848 exact** on 2015–2025 |
| `net_debt_to_ebitda` | `net_debt`, `ebitda` | `net_debt / ebitda` — **16,848 / 16,848 exact** |
| `current_ratio` | `current_assets`, `current_liabilities` | `current_assets / current_liabilities` — **16,848 / 16,848 exact** |
| `market_cap` | `current_price`, `shares_outstanding` | product matches at ratio 1.0 (mean and median) — so **`shares_outstanding` is in millions of shares** |
| `eps_growth_f1` | `eps_f1`, `current_eps` | `(eps_f1 / current_eps − 1) × 100` — **15,521 / 16,848 = 92.1%**; the remainder use a rolled basis |
| `eps_forward_cagr` | `eps_f1`, `eps_f2`, `eps_f3`, `eps_basis` | basis *reported* → `(F3/F1)^(1/2) − 1`; basis *rolled* → `F2/F1 − 1`; clamped to `[−100, +10000]` (audited in event 164) |
| `net_debt` | `total_debt`, `cash_and_equivalents` | `total_debt − cash_and_equivalents` matches on only **5,290 / 16,848 = 31.4%** of rows — see §7 |

Also indirect: `company_id` (join key upstream), `calendar_year` (the screening-year concept),
`eps_forward_cagr_period` and `estimate_shift_applied` (provenance of the forward figures).

---

## 5. The funnel — where the 150k rows go

Closed years 1996–2025 only.

| step | gate | rows surviving | cut |
|---|---|---|---|
| 0 | rows with `performance_year` 1996–2025 | 122,223 | — |
| 1 | `companies.is_adr = false` | 104,014 | −18,209 |
| 2 | `sector is not null and <> 'Energy'` | 99,754 | −4,260 |
| 3 | 14 excluded industries | 55,490 | −44,264 |
| 4 | EPS chain (`current_eps>0`, `eps_f1>0`, `eps_f1<eps_f2`, `eps_f2>eps_2yr_ago`, `eps_f2>eps_3yr_ago`) | 13,234 | −42,256 |
| 5 | OI-or-EBITDA `> 0`, and `current_price >= 2 × eps_f1` | 11,440 | −1,794 |
| 6 | era market-cap floor | 10,221 | −1,219 |
| 7 | dedup by company name | 9,988 | −233 |
| 8 | `eps_growth_f1 > 25` | 3,464 | −6,524 |
| 9 | `eps_forward_cagr > 25` | **702** | **−2,762** |
| 10 | discount gate (25 / 75) | 563 | −139 |
| 11 | leverage gates | 423 | −140 |
| 12 | EMA gate (NORMAL only) | **380** | −43 |

380 name-years is the flagship's candidate pool, matching the figure used in the project's
selection-headroom study.

### A correction to the documented ML priority

The project's research notes name pool width as the place the remaining selection alpha lives
— ranking adds **+19.8 points/year** in years where the pool is at least 2× the book, against
**+3.9** when it is narrower — and propose relaxing `disc > 25` toward `disc > 0` to widen it.

**That targets the wrong gate.** Measured on the same 30 years:

| variant | pool (name-years) | years with pool ≥ 2×n | years with no choice at all |
|---|---|---|---|
| incumbent | 380 | **15 / 30** | 4 |
| relax `disc > 25` → `disc > 0` | 392 (+3%) | **15 / 30** — no change | 4 |
| relax `g1 > 25` → `g1 > 0` | 620 (+63%) | — | — |
| relax `fwd > 25` → `fwd > 10` | 776 | — | — |
| relax `fwd > 25` → `fwd > 0` | **835 (+120%)** | **26 / 30** | **0** |
| all three value/growth gates off | 3,068 (8×) | — | — |

The discount gate is nearly non-binding by the time it fires: `fwd > 25` has already cut the
field by 80% (3,464 → 702). **`eps_forward_cagr` is the gate that actually controls pool
width**, and relaxing it is what removes the four no-choice years (1996, 2000, 2011, 2024) in
which no selection decision existed at all.

This is a pool-size result only. It is a precondition for ranking to add value, **not** a
return claim — widening the funnel and measuring whether ranking still adds ~+19.8 points is
the experiment, and it has not been run.

---

## 6. What is not used

192 of 213 columns. By family:

| family | example columns | why it is not used |
|---|---|---|
| Revenue mirror of every EPS metric | `revenue_f1..f4`, `revenue_growth_f1/f2`, `revenue_cagr_1..5yr`, `revenue_forward_cagr`, `revenue_2..5yr_ago`, `revenue_historical_cagr` | the strategy is EPS-based throughout |
| Operating-income and gross-profit mirrors | `operating_income_cagr_1..5yr`, `gross_profit_cagr_1..5yr`, `*_2..5yr_ago`, `*_historical_cagr` | only `last_year_operating_income` is read, as a liveness test |
| Peer-relative valuation (48 columns) | `pe_ratio_*_5yr_avg`, `peg_*`, `ps_discount_from_*`, `pb_discount_from_*` | the flagship values on DCF discount only |
| Composite scores | `comprehensive_value_score`, `financial_health_score`, `dcf_quality_score`, `forward_growth_quality_score`, `forward_growth_consistency_score` | never referenced by any run #14 object |
| A pre-existing selection ranking | `final_selection_rank`, `quality_strategy_applied`, `return_category`, `return_category_label`, `return_percentile`, `year_p95_threshold`, `year_p75_threshold` | the flagship computes its own rank; this parallel ranking is unrelated |
| Momentum and price history | `price_return_1m/3m/6m/1y`, `price_52w_high/low`, `volume_avg_3m` | timing signals — closed as a research direction |
| Other EMAs | `ema_50w`, `ema_100w`, `ema_50w_rebal`, `ema_100w_rebal`, `ema_200w` | only the point-in-time `ema_200w_rebal` is used; plain `ema_200w` is **not** point-in-time |
| Cash flow and full balance sheet | `operating_cash_flow`, `free_cash_flow`, `total_assets`, `total_equity`, `total_liabilities`, `cogs` | no cash-flow screen exists |
| Returns and margins | `roe_percent`, `roic_percent`, `roa_percent`, `roce_percent`, all `*_margin_percent` | no quality screen exists |
| Live-quote block | `live_price`, `live_price_date`, `live_dcf_discount_percent`, `prev_close`, `daily_change_pct` | 95–98% null; the live year uses a different source entirely (§8) |
| Liquidity | `adv_usd_rebal` | used by option-availability runs (#73+), not by #14 |
| Bookkeeping | `id`, `fiscal_year`, `fiscal_month`, `reporting_period_id`, `created_at`, `last_updated` | — |

**There is no `data_flag` column in this table.** The flagship catalogue's
`universe_gates` claims *"data_flag quarantine applied"*; no such column exists and the
generator applies no such filter. Recorded as a defect in the run #14 audit.

---

## 7. Data-quality notes

- **`dcf_discount_percent` is 71.4% null** — the flagship's rank key exists for barely a
  quarter of the file. `eps_forward_cagr` is 67.6% null, `eps_f1` 54.6%. The scarcity of the
  forward-estimate and DCF block, not the gate thresholds, is what makes the candidate pool
  small in early years.
- **`net_debt` does not reproduce as `total_debt − cash_and_equivalents`** on 68.6% of rows
  (5,290 of 16,848 match on 2015–2025). Its definition is set upstream and is not documented
  here. It feeds `net_debt_to_ebitda`, a live gate, so this is worth pinning down before any
  work that depends on leverage.
- **`eps_basis` is 92.8% null** yet governs which formula produces `eps_forward_cagr`. Rows
  with a null basis take the standard branch.
- Ten columns are ≥95% null and effectively dead: `fundamentals_synced_at`, `eps_basis`,
  `quarters_reported`, `current_fiscal_year_fh`, `live_price`, `live_price_date`,
  `live_dcf_discount_percent`, `live_dcf_discount_absolute`, `prev_close`, `daily_change_pct`,
  `split_factor_since_rebal` (99.8%).
- **Point-in-time integrity of this file is untested.** The companion panel `bt.cand_stage`
  grows monotonically in distinct companies (3,060 in 2000 → 23,651 in 2025); the same question
  applies here, where distinct tickers number 10,311 across 1979–2027. Until several hundred
  names listed in 2010 and since delisted are confirmed present in this table for 2010, every
  backtest number derived from it should be treated as possibly inflated. This is the standing
  blocking item before any model is trained.

---

## 8. The live year does not use this file

For rebalance year 2026 the flagship does **not** read `apex_screening_master`. Per
`runs.config.current_cycle_source`, the live cycle is built on `public.mizan_rebalance_metrics`
— a **materialized view of 12 columns**:

`company_id`, `ticker`, `cfy`, `eps_basis`, `quarters_reported`, `current_eps`, `eps_f1`,
`eps_f2`, `eps_f3`, `eps_growth_f1`, `eps_forward_cagr`, `dcf_fair_value`

It carries **no** `current_price`, `market_cap`, `net_debt_to_ebitda`, `current_ratio` or
`ema_200w_rebal`, so those gates are sourced elsewhere for the live year. Discount is computed
as `(dcf_fair_value − current_price) / dcf_fair_value` (verified 7/7 in rank order, event 440).

The live candidate pool is `bt.us14_cand26` — a **frozen base table of 31 rows**, not a view,
commented *"Frozen run #14 candidate set for the 2026 live year."* `bt.v_us14_cand_all` is
`v_us14_cand` restricted to 1996–2025, `UNION ALL` that frozen table.

Consequence worth knowing: because `run_years.regime` for 2026 reads
`'LIVE-OPEN ($5B, strict Normal)'` rather than `'NORMAL'`, `bt.v_us14_cand` treats it as
non-RECOVERY and emits **29 candidate rows for 2026 from apex that the run does not use**. Two
2026 pools coexist — 29 from the master file, 31 frozen from the live source. Only the frozen
one is real.

---

## 9. Full column list — all 213

Null percentages are from `pg_stats` over the whole table, all years.
**USED — direct** = read by `bt.v_us14_cand`. *Used — indirect* = feeds a directly-used
derived column. Everything else is untouched by the flagship.

| # | column | type | null % | flagship |
|---|---|---|---|---|
| 1 | `id` | integer | 0.0% | Not used |
| 2 | `company_id` | integer | 0.0% | Used — indirect |
| 3 | `ticker` | varchar | 0.0% | **USED — direct** |
| 4 | `company_name` | varchar | 0.0% | **USED — direct** |
| 5 | `fiscal_year` | integer | 0.0% | Not used |
| 6 | `calendar_year` | integer | 0.0% | Used — indirect |
| 7 | `reporting_period_id` | integer | 0.0% | Not used |
| 8 | `sector` | varchar | 2.6% | **USED — direct** |
| 9 | `industry` | varchar | 2.6% | **USED — direct** |
| 11 | `last_updated` | timestamp | 0.0% | Not used |
| 12 | `created_at` | timestamp | 0.0% | Not used |
| 13 | `fiscal_month` | integer | 0.2% | Not used |
| 14 | `current_price` | numeric | 36.2% | **USED — direct** |
| 15 | `market_cap` | numeric | 41.2% | **USED — direct** |
| 16 | `shares_outstanding` | bigint | 41.2% | Used — indirect |
| 17 | `ema_50w` | numeric | 35.7% | Not used |
| 18 | `ema_100w` | numeric | 36.7% | Not used |
| 19 | `ema_200w` | numeric | 38.0% | Not used |
| 21 | `price_52w_high` | numeric | 35.5% | Not used |
| 22 | `price_52w_low` | numeric | 35.5% | Not used |
| 23 | `volume_avg_3m` | bigint | 36.3% | Not used |
| 24 | `price_return_1y` | numeric | 40.5% | Not used |
| 25 | `price_return_6m` | numeric | 38.3% | Not used |
| 26 | `price_return_3m` | numeric | 37.1% | Not used |
| 27 | `price_return_1m` | numeric | 36.5% | Not used |
| 28 | `total_return_pct` | numeric | 37.0% | **USED — direct** |
| 29 | `performance_year` | integer | 6.1% | **USED — direct** |
| 30 | `current_eps` | numeric | 3.8% | **USED — direct** |
| 31 | `last_year_eps` | numeric | 9.7% | Not used |
| 32 | `current_revenue` | numeric | 7.4% | Not used |
| 33 | `last_year_revenue` | numeric | 13.4% | Not used |
| 34 | `operating_income` | numeric | 25.8% | Not used |
| 35 | `last_year_operating_income` | numeric | 30.7% | **USED — direct** |
| 36 | `gross_profit` | numeric | 28.5% | Not used |
| 37 | `last_year_gross_profit` | numeric | 33.0% | Not used |
| 38 | `ebitda` | numeric | 22.5% | **USED — direct** |
| 39 | `net_income` | numeric | 4.5% | Not used |
| 40 | `total_debt` | numeric | 0.3% | Used — indirect |
| 41 | `cash_and_equivalents` | numeric | 26.2% | Used — indirect |
| 42 | `current_assets` | numeric | 25.0% | Used — indirect |
| 43 | `current_liabilities` | numeric | 25.3% | Used — indirect |
| 44 | `total_assets` | numeric | 0.9% | Not used |
| 45 | `total_equity` | numeric | 2.2% | Not used |
| 46 | `roe_percent` | numeric | 4.3% | Not used |
| 47 | `roic_percent` | numeric | 22.3% | Not used |
| 48 | `roa_percent` | numeric | 11.9% | Not used |
| 49 | `roce_percent` | numeric | 28.8% | Not used |
| 50 | `gross_margin_percent` | numeric | 26.4% | Not used |
| 51 | `operating_margin_percent` | numeric | 26.4% | Not used |
| 52 | `net_margin_percent` | numeric | 5.0% | Not used |
| 53 | `ebitda_margin_percent` | numeric | 23.3% | Not used |
| 54 | `eps_2yr_ago` | numeric | 15.6% | **USED — direct** |
| 55 | `revenue_2yr_ago` | numeric | 15.8% | Not used |
| 56 | `operating_income_2yr_ago` | numeric | 35.1% | Not used |
| 57 | `gross_profit_2yr_ago` | numeric | 35.3% | Not used |
| 58 | `eps_3yr_ago` | numeric | 20.8% | **USED — direct** |
| 59 | `revenue_3yr_ago` | numeric | 21.0% | Not used |
| 60 | `operating_income_3yr_ago` | numeric | 39.4% | Not used |
| 61 | `gross_profit_3yr_ago` | numeric | 39.6% | Not used |
| 62 | `eps_4yr_ago` | numeric | 25.8% | Not used |
| 63 | `revenue_4yr_ago` | numeric | 25.9% | Not used |
| 64 | `operating_income_4yr_ago` | numeric | 43.3% | Not used |
| 65 | `gross_profit_4yr_ago` | numeric | 43.4% | Not used |
| 66 | `eps_5yr_ago` | numeric | 30.5% | Not used |
| 67 | `revenue_5yr_ago` | numeric | 30.6% | Not used |
| 68 | `operating_income_5yr_ago` | numeric | 47.1% | Not used |
| 69 | `gross_profit_5yr_ago` | numeric | 47.2% | Not used |
| 70 | `eps_cagr_1yr` | numeric | 18.8% | Not used |
| 71 | `eps_cagr_2yr` | numeric | 47.0% | Not used |
| 72 | `eps_cagr_3yr` | numeric | 50.0% | Not used |
| 73 | `eps_cagr_4yr` | numeric | 52.4% | Not used |
| 74 | `eps_cagr_5yr` | numeric | 55.1% | Not used |
| 75 | `revenue_cagr_1yr` | numeric | 21.2% | Not used |
| 76 | `revenue_cagr_2yr` | numeric | 26.0% | Not used |
| 77 | `revenue_cagr_3yr` | numeric | 30.1% | Not used |
| 78 | `revenue_cagr_4yr` | numeric | 33.9% | Not used |
| 79 | `revenue_cagr_5yr` | numeric | 38.0% | Not used |
| 80 | `operating_income_cagr_1yr` | numeric | 35.9% | Not used |
| 81 | `operating_income_cagr_2yr` | numeric | 59.4% | Not used |
| 82 | `operating_income_cagr_3yr` | numeric | 61.5% | Not used |
| 83 | `operating_income_cagr_4yr` | numeric | 63.4% | Not used |
| 84 | `operating_income_cagr_5yr` | numeric | 65.2% | Not used |
| 85 | `gross_profit_cagr_1yr` | numeric | 39.1% | Not used |
| 86 | `gross_profit_cagr_2yr` | numeric | 43.4% | Not used |
| 87 | `gross_profit_cagr_3yr` | numeric | 46.8% | Not used |
| 88 | `gross_profit_cagr_4yr` | numeric | 49.9% | Not used |
| 89 | `gross_profit_cagr_5yr` | numeric | 53.1% | Not used |
| 90 | `eps_historical_cagr` | numeric | 19.5% | Not used |
| 91 | `revenue_historical_cagr` | numeric | 20.2% | Not used |
| 92 | `operating_income_historical_cagr` | numeric | 36.1% | Not used |
| 93 | `gross_profit_historical_cagr` | numeric | 38.1% | Not used |
| 94 | `eps_historical_cagr_period` | integer | 17.8% | Not used |
| 95 | `revenue_historical_cagr_period` | integer | 18.8% | Not used |
| 96 | `operating_income_historical_cagr_period` | integer | 34.1% | Not used |
| 97 | `gross_profit_historical_cagr_period` | integer | 36.6% | Not used |
| 98 | `eps_f1` | numeric | 54.6% | **USED — direct** |
| 99 | `eps_f2` | numeric | 52.4% | **USED — direct** |
| 100 | `eps_f3` | numeric | 51.8% | Used — indirect |
| 101 | `eps_f4` | numeric | 54.6% | Not used |
| 102 | `revenue_f1` | numeric | 57.2% | Not used |
| 103 | `revenue_f2` | numeric | 55.0% | Not used |
| 104 | `revenue_f3` | numeric | 56.5% | Not used |
| 105 | `revenue_f4` | numeric | 56.7% | Not used |
| 106 | `max_10yr_eps` | numeric | 3.1% | Not used |
| 107 | `estimate_shift_applied` | integer | 50.0% | Used — indirect |
| 108 | `eps_growth_f1` | numeric | 54.5% | **USED — direct** |
| 109 | `eps_growth_f2` | numeric | 54.7% | Not used |
| 110 | `eps_cagr_f1_to_f4` | numeric | 71.4% | Not used |
| 111 | `revenue_growth_f1` | numeric | 59.2% | Not used |
| 112 | `revenue_growth_f2` | numeric | 59.9% | Not used |
| 113 | `revenue_cagr_f1_to_f4` | numeric | 64.8% | Not used |
| 114 | `eps_f2_vs_10yr_max_growth` | numeric | 54.9% | Not used |
| 115 | `eps_f2_vs_10yr_max_multiplier` | numeric | 54.9% | Not used |
| 120 | `eps_forward_cagr` | numeric | 67.6% | **USED — direct** |
| 121 | `eps_forward_cagr_period` | integer | 23.4% | Used — indirect |
| 122 | `revenue_forward_cagr` | numeric | 59.5% | Not used |
| 123 | `revenue_forward_cagr_period` | integer | 20.3% | Not used |
| 124 | `forward_growth_quality_score` | numeric | 57.3% | Not used |
| 125 | `forward_growth_consistency_score` | numeric | 60.6% | Not used |
| 126 | `max_5yr_eps` | numeric | 4.0% | Not used |
| 127 | `max_5yr_revenue` | numeric | 5.4% | Not used |
| 128 | `max_5yr_eps_year` | integer | 4.0% | Not used |
| 129 | `max_5yr_revenue_year` | integer | 5.4% | Not used |
| 130 | `eps_f1_vs_5yr_max_growth` | numeric | 55.3% | Not used |
| 131 | `eps_f1_vs_5yr_max_multiplier` | numeric | 55.3% | Not used |
| 132 | `eps_f2_vs_5yr_max_growth` | numeric | 53.3% | Not used |
| 133 | `eps_f2_vs_5yr_max_multiplier` | numeric | 53.3% | Not used |
| 134 | `revenue_f1_vs_5yr_max_growth` | numeric | 58.6% | Not used |
| 135 | `revenue_f1_vs_5yr_max_multiplier` | numeric | 58.6% | Not used |
| 136 | `revenue_f2_vs_5yr_max_growth` | numeric | 56.7% | Not used |
| 137 | `revenue_f2_vs_5yr_max_multiplier` | numeric | 56.7% | Not used |
| 138 | `net_debt` | numeric | 0.2% | Used — indirect |
| 139 | `net_debt_to_ebitda` | numeric | 24.3% | **USED — direct** |
| 140 | `debt_to_equity` | numeric | 8.2% | Not used |
| 141 | `current_ratio` | numeric | 31.1% | **USED — direct** |
| 142 | `quick_ratio` | numeric | 31.1% | Not used |
| 143 | `interest_coverage_ratio` | numeric | 29.3% | Not used |
| 144 | `ebitda_coverage_ratio` | numeric | 28.9% | Not used |
| 145 | `cash_coverage_ratio` | numeric | 72.0% | Not used |
| 150 | `financial_health_score` | numeric | 11.2% | Not used |
| 151 | `dcf_fair_value` | numeric | 67.6% | Used — indirect |
| 152 | `dcf_discount_percent` | numeric | 71.4% | **USED — direct** |
| 153 | `dcf_discount_absolute` | numeric | 71.4% | Not used |
| 154 | `dcf_quality_score` | numeric | 6.1% | Not used |
| 155 | `pe_ratio` | numeric | 41.3% | Not used |
| 156 | `peg_ratio` | numeric | 67.4% | Not used |
| 157 | `price_to_sales` | numeric | 44.9% | Not used |
| 158 | `price_to_book` | numeric | 44.4% | Not used |
| 159 | `pe_ratio_5yr_avg` | numeric | 35.2% | Not used |
| 160 | `peg_ratio_5yr_avg` | numeric | 49.9% | Not used |
| 161 | `price_to_sales_5yr_avg` | numeric | 36.8% | Not used |
| 162 | `price_to_book_5yr_avg` | numeric | 34.8% | Not used |
| 163 | `pe_ratio_industry_5yr_avg` | numeric | 6.3% | Not used |
| 164 | `peg_ratio_industry_5yr_avg` | numeric | 6.9% | Not used |
| 165 | `price_to_sales_industry_5yr_avg` | numeric | 6.3% | Not used |
| 166 | `price_to_book_industry_5yr_avg` | numeric | 6.3% | Not used |
| 167 | `pe_ratio_sector_5yr_avg` | numeric | 6.2% | Not used |
| 168 | `peg_ratio_sector_5yr_avg` | numeric | 6.2% | Not used |
| 169 | `price_to_sales_sector_5yr_avg` | numeric | 6.2% | Not used |
| 170 | `price_to_book_sector_5yr_avg` | numeric | 6.2% | Not used |
| 171 | `pe_ratio_market_5yr_avg` | numeric | 6.2% | Not used |
| 172 | `peg_ratio_market_5yr_avg` | numeric | 6.2% | Not used |
| 173 | `price_to_sales_market_5yr_avg` | numeric | 6.2% | Not used |
| 174 | `price_to_book_market_5yr_avg` | numeric | 6.2% | Not used |
| 175 | `pe_discount_from_company_5yr` | numeric | 42.7% | Not used |
| 176 | `pe_discount_from_industry_5yr` | numeric | 40.7% | Not used |
| 177 | `pe_discount_from_sector_5yr` | numeric | 40.7% | Not used |
| 178 | `pe_discount_from_market_5yr` | numeric | 40.7% | Not used |
| 179 | `peg_discount_from_company_5yr` | numeric | 43.4% | Not used |
| 180 | `peg_discount_from_industry_5yr` | numeric | 42.7% | Not used |
| 181 | `peg_discount_from_sector_5yr` | numeric | 42.7% | Not used |
| 182 | `peg_discount_from_market_5yr` | numeric | 42.7% | Not used |
| 183 | `ps_discount_from_company_5yr` | numeric | 42.0% | Not used |
| 184 | `ps_discount_from_industry_5yr` | numeric | 41.1% | Not used |
| 185 | `ps_discount_from_sector_5yr` | numeric | 41.1% | Not used |
| 186 | `ps_discount_from_market_5yr` | numeric | 41.2% | Not used |
| 187 | `pb_discount_from_company_5yr` | numeric | 42.0% | Not used |
| 188 | `pb_discount_from_industry_5yr` | numeric | 40.9% | Not used |
| 189 | `pb_discount_from_sector_5yr` | numeric | 40.9% | Not used |
| 190 | `pb_discount_from_market_5yr` | numeric | 41.0% | Not used |
| 192 | `pe_attractive_vs_peers` | boolean | 1.5% | Not used |
| 193 | `comprehensive_value_score` | numeric | 11.2% | Not used |
| 212 | `quality_strategy_applied` | varchar | 11.2% | Not used |
| 223 | `final_selection_rank` | integer | 11.2% | Not used |
| 224 | `return_category` | integer | 40.9% | Not used |
| 225 | `return_category_label` | varchar | 40.9% | Not used |
| 226 | `return_percentile` | numeric | 40.9% | Not used |
| 227 | `year_p95_threshold` | numeric | 11.2% | Not used |
| 228 | `year_p75_threshold` | numeric | 11.2% | Not used |
| 229 | `interest_expense` | numeric | 9.5% | Not used |
| 230 | `fundamentals_synced_at` | timestamptz | 95.1% | Not used |
| 231 | `eps_basis` | text | 92.8% | Used — indirect |
| 232 | `quarters_reported` | integer | 97.6% | Not used |
| 233 | `current_fiscal_year_fh` | integer | 97.6% | Not used |
| 234 | `cogs` | numeric | 30.8% | Not used |
| 235 | `operating_expenses` | numeric | 30.8% | Not used |
| 236 | `non_operating_income` | numeric | 30.8% | Not used |
| 237 | `total_liabilities` | numeric | 8.4% | Not used |
| 238 | `operating_cash_flow` | numeric | 9.5% | Not used |
| 239 | `investing_cash_flow` | numeric | 9.5% | Not used |
| 240 | `financing_cash_flow` | numeric | 9.5% | Not used |
| 241 | `free_cash_flow` | numeric | 9.5% | Not used |
| 242 | `live_price` | numeric | 95.5% | Not used |
| 243 | `live_price_date` | date | 95.5% | Not used |
| 244 | `live_dcf_discount_percent` | numeric | 98.1% | Not used |
| 245 | `live_dcf_discount_absolute` | numeric | 98.1% | Not used |
| 246 | `ema_200w_rebal` | numeric | 43.3% | **USED — direct** |
| 247 | `prev_close` | numeric | 95.9% | Not used |
| 248 | `daily_change_pct` | numeric | 95.9% | Not used |
| 249 | `ema_50w_rebal` | numeric | 42.1% | Not used |
| 250 | `ema_100w_rebal` | numeric | 44.3% | Not used |
| 251 | `split_factor_since_rebal` | numeric | 99.8% | Not used |
| 252 | `adv_usd_rebal` | numeric | 40.0% | Not used |

**Totals: 21 direct · 13 indirect · 179 unused.**
