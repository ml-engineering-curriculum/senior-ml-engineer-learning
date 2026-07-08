# Uncertainty quantification

## Motivation

Calibration (chapter 05) tells you that when the model says "0.9" it means 0.9. Uncertainty tells you *how much the model actually knows about this specific input*. Those are different questions and they support different decisions.

A well-calibrated model may still be confidently wrong on an input from a distribution it never saw. Calibration is a statement about the population; uncertainty is a statement about this single point. At senior altitude you need both — calibration for populations of decisions and uncertainty for individual escalation, abstention, and safety behaviour.

Uncertainty quantification (UQ) is what lets you:

- **Abstain.** Route hard cases to a human reviewer, refuse to answer, ask a follow-up question.
- **Down-weight low-confidence inputs.** Multiply a bid by a confidence, prefer the certain option in a tie.
- **Detect out-of-distribution inputs.** Flag examples the model does not recognise as being in the training distribution.
- **Report intervals, not point estimates.** For regression, produce a prediction interval that has a stated coverage guarantee.

This chapter is the L30 vocabulary and toolbox: the aleatoric-vs-epistemic split, the practical methods (ensembles, MC dropout, Bayesian last layer), the modern distribution-free approach (conformal prediction), and the decision rubric that pins each method to a use case.

## The two flavours of uncertainty

Kendall and Gal's [*What Uncertainties Do We Need in Bayesian Deep Learning for Computer Vision?*](https://arxiv.org/abs/1703.04977) (NeurIPS 2017) is the standard reference; the distinction is older but that paper made it practical.

- **Aleatoric uncertainty — irreducible noise in the data.** The label is not a deterministic function of the input; two identical-looking inputs can have different labels. Aleatoric uncertainty cannot be reduced by collecting more data of the same kind. Predicting a coin flip: aleatoric uncertainty is 50 %, forever.
- **Epistemic uncertainty — reducible ignorance in the model.** The model has not seen enough data to be confident about this region of input space. Epistemic uncertainty *can* be reduced by collecting more data, especially targeted data near the uncertain region. A brand-new class the model has never trained on: epistemic uncertainty is high.

The distinction matters because the two flavours support different decisions:

- **High aleatoric → the data is genuinely ambiguous.** Escalating to a human will not help; the human is going to face the same ambiguity. Abstain, present as ambiguous, or accept the noise.
- **High epistemic → the model is out of its depth on this input.** Escalating to a human or a specialist system is exactly the right move; active-learning that example (chapter 07) is high-value.

## Practical methods for producing an uncertainty signal

Four families, listed by increasing engineering cost.

### 1. Softmax entropy or max-probability — the free baseline

For classification, the entropy (or 1 − max probability) of the softmax output is a cheap uncertainty proxy. It captures aleatoric uncertainty reasonably well when the model is calibrated. It fails on epistemic uncertainty — a badly-out-of-distribution input can still produce a spiky, confident-looking softmax because the model has never learned "I don't know" as an output.

Use as: the free first move. Report it. Do not rely on it for high-stakes escalation.

### 2. Deep ensembles — the modern gold standard

Train N (typically 5) independent models with different seeds. At inference, compute the mean prediction (for calibration and point estimate) and the variance or entropy across the ensemble (for epistemic uncertainty). If the ensemble disagrees strongly on a given input, epistemic uncertainty is high; if they agree, low.

Lakshminarayanan et al.'s [*Simple and Scalable Predictive Uncertainty Estimation using Deep Ensembles*](https://arxiv.org/abs/1612.01474) is the canonical reference. In head-to-head comparisons across many benchmarks and tasks, deep ensembles routinely match or beat more elaborate Bayesian methods on both calibration and OOD detection. The obvious tax is training and serving N models; chapter 04's distillation discussion is the pattern for buying the serving cost back (though distilling the *uncertainty* is much harder than distilling the point estimate).

### 3. MC Dropout — approximate Bayesian at low engineering cost

Train the model with dropout as normal. At inference, keep dropout enabled, forward-pass N times (typically 10–100), aggregate. The variance across the N passes approximates epistemic uncertainty. Gal and Ghahramani's [*Dropout as a Bayesian Approximation*](https://arxiv.org/abs/1506.02142) is the original reference.

Pros: minimal training-time change (dropout was already there); one model at rest. Cons: N-times inference cost at serving time (though the passes are batchable); lower quality than deep ensembles in most benchmarks.

Reach for it when you already have dropout and cannot afford N training runs.

### 4. Bayesian last layer / SNGP / Bayesian neural networks

A range of approaches that replace the last layer of a deterministic network with a Bayesian layer, or add spectral normalisation plus a Gaussian process head (SNGP — Liu et al., [*Simple and Principled Uncertainty Estimation with Deterministic Deep Learning via Distance Awareness*](https://arxiv.org/abs/2006.10108)), or train a full Bayesian neural network with variational inference or Hamiltonian Monte Carlo.

Higher engineering cost, generally comparable-to-slightly-worse than deep ensembles empirically. Useful vocabulary for research reviews; rarely the L30 default in production.

## Conformal prediction — the distribution-free interval

Conformal prediction (Vovk, Gammerman, Shafer, [*Algorithmic Learning in a Random World*](https://link.springer.com/book/10.1007/b106715); modern tutorial by Angelopoulos and Bates, [*A Gentle Introduction to Conformal Prediction*](https://arxiv.org/abs/2107.07511)) is a distribution-free framework for turning **any** point-prediction model into an interval or set predictor with a **finite-sample coverage guarantee**.

The recipe:

1. Take a held-out calibration set.
2. Compute a score of "how wrong" the model was on each calibration example (residual for regression, 1 − predicted probability of the true class for classification).
3. Take the $(1 - \alpha)$ quantile of those scores.
4. At inference, for a new input, produce the interval / set of outputs whose score is at most that quantile.

The remarkable result: under the exchangeability assumption (calibration and test data drawn from the same distribution), the returned interval / set contains the true label with probability at least $1 - \alpha$, exactly. No distributional assumptions on the model. Any regressor. Any classifier. Any black box.

For regression, this produces prediction intervals `[y_hat - q, y_hat + q]` with $1 - \alpha$ coverage. For classification, it produces a prediction *set* (subset of classes) with $1 - \alpha$ coverage — a set of size 1 for confident inputs, a larger set for uncertain ones. Set size is itself a natural uncertainty signal: small set → confident, large set → not.

Why senior ML engineers should care:

- It is model-agnostic. It works on top of gradient-boosted trees, deep networks, LLMs, whatever.
- It gives a mathematical guarantee, not a heuristic.
- It composes with the calibration step from chapter 05.
- It handles regression uncertainty in a way that most deep-learning methods are awkward at.

Modern extensions include split conformal (the standard practical variant), Mondrian / conditional conformal (per-slice coverage guarantees), and adaptive conformal prediction (Gibbs and Candès, [*Adaptive Conformal Inference Under Distribution Shift*](https://arxiv.org/abs/2106.00170)) for handling distribution shift.

The one requirement: exchangeability. Under strong distribution shift, coverage guarantees degrade. Monitor them.

## The decision rubric

Five questions.

### Q1 — Do you need to abstain, route, or take a probability-weighted action?

If yes, you need uncertainty. Cheap baselines (softmax entropy) may suffice for the ranking-and-abstain use case. Escalate as the stakes rise.

If no — the model always produces a single point prediction that is directly consumed — you may not need explicit uncertainty at all. Calibration alone (chapter 05) may be enough.

### Q2 — Do you need to distinguish aleatoric from epistemic?

If yes, you need a method that expresses epistemic uncertainty — deep ensembles, MC dropout, Bayesian last layer, or conformal set-size. Softmax entropy alone will not do it.

Common driver: OOD detection, active learning (chapter 07), routing to specialists.

### Q3 — Do you need a coverage guarantee?

If the answer is "regulator asks for a stated prediction interval with a stated coverage" or "the SLA promises a prediction interval that contains the true value 95 % of the time", you are in **conformal territory**. No heuristic method delivers the guarantee.

If a calibrated point estimate plus a heuristic confidence is enough, skip conformal.

### Q4 — What is the serving-cost tolerance?

Ensembles and MC dropout multiply inference cost. Conformal prediction is roughly free per inference (one lookup against the pre-computed quantile). If serving cost is tight, prefer conformal (built on a single point model) or a calibrated single model over an ensemble.

### Q5 — Where does distribution shift bite?

Conformal guarantees degrade under shift. Deep ensembles' point estimates degrade too, but the ensemble disagreement itself often *increases* under shift, which is a useful drift signal. Instrument uncertainty as a drift indicator regardless of which method you pick.

## Composition with calibration

Uncertainty and calibration are separate steps that both live on top of the base model. A common industrial shape:

1. Train the base model (or ensemble) — chapters 02–04.
2. Fit post-hoc calibration on a held-out set — chapter 05.
3. Fit conformal prediction on a separate held-out set — this chapter.
4. Ship model + calibrator + conformal quantile as one artefact in the registry (mod-302 chapter 03).
5. Monitor per-slice calibration and per-slice conformal coverage on production traffic; alarm on breach.

The three pieces are independent artefacts but they roll together. Rolling back the model without rolling back the calibrator or the conformal quantile is a footgun — the numbers no longer align with the model's outputs.

## A worked example — a medical triage classifier

**Scenario.** A model predicts one of three triage categories (routine, follow-up, urgent) on a lab result. Clinicians want: a most-likely category, a "confidence" level, and — for the "urgent" category — a stated 95 % coverage guarantee on the prediction set for regulatory reasons.

- Q1 — abstain / route? Yes. Uncertain cases go to a specialist. → Uncertainty required.
- Q2 — aleatoric vs. epistemic? Both matter. Epistemic uncertainty triggers the specialist route; aleatoric shapes the "confidence" wording clinicians see.
- Q3 — coverage guarantee? Yes, 95 % on the prediction set. → Conformal required.
- Q4 — serving cost? Modest; batched, not per-request. → Deep ensemble is affordable.
- Q5 — distribution shift? Modest quarter-over-quarter; monitor conformal coverage per slice.

→ Deep ensemble of 5 models for prediction and epistemic signal. Temperature scaling on a held-out calibration set for aggregate calibration. Split conformal on a separate held-out set for the 95 % coverage guarantee on prediction sets. All three artefacts versioned together in the registry, per-slice coverage monitored on production traffic.

## Three failure modes to catch in review

- **Softmax confidence sold as epistemic uncertainty.** The team routes low-confidence cases to a specialist based on softmax entropy, then discovers the model is confidently wrong on OOD inputs. Softmax does not capture epistemic uncertainty. Escalate to ensembles, MC dropout, or an explicit OOD detector.
- **Conformal guarantee not monitored under shift.** The team ships a conformal interval with a 95 % coverage claim, then never checks whether coverage is actually 95 % on production traffic. Empirical coverage per slice is a required release-harness metric when the guarantee is user- or regulator-facing.
- **Uncertainty threshold set once and forgotten.** The "abstain if uncertainty > 0.3" threshold was chosen at launch and never revisited; distribution shift moves the abstain rate from 5 % to 25 % and the human review queue overflows. Abstain rate is an SLI (mod-307): budget it, alarm on it, retrain when you breach.

## Summary

Uncertainty is the per-input companion to calibration's per-population statement. Aleatoric is irreducible noise; epistemic is reducible ignorance — the split matters because it drives different downstream actions. Softmax entropy is the free baseline; deep ensembles are the modern gold standard for epistemic signal and OOD detection; MC dropout is a cheaper approximation; conformal prediction delivers distribution-free coverage guarantees and composes with any base model. Uncertainty, calibration, and the base model are three independent artefacts that ship and roll back together. When distribution shifts, uncertainty itself is one of the most useful drift signals — instrument it as one.
