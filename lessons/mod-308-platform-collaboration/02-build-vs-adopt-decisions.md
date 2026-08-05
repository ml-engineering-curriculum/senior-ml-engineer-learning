# Build vs. adopt: the four-way decision at team scope

## Motivation

Chapter 01 established the paved road and named what it means to consume it idiomatically. This chapter is about the moments when the paved road is *not enough* — the feature store doesn't do X, the model registry can't represent Y, the serving stack won't hit the p99 target. The naïve responses at each altitude:

- **L20:** "Build it. We can spin up a K8s cluster and run our own serving stack this sprint." (Almost always the wrong answer.)
- **L30:** notice the four decision paths — *bend* to the platform, *escalate* the gap, *build in-house* with a migration plan, or *contribute back* — and pick the one whose blast radius, reversibility, and opportunity cost match the situation.
- **L40:** re-shape the platform's roadmap so the situation doesn't happen twice.

This chapter is at the L30 altitude: how to pick the right decision path for the team, and how to defend the pick to a peer L30 and to the platform team. The three tell-tales of an L30-level build-vs-adopt decision are (a) it explicitly considers all four paths before committing; (b) it names the reversibility and blast radius, not just the technical fit; (c) it has an *exit plan* for the "build" or "escalate" path so the choice is not permanent by default.

The load-bearing external reference is Will Larson's [*The three most important behaviors on adopting versus building software*](https://lethain.com/adopting-versus-building/) and his book [*The Engineering Executive's Primer*](https://lethain.com/eep/) chapter on platform investment. Camille Fournier's *[Bar-raising internal platforms](https://skamille.medium.com/bar-raising-internal-platforms-d5b8a72c8a1e)* is the platform-team-side view of the same decision. Both are worth skimming.

## §1 — The four paths and their default cost

At the moment of "the platform doesn't do X," a senior ML engineer has *four* options, not two. Naming all four explicitly is the first move; L20 collapses to a two-option "adopt vs. build" that mis-sizes the decision.

- **Bend.** Change your requirement so the paved road *does* cover you. The X you thought you needed turns out to be a nice-to-have; the P99 target you set was arbitrary; the feature-view shape you wanted can be re-expressed in the DSL the platform already supports. Cost: engineering effort to re-scope your feature. Reversibility: high — you can re-open the requirement later. Blast radius: your team only.
- **Escalate.** Ask the platform team to add X. File the gap as a quarterly-report item (chapter 01 §5.2); if they agree the problem is real, help scope the RFC (chapter 03). Cost: mostly waiting time, plus the effort to write the ticket / report / RFC. Reversibility: high — if the platform team says no, you fall back to one of the other paths. Blast radius: yours, but eventually the platform's if adopted.
- **Build in-house with a migration plan.** Build X on your team's side because you cannot bend and cannot wait. The critical qualifier is *with a migration plan* — the build is *interim* until the platform team paves this lane, and there is an explicit commitment to migrate off. Cost: engineering effort now, plus migration effort later, plus on-call for the interim. Reversibility: medium — the migration plan is what keeps it reversible; without a migration plan, this becomes permanent by default. Blast radius: your team, plus anyone else who ends up depending on your interim.
- **Contribute back.** Build X *on the platform side* — you write the code, the platform team reviews and owns it. This is the RFC + PR pattern of chapter 03. Cost: substantially higher engineering effort (platform-quality bar, platform-team review cycle, platform-team on-call training), but no migration debt. Reversibility: low once merged — the platform team now owns the code you wrote. Blast radius: the platform's fleet.

The senior discipline is that "build in-house" and "contribute back" are *different* answers to the same underlying question. They differ in *who ends up owning the code long-term*. If you build in-house, you own it; if you contribute back, the platform team owns it. That ownership difference is the load-bearing distinction the L20 build-vs-adopt framing collapses. Chapter 03 is exclusively about how to make the contribute-back option work.

## §2 — The five factors in the decision

Every serious build-vs-adopt discussion at senior altitude weighs the same five factors. The failure mode at L20 is to weigh only *technical fit*. The failure mode at platform-team altitude is to over-weight *fleet-wide reusability*. The senior read considers all five and names each one explicitly.

### 2.1 — Technical fit

Does the paved-road offering *cover* the requirement? Not "roughly" — cover in the specific dimensions that matter: latency budget, throughput budget, correctness guarantee, feature completeness, ergonomic fit. mod-307 chapter 01 §5's dependency-SLI composition math is the tool: your service's SLO cannot be tighter than the composition of the platform primitives it depends on.

The nuance at L30: *how much* does the offering not fit? A 5 % gap ("the SDK is missing a convenience method") is a bend or a small contribute-back. A 30 % gap ("the freshness SLO is 24 hours and we need 10 minutes") is a real design mismatch. A 90 % gap ("we need vector-search and the platform has none") is a "build or contribute a new module" decision.

### 2.2 — Reversibility (one-way vs. two-way doors)

Jeff Bezos's [*one-way vs. two-way doors*](https://www.aboutamazon.com/news/company-news/2016-letter-to-shareholders) framing is a load-bearing tool here. A two-way door is a decision you can walk back cheaply; a one-way door commits you.

- Bend → two-way. If it doesn't work, you re-scope.
- Escalate → two-way. If the platform team says no, you pick another path.
- Build in-house with a migration plan → mostly two-way. The migration plan is what keeps it reversible.
- Build in-house without a migration plan → *one-way*, and this is the case senior engineers most often mis-read as two-way. Once your feature depends on the in-house tool, the tool is *permanently* on your on-call rotation. This is the "we built our own X for our team" story chapter 01 §6 named as an anti-pattern.
- Contribute back → *one-way*. Once merged, the platform team owns it; you cannot un-contribute.

The senior discipline is to prefer two-way doors when the situation is uncertain and only walk through a one-way door when the situation is well-understood. Bezos's line: "type 2 decisions can and should be made quickly by high judgment individuals or small groups. Type 1 decisions must be made methodically, carefully, slowly."

### 2.3 — Blast radius

Who is affected if this decision turns out to be wrong?

- Your team's productivity? Blast radius is small.
- Your feature's user-facing SLO? Blast radius is your feature's users.
- The platform's fleet? Blast radius is the org's ML surface.
- The user-facing SLA? Blast radius is the business.

The blast radius rises up the escalate/build/contribute-back sequence. A bend has near-zero blast radius (only your team's re-scoped requirement); an escalate has small blast radius (the platform team's roadmap); a build-in-house has your-team-plus-anyone-who-adopts-it blast radius; a contribute-back has the fleet's blast radius (a bug you wrote runs on every team's traffic).

The senior discipline is that decisions with larger blast radius warrant more process: more reviewers, more testing, more staged rollout, more chances to bail. This is the *reason* the contribute-back path has a heavier RFC and code-review process (chapter 03) — the blast radius warrants it.

### 2.4 — Opportunity cost

What else could the team be building instead? An hour spent on a build-in-house of feature-view versioning is an hour not spent on the model quality work that is the team's job. The opportunity cost is often the tie-breaker between "build" and "escalate" — the build path looks cheap in engineering hours until you notice those hours are coming out of the team's model roadmap.

The senior discipline is to compute opportunity cost *in units the team defends its roadmap in* — number of experiments per quarter, number of features shipped, offline eval improvements. "Six weeks of one engineer" is only meaningful if the roadmap loses something specific to make room for it.

### 2.5 — Total cost of ownership over the time horizon

The build path's headline cost is the initial build. The *long-run* cost includes on-call, migration when the platform primitive underneath your build changes, evolution as your requirements change, hand-off if the original builder leaves the team. The [*true cost of building software*](https://queue.acm.org/detail.cfm?id=3572814) is dominated by these post-launch costs, not the initial build.

The senior discipline is a three-year total-cost estimate for every path. Bend has near-zero long-run cost. Escalate has near-zero long-run cost (once the platform team ships it). Build in-house has a *large* long-run cost unless the migration plan is executed. Contribute-back has near-zero long-run cost for your team, but a real long-run cost for the platform team — which is why they have veto power over what you contribute.

## §3 — The decision matrix

A compact decision matrix that combines §2's factors:

```markdown
| Gap size | Reversibility need | Blast radius | Recommended path |
|---|---|---|---|
| Small (≤ 10 %) | High | Team | Bend, or contribute back a small PR |
| Medium (10–30 %) | High | Team | Escalate to platform team |
| Medium (10–30 %) | Low | Team | Build in-house with 6-month migration plan |
| Medium (10–30 %) | Low | Fleet-relevant | Contribute back via RFC + PR |
| Large (30–90 %) | High | Team | Escalate; expect quarterly-roadmap timeline |
| Large (30–90 %) | Low | Team | Build in-house with 12-month migration plan; expect on-call cost |
| Large (30–90 %) | Low | Fleet-relevant | Contribute back a new module via multi-quarter RFC |
| Large (30–90 %) | Low | Fleet, and platform team unwilling | Escalate to platform leadership; do NOT unilaterally build in-house |
| Very large (> 90 %) | Any | Any | Escalate to platform leadership + engineering leadership; this is a strategy question, not a team-scope call |
```

The matrix is a starting point, not a rule. Two properties of it that L30s notice:

- **"Build in-house" is never the first-choice row.** It is always the *fallback* when reversibility is low and the platform team cannot help in time. If you find yourself in a "build in-house" row, the diagnostic question is *why*: what made escalate infeasible? The answer is either "the platform team has other priorities" (a real answer that lives in the tradeoff doc) or "we didn't ask" (a red flag).
- **"Very large" gaps escalate out of your seniority band.** A 90 %+ gap is a strategic-level question about what the platform should cover, not a team-scope call. The senior ML engineer's job in that row is to escalate it correctly (to platform leadership + eng leadership) with a written statement of the gap, not to make the call.

## §4 — The build-vs-adopt tradeoff doc

The artefact for a serious build-vs-adopt decision is a short document — one page, sometimes two — that a peer L30 and the platform team can review before you commit to a path. Exercise 02 authors this doc. The skeleton:

```markdown
# Build-vs-adopt: <specific gap>

## Context
- Team: <team>
- Feature the gap blocks: <feature>
- Deadline / cadence pressure: <what makes this time-bound>

## The gap
- The requirement: <specific, measurable — e.g., "feature-view freshness < 10 min for the fraud-blocklist view">
- The platform's current offering: <specific — e.g., "Chronon supports batch freshness; the shortest scheduled batch is 1 h">
- The gap in %: <coverage estimate>

## The four paths
### Bend
- Would this work? <yes/no; how the requirement would change>
- Cost: <hours; opportunity cost>

### Escalate
- Feasibility: <platform team's stated priority; the ticket / quarterly-report item; the response so far>
- Timeline: <best/worst-case quarters until shipped>

### Build in-house (with migration)
- Scope: <what we would build; the interfaces; the on-call surface>
- Initial cost: <engineer-weeks>
- Long-run cost: <on-call load; migration cost; risk of permanence>
- Migration trigger: <the platform-team milestone that starts our migration>

### Contribute back
- Scope: <what we would contribute; the RFC shape>
- Cost: <engineer-weeks including platform-team-review overhead>
- Sponsor on the platform side: <named person, or "unidentified — precondition to consider this path">

## Recommendation and defence
- Path: <one of the four>
- Why: <three-bullet defence weighing the five factors>
- Exit plan: <how we get out of this decision if it goes wrong; the review date to revisit it>

## Reviewers
- Team peer L30: <name>
- Platform team liaison: <name>
- Approver: <name — team TL or manager>
```

Two properties of this doc that are worth naming:

- **The four paths are always covered, even when the answer is obvious.** Naming the paths you rejected is what makes the recommendation defensible. A doc that only argues for the winner reads as advocacy; a doc that names all four and rejects three reads as a decision.
- **The exit plan is mandatory.** If you cannot name how you would walk back the decision if it went wrong, the decision is a one-way door and the process should be heavier before you walk through it. "The exit plan is: we accept this is permanent" is a valid exit plan — but naming it forces the recognition that the decision is one-way.

## §5 — The migration plan (if you must build in-house)

If the tradeoff doc lands on "build in-house," the plan must include how you get *off* the in-house build. Without a migration plan, "in-house with migration" degrades into "in-house forever," which is the anti-pattern of chapter 01 §6.

A serious migration plan names:

- **The trigger.** The platform-team milestone that starts the migration. "When Chronon's sub-hour freshness ships in Q4" is a trigger; "eventually" is not.
- **The interface.** The API the team consumes from the in-house tool is designed to *match* the eventual platform API as closely as possible. When the platform ships, the migration is `import statements`, not a re-architecture.
- **The interim cost cap.** The tool is scoped to the specific requirement, not extended over time. Feature creep on the in-house tool is what makes migration hard. A one-sentence "in-scope / out-of-scope" list, enforced at code-review time on the interim tool's PRs, is how this is defended.
- **The review cadence.** Every quarter, the migration plan is revisited. Is the platform still on track to hit the trigger? Has the trigger slipped? Has the team's requirement changed? The migration plan is a living doc, not a one-time write-up.
- **The commitment.** Signed by the team's TL / manager and by the platform-team liaison. When the trigger fires, the team is *committing* to a migration in a bounded number of quarters. Without this commitment, the interim becomes the permanent by default when the trigger fires.

Google's [SRE workbook chapter on deprecation](https://sre.google/workbook/managing-load/) has adjacent discussion of the same discipline for internal service migrations. The equivalent for the in-house build path is that your team is *pre-committing* to being an obedient migrator when the platform primitive lands.

## §6 — The "escalate correctly" discipline

The escalate path is often mis-executed even when it is the right choice. Two common failure modes:

- **Escalating to the wrong altitude.** Escalating a per-feature-view freshness gap directly to the platform team's staff engineer is a distraction; escalating a "the platform lacks any freshness guarantees" gap to an on-call engineer is under-escalation. The senior discipline is to match the *escalation altitude* to the *problem altitude*.
- **Escalating without evidence.** "The feature store is annoying" is not an escalation, it is a complaint. "The feature store's fetch p99 breaches our 20 ms budget for 4 % of requests, costing us 20 engineering-hours per team-quarter in on-team workarounds, and here is the ticket log" is an escalation.

The senior discipline for escalation is the same as for the RFC (chapter 03): the problem statement is quantified, the cost of *not* fixing it is quantified, and the *specific* platform-team action being requested is named. "Please prioritise this in Q4" is a specific action; "please look at this" is not.

## §7 — The "contribute back" trap

The contribute-back path is the most senior-flattering of the four — it feels like "leading" — and is the most often mis-taken. Three tests before you commit to it:

- **Is the problem fleet-relevant?** If the requirement is genuinely team-specific, contributing back forces the platform team to maintain your special-case forever. Bend or build-in-house is the right choice.
- **Is the platform team resourced to *review* it?** The reviewer cost on the platform side is real. An RFC + PR that lands during the platform team's quarterly-planning crunch will sit in review for months. Preview the pitch with the platform-team liaison first.
- **Is your team resourced to *maintain* it through the review cycle?** Contribute-back typically takes 2–4× longer than the equivalent in-house build because of the higher bar, the review latency, and the platform-quality integration testing. If your team's deadline is inside that window, contribute-back is not going to unblock you in time — build-in-house-with-migration is the fallback and the contribute-back can happen *after* the deadline.

Chapter 03 is entirely about doing the contribute-back well when it is the right choice.

## §8 — Worked example: a build-vs-adopt call

A recommendation-team feature needs sub-15-minute freshness on a feature-view (the user's active session). The platform's Chronon supports 1-hour batch freshness; the sub-hour work is on the roadmap but not committed.

The team walks the four paths:

- **Bend.** Could the feature use a stale session? The product team says no — recommendations against a 1-hour-old session are user-visibly wrong. Bend is rejected.
- **Escalate.** The team files a ticket and a quarterly-report line-item. The platform team's response: the sub-hour work is Q1 next year at earliest. The feature ships this quarter. Escalate alone will not unblock the deadline.
- **Build in-house with migration.** The team drafts an interim: a small Flink job co-owned with the platform team, writing to a per-team online store, exposing the platform's SDK shape so migration is import-only. Initial cost: 3 engineer-weeks. On-call: bounded, since the Flink pipeline mirrors an existing pattern. Migration trigger: Chronon sub-hour ships. Migration signed by team TL and platform-team liaison.
- **Contribute back.** The team could contribute the sub-hour path to Chronon directly. Rejected: (a) the platform team owns the architectural design of freshness at that altitude; a contribute-back that bypasses their design is fleet-fragile. (b) The review cycle is longer than the deadline.

Recommendation: **build in-house with migration**, path selected as the fallback under a hard deadline, with the escalate path also active for the durable fix.

Exit plan: at Q1 next year's checkpoint, revisit. If Chronon shipped sub-hour on schedule, migrate. If it slipped by more than 2 quarters, re-open the doc — is the interim now permanent-by-default, and if so is the right move to contribute back the interim to the platform?

The three-line summary a peer L30 could sign on: *bend* fails on product; *escalate* fails on deadline; *contribute back* fails on review cycle; *build in-house with migration* is the least-worst, with the exit plan pinning the migration commitment.

## Summary

The build-vs-adopt decision has *four* paths, not two: *bend* (change your requirement to fit), *escalate* (ask the platform team), *build in-house with a migration plan* (interim, reversible), *contribute back* (RFC + PR, one-way). Five factors decide: technical fit, reversibility (one-way vs. two-way door), blast radius, opportunity cost, three-year total cost of ownership. The decision matrix (§3) maps combinations of these factors to recommended paths, with the load-bearing observation that "build in-house" is never the first-choice row — always the fallback when reversibility is low and escalate is infeasible in the timeframe. The tradeoff doc (§4) names all four paths, defends the recommendation, and includes an exit plan; the doc is what a peer L30 and the platform team sign. The migration plan (§5) is what keeps "build in-house" from becoming "in-house forever" by default; the discipline is a named trigger, an interface that matches the eventual platform API, a scope cap, a review cadence, and a signed commitment. Escalation (§6) is done at the right altitude with quantified evidence. Contribute-back (§7) is the most flattering path and the most often mis-taken; three tests (fleet-relevant, platform-team resourced to review, team resourced to maintain through the review cycle) gate it. Chapter 03 covers the contribute-back RFC end-to-end.
