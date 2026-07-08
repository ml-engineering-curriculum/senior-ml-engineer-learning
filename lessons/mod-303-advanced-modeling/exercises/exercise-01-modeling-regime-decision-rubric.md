# exercise-01: Modeling Regime Decision Rubric

**Estimated effort:** 3 hours

## Objective

Take three concrete business problems and walk each one through the six-question rubric from chapter 01 (`01-choosing-the-modeling-regime.md`), producing a defensible modeling-regime decision for each. The deliverable is a short decision memo — one page per problem — that a peer L30/L40 could sign off on.

This exercise trains the muscle you use every time a new modeling problem lands on your desk. The point is to force the *regime* choice — multi-task, multi-modal, SSL / transfer, ensembling / distillation, cold-start / long-tail / label-scarce, or **escalate to a specialist track** — to be pinned to constraints from the problem statement rather than to whichever regime is currently fashionable.

Getting one of the three problems to the answer "this is not our team's problem" is a passing outcome. That is the point of chapter 08.

## Prerequisites

- Read chapter 01 (`01-choosing-the-modeling-regime.md`) — the six-question rubric is load-bearing.
- Skim chapter 08 (`08-when-to-escalate-to-a-specialist-track.md`) — you will not answer well without the escalation vocabulary.
- Optional but recommended: skim chapters 02–07 so you can name the sub-decisions each regime carries (fusion altitude, per-task baseline table, calibration slices, etc.).

## Pick your three scenarios

Pick **three** from the list below. Deliberately choose three that push you into different regimes; solving three ranker problems does not train the rubric.

- **S1 — Content-safety triage on a UGC platform.** ~50 policy categories, video + audio + text upload, 500 ms end-to-end SLO, ~10 M uploads/day, 1 M labelled examples.
- **S2 — Personalised product ranking for a mid-size marketplace.** ~40 M SKUs, ~5 M weekly active users, product wants to predict click, add-to-cart, purchase, and returns together, p99 <120 ms.
- **S3 — Product-question answering on a product-detail page.** Users type free-form questions; the answer should come from the product description, reviews, and prior Q&A on the same product page.
- **S4 — Fraud-scoring auto-decisioning at checkout.** ~800 rps peak, downstream auto-approve above 0.98 / auto-decline below 0.02, human review in between; regulator asks for stated calibration on the auto-decline slice.
- **S5 — Diagnostic triage for a new medical device.** Newly cleared for a small clinical population; ~5 000 labelled cases; regulator asks for a 95 %-coverage prediction set on the "urgent" tier.
- **S6 — New-language content-classification for expansion into a low-resource language.** ~10 k labelled examples in the target language; strong pre-trained encoders exist for the source language(s), not for the target language.
- **S7 — Bring-your-own.** A production or planned system you have access to. Anonymise anything sensitive; add a one-paragraph "context I already know" preamble.

## Steps

### 1. Problem statement — one paragraph per scenario (≈ 15 min per scenario)

Following chapter 01's step 1, name for each scenario:

- The prediction unit (per user, per item, per (user, item) pair, per document, per upload…).
- The consumer (human, system, automated action, dashboard).
- The freshness the consumer needs and the volume.
- The downstream cost of "wrong" — is a threshold triggering an action? A cost calculation? A human-facing display?

Where the scenario is ambiguous, state your assumption and briefly justify it. A memo that assumes p99 <100 ms without saying so is not defensible.

### 2. Walk the six-question rubric — for each scenario (≈ 30 min per scenario)

For each of Q1–Q6 from chapter 01:

- Quote or paraphrase the question.
- Answer it with reference to your step-1 statement.
- Note whether the answer forces a regime.

At minimum, produce a decision on:

- **Primary regime** — MTL, multi-modal, SSL / transfer, ensembling / distillation, cold-start / long-tail / label-scarce, or "escalate to specialist track X."
- **Cross-cutting decisions** — is calibration required? Uncertainty required? Which peer track(s) would you consume from?

Do not skip Q4 (variance vs. systematic error), Q5 (probability-dependent decisions), or Q6 (specialist territory). Those are the questions L30s most reliably skip and are where the sharpest calls sit.

### 3. Name at least one seriously-considered alternative you rejected — per scenario (≈ 15 min per scenario)

One paragraph per scenario on a regime you considered and rejected, grounded in the rubric. If your primary answer is MTL, is a set of single-task models what you rejected? If your primary answer is a hand-off to the RAG track, is an in-house RAG build what you rejected?

Memos without a rejected alternative read as unconsidered.

### 4. Name the follow-up decisions — per scenario (≈ 15 min per scenario)

Given the regime, name in one or two sentences each the second-order decisions the module already knows are load-bearing:

- MTL → hard sharing or MMoE, per-task baseline table required, loss balancing.
- Multi-modal → fusion altitude, missing-modality slices.
- SSL / transfer → fine-tune / linear-probe / LoRA, catastrophic forgetting risk.
- Ensembling → variance vs. systematic diagnosis, distillation to fit the budget.
- Cold-start / long-tail / label-scarce → which cold-start tool, which long-tail tool, weak-supervision / active-learning plan.
- Calibration / uncertainty → which method, per which slice, what coverage guarantee.
- Escalation → hand-off contract (problem statement, evaluation, interface, on-call).

You do not have to solve these; you have to name them. The reviewer wants to know you know they are coming.

## Deliverable

A single Markdown document `modeling-regime-decisions.md`, roughly three pages total (about a page per scenario), with the following structure:

- Header — date, author, scenarios picked, any assumed constraints.
- **Scenario 1** — problem statement, rubric walk (Q1–Q6), decision, rejected alternative, follow-up decisions.
- **Scenario 2** — same shape.
- **Scenario 3** — same shape.
- **Cross-scenario reflection** — one short paragraph on what surprised you across the three, especially any place you thought the answer was "in-house" and the rubric said "escalate."

## Acceptance criteria

- [ ] Three distinct scenarios covered, each pushing the rubric toward a different regime (not three variations of one theme).
- [ ] Every regime choice cites the specific rubric answer that forced it (or explicitly notes that Q6 chose the cheapest sufficient regime when nothing forced it).
- [ ] Where the scenario was ambiguous, assumptions are explicit and briefly justified.
- [ ] At least one seriously-considered rejected alternative per scenario, with a rubric-grounded reason.
- [ ] Follow-up decisions named for each regime, drawing from the correct chapter of the module.
- [ ] At least one scenario results in an "escalate to specialist track" decision, or you explain in the cross-scenario reflection why none did.
- [ ] The memo is at most about three pages. Reviewers hate long memos; brevity is part of the exercise.

## Stretch goals

- **Sensitivity redo.** Re-walk one scenario under a modified constraint ("what if the label budget were 100x smaller?" or "what if the SLO tightened to p99 <30 ms?") and describe how the regime changes. This is the single best training exercise for "constraint sensitivity."
- **Cost envelope.** Sketch a rough monthly cost envelope (training + serving) for the chosen regime and the rejected alternative. Order of magnitude is fine.
- **Peer review.** Trade memos with a peer working through the module. Reviewer runs the chapter-08 escalation rubric against every "escalate" call and the chapter-02/03 sub-decision checklists against every in-house regime. Each "cannot find" is a memo revision.
- **Feed into the paired project.** Reuse one of your memos as the seed for the modeling section of `project-301-ml-system-design-portfolio` or `project-302-llm-augmented-ml-feature`.
