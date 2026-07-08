# exercise-04: Cold-Start and Long-Tail Tactics

**Estimated effort:** 3 hours

## Objective

Design a modeling plan for a system whose training distribution has one or more of the three shapes chapter 07 covers: **cold-start**, **long-tail**, or **label-scarce**. Deliverable is a design memo and — for the top-priority tactic — a small proof-of-concept implementation or evaluation-slice specification.

The point is to force you to reason about the *training-distribution shape* before you reach for a model architecture. Most of the win in these regimes comes from picking the right tactic against the right shape, not from architecture search. At L30 the muscle you are training is the recognise-the-shape-early muscle.

## Prerequisites

- Chapter 07 (`07-cold-start-long-tail-label-scarce.md`) read carefully.
- Chapter 04 (`04-self-supervised-transfer-and-ensembling.md`) at least skimmed — several cold-start and label-scarce tactics compose with transfer learning.
- Familiarity with the mod-305 evaluation vocabulary (per-slice metrics, regression guards) — you will spec evaluation slices in step 4.

## Pick a scenario

Pick **one** of the following. Each has at least two of the three shapes explicitly baked in — cold-start + long-tail + label-scarce co-occur in practice.

- **S1 — Marketplace listing quality on a rapidly expanding catalogue.** New SKUs arrive daily (cold-start items). Catalogue class distribution is Zipfian (long-tail categories). Human-labelled quality set is ~20 k listings; unlabelled corpus is ~50 M listings.
- **S2 — Content-moderation triage for a newly launched product surface.** No historical interaction data yet (cold-start users and content); moderator queue can label ~500 posts/day (label-scarce); policy category distribution is long-tail (a few common violations, many rare ones).
- **S3 — Recommender for a specialised streaming platform.** Users span from days-old accounts (cold-start) to years-old accounts (warm); catalogue skew is Zipfian; ~5 M actives, ~500 k items.
- **S4 — Clinical triage classifier for a low-resource-language deployment.** ~5 000 labelled records in the target language; strong pre-trained encoders exist for other languages; long-tail category distribution across diagnosis codes.
- **S5 — Bring-your-own.** A real system where at least two of the three shapes apply. Anonymise anything sensitive.

## Steps

### 1. Diagnose the shape — one section per applicable shape (≈ 30 min)

For each of cold-start, long-tail, label-scarce that applies to your scenario, write one paragraph naming:

- **What kind is it?** Cold users, cold items, both? Long-tail classes, long-tail items, both? Label-scarce with abundant unlabelled data, or scarce across the board?
- **What is the size of the shape?** What fraction of impressions / examples fall in the cold or tail bucket? What is the labelled-to-unlabelled ratio?
- **What is the cost of getting it wrong?** New-user retention? Under-represented-class fairness? Regulatory coverage?

Do this diagnosis **before** you propose tactics. Rushing to tactics is the failure mode.

### 2. Propose the toolbox — one tactic per shape, defended (≈ 45 min)

For each shape, pick one tactic from chapter 07's toolbox and defend it in one paragraph.

- **Cold-start.** Content-based fallback? Two-tower with content features on the item side? Onboarding-driven warming? Exploration budget?
- **Long-tail.** Class-balanced sampling? Class-balanced or focal loss? Decoupled representation-and-classifier training? Hierarchical modeling?
- **Label-scarce.** Transfer / linear probe? Domain-adaptive SSL pre-training? Weak supervision (Snorkel-style)? Active learning?

Then, if multiple shapes apply, name **the order** you would implement the tactics in and why. Often the label-scarce fix (weak supervision) has to precede the long-tail fix (per-class metrics), because you need the labels first.

### 3. Design the evaluation slices (≈ 30 min)

For each shape you diagnosed, name at least one evaluation slice that must be tracked as a regression guard.

- **Cold-start.** Metric on users with < 7 days of history. Metric on items with < 7 days since first exposure. Cold coverage — what fraction of impressions are cold, trending which way.
- **Long-tail.** Metric per class-frequency bucket (head / body / tail, or explicit percentiles). Regression on the tail bucket blocks launch even if aggregate wins.
- **Label-scarce.** Metric on the small held-out gold set. Metric on subsets not covered by any weak-supervision labelling function (the "novel" slice — the gold set that the weak-labelling pipeline is not familiar with).

Every slice needs a threshold and a regression-guard behaviour ("block launch" vs. "notify but ship"). Reference the mod-305 discipline.

### 4. Build the small proof-of-concept (≈ 60 min)

Pick **one** of the tactics from step 2 and implement it end-to-end on a tractable slice of the problem (real data if available; a public reference dataset if not).

- Class-balanced sampling on a UCI or CIFAR long-tail split.
- A weak-supervision setup with three labelling functions on a small text dataset, combined into probabilistic labels.
- A two-tower model with content features on a public MovieLens or similar recommender split.
- A linear probe on top of a public text or image backbone on a small labelled set.

The PoC does not have to hit a benchmark. It has to be reproducible, evaluated against a fair baseline, and honest about where it fell short.

### 5. Write the design memo (≈ 45 min)

Sections (keep it to two pages plus one page for the PoC results):

- **Setup.** Scenario, applicable shapes, sizes.
- **Toolbox choice per shape.** One paragraph per shape, defending the tactic.
- **Order of operations.** Which tactic first and why.
- **Evaluation slices.** Named slices with thresholds and regression-guard behaviour.
- **PoC results.** What you built, the baseline it beats (or does not), what the honest read is.
- **What you would do next if this were the first quarter of a real project.**

## Deliverable

- The design memo `cold-start-long-tail-plan.md`.
- The PoC code (repo or notebook), reproducible and seeded.
- A one-page results appendix comparing your PoC against the honest baseline you picked (single-task supervised, uniform sampling, no content features, whatever the fair "did nothing about the shape" baseline is).

## Acceptance criteria

- [ ] Each applicable shape is named, sized, and its cost-of-wrong defended.
- [ ] Each tactic is defended by reference to a specific tool in chapter 07 (or an explicit note that the tactic is out-of-chapter and why).
- [ ] The order of operations is explicit and defensible.
- [ ] Every evaluation slice has a defined threshold and a regression-guard behaviour.
- [ ] The PoC is honestly compared against a "did nothing" baseline; the reader can tell what fraction of the win came from the tactic.
- [ ] The memo names at least one thing the plan is *not* addressing in v1 (a non-goal, mod-302 chapter 06 style).
- [ ] The memo is at most three pages including the PoC results.

## Stretch goals

- **Stack the tactics.** If your scenario has two applicable shapes, implement both tactics in the PoC and ablate: shape-A-only, shape-B-only, both. Which contributes more? Any negative interaction?
- **Active-learning loop.** For a label-scarce scenario, implement one iteration of an active-learning loop — score unlabelled examples with your current model, pick top-K by uncertainty, simulate labelling them (using held-out labels), retrain, re-evaluate.
- **Weak supervision from LLM labellers.** Use an LLM as one of your labelling functions in a Snorkel-style pipeline. Discuss the failure modes an LLM labeller introduces (correlated errors, prompt drift, confident-but-wrong regions).
- **Cold-start evaluation cross-check.** For a recommender scenario, verify that the aggregate metric hides the cold-start behaviour — plot metric vs. days-of-history and show that a launch decision made on the aggregate would have missed a cold-start regression.
- **Feed into the paired project.** Use the design memo as the modeling-plan input to `project-301-ml-system-design-portfolio` or `project-303-tech-lead-simulation`.
