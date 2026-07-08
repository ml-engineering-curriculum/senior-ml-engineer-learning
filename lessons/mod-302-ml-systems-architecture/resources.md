# Resources for mod-302-ml-systems-architecture

Curated external references. These are the primary and authoritative sources cited or leaned on by the chapters and exercises above. Every URL is publicly reachable at the time of authoring.

## Canonical ML systems texts (all chapters)

- **Chip Huyen, *Designing Machine Learning Systems*** (O'Reilly, 2022) — the architecture-altitude text this module operates at. Chapters 3 (data engineering), 5 (feature engineering), 7 (model deployment), and 8 (data distribution shifts and monitoring) map directly to chapters 02–05 of this module. Publisher page: <https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/> · Author page: <https://huyenchip.com/books/>.
- **Andriy Burkov, *Machine Learning Engineering*** (2020) — the build-altitude prerequisite. If the L20 material feels unfamiliar, this is the book to pick up before this module. <https://www.mlebook.com/>
- **Google Developers, *Rules of Machine Learning*** — 43 heuristics. Rules #29 and #32 are the classic training/serving skew warnings; Rules #4–#8 shape the "how to launch v1" thinking used in chapter 01. <https://developers.google.com/machine-learning/guides/rules-of-ml>

## The "hidden technical debt" line of research (chapters 01, 04)

- **Sculley et al., *Hidden Technical Debt in Machine Learning Systems*** (NeurIPS 2015) — the paper this module leans on to name the L30 shift: most of a mature ML system's cost is the glue around the model, and owning that glue is the senior job. Cited explicitly in chapters 01 and 05 (feedback-loop failure modes). <https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html>
- **Breck et al., *The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction*** (Google, 2017) — the production-readiness rubric this module's parity checklist (chapter 04) is inspired by. Re-read once you get to mod-305 (evaluation) and mod-307 (reliability). <https://research.google/pubs/pub46555/>
- **Polyzotis, Roy, Whang, Zinkevich, *Data Management Challenges in Production Machine Learning*** (SIGMOD 2017 tutorial) — the industrial-strength view of the offline/online feature axis and skew categories. <https://dl.acm.org/doi/10.1145/3035918.3054782>

## MLOps and the paved-road platform posture (chapters 03, 05)

- **Google Cloud, *MLOps: Continuous delivery and automation pipelines in machine learning*** — the canonical description of the "MLOps levels" (0, 1, 2). The level-2 shape is the target this module trains you to write RFCs against. <https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning>
- **Kreuzberger, Kühl, Hirschl, *Machine Learning Operations (MLOps): Overview, Definition, and Architecture*** — academic MLOps survey; useful vocabulary for the hand-off contracts to peer platform tracks (see [mod-308]). <https://arxiv.org/abs/2205.02302>
- **Google Cloud, *Practitioners Guide to MLOps*** (whitepaper, 2021) — a longer prose treatment of the same lifecycle. <https://cloud.google.com/resources/mlops-whitepaper>

## Feature stores (chapter 03, exercise-02)

- **Feast** (open source) — the reference open-source feature store; documentation covers the offline/online store split, point-in-time joins, and feature views. <https://docs.feast.dev/>
- **Tecton, *What is a feature platform?*** — a commercial reference architecture that is useful for cross-checking the four guarantees a feature store makes. <https://www.tecton.ai/blog/what-is-a-feature-platform/>
- **AWS SageMaker Feature Store documentation** — the managed AWS offering; useful for the online/offline dual-store semantics as documented in a production platform. <https://docs.aws.amazon.com/sagemaker/latest/dg/feature-store.html>
- **Google Cloud Vertex AI Feature Store documentation** — the managed GCP offering; the point-in-time-join semantics are documented here concretely. <https://cloud.google.com/vertex-ai/docs/featurestore>
- **Databricks Feature Store documentation** — the Lakehouse-native alternative; useful contrast for the "feature store on top of the warehouse" pattern. <https://docs.databricks.com/en/machine-learning/feature-store/index.html>

## Model registries (chapter 03)

- **MLflow Model Registry documentation** — the reference open-source model registry; stage transitions, version metadata, and the promotion contract are all here. <https://mlflow.org/docs/latest/model-registry.html>
- **AWS SageMaker Model Registry documentation.** <https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html>
- **Google Cloud Vertex AI Model Registry documentation.** <https://cloud.google.com/vertex-ai/docs/model-registry/introduction>
- **Weights & Biases Registry documentation** — the experiment-tracker-native registry, worth reading for the metadata-first view. <https://docs.wandb.ai/guides/registry>

## Batch, streaming, and the dataflow model (chapter 02)

- **Akidau et al., *The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing*** (VLDB 2015) — the canonical academic paper on the batch/streaming unification. <https://research.google/pubs/pub43864/>
- **Tyler Akidau, *The World Beyond Batch — Streaming 101 and 102*** — the two-part essay that turned the Dataflow paper into industry vocabulary. Read both. <https://www.oreilly.com/radar/the-world-beyond-batch-streaming-101/> · <https://www.oreilly.com/radar/the-world-beyond-batch-streaming-102/>
- **Kafka Streams documentation** — the state-management concepts (KTables, state stores, changelog topics) that make streaming feature stores possible and expensive. <https://kafka.apache.org/documentation/streams/>
- **Apache Flink documentation** — the alternative streaming engine used across the industry for streaming feature computation. <https://nightlies.apache.org/flink/flink-docs-stable/>
- **Apache Beam programming guide** — the batch/streaming-unified programming model derived from the Dataflow paper. <https://beam.apache.org/documentation/programming-guide/>

## Training/serving skew and data validation (chapter 04, exercise-03)

- **TensorFlow Data Validation (TFDV) — Getting started guide** — the Google-native tooling for schema drift, distribution comparison, and anomaly detection between training and serving data. <https://www.tensorflow.org/tfx/data_validation/get_started>
- **Great Expectations documentation** — the framework-agnostic alternative for data-quality checks in the training and serving paths. <https://greatexpectations.io/>
- **Sculley et al., *Machine Learning: The High-Interest Credit Card of Technical Debt*** (SE4ML workshop, NeurIPS 2014) — the precursor to *Hidden Technical Debt*; the "CACE" (Change Anything, Change Everything) principle is the L20 → L30 warning. <https://research.google/pubs/pub43146/>

## Retraining, rollout, canaries, and SRE (chapter 05)

- **Google, *Site Reliability Engineering*** (free book) — chapter 8 (*Release Engineering*) is the canonical reference for canary analysis and rollout strategy. Referenced heavily in chapter 05 and again in [mod-307]. <https://sre.google/sre-book/table-of-contents/>
- **Google, *The Site Reliability Workbook*** — companion volume with worked SLO and error-budget examples. <https://sre.google/workbook/table-of-contents/>
- **Ron Kohavi, Diane Tang, Ya Xu, *Trustworthy Online Controlled Experiments*** (Cambridge, 2020) — the canary and A/B guardrails vocabulary. Cited more heavily in [mod-306], but chapter 5's "abort conditions have to be pre-registered" is the same discipline. Publisher: <https://www.cambridge.org/core/books/trustworthy-online-controlled-experiments/D97B26382EB0EB2DC2019A7A7B518F59> · Companion site: <https://experimentguide.com/>.

## RFCs, design docs, and reviewer craft (chapter 06)

- **Google, *Engineering Practices Documentation*** — the industry-canonical description of the design-doc and code-review bar a senior engineer is expected to hold. <https://google.github.io/eng-practices/>
- **Malte Ubl, *Design Docs at Google*** (2020) — the specific shape of an approved design doc, including the supersede-not-mutate discipline used in chapter 06. <https://www.industrialempathy.com/posts/design-docs-at-google/>
- **IETF, *RFC 7322 — RFC Style Guide*** — the older cousin of the modern engineering RFC; useful for the "structured, defensible, immutable-once-approved" mental model. <https://www.rfc-editor.org/info/rfc7322>
- **Will Larson, *Introducing Architecture Review at Stripe*** (2019) — how an architecture-review programme is actually run. Adjacent to the reviewer craft in chapter 06. <https://lethain.com/introducing-architecture-review-at-stripe/>

## Applied MLOps courses (optional but useful supplements)

- **DeepLearning.AI / Coursera, *Machine Learning Engineering for Production (MLOps) Specialization*** — a well-scoped applied companion, especially the deployment and monitoring courses. <https://www.coursera.org/specializations/machine-learning-engineering-for-production-mlops>
- **Made With ML, *MLOps Course*** — a free applied course; treats the same lifecycle from a different angle. <https://madewithml.com/>

## Cloud certifications (what the industry hires against)

Cited for the practical requirements catalogue only, not as a curriculum substitute.

- **Google Cloud, *Professional Machine Learning Engineer* — exam guide.** <https://cloud.google.com/learn/certification/machine-learning-engineer>
- **AWS Certified Machine Learning Engineer – Associate — exam guide.** <https://aws.amazon.com/certification/certified-machine-learning-engineer-associate/>
- **AWS Certified Machine Learning – Specialty — exam guide** (predecessor exam still in wide use). <https://aws.amazon.com/certification/certified-machine-learning-specialty/>

## Peer-track linkage

The chapter cross-references above use track slugs from the wider AICG curriculum ecosystem. Read the READMEs of the peer tracks referenced from this module at least once so that the hand-off contracts are concrete:

- Peer platform (paved road): `ai-infra-ml-platform-learning`, `ai-infra-mlops-learning`, `training-pipeline-engineer-learning`, `model-evaluation-engineer-learning`, `ai-eval-engineer-learning`.
- Peer specialist (delegation): `llm-application-developer-learning`, `rag-engineer-learning`, `fine-tuning-engineer-learning`, `applied-ai-engineer-learning`.
- Review counterparts: `ai-infra-security-learning`, `ai-governance-analyst-learning`.
- Prerequisite: `ml-engineer-learning` (L20).
- Higher-level: `staff-ml-engineer-learning`, `principal-ml-engineer-learning`.

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->
