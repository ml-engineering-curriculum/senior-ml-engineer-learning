# mod-305-advanced-evaluation: Advanced Evaluation & Production-Readiness Reviews

**Estimated effort:** 12 hours

At L20 an evaluation is a notebook. At L30 an evaluation is a **program** — a harness that runs on every candidate model, produces a structured decision object, gates a release, and hands off cleanly to an online promotion path where an offline win still has to earn its shipment. This module installs that discipline. It replaces "the numbers look good in my notebook" with "the harness produced a `HarnessDecision`, every slice passed, every guardrail is green, the shadow run is clean, and the canary controller has a rollback threshold on every SLI."

The four sub-topics — offline harness shape, slices and adversarial guardrails, shadow-to-canary online promotion, the Google ML Test Score rubric — are the pieces of a *production-readiness review* a senior ML engineer holds a release to. LLM-as-judge is the specific extension the module adds for LLM-augmented systems (the ones mod-304 stood up), because a summariser and a draft-reply generator do not have programmatic metrics that a harness can score without a judge in the loop.

This module is the sibling of [mod-303](../mod-303-advanced-modeling/) (the modeling regime that produced the candidate), [mod-304](../mod-304-production-llm-integration/) (the LLM-augmented candidate whose prompt bundles now need harnessing), [mod-306](../mod-306-experimentation-at-scale/) (the depth of the A/B stage the online promotion path passes through), [mod-307](../mod-307-ml-reliability-slos/) (the SLIs the online guardrails are named against), and [mod-309](../mod-309-responsible-ai-governance/) (the slice policy the harness enforces).

## Learning objectives

- Design an offline evaluation harness with slice, holdout, and adversarial guardrails a team runs on every candidate model.
- Set shadow and canary deploy paths so an offline win must earn its online promotion.
- Apply the Google ML Test Score rubric during production-readiness reviews.
- Bring LLM-as-judge into the harness where warranted and know its failure modes.

## Chapters

1. [`01-offline-eval-harness-shape.md`](01-offline-eval-harness-shape.md) — the harness as a program with four inputs (candidate, baseline, eval config, statistical shape) and one output (a `HarnessDecision`); paired bootstrap CIs, versioned eval sets, primary-metric-chosen-in-advance discipline.
2. [`02-slice-and-adversarial-guardrails.md`](02-slice-and-adversarial-guardrails.md) — the slice matrix (business-critical, fairness, error-concentration), per-slice pass rates, the append-only adversarial / regression suite, and the small set of guardrail metrics (calibration, worst-group, refusal rate, cost, latency, robustness) that gate the harness.
3. [`03-shadow-and-canary-online-promotion.md`](03-shadow-and-canary-online-promotion.md) — the promotion ladder (shadow → canary → A/B or interleaving → full rollout), what each stage uniquely proves, auto-rollback controllers, and the exit criteria that make a release defensible.
4. [`04-ml-test-score-production-readiness.md`](04-ml-test-score-production-readiness.md) — Breck et al.'s 28-test rubric across data, model, ML infrastructure, and monitoring; the weakest-link scoring rule; how to use the rubric as a review packet and not a checklist.
5. [`05-llm-as-judge-when-and-how.md`](05-llm-as-judge-when-and-how.md) — the pointwise / pairwise / reference-based shapes, the four conditions LLM-as-judge belongs at all, the four documented failure modes (position bias, verbosity bias, self-preference, style-over-substance / rubric drift), and calibrating the judge against a human-labelled set.

## Exercises

- [`exercises/exercise-01-offline-harness-with-slices-and-guardrails.md`](exercises/exercise-01-offline-harness-with-slices-and-guardrails.md) — design and (optionally) implement the offline harness for a chosen feature: eval config, slice matrix, adversarial suite, guardrail set, and a `HarnessDecision` object.
- [`exercises/exercise-02-shadow-and-canary-deploy-design.md`](exercises/exercise-02-shadow-and-canary-deploy-design.md) — author the online promotion plan for a candidate model, from shadow entry criteria through the canary ladder to auto-rollback SLIs.
- [`exercises/exercise-03-ml-test-score-review-packet.md`](exercises/exercise-03-ml-test-score-review-packet.md) — score a system against the 28-test rubric, produce a review packet with per-test evidence, and name the three next-quarter investments.
- [`exercises/exercise-04-llm-as-judge-when-and-how.md`](exercises/exercise-04-llm-as-judge-when-and-how.md) — design (or defend against) an LLM-as-judge for an LLM-augmented feature, including the human-labelled calibration set and the failure-mode gates.

## Labs & quizzes

- `labs/` — reserved for a longer-form harness-authoring lab in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — the ML Test Score paper, MT-Bench / LLM-as-judge references, Google SRE canary and SLO references, statistical-inference references, and the peer-track pointers — are catalogued in [`resources.md`](resources.md).

## How this module hands off

- The `HarnessDecision` object from chapter 01 is the release contract [mod-304 chapter 02](../mod-304-production-llm-integration/02-prompts-as-engineering-artifacts.md) *promised* — every prompt-bundle change now has a defensible pass/fail gate.
- The shadow and canary path (chapter 03) is where the LLM-augmented cost / latency envelope from [mod-304 chapter 04](../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) and the observability from [mod-304 chapter 05](../mod-304-production-llm-integration/05-observability-for-llm-augmented-systems.md) first meet real traffic; regressions here re-feed the harness.
- The ML Test Score rubric (chapter 04) is the review packet [mod-309](../mod-309-responsible-ai-governance/) leans on for the responsible-AI review a regulated deployment has to survive.
- The A/B stage of chapter 03 is the depth [mod-306](../mod-306-experimentation-at-scale/) picks up — variance reduction, sequential testing, interaction effects at experimentation-track depth.
- The auto-rollback SLIs from chapter 03 are the alerting layer [mod-307](../mod-307-ml-reliability-slos/) authors at reliability-track depth.
- LLM-as-judge (chapter 05) delegates to peer tracks `ai-eval-engineer-learning` and `model-evaluation-engineer-learning` when the judge design or the eval-set governance is deeper than a senior ML engineer's day-to-day; the delegation-contract shape lives in [mod-304 chapter 06](../mod-304-production-llm-integration/06-delegation-contract-to-llm-specialists.md).
