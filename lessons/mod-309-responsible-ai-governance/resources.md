# Resources for mod-309-responsible-ai-governance

Curated external references. These are the primary and authoritative sources the chapters and exercises cite or lean on. Every URL is publicly reachable at time of authoring; where a paper's canonical URL is behind a paywall, an author-hosted preprint URL (arXiv) is provided instead.

## The load-bearing frames (all chapters)

- **[NIST AI Risk Management Framework (AI RMF 1.0)](https://www.nist.gov/itl/ai-risk-management-framework)** — the *Govern → Map → Measure → Manage* functions this module's cadence, review packets, and threat model compose against. The [AI RMF Playbook](https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook) is the practical accompaniment.
- **[NIST Generative AI Profile — NIST AI 600-1](https://csrc.nist.gov/publications/detail/nist-ai/600-1/final)** (2024) — GenAI-specific profile that extends the RMF for LLM-augmented deployments.
- **[EU AI Act](https://artificialintelligenceact.eu/)** — the regulatory framework whose risk tiers (unacceptable, high, limited, minimal) chapter 03's cadence table cites. The [Act's official text](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32024R1689) is authoritative.
- **[OECD AI Principles](https://oecd.ai/en/ai-principles)** — the multilateral principles most modern internal AI-use policies derive from. Useful as the intergovernmental baseline.
- **[Federal Reserve SR 11-7 — Model Risk Management guidance](https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm)** — the U.S. banking-regulator model-risk-management standard. The template many enterprise AI governance programs derive from even outside banking.
- **[ISO/IEC 42001:2023 — AI Management System](https://www.iso.org/standard/81230.html)** — the international standard for AI management systems. Increasingly cited in vendor and enterprise procurement.

## Model cards, data statements, and datasheets (chapter 01, exercise 01)

- **Margaret Mitchell et al., *Model Cards for Model Reporting*** (FAT* 2019). The paper that named the artefact. <https://arxiv.org/abs/1810.03993>
- **Emily Bender & Batya Friedman, *Data Statements for Natural Language Processing: Toward Mitigating System Bias and Enabling Better Science*** (TACL 2018). The data-statement schema chapter 01 §2 leans on. <https://aclanthology.org/Q18-1041/>
- **Timnit Gebru et al., *Datasheets for Datasets*** (CACM 2021). The general-purpose sibling of Bender-Friedman for non-NLP data. <https://arxiv.org/abs/1803.09010>
- **Google Model Card Toolkit** — reference implementation for programmatically generating model cards. <https://github.com/tensorflow/model-card-toolkit>
- **Google Model Cards examples**, including the widely-cited face detection card. <https://modelcards.withgoogle.com/about> · <https://modelcards.withgoogle.com/face-detection>
- **Hugging Face Model Cards documentation** — the current industry-standard model-card template, actively used by the open-model ecosystem. <https://huggingface.co/docs/hub/model-cards>
- **Sarah Holland et al., *The Dataset Nutrition Label*** (2018) — adjacent framing that predates and complements Bender-Friedman. <https://arxiv.org/abs/1805.03677>
- **Yonatan Zunger, *The Friendly Design Doc*** — the load-bearing reference for writing artefacts *to* the reviewer at their altitude; chapter 01 §3's discipline. <https://medium.com/@yonatanzunger/the-friendly-design-doc-2c96b8ba0842>
- **Ramya Ramaswamy et al., *A Framework for Deprecating Datasets: Standardizing Documentation, Identification, and Communication*** (FAccT 2022) — extends the datasheet discipline through the deprecation lifecycle. <https://arxiv.org/abs/2111.04424>
- **Inioluwa Deborah Raji et al., *Closing the AI Accountability Gap: Defining an End-to-End Framework for Internal Algorithmic Auditing*** (FAccT 2020) — the review-packet framing chapter 01 §6 composes with. <https://arxiv.org/abs/2001.00973>
- **Google Cloud Vertex AI Model Registry — model-card integration.** The registry-artefact-integration pattern chapter 01 §4 references. <https://cloud.google.com/vertex-ai/docs/model-registry/introduction>
- **AWS SageMaker Model Cards documentation.** Same pattern from AWS. <https://docs.aws.amazon.com/sagemaker/latest/dg/model-cards.html>

## Data lineage and PII handling (chapter 02, exercise 02)

### Regulatory and standards references

- **[GDPR — full text](https://gdpr.eu/tag/gdpr/)** and specifically [Article 30 (Records of Processing Activities)](https://gdpr.eu/article-30-records-of-processing-activities/). The regulatory driver for the lineage register.
- **[California Consumer Privacy Act (CCPA / CPRA)](https://oag.ca.gov/privacy/ccpa)** — the U.S. counterpart with deletion-rights and purpose-limitation implications.
- **[HIPAA Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html)** and the [Safe Harbor / Expert Determination methods](https://www.hhs.gov/hipaa/for-professionals/privacy/special-topics/de-identification/index.html) for de-identification.
- **[NIST Privacy Framework](https://www.nist.gov/privacy-framework)** — the *Identify → Govern → Control → Communicate → Protect* functions that structure the data-mapping discipline.
- **[NIST SP 800-188 — De-Identification of Personal Information](https://csrc.nist.gov/publications/detail/sp/800-188/final)** — the reference for PII taxonomy, direct vs. quasi-identifiers, and the re-identification-risk framework.
- **[NIST SP 800-122 — Guide to Protecting the Confidentiality of PII](https://csrc.nist.gov/publications/detail/sp/800-122/final)** — the practical PII-handling reference.
- **[COPPA — Children's Online Privacy Protection Act](https://www.ftc.gov/business-guidance/privacy-security/childrens-privacy)** — the sector rule for products with users under 13.
- **[FCRA — Fair Credit Reporting Act](https://www.ftc.gov/legal-library/browse/statutes/fair-credit-reporting-act)** — the sector rule for consumer credit signals.

### Foundational academic references

- **Latanya Sweeney, *Simple Demographics Often Identify People Uniquely*** (Carnegie Mellon LIDAP 2000). The empirical result that 87 % of the U.S. population is uniquely identifiable by (ZIP, birthdate, gender). <https://dataprivacylab.org/projects/identifiability/paper1.pdf>
- **Pierangela Samarati & Latanya Sweeney, *Protecting Privacy when Disclosing Information: k-Anonymity and Its Enforcement through Generalization and Suppression*** (1998). The founding paper of k-anonymity. <https://epic.org/wp-content/uploads/privacy/reidentification/Samarati_Sweeney_paper.pdf>
- **Cynthia Dwork & Aaron Roth, *The Algorithmic Foundations of Differential Privacy*** (2014). The reference text for the differential-privacy discipline chapter 02 §5.3 hands off to. <https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf>
- **Martin Abadi et al. (Google), *Deep Learning with Differential Privacy*** (CCS 2016). The DP-SGD reference. <https://arxiv.org/abs/1607.00133>
- **Nicholas Carlini et al., *Extracting Training Data from Large Language Models*** (USENIX 2021). The training-data-extraction attack. <https://arxiv.org/abs/2012.07805>
- **Nicholas Carlini et al., *Quantifying Memorization Across Neural Language Models*** (ICLR 2023). The follow-up quantifying memorisation as a function of model size and duplication. <https://arxiv.org/abs/2202.07646>

### Lineage tooling references

- **[OpenLineage — specification](https://openlineage.io/spec/)** and the [OpenLineage documentation](https://openlineage.io/). The emerging open standard for lineage event emission.
- **[Marquez](https://marquezproject.ai/)** — the OpenLineage reference backend.
- **[DataHub](https://datahubproject.io/)** — LinkedIn-origin open-source metadata platform with lineage tracking.
- **[Amundsen](https://www.amundsen.io/)** — Lyft-origin metadata / discovery / lineage platform.
- **[Apache Atlas](https://atlas.apache.org/)** — Hadoop-ecosystem lineage and governance platform.
- **[Great Expectations](https://greatexpectations.io/)** — data-quality validation that composes with lineage-based audits.
- **[Microsoft Presidio](https://microsoft.github.io/presidio/)** — open-source PII detection / redaction toolkit; useful as a T2 normalisation-node implementation reference.

### Additional papers on training-data provenance

- **Nithya Sambasivan et al. (Google), *"Everyone wants to do the model work, not the data work": Data Cascades in High-Stakes AI*** (CHI 2021). The empirical study of how ignoring data-provenance discipline compounds downstream. <https://research.google/pubs/pub49953/>
- **Frances Ding et al., *Retiring Adult: New Datasets for Fair Machine Learning*** (NeurIPS 2021). Reference for what dataset-provenance issues look like in the widely-used fairness benchmarks. <https://arxiv.org/abs/2108.04884>

## Fairness, robustness, and safety review (chapter 03, exercise 03)

### Fairness — foundational references

- **Solon Barocas, Moritz Hardt, Arvind Narayanan, *Fairness and Machine Learning: Limitations and Opportunities*** (fairmlbook.org). The canonical textbook, still in progress. <https://fairmlbook.org/>
- **Ninareh Mehrabi et al., *A Survey on Bias and Fairness in Machine Learning*** (ACM Computing Surveys 2021). The lay-of-the-land survey. <https://arxiv.org/abs/1908.09635>
- **Moritz Hardt, Eric Price, Nathan Srebro, *Equality of Opportunity in Supervised Learning*** (NeurIPS 2016). The equal-opportunity / equalised-odds definitions. <https://arxiv.org/abs/1610.02413>
- **Alexandra Chouldechova, *Fair Prediction with Disparate Impact: A Study of Bias in Recidivism Prediction Instruments*** (FAT/ML 2016 / Big Data 2017). One of the two founding impossibility papers. <https://arxiv.org/abs/1610.07524>
- **Jon Kleinberg, Sendhil Mullainathan, Manish Raghavan, *Inherent Trade-Offs in the Fair Determination of Risk Scores*** (ITCS 2017). The other founding impossibility paper. <https://arxiv.org/abs/1609.05807>
- **Joy Buolamwini & Timnit Gebru, *Gender Shades*** (FAT* 2018). The intersectional-audit paper on commercial face classifiers. <http://gendershades.org/> · <https://proceedings.mlr.press/v81/buolamwini18a.html>
- **Michael Kearns et al., *Preventing Fairness Gerrymandering: Auditing and Learning for Subgroup Fairness*** (ICML 2018). The subgroup-fairness intractability result. <https://arxiv.org/abs/1711.05144>
- **Julia Angwin et al. (ProPublica), *Machine Bias*** (2016). The COMPAS case study — the reference incident for redlining-style proxy variables. <https://www.propublica.org/article/machine-bias-risk-assessments-in-criminal-sentencing>
- **Mitchell Gordon et al., *Jury Learning: Integrating Dissenting Voices into Machine Learning Models*** (CHI 2022). Reference for annotator-population effects on fairness. <https://arxiv.org/abs/2202.02950>

### Fairness — tooling

- **[Fairlearn](https://fairlearn.org/)** (Microsoft-open-source). The most-used Python library for fairness metrics and mitigations.
- **[AIF360 — AI Fairness 360](https://aif360.res.ibm.com/)** (IBM-open-source). Broader metric-and-mitigation library; harder learning curve.
- **[Aequitas](http://aequitas.dssg.io/)** (University of Chicago DSaPP). Bias-audit toolkit.
- **[What-If Tool](https://pair-code.github.io/what-if-tool/)** (Google PAIR). Interactive fairness / counterfactual exploration UI.
- **[Fiddler Auditor](https://github.com/fiddler-labs/fiddler-auditor)** — open-source LLM-fairness auditor.

### Robustness — foundational references

- **Dan Hendrycks & Thomas Dietterich, *Benchmarking Neural Network Robustness to Common Corruptions and Perturbations*** (ICLR 2019). ImageNet-C / ImageNet-P — the reference corruption suite. <https://arxiv.org/abs/1903.12261>
- **Marco Tulio Ribeiro et al., *Beyond Accuracy: Behavioral Testing of NLP Models with CheckList*** (ACL 2020). The reference for NLP behavioural robustness testing. <https://arxiv.org/abs/2005.04118> · <https://github.com/marcotcr/checklist>
- **Percy Liang et al., *Holistic Evaluation of Language Models (HELM)*** (Stanford CRFM). Multi-metric, multi-slice LLM evaluation including a robustness scenario. <https://crfm.stanford.edu/helm/>

### Safety and harm — references

- **Laura Weidinger et al. (DeepMind), *Ethical and social risks of harm from Language Models*** (2021). The reference LLM-harm taxonomy. <https://arxiv.org/abs/2112.04359>
- **Deep Ganguli et al. (Anthropic), *Red Teaming Language Models to Reduce Harms*** (2022). Anthropic's LLM red-teaming methodology paper. <https://arxiv.org/abs/2209.07858>
- **Ethan Perez et al., *Red Teaming Language Models with Language Models*** (EMNLP 2022). Automated red-teaming corpus generation. <https://arxiv.org/abs/2202.03286>
- **Mantas Mazeika et al., *HarmBench: A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal*** (ICML 2024). A widely-used harm-category taxonomy for red-team eval sets. <https://arxiv.org/abs/2402.04249>
- **Sean McGregor, *Preventing Repeated Real World AI Failures by Cataloging Incidents: The AI Incident Database*** (AIES 2021). The reference incident catalogue. <https://incidentdatabase.ai/>

## Threat modelling — misuse and adversarial (chapter 04, exercise 04)

### Foundational threat-modelling references

- **Adam Shostack, *Threat Modeling: Designing for Security*** (Wiley, 2014). The general reference the ML-specific variant composes with. <https://shostack.org/books/threat-modeling-book>
- **Microsoft STRIDE threat-modelling documentation** — the tactical model classical threat models operationalise. <https://learn.microsoft.com/en-us/security/engineering/threat-modeling-tool-threats>
- **[OWASP Threat Modeling](https://owasp.org/www-community/Threat_Modeling)** — the open-source threat-modelling knowledge base.

### ML-specific threat taxonomies

- **[MITRE ATLAS — Adversarial Threat Landscape for Artificial-Intelligence Systems](https://atlas.mitre.org/)**. The canonical adversarial-ML taxonomy with tactic / technique IDs and case studies. Chapter 04's threat-modelling exercise cites this.
- **[NIST AI 100-2e2023 — Adversarial Machine Learning: A Taxonomy and Terminology](https://csrc.nist.gov/publications/detail/ai/100-2e2023/final)**. The reference taxonomy for evasion, poisoning, privacy, and abuse.
- **[OWASP Top 10 for Large Language Model Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)**. The LLM-specific hazard list every LLM-augmented deployment cross-references.
- **[OWASP Machine Learning Security Top 10](https://owasp.org/www-project-machine-learning-security-top-10/)**. The classical-ML sibling to the LLM list.
- **Ram Shankar Siva Kumar et al. (Microsoft), *Failure Modes in Machine Learning*** (2019). The intentional-vs.-unintentional failure-mode taxonomy. <https://learn.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning>
- **[Microsoft Responsible AI Standard](https://www.microsoft.com/en-us/ai/principles-and-approach)** — practical adjacent framing from a large deployer.

### Adversarial attacks — evasion

- **Ian Goodfellow et al., *Explaining and Harnessing Adversarial Examples*** (ICLR 2015). The gradient-based adversarial-example paper. <https://arxiv.org/abs/1412.6572>
- **Nicholas Carlini & David Wagner, *Towards Evaluating the Robustness of Neural Networks*** (IEEE S&P 2017). The strong-optimisation attack. <https://arxiv.org/abs/1608.04644>
- **Aleksander Madry et al., *Towards Deep Learning Models Resistant to Adversarial Attacks*** (ICLR 2018). PGD-based adversarial training. <https://arxiv.org/abs/1706.06083>
- **Kevin Eykholt et al., *Robust Physical-World Attacks on Deep Learning Visual Classification*** (CVPR 2018). The stop-sign patch paper. <https://arxiv.org/abs/1707.08945>
- **Anish Athalye et al., *Synthesizing Robust Adversarial Examples*** (ICML 2018). 3D-printed adversarial-object attacks. <https://arxiv.org/abs/1707.07397>
- **Andy Zou et al., *Universal and Transferable Adversarial Attacks on Aligned Language Models*** (2023). Adversarial-suffix jailbreaks for LLMs. <https://arxiv.org/abs/2307.15043>

### Adversarial attacks — poisoning and backdoor

- **Battista Biggio, Blaine Nelson, Pavel Laskov, *Poisoning Attacks against Support Vector Machines*** (ICML 2012). The founding poisoning paper. <https://arxiv.org/abs/1206.6389>
- **Tianyu Gu et al., *BadNets: Identifying Vulnerabilities in the Machine Learning Model Supply Chain*** (2017). The backdoor-trigger paper. <https://arxiv.org/abs/1708.06733>
- **Nicholas Carlini et al., *Poisoning Web-Scale Training Datasets is Practical*** (2023). Practical poisoning against internet-scale corpora. <https://arxiv.org/abs/2302.10149>
- **Javier Rando & Florian Tramèr, *Universal Jailbreak Backdoors from Poisoned Human Feedback*** (2023). RLHF-preference-data poisoning. <https://arxiv.org/abs/2311.14455>
- **Shyong Lam & John Riedl, *Shilling Recommender Systems for Fun and Profit*** (WWW 2004). The recommender-shilling attack. <https://dl.acm.org/doi/10.1145/988672.988726>

### Privacy attacks — extraction and inference

- **Florian Tramèr et al., *Stealing Machine Learning Models via Prediction APIs*** (USENIX 2016). The model-extraction paper. <https://arxiv.org/abs/1609.02943>
- **Reza Shokri et al., *Membership Inference Attacks Against Machine Learning Models*** (IEEE S&P 2017). The membership-inference paper. <https://arxiv.org/abs/1610.05820>
- **Nicholas Carlini et al. (see chapter 02 references above)** for training-data extraction from LLMs.

### LLM-specific attacks

- **Kai Greshake et al., *Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection*** (AISec 2023). The indirect prompt-injection reference. <https://arxiv.org/abs/2302.12173>
- **Fábio Perez & Ian Ribeiro, *Ignore Previous Prompt: Attack Techniques for Language Models*** (2022). The direct prompt-injection reference. <https://arxiv.org/abs/2211.09527>
- **John Kirchenbauer et al., *A Watermark for Large Language Models*** (ICML 2023). LLM output watermarking. <https://arxiv.org/abs/2301.10226>

### Misuse / social-risk references

- **Laura Weidinger et al. (DeepMind), *Ethical and social risks of harm from Language Models*** (2021). Duplicated from chapter 03 — the reference for the misuse taxonomy chapter 04 §3 leans on. <https://arxiv.org/abs/2112.04359>
- **Josh A. Goldstein et al. (OpenAI + Georgetown CSET), *Generative Language Models and Automated Influence Operations*** (2023). The industry treatment of abuse-at-scale. <https://openai.com/research/forecasting-misuse>
- **Allison Chaney, Brandon Stewart, Barbara Engelhardt, *How Algorithmic Confounding in Recommendation Systems Increases Homogeneity and Decreases Utility*** (RecSys 2018). Reference for feedback-loop harm. <https://arxiv.org/abs/1710.11214>

### Robustness / adversarial-ML tooling

- **[Adversarial Robustness Toolbox (ART)](https://adversarial-robustness-toolbox.readthedocs.io/)** (Linux Foundation AI). Adversarial-attack-and-defence library.
- **[Foolbox](https://foolbox.readthedocs.io/)** — Python adversarial-example library.
- **[CleverHans](https://github.com/cleverhans-lab/cleverhans)** — the early adversarial-example benchmark suite; still cited.
- **[TextAttack](https://github.com/QData/TextAttack)** — NLP-specific adversarial-attack framework.
- **[Garak — Generative AI Red-Teaming and Assessment Kit](https://github.com/leondz/garak)** — LLM red-teaming toolkit.

## Governance forums and industry / applied treatments

- **[Partnership on AI](https://partnershiponai.org/)** — the multi-stakeholder forum whose model-card and system-card guidance many enterprise deployers cite.
- **[Anthropic's Responsible Scaling Policy](https://www.anthropic.com/rsp)** — a public example of a graduated-mitigation policy tied to capability level.
- **[Google's Responsible AI Practices](https://ai.google/responsibility/responsible-ai-practices/)** — publicly published practical recommendations.
- **[OpenAI's Usage Policies](https://openai.com/policies/usage-policies/)** and their [system cards](https://openai.com/system-card) — public examples of the deployment-level artefact this module's model card composes with.
- **[Meta's System Cards](https://ai.meta.com/tools/system-cards/)** — deployment-level cards.
- **[Hugging Face's Ethical Consideration series](https://huggingface.co/docs/hub/model-card-appendix)** — practical appendix to model cards.
- **[UK AI Safety Institute (AISI)](https://www.aisi.gov.uk/)** — publicly-published safety-evaluation methodology.

## Cross-references inside this module

- Chapter 01 (`01-model-cards-and-data-statements.md`) — the model card as decision artefact; Mitchell nine sections; Bender-Friedman data statement; governance-analyst altitude; template; five failure modes; hand-off; cadence policy.
- Chapter 02 (`02-data-lineage-and-pii-handling.md`) — lineage graph as evidence; five-tier canonical flow; four-category PII taxonomy; five-column PII register; consent / purpose / retention; three collection patterns; seven-step audit.
- Chapter 03 (`03-fairness-robustness-safety-review-cadence.md`) — the three review dimensions; metric-choice discipline with impossibility results; always-on vs. periodic-review split; cadence choice; four scoping moves; F0 / F1 / F2 / F3 findings ladder; review packet.
- Chapter 04 (`04-misuse-and-adversarial-threat-modeling.md`) — five-question ML frame; four adversarial categories; five misuse patterns; seven mitigation landing sites; test-anchor discipline; threat-model artefact; five failure modes.

## Cross-references to other modules

- [mod-302 chapter 05 (retraining and rollout)](../mod-302-ml-systems-architecture/) — the retraining-trigger set the review cadence syncs to.
- [mod-304 chapter 03 (prompts, tools, retrieval)](../mod-304-production-llm-integration/) — the LLM composition surface chapter 04's prompt-injection threats are named against.
- [mod-305 chapter 02 (slice and adversarial guardrails)](../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) — the always-on layer under chapter 03's review cadence.
- [mod-305 chapter 04 (ML Test Score / production-readiness)](../mod-305-advanced-evaluation/04-ml-test-score-production-readiness.md) — one of the review-packet documents that composes with this module's card + audit + review + threat model.
- [mod-306 chapter 02 (SRM and trust diagnostics)](../mod-306-experimentation-at-scale/02-srm-and-trust-diagnostics.md) — the online slice-diagnostic twin of chapter 03's offline review.
- [mod-307 chapter 01 (SRE fundamentals for ML)](../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md) — the SLO document that composes with the review packet.
- [mod-307 chapter 02 (ML-specific SLIs)](../mod-307-ml-reliability-slos/02-ml-specific-slis-and-slos.md) — the SLI menu chapter 04's monitoring-mitigation column reads.
- [mod-307 chapter 04 (incident response)](../mod-307-ml-reliability-slos/04-ml-incident-response-and-postmortems.md) — the taxonomy chapter 04's *Adversarial abuse* category extends and the *governance-visible incident* branch chapter 02's audit findings feed.
- [mod-307 chapter 05 (retraining as deploy)](../mod-307-ml-reliability-slos/05-retraining-as-a-first-class-deploy.md) — the deploy lifecycle the model-card cadence composes with.
- [mod-308 chapter 01 (paved-road consumption)](../mod-308-platform-collaboration/01-paved-road-consumption.md) — the platform-primitive inventory chapter 01 §9 caveats cite; the registration-of-standard-artefacts discipline chapter 01 §4 composes with.
- [mod-308 chapter 03 (contribute-back RFCs)](../mod-308-platform-collaboration/03-contribute-back-rfcs.md) — the sponsor-conversation discipline chapter 01 §6 governance-analyst hand-off inherits.
- [mod-308 chapter 04 (hand-off contracts)](../mod-308-platform-collaboration/04-handoff-contracts-to-specialists.md) — the six-section hand-off contract this module reuses when handing depth to `ai-governance-analyst` and `ai-infra-security-learning`.
- [mod-310](../mod-310-technical-leadership/) — the technical-leadership altitude at which the review packet is defended when the deployment goes cross-org.

## Peer specialist tracks (hand-off targets)

Consult each track's own README for the specialist-side delegation vocabulary.

- **`ai-governance-analyst-learning`** — the peer track for governance and compliance depth (chapter 01 §6 hand-off target, chapter 03 §6 packet signer).
- **`ai-infra-security-learning`** — the peer track for ML/AI security and adversarial-ML depth (chapter 04 hand-off target).
- **`model-evaluation-engineer-learning`** / **`ai-eval-engineer-learning`** — the peer tracks for evaluation-platform depth; chapter 03's harness composes with mod-305's delegation.
- **`ai-infra-mlops-learning`** — the peer track whose registry-gate discipline chapter 01 §7's promotion-gate enforcement leans on.
- **`ai-infra-ml-platform-learning`** — the peer track whose paved-road primitives chapter 02's lineage graph is built on.

<!-- needs-research: verify the peer-track repo URLs once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->
