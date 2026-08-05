# exercise-03: Contribute-back RFC to a peer platform team

**Estimated effort:** 4 hours

## Objective

Draft a **full contribute-back RFC** to a peer platform team, following the structure of chapter 03 §3. The RFC is not a skeleton — it is the artefact a real platform-team reviewer could open, read in one sitting, and either say "yes, sign," "no, and here's why," or "yes if you address these three things." Every section is populated with real content, at the altitude the platform team will read it.

The output is a Markdown document, 4–6 pages, titled `rfc-<slug>.md`. This is the third and heaviest of the three module deliverables — the paved-road inventory named the pain, the tradeoff doc chose contribute-back as the path, and the RFC is the actual proposal that lands on the platform team's desk.

This is the L30 tell that separates "we asked the platform team to fix X" from "we drafted an RFC that the platform team's staff engineer signed and their team implemented." An L20 writes tickets; an L30 writes RFCs that ship.

## Prerequisites

- Read chapter 03 (`03-contribute-back-rfcs.md`) end-to-end — what an RFC is, the sponsor conversation as precondition, the structure, the platform-team-altitude discipline, alternatives-considered as load-bearing, migration and deprecation as first-class sections, the review cycle, and rejection as a valid outcome.
- Read chapters 01 and 02 — the paved-road inventory is the source of your problem-statement evidence; the tradeoff doc is what determined that contribute-back is the right path.
- Skim Gergely Orosz's [*Software Engineering RFCs*](https://www.pragmaticengineer.com/rfcs-and-design-docs/) — the industry-standard treatment of RFC culture.
- Skim the [Rust RFC template](https://github.com/rust-lang/rfcs/blob/master/0000-template.md) — the specific structure the "alternatives considered" and "drawbacks" sections in this exercise lean on.
- Skim [Google's public API deprecation policy](https://cloud.google.com/terms/deprecation) — the industry standard for what a *published* deprecation policy looks like.
- Bring the tradeoff doc from exercise 02 (or an equivalent decision artefact). This RFC's problem statement borrows its evidence from that doc; if the tradeoff doc doesn't exist, the sponsor conversation of §2 below is your alternative preparation.

## Pick your RFC

Pick **one** of the following, or bring your own (R5). The RFCs correspond to the gaps from exercise 02 for continuity:

- **R1 — Per-feature-view freshness SLOs in the feature-store SDK.** Corresponds to G1 in exercise 02. The RFC of chapter 03 §9's worked skeleton. The exercise here is to author the *full* RFC, not just the skeleton — every section populated.
- **R2 — Vector-index primitive in the serving platform.** Corresponds to G2. The RFC introduces a new module (a vector-index primitive with lifecycle, freshness contract, and consumer-team SDK) — a much larger RFC that is at the border of "big enough that it needs to be a multi-quarter effort."
- **R3 — Per-tenant model isolation on the serving path.** Corresponds to G3. The RFC changes the serving-platform's assumption of a single model per endpoint; touches the model registry, the routing layer, and the observability surface.
- **R4 — Fine-grained experimentation-platform ramp steps.** Corresponds to G4. The RFC extends the experimentation platform's ramp-step schema and the underlying orchestration to support minute-granular ramp cadences with guardrail-integration parity.
- **R5 — Bring-your-own.** A real or planned contribute-back from your team. Anonymise anything sensitive; add a preamble covering the problem, the platform-team relationship, and the deadline pressure (if any).

## Steps

### 1. The sponsor conversation (≈ 30 min)

Before writing the RFC, write out the *sponsor conversation* from chapter 03 §2 as a pre-work section:

- **The two-sentence problem statement.** In the platform team's vocabulary.
- **The evidence.** The numbers from the quarterly consumer report and the tradeoff doc. Not "many teams," but "four teams; ~120 engineer-hours per quarter of workaround cost."
- **The directional proposal.** One paragraph. What you'd propose the RFC do. Not the full design — the shape.
- **The ask.** "Are you willing to sponsor an RFC on this? If so, when should I circulate a draft?"

If you are doing the exercise alone, imagine the sponsor conversation and write the imagined response — one of the three outcomes from chapter 03 §2. If the imagined response is "yes, problem — no, RFC" or "no, not a fit," *use that as the exercise*: write a short "why I stopped here" note instead of an RFC, and update the tradeoff doc accordingly. Not every problem should become an RFC; recognising when to stop is part of the discipline.

If the imagined response is "yes, sponsor," proceed to step 2. Record the sponsor's name (real or roleplayed) at the top of the RFC.

If this exercise is being done in a group, one participant roleplays the sponsor. The sponsor conversation happens live — 15 min, over a synchronous call or a Slack thread — and its outcome is recorded.

### 2. RFC header and summary (≈ 20 min)

Author the header and the summary section (chapter 03 §3):

- **Title.** Verb phrase — "Add X to Y," "Deprecate X in favour of Y," "Introduce X." The title is scannable in a table of contents.
- **Author(s), sponsor, reviewers, approver.** Named individuals, not "the ml-platform team." The reviewers list is 3–5 people; the approver is 1 person.
- **Status.** `draft` at the start.
- **Created / last updated.** Dates.
- **Summary.** Two or three sentences. What the RFC proposes, at a level a reviewer can decide whether to read the rest. The summary is the most-read section — invest.

### 3. Motivation (≈ 30 min)

Author the motivation section:

- **Problem.** In the reviewer's vocabulary. What consumers observe. Why it matters. **Quantified evidence** — the paragraph is not "we think X is a problem" but "consumers X, Y, Z have hit this in the last N quarters, aggregate cost ~M engineer-hours."
- **Why now.** The trigger. A deadline, an incident, a compounding cost. "Why this quarter, not next" is the load-bearing question.
- **Goals.** Bulleted. Measurable where possible. "Freshness SLO field in manifest; fetch-time SLI available; scheduler-level guarantee" is a goals list; "make freshness better" is not.
- **Non-goals.** What this RFC deliberately does *not* address. This scoping is 30 % of the work; a well-scoped RFC has more non-goals than goals.

### 4. Proposal (≈ 40 min)

Author the proposal section. This is where the design lives, but *not* where the RFC spends its bulk — the alternatives-considered and impact sections often equal the design section in length.

- **Design.** The specific change. API shape, interface signatures, YAML manifest schema, wire format — at a level a reviewer can imagine the implementation. Include diagrams if the change is architectural.
- **API changes.** The new methods / fields / manifests, with their exact signatures. If existing API surface changes, name the SemVer implication.
- **Data model / schema changes.** If applicable. Migration flagged.
- **Rollout plan.** How the change ships. Feature flag? Opt-in first? Version bump? Named milestones with dates.

Two failure modes to avoid at this stage:

- **Over-specification.** A three-page implementation walkthrough that pins the platform team's implementation choices. The RFC's design section is API-first, implementation-second. Leave implementation flexibility for the platform team unless a specific implementation is load-bearing for the RFC's argument.
- **Under-specification.** A single paragraph that says "add a freshness field." The reviewer cannot decide. The design section is *enough* detail that the reviewer can imagine the shape and reason about impact.

### 5. Alternatives considered (≈ 40 min)

The load-bearing section (chapter 03 §5). Author **at least three alternatives**, including *do nothing* and *solve it downstream*:

- **Alternative 1: Do nothing.** What happens if the platform ships nothing. What is the ongoing cost. Why it is rejected.
- **Alternative 2: Solve it downstream (in the consumer's build-in-house layer).** The "build in-house" path from exercise 02, considered here as an *alternative to the RFC*. Named, costed against the RFC's cost, and rejected (or if it's not rejected, the RFC should not exist — the tradeoff doc should have landed on build-in-house).
- **Alternative 3: A different platform-side design.** A different shape the RFC's proposal could take. A weaker version, a heavier version, a differently-scoped version. Rejected with a named deciding factor.
- **(Optional) Alternative 4+.** Any other real alternative the sponsor conversation or a peer review surfaced.

Each alternative gets the same rigor as the main proposal — a short summary, its design, its tradeoffs, why it was rejected. A straw-man alternative (a weak alternative dismissed in one paragraph) is a red flag; a reviewer will notice and read it as sloppy. Every alternative is treated seriously; the reject is on merits.

### 6. Impact (≈ 30 min)

Author the impact section — often the section reviewers scrutinise hardest:

- **Compatibility.** Backwards-compat implications. Which consumers break, when. If none, name that explicitly.
- **Migration.** How existing consumers move to the new shape. Automated (codemod, schema translator), semi-automated (lint), or manual (docs + Slack ping). Migration cost per consumer in engineer-hours. Migration timeline (the window during which old and new both work). Migration owner (platform team providing help, or each consumer team on their own).
- **Deprecation.** What is being deprecated. The specific notice period. The deprecation signal (warning logs, docs, linter, telemetry). The sunset date and sunset action.
- **On-call / support.** Who owns the on-call. What runbook lands. What alerting the platform's SLO doc needs. Reference [mod-307 chapter 04](../../mod-307-ml-reliability-slos/04-ml-incident-response-and-postmortems.md) for the runbook shape.
- **Cost.** Compute, storage, network, engineer-hours. Both the platform team's cost and the fleet's cost.
- **Blast radius.** Who is affected if this ships and turns out to be wrong. The rollback plan.

The migration and deprecation subsections are non-negotiable for a contribute-back RFC. "TBD" in either is a reviewer-side blocker — the platform team will not approve an RFC that leaves them holding a migration or deprecation burden that has not been thought through.

### 7. Risks, open questions, and references (≈ 20 min)

Author the tail sections:

- **Risks.** Three to five risks worth naming. For each, the mitigation. "Scheduler queue starvation if too many consumers opt-in to aggressive SLOs. Mitigation: per-team quota, escalation to sponsor for exceptions." is a real risk-and-mitigation; "unforeseen bugs" is not.
- **Open questions.** Explicitly named. The reviewer can respond to these. "Does the SLO apply to online-only feature-views, offline-only, or both?" is an open question; "how do we handle edge cases" is not.
- **References.** Links to the sponsor conversation notes, prior tickets, the quarterly consumer report, related RFCs, external references. Every citation is a specific link — no "our internal docs."

### 8. Decision log (≈ 10 min)

Add an empty decision-log section at the bottom:

```markdown
## Decision log
- <date> — <status change or reviewer comment>
- ...
```

Populated during review; empty in the initial draft. The decision log is what makes the RFC auditable years later — someone reconstructing "why did we build the freshness SLO this way, not that way" reads the decision log and finds the specific reviewer comment that tipped the design.

### 9. Peer review (≈ 30 min)

If this exercise is being done in a group, trade RFCs with a peer. The reviewer walks:

- **Sponsor test.** Would the named sponsor open this RFC and recognise it as the thing they agreed to sponsor? Or has it drifted from the sponsor conversation?
- **Reviewer-altitude test.** Is the RFC written to the platform team's altitude (consumers, API-first, fleet-scale reasoning)? Or is it written to the author's own team?
- **Alternatives-considered test.** Are the three alternatives real, or are two of them straw men?
- **Migration-and-deprecation test.** Are both concrete, or does either say "TBD"?
- **Approval-readiness test.** If the reviewer were the approver, could they say yes / no / conditionally-yes from the RFC alone? Or would they need a synchronous meeting to fill in blanks?

The reviewer's marginal notes go into the RFC as `<!-- reviewer note: ... -->` comments. Some become updates before the RFC is circulated; some remain as reviewer artefacts.

### 10. (Stretch) Prepare for the review cycle (≈ 20 min)

For the RFCs that go on to be circulated, sketch the review-cycle plan (chapter 03 §7):

- **Round 1 circulation list.** Sponsor + platform-team domain reviewers + 1–2 consumer teams. Named.
- **Round 2 circulation.** The platform-team review meeting or async equivalent.
- **Ping cadence.** The weekly-nudge schedule you'll use to keep the review moving.
- **Escalation.** The named person on the platform team to escalate to if a round takes > 2 weeks.

## Deliverable

A single Markdown document, 4–6 pages, titled `rfc-<slug>.md`, structured as:

- **Pre-work section** (kept at top or in appendix): the sponsor conversation notes (§1).
- Header (§2).
- Summary (§2).
- Motivation — problem, why now, goals, non-goals (§3).
- Proposal — design, API changes, data model / schema changes, rollout plan (§4).
- Alternatives considered — at least three, including do-nothing and solve-it-downstream (§5).
- Impact — compatibility, migration, deprecation, on-call, cost, blast radius (§6).
- Risks (§7).
- Open questions (§7).
- References (§7).
- Decision log (§8).

The RFC is written to be *reviewed*, not archived. A named reviewer should be able to open it cold and, within 20 minutes of reading, form an opinion strong enough to comment on. A named approver should be able to sign it, reject it, or condition it on specific changes — from the RFC alone, without a synchronous meeting.

## Acceptance criteria

- [ ] The RFC has a named sponsor (real or roleplayed), and the sponsor-conversation pre-work is documented.
- [ ] Reviewers and approver are named individuals, not "the platform team."
- [ ] The summary is 2–3 sentences and states what the RFC proposes.
- [ ] The problem statement has *quantified* evidence — a number of affected consumers, engineer-hours, dollars, or incidents — not "many teams."
- [ ] The proposal is API-first — the SDK / manifest / wire-format changes come before any internal-implementation walkthrough.
- [ ] Non-goals are explicitly named; the RFC has at least as many non-goals as goals.
- [ ] At least three alternatives are considered, including *do nothing* and *solve it downstream*, each with its own design sketch and rejection reason.
- [ ] Migration is concrete — a mechanism (automated / semi-automated / manual), a cost per consumer, a timeline, an owner. No "TBD."
- [ ] Deprecation is concrete — what's deprecated, the notice period, the signal, the sunset date and action. Or the RFC is explicitly additive-only and says so.
- [ ] On-call ownership is named, with the runbook and SLO-doc updates flagged as follow-ups.
- [ ] Blast radius is named — who is affected if this ships and turns out to be wrong.
- [ ] Risks list is 3–5 items with mitigations; each risk is specific.
- [ ] Open questions are stated as answerable questions, not as "handwaves."
- [ ] References cite the source material — the sponsor conversation notes, the tradeoff doc, the quarterly consumer report, prior tickets, external references from `resources.md`.
- [ ] The decision log section exists and is empty (populated during review).
- [ ] The RFC is written to the platform team's altitude, not the author's team's — consumers, API-first, fleet-scale reasoning.

## Stretch goals

- **Live review round.** Recruit a peer to play the platform-team domain reviewer. Get one round of marginal comments. Update the RFC. Note in the decision log what changed and why.
- **The PR that implements it.** Draft the PR description (or the PR skeleton) that would follow the RFC. What's the branch strategy, the test plan, the rollout gate? The PR description is the second decision artefact after the RFC.
- **The migration codemod.** For migration paths marked as "automated," sketch what the codemod does — the input pattern, the output pattern, edge cases. A real codemod sketch is what makes "automated" credible.
- **The deprecation telemetry.** For deprecation paths, sketch the telemetry that will report "consumer X is still using the deprecated method, called N times/day." Without this, the sunset date is a gamble.
- **The composed-SLO argument.** If the RFC changes an SLI or SLO on a primitive, compute the composed downstream SLO impact ([mod-307 chapter 01 §5](../../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md)). Consumer teams reading the RFC will thank you for doing the math once.
- **Rejection walk-through.** As an alternate exercise, imagine the RFC is rejected. Author the *response* — the withdraw-and-re-frame, the re-scope, or the path-change back to build-in-house. Rejection is a valid outcome; practising the response is part of the discipline.
- **RFC-to-implementation timeline.** Author the plan for the two months after the RFC is approved — who writes the code, who reviews, what the rollout looks like, when the RFC's status flips to `implemented`. RFCs that get approved and then stall are a common failure; the implementation timeline is what prevents it.
- **Feed the paired project.** Attach this RFC as the "platform contribution" section of a paired project. The RFC should be *submitted*, not archived.
- **Peer review with a platform-team member.** The single most valuable review — trade the RFC with someone on the actual (or roleplayed) platform team. Their reaction will pinpoint the sections where the RFC drifts off the platform's altitude, the alternatives you missed, or the migration cost you under-estimated.
