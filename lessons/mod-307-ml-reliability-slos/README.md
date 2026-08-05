# mod-307-ml-reliability-slos: ML Reliability — SLOs, Cost Budgets, and Incident Response

**Estimated effort:** 12 hours

At L20, "the model is up" is a green Grafana panel. At L30, "the model is up" is a follow-up question — *up for whom, over what window, on which SLI, and how much error budget do we have left this quarter?* Every mature ML org eventually converges on the same discipline: the classical SRE framework of SLIs, SLOs, error budgets, and incident response is the *floor*, and the ML-specific extensions — prediction freshness, quality drift, calibration, retraining SLA, cost budgets that engineering enforces, ML-specific runbooks and postmortems, retraining as a first-class deploy path — are what a senior ML engineer designs into the system.

This module installs that discipline. It is the reliability-facing sibling of [mod-305](../mod-305-advanced-evaluation/) (which established the offline / online promotion path), [mod-306](../mod-306-experimentation-at-scale/) (whose guardrails borrow their SLI vocabulary from this module), [mod-304](../mod-304-production-llm-integration/) chapter 04–05 (which established the per-feature cost / latency envelope this module aggregates), and [mod-302](../mod-302-ml-systems-architecture/) chapter 05 (which established the retraining cadence this module turns into a rollback-able deploy). It is the module the on-call reads at 3 a.m.

## Learning objectives

- Define ML-specific SLIs and SLOs — prediction freshness, quality drift, retraining SLA — alongside classical uptime SLOs.
- Set cost budgets and quotas that engineering can *enforce*, not just report.
- Run incident response for ML failures — bad predictions, silent drift, feature-pipeline breakage — and write the postmortem.
- Treat retraining as a first-class deploy with rollback and canary paths.

## Chapters

1. [`01-sre-fundamentals-for-ml.md`](01-sre-fundamentals-for-ml.md) — the classical SRE vocabulary (SLI, SLO, SLA, error budget, burn rate, multi-window alerting) at senior fluency, the four SRE assumptions that break for ML, and the classical SLI menu (availability, latency, throughput, feature-fetch, dependency SLIs) that is the *floor* every ML service sits on.
2. [`02-ml-specific-slis-and-slos.md`](02-ml-specific-slis-and-slos.md) — the ML-specific SLI menu that sits on top of the classical floor: prediction freshness (feature-age vs. target), quality drift (delta against a locked baseline, adapted to the label-landing regime), calibration and distribution drift (ECE / PSI / K-S), and retraining SLA (time-since-last-promoted-retrain against a cadence target).
3. [`03-cost-budgets-and-quotas.md`](03-cost-budgets-and-quotas.md) — the difference between *reporting* cost and *enforcing* cost; the budget hierarchy (feature → team → org → emergency); the four enforcement primitives (rate limits and quotas, circuit breakers, priority queues, kill switches); the composed cost-budget section of the SLO document.
4. [`04-ml-incident-response-and-postmortems.md`](04-ml-incident-response-and-postmortems.md) — the ML incident taxonomy (serving degradation, feature-pipeline breakage, model quality regression, silent drift, cost runaway, adversarial abuse); ML-specific runbooks as decision trees; the four rollback surfaces (model, feature-view, prompt, schema) and their one-action property; gamedays and drills; the ML-adapted blameless postmortem template with its prevent / detect / mitigate / practise action-item categories.
5. [`05-retraining-as-a-first-class-deploy.md`](05-retraining-as-a-first-class-deploy.md) — the model-registry state machine (Training → Evaluated → Staging → Canary → Production → Rolled-back → Archived); the retraining pipeline as its own reliability service with its own SLIs; the retraining-specific canary shape (shorter than a novel-feature canary because the baseline is a known-good champion); rollback across the four surfaces; the retention policy that keeps rollback targets available; the retraining-program-level metrics (rollback rate, offline / online agreement, evaluation rejection rate).

## Exercises

- [`exercises/exercise-01-ml-slis-and-slos.md`](exercises/exercise-01-ml-slis-and-slos.md) — author a full SLO document for a chosen ML feature: classical SLIs, ML-specific SLIs (freshness, quality drift, calibration, distribution drift, retraining SLA), error budgets, burn-rate alerting, and the error-budget policy staircase.
- [`exercises/exercise-02-cost-budgets-and-quotas.md`](exercises/exercise-02-cost-budgets-and-quotas.md) — build the *enforced* cost budget for the same feature: budget hierarchy, per-feature and per-tenant quotas, circuit-breaker configuration, kill-switch mechanism, degradation path, and runaway-detection thresholds.
- [`exercises/exercise-03-ml-incident-response-drill.md`](exercises/exercise-03-ml-incident-response-drill.md) — run a scripted incident drill against a chosen scenario (feature-pipeline lateness, silent calibration drift, cost runaway, or bad-promotion rollback): execute the runbook, hit the rollback, write the postmortem in the template.
- [`exercises/exercise-04-retraining-as-a-deploy.md`](exercises/exercise-04-retraining-as-a-deploy.md) — spec the retraining-as-deploy plan for a chosen feature: registry state machine, per-transition gates, retraining canary ladder, rollback plan across the four surfaces, retention policy, and program-level metrics.

## Labs & quizzes

- `labs/` — reserved for a longer-form hands-on lab (end-to-end SLO + budget + incident drill + retraining deploy against a running toy system) in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — the Google SRE canon, the ML monitoring literature (drift detection, ML Test Score, calibration), the incident-response canon (blameless postmortems, PagerDuty, chaos engineering), the cost-management canon (FinOps Framework, cloud-provider cost tools), and the peer-track pointers — are catalogued in [`resources.md`](resources.md).

## How this module hands off

- The SLI vocabulary (chapters 01–02) is what [mod-306 chapter 01](../mod-306-experimentation-at-scale/01-experiment-design-power-mde-and-ramp.md)'s guardrail set names its latency, cost, and slice thresholds against.
- The canary and rollback discipline (chapters 04–05) is the reliability layer under [mod-305 chapter 03](../mod-305-advanced-evaluation/03-shadow-and-canary-online-promotion.md)'s promotion path; the auto-rollback controller referenced there is authored here.
- The cost-budget framework (chapter 03) inherits its per-feature envelope from [mod-304 chapter 04](../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) and adds the aggregate enforcement — kill switches, circuit breakers, priority queues — the LLM chapter names as its degradation path.
- The retraining state machine and its reliability contract (chapter 05) is what makes [mod-302 chapter 05](../mod-302-ml-systems-architecture/05-retraining-and-rollout.md)'s four retraining triggers deliver against an SLO instead of an aspiration.
- The incident-response taxonomy (chapter 04) hands adversarial and abuse-shaped incidents to [mod-309](../mod-309-responsible-ai-governance/) for policy-level review, and hands the multi-team escalation contract to [mod-308](../mod-308-platform-collaboration/).
- The SLI, SLO, and cost-budget documents this module authors are what a senior ML engineer defends in the same forums the release memo (mod-305 chapter 04's ML Test Score review packet) is signed in.
