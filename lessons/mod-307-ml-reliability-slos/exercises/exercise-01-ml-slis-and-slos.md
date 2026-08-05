# exercise-01: ML SLIs and SLOs — author the full SLO document

**Estimated effort:** 3 hours

## Objective

Author the **full SLO document** for a specific ML feature at senior altitude — classical SLIs on the request path, the ML-specific SLI menu on top (freshness, quality drift, calibration, distribution drift, retraining SLA), error budgets, burn-rate alerting for each, and the *error-budget policy staircase* that names what the team stops doing when the budget depletes. The output is a Markdown document that could go straight into a repo and be signed off by a peer L30 and the on-call rotation.

This is the L30 tell that separates "we have monitoring" from "we have a reliability contract." An L20 has a Grafana dashboard; an L30 has a signed SLO document with an error-budget policy that a VP-eng understands and enforces.

## Prerequisites

- Read chapter 01 (`01-sre-fundamentals-for-ml.md`) — the SLI / SLO / SLA distinction, the good-events/valid-events shape, error budgets, burn-rate alerting, the classical SLI menu, and the reasons the framework transfers to ML.
- Read chapter 02 (`02-ml-specific-slis-and-slos.md`) — freshness, quality drift, calibration, distribution drift, and retraining SLA SLIs and their SLO shapes.
- Skim Google's [*Implementing SLOs*](https://sre.google/workbook/implementing-slos/) chapter of the SRE Workbook — the source of the good-events / valid-events shape and multi-window burn-rate alerting.
- Skim [mod-304 chapter 04](../../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) and [mod-305 chapter 03](../../mod-305-advanced-evaluation/03-shadow-and-canary-online-promotion.md) — the cost envelope and the online promotion path whose auto-rollback controller reads these SLOs.

## Pick your feature

Pick **one** of the following, or bring your own (F5, with a one-paragraph "context I already know" preamble):

- **F1 — Ranking / recommender model on a high-traffic surface.** ~10 M sessions/day, per-request p99 target ~80 ms, weekly retraining cadence, ground truth (clicks) lands within seconds, non-trivial slice matrix (region × device × logged-in state).
- **F2 — Fraud-detection model on transaction path.** ~1 M transactions/day, per-request p99 target ~50 ms, monthly retraining cadence, ground truth (chargebacks) lands within 30–90 days, adversarial pressure means input drift is expected and matters.
- **F3 — LLM-augmented ticket triage classifier.** ~5 K tickets/day, per-request p99 target ~3 s, quarterly (or event-driven) retraining, ground truth (routing correctness verified by analyst) lands within hours, cost is dominated by LLM vendor spend, safety-classifier fires are a first-class signal.
- **F4 — Ad-conversion prediction on the auction path.** ~50 M requests/day, per-request p99 target ~15 ms (hard), daily retraining cadence, ground truth (conversion) lands within 24–72 hours, feature freshness is a hard business requirement (advertiser-facing bidders depend on it).
- **F5 — Bring-your-own.** A production or planned feature you have access to. Anonymise anything sensitive; add a preamble covering traffic scale, per-request latency budget, retraining cadence, label-landing regime, and the current reliability posture.

## Steps

### 1. Classical SLI baseline (≈ 30 min)

Author the classical SLI section of the document — availability, latency (tail percentiles), throughput ceiling if relevant, plus feature-fetch and downstream-dependency SLIs (chapter 01 §5). For each:

- Metric name and PromQL / SQL definition (the actual query, not a description).
- Numerator (`good_events`) and denominator (`valid_events`) — spell out the `WHERE` clauses that shape both.
- SLO target and compliance window (28-day rolling is the default; justify anything else).
- Error budget in *human units* (e.g., "20 minutes per 28 days," not "0.05 %").
- Multi-window burn-rate alerting: fast-burn threshold and window; slow-burn threshold and window.

At least four SLIs in this section, at least one being a feature-fetch or dependency SLI.

### 2. Prediction freshness SLI (≈ 30 min)

Author the freshness SLI for the feature's critical feature-views. Cover:

- **Per-feature-view freshness target.** For each of at least three feature-views the feature depends on, name the target and justify it (chapter 02 §1 — half-life of the signal, downstream tolerance, cost-of-freshness trade-off). "24 hours because that's what we always do" is not a defense.
- **The SLI shape.** Per-prediction (with `feature_ages` tags) or per-feature-view (window-fraction-fresh). Justify the choice against cost and diagnostic power.
- **The SLO** — usually a percentage of predictions or of monitoring windows with all features fresh.
- **Burn-rate alerting.** Freshness supports both fast and slow burn; author both.
- **Runbook stub.** For each expected failure mode of this SLI, one-line pointer to what the runbook does.

### 3. Quality-drift SLI (≈ 30 min)

Author the quality-drift SLI adapted to your feature's label-landing regime:

- **Label-landing model.** Fast (seconds/minutes), medium (hours/days), slow (weeks+), or unlabelled. If your feature has a mixed regime, name the split.
- **SLI shape.** Live paired metric, delta-against-baseline, or proxy metric with a validated correlation (chapter 02 §2). Justify the choice.
- **The specific metric.** AUC, F1, precision-at-K, NDCG, calibrated log-loss — pick the one whose *change* is the ship-blocking signal.
- **Locked baseline.** Named baseline model version and its offline metric values. If you use a delta SLO, this is the reference.
- **Per-slice quality SLIs.** For the feature's business-critical slice matrix (borrow from mod-305 chapter 02), a per-slice quality SLI so a slice-level regression is not hidden inside a green global SLI.
- **The `valid_events` filter.** Which predictions count — only those whose labels have landed? Only within N days? Spell it out.
- **Burn-rate alerting.** Quality drift is usually slow-burn only; author the slow-burn window and threshold, and justify not paging on single-day movement.

### 4. Calibration and distribution-drift SLIs (≈ 20 min)

Author both:

- **Calibration SLI.** ECE (or a chosen variant) against a locked baseline. SLO on the delta, not the absolute number.
- **Distribution-drift SLI.** PSI or K-S on the input features, with either a per-feature or an aggregated "fraction-of-features-in-drift" shape. Explain how you handle new categorical values and low-signal features.
- **Alerting.** Both are usually slow-burn against sustained divergence, not single-window movement.

### 5. Retraining SLA SLI (≈ 20 min)

Author the retraining SLA SLI:

- **Target cadence.** Derived from the shortest half-life of any critical feature or label distribution (chapter 02 §4).
- **SLI definition.** `time_since_last_promoted_retrain`. "Promoted" is important — passing the offline gate is not enough; the version has to have reached Production. Chapter 05 §1 is the state machine.
- **SLO.** A deadline (in days) plus an error budget of "number of SLA breaches allowed per calendar quarter."
- **Alerting.** SLA + 25 % → ticket; SLA + 50 % → page. Tune to your cadence.
- **Runbook stub.** When the SLI is red but the pipeline is green, what does the on-call check?

### 6. Error-budget policy staircase (≈ 20 min)

Chapter 01 §6 sketched an error-budget policy staircase. Author the specific staircase for your feature:

- **≥ 50 % budget remaining.** Normal shipping.
- **25–50 % remaining.** Pre-launch reliability review required. Name the specific reviewer.
- **≤ 25 % remaining.** Freeze on non-reliability changes; retraining pipeline pauses (chapter 05 §5). Name the specific decision-maker for exceptions.
- **Budget exhausted.** Full freeze; all engineering effort on burn-down; incident review at platform level.

The staircase must be *specific* — a policy that says "we might slow down if things get bad" is not a policy. Also name:

- **The forum** where the policy is enforced (release meeting, on-call sync, weekly reliability review).
- **The escalation path** when the policy is breached (VP-eng approval, exec review).

### 7. What deliberately is NOT in this document (≈ 10 min)

Chapter 02 §6 named three things that look like SLIs but are not (fairness, product KPIs, cost). Write a short "out of scope for this document" section that names where these live instead — mod-309 for fairness, product-analytics for KPIs, the cost-budget document (exercise-02) for cost.

## Deliverable

A single Markdown document, 3–5 pages, titled `slo-<feature>.md`, structured as:

- Header — feature name, owner, on-call rotation, consumers, last-reviewed date, next-review date.
- Classical SLIs section (§1).
- Prediction-freshness SLI (§2).
- Quality-drift SLI, including per-slice (§3).
- Calibration and distribution-drift SLIs (§4).
- Retraining SLA SLI (§5).
- Error-budget policy staircase (§6).
- Out of scope (§7).

The document is written to be *executed*, not just archived. An on-call should be able to page against these SLIs; a reviewer six months later should be able to reconstruct why each threshold was set to the number it is.

## Acceptance criteria

- [ ] At least four classical SLIs, each with a PromQL/SQL definition, SLO target, error budget in human units, and multi-window burn-rate thresholds.
- [ ] Freshness SLIs cover at least three distinct feature-views; each target has a named justification (half-life / tolerance / cost).
- [ ] Quality-drift SLI is adapted to the actual label-landing regime; the `valid_events` filter is written out; per-slice SLIs cover the business-critical slice matrix.
- [ ] Calibration SLI uses a delta against a *locked baseline*, not an absolute floor.
- [ ] Distribution-drift SLI handles new categorical values (or explicitly acknowledges it does not).
- [ ] Retraining SLA target is derived from the environment's half-life, not "we always do 30 days."
- [ ] Every SLI has an alerting section with specific windows and thresholds — no "TBD" alert configurations.
- [ ] Error-budget policy staircase names specific behaviours at each budget threshold and specific approvers for exceptions.
- [ ] Every SLI has at least a *stub* runbook link (the full runbook is exercise-03; here, the SLI names what the runbook must cover).
- [ ] The document distinguishes SLIs from fairness metrics, product KPIs, and cost budgets; the last is a pointer to exercise-02.

## Stretch goals

- **Cost-line SLI companion.** For every SLI whose failure is *cheap-to-recover-from-but-expensive-to-tolerate* (a canary rollback), add a companion cost-attribution metric so the postmortem can reason about the dollar cost of the SLI breach.
- **Multi-window burn-rate defence.** For your fastest-critical SLI (usually availability), do the SRE-workbook calculation of the alert thresholds ([*Alerting on SLOs*](https://sre.google/workbook/alerting-on-slos/)): what burn rate should the fast-burn alert use, and what compliance-window window should slow-burn use? Numeric answers, not "roughly."
- **Cross-team dependency SLI.** For any SLI whose numerator depends on another team's service (label pipeline, feature-store), an explicit dependency SLI — the *other* team's SLO for the thing your SLI is reading. If the label-pipeline SLO is 99 % over 7 days and your quality SLI has to be 99 % over 7 days, the composition math says you cannot beat the *product* of the two; document it.
- **Feed the paired project.** Attach this SLO document as the reliability section of a paired project (e.g., `project-302-llm-augmented-ml-feature` or another). The document should be operable, not archived.
- **Peer review.** Trade documents with a peer. The reviewer walks the SLO document as an on-call would: "if the freshness SLI is red at 3 a.m., can I find the runbook link, execute the diagnostics, and know when to page the model owner?" Gaps become follow-up work.
