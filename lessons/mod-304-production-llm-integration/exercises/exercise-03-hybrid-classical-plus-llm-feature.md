# exercise-03: Hybrid Classical + LLM Feature

**Estimated effort:** 4 hours

## Objective

Design a single hybrid classical + LLM feature end-to-end and produce a design memo a peer reviewer could sign off on. The memo names one (or more) of the five hybrid shapes from chapter 03 (`03-hybrid-classical-plus-llm-patterns.md`), pins the LLM to a specific position in the request lifecycle, specifies the composition contract between the classical and LLM components, and lays out the two-layer evaluation shape.

This is the exercise where the L30 differentiator lives most visibly. An L20 will draw a box labelled "LLM" and route every request through it. An L30 will sketch a router or an offline enrichment pipeline that keeps 95 % of requests off the frontier model, keeps the calibrated decision on the classical side, and puts the LLM exactly where it earns its keep. This exercise trains that pattern-picking muscle against a concrete design.

## Prerequisites

- Read chapter 03 (`03-hybrid-classical-plus-llm-patterns.md`) — the five shapes, the request-lifecycle framing, and the evaluation-per-shape rules are load-bearing.
- Skim chapter 01 (`01-prompt-rag-finetune-hybrid-decision.md`) — Q5 is the hybrid gate.
- Skim chapter 04 (`04-cost-and-latency-guardrails.md`) — your composition must fit an envelope.
- Skim chapter 05 (`05-observability-for-llm-augmented-systems.md`) — the two-layer evaluation is unshippable without the observability that watches it.
- Optional: skim mod-303 chapter 05 (calibration) if the classical component in your design owns a decision whose threshold matters.

## Pick your scenario

Pick **one** of the four scenarios below, or bring your own following the same template (see stretch goals).

### H1 — Content-moderation triage

Every user post is routed through a classical multilabel classifier for ~40 policy categories. Most posts (say 95 %) are trivially benign or trivially violating; the remaining 5 % are ambiguous and today go to a human moderator queue that is chronically backlogged. Product wants the LLM to reduce the human queue by adjudicating the ambiguous slice. Volume ~2 M posts / day.

### H2 — Fraud-review analyst workbench

A calibrated fraud classifier scores every checkout. Above 0.98 auto-approves; below 0.02 auto-declines. The 0.02–0.98 grey zone (~10 % of traffic) goes to a fraud analyst who currently has no explanation for the score. Product wants an LLM to write a short "why the classifier flagged this" panel for the analyst on every grey-zone case, including which prior transactions look similar and which features contributed most.

### H3 — Search ranker with LLM re-ranker

Existing production search: BM25 retrieval → learned ranker (LTR) → top-10 results. Product wants an LLM to re-rank the top-50 candidates using the product description and the user query, then return the top-10 to the user. Peak 400 rps, p99 latency ≤ 400 ms end-to-end, catalogue of ~40 M SKUs.

### H4 — Product-catalogue enrichment

New SKUs arrive at ~500 k / day. For each, you need a normalised title, a category from a fixed taxonomy of ~1 200, four structured attributes (colour, size, material, brand), and a short marketing blurb. Downstream: the ranker, the search index, and the product-detail page all read from these.

## Steps

### 1. Problem statement — one paragraph (≈ 20 min)

For your chosen scenario, write a mod-302 chapter 01 shaped problem statement:

- Prediction / generation unit, consumer, freshness, volume, downstream cost, latency budget.
- Where the scenario is under-specified, state your assumption. A memo that assumes p99 ≤ 300 ms without saying so is not defensible.

### 2. Pick the hybrid shape(s) — chapter 03 (≈ 45 min)

Which of the five shapes applies? Almost every real feature composes at least two.

- **Shape 1 — Router.** Cheap first, expensive on the tail. Named router quality metrics (false-negative rate on hard slice, false-positive on easy slice).
- **Shape 2 — LLM as feature extractor.** Structured LLM output feeding a classical model. On-path or off-path?
- **Shape 3 — Classical decides, LLM presents.** Decision is classical, presentation is LLM, composition gate enforces consistency.
- **Shape 4 — LLM as verifier / fallback.** Golden path is classical, tail is LLM. Calibration of the classical side matters (mod-303 ch 05).
- **Shape 5 — Offline enrichment.** LLM off the request path, results cached in the feature store.

For each shape you pick, cite the "where it shows up" or "bet" reasoning from chapter 03 that made it the right shape for *this* scenario. If your design composes two shapes (shape 5 for the catalogue plus shape 1 for the runtime router, for example — chapter 03 gives that exact composition as an example), name that explicitly.

Chapter 03's framing question, applied here: **where in the request lifecycle does the LLM run?** On-path always? On-path sometimes (router / fallback)? Off-path (batch enrichment)? Answer for your design.

### 3. Author the composition contract (≈ 45 min)

For each shape you picked, the contract is:

- **Who owns the decision?** A calibrated classifier / ranker / rules engine, or the LLM. Chapter 03: "let classical models own decisions whose calibration matters."
- **What can the LLM see and change?** For shape 3 (classical decides + LLM presents), the LLM sees the classifier's output but cannot override the decision, and the composition gate rejects LLM outputs that contradict the classifier. Write the exact rule: "if the LLM output disagrees with the classifier's top-1 category, fall back to a default template."
- **What happens on failure?** Shape 4 fallback logic when the LLM is slow / errored / rate-limited: does the request fail closed (no LLM output shown), fall back to the classical branch alone, serve a cached prior answer, or degrade to a template? This is the chapter 04 degradation path applied to your specific composition.
- **What is versioned together?** If the LLM extractor version and the classical model version can drift independently, name how you attribute a downstream regression. Chapter 03 shape 2: "fingerprint the extractor version alongside the classical model version so an incident review can attribute the regression."

Draw the pipeline diagram. Boxes and arrows are fine — the reviewer cares that (a) the request-path is unambiguous, (b) the failure paths are visible, and (c) each box is labelled with its owner (classical model, prompt bundle, cache, router, LLM, ...).

### 4. Author the two-layer evaluation shape (≈ 45 min)

Chapter 03: "the extractor is evaluated on structured-output quality; the classical model is evaluated on its own end task; end-to-end is evaluated on the product metric." Apply this to your design:

- **Layer 1 — classical component metrics.** Whatever the classical model / router / ranker already owns: accuracy, calibration (ECE), nDCG, precision-at-k, false-negative rate on the hard slice for a router. Per-slice, following the mod-303 discipline.
- **Layer 2 — LLM component metrics.** Groundedness against the classical decision (for shape 3), structured-output field accuracy (for shape 2), tail catch rate (for shape 4), staleness / re-enrichment freshness (for shape 5). Use the chapter 05 quality-signal taxonomy: schema conformance, refusal rate, LLM-as-judge on samples, downstream user signal.
- **End-to-end product metric.** The single number the product manager cares about. Almost never a per-component metric.

Note explicitly which of the three layers each shape you picked owns. For a composition of shape 5 (catalogue enrichment) + shape 1 (runtime router), layer 2 splits across the offline extractor and the runtime router; layer 3 is the same product metric either way. The point is that these layers are *distinguishable* — if a regression drops end-to-end quality, you can attribute it to a specific layer without a week of debugging.

### 5. Sketch the guardrails and observability — one paragraph each (≈ 30 min)

You are not doing the full chapter 04 / 05 wire-up here (that is exercise 04). But every hybrid design needs a paragraph on:

- **Cost envelope.** Rough p50 and p99 dollar cost per user request under your composition. For shape 5 (offline), the dollar cost is per-batch not per-request; sketch both a per-batch and a per-request-served number.
- **Latency envelope.** p50 / p95 / p99 breakdown per branch. For a router, the two branches have different distributions; report both.
- **Observability day-one.** Which of the five signal families from chapter 05 you need instrumented on day one for the LLM component to be attributable when a regression fires. At minimum: `prompt_version`, `model_version`, per-request tokens, cost, branch (which router bucket the request landed in).
- **Failure and degradation.** How the feature degrades when the LLM component is slow / errored / over-budget. This is the chapter 04 degradation-path enumeration applied to your composition.

### 6. Name what you would escalate — one paragraph (≈ 15 min)

Chapter 06's Q1–Q6 apply: is any part of this design outside your team's remit? For H2 (fraud analyst workbench), the LLM component is a small prompt-bundle — keep. For H1 (content moderation with retrieval over a policy corpus), the "adjudication" might quietly need real retrieval and grounding — that is a hand-off to `rag-engineer-learning`. Be explicit; a memo that pretends everything is in-house is missing the escalation muscle chapter 06 trains.

## Deliverable

A single Markdown document `hybrid-feature-design.md`, three to four pages, with:

- Header — scenario, author, date, assumed constraints (RPS, latency, cost, freshness).
- **Problem statement** — one paragraph.
- **Chosen shape(s)** — the shape(s) picked with rubric-grounded reasoning; the position in the request lifecycle.
- **Composition contract** — decision owner, LLM authority, failure paths, versioning discipline. Includes the pipeline diagram (ASCII art, embedded image, or linked file).
- **Two-layer evaluation** — layer 1 (classical), layer 2 (LLM), end-to-end product metric. Per-slice where applicable.
- **Guardrails and observability preview** — one paragraph each on cost, latency, day-one observability, degradation.
- **Escalation call** — what you keep, what you would hand off (or explicitly say "nothing").
- **Rejected alternative** — one paragraph on a shape you considered and rejected and why the composition you chose beats it.

## Acceptance criteria

- [ ] The design picks at least one (usually two composed) shape(s) from chapter 03's five, with the shape name and the rubric reasoning explicit.
- [ ] The position in the request lifecycle (on-path always / sometimes / off-path) is named and defended.
- [ ] The pipeline diagram is present; every box has an owner label; failure paths are drawn, not implied.
- [ ] The composition contract answers: who owns the decision, what the LLM is allowed to change, what happens when the LLM fails, and how versions are joined for attribution.
- [ ] The two-layer evaluation is present and each metric names the slice it is measured on. No aggregate "was the whole answer good" metric with no slice.
- [ ] Cost, latency, day-one observability, and degradation are each covered in at least one paragraph, grounded in chapters 04 / 05.
- [ ] The escalation call is present — even if the answer is "nothing to escalate," it is stated and defended against the chapter 06 rubric.
- [ ] At least one rejected shape is named, with a rubric-grounded reason for rejection.
- [ ] The memo is at most four pages. Reviewers hate long memos; brevity is part of the exercise.

## Stretch goals

- **Envelope stress test.** Take your composition and re-do the cost / latency envelope under a 10× traffic scenario (viral launch) and a 100× scenario (hit product). Which shape breaks first? Chapter 04's "sanity check both directions" question, applied.
- **Instrument the observability.** Sketch the exact metric names, labels, and alert thresholds a chapter 05 wire-up would need for your composition. This overlaps with exercise 04 — combine them if you want the full end-to-end.
- **Bring your own.** Repeat against a real production feature you have access to. Anonymise as needed. Especially valuable if the current production shape is "LLM everything" and the exercise shows a better composition.
- **Peer review with a specialist.** If you have access to someone in `rag-engineer-learning`, `fine-tuning-engineer-learning`, or `llm-application-developer-learning`, walk them through the design and ask which pieces they would want a proper hand-off contract on (chapter 06). Their read of "you should not be building this yourself" is a signal.
- **Feed into the paired project.** The design memo produced here is the natural "system-design section" of `project-302-llm-augmented-ml-feature`.
