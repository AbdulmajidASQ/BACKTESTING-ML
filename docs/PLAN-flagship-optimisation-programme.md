# Programme plan — raising return and cutting drawdown on the flagship

**Date:** 2026-08-11 · Run #14, `United States (US) · Conc-7`
**Objective:** materially raise compound return and materially reduce drawdown, with every claim
validated out of sample and nothing shipped on one lucky split.

---

## 0. Where we start

Verified this session, closed window 1996-04-01 → 2026-04-01, 30 years:

| metric | value |
|---|---|
| CAGR | **69.22%** (reproduces exactly from `run_years`) |
| Benchmark CAGR | 10.29% |
| Max drawdown | **−52.74%** — peak 2020-02-20, trough 2020-03-18, a **27-day** event |
| Annualised volatility | 33.11% (own spine, ppy 365.3 — never 252) |
| Sharpe, rf = 0 | 2.09 |
| Live 2026, restated to the Apr-1 basis | +102.66% as of 2026-08-10 |

### The mandate, and what it rules out

**Owner constraint, 2026-08-11: the book is 100% invested in equities at all times. No options of
any kind — not bought, not written. No cash allocation, no de-risking, no timing.**

That is a hard boundary and it closes three of the four instruments a portfolio normally uses
against drawdown:

| instrument | status |
|---|---|
| Reduce exposure / hold cash | **Excluded by mandate.** Also arithmetically self-defeating: `e* = μ/σ²` is **5.22** for this book, so any reduction below 100% costs compound growth |
| Time the market | **Closed by evidence.** ~130 states tested across 25 markets and 559 market-years; volatility predicts drawdown at p = 6.7 × 10⁻⁹ but returns at p = 0.41 |
| Buy convexity (long calls) | **Excluded by mandate**, though it was the only intervention in this project's record that improved return *and* drawdown at once |
| **Portfolio construction and selection** | **The only levers left** |

**The consequence has to be stated plainly: without convexity, return and drawdown are in direct
tension.** Concentration is what produces the return, and concentration is what produces the
drawdown. Diversifying cuts both. There is no free lunch left on the sizing side.

**One lever escapes the trade-off: selection quality.** Picking companies that are both cheaper and
more durable raises return *and* avoids disasters. That is why the selection workstream (M2)
carries most of the programme's weight, and why the drawdown workstream (M3) is now a
portfolio-construction exercise rather than a hedging one.

**Accepted limitation.** The universe is survivor-only (96.9% of 1996 names still present; no
holding in 193 ever lost more than 45.95%). Accepted by owner decision, recorded as run event 6.
Consequence for this plan: **every result is a delta against a control on the same pool, never a
level.** Re-measurement is scheduled for M4 when the panel is rebuilt.

---

## 1. Governing principles

Every failure in this project's history violated one of these. They are not negotiable.

1. **Learn where n is large; decide with fixed rules where n is small.** The cross-section is
   432,705 company-years. The decision layer is ~210 decisions. Annual re-selection among rankers
   scored 63.66 against an incumbent of 67.67; walk-forward half-Kelly sleeve sizing scored 147.7
   against a fixed rule's 185.7. Both learned in the small-n layer.
2. **One frozen model, not a per-period winner.** If a rule needs re-choosing each year, it is
   variance, not signal.
3. **Compare against the incumbent, never against zero.** For selection the control is
   rank-by-discount on the same pool. For the sleeve it is #77. For timing it is always-100%.
4. **Nothing ships on one split.** Out-of-sample, then multi-split, then permutation, then
   leave-one-market-out. In that order, with a kill criterion defined *before* the run.
5. **Every experiment is registered whether it wins or loses.** A negative result that isn't
   written down gets re-run by the next person, and eventually one of the re-runs gets lucky.

---

## 2. Milestone map

| # | Milestone | Depends on | Effort | Owner |
|---|---|---|---|---|
| **M0** | Close open defects — trustworthy baseline | — | small | me |
| **M1** | Research rig — one backtester, one validator, one registry | — | medium | me |
| **M2** | Selection alpha — the return half | M1 | large | me |
| **M3** | Portfolio construction — the drawdown half, 100% invested | M1, M0 | large | me |
| **M4** | Data foundation — point-in-time rebuild | — | large | data expert |
| **M5** | Ship, register, monitor | M2 or M3 | medium | me |

M0, M1 and M4 run in parallel starting now. M2 and M3 both gate on M1 and can then run
concurrently. M5 triggers per-result, not once at the end.

---

## M0 — Close the open defects

**Why first:** you cannot measure an improvement against a baseline that disagrees with itself.

| # | Task | Definition of done |
|---|---|---|
| M0.1 | Rebuild `btd_rel` for 2026 per constituent, not one pre-aggregated row | 7 rows per date, `v_btd_weight_audit` sees it, per-name contribution recoverable |
| M0.2 | Re-mark the 2026 daily layer onto the Apr-1 basis | `btd_daily` year-end agrees with the restated `run_years` 102.66 |
| M0.3 | Un-hardcode the live-year path in `bt.fn_btd_daily` so this cannot recur | a live year built by cron has the same shape as a closed year |
| M0.4 | Apply the filing-date guard in the generator | rows filed on/after 1 April excluded; guard exposed as a column, not silent |
| M0.5 | Catalogue corrections — regime description, caveats, `last_year`, `currency`, risk block basis | catalogue describes what the run actually does; caveats name the live-year basis and the regime list |
| M0.6 | Decide `is_canonical` for the flagship | set deliberately, either way |

**Kill criteria:** none — this is hygiene.
**Risk:** M0.3 touches shared machinery used by 59 runs with a daily layer. Change it behind a
reconciliation test across all of them, not just #14.

---

## M1 — The research rig

**Why:** every experiment so far has been bespoke SQL. That does not scale, and it makes results
hard to reproduce or compare. This milestone converts ad-hoc work into a platform, and everything
after it gets faster and more trustworthy.

| # | Task | Detail |
|---|---|---|
| M1.1 | Canonical backtest function | Input: pool definition, ranking expression, `top_n` rule, weighting. Output: annual returns, CAGR, MDD, vol, Sharpe, Sortino, per-year book. One implementation, used by every experiment. |
| **M1.1b** | **Daily paths for hypothetical books — critical path** | Today the daily layer exists only for books that were actually registered, so **no candidate rule's drawdown can be measured**. Annual-path drawdown is not a substitute: it reads −7.27% for run #14 against a true −52.74%, because the real damage occurred inside a single year (2020-02-20 → 2020-03-18). Build daily constituent paths for any hypothetical book from `public.historical_daily_prices` via `company_id`, forward-filled onto a common spine. **Under a 100%-invested mandate this is the single most important piece of infrastructure in the programme** — without it, drawdown is unmeasurable and half the objective is unfalsifiable. |
| M1.2 | Correct-by-construction statistics | CAGR from actual dates; `ppy` derived per run; exposure scales the **arithmetic** return; forward-fill onto a common spine before any weighted sum. Each of these has produced a wrong published number here before. |
| M1.3 | Multi-split validator | Rolling-origin: expanding-window fits with 5+ distinct test periods, not one 15/15 cut. Report the distribution of the edge, not a point estimate. |
| M1.4 | Permutation harness | For any episode- or regime-based claim, re-place episodes at random N times; report percentile and p-value. Project standard. |
| M1.5 | Purged + embargoed CV | April→April labels overlap; naive CV leaks. Purge the overlapping year, embargo the adjacent one. |
| M1.6 | Leave-one-market-out | The cross-sectional analogue, using the 25-market panel. Already the project's standard for the exposure study. |
| M1.7 | Experiment registry table | One row per rule tested: definition, control, train/test/multi-split results, verdict, date. RLS in the same batch. **Every negative result recorded**, so nothing is re-run by accident. |
| M1.8 | Backfill the registry | Load everything already tested: 5 alternative rankers, 92 exposure rules, the drop-largest-third result, the Phase 1 negatives (Sloan, Beneish, buyback yield, ROIC−WACC, Piotroski). |

**Definition of done:** any new ranking rule can be specified in one line and returns a validated
verdict with no bespoke SQL.
**Kill criteria:** none — this is infrastructure, and its payback is every milestone after it.

---

## M2 — Selection alpha (the return half)

**The prize, measured.** On the widened pool the span between a random draw and perfect foresight
is **61.30 CAGR points**; the incumbent captures 16.32, leaving **44.98 open**. On the incumbent
pool the open headroom is only 21.65. That is the entire case for working on the wider pool.

### M2.1 — Validate and ship the size rule *(next action)*

Current status: `drop the largest third by total assets, then rank by discount` scores **+2.56
train / +3.54 out of sample**. Mechanism established — dropping the largest third captures the
whole effect (75.42 test) while restricting to the smallest third *destroys* value (63.46 test,
8.4 points below incumbent). All market-cap floors unchanged and load-bearing.

| # | Task |
|---|---|
| M2.1.1 | Multi-split validation, 5+ rolling origins — is +3.5 the centre of a distribution or an artefact of one cut? |
| M2.1.2 | Permutation test — does it beat randomly-chosen exclusion sets of the same size? |
| M2.1.3 | Leave-one-market-out on the global panel — does "avoid the giants" replicate outside the US? |
| M2.1.4 | Sector-concentration check — the 2026 book is 7-for-7 Technology; does the rule make concentration worse? |
| M2.1.5 | Sensitivity to the cut point — third vs quartile vs half. A result that only works at exactly one-third is a fitted parameter. |
| M2.1.6 | Turnover and capacity — does it push the book into names that cannot absorb the capital? |

**Ship criterion:** positive in ≥4 of 5 splits, permutation p < 0.05, replicates in ≥60% of
LOMO folds, no material sector concentration increase, stable across cut points.
**Kill criterion:** fails multi-split, or the edge is concentrated in fewer than 3 years.
**Expected value:** +2 to +4 CAGR points, from a one-line generator change.

### M2.2 — Exhaust fixed formulas before fitting anything

| # | Task |
|---|---|
| M2.2.1 | `disc` residualised on `fwd` within year — the two gates correlate at ρ = 0.494, so half the "second signal" is the first one repeated |
| M2.2.2 | Re-test gross-profit-to-assets properly — strongest new standalone signal (+0.147, 19/26 years) but it *failed* validation (worst on train, best on test). Multi-split will settle whether it is real |
| M2.2.3 | `fwd` as rank key instead of `disc` — more consistent on sign (23/26 vs 19/26) and currently unused for ordering |
| M2.2.4 | Interaction terms: cheap × profitable, cheap × not-giant |
| M2.2.5 | Register every negative so they are never re-run |

**Why before the model:** if a two-term formula gets most of the available edge, a fitted model has
to beat *that*, and it usually cannot justify its complexity.

### M2.3 — Pool construction as a decision

The gates currently do two jobs: enforce economics *and* control book size. Those should be
separated and each measured.

| # | Task |
|---|---|
| M2.3.1 | Gate-by-gate marginal value — for each gate, measure the return contribution of the names it excludes |
| M2.3.2 | `fwd` threshold sweep — 25 → 15 → 10 → 5 → 0, measuring pool width and ranking edge at each |
| M2.3.3 | Revisit the EMA gate — it removes 43 of 423 candidates in NORMAL years; is it paying? |
| M2.3.4 | Revisit the 14 industry exclusions — the largest single cut in the funnel (55,490 from 99,754). Are all fourteen earning their place? |
| M2.3.5 | Market-cap floors — keep (they are load-bearing for the size rule) but measure their cost |

### M2.4 — The learning-to-rank model

Only after M2.2 and M2.3, with the bar set by whatever formula won.

| # | Task |
|---|---|
| M2.4.1 | Target: **within-year percentile** of forward return, not raw return. Fat tails; squared error chases outliers |
| M2.4.2 | Pairwise/listwise objective (LambdaMART or XGBoost `rank:pairwise`), year as query group |
| M2.4.3 | GBM first. Escalate to a neural architecture only if it demonstrably beats the GBM out of sample — on tabular data at this scale it usually will not |
| M2.4.4 | Train on the 432k global panel, deploy on the US pool. The panel has a real left tail — 21,091 outcomes at −50% or worse, minimum −100% — so distress features have signal to learn |
| M2.4.5 | Feature set from `bt.ml_us14_pool_v1`, prioritising the axis orthogonal to forward EPS |
| M2.4.6 | **One frozen model.** No annual re-selection. This is the discipline that the −4-point walk-forward failure violated |
| M2.4.7 | Ablation — which features carry the edge, and is it robust to dropping any one |
| M2.4.8 | Sanity gate: if the model cannot beat a two-term z-score formula out of sample, it does not ship |

**Expected value:** genuinely uncertain. The honest prior from the project's own record is that
capturing half the remaining headroom takes the pool-basis figure from ~68 to ~80. Treat +5 to +10
as success, and be prepared for zero.

---

## M3 — Portfolio construction (the drawdown half, 100% invested)

**Scope changed 2026-08-11.** This milestone previously covered the option sleeve. That is
withdrawn under the no-options mandate. Everything below keeps the book **100% invested in
equities at all times** and attacks drawdown purely through *how the book is built*.

**Set expectations honestly.** A fully-invested, concentrated equity book will absorb most of a
market crash. The benchmark fell roughly a third peak-to-trough in Feb–Mar 2020; this book fell
52.74%. Nothing in this milestone changes that structurally. Realistic ambition is **a few points
of drawdown, and most of it bought with return** — except where selection quality improves both.
Anyone expecting −52% to become −25% without hedging or de-risking is expecting arithmetic to bend.

### M3.0 — Prerequisite: make drawdown measurable (M1.1b)

Nothing in M3 can start until daily paths can be built for hypothetical books. This is now the
gating dependency for half the programme.

### M3.1 — Drawdown taxonomy *(first — it scopes everything after)*

| # | Task |
|---|---|
| M3.1.1 | Enumerate every drawdown >20% on the daily series: peak, trough, depth, **duration**, recovery time |
| M3.1.2 | Classify fast-crash vs slow-bear. The maximum is a 27-day COVID event |
| M3.1.3 | **Attribution — the key question.** Decompose each drawdown into market beta, sector concentration, and single-name blow-ups. Only the last two are addressable under this mandate; if the drawdowns are overwhelmingly market beta, M3's realistic ceiling is small and we should say so and redirect effort into M2 |

### M3.2 — Position count

`top_n = 7` (5 in RECOVERY) is a choice, never tested against alternatives.

| # | Task |
|---|---|
| M3.2.1 | Sweep n from 5 to 20 on the widened pool, reporting **CAGR and daily MDD jointly** |
| M3.2.2 | Map the efficient frontier — how many CAGR points does each point of drawdown cost? |
| M3.2.3 | Test whether the answer differs by regime; RECOVERY currently runs *more* concentrated (5), which is the opposite of what risk control would suggest |
| M3.2.4 | Idiosyncratic vs systematic decomposition — diversification only helps the former, and there is a point past which added names buy nothing |

### M3.3 — Weighting

Equal weight is also a choice, and a weighting change keeps the book fully invested by construction.

| # | Task |
|---|---|
| M3.3.1 | Inverse-volatility weighting — same 100% exposure, lower portfolio variance |
| M3.3.2 | Rank-based weighting — more into higher-conviction names. Likely raises both return and drawdown; measure the trade |
| M3.3.3 | Volatility-capped weights — cap any single name's *risk* contribution rather than its dollar weight |
| M3.3.4 | Constraint: weights must sum to 1.0 with no cash residual. Any scheme leaving a residual is out of mandate and would also trip `v_btd_weight_audit` |

### M3.4 — Sector and correlation constraints

**The most promising item in M3.** The live 2026 book is **7 of 7 Technology** — SNDK, MU, STX, WDC,
CRDO, AMD, MRVL, essentially one bet on memory and semiconductors. That is a concentration risk the
strategy never explicitly chose.

| # | Task |
|---|---|
| M3.4.1 | Measure historical sector concentration by year — how often has the book been effectively a single sector? |
| M3.4.2 | Test a cap of max 2, 3 or 4 names per sector, filling from the next-ranked candidate. Fully invested throughout |
| M3.4.3 | Measure the cost in CAGR and the gain in drawdown; this is the cleanest frontier trade in the programme |
| M3.4.4 | Test a pairwise-correlation cap as an alternative to sector labels, which are a crude proxy for what we actually mean |
| M3.4.5 | Check interaction with M2's size rule — dropping the largest third may *increase* sector concentration, and the two rules must be tested jointly, not separately |

### M3.5 — Fragility screening

Reduce drawdown through *what is bought* rather than how much. This overlaps M2 and is the one
place where both objectives move together.

| # | Task |
|---|---|
| M3.5.1 | Re-test the quality features against **drawdown** rather than return. Altman Z, Piotroski and Beneish all failed as *return* predictors — they are distress measures and were being graded on the wrong outcome |
| M3.5.2 | Tighten leverage gates and measure the drawdown/return trade explicitly |
| M3.5.3 | Beta and realised-volatility screens at selection — available in the feature table already |
| M3.5.4 | Test a maximum-drawdown-history screen: do names that previously fell hardest keep doing so? |

**M3.5.1 is important and cheap.** Those three features were rejected this session for failing to
predict returns. Under the new mandate the question is whether they predict *disasters*, which is
what they were designed for. Re-grading them costs one experiment.

---

## M4 — Data foundation *(parallel — data expert)*

Full specification in [`DATA-REQUEST-ml-features.md`](DATA-REQUEST-ml-features.md). Priority order,
revised after this session's audits:

1. **Point-in-time universe rebuild** — every company listed and meeting the data requirements as
   at each 1 April, whether or not it exists today, keyed on a **stable company identifier, not a
   ticker string**. Plus delisting date, reason, and a terminal return for names that die
   mid-holding-year. Without a terminal return a failed position has no label and vanishes again.
2. **Point-in-time identity** — `company_name`, `sector`, `industry`, `country`, `is_adr` as at
   each rebalance. Proven broken: ticker `WBD` in performance year 2008 is named "Warner Bros.
   Discovery Inc", an entity created in 2022. These fields drive the dedup key and three exclusions.
3. **Estimate revision history** — `eps_f1`/`eps_f2` as at T−30/−90/−180 days, analyst count,
   estimate dispersion, revision counts. The DCF discount is a *shadow* of estimates; revisions are
   the thing itself. Highest-value genuinely new data.
4. **Tier 0 feature view** — mostly done; I built `bt.ml_us14_pool_v1` covering the priority subset.
   The remaining ~100 columns of the `Annual*` family can be exposed the same way.
5. **DCF assumptions** — discount rate, terminal growth, horizon, method. Needed to know how much
   of `disc` is independent judgement versus a restatement of `fwd` (they correlate at 0.494).

**When M4.1 lands, everything in M2 and M3 gets re-measured on the rebuilt panel.** The delta
between old and new is the survivorship correction and is the single most valuable number this
project can produce.

---

## M5 — Ship, register, monitor

Per-result, not once at the end. For each adopted change:

| # | Task |
|---|---|
| M5.1 | Register as a **new run**, never overwrite #14 — the two must stay comparable |
| M5.2 | Full checklist: `runs`, `run_years` incl. idle years, `picks`, daily NAV chain, `catalog`, `run_events`, live year, audit gates |
| M5.3 | Catalogue written for a subscriber, with every known caveat present, not omitted |
| M5.4 | Live-vs-backtest monitoring — does realised performance track the backtest, and at what tracking error |
| M5.5 | A pre-committed review trigger: if live underperforms the backtest by more than X over Y months, the rule is re-examined |

---

## Decision gates

| Gate | Question | If no |
|---|---|---|
| G1 | Does the size rule survive multi-split, permutation and LOMO? | Do not ship; Phase 2's bar reverts to the incumbent |
| G2 | Does any fixed formula beat the incumbent robustly? | Question whether a fitted model will do better — it usually won't |
| G3 | Does the learned ranker beat the best fixed formula out of sample? | Do not ship the model. A negative here is a real result |
| G4 | Are the drawdowns driven by sector concentration and single-name blow-ups, or by market beta? | If overwhelmingly beta, M3's ceiling is small under a 100%-invested mandate. Say so, cap the effort, redirect into M2 |
| G5 | Does any construction change cut drawdown at an acceptable cost in CAGR? | Accept the current risk profile as the price of concentration, and state it plainly in the catalogue |

---

## Risk register

| Risk | Mitigation |
|---|---|
| Overfitting through repeated testing on one dataset | Experiment registry (M1.7) makes the multiple-comparisons problem visible; pre-committed kill criteria |
| Survivorship inflates everything | Accepted; all results are deltas; re-measure at M4 |
| Small-n whipsaw in the decision layer | One frozen model; fixed mapping from forecast to weight |
| Shared-machinery changes break other runs | M0.3 behind a reconciliation test across all 59 daily-layer runs |
| Sector concentration — the book is 7-for-7 Technology | Explicit check at M2.1.4 and again before any ship |
| Reopening closed questions | Timing is closed. ~130 states, 559 market-years. Do not revisit without extraordinary new data |

---

## What "exceed" looks like

Under a 100%-invested, no-options mandate the upside is narrower and comes from three places:

1. **Selection quality is the only lever that improves both objectives at once.** Everything else
   in M3 is a frontier trade — pay return, get drawdown. A better-chosen book is cheaper *and*
   more durable. This is why M2 carries most of the programme's weight and why M3.5 (re-grading the
   quality features against drawdown instead of return) is disproportionately valuable for its cost.
2. **Sector concentration is an unpriced risk the strategy never chose.** The live book is 7 of 7
   Technology. If the drawdown attribution at M3.1.3 shows concentration is a material contributor,
   a sector cap is the cheapest real drawdown reduction available — and it costs less return than
   simply holding more names.
3. **Whatever works on the US pool can be tested across 25 markets.** The infrastructure exists —
   `lp2_cand`, `lp2_sel`, 83 registered runs. A rule that replicates cross-market is worth far more
   than one that works in one country, and leave-one-market-out validation gives it for free.
   Estimate revisions (M4.3) are the one genuinely new signal axis available, since every signal in
   the book today is a derivative of forward EPS estimates.

**Honest expectation.** Selection: +5 to +10 CAGR points, with wide error bars. Drawdown: a few
points from construction, most of it paid for in return, with the exception of whatever fragility
screening delivers. **−52.74% is not going to become −25% without hedging or de-risking, and no
amount of modelling will change that.** The realistic outcome is a book that earns more per unit of
the same risk, not one that carries materially less risk.

Anyone promising more than that from this data is selling the walk-forward result that already came
in four points behind the incumbent.
