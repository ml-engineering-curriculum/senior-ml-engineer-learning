# ML-specific SLIs and SLOs: freshness, quality drift, calibration, retraining SLA

## Motivation

Chapter 01 installed the classical SRE vocabulary — SLI, SLO, error budget, burn rate — and named the four gaps where it stops being enough for ML: it does not see "confidently wrong" outputs, it does not see stale features, it does not see quality drift, it does not see retraining lag. This chapter fills those gaps.

The specific claim: an ML system needs a **second SLI menu** running alongside the classical one. Every entry in that second menu has the same `good_events / valid_events` shape as any other SLI, has an SLO, has an error budget, and is monitored with the same burn-rate discipline. The menu is short and it is *load-bearing* — a team that ships the classical SLIs but not these will meet 100 % uptime while their model quietly regresses.

The four SLI families this chapter authors:

- **Prediction freshness** — is the model deciding against features that reflect the world as it is now, or the world as it was N hours ago?
- **Quality-drift** — does the model's live output still match, at whatever fidelity ground truth allows, what the offline eval predicted?
- **Calibration and distribution drift** — do the model's confidences still mean what they meant at training time? Is the input distribution still the one the model was trained on?
- **Retraining SLA** — has a successful, evaluated, promotable retrain landed within the cadence the reliability budget demands?

Every family has a specific SLI shape, a specific SLO discipline, and a specific failure mode it uniquely catches. All four sit on top of the classical SLI floor from chapter 01, not in place of it.

## §1 — Prediction freshness

### What "freshness" means for an ML system

Classical software: freshness is a caching concern. A cached response ten seconds old is fine; the same response ten minutes old is still fine for most read paths.

ML: freshness is a *correctness* dimension. A recommender that ranks items based on a user's session behaviour from four hours ago is not giving the same answer it would give against current session data. A fraud model deciding against a merchant-risk score computed on yesterday's snapshot is deciding against yesterday's threats. Both cases look 100 % available to the classical availability SLI. Both cases are wrong for the user in a way the classical latency SLI cannot see.

The senior read: **every ML prediction has a chain of features behind it, and each feature has an "age" — the time between when the value was computed and when the prediction consumed it.** The freshness SLI measures that age against a per-feature target and reports the fraction of predictions whose feature chain was within target.

Chip Huyen's [*Designing Machine Learning Systems*](https://huyenchip.com/books/) chapter 8 covers this under "data freshness"; the [Feast documentation](https://docs.feast.dev/) frames it as *TTL enforcement* on feature-view definitions; Uber's [*Michelangelo*](https://www.uber.com/blog/michelangelo-machine-learning-platform/) blog series describes it as *feature staleness monitoring*.

### The freshness SLI, in the good-events / valid-events shape

Two ways to write it, depending on architecture:

**Per-prediction freshness (fine-grained).** Every prediction event carries a `feature_ages` map — for each feature the model consumed, the timestamp the feature was computed. The freshness SLI counts a prediction as "good" if every consumed feature was younger than its per-feature target at prediction time.

```python
FRESHNESS_TARGETS = {
    "user_recent_clicks_30m": 300,   # seconds; must be < 5 min old
    "item_ctr_last_24h":     3600,   # < 1 h old
    "user_lifetime_value":  86400,   # < 24 h old
    "catalog_availability":     60,  # < 1 min old
}

def is_fresh(feature_ages: dict[str, float]) -> bool:
    return all(feature_ages[k] <= FRESHNESS_TARGETS[k] for k in feature_ages)

# Aggregated as SLI over a rolling window:
# SLI_freshness = count(is_fresh(pred.feature_ages)) / count(all predictions)
```

**Feature-view freshness (coarse-grained).** Instead of tagging every prediction, monitor the feature view's *own* freshness: when did the batch pipeline last successfully commit new values for feature-view `user_recent_clicks_30m`? Every 5 minutes? Every 15? An SLI on the fraction of 1-minute windows in which every feature-view was within its TTL. Coarser, cheaper, and often enough.

Both shapes have the same SLO structure:

- **SLI:** fraction of predictions (or of monitoring windows) with every feature within its per-feature freshness target.
- **SLO:** 99 % over rolling 28 days, common. Budget: 6.7 hours per window across 28 days.
- **Burn-rate alerting:** fast-burn when a single feature-view is late by more than 3× its TTL for more than 15 minutes.

### What the freshness SLI uniquely catches

- **A batch pipeline that silently fell behind.** The feature-store dashboard shows "fresh"; the Airflow DAG shows "successful"; the pipeline ran but the *values* did not update because the upstream source hadn't landed. The freshness SLI catches this if it measures the *timestamp on the value*, not "the job finished."
- **A streaming pipeline that lost consumer position.** The Kafka lag SLI eventually catches this, but the freshness SLI catches it from the *consumer* side — the predictions are consuming values that are 8 hours old, regardless of whether the producer is catching up.
- **A feature TTL that is longer than the feature's true half-life.** The team set the TTL to 24 hours because "that's what we always do"; the feature's information decays in 30 minutes. The freshness SLI is technically green but the *model quality* SLI (§2) will fire; comparing the two tells you the TTL is wrong.
- **A cascading miss.** The prediction service falls back to a default value when the feature is missing, and the default is what an untrained model would consume — an all-zero vector, an "unknown" bucket. Missing-features-as-defaults is the single most common cause of a live quality regression that no monitoring catches. A separate SLI on the fallback rate (chapter 01 §5) plus this freshness SLI together catch it.

The freshness SLI is the *first* one to add on top of the classical menu. It is the one most likely to fire in the first month, because most ML systems have at least one feature pipeline that quietly runs late.

### Setting the target — the two failure modes

- **Target too loose.** "24 hours is fine, users don't notice." Then the SLI stays green during a real failure; you find out about a stale-feature bug from a customer complaint. Symptom: quality-drift SLI (§2) fires when freshness SLI is green.
- **Target too tight.** "1 minute is fine, we have streaming." Then the SLI is red every time a Kafka consumer restarts and every alert is noise. Symptom: on-call fatigue, the team disables the alert.

The senior read is to *derive* the freshness target from the feature's business meaning:

- The **half-life of the underlying signal.** A session-scoped feature has a half-life of minutes; a monthly-retention feature has a half-life of days. The TTL is a fraction (say, 1/4) of the half-life.
- The **downstream tolerance.** If a customer-support ticket triage model is expected to reroute within an hour of the ticket being opened, features derived from customer-open state can be at most an hour stale before the model's decision becomes meaningless.
- The **cost of freshness.** Sub-minute freshness on a batch source means streaming; streaming is more expensive to operate than nightly batch. The freshness SLI's target is where the cost trade-off gets made explicit, and it should be defensible in the reliability review.

## §2 — Quality-drift SLIs

### The delayed-label problem

Classical SLIs are computed in-window: the failure happens, the number moves. Quality SLIs are computed against ground truth, and ground truth for ML systems lands at *different rates* than the predictions do. Categories worth knowing:

- **Fast-labelled.** Ad clicks (seconds), search clicks (seconds), moderation review decisions (minutes to hours). The quality SLI can be computed against labels within the same window as the SLI itself.
- **Medium-labelled.** Purchase decisions (hours), fraud outcomes (days), support-ticket resolution (hours to days). The quality SLI runs on a delayed window — the SLI for "last week's predictions" lands this week.
- **Slow-labelled.** Long-horizon outcomes: churn (30, 60, 90 days), lifetime value (months). Quality SLIs on the true outcome are impractical; proxies must stand in.
- **Unlabelled.** Recommender ranker outputs (no direct label — user might have chosen the second item even if the first was best), generative outputs (no scalar label). Proxy metrics (chapter 04 of mod-305) are the SLI.

The SLI shape must match the label regime. A team that ships a "quality-drift SLI" with no attention to when labels land ships an SLI that reports whatever noise the label pipeline emits.

### Three quality-drift SLI shapes

**Shape 1 — Live paired metric.** For fast-labelled systems, the SLI is the metric itself (AUC, accuracy, F1, log-loss) over a rolling window of the last N predictions whose labels have landed. The SLO is expressed as a *floor*:

```
SLI_quality = AUC(predictions_last_7d, labels_landed_last_7d)
SLO: SLI_quality > 0.87 over 28-day rolling windows, 95% of the days
```

The SLO's compliance-window shape mirrors classical availability — 95 % of the days meet the target, budget of ~1.4 days per 28-day window.

**Shape 2 — Delta against baseline.** The SLI is the *change* in the metric compared to a locked baseline snapshot (usually the current champion model's offline eval numbers). This is what most mature teams actually monitor, because a floor SLO ("AUC > 0.87") drifts stale as models improve; a delta SLO ("no more than 1 point below the current champion's offline AUC") stays honest as the model gets better.

```
SLI_quality_delta = baseline_metric - live_metric
SLO: SLI_quality_delta < 0.01 (1 point) over 7-day rolling windows
```

**Shape 3 — Proxy metric with a validated correlation.** For slow-labelled or unlabelled systems, the SLI is a proxy — click-through rate for a ranker, engagement time for a recommender, refusal rate for an LLM feature — validated against the eventual outcome on prior data. Mod-305 chapter 04 (LLM-as-judge) discusses proxy metrics for content quality; mod-306 chapter 01 §1 discusses proxy-metric validation for experimentation windows. Both apply here.

The proxy-SLI failure mode is that the proxy stops tracking the outcome. Any team that runs on proxy SLIs must run a periodic *proxy-vs-outcome correlation audit* — quarterly is common — and refresh the SLO if the correlation has slipped.

### What quality-drift SLIs uniquely catch

- **A gradual regression that no per-day comparison catches.** A model drifting from 0.87 AUC to 0.83 over four months is invisible to a daily change alarm; a rolling 28-day SLI with a delta SLO catches it.
- **A "correct on average, catastrophically wrong on a slice."** If the SLI is only computed globally, a slice-level catastrophic regression (mod-305 chapter 02) will show a small global movement. The right shape is to have quality SLIs *per slice* — the same slice matrix as offline — so the SLI can fire on the specific slice.
- **A calibration collapse hidden inside an unchanged accuracy.** Accuracy stays flat while confidence scores stop meaning what they meant (§3). The quality SLI misses this; the calibration SLI catches it.
- **A silent adversarial shift.** For fraud, spam, moderation — models with adversarial pressure — the quality SLI is the first place adversarial shift becomes visible. The classical SLIs cannot see "the adversary changed strategies last Tuesday."

### The delayed-label design pattern

The reason quality SLIs are rarer than freshness SLIs is the plumbing: you need a pipeline that pairs predictions with their eventual labels, and the pairing has to be exact enough that (prediction, label) matches without leakage.

The load-bearing bits:

- **Every prediction is logged with a stable `prediction_id`.** The label pipeline joins on `prediction_id`. No prediction, no join.
- **The pairing window is *known and bounded*.** If purchase decisions land within 48 hours in 95 % of cases and never after 30 days, the pairing job runs at 48 hours + a grace window and the SLI is computed against that snapshot. Predictions whose label has not landed yet do not count in the denominator; that is the `valid_events` filter.
- **The pairing is idempotent.** Re-running the join produces the same result. Alexandra Chen and Emmanuel Ameisen's [*Building Machine Learning Powered Applications*](https://www.oreilly.com/library/view/building-machine-learning/9781492045106/) covers the operational shape; the [Feast](https://docs.feast.dev/) and [Tecton](https://docs.tecton.ai/) docs cover it in the feature-store context.
- **The pairing job's own SLI is monitored.** If the label-join job runs late, the quality SLI is computing against yesterday's data as if it were today's. A separate SLI on the *label-pipeline freshness* catches that.

## §3 — Calibration and distribution-drift SLIs

### Why calibration is a separate SLI

For classifiers whose output is a probability, an unchanged accuracy does not imply an unchanged decision quality. A model that says "0.9" and is right 90 % of the time is calibrated; a model that says "0.9" and is right 70 % of the time is *miscalibrated* even though every top-1 prediction may still be correct. Any downstream that uses the score as a probability — a fraud model whose 0.9 gates a manual-review queue, a recommender whose calibrated CTR feeds a bidder, a triage model whose "high-severity" score triggers a page — breaks under miscalibration in ways the accuracy SLI cannot see.

Mod-303 chapter 05 (Calibration) is the calibration-theory chapter; this chapter is where calibration becomes an SLI.

The SLI shape is a **binned reliability check** — Expected Calibration Error (ECE) or Maximum Calibration Error (MCE) — with an SLO on the drift from a locked baseline:

```
ECE_live = sum over bins of (frequency * |avg_predicted - avg_actual|)
SLO: ECE_live - ECE_baseline < 0.02 over 7-day rolling window
```

Guo, Pleiss, Sun, Weinberger's [*On Calibration of Modern Neural Networks*](https://arxiv.org/abs/1706.04599) is the modern reference; [Kumar, Liang, Ma's *Verified Uncertainty Calibration*](https://arxiv.org/abs/1909.10155) covers ECE variants and their failure modes.

### Distribution-drift SLIs — the input-side shape

The other side of the drift problem is the input: has the distribution of features the model sees today diverged from the distribution the model was trained on? This is a *leading* indicator of quality drift — the input shifts, and the quality regression follows.

Two SLI shapes are standard:

**Population Stability Index (PSI).** For each monitored feature, bucket its values (usually 10 equal-frequency bins from the training distribution) and compute:

```
PSI = sum over bins of (live_pct - train_pct) * log(live_pct / train_pct)
```

Rules of thumb from the credit-scoring literature (where PSI originated): PSI < 0.1 is "no significant shift," 0.1 to 0.2 is "moderate," > 0.2 is "significant." The [OpenScoring PSI documentation](https://github.com/openscoring/openscoring/wiki/psi) and Sicular et al.'s treatment in the Gartner ML monitoring reports cover the industry usage.

**Kolmogorov-Smirnov (K-S) test statistic.** For each continuous feature, the K-S statistic between the live distribution and the training distribution. Reifman, Feldman, Cohen's [*Distribution shifts in machine learning: A survey*](https://arxiv.org/abs/2005.12852) is a modern reference; Rabanser, Günnemann, Lipton's [*Failing Loudly: An Empirical Study of Methods for Detecting Dataset Shift*](https://arxiv.org/abs/1810.11953) is the load-bearing empirical evaluation of drift-detection methods.

The SLI in `good_events / valid_events` shape:

```
SLI_dist_drift = features_with_PSI_below_threshold / total_monitored_features
SLO: SLI_dist_drift > 0.95 (i.e., at most 5% of features in drift) over 7-day rolling
```

### What calibration and drift SLIs uniquely catch

- **A quiet input shift ahead of the quality regression.** PSI moves on the input side days before quality moves on the output side (for delayed-label systems); the drift SLI is the fast-burn alarm that predicts the quality SLI going red.
- **A calibrated model that regressed on calibration only.** Accuracy flat, ECE up — the model's score no longer means what the downstream expects. This is the classic "we changed the loss function and forgot to recalibrate" failure.
- **A prediction-drift signal without labels.** For unlabelled systems, monitoring the *output* distribution (the fraction of predictions above a threshold, the histogram of confidence scores) with the same PSI/KS machinery gives an early-warning signal even when labels are unavailable. Chip Huyen's chapter 8 covers this under *prediction drift*.
- **Feature-vs-population shift decomposition.** When multiple features drift together, PSI on each isolates *which* features are moving. That is what the runbook (chapter 04) walks — "PSI moved on `user_recent_clicks_30m` and not on the others → look at the upstream events pipeline."

### Failure modes to name

- **Alerting on any drift, not on drift *that affects the model*.** Not every distribution shift matters. Features the model weighs weakly can drift without any quality impact. The SLO threshold should be calibrated against *observed* quality-vs-PSI correlation from prior incidents, not against a Basel-II-era rule of thumb.
- **PSI on features that are logically discontinuous.** A categorical feature with a "new category added last week" is going to look like massive drift on PSI. The SLI needs a per-feature handling — drop new categories from the histogram, or use a K-S / Chi-squared shape that handles it explicitly. NannyML's [*Drift detection under the hood*](https://nannyml.readthedocs.io/en/stable/how_it_works/drift_detection.html) documentation walks the trade-offs.
- **Calibration SLI computed on a biased sample.** If the ECE is computed only on predictions where the label happened to land quickly, the ECE is biased toward the fast-label sub-population. Weight the reliability bins accordingly, or acknowledge the bias explicitly.

## §4 — Retraining SLA — the "how fresh is the model" SLI

### Why retraining time-since-last-successful is a first-class SLI

The four SLIs above measure whether the *deployed* model is behaving. This SLI measures whether the *deployed* model is *the model it should be*. For any system whose environment shifts on a knowable cadence — fraud (weeks), recommender catalogue (days to weeks), search intent (weeks), pricing (seasonally), moderation (adversarial, sometimes hours) — a model that has not been retrained within its cadence is a reliability risk even if every other SLI is green today.

Mod-302 chapter 05 (Retraining cadence, data flywheel, and shadow/canary rollout) established the *four triggers* for retraining. This SLI is the reliability discipline that ensures the trigger fires and the outcome is *a successfully promoted model*, not a paused pipeline no one is watching.

The SLI shape:

```
SLI_retraining_sla = min(1, time_since_last_promoted_retrain / retraining_target_window)
inverted: SLI_within_sla = (time_since_last_promoted_retrain <= retraining_target_window) ? 1 : 0
SLO: SLI_within_sla = 1 (always within the window)
```

Practically, the SLI is a *deadline-based* SLO: for a fraud model with a target 7-day cadence, the SLI is a boolean "was a successful retrain promoted within 7 days," monitored continuously. The error budget is *how many retrains can slip past the target in a quarter* before the reliability review flags the system.

### What "successful retrain promoted" actually means

The SLI's `good_events` counter is *not* "the pipeline finished." It is:

- The retraining pipeline ran and produced a registered artefact.
- The candidate passed the offline evaluation gate (mod-305 chapter 01).
- The candidate walked shadow and canary (chapter 05, and mod-305 chapter 03) without rollback.
- The candidate was promoted to `Production`.

A pipeline that runs weekly and every week fails the eval gate is *not* a system meeting its retraining SLA. It is a system with a broken retraining pipeline and no one calling it out. That is exactly the failure mode this SLI is designed to catch — the *pipeline SLI* is green (the DAG succeeded); the *retraining SLA SLI* is red (no promoted candidate in three weeks).

### Setting the target

The retraining target is driven by:

- The **half-life of the problem** — the same reasoning as mod-302 chapter 05's cadence selection. If the fraud typology shifts on a 4-week cycle, a 2-week retraining target is defensible; a 90-day one is not.
- The **cost of retraining** — retraining a large model daily might be technically possible but economically indefensible. If daily retraining costs $10 K in compute for a $50 K/month feature, the ratio does not survive a budget review.
- The **stability of the pipeline** — a mature pipeline that succeeds 95 % of the time can be scheduled weekly with confidence; a fragile pipeline should either be stabilised first or the SLA should include a "median time to promoted retrain" alongside "worst-case time."

### What the retraining SLA SLI uniquely catches

- **Silent pipeline degradation.** The Airflow DAG succeeded but the trained model does not pass the offline gate. Without this SLI, the team notices only when someone asks "when did we last ship a new fraud model?" Answer: eight weeks ago.
- **Approval-latency incidents.** The candidate passed offline; the shadow was clean; the canary is ready. But the release meeting has been deferred three weeks running. The retraining SLA SLI is red; the pipeline is fine; the *process* is the failure.
- **Rollback-and-stay-rolled-back.** A promotion rolled back; the team addressed the incident but never re-promoted. The last-promoted-retrain clock has been running for weeks. The SLI catches the "we intended to re-promote and forgot" case.
- **A pipeline dependency that has become invalid.** The pipeline used to succeed; then an upstream schema change broke it silently (the pipeline emits zeros for missing fields, evaluation fails on the eval gate for realistic reasons the runbook says are "unclear"). This SLI holds the calendar the pipeline runbook accountable to.

## §5 — Composing the SLI menu into a single SLO document

Chapter 01 §6 sketched an SLO document with three classical SLIs. The ML-specific menu extends that document, not replaces it. A worked extension:

```markdown
### 4. Prediction freshness
- SLI: fraction of predictions in a 1-minute window with every consumed feature within its per-feature TTL
- SLO: freshness ≥ 99.5% over 28-day rolling window
- Error budget: 3.4 hours per 28-day window
- Alerting: fast burn (14.4× budget over 1 h) → page. Slow burn (3× budget over 6 h) → ticket.
- Runbook: check feature-store freshness dashboard by feature-view, then upstream pipeline lag.

### 5. Quality-drift (delta against locked baseline)
- SLI: AUC_baseline - AUC_live over rolling 7-day window (labels lag ≤ 48 h)
- SLO: delta ≤ 0.01 (1 point) on 95% of days over 28-day window
- Error budget: 1.4 days per 28-day window
- Alerting: 3 consecutive days of delta ≥ 0.01 → ticket. delta ≥ 0.02 in any 1-day window → page.
- Runbook: check calibration SLI, distribution-drift SLI, and freshness SLI in order.

### 6. Calibration drift (ECE against locked baseline)
- SLI: ECE_live - ECE_baseline over rolling 7-day window
- SLO: ΔECE ≤ 0.02 on 95% of days over 28-day window
- Error budget: 1.4 days per 28-day window
- Alerting: 5 consecutive days over threshold → ticket.

### 7. Input distribution drift (feature PSI)
- SLI: fraction of monitored features with PSI ≤ 0.2 over 24-hour windows
- SLO: SLI ≥ 0.95 on 95% of days over 28-day window
- Alerting: any single feature with PSI > 0.4 for 3 consecutive days → ticket.

### 8. Retraining SLA
- SLI: time_since_last_promoted_retrain (in days)
- SLO: SLI ≤ 14 days
- Error budget: at most 1 SLA breach per calendar quarter
- Alerting: SLA + 25% (17.5 days) → ticket. SLA + 50% (21 days) → page.
- Runbook: check retraining pipeline status; if pipeline is green but promotion is stalled, check the release-meeting calendar and offline-gate results.
```

Two properties of this composed document worth naming:

- **Each ML-specific SLI has an explicit runbook link.** When the SLI goes red, the on-call has a first move. Chapter 04 walks the runbook shape in detail.
- **The alerting thresholds differ by SLI family.** Freshness supports both fast and slow burn because latency-style monitoring is well-understood. Quality drift is only slow-burn against a delta; a single day's drop is often noise. Calibration and distribution drift are almost always slow-burn. Retraining SLA is a one-shot deadline. Copying the availability-SLI alerting posture to every other SLI is the single most common mis-implementation of the SLO framework in an ML team.

## §6 — What deliberately does not go in this menu

The ML-specific SLI menu is finite. Some things that *look* like SLIs are not, and the failure mode of treating them as SLIs is worth naming.

- **Model bias / fairness metrics are not reliability SLIs.** Fairness slices are guardrails — a decision-blocking constraint on release. Mod-309 (Responsible AI Governance) authors them as their own gate, and mod-305 chapter 02 enforces them in the eval harness. A "worst-group AUC" SLI is fine as an observability line; but the fairness *decision* is not error-budgeted the way availability is. You do not spend a "fairness error budget" on a shipping decision.
- **User-facing product KPIs (conversion, engagement, revenue) are not reliability SLIs.** They belong to product analytics and to the experimentation platform (mod-306). A reliability SLI is about *whether the model is doing what it was designed to do*; a product KPI is about *whether the product design was right*. Confounding them turns the reliability review into a product review.
- **Cost is a first-class budget but is not an SLI in the sense of `good_events / valid_events`.** Cost is a running total against a budget, monitored on a different mathematical shape. Chapter 03 authors the cost-budget framework as its own discipline alongside the SLO framework, not as an SLI *inside* the SLO framework.

## §7 — Cross-links to other chapters

- The **freshness SLI** is the reliability instrument on top of mod-302 chapter 02 (batch/streaming/online/offline) and chapter 03 (feature store). Feature-store TTLs are where the target numbers come from.
- The **quality-drift SLI** is the online counterpart of mod-305 chapter 01 (offline eval harness); the delta baseline it references is the harness baseline.
- The **calibration SLI** is the operational discipline built on mod-303 chapter 05 (Calibration). Miscalibration surfaced in production is often a signal to re-run the calibration step in the training pipeline, not to re-train from scratch.
- The **retraining SLA SLI** is what mod-302 chapter 05's four retraining triggers *deliver against*. This chapter's SLA is how you know the pipeline is doing its job.
- The **error budget policy** (chapter 01 §2) governs all five ML SLIs. A quality-drift budget being consumed drives the same freeze-on-empty policy as an availability budget.

## Summary

Classical SLIs — availability, latency, throughput — are the floor. The ML-specific SLI menu extends the SLO document with four families: prediction freshness (feature-age vs. target), quality-drift (a live metric against a locked baseline delta, adapted to the label-landing regime), calibration and distribution drift (ECE / PSI / KS with drift-vs-baseline SLOs), and retraining SLA (time-since-last-promoted-retrain against a cadence target). Each family has a specific SLI shape, a specific SLO, a specific burn-rate discipline, and a specific failure mode it uniquely catches. Fairness metrics and product KPIs are *not* SLIs and belong elsewhere. Cost is a first-class budget but not an SLI. The next chapter authors the cost-budget framework; chapter 04 authors the incident-response runbooks these SLIs point at; chapter 05 authors the retraining pipeline whose promotion event drives the retraining SLA SLI.
