# exercise-04: Retraining Cadence and Data Flywheel

**Estimated effort:** 4 hours

## Objective

Design a full change-management posture for an ML system — retraining trigger set, cadence, data-flywheel wiring, rollout strategy, abort conditions — that a team could operate for the next six months without heroics. The deliverable is one memo, roughly 2 pages, that would populate section 4.4 (change management) and section 9 (rollout plan) of an ML system RFC.

The exercise trains the two things senior reviewers most often send RFCs back for: an under-specified retraining trigger set ("weekly retrain" with no fallback) and a naive flywheel ("we'll use production data") with no protection against feedback-loop bias, contamination, or automation bias.

## Prerequisites

- Read chapter 05 (`05-retraining-and-rollout.md`) in full — the four retraining triggers, the four flywheel failure modes, and the rollout toolkit are the load-bearing tools.
- Skim chapter 04 (`04-training-serving-skew.md`) so you can name the skew implications of your retraining cadence.
- Optional but recommended: skim the SRE Book chapter on release engineering (<https://sre.google/sre-book/release-engineering/>) for the canary-analysis framing you will reuse in step 5.

## Pick a scenario

Pick **one**.

### Option A — bring your own

An ML system with a real cadence question in front of it. Include a short "current state" paragraph: is there a retraining pipeline today? What triggers it? What is the flywheel shape? What is the rollout story?

### Option B — pick a case scenario

- **Scenario B1 — Fraud model, quarterly cadence, high-signal labels.** Labels arrive within 30 days (chargeback data). Fraudsters' tactics shift monthly. Current cadence is quarterly retraining. Rollout is a "swap the file on the pod on a Friday." No shadow, no canary, no abort condition.
- **Scenario B2 — Recommender with a strong flywheel and an active exploration budget.** Ranker learns from clicks. Currently 100% of production traffic is treated as training data. Product team asked for an "exploration slot" to try new items but has not defined it. Rollout is a percentage ramp (10% → 50% → 100%) with no automatic rollback.
- **Scenario B3 — LLM-augmented classifier with human review in the loop.** A model routes support tickets; agents accept-or-correct the routing. Correction rate is 8%. Training data comes from a nightly export of "final routes after human review." No distinction between "agent explicitly verified" and "agent didn't disagree."
- **Scenario B4 — Batch churn model with a cadence-triggered retrain.** Nightly retraining, cadence-only. No drift monitor, no performance monitor, no shadow, no canary. Rollout is "the nightly job overwrites the model artefact in place." Product team is asking why the model quality is "erratic."
- **Scenario B5 — A team migrating from ad-hoc to a real change-management posture.** The team has just deployed their first model. They have to write the retraining and rollout plan from scratch. Freshness budget: labels arrive within 7 days; user-facing metric moves within a day.

## Steps

### 1. Establish the constraint envelope (≈ 30 min)

Write, in one page or less:

- The freshness budget (how stale can predictions be? how quickly does the world underneath move?).
- Label latency (how fast do ground-truth labels arrive? are they cheap or expensive?).
- Cost envelope (how much can a retrain cost? how often is that affordable?).
- Traffic shape (is there enough traffic for a canary to detect a 1% regression on the metric you care about?).
- Any existing pipeline pieces (registered artefacts? evaluation harness? shadow capability?).

Every subsequent decision cites back to this envelope.

### 2. Design the retraining trigger set (≈ 45 min)

Following chapter 05's priority order (performance-triggered → drift-triggered → cadence-triggered → event-triggered):

- **Primary trigger** — the strongest signal you have available. Name the metric or event, the threshold, and where it lives.
- **Secondary triggers** — the backstops. Cadence-triggered is nearly always the backstop; drift-triggered often plays alongside if labels are slow.
- **Cadence choice** — if cadence-triggered is in play (primary or secondary), what cadence, and what is the reasoning (shortest-half-life heuristic from chapter 05)?
- **Event triggers** — any known upstream changes that should also kick off a retrain (schema changes, dependency-model promotions, product launches).

For each trigger, name a specific *implementation surface* — Airflow sensor, Vertex Pipelines schedule, Prometheus alert routing to a webhook, etc. Do not stop at "we'll trigger a retrain."

### 3. Design the pipeline properties (≈ 30 min)

For the retraining pipeline itself, name the five chapter-05 must-haves:

- Deterministic input contract (pinned data versions).
- Registered output (every run produces a model registry version with required fields).
- Evaluation gate (which slices from mod-305 must pass).
- Promotion policy (Staging → Production is a separate step, gated by which artefact).
- Idempotent, replayable design (how a crashed run is restarted).

Where a must-have is not yet present in the system, name it as an *open question* with an owner.

### 4. Design the data flywheel (≈ 45 min)

Even if the answer is "no flywheel — training data comes only from a curated set," you have to *say so* explicitly. If the answer is "flywheel in v1," name the four failure modes from chapter 05 and how each is mitigated:

- **Feedback-loop bias.** What fraction of traffic is deliberately randomised (exploration slot)? Where is it logged separately?
- **Direct label leakage in the flywheel.** How do you guarantee "true" labels come from an independent source, not from the model's own output?
- **Concept drift compounding.** What canonical, flywheel-independent holdout does the evaluation harness (mod-305) score against?
- **Automation bias in labels.** If a human is in the loop, how do you distinguish "actively verified" from "did not disagree"? What is the schema for the label source?

If your flywheel is high-stakes (e.g., a moderation system where mistakes cascade), the mitigations for feedback-loop bias and automation bias have to be first-class engineering, not policy. Say what that looks like.

### 5. Design the rollout strategy (≈ 30 min)

Using the chapter-05 rollout matrix:

- Is shadow required, optional, or skipped? For how long?
- Canary fraction and ramp schedule (1% → 5% → 25% → 50% → 100%, or whatever the traffic shape supports).
- Pre-registered abort conditions: metrics, thresholds, remediation (automatic rollback vs. page), approver.
- Kill switch: single-flag revert path, tested by whom, on what cadence.

For the abort conditions, be specific. "p99 latency" is not an abort condition; "p99 model-call latency exceeds 220 ms for 3 consecutive minutes, triggering automatic rollback" is.

### 6. Write the change-management section (≈ 20 min)

Compress steps 2–5 into the one-page format from chapter 05's "in one page" template. That page is what belongs in the RFC.

## Deliverable

A single Markdown document `retraining-and-flywheel-<scenario>.md`, roughly 2–3 pages, structured as:

- Header — scenario, date, author.
- **1. Constraint envelope.**
- **2. Retraining triggers.**
- **3. Pipeline properties.**
- **4. Data flywheel** — including the four-failure-mode walk.
- **5. Rollout strategy** — including pre-registered abort conditions.
- **6. One-page summary** — the RFC section.

## Acceptance criteria

- [ ] The retraining trigger set follows the chapter-05 priority order and names an implementation surface, not just a signal.
- [ ] The cadence choice is defended against a shortest-half-life heuristic — not a round-number ("weekly because weekly").
- [ ] The data flywheel section names all four failure modes and gives a concrete mitigation (or an accepted risk) for each. A memo that skips flywheel failure modes fails this criterion.
- [ ] Rollout strategy includes at least one *pre-registered* abort condition with a metric, threshold, remediation, and approver.
- [ ] Kill switch is named, along with the cadence on which it is tested (game day, see [mod-307]).
- [ ] The one-page summary in step 6 could be lifted verbatim into an RFC section 4.4 / section 9.

## Stretch goals

- **Cost model.** For your chosen retraining cadence, sketch the monthly cost envelope (compute, on-call, canary tax). Contrast against a cadence one step more frequent and one step less frequent. This is the calibration exercise that makes "weekly vs. daily" a defensible choice rather than a preference.
- **Game-day rehearsal.** Design the game-day drill that exercises the kill switch. What incident does it simulate? What is the pass/fail bar? This previews the reliability craft in [mod-307].
- **Contrast with a batch-only system.** If your scenario is online / streaming, redo the memo for a batch-only variant of the same problem. Note which of the six sections change and which do not. Change-management for batch systems is often skipped entirely at L20 and is not optional at L30.
- **Feed into the RFC.** Reuse this memo's steps 2–5 as sections 4.4 and 9 of the RFC you will produce in the paired project (`project-301-ml-system-design-portfolio`).
