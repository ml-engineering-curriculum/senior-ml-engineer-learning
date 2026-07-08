# Resources for mod-303-advanced-modeling

Curated external references. These are the primary and authoritative sources cited or leaned on by the chapters and exercises above. Every URL is publicly reachable at the time of authoring.

## Canonical ML texts (all chapters)

- **Chip Huyen, *Designing Machine Learning Systems*** (O'Reilly, 2022) — the ML-systems altitude reference this module operates on top of. Chapter 6 (model development and offline evaluation) and Chapter 8 (data distribution shifts and monitoring) map directly to chapters 05–07 of this module. <https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/> · <https://huyenchip.com/books/>
- **Kevin Murphy, *Probabilistic Machine Learning: An Introduction*** (MIT Press, 2022) — the probabilistic-modeling reference the calibration and uncertainty chapters lean on. Freely available for reading. <https://probml.github.io/pml-book/book1.html>
- **Kevin Murphy, *Probabilistic Machine Learning: Advanced Topics*** (MIT Press, 2023) — deeper coverage of Bayesian deep learning, uncertainty, and probabilistic modeling used in chapter 06. <https://probml.github.io/pml-book/book2.html>
- **Ian Goodfellow, Yoshua Bengio, Aaron Courville, *Deep Learning*** (MIT Press, 2016) — chapters 7 (regularization) and 15 (representation learning) underlie chapters 02 and 04. <https://www.deeplearningbook.org/>

## Multi-task learning (chapter 02, exercise-02)

- **Rich Caruana, *Multitask Learning*** (Machine Learning, 1997) — the canonical original paper on why representation-sharing generalises. <https://link.springer.com/article/10.1023/A:1007379606734>
- **Sebastian Ruder, *An Overview of Multi-Task Learning in Deep Neural Networks*** (2017) — the modern survey; the hard-sharing / soft-sharing vocabulary comes from here. <https://arxiv.org/abs/1706.05098>
- **Yu Zhang and Qiang Yang, *A Survey on Multi-Task Learning*** (IEEE TKDE, 2021) — a broader academic survey. <https://arxiv.org/abs/1707.08114>
- **Ma, Zhao, Yi, Chen, Hong, Chi, *Modeling Task Relationships in Multi-task Learning with Multi-gate Mixture-of-Experts (MMoE)*** (KDD 2018) — the industrial escalation from hard sharing to gated MoE. <https://dl.acm.org/doi/10.1145/3219819.3220007>
- **Zhao, Hong, Wei, Chen, Nath, Andrews, Kumthekar, Sathiamoorthy, Yi, Chi, *Recommending What Video to Watch Next: A Multitask Ranking System*** (RecSys 2019) — the YouTube MTL ranker; production-scale application of MMoE. <https://dl.acm.org/doi/10.1145/3298689.3346997>
- **Kendall, Gal, Cipolla, *Multi-Task Learning Using Uncertainty to Weigh Losses for Scene Geometry and Semantics*** (CVPR 2018) — uncertainty-weighted loss balancing. <https://arxiv.org/abs/1705.07115>
- **Chen, Badrinarayanan, Lee, Rabinovich, *GradNorm: Gradient Normalization for Adaptive Loss Balancing in Deep Multitask Networks*** (ICML 2018). <https://arxiv.org/abs/1711.02257>
- **Yu, Kumar, Gupta, Levine, Hausman, Finn, *Gradient Surgery for Multi-Task Learning (PCGrad)*** (NeurIPS 2020). <https://arxiv.org/abs/2001.06782>
- **Fifty, Amid, Zhao, Yu, Anil, Finn, *Efficiently Identifying Task Groupings for Multi-Task Learning*** (NeurIPS 2021) — task clustering; useful when hard sharing fails on a task subset. <https://arxiv.org/abs/2109.04617>
- **Misra, Shrivastava, Gupta, Hebert, *Cross-stitch Networks for Multi-task Learning*** (CVPR 2016) — the learned per-layer sharing shape. <https://arxiv.org/abs/1604.03539>

## Multi-modal modeling (chapter 03)

- **Baltrušaitis, Ahuja, Morency, *Multimodal Machine Learning: A Survey and Taxonomy*** (IEEE TPAMI, 2018) — the canonical taxonomy the fusion-altitude vocabulary is drawn from. <https://arxiv.org/abs/1705.09406>
- **Radford, Kim, Hallacy, Ramesh, Goh, Agarwal, Sastry, Askell, Mishkin, Clark, Krueger, Sutskever, *Learning Transferable Visual Models From Natural Language Supervision (CLIP)*** (ICML 2021) — dual-encoder contrastive multi-modal. <https://arxiv.org/abs/2103.00020>
- **Alayrac, Donahue, Luc, Miech, Barr, Hasson, Lenc, Mensch, Millican, Reynolds, Ring, Rutherford, Cabi, Han, Gong, Samangooei, Monteiro, Menick, Borgeaud, Brock, Nematzadeh, Sharifzadeh, Binkowski, Barreira, Vinyals, Zisserman, Simonyan, *Flamingo: a Visual Language Model for Few-Shot Learning*** (NeurIPS 2022) — perceiver-based middle fusion. <https://arxiv.org/abs/2204.14198>
- **Li, Li, Savarese, Hoi, *BLIP-2: Bootstrapping Language-Image Pre-training with Frozen Image Encoders and Large Language Models*** (ICML 2023) — Q-Former bridging modality-specific encoders. <https://arxiv.org/abs/2301.12597>

## Self-supervised and transfer learning (chapter 04)

- **Devlin, Chang, Lee, Toutanova, *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding*** (NAACL 2019) — the canonical masked-language-modeling reference. <https://arxiv.org/abs/1810.04805>
- **Chen, Kornblith, Norouzi, Hinton, *A Simple Framework for Contrastive Learning of Visual Representations (SimCLR)*** (ICML 2020). <https://arxiv.org/abs/2002.05709>
- **He, Fan, Wu, Xie, Girshick, *Momentum Contrast for Unsupervised Visual Representation Learning (MoCo)*** (CVPR 2020). <https://arxiv.org/abs/1911.05722>
- **He, Chen, Xie, Li, Dollar, Girshick, *Masked Autoencoders Are Scalable Vision Learners (MAE)*** (CVPR 2022). <https://arxiv.org/abs/2111.06377>
- **Baevski, Zhou, Mohamed, Auli, *wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations*** (NeurIPS 2020). <https://arxiv.org/abs/2006.11477>
- **Yosinski, Clune, Bengio, Lipson, *How transferable are features in deep neural networks?*** (NeurIPS 2014) — the classical "when transfer works" reference. <https://arxiv.org/abs/1411.1792>
- **Howard, Ruder, *Universal Language Model Fine-tuning for Text Classification (ULMFiT)*** (ACL 2018) — discriminative learning rates, gradual unfreezing. <https://arxiv.org/abs/1801.06146>
- **Gururangan, Marasovic, Swayamdipta, Lo, Beltagy, Downey, Smith, *Don't Stop Pretraining: Adapt Language Models to Domains and Tasks*** (ACL 2020) — the domain-adaptive pre-training pattern. <https://arxiv.org/abs/2004.10964>
- **Hu, Shen, Wallis, Allen-Zhu, Li, Wang, Wang, Chen, *LoRA: Low-Rank Adaptation of Large Language Models*** (ICLR 2022) — the reference for parameter-efficient fine-tuning. <https://arxiv.org/abs/2106.09685>
- **Kirkpatrick, Pascanu, Rabinowitz, Veness, Desjardins, Rusu, Milan, Quan, Ramalho, Grabska-Barwinska, Hassabis, Clopath, Kumaran, Hadsell, *Overcoming catastrophic forgetting in neural networks (EWC)*** (PNAS 2017). <https://arxiv.org/abs/1612.00796>

## Ensembling and distillation (chapter 04)

- **Hinton, Vinyals, Dean, *Distilling the Knowledge in a Neural Network*** (NeurIPS 2014 Deep Learning Workshop) — the canonical distillation reference. <https://arxiv.org/abs/1503.02531>
- **Sanh, Debut, Chaumond, Wolf, *DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter*** (NeurIPS 2019 EMC^2 workshop) — production application of distillation. <https://arxiv.org/abs/1910.01108>
- **Romero, Ballas, Kahou, Chassang, Gatta, Bengio, *FitNets: Hints for Thin Deep Nets*** (ICLR 2015) — intermediate-layer distillation. <https://arxiv.org/abs/1412.6550>
- **Huang, Li, Pleiss, Liu, Hopcroft, Weinberger, *Snapshot Ensembles: Train 1, get M for free*** (ICLR 2017). <https://arxiv.org/abs/1704.00109>
- **Lakshminarayanan, Pritzel, Blundell, *Simple and Scalable Predictive Uncertainty Estimation using Deep Ensembles*** (NeurIPS 2017) — deep ensembles for both accuracy and uncertainty; the modern reference on both fronts. <https://arxiv.org/abs/1612.01474>

## Calibration (chapter 05, exercise-03)

- **Guo, Pleiss, Sun, Weinberger, *On Calibration of Modern Neural Networks*** (ICML 2017) — the canonical modern reference documenting deep-network miscalibration and introducing temperature scaling as a default. <https://arxiv.org/abs/1706.04599>
- **Platt, *Probabilistic Outputs for Support Vector Machines and Comparisons to Regularized Likelihood Methods*** (1999) — the original Platt scaling reference. <https://www.researchgate.net/publication/2594015_Probabilistic_Outputs_for_Support_Vector_Machines_and_Comparisons_to_Regularized_Likelihood_Methods>
- **Zadrozny, Elkan, *Transforming Classifier Scores into Accurate Multiclass Probability Estimates*** (KDD 2002) — isotonic calibration. <https://www.cs.cornell.edu/~alexn/papers/zadrozny.kdd02.pdf>
- **Kull, Silva Filho, Flach, *Beyond sigmoids: How to obtain well-calibrated probabilities from binary classifiers with beta calibration*** (Electronic Journal of Statistics, 2017) — the Beta calibration reference. <https://projecteuclid.org/journals/electronic-journal-of-statistics/volume-11/issue-2/Beyond-sigmoids--How-to-obtain-well-calibrated-probabilities-from/10.1214/17-EJS1338SI.full>
- **Naeini, Cooper, Hauskrecht, *Obtaining Well Calibrated Probabilities Using Bayesian Binning*** (AAAI 2015) — the ECE formulation this chapter uses. <https://ojs.aaai.org/index.php/AAAI/article/view/9602>
- **scikit-learn documentation on calibration** — practical implementations of Platt, isotonic, and reliability diagrams. <https://scikit-learn.org/stable/modules/calibration.html>

## Uncertainty quantification (chapter 06, exercise-03)

- **Kendall, Gal, *What Uncertainties Do We Need in Bayesian Deep Learning for Computer Vision?*** (NeurIPS 2017) — the aleatoric-vs-epistemic split reference. <https://arxiv.org/abs/1703.04977>
- **Gal, Ghahramani, *Dropout as a Bayesian Approximation: Representing Model Uncertainty in Deep Learning (MC Dropout)*** (ICML 2016). <https://arxiv.org/abs/1506.02142>
- **Liu, Lin, Padhy, Tran, Bedrax-Weiss, Lakshminarayanan, *Simple and Principled Uncertainty Estimation with Deterministic Deep Learning via Distance Awareness (SNGP)*** (NeurIPS 2020) — a deterministic-network uncertainty method. <https://arxiv.org/abs/2006.10108>
- **Vovk, Gammerman, Shafer, *Algorithmic Learning in a Random World*** (Springer, 2005) — the canonical book on conformal prediction. <https://link.springer.com/book/10.1007/b106715>
- **Angelopoulos, Bates, *A Gentle Introduction to Conformal Prediction and Distribution-Free Uncertainty Quantification*** (2021) — the accessible modern tutorial. <https://arxiv.org/abs/2107.07511>
- **Gibbs, Candès, *Adaptive Conformal Inference Under Distribution Shift*** (NeurIPS 2021) — conformal under shift. <https://arxiv.org/abs/2106.00170>

## Cold-start, long-tail, label-scarce (chapter 07, exercise-04)

- **Zhang, Kang, Hooi, Yan, Feng, *Deep Long-Tailed Learning: A Survey*** (IEEE TPAMI, 2023) — the modern survey. <https://arxiv.org/abs/2110.04596>
- **Cui, Jia, Lin, Song, Belongie, *Class-Balanced Loss Based on Effective Number of Samples*** (CVPR 2019). <https://arxiv.org/abs/1901.05555>
- **Lin, Goyal, Girshick, He, Dollar, *Focal Loss for Dense Object Detection*** (ICCV 2017). <https://arxiv.org/abs/1708.02002>
- **Kang, Xie, Rohrbach, Yan, Gordo, Feng, Kalantidis, *Decoupling Representation and Classifier for Long-Tailed Recognition*** (ICLR 2020) — the reliably-strong long-tail baseline. <https://arxiv.org/abs/1910.09217>
- **Covington, Adams, Sargin, *Deep Neural Networks for YouTube Recommendations*** (RecSys 2016) — cold-start item embeddings and two-tower retrieval in production. <https://research.google/pubs/pub45530/>
- **Yi, Yang, Hong, Cheng, Heldt, Kumthekar, Zhao, Wei, Chi, *Sampling-Bias-Corrected Neural Modeling for Large Corpus Item Recommendations*** (RecSys 2019) — the industrial two-tower recipe with content features on the item side. <https://dl.acm.org/doi/10.1145/3298689.3346996>
- **Joachims, Swaminathan, Schnabel, *Unbiased Learning-to-Rank with Biased Feedback*** (WSDM 2017). <https://arxiv.org/abs/1608.04468>
- **Ratner, Bach, Ehrenberg, Fries, Wu, Ré, *Snorkel: Rapid Training Data Creation with Weak Supervision*** (VLDB 2018) — the canonical weak-supervision reference. <https://arxiv.org/abs/1711.10160>
- **Settles, *Active Learning Literature Survey*** (Univ. Wisconsin Computer Sciences Technical Report, 2010) — the classical active-learning reference. <http://burrsettles.com/pub/settles.activelearning.pdf>
- **Data-Centric AI Resource Hub** — the mindset shift for label-scarce problems. <https://datacentricai.org/>

## Escalation and specialist tracks (chapter 08)

The escalation chapter refers to peer tracks in the wider AICG curriculum ecosystem. Read the READMEs of each so the hand-off contracts are concrete.

- Peer specialist tracks (consume-from targets): `llm-application-developer-learning`, `rag-engineer-learning`, `fine-tuning-engineer-learning`, `training-pipeline-engineer-learning`, `applied-ai-engineer-learning`.
- Peer evaluation tracks: `model-evaluation-engineer-learning`, `ai-eval-engineer-learning`.
- Peer platform tracks: `ai-infra-ml-platform-learning`, `ai-infra-mlops-learning`.
- Governance / security counterparts: `ai-governance-analyst-learning`, `ai-infra-security-learning`.
- Higher-level tracks: `staff-ml-engineer-learning`, `principal-ml-engineer-learning`.

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->

## Cross-references inside this module

- Chapter 01 (`01-choosing-the-modeling-regime.md`) is the decision map; every other chapter drills into one of its branches.
- Chapters 02–04 (regime chapters) feed the RFC skeleton in mod-302 chapter 06.
- Chapters 05–06 (calibration + uncertainty) feed the release-harness discipline in mod-305.
- Chapter 07 (cold-start / long-tail / label-scarce) feeds the evaluation-slice discipline in mod-305 and the SLO framing in mod-307.
- Chapter 08 (escalation) feeds the peer-collaboration discipline in mod-308.
