# Self-supervised, transfer, ensembling and distillation

## Motivation

Two economic realities dominate modeling at senior altitude.

- **Labels are expensive; unlabelled data is cheap.** Every domain has orders of magnitude more unlabelled data than labelled. Self-supervised learning (SSL) and transfer learning are the toolbox that converts that asymmetry into models.
- **Serving is one budget; training is another.** Ensembles reliably win offline metrics; they also reliably blow through serving budgets. Distillation is the mechanism that lets you buy the ensemble's accuracy back into a single deployable model.

Together, these two toolboxes make most modern production ML feasible at all. A senior ML engineer who cannot reach for them wastes labelling budget training weak backbones from scratch and wastes serving budget shipping ensembles that could have been distilled.

This chapter is the practical guide. It answers: when to reach for a pre-trained backbone versus training from scratch; how to pick fine-tune vs. linear-probe vs. frozen features; when ensembling is worth the training tax; when distillation is worth the operational tax.

## Part 1 — Self-supervised learning and transfer

### The bet, in one paragraph

The bet is that a large model trained on a self-supervised objective (predict the masked token, contrastively pull matched pairs together, predict the next token) learns internal representations that transfer to your labelled task. If the bet pays, you get a small fine-tuning run against your labels instead of a from-scratch training run. If the bet fails — the pre-training distribution and your fine-tuning distribution are far apart — you get modest gains at best and catastrophic behaviour at worst.

The empirical evidence for the bet is enormous and mostly one-way: pre-trained backbones consistently outperform from-scratch models on downstream tasks when the label budget is small to moderate. See Devlin et al.'s [*BERT*](https://arxiv.org/abs/1810.04805), Chen et al.'s [*SimCLR*](https://arxiv.org/abs/2002.05709), Radford et al.'s [*CLIP*](https://arxiv.org/abs/2103.00020), Baevski et al.'s [*wav2vec 2.0*](https://arxiv.org/abs/2006.11477), and — for the transfer question specifically — Yosinski et al.'s [*How transferable are features in deep neural networks?*](https://arxiv.org/abs/1411.1792).

### The three SSL objective families

You should be able to recognise these three in a paper or an artefact card.

- **Masked-input reconstruction.** Mask a fraction of the input tokens (or patches) and train the model to reconstruct them. BERT for text (masked language modelling), MAE for images (masked auto-encoder — He et al., [*Masked Autoencoders Are Scalable Vision Learners*](https://arxiv.org/abs/2111.06377)). Trains representations that capture local context.
- **Contrastive learning.** Given a pair of views of the same input (a document and its noisy paraphrase, two augmentations of the same image), pull matched pairs together in embedding space and push unmatched pairs apart. SimCLR, MoCo (He et al., [*MoCo*](https://arxiv.org/abs/1911.05722)), CLIP for cross-modal. Trains representations invariant to nuisance augmentations.
- **Autoregressive prediction.** Predict the next token. GPT-family; also the natural fit for time-series and speech. Trains representations that capture sequential structure.

Which family fits depends on your downstream task and the structure of your data. Text: masked or autoregressive. Vision: contrastive or masked. Speech: usually contrastive or masked. If you are consuming a pre-trained backbone from a specialist track (fine-tuning, NLP), you rarely make this choice yourself — you inherit it.

### The transfer decision — fine-tune, linear-probe, or frozen features

Given a pre-trained backbone and a downstream labelled task, you have three levers. Pick by data size, distribution shift, and compute budget.

| Approach | What you update | When it wins | Cost | Risk |
|---|---|---|---|---|
| **Frozen features + shallow head** | Head only; backbone activations cached | Very small labelled set; backbone distribution close to yours | Cheapest | Ceiling limited by frozen representation |
| **Linear probe** | A single linear layer over frozen features | Small labelled set; want interpretable diagnostic of feature quality | Cheap | Same as above |
| **Head + top-k layers fine-tuned** | Head and top few backbone layers | Moderate labelled set; some distribution shift | Medium | Overfitting on small data |
| **Full fine-tune** | All backbone parameters | Large labelled set; significant distribution shift | Expensive | Catastrophic forgetting; overfitting; needs regularisation |
| **Parameter-efficient fine-tune (LoRA, adapters)** | Small added modules | Large backbone, moderate labels, tight compute | Medium | Ceiling below full fine-tune; extra artefact to manage |

Two rules of thumb from experience and the literature:

- **Distribution shift is the load-bearing variable.** If your data is close to the pre-training distribution (English news text vs. English social media text), lighter transfer works. If your data is far (English news vs. clinical notes vs. legal opinions in a low-resource language), you need to fine-tune more layers, and you should consider domain-adaptive pre-training as an intermediate step (Gururangan et al., [*Don't Stop Pretraining*](https://arxiv.org/abs/2004.10964)).
- **Linear probe first, always.** Even if you plan to full-fine-tune, run a linear probe on frozen features as a baseline. It tells you how much of your metric comes from the pre-trained representation versus your task-specific training. It is nearly free to run and it is the single best diagnostic of "is my transfer bet paying off?"

### Catastrophic forgetting and how to defend against it

Full fine-tuning updates every parameter of the backbone. If your fine-tuning set is small and narrow, the backbone can lose the general capability it was pre-trained for and become excellent at your task and useless at everything else. This matters when:

- The backbone is shared across multiple downstream tasks (a shared foundation model in an org).
- You expect the model to generalise beyond the fine-tune distribution.
- You may need to re-fine-tune later on new tasks and want the general capability preserved.

Defenses: low learning rates on the backbone (Howard and Ruder, [*ULMFiT*](https://arxiv.org/abs/1801.06146), popularised discriminative learning rates), parameter-efficient fine-tuning (LoRA — Hu et al., [*LoRA: Low-Rank Adaptation of Large Language Models*](https://arxiv.org/abs/2106.09685) — leaves the backbone intact and adds small trainable modules), elastic-weight consolidation (Kirkpatrick et al., [*Overcoming catastrophic forgetting in neural networks*](https://arxiv.org/abs/1612.00796)), or the pragmatic option of keeping the pre-trained checkpoint alongside the fine-tuned one.

The escalation question in chapter 08 becomes relevant here: full fine-tuning of a large model is often a specialist-track deliverable (`fine-tuning-engineer-learning`), not an ML-team deliverable. The senior read is: know when your fine-tune is small enough to run in-house and when to hand it off.

## Part 2 — Ensembling and distillation

### When ensembling is worth it

Ensembling reliably reduces variance. That means:

- **When your best single model is high-variance across seeds.** Train the same architecture with N different seeds; if the metric variance across seeds is a meaningful fraction of the improvement you are chasing, an ensemble of the seeds will produce a materially better single number.
- **When the failure modes across constituents are diverse.** The best ensembles combine constituents that are wrong in *different* ways — different architectures, different feature subsets, different training subsets. Averaging correlated errors buys you little; averaging uncorrelated ones is where the win comes from.
- **When calibration matters (chapter 05) and you cannot afford full Bayesian machinery.** Deep ensembles (Lakshminarayanan et al., [*Simple and Scalable Predictive Uncertainty Estimation using Deep Ensembles*](https://arxiv.org/abs/1612.01474)) are the pragmatic gold standard for both calibration and epistemic uncertainty on classification and regression tasks.

Ensembling is *not* the fix for systematic error. If your model is consistently wrong on a slice, N versions of the same model will be consistently wrong on the same slice. Chase the systematic issue in the data or loss before you burn budget on ensembling.

### The ensembling toolbox

- **Averaging predictions.** The simplest, near-universal baseline. Take the softmax outputs (or probabilities) from N models, average, argmax. Works when constituents are approximately calibrated; can degrade calibration if they are not.
- **Averaging logits.** Numerically stabler for softmax outputs and often better calibrated than probability averaging. Try both.
- **Stacking (blending).** Train a small meta-model on a held-out set that learns how to combine the constituents' outputs. Kaggle-canonical for tabular problems. Adds an operational artefact (the meta-model) and a held-out data requirement.
- **Boosting.** Sequentially train new models to correct the ensemble's residuals. Baked into gradient-boosted-tree libraries (XGBoost, LightGBM); the reason those libraries dominate on tabular problems is that they are ensembles internally.
- **Snapshot ensembling.** Save multiple checkpoints from a single training run at different learning-rate schedule minima and ensemble them (Huang et al., [*Snapshot Ensembles*](https://arxiv.org/abs/1704.00109)). Cheaper than training N independent models.

### The serving-cost problem

An ensemble of N models is N times the serving cost. In a p99 <100 ms budget with a 50 ms single-model inference, an ensemble of five does not fit. In an offline batch context you may accept the cost; in the request path you rarely can.

The pragmatic senior-altitude patterns are: (1) accept the ensemble tax if the metric justifies it and your serving stack can amortise it (batched inference on GPUs is often cheaper per prediction than the request-per-model cost suggests), (2) distil.

### Distillation — the ensemble-to-student pattern

Hinton, Vinyals, and Dean's [*Distilling the Knowledge in a Neural Network*](https://arxiv.org/abs/1503.02531) is the canonical reference. The recipe:

1. Train the teacher (usually an ensemble, sometimes a very large single model) on the labelled data.
2. Score a large unlabelled or lightly-labelled dataset with the teacher, producing soft targets (the full probability distribution, not just the argmax).
3. Train a smaller student to match the teacher's soft targets on that distillation set. Optionally combine with the hard-label loss.

The soft targets carry more information than the hard labels because they reveal the teacher's uncertainty over classes. The student learns to approximate the teacher's decision surface with a fraction of the parameters. In practice, distilled students can retain 90 %+ of the teacher's accuracy at 10x or greater speed-up.

Extensions to know by name: intermediate-layer distillation (Romero et al., [*FitNets*](https://arxiv.org/abs/1412.6550)), attention-map distillation (used in DistilBERT — Sanh et al., [*DistilBERT*](https://arxiv.org/abs/1910.01108)), self-distillation (student and teacher share architecture; sometimes improves generalisation).

### When distillation is worth it

- The teacher (ensemble or large model) has measurably higher offline metric than your best student-class architecture.
- The serving budget rules out shipping the teacher.
- You have a large unlabelled distillation pool (or a lightly-labelled one; label quality can be lower here because the teacher supplies the signal).

If any of those is false, do not distil. If you cannot ship the teacher because there is no student that fits the budget, distillation buys you the win. If your teacher is barely better than the student baseline, distillation is not worth the extra pipeline.

### The distillation pipeline as an operational artefact

Distillation adds a *second training pipeline* to your system: the teacher and the student are both retrained on a cadence, and the distillation dataset is a separate artefact with its own drift risks. The mod-302 chapter-04 skew categories apply here too — the distillation dataset's distribution has to match production, or you distil the teacher's behaviour on a distribution the student will not see in serving.

Concretely — the shape of the pipeline (pseudocode):

```python
# 1. Train / retrain teacher (ensemble or large single model)
teacher = train_teacher(labeled_train)

# 2. Score distillation pool with teacher
distill_pool = load_unlabeled_or_lightly_labeled()
soft_targets = teacher.predict_proba(distill_pool)  # full distribution

# 3. Train student on soft targets, optionally with hard-label term
student = train_student(
    distill_pool,
    soft_targets=soft_targets,
    labels=distill_pool.labels,  # if available
    alpha=0.7,                   # weight of soft-target loss vs. hard-label loss
    temperature=4.0,             # softens teacher's distribution
)

# 4. Serve the student
```

## Composing the toolbox

The regimes compose. A common industrial shape:

- Pre-trained (SSL / transfer) modality encoders per input modality — chapter 03 (multi-modal).
- Fine-tuned in a multi-task ranker — chapter 02 (MTL).
- Ensembled at training time across seeds for calibration and variance reduction.
- Distilled into a single student that fits the serving budget.
- Calibrated post-hoc — chapter 05.

Each layer of the composition is an independent architectural decision. Each layer has its own baseline (single-model no-ensemble no-distillation) that the launch review needs to compare against. If your evaluation harness (mod-305) cannot report per-layer contributions to the offline metric, the system is one incident away from being unmaintainable, because you cannot say which layer of the composition regressed when the metric drops.

## Three failure modes to catch in review

- **Fine-tuning from scratch when a pre-trained backbone was available.** The team labels 100 k examples and trains a small transformer from scratch; a linear probe on a public backbone would have beaten it in an afternoon. Ask for the linear-probe baseline on every SSL/transfer proposal.
- **Ensembling as a substitute for fixing systematic error.** The team ships a five-model ensemble because "it wins offline"; every constituent is wrong on the same minority slice, and the ensemble is wrong on the same slice. Ask for the per-slice metrics on the single best model *and* on the ensemble before signing off.
- **Distillation without a distillation-pool drift check.** The student regresses in production because the distillation pool was scored months ago and the production distribution has moved. The distillation pool needs the same drift monitoring as the training set.

## Summary

Self-supervised learning and transfer learning are how modern ML economically reaches new domains with small label budgets. The transfer decision — full fine-tune vs. top-k layers vs. linear probe vs. frozen features vs. parameter-efficient — is pinned by label size, distribution shift, and compute. Linear-probe first as a diagnostic; escalate as needed. Ensembling reliably reduces variance and improves calibration; it does not fix systematic error. Distillation buys back the ensemble's serving cost when the metric justifies it. The regimes compose — pre-trained encoders inside a multi-task ranker, ensembled, distilled, calibrated — and each layer of the composition needs its own baseline and its own eval-harness signal.
