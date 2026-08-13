# M2.3 — What each gate is worth

**Date:** 2026-08-11 · Built `bt.gate_lab_us14`, 81,580 rows, every gate as an independent flag

---

## Method

The shipped views bake every gate in, so no gate can be examined on its own. `bt.gate_lab_us14`
holds all US candidates for 1996–2025 with **each selection gate as a separate boolean**, which
makes "relax exactly one thing" a one-line change.

The rig reproduces both shipped pools: **379 of 380** incumbent and **833 of 835** widened (the
one- and two-row gaps are rows with null returns, excluded here).

Baseline is the widened pool with the `disc + fwd + gpa` score. Returns are apex
`total_return_pct` — used identically across every arm, so **comparisons are valid but the levels
are not comparable to runs #85–#87**, which use the 1 April adjusted-close basis.

---

## Result

| variant | pool | CAGR | vs baseline | worst year |
|---|---|---|---|---|
| **0 — baseline, all gates** | 777 | **73.57** | — | −6.5 |
| **8 — drop market-cap floor** | 920 | **78.58** | **+5.01** | −6.5 |
| **1 — drop EMA gate** | 867 | **75.19** | **+1.62** | **+6.6** |
| 5 — drop P/F1 ≥ 2 | 784 | 73.77 | +0.20 | −6.5 |
| 9 — drop dedup | 787 | 73.76 | +0.19 | −6.5 |
| 4 — allow Energy sector | 777 | 73.57 | **0.00** | −6.5 |
| 10 — relax EPS chain | 1,025 | 69.62 | −3.95 | −15.7 |
| 3 — drop 14 industry exclusions | 936 | 67.85 | −5.72 | −6.5 |
| 6 — drop leverage gates | 1,055 | 63.26 | −10.31 | −10.4 |
| 7 — drop discount gate | 1,900 | 59.23 | −14.34 | −6.0 |
| 2 — relax `g1 > 25` to `g1 > 0` | 1,698 | 57.53 | **−16.04** | −20.8 |

---

## 1. The market-cap floor "costs" 5 points — and the number is a trap

**Recommendation: keep the floor. Do not act on this result.**

The measurement says removing it gains 5.01 CAGR points. Here is what it actually buys:

| | with floor | **without floor** |
|---|---|---|
| smallest holding | $66m | **$18m** |
| 10th percentile | $330m | $160m |
| median | $1,619m | $855m |
| picks under $250m | 10 | **37** |
| picks under $100m | 5 | **15** |
| **picks under $50m** | **0** | **7** |

Two reasons the number should not be believed:

**Survivorship contamination is worst exactly here.** The data agent's probe found only **10.5%
of delisted companies present in the panel**. Micro-caps are the most likely to delist and the
least likely to survive into a present-day-sourced ticker list. An $18m company that went bankrupt
in 2005 is almost certainly absent. Removing the floor pushes the book into precisely the size
band where the missing failures concentrate, so the +5.01 is measured against a universe that has
deleted the losers. **This is the single least trustworthy result in the programme.**

**It is not investable.** An $18m company cannot absorb meaningful capital at any sensible
participation rate. Even a real return here would not be reachable.

The owner's instinct that the floors are protective was right, and the arithmetic appearing to
disagree is an artefact of the data, not a finding.

## 2. The EMA gate costs 1.62 points and one losing year — drop it

This one is clean, and it is not size-related, so it carries none of the contamination above.

The gate requires price above its own 200-week average at rebalance, in NORMAL years only —
RECOVERY already waives it, on the owner's rule that troughs sit below the EMA. This result says
the same logic applies more broadly.

| window | with EMA | without | edge |
|---|---|---|---|
| 1996–2000 | 71.96 | 71.96 | 0.00 |
| 2001–2005 | 84.30 | 88.05 | +3.75 |
| 2006–2010 | 54.84 | 58.02 | +3.18 |
| 2011–2015 | 42.00 | 39.14 | **−2.86** |
| 2016–2020 | 93.62 | 96.06 | +2.44 |
| 2021–2025 | 102.70 | 107.36 | **+4.66** |
| **all 30 years** | **73.57** | **75.19** | **+1.62** |

Four windows positive, one flat, one negative — and the worst window is only −2.86, far more stable
than the size rule's −12.45.

**It also removes the only losing year.** Worst year improves from **−6.5% to +6.6%**, so the book
would have had thirty positive years out of thirty. That is counter-intuitive: dropping a trend
filter admits beaten-down names, which ought to raise risk. It does the opposite here, and the
mechanism is visible — in 2008 nearly everything traded below its 200-week average, so the gate
was rejecting the whole opportunity set at the one moment cheapness mattered most.

**This is the actionable finding of M2.3.** Modest, stable, improves both return and the worst year,
and no exposure to the survivorship problem.

## 3. Three gates do nothing at all

| gate | effect |
|---|---|
| **Energy sector exclusion** | **exactly 0.00** |
| P/F1 ≥ 2 | +0.20 |
| dedup by company name | +0.19 |

The Energy exclusion is **dead code**. Removing it changes the pool by zero rows, because the 14
industry exclusions already contain Oil & Gas and Other Energy Sources. It has been carried in the
config, the catalogue and every derived run for no effect.

P/F1 and the dedup rule are similarly inert on this pool — worth keeping for safety, since they
guard against pathological inputs rather than earning return, but neither is doing work.

## 4. Five gates genuinely earn their keep

Relaxing any of these is expensive, which is a good sign about the screen's original design:

| gate | cost of relaxing |
|---|---|
| `g1 > 25` — next-year profit growth | **−16.04** |
| discount gate (25 / 75) | −14.34 |
| leverage (NDE, current ratio) | −10.31 |
| 14 industry exclusions | −5.72 |
| EPS chain | −3.95 |

**`g1 > 25` is the most valuable single gate in the strategy** — worth 16 CAGR points, more than
any ranking improvement found all session. Worth noting given it never appears in the ranking
score: it does its work entirely as a filter.

This also revises an earlier expectation. The programme plan flagged the 14 industry exclusions as
suspect because they are the largest single cut in the funnel. They are not suspect: relaxing them
costs 5.72 points.

---

## Recommendations

1. **Keep every market-cap floor.** The apparent +5.01 is survivorship contamination at the
   micro-cap end and is not investable regardless.
2. **Drop the EMA gate in NORMAL years.** +1.62 CAGR, worst year −6.5 → +6.6, stable across
   windows, no survivorship exposure. Register as run #88 after a permutation check.
3. **Remove the Energy sector exclusion from the config and catalogue** — not to change behaviour,
   which is provably unchanged, but because carrying a rule that does nothing invites the belief
   that it does something.
4. **Leave the remaining gates alone.** Five of them are load-bearing and two are harmless.

## Caveats

Returns here are apex `total_return_pct`, so levels differ from runs #85–#87. Every arm shares the
basis, so the differences hold.

All results inherit the survivor-only universe. The EMA finding is largely insulated because it is
not size-conditioned; the market-cap finding is the opposite and is rejected on those grounds.
