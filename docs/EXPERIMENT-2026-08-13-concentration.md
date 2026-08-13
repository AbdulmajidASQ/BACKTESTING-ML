# Concentration — how many names should the book hold?

**Date:** 2026-08-13 · Built on `bt.run87_sel` (806 ranked candidates, 1996–2026)
**Question:** the book has held 7 names (5 in RECOVERY) since inception. Nobody has ever tested it.

---

## Why this was the right thing to test next

M2.3 finished by showing there is **no CAGR left in the gates** — every one of them either earns
its keep against a random filter of equal selectivity or does nothing measurable. The ranking layer
has already been worked hard (runs #85–#87, 69.35 → 74.19).

The one lever nobody had touched is **how many of the ranked names you actually buy**. It is a
single integer, it has never been justified anywhere in the config or the catalogue, and under the
owner's mandate — fully invested, no options, maximise CAGR — it is the most direct control there
is.

---

## 1. First, does the ranking have skill deep enough to justify concentrating?

Mean realised return by rank position, run #87's score:

| rank | 1 | 2 | 3 | 4 | 5 | 6 | **7** | 8 | 9 | 10 | 11–16 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| mean return % | 86.4 | 95.5 | 88.3 | 73.1 | 85.3 | 78.3 | **69.0** | 41.2 | 40.9 | 59.0 | ~40 |

Two things:

- **There is a clean break at 7/8.** Ranks 1–7 average 82%; ranks 8–16 average 45%. The existing
  cut is well-placed — this is not a case of the book being obviously too small or too large.
- **Rank 7 is the weakest of the seven.** It is the only one of the top seven below 73%.

So the ranking works all the way down, which means a smaller book has a higher expected return per
name. Whether that beats the cost of extra concentration is an empirical question.

## 2. The sweep — and why the raw sweep is not trustworthy

Holding the RECOVERY rule scaled proportionally, sweeping the book size:

| names | 2 | 3 | 4 | 5 | **6** | **7 (today)** | 8 | 10 | 12 | 15 |
|---|---|---|---|---|---|---|---|---|---|---|
| CAGR, 30y | 75.59 | 71.01 | 69.98 | 73.94 | **76.01** | **74.19** | 69.94 | 66.38 | 64.21 | 62.46 |
| annual σ | 100.0 | 82.1 | 101.4 | 87.2 | 86.3 | **74.3** | 67.7 | 67.6 | 62.8 | 63.1 |
| worst year | −42.0 | −29.9 | −21.5 | −1.4 | −6.2 | −7.1 | −10.1 | −0.4 | −1.9 | −1.9 |

**Above 8 the decline is monotone and clean.** Below 8 the curve is noisy and non-monotone — 3 and
4 are bad, 5 and 6 are good, 2 is good again. That shape is a warning: if concentration were simply
good, 5 and 4 would beat 7 too. They do not. **Picking the maximum of a noisy curve is exactly how
false findings are manufactured**, so nothing here can be acted on without more work.

Note also that n = 2 looks best on 30-year CAGR after n = 6, and it should be dismissed regardless:
annual σ of 100 and a −42% year on a two-name book, in a dataset that has deleted roughly 90% of
delisted companies. A single failure is half the book, and the historical record has removed the
failures.

## 3. The sweep confounded two separate changes

Scaling the RECOVERY count with the NORMAL count mixed two decisions together. Separated:

| normal / recovery | CAGR 30y | CAGR last 20y | annual σ | worst year |
|---|---|---|---|---|
| **7 / 5 — today** | 74.19 | 73.46 | **74.3** | −7.1 |
| **6 / 5 — normal only** | **75.24** | **76.23** | **74.9** | **−6.2** |
| 7 / 4 — recovery only | 74.95 | 75.39 | **85.9** | −7.1 |
| 6 / 4 — both | 76.01 | 78.19 | 86.3 | −6.2 |

The two changes are roughly additive on return but **completely different on risk**:

- **Cutting NORMAL from 7 to 6 is nearly free.** Annual σ moves 74.3 → 74.9, and the worst year
  *improves*.
- **Cutting RECOVERY from 5 to 4 costs 11.6 points of annual σ** for 1.93 CAGR points. That is a
  bad trade on its own terms, and it is worse than it looks: there are only 11 RECOVERY years in
  the sample, so the recovery count is close to unfittable, and RECOVERY years are when the book is
  most fragile.

**Only the NORMAL change is worth pursuing.** Everything below tests 6/5 against 7/5.

## 4. Testing 6/5 against 7/5

The two books are **identical in 11 of 30 years** — in those years the pool held fewer than 7
candidates, so the constraint never bound. The comparison rests on the 19 years where a 7th name
actually existed.

### The mechanism

In those 19 years the 7th name returned **48.2%** against the top six's **62.7%** — a drag of 14.5
points, and below the top-six average in 14 of 19 years.

This is *expected* if the ranking works, and should not be presented as independent evidence: the
7th name is the lowest-ranked, so a functioning ranking guarantees it underperforms on average.
What it establishes is only that the mechanism is real and points the right way.

### Frequency and magnitude disagree

| | differing years | 6/5 wins | 7/5 wins | sign test |
|---|---|---|---|---|
| all 30 years | 19 | **14** | 5 | p = 0.032 |
| last 20 years | 15 | **12** | 3 | p = 0.018 |

But: **average win +6.2, average loss −9.6.** Dropping the 7th name wins more often and loses
bigger. The two large losses are **1998 (−21.5)** and **2004 (−19.5)**; every loss after 2005 is
under 5 points.

### Bootstrap over years, 3,000 resamples

| window | mean gain | 5th pct | bootstraps where 6/5 wins |
|---|---|---|---|
| all 30 years | +1.08 | **−1.46** | **76.7%** |
| **last 20 years** | **+2.79** | **+1.11** | **99.6%** |

On the full history the interval **includes zero** — inconclusive. On the last 20 years, the window
the owner's standing guidance says to weight, it excludes zero and the result is decisive.

That split is explained entirely by 1998 and 2004.

### Live year

April 2026 to date: **82.13% at six names against 79.53% at seven.** The 7th name is SMTC.

## 5. The most important result — do not re-tune this each year

Walk-forward: at each year from 2006, pick the book size that had the highest CAGR over all prior
years, then use it for that year.

| | CAGR, 2006–2025 |
|---|---|
| **walk-forward re-selection of n** | **70.16** |
| fixed at 7 (today) | 73.46 |
| fixed at 6 | 76.23 |

**Re-optimising the book size annually loses 3.3 points against simply leaving it at 7.** It picked
7 through 2018, 6 through 2024, then switched to 5 for 2025 — which returned 48.5% where 6 returned
136.2%. One bad switch cost the whole 20-year margin.

This is the project's known failure mode (the −4-point walk-forward failure recorded in the plan)
appearing again in a new place, and it carries two lessons:

1. **If the book size changes, it changes once and stays fixed.** No annual review, no
   "n was better last year". Codify it and leave it.
2. It is honest evidence *against* my own recommendation. The full-sample choice of 6 has hindsight
   in it that a live operator would not have had. What defends it is not the backtest maximum but
   the mechanism (the ranking demonstrably works down to rank 7), the fact that σ does not rise,
   and the fact that the gain is concentrated in the *recent* window rather than the distant one.

## 6. Recommendation

**Cut the NORMAL book from 7 names to 6. Leave RECOVERY at 5. Register as run #88.**

| | run #87 | **proposed #88** |
|---|---|---|
| NORMAL book | 7 | **6** |
| RECOVERY book | 5 | 5 |
| everything else | — | **unchanged** |
| CAGR, 30 closed years | 74.19 | **75.24** |
| CAGR, last 20 | 73.46 | **76.23** |
| annual σ | 74.3 | 74.9 |
| worst year | −7.1 | **−6.2** |
| live 2026 to date | 79.53 | **82.13** |

Honest expectation: **+1 to +2.8 CAGR points**, at the lower end if the next twenty years look more
like 1996–2005 than like 2006–2025.

### Caveats

- Selected from a sweep. The 30-year evidence is inconclusive on its own (76.7% bootstrap); the
  case rests on the recent window, the mechanism, and the flat volatility.
- Wins are frequent and small, losses rare and large. A −20-point year from this change is inside
  the historical range.
- All levels inherit the survivor-only universe. Only differences are meaningful.

---

# Registered as run #88

Built 2026-08-13, fully populated, all audit gates passing.

**`United States (US) · Conc-6 · 3-Factor Score, growth-orthogonalised discount`**

| | #14 | #85 | #86 | #87 | **#88** |
|---|---|---|---|---|---|
| ranking | `disc` | `disc+fwd+gpa`, size screen | `disc+fwd+gpa` | `disc_resid+fwd+gpa` | **same as #87** |
| NORMAL book | 7 | 7 | 7 | 7 | **6** |
| CAGR, 30 closed years | 69.22 | 72.57 | 73.77 | 74.19 | **75.24** |
| CAGR, last 20 years | — | — | — | 73.46 | **76.23** |
| Hit rate | 81.2% | 85.4% | 85.4% | 86.5% | **87.3%** |
| Worst year | −7.61 | −7.15 | −7.15 | −7.15 | **−6.23** |
| Annualised volatility | 33.11\* | 32.09 | 32.36 | **31.86** | 32.59 |
| Max drawdown | −52.74\* | −56.69 | −56.69 | −56.69 | −56.69 |
| Calmar | 1.31\* | 1.28 | 1.30 | 1.31 | **1.33** |
| Live 2026 to date | **102.66** | 72.55 | 72.00 | 79.53 | **82.13** |
| NAV reconciliation | — | 30/30 | 30/30 | 30/30, 0.022% | **30/30, 0.0066pp** |
| Weight exceptions | 0 | 0 | 0 | 0 | **0** |

\* #14 is measured on a ~364-row spine with weekend interpolation; #85–#88 share one trading-day
spine and are directly comparable to each other.

Registration: `runs` with reproducible config · 31 `run_years` · 172 `picks` · 172 `btd_books` ·
42,425 `btd_rel` · 7,630 `btd_daily` · catalogue with 11 caveats (draft, not subscribable) ·
4 events. NAV reconciliation 30/30 at worst 0.0066pp — tighter than #87. Weight audit clean across
7,660 date slices, weights summing to exactly 1.000000 in every one.

## The drawdown result, measured rather than assumed

I set a withdrawal condition before building: *"if the daily rebuild shows max drawdown materially
worse than #87's −56.69%."* It does not. **Max drawdown is −56.69%, unchanged to two decimals.**

**That is not the win it looks like, and it should not be reported as one.**

The deepest episode is peak **2020-02-20** to trough **2020-03-18** — inside rebalance year **2019**,
which is a **RECOVERY** year holding five names. #88 does not change RECOVERY. Both books held the
same five names through COVID, so the figure is identical *by construction*, not by merit.

The honest measurement is drawdown within NORMAL years, which are the only years #88 alters:

| rebalance year | regime | #87 | **#88** | change |
|---|---|---|---|---|
| 2019 | RECOVERY | −56.69 | −56.69 | **0.00 — untouched** |
| 2020 | RECOVERY | −49.35 | −49.35 | **0.00 — untouched** |
| **2008** | NORMAL | −39.16 | **−34.30** | **+4.87** |
| 2018 | NORMAL | −38.05 | −39.25 | −1.19 |
| 2025 | NORMAL | −34.43 | −36.23 | −1.80 |
| **2015** | NORMAL | −33.39 | **−36.84** | **−3.45** |
| 2022 | NORMAL | −32.06 | −30.64 | +1.42 |
| 2024 | NORMAL | −31.98 | −34.32 | −2.34 |
| 2021 | NORMAL | −31.02 | −32.39 | −1.37 |
| 1998 | NORMAL | −25.74 | −27.12 | −1.38 |
| 2006 | NORMAL | −21.03 | −22.81 | −1.78 |

Worse in seven of the nine NORMAL years, better in two, **mean about 0.8 points deeper**.

**So the trade is: +1.05 CAGR over 30 years and +2.77 over the last 20, for roughly 0.8 points of
deeper within-year drawdown in normal years, and no change to the historical maximum.** Volatility
rises 31.86 → 32.59. The Calmar improvement to 1.33 is real arithmetic but its denominator sits in
a regime this change never touches, so it overstates the case.

Under the owner's standing mandate — fully invested, maximise CAGR now, address drawdown in a
later session — that is an acceptable trade, and it is a smaller drawdown cost than the step from
#14 to #85 already accepted (−52.77 → −56.69, nearly four points).

## Note for whoever picks up the drawdown workstream

Two findings from this session belong to you, not here:

1. The EMA gate shows a marginal one-sided indication (p ≈ 0.09) that it deepens the worst calendar
   year relative to a random filter of equal selectivity, driven entirely by 2008. It has no CAGR
   case at all. Re-test it on the drawdown series rather than on worst calendar year.
2. The deepest drawdown in every run of this family occurs in a **RECOVERY** year holding five
   names. The RECOVERY rule — both the five-name count and the regime list itself — has never been
   validated, and it is where the drawdown actually lives. That is the highest-value target for
   drawdown work, ahead of anything in the NORMAL book.
