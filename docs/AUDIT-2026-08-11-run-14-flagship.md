# Audit — Run #14, the US flagship

**Date:** 2026-08-11
**Run:** `bt.runs.id = 14` — `United States (US) · Conc-7 · LIVE-RULES + Recovery branch ($5B era 2026+)`
**Scope:** the full registration checklist and the audit gates (NAV reconciliation, weight
audit, option reconciliation, generator reproduction), plus the catalogue record.
**Method:** read-only SQL against `pvqflvqfdlmiyjjetfer`. Nothing was written to the database.
`bt.run_events` (23 events, 2026-07-22 → 2026-08-08) was read first, per house practice.

---

## Verdict

**The mechanics are sound; the live year is not.** All 30 closed years reproduce exactly from
the generator view, and every structural gate passes. Four defects sit on the 2026 live row
and on what the catalogue tells subscribers. Two of them move the headline number.

---

## 1. What passes

| Gate | Result |
|---|---|
| Selection reproduction (`bt.v_us14_cand`) | **186/186 exact including rank**, 1996–2025; 0 generated-not-stored |
| `run_years` coverage | 31 rows, 1996–2026, no gaps, no idle years |
| `picks` integrity | 193 rows; ranks contiguous 1..n every year; no null sector or gate values |
| Annual chain `picks` → `run_years.strat_return` | reproduces all 31 years within 0.05pp |
| NAV chain (`nav_strat` vs `strat_return`) | clean 1996–2025 (1997 off 0.08pp, a 1-dp rounding artefact) |
| `runs.cagr` = 69.22 | **reproduces exactly** from `run_years` over the 30 closed years; index 10.29 likewise |
| Weight audit (`bt.v_btd_weight_audit`) | **zero exceptions** across 68,209 `btd_rel` rows |
| `btd_books` | weights sum to 1.0 in all 31 years; 193/193 symbols and returns agree with `picks` |
| Daily NAV vs `run_years` | 30/30 closed years within 0.1% (worst 0.031%) |
| Max drawdown | −52.74% confirmed |
| `bt.run_options` | 0 rows — correct, #14 is stock-only; checklist item 7 is N/A |
| `bt.run_events` | 23 events including a `registered` event; provenance is complete |

### Generator reproduction

```sql
with reg as (select rebal_year, regime from bt.run_years where run_id=14 and rebal_year<=2025),
ranked as (
  select v.ry, v.ticker,
         row_number() over (partition by v.ry order by
            case when r.regime='RECOVERY' then v.fwd else v.disc end desc,
            case when r.regime='RECOVERY' then v.disc else v.fwd end desc,
            v.ticker) as rnk,
         case when r.regime='RECOVERY' then 5 else 7 end as topn
  from bt.v_us14_cand v join reg r on r.rebal_year = v.ry
)
select count(*) from ranked s join bt.picks p
  on p.run_id=14 and p.rebal_year=s.ry and p.symbol=s.ticker and p.rank=s.rnk
where s.rnk <= s.topn;   -- 186
```

Every generated pick matched a stored pick at the same rank; there are exactly 186 stored
picks for closed years. The generator is faithful.

### Drawdown detail

The −52.74% maximum drawdown is the COVID crash: peak **2020-02-20**, trough **2020-03-18** —
a 27-calendar-day event, not 2008. The catalogue's risk block reports the magnitude without
this context.

---

## 2. Defects

### D1 — The two logged corrections to the live 2026 return were never applied

`bt.run_events` records the problem twice and the fix once:

- **Event 441** (2026-07-29, `defect_found`): recomputing the 2026 book from
  `public.historical_daily_prices` over 1 Apr – 28 Jul 2026 gives **84.9%**, against the
  **143.00%** carried in the registry. The registered figure therefore uses an entry price
  below the 1 April close, breaking the April→April convention. *"Flagged for review, not patched."*
- **Event 546** (2026-07-30, `corrected`): restates it on the convention (1 Apr 2026 close to
  29 Jul 2026 close) as **75.17**, with SNDK at +46.65 not +58.23 and MU at +100.93 not +123.10.

Today `bt.run_years` for 2026 still reads `strat_return = 143.00`, and `bt.picks` still carries
the 2026-07-23 per-name marks that event 546 explicitly superseded:

| rank | symbol | stored `fwd_return` | event 546 restatement |
|---|---|---|---|
| 1 | SNDK | 151.7 | 46.65 |
| 2 | MU | 184.0 | 100.93 |
| 3 | STX | 131.8 | — |
| 4 | WDC | 105.8 | — |
| 5 | CRDO | 143.2 | — |
| 6 | AMD | 171.5 | — |
| 7 | MRVL | 113.0 | — |

Twelve days unpatched, on the flagship. This is the root defect — D2 and part of D3 follow
from it.

### D2 — The 73.19% headline is the open-stub window, and it is hostage to D1

On the closed window 1996-04-01 → 2026-04-01 the daily NAV gives **69.22%**, identical to the
registry. The 73.19 figure comes from running to today (30.36 years) and **recomputes to 73.17
as of 2026-08-10; it moves every day.**

| live-2026 basis | full-window CAGR |
|---|---|
| 143.00 (registered) | 73.17 |
| 84.9 (event 441) | 71.61 |
| 75.17 (event 546) | 71.31 |
| closed years only | **69.22** |

So roughly **1.9 CAGR points hang on an unresolved basis dispute**, and **3.95 points on
quoting an open stub at all**. Any comparison against another run must use the closed window
or state the as-of date.

Risk statistics on the closed window, computed with the run's own spine (`ppy = 365.3`, not 252):

| metric | value |
|---|---|
| CAGR | 69.22% |
| annualised volatility | 33.11% |
| Sharpe (rf = 0) | 2.09 |
| max drawdown | −52.74% |

### D3 — The 2026 daily layer is one pre-aggregated row per date, not seven

Every closed year holds `btd_rel` at **one row per constituent per date**, with full `btd_px`
price coverage. The 2026 year alone holds **one row per date with `w = 1`**, across 90 dates —
even though `btd_px` carries all seven names for 2026.

```sql
select ry, count(distinct dt) as n_dates, count(*)::numeric/count(distinct dt) as rows_per_date
from bt.btd_rel where run_id = 14 group by ry;   -- 2026 → 1.00; every other year → n_picks
```

Consequences:

1. This is the same defect class as **event 380** — daily NAV following a single series rather
   than the weighted book — and it bypasses the **event 381** fix that aggregates
   `sum(w*rel) + (1 - sum(w))` per `(run, ry, dt)`.
2. The weight audit cannot see it: a single row summing to 1 passes trivially.
3. Per-name contribution to the live NAV is unverifiable.
4. It produces a **third** live figure. On 2026-07-23 the daily series reads **+170.5%**, the
   same date on which the registry recorded **+143.0%**. On 2026-07-29 the daily series reads
   +108.2% against event 546's +75.17.

The live year currently has no agreed entry basis, and the three coexisting series disagree by
30–70 points.

### D4 — The regime branch is a hand-written year list, and the catalogue calls it a mechanical rule

`bt.catalog.filters_exact.regime_trigger` tells subscribers:

> *"RECOVERY when the market index closes below its own 200-week EMA on the rebalance date;
> NORMAL otherwise. Point-in-time, no look-ahead."*

The actual assignment comes from `bt.runs.config.recovery_branch_2026_07_22.years` — eleven
calendar years typed in by hand (1995, 96, 99, 2000, 01, 02, 08, 15, 18, 19, 22) which map
exactly, shifted by one, onto the eleven RECOVERY rows in `run_years`.

**The stated trigger cannot be evaluated for the US at all.** `bt.lp_regime` and
`bt.index_daily` cover 31 markets — ADX, BAH, BKK, DFM, DSMD, IST, KUW, LSE, MIL, MUS, OCSE,
OHEL, OSL, OSTO, SA, SGX, TPE, TSE, TSX, TSXV, WAR, WBO, XAMS, XBRU, XKLS, XKRX, XLIS, XMAD,
XPAR, XSWX, XTER — **none of them US**. There is no SPY, VOO, IVV or QQQ in `public.companies`
either; event 130 records the same fact ("The S&P series is not in the database").

Materiality: by **event 41**'s own record, adding the Recovery branch lifted the 30-year CAGR
from **53.61% to 62.31% (+8.7 points)**. It fires on 2009, 2020 and 2023 — three of the four
biggest years in the run. The run's catalogue row is `subscribable: true`.

This is a look-ahead exposure presented to subscribers as a point-in-time rule. It is the most
serious finding in this audit, and it is a documentation and governance problem rather than a
computational one: nothing about the reproduction is wrong, but the run is not what its
catalogue says it is.

---

## 3. Structural observations

### The NAV reconciliation gate is tautological for this run

`btd_rel` year-end endpoints match `run_years.strat_return` to **≤ 0.005pp in all 31 years** —
the within-year path is geometrically anchored to the registered annual return by construction.

| statistic | value |
|---|---|
| worst absolute endpoint gap, 31 years | 0.0043 pp |
| years exceeding 0.05 pp | 0 |

The gate therefore verifies weights and path shape, **not** the annual returns. It cannot fail
for a correctly built run and should not be read as independent confirmation of the CAGR. Worth
recording in the gate's own description so the next reader does not over-trust it.

### Internal NAV inconsistency on the live row

`run_years` 2026 has `nav_strat` implying **+140.57%** while `strat_return` reads **143.00** — a
2.43pp gap, left over from the 2026-07-23 re-mark (event 45) not propagating into NAV. All 30
other years reconcile within 0.05pp.

### Registry and catalogue metadata

- **`runs.is_canonical = false`** on the flagship, while its catalogue row is `status: published`,
  `subscribable: true`, `sort_rank: 4`.
- **`catalog.caveats` has a single entry** — *"Concentrated: a single holding can move the
  yearly result materially."* Nothing on the live-year basis, the restatement, or the regime
  list. The run recipe requires known caveats be present, not omitted.
- **`catalog.last_year = 2025`** and `window_label = "1996–2025 (30 years)"`, so the live year
  is absent from the catalogue entirely — checklist item 8 (live year populated and its basis
  documented) fails at the catalogue layer.
- **`catalog.currency` is null.**
- **`catalog.filters_exact.universe_gates` claims "data_flag quarantine applied".** There is no
  `data_flag` column anywhere in `public.apex_screening_master` (213 columns, zero matching
  `%flag%` other than `estimate_shift_applied`), and the generator applies no such filter.
- **Three unreconciled risk triplets.** Catalogue: vol 36.5 / Sharpe 1.85 / Sortino 2.31.
  Event 209: sigma 28.6 / Sharpe 2.56. This audit, closed window, correct spine: vol 33.11 /
  Sharpe 2.09. Max drawdown agrees at −52.74 across all three; the volatility-based figures do
  not, and none states its basis.
- **The 2026 book is the only set of picks in the run with `mcap_usd` null** (7 of 193 rows), so
  the $5B floor — the defining change for the 2026+ era — is not evidenced in `picks`.
- **`run_years.regime` for 2026 is `'LIVE-OPEN ($5B, strict Normal)'`, not `NORMAL`.** Config
  declares that column authoritative for the branch, so any generator keying on
  `regime = 'NORMAL'` silently skips the live year. `bt.v_us14_cand` reads it as non-RECOVERY
  and emits 29 candidate rows for 2026 that the run does not use — the live year is served by
  the frozen table `bt.us14_cand26` (31 rows) instead.

---

## 4. Corrections to the project's own reference notes

The skill documentation for this project states figures that no longer hold:

| stated | measured 2026-08-11 |
|---|---|
| CAGR 73.19% (daily-NAV basis) | 73.17% and falling daily — it is an open-window figure, not a property of the run. Closed-window CAGR is 69.22%. |
| Annualised vol 34.1% | 33.11% on the closed window with `ppy = 365.3` |
| "Rank: `dcf_discount_percent desc`" | true for NORMAL years only; RECOVERY years rank by forward growth, per both `config` and the catalogue |

---

## 5. Recommended actions, in order

1. **Settle the live-2026 basis and write it through.** Event 546 already ruled: 1 April close
   to current close, from `historical_daily_prices`. Apply it to `bt.picks.fwd_return`,
   `bt.run_years.strat_return` and `nav_strat`, then re-derive the daily layer. Record a
   `corrected` event when done. This closes D1, D2 and the NAV inconsistency together.
2. **Rebuild the 2026 `btd_rel` rows per constituent** so the live year uses the same
   construction as every closed year and the weight audit can see it. Closes D3.
3. **Fix the catalogue's regime description** to state what the run actually does: eleven
   RECOVERY years assigned from a fixed list, with no mechanical US trigger available in the
   data. Then decide the governance question — either build a point-in-time US index series
   and derive the regime from it, or keep the list and reconsider `subscribable`. This is an
   owner decision, not an engineering one.
4. **Add the missing caveats** (live-year basis, regime list) and extend `last_year` / the
   window label to cover the live year.
5. **Reconcile the risk block** to a single stated basis, and record which one.
6. Decide `is_canonical` for the flagship, remove the unimplemented "data_flag quarantine"
   claim, and backfill `mcap_usd` on the 2026 picks.

Items 1, 2, 4, 5 and 6 are mechanical. Item 3 is not.
