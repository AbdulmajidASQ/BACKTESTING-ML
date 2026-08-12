# Instruction for the data agent — supporting the three-factor score

**Date:** 2026-08-11 · Project `pvqflvqfdlmiyjjetfer` · relates to runs #85 and #86

---

## Current state — nothing has been added to the master file

`public.apex_screening_master` has **not** been modified. No score column, no gross-profitability
column. The runs were built by joining out to the `Annual*` family at query time.

---

## The most important point: do NOT add the score as a column

**The three-factor score is relative, not absolute.** Each leg is standardised against *the other
candidates that passed the screen in that same April*:

```
z = (this company's value − mean of THIS YEAR'S CANDIDATE POOL) ÷ stdev of THAT POOL
```

It therefore depends on:

1. **Which gates ran** — the incumbent pool (`fwd > 25`, ~13 names) and the widened pool
   (`fwd > 0`, ~28 names) produce *different z-scores for the same company in the same year*.
2. **Which run** — #14, #85 and #86 have different pools, so a single stored number cannot serve
   all three.
3. **The rebalance date** — the pool is defined as at 1 April.

A static `score` column on the master file would be wrong for at least two of the three runs, and
whoever read it later would have no way of knowing which pool it was computed against. **The score
must be computed at selection time, inside the view or function that builds the candidate list.**

What belongs in the master file is the **ingredient**, not the score.

---

## What to add — one column

### `gross_profit_to_assets` (numeric)

The only genuinely missing input. Absolute, per company-year, stable — exactly the kind of thing
the master file already holds.

**Source, in priority order:**

1. **Preferred — `public."AnnualCommonSizeRatios"."Gross-Profit-to-Asset %"`**, joined via
   `apex_screening_master.reporting_period_id → public."AnnualReportPeriods".annual_report_id`.
   That join resolves at **150,224 of 150,224 rows**. This is the source runs #85 and #86 use, so
   it is what reproduces them.

2. **Fallback — computed from the master's own columns**: `gross_profit / total_assets × 100`.
   Available at 94.9% / 99.9% coverage respectively.

**These two are NOT interchangeable — please do not treat them as equivalent.** Measured over
94,009 overlapping rows, only 36% agree within 0.5pp and 64% within 2.0pp, with the ratios table
running about 1.25pp higher on average. The likely cause is a different asset base (period-average
rather than period-end) or a different gross-profit definition. Consequence for selection: using
the fallback changes 10 of 186 picks and moves the 30-year CAGR from 73.77 to 72.82.

**Use source 1.** If you must use source 2, say so explicitly, because it will not reproduce the
registered runs.

### Please also confirm

- Null rate by era, using the buckets 1996–99 / 2000–09 / 2010–19 / 2020–26.
- That the value is **as-of the fiscal year in the row**, not restated later.
- Whether the underlying ratio uses **average** or **period-end** total assets — this is the likely
  explanation for the discrepancy above and it should be documented either way.

---

## What would be valuable but is not required

### Expose the rest of the `Annual*` family

The same 1:1 join reaches roughly **149 columns** across five tables that no run has ever read:
`AnnualValuationQuality`, `AnnualCashFlowStatements`, `AnnualCommonSizeRatios`,
`AnnualValuationRatios`, `AnnualPerShareMetrics`. Gross profitability was found there; there may be
more. A single wide view at the candidate grain would let future research use all of it without
re-deriving the join each time. Full specification in
[`DATA-REQUEST-ml-features.md`](DATA-REQUEST-ml-features.md).

### Add two columns to `bt.cand_stage` — highest leverage item

`bt.cand_stage` (432,705 rows, 25 markets) carries **neither a DCF discount nor
gross-profit-to-assets**. That is why the cross-market validation which confirmed the size effect —
negative in 21 of 24 markets — **could not be run on the three-factor score**.

Adding `disc` and `gpa` to `cand_stage` would let every future selection rule be tested across 25
markets instead of one. For validating research, this is worth more than any other single addition.

---

## What the engineering side needs (not a data task)

For the live April screen to actually produce this book, something must compute the score at
selection time. The logic is in `bt.run86_sel`'s definition and is four steps:

1. Build the candidate list with the existing gates, `eps_forward_cagr > 0` instead of `> 25`.
2. Within that list, z-score each of `dcf_discount_percent`, `eps_forward_cagr`,
   `gross_profit_to_assets`. Null → 0.
3. Sum the three with equal weight.
4. Take the top 7 (top 5 in a recovery year), equal weight.

A view mirroring `bt.v_us14_cand` — call it `bt.v_us86_cand` — is the natural home. That is an
engineering change, not a data one, and I can build it.

---

## Summary

| item | who | required? |
|---|---|---|
| `gross_profit_to_assets` on the master file, from `AnnualCommonSizeRatios` | data agent | **yes** |
| Confirm null rates by era, as-of semantics, and the average-vs-period-end asset base | data agent | **yes** |
| A `score` column on the master file | **nobody — do not do this** | **no** |
| `disc` and `gpa` on `bt.cand_stage` | data agent | high value |
| Wide view over the full `Annual*` family | data agent | nice to have |
| Live selection view computing the score | engineering (me) | yes, separately |
