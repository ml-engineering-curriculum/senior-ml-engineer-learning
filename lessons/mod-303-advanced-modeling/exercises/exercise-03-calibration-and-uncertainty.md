# exercise-03: Calibration and Uncertainty

**Estimated effort:** 4 hours

## Objective

Take a classifier — one you trained in exercise-02, or a fresh one on a chosen dataset — and put it through the full **calibration + uncertainty** pipeline from chapters 05 and 06:

- Reliability diagrams and per-slice ECE (chapter 05).
- Temperature scaling and one alternative calibrator (Platt / isotonic / Beta) (chapter 05).
- A deep-ensemble or MC-dropout uncertainty signal (chapter 06).
- A split-conformal prediction set / interval with a stated coverage guarantee (chapter 06).

Then write the launch-review memo that a peer L30 could sign off on, defending each artefact against a specific downstream decision.

The point of the exercise is muscle memory: after this exercise you should be able to plot a reliability diagram from memory, fit temperature scaling in twenty lines, and produce a conformal set without reaching for a paper.

## Prerequisites

- Chapters 05 (`05-calibration.md`) and 06 (`06-uncertainty-quantification.md`) read carefully.
- The classifier from exercise-02 or a fresh classifier. If fresh, pick a dataset where classes are meaningfully imbalanced or where slices matter — e.g., CIFAR-10 with a subset restricted to a few classes, a fraud-like tabular dataset, or a text classifier on AG News.
- Python + numpy + your training stack. `scipy`, `sklearn.isotonic`, and a plotting library are useful; no exotic dependencies required.

## Steps

### 1. Establish the base classifier and per-slice split (≈ 30 min)

- Confirm you have three disjoint splits: train / calibration / test. If you only have train / test today, cut a slice out of the training set (say, 10 %) into a dedicated calibration split. Fitting the calibrator on training data is one of the failure modes chapter 05 warns you about; do not skip this.
- Define at least three meaningful **slices** — subpopulations you will report separately. Class-based slices are the easy default; slices that matter more are ones that map to real product concerns (rare classes, hard classes, demographic groups, cold-start users, high-value transactions). Justify your slice choice in one sentence.

### 2. Plot the reliability diagram and report ECE — before calibration (≈ 45 min)

- On the calibration split, bin predictions into ~10 equal-width bins and plot mean predicted probability vs. empirical positive rate, per bin.
- Compute ECE overall and per slice. Report both.
- In one paragraph, describe what shape the miscalibration takes — under-confident, over-confident, non-monotonic? Which slices are worst?

Reference: Guo et al., [*On Calibration of Modern Neural Networks*](https://arxiv.org/abs/1706.04599).

### 3. Fit temperature scaling and one alternative (≈ 45 min)

- Implement temperature scaling from scratch (chapter 05 has the ~20-line PyTorch sketch). Fit `T` by minimising NLL on the calibration split. Report the learned `T` and the post-calibration reliability diagram + ECE.
- Fit one of Platt / isotonic / Beta calibration as the alternative. `sklearn.isotonic.IsotonicRegression` is one line; Platt / Beta are almost as easy.
- Compare the two calibrators on both aggregate and per-slice ECE. Which wins where? Interpret honestly.

### 4. Produce an epistemic-uncertainty signal (≈ 45 min)

Pick one:

- **Deep ensemble** — retrain the same classifier with 3–5 different seeds; at inference, aggregate predictions across the ensemble; report the standard deviation across constituents per input.
- **MC dropout** — enable dropout at inference and run 10–20 forward passes; report the standard deviation across passes.

Report:

- The distribution of the uncertainty signal across your test set.
- The correlation between the uncertainty signal and correctness (do harder examples get higher uncertainty? Compute AUROC of uncertainty vs. "is the top-1 prediction wrong?").
- A worked "abstain policy": at what uncertainty threshold does your test-set abstain rate hit 5 %? What is the accuracy on the *retained* examples?

### 5. Produce a split-conformal prediction set with a stated coverage guarantee (≈ 30 min)

- Choose a target coverage — 90 % is standard for exercise purposes.
- On the calibration split, compute the non-conformity score per example (for classification, `1 − p(true class)`).
- Take the `(1 − α)`-quantile.
- On the test split, produce the prediction set (all classes whose score is at most the quantile). Report empirical coverage on the test split — it should land close to your target. Report the *average set size* and the set-size distribution per slice.

Reference: Angelopoulos and Bates, [*A Gentle Introduction to Conformal Prediction*](https://arxiv.org/abs/2107.07511).

### 6. Write the launch-review memo (≈ 60 min)

Sections (keep it to two pages):

- **Setup.** Dataset, classifier, splits, slice definitions.
- **Pre-calibration diagnostic.** Aggregate + per-slice reliability diagrams and ECE.
- **Calibration.** Which calibrator you chose (and why), post-calibration ECE aggregate + per slice.
- **Uncertainty.** Which method, uncertainty-vs-correctness AUROC, worked abstain policy.
- **Conformal set.** Target coverage, empirical coverage, average set size, per-slice set size.
- **Downstream decision defence.** Pick one downstream decision (auto-approve above 0.95, abstain-and-route policy, prediction-set display) and defend how each artefact supports it. Include the fallback (what if calibration or coverage regresses at retrain time?).

## Deliverable

- A notebook or repo containing the reproducible code for each step, plots exported to PNG, and the artefacts (calibrator parameters, conformal quantile).
- The launch-review memo `calibration-uncertainty-review.md`.

## Acceptance criteria

- [ ] Calibration set is genuinely held out from training and the calibrator is fit on it.
- [ ] Reliability diagram and per-slice ECE are reported both before and after calibration.
- [ ] At least two calibrators are compared; the choice between them is justified with numbers, not preference.
- [ ] Epistemic-uncertainty signal is produced by a method chapter 06 endorses; correlation with correctness is reported, not asserted.
- [ ] Split-conformal is implemented from scratch (or via a library, but the mechanics are described in the memo); empirical coverage on the test set is reported and lands close to the target.
- [ ] Per-slice numbers are reported for every metric where slices exist; no aggregate-only reporting.
- [ ] The memo names a specific downstream decision the artefacts support and describes the fallback if any of them regresses.
- [ ] The memo is at most two pages.

## Stretch goals

- **Distribution-shift stress test.** Split the test set into "close to training distribution" and "far from training distribution" slices (by feature-space distance, or by an explicit domain axis). Report ECE and empirical conformal coverage on each. Does coverage degrade under shift, as the theory warns?
- **Adaptive conformal.** Implement adaptive conformal prediction (Gibbs and Candès, [*Adaptive Conformal Inference Under Distribution Shift*](https://arxiv.org/abs/2106.00170)) and compare against split conformal on a shifted stream.
- **Class-conditional conformal.** Implement per-class (Mondrian) conformal to get per-class coverage guarantees. Discuss when this matters (regulated slice-based coverage) and when it does not.
- **Composed pipeline artefact.** Package the base classifier + temperature `T` + conformal quantile as a single deployable artefact with one `predict(x) → (probability, prediction_set)` interface. Discuss the versioning and rollback implications.
- **Feed into mod-305.** If you are also going through mod-305 (advanced evaluation), reuse the per-slice reliability diagrams and per-slice conformal coverage as evaluation-harness metrics with regression guards.
