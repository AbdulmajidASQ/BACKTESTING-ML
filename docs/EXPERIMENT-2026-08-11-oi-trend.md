# Operating-income trend (`oi_tau`) — what it is, and why it doesn't help us

**Date:** 2026-08-11 · Registered as a tested negative so it is not re-run

---

## What the column is

There is a family of trend tables in `public` that no run has used:

| table | grain | contents |
|---|---|---|
| `mizan_trend_score` | company × calendar year, **135,253 rows, 1979–2025** | `rev_tau`, `gp_tau`, `oi_tau` plus scores and a `tier` |
| `mizan_trend_oi` | company × calendar year | the operating-income slice on its own |
| `mizan_book_trend_bt` | strategy × year × ticker | already joined to backtest books with returns |

**`oi_tau` is Kendall's tau of operating income over time**, computed over roughly 5 years of
history (mean 5.3 points). It runs from −1 to +1 and measures **consistency of direction, not
magnitude**:

- `+1.00` — operating income rose in every single year, without exception
- `0.00` — as many up years as down, no discernible direction
- `−1.00` — fell every year

This is a different question from growth *rate*. A company growing 40%, then 5%, then 30% has a
lower tau than one growing 6%, then 7%, then 8% — the second is more *reliable*, the first is
faster. `tier` buckets it into Strong / Improving / Flat / Deteriorating / Insufficient.

It is a genuinely sensible idea: durability of earnings improvement, which none of our three
factors measures.

---

## Coverage on our pool

**100%.** Every one of the 777 closed-year candidates in run #87's pool has a trend record when
joined on `company_id` and `calendar_year = rebalance_year − 1`. No data work needed.

---

## Result — it does not help, and adding it hurts badly

### As a ranking factor

Within-year Spearman against realised return, 26 years, compared with the factors we already use:

| factor | mean ρ | positive years |
|---|---|---|
| `fwd` (in use) | **+0.181** | 23/26 |
| `disc` (in use) | +0.160 | 19/26 |
| `gpa` (in use) | +0.147 | 19/26 |
| **`oi_tau`** | **+0.075** | **15/26 (58%)** |
| `score_profit` | +0.051 | 14/26 |
| `gp_tau` | +0.041 | 15/26 |
| `rev_tau` | +0.025 | 12/26 |

About half the strength of our weakest live factor, and barely better than a coin flip on sign.

### Added to the score

| rule | CAGR |
|---|---|
| **current 3-factor (#87)** | **74.19** |
| exclude the Deteriorating tier, then rank | 73.32 (−0.87) |
| + `oi_tau` at half weight | 69.36 (−4.83) |
| + `oi_tau` as a full fourth leg | **63.92 (−10.27)** |

Adding it as a fourth equal leg costs **10.3 CAGR points**. Even the gentlest use — simply dropping
the Deteriorating names before ranking — loses 0.87.

---

## Why — the gates already spent this signal

Mean return by tier, across the pool:

| tier | candidates | mean return | median | % positive |
|---|---|---|---|---|
| Strong | 406 | 51.7 | 36.4 | 83.3 |
| Improving | 199 | 52.0 | 33.2 | 85.4 |
| Flat | 85 | 52.7 | 35.2 | 83.5 |
| Deteriorating | 87 | **43.1** | 28.1 | 83.9 |

**Strong, Improving and Flat are indistinguishable** — 51.7, 52.0, 52.7. Only Deteriorating lags,
and even there the hit rate is identical at 83.9%; it is a slightly thinner right tail, not a
higher failure rate.

The reason is that **the screen already requires rising earnings before a company can be a
candidate at all**: current EPS > 0, `eps_f1 < eps_f2`, and `eps_f2` above both the 2-year-ago and
3-year-ago figures. By the time a name reaches the pool, mean `oi_tau` is **+0.42** and 78% are
already Strong or Improving.

The information has been consumed by the gates. Re-applying it at the ranking stage adds no new
discrimination — it just dilutes the weight on the three factors that do carry information. Adding
a fourth equal leg cuts each real factor's influence from a third to a quarter, and that alone
explains most of the 10-point loss.

---

## The general lesson, which is the valuable part

This is the **same failure mode as the discount/growth double-count**, appearing in a new place.

- `disc` already contained `fwd`, so ranking on both counted growth 1.5×. Fixed by residualising.
- The **gates** already contain the earnings-trend signal, so ranking on it too counts it twice.

**Before adding any factor, ask what the gates have already removed.** A screen that filters on a
property destroys the variation in that property, and a factor with no variation left cannot rank.

This is worth applying to the existing three as well: our gates enforce `disc > 25`, `g1 > 25` and
`fwd > 0`, so some of the surviving range in those variables is already truncated. `gpa` is the
only one of the three that **no gate touches**, which may be part of why it added value.

---

## Verdict

**Do not use `oi_tau`, `rev_tau`, `gp_tau`, `score_profit` or the `tier` classification as ranking
factors or as a screen, on this pool.** Registered as tested and rejected.

Two situations where it might still be worth revisiting:

1. **On a much wider pool.** If the EPS chain gates were relaxed — the way `fwd` was relaxed to
   widen the pool — trend consistency would regain variation and might then discriminate. It is a
   candidate for the gate-by-gate work in M2.3, not for the ranking layer.
2. **As a risk rather than return measure.** The Deteriorating tier has a thinner right tail at the
   same hit rate. That is a mild drawdown-flavoured signal, not a return signal, and belongs in the
   deferred drawdown workstream rather than here.

Note also that `public.mizan_book_trend_bt` already joins these measures to backtest books with
returns, which suggests this line was explored before. Worth asking whether an earlier conclusion
exists before spending more on it.
