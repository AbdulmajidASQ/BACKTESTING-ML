# Run #88 vs the flagship — what changed, and how to run it live

**Date:** 2026-08-13 (originally 2026-08-11) · Corrects the risk comparison in earlier documents · Updated for run #88

---

## 1. Like-for-like risk comparison — CORRECTED

Earlier documents said the new rule improved drawdown. **That comparison was against the wrong
baseline** and is withdrawn. It compared against *the incumbent ranking rule applied to the widened
pool* (−60.66%), not against run #14's actual registered book.

All four books rebuilt on an **identical trading-day spine**, 1996–2025, equal weight, same price
source, same forward-fill method:

| | #14 (today) | #85 | #86 | #87 | **#88 (current)** |
|---|---|---|---|---|---|
| CAGR | 69.35 | 72.57 | 73.77 | 74.19 | **75.24** |
| Annualised volatility | 33.05 | 32.09 | 32.36 | **31.86** | 32.59 |
| **Max drawdown** | **−52.77** | −56.69 | −56.69 | −56.69 | −56.69 |
| Sharpe (rf = 0) | 2.10 | 2.26 | 2.28 | **2.32** | 2.31 |
| **Calmar (CAGR ÷ drawdown)** | 1.31 | 1.28 | 1.30 | 1.31 | **1.33** |

### What this actually says

**#14 has the best drawdown.** All four new runs are ~4 points worse at −56.69% against −52.77%.

The honest trade against today's flagship is: **+5.89 points of return for 3.92 points of
drawdown** (#14 → #88).

One caveat on the identical −56.69 across #85–#88: the deepest episode falls in rebalance year
2019, a **RECOVERY** year holding five names, which none of these runs changes. It is the same five
names in all four books, so the figure is identical by construction rather than by merit. Within
NORMAL years #88 is about 0.8 points deeper than #87.

Against #14:

- **Volatility improves** — 33.05 → 32.59, so the day-to-day ride is slightly smoother.
- **Sharpe improves** — 2.10 → 2.31, return per unit of volatility is genuinely better.
- **Calmar improves slightly** — 1.31 → 1.33, but read the caveat above before leaning on it.

The line to hold on to is unchanged from the earlier version of this document. **You are not
getting drawdown reduction. You are getting more return at roughly proportional drawdown, with a
slightly smoother path.** For a fully-invested concentrated book with no hedging, that is the
expected shape — and it is consistent with the programme's finding that under this mandate, return
and drawdown trade off against each other.

---

## 2. What changed, in plain terms

### Run #14 — what you do today

Screen the universe. Of the survivors, **sort by discount to fair value. Buy the top 7.**

One question: *how cheap is it?*

### Run #88 — what changes

Three changes from #14, in order of how much they matter.

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

**Change 3 — buy six, not seven.** In a normal year the book holds the top six instead of the top
seven. Recovery years still hold five.

The reason is simply that the ranking works. Across thirty years, positions one to seven returned
82% on average and positions eight to sixteen returned 45% — so the cut at seven was well placed —
but **position seven was the weakest of the seven**, at 69%. In the nineteen years where a seventh
name actually existed, it returned 48.2% against the top six's 62.7%.

Worth knowing: in eleven of thirty years this changes nothing at all, because the screen produced
fewer than seven candidates and the limit never bound. In the live 2026 year the whole change is
dropping one name, SMTC.

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

### Step 4 — Standardise the three, add, and take the top 6

For each of the three measures — column E (net cheapness), C (growth), D (gross profit/assets):

```
F2 = (E2 - AVERAGE(E:E)) / STDEV(E:E)
G2 = (C2 - AVERAGE(C:C)) / STDEV(C:C)
H2 = (D2 - AVERAGE(D:D)) / STDEV(D:D)

I2 = F2 + G2 + H2                       ← total score
```

Sort by column I. **Take the top 6** (top 5 in a recovery year), equal weight, hold to the next
1 April. Missing value → that z-score is 0.

> **Changed 2026-08-13.** This said "top 7" until run #88 tested the book size for the first time.
> Six is now the number for a normal year; **the recovery count stays at 5**. See
> [`EXPERIMENT-2026-08-13-concentration.md`](EXPERIMENT-2026-08-13-concentration.md). Two things
> matter operationally: the 7th name has historically been the weakest of the seven, and **the book
> size must never be re-tuned year to year** — re-optimising it annually from prior history loses
> 3.3 CAGR points against simply leaving it alone. Six is now fixed.

### Full spreadsheet layout

| A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|
| Ticker | Discount | Growth | GP/Assets | Net cheapness | z(E) | z(C) | z(D) | **Total** |

Four formulas, one sort. That is the whole method.

---

## 4. The April 2026 book, all four rules

| rank | **#88 (current)** | #87 | #86 | #14 (today) |
|---|---|---|---|---|
| 1 | NVDA | NVDA | NVDA | SNDK |
| 2 | MU | MU | SNDK | MU |
| 3 | STX | STX | STX | STX |
| 4 | WDC | WDC | VICR | WDC |
| 5 | CRDO | CRDO | WDC | CRDO |
| 6 | VICR | VICR | SMTC | AMD |
| 7 | — *(SMTC dropped)* | SMTC | CRDO | MRVL |
| **live return to date** | **82.13%** | 79.53% | 72.00% | **102.66%** |

#87 recovers MU (which #86 ranked 8th) because MU's cheapness-net-of-growth is the highest in the
list at +50.2. It still misses AMD and MRVL, which are the reason #14 leads the live year.

**#88 is #87 without SMTC**, the 7th name, which is running behind the rest of the book — hence
82.13% against 79.53%. The whole change, in the live year, is one name.

---

## 5. What to expect

- **+3 to +4 CAGR points** over the long run against #14. The +5.89 full-history figure leans on
  1996–2000.
- **Drawdown ~4 points worse** than #14. Return per unit of drawdown is modestly better (Calmar
  1.31 → 1.33), with the caveat above about where that drawdown lives.
- **Slightly smoother day-to-day than #14** — volatility 33.05 → 32.59, Sharpe up 0.21.
- **Multi-year stretches behind #14.** It is behind now. Judge over five years, never one.
- Every historical figure is measured on a universe missing about 90% of delisted companies, so all
  four runs are flattered by an amount not yet quantified. The *differences* between them are the
  trustworthy part.
