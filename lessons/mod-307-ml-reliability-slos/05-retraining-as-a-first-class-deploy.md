# Retraining as a first-class deploy: rollback, canary, and the registry state machine

## Motivation

Ask an L20 ML engineer "how do you promote a new model?" and you get a Jupyter notebook, an `mlflow.register_model` call, and a hope. Ask an L30 the same question and you get a *state machine* — a registered artefact, a signed-off evaluation, a shadow window, a canary, a rehearsed rollback path, and a documented promotion decision.

The claim of this chapter: **a retraining run that produces a promoted model is a deploy**. It has all the reliability characteristics of a deploy — it changes production behaviour, it can regress, it can cascade a failure across dependent systems, and it must be rollback-able within seconds. Treating it as anything less is why "the retraining pipeline ran and the model got worse" is one of the most common ML incident shapes (Category C in chapter 04 §1).

Two related chapters do parts of this job:

- **Mod-302 chapter 05** authored the retraining *cadence* (the four triggers) and the *rollout tools* (shadow, canary) at the architecture level. It is the sibling architectural chapter to this one.
- **Mod-305 chapter 03** authored the *online promotion path* (shadow → canary → A/B → full rollout) for any candidate. That chapter is about the *decision path*.

This chapter is about the *retraining pipeline* specifically — the pipeline that produces the candidate, the machinery that promotes it, the state the registry has to hold, the rollback the on-call has to reach for, and the SLO discipline (chapter 02 §4's retraining SLA SLI) that gates whether the pipeline is doing its job. If mod-302 chapter 05 is "when to retrain and roughly how to roll out," and mod-305 chapter 03 is "how any candidate walks the online promotion ladder," this chapter is "how the retraining pipeline as a system meets the reliability contract."

## §1 — The registry state machine

Every model version has a *state* in the model registry. The state machine is short and load-bearing:

```
Training  ──►  Evaluated  ──►  Staging  ──►  Canary  ──►  Production  ──►  Archived
                  │                │           │              │
                  ▼                ▼           ▼              ▼
               Rejected         Rejected    Rolled back   Rolled back
                                                              │
                                                              ▼
                                                          Retention
                                                          (rollback target)
```

Every transition is gated. Every transition is auditable. Every transition has an owner. The MLflow, Vertex AI Model Registry, Kubeflow / Kubeflow Pipelines, Weights & Biases Model Registry, Sagemaker Model Registry, and BentoML documentation each expose approximately this state machine; the [MLflow Model Registry documentation](https://mlflow.org/docs/latest/model-registry.html) is a good concrete reference.

- **Training.** The pipeline is running. No artefact yet, or an artefact exists but has not been evaluated. Should never be promoted.
- **Evaluated.** The offline harness (mod-305 chapter 01) has run. `HarnessDecision.passes_release_gate` is `True` (or an override is signed). This is the *precondition* for promotion, not the promotion itself.
- **Rejected.** The offline harness failed and the version is out. Kept for the postmortem; not deleted.
- **Staging.** The evaluated candidate is loaded into the serving stack but *not* in the request path (shadow mode).
- **Canary.** A fraction of live traffic is being served by this candidate (mod-305 chapter 03 §2).
- **Production.** The current champion.
- **Rolled back.** The previous champion has been re-pinned; this version is no longer serving. Kept in the registry as an audit artefact.
- **Archived.** Retained per the retention policy (§8) so it can serve as a rollback target if the next promotion needs to revert past the previous one.

The important structural properties:

- **Every state transition is a distinct action with a distinct approver.** The pipeline promotes to Evaluated. The release meeting (or an automated gate) promotes to Staging. The canary controller promotes to Canary. The final promotion to Production is either the auto-rollback controller's non-firing over a window or an explicit approver.
- **The Production version is a single flag.** The serving path resolves "which model is champion" by reading a config flag, not by walking the registry. Chapter 04 §4's one-action rollback property depends on this: a rollback is a flag flip, and the flag is a live, atomic pointer.
- **The Rolled-back state exists.** A version that used to be Production and got reverted goes to Rolled-back, not to Archived, not to Rejected, not to Production. That distinction is important for the postmortem: "how many promotions rolled back this quarter" is a *reliability metric on the retraining program itself* (§7 below).

## §2 — What each state actually requires

Each state has a *gate* — the specific checks that let a version enter the state. Rougher-boundary programs let these implicit; senior programs make them explicit.

### Gate: Training → Evaluated

- **Deterministic input contract.** The pipeline received a pinned data version (Delta / Iceberg version, warehouse snapshot ID, feature-store snapshot) and a pinned code version.
- **Artefact registered.** The model artefact was uploaded to the registry with metadata: training-data snapshot ID, code commit, feature-view versions consumed, hyperparameters, training metrics.
- **Offline evaluation ran.** The harness (mod-305 chapter 01) evaluated against the frozen eval set with the same slice matrix as prior runs.
- **Reproducibility check.** The training run has enough recorded state that a re-run against the same inputs would produce the same artefact (or, when non-determinism is fundamental — most deep-learning training is — an equivalent artefact within a tolerance).
- **Test suite passed.** Chapter 04 of mod-305 (ML Test Score) is the checklist: are the invariance tests, directional-expectation tests, minimum-functionality tests all green?

If any of these fails, the transition is to Rejected, not Evaluated.

### Gate: Evaluated → Staging (shadow)

- **`HarnessDecision.passes_release_gate = True`**, without override, or with a *signed* override from an approver.
- **Shadow serving capacity available.** Shadow runs the candidate on 100 % of production requests in parallel; the serving stack has to have the headroom, and the cost budget (chapter 03) has to have the room.
- **Comparison plumbing wired.** The shadow's predictions are being logged against the champion's predictions and against ground-truth-when-available.

### Gate: Staging → Canary

Mod-305 chapter 03 §1 authored the shadow exit criteria; they are the entry to canary. Restated:

- **Latency:** candidate p50/p95/p99 within the SLO (chapter 01 §5).
- **Error rate:** candidate error rate ≤ baseline within CI on shadow traffic.
- **Cost:** candidate per-request cost within envelope (chapter 03 §4 and mod-304 chapter 04).
- **Serving-path correctness:** no schema failures, no downstream-consumer errors.
- **Where ground truth lands:** online primary metric lift within offline CI or within agreed tolerance.

### Gate: Canary → Production

Mod-305 chapter 03 §2 authored the canary exit criteria. Restated at compact form:

- **The canary ramp completed without rollback.** No SLI breach at any ramp step; no auto-rollback controller trip; no manual rollback for any reason.
- **All slice-level guardrails green.** The offline slice matrix (mod-305 chapter 02) enforced on canary traffic.
- **A signed decision.** The release meeting has produced a signed record — the decision maker, the decision, the observed metrics, the guardrail status. Not "we forgot to hit stop and it drifted to full traffic."

### Gate: Production → Rolled-back

The rollback is a *runtime* event, not a decision-meeting event. Triggers:

- **SLI breach that meets the auto-rollback condition** (chapter 02 §5).
- **Manual on-call decision** during an incident (chapter 04 §4).
- **A parent-level policy** — a cost-budget exhaustion (chapter 03) or an org-level freeze (chapter 01 §2).

Every rollback triggers a postmortem (chapter 04 §6) *and* an update to the retraining-program-level metrics (§7).

## §3 — The retraining pipeline as reliability infrastructure

The retraining pipeline is not "the notebook we run every Monday." It is a *service*, with its own SLIs, its own incident response, and its own reliability contract.

### The pipeline's own SLIs

- **Pipeline success rate.** `successful_runs / attempted_runs` over a rolling 90-day window. SLO: 90 %+ for cadence-triggered pipelines. Any lower and the retraining SLA SLI (chapter 02 §4) becomes hard to meet.
- **End-to-end pipeline latency.** From trigger fired to `Evaluated` state, or to `Rejected` if the offline gate fails. SLO: some fraction of a day for a small model, some fraction of a week for a large one — depends on model scale.
- **Data-freshness at run-start.** The training-data snapshot used has to be within a target lag of the actual current world. If the pipeline is running against data three weeks stale because a warehouse snapshot got frozen, the resulting model will be stale-by-construction.
- **Evaluation-gate pass rate.** `runs_that_pass_offline_gate / runs_that_reach_evaluation` over a 90-day window. A low rate is a signal that either the model is regressing or the eval gate is too strict; either way, it is a reliability signal on the retraining program.
- **Promotion pass rate.** `runs_that_reach_Production / runs_that_pass_offline_gate` over a 90-day window. A low rate is a signal that shadow or canary consistently blocks candidates; the promotion path or the offline evaluation is not aligned.

The first three SLIs are what tell you the pipeline's *plumbing* is healthy. The last two are what tell you the pipeline's *product* — the candidates it produces — is worth promoting.

### The pipeline's incident-response posture

The retraining pipeline has its own runbook (chapter 04 §3's `runbooks/retraining-pipeline-failure.md`). The runbook covers:

- **The pipeline is down.** Scheduled trigger fired, pipeline did not run. Classical DAG orchestration incident (Airflow / Argo / Vertex Pipelines).
- **The pipeline failed mid-run.** A step errored. Depending on which step, the rollback machinery differs — a training step failure lets the previous champion keep serving; an evaluation step failure means an artefact exists but is untrusted.
- **The pipeline produced a candidate that failed evaluation.** The Evaluated → Rejected transition. On its own not an incident, but a *pattern* of consecutive rejections is a Category C-shaped incident on the retraining program itself.
- **The candidate got past evaluation but was rejected at shadow / canary.** Points to a training-serving skew or a distribution shift the offline harness did not catch. Postmortem action item: strengthen the harness.

The pipeline runbook is *practised* the same way other runbooks are — a gameday where a specific pipeline failure is injected and the on-call walks the response.

## §4 — Retraining canary — the ML-specific rollout shape

Mod-305 chapter 03 §2 authored the general canary shape. This section names the ML-specific extensions the retraining path needs.

### The dependency-canary problem

Classical service canaries route a fraction of traffic to the candidate. ML retraining candidates have a wrinkle: **the candidate model interacts with the *current* feature-view versions and the *current* downstream systems**. If the retraining pipeline pinned a specific feature-view version at training time and the feature-view has since drifted, the candidate is trained against one distribution and serving against another — an in-production training-serving skew.

The mitigation is that the retraining pipeline must record, in the model artefact:

- The feature-view versions the training data was drawn from.
- The training-time distribution snapshot for each critical feature.

At canary-entry the serving path checks that:

- The feature-view versions currently deployed match (or are compatible with) the versions the model was trained against.
- The live feature distributions are within a tolerance of the training-time distributions (this is exactly the distribution-drift SLI from chapter 02 §3, applied at deploy time).

Any mismatch is a signal to *stop the canary before it starts* — the pipeline has to re-run against the current feature-views, or the version compatibility has to be explicitly asserted by the model owner.

### The rollout-fraction rule

For a retraining canary specifically, the rollout ladder is *shorter* than for a novel-feature canary because the current champion is a known-good baseline. Common shape:

```
5%   → 60 min   SLI diagnostic pass (freshness / quality / calibration / cost)
25%  → 4 h     first per-slice quality look
50%  → 24 h    full weekly cycle span if the model has weekly seasonality
100% → deploy  after 50% is green for a full business day
```

For a novel-feature canary, the ladder was longer because the failure surface was broader (mod-305 chapter 03). For a retraining canary, the failure surface is narrower — it is another version of a model that already exists — so the ladder is proportionally shorter. Google's [*Canarying Releases*](https://sre.google/workbook/canarying-releases/) chapter discusses the ladder-length trade-off.

### Slice-level canary requirements

For a retraining canary, the slice-level check is *pre-run*, not just on the fly. The offline harness (mod-305 chapter 01) reports per-slice quality; the release meeting reviews it; the canary is expected to *reproduce* the per-slice offline signal within CI.

If the online slice-level signal *disagrees* with the offline per-slice numbers by more than a tolerance, the canary is rolled back and the harness is investigated for training-serving skew or slice-composition drift. This is what makes the harness *self-improving* — every mismatch between offline and online per-slice numbers is a postmortem action to grow the harness.

## §5 — Rollback for a retraining deploy

The rollback surface for a retraining deploy is larger than for a code deploy, because there are four things that could be causing the failure:

- **The model artefact.** Rollback: re-pin the previous champion in the registry.
- **The feature-view definitions.** If the retraining also promoted a new feature-view, rollback: re-pin the previous feature-view versions.
- **The prompt / prompt template** (for LLM-augmented features). Rollback: re-pin the previous prompt version.
- **The downstream schema.** If the new model changed its output schema (added a new field, changed a scale), rollback: revert the schema-consuming code.

Every rollback surface has its own flag; the on-call executes the specific one that the postmortem-in-progress diagnoses. Chapter 04 §4's one-action property applies to each.

Two properties of the retraining-rollback discipline:

- **The previous champion is retained.** For at least the length of the promotion window plus a safety margin — often 90 days for a monthly cadence, or three cadence periods, whichever is longer. If the previous champion has been deleted from the registry to "save disk," the rollback is not available.
- **The previous feature-view versions are retained.** Same rule. Feature stores that support versioning ([Feast](https://docs.feast.dev/), [Tecton](https://docs.tecton.ai/), [Hopsworks](https://www.hopsworks.ai/), [Databricks Feature Store](https://docs.databricks.com/en/machine-learning/feature-store/index.html)) make this cheap; ad-hoc pipelines make it expensive.

### The rollback-and-stay-rolled-back failure mode

Chapter 02 §4 named this: a rollback happens, the team fixes the underlying issue, and *never re-promotes*. The retraining SLA SLI is red; the pipeline is technically fine; the process failed.

The discipline that prevents this:

- **Every rollback opens a ticket for re-promotion.** With an owner, a due date matched to the retraining SLA target.
- **The retraining SLA SLI is *watched* during the rolled-back state.** If the SLI enters the alert threshold before a re-promotion, that itself is an escalation trigger.
- **The postmortem for the rollback names the re-promotion date.** Not "we will re-promote when ready"; a specific date, or a specific condition, whose satisfaction unblocks re-promotion.

## §6 — The staleness question

For any model with a long retraining cadence (weekly, monthly), there is a class of failure that is *not* an incident but *is* a reliability concern: **the model has been Production long enough that the world has moved past it**. The retraining SLA SLI (chapter 02 §4) is one instrument for this; a complementary check is the *live drift diagnostic* against the training-time distribution.

The shape:

- At training time, snapshot the distribution of each critical feature and the distribution of each critical target (label).
- At runtime, monitor the same distributions on live inputs and (when labels land) live outputs.
- The drift SLI (chapter 02 §3) reports the divergence; the *training-time distribution* is the reference.

A staleness incident is: the input distribution has drifted from training-time to today, the calibration SLI is trending toward its threshold, and the retraining SLA SLI is at 90 % of budget. None of the SLIs is red yet; the pattern is what tells you the *next* promotion is more urgent than the calendar says.

The senior discipline: **the reliability review reads not just the current SLI status but the trajectory across the retraining cadence.** A three-month trend of degrading calibration between successive promotions is a reliability signal even when every single SLI is nominally green.

## §7 — Retraining-program-level metrics

Above the pipeline-level SLIs (§3), the retraining program itself has a set of health metrics that live at the reliability review, not the on-call dashboard.

- **Rollback rate.** Fraction of promotions that rolled back within N days. A rate above ~5 % is a signal the promotion path is over-permissive; a rate of 0 % (never had to roll back) is often a signal the promotion path is *under-permissive* and the team is doing manual work upstream to make sure a rollback never happens (defensive over-training, over-tight offline gates).
- **Time from trigger to Production.** Median and worst-case across a quarter. Feeds into the retraining SLA target (chapter 02 §4).
- **Evaluation-gate rejection rate.** Fraction of runs Rejected at evaluation. A rising rejection rate is a signal that either the model is regressing or the eval gate is too tight; either way, action.
- **Offline / online agreement.** For each promoted model, the difference between the offline-predicted primary-metric lift and the online-observed one. This is the metric that says "our harness is calibrated." A worsening agreement over time is a signal that the offline eval is drifting from production reality; the harness needs slices or adversarial cases added (mod-305 chapter 01 §7).
- **Number of promotions per quarter.** A low number is *not* automatically bad — some models don't need frequent promotion — but the trend matters against the retraining-cadence target.

These metrics are reported quarterly. The reliability review reads them alongside the SLI status; the two together shape the reliability roadmap.

## §8 — Retention policy

Every promoted-and-then-rolled-back version, every rejected version, and every canary-only version is an artefact with a retention question. The load-bearing pieces:

- **Every version that was ever in Production is retained for at least 12 months.** So that a delayed problem (an audit finding, a fairness complaint, an outcome-metric regression that only surfaces months later) can be diagnosed against the exact model that produced the observation.
- **Every version rejected at evaluation is retained for 3 months.** For postmortem work and for pattern analysis on why models keep failing this gate.
- **Every version that reached canary but was rolled back is retained for 12 months.** Same rationale as retention of ever-Production versions.
- **The training-data snapshot for each retained version is retained too.** Otherwise reproducing the version is impossible. Delta / Iceberg time-travel and warehouse snapshots make this cheap; the discipline is *knowing to invoke it* and pinning the version at training time.

The retention policy is documented in the same repo file as the model registry configuration. Deleting an artefact requires a signed-off "retention expiry" event that logs *what* was deleted and *why*.

## §9 — Where this chapter hands off

- **Chapter 04 (Incident response)** owns the runbook the on-call reads when a canary rolls back or the pipeline breaks.
- **Chapter 02 §4 (Retraining SLA SLI)** is the SLO that gates whether the retraining program is meeting its reliability target.
- **Chapter 03 (Cost budgets)** is where the retraining pipeline's cost line lives, and where a runaway training run trips a circuit breaker.
- **Mod-305 chapter 01 (Offline eval harness)** is the gate the retraining pipeline runs a candidate through; mod-305 chapter 03 is the online promotion path this chapter's state machine follows.
- **Mod-302 chapter 05 (Retraining cadence, data flywheel, rollout)** is the sibling architectural chapter — the *when* and *rough how* of retraining, that this chapter turns into a reliability-gated deploy path.
- **Mod-306 chapter 01 (Experiment design)** is where the A/B stage in the promotion path is authored; this chapter's canary rollout precedes the A/B stage.
- **Mod-308 (Platform collaboration)** is where the retraining pipeline as a shared service with the ML platform team is authored — who owns the pipeline, who owns the registry, and what the delegation contract looks like.

## Summary

A retraining run that produces a promoted model is a *deploy*, and it earns all the reliability discipline of a deploy: a state machine (Training → Evaluated → Staging → Canary → Production → Archived) with explicit gates at every transition; a set of SLIs on the pipeline itself (success rate, latency, freshness, evaluation-gate pass rate, promotion pass rate); a retraining-specific canary shape shorter than a novel-feature canary because the baseline is a known-good champion; a rollback discipline covering four surfaces (model artefact, feature-view versions, prompt version, downstream schema), each with its own one-action flag; a retention policy that keeps rollback targets available for the length of the audit window; and a set of program-level metrics (rollback rate, offline/online agreement, evaluation rejection rate) that live at the quarterly reliability review. This chapter's discipline is what makes chapter 02 §4's retraining SLA SLI meetable, chapter 04's runbooks executable, and mod-305 chapter 03's promotion path real for the pipeline that *keeps* producing the candidates the promotion path evaluates. The retraining pipeline is not "a thing the ML team runs on Mondays"; it is the reliability-critical deploy path of an ML system, and it deserves the same discipline as any other deploy path.
