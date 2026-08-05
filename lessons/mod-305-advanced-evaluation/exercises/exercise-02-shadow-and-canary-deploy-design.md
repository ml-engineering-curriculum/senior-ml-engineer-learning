# exercise-02: Shadow And Canary Deploy Design

**Estimated effort:** 3 hours

## Objective

Author the **online-promotion plan** for a candidate model that has already passed an offline harness (exercise-01). Cover the four stages of chapter 03: shadow, canary, A/B (or interleaving, where applicable), and full rollout — with predefined entry / exit criteria for each stage, an auto-rollback controller for every stage that changes what a user sees, and a rehearsed kill-switch path.

The deliverable is a one-to-two-page release-plan memo that a peer L30 could sign, an on-call could execute, and a reviewer six months later could reconstruct the decision from. The point is not to over-engineer a rollout — it is to make sure that **no stage of the promotion happens on unstated criteria** and that every stage has a rollback trigger before it starts.

This is the L30 tell that separates "we shipped the offline win to 100 %" from "we shipped defensibly." Chapter 03 is what says an offline win *has to earn* its online promotion; this exercise is where you put that principle in the release record.

## Prerequisites

- Read chapter 03 (`03-shadow-and-canary-online-promotion.md`) — the promotion ladder, exit criteria, and auto-rollback vocabulary are load-bearing.
- Skim chapter 01 (`01-offline-eval-harness-shape.md`) — the `HarnessDecision` object is the precondition of shadow.
- Skim chapter 04 (`04-ml-test-score-production-readiness.md`) — Infra Test 6 (canary) and Infra Test 7 (rollback) are what your plan is evidence for.
- Skim [mod-306](../../mod-306-experimentation-at-scale/) for the depth of the A/B stage; skim [mod-307](../../mod-307-ml-reliability-slos/) for the SLI vocabulary the rollback controller names.

## Pick your candidate

Pick **one** — ideally the same feature you used in exercise-01, so the offline harness and the online promotion plan snap together:

- **C1 — New ranker candidate.** Two-tower retriever + gradient-boosted reranker replacing lexical BM25 + hand-tuned rules on a shopping-site search. Offline NDCG@10 lift 0.03 with 95 % CI clear of zero; all business-critical slice gates green.
- **C2 — LLM-augmented summariser candidate.** New prompt bundle + base-model version bump for a nightly review summariser (batch enrichment). LLM-as-judge score up, human calibration set agreement stable, no adversarial regression.
- **C3 — Fraud classifier candidate.** Deep model replacing a gradient-boosted baseline. Offline AUC lift 0.008, worst-group AUC not regressed, calibration ECE unchanged.
- **C4 — Support-ticket triage candidate.** Fine-tuned smaller LLM replacing a rule-based triage layer. Offline macro-F1 up 3 points; per-slice pass rate improved; latency budget still met offline.
- **C5 — Bring-your-own.** A production or planned candidate with an existing (or exercise-01) harness decision.

## Steps

### 1. Assemble the entry package (≈ 15 min)

Before the first stage starts, write down what has been decided *offline*:

- The `HarnessDecision` — candidate ID, baseline ID, eval config version, primary metric, overall lift + CI, and `passes_release_gate=True`.
- The three or four SLIs the auto-rollback controller will watch. These are named against mod-307 SLIs (latency p99, error rate, per-request cost, product KPI) and *not* against ad-hoc dashboards. Name each SLI and the threshold.
- The kill-switch mechanism. How is the candidate turned off in one action? A feature-flag flip? A model-registry pointer revert? A deploy of the previous artefact? Name it in one sentence.
- The stakeholders. Who is paged on rollback? Who is called before rollout is expanded past 10 %? Product-manager, on-call ML engineer, safety-team lead, someone else.

Anything you cannot name here is a gap the promotion plan will inherit. State it explicitly rather than leaving it unnamed.

### 2. Shadow-stage plan (≈ 30 min)

Write the shadow-stage section:

- **Setup.** Traffic-mirroring mechanism (side-by-side inference, request replay from a log, an async fan-out at the load balancer). How is the candidate loaded through the actual serving path — the actual model registry, feature fetch, downstream schema — rather than a shortcut?
- **Sampling.** 100 % of traffic or stratified sample across the slice matrix (chapter 02)? If sampled, why is the sample defensible against the "sampling under-represents the p99 tail" anti-pattern chapter 03 §1 flags?
- **Duration.** How long does shadow run? A business day at minimum, a business cycle for high-stakes features. Name it.
- **Exit criteria.** The five from chapter 03 §1 — latency, error rate, cost, serving-path correctness, and, if ground truth is on the request path, online primary-metric lift within the offline CI. Each has a threshold.
- **What shadow will *not* prove.** One paragraph. User-behaviour effects, feedback-loop shifts, downstream KPI direction. Name the stages (canary, A/B) that pick these up.

### 3. Canary-stage plan (≈ 45 min)

Write the canary-stage section:

- **Traffic-split choice.** Random per-request, sticky per-user, or slice-selected (chapter 03 §2)? Defend the choice against the alternative in a sentence.
- **The ladder.** Chapter 03's example ladder is 1 % → 5 % → 10 % → 25 % with defined guardrail hold-times at each step. Adapt it to your feature — a high-volume product may compress the ladder; a regulated deployment may stretch it. Name every step with a percentage, a duration, and the gate to advance to the next step.
- **Auto-rollback contract.** For each SLI from step 1, name the rollback threshold, the hold-time, and the action (page + auto-revert, page + human decide, ticket + investigate). Chapter 03 §2's list plus business-KPI direction, business-critical slice regression, safety / refusal-rate signal, cost overrun, and vendor-side degradation.
- **What the canary produces.** A canary report — the per-step guardrail readings, any rollback events, the product-KPI direction at the largest canary step. The report is the input to the A/B decision.

### 4. A/B or interleaving-stage plan (≈ 30 min)

- **A/B or interleaving?** If the feature is a user-facing ranker where interleaving applies, cite Radlinski et al. and explain why interleaving is the acceleration; A/B still runs afterward to confirm the product-KPI story. Otherwise A/B is the primary stage.
- **Metrics named in advance.** Primary product KPI, at least two guardrail KPIs, the slice matrix from exercise-01 re-enforced online. State the "no post-hoc metric addition" rule explicitly.
- **Power calculation, or a defended handwave.** For the primary KPI, name the smallest effect that would matter to the business, the significance level (5 % is common), the power target (80 % is common), and the sample size that follows. Mod-306 is where the details live; for this exercise, an order-of-magnitude number is enough — the point is that "we ran the A/B for a week" without a power target is not a defensible plan.
- **Interaction with other experiments.** If more than one experiment is running at once, what mutual-exclusion, layering, or variance-reduction machinery is applied? Handle mod-306's territory at pointer altitude.

### 5. Full rollout and postmortem (≈ 20 min)

- **Rollout mechanism.** How does traffic move from 100 % A/B to 100 % candidate — is it a flag flip, an automated pipeline, a phased regional rollout? Name it.
- **State that must survive rollout.** The prior production model artefact retained in the registry. The prior `HarnessDecision`. The A/B result. The kill-switch flag still in place.
- **Postmortem calendar.** One week and one month after rollout, look at online metrics against the offline predictions. Where did offline and online agree? Where did they disagree? Name who is on the hook to run it.

### 6. Failure-mode rehearsal (≈ 10 min)

Two paragraphs — one for each of chapter 03's named failure modes ("the offline win is real; the online serving pipe is broken" and "the offline win was on the wrong metric"). For each: which stage of *your* plan would catch it, and what specifically would trigger.

Add one domain-specific failure mode. For an LLM-augmented candidate, this often includes vendor-status-page red (mod-304 chapter 04's degradation-path work).

## Deliverable

A single Markdown memo, one to two pages, titled `release-plan-<candidate>.md`, structured as:

- Header — candidate ID, baseline ID, target rollout date, author, stakeholders.
- Entry package — `HarnessDecision` reference, SLIs, kill switch, stakeholders.
- Shadow stage.
- Canary stage (with the ladder as a table).
- A/B or interleaving stage.
- Full rollout and postmortem.
- Failure-mode rehearsal.

The memo is short by design. Reviewers hate release plans they cannot read in ten minutes; brevity is part of the deliverable.

## Acceptance criteria

- [ ] Every stage of the promotion path has explicit entry criteria, exit criteria, a duration, and an auto-rollback trigger.
- [ ] The `HarnessDecision` from an offline harness (exercise-01 or equivalent) is named as the *precondition of shadow*, not a substitute for it.
- [ ] The auto-rollback controller lists at least four SLIs (typically latency, error rate, cost, product KPI direction) with thresholds and hold-times, and each is named against mod-307 SLI vocabulary rather than an ad-hoc dashboard.
- [ ] The kill switch is a one-action operation. If it is not, that is a red rubric line to close before this candidate proceeds — cite chapter 04 Infra Test 7.
- [ ] The canary ladder has at least three steps with percentages, durations, and advance gates. Steps do not overlap; nobody negotiates skipping a step during the release.
- [ ] The A/B / interleaving section names metrics *in advance* (chapter 03 §3), states the "no post-hoc metric addition" rule, and gives an order-of-magnitude power number or a defended handwave.
- [ ] The postmortem calendar is on the plan — one week and one month post-rollout — with a named owner.
- [ ] Both of chapter 03's failure modes are rehearsed against the plan, plus one domain-specific one.
- [ ] The memo is at most two pages.

## Stretch goals

- **Interleaving for a ranker.** If you picked C1, cite the Radlinski et al. and Chapelle et al. references and design the interleaving stage in detail — how you split queries, how you count wins, how many queries you need for statistical power, and how the interleaving result feeds the classical A/B afterward.
- **Auto-rollback controller pseudocode.** Sketch the controller in pseudo-Python — a loop that reads SLI values, checks thresholds with a hold-time, and issues the rollback action. Include a unit-testable structure.
- **Vendor-outage runbook.** For an LLM-augmented candidate, extend the failure-mode rehearsal with a specific vendor-outage runbook — the fallback path (degraded response, cached response, fall back to previous prompt bundle) and the trigger.
- **Chapter 04 evidence pass.** Explicitly map each stage of your plan to an ML Test Score test (Infra Test 6 canary, Infra Test 7 rollback, Monitoring Test 6 resource regression, Monitoring Test 7 prediction quality regression) so the review packet from exercise-03 has the evidence pre-linked.
- **Feed into the paired project.** Attach this plan as the deployment section of [`project-302-llm-augmented-ml-feature`](../../../projects/project-302-llm-augmented-ml-feature/). The paired project is where the plan actually gets executed rather than authored.
