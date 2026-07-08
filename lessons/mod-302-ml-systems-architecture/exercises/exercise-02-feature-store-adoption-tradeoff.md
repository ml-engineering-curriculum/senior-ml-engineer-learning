# exercise-02: Feature Store Adoption Trade-off

**Estimated effort:** 3 hours

## Objective

Produce a defensible trade-off memo for whether a specific ML system should adopt a feature store or stay with an ad-hoc feature-computation approach. Walk the chapter-03 adoption signals, name the concrete cost items, and land on one of three answers: adopt, defer, or explicitly not adopt. The memo is what an L30 hands to their tech lead when the "should we build a feature store?" or "should we onboard onto the platform team's feature store?" question first shows up in the roadmap.

This exercise trains the muscle of *saying no to sophistication* when the signals do not support it — and *saying yes with a plan* when they do. Both answers are L30 answers when they are pinned to constraints; neither is when it is pinned to fashion.

## Prerequisites

- Read chapter 03 (`03-feature-store-and-model-registry.md`) in full — the adoption signals A/B/C and the ad-hoc discipline are the load-bearing tools.
- Skim chapter 04 (`04-training-serving-skew.md`) so you can name the skew categories the memo has to defend against under each option.
- Optional but recommended: skim the Feast documentation (<https://docs.feast.dev/>) so the concrete costs are not hypothetical.

## Pick a scenario

Pick **one** of the following. In both cases the memo will read differently, and that is the point — the correct answer to "should we adopt a feature store?" depends heavily on the *organisational* preconditions, not on the model.

### Option A — bring your own

An ML system you have real access to (or an analogue you know well). Anonymise as needed. Include one paragraph on how many models the organisation has today, how many share features, and whether a paved-road feature-store platform exists.

### Option B — pick a case scenario

- **Scenario B1 — Single-team, single-model, batch-only.** Your team owns one model (a churn scorer). Latency budget: daily. No peer platform team offers a feature store. Two other teams do ML but on unrelated features. Roadmap: no additional ML systems planned for the next year.
- **Scenario B2 — Multi-team, multi-model, mixed latency.** Your ML org has six models across three teams. Three of them consume overlapping features (user-30d aggregates, merchant risk aggregates). Two of the models sit in the request path with p99 <200 ms budgets. A peer platform team (`ai-infra-ml-platform-learning` analogue) offers a paved-road Feast deployment.
- **Scenario B3 — Migration in flight.** The org runs a legacy in-house "feature service" that was built two years ago. It is a database of feature values updated by scheduled jobs; it has no point-in-time joins, no CI, and one on-call engineer who is leaving. The org is deciding: rebuild against the paved-road option, migrate to a commercial feature store, or return to ad-hoc.
- **Scenario B4 — Streaming-heavy, LLM-adjacent.** Your team runs a session-personalisation model that consumes streaming features from a Kafka topic. An adjacent team is building an LLM-based feature (embedding lookup) they want to publish for reuse. Latency budget: p99 <100 ms. No paved-road feature store exists.

## Steps

### 1. Establish the baseline (≈ 20 min)

Write, in one page or less:

- Which model(s) are in scope for the decision.
- Which features they consume today and how those features are computed (offline SQL, online lookup, streaming aggregate).
- Whether the current shape has documented training/serving skew incidents (chapter 04 categories 1–5).
- Whether a peer platform team exists that could own a feature store paved road.
- The roadmap horizon (12 months): what other ML systems are likely to appear, and would they share features?

This section is the reviewer's ground truth — every subsequent decision cites back to it.

### 2. Walk the three adoption signals (≈ 30 min)

For each signal from chapter 03:

- **Signal A — Reuse.** Are there (or will there be within 12 months) multiple consumers of the same features? Cite specific models and features.
- **Signal B — Latency budget.** Do any consumers require online serving with a latency budget below a few hundred milliseconds? Cite the specific SLOs.
- **Signal C — Ownership.** Is there a team — yours or a peer platform team — that will operate the feature store for the next 24+ months? Name the team, and if it is a peer platform team, note the hand-off contract (see [mod-308]).

For each signal, answer **present / absent / at-risk** and defend it with one or two sentences.

### 3. Name the honest cost ledger (≈ 30 min)

Chapter 03's "honest ledger" — platform, adoption, latency, consistency — restated for your specific scenario. Do not use generic language: put numbers or ranges where you can (peer platform on-call cost per week; migration effort per feature; expected p99 hit for a feature-store fetch on the paths you care about). Order of magnitude is fine.

If any cost is unknown at your altitude, name it as an *open question* with a named owner and a date to close it.

### 4. Compare against the ad-hoc discipline (≈ 30 min)

If the answer is not-adopt, you owe a concrete plan for the ad-hoc alternative. From chapter 03:

- A single feature-computation library shared between training and serving.
- A shared feature-vector schema (protobuf, Pydantic, or equivalent).
- Point-in-time semantics enforced in SQL, CI-tested against a known-leakage fixture.
- Named per-feature offline/online parity tests.

Sketch what the code and test surface would look like. This is what makes the not-adopt answer L30 rather than L20: you name what the discipline is, not just what you are avoiding.

### 5. Land on one of three answers (≈ 15 min)

- **Adopt.** Consume a paved-road platform or stand one up. Include a rollout plan (which model migrates first, over what period, at what parity gate).
- **Defer.** Ad-hoc for now with the discipline of step 4; adoption re-evaluated when a named condition is met (e.g., "when the second model consuming user-30d aggregates lands," "when p99 <200 ms serving is required").
- **Do not adopt.** Ad-hoc permanently, with an explicit rationale. This is the answer for genuinely single-model, batch-only systems where the ledger clearly loses.

Do not include an "adopt someday" answer. Someday is not a plan.

### 6. Sketch the non-goals (≈ 15 min)

Chapter 06's non-goals section, applied to this decision. Examples of the shape you are aiming for:

- "We are not building a feature-store platform inside our team. If the paved-road option is missing X, we file a contribute-back RFC — see [mod-308]."
- "We are not migrating the legacy feature service to the new store in v1. Legacy features stay ad-hoc; new features go on the paved road."
- "We are not addressing streaming features in v1. Streaming feature adoption is scoped for RFC-XXX+1."

## Deliverable

A single Markdown document `feature-store-adoption-<scenario>.md`, roughly 2 pages, structured as:

- Header — scenario, date, author, systems in scope.
- **1. Baseline.**
- **2. Adoption signals** — A/B/C, with a two-sentence defence each.
- **3. Cost ledger.**
- **4. Ad-hoc alternative** — the discipline sketch.
- **5. Decision** — adopt / defer with condition / not adopt, with a one-paragraph rationale.
- **6. Non-goals.**

## Acceptance criteria

- [ ] All three adoption signals answered with specifics (models, features, teams named), not generic present/absent.
- [ ] The cost ledger includes at least one item with a rough number or range, not just a qualitative label.
- [ ] The ad-hoc alternative is described concretely enough that a mid-level engineer could implement the discipline from your memo.
- [ ] The decision is one of the three named options — no "adopt someday."
- [ ] Non-goals present and specific.
- [ ] The memo is *at most two pages*.

## Stretch goals

- **Two-scenario contrast.** Do the exercise twice — once on a scenario where the signals support adopting, once on one where they do not. Note in a short paragraph which parts of the memo change and which are stable. This is the fastest way to internalise which signals are load-bearing.
- **Talk to the peer platform team.** If your organisation has a peer platform track (`ai-infra-ml-platform-learning` analogue), take the memo to their tech lead as a real hand-off conversation (see [mod-308]). Note the questions they raise that you did not anticipate.
- **Contribute-back RFC sketch.** If your answer is "adopt but the paved-road option is missing X," sketch the contribute-back RFC in a paragraph. This previews the collaboration craft covered in [mod-308].
- **Feed into the RFC.** Reuse this memo as the seed for section 4.2 (feature substrate) of the RFC in the paired project (`project-301-ml-system-design-portfolio`).
