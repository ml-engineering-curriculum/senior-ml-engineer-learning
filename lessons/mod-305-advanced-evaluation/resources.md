# Resources for mod-305-advanced-evaluation

Curated external references. These are the primary and authoritative sources cited or leaned on by the chapters and exercises above. Every URL is publicly reachable at the time of authoring.

## The load-bearing rubric — Google ML Test Score (chapter 04, exercise 03)

- **Eric Breck, Shanqing Cai, Eric Nielsen, Michael Salib, D. Sculley, *The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction*** (IEEE BigData 2017). The 28-test rubric across data, model, ML infrastructure, and monitoring. Chapter 04 is a review-oriented reading of this paper. <https://research.google/pubs/the-ml-test-score-a-rubric-for-ml-production-readiness-and-technical-debt-reduction/>
- **D. Sculley, Gary Holt, Daniel Golovin, Eugene Davydov, Todd Phillips, Dietmar Ebner, Vinay Chaudhary, Michael Young, Jean-François Crespo, Dan Dennison, *Hidden Technical Debt in Machine Learning Systems*** (NeurIPS 2015). The paper the ML Test Score is the follow-up to. Names training–serving skew, feature debt, and configuration debt in the terms chapter 04 uses. <https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf>

## Statistical inference for offline eval (chapter 01, exercise 01)

- **Bradley Efron, Robert J. Tibshirani, *An Introduction to the Bootstrap*** (Chapman & Hall, 1993). The canonical reference on bootstrap methods used by chapter 01's paired CI framing. <https://www.routledge.com/An-Introduction-to-the-Bootstrap/Efron-Tibshirani/p/book/9780412042317>
- **Quinn McNemar, *Note on the sampling error of the difference between correlated proportions or percentages*** (Psychometrika, 1947) — the origin of the paired discrete-outcome test chapter 01 cites. <!-- needs-research: verify a stable public URL for the McNemar 1947 paper; the standard psychometric-journal URL is behind a paywall. -->
- **Jesse Frederik and Maurits Martijn, and countless others, on the *garden of forking paths* / metrics shopping.** Chapter 01's "primary metric is chosen before the candidate exists" rule has a broader statistical-discipline background; Andrew Gelman's [*The garden of forking paths*](http://www.stat.columbia.edu/~gelman/research/unpublished/p_hacking.pdf) is a widely-cited exposition.

## Slice and adversarial evaluation (chapter 02, exercise 01)

- **Joy Buolamwini, Timnit Gebru, *Gender Shades: Intersectional Accuracy Disparities in Commercial Gender Classification*** (FAT* 2018) — canonical demonstration that aggregate accuracy hides sub-population failure. <http://proceedings.mlr.press/v81/buolamwini18a/buolamwini18a.pdf>
- **Shiori Sagawa, Pang Wei Koh, Tatsunori B. Hashimoto, Percy Liang, *Distributionally Robust Neural Networks for Group Shifts*** (ICLR 2020). Worst-group generalisation. <https://arxiv.org/abs/1911.08731>
- **Yeounoh Chung, Tim Kraska, Neoklis Polyzotis, Ki Hyun Tae, Steven Euijong Whang, *Slice Finder: Automated Data Slicing for Model Validation*** (ICDE 2019). Representative automated slice-discovery method. <https://arxiv.org/abs/1807.06068>
- **Sabri Eyuboglu, Maya Varma, Khaled Saab, Jean-Benoit Delbrouck, Christopher Lee-Messer, Jared Dunnmon, James Zou, Christopher Ré, *Domino: Discovering Systematic Errors with Cross-Modal Embeddings*** (ICLR 2022). Modern automated slice discovery over embedding space. <https://arxiv.org/abs/2203.14960>
- **Marco Tulio Ribeiro, Tongshuang Wu, Carlos Guestrin, Sameer Singh, *Beyond Accuracy: Behavioral Testing of NLP Models with CheckList*** (ACL 2020, best paper). Invariance, directional-expectation, and minimum-functionality tests. <https://aclanthology.org/2020.acl-main.442/>
- **Dan Hendrycks, Thomas Dietterich, *Benchmarking Neural Network Robustness to Common Corruptions and Perturbations*** (ICLR 2019). Canonical robustness benchmark methodology. <https://arxiv.org/abs/1903.12261>
- **Andrew Ng, *Machine Learning Yearning*** (draft book, freely available). The classical error-analysis discipline chapter 02's error-concentration slice tier is a modern restatement of. <https://www.deeplearning.ai/machine-learning-yearning/>

## Fairness metrics and disaggregation (chapter 02, chapter 04 model test 7)

- **Alexandra Chouldechova, *Fair prediction with disparate impact: A study of bias in recidivism prediction instruments*** (Big Data journal, 2017). Names the impossibility results between fairness definitions. <https://arxiv.org/abs/1610.07524>
- **Moritz Hardt, Eric Price, Nathan Srebro, *Equality of Opportunity in Supervised Learning*** (NeurIPS 2016). Equal opportunity as a disaggregation metric. <https://arxiv.org/abs/1610.02413>
- **Solon Barocas, Moritz Hardt, Arvind Narayanan, *Fairness and Machine Learning: Limitations and Opportunities*** (freely available textbook). The modern synthesis. <https://fairmlbook.org/>

## Shadow, canary, A/B, interleaving (chapter 03, exercise 02)

- **Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (eds.), *Site Reliability Engineering*** (O'Reilly, 2016). The SLI / SLO vocabulary that the auto-rollback controller thresholds are named against. Freely available online. <https://sre.google/sre-book/table-of-contents/>
- **Betsy Beyer, Niall Richard Murphy, David K. Rensin, Kent Kawahara, Stephen Thorne (eds.), *The Site Reliability Workbook*** (O'Reilly, 2018). The canarying-releases chapter is the direct reference for chapter 03 §2. <https://sre.google/workbook/canarying-releases/>
- **Michael T. Nygard, *Release It! Second Edition*** (Pragmatic Bookshelf, 2018). Circuit breaker, bulkhead, and resilience patterns for auto-rollback controllers. <https://pragprog.com/titles/mnee2/release-it-second-edition/>
- **Gil Tene, *How NOT to Measure Latency*** (InfoQ). Coordinated-omission bias — the reason offline latency measurements systematically under-report. <https://www.infoq.com/presentations/latency-response-time/>
- **Ron Kohavi, Diane Tang, Ya Xu, *Trustworthy Online Controlled Experiments*** (Cambridge University Press, 2020). Modern textbook on A/B testing at scale. <https://experimentguide.com/>
- **Filip Radlinski, Madhu Kurup, Thorsten Joachims, *How Does Clickthrough Data Reflect Retrieval Quality?*** (CIKM 2008). Classical interleaving reference. <https://www.cs.cornell.edu/people/tj/publications/radlinski_etal_08b.pdf>
- **Olivier Chapelle, Thorsten Joachims, Filip Radlinski, Yisong Yue, *Large-scale Validation and Analysis of Interleaved Search Evaluation*** (ACM TOIS 2012). Industrial validation of interleaving for search ranker evaluation. <https://www.cs.cornell.edu/people/tj/publications/chapelle_etal_12a.pdf>
- **Alexander Deng, Ya Xu, Ron Kohavi, Toby Walker, *Improving the sensitivity of online controlled experiments by utilizing pre-experiment data*** (WSDM 2013). CUPED — variance reduction for A/B testing. <https://www.exp-platform.com/Documents/2013-02-CUPED-ImprovingSensitivityOfControlledExperiments.pdf>
- **Joeran Beel, Marcel Genzmehr, Stefan Langer, Andreas Nürnberger, Bela Gipp, *A Comparative Analysis of Offline and Online Evaluations and Discussion of Research Paper Recommender System Evaluation*** (RepSys 2013). Widely cited demonstration that offline and online recommender evaluation can disagree. <https://arxiv.org/abs/1508.04808>

## LLM-as-judge and its failure modes (chapter 05, exercise 04)

- **Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric P. Xing, Hao Zhang, Joseph E. Gonzalez, Ion Stoica, *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*** (NeurIPS 2023). The canonical modern reference. Position and verbosity bias, swap consistency, self-enhancement bias. <https://arxiv.org/abs/2306.05685>
- **Peiyi Wang, Lei Li, Liang Chen, Zefan Cai, Dawei Zhu, Binghuai Lin, Yunbo Cao, Qi Liu, Tianyu Liu, Zhifang Sui, *Large Language Models are not Fair Evaluators*** (ACL 2024). Focused study of position bias. <https://arxiv.org/abs/2305.17926>
- **Arjun Panickssery, Samuel R. Bowman, Shi Feng, *LLM Evaluators Recognize and Favor Their Own Generations*** (2024). Self-preference / self-enhancement bias. <https://arxiv.org/abs/2404.13076>
- **Minghao Wu, Alham Fikri Aji, *Style Over Substance: Evaluation Biases for Large Language Models*** (2023). Style-over-substance bias across judge families. <https://arxiv.org/abs/2307.03025>
- **Seungone Kim, Jamin Shin, Yejin Cho, Joel Jang, Shayne Longpre, Hwaran Lee, Sangdoo Yun, Seongjin Shin, Sungdong Kim, James Thorne, Minjoon Seo, *Prometheus: Inducing Fine-grained Evaluation Capability in Language Models*** (ICLR 2024). Open-source, fine-tuned judge model with rubric-scoring capability. <https://arxiv.org/abs/2310.08491>
- **Nathan Lambert, Valentina Pyatkin, Jacob Morrison, LJ Miranda, Bill Yuchen Lin, Khyathi Chandu, Nouha Dziri, Sachin Kumar, Tom Zick, Yejin Choi, Noah A. Smith, Hannaneh Hajishirzi, *RewardBench: Evaluating Reward Models for Language Modeling*** (NAACL 2024). Benchmark for reward / judge models. <https://arxiv.org/abs/2403.13787>
- **Cheng-Han Chiang, Hung-yi Lee, *Can Large Language Models Be an Alternative to Human Evaluation?*** (ACL 2023). Direct empirical study of judge-vs-human agreement. <https://arxiv.org/abs/2305.01937>
- **Databricks, *LLM Auto-Eval Best Practices for RAG Applications*** — production-oriented tutorial on LLM-as-judge for RAG groundedness. <https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG>

## Data validation and feature-code testing (chapter 04 data tests)

- **TensorFlow Data Validation (TFDV) — statistical data-quality validation for ML pipelines.** <https://www.tensorflow.org/tfx/data_validation/get_started>
- **Great Expectations — open-source data-quality validation.** <https://greatexpectations.io/>
- **Amazon Deequ — open-source data-quality library for Spark.** <https://github.com/awslabs/deequ>
- **Emre Kıcıman, Robert Nichol, Neoklis Polyzotis, Sudip Roy, Steven Whang, Martin Zinkevich, *Data Validation for Machine Learning*** (MLSys 2019). The paper behind the TFDV design. <https://mlsys.org/Conferences/2019/doc/2019/167.pdf>

## Governance and safety inputs (chapters 02, 04, 05; exercises 03, 04)

- **NIST, *AI Risk Management Framework — Generative AI Profile* (NIST AI 600-1).** <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/NIST_AI_600-1.pdf>
- **NIST, *AI Risk Management Framework 1.0* (NIST AI 100-1).** <https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf>
- **OWASP, *Top 10 for Large Language Model Applications*.** Starting taxonomy for the safety-and-refusal adversarial suite for LLM-augmented systems. <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **Google, *People + AI Research (PAIR) — Model Cards*.** Foundation for the model-card artifact mod-309 authors and chapter 04 leans on. <https://modelcards.withgoogle.com/about>

## Canonical texts (all chapters)

- **Chip Huyen, *Designing Machine Learning Systems*** (O'Reilly, 2022). Chapters on evaluation, monitoring, and deployment are the classical framing this module extends. <https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/> · <https://huyenchip.com/books/>
- **Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (eds.), *Site Reliability Engineering*** (O'Reilly, 2016). <https://sre.google/sre-book/table-of-contents/>
- **Betsy Beyer, Niall Richard Murphy, David K. Rensin, Kent Kawahara, Stephen Thorne (eds.), *The Site Reliability Workbook*** (O'Reilly, 2018). <https://sre.google/workbook/table-of-contents/>
- **Andriy Burkov, *Machine Learning Engineering*** (True Positive Inc., 2020). Complementary treatment of evaluation and deployment. <http://www.mlebook.com/>

## Vendor pointers (for the LLM-augmented cases)

Where the harness scores an LLM-augmented feature, vendor pricing and rate limits change often; consult these directly rather than caching numbers.

- **Anthropic — API documentation.** <https://docs.anthropic.com/en/api/overview>
- **OpenAI — API reference.** <https://platform.openai.com/docs/api-reference>
- **Google — Gemini API.** <https://ai.google.dev/gemini-api/docs>
- **AWS Bedrock — user guide.** <https://docs.aws.amazon.com/bedrock/latest/userguide/>

## Peer specialist tracks (delegation targets)

The chapter-05 hand-off notes and the exercise-04 stretch goals point at the AICG peer tracks below. Consult the READMEs of each for the delegation-contract vocabulary.

- `model-evaluation-engineer-learning` — evaluation-engineering depth: benchmark curation, held-out design, dataset governance.
- `ai-eval-engineer-learning` — LLM-eval systems depth: industrial judge harnesses, judge design at scale, judge calibration methodology.
- `rag-engineer-learning` — RAG evaluation depth for the reference-based judge case.
- `fine-tuning-engineer-learning` — depth for the case where the judge itself is a fine-tuned evaluator model (Prometheus-style).
- `ai-infra-mlops-learning` / `ai-infra-ml-platform-learning` — the platform stack the harness runs on and the promotion path executes through.

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->

## Cross-references inside this module

- Chapter 01 (`01-offline-eval-harness-shape.md`) defines the harness shape and `HarnessDecision` object every other chapter builds on.
- Chapter 02 (`02-slice-and-adversarial-guardrails.md`) grows the slice matrix and adversarial suite inside the harness.
- Chapter 03 (`03-shadow-and-canary-online-promotion.md`) is the online path an offline `passes_release_gate=True` earns.
- Chapter 04 (`04-ml-test-score-production-readiness.md`) is the review packet that says the whole apparatus is fit for purpose.
- Chapter 05 (`05-llm-as-judge-when-and-how.md`) is the extension for LLM-augmented features.

## Cross-references to other modules

- [mod-302 chapter 02](../mod-302-ml-systems-architecture/) (batch / streaming / online) sets the training–serving parity vocabulary chapter 04 Monitoring Test 3 refers to.
- [mod-303 chapter 05 (calibration)](../mod-303-advanced-modeling/05-calibration.md) is the calibration guardrail's canonical treatment.
- [mod-304 chapter 02](../mod-304-production-llm-integration/02-prompts-as-engineering-artifacts.md) is the prompt-bundle discipline the LLM-as-judge chapter inherits.
- [mod-304 chapter 04](../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) is the cost / latency envelope the offline guardrail and the online shadow are measured against.
- [mod-306](../mod-306-experimentation-at-scale/) is the depth of the A/B stage the promotion path passes through.
- [mod-307](../mod-307-ml-reliability-slos/) is the SLI vocabulary the auto-rollback controller and monitoring tests are named against.
- [mod-309](../mod-309-responsible-ai-governance/) is where the fairness-slice policy and the human-eval review packet are authored.
