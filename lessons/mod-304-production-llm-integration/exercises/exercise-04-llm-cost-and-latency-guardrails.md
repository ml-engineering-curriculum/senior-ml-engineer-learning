# exercise-04: LLM Cost and Latency Guardrails

**Estimated effort:** 3 hours

## Objective

Take one LLM-augmented feature — the one you designed in exercise 03, or one from your own work — and produce the **full five-axis envelope** from chapter 04 (`04-cost-and-latency-guardrails.md`) plus the **observability wire-up** from chapter 05 (`05-observability-for-llm-augmented-systems.md`) that lets you see the envelope slip in production.

Chapter 04 lays out the design commitment; chapter 05 lays out the sensor grid that watches it; this exercise binds the two together into a single deliverable. The output is a one-page guardrails design doc plus a metric-and-alert spec that an on-call engineer could read at 03:00 without needing the author present.

This is the exercise that separates "we shipped the feature" from "we shipped the feature and it is not going to melt the monthly budget or the p99 SLO the first time traffic doubles." The chapter 04 opening line says it: **guardrails are a design step, not an operations aftermath.**

## Prerequisites

- Read chapter 04 (`04-cost-and-latency-guardrails.md`) — the five-axis envelope, the levers, and the degradation path are load-bearing.
- Read chapter 05 (`05-observability-for-llm-augmented-systems.md`) — the five signal families, the `prompt_version` axis, and the alert-design section are load-bearing.
- Skim chapter 03 (`03-hybrid-classical-plus-llm-patterns.md`) — if your feature is a hybrid, the shape affects the envelope shape.
- Have vendor pricing and rate-limit pages open: Anthropic ([pricing](https://www.anthropic.com/pricing), [rate limits](https://docs.anthropic.com/en/api/rate-limits)), OpenAI ([pricing](https://openai.com/api/pricing/), [rate limits](https://platform.openai.com/docs/guides/rate-limits)), Google ([pricing](https://ai.google.dev/pricing), [rate limits](https://ai.google.dev/gemini-api/docs/rate-limits)), or Bedrock ([pricing](https://aws.amazon.com/bedrock/pricing/), [quotas](https://docs.aws.amazon.com/bedrock/latest/userguide/quotas.html)).
- OpenTelemetry's [gen-ai semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) open for reference.

## Pick your feature

Pick **one**:

- The hybrid feature you designed in exercise 03.
- The prompt bundle you promoted in exercise 02.
- A production feature from your own work (anonymise as needed). Especially valuable if it is currently live without a full envelope.
- The chapter 01 worked example: the ticket summariser (classifier + prompt-only summariser, 500 rps peak, p99 ≤ 1.2 s from ticket open).

Whatever you pick, the rest of this exercise assumes:

- You know the expected steady-state RPS and the "we might trend viral" RPS.
- You know the model tier you plan to call (specific version identifier, not a family alias).
- You know the shape of the prompt (rough input token count, rough output token count, whether prompt caching applies).

## Steps

### 1. Sketch the five-axis envelope — chapter 04 (≈ 45 min)

For every one of the five axes in chapter 04, produce a specific number and a one-line justification.

- **Per-request cost budget.** Compute expected `input_tokens × input_price + output_tokens × output_price` at your target model tier. Produce p50 and p99. Vendor pricing page cited by URL. Include the effect of prompt caching if you enabled it. Chapter 04's rough formula is the starting point; refine it for cache hit rate.
- **Per-request latency budget.** p50, p95, p99 in milliseconds, end-to-end from user action to rendered result. Break it down into the components chapter 04 names: network, vendor queue, TTFT, `output_tokens × TPOT`, client-side parsing. Where you cannot measure directly, state your assumption (e.g. "TTFT on `claude-*-*` at the small tier is ~300 ms based on vendor blog benchmarks — [needs verification against your own measurement]").
- **Concurrency ceiling.** The vendor RPM and TPM limits on your account tier, cited from the vendor rate-limit page. What fraction of that ceiling this feature is allowed to consume at peak. What happens when it is exceeded (queue, back off, degrade).
- **Aggregate spend budget.** Dollars per day or per month. Chapter 04: "set the cap at 2×–3× the expected steady-state so you have headroom for growth, not 10× so a leak can run unnoticed." What alert thresholds fire at 50 / 75 / 90 %.
- **Degradation path.** Chapter 04's five degradation options, ranked in the order you would apply them for this feature: cached answer, cheaper model, classical fallback, static template, missing feature. Which of the five is the default for a slow LLM, an errored LLM, a rate-limited LLM, an over-budget LLM. Every combination needs an answer.

If any of the five is "I don't know," chapter 04's rule holds: the feature is not shippable.

### 2. Name the levers you turned on — chapter 04 (≈ 30 min)

Chapter 04 lists levers on the cost line (model tier, prompt design, `max_tokens`, prompt caching, response caching, semantic caching, batch API, distillation / self-hosting) and levers on the latency line (model tier, concise prompts, streaming, timeout, retries, parallelism, fallback path).

For each lever, state:

- **Whether you turned it on.** Yes / no / not applicable.
- **Why.** One sentence. "We enabled prompt caching because 80 % of the prompt is a stable system instruction + few-shot library" is a defence; "we didn't turn it on because we didn't get to it" is a to-do.
- **What it saved (or cost).** Order-of-magnitude estimate. "Prompt caching drops per-call input cost from ~$X to ~$Y — a ~90 % reduction on the cached prefix."

The point is not to turn on every lever. It is to have thought about each one deliberately and to have a paper trail. A future incident review will ask "did you consider Y?" and the answer is either "yes, here's why we didn't" or "no, we should now."

### 3. Author the timeout, retry, and circuit-breaker spec — chapter 04 (≈ 30 min)

Chapter 04 is explicit: **every LLM call must have a hard timeout.** Produce a small spec that answers, in three lines:

- Client-side timeout on the LLM call — the specific millisecond value, chosen tighter than the p99 SLO.
- Retry policy — how many retries, with what backoff, capped at what total wall-clock. Idempotency keys used, so retries do not double-charge.
- Circuit breaker — the error rate / latency threshold that trips it open, the cooldown period before it half-opens, the degradation path (from step 1) it routes to when open. Nygard's *Release It!* pattern applied.

### 4. The observability wire-up — chapter 05 (≈ 45 min)

Produce a metric-and-alert spec grounded in the chapter 05 five-signal-families discipline. Structured as:

**Logs.** One structured JSON line per LLM invocation. Include, at minimum, the fields from chapter 05's worked example: `request_id`, `feature_id`, `prompt_version`, `model_id`, `model_version`, `input_token_count`, `output_token_count`, `dollars`, `ttft_ms`, `wallclock_ms`, `finish_reason`, `structured_output_conformant`. Sampling rule (100 % on error and tail, 1 % on golden path). PII redaction rule.

**Metrics.** Explicit list. Each item is `<metric name> (<type>) — <labels>`. Following chapter 05:

- Histograms: `llm_ttft_ms`, `llm_wallclock_ms`, `llm_input_tokens`, `llm_output_tokens`, `llm_dollars`.
- Counters: `llm_invocations_total`, `llm_429_total`, `llm_5xx_total`, `llm_timeouts_total`, `llm_schema_failures_total`, `llm_refusals_total`, `llm_cache_hits_total`, `circuit_breaker_trips_total`.
- Labels — always `feature_id`, `prompt_version`, `model_id`, `model_version`, `branch`. Never `user_id` or any high-cardinality free-form field. Chapter 05's failure-mode-3: "someone added `user_id` as a Prometheus label; the metrics backend melts."

**Traces.** OpenTelemetry spans, using [gen-ai semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/). At minimum: `gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.response.finish_reasons`. Trace ID logged with each log line for join-back.

**Alerts.** Explicit list, at least six of the following (following chapter 05's alert-design section — user-visible symptoms, cost burn rate, vendor signal, quality leading indicators):

- `p99 llm_wallclock_ms > <SLO>` for <window> → page.
- `schema_failure_rate > <threshold>` for <window> → page.
- `cost_burn_rate` on trajectory to breach monthly cap in < 24 h → page.
- `circuit_breaker_state == open` for > <cooldown> → page.
- `429_rate > <threshold>` sustained for <window> → page.
- `refusal_rate > 3× baseline` sustained for <window> → ticket (or page, if user-visible).
- `thumbs_down_rate > 2× baseline` for <window> → ticket. (Lagging signal, not urgent.)
- Weekly reconciliation job: `abs(sum(llm_dollars) - vendor_usage_api) / vendor_usage_api > 3 %` → ticket.

For each alert, name: the threshold, the window, whether it pages or files a ticket, and (very short) the runbook step. Chapter 05: "every page that fires and turns out to be nothing weakens the response to the page that fires and is real." Don't over-page.

### 5. The one-page guardrails design doc (≈ 30 min)

Roll the pieces above into a single `guardrails.md` file. Layout, no more than a page:

- Feature name, author, date, target launch date, RPS assumptions.
- **Five-axis envelope table** — one row per axis, one column per metric (p50, p99, cap, alert thresholds).
- **Levers turned on** — bulleted list, one line each with the "why."
- **Timeout / retry / circuit breaker** — three lines from step 3.
- **Observability wire-up** — pointer to the metric-and-alert spec (which can live in a separate file).
- **Degradation path** — chapter 04's five options ranked for this feature.
- **Runbook link** — pointer to (or stub of) the runbook the alert points to.

The reviewer's check is: could an on-call engineer at 03:00 read this and know (a) what the envelope was, (b) what alert just fired, (c) what to degrade to.

## Deliverable

Two files:

- `guardrails.md` — the one-page design doc.
- `observability.md` (or `.yaml` / `.json`) — the metric-and-alert spec from step 4.

Optional: a diff / sketch of the code changes needed to instrument the metrics on the serving path (client wrapper, prompt loader, metric emissions). Pseudocode is fine; the point is that the instrumentation is *specific*, not aspirational.

## Acceptance criteria

- [ ] All five axes of the envelope have a specific number, not "TBD," and each cites the vendor page or measurement it came from.
- [ ] Per-request cost is computed with the current pricing from the vendor page (URL cited) at the tier you are targeting, and includes prompt caching's effect if applicable.
- [ ] The concurrency ceiling is stated against the vendor's *actual* RPM / TPM limits from the rate-limit page, not a guess.
- [ ] Aggregate spend cap is a real number (dollars per month), with alert thresholds at 50 / 75 / 90 %.
- [ ] Degradation path is explicit for each failure mode (slow, errored, rate-limited, over-budget) and ranked from chapter 04's five options.
- [ ] Every applicable chapter 04 lever is addressed — turned on with a "why," or turned off with a defended reason.
- [ ] The LLM call has a hard timeout with a specific value; retry and circuit-breaker behaviour is specified.
- [ ] The metric list follows chapter 05: histograms for distributions, counters for count-like events, always-and-only low-cardinality labels including `prompt_version` and `model_version`.
- [ ] OpenTelemetry gen-ai semantic-convention attribute names are used, not vendor-specific attributes.
- [ ] At least six alerts specified, with threshold, window, page-vs-ticket, and runbook step; no alerts that would fire on a single one-shot event.
- [ ] The `guardrails.md` fits on one page and covers all bullet points from step 5.

## Stretch goals

- **Load-test the envelope.** Fire synthetic traffic at your feature under realistic input distributions at 1× and 10× your steady-state RPS. Record the observed p50 / p95 / p99 latency and cost per request; compare against the envelope. Where the observed distribution violates the envelope, fix the envelope (it was optimistic) or fix the design (it was under-provisioned). Watch out for coordinated-omission bias in your load generator — Gil Tene's [*How NOT to Measure Latency*](https://www.infoq.com/presentations/latency-response-time/) is the reference.
- **Wire the reconciliation job.** Set up the weekly job that compares your client-side `sum(llm_dollars)` against the vendor's usage-and-cost API (Anthropic's [usage and cost API](https://docs.anthropic.com/en/api/usage-cost-api), OpenAI's [usage endpoints](https://platform.openai.com/docs/api-reference/usage)) and alerts on > 3 % drift. Chapter 05: "a drift between the two is a sign of a missing invocation path in your instrumentation."
- **Author the runbook.** Turn each of the alerts into a runbook page with the specific investigation steps and the specific kill-switch flag names. The senior read is that an alert without a runbook is a page that will not be acted on well.
- **Feed into paired project and downstream module.** The guardrails doc is a direct fit for `project-302-llm-augmented-ml-feature`. The observability spec feeds into mod-307's SLO chapter — cost, latency, and quality SLIs from this exercise are the ones that get formal SLOs and error budgets in mod-307.
