# Experiment — pool widening control, before any model

**Date:** 2026-08-11 · Project `pvqflvqfdlmiyjjetfer` · read-only SQL
**Question:** does widening run #14's candidate pool help or hurt, using the incumbent ranking rule
and nothing else? This is the control the learned ranker must be measured against.

---

## Standing assumption

The survivorship finding in [`AUDIT-2026-08-11-survivorship.md`](AUDIT-2026-08-11-survivorship.md)
is **accepted as a known limitation and not treated as blocking**, by owner decision. Every figure
below is measured on the survivor-only universe and inherits that bias. Because every arm of this
experiment runs on the same pool, the **differences** between arms are far more robust than the
levels. Read the deltas, not the absolutes, and do not quote these CAGRs as forecasts.

One supporting measurement, which narrows the concern for modelling purposes: `bt.cand_stage` —
the 432k-row global panel that would be the training set — carries the same back-filled ticker
list (97.1% of 1996 names survive into 2024–26), **but its return distribution has a real left
tail**: 21,091 labelled outcomes at −50% or worse (6.97%), 4,470 at −75% or worse, 849 at −90% or
worse, and a minimum of −100.0%. Companies that collapsed while remaining listed are present in
force. Distress and quality features therefore have ample signal to learn from; the residual gap is
specifically companies that were delisted and removed.

---

## Setup

The generator was rebuilt from `apex_screening_master` rather than reading `bt.v_us14_cand`, so the
`fwd` threshold could be varied. All other gates are identical to the flagship: core universe,
ADR exclusion, sector and 14-industry exclusions, the EPS chain with non-null backward legs, the
OI-or-EBITDA test, P/F1 ≥ 2, the era market-cap floor keyed on the screening year, dedup by company
name keeping the largest market cap, regime-dependent leverage and the point-in-time 200-week EMA
gate in NORMAL years only. `bt.run_years.regime` is authoritative; `top_n` is 7 NORMAL, 5 RECOVERY.

- **Incumbent pool:** `fwd > 25` → 380 name-years
- **Widened pool:** `fwd > 0`, everything else unchanged → 835 name-years
- Equal weight, returns chained geometrically over the 30 closed years 1996–2025.

**Reconstruction validated.** The incumbent arm returns **69.22**, matching the registered
`runs.cagr` exactly, and the random arm returns **58.55**, matching the figure in the project's
existing selection-headroom study exactly. (That study quotes 67.67 for the actual arm against my
69.22 — a small methodological difference in how the actual selection is chained, not a
discrepancy in the pool.)

---

## Result

| selection rule, 30 closed years | incumbent pool (380) | widened pool (835) |
|---|---|---|
| **actual — top-n by the incumbent rank rule** | **69.22** | **67.64** |
| random draw from the pool | 58.55 | 51.32 |
| perfect foresight — best-n each year | — | 112.62 |
| worst-n each year | — | 3.78 |

### Three readings, in order of importance

**1. Widening alone costs 1.58 points.** 69.22 → 67.64. The incumbent formula does not monetize a
wider pool. Anyone hoping to improve the flagship by simply relaxing `fwd` should stop here — it
makes the strategy slightly worse.

**2. The ranking edge grows by half.** Value added over a random draw from the same pool:

| | edge over random |
|---|---|
| incumbent pool | +10.67 |
| widened pool | **+16.32** |

The same rule extracts 53% more value when there is more to choose from. This corroborates the
project's existing finding that ranking adds +19.8 points/year when the pool is at least twice the
book and +3.9 when it is not — widening on `fwd` takes the count of years meeting that condition
from 15/30 to 26/30 and removes all four years in which no selection decision existed.

**3. The prize roughly doubles.** Span between a random draw and perfect foresight:

| | random | perfect | span | captured | **open** |
|---|---|---|---|---|---|
| incumbent pool | 58.55 | 90.87\* | 32.32 | 10.67 (33%) | 21.65 |
| widened pool | 51.32 | 112.62 | **61.30** | 16.32 (27%) | **44.98** |

\* incumbent perfect-foresight figure from the project's existing study, on the same 380 pool.

Open headroom goes from 21.7 points to **45.0**. The fraction captured is similar in both, but the
base it is a fraction of is nearly twice as large.

---

## Conclusion and decision

**Build the model on the widened pool.** Not because widening improves the strategy — it does not,
it costs 1.58 points — but because it is where the alpha is. A better ranker on the incumbent pool
is chasing at most 21.7 points; on the widened pool it is chasing 45.0.

This also sets the bar precisely. A learned ranker deployed on the widened pool must clear **69.22**
to be worth adopting at all, not 67.64 — it has to pay back the 1.58 points that widening costs
before it earns anything. That is a harder and more honest hurdle than beating the widened-pool
control.

### Next steps

1. Rank-based target: within-year percentile of forward return, not raw return. The raw
   distribution is fat-tailed and a squared-error objective will chase outliers.
2. Pairwise learning-to-rank, year as the query group, one frozen model — no annual re-selection
   among candidate rankers. Annual re-selection is what produced the previous walk-forward result
   of 63.66 against an incumbent of 67.67.
3. Features from the Tier 0 join described in
   [`DATA-REQUEST-ml-features.md`](DATA-REQUEST-ml-features.md), prioritising those orthogonal to
   the forward-EPS axis — within-year Spearman between `disc` and `fwd` is 0.494, so the two
   existing signals are roughly half redundant.
4. Validation: purged and embargoed walk-forward, plus leave-one-market-out on the global panel.
5. Report every result as a delta against the 69.22 control on the same pool construction, never
   as an absolute forecast.
