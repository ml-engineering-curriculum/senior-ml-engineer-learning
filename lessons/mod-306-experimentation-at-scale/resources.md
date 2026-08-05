# Resources for mod-306-experimentation-at-scale

Curated external references. These are the primary and authoritative sources the chapters and exercises above cite or lean on. Every URL is publicly reachable at the time of authoring; where a paper's canonical URL is behind a paywall, an author-hosted preprint URL is provided instead.

## The load-bearing textbook (all chapters, all exercises)

- **Ron Kohavi, Diane Tang, Ya Xu, *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*** (Cambridge University Press, 2020). The modern textbook on industrial A/B testing at scale. Chapters 3, 6, 15, 17, and 22 map directly onto this module. <https://experimentguide.com/>
- **Ron Kohavi, Alex Deng, Lukas Vermeer, *A/B Testing Intuition Busters: Common Misinterpretations in Online Controlled Experiments*** (2022). Concise catalogue of the failure modes chapters 01 and 02 formalise. <https://exp-platform.com/Documents/2022-A_BTestingIntuitionBusters.pdf>

## Experiment design, power, and MDE (chapter 01, exercise 01)

- **Ron Kohavi, Roger Longbotham, Dan Sommerfield, Randal M. Henne, *Controlled experiments on the web: survey and practical guide*** (Data Mining and Knowledge Discovery, 2009). The classical industrial reference for A/B testing at scale. <https://ai.stanford.edu/~ronnyk/2009controlledExperimentsOnTheWebSurvey.pdf>
- **Chip Huyen, *Designing Machine Learning Systems*** (O'Reilly, 2022). Chapter 6 on evaluation and chapter 7 on model deployment are the classical framing this module's design contract extends. <https://huyenchip.com/books/> · <https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/>
- **Diane Tang, Ashish Agarwal, Deirdre O'Brien, Mike Meyer, *Overlapping Experiment Infrastructure: More, Better, Faster Experimentation*** (KDD 2010). The paper describing Google's layered / mutually-exclusive experiment platform — the depth behind chapter 01 §5's "interaction with other experiments" line. <https://research.google/pubs/pub36500/>
- **Andrew Gelman, Eric Loken, *The garden of forking paths: Why multiple comparisons can be a problem*** (2013). The statistical-methodology reference for chapter 01 §1's "primary metric chosen before the candidate exists" rule. <http://www.stat.columbia.edu/~gelman/research/unpublished/p_hacking.pdf>
- **Ron Kohavi, Roger Longbotham, Toby Walker, *Online Experimentation at Microsoft*** (Data Mining Case Studies workshop, 2009). Applied treatment of running an experimentation program. <https://ai.stanford.edu/~ronnyk/ExPThinkWeek2009Public.pdf>

## Trust diagnostics — SRM, interference, novelty (chapter 02, exercise 02)

- **Aleksander Fabijan, Jayant Gupchup, Somit Gupta, Jeff Omhover, Wen Qin, Lukas Vermeer, Pavel Dmitriev, *Diagnosing Sample Ratio Mismatch in Online Controlled Experiments: A Taxonomy and Rules of Thumb for Practitioners*** (KDD 2019). The canonical industrial paper on SRM causes, thresholds, and remediation. <https://www.microsoft.com/en-us/research/publication/diagnosing-sample-ratio-mismatch-in-a-b-testing-a-lightweight-approach/>
- **Nanyu Chen, Min Liu, Ya Xu, *How A/B Tests Could Go Wrong: Automatic Diagnosis of Invalid Online Experiments*** (WSDM 2019). LinkedIn's automated trust-diagnostic system. <https://www.microsoft.com/en-us/research/publication/how-a-b-tests-could-go-wrong-automatic-diagnosis-of-invalid-online-experiments/>
- **Thomas Blake, Dominic Coey, *Why Marketplace Experimentation Is Harder to Design and Analyze*** (eBay research white paper, 2014). The load-bearing industrial exposition of marketplace interference. <https://sites.google.com/site/danielkcoey/marketplace-experimentation>
- **Johan Ugander, Brian Karrer, Lars Backstrom, Jon Kleinberg, *Graph cluster randomization: network exposure to multiple universes*** (KDD 2013). Cluster randomisation for network / social interference. <https://arxiv.org/abs/1305.6979>
- **Eytan Bakshy, Dean Eckles, Michael S. Bernstein, *Designing and Deploying Online Field Experiments*** (WWW 2014). Facebook's PlanOut framework and the experimental designs used for network features. <https://arxiv.org/abs/1409.3174>
- **DoorDash Engineering, *Switchback Tests and Randomized Experimentation Under Network Effects at DoorDash*** — industry write-up on switchback designs for marketplace surfaces. <https://careersatdoordash.com/blog/switchback-tests-and-randomized-experimentation-under-network-effects-at-doordash/>
- **Iavor Bojinov, Neil Shephard, *Time Series Experiments and Causal Estimands: Exact Randomization Tests and Trading*** (JASA 2019). Statistical treatment of switchback / time-series experiment estimators. <https://arxiv.org/abs/1706.07840>
- **Ron Kohavi, Randal Henne, Dan Sommerfield, *Practical Guide to Controlled Experiments on the Web: Listen to Your Customers not to the HiPPO*** (KDD 2007). Includes the classical treatment of novelty / primacy effects. <https://exp-platform.com/Documents/2007-08KDDHiPPO_LongVersion.pdf>

## Variance reduction — CUPED, CUPAC, stratification (chapter 03, exercise 03)

- **Alex Deng, Ya Xu, Ron Kohavi, Toby Walker, *Improving the sensitivity of online controlled experiments by utilizing pre-experiment data*** (WSDM 2013). The CUPED paper. <https://exp-platform.com/Documents/2013-02-CUPED-ImprovingSensitivityOfControlledExperiments.pdf>
- **Huizhi Xie, Juliette Aurisset, *Improving the sensitivity of online controlled experiments: Case studies at Netflix*** (KDD 2016). Netflix's applied write-up of CUPED and its variants. <https://netflixtechblog.com/reducing-variance-in-online-experiments-by-utilizing-post-experiment-data-4c60ea11b025>
- **Alexey Poyarkov, Alexey Drutsa, Andrey Khalyavin, Gleb Gusev, Pavel Serdyukov, *Boosted Decision Tree Regression Adjustment for Variance Reduction in Online Controlled Experiments*** (KDD 2016). The canonical academic reference for the regression-adjustment generalisation of CUPED. <https://research.yandex.com/publications/121>
- **DoorDash Engineering, *Improving Experimental Power Through Control Using Predictions as Covariate (CUPAC)*** (2020). The widely-cited industry write-up under the CUPAC name. <https://doordash.engineering/2020/06/08/improving-experimental-power-through-control-using-predictions-as-covariate-cupac/>
- **Victor Chernozhukov, Denis Chetverikov, Mert Demirer, Esther Duflo, Christian Hansen, Whitney Newey, James Robins, *Double/Debiased Machine Learning for Treatment and Structural Parameters*** (Econometrics Journal, 2018). The reference for combining flexible ML predictions with unbiased treatment-effect estimation. <https://arxiv.org/abs/1608.00060>
- **Luke W. Miratrix, Jasjeet S. Sekhon, Bin Yu, *Adjusting Treatment Effect Estimates by Post-Stratification in Randomized Experiments*** (JRSSB 2013). The modern reference on post-stratification. <https://arxiv.org/abs/1109.6402>
- **Alex Deng, Ulf Knoblich, Jiannan Lu, *Applying the Delta Method in Metric Analytics: A Practical Guide with Novel Ideas*** (KDD 2018). The industrial reference for ratio-metric variance estimation. <https://exp-platform.com/Documents/2018KDDDeltaMethod.pdf>

## Sequential testing and peeking (chapter 04)

- **Ramesh Johari, Leo Pekelis, David J. Walsh, *Always Valid Inference: Continuous Monitoring of A/B Tests*** (Operations Research, 2022). The modern reference for always-valid inference in industrial A/B testing. <https://arxiv.org/abs/2103.14043>
- **Ramesh Johari, Pete Koomen, Leo Pekelis, David Walsh, *Peeking at A/B Tests: Why It Matters, and What to Do About It*** (KDD 2017). The applied paper that quantifies the peeking problem. <https://arxiv.org/abs/1512.04922>
- **Abraham Wald, *Sequential Analysis*** (Wiley, 1947). The origin of the sequential probability ratio test. <https://archive.org/details/sequentialanalys00wald_0>
- **David Siegmund, *Sequential Analysis: Tests and Confidence Intervals*** (Springer, 1985). Classical treatment of sequential inference and group-sequential boundaries. <https://link.springer.com/book/10.1007/978-1-4757-1862-1>
- **Alex Deng, Jiannan Lu, Shouyuan Chen, *Continuous Monitoring of A/B Tests without Pain: Optional Stopping in Bayesian Testing*** (2016). Bayesian framing of the peeking question. <https://arxiv.org/abs/1602.05549>
- **Ian Waudby-Smith, Aaditya Ramdas, *Estimating means of bounded random variables by betting*** (JRSSB 2024). The betting / e-value approach to confidence sequences. <https://arxiv.org/abs/2010.09686>
- **Aaditya Ramdas, Peter Grünwald, Vladimir Vovk, Glenn Shafer, *Game-theoretic statistics and safe anytime-valid inference*** (Statistical Science, 2023). Modern overview of the anytime-valid inference program. <https://arxiv.org/abs/2210.01948>

## Causal inference (chapter 05, exercise 04)

- **Scott Cunningham, *Causal Inference: The Mixtape*** (Yale University Press, 2021). Freely available online. Excellent applied introduction to PSM, DID, IV, RDD, and synthetic control. <https://mixtape.scunning.com/>
- **Joshua Angrist, Jörn-Steffen Pischke, *Mostly Harmless Econometrics: An Empiricist's Companion*** (Princeton University Press, 2009). Classical applied econometrics text; the modern reference for IV, DID, RDD in the applied setting. <https://www.mostlyharmlesseconometrics.com/>
- **Guido Imbens, Donald Rubin, *Causal Inference for Statistics, Social, and Biomedical Sciences: An Introduction*** (Cambridge University Press, 2015). The graduate-level statistical treatment. <https://www.cambridge.org/core/books/causal-inference-for-statistics-social-and-biomedical-sciences/71126BE90C58F1A431FE9B2DD07938AB>
- **Judea Pearl, *Causality: Models, Reasoning, and Inference* (2nd ed.)** (Cambridge University Press, 2009). Do-calculus and structural causal models. <http://bayes.cs.ucla.edu/BOOK-2K/>
- **Miguel Hernán, James Robins, *Causal Inference: What If*** (Chapman & Hall / CRC, 2020). Freely available online. Modern potential-outcomes treatment with an epidemiology bent. <https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/>
- **Susan Athey, Guido Imbens, *The State of Applied Econometrics: Causality and Policy Evaluation*** (Journal of Economic Perspectives, 2017). Compact modern survey. <https://www.aeaweb.org/articles?id=10.1257/jep.31.2.3>

### Propensity score matching and IPW

- **Paul R. Rosenbaum, Donald B. Rubin, *The Central Role of the Propensity Score in Observational Studies for Causal Effects*** (Biometrika, 1983). The origin of propensity-score methods. <https://academic.oup.com/biomet/article-abstract/70/1/41/240879>
- **Elizabeth A. Stuart, *Matching Methods for Causal Inference: A Review and a Look Forward*** (Statistical Science, 2010). The modern applied review. <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2943670/>
- **Tyler VanderWeele, Peng Ding, *Sensitivity Analysis in Observational Research: Introducing the E-Value*** (Annals of Internal Medicine, 2017). The E-value sensitivity analysis chapter 05 recommends. <https://www.acpjournals.org/doi/10.7326/M16-2607>

### Difference-in-differences

- **David Card, Alan B. Krueger, *Minimum Wages and Employment: A Case Study of the Fast-Food Industry in New Jersey and Pennsylvania*** (American Economic Review, 1994). The canonical modern DID exemplar. <https://davidcard.berkeley.edu/papers/njmin-aer.pdf>
- **Brantly Callaway, Pedro H. C. Sant'Anna, *Difference-in-Differences with Multiple Time Periods*** (Journal of Econometrics, 2021). Staggered-adoption DID estimator. <https://arxiv.org/abs/1803.09015>
- **Andrew Goodman-Bacon, *Difference-in-Differences with Variation in Treatment Timing*** (Journal of Econometrics, 2021). The paper documenting TWFE bias under staggered adoption. <https://arxiv.org/abs/1908.05481>
- **Liyang Sun, Sarah Abraham, *Estimating Dynamic Treatment Effects in Event Studies with Heterogeneous Treatment Effects*** (Journal of Econometrics, 2021). Companion staggered-DID estimator. <https://arxiv.org/abs/1804.05785>

### Instrumental variables

- **Joshua D. Angrist, Guido W. Imbens, Donald B. Rubin, *Identification of Causal Effects Using Instrumental Variables*** (JASA, 1996). The modern statistical statement of IV as a causal identification strategy. <https://www.jstor.org/stable/2291629>
- **Douglas Staiger, James H. Stock, *Instrumental Variables Regression with Weak Instruments*** (Econometrica, 1997). Origin of the "F > 10" weak-instruments rule of thumb. <https://scholar.harvard.edu/stock/publications/instrumental-variables-regression-weak-instruments>

### Regression discontinuity

- **Guido Imbens, Thomas Lemieux, *Regression Discontinuity Designs: A Guide to Practice*** (Journal of Econometrics, 2008). The applied reference. <https://scholar.harvard.edu/imbens/publications/regression-discontinuity-designs-guide-practice>
- **David S. Lee, Thomas Lemieux, *Regression Discontinuity Designs in Economics*** (Journal of Economic Literature, 2010). Broader survey. <https://www.aeaweb.org/articles?id=10.1257/jel.48.2.281>
- **Justin McCrary, *Manipulation of the Running Variable in the Regression Discontinuity Design: A Density Test*** (Journal of Econometrics, 2008). The mandatory density-continuity diagnostic. <https://eml.berkeley.edu/~jmccrary/DCdensity/>

### Synthetic control

- **Alberto Abadie, Alexis Diamond, Jens Hainmueller, *Synthetic Control Methods for Comparative Case Studies*** (JASA, 2010). The origin of synthetic-control methods. <https://economics.mit.edu/sites/default/files/publications/synthetic-control-methods.pdf>
- **Alberto Abadie, *Using Synthetic Controls: Feasibility, Data Requirements, and Methodological Aspects*** (Journal of Economic Literature, 2021). Modern practical guide. <https://economics.mit.edu/sites/default/files/publications/Using%20Synthetic%20Controls-%20Feasibility%2C%20Data%20Requirements.pdf>
- **Susan Athey, Mohsen Bayati, Nikolay Doudchenko, Guido Imbens, Khashayar Khosravi, *Matrix Completion Methods for Causal Panel Data Models*** (JASA, 2021). Modern synthetic-control generalisation via matrix completion. <https://arxiv.org/abs/1710.10251>

## Tooling and platforms

- **statsmodels — Python statistical library.** Power / MDE calculations, regression, sequential tests. <https://www.statsmodels.org/>
- **`gsDesign` — R package for group-sequential design.** Pocock, O'Brien-Fleming, α-spending. <https://cran.r-project.org/package=gsDesign>
- **`confseq` — Python / R libraries for confidence sequences.** From Waudby-Smith / Ramdas. <https://github.com/gostevehoward/confseq>
- **`MatchIt` — R package for matching methods.** PSM, nearest-neighbour, exact matching. <https://kosukeimai.github.io/MatchIt/>
- **`causalinference` — Python library for causal inference on observational data.** IPW, matching, regression. <https://laurencewong.com/software/>
- **`did` (Callaway–Sant'Anna) and `differences` (Python) — staggered-DID implementations.** <https://bcallaway11.github.io/did/> · <https://bernardodionisi.github.io/differences/>
- **`Synth` (R) / `SparseSC` (Python) — synthetic-control implementations.** <https://cran.r-project.org/package=Synth> · <https://github.com/microsoft/SparseSC>
- **Optimizely Stats Engine.** Productionised mSPRT-based sequential testing. <https://docs.developers.optimizely.com/web-experimentation/docs/stats-accelerator>
- **Statsig, Eppo, GrowthBook, LaunchDarkly Experimentation.** Modern experimentation platforms with built-in SRM checks, sequential-testing options, and CUPED variance reduction. Consult vendor docs directly.

## Governance and ethics for experimentation

- **NIST, *AI Risk Management Framework 1.0* (NIST AI 100-1).** The framework mod-309 leans on; relevant here for observational-causal analyses that inform regulator-visible decisions. <https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf>
- **Association of Internet Researchers, *Ethical Decision-Making and Internet Research 3.0*.** Reference on ethical considerations for online experiments involving human subjects. <https://aoir.org/reports/ethics3.pdf>
- **Michelle N. Meyer, *Two Cheers for Corporate Experimentation: The A/B Illusion and the Virtues of Data-Driven Innovation*** (Colorado Technology Law Journal, 2015). Ethical grounding for corporate A/B testing. <https://ssrn.com/abstract=2605132>

## Cross-references inside this module

- Chapter 01 (`01-experiment-design-power-mde-and-ramp.md`) defines the seven-piece contract every other chapter builds on.
- Chapter 02 (`02-trust-diagnostics-srm-interference-novelty.md`) is the load-bearing trust layer that gates every scorecard the experiment produces.
- Chapter 03 (`03-variance-reduction-cuped-cupac.md`) is the lever that shrinks the MDE inside a trustworthy experiment.
- Chapter 04 (`04-sequential-testing-and-peeking.md`) is the framing change that makes peeking valid.
- Chapter 05 (`05-causal-inference-when-ab-is-impossible.md`) is the fallback when no experiment can be run.

## Cross-references to other modules

- [mod-303 chapter 05 (calibration)](../mod-303-advanced-modeling/05-calibration.md) is where guardrail-level calibration is authored; chapter 01 §2 references it as a common guardrail metric.
- [mod-304 chapter 04 (cost and latency guardrails)](../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) is the cost / latency envelope the online guardrails inherit.
- [mod-305 chapter 03 (shadow and canary online promotion)](../mod-305-advanced-evaluation/03-shadow-and-canary-online-promotion.md) is the promotion path whose A/B stage this module deepens.
- [mod-305 chapter 02 (slices and adversarial guardrails)](../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) is the slice matrix re-enforced online in chapter 01's guardrail set.
- [mod-307](../mod-307-ml-reliability-slos/) is the SLI vocabulary the auto-rollback controller and guardrail metrics are named against.
- [mod-309](../mod-309-responsible-ai-governance/) is where regulator-visible slice policy and observational-causal analyses for fairness-visible decisions are authored.

## Peer specialist tracks (delegation targets)

Where the experimentation platform itself, or the depth of a specific technique, exceeds a senior ML engineer's day-to-day, the AICG peer tracks below are the delegation targets. Consult the READMEs of each for the delegation-contract vocabulary.

- `ai-infra-mlops-learning` — the MLOps / feature-store / model-registry platform the experimentation platform runs on.
- `ai-infra-ml-platform-learning` — the ML platform layer that owns the experimentation platform itself.
- `data-science-generalist-learning` / `product-data-science-learning` — deeper depth on experiment design, causal inference, and long-horizon metric strategy.
- `experimentation-platform-engineer-learning` — depth on building the experimentation platform (SRM detection, bucketing service, sequential-testing engine).

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->
