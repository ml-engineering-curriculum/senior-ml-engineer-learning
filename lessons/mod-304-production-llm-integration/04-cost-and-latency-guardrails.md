# Cost and latency guardrails

## Motivation

Classical ML systems have a cost profile that mostly looks like a compute bill and a serving-infrastructure bill. Both are relatively easy to reason about — you know your peak RPS, your feature-fetch p99, your model's inference cost. LLM-augmented systems look different in three uncomfortable ways:

- **Cost is variable per request.** Token counts vary; a bad prompt or a long tool loop can be 100× the "typical" call.
- **Latency is variable per request.** The same prompt on the same model can return in 400 ms or 12 s depending on load, output length, and back-end.
- **The cost is a dollar-per-request line-item on someone else's invoice**, not amortised infrastructure. A launch that goes 10× on RPS is 10× on the invoice tomorrow morning.

A team that ships an LLM feature without cost and latency guardrails will inevitably learn about them from a Slack thread that starts with "hey I noticed the LLM bill was $X yesterday" or "customer support says the assistant is timing out." The senior read is: **guardrails are a design step, not an operations aftermath.** Every LLM-augmented feature carries a designed cost envelope, a designed latency envelope, and a designed degradation path when either envelope is stressed. This chapter is how to build them.

Chapter 05 handles the observability that lets you *see* the envelope. This chapter is the envelope itself.

## The five axes of the envelope

Sketch these five before you ship. If a number in any column is "I don't know," the feature is not shippable.

- **Per-request cost budget** — expected input tokens × input price + expected output tokens × output price, in dollars per request; a p50 and a p99.
- **Per-request latency budget** — p50, p95, p99 in ms, end-to-end from the user's action to the user's rendered result.
- **Concurrency ceiling** — the maximum LLM-provider RPS your account allows, and the fraction of that ceiling this feature is allowed to consume at peak traffic.
- **Aggregate spend budget** — dollars per day or per month allotted to this feature.
- **Degradation path** — what the user sees when any of the above budgets is exceeded.

You cannot ship an LLM feature that leaves any of these blank. Product may want you to pretend, but the invoice will not.

## Cost: the anatomy of the token bill

Every hosted LLM's price is roughly:

`cost per request ≈ input_tokens × input_price + output_tokens × output_price`

The two prices are asymmetric (output is typically 3–5× more expensive per token than input) and the multipliers can shift with model tier, cached-prefix pricing, and batch pricing. Consult vendor pricing pages directly:

- Anthropic Claude pricing — <https://www.anthropic.com/pricing>
- OpenAI API pricing — <https://openai.com/api/pricing/>
- Google Gemini API pricing — <https://ai.google.dev/pricing>
- AWS Bedrock pricing — <https://aws.amazon.com/bedrock/pricing/>

The point-in-time numbers on those pages change often. The *structure* — input vs output pricing, cached input pricing, batch pricing, model tier differences — is stable.

### Levers you have on the cost line

- **Model tier.** Bigger models cost more per token. Smaller models cost less. Pick the smallest tier that clears the eval gate on the pass-rate threshold. "Use the frontier model everywhere" is a mistake if a smaller tier passes eval on 90 % of the traffic; use a router (chapter 03 shape 1) for the rest.
- **Prompt design for input tokens.** Concise system instructions, tight few-shot examples, avoiding verbatim inclusion of large corpora. Every token repeated in the prompt is billed per request.
- **Structured / capped output.** Set `max_tokens` conservatively — a value that fits your worst legitimate output plus a small buffer, not "large so it never truncates." Uncapped output is where surprise bills come from. Combine with structured output (JSON schema, tool use) to keep the model from writing narrative preamble.
- **Prompt caching.** For prompts with a large stable prefix (system instructions, tool specs, long few-shot library) and a small variable suffix (the user input), vendor prompt caching drops the marginal cost of the stable prefix substantially. See Anthropic's [prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching), OpenAI's [prompt caching](https://platform.openai.com/docs/guides/prompt-caching), and Gemini's [context caching](https://ai.google.dev/gemini-api/docs/caching). Layout the prompt with static bits first, dynamic bits last.
- **Response caching.** For prompts whose input is likely to repeat (offline enrichment, common queries), a plain key-value cache with `(prompt_fingerprint, input_hash) → output` avoids the LLM call entirely on a hit. Combine with idempotency keys.
- **Semantic caching.** For inputs that are near-duplicates (different wordings of the same query), an embedding-based cache retrieves the previous response when a new query is close enough. Higher precision cost (false-positive cache hits are user-visible), but real for chat-style features. GPTCache is a common OSS implementation — <https://github.com/zilliztech/GPTCache>.
- **Batch API.** For anything that does not need real-time response, vendor batch APIs (OpenAI [batch](https://platform.openai.com/docs/guides/batch), Anthropic [message batches](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing)) reduce cost by 50 %-ish. Offline enrichment (chapter 03 shape 5) is where this pays off the most.
- **Distillation and self-hosting** where economics justify it. If your workload has enough steady volume that the fixed cost of a self-hosted small model beats the per-token cost of a hosted frontier one, and if a smaller model can be tuned to your task with quality that clears the eval gate, this is a real lever. Escalate to `fine-tuning-engineer-learning` and `training-pipeline-engineer-learning` (chapter 06). Do not attempt this to save $200/month.

### The per-request cost estimate — sketch it before you ship

Rough formula for a feature that calls one prompt per request:

```
per_request_cost = expected_input_tokens * input_$_per_1M_tok / 1e6
                 + expected_output_tokens * output_$_per_1M_tok / 1e6
                 * (1 - cache_hit_rate * cache_savings_fraction)
```

Multiply by `expected_RPS * seconds_per_month`. That gives an order-of-magnitude monthly bill. Sanity check both directions: what does this look like at 2× traffic? At 10× traffic? At 100× traffic (viral scenario)? "The cost scales linearly and I am comfortable with the trajectory" is a real design answer. "I have not thought about the 10× case" is where the incident starts.

## Latency: the anatomy of the p99

An LLM call's latency has more components than a classical model's:

- **Network to vendor.** Regionally-hosted vs cross-region matters; use vendor endpoints in the region your service runs in.
- **Vendor queue time.** Under vendor load, requests can queue behind other tenants' requests; batch APIs deliberately push into this queue for cost savings.
- **Time to first token (TTFT).** How long until the model starts producing output — dominated by the prompt-processing pass. Long inputs make TTFT worse.
- **Time per output token (TPOT).** Dominated by the model tier and output length.
- **Client-side parsing / post-processing.** JSON parsing, schema validation, downstream calls.

Total latency ≈ TTFT + `output_tokens × TPOT + parsing`. Two implications:

- **Long outputs are latency killers.** Every extra output token adds ms. Structure outputs to be as short as they can be while still satisfying the schema.
- **Streaming buys perceived latency, not real latency.** If the user-facing artefact can start rendering as tokens arrive (a chat response, a streaming JSON parse), the *perceived* p50 drops to TTFT even though the total time is unchanged. If the user cannot see anything until the full response is parsed (a decision, a structured object), streaming does not help perceived latency at all. Wire it in the cases where it does; do not wire it where it does not.

### Levers you have on the latency line

- **Model tier.** Smaller models are usually faster. The same routing decision that saves cost saves latency.
- **Concise prompts.** Prompt tokens are billed on latency too — long prompts increase TTFT. This is a design lever for shape 5 (offline enrichment): moving the heavy prompt off the request path replaces the LLM latency with a cache read.
- **Streaming.** Real when the artefact can render progressively; not real when it cannot. Wire it accordingly.
- **Timeout and deadline propagation.** Every LLM call must have a **hard timeout**, set to a value tighter than the p99 you can tolerate. Callers that block indefinitely on a slow LLM call are how outages start. The timeout should propagate along with the request deadline — if the user request already has 400 ms left, the LLM call cannot ask for a 2 s timeout.
- **Retries with capped delay and idempotency.** Retries on transient failures (429, 500) are worthwhile, but a naive exponential backoff can exceed the user's budget. Cap the number of retries; cap total time; use idempotency keys so retries do not double-charge.
- **Parallelism.** If the feature can call several LLMs concurrently (a router calls two candidate models in parallel and picks the faster response, or a RAG pipeline parallelises retrieval and generation), the effective latency is dominated by the slow call, not the sum. This is a shape-3 lever more than an efficiency trick.
- **Fallback path.** If the LLM does not respond in time, the request must still return something. See "the degradation path" below.

### The p99, the coordinated omission trap, and the per-slice read

LLM latencies are heavy-tailed. The p99 is often 5–10× the p50. Report the full distribution, not just the mean; alert on p95 and p99 rather than the mean. If the load-testing setup lets a slow request block the harness from starting the next request, the observed p99 is under-estimated by a wide margin — Gil Tene's [*How NOT to Measure Latency*](https://www.infoq.com/presentations/latency-response-time/) is the reference on coordinated-omission bias.

Split by slice: latency on cache-hit vs cache-miss, on router-easy vs router-hard, on short vs long inputs. A p99 that looks acceptable in aggregate can be 10× worse on the "hard" slice, and that slice may be exactly the one the LLM is there to catch (chapter 03 shape 4).

## Concurrency and rate limits

Every hosted LLM has account-level rate limits, expressed in requests-per-minute (RPM) and tokens-per-minute (TPM). See Anthropic's [rate limits documentation](https://docs.anthropic.com/en/api/rate-limits), OpenAI's [rate limits](https://platform.openai.com/docs/guides/rate-limits), Google's [Gemini rate limits](https://ai.google.dev/gemini-api/docs/rate-limits), and Bedrock's [quotas](https://docs.aws.amazon.com/bedrock/latest/userguide/quotas.html) for the current shape.

Implications:

- **Your rate-limit budget is shared.** Every feature on the same account competes. A rogue backfill can crowd out a request-path feature.
- **You will hit 429s.** Under peak load, retries alone will not solve it; you need a **client-side concurrency limiter** that caps the number of in-flight requests below the vendor's ceiling.
- **Increases take time.** Vendor limit increases are not instant — usage-history-dependent, sometimes hours, sometimes days. Plan them ahead of launches, not the morning of.
- **Failover.** For high-availability features, plan a fallback provider or a fallback model tier when the primary is rate-limited. This is a shape-1 / shape-4 pattern applied at the reliability layer rather than the quality layer.

## The aggregate spend budget

Per-request cost and RPS together give expected daily spend. Turn that into a hard budget:

- **A monthly spend cap per feature.** Set the cap at 2×–3× the expected steady-state so you have headroom for growth, not 10× so a leak can run unnoticed.
- **Alerts at 50 %, 75 %, 90 %.** Not "at 100 %."
- **A kill switch.** A feature flag that degrades the feature to the fallback path when the cap is approached. See below.
- **Attribution.** Every LLM call carries a tag identifying the feature and the prompt version. When the bill is unexpectedly large, you can attribute it to the responsible feature within minutes, not days.

## The degradation path

Every LLM-augmented feature has a designed answer to: "the LLM is slow / errored / rate-limited / over-budget. What does the user see?" The answers are almost always in this order:

1. **Cached answer.** If the input is a cache hit, serve the cached answer even if the freshness is imperfect.
2. **Cheaper model.** Route to a smaller / cheaper model; degrade gracefully on quality.
3. **Classical fallback.** For hybrids (chapter 03), degrade to the classical component alone, without the LLM's contribution.
4. **Static template.** A hand-authored default (`"We're preparing your summary — please refresh in a moment"`) that at least lets the rest of the UI render.
5. **Missing feature.** Hide the LLM-augmented widget entirely and let the base experience render.

Options 4 and 5 are last resorts and should be visibly signalled ("estimated" or "generated" badges). Choosing which of 1–5 is correct is a product decision the ML engineer surfaces, not decides alone.

The **circuit breaker** is the runtime shape of the degradation path. When the LLM call's error rate or latency exceeds a threshold, the circuit trips open and all requests short-circuit to the degradation path for a cooldown period. Nygard's *Release It!* is the reference on this pattern in production — the [circuit-breaker chapter](https://pragprog.com/titles/mnee2/release-it-second-edition/) is worth reading in full for anyone shipping a hosted-vendor dependency.

## A worked example — the ticket summariser guardrails

Feature from chapter 01: classifier + prompt-only summariser, shown on ticket open.

Envelope sketch:

- **Per-request cost.** Input tokens ≈ 800 (ticket + last-ten-history + system prompt); output ≈ 120 (a short summary). At a small-model tier's prices, ~$0.0004 per request. p99 ≈ ~$0.001 if the history stuffs longer. Sanity check: 500 rps × ~$0.0004 × 3600 × 24 ≈ $17 000/day at steady peak. If product accepts that, ship; if not, negotiate a cheaper model tier or offline enrichment (shape 5) for common tickets.
- **Per-request latency.** Product wants the summary visible within 1.2 s of ticket open. Budget: 200 ms for feature-fetch + classifier, 800 ms for LLM (streamed), 200 ms for parsing + render. Timeout on the LLM call is 1.0 s. Streaming is wired so the summary begins rendering at TTFT ≈ 300 ms.
- **Concurrency.** Vendor tier allows 10 k RPM; peak feature RPS is 500 = 30 k RPM, so we already need three account tiers or a lower model tier. Fix before launch.
- **Aggregate budget.** $500 k/month cap for the feature. Alerts at 50/75/90 %. Kill switch: on breach, degrade the LLM branch to "no summary — click to generate on demand."
- **Degradation.** LLM slow → serve the classifier decision alone with no summary (option 3). LLM errored → cached summary if available (option 1), else classifier only (option 3). LLM budget exhausted → static template (option 4). LLM upstream outage → classical-only mode (option 3), with a status banner on the internal tool.

None of these decisions live in a runbook after launch. They live in the design doc before launch, with the numbers filled in.

## Instrumenting the guardrails (preview of chapter 05)

The guardrails are only real if you can *see* them slip. Chapter 05 covers observability in depth; for guardrails specifically, the load-bearing signals are:

- Per-feature and per-prompt-version input tokens, output tokens, dollar cost, TTFT, TPOT, wall-clock latency — as time series with p50 / p95 / p99.
- Cache hit rate on the prompt cache and the response cache.
- Timeout rate, error rate, 429 rate, circuit-breaker trips.
- Router branch rates, fallback-branch rates.
- Aggregate spend against budget, per-day.

Wire these on day one. Retrofitting them after an incident is much harder than adding them before launch.

## Three failure modes to catch in review

- **"No timeout on the LLM call."** The code awaits the LLM indefinitely. When the vendor is slow, the entire request pipeline stalls; when the vendor is unresponsive, the ingress backpressure cascades. Ask for the hard timeout, the deadline propagation, and the circuit breaker.
- **"max_tokens is set to a large number so we never truncate."** The output is uncapped in practice; a prompt that goes off the rails writes 4 000 tokens and costs 40× the expected. Ask for the concrete max_tokens number and the reasoning.
- **"No degradation path."** Asked "what does the user see when the LLM is down?" the team has no rehearsed answer. This is the shape of an incident-in-waiting. Force the degradation path into the design doc and verify it works in staging.

## Summary

LLM cost and latency are variable, expensive, and paid to a third party. Design an explicit envelope: per-request cost and latency, concurrency ceiling, aggregate spend, degradation path — before you ship. Pull the levers deliberately: right model tier, concise prompts, capped output, prompt caching, response caching, batch APIs where async is possible. Set hard timeouts and circuit breakers on every LLM call; propagate deadlines; use idempotency keys on retries. When any envelope is stressed, the degradation path — cached answer, cheaper model, classical fallback, static template, hidden feature — is not an afterthought; it is a designed part of the feature. Chapter 05 is where you wire the observability that lets you *see* the envelope slip.
