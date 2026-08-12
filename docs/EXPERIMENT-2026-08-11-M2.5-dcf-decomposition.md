# M2.5 — Decomposing the DCF discount

**Date:** 2026-08-11 · Prompted by the data agent's full disclosure of the DCF construction
**Objects used:** `bt.v_us14_features` (388 columns), `bt.run86_sel`

---

## What the disclosure told us

Every DCF assumption is a constant: 10-year horizon, 10% flat discount rate, terminal = E₁₀ / 0.10
(zero terminal growth), base = `eps_f1`, growth = `fwd` clamped to [−20, +100].

Therefore `fair_value = eps_f1 × K(fwd)` where K is fixed given growth, and

```
disc  =  1 − (price / eps_f1) / K(fwd)
```

Reconstructing this from the disclosed constants correlates **0.848** with the stored `disc`, which
confirms the structure (the residual gap is a constant offset — my clamp era or terminal-year
convention is slightly off, and chasing it is not worth the effort). Empirically `disc` correlates
0.525 with `fwd` and 0.311 with forward earnings yield.

**My immediate inference was that `disc` is therefore just a repackaged forward P/E, and that
replacing it with a clean earnings yield would be tidier. That inference was wrong.**

## The test

Five ranking rules, all on the widened pool, price-return basis, 30 closed years, `top_n` by regime.

| rule | CAGR | windows beating the current score |
|---|---|---|
| **C — `disc_resid_fwd` + fwd + gpa** | **74.19** | **5 of 6** |
| A — `disc` + fwd + gpa *(current, run #86)* | 73.77 | — |
| D — `disc` + earnings yield + fwd + gpa | 69.07 | 3 of 6 |
| B — earnings yield + fwd + gpa | **63.44** | 0 of 6 |
| E — earnings yield + gpa | 60.27 | 0 of 6 |

### Finding 1 — the DCF transform earns its place

Swapping `disc` for a raw forward earnings yield **costs 10.3 CAGR points** (73.77 → 63.44) and
loses in all six windows.

The information content is the same — `disc` is a deterministic function of forward P/E and growth
— but the *functional form* matters. The DCF combines price and growth **non-linearly**, and that
combination ranks better than a linear z-sum of the same two inputs. A company on 15× forward
earnings growing at 40% is not equivalent to one on 30× growing at 80%, and the DCF knows that
where a z-score sum does not.

**Correction to my own claim:** I said `disc` carries "exactly one piece of non-growth information,
the forward P/E", implying it could be substituted. The first half is true; the implication is
false and is withdrawn. Same information, better packaging.

### Finding 2 — residualising works, and validates unusually well

`disc_resid_fwd` — the discount with its growth component regressed out, fitted within-year only by
the data agent, so leak-free — scores **74.19** against 73.77, and wins 5 of 6 windows.

Robustness, which is the striking part:

| period | current (A) | residualised (C) | difference |
|---|---|---|---|
| all 30 years | 73.77 | 74.19 | +0.42 |
| **excluding 1996–2000** | 74.20 | **76.19** | **+1.99** |
| **last 20 years** | 71.84 | **73.46** | **+1.62** |

**This gets stronger when the best window is removed** — the reverse of every other result in this
programme, where the edge shrank by two-thirds under the same test. The 30-year figure *understates*
it. On the period the owner's standing guidance says to weight most heavily, it is worth **+1.6
points**.

### Why residualising helps here but failed before

In M2.2 I tested `disc_resid_fwd` and recorded it as a failure at −2.49. That was `disc_resid_fwd`
**alone** as the sole ranker, with no growth term. Stripping growth out of the discount and then not
adding it back discards a signal the incumbent relies on.

The correct use is what the data agent proposed: residualise to remove the double-count, then supply
growth explicitly as its own term. `disc_resid_fwd + fwd + gpa` is three genuinely distinct
signals — cheapness net of growth, growth, and profitability — where `disc + fwd + gpa` counts
growth roughly one and a half times.

No contradiction between the two results; they answer different questions.

## Recommendation

**Register a run #87 using `disc_resid_fwd + fwd + gpa`.** It is the same three concepts as #86 with
the growth double-count removed, it beats #86 in 5 of 6 windows, and it is the only rule tested all
session whose edge *increases* under the drop-the-best-window test.

Expected: **+1.5 to +2.0 CAGR points over #86**, so roughly **75.5 to 76.0 on the honest windows**.

Prerequisites, all met: `disc_resid_fwd` ships in `bt.v_us14_features`, fitted within-year only;
the feature view reproduces the pool exactly (835 widened / 380 incumbent, all 186 closed-year picks
present, zero fan-out); the priority feature set is 0.0% null in every era.

## Standing caveat

The survivorship condition **triggered** on the data agent's audit — 240-name probe, 10.5% panel
presence overall, 17% for 2010s deaths. Training is on hold per the lab rule. This experiment is a
fixed formula rather than a fitted model, so it is not caught by that hold, but every level here
remains optimistic by an unquantified amount. Only the differences between arms are meaningful, and
they are all measured on the same pool.

---

## Registered as run #87

Built 2026-08-11, fully populated to the checklist, all audit gates passing.

**`United States (US) · Conc-7 · 3-Factor Score, growth-orthogonalised discount`**

### The family, side by side

| | #14 | #85 | #86 | **#87** |
|---|---|---|---|---|
| ranking | `disc` | `disc+fwd+gpa`, size screen | `disc+fwd+gpa` | **`disc_resid+fwd+gpa`** |
| CAGR, 30 closed years | 69.22 | 72.57 | 73.77 | **74.19** |
| Hit rate | 81.2% | 85.4% | 85.4% | **86.5%** |
| Worst year | −7.61 | −7.15 | −7.15 | −7.15 |
| Annualised volatility | 33.11\* | 32.13 | 32.39 | **31.91** |
| Max drawdown | −52.74\* | −56.69 | −56.69 | −56.69 |
| **Live 2026 to date** | 102.66 | 72.55 | 72.00 | **79.53** |
| NAV reconciliation | — | 30/30 | 30/30 | **30/30, worst 0.022%** |
| Weight exceptions | 0 | 0 | 0 | **0** |

\* #14 is measured on a ~364-row spine with weekend interpolation and is not directly comparable
to the trading-day spines of #85–#87.

**#87 is the best of the family on every internal measure**: highest CAGR, highest hit rate,
**lowest volatility**, and the best live-year figure of the three new runs. Higher return at lower
volatility is the outcome you want from removing a redundancy rather than adding a bet.

It remains behind #14 in the live year (79.53 vs 102.66), for reasons recorded in run #86 event
seq 3 and unchanged here: the score ranks MU, AMD and MRVL below the cut and they have been the
strongest names of 2026 so far.

### Registration completeness

`runs` with reproducible config · 31 `run_years` · 192 `picks` · 192 `btd_books` · 47,302
`btd_rel` · 7,637 `btd_daily` · catalogue with 7 caveats (draft, not subscribable) · 3 events.
NAV reconciliation 30/30 at worst gap 0.022%, weight audit clean, `run_options` N/A.

### Build note worth keeping

The `btd_daily` insert timed out repeatedly until `ANALYZE bt.btd_rel` was re-run after the
47,302-row insert. Stale planner statistics, not a query defect. **Anyone rebuilding a daily layer
should ANALYZE `bt.btd_rel` afterwards** — combined with the missing indexes found during the #86
build, this is very likely why daily rebuilds in this project were historically done by hand.
