# Feature store, model registry — sources of truth

## Motivation

Chapter 02 gave you the two axes — serving posture and feature freshness — that shape the outside of an ML system. This chapter is about the two **sources of truth** that live inside it: the feature substrate and the model artefact contract.

Both look like infrastructure decisions and are actually decisions about **who owns what for how long**. The feature store answers "who owns the feature definition, and how do we guarantee the offline and online versions match?" The model registry answers "which model is live in production right now, on what data was it trained, and who authorised that promotion?" Systems without clear answers to those two questions accumulate technical debt exactly as *Hidden Technical Debt in Machine Learning Systems* (Sculley et al., NeurIPS 2015) predicts — the model is the small part, and the machinery around it eats the team.

This chapter gives you the decision rubric for each — and, as important, the rubric for when *not* to adopt one.

## Part 1 — The feature substrate

### What a feature store actually gives you

A feature store is not just "a database of features." It is a contract with four load-bearing guarantees:

1. **A single definition per feature.** The transformation logic for `user_purchases_30d` lives in one place. Training and serving read from the same definition.
2. **An offline/online dual store.** The same feature is materialised into an offline store (warehouse table, Parquet on S3) for training and into an online store (Redis, DynamoDB, Bigtable, Cassandra) for low-latency serving. The feature store is responsible for keeping them in sync.
3. **Point-in-time correctness.** When you build a training set at label time `T`, the feature store returns feature values as-of-`T` — not the current value. This is the single most important guarantee against label leakage (chapter 04).
4. **A discoverable catalogue.** Other teams can find, reuse, and register consumers of your features. Feature reuse compounds; feature duplication compounds worse.

Feast's [documentation](https://docs.feast.dev/) is the cleanest open-source reference for these four guarantees. Tecton's [feature-platform reference architecture](https://www.tecton.ai/blog/what-is-a-feature-platform/) is a good commercial contrast. AWS's [SageMaker Feature Store](https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store.html) and Google's [Vertex AI Feature Store](https://cloud.google.com/vertex-ai/docs/featurestore) are the two dominant managed offerings.

### What a feature store *costs* — the honest ledger

The feature store literature undersells the cost. At L30 you have to know it:

- **Platform cost.** A feature store is another distributed system: an offline table format, an online KV store, a materialisation pipeline (batch, streaming, or both), a metadata catalogue. Someone has to operate it. If your organisation has a peer platform team (`ai-infra-ml-platform-learning`), you consume it (see [mod-308]) and pay a coordination cost. If it does not, you either build one or you buy one — both real costs.
- **Adoption cost.** Every feature has to be *migrated* onto the feature store. Existing pipelines have their own feature code. Rewriting that code, revalidating it, and cutting over is real engineering work, often multi-quarter for a mature system.
- **Latency cost.** An online feature fetch adds a network hop. Feature-store fetches routinely dominate p99 latency for online serving. The design has to account for it — batched fetches, connection pooling, sometimes an in-process cache.
- **Consistency cost.** Even a well-run feature store has skew opportunities: schema drift between offline and online, backfill bugs, materialisation lag. Chapter 04 catalogues these.

The feature store is worth all of that when the *organisational* preconditions hold. It is not worth it when they do not.

### The adoption rubric

Do not adopt a feature store just because "we're doing ML now." The three signals below jointly indicate feature-store readiness. Missing one is a caution flag; missing two or more is a "not yet" answer.

- **Signal A — Reuse.** You have (or plausibly will have inside 12 months) more than one model consuming the same features. One model consuming its own features is an ad-hoc pipeline in disguise; the feature store's value proposition is amortised over multiple consumers.
- **Signal B — Online-serving latency budget below a few hundred milliseconds.** If your system is batch-only or has a generous latency budget, the online store is optional; you can materialise features directly into the training warehouse.
- **Signal C — A team (yours or a peer platform team) that will operate it for the next 24+ months.** Feature stores rot without ownership. A feature store no one owns becomes a feature store no one trusts.

### When "ad-hoc" is the right answer

Ad-hoc feature computation — separate offline and online implementations, with an explicit skew-checking discipline — is the correct answer when at least two of Signals A/B/C are missing. The tell-tale L30 decision is naming it explicitly in the RFC: "we are choosing ad-hoc feature computation for the following reasons, and the mitigations for skew are X, Y, Z" (chapter 04). This is *not* "no feature store because we didn't think about it" — that is the anti-pattern.

Concrete ad-hoc discipline for a system without a feature store:

- **A single feature-computation library** consumed by both training and serving code paths, with tests that assert offline-computed and online-computed values match on a shared fixture.
- **A shared schema** for feature vectors (protobuf or Pydantic) so drift in either path fails a type check.
- **Point-in-time semantics enforced in SQL** for the training set — you write "as-of-`T`" joins by hand and CI-test them against known leakage cases.

Ad-hoc *without* those disciplines is how skew ships to production.

### Worked adoption example

**Situation.** Your team owns a churn model. It is the only ML system on the team. Latency budget: overnight. Reuse expectation over the next year: none.

Rubric: Signal A missing, Signal B not applicable (batch), Signal C at risk (no peer platform team). Decision: **do not adopt a feature store**. Consume features from the warehouse in a batch job, share the feature-computation code between the training notebook and the batch scoring job via a small library, ship a schema test.

**Situation.** Your team owns a real-time recommender. Two other teams are building rankers on adjacent surfaces. Latency budget: p99 <150 ms. Peer platform team owns Feast on Kubernetes and offers it as a paved road.

Rubric: Signal A present, Signal B forcing, Signal C present. Decision: **adopt the feature store**, consume the paved-road offering, contribute back RFCs (see [mod-308]) if the shape does not fit. Write down in the RFC what happens if you push a bad feature definition (safety net, revert path).

## Part 2 — The model artefact contract

### Why a model registry is not optional at L30

At L20, "which model is live?" is usually answered by `ls` on a serving pod. At L30 that answer is insufficient — and, more importantly, undefensible when the system misbehaves.

The load-bearing question a model registry answers is: **for any past moment in time, which model version was live, on what training data, evaluated against which slices, promoted by whom?** Without an answer, the following operations cannot be done reliably:

- **Rollback.** Revert to the previous version. You need to know what "previous" is and where the artefact lives.
- **Reproducibility.** Rebuild the model from the same inputs to debug a regression. You need the training data hash and the code SHA.
- **Incident response.** When production behaviour changes, was it a model change, a data change, or a code change? The registry tells you when the model last changed.
- **Governance and audit.** Responsible-AI review (see [mod-309]) requires provenance. Regulated industries require it as a matter of law.

The registry is the deployment-time twin of the feature store. Both are sources of truth. Neither is optional at senior altitude.

### What a good model registry entry contains

The concrete artefact that carries the contract is a registered model *version*. A good version has, at minimum:

- **Model artefact URI** — where the serialised weights and config live (S3, GCS, artefact store).
- **Framework and format** — PyTorch state dict / SavedModel / ONNX / GGUF / …
- **Training data snapshot** — hash or version of the dataset(s), often a Delta / Iceberg version or a data-catalogue snapshot ID.
- **Training code SHA** — git commit for the training pipeline.
- **Feature contract** — schema (feature names, types, versions) the model expects. If a feature store is in use, the feature-service or feature-view name and version.
- **Evaluation results** — offline metrics on the golden holdout, plus slice-level results (mod-305 owns the slice discipline).
- **Signature** — the input/output signature the serving path enforces. MLflow, Vertex, and SageMaker registries all support this natively.
- **Owner** — human or team on the hook for the version.
- **Stage** — `None`, `Staging`, `Production`, `Archived` (canonical MLflow states — most registries have equivalents).

Missing any of these is a smell. Missing the training data snapshot or the feature contract is a P1 bug — it means the model cannot be reproduced or safely re-served.

### Stage transitions as durable events

The registry's value is not the metadata; it is the **stage transitions**. Promoting a version from `Staging` to `Production` is a durable event with an author, a timestamp, and an approval trail. That event is what:

- Triggers the deployment pipeline (via a webhook, a registry event, or a poll).
- Records who authorised the promotion — required for audit and incident post-mortems.
- Marks the previous `Production` version as the rollback target.
- Signals the retraining pipeline that the "current best" has changed.

MLflow's [Model Registry documentation](https://mlflow.org/docs/latest/model-registry.html) documents these transitions concretely. SageMaker Model Registry, Vertex AI Model Registry, and Weights & Biases Registry follow the same shape.

### The three anti-patterns of model-artefact management

- **The "artefact in the container image" anti-pattern.** The model weights are baked into the serving image. There is no separate registry entry. Rollback means rebuilding an old image tag; provenance is `git log` on the deployment repo. Failure mode: no way to promote/demote model version independently of code, no shared metadata, no cross-environment auditability. Fix: keep the model artefact separate from the serving image; the image is the *runtime*, the registry version is the *model*.
- **The "model in S3" anti-pattern.** Artefacts live in `s3://models/prod/latest.pt` with no version pointer, no metadata, and a mystery `latest.pt` symlink that changes on Fridays. Failure mode: no reproducibility, no rollback, no answer to "when did this change?" Fix: adopt a registry (any registry) with immutable versions.
- **The "registry as a filing cabinet" anti-pattern.** A registry is installed, but no one records evaluation metadata, no one enforces stage transitions, no one archives old versions. It exists on paper. Failure mode: same as no registry, plus a false sense of security. Fix: enforce a version-required-fields policy in CI on the training pipeline, and gate `Staging`→`Production` on an evaluation-slice check (mod-305).

### A concrete promotion contract

An L30 team should be able to state the promotion contract in six bullets:

1. Training pipeline **always** registers a model version with all required fields, or the run fails.
2. New versions land in `Staging`. Never directly in `Production`.
3. Promotion to `Production` requires:
   - The evaluation harness (mod-305) has run on the version and produced slice results.
   - The slice results pass the pre-registered gates (no regression beyond `X` on any critical slice).
   - A human on the ML team has approved the promotion in the registry (audit trail).
4. Promotion is a **durable event** that the deployment path consumes — a webhook or event stream, not a Slack ping.
5. Rollback = re-promote the previous `Production` version. The rollback path is exercised at least quarterly (game day, see [mod-307]).
6. Archival happens on a schedule for versions older than `N` months, but archival is soft (the artefact is retained; the stage is `Archived`).

Any deviation from that contract is written down in the RFC as an explicit non-goal (chapter 06).

## The relationship between the feature store and the model registry

The two sources of truth interact:

- A model registry entry names the **feature contract** it expects.
- The feature store (or ad-hoc feature library) publishes **feature versions** the registry entry can reference.
- Promoting a model to production without the referenced feature version being present in the online store is a class of skew (chapter 04). The deployment gate should refuse to promote if the feature version is not live.
- Feature deprecations must fan out to registered model versions that depend on them. This is why the discoverable catalogue (Signal A above) is not optional at scale.

Google's [*MLOps: Continuous delivery and automation pipelines in machine learning*](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) diagrams these dependencies at "MLOps level 2." Kreuzberger et al.'s [*MLOps: Overview, Definition, and Architecture*](https://arxiv.org/abs/2205.02302) survey documents the same shape academically.

## Summary

The feature substrate and the model artefact contract are the two sources of truth an ML system architecture has to name. A feature store is the right answer when the reuse, latency, and ownership signals are jointly present; ad-hoc feature computation with a shared library and explicit skew tests is the right answer when they are not. A model registry with immutable versions, required metadata, and durable stage-transition events is not optional at L30 — it is what makes rollback, reproducibility, incident response, and governance possible. Exercise-02 walks a real feature-store adoption trade-off; chapter 04 picks up the skew failure modes that ambient decisions here can either prevent or invite.
