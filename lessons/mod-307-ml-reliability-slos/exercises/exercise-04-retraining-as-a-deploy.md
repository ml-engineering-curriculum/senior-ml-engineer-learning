# exercise-04: Retraining as a deploy — state machine, canary, rollback

**Estimated effort:** 3 hours

## Objective

Author the **retraining-as-deploy** plan for a specific ML feature — the model-registry state machine, per-transition gates, the retraining-specific canary ladder, the rollback plan across the four surfaces (model, feature-view, prompt, schema), the retention policy, and the retraining-program-level metrics. The output is a Markdown design document that a peer L30 could review and an SRE on-call could execute.

This is the L30 tell that separates "we retrain the model on a schedule" from "we operate the retraining pipeline as a reliability-critical deploy path with the discipline of any other deploy." An L20 promotes a model when the notebook shows a lift; an L30 walks the candidate through a state machine with explicit gates and rehearsed rollback, and reports on program-level metrics quarterly.

## Prerequisites

- Read chapter 05 (`05-retraining-as-a-first-class-deploy.md`) — the state machine, per-transition gates, retraining canary shape, rollback across the four surfaces, retention policy, program-level metrics.
- Read chapter 04 §4 (`04-ml-incident-response-and-postmortems.md`) — the rollback machinery's one-action property.
- Skim chapter 02 §4 — the retraining SLA SLI this plan has to deliver against.
- Skim [mod-302 chapter 05](../../mod-302-ml-systems-architecture/05-retraining-and-rollout.md) — the four retraining triggers and the rollout tools at architectural altitude.
- Skim [mod-305 chapter 03](../../mod-305-advanced-evaluation/03-shadow-and-canary-online-promotion.md) — the general promotion path this plan turns into a state machine for retrains specifically.
- Bring the feature context from exercise-01. This exercise reuses the same feature; the retraining plan slots into the same reliability posture.

## Steps

### 1. State machine for your feature (≈ 30 min)

Chapter 05 §1 authored the state machine. For your feature:

- **Draw or describe the state machine** — Training, Evaluated, Rejected, Staging, Canary, Production, Rolled-back, Archived. Adapt names if your existing tooling uses different terms; keep the *semantics* intact.
- **For each transition, name the gate** (chapter 05 §2):
  - Training → Evaluated: deterministic input contract, artefact registered, offline evaluation ran, reproducibility check, test suite passed. State each as it applies to *your* pipeline.
  - Evaluated → Staging: harness pass (with signed override if any), shadow capacity available, comparison plumbing wired.
  - Staging → Canary: shadow exit criteria (latency, error rate, cost, serving-path correctness, ground truth).
  - Canary → Production: canary ramp completed without rollback, all slice guardrails green, signed decision.
  - Production → Rolled-back: SLI breach with auto-rollback condition met, manual on-call decision, or parent-level policy (cost, org freeze).
- **For each transition, name the approver** — a specific role (not "the team"). Explicit approvers for Staging → Canary and Canary → Production are the load-bearing ones.
- **For each transition, name the *action* the approver takes** — a PR merged, a flag flipped, a CLI command run. Concrete.

### 2. Retraining pipeline SLIs (≈ 20 min)

Chapter 05 §3 named five pipeline-level SLIs:

- Pipeline success rate.
- End-to-end pipeline latency.
- Data-freshness at run-start.
- Evaluation-gate pass rate.
- Promotion pass rate.

For each, author:

- The specific definition (query or computation).
- The SLO target and window (chapter 05 §3 gives defaults; adapt to your cadence).
- The alerting shape — most are slow-burn on a rolling window, not fast-burn.
- The runbook stub — where does the on-call go when this SLI fires?

### 3. Retraining canary ladder (≈ 30 min)

Chapter 05 §4 gave a default ladder — 5 % → 25 % → 50 % → 100 %. For your feature:

- **Author the specific ladder**, adjusting step-percentages and step-durations for your traffic scale and business-cycle needs. Justify the choices against the general ladder.
- **Every step names:**
  - The advance criterion — a specific SLI or scorecard reading, not "looks fine."
  - The rollback criterion — the SLI-breach threshold and hold-time that trips the controller.
  - The on-call action — the human involvement at that step.
- **Slice-level advance criteria.** The offline harness produced per-slice quality numbers. The canary is expected to *reproduce* the offline per-slice numbers within CI. Name the tolerance and the diagnostic if the numbers disagree.
- **The dependency-canary check.** Chapter 05 §4 named the ML-specific extension: at canary-entry, the model's training-time feature-view versions must match (or be compatible with) the currently-deployed versions, and the live feature distributions must be within tolerance of the training-time distributions. Author the check.

### 4. Rollback plan across the four surfaces (≈ 30 min)

Chapter 05 §5 named the four rollback surfaces. For your feature:

- **Model artefact.** The flag or config that points at the champion model version. Where it lives. Who can flip it. Time-to-propagate.
- **Feature-view versions.** The flag(s) or config for the feature-views this model depends on. If your feature has 20 feature-views, name the *critical* ones whose rollback is a first-line action.
- **Prompt / prompt template** (LLM-augmented features only). The prompt-version flag. Where prompt versions are registered. Rollback discipline.
- **Downstream schema.** If your model changed its output schema between versions, the schema-consuming code's rollback path. If the schema is stable across versions, say so explicitly.

For each surface:

- The **one-action property** — is the rollback truly one flag flip? If not (a YAML edit, a re-deploy), that is a gap and an action item.
- The **rehearsal** — when was this surface's rollback last exercised? Chapter 04 §4 established quarterly as the default.
- The **retention** — how long is the previous version kept so the rollback target is available?

### 5. Retention policy (≈ 15 min)

Chapter 05 §8 authored the retention rules:

- Ever-Production versions retained ≥ 12 months.
- Rejected versions retained ≥ 3 months.
- Canary-only-then-rolled-back retained ≥ 12 months.
- Training-data snapshots retained for at least as long as the model version.

For your feature, author the specific retention numbers and the storage plan — where the artefacts live, the estimated storage cost, the deletion policy (retention expiry requires a signed-off event; the log names *what* was deleted and *why*).

### 6. Retraining-program-level metrics (≈ 20 min)

Chapter 05 §7 named five metrics:

- Rollback rate.
- Time from trigger to Production (median + worst).
- Evaluation-gate rejection rate.
- Offline / online agreement.
- Number of promotions per quarter (trend).

For your feature (or the team-level aggregation across features), author:

- The definition of each metric.
- The target (or the "no target, tracked as trend" note).
- The forum the metric is reported in (quarterly reliability review, monthly ML review).
- The reliability-roadmap linkage — what does a red metric mean the roadmap should include?

### 7. The rollback-and-stay-rolled-back guard (≈ 10 min)

Chapter 05 §5 named the "rollback-and-stay-rolled-back" failure mode: a rollback happens, the underlying issue is fixed, and re-promotion never happens. The retraining SLA SLI (chapter 02 §4) is red; nobody notices.

Author the discipline that prevents this:

- Every rollback opens a re-promotion ticket with an owner and a due date matched to the retraining SLA.
- The retraining SLA SLI is watched during the rolled-back state; entering the alert threshold before re-promotion is an escalation trigger.
- The rollback postmortem names the re-promotion date, not "when ready."

## Deliverable

A single Markdown document, 4–6 pages, titled `retraining-plan-<feature>.md`, structured as:

- Header — feature name, retraining owner, ML platform team dependency, last reviewed, next review.
- State machine and per-transition gates (§1).
- Pipeline-level SLIs (§2).
- Canary ladder (§3), including slice-level advance criteria and dependency-canary check.
- Rollback across the four surfaces (§4).
- Retention policy (§5).
- Program-level metrics (§6).
- Rollback-and-stay-rolled-back guard (§7).

The document is written to be *executed* — a retraining engineer should be able to walk it end-to-end and produce a promoted model version; an SRE on-call should be able to invoke any of the rollback surfaces in one action.

## Acceptance criteria

- [ ] The state machine has at least the seven canonical states — Training, Evaluated, Rejected, Staging, Canary, Production, Rolled-back — plus Archived.
- [ ] Every transition names a gate, an approver (specific role), and an action (specific mechanism).
- [ ] At least five pipeline-level SLIs, each with a definition, SLO target, and alerting shape.
- [ ] The canary ladder has at least three steps; every step names advance and rollback criteria as *specific numbers on specific SLIs*, not "looks fine."
- [ ] Slice-level advance criteria are named — the canary reproduces offline per-slice signal within a stated tolerance.
- [ ] The dependency-canary check (training-time vs. serving-time feature-view versions and distributions) is authored.
- [ ] All four rollback surfaces are named — model, feature-view, prompt (or "N/A: not LLM-augmented"), schema (or "N/A: schema is stable across versions").
- [ ] Every rollback surface has the one-action property, or names the *specific gap* if it does not — a follow-up action item.
- [ ] Retention numbers are specific (months / days), not "for a while."
- [ ] Program-level metrics have targets or explicit "trend-tracked, no target" annotations.
- [ ] The rollback-and-stay-rolled-back guard is authored — specific ticket, specific owner, specific due-date policy.
- [ ] The retraining SLA SLI from exercise-01 §5 is cross-referenced as the outer contract this plan delivers against.

## Stretch goals

- **The Shadow → Canary automatic-promotion policy.** For low-risk retrains (a scheduled cadence retrain that shows *no* offline delta and no distribution shift), can the Shadow → Canary transition be automatic? Under what specific conditions? Author the policy. Google's [*Managing Incidents*](https://sre.google/sre-book/managing-incidents/) has adjacent discussion of automation-of-routine.
- **The all-champion-versions comparison plot.** For your last four champions, plot (or table) their offline and online primary metric, calibration ECE, per-slice quality, per-request cost, and canary-rollback events. What does the *trend* say about the retraining program's health? Feeds the program-level metrics section (§6).
- **The retraining pipeline's own runbook.** Author the runbook for chapter 05 §3's failure modes — pipeline down, mid-run failure, consecutive evaluation-gate rejections, canary rollback. Link it into the runbook library from exercise-03.
- **The offline / online agreement metric — instrumented.** Chapter 05 §7 named it as a program-level metric. Author *how* it is computed: paired offline and online primary-metric values per promoted version, over the same time window. What tolerance is "agreeing"? Feeds mod-305 chapter 01 §7's harness-growth pattern.
- **Cross-link with the incident-drill artefact.** If you ran exercise-03 with S4 (bad promotion), attach its postmortem's action items to this plan's roadmap. The postmortem's "prevent" and "detect" items should be visible in the retraining plan's state-machine gates.
- **Feed the paired project.** Attach this retraining plan as the reliability-of-retraining section of a paired project. The retraining plan should be *executed*, not archived.
- **Peer review.** Trade retraining plans with a peer. The reviewer walks: "if I am the on-call at 3 a.m. and the retraining SLA SLI is at alert threshold, what does this document tell me to do?" Gaps become follow-up work.
