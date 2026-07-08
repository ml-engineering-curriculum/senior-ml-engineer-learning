# Observability for LLM-augmented systems

## Motivation

Chapter 04 sketched the cost and latency *envelope* every LLM-augmented feature has to carry. This chapter is how you *see* the envelope — the instrumentation that turns a design commitment into a runtime signal a team can actually read at 03:00 when the alert fires.

Classical ML observability is largely solved territory. Feature freshness, model score distributions, calibration drift, and prediction volume are metrics the ML platform track paved a road for years ago. Chip Huyen's *Designing Machine Learning Systems* chapter 8 remains the canonical framing — the "distribution shift" plus "prediction distribution" plus "quality drift" triad the ML platform monitors on your behalf.

LLM-augmented systems break that triad in three uncomfortable ways:

- **The "score distribution" is a token distribution.** There is no calibrated probability to alert on; the closest analogues are structured-output conformance, refusal rate, and downstream user metrics. All are noisier signals than a classifier's probability histogram.
- **The bill is a first-class SLI.** Every request costs dollars per invoice line-item (chapter 04). Cost is not "an ops concern" — it is a signal the on-call watches beside latency.
- **The dependency is on someone else's stack.** Vendor rate limits, vendor incidents, vendor model-version cutovers, and vendor pricing changes all show up first in *your* observability, not in a vendor status page. If you cannot attribute a Tuesday-afternoon quality regression to Tuesday morning's silent model-version snap on the vendor side, you will spend a week debugging your own prompt.

The senior read is: **LLM observability is not an add-on to classical observability, and it is not "just add tracing." It is a distinct instrumentation surface with its own signals, its own attribution model, and its own alerting shape.** Chapter 04 set the envelope; this chapter is the sensor grid that watches it. Chapter 06 is where you make the delegation contract that says who owns which sensors when a peer track takes over the deeper LLM work.

## What you are instrumenting — the five signal families

Every LLM-augmented feature emits (or should emit) five families of signal. If any family is missing from your production system, an entire class of incident is invisible.

1. **Invocation metadata.** Which prompt, which prompt version, which model, which model version, which routing branch, which caller (feature ID), which tenant / user (hashed), which request ID. Without this, downstream metrics have no attribution.
2. **Token and cost accounting.** Input tokens, output tokens, cached-input tokens, tool-use tokens, dollars per call, dollars per feature per day. This is the cost SLI from chapter 04 made observable.
3. **Latency components.** TTFT, TPOT, wall-clock latency, network component, parsing / post-processing component, retry count, timeout hits. This is the latency SLI from chapter 04 made observable.
4. **Quality signals.** Structured-output conformance rate, refusal rate, safety-filter trips, downstream metric (user click / edit / thumbs-down), online-eval sample score, LLM-as-judge sample score. This is the quality SLI — the hardest to define, the one this chapter spends the most words on.
5. **Reliability signals.** 429 rate, 5xx rate, timeout rate, circuit-breaker state, fallback-branch invocation rate, cache hit rate. This is the reliability SLI paired with the cost + latency SLIs.

Every dashboard, alert, and per-slice breakdown you build maps onto one of those five families. Chapter 07 of `ml-engineer-learning` covers classical monitoring; this chapter's five families are what you add on top when an LLM is in the request path.

## Structured logs, metrics, traces — pick the tool per signal

Three complementary sinks, each doing what it is good at. The mistake most L20s make is to try to make one of them do all three jobs.

### Structured logs

Emit one structured log line per LLM invocation, containing the invocation metadata plus a redacted / truncated version of the request and response. This is your evidence store. When someone asks "why did this specific request return that specific answer" — a support escalation, a legal-hold request, a post-incident review — you need the log line, not the metric.

- **Redaction matters.** The prompt often carries PII (the user's ticket text); the response often carries PII (a summary the user has not yet consented to store). Follow the same redaction discipline as any other logging surface. See mod-309 for the governance shape.
- **Sampling matters.** Full-fidelity logs at high RPS are expensive. Sample: 100 % on error, 100 % on the tail (p99+ latency), a fixed fraction (e.g. 1 %) on the golden path. Retain enough to reconstruct a debugging session.
- **Fingerprints, not raw text, on the metrics side.** A `sha256` of the redacted prompt lets you count "how many distinct prompts" without indexing raw text.

### Metrics

Pre-aggregated numbers per invocation, exported to a metrics backend (Prometheus, Datadog, CloudWatch) with labels — feature ID, prompt version, model, routing branch. Metrics are what the dashboards render and what the alerts fire on. Rules that keep the metrics tractable:

- **Fixed low-cardinality labels only.** `feature_id`, `prompt_version`, `model_id`, `model_version`, `branch`, `region`, `tenant_bucket`. Never a raw user ID, never the raw prompt, never a free-form error message. Prometheus's [naming and labels best practices](https://prometheus.io/docs/practices/naming/) is the canonical reference on why unbounded labels destroy metric backends.
- **Histograms, not just gauges.** Latency, token counts, cost per call all get histograms so you can query p50 / p95 / p99. A gauge of "mean latency" hides the tail; the tail is where LLM incidents live (chapter 04, the coordinated-omission section).
- **Counters for count-like events.** Number of invocations, number of 429s, number of timeouts, number of circuit-breaker trips, number of cache hits — all counters, so alerts can express "rate of change over five minutes."

### Traces

One distributed trace per user request, with spans for each LLM invocation, retrieval call, tool call, and post-processing step. Traces show *where* time is spent inside a request. When a p99 latency alert fires and the metrics say "the LLM span is 8 s," the trace tells you *which* LLM span, in which routing branch, on which retry.

The [OpenTelemetry Generative AI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) define the standard span attributes: `gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.response.model`, `gen_ai.response.finish_reasons`. Use the standard names. Every observability vendor has decided to converge on them, and using non-standard names is a self-inflicted lock-in.

## The vendor half of the story

You are not the only one instrumenting the LLM call. The vendor is emitting its own signals about your traffic, and the senior on-call knows which vendor surface to read for what.

- **Usage APIs.** Anthropic's [usage and cost APIs](https://docs.anthropic.com/en/api/usage-cost-api) and OpenAI's [usage endpoints and dashboard](https://platform.openai.com/docs/api-reference/usage) return per-day, per-model, per-key aggregates. Reconcile against your own token counters weekly. A drift between the two is a sign of a missing invocation path in your instrumentation.
- **Status pages.** Anthropic's [status page](https://status.anthropic.com/), OpenAI's [status page](https://status.openai.com/), Google Cloud AI's [status](https://status.cloud.google.com/), and AWS Bedrock's [health dashboard](https://health.aws.amazon.com/) are the first place to check when your error-rate SLO trips. Wire them into your on-call runbook.
- **Provider metrics and logs.** Vendor consoles surface per-request logs (Anthropic's [console](https://console.anthropic.com/), OpenAI's [platform dashboard](https://platform.openai.com/logs)) that can back-fill missing detail your own logs did not capture. Do not depend on them as the primary log store — they are not indexed the way your logs are — but they are worth consulting during an incident.
- **Rate-limit and quota telemetry.** Vendor rate-limit headers (`anthropic-ratelimit-*`, OpenAI's `x-ratelimit-*`) are emitted on every response. Log them. When you approach the ceiling, they are the leading indicator; the 429s are the lagging one.

## The prompt-version axis

Every metric and log line above is nearly useless if you cannot attribute it to a specific prompt version and model version. Chapter 02 made "the prompt is versioned like code" a discipline; this is the payoff.

- **Emit `prompt_version` as a label / attribute on every metric and span.** When someone edits the prompt and quality regresses, you see the regression aligned with the version snap on the same time-series chart. Without this, quality drifts are invisible until the user metric moves — and by then the deploy is a week old.
- **Emit `model_id` and `model_version` too.** Hosted models version silently under stable names ("we're rolling out a small update to the default model") on the vendor side. Pin explicit versioned model identifiers when the vendor exposes them (Anthropic's [model deprecations and versions](https://docs.anthropic.com/en/docs/about-claude/models/overview) and OpenAI's [model list](https://platform.openai.com/docs/models) both document this). Your metrics carry the pinned version so a silent vendor cutover is at least *visible* even if you did not authorise it.
- **Chart per-version.** A quality metric plotted "by prompt version" and "by model version" is where regressions announce themselves. A single line labelled "quality" is a metric that catches nothing.

## Quality signals — the hard part

For a classifier, you monitor calibration, ranking metrics, and downstream user behaviour. For an LLM invocation, none of those apply directly. You need signals that are (a) cheap enough to compute on every request, (b) meaningful enough that a change in the number means something in the world.

The stack the senior ML engineer builds — in order from cheapest / weakest to most expensive / strongest:

### 1. Structured-output conformance

If the prompt asks for JSON conforming to a schema, emit a counter on schema-validation success and failure. Rate of failure over time is a leading indicator of prompt drift, model change, or a distribution shift in the input.

- OpenAI's [structured outputs](https://platform.openai.com/docs/guides/structured-outputs) and Anthropic's [tool use / structured output](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview) are the vendor-provided disciplines. Even outside these, JSON schema validation on the client is a five-line addition to the client wrapper and pays off the day the vendor changes something subtly.
- A rising conformance-failure rate on a stable prompt version is a strong signal that the model version has snapped under you.

### 2. Refusal and safety-filter rate

Vendors' safety filters trip and return refusals in a well-defined way (Anthropic returns a `stop_reason` of `refusal`; OpenAI surfaces safety and moderation via [moderation APIs](https://platform.openai.com/docs/guides/moderation)). Log and count these. A sudden climb in refusal rate on a formerly-clean prompt is a vendor policy change; a slow climb is often an input-distribution shift bringing more adversarial content into the funnel.

### 3. Downstream user signal

Whatever the user does *after* the LLM output — click, edit, thumbs-down, complete the funnel, escalate to a human — is a real quality signal, and one that does not require any labelling investment. Wire it. `edit_rate` on a summariser, `thumbs_down_rate` on a draft-reply feature, `escalation_rate` on an assistant are all first-class quality SLIs even though none of them look like `precision` or `recall`.

The trap is treating downstream signals as the *only* quality signal. They are lagging; a step-change in prompt quality will show up on the downstream signal only when enough traffic has run through the new version. Combine with the leading indicators (structured-output conformance, LLM-as-judge samples) so regressions are visible in hours, not weeks.

### 4. Online LLM-as-judge sampling

Sample a small fraction of requests (0.1 %–1 %) and run an LLM-as-judge evaluation on the (input, output) pair, emitting the judge's score as a metric. mod-305 covers when LLM-as-judge is trustworthy and its failure modes. For observability, the rule is: use it as a *sample* signal, not as an inline gate; alert on shifts in the sample distribution.

### 5. Offline replay against the eval harness

Sample the full production request into a replay buffer. Nightly, run the buffer through the offline eval harness (mod-305) to produce the same per-slice metrics you use for release qualification. This is the strongest quality signal and the most expensive to run. It catches regressions the online signals miss because it uses the same rubric as the release gate.

Pick two or three from that stack, not all five, until you understand which ones are noisy and which are load-bearing for your feature. "Instrumented all five weakly" is worse than "instrumented two of them well."

## Cost attribution — the invoice-shaped SLI

Cost is a first-class SLI (chapter 04). The observability discipline: **every LLM call is tagged with the feature and prompt version, and dollar cost is a queryable metric per tag.**

- **Client-side computation.** Compute `dollars_per_call` in your client wrapper from the vendor's returned token counts and your pinned per-token prices. Do not depend on end-of-month invoices to know today's cost.
- **Per-feature spend dashboards.** Time series of `sum(dollars)` grouped by feature ID and prompt version. This is the panel that answers "who spent what today" — a question that lands in your inbox weekly.
- **Alerts against the aggregate budget.** From chapter 04: at 50 %, 75 %, 90 % of the monthly cap, not at 100 %. Fire on the burn rate, not just the cumulative — a feature that will breach the cap in six hours is more urgent than one that will breach it in six days.
- **Reconciliation against the vendor.** Weekly (or on a triggered mismatch alert), compare `sum(dollars_per_call)` from your metrics against the vendor's [usage API](https://docs.anthropic.com/en/api/usage-cost-api) / OpenAI's usage endpoints. A drift of more than a few percent means an invocation path is not being instrumented.

Google's SRE Book chapter on the [four golden signals](https://sre.google/sre-book/monitoring-distributed-systems/#xref_monitoring_golden-signals) is the classical framing (latency, traffic, errors, saturation). Add **cost** as a fifth for LLM-augmented systems. That is the entirety of the SLO conversation for cost. Chapter 07 (SLOs) in mod-307 makes cost SLIs a first-class citizen alongside latency and error rate.

## What to alert on

Alerts are how observability escapes the "nobody looks at the dashboard" trap. Rules from the SRE literature apply — [Google's alerting philosophy chapter](https://sre.google/workbook/alerting-on-slos/) is the reference — plus a few LLM-specific ones.

- **Alert on user-visible symptoms.** p99 wall-clock latency over its SLO. End-to-end request error rate. User-facing quality signal (`thumbs_down_rate`, `escalation_rate`) crossing a threshold. These wake someone up.
- **Alert on cost burn rate.** The 50 / 75 / 90 % monthly-cap thresholds from chapter 04. Also alert on `hour-over-hour cost jumped by ≥ 3×` — this catches a runaway loop faster than a monthly cap does.
- **Alert on vendor signal.** 429 rate over a threshold sustained for five minutes. 5xx rate over a threshold. Circuit-breaker in the open state for longer than the cooldown. Vendor status-page incident (via webhook, if the vendor exposes one).
- **Alert on quality leading indicators, not lagging ones only.** Structured-output conformance dropping ≥ 5 % on a stable prompt. Refusal rate more than 3× baseline sustained for ten minutes. LLM-as-judge sample distribution shifted (KS test, or a simpler p50 shift).
- **Do not alert on absolute metrics that will page every day.** "Latency > 800 ms once" or "any 429 in an hour" is noise. Alert on rates, on sustained thresholds, on burn rates. Every page that fires and turns out to be nothing weakens the response to the page that fires and is real.

The SRE Workbook's chapter on [alerting on SLOs with multiple burn rates](https://sre.google/workbook/alerting-on-slos/#6-multiwindow-multi-burn-rate-alerts) is the pattern to adopt for both latency and cost.

## Tooling landscape

You do not build a full observability stack from scratch. The mature options as of authoring:

- **OpenTelemetry** — the vendor-neutral instrumentation standard. The [gen-ai semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) plus a language SDK give you traces, metrics, and logs that any observability backend can consume. This is the load-bearing standard; everything else plugs into it.
- **Metrics + logs backends** you probably already have — Prometheus + Grafana, Datadog, New Relic, CloudWatch, Cloud Monitoring. They all speak OpenTelemetry now.
- **LLM-focused observability tools** — [Langfuse](https://langfuse.com/) (open source), [Helicone](https://www.helicone.ai/), [Arize Phoenix](https://phoenix.arize.com/), [OpenLLMetry](https://www.traceloop.com/openllmetry) — all provide LLM-shaped dashboards (per-prompt-version quality, per-model cost, session-level trace views) that generic APM tools do not surface out of the box. Evaluate the trade-off between "another vendor in the critical path" and "someone else has already built the dashboards I need."
- **The vendor console** — Anthropic's [console](https://console.anthropic.com/) and OpenAI's [platform dashboard](https://platform.openai.com/) are the free tier. Adequate for initial development; insufficient for production incident response because they cannot join to your own request IDs.

The senior read: OpenTelemetry is the standard your instrumentation targets; the storage / dashboarding tool is a choice you can revisit; the LLM-specific tools are a genuine accelerator when the shape of the dashboard matters more than raw flexibility.

## A worked example — the ticket summariser observability wire-up

Same feature as chapters 01, 03, 04: classifier + prompt-only summariser, shown on ticket open.

Instrumentation:

- **Structured logs.** One JSON line per invocation with `request_id`, `feature_id="ticket_summariser"`, `prompt_version`, `model_id="claude-*"`, `model_version` from the vendor response, `input_token_count`, `output_token_count`, `dollars`, `ttft_ms`, `wallclock_ms`, `finish_reason`, `structured_output_conformant` (boolean), `redacted_prompt_hash`, `redacted_output_hash`. Full prompt / output on error and on 1 % sample; PII scrubbed.
- **Metrics.** Histograms on `llm_ttft_ms`, `llm_wallclock_ms`, `llm_input_tokens`, `llm_output_tokens`, `llm_dollars`; counters on `llm_invocations_total`, `llm_429_total`, `llm_5xx_total`, `llm_timeouts_total`, `llm_schema_failures_total`, `llm_refusals_total`, `llm_cache_hits_total`, `circuit_breaker_trips_total`. Labels: `feature_id`, `prompt_version`, `model_id`, `model_version`, `branch` (`primary` / `cheap_fallback` / `classical_only`).
- **Traces.** OpenTelemetry spans per request: outer request span → classifier span → LLM span (with `gen_ai.*` attributes) → post-processing span. Trace ID and span ID logged with each log line for join-back.
- **Dashboards.** Four Grafana panels: (a) latency histogram by branch, (b) cost per day by prompt version, (c) quality signal panel (schema failure rate, refusal rate, `thumbs_down_rate` from the downstream product event), (d) reliability panel (429 / 5xx / timeout / circuit-breaker state).
- **Alerts.** p99 wall-clock over 1.5 s for 10 minutes → page. Schema failure rate above 5 % for 10 minutes → page. Cost burn rate on trajectory to breach monthly cap in < 24 h → page. Circuit breaker open for > 5 minutes → page. `thumbs_down_rate` above 2× seven-day baseline for 30 minutes → ticket (not page — lagging signal, not urgent).
- **Reconciliation.** Weekly job compares `sum(llm_dollars)` per model to Anthropic's usage-and-cost API, alerts on > 3 % drift.

That wire-up is a day of setup once the OpenTelemetry client wrapper exists, and it is the difference between "we noticed the LLM was slow" and "we can attribute the p99 regression to prompt-version 2.3, deployed at 10:47, on the primary branch, on the small-model tier."

## Three failure modes to catch in review

- **"No prompt_version label."** The team's metrics have `feature_id` and `model_id` but not `prompt_version`. Every quality regression debug session starts with "which prompt version was that?" and answers with "let's grep the deploy log." Fix: `prompt_version` is a first-class metric label wired from day one, with an alert on unknown / missing values so an unversioned deploy fires.
- **"Downstream user signal is the only quality signal."** The team monitors `thumbs_down_rate` and nothing else. A regression from a bad prompt change lands in the metric a week later, after 100 % of traffic has run through the new version. Fix: add at least one leading indicator (schema conformance is the cheapest) and one sampled evaluation (LLM-as-judge on 0.1 % of traffic).
- **"Metrics have per-user labels."** Someone added `user_id` as a Prometheus label to help debug a single-user complaint; the metrics backend melts under label cardinality. The whole team's dashboards go dark for two hours. Fix: never per-user labels on metrics; per-user context lives in the log line, joinable by request ID from the trace.

## Summary

LLM observability is not classical observability with an extra label. It has five distinct signal families — invocation metadata, token / cost, latency components, quality, reliability — and each maps to logs, metrics, and traces in its own way. Structured logs are the evidence store; metrics are what the dashboards render and the alerts fire on; OpenTelemetry's gen-ai semantic conventions are the standard everyone is converging on. Every metric carries `prompt_version` and `model_version` labels because vendor cutovers and prompt edits are the two most common invisible failure modes. Cost is a first-class SLI, wired against per-request client-side computation and reconciled weekly against the vendor's usage APIs. Alerts fire on user-visible symptoms, on cost burn rates, on vendor signal, and on leading indicators of quality regression — not on absolute one-shot thresholds that page every day. Chapter 06 is where the delegation contract to a specialist track defines who owns which of these sensors after hand-off.
