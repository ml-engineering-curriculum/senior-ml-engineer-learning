# Calibration: making the probability mean what it says

## Motivation

Modern classifiers output numbers between 0 and 1 that look like probabilities. Most of the time they are not. A deep network that scores a transaction at 0.97 fraud may be correct 80 % of the time, not 97 %. A gradient-boosted classifier that outputs 0.05 for benign cases may in fact be wrong 20 % of the time. The scores are useful for **ranking** — the 0.97 case is riskier than the 0.05 case — but they are not usable as **probabilities** in downstream calculations.

That gap matters wherever a downstream decision depends on the number, not the rank:

- **Automated decisioning.** "Auto-decline above 0.95" only works if 0.95 means what you think.
- **Expected-value calculations.** "Bid = probability * value" is nonsense if the probability is off by a factor of two.
- **Human review routing.** "Send anything between 0.4 and 0.6 to review" only works if that interval means "genuinely uncertain."
- **Cross-model composition.** If model A's score is fed into model B's cost function, a miscalibrated A silently corrupts B.

Calibration is the discipline of making the probability mean what it says. It is a **cheap post-hoc step** with a **big downstream payoff** and it is one of the most reliably-skipped items on the L20 checklist. This chapter is how you own it at senior altitude.

## Two definitions to keep separate

- **Discrimination** (or **ranking**) — the ability of the model to place positives above negatives. Measured by AUC, average precision, NDCG. A model with AUC = 0.9 ranks well.
- **Calibration** — the alignment of the model's stated probability with the empirical frequency. A well-calibrated model predicts p on many examples and is right p × N times. Measured by reliability diagrams, expected calibration error (ECE), Brier score.

These are **orthogonal**. A model can rank perfectly (AUC = 1.0) and be terribly miscalibrated (every positive gets probability 0.6, every negative 0.4). Fixing calibration is a separate exercise from improving AUC. Guo et al.'s [*On Calibration of Modern Neural Networks*](https://arxiv.org/abs/1706.04599) (ICML 2017) is the canonical modern reference documenting how badly modern deep networks are calibrated out of the box.

## Diagnostics — the reliability diagram and ECE

### The reliability diagram

Bin predictions by predicted probability (say, 10 or 20 equal-width bins between 0 and 1). For each bin: compute the mean predicted probability and the empirical positive rate. A perfectly calibrated model has empirical rate = mean predicted probability in every bin. Plot the two against each other and against the identity line.

The diagram tells you *where* the miscalibration is. Deep classifiers typically show a **confidence bias** — the model over-states probabilities in the high-probability bins (says 0.95 when true rate is 0.85). Gradient-boosted classifiers and Naive Bayes typically show a **compression bias** — probabilities are pushed toward the extremes.

### Expected calibration error (ECE)

Summary of the reliability diagram to a single number: the weighted average distance between predicted probability and empirical rate across bins.

$$\text{ECE} = \sum_{b} \frac{|B_b|}{N} \cdot |\bar{p}_b - \bar{y}_b|$$

Where $|B_b|$ is the count of examples in bin b, $\bar{p}_b$ is the mean predicted probability in bin b, $\bar{y}_b$ is the empirical positive rate in bin b, N is the total.

Watch out for the sharp edges: ECE depends on binning; adaptive binning (equal-mass bins rather than equal-width) is more robust; ECE hides direction (over- vs. under-confidence) and slice structure. Report the reliability diagram alongside ECE; do not report ECE alone.

### The Brier score

A proper scoring rule — the mean squared error between predicted probability and label. Decomposes into calibration + refinement (roughly, "the calibration part" + "the discrimination part"). Useful because it is a single number and it is proper; less directly interpretable than ECE + reliability diagram. Report all three when calibration matters to the launch.

### Calibration is a per-slice property

Aggregate calibration can be excellent while calibration on a critical slice is terrible. A fraud model may look well-calibrated on the overall population and be systematically overconfident on new-account transactions. A medical model may be well-calibrated overall and badly miscalibrated on under-represented demographic groups. Report reliability diagrams **per slice** — the same slices [mod-305] uses for evaluation. Slice-wise calibration is often the load-bearing evaluation for regulated domains (see [mod-309] for the fairness angle).

## Post-hoc calibration methods

The four methods you should reach for. They all use a held-out **calibration set** — a slice of your labelled data not used for training and not used for final testing. Fit on the calibration set; report ECE on the test set.

### 1. Platt scaling — a two-parameter sigmoid

Fit a logistic regression on top of the raw model scores (or logits): $p = \sigma(a \cdot z + b)$ where $z$ is the raw score. Two parameters, easy to fit, easy to serve. Works well when miscalibration is a simple sigmoid shift/scale. Originally from Platt's [*Probabilistic Outputs for Support Vector Machines and Comparisons to Regularized Likelihood Methods*](https://www.researchgate.net/publication/2594015_Probabilistic_Outputs_for_Support_Vector_Machines_and_Comparisons_to_Regularized_Likelihood_Methods).

### 2. Isotonic regression — a monotonic non-parametric fit

Fit a monotonic step function from raw score to calibrated probability. More expressive than Platt: can handle non-sigmoid miscalibration. Needs more calibration data than Platt to avoid overfitting (~1 000+ per class as a rule of thumb). Zadrozny and Elkan's [*Transforming Classifier Scores into Accurate Multiclass Probability Estimates*](https://www.cs.cornell.edu/~alexn/papers/zadrozny.kdd02.pdf) is the classical reference. Standard implementation: `sklearn.isotonic.IsotonicRegression`.

### 3. Temperature scaling — the one-parameter deep-learning default

For a softmax classifier over $K$ classes, divide the logits by a learned scalar $T > 0$ before softmax: $p_k = \exp(z_k / T) / \sum_j \exp(z_j / T)$. One parameter, fit by minimising negative log-likelihood (or ECE) on the calibration set. Preserves the argmax (does not change which class wins) and thus does not affect accuracy or AUC. This is what Guo et al. (2017) recommended as the modern deep-learning default; it is the first thing to try for a neural network.

Sample sketch:

```python
import torch
import torch.nn as nn
import torch.optim as optim

class TemperatureScaler(nn.Module):
    def __init__(self):
        super().__init__()
        self.log_T = nn.Parameter(torch.zeros(1))  # log for positivity

    def forward(self, logits):
        return logits / self.log_T.exp()

def fit_temperature(logits_val, labels_val, max_iter=100):
    scaler = TemperatureScaler()
    optimizer = optim.LBFGS(scaler.parameters(), lr=0.01, max_iter=max_iter)
    loss_fn = nn.CrossEntropyLoss()

    def closure():
        optimizer.zero_grad()
        loss = loss_fn(scaler(logits_val), labels_val)
        loss.backward()
        return loss

    optimizer.step(closure)
    return scaler.log_T.exp().item()
```

At serving time, divide logits by the learned `T` before softmax. One float in the model artefact; no accuracy change; substantial ECE improvement in most deep classifiers.

### 4. Beta calibration — a three-parameter parametric fit

Kull, Silva Filho, Flach [*Beyond sigmoids: How to obtain well-calibrated probabilities from binary classifiers with beta calibration*](https://projecteuclid.org/journals/electronic-journal-of-statistics/volume-11/issue-2/Beyond-sigmoids--How-to-obtain-well-calibrated-probabilities-from/10.1214/17-EJS1338SI.full) (2017) — a Beta-family fit for binary classifiers, more expressive than Platt, less data-hungry than isotonic. Useful when Platt underfits and isotonic overfits given your calibration-set size.

### Picking one

- Deep network, softmax, plenty of calibration data → **temperature scaling** first.
- Binary classifier, small calibration set (≤ 1 000) → **Platt** or **Beta**.
- Binary classifier, large calibration set (≥ 5 000), potentially non-sigmoid miscalibration → **isotonic**.
- Multi-class with class-specific miscalibration → **vector or matrix scaling** (Guo et al.'s extensions of temperature scaling).

## When to calibrate — the launch-time checklist

The senior read is: calibration is a required launch step **whenever a downstream decision depends on the probability being a probability**. Concretely:

- A threshold triggers an automated action.
- The score enters a cost or expected-value calculation.
- The score is compared or composed across models.
- The score is presented to a human as a probability (dashboards, review UIs).
- The system is regulated in a domain where probabilistic claims are audited.

If any of those is true, calibration is not optional. It goes into the release harness (mod-305) and it gets a per-slice regression guard. If none of those is true — the system is pure ranking, top-k retrieval, learning-to-rank — you can defer calibration, but you owe a note in the RFC (mod-302 chapter 06) explaining why.

## Retraining and calibration drift

Calibration decays under distribution shift just like every other metric. When retraining, the calibration set has to be re-fit against a **recent** hold-out sample from the production distribution — not the old training-time calibration set, which is by construction stale.

Concretely: the retraining pipeline (mod-302 chapter 05) produces the trained model, then the calibration step runs against a fresh, held-out slice from the last N days of scored production data. The two-stage artefact (model + calibrator) is what gets registered in the model registry. Both parts are versioned together and rolled back together.

## Ensembling and calibration — a happy accident

An unrelated observation with a real practical consequence: **deep ensembles are often better-calibrated than any single constituent**, essentially for free (Lakshminarayanan et al., [*Simple and Scalable Predictive Uncertainty Estimation using Deep Ensembles*](https://arxiv.org/abs/1612.01474)). If you were going to ensemble for accuracy anyway (chapter 04), you may not need a separate calibration step. If you distilled an ensemble into a single student, you probably do — the student inherits the teacher's discrimination but not necessarily the teacher's calibration.

## A worked example — fraud auto-decisioning

**Scenario.** A model outputs a fraud probability. Product wants: auto-decline above 0.99, auto-approve below 0.02, human review in between.

Rubric walk:

- Are we calibration-sensitive? Yes — the exact 0.99 and 0.02 thresholds are what drives the auto-decision rate and the human-review load. Miscalibrated by 5 % on either side, either the queue floods or the fraud loss triples.
- Diagnostic. Plot the reliability diagram on a recent hold-out. Report ECE overall and on the top three risk-relevant slices (new accounts, high-value baskets, international transactions).
- Fix. Fit temperature scaling on the hold-out set (~50 k examples). Re-plot. Confirm ECE has dropped materially in every high-priority bin.
- Regression guard. Add per-slice ECE against a threshold to the release harness. Any retrain that regresses per-slice ECE beyond the threshold blocks the launch.
- Deploy. The calibrator (a single float `T`) is packaged with the model artefact in the registry; both roll back together.

## Three failure modes to catch in review

- **Reporting AUC as calibration evidence.** The team says "our model is well-calibrated because AUC is 0.92." AUC has nothing to do with calibration. Ask for the reliability diagram and per-slice ECE.
- **Fitting the calibrator on the training set.** The calibrator learned to correct the model on data the model has already memorised; test-time ECE is unchanged. Calibration data must be held out from training.
- **Aggregate calibration only.** Overall ECE is 0.03; the calibration on the demographic slice the fairness review cares about is 0.15. The aggregate number hid the failure. Per-slice reliability diagrams are non-negotiable when the launch has any fairness or regulatory review.

## Summary

Calibration is orthogonal to discrimination and rarely comes free from modern classifiers. It is a required launch step whenever a downstream decision, cost calculation, or human-facing display treats the model's output as a probability. Diagnose with reliability diagrams and expected calibration error, per slice. Fix with temperature scaling (deep learning default), Platt or Beta (small data), or isotonic regression (large data, non-sigmoid miscalibration). Retraining pipelines have to re-fit the calibrator on a fresh hold-out. Deep ensembles come well-calibrated for free; distilled students often do not. Chapter 06 picks up the closely related question of uncertainty quantification — how much the model *knows* about its own predictions.
