# Audit — survivorship in `apex_screening_master`

**Date:** 2026-08-11 · Project `pvqflvqfdlmiyjjetfer` · read-only SQL, nothing written to the database
**Test:** the blocking survivorship / point-in-time question that has stood open in this project's
research notes.

---

## Verdict

**The panel is not point-in-time. Survivorship bias is confirmed.** Three independent tests agree,
and the third is conclusive on its own.

The consequence, stated plainly: run #14's registered CAGR of 69.22% is measured on a universe
composed almost entirely of companies that still exist in 2026. It is inflated by an amount this
audit cannot size from inside the panel, because the missing rows are missing. **No model should be
trained on this data until the panel is rebuilt.**

---

## Test 1 — survival rate by vintage (structural, self-contained)

For each performance year, what fraction of that year's companies are still present in the
2024–2026 panel?

| performance year | tickers | also in 2024–26 | survival |
|---|---|---|---|
| 1996 | 1,750 | 1,696 | **96.9%** |
| 1998 | 2,048 | 1,988 | 97.1% |
| 2001 | 2,516 | 2,436 | 96.8% |
| 2004 | 2,829 | 2,719 | 96.1% |
| 2007 | 3,050 | 2,930 | 96.1% |
| 2010 | 3,420 | 3,286 | 96.1% |
| 2013 | 3,971 | 3,816 | 96.1% |
| 2016 | 4,426 | 4,276 | 96.6% |
| 2019 | 5,497 | 5,372 | 97.7% |
| 2022 | 7,844 | 6,968 | 88.8% |
| 2023 | 7,690 | 7,175 | 93.3% |

**96.9% of the companies in the 1996 universe are still in the panel thirty years later.** A real
US equity universe loses constituents continuously to acquisition, bankruptcy, going-private and
delisting; survival over a 28-year span should be somewhere near half, not 97%.

The rate is flat at ~96–97% across every vintage from 1996 to 2019, which is the signature of a
panel assembled from a recent ticker list and back-filled. Genuine attrition appears only in
2022–23 (88.8% and 93.3%) — the years recent enough that delistings have not yet been pruned from
the source list. That the *only* attrition visible is in the newest cohorts is itself the
confirmation.

A second tell: the ticker count grows monotonically, 1,750 → 3,420 → 7,844. The number of US
listed companies moved in the opposite direction over the same period, peaking in the late 1990s
and falling by roughly half by 2020. The panel's shape is the inverse of the market's.

## Test 2 — do known-dead companies appear at all?

| | count |
|---|---|
| rows in `public.companies` | 26,793 |
| marked `active = false` | **1,483 (5.5%)** |
| inactive companies with any `apex_screening_master` rows | 1,432 |
| inactive companies present in performance years 2005–2012 | **348** |

A database spanning 1979–2027 with 26,793 companies carrying only 5.5% dead is not a record of the
market. And 348 dead companies across the 2005–2012 window, against a universe of roughly
2,800–3,500 names per year, is far below the attrition a real panel would show.

**Correction to my earlier guidance:** I twice suggested using `companies.universe_last_seen` to
date delistings. That is wrong — the column holds a single constant value
(`2026-07-12 18:05:07.832455+00`) for every one of the 26,793 rows. It is a refresh stamp, not a
per-company last-seen date, and it cannot support this test or any other. There is no usable
delisting date anywhere in the schema.

## Test 3 — the outcome distribution (conclusive)

Run #14 holds 7 names at a time, ranks on a DCF discount above 25% in normal years and **above 75%
in recovery years**, and tolerates net debt / EBITDA up to 5 with a current ratio as low as 0.8 in
those recovery years. That is a deep-value, distress-tolerant, concentrated mandate, run through
2000–02, 2008–09 and 2020.

| | value |
|---|---|
| picks, all years | 193 |
| picks losing more than 50% | **0** |
| picks losing more than 80% | **0** |
| **worst single position, 30 years** | **−45.95%** |
| losing picks | 18.1% |
| distinct pool names now delisted | 9 of 224 (4.0%) |
| picks in companies now delisted | 8 of 193 |

**In 193 concentrated deep-value positions across thirty years, not one lost more than 46%, and
none went to zero.** That does not happen. Deep value with leverage tolerance produces
bankruptcies; the entire left tail of the outcome distribution is absent.

This is the same fact as tests 1 and 2 seen from the other end. Companies that failed are not in
the panel, so they were never candidates, so they were never picked, so no pick ever failed.

---

## What this invalidates, and what it does not

**Affected — treat as upper bounds, not estimates:**

- The registered CAGR of 69.22% (closed years) and every figure derived from it.
- The maximum drawdown of −52.74%. A book that could hold a name to zero would have drawn down
  further.
- The catalogue's headline claim of **one losing year in thirty**. That statistic is a direct
  product of the missing left tail and is subscriber-facing.
- The measured "selection headroom" of 28% captured / 72% open. Both the actual and the
  perfect-foresight bounds were computed on a survivor pool.
- Every cross-run comparison that shares this panel — which is all of them.

**Not affected:**

- The *reproduction* results. The generator still reproduces `bt.picks` 186/186 exactly; the
  arithmetic is sound. What is wrong is the input universe, not the engine.
- The relative findings about pool width — that ranking adds more when the pool is wide is a
  statement about the shape of the pool and is likely to survive a rebuild, though the magnitudes
  will move.
- The options-sleeve comparisons *relative to each other*, since all sleeve runs share the same
  underlying book and the same bias.

**Cannot be sized from here.** The magnitude of the inflation is not measurable from inside the
panel, because the evidence needed to measure it is exactly what is absent. Published estimates of
survivorship bias for broad equity portfolios run 1–4 percentage points a year; concentrated
deep-value mandates sit well above that range, because the strategy's whole risk profile lives in
the tail that has been removed. Anyone quoting a specific correction without a rebuilt panel is
guessing.

---

## What has to happen

1. **Rebuild the universe from a point-in-time source.** Every company that was listed and met the
   data requirements as at each 1 April, whether or not it exists today, keyed on a stable company
   identifier rather than a ticker string. This is not a patch to the existing table; it is a
   re-derivation.
2. **Source delisting events** — date and reason (acquired · merged · bankrupt · taken_private ·
   exchange_rule), plus a terminal return for names that die mid-holding-year. Without a terminal
   return, a failed position has no label and silently disappears again.
3. **Re-run run #14 on the rebuilt panel** and register the result as a separate run rather than
   overwriting #14, so the two are comparable and the history is preserved. The delta between them
   is the survivorship correction, and it is the single most valuable number this project could
   produce.
4. **Only then train anything.** A ranker fitted on the current panel learns what distinguishes
   good survivors from mediocre survivors. Deployed live, it will meet companies that fail —
   a category it has never seen.

Until step 3 exists, the honest description of run #14's track record is: *69.22% CAGR on a
survivor-only universe, magnitude of bias unquantified.*

---

## Related

- [`AUDIT-2026-08-11-point-in-time-integrity.md`](AUDIT-2026-08-11-point-in-time-integrity.md) —
  the filing-date look-ahead test, and the proof that company identity is carried backward
  (ticker `WBD` in performance year 2008 named "Warner Bros. Discovery Inc", an entity created in
  2022). That identity defect and this survivorship result are two symptoms of one cause: the
  panel is keyed on present-day tickers.
- [`DATA-REQUEST-ml-features.md`](DATA-REQUEST-ml-features.md) — item 2.2 (delisted companies)
  is now the top priority in that request, ahead of the feature work.
