# exercise-02: Multi-Task Learning in Practice

**Estimated effort:** 4 hours

## Objective

Build a multi-task model on a real dataset, produce the **per-task single-task-baseline table** from chapter 02, and diagnose whether MTL wins, ties, or loses on each task. Deliverable is a small experiment repository (or notebook) plus a short launch-review memo — not a fully productionised system.

The point is to get first-hand experience with the two failure modes chapter 02 warns you about: **loss balancing** across tasks with different loss magnitudes, and **negative transfer** on task subsets. You cannot know either exists until you compare against the single-task baseline, and you cannot make the MTL launch defensible until you have the comparison table.

## Prerequisites

- Chapter 02 (`02-multi-task-learning.md`) read carefully — the architectural shapes, the loss-balancing families, and the negative-transfer diagnostic are all load-bearing.
- Python + PyTorch (or TensorFlow / JAX equivalents) at a comfortable working level.
- Access to a modest compute environment — CPU-only is enough for the reference datasets, though a small GPU accelerates the loop.

## Pick a dataset and task set

Pick **one** of the following and identify at least three tasks the dataset supports. Do not pick a dataset where the tasks are obviously the same target relabelled.

- **UCI Adult** — jointly predict `income > 50k`, `marital-status`, and `occupation` from the demographic features. Tabular, small, fast to iterate. Fine as the "get the pipeline working" dataset.
- **CelebA attributes** — jointly predict several facial attributes (e.g., `smiling`, `eyeglasses`, `young`). Classical MTL benchmark; strong pre-trained image backbone available. Note: as the wider curriculum's governance module (mod-309) discusses, some CelebA attributes have well-documented labelling issues and demographic sensitivities — use this as a technical exercise, not a production template, and stay away from the attributes flagged as problematic in the literature.
- **AG News + a derived multi-task setup** — predict the topic (4 classes) and a second and third derived task (e.g., predicted length bucket, presence of a proper noun via a heuristic labelling function). Text, small, easy to iterate.
- **Bring-your-own** — a real dataset from your work (anonymised) with at least three genuinely correlated tasks.

Whichever dataset you pick, do not skip the "genuinely correlated" filter. MTL on three uncorrelated tasks is not a fair test.

## Steps

### 1. Establish the single-task baselines (≈ 60 min)

Train one model per task, on the same features, with the same backbone architecture and the same regularisation. Record:

- Best single-task metric per task (accuracy / AUC / F1 / whatever fits the task).
- Loss magnitude per task at the end of training.

This is the baseline table you will compare against. Save it as a Markdown table in the memo.

### 2. Build the MTL model — hard parameter sharing (≈ 30 min)

One shared backbone (same architecture as the single-task baseline's backbone), N cheap task-specific heads, one loss per head. Start with **equal loss weights** (`w_i = 1`). Train.

Record the same per-task metrics on the same eval split.

### 3. Diagnose loss-balancing pathologies (≈ 30 min)

- Log per-task loss magnitude at the end of training. Are they within an order of magnitude of each other, or is one dominating?
- Try at least one of the fixes from chapter 02:
  - Hand-tuned weights (`w_i` proportional to inverse loss magnitude, or a small grid search).
  - Uncertainty weighting (Kendall, Gal, Cipolla).
- Retrain and record.

You should now have two or three MTL configurations to compare against the single-task baselines.

### 4. Compare against the baselines and diagnose negative transfer (≈ 30 min)

Produce the comparison table (single-task vs. MTL-equal-weights vs. MTL-tuned):

| Task | Single-task metric | MTL (equal weights) | MTL (tuned / uncertainty) |
|---|---|---|---|
| Task A | 0.87 | 0.86 | 0.88 |
| Task B | 0.72 | 0.71 | 0.73 |
| Task C | 0.64 | 0.58 | 0.61 |

Interpret honestly:

- Which tasks won under MTL, tied, or lost?
- If any task lost, is it negative transfer (task interference) or under-training (needs more epochs)? Run a controlled experiment to distinguish — e.g., train longer, or train that task alone with the shared backbone frozen at the MTL trunk's weights.
- If negative transfer is confirmed, what would you propose next? Drop the losing task, split into two MTL groups, escalate to MMoE?

### 5. Write the launch-review memo (≈ 60 min)

The deliverable memo has these sections. Keep it to two pages.

- **Setup.** Dataset, tasks, backbone architecture, training details in one paragraph.
- **Baseline table.** Single-task metrics + the MTL configurations you tried.
- **Loss balancing.** Which pathologies you saw, which fix you applied, why.
- **Negative-transfer diagnosis.** Which tasks lost (if any), whether you confirmed it was interference vs. under-training.
- **Decision.** Would you ship this MTL model? On which subset of tasks? What is your fallback if a task regresses in the next retrain?
- **What you would do next if this were a real launch.** One paragraph. Deep-ensemble it? Distil? Move to MMoE? Split the task groups?

## Deliverable

A repository or single notebook containing:

- The training script(s) — reproducible, seeded.
- The comparison-table code and the numeric outputs.
- A Markdown memo `mtl-launch-review.md` with the sections above.

You do not need a fully productionised training pipeline; you need the artefact that a reviewer can read in five minutes and know whether the launch is defensible.

## Acceptance criteria

- [ ] Single-task baselines exist for every task, using the same backbone as the MTL model.
- [ ] At least two MTL configurations are compared (equal weights + at least one loss-balancing fix).
- [ ] The comparison table shows per-task metrics, not just an aggregate.
- [ ] Loss-magnitude distribution across tasks is reported and interpreted.
- [ ] Any per-task regression is diagnosed (negative transfer vs. under-training vs. other), not hand-waved.
- [ ] The memo names a specific ship / do-not-ship / redo recommendation and defends it.
- [ ] Follow-up options (drop task, split groups, MMoE, distillation, calibration downstream) are named where relevant.
- [ ] The memo is at most two pages.

## Stretch goals

- **MMoE variant.** Reimplement the MTL model as a small MMoE (a few experts, per-task gates) and add its row to the comparison table. Comment on whether the added complexity pays for itself.
- **Uncertainty weighting.** Implement Kendall-Gal-Cipolla uncertainty weighting from scratch and compare against hand-tuned weights.
- **PCGrad or GradNorm.** Implement one gradient-manipulation method and add its row.
- **Ablate the backbone size.** Does MTL win by more on small backbones (where representation-sharing pays) or on large backbones (where capacity is not the bottleneck)?
- **Cross-task confusion analysis.** For classification tasks, plot the per-task confusion matrix under single-task and MTL. Are the confusions the same, or is MTL trading one confusion pattern for another?
- **Contribute a decision to the paired project.** If you are also working on `project-301-ml-system-design-portfolio`, reuse the memo as the "modeling regime" section of your RFC.
