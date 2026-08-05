# mod-308-platform-collaboration: Cross-Team Collaboration with Platform, MLOps, and Training Tracks

**Estimated effort:** 10 hours

At L20 the platform is a set of tools you had to install. At L30 the platform is a *contract you are a party to* — with SLOs, escalation paths, deprecation policies, and a feedback loop that turns your team's pain into the platform's roadmap. This module is the senior discipline of *consuming the paved road well* and *changing it correctly when it doesn't fit*.

The 26 in-window Senior Machine Learning Engineer postings the [job-requirements catalogue](../../JOB_REQUIREMENTS.md) sampled put cross-team collaboration at 0.77 frequency — the second-most common seniority-differentiating theme, behind role-scope and ahead of architecture. The verbs that surface — *own*, *partner*, *influence without authority*, *set direction* — are the ones this module operationalises.

This module is the *cross-team* sibling of [mod-307](../mod-307-ml-reliability-slos/) (whose SLI vocabulary you point at the platform's dependency SLOs), [mod-305](../mod-305-advanced-evaluation/) (whose eval harness is one of the paved-road primitives you consume), [mod-302](../mod-302-ml-systems-architecture/) (whose architectural decisions live on top of the platform's primitives), and [mod-310](../mod-310-technical-leadership/) (whose leadership discipline this module's RFC and hand-off contracts are a specific application of).

## Learning objectives

- Consume paved-road ML platforms (feature stores, model registries, training clusters) idiomatically and give the platform team feedback that improves the road.
- Make build-vs-adopt calls at team-scope: when to bend to the platform, when to escalate a gap, when to build in-house, when to contribute back.
- Write a contribute-back RFC that a peer platform team can review and act on.
- Design the hand-off contracts to `training-pipeline-engineer` and `model-evaluation-engineer` so specialist depth arrives without churn.

## Chapters

1. [`01-paved-road-consumption.md`](01-paved-road-consumption.md) — what a paved road is; why platform teams build it; the four disciplines of idiomatic consumption (declared API, versioning, standard telemetry, land-on-top-not-underneath); the consumer's contract with SLOs / escalation / deprecation; the three forms of feedback (bug report → quarterly consumer report → RFC); the paved-road inventory as the artefact this module reuses.
2. [`02-build-vs-adopt-decisions.md`](02-build-vs-adopt-decisions.md) — the four paths (bend, escalate, build-in-house-with-migration, contribute back); the five factors (technical fit, reversibility, blast radius, opportunity cost, three-year TCO); the decision matrix; the tradeoff doc; the migration plan that keeps "build in-house" reversible; the "escalate correctly" and "contribute back trap" disciplines.
3. [`03-contribute-back-rfcs.md`](03-contribute-back-rfcs.md) — what an RFC is (and is not); the sponsor conversation as precondition; the RFC structure; writing to the platform team's altitude, not your team's; alternatives-considered as the load-bearing section; migration and deprecation as first-class subsections; reviewers, approvers, and driving the review cycle; rejection as a valid outcome.
4. [`04-handoff-contracts-to-specialists.md`](04-handoff-contracts-to-specialists.md) — the shape of a hand-off contract (six sections); the training-pipeline-engineer hand-off (parity-preserving speed-ups, on-call burn-in, regression clause); the model-evaluation-engineer hand-off (judge-human agreement as acceptance criterion, cost budgets, promotion-gate integration); cross-team vocabulary translation; when *not* to hand off; the program-level portfolio view.

## Exercises

- [`exercises/exercise-01-paved-road-consumption-audit.md`](exercises/exercise-01-paved-road-consumption-audit.md) — author the paved-road inventory for a specific team, self-audit idiomatic consumption for each primitive, and produce the quarterly consumer report that packages the resulting pain points as feedback the platform team can act on.
- [`exercises/exercise-02-build-vs-adopt-tradeoff-doc.md`](exercises/exercise-02-build-vs-adopt-tradeoff-doc.md) — pick a specific gap between your team's requirement and the paved road; walk all four paths (bend, escalate, build-in-house-with-migration, contribute back) against the five-factor rubric; author the tradeoff doc with recommendation, defence, and exit plan.
- [`exercises/exercise-03-contribute-back-rfc-to-platform.md`](exercises/exercise-03-contribute-back-rfc-to-platform.md) — draft a full contribute-back RFC (title through decision log) that a peer platform team could actually review, with alternatives-considered, migration, deprecation, and named sponsor / reviewers / approver.

## Labs & quizzes

- `labs/` — reserved for a longer-form hands-on lab (a multi-round exercise where a peer plays the platform team, reviews your RFC, and the group walks the decision cycle end-to-end) in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — the platform-engineering canon (Team Topologies, Netflix paved road, Camille Fournier), the RFC / design-doc canon (Rust RFCs, Google-style design docs, Gergely Orosz), the deprecation / migration canon (Google's public deprecation policy), the specialist-track pointers (training-pipeline-engineer and model-evaluation-engineer curriculum tracks) — are catalogued in [`resources.md`](resources.md).

## How this module hands off

- The paved-road inventory (chapter 01 §7) is what [mod-307 chapter 01 §6](../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md)'s SLO document points at when it names dependency-SLI composition. Your feature's SLO cannot be tighter than the platform primitives it consumes.
- The build-vs-adopt tradeoff doc (chapter 02 §4) is the artefact [mod-302's](../mod-302-ml-systems-architecture/) architectural decisions cite when the "why we built X ourselves" question shows up in an architecture review.
- The contribute-back RFC (chapter 03) is a specific instance of the RFC discipline [mod-310](../mod-310-technical-leadership/) generalises to project RFCs, technical-strategy docs, and roadmap RFCs — the shape transfers.
- The hand-off contracts (chapter 04) are what let mod-303 (advanced modelling) hand off to `training-pipeline-engineer` for distributed-training depth and what let mod-305 (advanced evaluation) hand off to `model-evaluation-engineer` for LLM-judge and eval-harness depth. Both are peer-track delegations named in [`JOB_REQUIREMENTS.md`](../../JOB_REQUIREMENTS.md).
- The escalation path from chapter 04 hands the multi-team, cross-org escalation to [mod-309](../mod-309-responsible-ai-governance/) for governance-visible incidents and to [mod-310](../mod-310-technical-leadership/) for the technical-leadership altitude of the same relationship.
