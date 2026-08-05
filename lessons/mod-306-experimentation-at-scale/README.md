# mod-306-experimentation-at-scale: Experimentation at Scale — A/B, Variance Reduction, Causal Framing

**Estimated effort:** 12 hours

At L20 an A/B test is a treatment column and a p-value at the bottom of a notebook. At L30 an A/B test is a **contract** — a hypothesis, a primary metric, a guardrail set, a power / MDE design, a randomisation unit, a ramp plan, and a stopping rule — plus the trust diagnostics that decide whether the number the analysis reports is *readable at all*. This module installs that discipline for the experimentation stage that mod-305 chapter 03 handed off as *the A/B stage of the promotion path*. It also covers the two levers a senior ML engineer reaches for when the naive design is under-powered (variance reduction and sequential testing) and the fallback framing when an A/B is genuinely impossible (causal inference).

The four sub-topics — designing an experiment with power and ramp, diagnosing SRM / interference / novelty, applying variance reduction, and framing observational questions causally — are the pieces of the *release-blocking A/B stage* that a senior ML engineer holds a candidate model to. They also give the vocabulary the online promotion path (mod-305 chapter 03) uses when it hands a candidate to the experimentation platform.

This module is the sibling of [mod-303](../mod-303-advanced-modeling/) (the modeling regime that produced the candidate), [mod-304](../mod-304-production-llm-integration/) (the LLM-augmented candidate whose cost / latency envelope the guardrail set inherits from), [mod-305](../mod-305-advanced-evaluation/) (the offline harness and online promotion path this A/B stage is the last gate of), [mod-307](../mod-307-ml-reliability-slos/) (the SLIs guardrails are named against), and [mod-309](../mod-309-responsible-ai-governance/) (the slice policy re-enforced online).

## Learning objectives

- Design A/B tests with correct power / MDE calculations, primary / guardrail metrics, and ramp plans.
- Diagnose sample ratio mismatch, interference, and novelty effects.
- Apply variance reduction (CUPED / CUPAC) and sequential-testing framings.
- Frame observational questions as causal-inference problems (DID, PSM) when an A/B test is impossible.

## Chapters

1. [`01-experiment-design-power-mde-and-ramp.md`](01-experiment-design-power-mde-and-ramp.md) — the seven-piece experiment contract (hypothesis, primary metric, guardrails, power / MDE design, randomisation unit, ramp plan, stopping rule); the `n ≈ 16 σ² / Δ²` back-of-envelope; the three interpretations of a flat experiment; the randomisation-unit / analysis-unit rule; the ramp ladder and the 50 %-to-100 % question.
2. [`02-trust-diagnostics-srm-interference-novelty.md`](02-trust-diagnostics-srm-interference-novelty.md) — SRM as the load-bearing trust check that suppresses the primary read; the interference taxonomy (marketplace, network, shared-budget) and its cluster / switchback / ego-network mitigations; novelty and primacy effects and how a hold-back cohort catches the "day-1 lift that fades" pattern.
3. [`03-variance-reduction-cuped-cupac.md`](03-variance-reduction-cuped-cupac.md) — CUPED's `Y − θ (X − mean X)` construction and its `1 − ρ²` variance shrink; CUPAC as the regression generalisation; stratification, winsorisation, and the delta method for ratio metrics; where each composes, and what variance reduction pointedly does not fix.
4. [`04-sequential-testing-and-peeking.md`](04-sequential-testing-and-peeking.md) — why peeking at a fixed-horizon test inflates α; always-valid inference via Group Sequential Tests, mSPRT, and betting-based confidence sequences; where Bayesian framings help and where they don't; the peeking-rule policy a senior ML engineer commits to before starting an experiment.
5. [`05-causal-inference-when-ab-is-impossible.md`](05-causal-inference-when-ab-is-impossible.md) — the potential-outcomes framing; propensity-score matching / IPW under conditional ignorability; difference-in-differences under parallel trends (and the staggered-rollout complication); instrumental variables under the exclusion restriction; regression discontinuity under non-manipulation; synthetic control for single-treated-unit rollouts; the "randomise the decision, the order, or the default" rescues to try *before* reaching for observational inference.

## Exercises

- [`exercises/exercise-01-power-mde-and-ramp-plan.md`](exercises/exercise-01-power-mde-and-ramp-plan.md) — design the seven-piece contract for an A/B test on a chosen feature, from hypothesis and MDE through ramp plan and stopping rule.
- [`exercises/exercise-02-srm-and-trust-diagnostics.md`](exercises/exercise-02-srm-and-trust-diagnostics.md) — implement (or spec) an SRM check, an interference diagnostic, and a novelty-decay analysis on synthetic or real experiment data; author the trust-scorecard the experimentation platform enforces.
- [`exercises/exercise-03-variance-reduction-with-cuped.md`](exercises/exercise-03-variance-reduction-with-cuped.md) — implement CUPED on a real or synthetic pre-experiment metric, measure the achieved variance reduction, and rewrite exercise-01's MDE calculation with the CUPED-adjusted σ.
- [`exercises/exercise-04-causal-inference-when-ab-is-impossible.md`](exercises/exercise-04-causal-inference-when-ab-is-impossible.md) — for a scenario where A/B is not available, pick a causal-inference method (PSM, DID, IV, or RDD), spec the estimator + diagnostics, and defend the assumption load in a release memo.

## Labs & quizzes

- `labs/` — reserved for a longer-form hands-on lab (end-to-end experiment design + trust + variance reduction + read) in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — Kohavi/Tang/Xu's *Trustworthy Online Controlled Experiments*, the CUPED and CUPAC papers, the peeking / always-valid literature, the DID / PSM / IV / RDD canon, and the peer-track pointers — are catalogued in [`resources.md`](resources.md).

## How this module hands off

- The seven-piece experiment contract (chapter 01) is what mod-305 chapter 03's "A/B or interleaving" stage runs. `passes_release_gate` becomes online: primary lift *and* every guardrail non-inferior.
- The SRM discipline (chapter 02) is the load-bearing precondition every scorecard the release meeting reads has to pass. It is *not* a statistics exercise — it is the reason the *other* numbers are readable.
- The variance-reduction machinery (chapter 03) shrinks the MDE the experimentation platform can commit to, which shortens the ramp step at full power. Every downstream chapter's MDE assumes the CUPED-adjusted σ.
- The sequential-testing framings (chapter 04) are what the experimentation platform offers when the team's cultural pattern of peeking is the failure mode.
- The causal-inference toolkit (chapter 05) is the fallback framing mod-309 (Responsible AI) leans on when a randomised A/B is not available and a decision still has to be made — a fair-lending model change, a pricing policy, a regulator-visible rule change.
- The guardrail set (chapter 01) borrows its cost / latency envelope from [mod-304 chapter 04](../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) and its SLI vocabulary from [mod-307](../mod-307-ml-reliability-slos/); its slice matrix is the online enforcement of the [mod-305 chapter 02](../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) slice policy.
