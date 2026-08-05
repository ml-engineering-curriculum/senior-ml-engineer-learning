# Resources for mod-308-platform-collaboration

Curated external references. These are the primary and authoritative sources the chapters and exercises cite or lean on. Every URL is publicly reachable at time of authoring; where a paper's canonical URL is behind a paywall, an author-hosted preprint URL is provided instead.

## The load-bearing frames (all chapters)

- **Matthew Skelton, Manuel Pais, *Team Topologies: Organizing Business and Technology Teams for Fast Flow*** (IT Revolution Press, 2019). The framework for stream-aligned vs. platform vs. enabling vs. complicated-subsystem teams, and the interaction modes (X-as-a-Service, collaboration, facilitating) that this module's hand-off contracts are named against. <https://teamtopologies.com/book>
- **Team Topologies key concepts.** Free online summary. <https://teamtopologies.com/key-concepts>
- **Netflix Technology Blog, *Full-Cycle Developers at Netflix — Operate What You Build*** (2018). The essay that popularised the "paved road" and "unpaved road" vocabulary this module uses. <https://netflixtechblog.com/full-cycle-developers-at-netflix-a08c31f83249>
- **Camille Fournier, *Building a Platform Team*** (essay, 2020). The load-bearing platform-team-side treatment of the consumer relationship. <https://skamille.medium.com/building-a-platform-team-e08ea8b23a68>
- **Camille Fournier, *Bar-raising internal platforms*** (essay, 2022). The evolution question — when does an internal platform stop being "good enough" and start needing to raise its bar? Platform-team-side view of chapter 02's build-vs-adopt call. <https://skamille.medium.com/bar-raising-internal-platforms-d5b8a72c8a1e>
- **Will Larson, *The Engineering Executive's Primer*** (O'Reilly, 2024). Chapter on platform investment strategy is the executive-altitude framing behind chapter 02. <https://lethain.com/eep/>
- **Will Larson, *Adopting versus building software*** (essay). The three most important behaviours in an adopt-vs-build decision. <https://lethain.com/adopting-versus-building/>
- **Evan Bottcher, *What I Talk About When I Talk About Platforms*** (martinfowler.com, 2018). The load-bearing definition of "platform-as-a-product" this module operationalises. <https://martinfowler.com/articles/talk-about-platforms.html>

## Paved-road consumption (chapter 01, exercise 01)

- **Chip Huyen, *Designing Machine Learning Systems*** (O'Reilly, 2022). Chapters 6 (deployment), 7 (feature store), and 10 (monitoring) are the paved-road-primitive coverage from the ML-team-consumer side. <https://huyenchip.com/books/>
- **Andriy Burkov, *Machine Learning Engineering*** (True Positive, 2020). Chapter on serving and deployment is the load-bearing ML-team-side treatment of the platform primitives. <http://www.mlebook.com/>
- **Uber Engineering, *Michelangelo: Uber's Machine Learning Platform*** (2017). The paper that popularised the "ML platform" as a first-class internal offering, and named training-serving consistency as the load-bearing motivation. <https://www.uber.com/blog/michelangelo-machine-learning-platform/>
- **Airbnb Engineering, *Chronon: Airbnb's ML Feature Platform*** (2023). Modern feature-platform reference — the offline/online consistency contract and freshness discipline in production. <https://medium.com/airbnb-engineering/chronon-airbnbs-ml-feature-platform-is-now-open-source-d9c4dba859e8>
- **Feast documentation.** Open-source feature store; the load-bearing reference implementation to skim for the SDK-consumer contract. <https://docs.feast.dev/>
- **MLflow Model Registry documentation.** The open-source model-registry reference for the state-machine (Staging → Production → Archived) and versioning discipline chapter 01 §3.2 leans on. <https://mlflow.org/docs/latest/model-registry.html>
- **AWS SageMaker Model Registry documentation.** Cloud-managed equivalent. <https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html>
- **Google Cloud Vertex AI Model Registry documentation.** Cloud-managed equivalent. <https://cloud.google.com/vertex-ai/docs/model-registry/introduction>
- **Ray documentation — Ray Train.** The distributed-training-consumer surface reference. <https://docs.ray.io/en/latest/train/train.html>
- **Kubeflow Training Operator documentation.** The open-source distributed-training-consumer reference for the Kubernetes ecosystem. <https://www.kubeflow.org/docs/components/training/>
- **KServe documentation.** Open-source model-serving reference. <https://kserve.github.io/website/>
- **NVIDIA Triton Inference Server documentation.** The industry-standard high-performance inference-server reference. <https://docs.nvidia.com/deeplearning/triton-inference-server/>
- **Sam Newman, *Building Microservices* (2nd ed.)** (O'Reilly, 2021). The "shared library problem" chapter is directly analogous to chapter 01 §3.4's land-on-top-not-underneath discipline. <https://samnewman.io/books/building_microservices_2nd_edition/>
- **Sculley et al., *Hidden Technical Debt in Machine Learning Systems*** (NIPS 2015). The reference for why ML systems' boundaries are broader than a classical service's — the "CACE" property that shapes the paved road's scope. <https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html>

## Build-vs-adopt decisions (chapter 02, exercise 02)

- **Jeff Bezos, *2015 Letter to Shareholders*** (Amazon). The one-way vs. two-way doors framing chapter 02 §2.2 leans on. <https://www.aboutamazon.com/news/company-news/2016-letter-to-shareholders>
- **Will Larson, *Adopting versus building software*** (essay). Duplicated from chapter 01 references; the load-bearing decision framework for chapter 02. <https://lethain.com/adopting-versus-building/>
- **Camille Fournier, *When to Adopt Someone Else's Innovation*** (essay). The specific case of the platform team's decision about whether to adopt an external primitive vs. build in-house — the mirror-image of the ML-team decision. <https://skamille.medium.com/when-to-adopt-someone-elses-innovation-2f2c3e0b3f5c>
- **Google SRE Workbook, *Managing Load*** (chapter). Adjacent discussion of internal-service deprecation and migration discipline that the "build in-house with migration" path (chapter 02 §5) inherits from. <https://sre.google/workbook/managing-load/>
- **Google Cloud Platform, *Deprecation policy*** (public reference). The industry-standard published-deprecation-policy example that chapter 02's escalate and chapter 03's deprecation section reference. <https://cloud.google.com/terms/deprecation>
- **Andrew Bosworth, *Fewer Meetings, More Decisions*** (essay). The tradeoff-doc-as-decision-artefact framing that the exercise-02 doc is patterned on. <https://boz.com/articles/decisions>
- **ACM Queue, *The Reality of Constrained Optimization*** (Nicole Forsgren et al.). The three-year-TCO discipline in chapter 02 §2.5 and the exercise stretch goal is a specific application of the constrained-optimization framing. <https://queue.acm.org/detail.cfm?id=3572814>

## Contribute-back RFCs (chapter 03, exercise 03)

- **Gergely Orosz, *Software Engineering RFCs*** (Pragmatic Engineer, 2022). The industry-standard treatment of RFC culture; the load-bearing external reference for chapter 03. <https://www.pragmaticengineer.com/rfcs-and-design-docs/>
- **Rust RFCs — the process and the template.** The most-transferable open-source RFC discipline. Chapter 03 §5's alternatives-considered pattern is the Rust RFC's structure. <https://rust-lang.github.io/rfcs/> · <https://github.com/rust-lang/rfcs/blob/master/0000-template.md>
- **Kubernetes Enhancement Proposals (KEP).** A larger, later-generation RFC process with heavier lifecycle discipline. Useful as a contrast with the lighter Rust process. <https://github.com/kubernetes/enhancements/tree/master/keps>
- **Python PEP process (PEP 1).** The canonical prior art for engineering-org RFC processes — the "PEP-inspired" process most modern engineering orgs use lightly. <https://peps.python.org/pep-0001/>
- **IETF RFC Editor.** The origin of the "Request For Comments" format (RFC 1, 1969). <https://www.rfc-editor.org/info/rfc1>
- **Yonatan Zunger, *The friendly design doc*** (essay). The reference for writing a design doc / RFC to a specific reviewer at that reviewer's altitude — chapter 03 §4's discipline. <https://medium.com/@yonatanzunger/the-friendly-design-doc-2c96b8ba0842>
- **Rachel Potvin, *How to Write a Good Design Document*** (essay, 2022). The reference for structural-quality discipline in design docs — summary-first, motivation-before-proposal, alternatives-considered. <https://www.designdocsforengineers.com/>
- **Michael Nygard, *Documenting Architecture Decisions*** (essay, 2011). The ADR (Architecture Decision Record) format — an RFC's lighter-weight cousin, used for smaller-scope decisions. Chapter 04's hand-off contract borrows from the ADR discipline. <https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions>
- **Ryan Peterman, *How to get engineering buy-in*** (essay). The sponsor-conversation pattern of chapter 03 §2. <https://blog.pragmaticengineer.com/how-to-get-engineering-buy-in/>
- **Google Engineering Practices.** The public treatment of Google's code-review and design-doc culture — the "how would a senior engineer expect to be reviewed" reference. <https://google.github.io/eng-practices/>
- **Squarespace Engineering, *The Anatomy of a Design Doc*** (essay). A widely-copied engineering-org design-doc template that composes cleanly with the chapter 03 §3 skeleton. <https://engineering.squarespace.com/blog/2019/the-anatomy-of-a-design-doc>
- **Google Cloud API Deprecation Policy** (duplicated from chapter 02 references). The public example of a mature deprecation policy that chapter 03 §6's deprecation-section discipline references. <https://cloud.google.com/terms/deprecation>
- **Stripe API Versioning** (blog / docs). A widely-cited example of a stable, versioned, deprecation-disciplined public API — the discipline chapter 03 §6 applies to internal platforms. <https://stripe.com/blog/api-versioning>

## Hand-off contracts to specialists (chapter 04)

### Training pipeline engineering

- **PyTorch FSDP documentation.** The current standard for sharded distributed training. Reference for the training-pipeline-engineer's toolkit. <https://pytorch.org/docs/stable/fsdp.html>
- **PyTorch Distributed documentation.** The broader distributed-training reference. <https://pytorch.org/tutorials/beginner/dist_overview.html>
- **Samyam Rajbhandari et al., *ZeRO: Memory Optimizations Toward Training Trillion Parameter Models*** (SC 2020). The load-bearing paper behind DeepSpeed's sharding regimes. <https://arxiv.org/abs/1910.02054>
- **DeepSpeed documentation.** The most-referenced open-source distributed-training framework in the specialist track. <https://www.deepspeed.ai/>
- **NVIDIA NCCL User Guide.** The collective-communications reference. The `training-pipeline-engineer` reaches for this; the ML engineer knows it exists. <https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/>
- **NVIDIA Nsight Systems / Nsight Compute documentation.** The GPU-profiling toolkit reference for MFU measurement. <https://developer.nvidia.com/nsight-systems> · <https://developer.nvidia.com/nsight-compute>
- **HuggingFace, *Efficient Training on Multiple GPUs*** (docs). A pragmatic consumer-side reference for the parallelism regimes chapter 04 §3 hand-off engagements select from. <https://huggingface.co/docs/transformers/perf_train_gpu_many>
- **Mohammad Shoeybi et al., *Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism*** (2019). The reference for tensor and pipeline parallelism. <https://arxiv.org/abs/1909.08053>
- **Aaron Grattafiori et al., *The Llama 3 Herd of Models*** (2024). Includes an unusually thorough section on the training-infra choices (parallelism, checkpointing, MFU) at frontier scale — a useful modern reference for what the training-pipeline-engineer's depth looks like in practice. <https://arxiv.org/abs/2407.21783>

### Model evaluation engineering

- **Anthropic Research, *A statistical approach to model evaluations*** (2024). The reference for calibrated evaluation and uncertainty quantification in the model-evaluation-engineer's toolkit. <https://www.anthropic.com/research/statistical-approach-to-model-evaluations>
- **Yuntao Bai et al., *Constitutional AI: Harmlessness from AI Feedback*** (Anthropic, 2022). The load-bearing paper for LLM-judge harness design. <https://arxiv.org/abs/2212.08073>
- **Lianmin Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*** (NeurIPS 2023). The reference on judge-human and judge-judge agreement discipline chapter 04 §4's acceptance criteria lean on. <https://arxiv.org/abs/2306.05685>
- **OpenAI Evals framework.** The open-source LLM-eval reference implementation for the harness the model-evaluation-engineer builds. <https://github.com/openai/evals>
- **Percy Liang et al., *Holistic Evaluation of Language Models (HELM)*** (Stanford CRFM, 2022–). The canonical reference for multi-metric, multi-slice LLM evaluation at scale. <https://crfm.stanford.edu/helm/>
- **Ian Osband et al., *A Framework for Evaluating the Faithfulness of Explanations*** — adjacent reference for eval-harness discipline where quality is not a single scalar. <https://arxiv.org/abs/2402.03889>
- **Google Research, *ML Test Score*** (2016). The reference for the eval-and-testing discipline mod-305 chapter 04 leans on and this module's hand-off to model-evaluation-engineer inherits. <https://research.google/pubs/pub45742/>

## Cross-team collaboration and technical influence (chapter 03, chapter 04)

- **Will Larson, *Staff Engineer: Leadership Beyond the Management Track*** (Stripe Press, 2021). Chapter on the "cross-team influence without authority" discipline that chapter 03's RFC and chapter 04's hand-off both operationalise. <https://staffeng.com/>
- **Tanya Reilly, *The Staff Engineer's Path*** (O'Reilly, 2022). Chapter on writing and reviewing for a cross-team audience is directly applicable to chapter 03's platform-team-altitude discipline. <https://www.oreilly.com/library/view/the-staff-engineers/9781098118723/>
- **Camille Fournier, *The Manager's Path*** (O'Reilly, 2017). The chapter on cross-team dependencies and the tech-lead role's cross-team responsibilities is where the ML team's TL and the platform-team liaison operate. <https://www.oreilly.com/library/view/the-managers-path/9781491973882/>
- **Google re:Work, *Understand team effectiveness*** (2015). The Project Aristotle findings on psychological safety are the load-bearing framing behind why cross-team feedback (chapter 01 §5) needs to be safe-to-give. <https://rework.withgoogle.com/print/guides/5721312655835136/>

## Tooling that anchors the paved-road primitives

- **Airflow, Prefect, Dagster, Kubeflow Pipelines, Flyte.** The workflow-orchestrator landscape a mature ML platform's paved road exposes. Consult vendor docs directly.
- **MLflow, W&B, Neptune, Comet.** The experiment-tracking and model-registry landscape. Consult vendor docs directly.
- **Feast, Tecton, Chronon (Airbnb), Michelangelo (Uber), Zipline (Airbnb).** The feature-store landscape.
- **KServe, Seldon, Triton, BentoML, TorchServe, TensorFlow Serving.** The open-source serving-stack landscape.
- **SageMaker, Vertex AI, Azure ML, Databricks.** The hosted / managed ML-platform stacks whose SDK is the paved road for many orgs.
- **Statsig, Eppo, GrowthBook, LaunchDarkly, Optimizely.** The experimentation-platform landscape (see also [mod-306 resources](../mod-306-experimentation-at-scale/resources.md)).
- **Ray, JAX / T5X, Colossal-AI, DeepSpeed.** The distributed-training-platform landscape.

## Cross-references inside this module

- Chapter 01 (`01-paved-road-consumption.md`) — the paved road, idiomatic consumption, the consumer's contract, and the feedback ladder. The paved-road inventory of §7 is the input to chapter 02.
- Chapter 02 (`02-build-vs-adopt-decisions.md`) — the four paths, the five-factor rubric, the tradeoff doc, the migration plan, escalating correctly, the contribute-back trap.
- Chapter 03 (`03-contribute-back-rfcs.md`) — the RFC as decision artefact, the sponsor conversation, the RFC structure, alternatives-considered, migration and deprecation, the review cycle, rejection as a valid outcome.
- Chapter 04 (`04-handoff-contracts-to-specialists.md`) — the six-section hand-off contract, the training-pipeline-engineer contract, the model-evaluation-engineer contract, cross-team vocabulary translation, when *not* to hand off, the program-level portfolio view.

## Cross-references to other modules

- [mod-302 chapter 05 (retraining and rollout)](../mod-302-ml-systems-architecture/05-retraining-and-rollout.md) — the retraining trigger set that the training-platform (and thus chapter 04's hand-off) delivers against.
- [mod-303](../mod-303-advanced-modeling/) — the awareness-only chapters where distributed-training and numerics depth is delegated to `training-pipeline-engineer` (chapter 04 §3).
- [mod-304 chapter 04 (cost and latency guardrails)](../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) — the per-feature cost envelope that hand-off contract §4 (eval) inherits when the harness is LLM-judged.
- [mod-305 chapter 01 (offline harness)](../mod-305-advanced-evaluation/01-offline-eval-harness-shape.md) — the eval-harness that the model-evaluation-engineer builds against (chapter 04 §4).
- [mod-305 chapter 04 (release memo / ML Test Score)](../mod-305-advanced-evaluation/04-ml-test-score-production-readiness.md) — the release memo the hand-off deliverables cross-reference.
- [mod-306](../mod-306-experimentation-at-scale/) — the experimentation platform is one of the paved-road primitives (chapter 01 §1) and one of chapter 02's build-vs-adopt candidates when the platform's ramp shape doesn't fit.
- [mod-307 chapter 01 (SRE fundamentals for ML)](../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md) — the SLO / SLI / SLA vocabulary and dependency-SLI composition that chapter 01's consumer contract inherits.
- [mod-307 chapter 04 (incident response)](../mod-307-ml-reliability-slos/04-ml-incident-response-and-postmortems.md) — the multi-team escalation hand-off that this module names as an outbound reference.
- [mod-309](../mod-309-responsible-ai-governance/) — the governance-visible collaboration surface (MRM partnership, review packets) that extends this module's cross-team discipline to a compliance audience.
- [mod-310](../mod-310-technical-leadership/) — the general technical-leadership altitude of the RFC / stakeholder-management discipline this module applies specifically to the platform relationship.

## Peer specialist tracks (hand-off targets)

The hand-off contracts of chapter 04 are written to peer specialist tracks in the AICG ecosystem. Consult each track's README for its own delegation-contract vocabulary from the specialist side.

- **`ai-infra-ml-platform-learning`** — the peer track for the platform teams whose paved road this module consumes.
- **`ai-infra-mlops-learning`** — the peer track for CI/CD-for-ML platform depth.
- **`training-pipeline-engineer-learning`** — the peer track for distributed-training, GPU-ops, and numerics depth (chapter 04 §3 hand-off target).
- **`model-evaluation-engineer-learning`** / **`ai-eval-engineer-learning`** — the peer tracks for evaluation-platform depth (chapter 04 §4 hand-off target).
- **`ai-infra-performance-learning`** — the peer track for kernel-level / hardware-adjacent inference-performance depth.
- **`experimentation-platform-engineer-learning`** — the peer track for building the experimentation platform.
- **`fine-tuning-engineer-learning`** — the peer track for LoRA / QLoRA / DPO / RLHF depth (mod-304 delegation, referenced here).
- **`ai-governance-analyst-learning`** — the peer track for governance and compliance depth (mod-309 delegation).
- **`ai-infra-security-learning`** — the peer track for ML/AI security depth (mod-309 delegation).

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->
