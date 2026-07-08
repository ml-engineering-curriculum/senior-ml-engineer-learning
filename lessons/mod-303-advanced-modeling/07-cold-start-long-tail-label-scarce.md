# Cold-start, long-tail, label-scarce

## Motivation

The methods in chapters 02–06 assume you have a training distribution that looks like your production distribution. In many production ML systems that assumption is violated on the day of launch and stays violated forever:

- **Cold-start.** New users have no interaction history. New items have no click data. A recommender that only knows how to score based on prior clicks scores those cases uniformly.
- **Long-tail.** A small number of head classes (or items, or users) account for most of the data; a very long tail accounts for a diminishing per-class share. A classifier trained end-to-end on the raw distribution learns the head perfectly and the tail badly.
- **Label-scarce.** You have plenty of unlabelled data (or supervisory signal in some form) and very few labelled examples. Straight supervised training does not fit.

These are not separate advanced regimes on top of the ones you already know — they are *shapes of the training distribution* that force you back through the earlier chapters with different weights. Cold-start pulls you toward content-based features and transfer learning (chapter 04); long-tail pulls you toward specialised losses and hierarchical modeling; label-scarce pulls you toward SSL (chapter 04), weak supervision, and active learning.

This chapter is the toolbox. The senior job is to recognise the shape early — before you have written the loss — and match it to the right tools.

## Part 1 — Cold-start

### The two flavours

- **Cold users.** The recommender / classifier / ranker has no history for this user. First session, first login, freshly created account.
- **Cold items.** The recommender / classifier has no interaction data for this item. Brand-new SKU, freshly indexed article, freshly uploaded video.

Both are common. Both are addressable. The tools differ.

### Tools for cold users

- **Content-based fallback.** Score cold users using features that do not require history — inferred demographics, referral source, contextual features (device, time of day, geography), the initial signup form. In production this is often a separate model whose output is blended into the ranker output for cold users.
- **Onboarding signals.** A short, high-signal onboarding flow (pick 3 categories, follow 5 accounts) generates enough history to switch into the warm regime quickly. Product decision, but usually the senior ML engineer negotiates the trade.
- **Exploration.** Explore-exploit strategies (Thompson sampling, ε-greedy on top of the cold-user model) accelerate warming by producing informative interactions. This is the natural bridge from cold-start into the [mod-306] experimentation-at-scale territory.
- **User-similarity transfer.** Use embeddings from warm users who match the cold user's demographic / behaviour features. Blends into the ranker as a similarity-weighted average of warm-user preferences.

### Tools for cold items

- **Content-based item embeddings.** Score cold items on features you can extract without interactions — the item's title, description, image, category, price. This is where multi-modal encoders (chapter 03) pull their weight for cold-start: a text-and-image embedding of a new SKU sits in the same embedding space as your warm items, so nearest-neighbour recommendations work from day one.
- **Two-tower models with content features.** In the classical two-tower retrieval architecture, the item tower is trained to accept content features (not just item ID embeddings), so cold items can be scored the moment they enter the catalogue. See the [YouTube recommendations paper](https://research.google/pubs/pub45530/) for the industrial pattern.
- **Item cold-start decay.** Track "days since first exposure" as a feature and let the ranker learn to blend content-based signal into interaction signal as history accumulates.
- **Exploration.** Same story as cold users — cold items need extra impressions to warm; explicit exploration budget accelerates it.

### The cold-start evaluation slice

Every recommender or ranker system that has cold-start users or items needs a **dedicated cold-start evaluation slice**. Aggregate metrics hide cold-start behaviour because the cold segment is a small fraction of impressions. Report:

- Metric on users with < N days of history (for chosen N — often 1, 7, 30).
- Metric on items with < N days of history.
- Coverage — what fraction of impressions are cold, and is that fraction going up (bad) or down (good).

This is the mod-305 hand-off: if the release harness cannot show cold-start metrics, the launch is unshippable. Cold-start regressions are hard to notice from aggregate metrics until the acquisition funnel goes sour.

## Part 2 — Long-tail

### The problem

Under a skewed class distribution — 90 % of examples fall in 10 classes, the remaining 10 % fall across 10 000 classes — a naive classifier learns the head accurately and the tail poorly. Zhang et al.'s [*Deep Long-Tailed Learning: A Survey*](https://arxiv.org/abs/2110.04596) is the modern reference.

There are two adjacent long-tail settings you should keep separate:

- **Long-tail classification** — many classes, most examples in a small subset.
- **Long-tail recommendation** — many items, most impressions on a small subset (Zipfian catalogue distribution). Different toolbox; overlap is partial.

### Tools for long-tail classification

Four families, in rough order of typical L30 reach-for.

- **Class-balanced sampling.** Oversample tail classes during training so batches see a more uniform class distribution. Cheap. Cui et al.'s [*Class-Balanced Loss Based on Effective Number of Samples*](https://arxiv.org/abs/1901.05555) gives a principled sample-count adjustment.
- **Class-balanced or focal loss.** Reweight the loss per class (inversely proportional to frequency, or effective-number-of-samples weighting). Focal loss (Lin et al., [*Focal Loss for Dense Object Detection*](https://arxiv.org/abs/1708.02002)) down-weights well-classified examples so hard tail examples dominate the gradient.
- **Decoupled representation + classifier training.** Kang et al.'s [*Decoupling Representation and Classifier for Long-Tailed Recognition*](https://arxiv.org/abs/1910.09217) argues that the *representation* is best learned on the natural (imbalanced) distribution, while the *classifier* head should be re-trained on a balanced distribution. Practically effective and cheap; often the strongest baseline.
- **Hierarchical modeling.** Model the class taxonomy explicitly — predict the top-level category first, then the sub-category within it. Puts more capacity where the data is (head) while giving tail classes a shared parent's signal.

### Tools for long-tail recommendation

- **Popularity de-biasing.** The interaction data itself is biased (users interact with what the ranker showed them). Train on log data with propensity weighting, use unbiased learning-to-rank (Joachims et al., [*Unbiased Learning-to-Rank with Biased Feedback*](https://arxiv.org/abs/1608.04468)).
- **Exploration budget.** Reserve a percentage of impressions for exploration to expose tail items.
- **Multi-objective ranking.** Add a "catalogue coverage" or "diversity" objective to the ranker so the head does not monopolise impressions. Composes with the MTL toolbox from chapter 02.

### Evaluation for long-tail

The metric that matters is not aggregate accuracy — it is per-tier accuracy. Split the classes into head / body / tail buckets (say, by frequency percentile), report the metric per bucket, and put a regression guard on each. Aggregate accuracy is the number that hides the failure.

## Part 3 — Label-scarce

### Recognising the regime

You are in the label-scarce regime when:

- Your labelled set is small enough that a from-scratch model overfits or underperforms.
- You have (or can get) a much larger *unlabelled* set from the same distribution.
- You have (or can get) *weak* supervisory signal — heuristic rules, aligned metadata, crowdsourced approximations.

Label-scarce is common in industrial ML: content-moderation categories that emerge quickly, low-resource languages, specialised medical or legal domains, brand-new products.

### Toolbox

Four tools, in the order you should reach for them.

#### 1. Transfer learning (chapter 04)

The first move. If a pre-trained backbone is available for your modality, fine-tune it on your small labelled set (or linear-probe if the label set is really small). The linear-probe baseline is nearly free and answers the question: "how much of my task can be done by frozen public representations?"

#### 2. Self-supervised pre-training on your own unlabelled corpus (chapter 04)

If your domain is far from the public pre-training distribution and you have a large unlabelled in-domain corpus, do a domain-adaptive SSL pre-training pass on your corpus, then fine-tune on the small labelled set. Gururangan et al.'s [*Don't Stop Pretraining*](https://arxiv.org/abs/2004.10964) documents the reliable gains.

#### 3. Weak supervision — programmatic labelling

Weak supervision converts human expertise into labels-at-scale by writing *labelling functions* (heuristics, patterns, external-model outputs) that each label a subset of the data with some noise, and then modeling the disagreement across the functions to produce a probabilistic label per example. The Snorkel line of work (Ratner et al., [*Snorkel: Rapid Training Data Creation with Weak Supervision*](https://arxiv.org/abs/1711.10160)) is the canonical reference; modern LLM-based labelling functions are the natural evolution.

The result is a large *noisy* labelled set that you train against — usually with confidence-weighted loss so noisier examples contribute less. Weak supervision is what turns a 5 000-example gold set and a domain expert into a 500 000-example noisy set at low marginal cost.

#### 4. Active learning — spend the labelling budget where it matters

If you have a fixed labelling budget, active learning tells you *which* unlabelled examples to send to a human labeller next. The two mainstream strategies:

- **Uncertainty sampling.** Pick examples where the current model is most uncertain (chapter 06 — epistemic uncertainty, not softmax entropy). Cheap and effective.
- **Diversity sampling.** Pick examples that cover under-represented regions of input space; often a k-means or coreset formulation. Guards against the uncertainty-sampling failure mode of oversampling one weird cluster.

Settles's [*Active Learning Literature Survey*](http://burrsettles.com/pub/settles.activelearning.pdf) is the classical reference. Modern active learning composes with ensembles for uncertainty (chapter 06 — ensemble disagreement is a strong active-learning signal), with weak supervision, and with LLM-driven pre-labelling (LLM proposes labels, humans confirm the uncertain ones).

### Data-centric mindset

The load-bearing shift at senior altitude in the label-scarce regime is from *model-centric* to *data-centric* — the win comes from the labelling function set, the corpus, and the labelling budget allocation, not from architecture search. Andrew Ng and colleagues' [*Data-Centric AI Resource Hub*](https://datacentricai.org/) is the vocabulary. In practice: the RFC for a label-scarce project should spend more pages on the data pipeline (weak-supervision functions, active-learning loop, gold-set curation) than on the model architecture.

## Composition — cold-start + long-tail + label-scarce often co-occur

A new product line with a new taxonomy: cold-start items, long-tail category distribution, small labelled set. The tools compose:

- Content-based embeddings for cold-start items — chapter 03 (multi-modal), chapter 04 (transfer).
- Class-balanced sampling and decoupled representation / classifier training for long-tail — this chapter.
- Weak supervision + active learning to grow the labelled set — this chapter.
- Uncertainty-based routing for genuinely hard cases — chapter 06.

Each piece is drawn from a different chapter of the module. The senior read is: pick the shape of the problem first (this chapter), then reach into the toolbox in the right order.

## Three failure modes to catch in review

- **"Cold-start is a corner case, we'll handle it later."** Cold-start is usually the growth funnel. Deferring it means new users and new items are silently underserved on day one. Cold-start eval slice is required from the first launch, not the second.
- **"We'll oversample the tail and call it done."** Oversampling helps modestly; decoupled classifier retraining or class-balanced loss usually helps more. Ship the strongest baseline, not the cheapest.
- **"We only have 10 k labels but we're going to fine-tune a 7B model."** Full fine-tuning of a large backbone on a small labelled set overfits and often loses the transfer benefit. Reach for linear probe or LoRA (chapter 04), or for weak supervision to grow the labelled set, before you spend the compute on full fine-tuning.

## Summary

Cold-start, long-tail, and label-scarce are shapes of the training distribution that force different tool selections from the earlier chapters. Cold-start is fixed by content-based features, two-tower architectures with content on the item side, and cold-start evaluation slices — not by more interaction data you do not have. Long-tail is fixed by class-balanced sampling, decoupled representation-and-classifier training, and per-tier evaluation — not by aggregate metrics that hide the tail. Label-scarce is fixed by transfer learning, weak supervision, active learning, and a data-centric mindset — the win comes from the labelling function set and the corpus, not from model architecture. These regimes often co-occur; the senior read is to recognise the shape early and compose the toolbox in the right order.
