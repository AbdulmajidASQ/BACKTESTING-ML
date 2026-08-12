# M2.1 — Validation of the drop-largest-third rule

**Date:** 2026-08-11 · Widened pool `bt.ml_us14_pool_v1`, 30 closed years, look-ahead guarded
**Rule under test:** drop the largest third of candidates by total assets each year, then rank by
discount and take `top_n` exactly as now. All market-cap floors unchanged.
**Control:** the incumbent rank rule on the same pool — 68.16 CAGR.

---

## Verdict

**The effect is real but roughly half the size first measured, and highly regime-dependent.**

Honest estimate: **+2 CAGR points**, not the +3.5 reported from the single split. It survives a
permutation test decisively, is robust in direction across a wide range of cut points, and does not
worsen sector concentration — but it delivers negative edge for multi-year stretches and wins in
only 17 of 30 individual years.

Recommend adoption at a **less-fitted cut point** with explicit expectation-setting about
multi-year underperformance. One test (leave-one-market-out) is still outstanding.

---

## Test 1 — Permutation: is it size, or just exclusion? **PASS, decisively**

Thirty pseudo-random exclusion sets of identical size, drawn by hashing ticker against a seed, each
dropping a third of that year's candidates before ranking by discount.

| | CAGR |
|---|---|
| incumbent | 68.16 |
| **drop largest third** | **71.20** |
| random third — mean of 30 draws | **63.26** |
| random third — best of 30 | 67.70 |
| random third — worst of 30 | 59.38 |

**Zero of 30 random draws beat the actual rule; p ≤ 0.033.**

The more informative number is the *direction* of random exclusion: dropping a random third **costs
4.9 points** against the incumbent, because it discards good candidates at random. Dropping
specifically the largest **gains 3.0**. The spread between the rule and random exclusion is **7.9
points**. Whatever is happening, it is about size, not about holding fewer candidates.

## Test 2 — Multi-split stability: **MARGINAL**

Six non-overlapping five-year windows. The rule has no fitted parameters, so the question is
whether the edge is stable across periods rather than train-versus-test.

| window | incumbent | rule | edge | years rule wins |
|---|---|---|---|---|
| 1996–2000 | 50.27 | 63.15 | **+12.89** | 3/5 |
| 2001–2005 | 87.73 | 75.29 | **−12.45** | 1/5 |
| 2006–2010 | 57.84 | 63.09 | +5.25 | 4/5 |
| 2011–2015 | 34.33 | 36.02 | +1.69 | 4/5 |
| 2016–2020 | 94.43 | 91.04 | **−3.38** | 1/5 |
| 2021–2025 | 94.42 | 107.73 | **+13.32** | 4/5 |

Positive in 4 of 6 windows, mean edge ≈ **+2.9pp**, range **−12.45 to +13.32**, and the rule beats
the incumbent in only **17 of 30 individual years** — a 57% hit rate.

The single 15/15 split reported earlier (+3.54) was not wrong; it sat near the mean and concealed
the dispersion. **This is the test that changes the recommendation.**

Note the two negative windows are economically coherent rather than random: 2016–2020 was the peak
of mega-cap dominance, exactly the regime in which a size tilt should fail. That makes the effect
more believable as a real phenomenon and *less* attractive as a reliable one.

## Test 3 — Cut-point sensitivity: **PASS in direction, PEAK-FITTED in magnitude**

| % largest dropped | CAGR | worst year | losing years |
|---|---|---|---|
| 0 (control) | 67.50 | −7.3 | 1 |
| 10 | 69.42 | −7.3 | 1 |
| 20 | 69.55 | −6.5 | 1 |
| 25 | 69.12 | −6.5 | 1 |
| **33** | **71.20** | +8.4 | 0 |
| 50 | 68.70 | +1.2 | 0 |
| 60 | 65.47 | +2.9 | 0 |
| 67 | 64.64 | +2.9 | 0 |

Every level from 10% to 50% beats the control, so the direction is not knife-edge. But **33% is a
local peak** with neighbours at 69.12 and 68.70. The 10–50% plateau averages ≈ 69.6 against a
control of 67.50, giving an honest effect of **≈ +2.1 points**. Roughly 1.6 points of the headline
came from landing on the best cut.

Degradation beyond 50% is sharp and consistent with the earlier finding that restricting to the
smallest third destroys value.

## Test 4 — Sector concentration: **PASS**

| rule | mean sector HHI | mean names in largest sector | years book is 100% one sector | years ≥70% one sector |
|---|---|---|---|---|
| incumbent | 0.337 | 2.80 | 0 | 0 |
| drop largest 25% | 0.345 | 2.80 | 0 | 1 |
| drop largest 33% | 0.354 | 2.87 | 0 | 1 |

Concentration rises marginally and immaterially. The rule does not make the Technology-heavy
problem worse.

## Test 5 — Leave-one-market-out: **NOT YET RUN**

Does "avoid the giants" replicate outside the US? Outstanding, and required before shipping.

## Drawdown watch (proxy only)

Per the standing protocol, drawdown is monitored but not optimised in this phase. On the annual
proxy the rule **improves** risk: worst year −7.3 → +8.4, losing years 1 → 0, and the improvement
is monotonic in the drop fraction.

**This proxy is not trustworthy** — annual-path drawdown reads −7.27% for a book whose true daily
maximum drawdown is −52.74%. Real drawdown must be measured on daily paths (M1.1b) before this rule
ships. The proxy is at least not flashing red.

---

## Scorecard against the pre-committed ship criteria

| criterion | required | result |
|---|---|---|
| positive in ≥4 of 5 splits | yes | **4 of 6** — pass, but with large negative windows |
| permutation p < 0.05 | yes | **p ≤ 0.033** — pass decisively |
| replicates in ≥60% of LOMO folds | yes | **not yet run** |
| no material sector concentration increase | yes | **pass** — HHI 0.337 → 0.354 |
| stable across cut points | yes | **pass in direction**, peak-fitted in magnitude |

## Recommendation

1. **Adopt, but at ~+2 points of expectation rather than +3.5.** The permutation result is strong
   enough that the direction should be believed; the multi-split and cut-point results say the
   magnitude was flattered.
2. **Use a less-fitted cut point.** Choose 25% (the largest quartile) on the grounds that it is a
   natural fraction, not because it scored best — 33% scoring highest is precisely why it should
   not be the choice. Expect ≈ +1.6 to +2.1 points.
3. **Set expectations for multi-year underperformance.** The rule lost 12.45 points over 2001–2005
   and 3.38 over 2016–2020. Anyone watching this quarterly will conclude it is broken at some
   point, and that judgement will be wrong. Write it into the catalogue caveats before it ships.
4. **Run leave-one-market-out before shipping.** If the effect does not replicate across markets it
   is a US artefact and the case weakens considerably.
5. **Measure real drawdown on daily paths before shipping**, per the standing protocol.

## Registry entry

| field | value |
|---|---|
| rule | drop largest third (and quartile variant) by total assets, then rank by discount |
| control | incumbent rank rule, widened pool, 68.16 |
| full-period result | 71.20 at 33%, 69.12 at 25% |
| permutation p | ≤ 0.033, 0 of 30 |
| multi-split | 4 of 6 windows positive, −12.45 to +13.32 |
| honest effect | ≈ +2.1pp |
| verdict | **provisionally adopt at 25%; LOMO and daily drawdown outstanding** |
