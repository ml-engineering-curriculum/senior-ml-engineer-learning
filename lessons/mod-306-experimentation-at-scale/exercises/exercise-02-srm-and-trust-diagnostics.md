# exercise-02: SRM and Trust Diagnostics

**Estimated effort:** 3 hours

## Objective

Build the **trust layer** an experimentation platform runs on every experiment scorecard, at senior altitude. Implement (or spec) the three diagnostics chapter 02 makes load-bearing — SRM, interference sanity, novelty / primacy decay — on synthetic or real experiment data, and author the trust-scorecard the platform enforces so that a red trust check *suppresses* the primary-metric read rather than being reported alongside it.

This is the L30 tell that separates "the A/B lifted 4 %" from "the A/B's trust diagnostics were green, so the 4 % lift is a lift." An L20 reports the primary; an L30 gates on the trust layer before the primary is ever read.

## Prerequisites

- Read chapter 02 (`02-trust-diagnostics-srm-interference-novelty.md`) — the SRM chi-squared threshold, the interference taxonomy, and the novelty-decay diagnostics are load-bearing.
- Read chapter 01 (`01-experiment-design-power-mde-and-ramp.md`) — the randomisation-unit and ramp-step structure the diagnostics run against.
- Skim mod-305 chapter 03 for the online promotion context: SRM at the top of a scorecard is the same discipline as an auto-rollback controller — a *blocking* precondition, not a metric.

## Pick your data

Pick **one**:

- **D1 — Synthetic.** Simulate an experiment with `N ≈ 200 000` users, 50/50 split, one continuous primary metric with `ρ = 0.6` correlation to a pre-experiment covariate, one binary conversion metric. Simulate three variants:
  - *Healthy.* Bucket split is exact 50/50 (up to noise). Effect is +2 %. No interference. No novelty.
  - *SRM-injected.* Deliberately drop 3 % of treatment log rows (a plausible log-pipeline drop). Effect is truly +2 %.
  - *Novelty-injected.* Day-1 effect is +5 %, decays exponentially to +0 % by day 14.
- **D2 — Public dataset.** Use a public experiment dataset such as the [Upworthy Research Archive](https://osf.io/jd64p/) headline-testing dataset or the Recsys Challenge datasets. Author the diagnostics against the real assignment column and see what fires.
- **D3 — Bring-your-own.** An anonymised experiment log from your work with a bucket column, per-user metrics, and a pre-experiment covariate. Anonymise IDs; keep only the columns the diagnostics need.

The synthetic path (D1) is the fastest and lets you *verify* that your diagnostics catch what they should — recommended if you have not built these before.

## Steps

### 1. Implement the SRM check (≈ 30 min)

Write a function `srm_check(observed_counts, design_ratio, threshold=1e-3) -> SRMResult` that:

- Runs a chi-squared goodness-of-fit test against the design ratio.
- Returns the observed counts, expected counts, chi-squared statistic, p-value, and a `bool` `fired`.
- Fires when `p < threshold`. Use `1e-3` as the default; make it configurable.

Run it against all three synthetic variants (or all your D2/D3 experiments). Expected behaviour: SRM fires on the SRM-injected variant, does not fire on healthy or novelty-injected variants.

Add three unit tests:

- Perfect 50/50 counts on 100 K users → `fired=False`.
- 51.5/48.5 counts on 10 M users → `fired=True` (should trigger with p far below 10⁻⁶).
- 90/10 on a design ratio of 50/50, small N — check the test's sensitivity at low N.

### 2. Implement a cross-arm parity check (≈ 20 min)

A helper diagnostic to catch the SRM cause when SRM fires — asymmetries in metrics *unrelated* to the treatment that suggest something in the pipeline is arm-dependent. At minimum:

- **Trigger rate** parity — of users who entered the experiment, what fraction actually saw the treated surface? Should be near-identical across arms.
- **Error rate** parity — 5xx rate, timeout rate, or event-drop rate per arm. Should be near-identical.
- **Session start time distribution** parity — a KS-test on session start times per arm, or a simple bucket-count comparison.

Return the per-metric result as part of the trust scorecard.

### 3. Implement a novelty-decay diagnostic (≈ 45 min)

Write a function `novelty_check(events, treatment_col, metric_col, day_col) -> NoveltyResult` that:

- Computes the treatment effect (difference in means) *per day* over the experiment window.
- Reports the day-1, day-7, and day-14 (or window-appropriate) effect estimates.
- Reports a decay-ratio (final-window effect / initial-window effect); flags a suspicious pattern when the ratio is below a threshold (0.5 is a defensible default).
- Optionally fits an exponential-decay model `effect(t) = α + β * exp(-λ * t)` and reports the plateau estimate `α`.

Run it against the healthy and novelty-injected synthetic variants. Expected behaviour: healthy shows a roughly flat effect line; novelty-injected shows a clear decay to zero by day 14.

Bonus: split treated users by exposure-recency cohort (users who first saw treatment on day 1, day 4, day 8...) and compare each cohort's *day-1-of-exposure* effect to the same cohort's *day-7-of-exposure* effect. This is the *within-cohort* novelty read chapter 02 §3 describes.

### 4. Reason through an interference diagnostic (≈ 30 min)

Interference is harder to detect than SRM or novelty — the standard diagnostics are *design* diagnostics (was this experiment run on a marketplace surface with user-level randomisation) plus *empirical* checks (does the effect shrink as exposure grows). For this step, write:

- A short (half-page) *checklist* — five questions the experimentation platform asks the analyst at experiment creation. Sample questions: "does the treated unit share a resource, market, or graph with control units?" "does the treatment change the shared state visible to control?" "is the treatment large enough that the second-order effect is material?" The answers are recorded on the experiment; the platform routes marketplace / social / budget experiments to a *cluster / switchback* design template.
- If D1/D2/D3 has ramp-step lift data (multiple exposure percentages), plot the per-step effect and comment on whether the shape is consistent with interference (shrinking effect with increased exposure) or not (flat effect).

You are not expected to implement a graph-cluster-randomization scheme; the deliverable is the *design gate* an experimentation platform uses to catch the pattern.

### 5. Author the trust scorecard (≈ 30 min)

Produce a rendered scorecard for each of the synthetic variants (or your D2/D3 experiments) in the shape chapter 02 §4 shows:

```
Experiment: <id>
Status: <READY TO READ | TRUST FAILURES — DO NOT READ>

Trust diagnostics
  SRM         — chi-squared p, observed vs expected, PASS/FAIL
  A/A parallel — <if run>, PASS/FAIL
  Cross-arm parity — trigger rate, error rate, PASS/FAIL
  Novelty check — day-1, day-7, day-14 effects; decay-ratio; PASS/FAIL
  Interference — ramp-step lifts if available; commentary
Primary metric — SUPPRESSED if trust FAILED, else <lift + CI>
Guardrail metrics — SUPPRESSED if trust FAILED, else <lift + CI per metric>
```

The suppression behaviour is a *policy*, not a suggestion. If your implementation is a notebook, the suppression is a large red banner and the metric cells left blank; if it is a service, the API omits the metric fields.

### 6. Write the runbook (≈ 15 min)

One page: when a diagnostic fires, what does the on-call do? For each diagnostic:

- The signal (which alert fired, threshold, actual value).
- The first three steps of triage — the *specific* things to check (bucketing service telemetry, log-pipeline drop counts, trigger-rate per arm).
- The escalation — who is paged, who is on the call, who has authority to override / continue the experiment (usually nobody, until the diagnostic is explained).

Runbooks are the difference between "we caught SRM" and "we caught SRM, root-caused it, fixed the pipeline, and re-ran the experiment cleanly."

## Deliverable

A folder `trust-layer/` (or a Markdown doc with sections inlined) containing:

- `srm.py` (or pseudocode) — `srm_check`, with three unit tests.
- `parity.py` — cross-arm parity checks.
- `novelty.py` — `novelty_check` and (optional) exponential-decay fit.
- `interference_checklist.md` — the design-time questions.
- Rendered scorecards for each experiment variant.
- `runbook.md` — the on-call runbook.

## Acceptance criteria

- [ ] `srm_check` is implemented, threshold-configurable, and returns a structured result including chi-squared statistic and observed / expected counts.
- [ ] SRM unit tests exist and cover: healthy split, injected SRM, and low-N sensitivity.
- [ ] SRM correctly fires on the SRM-injected experiment and correctly does not fire on the healthy experiment.
- [ ] Cross-arm parity checks cover trigger rate and error rate at minimum, and are reported in the scorecard.
- [ ] `novelty_check` computes per-day effects and a decay-ratio, and correctly flags the novelty-injected experiment while passing the healthy one.
- [ ] The interference checklist has at least five design questions and is written from the perspective of a platform's experiment-creation flow.
- [ ] The trust scorecard suppresses the primary and guardrail metrics when any trust diagnostic fails; the suppression is a policy, not a display suggestion.
- [ ] The runbook covers all three diagnostics (SRM, parity, novelty) with signal + triage steps + escalation, and is at most one page.
- [ ] Nothing in the deliverable reads a primary-metric lift on a trust-failed experiment without a *record* of the override.

## Stretch goals

- **Real-time SRM.** Implement SRM as a *streaming* diagnostic — it runs every hour of the experiment against cumulative counts and fires as soon as the p threshold is crossed. Include a hold-time (e.g., "fire only if SRM has held for three consecutive hourly checks").
- **A/A test corpus.** Simulate 200 A/A experiments and report the empirical false-positive rate of your SRM check at your chosen threshold. It should be at or below the threshold; if it is much higher, the threshold is mis-set.
- **Interference simulation.** Simulate a marketplace with `N` sellers and `M` buyers. Give treated sellers a boost; measure the naive per-user difference-in-means and compare it to a switchback design's estimate. Report the interference bias.
- **CUPED handoff.** After the trust layer is green, run the primary-metric analysis with a CUPED-adjusted metric (feed into exercise-03). Show that the CI narrows by the expected factor.
- **Platform integration.** Wire the trust layer as a check in the experimentation platform your team uses (Statsig, Eppo, Optimizely, an internal one). The check publishes a "trust status" that gates the scorecard.
