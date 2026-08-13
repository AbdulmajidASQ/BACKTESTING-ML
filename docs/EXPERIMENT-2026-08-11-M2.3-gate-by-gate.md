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

---

## Correction: the EMA mechanism is not a crash story

Prompted by the owner asking whether the RECOVERY waiver already covers 2008. Two things came out
of checking, one confirming and one correcting.

### 2008 is a NORMAL year — the waiver does not cover it

The manual regime list marks **calendar 2008 → rebalance 2009** as RECOVERY. Rebalance year 2008,
which runs April 2008 to April 2009 and is the year that lost 7.6%, is **NORMAL**. The EMA gate
applied in full straight through the crash, cutting that year's pool from **24 candidates to 11**.

So the current design **applies the trend filter during the descent and waives it during the
recovery**. If the belief behind the RECOVERY waiver is right — that broadly depressed prices make
a 200-week-EMA test reject the whole opportunity set — then the waiver is switched on one year too
late.

### But 2008 is not where most of the gain comes from

My earlier framing — "the mechanism is visible in 2008" — was true for that year and **overstated
as the general explanation**. Year-by-year attribution, all NORMAL years (RECOVERY years show zero
difference by construction, since the gate is already off):

| year | with EMA | without | diff |
|---|---|---|---|
| **2005** | 105.0 | 135.6 | **+30.6** |
| 2025 | 106.7 | 124.8 | +18.1 |
| 2008 | −6.5 | +6.6 | +13.1 |
| 2018 | 78.8 | 91.2 | +12.4 |
| 2024 | 120.3 | 131.1 | +10.8 |
| 2007 | 51.1 | 54.2 | +3.1 |
| 2017 | 47.9 | 47.3 | −0.6 |
| 2006 | 36.8 | 34.7 | −2.2 |
| 2022 | 49.4 | 46.8 | −2.7 |
| 2010 | 88.9 | 82.6 | −6.2 |
| 2004 | 78.7 | 71.9 | −6.7 |
| **2015** | 24.7 | 12.6 | **−12.1** |

The largest single gain is **2005 at +30.6**, an ordinary year in a rising market. Six years gain,
six lose, and the losses are real — 2015 costs 12.1 points.

**Revised characterisation: a broad, noisy effect with a positive mean, not a crash hedge.** Gains
total roughly +88 and losses roughly −31 across the years where anything changes.

### What this does to the recommendation

It survives, but with lower confidence and a different rationale.

- The **worst-year improvement is entirely 2008** (−6.5 → +6.6), and that single year is the whole
  of the "thirty positive years out of thirty" claim. It rests on one observation.
- The **CAGR gain is diffuse** and roughly two-to-one in favour across NORMAL years, which is a
  weaker basis than a mechanism.
- A permutation test is now more important than it looked, since a six-up / six-down split with a
  positive mean is exactly what noise produces some of the time.

Still worth registering as run #88, but as a modest, uncertain improvement rather than a
structural fix — and the honest headline is **+1.62 with high year-to-year variance**, not
"removes the losing year".

> **Superseded.** The permutation test below was run before registering anything. It says do not
> register run #88 at all. See "The permutation test, and what it does to this whole table".

---

# The permutation test, and what it does to this whole table

**Date:** 2026-08-13 · `bt.gate_perm_long`, `bt.gate_perm_res` · 300 permutations × 9 gates

The permutation test was run on the EMA gate before registering run #88. It did not just weaken
that result — it invalidated the way **every row of the table above** was measured.

## The measurement error

Every number in the original table is *"CAGR without gate X"* minus *"CAGR with all gates"*. That
difference silently contains two things:

1. whether gate X removes the *wrong* names, and
2. the fact that **gate X removes names at all**.

The second term is not zero, and it is not small. The book takes the top 7 from a ranked pool. Any
filter that shrinks that pool deletes some of the names the ranking would have chosen, and the
replacements are by definition ranked lower. **Shrinking a ranked pool costs CAGR mechanically,
whatever the filter is filtering on.**

So "dropping gate X gains N points" was never evidence that gate X is bad. It is what you would see
from a gate that does nothing at all, provided it is selective.

## The correct null

For each gate, shuffle its pass/fail flag among that year's candidates, preserving the number of
passes per year, then re-run the whole selection. Repeat 300 times. That gives the distribution of
outcomes for **a filter of identical selectivity that carries no information**.

The question then becomes the right one: *does the real gate beat a random cut of the same size?*

## Result — all nine gates against a random filter of equal selectivity

| gate | pool with → without | drop-gate CAGR | **observed "gain" from dropping** | **gain from dropping a RANDOM filter of the same size** | **excess over random** | σ | p (gate is harmful) |
|---|---|---|---|---|---|---|---|
| discount 25/75 | 769 → 1,876 | 60.04 | −12.59 | **+11.90** | **+24.49** | **6.49** | 0.000 |
| `g1 > 25` | 769 → 1,678 | 58.01 | −14.62 | +3.16 | **+17.78** | 4.53 | 0.000 |
| EPS chain | 769 → 1,012 | 69.45 | −3.18 | +7.16 | **+10.33** | 3.56 | 0.000 |
| leverage | 769 → 1,046 | 63.09 | −9.54 | −1.21 | **+8.33** | 2.70 | 0.000 |
| 14 industry exclusions | 769 → 927 | 67.85 | −4.78 | +1.71 | **+6.49** | 2.68 | 0.000 |
| P/F1 ≥ 2 | 769 → 776 | 72.83 | +0.20 | +1.03 | +0.83 | 0.77 | 0.173 |
| **EMA** | 769 → 858 | 74.41 | **+1.78** | **+2.26** | **+0.48** | **0.24** | **0.447** |
| dedup | 769 → 777 | 72.82 | +0.19 | +0.01 | −0.18 | −0.42 | 0.753 |
| **market-cap floor** | 769 → 909 | 77.06 | **+4.43** | **+2.24** | **−2.19** | **−0.71** | **0.763** |

Baseline with all gates: **72.63**. *(This rig is ~8 rows tighter than the original table's — I apply
the filing-date guard `g_knowable` — so levels differ by under a point. Both arms and all
permutations share one base, so every comparison here is internally consistent.)*

## 1. The EMA finding is dead — do not register run #88

**Dropping the EMA gate gains +1.78. Dropping a random filter that cuts the same number of names
gains +2.26.** The real gate sits at the **55th percentile** of the random distribution: 45% of
meaningless filters of the same selectivity do *better* than it, 55% do worse. That is the middle.

On the last 20 rebalance years — the window the owner's standing guidance says to weight — it is
worse still: observed gain +1.78 against a random-filter gain of **+3.57**, with the real gate at
the **70th percentile** of the null. The EMA gate looks *better* than a coin-flip filter there.

There is no CAGR case for dropping it. **Run #88 is cancelled.** The entire +1.62/+1.78 was the
price of pool shrinkage, and the EMA gate happens to be a marginally better-than-random way to pay
it.

### The worst-year claim also mostly dissolves

I made a lot of "thirty positive years out of thirty". Under the null, **72% of random filters of
the same selectivity also produce thirty positive years**. Having no losing year is the *normal*
outcome at this pool size, not an achievement.

What survives is thin and one-sided: the real EMA gate is in the 28% of filters that do retain a
losing year, and only **8.7%** of random filters produce a worst year as bad as its −6.5%. So there
is a marginal indication (p ≈ 0.09, single-tailed, driven entirely by 2008) that this specific gate
hurts the worst year more than an arbitrary cut would.

That is a **drawdown** claim, not a return claim, resting on one observation. It belongs in the
deferred drawdown workstream, where it should be re-tested on the drawdown series rather than on
worst calendar year. Recorded, not acted on.

## 2. The market-cap floor was never a +5 opportunity either

Rejecting it on survivorship grounds was right, and the statistics now say the same thing
independently. The floor's apparent +4.43 gain becomes **−2.19 against a random filter** — the
floor is at the 24th percentile of the null, weakest of the nine, but at −0.71σ that is
comfortably inside noise.

**Correct reading: the market-cap floor is statistically indistinguishable from a filter that does
nothing, in either direction.** It costs no measurable return. Combined with the fact that it keeps
the book out of the $18m–$50m band where the missing delistings concentrate, and out of names that
cannot absorb capital, it is close to free protection. The owner's instinct was right twice over.

## 3. The load-bearing gates are far more load-bearing than the table said

This is the upside of the correction, and it is large. Against a random cut of equal size:

| gate | old headline | **true value vs a random filter** |
|---|---|---|
| **discount gate** | −14.34 | **+24.49 (6.5σ)** |
| `g1 > 25` | −16.04 | +17.78 (4.5σ) |
| EPS chain | −3.95 | +10.33 (3.6σ) |
| leverage | −10.31 | +8.33 (2.7σ) |
| industry exclusions | −5.72 | +6.49 (2.7σ) |

**The ranking changes at the top.** The original table named `g1 > 25` "the most valuable single
gate in the strategy". It is not. **The discount gate is**, by a wide margin — 24.5 points better
than a random cut against `g1`'s 17.8, and at 6.5σ the most statistically secure result anywhere in
this programme. The old table under-ranked it because it is the most selective gate (769 from
1,876), so it paid the largest mechanical shrinkage penalty, which was being charged against it.

The EPS chain is the other revision: headline −3.95, described as merely useful, actually **+10.33
at 3.6σ** — the third most informative gate we have. It is small in raw effect only because it is
not very selective.

**All five clear 2.6σ.** Whoever designed this screen chose five genuinely informative filters and
two harmless ones. That is a much stronger endorsement of the original design than the first pass
gave it.

## 4. The methodological rule this establishes

**Never report "dropping gate X gains N points" without the equal-selectivity null.** Pool size and
gate quality are confounded in the raw difference, and the confound runs in a fixed direction: it
flatters every proposal to remove a gate and penalises every selective gate.

This is the third appearance of one underlying idea in this programme:

- `oi_tau` failed because **the gates had already spent the signal** — no variation left to rank on.
- The discount/growth double-count failed because **two ranking terms carried one signal**.
- The gate table failed because **a measured difference contained a second effect nobody subtracted.**

All three are the same discipline: before believing a difference, ask what else changed.

## What actually changes

| | before | after |
|---|---|---|
| Run #88 (drop EMA) | recommended | **cancelled — no CAGR case** |
| Market-cap floor | keep, on survivorship grounds | keep, on survivorship grounds **and** because it costs nothing measurable |
| Most valuable gate | `g1 > 25` | **the discount gate, 6.5σ** |
| EPS chain | minor | third most informative gate, 3.6σ |
| Energy sector exclusion | dead code, remove | unchanged — it filters zero rows, so no null applies |
| EMA and worst-year | "removes the only losing year" | withdrawn; 72% of random filters do the same |

Net effect on the strategy: **nothing changes.** Run #87 keeps every gate it has. The value of this
experiment is entirely negative — it stopped a run being registered on a measurement artefact, and
it tells us the screen we already own is better than we could previously prove.

## Reproducing

`bt.gate_perm_long` — one row per (gate under test, candidate passing all *other* gates), with the
gate's own flag retained. `bt.gate_perm_res` — 2,700 rows, one per (permutation, gate), holding that
permutation's CAGR and worst year. Baseline 72.63, 30 closed years, widened pool, `disc+fwd+gpa`,
apex `total_return_pct`.
