# Cost budgets and quotas that engineering enforces

## Motivation

The uncomfortable truth about ML cost is that most teams *report* cost — a dashboard, a monthly finance review, a slide with a curve going up and to the right — and never *enforce* cost. Reporting and enforcing are different disciplines. The reporting posture answers "how much did we spend?" a month after the fact. The enforcing posture answers "how much are we allowed to spend today, and what happens when we try to spend more?" — and the answer is a rate-limit, a queue, a fallback, or a refusal, at the point the request is made.

Chapter 02 named cost as *not an SLI in the strict sense* — it does not have the `good_events / valid_events` shape. But it is a first-class reliability discipline, and the SRE framework has a cousin concept for it: the **quota**. Google's [*SRE Workbook* chapter on managing critical state](https://sre.google/workbook/managing-critical-state/) and [*Managing load*](https://sre.google/sre-book/handling-overload/) both treat quotas and rate-limits as reliability mechanisms — not just cost controls. They apply directly to ML cost management.

The uncomfortable part of ML cost, and specifically LLM-augmented ML cost, is that the invoice is real, arrives fast, and the failure mode of "we shipped the feature and forgot to enforce the budget" is a dollar-denominated incident that a good SRE program stops at the request path. Mod-304 chapter 04 (Cost and latency guardrails) established the envelope for a single LLM feature; this chapter installs the discipline for the *whole ML footprint* — models, feature pipelines, retraining, batch scoring, vendor calls — and it installs the *enforcement primitives* that keep the budget from being an aspiration.

Two claims for this chapter:

- **Every cost budget must be enforced at the point the resource is consumed, not audited a month later.** Enforcement is a rate-limit, a per-tenant quota, a max-spend controller, a request-blocker. Reporting alone means the invoice is the enforcement mechanism.
- **Every cost budget is a policy artefact with an owner, a target, a hierarchy, and a policy-on-breach.** "Reduce cost" is not a budget; "$8 K per month on this feature, hard cap at $10 K, degradation path when we cross $9 K" is a budget.

## §1 — Where ML cost actually accrues

Before designing enforcement, sketch the cost surface. ML systems have five recurring cost lines, and a budget that only covers one of them is only enforcing one of them.

- **Serving compute.** GPU / CPU hours for the online prediction path, feature-fetch infrastructure, model-serving replicas. Usually cloud-provider metered (EC2, GKE, EKS, GPU instances). The FinOps Foundation's [*FinOps Framework*](https://www.finops.org/framework/) is the reference for cloud-native cost management practices.
- **Training compute.** GPU hours for the retraining pipeline. Often bursty (once a week, but a large burst), sometimes continuous (retraining daily on a rolling window). This line is the one most likely to have a "we set up automated retraining and forgot to cap it" incident.
- **Data-plane storage and I/O.** Object storage for feature snapshots, warehouse queries for training-data assembly, streaming pipeline throughput. Warehouse costs (BigQuery, Snowflake, Redshift) are meter-per-scan and are the second most common surprise line.
- **Vendor API calls.** LLM providers (Anthropic, OpenAI, Gemini, Bedrock), embedding APIs, vector-search vendors, third-party ML APIs. Mod-304 chapter 04 covered this in depth for LLM-augmented features. The per-request cost is variable, the invoice is real, and vendor budgets are usually the first place enforcement matters.
- **Data-labelling and human review.** Human-annotator time, RLHF budget, manual-review queues. Not always attributed to "ML cost" but structurally part of the same envelope; a labelling budget that runs out stops the retraining pipeline as surely as a compute budget that runs out.

Every ML feature's cost budget covers at minimum lines 1, 2, 3, 4 for that feature. Skipping any of them is where the "the total was 3× what we projected" postmortem starts.

## §2 — Report vs. enforce: the difference

The two postures produce different systems.

**Reporting posture.**

- Metric: a dashboard showing dollars per hour, per day, per month, per team.
- Feedback loop: monthly finance review; someone frowns; the team promises to look into it.
- Enforcement mechanism: none. The invoice is the enforcement mechanism.
- Failure mode: unbounded spend during an incident, a launch, a bug, a runaway loop. The reporting posture cannot stop a bug from consuming $500 K in a weekend.

**Enforcing posture.**

- Metric: a *live* current-spend counter compared against a budget.
- Feedback loop: at the moment the counter crosses a threshold, some *system behaviour changes*. Requests get rate-limited; a lower tier gets selected; a cache-only path takes over; queued jobs are paused; the retraining pipeline stops.
- Enforcement mechanism: code. A rate-limiter, a quota system, a max-spend controller, a request-blocker. The enforcement is inside the service, not inside the invoice.
- Failure mode: the enforcement is too aggressive and legitimate traffic is dropped. Predictable, tunable, incident-shaped in an ordinary way — not a $500 K invoice-shaped incident.

The senior read: **the enforcement posture is a design step, not an operations aftermath.** The team that ships without enforcement will get to the enforcement posture eventually, after a cost incident. The team that ships with enforcement gets there before the incident. Same destination, different price.

Every mature cost-management program has both:

- **Reporting for planning.** Where is spend going? Which features are cost-efficient? What is the marginal cost per unit of business value? The FinOps [Capabilities](https://www.finops.org/framework/capabilities/) framework is the industry reference.
- **Enforcement for reliability.** Is a bug about to consume the month's budget in an afternoon? Is a runaway loop dumping 100× the expected calls to Anthropic? Enforcement is what stops the incident.

## §3 — The budget hierarchy

Every cost budget lives in a hierarchy. Google's [*Managing load*](https://sre.google/sre-book/handling-overload/) chapter uses "utilization budget" the same way; the FinOps allocation model uses "budget hierarchy" with the same shape.

- **Feature budget.** Per-feature, per-service. Common shape: monthly spend cap for a specific ML feature — the recommender, the fraud model, the LLM-augmented triage. This is the level where the "how much does this feature cost per month" question is answered.
- **Team budget.** The team's aggregate of all their feature budgets. Usually with slack — a team's team budget is *less* than the sum of feature budgets because not every feature runs at ceiling simultaneously.
- **Org budget.** Finance-signed dollar allocation. The team budgets sum to less than the org budget.
- **Emergency budget.** A reserve at the org level for incidents. When a feature or team blows its budget for a legitimate reason (a launch, an outage response, a spike), the emergency budget covers the overage — with the *understanding* that a postmortem follows.

Every level in the hierarchy has:

- **A hard cap** — a maximum that triggers enforcement action.
- **A soft cap or "alert" threshold** — usually 80 % of hard cap — that triggers a warning.
- **A degradation path** — what happens at the cap.
- **An owner** — a specific engineer or team accountable for the number.

The hierarchy is *itself* an enforcement mechanism: a team that has consumed 90 % of its team budget cannot spin up a new feature that would exceed the remainder without escalating. The team budget is a real ceiling, not a report.

## §4 — Enforcement primitives

Four primitives cover most of the enforcement surface. Each has a specific failure mode and a specific implementation shape.

### Rate limits and quotas

The most common enforcement primitive. Every mature request path has one.

- **Per-tenant quota.** Each tenant (a customer, a service, a team) is allowed N requests per second and M requests per day. Enforcement is a token-bucket or leaky-bucket at the ingress. Google's [*Handling overload*](https://sre.google/sre-book/handling-overload/) chapter covers the classical shape.
- **Per-model quota.** A specific model gets N inference calls per minute. The rest are rejected or queued. This is where the vendor-API budget lives — an LLM router that stops calling Anthropic when the daily quota is exhausted.
- **Per-request cost cap.** Each request gets a maximum cost budget (mod-304 chapter 04's per-request envelope). A request that would exceed the cap is downgraded to a cheaper model tier, or served a fallback, or rejected outright.

Two failure modes to watch:

- **Rate limits without observability.** A request rejected by a quota looks like a service failure to the caller. Every rate-limited request must be *logged* with a reason, *reported* in a metric (`rate_limit_events_total{reason=<quota>}`), and *visible* on the SLO document as a distinct signal.
- **Rate limits at the wrong layer.** A quota at the ingress does not enforce the LLM vendor's per-second limit; a quota at the LLM-vendor SDK does not enforce fair sharing between tenants. Multiple layers of quota, each doing one job.

### Circuit breakers

When a downstream is failing or degrading (a slow LLM vendor, an overloaded feature store), the circuit breaker refuses to make the call *before* the timeout, returning the degradation-path answer instead. Michael Nygard's [*Release It!*](https://pragprog.com/titles/mnee2/release-it-second-edition/) is the load-bearing reference; the Netflix [Hystrix](https://github.com/Netflix/Hystrix) documentation (and its successors [resilience4j](https://resilience4j.readme.io/), [Sentinel](https://sentinelguard.io/en-us/)) covers the pattern in the JVM ecosystem.

For cost specifically, the circuit breaker is what stops a *runaway* — a bug that produces 100× the expected calls to a paid vendor. The breaker trips on the *rate* of spend, not the *total* — a per-minute spend rate that exceeds N× the expected rate opens the breaker for K minutes, forcing every request through the cheaper degradation path.

The shape:

```python
class SpendRateLimiter:
    """Opens when spend/minute exceeds N× baseline for W consecutive windows."""

    def __init__(self, baseline_dollars_per_min, factor, window_count):
        self.baseline = baseline_dollars_per_min
        self.factor = factor
        self.windows_needed = window_count
        self.recent_windows = deque(maxlen=window_count)

    def observe(self, dollars_spent_last_minute):
        self.recent_windows.append(dollars_spent_last_minute)
        return sum(1 for w in self.recent_windows if w > self.baseline * self.factor) \
            >= self.windows_needed

    def call_downstream(self, req):
        if self.is_open():
            return self.degraded_answer(req)
        return self.upstream_call(req)
```

The circuit breaker is *not* a monthly-budget enforcement — it is a runaway-detection enforcement. The two do different jobs; both are needed.

### Priority queues and load shedding

When cost is bounded by throughput (batch scoring, training pipelines, offline enrichment), the enforcement primitive is a *priority queue* that runs high-priority jobs first and *sheds* low-priority ones when the budget is exhausted for the window. Google's [*Handling overload*](https://sre.google/sre-book/handling-overload/) chapter discusses this as "graceful degradation."

For an ML system, the pattern is worth spelling out because the priority is often *business* priority, not engineering priority:

- **Priority 1.** Real-time serving for user-facing paths. Never shed.
- **Priority 2.** Retraining runs whose SLA (chapter 02 §4) is imminent. Shed with reluctance.
- **Priority 3.** Ad-hoc backfills and re-scoring runs. Shed first.
- **Priority 4.** Experimentation / offline evaluation runs. Shed silently.

The shedding decision is a *policy* — which tier gets shed at which budget threshold — and the policy is a repo artefact reviewed as code.

### Kill switches

The nuclear option. A single flag that stops the resource-consuming behaviour entirely. Every LLM-augmented feature has one; every retraining pipeline has one; every batch scoring pipeline has one.

Kill switches are used in two scenarios:

- **The cost incident.** A runaway loop is consuming budget faster than the circuit breaker can react; the on-call turns the feature off with one flag. Chapter 04 discusses the incident-response usage.
- **The end-of-budget freeze.** The org's monthly budget is exhausted for a feature; the feature is turned off until the next window. This is rare in mature orgs (because the budget hierarchy caught it upstream), but the kill switch is what makes it possible.

The kill switch has three properties:

- **One action.** Not "run the deploy pipeline in reverse"; not "page the ML engineer." One flag flip.
- **Rehearsed.** Every quarter, the kill switch is exercised against a canary or staging tenant. A kill switch that has never been tested is not a kill switch.
- **Auditable.** Every kill-switch invocation is logged with the reason and the invoker. Repeated invocations are a signal that the primary enforcement primitives are undersized.

## §5 — Instrumentation: what to actually meter

The enforcement primitives above require live cost signals. Two data-plane properties they need:

- **Cost per unit of work is *knowable at call time*.** The service must be able to estimate, for a request in flight, what it will cost. This is where the mod-304 chapter 04 per-request cost estimate lives; for classical ML, the analogue is per-inference GPU-seconds and per-feature-fetch storage-I/O bytes.
- **Cost is aggregated in a fast-enough window.** The cost signal has to be current within *seconds*, not *hours*, for enforcement to be useful. A monthly bill from AWS is not an enforcement signal; a real-time cost counter emitted by the service is.

Practical instrumentation:

- **Every request emits a `cost_units` metric.** Even if the cost is estimated (not yet billed), the estimate goes into a metric. Vendor providers usage APIs (Anthropic's [`usage`](https://docs.anthropic.com/en/api/messages) field, OpenAI's `usage` object) give the actual token counts per response; converting them to dollars is arithmetic against the [pricing page](https://www.anthropic.com/pricing).
- **Per-feature, per-tenant, per-model tags.** The `cost_units` metric carries labels for the feature, the tenant, and the model tier. That lets the rate-limiter and the priority queue answer "which slice consumed the last 20 minutes' spend?"
- **A running total against the budget.** A `spend_dollars_current_window` gauge is compared against the budget in the enforcement code. The gauge is updated in-memory and flushed to a store (Redis, a time-series DB) frequently enough to survive a service restart without losing accumulated spend.
- **A "would-have-cost" counter under the kill switch.** When the feature is turned off, keep counting what it *would* have cost if it had been on. That is the signal for when to turn it back on and the signal for the postmortem.

The AWS [Cost Anomaly Detection](https://aws.amazon.com/aws-cost-management/aws-cost-anomaly-detection/), GCP [Budget Alerts](https://cloud.google.com/billing/docs/how-to/budgets), and Azure [Cost Management](https://azure.microsoft.com/en-us/services/cost-management/) are the cloud-provider-side reporting layer. They are complements to the in-service enforcement, not substitutes.

## §6 — Composing the cost budget with the SLO document

Chapter 01 §6 and chapter 02 §5 built a composed SLO document. The cost budget extends it:

```markdown
## Cost budget

### Feature: LLM-augmented ticket triage

- Monthly spend cap: $8,000
  - Alert threshold: $6,400 (80% of cap) → ticket, 24h SLA to reduce burn rate or request budget lift.
  - Hard cap: $8,000 → kill switch trips.
- Daily spend cap: $400 (nominal), $600 (hard)
  - Alert: $320 (80% of nominal) → notify feature team.
  - Hard cap: $600 → circuit breaker opens; all non-cache-hit requests take fallback path.
- Per-request cost envelope (from mod-304 chapter 04): $0.02 p50, $0.08 p99.
  - Requests exceeding $0.08 are downgraded to a smaller-tier model.
  - Requests exceeding $0.20 (12 × p99) are logged as anomalies and served the deterministic fallback.
- Runaway detection: 5× normal spend rate sustained for 3 consecutive 1-minute windows → circuit breaker opens for 15 minutes, escalating to on-call.

### Owners

- Feature owner: ml-ticket-triage team.
- Reliability owner: rec-platform on-call.
- Finance stakeholder: platform-finance@.

### Degradation paths

- At soft cap (80%): downgrade routing threshold — more requests get the smaller-tier model.
- At hard cap: kill switch trips; fallback is a rule-based routing table with a "we could not use the ML system, this ticket is routed by rule" annotation for the analyst.
- The fallback's SLI (fraction-of-tickets-routed-by-rule) is monitored — a rising rule-routed rate is a leading indicator that either the budget needs a lift or the model has an efficiency regression.

### Policy on budget-blown incident

- Postmortem within 5 business days.
- If the cause is a spike in legitimate traffic: reassess whether the budget was undersized. Finance-review to adjust cap.
- If the cause is a bug (runaway loop, prompt regression, cache poisoning): fix and add a specific test for the failure mode.
- If the cause is adversarial (prompt injection, denial-of-wallet): escalate to security review — mod-309 (Responsible AI) chapter on adversarial patterns.
```

Two properties of the document worth naming:

- **The budget document names the *degradation path* explicitly.** Not "we'll turn the feature off if we blow the budget"; a specific fallback with its own SLI. The user's experience under degradation is designed.
- **The runaway detection is a *separate* mechanism from the budget cap.** The budget cap answers "we spent enough this month." The runaway detector answers "we are spending 100× the expected rate right now." Both are needed; they trigger different actions.

## §7 — What deliberately does not go here

Two adjacent disciplines that are close to cost but are their own:

- **Capacity planning.** The forecast of *what* to buy. Is a separate exercise, informed by cost reporting but not equivalent to it. Google's [*Data Processing Pipelines*](https://sre.google/sre-book/data-processing-pipelines/) chapter and the FinOps [Capacity Planning](https://www.finops.org/framework/capabilities/optimize-cloud-usage-cost/) capability cover it.
- **Cost-of-goods analysis for pricing.** How much does an ML-augmented feature actually cost to serve per user, and what does that mean for pricing tiers? A product-finance question, not a reliability question. The instrumentation from §5 feeds it, but the decision is elsewhere.

Both are downstream of the instrumentation this chapter installs, and both benefit from a mature reporting posture — but the *reliability* discipline is the enforcement mechanism, and that is what this chapter is about.

## §8 — Where this chapter hands off

- **Mod-304 chapter 04 (Cost and latency guardrails for LLM-augmented features)** is the per-feature envelope shape that this chapter's aggregate budget builds on. A team that has not read mod-304 chapter 04 will under-specify the per-request lever inside the budget.
- **Chapter 04 (ML incident response)** uses the enforcement primitives — kill switches, circuit breakers, priority queues — as first-line responses to a runaway or budget-blown incident.
- **Chapter 05 (Retraining as a deploy)** is where the retraining-pipeline cost sits inside this framework; the retraining pipeline has its own budget line, and a retraining run that consumes more than its expected budget is a signal that something has regressed (data volume, model size, training convergence).
- **Mod-306 chapter 01 §2 (guardrails)** is where the online-A/B *cost guardrail* is enforced during an experiment; the cost budget of this chapter is the number the A/B guardrail is named against.
- **Mod-308 (Platform collaboration)** is where the cost-attribution schema is agreed with the platform team — a chargeback model that lets each feature's cost line be actually attributed, not shared as overhead.

## Summary

Cost is a first-class reliability discipline, and the difference between a reporting posture (dashboard, monthly review, a frown at the invoice) and an enforcing posture (rate-limits, quotas, circuit breakers, priority queues, kill switches — every one wired into the request path) is what makes a budget an enforced ceiling rather than an aspirational number. Every ML feature has a budget hierarchy (feature → team → org → emergency reserve) with a hard cap, a soft cap, an owner, and a degradation path. The enforcement primitives — quotas, circuit breakers, priority queues, kill switches — each have a specific failure mode they catch and a specific place they belong. Instrumentation must be *live* (seconds, not hours) and *granular* (per feature, per tenant, per model). The composed reliability document extends the SLO document from chapters 01 and 02 with a cost-budget section, complete with degradation paths and runaway-response mechanisms. Chapter 04 turns the enforcement primitives into incident-response tools; chapter 05 wires the retraining pipeline into the same discipline.
