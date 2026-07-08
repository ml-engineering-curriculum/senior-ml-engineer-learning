# From business problem to system shape

## Motivation

A mid-level ML engineer is usually handed the shape of the system and asked to build a piece of it: "train a churn model against this table," "wire the ranker into the serving stack." A senior ML engineer is handed the **business problem** and asked to defend the shape. Where does the data live? Batch or streaming? Feature store or ad-hoc? What is the source of truth for "which model is live"? What are we deliberately not doing?

Those decisions get baked into the system on day one and become extremely expensive to reverse by month six. This chapter gives you a repeatable way to walk from the business problem to a defensible system architecture — the four load-bearing decisions, the constraints that pin each one down, and the artefact (the RFC, taught in chapter 06) that carries them.

The rest of the module drills into each decision. This chapter is the map.

## The four load-bearing decisions

Every production ML system architecture is, at bottom, a small set of decisions repeated at different altitudes. At L30 you should be able to reach for the same four when a new business problem lands on your desk:

1. **Serving posture.** Are predictions produced ahead of time on a schedule (**batch**) or on demand when a request arrives (**online**), or in a continuous stream against arriving events (**streaming**)? This decision drives everything downstream — feature freshness, infrastructure, cost, failure modes. Chapter 02 is entirely about this decision.
2. **Feature substrate.** Where do features live at training time and at serving time? Are they materialised into a **feature store** with a shared online/offline contract, or produced ad-hoc by whichever pipeline needs them? Chapter 03 covers this decision and its trade-offs.
3. **Model artefact contract.** What is the source of truth for "which model is live in which environment"? A **model registry** with stage transitions, or a bespoke deployment path? What metadata travels with the artefact (training data hash, evaluation slice results, provenance)? Chapter 03 also covers this — the model registry is the deployment-time twin of the feature store.
4. **Change-management posture.** How does a new model get to production? Shadow, canary, percentage rollout, kill switch, automatic rollback? How often is the model retrained, and by what trigger? Chapter 05 is about this decision.

Everything else in the module — training/serving skew (chapter 04), the RFC (chapter 06) — is downstream of these four. Skew is what happens when decisions 1–3 are not made coherently. The RFC is the artefact where you write decisions 1–4 down so the team can operate them for months.

## Walking from business problem to system shape

You do not pick the four decisions in isolation. They are pinned down by the business problem's constraints. Walk them in this order:

### Step 1 — Name the prediction and its consumer

"Predict churn" is not enough. Name:

- **The prediction unit.** Is it per user, per session, per (user, item, timestamp) tuple, per document?
- **The consumer.** Is a human reading it (analyst dashboard), a system routing on it (ranker), an automated action (auto-approve/deny)? The consumer determines what "wrong" costs.
- **The freshness the consumer needs.** Is a day-old score fine? A minute? A millisecond in the request path?
- **The volume.** How many predictions per second at peak? Per day in total?

If you cannot answer those four bullets in one paragraph, stop. Everything below assumes they are pinned down.

### Step 2 — Pin the serving posture

The freshness and volume from step 1 drive the serving posture (chapter 02). A dashboard that needs daily churn scores for 50 M users is a batch job. A ranker that needs a fraud score in the checkout request path is online. A feature that reacts to a click stream and updates a session score every event is streaming.

Cost and operability differ by an order of magnitude across these three. Choosing batch when the business needs online is a wall you hit at launch; choosing online when the business would tolerate batch is a cost and complexity tax you pay every month for years.

### Step 3 — Pin the feature substrate

Every feature the model consumes has to exist at **both** training time (offline, historical, over a large window) and serving time (online, current, low latency). The feature substrate decision (chapter 03) is: do you make that guarantee via a feature store — one code path defines the feature, and both training and serving read from the same place — or ad-hoc, with separate offline and online implementations that you promise to keep in sync?

Feature stores have a real cost (platform, adoption, contribute-back to a peer platform team, per Google's *MLOps* guide and the [mod-308] hand-off). Ad-hoc has a real cost too: training/serving skew (chapter 04), duplicate feature code, and re-implementing the same computation twice. The right answer depends on how many models share features, how volatile the features are, and whether your organisation already has a paved-road platform. Chapter 03 walks the decision.

### Step 4 — Pin the model artefact contract

Which model is live in production right now? At L20 the answer is often "the one behind that endpoint, whichever it is." At L30 that answer is not defensible. You need a **source of truth** — canonically a model registry (MLflow, SageMaker Model Registry, Vertex AI Model Registry, or an in-house equivalent) with a small number of stages (`Staging`, `Production`, `Archived`), promotion recorded as a durable event, and evaluation artefacts attached to every registered version.

Without this contract, retraining, rollback, incident response, and reproducibility all fail in the same way: you cannot say what was live, when, on what data. Chapter 03 makes this concrete.

### Step 5 — Pin the change-management posture

How does a new model get to production? Choices to make (chapter 05):

- **Rollout strategy** — shadow (traffic mirrored, no user impact), canary (small % live traffic), percentage ramp, full cut-over.
- **Retraining trigger** — cadence-based (weekly, daily), drift-based, performance-based, event-based.
- **Kill switch** — how do you revert to the previous model in under N minutes, and who authorises it?
- **Data flywheel** — do the model's own predictions and user feedback become new training data? If yes, chapter 05 warns about the feedback-loop failure modes.

These are non-negotiable at L30. If the RFC does not name a rollout strategy and a rollback path, the reviewer will send it back.

### Step 6 — Write down what you are NOT doing

The single most-under-used section in an ML system RFC is **non-goals** (chapter 06). Say out loud:

- Which slices of users the system does not serve well and is not trying to.
- Which failure modes are accepted in the first release and deferred to the roadmap.
- Which alternative architectures were considered and rejected, and why.

Non-goals bound the review. They are also how you protect your team from scope creep six months in ("but we did not sign up for that; the RFC said non-goal").

## A concrete example — the same problem, two system shapes

**Business problem.** "The product team wants to reduce credit-card fraud on our checkout."

Both shapes below are defensible. Which is right depends on the constraints. The point of the example is that the shape is not obvious from the words "fraud detection" — the six-step walk is what pins it down.

### Shape A — Batch-first, ad-hoc features, MLflow registry, shadow-then-cut

- **Prediction unit / consumer / freshness / volume.** Per-transaction, consumed by the risk team's review queue (a human), overnight refresh is acceptable, ~2 M transactions/day.
- **Serving posture.** Batch. A nightly Airflow job scores every previous-day transaction and pushes to the review queue.
- **Feature substrate.** Ad-hoc. Features are computed inside the batch scoring job from the same warehouse tables the training pipeline reads. Skew risk is low because both paths run the same Spark SQL.
- **Model artefact contract.** MLflow Model Registry, two stages (`Staging`, `Production`).
- **Change management.** Weekly retrain gated by the offline eval harness. New model shadowed (predictions written to the queue but not shown to reviewers) for a week, then promoted.
- **Non-goals.** No online scoring. No streaming. No integrated rules engine. No auto-decisioning.

### Shape B — Online, feature-store-backed, registry, canary-with-auto-rollback

- **Prediction unit / consumer / freshness / volume.** Per-transaction, consumed by the checkout service (a system), p99 <100 ms in the request path, ~800 rps at peak.
- **Serving posture.** Online. A low-latency model server behind the checkout call.
- **Feature substrate.** Feature store. Merchant-level and user-level aggregates are precomputed into an online store (Redis / DynamoDB / Bigtable). The training pipeline reads point-in-time-correct snapshots from the offline store (chapter 04 covers the point-in-time guarantee).
- **Model artefact contract.** Model registry with `Staging` / `Production` and an attached evaluation-slice card.
- **Change management.** Daily retrain, drift-triggered emergency retrains, canary (1% → 10% → 50% → 100% over 24 h) with automatic rollback on a p99-latency or false-positive-rate breach.
- **Non-goals.** No manual review queue (that is a downstream system). No text/image features in v1. No cross-user personalisation.

Shape B is dramatically more expensive to build and operate. Shape A is dramatically less capable. The correct answer is whichever one the business actually needs — and that is why step 1 (name the prediction, the consumer, the freshness, the volume) is load-bearing.

## The failure modes this walk avoids

Three failure modes are worth naming so you can catch them in your own RFCs and in reviews:

- **Over-architecting from vibes.** The team reaches for streaming, a feature store, and a bespoke registry because those are the "modern" answers. Six months later they have a Kafka topic no one owns and a feature-store platform of one. Fix: the six-step walk pins each decision to a constraint from step 1. If step 1 does not force streaming, don't build streaming.
- **Under-architecting from cost fear.** The team ships a Python script that scores from CSV nightly because "we don't need Kafka." Twelve months later the ML system has no registry, no evaluation harness, features are re-implemented in three different places, and every retrain is a heroic manual event. Fix: the model artefact contract (step 4) and the change-management posture (step 5) are not optional even for batch systems.
- **Missing non-goals.** The RFC promises everything and defends nothing. Every reviewer asks a scope question and the answer is "yes, we'll do that too." The system ships late and the team burns out. Fix: step 6 is where you say "not this, not now."

## Summary

An ML system architecture at senior altitude is four decisions repeated at different depths: **serving posture** (chapter 02), **feature substrate** (chapter 03), **model artefact contract** (chapter 03), and **change-management posture** (chapter 05). You do not pick them in isolation — a six-step walk pins each one to a business-problem constraint. Skew (chapter 04) is what happens when the four decisions are not made coherently. The RFC (chapter 06) is the artefact where the walk is written down and defended. The rest of this module drills into each decision.
