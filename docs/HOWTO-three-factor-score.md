# How to calculate the three-factor score — practical guide

**The single most important thing to understand first:**

> **You cannot calculate this score for one company on its own.**

The score is **relative**. It measures how a company compares to *the other candidates that passed
the screen in the same April*. A company with a 60% discount might score well in a year when the
average candidate is at 40%, and badly in a year when the average is at 80%. The same company, the
same number, two different scores.

So the unit of work is **the year's candidate list**, never a single stock.

---

## Step by step

### Step 1 — Build the candidate list

Run the existing screen exactly as it stands today. All the usual gates: US-listed, profitable,
forward earnings improving, leverage limits, price above the 200-week average in normal years, the
excluded industries, the market-cap floor for the era.

**One change:** the forward-growth gate loosens from `> 25%` to `> 0%`. That is what widens the list
from ~13 names a year to ~28.

You now have a list — say 28 companies. Everything below happens *within* that list.

### Step 2 — Collect three numbers for each candidate

| # | measure | where it comes from |
|---|---|---|
| 1 | **Discount to fair value** | `dcf_discount_percent` — already in the screen |
| 2 | **Forward profit growth** | `eps_forward_cagr` — already in the screen |
| 3 | **Gross profit ÷ total assets** | `Gross-Profit-to-Asset %` — new. From the annual accounts: gross profit divided by total assets. It is in `public."AnnualCommonSizeRatios"` |

If you had to compute measure 3 by hand from a company's accounts:

```
gross profit  =  revenue − cost of goods sold
gross profitability  =  gross profit ÷ total assets  × 100
```

That is all it is. A company with $400m gross profit on $2,000m of assets scores 20%.

### Step 3 — Turn each measure into a comparable score

For each of the three measures separately, across your candidate list:

```
average    = the mean of that measure across all candidates this year
spread     = the standard deviation of that measure across all candidates this year

score      = (this company's value − average) ÷ spread
```

That is a **z-score**. It says "this company is 1.4 standard steps above this year's average
discount", which is a number you can compare and add across measures that were originally
percentages, growth rates and ratios.

- Positive = better than this year's average
- Zero = exactly average
- Negative = worse than average
- Typical range is about −2 to +2

**If a number is missing, use 0** — treat it as average rather than guessing or dropping the company.

### Step 4 — Add the three and rank

```
total score = z(discount) + z(forward growth) + z(gross profitability)
```

Equal weight. No tuning. Sort descending, take the top 7 — or top 5 in a recovery year — equal
weight, hold to the next 1 April.

---

## Worked example — the real April 2026 list

The 2026 candidate list had 31 names. Here is the top of it, scored:

| rank | company | total score | live return since 1 Apr |
|---|---|---|---|
| 1 | NVDA | 3.46 | +23.9% |
| 2 | SNDK | 3.31 | +83.5% |
| 3 | STX | 3.21 | +94.1% |
| 4 | VICR | 2.66 | +33.3% |
| 5 | WDC | 2.32 | +47.1% |
| 6 | SMTC | 1.88 | +63.9% |
| 7 | CRDO | 1.88 | +158.2% |
| — | — | *cut line* | — |
| 8 | MU | 1.62 | +136.1% |
| 9 | AMD | 1.57 | +125.6% |
| 10 | MRVL | 1.42 | +99.1% |

NVDA scores 3.46 — the highest in the list — because it was simultaneously well above average on
all three measures relative to that year's other candidates. Note this is *not* a claim that NVDA is
cheap in absolute terms; it is a claim that it ranked well against the 30 other names that passed
the same screen in April 2026.

---

## In a spreadsheet

One row per candidate, one sheet per year:

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| Ticker | Discount | Fwd growth | Gross profit / assets | z(B) | z(C) | z(D) | Total |

- `E2 = (B2 - AVERAGE(B:B)) / STDEV(B:B)`
- `F2 = (C2 - AVERAGE(C:C)) / STDEV(C:C)`
- `G2 = (D2 - AVERAGE(D:D)) / STDEV(D:D)`
- `H2 = E2 + F2 + G2`

Sort by column H, take the top 7. That is the whole method.

---

## Should the size screen stay?

**Recommendation: drop it.** The evidence marginally favours removing it, and simpler is safer.

| period | with size screen | without | difference |
|---|---|---|---|
| all 30 closed years | 72.57 | **73.77** | +1.20 |
| excluding 1996–2000 | 73.57 | **74.20** | +0.63 |
| last 20 years | 71.53 | **71.84** | +0.31 |
| live 2026 | 72.55 | 72.00 | −0.54 |

Every closed-year window is slightly better without it. All four differences sit inside noise, so
this is a **simplicity** argument rather than a performance one: one fewer rule, one fewer
parameter, one fewer thing to get wrong.

The size effect itself is real and independently validated — permutation p ≤ 0.033, negative in 21
of 24 markets. The point is that **the score already captures it**. Both the discount leg and the
gross-profitability leg tilt toward smaller companies on their own, so screening on size afterwards
is largely doing the same job twice.

### Correction to an earlier claim

I previously said the size screen is what excludes MU, AMD and MRVL from the 2026 book, implying it
caused this run to trail #14. **That was wrong and is withdrawn** (recorded as run 85 event seq 5).
Ranked on score alone with no size screen, MU is 8th, AMD 9th and MRVL 10th — below the cut anyway.
Removing the screen changes the 2026 book only by swapping ALGM for NVDA, and makes the live year
slightly *worse*.

The 2026 shortfall is caused by the **score itself** preferring VICR (+33%), SMTC (+64%) and ALGM
(+28%) over MU (+136%), AMD (+126%) and MRVL (+99%). That is the rule working as designed and
having a bad year, which is a different thing from a rule with a faulty component.

---

## What to expect

- **+2 to +3 CAGR points** over the long run against ranking on discount alone. Not the +5.4 that
  the full-history figure suggests — that number leans on 1996–2000.
- **Multi-year stretches of underperformance.** It is behind right now. Judge it over five years or
  more, never over one.
- Every historical figure here is measured on a universe containing only companies that still exist,
  so it flatters this strategy and every other one on the platform by an unmeasured amount.
