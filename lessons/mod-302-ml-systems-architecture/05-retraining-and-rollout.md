# Retraining cadence, data flywheel, and shadow/canary rollout

## Motivation

An ML system is not a program you ship once. It is a program that has to be *re-shipped continuously* because the world underneath it moves. The senior job is to design a **change-management posture** — retraining cadence, data flywheel, rollout strategy — that a team can operate for months without heroics and without incidents.

Chapters 01–04 taught you how to make the system correct. This chapter is about how to keep it correct as things change. It splits into three questions: **when do we retrain?** **what does newly-generated data get to do?** and **how does a new version reach production without breaking anyone?**

## Part 1 — Retraining cadence

### Four triggers, in a priority order

At L30, retraining is triggered by one of four signals — in this priority order:

1. **Performance-triggered.** A production quality metric (offline or online) breaches a pre-registered threshold. This is the strongest signal because it is the thing you actually care about. Requires ground-truth labels arriving quickly enough to notice.
2. **Drift-triggered.** A distribution monitor (feature drift, prediction drift, PSI, KS) breaches a threshold. Weaker than performance-triggered (drift is a proxy) but faster because it does not need labels.
3. **Cadence-triggered.** A schedule (nightly / weekly / monthly). The default backstop when the above two are absent or unreliable. Google Cloud's [*MLOps* guide](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) treats this as the "MLOps level 1" retraining shape.
4. **Event-triggered.** A known upstream change — a schema change on an input source, a promotion of a dependency model, a product-team feature launch — kicks off a retrain because the reviewer knows the model's distribution just shifted.

The strongest signal you have available wins. Weaker signals stay in place as backstops.

### Picking the cadence

For cadence-triggered retraining, the cadence itself is a design choice. Wrong answers lurk on both sides:

- **Too frequent.** Retraining every day when the underlying data moves every quarter — you pay compute, add churn, invite instability, and create a rolling target for post-mortems.
- **Too infrequent.** Retraining quarterly when the world moves weekly — the model silently decays; performance monitoring catches it only after user harm.

The senior heuristic is to time the cadence to the **shortest half-life of any critical feature or label distribution**. If your fraud model relies on a fraud-typology mix that shifts monthly, retrain at least monthly. If your recommender's item catalogue turns over weekly, retrain at least weekly. Then err *slightly* on the frequent side to leave room for shocks.

Chapter 04's skew categories are load-bearing here too: a system with strong point-in-time primitives and shared feature code (chapter 03) can retrain safely more often than one with ad-hoc pipelines. Retraining cadence is downstream of feature-substrate quality.

### The retraining pipeline is a system

A retraining pipeline that runs unattended has to have:

- **A deterministic input contract.** Given a start and end timestamp, the pipeline produces the same training set. Data versions are pinned (Delta / Iceberg version, warehouse snapshot ID). Ad-hoc "SELECT ... WHERE ts > NOW() - 30 days" is the anti-pattern.
- **A registered output.** Every run produces a registered model version with the fields chapter 03 requires. No successful run without a registered artefact.
- **An evaluation gate.** The evaluation harness (mod-305) runs on every candidate; failing the gate does *not* promote. The next scheduled run will try again with fresh data.
- **A promotion policy.** Passing the gate marks the version `Staging`. Promotion to `Production` is a separate step (see Part 3), typically after shadow or canary observation.
- **An idempotent, replayable design.** A crashed run can be safely restarted from a checkpoint or re-executed against the same inputs. This is the Google *MLOps* "level 2" property.

The Kreuzberger et al. [*MLOps* survey](https://arxiv.org/abs/2205.02302) documents the shape of these pipelines academically; the MLflow, Kubeflow Pipelines, and Vertex Pipelines documentation covers the tooling.

## Part 2 — Data flywheels (and their failure modes)

### What a data flywheel is

Systems that log their own predictions and receive feedback from their consumers are producing new training data as they run. That data — cheap, plentiful, distributionally on-target — is the "data flywheel": the more the system runs, the more training data it accumulates, the better the next model gets.

Recommenders, rankers, search, ad prediction, moderation, active-learning loops, and any system with human feedback all have flywheel dynamics. LLM applications with thumbs-up/thumbs-down feedback are a particularly hot version of it. Chip Huyen's [*Designing Machine Learning Systems*](https://huyenchip.com/books/) covers the good and bad shapes at length.

### The four failure modes

Data flywheels *look* like free training data. They are not. Four failure modes are worth naming, because each is common and each is architectural:

- **Feedback-loop bias.** The model produces predictions that cause the actions that get logged as labels. A ranker that never surfaces item `X` never gets feedback on `X`; the training data comes to look like "no user ever wants `X`." Fix: reserve exploration traffic (e.g., epsilon-greedy exploration, contextual bandits, or a fixed "randomised" fraction) and log it separately for training. Sculley et al.'s [*Hidden Technical Debt*](https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html) names this class as "feedback loops" and it is the archetypal source of silent degradation.
- **Direct label leakage in the flywheel.** The flywheel logs the *predicted* label and downstream systems re-use it as the "true" label. Fix: labels used for training must come from an independent source (human labelling, delayed ground truth, an oracle), not from the model's own output.
- **Concept drift compounding.** The flywheel amplifies whatever direction the world is moving in — good if the direction is a genuine trend, catastrophic if the direction is a bug or an abusive traffic source. Fix: hold out a "canonical" evaluation set that is *not* affected by the flywheel, and gate every retrain against it (mod-305).
- **Automation bias in labels.** When a human is "reviewing" model predictions and their default action is to accept, the log of accepted predictions is not a training signal — it is a self-reinforcing endorsement. Fix: measure the human's disagreement rate, sample randomly for full re-labelling, and separate "accepted" from "actively verified" labels in the schema.

### Wiring a flywheel that does not eat you

An operable data flywheel has the following load-bearing properties:

- **Exploration traffic is a first-class flag.** A configurable fraction of serving traffic is deliberately randomised (or epsilon-greedy) and logged separately. That data is the debiased sample the next training run consumes.
- **The training pipeline distinguishes label sources.** Automatically-logged labels, human-verified labels, and randomised-exploration labels are three separate columns; the training recipe recomputes with each subset and reports how much each contributes.
- **The canonical holdout is immune.** The offline evaluation harness (mod-305) reads from a snapshotted, flywheel-independent evaluation set. Its purpose is to detect flywheel-induced drift.
- **The kill switch is model-independent.** If the flywheel is contaminated (a bad model version poisoned the label stream for a week), you have a way to roll the *dataset* back to a pre-contamination snapshot, not just the model. Delta / Iceberg time-travel is the mechanism; the discipline is knowing to invoke it.

## Part 3 — Shadow, canary, and rollout patterns

### Why "just deploy it" is not a rollout strategy

A new model version, no matter how well-evaluated offline, can behave differently in production for reasons chapter 04 catalogued: skew, timing, population, latency. The rollout strategy is what lets you *observe* the new version's real-world behaviour before it can hurt anyone, and *revert* it fast when it does.

L30 systems have at least three rollout tools available and know which one to reach for.

### Shadow deployment

The new version runs alongside the current production version, receives the same inputs, and its predictions are logged but not returned to the caller. The user sees the current model's output; the team sees both.

Use for:

- Comparing the new model's predictions to the current model's on real traffic before any user impact.
- Latency and throughput profiling under real load (shadow catches the "it doesn't fit in the p99 budget" surprise cheaply).
- Skew detection (chapter 04): the shadow's feature histograms on real traffic, compared to the training-time histograms.

Caveats:

- Shadow doubles the model-serving cost while running. Budget for it.
- Shadow does not measure user-facing metrics — no clicks, no conversions, no session data on the shadow. It measures *the model's output*, not the *product outcome*.
- Shadow can have side effects if the model has any (e.g., calling an external API). Enforce read-only shadow paths.

### Canary deployment

A small percentage of live traffic (typically 1%, then 5%, 10%, 50%, 100%) is routed to the new version. The caller sees the new model's output for that fraction; automated monitors watch for degradation on:

- User-facing metrics (CTR, conversion, revenue-per-user, complaint rate).
- System metrics (p50 / p95 / p99 latency, error rate).
- Model quality proxies (prediction distribution vs. baseline).

A pre-registered abort condition triggers **automatic rollback** — the canary fraction snaps to 0% and the previous model resumes serving 100%. Google's [SRE Book, chapter 8 — *Release Engineering*](https://sre.google/sre-book/release-engineering/) and the *Workbook*'s canary-analysis material are the canonical references.

Use for:

- Any change that can affect user-facing metrics (i.e., basically every model change).
- Any change to serving-time infrastructure (feature-service versions, model server versions).

Caveats:

- The canary fraction has to be *large enough to be statistically detectable* on the metric you care about. A 0.1% canary against a low-signal metric is theatre.
- The abort condition has to be pre-registered — deciding it in the moment during an incident is a recipe for slow rollbacks.
- The canary must sample fairly (per-user hashing, not per-request random, to keep a user in a consistent bucket).

### Percentage ramp

Once the canary has held, a scheduled ramp increases the fraction on a cadence (e.g., 1% → 10% at hour 1, 10% → 50% at hour 6, 50% → 100% at hour 24). Automatic rollback stays armed the whole time.

### Kill switch

Independent of the ramp, the system exposes a single-flag revert to the previous `Production` version. The revert path is exercised at least quarterly as a game-day drill (see [mod-307]).

### The rollout matrix

A useful mental table for choosing among the tools:

| Situation | Shadow | Canary | Ramp | Kill switch |
|---|---|---|---|---|
| New model, same architecture, low risk | Optional | 1% → ramp | Yes | Always armed |
| New architecture / serving stack change | Required (latency, skew) | Yes, small (0.5–1%) | Slow (days) | Always armed |
| Emergency retrain in response to incident | Skip if urgency justifies | 1% → 5% inside minutes | Aggressive | Primary revert |
| Fine-tune / prompt change on LLM-integrated feature (see [mod-304]) | Required (eval slice) | Yes | Yes | Always armed |

### The pre-registered abort condition

The single most-under-specified element in L20 rollout plans is the abort condition. At L30 it is required, and it is *pre-registered*:

- On which metrics? (Named quantities — "checkout success rate," "p99 model-call latency," "false-positive rate on the fraud slice.")
- At what threshold? (Absolute or relative, with a statistical significance bar or a raw guardrail — "checkout success rate drops by >0.2 pp for 5 consecutive minutes.")
- With what remediation? (Automatic rollback vs. page on-call.)
- Approved by whom? (Not the person doing the rollout — an on-call peer.)

The abort condition lives in the RFC (chapter 06) and in the runbook (mod-307). Neither the reviewer nor the on-call should be discovering it in the moment.

## Putting it together — a change-management posture in one page

A defensible change-management section of an ML system RFC looks like this:

- **Retraining trigger.** [Performance-triggered on metric M / drift-triggered on PSI on features F / cadence-triggered weekly / event-triggered on upstream change U.] Fallback: cadence-triggered `<N>`.
- **Retraining pipeline.** [Which platform, which orchestrator, which registry, gated by which evaluation slices.]
- **Data flywheel.** [What labels come from where; what fraction is exploration traffic; what canonical holdout is immune.]
- **Rollout strategy.** [Shadow for the first `<W>` weeks / canary at `<X>%` for `<Y>` hours / ramp to 100% over `<Z>` hours / kill switch armed.]
- **Abort conditions.** [Named metrics, thresholds, remediation, approvers.]
- **Rollback path.** [How to revert in `<T>` minutes. Practised quarterly.]

That is the artefact the reviewer signs, the on-call inherits, and the team lives with for months.

## Summary

Retraining cadence is picked to the shortest half-life of any critical distribution and backstopped by drift and performance triggers. Data flywheels are useful and dangerous — they need exploration traffic, a flywheel-independent holdout, and label-source discipline. Rollout is a three-tool kit: shadow (observe cheaply), canary (fail small and fast), ramp (widen deliberately), with a pre-registered abort condition and a rehearsed kill switch. Chapter 06 is where all of this gets written down as an artefact the team can defend and operate. Exercise-04 is a real retraining-cadence and data-flywheel design; the RFC exercise assembles all five chapter decisions into one document.
