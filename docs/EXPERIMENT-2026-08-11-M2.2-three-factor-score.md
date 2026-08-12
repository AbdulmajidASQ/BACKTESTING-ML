# M2.2 — Fixed-formula sweep: the three-factor score

**Date:** 2026-08-11 · Widened pool `bt.ml_us14_pool_v1`, 30 closed years, look-ahead guarded
**Control:** incumbent rank rule on the widened pool — **68.16**. Shipped strategy — **69.22**.

---

## Headline

A three-factor z-score — **discount + forward growth + gross-profit-to-assets**, equal weight,
ranked within year — scores **73.57** against an incumbent of 68.16 on the same pool.

**+5.41 points over the pool control, +4.35 over the shipped strategy.**

It also **subsumes the size rule** validated in M2.1: adding the size screen on top *lowers* the
result to 72.40. The size tilt was a crude proxy for something the score captures directly.

**Status: promising, not yet shippable.** See §4 — this was selected from six candidate rules on
the same thirty years, and it wins only 3 of 6 windows.

---

## 1. Result by window

| window | incumbent | size only | **3-factor score** | size + score |
|---|---|---|---|---|
| 1996–2000 | 50.27 | 59.22 | **71.96** | 68.28 |
| 2001–2005 | 87.73 | 78.77 | 84.30 | 82.09 |
| 2006–2010 | 57.84 | 58.21 | 54.84 | 55.08 |
| 2011–2015 | 34.33 | 36.07 | **42.00** | 37.38 |
| 2016–2020 | 94.43 | 96.27 | 93.62 | **99.72** |
| 2021–2025 | 94.42 | 94.53 | **102.70** | 101.42 |
| **all 30 years** | **68.16** | 69.12 | **73.57** | **72.40** |

Per-window edge of the score against the incumbent: **+21.69, −3.43, −3.00, +7.67, −0.81, +8.28**.

**It wins big and loses small.** Three windows positive, three negative, but the average win is
+12.5 and the average loss is −2.4. That asymmetry is the reason the 30-year compound figure is
strong despite a 50% window hit rate.

The size+score variant is *more consistent* (4 of 6 windows) but compounds lower (72.40). Given how
close the two are, consistency is worth more than the 1.2 points.

## 2. Ablation — all three components are needed

Mean edge across the six windows, each rule applied after the size screen so they are comparable:

| rule | mean edge | windows won |
|---|---|---|
| **disc + fwd + gpa** | **+3.87** | **5 / 6** |
| disc + gpa | +0.86 | 4 / 6 |
| disc + fwd | −0.51 | 3 / 6 |
| fwd only | −0.90 | 2 / 6 |
| disc residualised on fwd | −2.49 | 3 / 6 |
| gpa only | −6.61 | 1 / 6 |

**Gross profitability is useless as a standalone ranker (−6.61) and valuable in combination.** That
is coherent — it is a quality measure, not a valuation measure, and quality without a price
constraint just buys expensive good companies. But note the tension in §4.

`disc` residualised on `fwd` **fails** (−2.49), which is a clean negative result: the ρ = 0.494
correlation between them is not redundancy to be stripped out, it is shared signal worth keeping.

## 3. Why this is economically plausible

The three factors are three distinct questions, and the flagship currently asks only the first two,
one of which it does not use for ordering:

| factor | question |
|---|---|
| `disc` | is it cheap against its own DCF? |
| `fwd` | are earnings expected to grow? |
| `gpa` | does the business actually generate profit per unit of assets? |

Gross-profit-to-assets is Novy-Marx gross profitability, among the most replicated cross-sectional
factors in the published literature, and it was sitting unused one join away from the master file.
It is the quality leg that a pure value-plus-growth screen lacks — the thing that separates a
genuine bargain from a cheap, growing, capital-destroying business.

## 4. Why this is not yet shippable

**Multiple comparisons.** This was the best of six rules tested on the same thirty years. With six
draws, some separation is expected by chance. The +5.41 is an upper bound on the true effect.

**Window hit rate is 50%.** Three of six windows negative. The compound figure depends on the
asymmetry between wins and losses holding out of sample, which is a stronger assumption than it
looks.

**One window carries disproportionate weight.** 1996–2000 contributes +21.69 of the +30.4 total
edge. That is the earliest, smallest-pool period, where candidate counts are thinnest and single
names move the result most. Strip it and the case is much weaker.

**The ablation pattern is a mild warning sign.** No pair of the three works well (best pair +0.86),
yet all three together give +3.87. Genuine three-way diversification of independent signals does
behave this way — but so does overfitting. The two are hard to separate on 30 observations.

**Cannot be validated cross-market with current data.** `bt.cand_stage` carries neither a DCF
discount nor gross profitability, so the leave-one-market-out test that vindicated the size rule
cannot be run on this one. That is a gap worth closing — see recommendations.

## 5. Recommendation

1. **Adopt provisionally the `size + 3-factor score` variant**, not the pure score. It compounds
   1.2 points lower but wins 4 of 6 windows rather than 3, and consistency is the scarcer property
   here. Expected improvement over the shipped strategy: **≈ +3.2 points**, not the +4.35 headline.
2. **Do not ship on this evidence alone.** Required first:
   - real daily drawdown on hypothetical books (M1.1b), per the standing protocol
   - a re-run excluding 1996–2000 to confirm the result does not depend on one thin window
   - a permutation test against random three-feature z-combinations drawn from the same table
3. **Add `gross_profit_to_asset_pct` and a DCF discount to `bt.cand_stage`** so the cross-market
   test becomes possible. This is the single most valuable addition for validating any future
   selection rule, and it belongs in the data request.
4. **Register the negative results** so they are not re-run: `disc` residualised on `fwd`, `fwd`
   alone, `gpa` alone, and the `disc+fwd` pair all fail.

## 6. Where this leaves the CAGR programme

| stage | CAGR | note |
|---|---|---|
| shipped strategy | 69.22 | registered, reproduces exactly |
| widened pool, incumbent rule | 68.16 | widening alone costs 1.06 |
| + size screen (M2.1) | 69.12 | validated: permutation p ≤ 0.033, 21/24 markets |
| **+ 3-factor score (M2.2)** | **72.40** | **provisional, needs the checks in §5** |
| pure 3-factor score, no size screen | 73.57 | higher but less consistent |
| perfect foresight on this pool | 112.62 | the ceiling |

Roughly **+3.2 points banked provisionally**, against 44 points of open headroom identified at the
start of M2. The remaining gap is what the learning-to-rank model (M2.4) is for, and the bar it
must now clear is 72.40 rather than the incumbent.
