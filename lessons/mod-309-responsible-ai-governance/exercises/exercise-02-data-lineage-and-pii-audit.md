# exercise-02: Data lineage graph, PII register, and lineage audit

**Estimated effort:** 3 hours

## Objective

Author the **data lineage graph**, the **PII register**, and a **lineage-audit report** for a specific ML deployment. Together the three artefacts are the evidence base under the model card's data-statement claims (chapter 01 §2). The audit produces a findings list keyed to the model card's PII claims; findings flow back into the card (as fixes) or into the review packet (as F0 / F1 / F2 / F3 items).

The output is three linked Markdown documents (`lineage-graph-<feature>.md`, `pii-register-<feature>.md`, `lineage-audit-<feature>-<yyyy-qN>.md`) that a governance analyst can consume as the *provenance layer* of the review packet.

This is the L30 tell that separates "we have some documentation on data flows" from "we can trace any inference-response field back to its ingestion origin and prove what happened to it at every hop." An L20 team's data-flow documentation is a Confluence diagram; an L30 team's is a versioned graph that gates promotion.

## Prerequisites

- Read chapter 02 (`02-data-lineage-and-pii-handling.md`) — the five-node / two-edge graph, the five-tier canonical flow, the four-category PII taxonomy, the five-column PII register, consent / purpose limitation / retention, the three lineage-collection patterns, and the seven-step audit.
- Complete or skim exercise 01 (`exercise-01-model-card-authoring.md`) — the model card whose PII claims this exercise's audit is verifying. You can do this exercise standalone with a stub card, but the artefacts compose best when both are done.
- Skim the [OpenLineage spec](https://openlineage.io/spec/) and a lineage-visualisation tool like [DataHub](https://datahubproject.io/) or [Marquez](https://marquezproject.ai/) — you will not implement lineage collection here, but you should recognise what a runtime-emitted lineage graph looks like.
- Skim [NIST SP 800-188 (De-Identification of Personal Information)](https://csrc.nist.gov/publications/detail/sp/800-188/final) sections 3–4 — the taxonomy for direct identifiers, quasi-identifiers, and the re-identification-risk framework.

## Pick your deployment

Use the same deployment you picked in exercise 01, or pick a fresh one from the D1–D5 list there. Deployments involving PII (D1 with logged-in users, D2 with transaction data, D3 with ticket text and free-form user data) are best for this exercise; D4 (perception on embedded system) is workable but the PII surface will be thinner — the exercise then focuses on the sensor-provenance layer.

## Steps

### 1. Author the lineage graph (≈ 40 min)

Draft the lineage graph for the deployment in Markdown, using the canonical five-tier flow from chapter 02 §2 as the skeleton. For each tier, name:

- **T1 — Sources.** At least four sources. For each: name, owner team, contract (schema, freshness, retention), and jurisdiction.
- **T2 — Normalisation and PII handling.** At least three transformation nodes, including the PII-detection / redaction / hashing node and the consent-scope filter (chapter 02 §5.1). Each named with input / output schema and code reference.
- **T3 — Feature computation.** At least three feature-view materialisation nodes, including any join to reference data. Each named with feature-view URI and version.
- **T4 — Training and serving.** The training pipeline's data ingestion, the model artefact, and the inference-time feature-fetch path.
- **T5 — Sinks.** Every place data leaves the system — inference response, prediction log, monitoring metric, audit log, aggregated report, retention-bound archive.

Draw the graph either as an ASCII diagram, a Mermaid `flowchart`, or an included figure. The graph must be *readable* — a reviewer should be able to trace any T5 field back to its T1 origin without asking questions.

Two properties the graph must have:

- **Every node is versioned.** Reference the specific code that runs the node.
- **Every edge is a specific data flow.** No "some data goes here" — the specific fields that traverse the edge, with schema.

### 2. Author the PII register (≈ 40 min)

Draft the PII register in the five-column shape from chapter 02 §4. For each field that carries PII (or is a candidate for it), fill in:

- **Field.** Fully qualified name (`source.table.column` or `feature_view.feature.name`).
- **Category.** Direct / quasi / sensitive / derived / non-PII, with sub-category (email, DOB, ZIP, race, inferred-age).
- **Provenance.** Which T1 source, which T2 normalisation node, and — if derived — which model or feature-view produced it, from which parents.
- **Transformation policy.** The specific rule at each tier — passed through, hashed with named salt, tokenised, aggregated (with rule), dropped, redacted (with detection rule).
- **Exposure surface.** Every T5 sink the field can appear in, with an allowed-audience label (end user, on-call, security team, external partner) and a retention value.

At least eight register rows. Cover at least one direct identifier, at least two quasi-identifiers, at least one derived attribute, and at least one non-PII field for contrast.

Discipline notes:
- Where a field is *dropped* or *never present*, note the register row explicitly rather than omitting it. Explicit-absent is more auditable than silent-absent.
- If your deployment ingests sensitive-category attributes (GDPR Art. 9 or HIPAA PHI), note the legal basis for processing them (consent, contract performance, vital interests, etc.).
- Cross-reference each register row to the specific graph nodes that implement its transformation policy.

### 3. Consent / purpose / retention pass (≈ 20 min)

For each source at T1 and each sink at T5, note:

- **Consent scope.** What consent the data was collected under. If the deployment cannot use data outside a specific consent scope, name the enforcement node (chapter 02 §5.1 — the consent-scope filter must be an *explicit* node in the graph, not a `WHERE` clause).
- **Purpose limitation.** Is the deployment's use consistent with the original collection purpose? If it is a secondary use, cite the additional consent or legal basis.
- **Retention.** How long is the data at this node retained? Who enforces the retention?

Cross-reference each of these to the model card's section 6 claims. If a model-card claim says "no retention beyond 90 days" but the audit-log sink at T5 retains for 3 years, that is a *finding*.

### 4. Run the seven-step lineage audit (≈ 40 min)

Chapter 02 §7 named the seven audit steps. Run each step against your graph and register:

- **Step 1 — Walk backwards from T5.** For each T5 sink, verify every field traces to a T1 origin through named nodes. Any un-traced field is a *ghost field* finding.
- **Step 2 — Walk forwards from T1 for lineage bypasses.** Any path from T1 to T5 that skips the T2 PII-handling node is a *lineage bypass* finding.
- **Step 3 — Register vs. graph reconciliation.** Register rows without matching graph nodes are *stale register* findings; graph nodes carrying candidate-PII fields not in the register are *silent register* findings.
- **Step 4 — Snapshot lineage for a single inference request.** Pick a sample inference request (invent one if necessary). Reconstruct its snapshot lineage — every feature-view read, model version, prediction log, retention-bound artefact. Any failure to reconstruct is a *snapshot gap* finding.
- **Step 5 — Snapshot lineage for a deleted-user request.** Pick a sample deleted user. Verify no T3 / T5 sink contains their data or that the exception is documented (model-artefact residual risk per chapter 02 §5.3). Any un-explained residual is a *deletion coverage* finding.
- **Step 6 — Cross-tier joins that skip normalisation.** Any T3 feature-view joining directly to a T1 source without T2 traversal is a *bypass* finding.
- **Step 7 — Prediction-log drift.** Compare the prediction log's schema to the sanctioned exposure surface. Any field in the log that is not in the register's exposure column is a *log drift* finding.

Every finding gets a *severity* (P0 blocking, P1 fix before next review, P2 backlog) and an *owner*.

### 5. Author the audit report (≈ 20 min)

Wrap the findings into an audit report — `lineage-audit-<feature>-<yyyy-qN>.md` — with:

- **Cycle window and version.** When this audit was run, what graph and register versions it audited.
- **Scope.** Which sources, tiers, and sinks were in scope; any explicitly out-of-scope areas and why.
- **Coverage-of-collector note.** Which of the three collection patterns (manual, metadata-store scraping, runtime event emission) the graph is built on, and what the *known coverage gaps* of that pattern are for this deployment.
- **Findings.** The seven-step results, each with a specific field / node reference, severity, owner, and close date.
- **Cross-references.** Which model-card claims each finding challenges. Which mod-307 chapter 04 incident-response taxonomy entries the findings hit.
- **Signoff.** ML team owner and (mock) governance-analyst reviewer.

### 6. Model-card reconciliation (≈ 15 min)

For each finding at P0 or P1 severity, name the specific model-card claim it challenges and the required card update. Update the model card (from exercise 01) accordingly — or, if the card is a stub for this exercise, write the change as a diff-style annotation to your stub.

Discipline: no P0 finding leaves this exercise without a card update *or* a decision to hold promotion until the finding closes. F3-style accepted-risk paths (chapter 03 §5.4) exist but require an explicit disclosure to be added to card §9.

### 7. Peer or mock review (≈ 15 min)

Either self-review with the reviewer hat on or trade with a peer. The reviewer walks:

- Does every model-card PII claim have an audit hook (a graph node or register row) that supports it?
- Are the findings *specific* (a particular field, node, or sink) rather than generic ("we should improve lineage")?
- Are severities defensible (P0 blocking is a *deployment-visible* violation, not a "documentation gap")?
- Does the coverage-of-collector note honestly state what the graph does *not* capture?

Bring the review notes back into the audit report and the register.

## Deliverable

Three Markdown documents:

- `lineage-graph-<feature>.md` — the graph and its node / edge inventory, 2–5 pages.
- `pii-register-<feature>.md` — the five-column register with at least 8 rows, 2–3 pages.
- `lineage-audit-<feature>-<yyyy-qN>.md` — the audit report with findings, 2–4 pages.

All three are versioned and dated; all three cross-reference each other and the model card.

## Acceptance criteria

- [ ] The lineage graph names all five tiers, with at least four sources, three T2 nodes, three T3 feature-view nodes, and every T5 sink.
- [ ] The graph is *readable* — a reviewer can trace at least one full path from T1 to T5 without questions.
- [ ] Every graph node cites its code / configuration version.
- [ ] The PII register has at least 8 rows, covers all four categories (direct, quasi, sensitive, derived) plus a non-PII field for contrast.
- [ ] Every register row's transformation policy is a *specific* rule (not "we handle it").
- [ ] Each source at T1 has a consent-scope note; the consent-scope filter is present as an explicit T2 node if the deployment requires one.
- [ ] Retention is named for every T5 sink; the audit-log sink's retention is compared to the model card's retention claim.
- [ ] The seven-step audit produces at least three findings (findings-of-zero is a red flag — either the deployment is unusually mature or the audit is not looking hard enough).
- [ ] At least one finding is a *ghost field*, *lineage bypass*, *log drift*, or *deletion coverage* item — the ones that catch the real gaps.
- [ ] Every P0 or P1 finding names the model-card claim it challenges and the required card update.
- [ ] The coverage-of-collector note honestly states which real-world flows the graph does *not* capture.
- [ ] The audit report is signed (mock signoffs are acceptable for the exercise).

## Stretch goals

- **The runtime-event exemplar.** For one T2 → T3 edge, write out the OpenLineage-style JSON event that would emit at run time when the transformation ran. The event should carry input datasets, output dataset, job facets, and enough provenance to reconstruct the snapshot lineage. Reference the [OpenLineage spec](https://openlineage.io/spec/).
- **Right-to-deletion drill.** Walk a hypothetical deletion request end-to-end. Name every node the deletion touches, every artefact the deletion cannot reach (model artefacts memorising the row per chapter 02 §5.3), and the position the model card takes on those residuals. If the position is "we accept the residual risk," it needs an F3 governance signoff before the card can carry it.
- **Cross-tier query bypass hunt.** Interview (or self-simulate) the team's data analysts. Are any of them running notebook queries that join T1 sources directly into T3 or that read T5 sinks past their sanctioned retention? Every such query is a lineage bypass the graph does not know about. Add findings for each.
- **Compose with the paved-road inventory.** For each T2 / T3 / T4 platform primitive your graph depends on, cross-reference the [mod-308 paved-road inventory](../../mod-308-platform-collaboration/exercises/exercise-01-paved-road-consumption-audit.md). Any primitive with an unpublished SLO (mod-308 chapter 01 §4) is also a lineage-audit risk — if the platform team can silently change a transformation, the graph's audit hooks may go stale.
- **Cross-check against the retraining pipeline.** Every retraining run consumes a slice of the graph. Verify that the retraining pipeline honours the consent-scope filter, the retention policy, and the PII-handling node. If a retraining run pulled data that a subsequent deletion should have removed, that is a finding.
- **Compose with the threat model.** For each finding, note whether it also appears as a threat-model entry from exercise 04 — some lineage gaps (T5 log drift storing sensitive data) are simultaneously privacy findings and abuse-at-scale threats. Cross-reference explicitly.
