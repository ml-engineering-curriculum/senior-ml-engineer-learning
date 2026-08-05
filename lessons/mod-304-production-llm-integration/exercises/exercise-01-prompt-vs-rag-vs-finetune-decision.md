# exercise-01: Prompt vs RAG vs Fine-Tune Decision

**Estimated effort:** 3 hours

## Objective

Take three concrete business features and walk each one through the six-question rubric from chapter 01 (`01-prompt-rag-finetune-hybrid-decision.md`), producing a defensible pattern decision — prompt-only, RAG, fine-tune, hybrid, or **escalate to a specialist track** — for each. The deliverable is a short decision memo, roughly a page per feature, that a peer L30/L40 could sign off on.

This exercise trains the muscle you use every time product walks in with "we want the assistant to do X." The point is to force the pattern choice to be pinned to constraints from the problem statement rather than to whichever pattern is currently fashionable. At L20 the temptation is "fine-tune everything"; at L30 the temptation is "prompt everything and hope." Neither is defensible; the rubric is.

Getting one of the three features to the answer "this is not our team's problem — escalate to the RAG / fine-tuning / LLM-application track" is a passing outcome. That is the point of chapter 06.

## Prerequisites

- Read chapter 01 (`01-prompt-rag-finetune-hybrid-decision.md`) — the six-question rubric is load-bearing.
- Skim chapter 03 (`03-hybrid-classical-plus-llm-patterns.md`) so you can name the five hybrid shapes when a feature calls for one.
- Skim chapter 06 (`06-delegation-contract-to-llm-specialists.md`) — you cannot answer Q6 well without the four specialist tracks' territories.
- Optional: skim mod-303 chapter 08 (`08-when-to-escalate-to-a-specialist-track.md`) for the underlying escalation shape this rubric specialises.

## Pick your three scenarios

Pick **three** from the list below. Deliberately choose three that push you into different pattern answers; solving three "summarise a document" features does not train the rubric.

- **F1 — Support-ticket triage panel.** When an agent opens a ticket, show the top-3 suggested next actions from a fixed taxonomy of 12 (refund, escalate to L2, ask for order ID, etc.) plus a one-paragraph summary of the customer's last ten interactions. ~500 rps peak, p99 latency ≤ 1.5 s from panel open, ~2 M historical (ticket, action) label pairs available.
- **F2 — Documentation Q&A assistant.** Users on the public docs site type a natural-language question ("how do I roll a token?"); the assistant answers in natural language and cites the relevant doc page. Corpus is ~200 k docs pages plus SDK reference plus community forum threads, updated daily.
- **F3 — Product-listing enrichment.** Merchants upload new SKUs; you need a normalised title, a category from a fixed taxonomy of ~1 200, extracted structured attributes (colour, size, material), and a short marketing blurb. ~500 k new SKUs / day, no request-path latency requirement.
- **F4 — Brand-voice draft-reply generator.** In an internal support tool, generate a first-draft reply in the company's established brand voice (specific tone, register, and formatting conventions) for the agent to edit. Prompting a frontier model produces good English but does not hold the brand voice; the marketing team has strong opinions and ~30 k historical exemplar replies.
- **F5 — Fraud-scoring at checkout.** Score every checkout event and auto-approve above 0.98, auto-decline below 0.02, human review in between. ~800 rps peak, p99 latency ≤ 120 ms, regulator asks for stated calibration on the auto-decline slice.
- **F6 — Field-agent multi-step assistant.** A technician in the field asks the assistant a plain-English question, and the assistant may need to look up several internal systems (inventory, work-order history, part manuals), summarise them, and produce an action recommendation. Multi-step tool use, streaming, session memory.
- **F7 — Bring-your-own.** A production or planned feature you have access to. Anonymise anything sensitive and add a one-paragraph "context I already know" preamble.

## Steps

### 1. Problem statement — one paragraph per feature (≈ 15 min per feature)

For each chosen feature, name:

- The prediction / generation unit (per ticket, per query, per SKU, per checkout).
- The consumer (human, another system, an automated action, a dashboard).
- The freshness and volume the consumer needs.
- The downstream cost of "wrong" — is a threshold triggering an action? A cost calculation? A user-facing display? A regulator-visible slice?
- The latency budget and any hard constraints (regulatory, brand, safety).

Where the scenario is ambiguous, state your assumption and justify it briefly. A memo that assumes p99 ≤ 300 ms without saying so is not defensible.

### 2. Walk the six-question rubric — for each feature (≈ 30 min per feature)

For each of Q1–Q6 from chapter 01:

- Quote or paraphrase the question.
- Answer it with reference to your step-1 problem statement.
- Note whether the answer *forces* a pattern (Q1's classification vs generation call, Q2's proprietary-data call, Q4's fine-tune-warranted call) or whether the pattern is picked in a later question.

At minimum, produce a decision on:

- **Primary pattern** — prompt-only, RAG, fine-tune, one of the five hybrid shapes (chapter 03), or "escalate to specialist track X."
- **Position in the request lifecycle** — on-path always, on-path sometimes (router / fallback), or off-path (batch enrichment). Chapter 03's framing.
- **Cross-cutting decisions** — what does the eval set look like? What are the load-bearing per-slice metrics? Is there a classical component the LLM composes with?

Do not skip Q3 (prompting-sufficient) or Q4 (fine-tune-warranted). Those are the two the L20 instinct most reliably answers wrong; they are exactly the questions that separate "we fine-tuned because we did not try harder prompts" from "we shipped prompting because it works."

### 3. Name at least one seriously-considered alternative you rejected — per feature (≈ 15 min per feature)

One paragraph per feature on a pattern you considered and rejected, grounded in the rubric. If your primary answer is prompt-only, is a fine-tune what you rejected, and why? If your primary answer is "escalate to `rag-engineer-learning`," is an in-house RAG build what you rejected, and why?

Memos without a rejected alternative read as unconsidered.

### 4. Name the follow-up decisions — per feature (≈ 15 min per feature)

Given the pattern, name in one or two sentences each the second-order decisions the module already knows are load-bearing:

- Prompt-only → the prompt bundle shape (chapter 02), the eval slices, the max-tokens cap.
- RAG (escalated) → what the hand-off contract to `rag-engineer-learning` looks like (chapter 06), what stays with your team (the problem statement, the eval slices, the user-facing metric).
- Fine-tune (escalated) → what the hand-off contract to `fine-tuning-engineer-learning` looks like, what training data your team owns and what the peer track has to source.
- Hybrid → which of the five chapter-03 shapes, which component owns the decision, which component owns the presentation.
- Cost / latency envelope (chapter 04) — sketch a p50 and p99 for cost, latency, and concurrency.
- Observability (chapter 05) — one line on which of the five signal families your team needs to instrument on day one.

You do not have to solve these; you have to name them. The reviewer wants to know you know they are coming.

## Deliverable

A single Markdown document `pattern-decisions.md`, roughly three pages total (about a page per feature), with the following structure:

- Header — date, author, features picked, any assumed constraints (RPS, latency, cost, regulator posture).
- **Feature 1** — problem statement, rubric walk (Q1–Q6), pattern decision, rejected alternative, follow-up decisions.
- **Feature 2** — same shape.
- **Feature 3** — same shape.
- **Cross-scenario reflection** — one short paragraph on what surprised you across the three, especially any place your first instinct said "fine-tune" and the rubric said "prompt harder," or your first instinct said "prompt-only" and the rubric said "escalate to a specialist track."

## Acceptance criteria

- [ ] Three distinct features covered, each pushing the rubric toward a different pattern (not three variations of one theme, not all three prompt-only).
- [ ] Every pattern choice cites the specific rubric answer that forced it — Q1's generation-vs-classification, Q2's proprietary-data, Q3's prompting-sufficient, Q4's fine-tune-warranted, Q5's hybrid-lifts-the-trade-off, or Q6's specialist-territory.
- [ ] Where the scenario was ambiguous, assumptions are explicit and briefly justified.
- [ ] At least one seriously-considered rejected alternative per feature, with a rubric-grounded reason for rejection.
- [ ] Follow-up decisions named for each pattern, drawing from the correct chapter of the module (02 for prompt-only shape, 03 for hybrid, 04 for envelope, 05 for observability, 06 for hand-off).
- [ ] At least one feature results in a "escalate to specialist track" decision, or you explain in the cross-scenario reflection why none did.
- [ ] The memo is at most about three pages. Reviewers hate long memos; brevity is part of the exercise.

## Stretch goals

- **Constraint sensitivity.** Re-walk one feature under a modified constraint ("what if the training-label budget were zero?" or "what if the latency budget tightened to p99 ≤ 200 ms?" or "what if the model tier price doubled overnight?") and describe how the pattern changes. This is the single best training exercise for pattern flexibility.
- **Cost envelope.** Sketch a rough monthly cost envelope (chapter 04) for the chosen pattern and the rejected alternative. Order of magnitude is fine; the point is to see whether the rejected alternative would have been affordable at expected scale.
- **Peer review.** Trade memos with a peer working through the module. Reviewer runs the chapter-06 delegation rubric against every "escalate" call and the chapter-03 hybrid-shape checklist against every hybrid call. Each "cannot find" is a memo revision.
- **Feed into the paired project.** Reuse one of your memos as the seed for `project-302-llm-augmented-ml-feature`. The pattern decision from this exercise is the natural first section of that project's design doc.
