# exercise-01: Model card and data statement authoring

**Estimated effort:** 3 hours

## Objective

Author the **full model card and data statement** for a specific ML deployment at governance-analyst altitude — every one of Mitchell's nine sections filled or explicitly marked `n/a`, the Bender-Friedman data statement fields covered in section 6, every claim carrying an audit hook, and the artefact ready to submit as part of a review packet.

The output is a Markdown document (`model-card-<feature>-v<n>.md`) that could go straight into a model's repo, be registered against the model version, and be handed to a governance analyst as the first document in a promotion review packet. A mock reviewer walk (self- or peer-run) closes the loop.

This is the L30 tell that separates "we wrote a model card once" from "we ship a model card with every promotion and defend it in review." An L20 files a template with the fields filled; an L30 files a template with every field *load-bearing*, every claim *sourced*, and every out-of-scope statement *specific*.

## Prerequisites

- Read chapter 01 (`01-model-cards-and-data-statements.md`) — the model card's nine sections, the data-statement schema, the governance-analyst altitude, the template, and the five failure modes.
- Skim chapter 02 (`02-data-lineage-and-pii-handling.md`) — you will not run the full lineage audit here (that is exercise 02), but section 6 of the card claims things the lineage graph would have to verify; leave audit hooks even if the audit is a stub.
- Skim [mod-305 chapter 01](../../mod-305-advanced-evaluation/01-offline-eval-harness-shape.md) and [chapter 02](../../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) — the model-card sections 4 (metrics), 5 (evaluation data), and 7 (quantitative analyses) point at the eval harness's outputs.
- Skim the [Google Model Card for Face Detection](https://modelcards.withgoogle.com/face-detection) — the canonical worked example of the intended-use / out-of-scope pairing.

## Pick your deployment

Pick **one** of the following, or bring your own (D5, with a one-paragraph "context I already know" preamble):

- **D1 — Recommendation ranker on a high-traffic consumer surface.** ~10 M sessions/day. Retrained weekly, product-visible, non-trivial slice matrix (region × device × logged-in state), user-behaviour training data with logged consent scopes.
- **D2 — Fraud-detection model on a transaction path.** ~1 M transactions/day. Retrained monthly, decisions have a review workflow but can auto-hold transactions above a score threshold. Training data blends transactional records, historical chargebacks, and (in some regions) third-party bureau signals.
- **D3 — LLM-augmented ticket triage.** ~5 K tickets/day. Uses a vendor LLM for classification and drafting; a safety classifier for policy violations; retrieval over an internal knowledge base. Retrained (prompt-bundle and knowledge-base refresh) event-driven. Handles PII in ticket text.
- **D4 — Perception model on a safety-critical embedded system.** e.g., automotive lane detection or industrial-inspection defect detection. Trained quarterly, sensor-input based, decisions feed a downstream controller with human-oversight requirements. Regulatory-regime bound (safety-of-the-intended-function standards).
- **D5 — Bring-your-own.** A real or planned deployment you have access to. Anonymise anything sensitive; add a preamble covering scale, deployed decision, data sources, retraining cadence, regulatory context, and current model-card / documentation state.

## Steps

### 1. Scope frame (≈ 15 min)

Before the card: write one paragraph on *who reviews this deployment* and *what regulatory or policy regime it lives under*. NIST AI RMF risk tier, EU AI Act tier, SR 11-7, HIPAA, COPPA, sector-specific, none-external-but-internal-only — name the regime and its consequences for the card's cadence (chapter 01 §7). This paragraph goes in the card's `Last reviewed` and `Next review due` metadata frontmatter.

### 2. Author sections 1–2 (identity and intended use) (≈ 15 min)

- **Section 1 — Model details.** Name, version (semver + git commit + training-run ID or LLM prompt-bundle hash), type / algorithm, owning team, point of contact for card questions, license (both artefact license and training-data license), and registered artefacts (training-run URI, eval-report URI, SLO-doc URI, runbook URI). Use invented URIs; they should look like production URIs even if placeholder.
- **Section 2 — Intended use.** One paragraph of primary intended use in plain language. Named primary intended users. **At least three specific out-of-scope uses**, each with the reason it is out of scope. Human-in-the-loop expectations named explicitly.

Discipline: the section 2 out-of-scope list must contain uses a governance analyst would *worry about* — not "cannot be used for medical diagnosis" if the deployment is a rec-sys. Pick misuses realistic for the domain.

### 3. Author sections 3–5 (factors, metrics, evaluation data) (≈ 30 min)

- **Section 3 — Factors.** Enumerate at least three factor kinds — population / demographic factors, environmental factors, instrumentation factors — plus at least one factor you *chose not to* stratify on, with justification.
- **Section 4 — Metrics.** Name the *primary metric* (single, chosen-in-advance), the guardrail metrics, the uncertainty-quantification method (paired bootstrap CI shape, or seed variance, or holdout replication), the decision threshold, and the fairness metric(s). For the fairness metric — pick one, and explicitly name at least one you did *not* pick and why (chapter 03 §2). No card gets away with "we measure fairness" without naming which fairness.
- **Section 5 — Evaluation data.** Named datasets with URIs and version hashes, motivation for each, preprocessing pipeline reference, slice matrix (from chapter 03 planning or from mod-305 chapter 02), and known gaps — populations you *cannot* evaluate because the eval data does not contain them.

### 4. Author section 6 (training data + Bender-Friedman data statement) (≈ 40 min)

This is the load-bearing section for governance. Every Bender-Friedman field:

- **Curation rationale.** Why this data. What was excluded and on what basis.
- **Sources.** Specific dataset names, URIs, and date ranges.
- **Language / domain variety.** BCP-47 tags for text; equivalent taxonomy for other modalities (image resolution regimes, sensor modalities, geographic distribution).
- **Producer demographics.** Who produced the data — or *not collected* if not collected. Never *presumed representative*.
- **Annotator demographics** (if labelled data). Who labelled, using what schema, with what inter-annotator agreement.
- **Speech / capture situation.** The context of production.
- **Recording / capture quality.** Fidelity properties; excluded-quality thresholds.
- **PII handling.** What PII was in the raw source, what the scrubbing pipeline does, what remains. Cross-reference to the PII register (exercise 02).
- **Consent and license.** What the users / subjects consented to.
- **Retention.** How long the training data is retained and where; the deletion policy; the position on model-artefact memorisation risk (chapter 02 §5.3).

Discipline: at least three "not collected" or "not applicable" entries are expected honestly. If every field is filled with a confident answer, either the deployment is genuinely well-documented or the card is asserting where it should be admitting.

### 5. Author sections 7–9 (quantitative analyses, ethics, caveats) (≈ 30 min)

- **Section 7 — Quantitative analyses.** Unitary results table (metric × factor), intersectional results table for at least the two factor pairs regulation names, threshold sweeps, and ablations relevant to fairness / robustness. CIs on every number. Point at the specific eval-harness run that produced them.
- **Section 8 — Ethical considerations.** Sensitive data used, human-life implications, mitigations (HITL, refusal, threshold choice, review cadence), known risks and harms (from the chapter 04 threat model — even if that's a stub for this exercise), use cases you advise against.
- **Section 9 — Caveats and recommendations.** Generalisation warnings, additional testing you recommend but did not do, infrastructural caveats (cite the paved-road inventory from mod-308 if in scope), sunset / re-review triggers (chapter 01 §7's three axes).

### 6. Walk the five failure modes on your own draft (≈ 15 min)

Chapter 01 §5 named the five failure modes. Walk your draft against each and note any hit:

- **The marketing card.** Do sections 1–2 read like a product page? Prune.
- **The single-number card.** Does section 7 report one metric per row? Add slice stratification and CIs.
- **The confident-about-empty card.** Are there "diverse and representative" claims in section 6 without sources? Replace with "not collected" or specific evidence.
- **The stale card.** Are dates and reviewer names filled in? Frontmatter metadata (`Last reviewed`, `Next review due`) present?
- **The self-inconsistent card.** Do intended use (§2), quantitative analyses (§7), and ethical considerations (§8) tell the same story? Any contradictions?

Every hit is a change to the card *before* review, not after.

### 7. Mock governance review (≈ 15 min)

Either self-review with the reviewer hat on, or trade with a peer. The reviewer walks with three questions from chapter 01 §3:

- **Is this deployment scope-legal against the policies I am accountable for?** (Name the regime — GDPR, HIPAA, EU AI Act, SR 11-7, internal AI-use policy — and check whether the card supports the answer.)
- **Is the artefact self-consistent?**
- **Is there evidence for every claim that could be a policy hook?**

The reviewer produces findings on the F0 / F1 / F2 / F3 ladder from chapter 03 §5.4. Bring the review notes back into the card as either fixes or acknowledged open items.

## Deliverable

One Markdown document — `model-card-<feature>-v<n>.md`, following the chapter 01 §4 template, 4–8 pages — plus a short review-note appendix that lists what the mock reviewer said and how you handled each note.

## Acceptance criteria

- [ ] Frontmatter metadata block filled: card version, model version, card owner, last reviewed (with date), next review due (with date).
- [ ] Section 2 (intended use) lists at least three specific out-of-scope uses, each with a reason.
- [ ] Section 3 names at least one factor you *chose not to* stratify on, with justification.
- [ ] Section 4 names the primary metric (single, chosen-in-advance), the guardrail set, the CI shape, and the fairness metric picked *and* one fairness metric not picked with reason.
- [ ] Section 6 covers every Bender-Friedman field, with at least three honestly-noted "not collected" or "not applicable" entries.
- [ ] Section 6 cross-references the PII register (even if exercise 02's register is a stub).
- [ ] Section 7 reports at least one intersectional result (not just unitary), with CIs.
- [ ] Section 8 names at least three specific "advise against" use cases with reasons.
- [ ] Section 9 names the sunset / re-review triggers on all three axes (model-triggered, time-triggered, event-triggered).
- [ ] Every "claim" in the card carries an audit hook — a link to the eval-run URI, the lineage graph node, the PII register entry, the release memo, or the threat-model artefact.
- [ ] The five-failure-mode walk (step 6) is documented — either "none hit" with the walk noted, or the hits and their fixes named.
- [ ] The mock-review appendix lists at least one finding at each of F0 / F1 / F2 / F3 (or an explicit "no F0 findings" with defence).

## Stretch goals

- **The vendor-vs.-in-house model boundary.** If your deployment uses a vendor foundation model (D3), author a section 1 sub-section on which artefacts you own, which the vendor owns, and how the responsibility for cards / statements is divided. The [Partnership on AI documentation guidance](https://partnershiponai.org/) frames this well; internal cards for vendor components are a growing pattern.
- **Compose with an ML Test Score review.** If you did [mod-305 exercise 03](../../mod-305-advanced-evaluation/exercises/exercise-03-ml-test-score-review-packet.md), fold that packet's findings into the model card's §7 and §9. The two artefacts should tell the same story.
- **Cross-model card composition.** For an LLM-augmented deployment with multiple models in a pipeline (retriever + reranker + generator + safety classifier), author a *composed* card that summarises each component and points at the individual cards. Chapter 04 §3's cross-model composition misuse is what this section defends against.
- **Draft the registry-gate PR.** Author the change (in text, no need to implement) that would make the model registry refuse to promote a version without a matching model card whose "Next review due" date is in the future. Reference [mod-308 chapter 01 §3.3](../../mod-308-platform-collaboration/01-paved-road-consumption.md).
- **Feed the paired review packet.** Attach the card as document 1 of the review packet the chapter 03 exercise and the chapter 04 exercise both feed. The packet is the artefact governance signs; this exercise produces its first component.
