# exercise-02: Build-vs-adopt tradeoff doc

**Estimated effort:** 3 hours

## Objective

Pick a specific gap between your team's ML requirement and the paved-road platform, walk all four paths from chapter 02 §1 (*bend*, *escalate*, *build-in-house with migration*, *contribute back*) against the five-factor rubric of §2, and author the **tradeoff document** that a peer L30 and the platform team could sign. Include the recommendation, the defence, and — the load-bearing part — the *exit plan* that keeps the decision reversible.

The output is a Markdown document, 2–3 pages, titled `build-vs-adopt-<gap>.md`, structured per chapter 02 §4. It is the artefact that turns a build-vs-adopt call from "the loudest voice in the room" into a documented, reviewed, and reversible decision.

This is the L30 tell that separates "we decided to build it ourselves" from "we walked the four paths, considered the tradeoffs, picked one with an explicit exit plan, and had it reviewed by a peer L30 and the platform team." An L20 declares a build decision by conviction; an L30 defends it in writing against the alternatives.

## Prerequisites

- Read chapter 02 (`02-build-vs-adopt-decisions.md`) — the four paths, the five factors, the decision matrix, the tradeoff-doc skeleton, the migration plan discipline, escalating correctly, the contribute-back trap.
- Read chapter 01 (`01-paved-road-consumption.md`) — the paved-road inventory is the input to this exercise; the gap you pick is one of the ✗ or Partially cells from exercise 01.
- Skim chapter 03 §1–§2 — the contribute-back option is one of the four; you need to know what it entails before you can compare it fairly.
- Skim [mod-307 chapter 01 §5](../../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md) — dependency-SLI composition, so your "technical fit" section can reason about SLO composition when the gap is a reliability gap.
- Bring the paved-road inventory and the quarterly consumer report from exercise 01. The specific gap you pick in this exercise should be one of the pain points from the report.

## Pick your gap

Pick **one** of the following, or bring your own (G5). Where the exercise names a specific team (T1–T4 from exercise 01), adapt to your team of choice.

- **G1 — Sub-hour freshness on a single feature-view.** T1's recommendation feature needs sub-15-minute freshness on the user's active-session feature; the platform's feature store supports hourly batch. This is the worked example of chapter 02 §8; the exercise is to *fully* walk it (not just skim it) for your team and your specific numbers.
- **G2 — Vector-index freshness / re-index cadence.** T3's ticket-triage LLM needs a nightly-refreshed vector index of the ticket-KB; the platform's serving stack has no vector-index primitive at all — teams roll their own. The gap is ~90 %, which puts §3's very-large-gap row in play.
- **G3 — Per-tenant model isolation on the serving path.** T2's fraud team needs a serving-path primitive that isolates per-tenant model versions (large customers get their own model artefact); the platform's serving stack assumes a single model per endpoint. The gap is medium-to-large.
- **G4 — Sub-minute experiment ramp step size.** T4's ad team needs experimentation-platform ramps at 1 % steps every 5 minutes for auction-path features; the platform's experimentation service supports 10 % steps every 6 hours. Gap is medium; reversibility is high (the requirement could bend to slower ramps if the cost of not-bending is small).
- **G5 — Bring-your-own.** A real or planned gap from your team's inventory or quarterly report. Anonymise anything sensitive; add a preamble covering the requirement, the platform's current offering, and the deadline pressure.

## Steps

### 1. Context and gap statement (≈ 20 min)

Author the top of the doc — the context and gap sections. Cover:

- **Team and feature.** Which team, which feature (or which upcoming feature) the gap blocks. Link to the feature's SLO doc (mod-307 chapter 01 §6) if it exists.
- **Deadline / cadence pressure.** What makes this time-bound. "The Q3 ranking-v4 launch requires this by 2026-09-15" is a real deadline. "Sometime this year" is not.
- **The requirement.** Specific and measurable. "Feature-view freshness p99.5 ≤ 15 minutes over 28 days" is a requirement; "faster features" is not.
- **The platform's current offering.** Specific — the SLI + target + window the platform publishes, not "it's slow." If the platform doesn't publish the number, name that gap too (this may be a feedback item on top of the primary gap).
- **The gap in %.** A rough coverage estimate (chapter 02 §2.1). A 5 % gap and a 90 % gap warrant different responses.

### 2. Walk path 1 — bend (≈ 20 min)

For the *bend* path (change your requirement so the paved road covers you):

- **Would this work?** What would the requirement become? Would the feature still ship? Does the product-side accept the bent requirement?
- **Cost.** Engineering effort to re-scope; opportunity cost of the reduced feature.
- **Recommendation on this path.** Under what conditions is bend the right answer? What are the deal-breakers?

Bend is often *rejected quickly* — but not without being considered. The exercise requires you to walk it explicitly and record the rejection reason.

### 3. Walk path 2 — escalate (≈ 20 min)

For the *escalate* path (ask the platform team to add it):

- **Feasibility.** Has the platform team already stated a priority for this? Is it on the roadmap? Have you filed the ticket / raised the quarterly-report item? What was the response?
- **Timeline.** Best-case and worst-case quarters until the platform ships. If the platform team has not committed, name the *inference* — "based on the platform team's stated capacity and their published roadmap, we estimate best-case Q1 next year, worst-case Q4 next year."
- **Does escalate alone unblock the deadline?** If yes, escalate is the recommended path. If no, escalate is a *complementary* path that runs alongside another chosen path (the durable fix while a short-term path handles the deadline).

The escalate path is *almost always* pursued in parallel with any other path chosen. Note the parallel-track explicitly.

### 4. Walk path 3 — build in-house with migration (≈ 30 min)

For the *build-in-house-with-migration* path (interim on-team build, migrated off when the platform ships):

- **Scope.** What you would build. The interfaces, the on-call surface, the storage / compute footprint.
- **Initial cost.** Engineer-weeks. Broken down by design, implementation, integration, testing, initial on-call.
- **Long-run cost.** The on-call load per quarter; the migration cost when the platform ships; the risk of the interim becoming permanent-by-default if the migration trigger never fires.
- **Interface-match strategy.** How you would design the in-house build's API to match the eventual platform API, so the migration is import-only.
- **Migration trigger.** The specific platform-team milestone that starts your migration. "Chronon sub-hour freshness ships in beta" is a trigger; "eventually" is not.
- **The signed commitment.** Who signs the commitment to migrate on the trigger — team TL + platform-team liaison. Named individuals.
- **Recommendation on this path.** Under what conditions is build-in-house the right answer? What is the deal-breaker (usually reversibility — if there's no plausible migration trigger, the "with migration" qualifier is fiction and this is a one-way door).

### 5. Walk path 4 — contribute back (≈ 30 min)

For the *contribute-back* path (RFC + PR on the platform side):

- **Scope.** What you would contribute. The RFC's headline change.
- **Cost.** Engineer-weeks including the platform-quality bar, the review latency, the platform-team on-call training. Typically 2–4× the equivalent in-house build.
- **Sponsor.** Named person on the platform team. If not named, this is a *precondition* — you need the sponsor conversation from chapter 03 §2 before you can seriously consider this path.
- **The three tests from chapter 03 §7.** Is the problem fleet-relevant? Is the platform team resourced to review? Is your team resourced to maintain through the review cycle? Answer each.
- **Timeline.** Best-case and worst-case quarters until merged and deployed. Compare to the deadline.
- **Recommendation on this path.** Under what conditions is contribute-back the right answer? What is the deal-breaker (usually deadline pressure or missing sponsor)?

### 6. The decision (≈ 20 min)

State the recommendation and defend it:

- **Path.** One of the four, or a *composite* (e.g., "build-in-house with migration, escalate in parallel for the durable fix").
- **Why.** A three-bullet defence weighing the five factors (§2). The bullets are not "the path is technically best" — they are "the path is best given <reversibility>, <blast radius>, and <deadline / opportunity cost>."
- **Why not each of the other three.** One or two sentences per rejected path, referencing the deal-breaker named in the earlier step.
- **The decision matrix row.** Which row from §3 does this decision map to? If it maps to no row cleanly, name the closest row and why the specific circumstances warrant the deviation.

### 7. The exit plan (≈ 20 min)

The most-mis-handled section of the tradeoff doc. Author explicitly:

- **How you would walk back the decision if it goes wrong.** For each of the recommended path's failure modes, the walk-back action.
- **The review date.** The specific date the tradeoff doc is re-opened and the decision re-evaluated. Typical: 2 quarters after the decision, or the migration trigger — whichever is sooner.
- **The success criteria at review.** What has to be true at review time for the current path to remain the right choice. What has to be true for a re-decision.
- **The permanence acknowledgment (if applicable).** If the decision is a one-way door (usually the contribute-back or the "build in-house without a plausible migration trigger" case), name it. "The exit plan is: we accept this is permanent" is a valid exit plan — but naming it forces the recognition.

### 8. Reviewers and sign-off (≈ 15 min)

Fill out the reviewers section:

- **Team peer L30.** Named. Their sign-off is not required but their review is.
- **Platform team liaison.** Named. Their sign-off *is* required — the tradeoff doc is a decision that affects the platform relationship, so the platform team gets a say.
- **Approver.** Your team's TL or manager. The person who commits the team's engineering time.
- **Sign-off dates.** Populated when the reviewers sign; blank in the initial draft.

### 9. Peer review (≈ 15 min)

If this exercise is being done in a group, trade tradeoff docs with a peer. The reviewer walks:

- Are all four paths actually considered, or are one or two dismissed in a sentence?
- Does the decision matrix row match, or is the deviation named?
- Is the exit plan concrete, or is it "we'll figure it out"?
- Does the recommendation defend against the *strongest* rejected alternative, not the weakest?
- Would the platform-team liaison sign off on this doc without needing a conversation to fill in blanks?

The peer's marginal notes go back into the doc as "reviewer note — <date> — <reviewer>."

## Deliverable

A single Markdown document, 2–3 pages, titled `build-vs-adopt-<gap>.md`, structured as:

- Header (title, author, sponsor / liaison, reviewers, approver, status, dates).
- Context (§1).
- Gap statement (§1).
- Path 1: bend (§2).
- Path 2: escalate (§3).
- Path 3: build in-house with migration (§4), including the migration plan section (chapter 02 §5).
- Path 4: contribute back (§5), including the three tests.
- Recommendation and defence (§6).
- Exit plan (§7).
- Reviewers and sign-off (§8).

The document is written to be *decided on*, not archived. A peer L30 reading it should be able to say "yes, sign" or "no, address these two things" without needing a conversation to fill in blanks. The platform team's liaison should be able to say "yes, we agree with the framing" or "no, our roadmap disagrees with your escalate-path assessment."

## Acceptance criteria

- [ ] All four paths are walked, not just the recommended one — with a design sketch, a cost estimate, and a recommendation-conditions statement each.
- [ ] The gap is stated as a specific requirement vs. a specific platform offering, with the gap-size % named.
- [ ] The "build in-house with migration" path has a *named migration trigger* (a specific platform-team milestone), not "when the platform ships something."
- [ ] The "build in-house with migration" path has a *signed commitment* row — named individuals on both sides — even if the sign is pending in the draft.
- [ ] The "contribute back" path has the three chapter-03 §7 tests answered, and either names the sponsor or notes the sponsor conversation as a precondition.
- [ ] The recommendation names the decision-matrix row (§3 of chapter 02) or explicitly names the deviation.
- [ ] The recommendation's defence weighs the five factors (technical fit, reversibility, blast radius, opportunity cost, three-year TCO), not just the two obvious ones.
- [ ] The exit plan is concrete — a walk-back action per failure mode, a specific review date, specific success criteria at review.
- [ ] If the decision is a one-way door, the permanence acknowledgment is explicit.
- [ ] Reviewers and approver are *named individuals*, not "the team."
- [ ] The doc cross-references the paved-road inventory from exercise 01 (this gap is one of its rows).

## Stretch goals

- **The three-year TCO spreadsheet.** For each of the four paths, a three-year total-cost estimate broken down by year — initial build, on-call load, migration cost, evolution cost. The number changes the recommendation more often than intuition suggests; producing the spreadsheet forces the honest accounting.
- **The dependency-SLI composition check.** If the gap is a reliability gap (freshness, latency), compute the composed SLO ([mod-307 chapter 01 §5](../../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md)) under each path. Which paths let your feature's SLO be tight enough? Which don't?
- **The blast-radius map.** For the "contribute back" path specifically, draw a table of every consumer team on the platform primitive being changed, and estimate their exposure to the change. Fleet-scale blast radius makes the RFC's review process heavier — the map is what forces the recognition.
- **The escalation-history appendix.** For the escalate path, an appendix of every ticket / quarterly-report line-item / previous-RFC on this gap, with the platform team's response and the elapsed time. History matters — a gap the platform team has punted on for four consecutive quarters warrants a heavier response than a gap raised for the first time.
- **The pre-RFC sponsor pitch.** If the recommendation lands on contribute-back, draft the chapter-03 §2 sponsor-conversation pitch as an appendix. Feeds exercise 03 directly.
- **The build-in-house PR review checklist.** If the recommendation lands on build-in-house-with-migration, draft the PR-review checklist that will keep the interim tool from feature-creeping (chapter 02 §5). The checklist is what the team's PR reviewers will point at when someone proposes adding a feature to the interim.
- **Peer review with the platform-team liaison.** Trade the tradeoff doc with the actual (or roleplayed) platform-team liaison. Their review often surfaces "the platform team disagrees with your escalate-path assessment because our roadmap has X" — which is the highest-value feedback the doc can receive.
