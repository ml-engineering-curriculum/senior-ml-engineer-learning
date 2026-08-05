# Data lineage and PII handling from ingestion to inference

## Motivation

Chapter 01's model card includes a training-data section that says *what* data was used, *who* it represents, and *what PII* it contains. Every one of those claims has to be *provable*. Provable means the reviewer — governance analyst, compliance officer, privacy engineer, external auditor — can trace a specific field in a specific inference request back through the feature pipeline, back through the training set, back to the ingestion boundary, and see at each hop *what happened to the data*: was it transformed, joined, filtered, redacted, hashed, dropped?

That trace is the **data lineage graph**. Without it, the model-card claim "no direct identifiers appear in training data" is an assertion of hope. With it, the claim is a verifiable statement: the graph shows a specific PII-removal node between the raw ingest and the feature-view materialisation, the node's implementation is version-pinned, and its output has been sampled and scanned.

The load-bearing shift from L20 to L30 is that **lineage is not an observability nice-to-have; it is the evidence base under the model card**. The senior ML engineer designs the lineage collection *before* the training pipeline runs, not after the reviewer asks. And the discipline generalises past PII: lineage is what lets you answer "was the training set contaminated by test data," "which downstream model depends on a feature we are about to deprecate," and "what is the blast radius if source X was misconfigured for a week."

Primary references:

- **[GDPR Article 30](https://gdpr.eu/article-30-records-of-processing-activities/)** — the "records of processing activities" obligation. The regulatory model for why lineage exists in the first place, even outside the EU.
- **[NIST Privacy Framework](https://www.nist.gov/privacy-framework)** — the *Identify → Govern → Control → Communicate → Protect* functions, whose *Identify* function is where the data-mapping and lineage discipline lives.
- **[OpenLineage](https://openlineage.io/)** — the open standard for lineage event emission that plugs into Airflow, Spark, dbt, and modern orchestrators.
- **[NIST SP 800-188 — De-Identification of Personal Information](https://csrc.nist.gov/publications/detail/sp/800-188/final)** — the reference for PII taxonomy, direct vs. quasi-identifiers, and the k-anonymity / l-diversity / t-closeness / differential-privacy discipline.

## §1 — Lineage: the graph, the nodes, the edges

A data lineage graph, at ML-team altitude, has five node kinds and two edge kinds. The node kinds:

- **Sources.** Raw inputs. Application databases, event streams, third-party APIs, uploaded files, human labels. Each source has an *owner* (who runs it), a *contract* (schema, freshness, retention), and a *jurisdiction* (where the data physically lives and under whose law).
- **Transformations.** The computations that produce new data from existing data — SQL views, Spark jobs, Beam pipelines, dbt models, feature-view definitions, prompt-templating steps, embedding-generation nodes. Each transformation has a *version* (the code producing it) and a *materialisation policy* (batch, streaming, on-demand).
- **Storage.** Where data at rest sits — data lake objects, warehouse tables, feature-store online tables, model artefacts, prompt libraries, evaluation datasets. Each storage node has a *retention policy* and an *access-control policy*.
- **Consumers.** What reads the storage. Training pipelines, inference services, evaluation harnesses, human dashboards, analytics queries. Each consumer has an *identity* (which service, which role) and a *purpose* (what it does with the data).
- **Sinks.** Where data eventually leaves the system — the inference-response bytes returned to the user, the aggregated report sent to a partner, the logging pipeline that lands in the observability system, the audit trail that lands in the archive.

The edge kinds:

- **Data-flow edges.** The bytes go this way. An event lands in the ingest topic → the streaming job transforms it → the feature-view materialises it → the training set consumes it → the model reads it → the inference response emits it.
- **Reference edges.** The bytes do not flow, but a decision depends on them — the join key that maps a request to a feature-view row, the pointer that lets an inference response *cite* a retrieval result (mod-304 chapter 03's RAG context), the ID that lets a log line be linked to a user record.

Two properties of the graph that matter for governance:

- **The graph is directed, but the *provenance question* is bidirectional.** "Where did this inference-response field come from?" is a backwards traversal. "What is the blast radius if source S is bad?" is a forwards traversal. Lineage tooling that only supports one direction is only half-useful.
- **The graph is dynamic, but the *audit* is static.** For any single inference request or training run, you can pin the exact graph that produced it (the *snapshot lineage*). The audit is against the snapshot. The dynamic graph (all possible flows, over time) is for capacity planning and impact analysis, not for the model card.

## §2 — The reference lineage: ingest → feature → training → serving → sink

The canonical shape of an ML system's lineage graph is a five-tier flow. Naming the tiers explicitly makes the PII conversation tractable — you name at *which tier* PII is present, at *which tier* it is transformed, and at *which tier* it must be gone.

```
[T1: Ingest]
    raw event streams, application DB CDC, third-party API pulls, uploaded corpora, human labels
        │  raw records with source-native identifiers (email, phone, device ID, session ID, name)
        ▼
[T2: Normalisation and PII handling]
    schema validation, deduplication, PII detection, PII redaction / hashing / tokenisation,
    consent-and-purpose filtering, record-level lineage stamping
        │  normalised records with policy-classified identifier fields
        ▼
[T3: Feature computation]
    feature-view materialisation (offline + online), aggregations, embeddings, joins to reference data
        │  feature tables with join keys (often hashed pseudonymous IDs)
        ▼
[T4: Training + Serving]
    training set assembly, model fit, model artefact, inference pipeline (feature fetch → predict → post-process)
        │  model artefacts, prediction records, decision logs
        ▼
[T5: Sinks]
    inference responses returned, prediction logs, feedback signals, downstream analytics, audit logs, monitoring exports
```

Every arrow between tiers is a node in the lineage graph and a *checkpoint* the discipline asks about:

- **T1 → T2.** What is in the raw data? What identifiers are direct (name, email, phone), what are indirect / quasi-identifiers (ZIP + birthdate + gender, per Sweeney 2000), what is sensitive-category (health, financial, biometric, sexual orientation, religion — the GDPR Art. 9 categories)? What is the *lawful basis* for processing?
- **T2 → T3.** What survives after normalisation? What was hashed, tokenised, dropped, aggregated? Which fields carry policy-classification labels forward?
- **T3 → T4.** Which features enter the training set? Which of them, if any, are still-identifiable? Which of them are *proxies* for a sensitive category even if their name is neutral (ZIP as a proxy for race in US mortgage lending is the [Angwin et al. COMPAS](https://www.propublica.org/article/machine-bias-risk-assessments-in-criminal-sentencing) example every governance analyst knows)?
- **T4 → T5.** What appears in the inference response? Are we accidentally leaking a retrieval-augmented context that contains PII (the RAG hazard mod-304 chapter 03 named)? Are the prediction logs storing full requests including PII, and if so, under what retention?

The lineage graph is the *evidence* under each of these checkpoints. The governance question at each checkpoint is answered by *pointing at a specific node in the graph*, not by the ML team's assurance.

## §3 — PII taxonomy: direct, quasi, sensitive, and derived

The PII conversation is impoverished if PII is treated as a single label. Four categories, with different governance treatment:

- **Direct identifiers.** Fields that alone identify a person — name, email, phone, government ID, biometric template, exact home address. Regulatory treatment is strict: consent, minimisation, and (for GDPR / CCPA jurisdictions) explicit purpose limitation.
- **Quasi-identifiers.** Fields that alone do not identify a person but *in combination* do. Sweeney's 2000 result — 87 % of the U.S. population is uniquely identifiable by (5-digit ZIP, birthdate, gender) — is the load-bearing empirical result the modern k-anonymity literature ([Samarati & Sweeney, 1998](https://epic.org/wp-content/uploads/privacy/reidentification/Samarati_Sweeney_paper.pdf)) grew out of. Regulatory treatment depends on jurisdiction and use, but the operational discipline is that quasi-identifiers require joint analysis, not per-column analysis.
- **Sensitive-category attributes.** Under GDPR Art. 9 — racial or ethnic origin, political opinions, religious beliefs, trade-union membership, genetic data, biometric data, health data, sex life or sexual orientation. Under HIPAA — protected health information. Under FCRA — credit information. Under COPPA — data from children under 13. Each has its own rule set; sensitive-category attributes typically require *explicit* consent and *stronger* minimisation.
- **Derived / inferred attributes.** Attributes *your model produced* that are themselves sensitive — an inferred age, an inferred household income, an inferred pregnancy status (the [Target 2012 pregnancy-prediction incident](https://www.forbes.com/sites/kashmirhill/2012/02/16/how-target-figured-out-a-teen-girl-was-pregnant-before-her-father-did/) is the reference incident). Derived attributes are governed the same way as the raw attribute would be — the fact that you *inferred* it does not make it not-sensitive.

The senior discipline is that the model card's data-statement section names each category *separately*. "This model was trained on user account data" is not enough; "trained on user account data containing direct identifiers <A, B>, quasi-identifiers <C, D>, no sensitive-category attributes, and derives inferred attribute <E> which is treated as sensitive category <F>" is a claim a reviewer can act on.

## §4 — The five-column PII register

The artefact that anchors this chapter is a **PII register** — a table your team maintains that maps every field in the lineage graph to its PII category, its provenance, its transformation history, and its exposure surface. This is the artefact the model card's data-statement section (chapter 01 §2) points at when it makes PII claims.

The five columns:

- **Field.** Fully qualified name — `source.table.column` or `feature_view.feature.name`.
- **Category.** Direct / quasi / sensitive / derived / non-PII, with the specific sub-category (email, DOB, ZIP, race, inferred-age).
- **Provenance.** Where the field enters the graph (which source at T1) and how (which normalisation node at T2). If it is derived, name the model / feature-view that produced it and the fields it was derived from.
- **Transformation policy.** What happens at each tier — passed through, hashed with named salt, tokenised, aggregated (with the aggregation rule), dropped, redacted (with the detection rule). The *specific* policy, not "we handle it."
- **Exposure surface.** Where the field can appear in output — inference response, prediction log, monitoring metric, audit log, aggregated dashboard, retention-bound archive. Each exposure has an *allowed-audience* label (end user, on-call, security team, external partner) and a *retention* value.

Example:

```markdown
# PII register — recommendation-serving team, feature-name-v3

| Field | Category | Provenance | Transformation policy | Exposure surface |
|---|---|---|---|---|
| `events.raw.user_email` | Direct (email) | T1 source `iam.users` CDC, ingested via Kafka topic `users.v2` | T2 normalise: dropped from record; hashed with `HMAC-SHA256(salt=user_id_salt_v1)` into `user_id_h` for join key only | Not exposed. `user_id_h` used only for feature-store join; hash appears in prediction logs but is one-way and not reversible without the salt. |
| `events.raw.dob` | Quasi (DOB) | T1 source `iam.users` CDC | T2 normalise: bucketed into `age_band` (18-24, 25-34, ...) at record ingestion; raw DOB dropped from record before T3 | `age_band` appears in training set and prediction logs with 90-day retention. |
| `events.raw.zip5` | Quasi (ZIP) | T1 source `iam.users` CDC | T2 normalise: truncated to `zip3` before T3 to reduce re-identification risk; joined with census `median_income_by_zip3` to produce `median_income_estimate` feature | `zip3` appears in training set. `median_income_estimate` appears in training set and prediction logs. |
| `events.raw.session_click_stream` | Non-PII (behavioural signal, no direct identifier) | T1 source `events.clickstream` topic | T2 normalise: enrichment adds `user_id_h`; pass-through | Feature-view `click_history_features` at T3; not directly in response. |
| `inferences.raw.predicted_intent` | Derived / not sensitive | Model output at T4 | Aggregated and stored in prediction log with `user_id_h`, 90-day retention | Prediction log; inference response returns aggregate score, not raw intent labels. |
```

Three properties of this register that matter:

- **Every row's transformation policy is a *specific* rule, not a hand-wave.** "Dropped from record" names the transformation node; "hashed with HMAC-SHA256(salt=...)" names the exact primitive. This is what makes the register *auditable*.
- **The register enumerates fields, not features.** A single feature at T3 may be derived from many T1 fields; each T1 field gets its own register row. The lineage graph is the crosswalk from register rows to features.
- **The register is versioned with the model.** Every new field, every changed transformation rule, every new sink is a register update. The model card's data-statement section cites the register version.

## §5 — Consent, purpose limitation, and retention

Three regulatory disciplines that the lineage graph and the PII register have to *respect*, not merely record. These are where the ML team most often stumbles into a violation without noticing.

### 5.1 — Consent

Each user's data was collected under a specific consent. If the consent said "we use your data to provide the service," it did not say "we train ML models on it." The ML use may be permissible under the same legal basis (legitimate interest, contract performance) or may require *additional* consent. The lineage tag `consent_scope` on every record is the operational primitive that makes this enforceable — the training pipeline filters out records whose `consent_scope` does not include ML training. GDPR's *purpose limitation* principle (Art. 5(1)(b)) is where this discipline lives.

The senior discipline is to have the consent-scope filter as an *explicit* node in the lineage graph at T2, not as a `WHERE consent = 'true'` clause hidden inside a query. The explicit node is what the audit can point at.

### 5.2 — Purpose limitation and secondary use

Even with valid consent, using training data for a *different* purpose is a policy event. If the training consent was for a fraud-detection model and you now want to train a recommendation model on the same data, that is a secondary use — the analyst evaluates it as its own case. The PII register's *provenance* column and the model card's *intended-use* section are what let the analyst do that evaluation without archaeology.

### 5.3 — Retention and deletion

Retention is a hard time budget on the storage nodes in the lineage graph. GDPR's *storage limitation* principle, CCPA's deletion rights, sector-specific rules (HIPAA's 6-year retention for many artefacts, SR 11-7's 5-year retention for model documentation) all constrain how long data lives. The lineage discipline is that every T3 and T5 storage node has a *retention field* on its metadata; the enforcement is a scheduled job that deletes past-retention data and *emits a lineage event* recording the deletion.

The subtle case is **model artefacts as extensions of training data**. Large language models memorise training data verbatim — the [Carlini et al. 2020 extraction attack](https://arxiv.org/abs/2012.07805) and [Carlini et al. 2022 quantifying memorisation](https://arxiv.org/abs/2202.07646) established that a fraction of training records are recoverable from the trained model. This means the *retention obligation* on training data extends to the model artefact: deleting a training row without either retraining without it or applying an unlearning technique leaves the row recoverable. The senior discipline is to *know this* and to have a documented position (in the model card §9) on how deletion requests are handled — retrain-on-schedule, targeted retrain, unlearning, or documented residual risk with legal signoff.

## §6 — The three ways lineage collection actually happens

There are three practical patterns for collecting the lineage graph in production, in ascending order of coverage.

- **Manual documentation.** A Confluence page authored by hand. Cheap to start, impossible to trust past the first three edits. Useful only as a *stopgap* while a real collector is stood up.
- **Metadata-store scraping.** An automated crawler over the org's warehouse metadata (dbt manifest, Airflow DAG parsing, Spark logs, feature-store catalog). Covers the *declared* transformations. Misses ad-hoc queries, notebook joins, and any transformation that happened outside the crawled tools. The [DataHub](https://datahubproject.io/) and [Amundsen](https://www.amundsen.io/) open-source projects sit here.
- **Runtime event emission.** The transformation frameworks emit lineage events at run time — every job start, input read, transformation applied, and output write becomes an event that lands in a lineage backend. [OpenLineage](https://openlineage.io/) is the emerging standard; [Marquez](https://marquezproject.ai/) is the reference backend. Covers the *actual* transformations, at the cost of instrumenting every framework and running the backend.

The senior discipline is to know which pattern the org has, know its coverage gaps, and *not* claim more coverage in the model card than the collector supports. If the collector is metadata-store scraping and there is a Spark notebook that joins a T1 source directly into the T3 feature-view (bypassing the T2 normalisation), the lineage graph will *look* clean and the reality will not be. The gap-finding discipline of §7 exists to catch this.

## §7 — Finding the gaps: the lineage audit

The audit that anchors this chapter is a **lineage audit** — a systematic walk of every T1 source through to every T5 sink, verifying that the graph the collector shows matches the graph the code actually produces, and flagging every discrepancy.

The audit steps, in the order they catch the most problems:

- **Start at T5 (the sinks) and walk backwards.** Every field in every sink (inference response, prediction log, monitoring metric, audit log, aggregated report) must trace back through the graph to a T1 source. Any field the graph cannot explain is a *ghost field* — either the collector missed a transformation, or the code is producing a field the schema did not sanction.
- **Walk T1 forward, and flag anything that reaches a sink without traversing T2.** Any path from a raw source to a sink that does not go through the PII-handling node at T2 is a *lineage bypass*. This is where the register-vs.-reality gaps hide.
- **Compare the PII register against the graph.** Every register row must correspond to at least one node in the graph. Register entries with no matching node are stale (the field was removed but the register wasn't updated); graph nodes carrying candidate-PII fields not in the register are silent (the field is being handled without the register accounting for it).
- **Sample a real inference request and reconstruct its snapshot lineage.** For a single request, name every feature-view it read, every model version that predicted, every log line and metric it produced, and every retention-bound artefact it landed in. If reconstruction fails, the lineage collector is not capturing the request-scoped provenance.
- **Sample a single deleted-user request.** For a user whose deletion request was executed, verify that every T3 and T5 sink either no longer contains their data or is documented as an exception (model artefacts with the disclosed residual risk of §5.3, for instance). If the graph does not answer this, the deletion pipeline has a coverage gap.
- **Look for cross-tier joins that skip normalisation.** A T3 feature-view that joins directly to a T1 source without going through T2 is a bypass in disguise; the join key may be PII that was supposed to be tokenised before T3 saw it.
- **Look for prediction logs that store more than they should.** A common failure mode: the prediction-logger records the *entire feature vector* including PII quasi-identifiers, under a retention longer than either the source or the register sanctions. Log-schema drift is where lineage gets quietly wider over time.

The audit's output is a list of **findings** — each a specific gap between the graph, the register, and the code — plus a **severity** (P0 blocking the promotion, P1 requires fix before next review, P2 tracked for the roadmap) and an **owner**. Exercise 02 is the walk-through of authoring one of these audits.

## §8 — Composing with the model card

The lineage graph and the PII register are the *evidence base* for the model card's data-statement section (chapter 01 §2). Concretely, the model card cites:

- **The lineage graph version.** By commit hash of the graph export, so a reader at review time can look at the *same* graph the ML team was looking at when they authored the card.
- **The PII register version.** Same.
- **The audit report.** The most recent audit's findings, closed items, and open items.
- **The specific claims and their audit hooks.** For each PII-related claim in the model card ("no direct identifiers in training data"), the specific lineage-graph node or register row that supports it.

The reverse also holds: the model card's cadence policy (chapter 01 §7) drives the lineage-audit cadence. A card that is due for re-review needs a lineage audit no older than the review; a card that is model-triggered by a retrain needs a lineage audit that covers the retrain's inputs.

The senior discipline: no PII claim goes in the card that the lineage graph and the register cannot back. If the graph cannot prove the claim, the claim comes out of the card and the finding lands as an open item on the audit.

## Summary

Data lineage is the *evidence base* under the model card's data-statement claims — without it, the claims are assertions of hope. The graph has five node kinds (source, transformation, storage, consumer, sink) and two edge kinds (data-flow, reference), and the ML system's canonical shape is a five-tier flow (ingest → normalisation + PII handling → feature computation → training + serving → sinks). PII is not a single label but four categories — direct, quasi, sensitive, and derived / inferred — each with different regulatory treatment. The **PII register** is the five-column artefact (field, category, provenance, transformation policy, exposure surface) that maps every field in the graph to its handling; every register entry is a *specific* rule, not a hand-wave. Consent, purpose limitation, and retention are three regulatory disciplines the graph has to respect — consent-scope filter as an explicit node, purpose limitation as a decision the model card documents, retention as a scheduled deletion with emitted lineage events. Model artefacts extend the retention obligation to memorised training rows. Lineage collection happens in three patterns (manual, metadata-store scraping, runtime event emission); the senior discipline is to know which pattern is in force and not claim more coverage than the collector supports. The **lineage audit** is the systematic walk that catches ghost fields, lineage bypasses, register-vs.-reality gaps, deletion-coverage gaps, cross-tier-join bypasses, and prediction-log drift. The graph, the register, and the audit *compose with the model card* — every PII claim in the card cites the specific node, register row, or audit report that supports it.
