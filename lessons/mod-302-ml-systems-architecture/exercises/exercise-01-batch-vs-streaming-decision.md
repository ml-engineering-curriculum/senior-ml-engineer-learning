# exercise-01: Batch vs. Streaming Decision

**Estimated effort:** 3 hours

## Objective

Take a concrete business problem and walk it through the six-question rubric from `02-batch-streaming-online-offline.md`, producing a defensible decision on **serving posture** (batch / online / streaming) and **feature freshness** (offline / online / streaming). The deliverable is a short decision memo — one to two pages — that a peer L30/L40 reviewer could sign off on.

This exercise trains the muscle you use every time a new ML system RFC lands on your desk. It is deliberately small: no code, no diagrams beyond a rough box-and-arrow sketch. The whole point is to force the decisions to be *pinned to constraints*, not to a technology preference.

## Prerequisites

- Read chapter 02 (`02-batch-streaming-online-offline.md`) in full — the six-question rubric is the load-bearing tool.
- Skim chapter 01 (`01-from-problem-to-system-shape.md`) for the four load-bearing decisions.
- Optional but recommended: skim chapter 04 (`04-training-serving-skew.md`) so you can name the skew implications of your posture choice.

## Pick a scenario

Pick **one** of the two options below. Both are stated at the level of ambiguity a real product manager would deliver — you are expected to state the missing details as *assumptions* in your memo, not to ask a fictional PM. A real L30 makes assumptions explicit and defensible.

### Option A — bring your own

A production or planned ML system you have real access to. Anonymise anything sensitive. If you use this option, add a one-paragraph "Context I already know" section at the top of the memo describing the current system (if any) and the constraint that is forcing the design conversation now.

### Option B — pick a case scenario

Pick one of the following. Each has a real ambiguity you are expected to resolve as an assumption.

- **Scenario B1 — Fraud scoring for a mid-size e-commerce platform.** ~800 rps at peak, ~2 M transactions/day, current process is a rules engine plus a nightly human review queue. Product ask: "flag transactions the risk team should review, and eventually auto-decline the highest-confidence fraud." Ambiguities: is v1 in the request path? Is auto-decline a goal? What is the latency budget if so?
- **Scenario B2 — Personalised search re-ranking for a mid-size retail catalogue.** 40 M SKUs, 5 M weekly active users, current ranker is BM25 with a hand-tuned boost. Product ask: "personalise the top-20 results per user, without breaking p99 <120 ms." Ambiguities: session-level freshness or user-history-only? Cold-start behaviour for new users?
- **Scenario B3 — Delivery-time prediction for a logistics platform.** Displayed to consumers on checkout; ~500 orders/minute at peak; freshness of "current road/weather conditions" is unclear. Product ask: "give consumers an accurate ETA and stop over-promising during peak hours." Ambiguities: is the model consumed in real time in checkout, or precomputed per (origin, destination) pair on a schedule?
- **Scenario B4 — Content-moderation triage for a user-generated-content platform.** Every user upload has to be routed within seconds to auto-approve, auto-reject, or human review. Volume is bursty; false positives are worse than false negatives on some content types. Ambiguities: what latency SLO on the "auto-approve" branch? What does the human-review queue look like?

## Steps

### 1. State the problem in your own words (≈ 15 min)

Write, in one paragraph, the four bullets from chapter 01's step 1:

- Prediction unit — what does the model score, and what is the entity?
- Consumer — human? system? automated action? What is the cost of "wrong"?
- Freshness the consumer needs — hours? seconds? milliseconds?
- Volume — peak rps, total per day.

Where the scenario is ambiguous, state your **assumption** and one sentence on why it is reasonable ("assumed p99 <100 ms latency budget because the ranker sits in the request path"). If you are using Option A, this section is short and specific.

### 2. Walk the six-question rubric (≈ 60 min)

For each of Q1–Q6 from chapter 02:

- Quote or paraphrase the question.
- Answer it with reference to your step-1 statement.
- Note whether the answer *forces* a posture choice (if yes, stop and record).

At minimum, produce a decision on:

- **Serving posture** — batch / online / streaming.
- **Feature freshness for the top ~3 features you expect to use** — offline / online / streaming.

Do not skip Q5 (operability envelope) or Q6 (cheapest sufficient posture). Those are the questions juniors most consistently skip and are the ones that surface the "over-engineering from vibes" anti-pattern.

### 3. Name at least one alternative you rejected (≈ 30 min)

Following chapter 06's section-5 discipline: pick a shape you seriously considered and rejected. Write one paragraph on why. If your primary answer is batch, the rejected alternative is probably a streaming or online shape; if your primary answer is streaming, name the batch shape you rejected and why. A memo without a rejected alternative reads as unconsidered.

### 4. Sketch the parity implications (≈ 30 min)

Given your posture and freshness answers, name — in one or two sentences each — which of the five skew categories from chapter 04 you now have to defend against, and how. You do not have to solve them; you have to *name* them. This section is what makes the reviewer trust the memo.

### 5. Write the non-goals (≈ 15 min)

Three to five bullets, using the chapter-06 non-goals style. State clearly what the v1 system **is not** doing and why.

## Deliverable

A single Markdown document `batch-vs-streaming-<scenario>.md`, roughly 1–2 pages, with the following sections in order:

- Header — one line per: scenario, date, author, assumed constraints.
- **1. Problem statement** — prediction unit, consumer, freshness, volume.
- **2. Rubric walk** — Q1 through Q6 with answers and citations back to section 1.
- **3. Decision** — the serving posture and feature freshness choices, one sentence each.
- **4. Rejected alternative** — one paragraph on the shape you considered and did not choose.
- **5. Parity implications** — which of the five skew categories now apply, one sentence each.
- **6. Non-goals** — 3–5 bullets.

## Acceptance criteria

- [ ] Every posture decision cites the specific rubric answer that forced it (or explicitly notes that Q6 chose the cheapest sufficient shape when nothing forced it).
- [ ] Where the scenario was ambiguous, assumptions are stated explicitly and briefly justified.
- [ ] Feature-freshness decisions are named for at least three named features, not left generic.
- [ ] At least one seriously-considered rejected alternative is described with a reason grounded in the rubric.
- [ ] The parity-implications section names concrete skew categories from chapter 04, not a hand-wave.
- [ ] Non-goals are present, specific, and defensible.
- [ ] The memo is *at most two pages* — reviewers hate long memos; discipline is part of the exercise.

## Stretch goals

- **Rubric-walk contrast.** Redo the rubric walk for the same scenario under a modified constraint — e.g., "what if the freshness budget were 5 seconds instead of 5 minutes?" — and describe how the decisions change. This is the single best training exercise for the "constraint sensitivity" muscle.
- **Cost estimate.** Sketch a rough monthly cost envelope for your chosen posture and the rejected alternative. Order of magnitude is fine; the exercise is comparing shapes, not budget accuracy.
- **Peer review.** Trade memos with a peer working through the same module. Reviewer runs the chapter-06 "what is the reviewer actually reading for" checklist against your memo. Every "cannot find" is a memo revision.
- **Feed into the RFC.** Reuse this memo as the seed for section 4.1 (serving posture) of the RFC you will produce in the paired project (`project-301-ml-system-design-portfolio`).
