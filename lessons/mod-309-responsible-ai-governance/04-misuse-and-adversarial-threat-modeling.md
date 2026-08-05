# Threat-modelling misuse and adversarial patterns for a deployed ML system

## Motivation

A classical software threat model asks "what can an attacker do to this system," enumerates the STRIDE categories (spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege), and produces a list of mitigations. An ML threat model asks the same question, but the *attack surface* is different: the attacker can influence the *training data*, the *inputs at inference*, the *model outputs*, and — for LLM-augmented systems — the *prompts, tools, and retrieval sources* the model composes with. And beyond intentional attackers, an ML system has a *misuse* surface: users using the model for a purpose the deployment did not sanction, downstream teams building on the model in ways the model card does not endorse, and *pattern of use* that produces harm without any single call being malicious.

The senior discipline is to threat-model **both** the adversarial surface (intentional attackers) and the misuse surface (unintended-but-foreseeable use), pick the attacks and misuse patterns *realistic for this domain*, and land the mitigations in the deployed system's controls — evaluation slices (chapter 03), monitoring SLIs (mod-307 chapter 02), incident-response taxonomy (mod-307 chapter 04), model-card out-of-scope statements (chapter 01 §2), and where warranted, hand-off to the ML/AI security peer track for depth.

This chapter installs the ML-specific threat-model vocabulary, the two canonical taxonomies (MITRE ATLAS for adversarial, OWASP LLM Top 10 for LLM-augmented systems), the misuse-vs.-adversarial distinction, and the composition of the threat model with the rest of the review packet.

Primary references:

- **[MITRE ATLAS](https://atlas.mitre.org/)** — Adversarial Threat Landscape for Artificial-Intelligence Systems. The most-current taxonomy of adversarial techniques against ML systems, with case studies. The canonical starting point.
- **[OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)** — the LLM-specific hazard list every LLM-augmented deployment cross-references.
- **[NIST AI 100-2e2023 — Adversarial Machine Learning Taxonomy](https://csrc.nist.gov/publications/detail/ai/100-2e2023/final)** — the taxonomy and terminology reference for evasion, poisoning, privacy, and abuse attacks.
- **Kumar et al. (Microsoft), 2019 — [*Failure Modes in Machine Learning*](https://learn.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning)** — the canonical intentional-vs.-unintentional failure-mode taxonomy.
- **Adam Shostack, [*Threat Modeling: Designing for Security*](https://shostack.org/books/threat-modeling-book)** (Wiley, 2014) — the general threat-modelling reference the ML-specific version composes with.

## §1 — The four-question frame

Shostack's four-question threat-model frame — *What are we building? What can go wrong? What are we doing about it? Did we do a good job?* — transfers directly to ML with one addition. Because ML failures often are not *bugs* but *emergent behaviour of the model*, we add: *Which of these mitigations do we test, and how do we know the test still catches them next quarter?*

The five-question ML frame:

1. **What are we building?** — the deployment description, including *whom* it serves, *what decision* it enables, and *what data* it consumes and produces. Chapter 01's model card §1–§2 and chapter 02's lineage graph are the answers.
2. **What can go wrong — adversarially?** — the intentional-attack surface. §2 below.
3. **What can go wrong — through misuse?** — the unintentional-harm surface. §3 below.
4. **What are we doing about it?** — the mitigation set, mapped to the deployment's controls (evaluation, monitoring, response, hand-off). §5 below.
5. **How do we know the mitigation still works?** — the test that anchors each mitigation to a repeatable check. Chapter 03's periodic-review layer is where these tests live. §6 below.

The five-question frame is the *skeleton* of a threat model. The rest of the chapter fills in the vocabulary that lets a senior ML engineer answer question 2 and 3 concretely for their domain.

## §2 — The adversarial surface: attacks against ML systems

The adversarial ML literature has converged on four load-bearing attack categories. NIST AI 100-2e2023's taxonomy is the reference; MITRE ATLAS's tactic-and-technique catalogue is the operational instantiation.

### 2.1 — Evasion (test-time attacks)

The attacker manipulates the *input at inference* to produce a wrong output. This is the space [Goodfellow et al., 2014](https://arxiv.org/abs/1412.6572) opened with adversarial-example gradients on ImageNet classifiers; the space [Carlini & Wagner, 2016](https://arxiv.org/abs/1608.04644) extended with strong optimisation attacks; the space that today includes physical-world adversarial patches (Athalye et al., 2018 [*Synthesizing Robust Adversarial Examples*](https://arxiv.org/abs/1707.07397)) and adversarial suffixes for LLMs (Zou et al., 2023 [*Universal and Transferable Adversarial Attacks*](https://arxiv.org/abs/2307.15043)).

Domain relevance:
- **Content moderation and spam / abuse classifiers.** Adversaries paraphrase, obfuscate, or use homoglyph substitutions to evade a classifier that flags policy-violating content.
- **Fraud detection.** Adversaries stage transactions to appear benign to a scoring model.
- **Perception in safety-critical systems.** Adversarial patches on stop signs or lane markings (Eykholt et al., 2018 [*Robust Physical-World Attacks*](https://arxiv.org/abs/1707.08945)).
- **LLM-augmented systems.** Prompt suffixes crafted to bypass safety training; jailbreaking prompts.

Mitigation family: adversarial training (Madry et al., 2018 [*Towards Deep Learning Models Resistant to Adversarial Attacks*](https://arxiv.org/abs/1706.06083)), input preprocessing, ensemble defence, run-time input-anomaly detection, rate-limiting on the model surface, safety-classifier layer over an LLM.

### 2.2 — Data poisoning (train-time attacks)

The attacker manipulates the *training data* — either the labels, the features, or both — so that the trained model behaves incorrectly on specific inputs. [Biggio et al., 2012](https://arxiv.org/abs/1206.6389) opened the space with SVM label-flipping; the [BadNets backdoor family](https://arxiv.org/abs/1708.06733) established that a trigger pattern can be planted at train time and later activated at inference. [Carlini et al., 2023 *Poisoning Web-Scale Training Datasets*](https://arxiv.org/abs/2302.10149) showed practical poisoning attacks against internet-scale corpora.

Domain relevance:
- **Recommender systems.** Coordinated fake-user activity to promote or demote items (shilling attacks, [Lam & Riedl, 2004](https://dl.acm.org/doi/10.1145/988672.988726)).
- **Search relevance.** Coordinated content injection to influence downstream ranking or LLM retrieval.
- **Human-labelled corpora.** A subset of annotators intentionally mislabels; the model learns the mislabelled pattern.
- **RLHF and preference-tuning.** Poisoned preference data can steer LLM behaviour on specific triggers (Rando & Tramèr, 2023 [*Universal Jailbreak Backdoors from Poisoned Human Feedback*](https://arxiv.org/abs/2311.14455)).

Mitigation family: data-source authentication (only trusted sources contribute to the training set), lineage-based provenance (chapter 02 §7 gaps are also poisoning-surface findings), anomaly detection on training-data statistics, label-noise robustness training, and post-training backdoor scanning.

### 2.3 — Model extraction and membership inference (privacy attacks)

The attacker interacts with the model at inference and reconstructs either the model itself, the training data, or membership of a specific record in the training set.

- **Model extraction / stealing.** [Tramèr et al., 2016](https://arxiv.org/abs/1609.02943) showed that with black-box query access, an attacker can reconstruct an approximate copy of a target model. Modern LLM APIs face a version of this with *distillation attacks*.
- **Membership inference.** [Shokri et al., 2017](https://arxiv.org/abs/1610.05820) showed that overfit models leak whether a specific record was in their training set, with implications for privacy where training-set membership is itself sensitive.
- **Training data extraction.** [Carlini et al., 2020](https://arxiv.org/abs/2012.07805) and [Carlini et al., 2022](https://arxiv.org/abs/2202.07646) showed LLMs memorise and can be prompted to emit training-data verbatim, including PII.

Domain relevance:
- **Any deployment where the model itself is a business asset.** Extraction is theft of an artefact.
- **Any deployment trained on sensitive data.** Membership inference and extraction are privacy incidents even without a data breach in the classical sense.
- **LLM deployments generally.** Verbatim-memorisation risk is *default-on* unless deliberately mitigated.

Mitigation family: differential privacy (Abadi et al., 2016 [*Deep Learning with Differential Privacy*](https://arxiv.org/abs/1607.00133)), query rate-limiting, output post-processing (e.g., top-k restriction to hide the full logit distribution), PII scanning on outputs, prompt-level system rules that refuse to emit training data.

### 2.4 — Abuse of the model as a *tool* (integrity of use)

The attacker uses the model *legitimately* — no evasion, no poisoning, no extraction — but at *scale* or with *coordination* to produce harm. Automated influence operations that use LLMs to generate content, coordinated abuse of a translation model to produce plausible-but-wrong content in bulk, or use of a code-generation model to produce malware are examples. Weidinger et al. 2021 [*Ethical and social risks of harm from Language Models*](https://arxiv.org/abs/2112.04359) enumerates the LLM-specific abuse surface; the [OpenAI + Georgetown CSET 2023 threat report](https://openai.com/research/forecasting-misuse) is a concrete industry treatment of the misuse-at-scale category.

Mitigation family: rate-limiting, identity binding, use-case attestation, watermarking (Kirchenbauer et al., 2023 [*A Watermark for Large Language Models*](https://arxiv.org/abs/2301.10226)), pattern detection on aggregate usage.

## §3 — The misuse surface: unintended-but-foreseeable use

The misuse surface is what distinguishes the ML threat model from a classical security threat model most sharply. In classical security, a benign user is not a threat; in ML, a benign user using the model *outside its intended scope* is a first-class hazard the deployment has to consider.

Five common misuse patterns, in the shape a senior ML engineer will actually see:

- **Off-label use in production.** Another team wires the model into a decision the model card did not sanction (a fraud-scoring model used as a signal in credit-underwriting; a text-classification model used for legally-consequential decisions). The model card's out-of-scope statement (chapter 01 §2) is the mitigation *if the downstream team reads it*; the operational reinforcement is API authentication that pins the intended-use scope and rate-limits by scope, plus a paved-road integration pattern that surfaces the model card at every consumer.
- **Automation without oversight.** The deployment's intended-use assumes a human-in-the-loop; a downstream team removes the human. The mitigation is *technical* — an API contract that requires a decision-authority token from a human-in-the-loop system — not documentary.
- **Feedback-loop harm.** The model's outputs shape future training data. Rec-sys popularity biases amplify over time (Chaney et al., 2018 [*How algorithmic confounding in recommender systems increases homogeneity and decreases utility*](https://arxiv.org/abs/1710.11214)); moderation models trained on prior moderation decisions replicate historical biases. Mitigation is *program-level*: fresh data sourced independent of the model's own decisions, periodic audits of the feedback loop, monitoring for signal that a population is being pushed further from the training distribution over time.
- **Aggregation-based harm.** No single prediction is harmful; the aggregation of predictions over a population is (redlining-style disparate impact even when the model was fair on every individual case). Mitigation is *slice-level monitoring* — the chapter 03 review sees this before an incident does.
- **Cross-model composition.** The model's outputs are composed with another model's outputs to produce a decision neither model was evaluated for. LLM-augmented pipelines (retriever + reranker + generator + safety classifier) hit this often — each component may pass its own eval and the composed system may not (mod-304 chapter 03's composition discipline is the guardrail).

The senior discipline is to enumerate the misuse patterns *realistic for the domain*, not to try to enumerate every conceivable misuse. Misuse patterns without a plausible mechanism in this deployment are noise; the ones the deployment actually enables are the load-bearing threats.

## §4 — Threat modelling by domain: making it concrete

Threat models fail when they are generic. "Adversarial examples are a concern" is not a threat-model entry; "an adversary sending malformed transactions could evade our fraud model in the first minute of a coordinated attack" is. The senior discipline is to pick the two-to-five attacks or misuse patterns most *specific to your domain* and threat-model *those* deeply, rather than tick every category shallowly.

Sketches for four common deployment types (these are prompts for your own threat modelling, not a substitute):

- **Recommender / search-ranking system.** Priority threats: shilling attacks and content-injection poisoning; feedback-loop amplification of popularity biases; disparate exposure across creator groups. Priority misuse: off-label use in editorial decisions; cross-model composition into a downstream personalisation layer that removes diversity.
- **Fraud / abuse / trust-and-safety classifier.** Priority threats: evasion attacks (adversarial paraphrase, homoglyph substitution, transaction restructuring); coordinated abuse at scale; model-extraction to build a bypass tool. Priority misuse: cross-decision reuse in higher-consequence contexts (fraud signal used in account-suspension without human review).
- **LLM-augmented product feature.** Priority threats: prompt injection (indirect, through retrieval sources — Greshake et al., 2023 [*Not what you've signed up for*](https://arxiv.org/abs/2302.12173)); jailbreak prompts; training-data extraction; abuse at scale for content generation. Priority misuse: hallucinated content used as truth; automation without oversight; feedback loop where model outputs are re-ingested as training data.
- **Perception / safety-critical model.** Priority threats: physical-world adversarial patches; distribution-shift induced misclassification with high-consequence outcome; corner-case robustness failures. Priority misuse: off-label operational-condition deployment (system trained on daytime data used at night); automation without appropriate human-driver / operator model.

Each of the above is a *starting checklist*, not a complete threat model. The concrete threat model is authored *for this deployment*, referencing MITRE ATLAS tactic IDs where applicable, with domain-specific attacker profiles (who has the capability, what motivates them, what their access pattern looks like).

## §5 — The mitigation map: where each mitigation lands in the system

A threat-model entry without a mitigation is a wish; a mitigation without a *system location* is a suggestion. The senior discipline is to map every mitigation to a specific control in the deployed system.

The seven landing sites:

- **Model-card §2 (intended use / out-of-scope).** Statements the deployment's consumers are expected to respect. Weak on its own; strong when composed with an API-authentication scope check.
- **Model-card §8 (ethical considerations) and §9 (caveats).** Statements the deployment *discloses* to reviewers. The discipline for accepted-risk (F3) findings from chapter 03.
- **Feature pipeline (chapter 02).** Poisoning defences that live at ingest — trusted-source authentication, anomaly detection on incoming data, human-review sampling for label integrity.
- **Training pipeline.** Robustness measures at train time — adversarial training, differential privacy, label-noise-robust losses.
- **Inference path.** Runtime input-anomaly detection, rate-limiting, safety-classifier layers, prompt-injection defences (for LLMs, [Perez & Ribeiro, 2022](https://arxiv.org/abs/2211.09527) and follow-on literature), output post-processing.
- **Monitoring SLIs (mod-307 chapter 02).** Distribution-drift SLIs that would surface a poisoning campaign or adversarial-input burst. Rate-limit-hit rate as an SLI that surfaces coordinated abuse.
- **Incident-response taxonomy (mod-307 chapter 04).** *Adversarial abuse* as a named category in the taxonomy, with a runbook that covers detection, containment, and postmortem. The chapter 04 template's *Adversarial abuse* incident type is where the response to a materialised threat lives.
- **Peer-track hand-off.** For ML/AI security depth beyond the senior ML engineer's altitude — advanced adversarial-robustness certification, formal verification, red-team engagement design — the [`ai-infra-security-learning`](../mod-308-platform-collaboration/resources.md) track picks up. The hand-off contract discipline from [mod-308 chapter 04](../mod-308-platform-collaboration/04-handoff-contracts-to-specialists.md) transfers here.

A well-authored threat model produces a **mitigation matrix**: rows are threats and misuse patterns, columns are the seven landing sites, cells are `owned / partial / not-covered / not-applicable`. Empty columns for a high-severity row are the open items the review packet escalates.

## §6 — Anchoring the threat model to repeatable tests

The fifth ML frame question — *how do we know the mitigation still works?* — is what keeps the threat model from becoming a launch-day artefact that ages into a fiction. Every mitigation should anchor to a *test*.

Test types:

- **Safety-slice tests.** The chapter 03 §1.3 safety slices *are* the tests for LLM prompt-injection, jailbreak, harmful-content, and PII-leakage mitigations. The threat-model entry cites the specific safety slice; the safety-review packet reports the result.
- **Robustness-slice tests.** Chapter 03 §1.2's robustness slices are the tests for evasion-attack mitigations. The threat-model entry cites the specific corruption or shift the slice tests.
- **Monitoring alerts.** The threat-model entry references the mod-307 chapter 02 SLI that would detect the attack materialising in production; the alert *is* the test.
- **Red-team drills.** For high-severity threats, a periodic *drill* — humans acting as attackers, with the deployment's defences engaged — is the test. The mod-307 chapter 04 gamedays discipline extends here; a red-team drill is a gameday whose scenario is one of the threat-model entries.
- **Backdoor and poisoning scans.** For training-data-poisoning defences, the test is a scan run on the training set and the trained model as part of the retraining pipeline.

Every mitigation in the matrix cites the test that catches its failure. Mitigations that do not cite a test are mitigations that will silently rot; they either need a test added or a decision to accept the residual risk (with the chapter 03 F3 discipline and model-card disclosure).

## §7 — The threat-model artefact

The threat model ships as a document with the review packet (chapter 03 §6) and the model card (chapter 01 §6). Skeleton:

```markdown
# Threat model — <feature-name> — v<version>

**Model version.** <registry URI>
**Author.** <name, role>
**Reviewers.** <ML/AI security peer, governance analyst, product policy>
**Last reviewed.** <date>
**Next review due.** <date>

## 1. Deployment description
<Two-to-three paragraphs on what the deployment does, whom it serves, what decisions it enables. Point to model card §1 and §2. Point to the lineage graph and PII register (chapter 02).>

## 2. Attacker profiles
<For each realistic attacker: capability (query access, training access, physical access), motivation, resource level, access pattern. Two-to-five profiles maximum; anything more is unfocused.>

## 3. Adversarial threats (§2)
### 3.1 Evasion attacks
<Specific threats realistic for this deployment; cite MITRE ATLAS tactic ID where applicable.>
### 3.2 Data poisoning
<Same.>
### 3.3 Model extraction and membership inference
<Same.>
### 3.4 Abuse at scale
<Same.>

## 4. Misuse patterns (§3)
<Specific misuse patterns for this deployment. Each with a plausible mechanism.>

## 5. Mitigation matrix (§5)
<Threat / misuse × landing-site table. Cells: owned / partial / not-covered / not-applicable, with the specific control named.>

## 6. Test anchoring (§6)
<For each mitigation: the test type and the reference (safety slice ID, SLI name, drill schedule, scan pipeline).>

## 7. Open items and residual risk
<Findings ladder — F0 blocking, F1 roadmap, F2 backlog, F3 accepted risk with governance signoff — same shape as chapter 03 §5.4.>

## 8. Cross-references
- Model card version: <URI>
- Lineage audit: <URI>
- Review-cadence packet: <URI>
- SLO document: <URI>
- Incident-response taxonomy entries touched: <list>

## 9. Signoff
- ML team owner: <name, date>
- ML/AI security peer: <name, date>
- Governance analyst: <name, date>
```

Three properties of the artefact that matter:

- **The threat model is versioned with the model.** Every promoted model version has a threat-model version. A model change that adds a new capability or removes a mitigation forces a threat-model revision before promotion.
- **The threat model is *reviewed*, not just written.** ML/AI security peer review is the reason the artefact is signed by more than the ML team. The senior ML engineer's job is to write and drive the review; the peer-track expert's job is to challenge and sign.
- **The threat model composes with the review packet.** The safety-slice results in the chapter 03 packet are the *test evidence* for the threat model's mitigation matrix. Neither artefact is self-supporting; both live together in the release packet.

## §8 — Common failure modes

Five failure modes that recur across first-cycle threat models:

- **The threat model is a shopping list.** Every category in every taxonomy is listed with "we should think about this." No specific threat, no attacker profile, no mitigation, no test. The fix: prune to the two-to-five deployment-realistic threats and go deep, not wide.
- **The threat model is a launch-day artefact.** Written for the launch review, never touched again. Six months later the deployment has changed and the threat model has not. The fix: version-with-the-model discipline, plus the cadence policy of chapter 01 §7.
- **The mitigation is documentary but not enforced.** "The model card says do not use for X" without a scope-check on the API. The fix: every documentary mitigation is either paired with an enforcement mechanism or explicitly noted as *disclosure-only* with the residual risk accepted.
- **The threat model does not name misuse.** All adversarial, no misuse. In many deployments the misuse patterns are the higher-frequency real-world hazard; skipping them is a failure of imagination the reviewer will notice. The fix: force yourself to write §3.4 (Weidinger-style social-risk taxonomy) *before* §3.1.
- **The threat model is done alone.** No security-peer review, no governance-analyst signoff, no engagement with the incident-response team. The threat model is by definition an artefact multiple audiences depend on; it fails without their input. The fix: the review is *scheduled* into the calendar the day the threat model is drafted, not "when we get to it."

## Summary

A ML threat model asks the five-question frame (what are we building, what can go wrong adversarially, what can go wrong through misuse, what are we doing about it, how do we know it still works) and produces a documented artefact that the review packet cites. The adversarial surface has four canonical categories — evasion, poisoning, model extraction / membership inference, and abuse at scale — each with a domain-relevant literature and a mitigation family. The misuse surface — off-label use, automation without oversight, feedback-loop harm, aggregation-based harm, cross-model composition — is what most sharply distinguishes an ML threat model from a classical software one. Domain-specific concreteness is the discipline: two-to-five threats specific to the deployment, deeply modelled, beat every-category-shallow. The mitigation matrix maps each threat and misuse pattern to one of seven landing sites — model-card scope, model-card disclosure, feature pipeline, training pipeline, inference path, monitoring SLIs, incident-response taxonomy — with peer-track hand-off to `ai-infra-security-learning` for depth. Every mitigation anchors to a *test* — safety slice, robustness slice, monitoring alert, red-team drill, or poisoning scan — because otherwise the mitigation silently rots. The threat-model artefact ships versioned-with-the-model, ML/AI-security-peer-reviewed and governance-analyst-signed, and composes with the chapter 03 review packet. Five failure modes — the shopping list, the launch-day artefact, the unenforced mitigation, the missing misuse section, and the not-peer-reviewed artefact — are the ones a senior discipline catches in your own draft before the reviewer does.
