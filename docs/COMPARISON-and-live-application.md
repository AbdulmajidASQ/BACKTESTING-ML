# Run #87 vs the flagship — what changed, and how to run it live

**Date:** 2026-08-11 · Corrects the risk comparison in earlier documents

---

## 1. Like-for-like risk comparison — CORRECTED

Earlier documents said the new rule improved drawdown. **That comparison was against the wrong
baseline** and is withdrawn. It compared against *the incumbent ranking rule applied to the widened
pool* (−60.66%), not against run #14's actual registered book.

All four books rebuilt on an **identical trading-day spine**, 1996–2025, equal weight, same price
source, same forward-fill method:

| | #14 (today) | #85 | #86 | **#87** |
|---|---|---|---|---|
| CAGR | 69.35 | 72.57 | 73.77 | **74.19** |
| Annualised volatility | 33.05 | 32.15 | 32.41 | **31.92** |
| **Max drawdown** | **−52.77** | −56.69 | −56.69 | −56.69 |
| Sharpe (rf = 0) | 2.10 | 2.26 | 2.28 | **2.32** |
| **Calmar (CAGR ÷ drawdown)** | **1.31** | 1.28 | 1.30 | **1.31** |

### What this actually says

**#14 has the best drawdown.** All three new runs are ~4 points worse at −56.69% against −52.77%.

The honest trade is: **+4.84 points of return for 3.92 points of drawdown.**

- **Volatility improves** — 33.05 → 31.92, so the day-to-day ride is slightly smoother.
- **Sharpe improves** — 2.10 → 2.32, return per unit of volatility is genuinely better.
- **Calmar is identical** — 1.31 → 1.31. Return per unit of *drawdown* is unchanged.

That last line is the one to hold on to. **You are not getting drawdown reduction. You are getting
more return at proportionally more drawdown, with a slightly smoother path.** For a fully-invested
concentrated book with no hedging, that is the expected shape — and it is consistent with the
programme's finding that under this mandate, return and drawdown trade off against each other.

---

## 2. What changed, in plain terms

### Run #14 — what you do today

Screen the universe. Of the survivors, **sort by discount to fair value. Buy the top 7.**

One question: *how cheap is it?*

### Run #87 — what changes

**Change 1 — a wider net.** The forward-growth gate loosens from "must exceed 25%" to "must be
positive". Roughly doubles the candidate list, from ~13 names to ~28.

This on its own makes things slightly *worse*. It is done because a bigger list is what gives the
ranking something to work with — in half the years the old screen produced barely more candidates
than positions, so there was no real choice to make.

**Change 2 — three questions instead of one.**

| | question | why |
|---|---|---|
| 1 | **How cheap is it, beyond what its growth already explains?** | The fair value is *calculated from* the growth forecast, so raw discount and growth are largely the same signal. Stripping the growth out leaves genuine cheapness. |
| 2 | **How fast are profits expected to grow?** | Now asked separately and explicitly, instead of being buried inside the discount. |
| 3 | **How much gross profit does it earn per dollar of assets?** | Entirely new. Separates a real bargain from a cheap, growing business that destroys capital. |

Under #14 the ranking effectively asks question 1 and question 2 *mixed together*, and never asks
question 3.

### Why "beyond what growth explains" matters

The fair-value model uses fixed assumptions — 10-year horizon, 10% discount rate, no terminal
growth — and takes **expected profit growth as its only variable input**. So a high discount is
partly just a high growth forecast wearing a different hat.

Ranking on discount *and* growth therefore counts growth about one and a half times. Removing it
from the discount and adding it back as its own term gives three distinct signals instead of two
and a half.

**Concrete example from the April 2026 list.** MU and NVDA both score highly, for opposite reasons:

| | discount | growth | cheapness net of growth | gross profit/assets |
|---|---|---|---|---|
| MU | 98.3 (very high) | 54.9 | **+50.2** — far cheaper than its growth explains | 19.5 (modest) |
| NVDA | 72.6 (lower) | 31.9 | +13.8 | **96.4** (exceptional) |

On raw discount, MU beats NVDA comfortably. Under the new score they rank 2nd and 1st — because
NVDA's profitability carries it, and MU's genuine cheapness (not just its growth forecast) carries it.

---

## 3. How to run it live, each April

### Step 1 — Build the candidate list

Run your existing screen unchanged, with **one gate loosened**: forward growth `> 0%` instead of
`> 25%`. Everything else — the universe, EPS chain, leverage limits, 200-week average, excluded
industries, market-cap floors — stays exactly as it is.

You now have a list of roughly 25–35 names. **Everything below happens inside that list.**

### Step 2 — Four columns per candidate

| column | source |
|---|---|
| Discount | `dcf_discount_percent` — you already have it |
| Forward growth | `eps_forward_cagr` — you already have it |
| Gross profit ÷ assets | `Gross-Profit-to-Asset %` — the new input |
| *(computed below)* | cheapness net of growth |

### Step 3 — Strip growth out of the discount

This is the only genuinely new calculation. In a spreadsheet, with discount in column B and growth
in column C:

```
slope      =  SLOPE(B:B, C:C)          ← one number for the whole list
intercept  =  INTERCEPT(B:B, C:C)      ← one number for the whole list

E2  =  B2 - (intercept + slope * C2)   ← this company's cheapness net of growth
```

In words: draw the average line through this year's candidates relating discount to growth, then
measure **how far above or below that line each company sits**. Above the line = genuinely cheaper
than its growth forecast alone would justify.

### Step 4 — Standardise the three, add, and take the top 7

For each of the three measures — column E (net cheapness), C (growth), D (gross profit/assets):

```
F2 = (E2 - AVERAGE(E:E)) / STDEV(E:E)
G2 = (C2 - AVERAGE(C:C)) / STDEV(C:C)
H2 = (D2 - AVERAGE(D:D)) / STDEV(D:D)

I2 = F2 + G2 + H2                       ← total score
```

Sort by column I. **Take the top 7** (top 5 in a recovery year), equal weight, hold to the next
1 April. Missing value → that z-score is 0.

### Full spreadsheet layout

| A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|
| Ticker | Discount | Growth | GP/Assets | Net cheapness | z(E) | z(C) | z(D) | **Total** |

Four formulas, one sort. That is the whole method.

---

## 4. The April 2026 book, all three rules

| rank | #87 (new) | #86 | #14 (today) |
|---|---|---|---|
| 1 | NVDA | NVDA | SNDK |
| 2 | MU | SNDK | MU |
| 3 | STX | STX | STX |
| 4 | WDC | VICR | WDC |
| 5 | CRDO | WDC | CRDO |
| 6 | VICR | SMTC | AMD |
| 7 | SMTC | CRDO | MRVL |
| **live return to date** | **79.53%** | 72.00% | **102.66%** |

#87 recovers MU (which #86 ranked 8th) because MU's cheapness-net-of-growth is the highest in the
list at +50.2. It still misses AMD and MRVL, which are the reason #14 leads the live year.

---

## 5. What to expect

- **+2 to +3 CAGR points** over the long run. The +4.84 full-history figure leans on 1996–2000.
- **Drawdown ~4 points worse.** Return per unit of drawdown is unchanged.
- **Slightly smoother day-to-day** — volatility down about 1.1 points, Sharpe up 0.22.
- **Multi-year stretches behind #14.** It is behind now. Judge over five years, never one.
- Every historical figure is measured on a universe missing about 90% of delisted companies, so all
  four runs are flattered by an amount not yet quantified. The *differences* between them are the
  trustworthy part.
