# Hybrid classical + LLM patterns

## Motivation

The pattern decision in chapter 01 is not always a single answer. In production the honest read is that classical ML and LLMs are complements more often than substitutes:

- Classical models are cheap, low-latency, calibrated, evaluable at scale, and completely unable to write a paragraph of text.
- LLMs are expensive, high-latency, uncalibrated by default, and remarkable at unstructured input / output.

An engineer who reaches for the LLM to solve every problem builds a system that costs 100× what it should and is 100× harder to debug. An engineer who refuses to touch an LLM ships a feature that plateaus at the ceiling classical ML can reach — which for many product problems is well below what the LLM cheaply gets to.

The L30 differentiator is composition. The five hybrid shapes in this chapter are the compositions that show up over and over in production ML systems that ship LLM-augmented features and stay shipped. Each one has a clear "when," a clear "how," a clear "what breaks," and a clear evaluation shape.

## Framing: where is the LLM in the request lifecycle?

Before the shapes, one framing that saves a lot of arguments: for any LLM-augmented feature, ask **when in the request lifecycle does the LLM run?**

- **On the request path, always** — every user request calls the LLM. Highest cost, highest latency, most quality upside.
- **On the request path, sometimes** — a router or fallback decides whether to call the LLM. Cost / latency are amortised.
- **Off the request path** — the LLM runs in a batch job, an offline enrichment pipeline, or an async trigger. The user request reads pre-computed results and pays near-zero LLM latency.

The hybrid shapes below map to these positions. If you cannot ship the LLM on the request path (latency, cost, reliability), the shapes that push it off-path are almost always available.

## Shape 1: Router (cheap first, expensive on the tail)

**Setup.** Every request is scored by a cheap, fast classifier that decides whether the request is "easy" or "hard." Easy requests are handled by a small, fast, cheap component (a classifier, a template, a tiny LLM). Hard requests are routed to the expensive LLM.

**Bet.** The distribution is fat-headed — a small fraction of requests need the expensive treatment; the rest are cheap and stereotyped. Route them differently and the mean cost / latency drops sharply.

**Where it shows up.** Customer-service auto-reply (most tickets are refund-status queries handled by a template; complex ones go to a frontier LLM). Content-moderation triage (most posts are trivially benign or trivially violating; the ambiguous 5 % gets a slow, expensive analysis). Code assistants (single-token completions do not need the same model as a multi-file refactor).

**The router itself.** A classical classifier or a cheap LLM. Feature it on properties like input length, presence of specific tokens, difficulty scores from a lightweight model. Route calibration matters — the router's job is not to be perfect, it is to keep the false-negative rate on the "route to expensive" branch below a per-slice threshold (missing a hard case is worse than paying LLM cost on an easy one). Chip Huyen's [routing chapter in *AI Engineering*](https://huyenchip.com/2024/07/25/genai-platform.html) walks the pattern in production terms.

**What breaks.**

- **Router drift.** The "easy" distribution shifts (new product launches, seasonal traffic) and the router silently under-routes to the LLM. Alert on the LLM-route rate as a first-class SLI (chapter 05).
- **Latency variance.** Users on the LLM branch see 5× the p50 latency of users on the cheap branch. UI has to be built assuming variable latency, not a fixed budget.
- **Two-baseline evaluation.** You now need to evaluate two pipelines and their composition — the router's routing decision, the cheap-branch quality, the LLM-branch quality, and the end-to-end blend. Chapter 05 is where this lands.

**Evaluation shape.** Per-branch and end-to-end. The router quality is measured as "false-negative rate on the hard slice" and "false-positive rate on the easy slice." End-to-end quality is measured on the same eval set the LLM-only baseline would use.

## Shape 2: LLM as feature extractor (LLM → structured features → classical model)

**Setup.** The LLM is called on unstructured input (a ticket body, a product listing, a review) and asked to produce **structured features** — a category, a sentiment score, a set of extracted entities, an embedding, a JSON object with typed fields. Those features are fed into a classical model (a ranker, a classifier, a regression) that owns the final decision.

**Bet.** The LLM is much better than any hand-authored feature pipeline at turning unstructured input into structured features, but the *decision* the feature is used for is a classical modeling problem — the classical model is calibrated, evaluable, cheap, and does not hallucinate class labels outside its taxonomy.

**Where it shows up.** Fraud detection where an LLM extracts risk signals from a merchant description; product ranking where an LLM extracts intent and category signals from a search query; lead scoring where an LLM extracts firmographic fields from a company website; observability where an LLM converts logs into structured incident signals.

**Two versions.**

- **On the request path.** Feasible when the LLM call fits inside the latency budget and the extraction is per-request (query intent classification for a search system). Constrain the LLM output to a schema (chapter 02) and cap tokens aggressively.
- **Off the request path.** Feasible whenever the input is durable — a product listing, a user profile bio, a merchant description. LLM runs in a batch pipeline; the extracted structured features land in the feature store like any other feature; the classical model reads them at request time. Latency and cost of the LLM disappear from the request path.

The off-path version is by far the more common production shape. It is the LLM equivalent of a feature-engineering pipeline.

**What breaks.**

- **Schema drift.** The LLM's extracted schema changes because someone edited the prompt without updating the downstream feature-store schema. Chapter 02's structured-output discipline handles this; a feature-store contract makes it enforceable.
- **Silent quality regression.** The LLM's extraction gets subtly worse (base-model bump, prompt edit) and the classical model's metric moves a fraction of a point on the eval set — enough to notice, not enough to trace. Fingerprint the extractor version alongside the classical model version so an incident review can attribute the regression.
- **Cost when the input is huge.** A million-row batch job with a slow LLM extractor is a real bill. Use batch APIs (OpenAI [batch](https://platform.openai.com/docs/guides/batch), Anthropic [message batches](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing)), cache aggressively, and only re-extract when the input changes.

**Evaluation shape.** Two-layer. The extractor is evaluated on structured-output quality (schema conformance, field-level accuracy). The classical model is evaluated on its own end task. End-to-end is evaluated on the product metric.

## Shape 3: Classical decides, LLM presents (decision + explanation split)

**Setup.** A classical model owns the decision — the classification, the ranking, the score, the recommendation. An LLM generates the human-facing artefact that surrounds the decision — the summary, the explanation, the draft reply, the user-facing rationale.

**Bet.** The decision must be calibrated, auditable, and evaluated on a decision metric (accuracy, recall, revenue lift). Human-readable presentation is a separate axis where LLMs are uniquely good. Coupling them lets a "boring" classical decision system produce a "delightful" user experience without polluting the decision path with generation risk.

**Where it shows up.** Fraud reviews where a classifier picks the decision and an LLM writes the "why" panel for the analyst. Product ranking where a ranker orders items and an LLM writes the recommendation card copy. Alerting where an anomaly detector fires and an LLM drafts the on-call runbook step. Medical triage where a classifier assigns urgency and an LLM writes the summary the clinician sees first.

**The composition contract.** The LLM sees the classical model's output and the input; it must not override the decision, and its generated text must be consistent with the decision (do not write "your account is fine" if the classifier flagged fraud). This is enforceable with a few tactics:

- The LLM's prompt receives the classifier's decision and score as structured input and is instructed to explain *that* decision, not evaluate the input independently.
- The LLM has no authority to change the decision. If the LLM's output disagrees with the classifier's decision (a topic classifier says "shipping" but the LLM writes a return-policy answer), the composition gate rejects the LLM output and falls back to a default template.
- The generation is treated as its own evaluated artefact (chapter 02). Groundedness against the decision is the load-bearing metric.

**What breaks.**

- **The LLM contradicts the decision.** Ensure the composition gate catches this. A regression where 2 % of generated summaries contradict the classifier is a real product-quality issue.
- **Explanation without grounding.** The LLM writes a plausible explanation that is not actually the reason the classifier decided. This is not fixable purely at the LLM layer; the classifier must expose *why* (feature attributions, top rules, top nearest examples) so the LLM has real material to work with. See [mod-303] on interpretability tools and Ribeiro et al.'s [LIME paper](https://arxiv.org/abs/1602.04938) and Lundberg / Lee's [SHAP paper](https://arxiv.org/abs/1705.07874).
- **Latency budget on the LLM half.** The classifier is 5 ms; the LLM is 800 ms. If the UI blocks on the LLM, the decision looks slow. Stream the LLM output; render the decision first, then progressively fill the explanation.

**Evaluation shape.** Decision metrics on the classical model (calibration, per-slice pass rate, product metric). Presentation metrics on the LLM (groundedness against the decision, factual accuracy against the input, tone / brand consistency). No aggregate "was the whole answer good" metric — the composition is evaluated as two things.

## Shape 4: LLM as verifier or fallback

**Setup.** The classical model handles the golden path. The LLM handles the *tail* — inputs the classical model refuses to score (out-of-distribution), scores with low confidence, or scores as "needs review." Alternatively, the LLM *verifies* the classical model's output on the ambiguous slice and can escalate to a human.

**Bet.** The head of the distribution is stereotyped and classical models handle it correctly at low cost. The tail is where classical models silently fail — new items, weird inputs, adversarial inputs, edge cases the training set never covered. LLMs are much better at tail behaviour, in part because their pre-training saw far more of it.

**Where it shows up.** Fraud where the classifier scores in the "grey zone" (0.4–0.7) and an LLM does a second-look with the full transaction context. Content-moderation where the classifier says "unsure" and an LLM adjudicates. Extraction where a rule-based extractor covers 90 % of forms and the LLM handles unknown layouts. Search where the classical ranker fails to return anything above a relevance threshold and an LLM does the search from the retrieved documents.

**What breaks.**

- **Uncalibrated confidence on the classical side.** The "grey zone" only makes sense if the classifier is calibrated ([mod-303] chapter 05). If the score at 0.7 is not really 70 % likelihood, the fallback rate is wrong.
- **Silent tail growth.** The fallback rate is a first-class SLI. When it climbs — new product launch, distribution shift — the LLM budget explodes. Alert on the fallback rate and put a hard cap on it; when the cap trips, degrade to a default rather than exhausting the LLM budget.
- **The LLM cannot verify what the classifier saw.** If the classifier's decision used features the LLM cannot see (embeddings, session history, private signals), the LLM's verification is only as good as the input it can inspect. Design the fallback to give the LLM the same information the classifier had (or a compact summary), or the "verifier" is guessing.

**Evaluation shape.** Golden-path quality on the classical branch, tail-path quality on the LLM branch, plus the composition metric — does the fallback catch the errors the classifier makes? Measured as false-negative-rate reduction on the eval set at a fixed LLM budget.

## Shape 5: Offline enrichment (batch LLM → feature store / cache)

**Setup.** The LLM never runs on a user request. It runs in a batch or streaming pipeline over durable data — the product catalogue, the user profile, the knowledge base, the review corpus. Its outputs land in the same storage layers the rest of the ML system already uses (feature store, embedding store, key-value cache). The request-path serving code reads the pre-computed results at request-time.

**Bet.** For many features the LLM's slow, expensive call does not need to happen when the user is waiting. It can happen once, cached against the input's key, and served with classical latency thereafter.

**Where it shows up.** Product-catalogue enrichment (title normalisation, category prediction, description generation, image alt-text). Knowledge-base pre-processing (chunking, summarisation, structured-field extraction) — this is where RAG pipelines pre-compute. Review summarisation for product pages. Automated tagging of a media library.

**Implementation shape.**

- **Trigger.** Event-driven (new item arrives → enrich) or scheduled (nightly re-enrich for anything whose input changed).
- **Idempotency and caching.** The enrichment is keyed on the input hash + the extractor version. If neither changed, do not re-call the LLM.
- **Batch pricing.** Vendor batch APIs (50 %-off or similar) apply here directly. Backfill jobs go through the batch API by default.
- **Feature-store integration.** The enriched features land in the same feature store the classical model already reads from. The request path is unchanged.

**What breaks.**

- **Staleness.** The LLM enriched the product catalogue three months ago; the product has changed since. Solve at the trigger level: re-enrich when the source input changes, on a bounded schedule for anything that has not been touched.
- **Extractor version drift.** The prompt used for enrichment gets bumped and now half the catalogue is on `v3` and half on `v4`. Track the extractor version alongside the enriched value; backfill deliberately, not silently.
- **Cost of the backfill itself.** A million-row LLM backfill at $0.001 per call is $1 000; at $0.03 per call it is $30 000. Budget explicitly before running.

**Evaluation shape.** The extractor is evaluated independently on the same shape as shape 2. The downstream classical model or serving path is evaluated as usual. Because the LLM is off the request path, latency and cost dimensions collapse — the only remaining cost dimension is the batch-pipeline dollar bill, budgeted per-run.

## Composing the shapes

The shapes above are not mutually exclusive. A production feature can — and often does — use several at once:

- The product-detail page pre-computes review summaries offline (shape 5), scores every request through a classical ranker (base pattern), sends explanation text to the frontier LLM only for the flagged low-confidence slice (shape 4), and uses an LLM-extracted structured category feature from the batch pipeline (shape 2). All in the same feature.

Reading a design proposal at senior altitude is often the exercise of naming which shape is at each layer and pushing back on the ones that are picking the wrong shape.

## The chapter-01 rubric, revisited for hybrids

Chapter 01 Q5 ("would classical + LLM composition beat either alone on cost, latency, or accuracy?") is where hybrids enter. The concrete follow-up questions in a design review are:

- **Which position in the request lifecycle?** If cost or latency is the constraint, push toward off-path (shape 5) or amortised (shape 1, shape 4). If quality on every request is the constraint, push toward on-path with a router (shape 1).
- **Who owns the decision?** If the decision is a classification / ranking / score whose calibration matters, the classical model should own it (shape 2, shape 3, shape 4). If the decision is generation of an artefact, the LLM owns it (base prompt-only or RAG pattern, chapter 01).
- **What is the tail?** Almost every classical system has a tail its training set does not cover well. Shape 4 (LLM as fallback) is the pattern for that tail. Ignoring the tail is the alternative and it is worse.
- **What is durable, what is per-request?** Durable inputs (catalogue, KB, profiles) belong in shape 5. Per-request inputs (queries, tickets, messages) belong in shapes 1–4.

## Three failure modes to catch in review

- **"LLM everywhere."** The design puts the LLM on the request path for every request even though 95 % of requests are stereotyped. Cost / latency will bite in weeks. Ask for a router (shape 1) or an offline enrichment (shape 5).
- **"LLM owns the decision the classical model should own."** The design uses the LLM to classify into a fixed taxonomy of ten categories. The LLM occasionally hallucinates a class name, cannot be calibrated, and is 100× slower and more expensive than the classifier. Ask for shape 2 or shape 3 — LLM as feature or presentation, classical model as decision.
- **"No plan for the tail."** The design assumes the classical model is enough. Six weeks after launch the fallback slice grows (new product, new region, distribution shift). Ask what shape 4 looks like at launch, before you need it.

## Summary

The five hybrid shapes — router, LLM as feature extractor, classical decides + LLM presents, LLM verifier / fallback, offline enrichment — are the compositions where senior ML engineers most clearly out-perform L20s who reach for pure prompt-only or pure classical. Position the LLM in the request lifecycle deliberately: on the path always, on the path sometimes (router / fallback), or off the path (batch enrichment). Let classical models own decisions whose calibration matters; let LLMs own generation and tail behaviour. Every shape carries its own evaluation shape and its own failure modes; chapter 05 is where the observability that catches those failures lives.
