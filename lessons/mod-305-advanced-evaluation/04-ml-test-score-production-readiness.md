# The Google ML Test Score as a production-readiness review

## Motivation

You are the reviewer on a release. The team asks you to sign off. The offline harness (chapter 01) has a green decision object, the slices (chapter 02) are all passing, shadow and canary (chapter 03) look clean. Are you ready to sign?

Not yet. The harness told you whether *this candidate* is better than the current one. What it did not tell you is whether the *system around the candidate* is a production-ready ML system. A candidate can pass every offline gate on a system whose training data is unschema'd, whose feature pipeline is untested, whose rollback path has never been rehearsed, and whose live serving code is silently computing features differently than the training pipeline did. That system will ship a candidate; a few weeks later it will ship an incident.

The **Google ML Test Score**, from Breck, Cai, Nielsen, Salib, Sculley's [*The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction*](https://research.google/pubs/the-ml-test-score-a-rubric-for-ml-production-readiness-and-technical-debt-reduction/) (IEEE BigData 2017), is the rubric that turns "does this system look ready" into "here are 28 specific tests; here is which ones are automated, which are manual, and which are not done at all." It is the review packet a senior ML engineer holds a candidate release to when the release conversation has to be defensible.

Chapters 01–03 built the offline harness, the slices, and the online promotion path. This chapter is the *review* that says the whole apparatus is fit for purpose.

## What the rubric is, in one paragraph

The paper defines **28 tests** across **four categories** — data, model, ML infrastructure, monitoring — with each test scored **0** (not done), **0.5** (manually executed and documented), or **1** (fully automated and repeatable). The overall ML Test Score is the sum of the *minimum* score across the four categories times two — a *weakest-link* score, so that a team with perfect model tests and zero data tests still scores low. The paper's interpretation table maps scores to production maturity, from "0 — more of a research prototype" through "1–2 — first pass at production" up to "5+ — exceptional levels of automated testing."

The paper is short, freely available, and worth reading in full. What follows is how a senior ML engineer *uses* it as a review packet — the vocabulary, the traps to avoid, and the way each test connects to the offline harness (chapters 01–02) and the online promotion path (chapter 03).

## The four categories, at review altitude

The four categories map to four questions a reviewer walks through in order. Each category has seven tests in the paper; the summaries below are how the tests read at senior review altitude, not exact quotes.

### 1. Data tests — is the data pipeline defensible?

Seven tests. The reviewer's version of each:

- **Feature schemas.** Every feature has an expected shape — type, range, presence, cardinality. The pipeline validates against the schema and fails loud when the schema is violated. TensorFlow Data Validation, Great Expectations, and Deequ are the canonical open-source implementations.
- **All features are beneficial.** Every feature in the model has been justified by an ablation, a permutation-importance check, or a documented feature-selection experiment. Features that no longer help are removed (the paper calls this feature debt).
- **No feature's cost is unreasonable.** Feature computation costs — CPU, latency, storage, upstream-dependency fragility — are known and documented. A feature that requires an expensive upstream service the ML team does not control is a technical-debt risk, not a free win.
- **Features adhere to meta-level requirements.** Legal, privacy, and policy constraints on features are enforced in the pipeline, not "checked once and remembered." No feature silently leaks a protected class into a model that must not use one; no feature crosses a regulatory boundary.
- **The data pipeline has appropriate privacy controls.** PII stripping, retention windows, access controls, audit logging — the same discipline the security and governance tracks own. Mod-309 is where the deep review lives; this test is the reviewer's confirmation that the discipline is in place.
- **New features can be added quickly.** The pipeline is a system, not an artifact. Adding a feature takes a PR and a run, not a re-architecture. Feature debt slows every model iteration; the test's absence is why teams eventually fear touching their pipeline at all.
- **All input feature code is tested.** Feature transformations have unit tests. A feature that is silently miscomputed for six months because nobody unit-tested the SQL is a category of incident that a five-line test would have prevented.

The chapter 01 harness assumed the eval set was clean and versioned. Every one of the data tests is the reason it *stays* clean and versioned across releases.

### 2. Model tests — is the model itself defensible?

Seven tests. The reviewer's version:

- **Model specs are reviewed and submitted.** The model — its architecture, hyperparameters, training data, evaluation results — is documented, reviewed, and version-controlled. Not "someone trained it in a notebook and copied the weights to S3."
- **Offline and online metrics correlate.** The offline harness and the online product KPI are known to move together — historically demonstrated on prior releases, and named in the release review. If they do not correlate, chapter 03's promotion path is the sanity check every release passes; the test flags the underlying gap for the team to close.
- **All hyperparameters have been tuned.** Learning rate, regularisation, depth, and any decoding parameters have been searched or defended. "We used the defaults" is not a passing answer for a production model.
- **The impact of model staleness is known.** How much does model quality decay per week / month without retraining? Chapter 03's postmortem is where this is measured; the test is the reviewer's check that the answer is *known*, not "we've never measured it."
- **A simpler model is not better.** A baseline — a linear model, a hand-tuned rule set, a well-prompted frontier model — has been compared and beaten. Chapter 01's baseline discipline is this test's day-to-day operationalisation.
- **Model quality is sufficient on all important data slices.** The slice matrix from chapter 02, gated and green. This is the single test that most reliably surfaces "the aggregate looks fine but a critical slice regressed."
- **The model is tested for considerations of inclusion.** Fairness disaggregation across protected attributes where they exist. Mod-309 is where the review packet is authored; this test is where the harness enforces it.

### 3. ML infrastructure tests — is the release process defensible?

Seven tests. The reviewer's version:

- **Training is reproducible.** Same code + same data + same seeds → same model. Bit-exact where possible, statistically identical elsewhere. A model that cannot be retrained to reproduce is a model that cannot be defended in incident response.
- **Model specs are unit tested.** The model-building code has unit tests — a small synthetic input produces a known output, model saving and loading round-trips exactly, gradient computations work.
- **The ML pipeline is integration tested.** An end-to-end integration test runs the pipeline over synthetic data through to a candidate model. This is the test that catches "we changed the feature pipeline and the training pipeline broke silently for two weeks."
- **Model quality is validated before serving.** The offline harness (chapter 01) is the check; the test is that the check *runs* in the release pipeline and blocks. If the harness runs only when a human clicks a button, this test scores 0.5 at best.
- **The model is debuggable.** Predictions can be explained — SHAP, LIME, integrated gradients for tabular; attention or saliency for sequence; a per-example trace for LLM-augmented pipelines. When an incident asks "why did this input get this output," the answer is not "we do not know."
- **Models are canaried before serving.** Chapter 03's canary stage. The test is not "we ran a canary once"; it is "every release runs a canary automatically with predefined guardrails and auto-rollback."
- **Serving models can be rolled back.** Rolled back *fast* — one action, rehearsed, with a known-good baseline retained. Chapter 03's *kill switch* is exactly this. In production this is the single most under-invested test until the first time it is needed.

### 4. Monitoring tests — does the live system tell you when it breaks?

Seven tests. The reviewer's version:

- **Dependency changes result in notification.** Upstream feature-source schema changes, base-model version bumps (mod-304 chapter 02), library upgrades — the team is notified before or immediately after the change affects the system.
- **Data invariants hold in training and serving.** The schema from data test 1 is enforced live; violations page or ticket, not silently degrade the model.
- **Training and serving features compute the same values.** Training–serving skew is the failure that Sculley et al.'s [*Hidden Technical Debt in Machine Learning Systems*](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf) singled out; the test is a live check that the same input to the training pipeline and the serving pipeline produces the same feature values. Mod-302 architecture chapters are where the *design* of feature stores addresses this at architectural altitude; the test is how the team knows it is still true.
- **Models are not too stale.** The "how stale is too stale" answer (model test 4) has an alert wired: a model whose training data or retraining cadence has slipped past that threshold pages someone.
- **The model is numerically stable.** NaN outputs, out-of-range predictions, softmax collapse — the serving path validates and alerts.
- **The model has not experienced dramatic or slow-leak regressions in training speed, serving latency, throughput, or RAM usage.** The four resource dimensions the paper singles out; each one has an alert and a dashboard.
- **The model has not experienced a regression in prediction quality on served data.** The online primary metric is monitored live; a regression pages. Chapter 03's postmortem uses the same signal to feed the harness back.

Mod-305 is where the evaluation harness lives; mod-307 (ML reliability, SLOs) is where the monitoring stack is authored at depth. The seven monitoring tests are the interface between the two.

## Scoring — the weakest-link rule

The paper's scoring rule is deliberately unforgiving:

```
category_score(k) = sum of the 7 tests in category k, each 0, 0.5, or 1
overall_score    = 2 * min(category_score(data), category_score(model),
                            category_score(infra), category_score(monitoring))
```

A team with 6/7 on data, model, and infrastructure but 1/7 on monitoring scores 2 × 1 = 2. Not "roughly ready with a bit of monitoring work to do" — the paper explicitly maps 1–2 to "first pass at production." A team can lift the average by working on the two categories that were already high; the *overall score* only moves when the *weakest* category moves.

The rule is a policy, not a bug. Production ML systems fail via their weakest category; monitoring gaps are the modal cause of "we shipped it working and heard from a customer that it broke." The score design forces attention on the weakest category, which is often the one the team wants to avoid.

## The score-to-maturity map, from the paper

The paper's interpretation table (paraphrased and rounded for a review conversation):

- **0** — more of a research prototype than a production system.
- **(0, 1]** — not fully tested; possibly some serious risks lurking. Not production-ready.
- **(1, 2]** — first pass at making the system production-ready. Some serious testing but likely some risk.
- **(2, 3]** — reasonably tested; likely to be free from major issues.
- **(3, 5]** — strong levels of automated testing and monitoring; the shape you want for high-stakes systems.
- **> 5** — exceptional levels of automated testing and monitoring; the paper's aspirational upper end.

A review that arrives at "we scored 2" is not a failure — it is a starting position. The valuable review outcome is not the number; it is the *specific tests scored 0 or 0.5*, in the order the team is going to fix them.

## Using the rubric as a *review packet*, not a homework assignment

Two failure modes turn the ML Test Score into a wasted exercise.

### Failure mode 1 — score-inflation

The team, faced with a red rubric, edits the description of what "manually executed and documented" means until the score is higher. The rule that catches this: the description of *how* each test is passed is written first, evidence is attached (a link to the CI job, a runbook, a schema file, the harness config), *then* the score is assigned. If a reviewer cannot find the evidence, the score is 0 regardless of what the team wrote. Chapter 01's `HarnessDecision` object is example evidence for several tests at once — model test 6 (per-slice quality), infra test 4 (validated before serving), monitoring test 7 (regression signal on served data) if the same signals are wired live.

### Failure mode 2 — treating the rubric as a checklist rather than a diagnostic

Every test can be passed by writing a script that does something in the vague neighbourhood of the test description. The reviewer's job is not to see whether a check exists; it is to see whether the *thing the test is guarding against* is genuinely being caught. If the training-serving skew test is a job that runs weekly and emails someone, and nobody has read the email in a quarter, the test is scoring 1 in the spreadsheet and 0 in reality.

The senior read: the ML Test Score is a *conversation prompt*, not a compliance form. The value of a review is the disagreement about what specific test evidence looks like, not the final number.

## How the rubric composes with the rest of the module

- **Chapter 01 (offline harness).** Every element of a well-authored harness scores several tests at once — infra 4 (validated before serving), model 6 (slice quality), model 5 (simple baseline beaten), data 7 (feature code tested via the harness's inputs).
- **Chapter 02 (slices and adversarial).** The slice matrix is model test 6; adversarial regressions become a durable part of the release gate. Fairness slices are model test 7.
- **Chapter 03 (shadow and canary).** Shadow is a monitoring precondition; canary is infra test 6; rollback is infra test 7; postmortem is how monitoring test 2 (invariants) and monitoring test 7 (prediction quality) are grown.
- **Chapter 05 (LLM-as-judge).** LLM-as-judge inside the harness is subject to the same rubric — data tests apply to the judge's eval set, model tests apply to the judge's calibration, monitoring tests apply to the judge's drift over time. LLM-as-judge does not exempt the harness from any test; it *adds* tests.

## Cross-module hand-offs

- **mod-302 (ML systems architecture)** owns the design of feature stores and training–serving parity. Monitoring test 3 is where mod-302's design is verified live.
- **mod-307 (ML reliability, SLOs)** owns the monitoring stack at depth. All seven monitoring tests eventually map to SLIs in mod-307.
- **mod-308 (peer collaboration)** owns the platform interface — the ML platform, the MLOps track, and how the team's release pipeline is composed of platform-owned services. Several infra tests read differently for a team that owns their own pipeline than for a team that consumes a platform's paved road.
- **mod-309 (responsible AI)** owns fairness disaggregation, model cards, and data-lineage evidence. Model tests 4 and 7 and data tests 4 and 5 are where the mod-309 packet has to survive the ML Test Score's evidence bar.

## Summary

The Google ML Test Score is 28 tests across four categories — data, model, ML infrastructure, monitoring — each scored 0, 0.5, or 1, with the overall score set by the *weakest* category × 2. A senior ML engineer uses the rubric as a review packet: for every release conversation the reviewer asks "what score does this system get, and what specific tests are red?" The offline harness (chapter 01) and the online promotion path (chapter 03) are the mechanisms the tests are passed through; the rubric itself is the standard those mechanisms are held to. A team's number is less important than the specific list of red tests the review produces; the value of the exercise is the conversation the list creates.
