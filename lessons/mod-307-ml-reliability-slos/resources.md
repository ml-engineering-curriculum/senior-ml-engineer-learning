# Resources for mod-307-ml-reliability-slos (ML Reliability: SLOs, Cost Budgets, and Incident Response)

Curated external references. These are the primary and authoritative sources the chapters and exercises above cite or lean on. Every URL is publicly reachable at the time of authoring; where a paper's canonical URL is behind a paywall, a preprint or author-hosted URL is provided instead.

## The load-bearing SRE canon (all chapters, all exercises)

- **Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (eds.), *Site Reliability Engineering: How Google Runs Production Systems*** (O'Reilly, 2016). The canonical SRE text. Chapters on SLIs/SLOs, embracing risk, managing incidents, postmortem culture, and handling overload map directly onto this module. Freely available online. <https://sre.google/sre-book/table-of-contents/>
- **Betsy Beyer, Niall Richard Murphy, David K. Rensin, Kent Kawahara, Stephen Thorne (eds.), *The Site Reliability Workbook: Practical Ways to Implement SRE*** (O'Reilly, 2018). The companion volume; the SLO-implementation, error-budget-policy, and alerting-on-SLOs chapters are the load-bearing operational references. Freely available online. <https://sre.google/workbook/table-of-contents/>

## SLI, SLO, error-budget vocabulary (chapter 01, exercise 01)

- **Chris Jones, John Wilkes, Niall Murphy, Cody Smith, *Service Level Objectives*** (SRE book, ch. 4). The chapter that defines SLI / SLO / SLA and the good-events/valid-events shape. <https://sre.google/sre-book/service-level-objectives/>
- **Marc Alvidrez, *Embracing Risk*** (SRE book, ch. 3). Origin of the error-budget-as-shipping-lever framing. <https://sre.google/sre-book/embracing-risk/>
- **Steven Thurgood, David Ferguson, Alec Warner, Anthony Lenton, *Implementing SLOs*** (SRE Workbook, ch. 2). The operational chapter that turns the SLO concept into a document, a budget, and an alerting posture. <https://sre.google/workbook/implementing-slos/>
- **Betsy Beyer, Kent Kawahara, Stephen Thorne, *Alerting on SLOs*** (SRE Workbook, ch. 5). The reference for multi-window, multi-burn-rate alerting. <https://sre.google/workbook/alerting-on-slos/>
- **Alec Warner, Vivek Rau, *Error Budget Policies*** (SRE Workbook, appendix). The policy-as-repo-artefact shape the SLO-document exercise leans on. <https://sre.google/workbook/error-budget-policy/>
- **Gil Tene, *How NOT to Measure Latency*** (Strange Loop 2015). The reference on coordinated omission and why every latency SLI is measured on the tail, not the mean. <https://www.infoq.com/presentations/latency-response-time/>
- **D. Sculley, Gary Holt, Daniel Golovin, Eugene Davydov, Todd Phillips, Dietmar Ebner, Vinay Chaudhary, Michael Young, Jean-François Crespo, Dan Dennison, *Hidden Technical Debt in Machine Learning Systems*** (NeurIPS 2015). The "CACE" (Changing Anything Changes Everything) property that motivates the ML-specific SLI extensions. <https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html>

## ML-specific SLIs — freshness, quality, calibration, drift (chapter 02, exercise 01)

### Textbook and applied references

- **Chip Huyen, *Designing Machine Learning Systems: An Iterative Process for Production-Ready Applications*** (O'Reilly, 2022). Chapter 8 ("Data Distribution Shifts and Monitoring") covers freshness, prediction drift, and drift-detection at the practitioner level. <https://huyenchip.com/books/> · <https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/>
- **Emmanuel Ameisen, *Building Machine Learning Powered Applications: Going from Idea to Product*** (O'Reilly, 2020). The operational shape of pairing predictions with delayed labels for online quality metrics. <https://www.oreilly.com/library/view/building-machine-learning/9781492045106/>
- **Uber Engineering, *Meet Michelangelo: Uber's Machine Learning Platform*** and follow-up posts. Industry write-up of feature-staleness monitoring and the model-registry state machine at scale. <https://www.uber.com/blog/michelangelo-machine-learning-platform/>

### Calibration

- **Chuan Guo, Geoff Pleiss, Yu Sun, Kilian Q. Weinberger, *On Calibration of Modern Neural Networks*** (ICML 2017). The modern reference on ECE and post-hoc calibration for deep classifiers. <https://arxiv.org/abs/1706.04599>
- **Ananya Kumar, Percy Liang, Tengyu Ma, *Verified Uncertainty Calibration*** (NeurIPS 2019). ECE variants and their failure modes; the reference for the "which ECE variant is your SLI actually on" question. <https://arxiv.org/abs/1909.10155>

### Distribution and prediction drift

- **Stephan Rabanser, Stephan Günnemann, Zachary C. Lipton, *Failing Loudly: An Empirical Study of Methods for Detecting Dataset Shift*** (NeurIPS 2019). The load-bearing empirical evaluation of drift-detection methods; explains why some detectors work and others do not on realistic shifts. <https://arxiv.org/abs/1810.11953>
- **NannyML documentation — *Drift detection under the hood***. Practitioner reference for PSI/KS/Chi-squared drift detectors and their handling of new categorical values. <https://nannyml.readthedocs.io/en/stable/how_it_works/drift_detection.html>
- **OpenScoring, *Population Stability Index***. Concise reference on the credit-scoring-era PSI rules of thumb (0.1 moderate / 0.2 significant). <https://github.com/openscoring/openscoring/wiki/psi>

### Feature stores and TTL / freshness semantics

- **Feast documentation — feature-view TTL and materialization**. The open-source feature-store reference implementation whose semantics the freshness SLI is named against. <https://docs.feast.dev/>
- **Tecton documentation — data freshness**. Managed feature-store reference; complements Feast and covers streaming-vs-batch freshness contracts. <https://docs.tecton.ai/>
- **Hopsworks Feature Store documentation**. Alternative open-source feature store with version-pinning primitives the retraining chapter's rollback discipline relies on. <https://www.hopsworks.ai/>
- **Databricks Feature Store documentation**. The Databricks-native feature-store; feature-view versioning and lineage. <https://docs.databricks.com/en/machine-learning/feature-store/index.html>

<!-- needs-research: chapter 02 §3 cites "Reifman, Feldman, Cohen — Distribution shifts in machine learning: A survey" at arxiv 2005.12852. Author list and title should be confirmed against the arxiv record before this reference is added to the resource list. -->

## Cost budgets and enforcement (chapter 03, exercise 02)

### Framework and industry references

- **FinOps Foundation, *The FinOps Framework***. The industry reference for cloud-native cost management practices; the capability model chapter 03 leans on for the reporting-vs-enforcement distinction. <https://www.finops.org/framework/>
- **Betsy Beyer, David N. Blank-Edelman, Christophe Hulton, *Handling Overload*** (SRE book, ch. 21). Load-shedding, quotas, and graceful-degradation as reliability mechanisms. <https://sre.google/sre-book/handling-overload/>
- **Chris Jones, Todd Underwood, Shylaja Nukala, *Data Processing Pipelines*** (SRE book, ch. 25). Reliability discipline for batch pipelines; complements chapter 03's priority-queue and shedding treatment. <https://sre.google/sre-book/data-processing-pipelines/>

### Circuit breakers and resilience libraries

- **Michael Nygard, *Release It! Design and Deploy Production-Ready Software* (2nd ed.)** (Pragmatic Bookshelf, 2018). The load-bearing reference on circuit breakers, bulkheads, and stability patterns. <https://pragprog.com/titles/mnee2/release-it-second-edition/>
- **Netflix Hystrix**. The historically canonical circuit-breaker library (now in maintenance mode, but its documentation remains the reference for the pattern's shape). <https://github.com/Netflix/Hystrix>
- **resilience4j documentation**. Modern JVM circuit-breaker / rate-limiter library; the practical successor to Hystrix. <https://resilience4j.readme.io/>
- **Alibaba Sentinel documentation**. Alternative modern circuit-breaker / traffic-shaping library. <https://sentinelguard.io/en-us/>

### Cloud-provider cost-management tooling

- **AWS Cost Anomaly Detection**. The AWS-side reporting layer that complements in-service enforcement. <https://aws.amazon.com/aws-cost-management/aws-cost-anomaly-detection/>
- **Google Cloud Budgets and alerts**. GCP-side budget-alert reference. <https://cloud.google.com/billing/docs/how-to/budgets>
- **Microsoft Cost Management + Billing**. Azure-side reference. <https://learn.microsoft.com/en-us/azure/cost-management-billing/>
- **Anthropic API — usage and pricing**. The vendor-side usage-field / pricing pages the per-request cost estimator reads from. <https://docs.anthropic.com/en/api/messages> · <https://www.anthropic.com/pricing>

## Incident response, runbooks, postmortems (chapter 04, exercise 03)

### The classical incident-response canon

- **Andrew Widdowson, *Managing Incidents*** (SRE book, ch. 14). Roles (IC / Ops / Comms), cadence, mitigate-first discipline. <https://sre.google/sre-book/managing-incidents/>
- **John Lunney, Sue Lueder, Gary O'Connor, *Postmortem Culture: Learning from Failure*** (SRE book, ch. 15). The blameless-postmortem discipline; template shape and review-forum expectations. <https://sre.google/sre-book/postmortem-culture/>
- **PagerDuty, *Incident Response Documentation***. Industry-standard reference for the on-call role structure and severity ladder; freely available. <https://response.pagerduty.com/>
- **Etsy Engineering, *Debriefing Facilitation Guide***. The applied reference for facilitating a blameless retrospective. <https://extfiles.etsy.com/DebriefingFacilitationGuide.pdf>
- **John Allspaw, *The Infinite Hows (or, the Dangers of the Five Whys)*** (ACM Queue, 2012). The load-bearing read on why "5 whys" is not the right postmortem tool and what to use instead. <https://queue.acm.org/detail.cfm?id=2353017>
- **Richard I. Cook, *How Complex Systems Fail*** (Cognitive Technologies Laboratory, 1998). The 18-thesis paper on the nature of failure in complex systems; the theoretical grounding for chapter 04's "there is always more than one cause" property. <https://how.complexsystems.fail/>

### Gamedays and chaos engineering

- **Netflix, *Chaos Monkey* and the Simian Army***. The canonical chaos-engineering tool suite; the industrial origin of the gameday discipline. <https://netflix.github.io/chaosmonkey/>
- **Principles of Chaos Engineering**. The manifesto / practitioner reference. <https://principlesofchaos.org/>
- **Betsy Beyer, Aaron Joyner, Adrian Hilton, *Disaster Recovery*** (SRE Workbook, ch. 9). Discipline of disaster-recovery drills; the reference for the rollback-rehearsal cadence. <https://sre.google/workbook/disaster-recovery/>

### Feature-flag and rollback-machinery tooling

- **Google Cloud, *Feature flags at scale***. The operational shape of feature-flag-driven rollout and rollback. <https://cloud.google.com/blog/products/devops-sre/feature-flags-at-scale-google>
- **LaunchDarkly, Flipt, Unleash, ConfigCat**. Common feature-flag / config-store vendor implementations; the rollback-machinery references consult vendor docs directly. <https://launchdarkly.com/> · <https://www.flipt.io/> · <https://www.getunleash.io/> · <https://configcat.com/>

## Retraining as a first-class deploy (chapter 05, exercise 04)

- **MLflow Model Registry documentation**. The reference implementation of the model-registry state machine (`None` → `Staging` → `Production` → `Archived`); the concrete tooling behind chapter 05 §1. <https://mlflow.org/docs/latest/model-registry.html>
- **Alec Warner, Bora Beran, Steven Thurgood, *Canarying Releases*** (SRE Workbook, ch. 16). The ladder-length trade-off and the canary-vs-rollback machinery. <https://sre.google/workbook/canarying-releases/>
- **Google Vertex AI, *Model Registry* documentation**. Managed-registry reference with an approximately-equivalent state machine. <https://cloud.google.com/vertex-ai/docs/model-registry/introduction>
- **AWS SageMaker Model Registry documentation**. AWS-native equivalent. <https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html>
- **Weights & Biases Model Registry documentation**. Managed-registry reference with a lineage-first shape. <https://docs.wandb.ai/guides/models>
- **Kubeflow Pipelines documentation**. Open-source pipeline-orchestration reference for the retraining pipeline as a first-class deployable. <https://www.kubeflow.org/docs/components/pipelines/>
- **BentoML documentation**. Model-packaging and deployment reference; the "model artefact plus serving contract" shape. <https://docs.bentoml.com/>

## Cross-references inside this module

- Chapter 01 (`01-sre-fundamentals-for-ml.md`) establishes the classical SRE vocabulary (SLI, SLO, error budget, burn rate) and the four assumptions that break for ML.
- Chapter 02 (`02-ml-specific-slis-and-slos.md`) authors the ML-specific SLI menu (freshness, quality drift, calibration, distribution drift, retraining SLA) on top of the classical floor.
- Chapter 03 (`03-cost-budgets-and-quotas.md`) authors the enforced-cost discipline — hierarchy, primitives, degradation paths — alongside the SLO framework.
- Chapter 04 (`04-ml-incident-response-and-postmortems.md`) authors the ML-adapted incident taxonomy, runbooks, rollback machinery, and postmortem template.
- Chapter 05 (`05-retraining-as-a-first-class-deploy.md`) authors the model-registry state machine, the retraining canary shape, and the rollback-across-four-surfaces discipline.

## Cross-references to other modules

- [mod-302 chapter 02](../mod-302-ml-systems-architecture/) (batch/streaming/online) supplies the pipeline-architecture vocabulary chapter 02's freshness SLI reads against.
- [mod-302 chapter 03](../mod-302-ml-systems-architecture/03-feature-store-and-online-features.md) is the feature-store chapter whose TTL semantics the freshness SLI enforces.
- [mod-302 chapter 04](../mod-302-ml-systems-architecture/04-training-serving-skew.md) is the training-serving skew chapter whose failure modes chapter 04's postmortem template asks about.
- [mod-302 chapter 05](../mod-302-ml-systems-architecture/05-retraining-and-rollout.md) is the sibling architectural chapter to chapter 05 — the retraining triggers and rough rollout tools this module turns into a reliability-gated deploy path.
- [mod-303 chapter 05 (calibration)](../mod-303-advanced-modeling/05-calibration.md) is the calibration-theory chapter whose operational monitoring becomes chapter 02 §3's calibration SLI.
- [mod-304 chapter 04](../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) is the per-feature LLM cost / latency envelope that chapter 03's aggregate cost budget builds on.
- [mod-304 chapter 05](../mod-304-production-llm-integration/05-observability-for-llm-augmented-systems.md) is the LLM-specific observability layer chapter 04's runbooks read from.
- [mod-305 chapter 01](../mod-305-advanced-evaluation/01-offline-evaluation-harness.md) is the offline-eval harness whose `passes_release_gate` is the Training → Evaluated transition gate in chapter 05.
- [mod-305 chapter 02](../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) is the slice matrix chapter 02's per-slice quality SLI and chapter 05's slice-level canary check inherit.
- [mod-305 chapter 03](../mod-305-advanced-evaluation/03-shadow-and-canary-online-promotion.md) is the online promotion path whose auto-rollback controller is named against the SLIs authored here.
- [mod-306 chapter 01](../mod-306-experimentation-at-scale/01-experiment-design-power-mde-and-ramp.md) is where guardrail metrics for online experiments are named against the SLIs this module authors.
- [mod-308](../mod-308-platform-collaboration/) is where the multi-team escalation contract (who pages whom, on which SLI, under which severity) is authored.
- [mod-309](../mod-309-responsible-ai-governance/) is where fairness incidents (chapter 04 Category F) and adversarial patterns get their policy-level review.

## Peer specialist tracks (delegation targets)

Where the SRE platform itself, the ML-platform tooling, or the depth of a specific reliability technique exceeds a senior ML engineer's day-to-day, the AICG peer tracks below are the delegation targets. Consult the READMEs of each for the delegation-contract vocabulary.

- `ai-infra-sre-learning` — the SRE / platform-reliability track where the classical SLO / error-budget / incident-response discipline is authored at depth.
- `ai-infra-mlops-learning` — the MLOps / feature-store / model-registry platform this module's SLIs and rollback machinery run on.
- `ai-infra-ml-platform-learning` — the ML-platform layer that owns the model registry, retraining pipeline orchestrator, and the flag store the one-action rollback property depends on.
- `data-engineering-learning` — depth on the upstream data pipelines whose lateness the freshness SLI catches.

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->
