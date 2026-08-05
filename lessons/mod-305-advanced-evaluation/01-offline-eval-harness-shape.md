# The offline evaluation harness: shape and contract

## Motivation

Every ML team ships models. Not every ML team ships them defensibly. The tell is not the model's headline accuracy — it is the answer to a simple question: **before this candidate model is allowed to be considered "better," what does it have to pass?** The teams that ship well answer with a list. The teams that ship badly answer with a shrug, a screenshot of a notebook, and a sentence that begins "the offline lift looks…".

The offline evaluation harness is the artefact that turns "the offline lift looks…" into evidence. It is the code that, on every candidate model, produces the same shape of decision object — pass / fail per metric, pass / fail per slice, pass / fail per guardrail — such that the team's release decision is not "does this model look better on the notebook I ran" but "does this model pass the harness the whole team agreed on last quarter." At L20 the harness is optional; at L30 it is the *precondition of a release conversation*.

This chapter is the harness shape. Chapter 02 drills into slices and adversarial guardrails. Chapter 03 is the online-promotion path that an offline win still has to earn. Chapter 04 is the Google ML Test Score rubric that reviews *whether the harness itself is fit for purpose*. Chapter 05 is when to bring LLM-as-judge into the harness at all.

## What the harness is, at senior altitude

The harness is a **program**, not a notebook. It has an entry point that takes a candidate model artefact (a model registry ID, a prompt bundle fingerprint, a fine-tune adapter path — anything with a version) and produces a **decision object**. It is what CI runs on every PR that touches the model, what the release checklist points at, and what the on-call runbook consults when a regression fires.

The shape:

```
harness(candidate, baseline, eval_config) → HarnessDecision
```

- `candidate` — the model under review. Pinned to a specific artefact ID.
- `baseline` — the model currently in production (or the last approved model). The comparison is *always* paired against a baseline. "Better than nothing" is not a release criterion; "better than the production model on every guardrail slice and better on the primary quality metric with a lift that is statistically distinguishable from zero" is.
- `eval_config` — the versioned eval sets, the slice definitions, the metric definitions, the pass-rate thresholds, and the adversarial suites. This config is a file in the repo, reviewed via PR.
- `HarnessDecision` — a structured object naming which gates the candidate passed, which it failed, and by how much. Human-readable summary at the top, machine-readable body underneath.

If any of those four is missing you do not have a harness — you have a script. The disciplines that keep the harness honest live in the other three; this chapter installs the contract.

## The five inputs, in order

### 1. The primary quality metric — one, chosen deliberately

Pick one primary metric for the harness and one only. This is the metric a release conversation is about. AUC for a classifier where the downstream use is ranking; NDCG@k for a ranker where the consumer is a ranked list; ROUGE-L or exact-match for extraction where the downstream is a structured feature; BLEU or a task-specific score for generation where a human reads the output; per-request cost as a primary metric when the *entire* release motivation is a cheaper model. Chip Huyen's [*Designing Machine Learning Systems*](https://huyenchip.com/books/) chapter 6 is the classical framing on choosing this metric to match the downstream decision.

Everything else in the harness — every per-slice pass rate, every guardrail, every adversarial check — is *around* this metric. If you cannot name the primary metric in one sentence, the harness does not have a decision it is producing yet; work stops until the team can name it.

The rule that saves teams from themselves: **the primary metric is chosen before the candidate exists.** Adding "we also improved metric X" *after* the candidate's numbers came in is how metrics-shopping happens. Chapter 03 says the same thing about online metrics.

### 2. The eval sets — versioned, per-slice, and immutable per run

The eval config points at named, versioned eval sets. Each one is a manifest — the dataset ID, the split, the row count, the label source, the schema, a fingerprint. The manifest lives in the repo. Rows do not silently change from one run to the next. If the eval set is regenerated, the version bumps and every historical harness run for the old version is still queryable.

Three tiers of eval set, at minimum:

- **Primary eval set.** The one the primary metric is computed on. Representative of the production distribution at the last snapshot, big enough that a statistically-distinguishable lift is detectable (see §4).
- **Slice eval sets.** One per production-critical slice (chapter 02). The slices are defined *by* the eval config, not decided ad hoc from the notebook.
- **Adversarial / regression suites.** The rules the model must not break (chapter 02): known-failure cases the last incident produced, safety cases, calibration cases, out-of-domain cases the deploy is *not* claiming to cover.

Held-out means held out. The primary eval set is not touched by training. It is not touched by hyperparameter search. It is not the same data the calibration step (mod-303 chapter 05) is fit on. It is not the data the LLM-as-judge (chapter 05) was tuned against. The most common way an offline harness silently starts lying is that the eval set has quietly leaked into training over months of iteration — either directly (someone added it to the training loader) or via a feature that is computed the same way in training and eval and thus overfits to the eval slice's peculiarities. Chapter 04 is where this becomes an *ML Test Score* failure with a name.

### 3. The baseline — the currently-shipped model, not a favourite

Every candidate is scored against a baseline. The default baseline is the **model currently in production**, not a research checkpoint you happen to like. If there is no production baseline (a new feature launch), the baseline is a simple, explainable **reference model** — the majority class, a linear classifier on hand-chosen features, a naive retrieval, a well-prompted frontier model — that a reviewer would accept as the floor.

The candidate has to beat the baseline **paired**, on the same eval set, in the same evaluation run. Two independent numbers from two independent notebooks are not a comparison; they are two disconnected data points. The harness runs both models over the same rows in the same order and reports a per-row delta as its first-class output. Paired comparisons unlock the statistical machinery in §4.

The baseline is *also* the reason "an offline win looks like a 0.4-point AUC lift" is a substantive claim rather than a hopeful one — 0.4 points off *what*, on *what data*, evaluated *how*. The baseline is the anchor for all three.

### 4. The statistical shape — bootstrap CIs, not raw means

The harness reports lifts with **confidence intervals**, not point estimates. A candidate whose primary-metric mean is 0.02 above baseline may be a real win or a coin flip depending on the CI. The three tools that cover 90 % of the offline-eval statistical needs:

- **Paired bootstrap on per-row deltas.** Compute the per-row difference (candidate metric − baseline metric) for every row of the eval set. Resample the row-level deltas with replacement 1 000–10 000 times, compute the mean of each resample, report the 2.5 / 97.5 percentiles as the 95 % CI. If the CI crosses zero, you do not have an offline win — you have noise. Efron and Tibshirani's [*An Introduction to the Bootstrap*](https://www.crcpress.com/An-Introduction-to-the-Bootstrap/Efron-Tibshirani/p/book/9780412042317) is the canonical reference; every mature stats library has a bootstrap primitive.
- **McNemar's test / paired sign test for discrete outcomes.** For binary correct/incorrect predictions on the same rows, the paired change from wrong-to-right vs right-to-wrong has a standard test with a known null. Cheaper to compute than a bootstrap and appropriate when the metric is naturally per-row correctness.
- **Standard errors from the eval set size.** For simple mean metrics (accuracy, RMSE), a normal-theory standard error and a two-sample paired test is often enough. The point is not which of the three you pick; the point is that the harness reports a *distribution* around the lift, not a single number.

The rule most teams learn painfully: **an offline "win" whose CI includes zero is not an online win in expectation, either.** A harness that reports only a mean lift is one whose owners will eventually spend a quarter chasing a difference that was noise from the start. Chapter 03 catches the same failure mode online, where the cost of chasing noise is 10× higher.

### 5. The metric definitions — one file, reviewed like code

Every metric — the primary quality metric, every slice-level metric, every guardrail metric — is a function in the repo, imported by the harness. Not a query the analyst re-writes each release. Not a notebook cell that redefines "accuracy" with a subtly different denominator. The metric functions have tests (a synthetic input with a known output), a docstring naming the paper or the operational reason the metric was chosen, and a version.

The failure mode this prevents is subtle and expensive: the eval set is stable, the model is stable, and the reported "accuracy" moved 3 points because someone changed the metric denominator in a helper file. Chapter 04 flags this as *Test 1 — Feature and metric code is under version control and tested*. The single most common regression the harness catches is one where the *metric* changed, not the model.

## The decision object — what the harness returns

A `HarnessDecision` is not "candidate won." It is a structured record. A workable shape:

```python
from dataclasses import dataclass

@dataclass
class SliceResult:
    slice_name: str                # e.g. "new_customer_first_ticket"
    primary_metric: float
    baseline_metric: float
    lift_mean: float
    lift_ci_low: float             # 95 % lower
    lift_ci_high: float            # 95 % upper
    n_rows: int
    passes_gate: bool              # per-slice pass-rate gate from eval_config

@dataclass
class GuardrailResult:
    guardrail_name: str            # e.g. "calibration_ece", "safety_refusal_rate"
    metric: float
    threshold: float
    passes: bool

@dataclass
class HarnessDecision:
    candidate_id: str
    baseline_id: str
    eval_config_version: str
    primary_metric_name: str
    overall_lift_mean: float
    overall_lift_ci: tuple[float, float]
    slice_results: list[SliceResult]
    guardrail_results: list[GuardrailResult]
    passes_release_gate: bool      # AND of all mandatory gates
    generated_at: str              # UTC ISO timestamp
    harness_version: str           # git SHA of the harness itself
```

Two useful properties fall out of this shape:

- **The decision is auditable.** The release conversation is not "did the candidate win"; it is "which gates passed, which failed, and by how much." A regulator, an incident review, or a peer reviewer can read the decision object months later and reconstruct what was known at release time. Chapter 04 makes this an explicit *ML Test Score* item.
- **The gate is composable.** `passes_release_gate` is the AND of the per-slice pass-rates, the guardrail passes, and the primary-metric-lift-with-CI-clear-of-zero. Any single failure blocks the release. If a team wants to override a specific gate, that override is a signed acknowledgement — not silently deleting the check.

Chapter 03 shows the same decision object as a *precondition* for online promotion: no offline `passes_release_gate=True`, no shadow traffic, and certainly no canary.

## Where offline metrics diverge from online outcomes

The harness answers "does this candidate meet the offline bar," not "will this candidate improve the product." Chapter 03 is where the second question gets answered. But you should know from the start where the two most predictably diverge, because the harness's job is *not* to hide the divergence — it is to make it visible.

- **Distribution shift.** The eval set is a snapshot of a past distribution; the production distribution is today's. If the shift is systematic (a new customer segment onboarded, a seasonal shift, a product-catalogue change), an offline win on stale data can be an online loss.
- **Feedback effects.** A recommender changes what users see, which changes what users click, which changes the training data. An offline win on historical click data can mean nothing online — Beel et al.'s [*Comparison of offline and online evaluation results for recommender systems*](https://arxiv.org/abs/1508.04808) is a standard reference on how badly the two can disagree.
- **Multi-metric trade-offs the offline harness does not see.** Latency, cost, engagement, downstream revenue — all measured online. An offline harness that only reports quality is silent on latency regressions that would kill the release online.
- **Calibration shifts under the online policy.** A classifier's threshold behaviour offline may not match the threshold behaviour under the *policy* that the classifier is composed with. Mod-303 chapter 05 (calibration) sets the vocabulary; chapter 03 wires the online check.

The harness's job is to name these gaps in the decision object, not to paper over them. A `HarnessDecision` that says "offline pass on quality; latency and cost are online-only signals; distribution-shift risk is `medium` because the eval set is 90 days stale" is a much more honest artefact than one that reports a mean lift and stops.

## Cadence: PR-blocking, nightly, weekly

The harness runs at three cadences, and each catches a different failure mode:

- **PR-blocking.** Every PR that touches model code, feature code, prompt bundles (mod-304 chapter 02), or the eval config runs a fast subset of the harness (typically a sampled eval set + the guardrails). The PR cannot merge if the fast harness fails.
- **Nightly on the full eval set.** The full harness runs against the currently-deployed model on the full eval set, on a schedule. This catches the class of failure where nothing on the model side changed but the *eval labels* changed — new label rows landed, an upstream data pipeline shifted a feature, someone edited a metric. Chapter 04 has this as a *Monitoring* item.
- **Weekly (or per-release) regression on adversarial suites.** The adversarial and regression suites (chapter 02) run at a slower cadence, because they are the tail — expensive to grow, expensive to run, but the difference between "we caught the regression before shipping" and "we shipped the regression and heard about it from support."

If your harness runs only when a human clicks a button, one of the three cadences is going to catch a regression when nobody was looking. The point of the harness is that "nobody was looking" is not a failure mode.

## What lives outside the offline harness

The offline harness is one artefact in a larger release story. Chapter 03 owns the shadow and canary paths that convert an offline `passes_release_gate=True` into an online promotion. Chapter 04 owns the *review* that says the harness itself is fit for purpose. Chapter 05 owns the special case where the metric itself is scored by an LLM. Mod-306 owns the experimentation platform the online path runs on. Mod-307 owns the SLOs the online candidate has to keep meeting *after* release.

The offline harness's altitude is precise: it converts "I ran a notebook" into "the team-agreed harness produced a decision object." Everything downstream of that decision — shadow, canary, A/B, SLO — assumes the decision object exists and is trustworthy. Chapter 04 is where you audit the *is trustworthy* part.

## Summary

The offline harness is a program with four inputs — candidate, baseline, eval config, and a paired statistical shape — that produces a structured `HarnessDecision` object. The primary metric is picked before the candidate exists; eval sets are versioned and held out; every candidate is compared *paired* against a production baseline with bootstrap CIs, not raw means; every metric is a versioned, tested function in the repo. The decision object is auditable, composable, and the precondition for the online promotion path in chapter 03. Chapter 02 grows the slice and adversarial machinery inside the harness; chapter 04 reviews whether the harness itself is fit for purpose; chapter 05 says when the metric can be scored by an LLM at all.
