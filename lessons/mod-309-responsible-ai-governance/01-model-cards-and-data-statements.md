# Model cards and data statements a governance analyst can act on

## Motivation

At L20, the release memo lives in a Slack thread and a Confluence page called `<model-name>-notes`. Some fields are filled, some are empty, some are contradicted by the training script. If a governance analyst — someone whose job is to sign off that the deployment is defensible against internal policy and external regulation — asks "what data was this trained on and who is excluded from the evaluation," the answer takes a week.

At L30, the release ships with a **model card** and a **data statement** as first-class artefacts. Both live in the repo next to the code. Both are versioned with the model. Both are structured to a template a governance analyst has seen before. Both name the *decisions* — what data was included, what was excluded, what evaluation slices were run, what the model is *not* fit for — in a form the analyst can quote in a compliance packet without re-writing.

The load-bearing shift is that the model card is not documentation *of* the model — it is a *decision artefact* between the ML team and the reviewers (governance, legal, compliance, risk, product policy). The senior ML engineer writes it *to* those reviewers, at their altitude, so the review is a signature and not a fact-finding expedition.

Two primary references anchor this chapter:

- **Mitchell et al., 2019 — [*Model Cards for Model Reporting*](https://arxiv.org/abs/1810.03993).** The paper that named the artefact and its nine sections. Every subsequent template (Google's [Model Card Toolkit](https://github.com/tensorflow/model-card-toolkit), Hugging Face's [Model Cards](https://huggingface.co/docs/hub/model-cards), the NIST AI RMF's `Documentation` profile) is a derivative of this shape.
- **Bender & Friedman, 2018 — [*Data Statements for Natural Language Processing*](https://aclanthology.org/Q18-1041/).** The paired paper for the *data* side of the artefact — the statement of who / what / how the training data represents. The datasheet-for-datasets literature ([Gebru et al., 2018](https://arxiv.org/abs/1803.09010)) is the general-purpose sibling that covers non-NLP data.

This chapter installs the template, the discipline for filling it in *honestly* (empty is better than wrong), and the governance-analyst hand-off. Chapter 02 traces the data lineage that a data-statement claim ("no PII in training") has to be *provable* from. Chapter 03 governs the evaluation slices the model card reports on. Chapter 04 threat-models the misuse patterns the model card names in its *not-fit-for* section.

## §1 — What a model card actually is

Mitchell et al.'s original nine sections, and what each one commits the team to:

1. **Model details.** Name, version, date, type, algorithm, license, owning team, contact. The identity layer — so a governance analyst reading a card three years later can identify the exact artefact they are looking at.
2. **Intended use.** Primary use cases in plain language, primary intended users, out-of-scope uses. This is the section that lets a reviewer say "this model is being used in a way its authors said it should not be."
3. **Factors.** The demographic, phenotypic, environmental, instrumentation, and situational factors *relevant to the model's behaviour* — the ones the evaluation is stratified against.
4. **Metrics.** The performance measures used, decision thresholds, and *variation approaches* (bootstrap CIs, cross-validation, or seed variance) that quantify uncertainty. A single-number metric is a red flag on the card.
5. **Evaluation data.** Datasets used for evaluation, motivation for selection, and any preprocessing.
6. **Training data.** Same as evaluation data, but for training. Bender-Friedman's data statement is the specific expanded form of this section for NLP; datasheets-for-datasets is the general form.
7. **Quantitative analyses.** Unitary results (per-metric, per-factor) *and* intersectional results (per-metric, per-factor-combination). This is the section fairness reviewers open first.
8. **Ethical considerations.** Sensitive data used, human life implications, mitigations attempted, risks and harms, use cases the authors advise against.
9. **Caveats and recommendations.** Everything else — additional testing that should have been done and was not, generalisation warnings, environmental / infrastructural caveats.

Sections 1–2 are the *identity and scope* layer. Sections 3–5 are the *what was tested and on what data* layer. Sections 6–7 are the *training data provenance and stratified results* layer. Sections 8–9 are the *what a reviewer needs in order to sign off* layer. A model card missing any of the four layers is a card that will fail its first review.

Two clarifications worth naming up front:

- **A model card is not a marketing artefact.** It is written to reviewers, not to users of the product surface. The tone is closer to a lab-notebook entry than to a press release. If the card reads like the model page on a vendor website, it is under-written.
- **A model card is not a static document.** Every retraining, every prompt-bundle change, every fine-tune produces a new *version* of the card. The card is versioned with the model artefact in the registry (mod-308 chapter 01 §1) and cross-references the specific evaluation harness run (mod-305 chapter 01) that produced its numbers.

## §2 — The data statement (Bender-Friedman) as the training-data layer

Mitchell's section 6 asks for "training data." Bender & Friedman's [data statement schema](https://aclanthology.org/Q18-1041/) is the *expanded* answer to that question — the artefact that lets a reviewer decide whether the training data actually supports the model's intended use.

The schema is targeted at NLP but the questions transfer to any modality. The sections:

- **Curation rationale.** Why was this dataset chosen? What alternatives were considered? What was excluded, and on what basis? "We used the data we had" is not a curation rationale.
- **Language variety.** For NLP, the specific language variety (BCP-47 tag) and dialectal features. For non-NLP, the *domain* variety — image resolution regimes, sensor modalities, geographic distribution of the sources.
- **Speaker / producer demographics.** For NLP, who wrote the text (age, gender, race/ethnicity, native language, socioeconomic status, disability status, where the demographics can be inferred or were reported). For non-NLP, the equivalent — who is represented in the images, who operated the sensor, whose behaviour produced the log. Missing demographics is *itself* a data-statement claim: "we did not collect this and cannot report it" is honest; silence is not.
- **Annotator demographics.** Who labelled the data, using what schema, with what agreement statistics. Annotator population shift is one of the largest sources of hidden dataset bias (Gordon et al., [*Jury Learning*](https://arxiv.org/abs/2202.02950), makes the point sharply).
- **Speech situation.** The *context* the data was produced in — was the text a formal document, a chat message, a review, a support-ticket transcript? Was the image a smartphone selfie, a professional stock photo, a medical scan? Context transfer failures are one of the classic silent evaluation gaps.
- **Text characteristics.** For NLP, structural features — genre, register, topic, whether the text is edited or raw. For non-NLP, the equivalent — capture-time metadata, transformation history, resampling.
- **Recording quality.** For sensor data, the fidelity — bit depth, sampling rate, compression regime. Failing quality-control on part of the corpus is a data statement.
- **Other.** Everything the schema does not name that a reviewer would ask about — consent, license, distribution, retention, deprecation date.

The senior ML engineer's discipline on this section is **empty-is-better-than-wrong**. If the training data's speaker demographics were not collected, the data statement says "not collected." It does *not* say "presumed representative." Presumption is where governance failures land.

## §3 — The governance-analyst altitude: writing for the reviewer

Yonatan Zunger's essay [*The Friendly Design Doc*](https://medium.com/@yonatanzunger/the-friendly-design-doc-2c96b8ba0842) has a load-bearing insight: **you write to the person who reads first, at their altitude, in their vocabulary**. mod-308 chapter 03 §4 named this discipline for the platform-team-altitude RFC; here the same discipline applies to the governance-analyst altitude.

The governance analyst is not the ML team. They are looking at three questions on the first pass:

1. **Is this deployment scope-legal against the policies I am accountable for?** (The internal AI-use policy, the EU AI Act obligations for this risk classification, the sector-specific compliance regime — SR 11-7 for banks, HIPAA for health, COPPA for children's products, GDPR for EU users, industry-specific rules like FCRA for consumer credit.)
2. **Is the artefact self-consistent?** Do the *intended-use* claims support the *evaluation slice* choices? Does the *training data* section allow the *deployed population* implied by intended use? Does the *quantitative analysis* section report on the sensitive factors named in the *ethical considerations* section?
3. **Is there evidence for every claim that could be a policy hook?** "The model is fair across gender" is a policy hook — it must cite the specific slice results, on the specific evaluation set, with the specific fairness metric.

The senior ML engineer's card is written to make questions 1–3 answerable from the card alone. The card is *self-contained enough* that the analyst does not need to open the training script to verify a claim. If verification requires the training script, the card cites the specific commit hash, file, and line number.

Concretely, this means:

- **Every claim has an audit hook.** "The model is trained on public web data" → cite the specific crawler manifest, the date range, the exclusion list. "The evaluation set contains no personally identifying information" → cite the PII-scan pipeline in the lineage graph (chapter 02) that produced the assertion.
- **Every quantitative result names its interval.** "82 % accuracy" is not a claim; "82 % accuracy [79.5, 84.3] on the held-out test set of 12,438 examples, primary metric declared 2026-05-14" is a claim. (mod-305 chapter 01's paired-bootstrap CI discipline lives here.)
- **Every intended-use statement names an out-of-scope counterpart.** The [Google Model Card for Face Detection](https://modelcards.withgoogle.com/face-detection) is the canonical example — "intended for face detection" is paired with "not intended for face recognition, biometric identification, or emotion classification."
- **Every populated section has a "last updated" date and a person responsible.** Section-level provenance so a stale claim can be traced to who was on the hook for keeping it fresh.

## §4 — The template you actually ship

Below is a minimum viable model card and data statement, in the shape that combines the Mitchell nine sections and the Bender-Friedman data statement into a single artefact. This is the file that lives at `docs/model-card-v<n>.md` in the model's repo and gets registered as an artefact against the model version in the registry.

```markdown
# Model card — <feature-name> — v<version>

**Card version.** <n>
**Model version.** <registry URI>
**Card owner.** <name, role>
**Last reviewed.** <date, by whom>
**Next review due.** <date>

## 1. Model details
- **Name.** <feature-name>
- **Version.** <semver + git commit + training-run ID>
- **Type / algorithm.** <e.g., gradient-boosted decision tree; fine-tuned Llama-3-8B; two-tower retrieval; retrieval-augmented generation with judge>
- **Owning team.** <team>
- **Point of contact for card questions.** <name, role, on-call schedule>
- **License.** <license for model artefact and for training data>
- **Registered artefacts.** Training run <URI>, eval report <URI>, SLO doc <URI>, runbook <URI>

## 2. Intended use
- **Primary intended use.** <one paragraph, plain language>
- **Primary intended users.** <who calls the model, in which product surface>
- **Out-of-scope uses.** <specific — not "everything else"; the two or three misuse patterns most reviewers ask about>
- **Human-in-the-loop expectations.** <what actions a human must take before a model output produces a consequence>

## 3. Factors relevant to model behaviour
- **Population factors.** <the demographic, phenotypic, or user-role factors the evaluation stratifies on and why>
- **Environmental factors.** <the deployment-context factors that shift behaviour — device type, network, region, time-of-day>
- **Instrumentation factors.** <the pipeline factors — feature-fetch freshness regime, upstream service availability>
- **Factors we chose NOT to stratify on and why.** <every exclusion is a defensible decision; name it>

## 4. Metrics and evaluation
- **Primary metric.** <chosen in advance; mod-305 chapter 01>
- **Guardrail metrics.** <the ones a regression on any of which blocks release; mod-305 chapter 02>
- **Uncertainty quantification.** <bootstrap CI shape, or seed variance, or holdout replication>
- **Decision threshold.** <the operating point and how it was chosen>
- **Fairness metrics.** <the specific metrics — demographic parity, equal opportunity, calibration by group; see chapter 03 §2 for the choice>
- **Evaluation harness reference.** <link to mod-305 harness config commit>

## 5. Evaluation data
- **Datasets.** <named eval sets with URIs and version hashes>
- **Motivation.** <why these datasets — what population and behaviour they are meant to probe>
- **Preprocessing.** <the exact transformation pipeline, versioned>
- **Slice matrix.** <the slice list from chapter 03 with per-slice size and coverage>
- **Known gaps.** <slices we cannot evaluate because the eval data does not contain the population, and what we do about it>

## 6. Training data (with the Bender-Friedman data statement)
- **Curation rationale.** <why this data — what was included, what was excluded, on what basis>
- **Sources.** <specific datasets, crawls, or logs with URIs and date ranges>
- **Language / domain variety.** <BCP-47 tags for NLP; equivalent for other modalities>
- **Producer demographics.** <who produced the data, or "not collected" — never "presumed representative">
- **Annotator demographics.** <if labelled data — who labelled, using what schema, with what inter-annotator agreement>
- **Speech / capture situation.** <the context the data was produced in>
- **Recording / capture quality.** <fidelity properties; excluded-quality thresholds>
- **PII handling.** <what PII was in the raw source, what the scrubbing pipeline does, what remains — see chapter 02>
- **Consent and license.** <what the users / subjects / data sources consented to and under what license>
- **Retention.** <how long the training data is retained and where — the deletion policy>

## 7. Quantitative analyses
- **Unitary results.** <table: metric × factor, with CIs>
- **Intersectional results.** <table: metric × (factor₁, factor₂), for at least the two factor pairs regulation names>
- **Threshold sweeps.** <how the metric moves as the operating point shifts>
- **Ablations relevant to fairness / robustness.** <e.g., performance with vs. without a sensitive feature; robustness to distribution shift>

## 8. Ethical considerations
- **Sensitive data used.** <what and why>
- **Human life implications.** <what a wrong prediction can cause; the "consequence taxonomy" from chapter 03>
- **Mitigations.** <the specific ones — HITL, refusal, threshold choice, review cadence>
- **Known risks and harms.** <the specific ones the threat model of chapter 04 named>
- **Use cases we advise against.** <specific, with the reason>

## 9. Caveats and recommendations
- **Generalisation warnings.** <populations and contexts the training data does not cover>
- **Additional testing we recommend.** <the ones we did not do and think should happen>
- **Infrastructural caveats.** <the platform primitives and dependencies whose failure changes behaviour — cite the paved-road inventory from mod-308>
- **Sunset / re-review triggers.** <the events that force a re-authoring — new data source, new region, new regulation, model-family change>
```

Two properties of this template that matter:

- **The template is opinionated but not rigid.** Every section can be short. Empty sections are marked `n/a` with a one-line justification, never left blank. A card whose section 3 is empty because "we don't know the factors that matter" is a card that failed its own audit — that emptiness is a chapter-03 exercise waiting to happen.
- **The template composes with the registry.** The card lives at a well-known path in the model repo (`docs/model-card-v<n>.md`), is registered as an artefact against the model version (mod-308 chapter 01 §3.3 — standard artefact registration), and the registry's promotion gate refuses to promote a version whose card is missing or whose "Last reviewed" date is older than the card's own re-review cadence.

## §5 — Common failure modes and how to catch them

Five failure modes recur across model-card first drafts. The senior discipline is to notice them in your own drafts before the governance analyst does.

- **The marketing card.** The intended-use section reads like the vendor product page: "our model reliably produces high-quality outputs." No claims, no numbers, no bounds. The fix: rewrite section 2 to name the two or three *specific* decisions the model is supposed to enable, and pair each with an out-of-scope decision the model must not be used for.
- **The single-number card.** The quantitative-analysis section reports one metric: "84 % accuracy." No slices, no CIs, no factor stratification. The fix: mod-305 chapter 02's slice matrix and mod-305 chapter 01's paired-CI discipline are what section 7 has to reflect.
- **The confident-about-empty card.** The training-data section claims "our data is diverse and representative." No BCP-47 tags, no dataset URIs, no demographic breakdown. The fix: replace every unsupported claim with the specific source, and where the data does not support the claim, say "not collected" and note that as a caveat in section 9.
- **The stale card.** The card is from the model's launch six months ago. The model has been retrained three times. Nobody has updated the card. The fix: registry gates that refuse promotion when the card is missing or older than the cadence policy the team publishes. Cadence is the fifth-cell discipline of chapter 03.
- **The self-inconsistent card.** Section 2 says "not intended for high-risk decisions." Section 8 lists a "sensitive-decision" mitigation. Somewhere in the deployment, sensitive decisions *are* being made. The fix: internal cross-check by an owner other than the section's author before the card is submitted for review.

The card is a claim you are signing. The senior discipline is that any claim you would not defend against a hostile-but-fair line of questioning from the reviewer is a claim that comes out of the card *before* submission.

## §6 — The governance-analyst hand-off

The card and the data statement are not artefacts you file and forget — they are artefacts you *hand off*. Three pieces of the hand-off:

- **The pre-review conversation.** Before the formal review, the ML team walks the card with the governance analyst — often the day the model enters the promotion path (mod-305 chapter 03). The conversation catches the questions the analyst was about to ask, and lets the team fix or acknowledge them before formal review. This is the same discipline as mod-308 chapter 03's *sponsor conversation*, adapted to the governance-review context.
- **The review packet.** The governance review does not consume the model card alone. It consumes a *packet*:
  - The model card (this chapter).
  - The lineage / PII audit summary (chapter 02).
  - The fairness / robustness slice results (chapter 03).
  - The misuse and adversarial threat model (chapter 04).
  - The ML Test Score review from [mod-305 chapter 04](../mod-305-advanced-evaluation/04-ml-test-score-production-readiness.md).
  - The SLO document from [mod-307 chapter 01 §6](../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md).
  - The release memo pointer.

  The packet is the artefact the analyst signs off on. The card is one of six or seven documents; the discipline is that they all *cross-reference* each other, not that any one covers everything.

- **The post-review follow-through.** Every review produces action items. Some are "add this to the card." Some are "delete this from the card because we cannot defend it." Some are "we cannot promote this version until <finding> is addressed." The senior discipline is that the review is *actionable* — an action-item list survives out of every review, with owners and dates, and the review-signoff is contingent on those items closing on schedule.

The governance analyst's own track (`ai-governance-analyst-learning`) has depth on the compliance-side vocabulary. The hand-off contract discipline of [mod-308 chapter 04](../mod-308-platform-collaboration/04-handoff-contracts-to-specialists.md) transfers here: the senior ML engineer's job is to write the artefact and drive the review, not to *be* the governance analyst. Where the review needs regulation-specific depth (EU AI Act risk classification, SR 11-7 model risk management, HIPAA de-identification) the governance analyst is the one bringing it.

## §7 — Cadence: the versioned, re-reviewed card

The card is versioned with the model, but it is *also* versioned with the *policy* environment. A new regulation, a new internal AI-use policy, a new incident that changed the deployment's risk posture — any of these can force a card re-authoring even if the model has not changed.

The cadence policy the team publishes has three axes:

- **Model-triggered.** Every retraining that produces a promoted version produces a new card. mod-307 chapter 05 named retraining as a first-class deploy; the card is one of the artefacts the deploy produces.
- **Time-triggered.** Even if the model has not been retrained, the card is re-reviewed on a fixed cadence — typically 6 months for standard-risk deployments, 3 months for elevated-risk, and out-of-cycle for high-risk. The [NIST AI Risk Management Framework's *Manage* function](https://www.nist.gov/itl/ai-risk-management-framework) is the standard reference for the categorisation.
- **Event-triggered.** New regulation in scope, new incident in the deployment's incident-response taxonomy (mod-307 chapter 04) that hits a policy hook, new sensitive population in the traffic, or a new deployment region that changes the applicable law.

The team publishes the cadence policy *in* the card (section 9's sunset / re-review triggers). A registry gate enforces the model-triggered axis (no promotion without a matching card version); a Grafana / dashboard SLI enforces the time-triggered axis (card freshness as an ML-specific SLI on top of the ones mod-307 chapter 02 enumerated); the event-triggered axis is the incident-response process's responsibility to hand off to card ownership.

## Summary

The model card is a decision artefact between the ML team and the reviewers — not marketing, not developer documentation. Its nine Mitchell-et-al. sections (identity, intended use, factors, metrics, evaluation data, training data, quantitative analyses, ethical considerations, caveats) are the load-bearing structure. Bender-Friedman's data statement is the expanded form of the training-data section — curation rationale, language / domain variety, producer and annotator demographics, capture situation, PII handling, consent, retention. Writing the card *at the governance analyst's altitude* means every claim has an audit hook, every quantitative result names its interval, every intended-use statement names an out-of-scope counterpart, and every populated section is dated and owned. The five failure modes to catch in your own drafts are the marketing card, the single-number card, the confident-about-empty card, the stale card, and the self-inconsistent card — each has a specific fix. The card ships as part of a **review packet** with the lineage audit (chapter 02), fairness / robustness slice results (chapter 03), threat model (chapter 04), ML Test Score review, SLO document, and release memo — the packet is the artefact the analyst signs. The cadence policy is model-triggered, time-triggered, and event-triggered, published *in* the card, and enforced by the registry's promotion gate.
