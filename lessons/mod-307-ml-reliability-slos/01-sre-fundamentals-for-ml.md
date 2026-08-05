# SRE fundamentals for ML: SLIs, SLOs, and error budgets

## Motivation

Ask an L20 ML engineer "is the model up" and you get a green Grafana panel. Ask an L30 ML engineer the same question and you get a follow-up: *"up for whom, over what window, on which SLI, and how much error budget do we have left this quarter?"* That difference is what this module installs.

Classical software SRE has a well-developed vocabulary — Service Level Indicator, Service Level Objective, Service Level Agreement, error budget — and a set of load-bearing practices that have kept internet-scale systems operable for two decades. Google's [*Site Reliability Engineering*](https://sre.google/sre-book/table-of-contents/) and the follow-up [*The Site Reliability Workbook*](https://sre.google/workbook/table-of-contents/) are the canonical texts; the [SLO chapter](https://sre.google/workbook/implementing-slos/) of the Workbook is the single most useful thing a senior ML engineer can read on this subject.

The uncomfortable part is that classical SRE was built for stateless request-response systems whose failure modes are "the request 500'd" or "the request took too long." ML systems fail in ways those two SLIs cannot see. A recommender that returns 200 OK in 40 ms with a technically-well-formed response is *up* by any classical SLI, and still catastrophically broken if the response is a stale ranking against a feature it thinks is fresh but is actually eight hours old.

This chapter is the SRE vocabulary at the level a senior ML engineer needs to speak fluently *before* extending it to ML-specific SLIs in chapter 02. The claim of this chapter is: the SRE framework is not wrong for ML — it is *incomplete* for ML. The right move is to keep the framework and add the ML-specific indicators it is missing, not to invent a parallel one.

## §1 — SLI, SLO, SLA: what each one actually is

The three terms are used interchangeably by teams that have never had to defend a P0. They are not interchangeable.

- A **Service Level Indicator (SLI)** is a *measurement*. A number, computed from live production data, over a window. Example: "the fraction of prediction requests in the last 5 minutes that returned within 100 ms." An SLI has no target attached to it — it is just the ratio a monitoring system emits.
- A **Service Level Objective (SLO)** is a *target* on an SLI. Example: "99.5 % of prediction requests must return within 100 ms, measured over a rolling 28-day window." An SLO has a threshold, a compliance window, and a defined action when the target is not met.
- A **Service Level Agreement (SLA)** is a *contract with an external party*, usually with financial consequences. Example: "we will refund X % of monthly fees if uptime falls below 99.9 %." SLAs are looser than SLOs by design — the SLO is where you want to be, the SLA is where you are legally required to stay. Most product ML teams have SLOs but no SLA; enterprise SaaS and vendor-facing platforms have both.

Chris Jones et al., ["Service Level Objectives"](https://sre.google/sre-book/service-level-objectives/) in the SRE book is the load-bearing chapter here. The three-layer distinction matters because *who* the number is aimed at differs — the SLI is aimed at the on-call, the SLO is aimed at engineering leadership and product, the SLA is aimed at customers and lawyers.

### The "good events / valid events" shape

Every SLI in the Workbook style has the same shape:

```
SLI = good_events / valid_events
```

*Good events* is the number the SLI wants to be high. *Valid events* is the denominator — the requests the SLI applies to, after filtering out things that should not count (health checks, synthetic traffic, requests from users who opted out of the treatment).

Two examples on a prediction service:

- **Availability SLI:** `good_events` = requests returning 2xx, `valid_events` = all requests except health-checks. SLO = 99.9 %.
- **Latency SLI:** `good_events` = requests returning within 100 ms, `valid_events` = all successful requests to the `/predict` endpoint. SLO = 99 %.

The `valid_events` filter is where a lot of the SLI's honesty lives. An SLI that only counts requests "when the model is actually being called" has quietly excluded the case where the request-router failed to invoke the model at all — which is a real user-facing outage.

## §2 — Error budgets: the shipping speed vs. reliability lever

The single most useful concept the SRE program introduced is the **error budget** — the complement of the SLO, spent by incidents, and used as a shipping-speed lever. Marc Alvidrez's chapter [*Embracing Risk*](https://sre.google/sre-book/embracing-risk/) is the origin.

If the SLO is 99.9 % availability over 28 days, the error budget is `1 - 0.999 = 0.1 %` of the 28-day window — roughly 40 minutes 20 seconds of allowable "bad." When incidents consume the budget, the team is *supposed* to slow down shipping until the budget refills. When incidents do not consume the budget, the team is *supposed* to be shipping faster — because the budget is being underspent, which usually means reliability is being over-invested at the cost of feature velocity.

The senior read is that an error budget is a *policy lever*, not a monitoring metric. Its purpose is to make the "should we ship this risky change" conversation quantitative. If the budget is at 78 % remaining and it is week 2 of the window, ship. If the budget is at 5 % remaining and it is week 2 of the window, do not ship — fix reliability instead. Google's [error budget policy chapter](https://sre.google/workbook/error-budget-policy/) documents both the mechanism and how it is enforced (freezing releases, requiring VP-level exception approval, etc.).

Two error-budget properties are especially load-bearing for ML systems:

- **The budget window is longer than any single incident.** A 28-day rolling window with a 40-minute budget survives one 20-minute outage without freezing shipping; two consecutive 30-minute outages exhaust the budget and trigger the freeze. The window length is a policy decision — too short (1 day) creates alert fatigue, too long (1 year) makes the budget useless for shipping-speed regulation.
- **The budget is spent by everything that breaches the SLO, not just outages.** Deploys that cause a latency regression, feature-pipeline bugs that mis-serve stale features, model rollouts that trip the error rate — all of these spend budget. That is the point.

### Burn rate and multi-window alerting

An SLO on its own tells you whether you are meeting the target. A **burn rate** tells you whether, at the *current rate of failure*, you are on track to burn the entire error budget before the window resets.

The pattern from the SRE workbook is [multi-window, multi-burn-rate alerting](https://sre.google/workbook/alerting-on-slos/):

- **Fast burn** — at the current failure rate, the entire budget will burn in a few hours. Page the on-call immediately.
- **Slow burn** — at the current failure rate, the budget will burn in days but is meaningfully depleting. Open a ticket, no page.

For an SLO of 99.9 % over 28 days, a burn rate of 14.4 means the entire 28-day budget will be consumed in 2 days at current failure rates — page. A burn rate of 1 means the failure rate exactly matches the SLO — the budget will last exactly one window and no more; a ticket, not a page.

The senior discipline: **alerting is on burn rate, not on the SLI itself.** Alerting on the raw SLI produces "the SLI dipped below the SLO for 30 seconds" pages that no on-call can act on. Alerting on burn rate answers "at this rate, do we have a problem this quarter" — which is the question that maps to action.

## §3 — What the SRE framework assumes that ML violates

The SRE framework was built for stateless request-response systems. The four assumptions that break when you point it at an ML system:

- **Assumption 1: correctness is binary.** A request succeeded (2xx) or failed (5xx). The response body's content is not part of the SLI. For ML, the response body's *content* is the whole point — a "successful" prediction that is confidently wrong is worse than a 500, because the 500 at least degrades gracefully.
- **Assumption 2: freshness is a caching concern.** A stale CDN response is a performance issue, not a correctness issue. For ML, freshness is a *correctness* dimension — a prediction made against a stale feature (the user's balance from yesterday, the fraud model's blocklist from last week) can be catastrophic even when the response is fast and well-formed.
- **Assumption 3: the system is deterministic modulo the request.** Same input, same output, until the code changes. For ML, the model itself changes on a retraining cadence; the *feature values* change with the world; the *label distribution* changes with users; every one of those is an implicit code change that never touched the request handler.
- **Assumption 4: the failure taxonomy is short.** Timeouts, 500s, rate limits, dependency failures. For ML the taxonomy is far longer — silent drift, feature-pipeline breakage, calibration collapse, adversarial-input handling, training-serving skew, cascading model degradation.

None of the four is a criticism of the SRE framework. Each is a signal that the SRE framework's SLI menu is *incomplete* for ML and the incident taxonomy is *incomplete* for ML. Chapter 02 authors the ML-specific SLIs that fill the gap; chapter 04 authors the incident taxonomy.

Sculley et al.'s [*Hidden Technical Debt in Machine Learning Systems*](https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html) is the load-bearing paper for why this gap exists — the "CACE" property (Changing Anything Changes Everything) means an ML system's boundaries are broader than a classical service's, and any SLI framework that stops at the request boundary is under-observing the system.

## §4 — What still transfers unchanged from classical SRE

The four assumptions above break for ML. Everything else transfers.

- **The good-events / valid-events shape.** Every ML SLI in chapter 02 is written in the same shape. A "prediction freshness SLI" is `predictions_using_fresh_features / all_predictions` over a window. A "calibration drift SLI" is `calibration_bins_within_tolerance / all_calibration_bins` over a window. The framework is unchanged; the SLIs are new.
- **The SLI / SLO / SLA distinction.** The prediction-freshness SLI is a measurement; the prediction-freshness SLO is a target; if the retraining SLA is contracted with a downstream team (the fraud model must be re-trained weekly and the fraud team is downstream), that is a real SLA even without money attached.
- **The error budget and burn rate.** An ML quality-drift SLO of "AUC on the daily eval set stays within 2 points of production baseline, 95 % of the days over a 90-day window" has an error budget: 4.5 days over 90. The burn rate discipline applies.
- **Multi-window alerting.** The fast-burn / slow-burn split applies to ML SLIs the same way it applies to classical ones. A calibration SLI that has burned 40 % of its 30-day budget in 3 days is a fast-burn page; one that is at 40 % of budget in 12 days is a slow-burn ticket.
- **The freeze-on-empty-budget policy.** When an ML SLO's budget is empty, the retraining pipeline should be paused (chapter 05), the feature-store deploys should freeze, the release meeting should not sign off on the next model. This is the same policy shape as a code-freeze on a classical SRE budget breach.
- **Postmortem culture.** Blameless, structured, action-item-driven. John Allspaw's [*The infinite hows*](https://queue.acm.org/detail.cfm?id=2353017) and Google's [postmortem chapter](https://sre.google/sre-book/postmortem-culture/) are the reference. Chapter 04 adapts the postmortem template for ML incidents.

The framework works. The gap is in *what to measure*. That is what chapter 02 fills.

## §5 — The classical SLI menu (uptime, latency, throughput)

Before adding ML-specific SLIs, make sure the classical ones are actually in place for the ML system. It is common for ML teams to skip the classical SLIs because "the model is the interesting part" — and to spend the first quarter of production incidents on classical failures (a load-balancer misconfiguration, a certificate expiry, a downstream schema break) that a classical SLI would have caught cheaply.

The three-item minimum every prediction service has:

- **Availability SLI.** `2xx_responses / (2xx + 5xx + timeouts)` over a rolling 5-minute window. SLO: 99.9 % or 99.95 % over 28 days is the industry-standard range for user-facing services. Google's Workbook [availability chapter](https://sre.google/workbook/implementing-slos/#choosing-a-target) discusses target selection.
- **Latency SLI.** `requests_faster_than_target_ms / total_requests` over a rolling 5-minute window. SLO: 99 % below the target (100 ms, 500 ms, 1 s — depending on feature). Gil Tene's [*How NOT to Measure Latency*](https://www.infoq.com/presentations/latency-response-time/) is the reference on coordinated omission and the reason the SLI is on the *tail* percentile, not the mean.
- **Throughput SLI or ceiling.** RPS the service is serving vs. the RPS the service is provisioned to serve. Not always expressed as an SLO — often as a ceiling with an auto-scale trigger — but the number is worth tracking because it is the leading indicator of the availability SLI going red.

The four less-common but still-classical SLIs that catch ML-specific classical failures:

- **Feature-fetch latency SLI.** The prediction service's inference time is often *not* the bottleneck; the feature-fetch call to the online feature store is. A separate SLI on the feature-fetch p99 catches "the feature store is slow" as its own signal, not as a mysterious tail latency on the predict endpoint.
- **Feature-fetch error rate SLI.** Feature-fetch failures (miss, timeout, schema error) that fall through to a default value are often the source of a silent quality regression. The SLI tracks the fallback rate; a rising fallback rate is a leading indicator of the quality-drift SLI going red (chapter 02).
- **Model-load success SLI.** `successful_model_loads / attempted_model_loads` on rollout. Catches "the retraining pipeline promoted a model artefact the serving path can't load" (see chapter 05).
- **Dependency SLIs.** If the model calls a downstream LLM vendor (mod-304), a downstream feature-vector index, or a downstream reranker, each of those has its own availability and latency SLI, and the composite SLI of the prediction service is bounded by their product. Ben Treynor Sloss et al.'s ["Service Level Objectives"](https://sre.google/sre-book/service-level-objectives/) chapter is where the compound-SLI math is worked out.

The three-item minimum plus the four extension SLIs are the *classical* SLI set for an ML system. The ML-specific SLIs from chapter 02 are *additions to* this set, not replacements.

## §6 — Worked example: the SLO document

An SLO is not a number — it is a *document*. Kept in the repo, reviewed on a schedule, referenced during incidents and release meetings. A worked skeleton for a prediction service:

```markdown
# SLO — recommendation-model-serving

## Ownership

Owner: recommendation-platform team
On-call rotation: rec-platform-oncall
Consumers: homepage-feed, category-page, cross-sell rail
Last reviewed: 2026-02-14
Next review: 2026-08-14

## SLIs and SLOs

### 1. Availability
- SLI: `sum(rate(http_requests_total{status=~"2.."}[5m])) / sum(rate(http_requests_total{status!~"5.."}[5m]))`
- SLO: 99.95% over 28 days
- Error budget: 20 minutes per 28-day window
- Alerting: fast burn (14.4× budget rate over 1 hour) → page. Slow burn (3× budget rate over 6 hours) → ticket.

### 2. Latency (p99 within 80 ms)
- SLI: `histogram_quantile(0.99, sum by (le)(rate(request_duration_seconds_bucket{route="/predict"}[5m])))`
- SLO: 99% of 5-minute windows have p99 below 80 ms, over 28 days
- Error budget: ~6.7 hours per 28-day window
- Alerting: fast burn (14.4×) → page. Slow burn (3×) → ticket.

### 3. Feature-fetch fallback rate
- SLI: `feature_fetch_fallback_events / total_feature_fetch_events`
- SLO: fallback rate < 0.5% over 24-hour windows
- Error budget: 72 minutes per day
- Alerting: fast burn only — 10× budget rate over 30 minutes → page.

### (ML SLIs from chapter 02 land here — freshness, quality drift, calibration.)

## Error budget policy

- ≥ 50% budget remaining: normal shipping cadence.
- 25–50% remaining: pre-launch reliability review required for any change touching the predict path.
- ≤ 25% remaining: freeze on non-reliability changes; retraining pipeline pauses; VP-eng sign-off for any release.
- Budget exhausted (< 0%): full freeze; all engineering effort on burn-down; incident review at team level and platform level.
```

Two properties of this document that are worth naming out loud:

- **The alerting section is inside the SLO document, not a separate runbook.** The alert threshold is a function of the SLO. Changing the SLO without updating the alerting is a common failure mode — the on-call is paging against an obsolete target.
- **The error budget policy is explicit.** Not "we might slow down if things get bad." A staircase of budget-remaining thresholds and the *specific policy consequence* at each. This is what makes the budget a lever instead of a monitoring line.

## §7 — Where classical SLOs stop being enough

Everything above is available to a well-run classical service team. Point it at an ML system and the following gaps appear:

- **The availability SLI misses "confidently wrong."** A recommender that returns a valid response with a stale ranking, or a fraud model that scores every transaction as safe because a feature-pipeline broke and every feature is defaulted, will show 100 % availability. The user is served nonsense; the SLI does not care.
- **The latency SLI misses "fresh vs. stale."** The prediction is fast; the feature it consumed is 8 hours old. Latency is green; the model's decision is against data no human would consider current.
- **No SLI covers "the model still knows what it's doing."** Quality drift, calibration drift, distribution shift — none of these show up on availability, latency, or throughput. The classical SLI set can be green while the model is silently degrading.
- **No SLI covers "we retrained recently enough."** For a fraud model with a monthly-shifting adversary or a recommender with a weekly-turning catalogue, the *time since last successful retrain* is a first-class reliability metric. There is no classical SLI for it.

Chapter 02 authors the SLIs that fill these gaps: prediction freshness, quality-drift, retraining SLA, calibration SLIs. The SRE framework of §1–§4 stays; the SLI menu of §5 extends.

## Summary

An SLI is a measurement (`good_events / valid_events` over a window); an SLO is a target on an SLI over a compliance window; an SLA is a contract with an external party. The error budget (`1 - SLO`) is a policy lever: budget available means ship; budget empty means freeze. Burn-rate alerting on fast and slow windows is the discipline that turns SLOs from monitoring lines into on-call actions. The classical SLI menu — availability, latency, throughput — plus feature-fetch and dependency SLIs is the *floor* every ML service has. What classical SLIs miss for ML systems — confidently-wrong predictions, stale features, quality drift, calibration collapse, retraining lag — is what chapter 02 authors. The framework is not wrong for ML; its SLI menu is incomplete for ML, and the right move is to keep the framework and add the ML-specific indicators.
