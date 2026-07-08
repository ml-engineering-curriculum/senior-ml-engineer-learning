# Choosing the modeling regime

## Motivation

At L20 the modeling question is usually "which model family fits this table?" — gradient-boosted trees, a transformer, a two-tower recommender, whichever the paved-road stack blesses. That question has one axis (which architecture) and a well-worn playbook. At L30 the modeling question is a different shape: **what regime is this problem?** Do you train one model per label or one model across many correlated labels (multi-task)? Do you fuse text and image or keep separate specialists (multi-modal)? Do you have enough labels to train the head at all, or do you need to lean on a self-supervised or pre-trained backbone (SSL / transfer)? Does the deployment budget one model or a small ensemble (ensembling / distillation)?

Those decisions get baked into the *training data plan*, the *evaluation plan*, and the *serving budget* — all of which are extremely expensive to reverse once the team has spent a quarter on them. Picking the wrong regime is the modeling equivalent of picking the wrong serving posture in [mod-302] chapter 02: you feel it on day one and you keep paying for it for months.

This chapter is the **map**. It gives you the five regimes you should be able to reach for at senior altitude, the questions that pin each one down, and the failure modes each is prone to. The rest of the module drills into the regimes and the two cross-cutting concerns (calibration, cold-start).

## The five regimes to have in your pocket

You should be able to describe each of these in one paragraph and name at least one production-scale reference for each. Chapter numbers next to each name are where you go to drill in.

### 1. Multi-task learning (chapter 02)

One model, one backbone, several heads producing several correlated predictions. Ranking systems that jointly predict click, dwell, and purchase; content-safety models that predict a dozen policy categories on one text; medical-imaging models that jointly predict disease presence and severity. The bet is that the tasks share representation, so shared training makes each task's model better and cheaper than N single-task models would be.

Caruana's [*Multitask Learning*](https://link.springer.com/article/10.1023/A:1007379606734) (Machine Learning, 1997) is the canonical reference for why representation-sharing generalises. Ruder's [*An Overview of Multi-Task Learning in Deep Neural Networks*](https://arxiv.org/abs/1706.05098) is the modern survey. The failure mode to name is **negative transfer** — tasks that trade off against each other and degrade under joint training. Chapter 02 makes it concrete.

### 2. Multi-modal modeling (chapter 03)

One model consuming more than one modality — text + image + tabular, audio + text, image + metadata. Product-listing quality (text description + product image + category), medical triage (imaging + clinical notes), content-moderation triage (video + transcript + comments), embedding stores that mix modalities.

The core design decision is **fusion altitude** — do you fuse features early (concatenate modality embeddings before any prediction head), late (train per-modality models and combine their scores), or in the middle (cross-attention between modality-specific encoders). Baltrušaitis, Ahuja, and Morency's survey [*Multimodal Machine Learning: A Survey and Taxonomy*](https://arxiv.org/abs/1705.09406) is the canonical taxonomy. Chapter 03 walks the trade-offs.

### 3. Self-supervised learning and transfer learning (chapter 04)

You do not have enough labels to train the head from scratch, but you have a lot of unlabelled data (or someone else does). The regime is: pre-train a representation on a self-supervised objective — masked-token prediction, contrastive learning, next-token prediction — then fine-tune (or linear-probe) the small head you actually care about with the small labelled set you have.

This regime is what makes modern vision (ImageNet-pretrained backbones, DINO, CLIP), speech (wav2vec 2.0), and language (BERT, GPT) systems economically feasible at all. The bet is that the pre-training objective learns representations that transfer. The failure mode is **domain shift** — the pre-training distribution and your fine-tuning distribution are different enough that the transfer does not work.

The classic references are Devlin et al.'s [*BERT*](https://arxiv.org/abs/1810.04805), Chen et al.'s [*SimCLR*](https://arxiv.org/abs/2002.05709), and Radford et al.'s [*CLIP*](https://arxiv.org/abs/2103.00020). Yosinski et al.'s [*How transferable are features in deep neural networks?*](https://arxiv.org/abs/1411.1792) is the older-but-load-bearing paper on when transfer works and when it does not. Chapter 04 walks the fine-tune-vs-linear-probe-vs-frozen-features decision.

### 4. Ensembling and distillation (chapter 04)

You train several models and combine them — averaging, stacking, boosting-style residuals — because the ensemble is measurably better than any single model on the metric you care about. Then, because you cannot deploy N models under a p99 budget, you **distil** the ensemble into a single student model that approximates its behaviour.

The bet is calibration and variance reduction (the ensemble is more accurate and better-calibrated than any single model), and distillation is the way to buy back the serving-cost tax. Hinton, Vinyals, and Dean's [*Distilling the Knowledge in a Neural Network*](https://arxiv.org/abs/1503.02531) is the canonical reference. This regime shows up quietly in almost every winning Kaggle solution and in a lot of production ranking systems.

### 5. Cold-start, long-tail, label-scarce (chapter 07)

You do not have the training distribution you would want. New items, new users, rare classes, few labels for the tail. The modeling toolbox is **different from the head-of-distribution toolbox**: content-based features instead of collaborative signals for cold-start users; hierarchical or class-balanced losses for long-tail; active learning, weak supervision, or programmatic labelling for label-scarce.

This is not a fifth "advanced" regime — it is a *shape of problem* that forces you back through the first four regimes with different weights. But it is common enough at senior altitude that it earns its own chapter and its own decision rubric.

## The regime-selection rubric

Six questions, in order. The first one that forces a regime pins it; later questions cannot override.

### Q1 — How many correlated predictions is the system responsible for?

If the system produces one prediction and the loss is a single scalar, you are not in a multi-task regime. Move on.

If the system produces N correlated predictions (a ranker whose score is a weighted combination of click / dwell / purchase probabilities; a content-moderation model that emits a dozen policy-category probabilities; a medical model that emits a joint disease-and-severity distribution) and the correlations are strong enough that a shared representation should help, chapter 02 (multi-task learning) is on the table. Cross-check the "negative transfer" question in chapter 02 before you commit — the shared-backbone bet is not free.

### Q2 — How many modalities does the input span?

If the input is a single modality — text only, image only, tabular only — you are not in a multi-modal regime. Pick the family for that modality and move on.

If the input spans two or more modalities and the modalities carry different signal, chapter 03 (multi-modal) is on the table. The key sub-question is fusion altitude: some multi-modal problems are best solved by two independent single-modality models whose scores are combined at read time (late fusion), which is architecturally simple and lets each modality be owned by different teams; others need genuine cross-modality reasoning (early or cross-attention fusion), which is much harder to build and evaluate.

### Q3 — How much labelled data do you have relative to the model's capacity?

If you can train a strong model on your labelled data from scratch — the labelled set dwarfs the model's parameter count in the classical statistical sense, or at least trains the target head to convergence without overfitting — you are in the *supervised* regime. Skip to Q5.

If your labelled set is small (thousands to hundreds of thousands of examples, not millions) and you have a strong pre-trained backbone available, chapter 04 (SSL / transfer) is on the table. The decision splits into: fine-tune the whole backbone (best performance, most compute, biggest risk of catastrophic forgetting), fine-tune the top few layers (compromise), linear-probe on frozen features (cheapest, robust, weakest ceiling).

If your labelled set is small and no strong pre-trained backbone is available for your modality, you are in the *label-scarce* regime — chapter 07 (weak supervision, active learning, programmatic labelling) is on the table. This is common in specialised industrial domains where you cannot lift ImageNet or BERT.

### Q4 — Is the offline metric limited by variance or by systematic error?

If your best single model is already high-variance — small changes to the training seed swing the metric by more than the improvement you are chasing — an **ensemble** of independently trained models will help materially. That is the empirical read of ensembling: it reduces variance more reliably than it reduces bias. Chapter 04 covers when to ensemble and when to distil.

If your best single model is systematically wrong on identifiable slices (a class it consistently confuses, a demographic it consistently under-ranks), an ensemble will not fix it — the ensemble is systematically wrong for the same reasons every constituent is. Fix the systematic issue (data, features, loss) before you spend serving-cost budget on ensembling.

### Q5 — Do downstream decisions depend on calibrated probabilities?

If a threshold on your model's output triggers an automated action (auto-approve above 0.95, auto-decline below 0.05, page a human if between), the *number* matters, not just the ranking. You are then in the **calibration** regime — chapter 05. If a downstream cost model multiplies your probability by an expected-value estimate, same. Ranking-only systems (learning to rank, top-k retrieval) can often get away with uncalibrated scores; automated-decisioning systems cannot.

Uncertainty quantification (chapter 06) is a related but distinct question: even with a well-calibrated probability, do you need to know *how much you know* — to route hard cases to a human, to abstain, to reduce a bid? If yes, you are in the uncertainty regime as well.

### Q6 — Are you about to build something a specialist track owns?

The senior altitude read is that not every advanced-modeling problem should be built in-house on the ML team. Some of them are the deliverable of a peer specialist track:

- **LLM prompting, RAG, agents.** Those live in `llm-application-developer-learning` and `rag-engineer-learning`, not here.
- **LLM fine-tuning, RLHF, DPO.** Those live in `fine-tuning-engineer-learning`.
- **Applied NLP pipelines at production scale.** Those live in `applied-ai-engineer-learning`.
- **Distributed pre-training and training-pipeline engineering.** Those live in `training-pipeline-engineer-learning`.

Chapter 08 is entirely about the escalation decision. The L30 job is not to build all of these — it is to know which ones your team should build and which ones it should hand off to (or consume from) a specialist track.

## The map, at a glance

The rubric compresses to a diagram you can hold in your head:

```
                       ┌── one prediction ──────────┐
   Q1: N correlated? ──┤                            │
                       └── N predictions ──► MTL (ch 02) — check for negative transfer

                       ┌── one modality ────────────┐
   Q2: N modalities? ──┤                            │
                       └── N modalities ──► multi-modal (ch 03) — pick fusion altitude

                       ┌── labels ≫ capacity ───────► supervised, chapters 05–06 for calibration/uncertainty
   Q3: label regime? ──┤
                       ├── labels ≪ capacity, backbone available ──► SSL / transfer (ch 04)
                       │
                       └── labels ≪ capacity, no backbone ────────► label-scarce (ch 07)

   Q4: variance vs. systematic error? ── high variance ──► ensemble + distil (ch 04)
                                                              │
                                                              └── systematic → fix data / features / loss first

   Q5: probability-dependent decisions? ──► calibration (ch 05), uncertainty (ch 06)

   Q6: specialist-track territory? ────────► escalate (ch 08)
```

## A worked example — the same "harmful content triage" problem, three regimes

**Business problem.** "Route every user upload to auto-approve, auto-reject, or human review inside 500 ms."

The rubric produces different regimes under different constraint sets. Reviewers who have not seen this exercise reach for the *same* regime every time, which is the tell. The L30 read is that the regime is a function of the problem shape, not a preference.

### Shape A — text-only uploads, ten policy categories, calibrated auto-decisions

- Q1: ten correlated policy predictions → multi-task learning (chapter 02).
- Q2: text only → not multi-modal.
- Q3: ~1 M labelled examples, strong text backbone available → fine-tune a pre-trained encoder (chapter 04).
- Q5: 0.95 auto-reject threshold, 0.05 auto-approve threshold → calibration required (chapter 05).
- Q6: specialist territory? No — a fine-tuned encoder for policy classification is squarely in-house. RAG or LLM-prompted moderation would be an escalation (chapter 08).

→ Multi-task, transfer-learned, calibrated. Standard shape for text-only content-safety systems.

### Shape B — video uploads, five policy categories, one modality dominates per category

- Q1: five predictions → MTL is on the table.
- Q2: video (frames + audio + text transcript) → multi-modal (chapter 03). But each category is dominated by a different modality: nudity by frames, hate speech by transcript, threats by audio prosody. Late fusion is the natural fit — three modality specialists, one combiner.
- Q3: strong per-modality backbones for image, audio, and text → transfer per specialist (chapter 04).
- Q5: same auto-decision thresholds → calibration required (chapter 05).

→ Late-fusion multi-modal, per-modality transfer, multi-task combiner head, calibrated. Structurally different from shape A.

### Shape C — a low-resource language for which no strong pre-trained backbone exists

- Q1: same policy categories.
- Q2: text.
- Q3: ~10 k labelled examples, no strong pre-trained text encoder for this language → label-scarce (chapter 07). Programmatic labelling and cross-lingual transfer are on the table; from-scratch supervised training is not.
- Q6: cross-lingual transfer from a large multilingual model is squarely LLM-application territory — the escalation question (chapter 08) is whether the ML team owns this or hands it to `applied-ai-engineer-learning`.

→ Label-scarce regime, likely cross-lingual transfer, likely a hand-off to a specialist track. The point of chapter 08 is that the answer "this is not our team's problem" is a valid — and often correct — senior-altitude answer.

## Three failure modes the rubric catches

- **"Advanced regime because it's cool."** A team reaches for multi-task learning on a problem that has one prediction, or multi-modal fusion on a problem where one modality carries 95% of the signal. The regime tax (evaluation complexity, negative transfer, distributed data pipelines) is real and permanent. Q1 and Q2 catch this.
- **"Supervised from scratch because we didn't think about transfer."** A team spends a quarter labelling and training from scratch a model whose backbone could have been pre-trained from public data in an afternoon. Q3 catches this.
- **"Built it here because we didn't know a specialist owned it."** A team builds a RAG pipeline or a small fine-tuned LLM inside the ML org because "we're the ML team," and then discovers three months in that a peer specialist track has a paved road for exactly this. Q6 catches this. Chapter 08 is where you learn to make the call.

## Summary

At L20 the modeling question is "which model?" At L30 it is "which regime?" — multi-task, multi-modal, SSL / transfer, ensembling / distillation, or the special-case cold-start / long-tail / label-scarce shape. A six-question rubric — correlated predictions, modalities, label regime, variance vs. systematic error, probability-dependent decisions, specialist-track territory — pins the regime to the problem shape rather than to fashion. Calibration (chapter 05) and uncertainty (chapter 06) are cross-cutting concerns that live on top of whichever regime you picked. The escalation question (chapter 08) is the most senior-altitude question in the module: knowing which advanced-modeling problems are yours and which belong to a peer track.
