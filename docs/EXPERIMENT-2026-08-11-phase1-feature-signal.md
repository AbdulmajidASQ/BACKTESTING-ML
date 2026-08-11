# Experiment — Phase 1: does the orthogonal feature axis carry signal?

**Date:** 2026-08-11 · Project `pvqflvqfdlmiyjjetfer`
**Question:** before fitting any model, do the newly-joined fundamental features carry
cross-sectional signal on run #14's candidate pool — and does any fixed formula built from them
beat the incumbent out of sample?

**Answer:** one validated improvement (a size tilt), one strong-looking feature that failed
validation (gross profitability), and several negative results. The validated improvement is also
the one most exposed to the panel's known survivorship bias — see §5.

---

## Standing assumption

Survivorship bias is accepted as a known limitation by owner decision and is not treated as
blocking. All arms run on the same pool, so **differences between arms are far more robust than
levels**. No figure here is a forecast.

---

## 1. The feature table

Built `bt.ml_us14_pool_v1` — the widened candidate pool joined to the `Annual*` fundamental family
via `apex_screening_master.reporting_period_id`. RLS and revoke applied in the same statement batch
as the create, per house standard; table comment records the caveats.

Acceptance tests, all passing:

| test | required | actual |
|---|---|---|
| widened pool rows | 835 | **835** |
| incumbent subset (`fwd > 25`) | 380 | **380** |
| distinct `(ry, ticker)` — no fan-out | 835 | **835** |
| look-ahead rows exposed, not filtered | — | 58 flagged, 8 undated |
| coverage: Sloan, ROIC−WACC, GP/assets, SBC, buyback, EV/EBITDA, Beneish | — | **100%** |
| coverage: Piotroski | — | 99.3% |

All five source tables are 1:1 on `annual_report_id` (144,068 / 136,763 / 142,657 / 150,758 /
145,077 rows, each equal to its distinct id count), so the join cannot fan out.

## 2. Signal diagnostic

Within-year Spearman against realised forward return, 26 years with at least 8 names, look-ahead
rows excluded. This is the same diagnostic the project's existing feature study used.

| feature | mean ρ | positive years | note |
|---|---|---|---|
| `fwd` | **+0.181** | 23/26 (88%) | incumbent gate; reconfirms the prior study's +0.192, 23/29 |
| `total_assets` | **−0.166** | 6/26 (23%) | **size — smaller is better** |
| `disc` | +0.160 | 19/26 (73%) | incumbent rank key |
| `book_value_ps` | −0.149 | 9/26 | size family |
| `gross_profit_to_asset_pct` | **+0.147** | 19/26 (73%) | **new, orthogonal to valuation** |
| `current_price` | −0.134 | 6/26 (23%) | size / price family |
| `mcap_m` | −0.121 | 9/26 | size family |
| `piotroski_f` | +0.119 | 16/26 (62%) | new |
| `capex` | +0.083 | 16/26 | |
| `ev_to_ebitda` | +0.079 | 16/26 | sign is counter-intuitive; likely a growth proxy in an already-cheap universe |
| `altman_z` | +0.062 | 13/26 (50%) | coin flip |

**Two findings.** First, `gross_profit_to_asset_pct` — Novy-Marx gross profitability — is
essentially tied with the incumbent rank key on both strength and consistency, and it is a pure
quality measure independent of valuation. It was sitting one join away and unused.

Second, a **cluster of scale variables all point the same way**: `total_assets`, `book_value_ps`,
`current_price`, `mcap_m`, `net_income`, `operating_cash_flow`. `total_assets` at −0.166 with only
23% positive years is more consistent than `disc` itself. Notably, size was *weak* in the project's
earlier study on the incumbent pool (`mcap` −0.043, 14/29). **Widening the pool surfaced it.**

**Negative results, recorded so they are not re-tested:** `sloan_ratio_pct` (accruals),
`beneish_m`, `buyback_yield_pct`, `shareholder_yield_pct`, `sbc`, `roic_less_wacc`,
`cash_conversion_cycle` and `deg_operating_leverage` all fell outside the top 25 by absolute mean
ρ. Accruals in particular were a prior expectation and did not deliver.

## 3. In-sample rule test

Equal weight, `top_n` by regime, look-ahead rows excluded, z-scores computed within year, null
z-scores coalesced to 0 (neutral).

| rule | CAGR, 30 years |
|---|---|
| control — incumbent rule on the widened pool | 68.16 |
| `z(disc) − z(ln assets)` | **71.65** |
| `z(disc) + z(gpa)` | 70.22 |
| `z(disc) + z(fwd) + z(gpa) − z(ln assets)` | 69.68 |
| `z(disc) + z(gpa) − z(ln assets)` | 69.29 |
| plus `z(piotroski)` | **65.90** |

Piotroski *hurts* when added to a composite despite a positive standalone ρ.

These are in-sample numbers produced by choosing among six rules on the same thirty years. The
project's own history says that is exactly how the previous attempt failed — walk-forward ranker
selection scored 63.66 against an incumbent of 67.67. **In-sample results here are reported for
completeness and should not be acted on.**

## 4. Out-of-sample test — the decisive one

Single split, no annual re-selection: fit on 1996–2010, freeze, apply to 2011–2025.

| rule | train 1996–2010 | **test 2011–2025** |
|---|---|---|
| incumbent (disc / fwd by regime) | 64.52 | 71.88 |
| **`z(disc) − z(ln assets)`** | **67.43 (+2.91)** | **75.97 (+4.09)** |
| `z(disc) + z(gpa)` | 61.69 (**−2.83**) | 79.19 (+7.31) |
| `z(disc) + z(gpa) − z(ln assets)` | 64.97 (+0.45) | 73.72 (+1.84) |

The incumbent itself scores 64.52 in the first half and 71.88 in the second, so the two periods are
not comparable and only **within-split deltas** carry meaning.

### The result

**`disc − size` is the only rule that beat the incumbent in both halves**: +2.91 in the fitting
period and **+4.09 out of sample**, with the sign consistent across both and consistent with the
independent Spearman diagnostic. That is a genuine, validated improvement — modest, but real.

### The trap, which is the more instructive half

**`disc + gpa` was the worst rule on train (−2.83) and the best on test (+7.31).** Selected
honestly on the first half, it would have been rejected. Adopting it now on the strength of its
test-half number is fitting the holdout, and its +7.31 is not evidence of anything. Gross
profitability has a strong standalone ρ and *still* failed validation — a clean demonstration of
why the project's own guidance is to freeze one model rather than pick a winner per period.

I am recording it as **unproven**, not as a discovery.

## 5. The caveat that matters most

**The one validated finding is the one most exposed to the accepted survivorship limitation.**

A size tilt says: prefer smaller companies. Smaller companies are also the ones that go bankrupt,
get delisted and disappear — and those are precisely the rows the panel is missing
([`AUDIT-2026-08-11-survivorship.md`](AUDIT-2026-08-11-survivorship.md): 96.9% of 1996's names
survive into 2024–26; no pick in 193 ever lost more than 45.95%). A small-cap tilt measured on a
universe with the failures deleted will look better than it is, and the bias operates in exactly
the direction of the finding.

This does not invalidate the result — the out-of-sample confirmation is real within the data we
have — but it means **the size tilt is the single finding here that most needs re-testing on a
point-in-time panel before it is deployed**. `gross_profit_to_asset_pct`, by contrast, is a
quality measure and is far less exposed to that bias; its failure to validate is more likely to be
genuine.

## 6. Conclusions and next steps

1. **Adopt provisionally:** a size tilt on the widened pool, `z(disc) − z(ln total_assets)`.
   +4.09 out of sample on 15 years, sign-consistent with the diagnostic. Flag it for re-test on a
   rebuilt universe before live deployment.
2. **Do not adopt:** gross profitability, Piotroski, accruals, Beneish, buyback yield, ROIC−WACC.
   Recorded as tested and not proven, so they are not re-run.
3. **Proceed to Phase 2** — the pairwise learning-to-rank model — with the bar set at the
   incumbent on the same split, and with `disc − size` as the new control to beat rather than
   `disc` alone. If a fitted model cannot beat a two-term z-score formula out of sample, it should
   not be deployed.
4. **Sample size discipline:** 15 test years, one split, one market. Everything above is a
   direction, not a calibrated estimate. Multi-split and leave-one-market-out validation on the
   global panel is the next validation step, not more feature search on this one.
