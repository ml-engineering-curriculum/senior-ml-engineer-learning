# Training/serving skew — at architectural altitude

## Motivation

Training/serving skew is the single largest source of "the model is 3 points AUC worse in production than on the holdout" incidents. Every mid-level ML engineer has fixed one instance of it — a bug in a feature transformation, a missing time-zone conversion, an off-by-one on a lag window. Diagnosing and fixing *an instance* of skew is a code-review problem.

The L30 job is different. Skew is the *inevitable emergent property* of the architectural choices in chapters 02 and 03: two paths (training and serving) computing "the same" feature, using different technologies (Spark SQL vs. Redis, Python vs. Go, batch vs. streaming), against different data (historical partitions vs. live requests). Any architecture that makes those two paths independent will accumulate skew. The senior job is to design the system so skew *cannot silently accumulate* — offline/online parity is a property of the architecture, not a promise from the code review.

This chapter names the categories of skew, gives you an architectural checklist that prevents each one, and shows how to instrument a system so skew that does slip through is caught in hours instead of quarters.

## The five categories of skew

Skew is not one bug — it is a family. Naming the family lets you write the mitigation into the architecture, not the ticket.

### 1. Feature-computation skew

The offline pipeline computes `avg_session_length_30d` in Spark SQL as `AVG(session_length) OVER (...)`. The online pipeline computes it in Go as an exponential moving average. The numbers do not match. Neither implementation is "wrong" — they compute *different quantities* with the same name.

Root cause: two independent implementations of the same feature.

Architectural mitigation:

- **Single definition, shared code.** If a feature store is in use (chapter 03), the transformation lives in one place and is materialised by the platform. If you are ad-hoc, package the feature computation in a library consumed by both paths, and CI-test that both callers return the same value on a shared fixture.
- **Schema pinning.** The feature vector's schema (names, types, order) is pinned in the model registry entry (chapter 03). Any change to the schema fails the deployment gate until both paths are updated.

Rules of Machine Learning ([Google Developers guide](https://developers.google.com/machine-learning/guides/rules-of-ml)) Rules #29 and #32 are the classic warnings on this failure mode.

### 2. Label leakage / point-in-time skew

The training set was built by joining today's feature values to two-week-old labels. The model learns to predict yesterday from today. Offline AUC is a fantasy; production is a disaster.

Root cause: training joins are not point-in-time correct.

Architectural mitigation:

- **Point-in-time joins are a first-class primitive.** Every training-set construction goes through an "as-of-`T`" join primitive. Feature stores (Feast, Tecton, Vertex, SageMaker) implement this natively via `get_historical_features(entity_df)`. Ad-hoc systems have to implement it explicitly with windowed joins and CI-test it against a known-leakage fixture.
- **A dedicated leakage test.** Build a training set on a fixture where the "future" is known and blocked; assert that no feature value seen at label time comes from later than label time. This test lives in the training pipeline's CI, not a notebook.
- **Feature timestamps are load-bearing metadata.** Every feature value has a `feature_ts` recorded alongside it. Training joins filter on `feature_ts <= label_ts`. Fivetran's [zero-copy time-travel](https://www.fivetran.com/) and warehouse-native time-travel features (Snowflake, Delta, Iceberg) are how you enforce this at scale.

### 3. Data-source skew

Training reads from `warehouse.users_snapshot_daily`, which is the frozen midnight state. Serving reads from `dynamo.users_live`, which is the moment-by-moment state. Even when the transformation is identical, the *input distribution* differs.

Root cause: training and serving pull from different sources of truth.

Architectural mitigation:

- **CDC into the warehouse.** Change-data-capture from the online store into the warehouse gives training a source that reflects what serving sees (with some lag). Debezium and Fivetran are the industry-standard CDC pipes; Snowflake, BigQuery, and Databricks all consume them.
- **Reconciliation checks.** A scheduled job compares samples from the training-side source and the serving-side source on the same key, and alerts on distribution divergence beyond a threshold. This is a monitoring hook, not a design promise.

### 4. Timing skew (offline pretending to be online)

The training data has "recent aggregates" that in production do not arrive until 30 minutes after the event. The model learns "in the last 5 minutes X happened." In production the last 5 minutes' data is not visible yet — the feature is systematically null or stale.

Root cause: the training pipeline sees data with less lag than the serving pipeline experiences at request time.

Architectural mitigation:

- **Simulate serving-time lag in the training window.** Every feature's real availability delay (materialisation lag, ingestion lag, warehouse ETA) is measured and *replayed* when building training sets. A feature that reliably has a 15-minute serving lag should be computed with a 15-minute lag in training.
- **Chip Huyen's *Designing Machine Learning Systems*** ([O'Reilly, 2022](https://huyenchip.com/books/)) names this as the most-under-caught category of skew in production — the code passes review because "the feature exists," but the temporal availability differs across environments.

### 5. Distribution / population skew

The training set was constructed from a filtered slice — logged-in users, US-only, non-bot traffic. Production serves every user. The model's expectation of its inputs does not match the traffic distribution.

Root cause: training population is not the serving population.

Architectural mitigation:

- **Explicit population contract in the registry entry** (chapter 03). Every model version declares "this model was trained on population `P`, which is defined as the following filter." Serving-time monitoring alerts if the observed request distribution drifts materially from `P`.
- **A slice-based evaluation harness** (mod-305) so that the offline evaluation is representative of the served population, not just easy-to-collect slices.

## The parity checklist — an architectural artefact

At L30 you write a **parity checklist** into every ML system RFC (chapter 06). Ten questions. Every "no" is either a mitigation you added or an accepted risk you named as a non-goal.

- [ ] Are training-time and serving-time features computed by the same code? If not, is there a CI test asserting the outputs match?
- [ ] Are feature values in training as-of the label timestamp, not the training-set construction timestamp?
- [ ] Do features have their real serving-time lag replayed into the training window?
- [ ] Do training and serving read from data sources that are provably in sync (CDC or shared source of truth)?
- [ ] Is the feature-vector schema pinned in the model registry entry, and enforced by the serving path?
- [ ] Is the training population contract (filters, sampling) recorded in the model registry entry?
- [ ] Is there a shadow-mode capability (chapter 05) so a new model's serving-time behaviour can be observed on real traffic before promotion?
- [ ] Is there a per-feature freshness monitor (last-materialised timestamp, hit rate on the online store)?
- [ ] Is there a per-feature distribution monitor (mean, quantile, PSI, KS) comparing training-time to serving-time?
- [ ] Is there a documented rollback path when skew is detected, with an SLO on rollback time?

The point of the checklist is not that a v1 system has to answer "yes" to every one. It is that the RFC has to *answer every one* — yes with the mitigation, or no with the accepted risk.

## Detection — how you know skew has happened

Prevention is architectural. Detection is instrumentation. At L30 you require both.

### Feature-level monitoring

For every feature that lands in the online store, emit:

- Freshness (how old is the value being served?)
- Hit rate (how often is the feature present vs. null-defaulted?)
- Value distribution (histograms, means, quantiles) — compared to the training-time distribution snapshot recorded at model registration.

Google's *Rules of Machine Learning* Rule #35 ("beware of the biases you introduce and depend on") sets the expectation; TFDV ([TensorFlow Data Validation](https://www.tensorflow.org/tfx/data_validation/get_started)) and Great Expectations ([https://greatexpectations.io/](https://greatexpectations.io/)) are common tooling. Modern MLOps platforms (Vertex Model Monitoring, SageMaker Model Monitor, Databricks Lakehouse Monitoring) offer this as managed services.

### Prediction-level monitoring

For every prediction the system emits, log a sample with the input features and the prediction. On a schedule, re-score those samples with a canonical implementation and diff against the served prediction. Non-zero diff on features that are supposed to be pure functions of the request is definitive evidence of feature-computation skew.

### Ground-truth backfill

When labels become available (30 days later for churn, 7 days later for fraud), backfill the label onto logged predictions and recompute the evaluation metrics. Compare to the offline-evaluation numbers from the registry entry. Any large gap is one of the five skew categories above.

## An architectural pattern that stitches it together

Below is a common shape that composes the chapter-02 and chapter-03 answers into a skew-resistant architecture. It is not the only shape, but it is a load-bearing one to have in your head as an L30:

- A **feature service** owns feature definitions. Every feature has one definition; the service materialises them into an **online store** (Redis/DynamoDB) and an **offline store** (warehouse table / Parquet). Training and serving both read from the feature service — no bespoke re-implementations.
- **Point-in-time joins** are the only supported way to build a training set. Ad-hoc joins fail CI.
- The **model registry** entry pins the feature service version, the population contract, and the schema. Promotion requires that the pinned feature version is present in the online store.
- A **prediction-log stream** emits `(request_id, features, prediction, model_version, feature_service_version, timestamp)` for every online prediction.
- A **replay job** runs the prediction log through the canonical feature-service definitions and asserts that the served features match the canonical features. Divergence pages on-call.
- A **backfill labeller** joins ground truth to the prediction log as it becomes available and recomputes production metrics.

This pattern is expensive. It is worth the cost for systems that have (a) a business consumer that reacts to model quality within a day, and (b) a team that will operate it. For systems without those, the ad-hoc discipline in chapter 03 is the honest answer — *with* the skew tests written into the RFC as first-class controls.

## A skew failure at architectural altitude — worked example

**Situation.** A recommender ships. Offline NDCG@10 is 0.61. Two weeks in, an A/B test shows the new model performs identically to the previous one. Product team is upset.

**L20 diagnosis path.** Look at the feature histograms in production. Compare to the training histograms. Look for feature-computation bugs. This will find *some* skew, but not the root cause.

**L30 diagnosis path.** Walk the five categories:

1. *Feature-computation skew* — replay job on 10 K live prediction samples: offline computation matches online computation on 99.98% of features. Not the cause.
2. *Label leakage* — rebuild the training set with the point-in-time primitive on the same window. NDCG drops from 0.61 to 0.52. **This is the cause.**
3. Continue to categories 3–5 to check they are not compounding.

Fix: not a code change. An architectural change — enforce the point-in-time primitive across all training pipelines, add the CI test for leakage on the known fixture, add a "training NDCG must be within `X` of a fresh point-in-time build" gate to the promotion pipeline. The specific model version is rolled back; the *system* is changed so this class of skew cannot silently recur.

That last sentence is the whole altitude difference: L20 fixes *a* leakage; L30 removes the *class* of leakage from the system.

## Summary

Training/serving skew is the emergent property of independent training and serving paths — not a code bug you can catch in review. The five categories (feature-computation, point-in-time / label leakage, data-source, timing, population) each have a specific architectural mitigation. A parity checklist in the RFC (chapter 06) forces the mitigations to be named or the risks to be accepted. Feature-, prediction-, and label-level monitoring catches what slips through. Exercise-03 has you take a real skew incident and design the *architectural* fix — the one that prevents the class, not just the instance.
