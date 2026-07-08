# Multi-modal modeling: fusion altitude

## Motivation

Most interesting production problems have more than one input modality. A product-listing quality model sees the title, the description, the primary image, the seller reputation, and the category. A content-moderation triage system sees video frames, audio, transcribed captions, and prior user reports. A medical-triage model sees an X-ray, the free-text referral note, and the patient's coded history. In every one of those, using only one modality throws away signal; using them all raises a design question — **at what altitude do you fuse them?**

That question — early, middle, or late fusion — is the single most consequential architectural decision in a multi-modal system. It determines: how the training data is joined, how each modality is versioned, which team owns which encoder, how the serving stack routes requests, and how you evaluate on modality-specific failure modes. Baltrušaitis, Ahuja, and Morency's [*Multimodal Machine Learning: A Survey and Taxonomy*](https://arxiv.org/abs/1705.09406) is the standard taxonomy. This chapter gives you the L30 decision rubric grounded in that taxonomy.

## The three fusion altitudes

### Late fusion — modality specialists combined at the score level

Each modality is scored by its own specialist model. The specialist outputs (scores, logits, or embeddings) are combined by a small combiner — a weighted average, a small MLP, a stacked classifier — that produces the final prediction.

```
      ┌─ text encoder ───► p_text  ┐
input ┼─ image encoder ──► p_image ┼──► combiner ──► prediction
      └─ tabular encoder ► p_tab   ┘
```

Properties:

- **Cheap to build and to reason about.** Each specialist has a single-modality training set, a single-modality eval harness, a single team that owns it.
- **Robust to missing modalities.** If the image is missing, the image specialist returns a "no signal" score and the combiner falls back on text and tabular.
- **Versionable per modality.** You can swap the text encoder without retraining the image encoder.
- **Limited representation.** The combiner sees only the specialists' scores or embeddings, not their internal representations. If a decision requires *combining* what a specific word means with a specific region of the image, late fusion cannot express it.

Late fusion is the correct default at senior altitude for most industrial multi-modal systems. The team-ownership property alone justifies it — each modality can be maintained by a different subteam without cross-modality coordination on every release.

### Early fusion — one shared encoder consumes concatenated features

All modalities are converted to a shared representation (e.g., all modalities are tokenised and stacked into one sequence a transformer consumes) and one model does the whole prediction from that shared representation.

```
      ┌─ text tokens ───┐
input ─┼─ image patches ─┼─► one shared encoder ──► prediction
      └─ tabular embeds ─┘
```

Properties:

- **Maximum expressivity.** The encoder can attend across modalities at every layer. If the decision depends on a fine-grained cross-modality interaction, early fusion is the shape that can learn it.
- **Data-hungry.** You need paired examples with all modalities to train the encoder, and you cannot easily borrow single-modality pre-trained weights.
- **Brittle to missing modalities.** A serving-time request missing one modality is a distribution-shift problem the encoder was not trained for; you need to explicitly train with modality dropout to be robust.
- **Ownership challenges.** One encoder, one team, one training run. Cross-modality coordination is on every release.

Early fusion is what you reach for when the task genuinely needs cross-modality reasoning and you have paired data at scale.

### Middle fusion — cross-attention between per-modality encoders

Each modality has its own encoder that produces a sequence of embeddings, and the encoders share information through cross-attention layers. This is the shape of most modern vision-language models — CLIP-style dual encoders trained contrastively (Radford et al., [*CLIP*](https://arxiv.org/abs/2103.00020)), Flamingo-style perceivers (Alayrac et al., [*Flamingo*](https://arxiv.org/abs/2204.14198)), BLIP-2's Q-Former (Li et al., [*BLIP-2*](https://arxiv.org/abs/2301.12597)).

Properties:

- **Combines the strengths of late and early fusion.** Each modality's encoder can be pre-trained separately (transfer learning stays available), while cross-attention layers let the model learn cross-modality interactions.
- **Serving cost is intermediate.** Each modality is encoded once; cross-attention adds a fixed multiplier.
- **Ownership can be split by encoder plus a shared combiner team.** Better than early fusion, worse than late.

Middle fusion is what most modern multi-modal foundation models use. In the industrial context this chapter is about, you often *consume* a middle-fusion foundation model (CLIP embeddings, a BLIP-2 head) as the "specialist" inside a larger late-fusion system.

## The fusion-altitude decision rubric

Five questions, in order.

### Q1 — Does the decision require cross-modality *reasoning*, or just cross-modality *evidence*?

A product-listing model that combines an image quality score and a text quality score into an overall quality score needs cross-modality **evidence** — each modality contributes a signal, but the modalities do not need to reason jointly about specific parts of one another. Late fusion is a fit.

A visual-question-answering model that needs to identify a specific object in the image based on a specific word in the question needs cross-modality **reasoning** at the token level. Late fusion cannot express it. You need middle or early fusion.

If you are not sure which category the task falls in, prototype late fusion first. Measure. If the late-fusion combiner cannot capture the signal, escalate.

### Q2 — Do you have paired training data at scale?

Early and middle fusion require examples where *every* modality is present and labelled together. Late fusion can be trained on the union of single-modality datasets and only needs paired data for the combiner (much smaller).

If your paired dataset is small (say, tens of thousands of examples), and you have larger single-modality datasets for each modality, late fusion is the natural fit and lets you leverage the larger single-modality corpora. If you have millions of paired examples, early and middle fusion become viable.

### Q3 — What is the missing-modality behaviour at serving time?

If serving-time requests reliably contain every modality — say, a product listing always has a title, a description, and a primary image because your product enforces it — early fusion is not a robustness risk.

If any modality can be missing at serving time — user uploads that sometimes have no image, medical records that sometimes have no radiology report — late fusion has the graceful-degradation property essentially for free (the specialist returns a "no signal" score) and the other altitudes need explicit modality dropout during training to match it. Missing-modality behaviour must be an explicit test in the release harness (hand-off to [mod-305]).

### Q4 — Who owns each modality's model, and how often does each modality change?

Ownership is an underrated axis. Late fusion lets modality teams work independently — the text team ships a new text encoder, retrains the combiner, done. Early fusion couples every modality to every retrain: if the image team wants to swap encoders, the whole model retrains.

If your organisation has modality specialists on different teams (a CV team, an NLP team, a tabular team), late fusion aligns with the organisational structure and is much cheaper to operate. This is a real, load-bearing consideration in most industrial contexts. (It is a variant of Conway's law — your model shape ends up mirroring your org shape whether you plan it or not.)

### Q5 — What is the serving budget?

Late fusion scales serving cost roughly as the sum of per-modality specialist costs plus a small combiner. Middle fusion adds cross-attention on top. Early fusion is roughly linear in the total token / patch count.

If p99 latency is tight, the specialists in a late-fusion system can be smaller than the shared encoder in an early-fusion system, because each specialist has less to do. That is often the deciding factor at industrial scale.

## Multi-modal + multi-task — the composite regime

The two regimes compose. Many production systems are **multi-modal in the input and multi-task in the output**. A YouTube-scale ranker sees text, image, and behavioural features; it produces click, dwell, and purchase probabilities. The serving-cost economics (chapter 02: MTL) push you to share a trunk across tasks; the fusion-altitude decision (this chapter) pushes you to compose that trunk carefully.

The most common shape at industrial scale: **modality-specific encoders → concatenated / cross-attention'd shared trunk → per-task heads**. The modality encoders are usually pre-trained (chapter 04: transfer). The trunk plus heads are trained end-to-end on your task data. The evaluation harness (mod-305) reports metrics per task and, separately, per modality-availability slice (with all modalities, missing image, missing text, and so on).

## Handling missing modalities — the corner case that trips releases

Every multi-modal system fails at least one release check on missing-modality behaviour. The patterns that work:

- **Explicit missing-modality tokens.** Reserve a special token / embedding for "modality absent" and train the model with modality dropout so it learns to handle it. This is what most late-fusion systems do.
- **Modality-dropout regularisation.** During training, randomly drop entire modalities from a fraction of batches. The model learns to produce sensible predictions from any subset. This is what you do to make early or middle fusion robust to missing modalities.
- **Per-modality reliability signal.** In late fusion, expose a "confidence" per specialist so the combiner can down-weight missing or noisy modalities. This is a small architectural investment that pays back on every release.

The release harness must have an explicit slice for each realistic missing-modality combination. Discovering a missing-modality regression from user reports rather than from the eval harness is a mod-305 hand-off failure.

## A worked example — product-listing quality

**Business problem.** "Score the quality of a seller's product listing so that low-quality listings are demoted in search."

Inputs available: title (text), description (text), primary image (image), category (categorical), seller quality score (numeric), price (numeric).

Rubric walk:

- **Q1 — reasoning or evidence?** Evidence. Each modality contributes a signal (is the title spammy, is the image blurred, is the description misleading) and the combiner sums them. → Late fusion is on the table.
- **Q2 — paired data at scale?** Single-modality datasets are much larger than paired-labelled listings (millions of images, hundreds of millions of texts vs. a hundred thousand human-quality-labelled listings). → Late fusion lets you leverage the large single-modality corpora.
- **Q3 — missing-modality behaviour?** Rare (product taxonomy enforces at least a title and one image) but non-zero (missing description happens). → Late fusion is robust for free.
- **Q4 — ownership?** The text team owns the text encoder, the CV team owns the image encoder, the marketplace-quality team owns the combiner. → Late fusion aligns with org shape.
- **Q5 — serving budget?** p99 <150 ms in the search pipeline. Late fusion with small distilled specialists fits.

→ **Late fusion.** Text encoder (distilled from a public backbone), image encoder (distilled from a public backbone), combiner (small MLP over the specialist embeddings plus the raw tabular features). Multi-task heads if the same trunk also predicts spam probability and category-mismatch probability.

Contrast: the *same* problem, one modality dominates. If 95% of the quality signal is in the text, a single-modality text model plus a rule on image presence would be a better v1 than any multi-modal shape. The rubric's job is to prevent you from building a multi-modal system for a problem that is not actually multi-modal.

## Three failure modes to catch in review

- **"Multi-modal because we have multi-modal data."** The team joins text and image because both are available, without checking whether the additional modality carries signal. Ablation: train each single-modality baseline and confirm the multi-modal system beats the strongest single-modality baseline by a meaningful margin. If it does not, ship the single-modality baseline.
- **No modality-dropout evaluation.** The system passes offline metrics on the paired eval set, then regresses at launch because a non-trivial fraction of real requests are missing one modality. Fix: modality-availability slicing is part of the release harness.
- **Late-fusion combiner is a black box.** The specialists' scores go into an MLP that no one can inspect, and when quality regresses the team cannot say which modality caused it. Fix: the combiner is small and interpretable, or the specialists' scores are logged so post-hoc attribution is possible.

## Summary

Multi-modal modeling is a fusion-altitude decision — late, middle, or early — pinned by five questions: reasoning vs. evidence, paired-data availability, missing-modality behaviour, ownership shape, serving budget. Late fusion is the correct default at industrial scale for most systems: it composes with per-modality transfer learning, it composes with organisational structure, and it degrades gracefully on missing modalities. Early and middle fusion are what you escalate to when the task genuinely requires cross-modality reasoning and you have paired data at scale. The multi-modal + multi-task composite — modality-specific encoders feeding a shared trunk feeding per-task heads — is the most common industrial shape. Every multi-modal system needs modality-availability evaluation slices; discovering a missing-modality regression from user reports is a release-harness failure.
