# mod-302-ml-systems-architecture: ML Systems Architecture at Senior Altitude

**Estimated effort:** 16 hours

This is the module where you learn to take a business problem all the way to an ML system architecture a team can operate for months — and to write it down in a document a peer L30/L40 will sign. It is the load-bearing module of the track: the decisions you learn to make here recur in evaluation (mod-305), reliability (mod-307), and cross-team collaboration (mod-308), and they are the deliverable of the paired project (`project-301-ml-system-design-portfolio`).

At level 20 you built ML systems. At level 30 you decide their shape. This module teaches the four load-bearing decisions — serving posture, feature substrate, model artefact contract, change-management posture — and the artefact (the RFC) where those decisions get written down and defended.

## Learning objectives

- Take a business problem to a defensible ML system architecture — batch vs. streaming, online vs. offline features, feature store vs. ad-hoc, model registry as source of truth.
- Diagnose and mitigate training/serving skew at architectural altitude, not just at code review.
- Design retraining cadence, data flywheel, and shadow/canary deploy paths that a team can operate for months.
- Write an ML system RFC with alternatives, trade-offs, and non-goals — and defend it in an architecture review.

## Chapters

1. [`01-from-problem-to-system-shape.md`](01-from-problem-to-system-shape.md) — the four load-bearing decisions and the six-step walk from a business problem to a defensible system shape.
2. [`02-batch-streaming-online-offline.md`](02-batch-streaming-online-offline.md) — the two axes people conflate (serving posture and feature freshness) and the six-question rubric for picking each.
3. [`03-feature-store-and-model-registry.md`](03-feature-store-and-model-registry.md) — feature substrate (feature store vs. ad-hoc) and the model artefact contract (registry as source of truth).
4. [`04-training-serving-skew.md`](04-training-serving-skew.md) — the five categories of skew, the architectural mitigation for each, and the parity checklist an RFC owes.
5. [`05-retraining-and-rollout.md`](05-retraining-and-rollout.md) — retraining triggers and cadence, data flywheels (and how they fail), and the shadow/canary/ramp/kill-switch toolkit.
6. [`06-writing-and-defending-the-rfc.md`](06-writing-and-defending-the-rfc.md) — the RFC skeleton, what reviewers actually read for, and how to hear "no" without churn.

## Exercises

- [`exercises/exercise-01-batch-vs-streaming-decision.md`](exercises/exercise-01-batch-vs-streaming-decision.md) — apply the chapter-02 six-question rubric to a real business scenario and defend the batch/online/streaming decision.
- [`exercises/exercise-02-feature-store-adoption-tradeoff.md`](exercises/exercise-02-feature-store-adoption-tradeoff.md) — walk the chapter-03 adoption signals and produce a defensible "feature store vs. ad-hoc" trade-off memo.
- [`exercises/exercise-03-training-serving-skew-architectural-fix.md`](exercises/exercise-03-training-serving-skew-architectural-fix.md) — diagnose a skew incident against the chapter-04 five-category taxonomy and design the *architectural* fix, not just the code fix.
- [`exercises/exercise-04-retraining-cadence-and-data-flywheel.md`](exercises/exercise-04-retraining-cadence-and-data-flywheel.md) — design a retraining trigger set and a defensible data flywheel for a system you know or one of the case scenarios.

## Labs & quizzes

- `labs/` — reserved for a longer-form design lab in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — books, papers, platform documentation — are catalogued in [`resources.md`](resources.md). The chapters cite Chip Huyen's *Designing Machine Learning Systems*, Sculley et al.'s *Hidden Technical Debt in Machine Learning Systems*, Google's *Rules of Machine Learning* and *MLOps* guide, Kreuzberger et al.'s *MLOps* survey, and the MLflow / Feast / Vertex / SageMaker docs for the concrete platform shapes.

## How this module hands off

- The RFC skeleton from chapter 06 is the deliverable format for the paired project `project-301-ml-system-design-portfolio`.
- The parity checklist and evaluation-gate hooks from chapter 04 hand off to the slice-based evaluation discipline in mod-305.
- The retraining-cadence and rollout patterns from chapter 05 hand off to the SLO / error-budget / on-call design in mod-307.
- The feature-store and model-registry consumer patterns from chapter 03 assume the paved-road platform contracts in mod-308 (peer platform collaboration).
