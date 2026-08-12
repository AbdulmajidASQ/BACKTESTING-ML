# Run #85 — registration record

**`United States (US) · Conc-7 · 3-Factor Score + Size Screen (widened pool, $5B era 2026+)`**
Registered 2026-08-11 · derives from run #14 · stock-only, 100% invested, no options

---

## Headline

| metric | run #85 | run #14 |
|---|---|---|
| CAGR, 30 closed years | **72.57%** | 69.22% |
| Benchmark | 10.29% | 10.29% |
| Max drawdown (daily) | **−56.69%** | −52.74% |
| Annualised volatility | 32.13% | 33.11% |
| Hit rate (profitable picks) | **85.4%** | 81.2% |
| Worst year | **−7.15%** | −7.61% |
| Best year | 308.56% | 292.55% |

Drawdown levels are not directly comparable — #85's spine is trading days only (251.7/year), #14's
carries weekend rows with interpolated NAV (~364/year). The like-for-like comparison on the same
reconstruction gave **−60.66% for the incumbent rule against −56.69% for this one**.

## What changed from #14 — exactly two things

Everything else is identical: the universe, every quality gate, every market-cap floor, the dedup
rule, the April rebalance, equal weighting, and the 7-normal / 5-recovery position count.

**1. The candidate pool is widened.** The forward-growth gate drops from `> 25` to `> 0`, taking
closed-year candidates from 380 to 835 name-years. Widening *by itself costs 1.58 CAGR points* —
it is done because it doubles the open selection headroom, from 21.65 to 44.98 points.

**2. Selection ranks on three factors instead of one, after a size screen.** The largest quarter of
each year's candidates by total assets are set aside, then the rest are ranked on an equal-weight
z-score of discount + forward growth + gross-profit-to-assets.

## How to calculate the score

Once a year, on 1 April, after every existing gate has run:

1. **Set aside the largest quarter.** Sort the surviving candidates by total assets and drop the
   top 25%. If fewer than 7 remain, fill the shortfall from the excluded names in score order.
2. **Standardise each factor within that year.** For each of the three measures, compute
   `(value − average of this year's candidates) ÷ standard deviation of this year's candidates`.
   This converts a percentage, a growth rate and a ratio onto one common scale so they can be
   added. A missing value scores 0 — neutral, neither helping nor hurting.
3. **Add the three standardised scores** with equal weight.
4. **Take the top 7** (or top 5 in a recovery year), equal weight, hold to the next 1 April.

The three factors:

| factor | column | question it asks |
|---|---|---|
| discount | `dcf_discount_percent` | how cheap against its own fair value? |
| forward growth | `eps_forward_cagr` | how fast are profits expected to grow? |
| gross profitability | `Gross-Profit-to-Asset %` | how much gross profit per dollar of assets? |

Gross-profit-to-assets is the new leg. It comes from `public."AnnualCommonSizeRatios"`, joined via
`apex_screening_master.reporting_period_id → AnnualReportPeriods.annual_report_id` — a link that
resolves at 150,224 of 150,224 rows and had never been used by any run.

## The evidence

| test | result |
|---|---|
| permutation, random exclusion sets | 0 of 30 beat the size screen, **p ≤ 0.033** |
| permutation, random 3-feature scores | 0 of 60 beat 73.57, **p ≤ 0.017** |
| cross-market size effect | negative in **21 of 24 markets**, 491 market-years |
| excluding the best window (1996–2000) | edge falls +5.42 → **+1.92**, stays positive |
| last 20 years only | **+3.14** |
| ablation | all three legs needed: best pair only +0.86 |

Honest expected effect: **+2 to +3 CAGR points**, not the +5.4 headline.

## Known weakness — read before judging it

**The size screen fails when mega-caps lead.** It lost **12.45 CAGR points over 2001–2005** and
3.38 over 2016–2020. It is losing now: this run trailed #14 by **36.42 points in 2025** and by
**30.11 in the live 2026 year to date**.

The 2026 books show why:

| | run #85 | run #14 |
|---|---|---|
| holdings | SNDK, STX, VICR, WDC, SMTC, CRDO, ALGM | SNDK, MU, STX, WDC, CRDO, AMD, MRVL |
| live return to 2026-08-11 | 72.55% | 102.66% |

The screen excludes MU, AMD and MRVL — and also **NVDA, which carries the highest three-factor
score in the 2026 pool at 3.46** but sits in the largest quartile by assets.

**Multi-year underperformance is the expected cost of this approach, not evidence it is broken.**
Recorded as run event seq 3 so it is not rediscovered and misread later.

## Registration completeness

| checklist item | status |
|---|---|
| `bt.runs` with full reproducible `config` | ✅ |
| `bt.run_years`, 31 rows incl. live year | ✅ |
| `bt.picks`, 192 rows | ✅ |
| Daily NAV layer: `btd_books` 192 · `btd_px` · `btd_rel` 47,302 · `btd_daily` 7,637 | ✅ |
| `bt.catalog`, subscriber-facing, 6 caveats | ✅ draft, not subscribable |
| `bt.run_events`, 4 events | ✅ |
| `bt.run_options` | N/A — stock only, no options |
| Live year populated with documented basis | ✅ |
| NAV reconciliation | ✅ 30/30, worst gap **0.022%** |
| Weight audit | ✅ **0 exceptions** |

## Two defects found during the build and fixed rather than shipped

**1. Return-basis conflict.** The first build took annual returns from `apex.total_return_pct` while
the daily layer came from actual price paths. They disagreed by up to 11.6 points in individual
years and **all 30 years failed** the 0.1% reconciliation. Resolved by rebuilding `run_years`,
`picks` and `btd_books` on the price-path basis — 1 April to 1 April adjusted closes — which is
verifiable and reproducible. The two bases give almost the same 30-year figure (72.57 vs 72.40), so
this cost nothing and bought a genuine reconciliation instead of a tautological one.

**2. Six phantom dates.** US market holidays — 2025-01-01, MLK Day, Presidents' Day and two others
— where a single name carried a stray price row while the rest of the book correctly had none,
producing `UNDER_POPULATED` weight-audit exceptions. Removed.

## Status and next steps

**Draft, not subscribable.** No live capital has been run on this.

1. Observe live against #14 — both now carry a 2026 book on the same restated basis, so the
   comparison accumulates from here.
2. Re-measure when the point-in-time universe is rebuilt (M4). The survivorship limitation applies
   to both runs equally, so the *difference* should survive; the levels will not.
3. The bar for the learning-to-rank model (M2.4) is now **72.57**, not the incumbent.
