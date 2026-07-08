# The delegation contract to LLM specialists

## Motivation

Chapter 01 named the six-question rubric, and its sixth question — "is any of this actually your team's job, or is it a specialist track's?" — is the one this chapter answers. Chapters 02–05 assumed the LLM-augmented feature was yours to build; this chapter is what you do when it *should not* be, or when part of it should not be.

Mod-303 chapter 08 covered the escalation rubric for modeling regimes at senior altitude. The LLM specialists are a specific case of that rubric, and one that L20-shaped instincts get wrong more often than any other: the "LLM everything" reflex says every LLM-adjacent piece is glamorous enough to keep in-house, and the "we'll figure it out" reflex assumes a full RAG or fine-tuning stack is a two-week weekend project. Neither is true. The senior read is: **the LLM specialist tracks exist because deep LLM work has its own paved roads, its own vocabularies, and its own failure modes that a generalist ML team ships badly.** Your job at L30 is not to build all of it. Your job is to recognise which part belongs where and to write a hand-off contract that survives the quarter.

This chapter is the LLM-specific version of that discipline. Same shape as mod-303 chapter 08 — a rubric, a per-track territory map, a hand-off checklist, and the reverse mistake — narrowed to the four peer tracks that own LLM depth.

## The four LLM specialist tracks — the map

Every one of these is a peer track in the AICG curriculum ecosystem, at (or approaching) L30. Each has its own paved road; each is a legitimate destination for delegation.

### `llm-application-developer-learning`

**Territory.** Prompt engineering at production depth, prompt orchestration, tool use and function calling, agent design (planner + tools + memory), agentic guardrails, streaming and structured output, LLM-application eval harnesses, LLM-application observability at depth.

**Consume from.** Any feature whose load-bearing work is *prompt design + orchestration* — the prompt is complex, the tool loop is non-trivial, the agent has to plan across multiple steps. Your team should not be prototyping four agent frameworks in parallel; the LLM-application track has done that homework.

**Escalate when.** The prompt is not a paragraph; it is a system. The tool use is not a single function call; it is a planner-tool-memory loop with retries and backtracking. The evaluation is not "did the summary look OK"; it is a full LLM-application eval harness with LLM-as-judge, human review, and per-slice pass rates.

**Keep in-house when.** The prompt is one or two paragraphs, the output schema is a fixed JSON object, there is no tool loop, and the eval harness is small enough to live in your team's repo (chapter 02).

### `rag-engineer-learning`

**Territory.** Chunking strategies, embedding-model selection and evaluation, vector store operations, hybrid retrieval (BM25 + dense), retriever evaluation (recall@k, context recall), citation and groundedness evaluation, ranker + LLM composition, RAG-specific observability.

**Consume from.** Any feature whose load-bearing quality driver is *retrieval*. The corpus is large enough to need indexing; the answer's correctness is grounded in "did we retrieve the right passage." Mod-303 chapter 08 already covered the escalation shape; the LLM angle here is that RAG has its own eval vocabulary that your team's classical eval harness cannot express.

**Escalate when.** The answer must cite the source; groundedness is a first-class requirement; retrieval and generation regressions need to be *separately* diagnosable; the vector store is now infrastructure you would be on-call for.

**Keep in-house when.** The "retrieval" is a fixed lookup against a small structured store your team already owns (a last-ten-tickets fetch, a per-user profile fetch). That is not RAG; that is prompt-time context injection (chapter 01, Q2). Do not summon a RAG specialist for a database read.

### `fine-tuning-engineer-learning`

**Territory.** Supervised fine-tuning (SFT), instruction tuning, preference tuning (RLHF, DPO), parameter-efficient fine-tuning (LoRA / QLoRA), preference-data collection, fine-tuning evaluation, catastrophic-forgetting mitigation, distributed training infrastructure for LLM fine-tuning.

**Consume from.** Any feature that has passed the chapter 01 Q4 gate — prompting has been genuinely exhausted, the desired behaviour is not achievable by prompt alone or is materially cheaper as a fine-tune, and you have (or can create) a training dataset in the thousands to tens of thousands.

**Escalate when.** The training is more than a small in-repo LoRA on a small open model. The moment RLHF, DPO, or a multi-GPU SFT enters the plan, you are in specialist territory. Preference-data collection alone is its own multi-quarter operation.

**Keep in-house when.** A LoRA on an 8B open model, trained on a labelled dataset your team already owns, evaluated in your team's own harness, deployed on a paved-road serving path. That can live on a senior ML team without the fine-tuning specialist stack.

### `training-pipeline-engineer-learning`

**Territory.** Distributed training infrastructure — data parallel, tensor parallel, pipeline parallel, ZeRO / FSDP sharding, distributed checkpointing, large-scale data loaders, throughput and hardware efficiency.

**Consume from.** Any training run that spills off a single node, needs sharded optimiser state, or is bottlenecked by data loading rather than model compute.

**Escalate when.** Multi-node fine-tuning, or single-node runs where you find yourself hand-rolling FSDP configuration and worrying about all-reduce hangs.

**Keep in-house when.** A single-GPU or single-node LoRA on a small open model, using a stock trainer library. That is *inside* fine-tuning territory but on the shallow end where the training-pipeline specialist is not the right hand-off.

Also worth noting on the map even though these are one step off the LLM specialist path:

- **`ai-eval-engineer-learning` and `model-evaluation-engineer-learning`** for LLM eval harnesses at platform scale — mod-305 covers when to consume from these. If your team is on its third bespoke LLM eval harness, consume the paved-road one.
- **`ai-infra-ml-platform-learning` and `ai-infra-mlops-learning`** for serving and monitoring the artefact once the specialists hand it back. Mod-308 is the whole discipline.
- **`ai-governance-analyst-learning` and `ai-infra-security-learning`** for regulated or high-risk LLM deployments. Mod-309 for the governance shape.

## The delegation rubric — six questions specific to LLM work

The mod-303 chapter 08 five-question rubric applies here too. This section adds the LLM-specific ones and re-runs the whole thing so a reviewer can read this chapter standalone.

### Q1 — Is the load-bearing skill in one of the four LLM tracks' titles?

*Prompt engineering, orchestration, agents* → `llm-application-developer-learning`. *Retrieval + grounding* → `rag-engineer-learning`. *Weight updates at production scale* → `fine-tuning-engineer-learning`. *Distributed training* → `training-pipeline-engineer-learning`.

If the load-bearing skill is squarely in a peer track's name, that is the strongest single signal to escalate.

### Q2 — Does the paved-road version exist in-org, and is your team really the right team to build it?

If a paved-road RAG stack or fine-tuning pipeline exists at your org, and you build your own, you are duplicating work and taking on debt. If the paved road does not exist yet, the escalation shape is: bring the requirement to the peer track's roadmap conversation. Do not launch a fork.

### Q3 — Is the LLM piece the whole feature, or one component of a larger ML system?

If the LLM is the whole feature (a chatbot, a RAG assistant, an agentic tool), the LLM specialist tracks are the natural home. If the LLM is one component of a larger ML system your team already owns (chapter 03's hybrid patterns), the composition itself is a senior-ML-engineer job even when a peer track owns the LLM component.

The pattern is: **the L30 owns the composition even when the LLM piece is delegated.** You do not hand off "the whole feature" — you hand off "the LLM component" and integrate it back.

### Q4 — Is the evaluation vocabulary yours or a peer track's?

LLM applications are evaluated in vocabulary your classical eval harness does not speak: LLM-as-judge, per-turn agent eval, groundedness, citation quality, refusal rate at the safety boundary. If the evaluation itself belongs in a peer track's harness, so does the build. Splitting build from eval across teams is a reliable way to ship a system nobody can measure. Mod-305 has the eval-track hand-off discipline in more depth.

### Q5 — What is the operational cost you are committing to?

Owning a fine-tuning pipeline is a multi-quarter on-call surface. Owning a RAG stack is a multi-quarter on-call surface. Owning a hosted-LLM prompt-only feature is not — it is a few observability panels and a runbook. Ask honestly what the on-call rotation looks like for the next twelve months. "We can build it" is not the same as "we want to be paged about it every fourth week."

### Q6 — What is the hand-off contract?

If you *do* escalate, you write the contract in one document. This is the checklist below. Every escalation the runbook remembers as a success has this contract on file; every one it remembers as a failure did not.

## The hand-off contract — the eight fields the L30 fills in

The document you hand a peer track has these fields. Nothing less. If any is blank, the hand-off is a shrug and it will come back to you.

### 1. Problem statement

What the feature does, expressed in the mod-302 chapter 01 vocabulary — prediction unit, consumer, freshness, volume, downstream cost, latency budget. "We want an LLM that helps users with X" is not a problem statement. "Given a customer's last ten tickets, produce a ≤ 120-word summary in the assistant's registered voice, shown on ticket open at 500 rps peak, p99 latency ≤ 1.2 s from ticket-open event, error path is graceful hide" is a problem statement.

### 2. Data

What you own and can share. What formats and what freshness. What PII / sensitivity discipline applies. What the peer track has to source themselves. Any legal / compliance framing that the peer track needs to know before they touch it. Mod-309 covers the governance shape.

For a fine-tune hand-off, this is where the training-data provenance conversation happens up-front: how labels were collected, what the annotator agreement was, what slices the dataset is under-represented on. That paperwork feeds the peer track's eval slices, not just their training.

### 3. Evaluation

Who owns the eval harness, what the acceptance criteria are (per-slice pass rates with thresholds, not a single top-line number), how release qualification is defined. The eval harness lives with whoever owns the *product surface* even if the model is trained by a peer track. This means: **for a fine-tune hand-off, the fine-tuning track produces the artefact; your team's eval harness is the release gate.** Do not let the eval go with the model unless you also handed off the product ownership.

For an LLM-application hand-off, the eval harness may legitimately move with the feature — but only if the peer track is also owning the user-facing metric, which is rare.

### 4. Interface

The runtime shape of the artefact. REST endpoint (what path, what request / response schema)? Batched job (what queue, what SLA)? In-process library (what language, what versioning)? Streaming (what topic, what protocol)? Mod-302 chapter 02 covers the batch / streaming / online decision; this field is that decision applied to the peer track's deliverable.

Specifics matter: the request schema, the response schema, the error contract (what does a refusal look like on the wire), the auth model (whose credentials, whose quota), the versioning discipline (how you bump the endpoint when the model changes).

### 5. Cost and latency envelope

The chapter 04 envelope, restated in the contract. Per-request cost budget, per-request latency budget, concurrency ceiling, aggregate spend budget, degradation path. The peer track needs the envelope up-front because their design decisions (model tier, prompt length, batching) fall out of it.

If the peer track pushes back on the envelope, that is a legitimate design conversation — engage. What is *not* legitimate is silence followed by a delivery that misses the envelope; the contract exists to make silence impossible.

### 6. Observability contract

Which of the chapter 05 signals the peer track emits on their side, and which your team wires on top of the delivered interface. As a rule: the peer track emits the LLM-specific signals (per-invocation logs, token counts, cost per call, model version); your team emits the end-to-end request signals (user-facing latency, downstream user metrics, feature-level quality). Both teams' signals are joined by a shared request ID.

Vendor rate-limit and status attribution: who watches the vendor's status page for this feature and how the on-call chain runs when the vendor is degraded. See mod-307 (SLOs, incident response) for the shape.

### 7. On-call and incident response

The single most-often-missing field. When the artefact regresses, whose phone rings first? What is the escalation chain? Whose runbook do they read? Whose kill switch do they flip to degrade the feature?

The senior read: for a delegated LLM component, the peer track owns "the component is broken" pages; your team owns "the feature is broken" pages; both are on the same incident when a component-broken causes a feature-broken. The kill switch to degrade to chapter 04's fallback path is *your team's* switch, because your team owns the user-facing metric.

### 8. Sunset / renegotiation clause

The default assumption is that the peer track keeps owning the artefact. But if the feature grows to the point where composition dominates depth (chapter 03 hybrid gets deeper than the LLM component), or the peer track's roadmap moves away, the contract has to be renegotiable. Name a review cadence — quarterly, semi-annually — and the conditions that trigger a re-scoping.

That is what turns "we escalated" from a hand-wave into a contract that survives the quarter. mod-308 covers the shape of platform-and-peer collaboration more broadly; this is the LLM-specific application of that discipline.

## What stays with your team even after escalation

The mod-303 chapter 08 line applies here too: **you delegate the how, not the what.** After you hand off to any of the four LLM specialist tracks, your team still owns:

- **The problem statement.** Only your team knows the business context.
- **The evaluation slices.** Only your team knows which failure modes are unacceptable in your product.
- **The integration.** How the peer track's artefact plugs into your ranker, your batch pipeline, your feature store, your serving path.
- **The chapter 04 envelope enforcement.** Cost budget, latency budget, degradation path, kill switch.
- **The user-facing metric.** The peer track will optimise their own metrics; your team owns the metric the product cares about.
- **The rollback / disable path.** If the peer track's artefact degrades, your team is the one that switches to the fallback.

Everything else is negotiable in the contract.

## Two worked examples

### Example A — a documentation-search assistant (escalate)

**Feature.** Users ask questions about the company's public documentation ("how do I roll a token?", "which regions is feature X available in?"). The assistant returns a natural-language answer that cites the relevant doc.

Rubric walk:

- Q1 — load-bearing skill? Retrieval + grounded generation with citations. → `rag-engineer-learning` territory.
- Q2 — paved road? Yes, the RAG track owns one. Building a second one is duplication.
- Q3 — LLM is the whole feature. There is no larger classical ML system this composes into.
- Q4 — eval vocabulary is RAG-shaped (context recall, groundedness, citation quality). Your classical eval harness cannot express it.
- Q5 — operational cost of an in-house RAG stack (vector DB, embedding pipeline, retriever eval, on-call) is not one your team wants.
- Q6 — the contract: problem statement + acceptance criteria + user-facing metric on your side; retrieval + generation + RAG-specific eval on the RAG track's side; shared observability; your team owns the kill switch to hide the feature.

→ Escalate. Your team's deliverable is the requirements, the acceptance criteria, and the integration into the product surface. The RAG track's deliverable is the running system that meets the acceptance criteria.

### Example B — a hybrid ranker with an LLM re-ranker (keep, but with a specialist consult)

**Feature.** A search ranker over the product catalogue. Classical retrieval + classical ranker for the top-1000, then an LLM re-ranks the top-50 using product descriptions and the query.

Rubric walk:

- Q1 — load-bearing skill? Ranking. The LLM is one component in a composition your team already owns.
- Q2 — paved road? Yes, the ML platform's ranker training / serving stack; you consume it. The LLM re-ranker is a prompt + call, no bespoke stack needed.
- Q3 — LLM is one component; the composition is the feature. Chapter 03 shape-3 territory.
- Q4 — eval vocabulary is ranking-shaped (nDCG, MRR, per-slice) plus a small LLM-application eval slice for the re-ranker. Both live in your team's harness.
- Q5 — operational cost is manageable; you already run rankers.
- Q6 — no hand-off of the whole feature. But *do* consult the LLM-application track on the re-ranker prompt design, and consult the RAG track only if the re-ranker starts wanting corpus retrieval it does not currently need.

→ Keep. Chapter 03's hybrid pattern is the shape; chapters 04–05 wire the guardrails and observability on the LLM branch. No hand-off contract needed for the whole feature; a light consult with the LLM-application track on prompt design is a reasonable use of a peer track without the full contract shape.

## The reverse mistake — escalating what you should own

This one is common with LLM work specifically. Product wants a feature; the ML team says "this is an LLM feature, we'll hand it to the LLM track"; the whole thing floats over to a peer team; six months later the ML team no longer owns the user-facing metric on a product surface that is still theirs to explain.

Rubric on that reverse case:

- Q1 — load-bearing skill is *composition* between a classical component your team already owns and an LLM component. That is senior ML engineer territory, not LLM-application territory.
- Q3 — the LLM is one component, not the whole feature. The composition owns the outcome.
- Q4 — the eval slices the product cares about are yours to define, not the LLM track's.

→ Own it. Delegate the depth of the LLM component (prompt design, orchestration) via a consult if you need to, but keep the composition and the user-facing metric.

## Three failure modes to catch in review

- **Escalating the whole feature to keep the LLM component on someone else's plate.** The team hands off "the AI feature" to the LLM-application track, then discovers three quarters later that the user-facing metric has drifted and no one is on the hook. Fix: the composition owner keeps the user-facing metric even when the LLM component is delegated. Chapter 04's degradation path and kill switch live with the composition owner too.
- **Escalating without a signed contract.** The team says "the RAG track is on it" without the eight-field checklist. Six months later there is a demo of something adjacent to what you asked for and no one can say whether it should ship. Fix: the eight-field hand-off contract is mandatory; if the peer track will not sign it, that is a signal you need to re-scope before starting the work.
- **Building a paved-road piece in-house because the peer track's is "not quite right".** The team builds a bespoke fine-tuning pipeline because the training-pipeline track's version is missing a feature they want. Nine months later they own a training pipeline no one else uses and the peer track has shipped the missing feature anyway. Fix: work with the peer track to extend the paved road; do not fork it.

## Summary

The four LLM specialist tracks — LLM application, RAG, fine-tuning, training pipeline — exist because deep LLM work has its own paved roads and its own failure modes. The senior ML engineer's job is not to build all of it; it is to recognise which part belongs where and to write a hand-off contract with eight fields: problem statement, data, evaluation, interface, cost / latency envelope, observability, on-call, sunset. You delegate the how — the LLM component's implementation, its eval vocabulary, its infra — but not the what: the problem statement, the release gate, the integration, the envelope, the kill switch, and the user-facing metric all stay with your team. The reverse mistake — escalating what you should own — is just as expensive as the forward mistake — building what a peer track already ships. The contract is what turns "we escalated" from a shrug into an agreement that survives the quarter. mod-308 covers the broader peer-track and platform collaboration discipline this chapter is a specific case of.
