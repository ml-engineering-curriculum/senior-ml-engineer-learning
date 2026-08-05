# mod-309-responsible-ai-governance: Responsible AI — Review Packets, Model Cards, and Data Lineage

**Estimated effort:** 10 hours

At L20, "responsible AI" is a slide in the launch review deck and a link to the company's AI-use policy. At L30, "responsible AI" is a **review packet** — the model card, the data statement, the lineage-and-PII audit, the fairness / robustness / safety review, and the misuse-and-adversarial threat model — that a governance analyst signs, that ships with the release, and that the retraining cadence renews. The senior discipline is producing the artefacts *in the shape a reviewer can act on*, closing the audit loop between the claims in the model card and the evidence in the code and the pipelines, and running the ongoing review program without stalling the roadmap.

The 26 in-window Senior Machine Learning Engineer postings the [job-requirements catalogue](../../JOB_REQUIREMENTS.md) sampled put responsible-AI / MRM partnership at 0.35 frequency — above the in-scope threshold, and concentrated in the regulated-industry and safety-critical postings (Flatiron, Capital One, GM Perception, Reddit Safety). The verbs those postings use — *partner with model risk management*, *authorise for production*, *responsible for the deployment's fairness posture*, *design the review* — are what this module operationalises for the ML-team-side of the relationship. Depth on the governance-analyst side belongs to [`ai-governance-analyst-learning`](../../JOB_REQUIREMENTS.md); depth on the security-specific adversarial-ML side belongs to [`ai-infra-security-learning`](../../JOB_REQUIREMENTS.md); this module trains the ML engineer to *author the packet*, *hand it off correctly*, and *run the review cadence*.

This module is the *responsibility-facing* sibling of [mod-305](../mod-305-advanced-evaluation/) (whose slice matrix the fairness / robustness / safety review extends), [mod-307](../mod-307-ml-reliability-slos/) (whose incident-response taxonomy the threat model composes with and whose SLIs the review program borrows), [mod-308](../mod-308-platform-collaboration/) (whose peer-track hand-off vocabulary and RFC discipline the governance-analyst partnership inherits), and [mod-310](../mod-310-technical-leadership/) (whose technical-leadership altitude these artefacts get defended at when the deployment goes cross-org).

## Learning objectives

- Author a model card and data statement that a governance analyst can act on.
- Trace data lineage and PII handling from ingestion to inference and flag the gaps.
- Run review-cadence fairness / robustness / safety slices without stalling the roadmap.
- Threat-model the deployed system for the misuse and adversarial patterns realistic for the domain.

## Chapters

1. [`01-model-cards-and-data-statements.md`](01-model-cards-and-data-statements.md) — the model card as a decision artefact (not marketing, not developer docs); the Mitchell nine sections; the Bender-Friedman data statement as the training-data layer; writing at the governance-analyst altitude; the template you ship; the five failure modes; the governance-analyst hand-off; the model-triggered / time-triggered / event-triggered cadence policy.
2. [`02-data-lineage-and-pii-handling.md`](02-data-lineage-and-pii-handling.md) — lineage as the evidence base under the model card; the five-node / two-edge graph; the canonical five-tier flow (ingest → normalisation + PII → feature → training + serving → sinks); the four-category PII taxonomy (direct, quasi, sensitive, derived); the five-column PII register; consent / purpose limitation / retention; the three lineage-collection patterns; the lineage audit that finds the gaps.
3. [`03-fairness-robustness-safety-review-cadence.md`](03-fairness-robustness-safety-review-cadence.md) — the three review dimensions (fairness, robustness, safety); fairness-metric choice as a value-laden decision (with the impossibility results named); the always-on layer vs. the periodic-review layer; cadence choice by retraining frequency, population shift, risk classification, incident history; scoping without stalling via rotation, capped always-on, day-not-quarter reviews, and the F0 / F1 / F2 / F3 findings ladder; the review packet.
4. [`04-misuse-and-adversarial-threat-modeling.md`](04-misuse-and-adversarial-threat-modeling.md) — the five-question ML threat-model frame; the four adversarial categories (evasion, poisoning, extraction / membership inference, abuse at scale); the misuse surface (off-label, automation without oversight, feedback-loop, aggregation, cross-model); domain-specific concreteness; the seven mitigation landing sites; anchoring every mitigation to a test; the threat-model artefact; five failure modes to catch in your own draft.

## Exercises

- [`exercises/exercise-01-model-card-authoring.md`](exercises/exercise-01-model-card-authoring.md) — author the full model card + data statement for a chosen deployment; walk the five failure modes on your own draft; run a mock governance-analyst review.
- [`exercises/exercise-02-data-lineage-and-pii-audit.md`](exercises/exercise-02-data-lineage-and-pii-audit.md) — draft the lineage graph for the deployment, author the PII register, run the seven-step lineage audit, and produce the audit findings list keyed to the model card's PII claims.
- [`exercises/exercise-03-fairness-and-robustness-slice-cadence.md`](exercises/exercise-03-fairness-and-robustness-slice-cadence.md) — pick the always-on slice set and the periodic-review sweep for the deployment; author the cadence-decision doc and the fairness-metric decision doc; produce a first-cycle review packet.
- [`exercises/exercise-04-misuse-and-adversarial-threat-model.md`](exercises/exercise-04-misuse-and-adversarial-threat-model.md) — enumerate the two-to-five deployment-realistic threats and misuse patterns; produce the mitigation matrix; anchor every mitigation to a test; run a peer review with an ML/AI-security stand-in.

## Labs & quizzes

- `labs/` — reserved for a longer-form end-to-end lab (author the full review packet for a running toy deployment, run the mock governance review, hold up the release for one F0 finding, and iterate to signoff) in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — the model-card and data-statement canon (Mitchell et al., Bender & Friedman, Gebru et al., Google Model Card Toolkit), the fairness canon (Barocas-Hardt-Narayanan, Buolamwini-Gebru, Chouldechova, Hardt et al., Fairlearn / AIF360), the lineage and privacy canon (GDPR, NIST Privacy Framework, OpenLineage / DataHub, NIST SP 800-188), the threat-modelling canon (MITRE ATLAS, OWASP LLM Top 10, NIST AI 100-2e2023, Shostack), the regulatory canon (NIST AI RMF, EU AI Act, SR 11-7), and the peer-track pointers (`ai-governance-analyst-learning`, `ai-infra-security-learning`) — are catalogued in [`resources.md`](resources.md).

## How this module hands off

- The **review packet** this module authors (model card + data statement + lineage audit + fairness / robustness / safety review + threat model) is the artefact the governance analyst signs and the release memo cites. It composes with the [mod-305 chapter 04 ML Test Score review packet](../mod-305-advanced-evaluation/04-ml-test-score-production-readiness.md) and the [mod-307 chapter 01 §6 SLO document](../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md); together they form the deployment's promotion contract.
- The **lineage audit** (chapter 02) is the evidence base under the PII claims the model card makes. Its gaps feed the [mod-307 chapter 04 incident-response taxonomy](../mod-307-ml-reliability-slos/04-ml-incident-response-and-postmortems.md) as *governance-visible incidents* — an unauthorised prediction-log retention gap is an incident even without a downstream compromise.
- The **review cadence** (chapter 03) extends the [mod-305 chapter 02 slice-and-guardrail](../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) discipline into a program with a schedule. The always-on layer *is* mod-305 chapter 02's harness; the periodic layer is the review-program addition. Findings feed the model card's next revision.
- The **threat model** (chapter 04) hands adversarial-ML depth to [`ai-infra-security-learning`](../mod-308-platform-collaboration/resources.md) via the [mod-308 chapter 04 hand-off contract](../mod-308-platform-collaboration/04-handoff-contracts-to-specialists.md) discipline, and hands governance-vocabulary depth to [`ai-governance-analyst-learning`](../mod-308-platform-collaboration/resources.md). The senior ML engineer authors and drives the review; the peer-track experts sign and challenge.
- The **governance-analyst partnership** operates in the same forum where the [mod-310](../mod-310-technical-leadership/) technical-leadership discipline lives — the cross-org promotion review is where the ML engineer's cross-team altitude (mod-308) and technical-leadership altitude (mod-310) both show up.
