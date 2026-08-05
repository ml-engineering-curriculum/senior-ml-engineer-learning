# Writing a contribute-back RFC a peer platform team can act on

## Motivation

Chapter 02 established the four paths and named *contribute back* as the one whose product is a piece of platform code with your name on it and the platform team's on-call rotation attached to it. This chapter is about the RFC that makes that path work.

The RFC (Request For Comments) is the *decision-making artefact* the platform team uses to say yes, no, or "yes if you change these three things." A well-written RFC is the difference between a contribute-back that ships in a quarter and one that sits in review for six months and dies of quiet neglect. RFCs are not new — the IETF invented [the format in 1969](https://www.rfc-editor.org/info/rfc1) — and their modern engineering-org form is descended from the [Rust RFC process](https://rust-lang.github.io/rfcs/) and Google's design-doc culture (Yonatan Zunger, *[The friendly design doc](https://medium.com/@yonatanzunger/the-friendly-design-doc-2c96b8ba0842)*). The load-bearing external reference is Gergely Orosz's *[Software Engineering RFCs](https://www.pragmaticengineer.com/rfcs-and-design-docs/)*.

The claim of this chapter: writing an RFC a peer platform team can act on is not the same as writing a design doc for your own team. The audience is different (a team you do not manage, do not report to, and whose priorities you do not control), the stakes are different (they will own the code you wrote for years), and the format is different (heavier on alternatives-considered, deprecation, and migration; lighter on the internal-team detail your own team already has context on). The RFC that works is the one that meets the platform team where they are.

## §1 — What an RFC is (and is not)

An RFC is a written proposal that:

- Names a *specific* change to a system, in enough detail that the reviewers can decide yes / no / conditionally-yes.
- Frames the *problem* the change solves — with evidence — before naming the solution.
- Considers *alternatives*, including "do nothing," so the reviewers can see the decision surface.
- Names the *impact* — on migration, deprecation, on-call, cost, blast radius.
- Names the *reviewers and approvers* — the specific people whose sign-off is required.
- Lands in a shared, versioned location (repo, wiki, docs site) so the decision is auditable years later.

An RFC is *not*:

- A slide deck. Decisions written into slides are un-reviewable at the speed reviewers can read, un-searchable after the fact, and un-linkable from a PR.
- A Slack thread. Same problems, plus the decision disappears when the channel archives.
- A design doc for the sponsor's team. The RFC is written to *another* team, so it carries context that team's internal design docs don't.
- The code. The PR that implements the RFC follows the RFC; landing the PR first and asking the platform team to review it is asking them to reverse-engineer the decision.

The senior discipline is to notice which of these three "not-RFCs" you are drifting toward and stop. If you find yourself pitching in a meeting, or arguing on Slack, or opening a PR without the RFC — write the RFC first.

## §2 — Before the RFC: the sponsor conversation

RFCs that show up cold to the platform team fail. The precondition to a successful RFC is a *sponsor* on the platform team who has already agreed the problem is real and is invested in seeing a solution reviewed.

The sponsor conversation is a small, private, low-stakes talk with the platform-team liaison — often the team's TL, staff engineer, or product manager — where you preview:

- **The problem statement.** In two or three sentences. What you observe, why it matters, what your team has tried.
- **The evidence.** Numbers: SLO gaps, quarterly-report line-items, ticket-log density, engineer-hours-per-quarter of workaround cost. Chapter 01 §5.2's quarterly consumer report is the source material.
- **The *directional* proposal.** Not the whole RFC — a paragraph. "We think a per-feature-view freshness SLO in the SDK, backed by a small change to the Chronon scheduler, would solve this." Enough that the sponsor can react.
- **The ask.** "Are you willing to sponsor an RFC on this? If so, when should I circulate a draft?"

Three outcomes to expect:

- **Yes, sponsor.** The sponsor agrees the problem is real and is willing to shepherd the RFC through review. You proceed to write. This is the case chapter 03 focuses on.
- **Yes, problem — no, RFC.** The sponsor agrees the problem is real but doesn't think an RFC is the right vehicle — maybe the fix is small enough to be a ticket + PR, maybe the roadmap already has it. Course-correct: file the ticket, or take the "build in-house with migration" fallback from chapter 02 §5.
- **No, not a fit.** The sponsor doesn't agree the problem is fleet-relevant, or doesn't think the platform is the right layer. This is a *good* outcome — it saves you writing an RFC that would have been rejected. Course-correct to bend or build-in-house.

The senior discipline is that the sponsor conversation *precedes* the RFC. If you write the RFC first and then look for a sponsor, you have (a) wasted the work if there's no sponsor and (b) written it to a general audience that doesn't exist. Ryan Peterman's *[How to get engineering buy-in](https://blog.pragmaticengineer.com/how-to-get-engineering-buy-in/)* walks the sponsor pattern in depth.

## §3 — The RFC structure

There is no single canonical structure — every mature engineering org has its own template — but the load-bearing sections are the same. The structure below composes elements from the Rust RFC process, Google-style design docs, and the Squarespace / Uber / Airbnb internal templates that have become the de-facto industry pattern.

```markdown
# RFC: <title — verb phrase, e.g., "Add per-feature-view freshness SLOs to Chronon SDK">

- Author(s): <name(s) + team>
- Sponsor: <name on platform team>
- Reviewers: <named individuals, not "the platform team">
- Approver(s): <name(s) with authority to say yes>
- Status: draft | in review | approved | rejected | withdrawn | implemented
- Created: <date>
- Last updated: <date>

## Summary
<Two or three sentences. What the RFC proposes, at a level a reviewer can decide whether to read the rest.>

## Motivation
### Problem
<The problem in the reviewer's vocabulary. What consumers observe. Why it matters. Quantified evidence.>

### Why now
<Why this quarter, not next. The trigger — a deadline, a re-org, an incident, a compounding cost.>

### Goals
<The specific outcomes this RFC is trying to achieve. Bulleted, measurable where possible.>

### Non-goals
<What this RFC deliberately does NOT address. Scoping is 30 % of the work.>

## Proposal
### Design
<The specific change. API shape, interface signatures, YAML manifest schema, wire format — enough that a reviewer can imagine the implementation. Include diagrams if the change is architectural.>

### API changes
<The new methods / fields / manifests, with their signatures. If existing API surface changes, name the SemVer implication.>

### Data model / schema changes
<If applicable. Migration considerations flagged.>

### Rollout plan
<How the change ships. Behind a feature flag? Opt-in first, opt-out later? Version bump? Named milestones with dates.>

## Alternatives considered
<At least three. "Do nothing" is one. For each, name why the RFC's proposal is preferred. Alternatives-considered is where the RFC earns its trust — this is the section the reviewer reads to check whether the author has thought about it.>

## Impact
### Compatibility
<Backwards-compat implications. Which consumers break, when.>

### Migration
<How existing consumers move to the new shape. What tooling the platform team provides. What the consumer's work looks like.>

### Deprecation
<If this deprecates existing surface, the notice period, the sunset date, the migration guide.>

### On-call / support
<Who owns the on-call. What runbook lands. What alerting the platform's SLO doc needs.>

### Cost
<Compute, storage, network, engineer-hours. Both the platform team's cost and the fleet's cost.>

### Blast radius
<Who is affected if this ships and turns out to be wrong. The rollback plan.>

## Risks
<The three-to-five risks worth naming. For each, the mitigation.>

## Open questions
<Explicitly named. The reviewer can respond to these. "None" means the author has already thought hard, not that the RFC is complete.>

## References
<Links to the sponsor conversation, prior tickets, the quarterly consumer report, related RFCs, external references.>

## Decision log
<Populated during review. Reviewer comments, decisions, when status changes.>
```

Three properties of this structure that are worth naming out loud:

- **Summary comes first, motivation before proposal.** The reviewer decides whether to read the rest in the first 200 words. If you bury the summary or the problem statement, the reviewer bounces. Rachel Potvin's *[How to write a good design document](https://www.designdocsforengineers.com/)* is the reference for this discipline.
- **Alternatives-considered has weight.** A single-alternative RFC reads as advocacy. A three-alternative RFC reads as a decision. The section is not padding — it is the load-bearing evidence the author has done the thinking.
- **Migration and deprecation are named, not "TBD."** A contribute-back that leaves migration and deprecation as TBD is not decidable. The platform team's fear is that the change lands, breaks consumers, and leaves them holding the bag. Named migration and deprecation is what makes the fear go away.

## §4 — Writing to the platform team's altitude

The most common author-side failure of a contribute-back RFC is writing to the *author's own team* rather than to the *platform team*. The RFC lands in the platform-team's repo, is reviewed by platform-team engineers, and will be owned by the platform team; it must be written in the platform team's vocabulary and at their altitude.

Three shifts to make:

- **Consumers, not users.** The platform team's users are *other engineering teams*, not end-users. When you say "user," clarify — the RFC to a platform team names *consumers* explicitly. "This will change the shape of the SDK response consumers see when they call `fetch()`."
- **API-first, implementation-second.** The platform team reviews the *interface* first, the implementation second. An RFC that spends three pages on the internal Flink DAG and one paragraph on the SDK method signature has its priorities backwards. Reverse the ratio.
- **Fleet-scale reasoning.** The platform team's blast-radius calculus includes every team that consumes the affected surface. When you say "this adds 5 ms," name whose fleet-p99 that composes into. When you say "this deprecates method X," name how many consumers use it and how the platform team knows.

Yonatan Zunger's design-doc essay names the "friendly" property: an RFC written to a specific reviewer, with the reviewer's constraints and vocabulary in mind, is easier to review and more likely to be approved than one written to no one in particular.

## §5 — Alternatives considered: the load-bearing section

The alternatives-considered section is where the RFC earns its trust. Three failure modes to avoid:

- **The straw-man alternative.** A weak alternative described in one paragraph, dismissed with "this doesn't scale." A reviewer notices and reads it as sloppy. Every alternative deserves the same rigor as the recommended proposal — a summary, its design, its tradeoffs, why it was rejected.
- **The missing "do nothing" option.** Every RFC has a "do nothing" alternative. Sometimes it is right. Naming it explicitly, describing what happens if the platform ships nothing, and then rejecting it, is what forces the RFC to defend its cost against the do-nothing baseline. If you cannot defend the cost against do-nothing, the RFC is not ready.
- **The missing "solve it downstream" option.** For a contribute-back RFC, "solve it in the consumer's build-in-house layer instead of on the platform" is an alternative that must be considered. If the answer is "the consumer could build it themselves, but the fleet's total cost is 12× the platform's cost to do it once," name the number. If you can't name the number, the "solve it downstream" alternative may actually be the right answer, and the RFC is arguing for the contribute-back path in error.

The reference format for a well-done alternatives-considered section is the Rust RFC template. Each alternative gets a `### Alternative N: <name>` block with the same subsections as the main proposal (design, rollout, tradeoffs) at a lighter level of detail, and then a `#### Why not <alternative N>` block that pinpoints the deciding factor.

## §6 — Migration and deprecation as first-class sections

The two sections a platform-team reviewer scrutinises hardest are *migration* and *deprecation*. Both share the same underlying anxiety: "if we approve this RFC, do we (the platform team) own a compat / migration burden forever?"

For **migration**, the RFC must name:

- **The migration mechanism.** Automated (a codemod, a schema translator), semi-automated (a lint that suggests the new form), or manual (a docs page + a Slack ping). Automated is preferred; manual is a red flag unless the migration surface is tiny.
- **The migration cost per consumer.** In engineer-hours. Ten consumers × 2 hours each = 20 fleet hours; ten consumers × 20 hours each = 200 fleet hours. The number changes the RFC's economic case.
- **The migration timeline.** The window during which the old and new API both work, with the deprecation date at the end.
- **The migration owner.** Is the platform team providing hands-on migration help, or is each consumer team on their own? The answer determines what commitments the platform team is signing up for.

For **deprecation**, the RFC must name:

- **What is being deprecated.** The specific method, field, manifest, environment variable — not "the old API."
- **The deprecation policy.** How long does the deprecated surface work before it is removed? [Google's public deprecation policy](https://cloud.google.com/terms/deprecation) is the industry standard (12 months for a stable API); internal platforms typically follow a shorter but published version.
- **The deprecation signal.** Warning logs, doc annotations, linter rules, telemetry counting deprecated-API usage per consumer. Without a signal, the deprecation happens quietly and the consumers find out on cutover day.
- **The sunset date and the sunset action.** The specific date the deprecated surface stops working, and what the platform team does at that date (return an error, silently fall through, hard-remove).

A migration + deprecation section that names all of this is what allows the reviewer to say yes. A section that leaves these to be figured out later is what makes the reviewer say "come back when you've thought about it."

## §7 — Reviewers, approvers, and the review cycle

An RFC has *reviewers* (people whose comments are expected) and *approvers* (people whose sign-off unlocks merging). Named explicitly:

- **Sponsor.** The platform-team person who committed to shepherd this. Named at the top of the RFC.
- **Consumer reviewers.** Two or three other consumer teams likely to be affected. Their sign-off is not required, but their absence is a red flag — either the RFC does not affect them (in which case, why?) or they were not consulted (in which case, expect surprises).
- **Platform-team domain reviewers.** The engineer or engineers who own the affected surface. Their technical review is required.
- **Platform-team approver.** The TL, staff engineer, or manager whose sign-off unlocks implementation. Named singularly — "the platform team" is not an approver.
- **Escalation.** The name to escalate to if reviewers are unresponsive for more than a stated window (chapter 03's tempo is typically 1–2 weeks per review round).

The review cycle for a contribute-back RFC is typically:

- **Round 1: Draft circulation.** RFC circulated to sponsor + platform-team domain reviewers + 1–2 consumer teams. Comments in-doc. Author revises.
- **Round 2: Wider circulation.** RFC circulated to the platform-team review meeting or async equivalent. Comments in-doc. Author revises.
- **Round 3: Approval or rejection.** Approver signs off, or names the blocking objections. If rejected, the RFC's decision log records why.
- **Implementation.** PR follows, referencing the RFC in every commit / PR message.
- **Post-implementation.** RFC status is updated to `implemented`. A short retrospective — a few weeks after ship — updates the RFC with what was learned.

The senior discipline is to *drive* the review cycle. Reviewers rarely prioritise a doc that is not being pinged. A weekly nudge to the sponsor, plus a named escalation if a round takes more than 2 weeks, is what keeps the cycle moving.

## §8 — When the RFC is rejected

Rejection is a real and common outcome. Three healthy responses:

- **Withdrawn.** The author agrees the reviewers' objections are correct; the RFC is withdrawn. The problem may be re-framed and re-proposed later; the *decision log* is what makes that re-proposal legible.
- **Re-scoped.** The reviewers accept the problem but reject the proposal. The RFC is closed; a new RFC with a different proposal is written. The old RFC is linked from the new one.
- **Path change.** The RFC is rejected as a contribute-back, but the underlying problem is real. The team's chapter 02 tradeoff doc is re-opened and one of the other three paths is taken. The rejection is a *piece of evidence* that contribute-back is not viable for this specific gap.

Rejection is not a failure of the RFC process — it is the process working. The failure mode to avoid is treating rejection as personal, arguing with the reviewers post-decision, or (worst) landing the change unilaterally in a per-team fork. The senior discipline is to accept the rejection, update the tradeoff doc, and pick another path.

## §9 — Worked example: an RFC skeleton

A team wants to add per-feature-view freshness SLOs to the Chronon SDK — the recommendation-team's sub-hour-freshness problem from chapter 02 §8, contributed back rather than built in-house.

```markdown
# RFC: Per-feature-view freshness SLOs in the Chronon SDK

- Author: <name>, recommendation-serving team
- Sponsor: <name>, ml-platform team (Chronon TL)
- Reviewers: <name>, ml-platform (Chronon serving); <name>, fraud-scoring team (consumer); <name>, ranking team (consumer)
- Approver: <name>, ml-platform staff engineer
- Status: draft
- Created: 2026-03-14

## Summary
Add a first-class per-feature-view freshness SLO field to the Chronon feature-view manifest, backed by a scheduler-level guarantee and a fetch-time SLI. Freshness becomes a declared property of a feature-view, not a runtime discovery.

## Motivation

### Problem
Chronon's current freshness contract is scheduler-cadence-implicit. Consumers infer freshness from their knowledge of the batch cadence; there is no way for a consumer to declare "this feature-view must be ≤ N minutes old at fetch time" and have the platform enforce it. In the last two quarters, four consumer teams (fraud, ranking, recommendation-serving, notifications) have hit this. Aggregate on-team workaround cost: ~120 engineer-hours.

### Why now
Two teams (recommendation-serving, notifications) have Q2 deadlines that need sub-hourly freshness. Without the platform-side fix, both teams will build in-house interim solutions with 12-month migration plans (chapter 02 pattern). Contributing this back once is cheaper than migrating four teams off four separate interims.

### Goals
- Per-feature-view freshness SLO as a first-class manifest field.
- Fetch-time SLI that consumers can point their SLO document at.
- Platform-scheduler guarantee that the SLO is enforced.

### Non-goals
- Sub-minute freshness. This RFC targets the sub-hour class, not the sub-second class.
- Retrofitting freshness contracts onto existing feature-views without opt-in.

## Proposal
### Design
Add a `freshness` block to the feature-view manifest:

```yaml
feature_view:
  name: user_active_session
  freshness:
    slo: "15m"
    p: 0.995
    window: 28d
```

At fetch time, the SDK emits a `chronon_feature_view_freshness_seconds` metric per feature. The Chronon scheduler is extended with a per-feature-view priority queue keyed off the declared SLO.

[...design continues at API-first altitude...]

### Alternatives considered
1. **Do nothing.** Consumers continue to work around. Cost: ~120 hours / quarter fleet-wide, growing.
2. **Consumer-side freshness monitoring only.** Add the SLI without the platform guarantee. Cheaper for the platform team, but does not solve the consumer's underlying problem — they still cannot rely on the freshness contract, only observe when it breaks.
3. **Per-team Flink jobs (build-in-house).** Each team builds interim. Cost: ~4 teams × 3 weeks + on-call. Fleet-fragile.

Chosen: the platform-side fix. Justification: cost of do-nothing exceeds the RFC's platform-side cost; consumer-side-only leaves the underlying gap; per-team is 4× the fleet cost.

### Impact
[...migration, deprecation (this is additive, no deprecation), on-call, cost, blast radius sections...]

### Risks
- Scheduler queue starvation if too many consumers opt-in to aggressive SLOs. Mitigation: quota per team, escalation to sponsor for exceptions.
- False-positive SLI alerts on newly-onboarded feature-views. Mitigation: SLI-observability-only mode for first 14 days.

### Open questions
- Does the SLO apply to online-only feature-views, offline-only, or both?
- What is the burn-rate alerting shape for the freshness SLO? (mod-307 chapter 01 §2 vocabulary.)

### References
- Quarterly consumer report 2026-Q1 (this problem, ranked #2).
- Sponsor conversation 2026-03-07.
- mod-307 chapter 01 §5 (dependency-SLI composition).
- [Google's SLO Workbook chapter](https://sre.google/workbook/implementing-slos/) — the SLO shape reference.
```

The RFC is short (this skeleton would be 4–5 pages when fully written). The alternatives-considered and impact sections are as substantial as the design section, deliberately. The reviewers can decide.

Exercise 03 authors an RFC of this shape — full, not skeletal.

## Summary

The contribute-back RFC is the artefact that makes chapter 02's contribute-back path work. An RFC is a specific proposal to a specific platform team, with a specific sponsor, in a shared and versioned location, that names problem → alternatives → proposal → migration → deprecation → reviewers and approvers. The precondition is the *sponsor conversation* — a low-stakes preview of the problem and directional proposal that establishes the sponsor exists and the problem is agreed. The RFC's structure (§3) puts summary and motivation before proposal, treats alternatives-considered as a load-bearing section (§5), and treats migration and deprecation as first-class subsections (§6). The RFC is written to the platform team's altitude, not the author's team's altitude (§4) — consumers not users, API-first, fleet-scale reasoning. Reviewers and approvers are named individuals; the review cycle is *driven* by the author with a weekly cadence and a named escalation. Rejection is a valid outcome — the healthy response is to update the tradeoff doc and pick another chapter-02 path, not to argue post-decision or land unilaterally in a fork.
