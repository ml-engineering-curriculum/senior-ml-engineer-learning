# Scoping and sequencing a multi-quarter ML initiative

## Motivation

At L20 you own a *feature* — a model, a retraining job, a monitoring dashboard. The unit of planning is the sprint. The commitment is "I will ship this thing by that date." When the plan slips, the blast radius is the two people who were waiting on the feature and the PM who has to reshuffle a launch.

At L30 you own a *multi-quarter initiative* — a search relevance rebuild, an LLM-augmented triage rollout, a fraud-model platform migration, a personalisation-ranker refactor. The unit of planning is the quarter, sometimes the half. The commitment is "here is what our team will deliver over the next three-to-six months, in this sequence, against these dependencies, at this cost, with these decision points." When the plan slips, the blast radius includes the product roadmap, the platform team's Q+1 capacity plan, the data team's warehouse migration, and the director's exec-review deck.

The senior discipline is producing a **roadmap leadership can commit to**. Two properties make it commitable:

- **It is defensible against a director-level question.** "Why this sequence, not that one? What breaks if we pull Q3 into Q2? What is the cheapest thing we could cut? Where are the hidden dependencies?" A roadmap that cannot answer these questions cold is a wish list, not a plan.
- **It has decision points that are legible.** Every multi-quarter ML initiative has moments where the team has to choose between two branches — commit to the classical baseline vs. wait for the LLM-augmented alternative, keep building against the old feature store vs. cut over to the new one, ship the smaller-scope MVP vs. hold for the full-quality version. The roadmap names those decision points *before* they arrive, with the evidence the decision will use.

This chapter installs the artefact and the discipline. Chapter 02 governs the reviews that hold the initiative to bar as it ships. Chapter 03 governs how the team's shared standards evolve as the initiative reveals new needs. Chapter 04 governs how the team is *staffed* to sustain the initiative through the interview loop.

Two primary external references anchor the chapter's technique:

- **Will Larson's *[An Elegant Puzzle: Systems of Engineering Management](https://lethain.com/elegant-puzzle/)*** — specifically the *Sizing Engineering Teams* and *Product Management with Just Enough Process* essays. Larson's [staff-engineer archetypes essay](https://staffeng.com/guides/staff-archetypes) is a companion — the *tech lead* archetype is the one this module trains against.
- **Camille Fournier, *[The Manager's Path](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)*** — the *Tech Lead* chapter names the roadmap discipline as the load-bearing shift from IC to lead. This module inherits that framing but keeps it inside the IC track.

The technique is not new. What is new at senior-ML is the *ML-specific shape* of the roadmap — the uncertainty a model brings, the retraining cadence that has to compose with product deadlines, the data-availability gates that pace the whole plan.

## §1 — What a roadmap is (and is not)

An ML initiative roadmap is a written artefact that:

- Names a **specific outcome** the initiative is committing to, in the vocabulary of the reviewer who will commit it (product outcome for a product team, cost outcome for a finance partner, capability outcome for a platform team). One or two sentences.
- Sequences the work into **quarter-or-half-scoped milestones**, each with a named deliverable and a named owner.
- Names the **decision points** — the moments where the plan branches — with the evidence each decision will use.
- Names the **dependencies** — the platform primitives, the peer-team deliverables, the data availability, the compliance sign-offs — and the risk if any dependency slips.
- Names the **cost and headcount** — how many engineers, how many GPU-months, how many vendor dollars, over what horizon.
- Names the **cutlines** — what gets dropped if the team ends up half-staffed, if a dependency slips, if the product deadline pulls in.
- Lives in a **shared, versioned location** — a repo, a wiki page, a Google Doc with edit history — that everyone with a stake can read and comment on.

It is *not*:

- A Gantt chart. Gantt charts imply a level of temporal precision ML initiatives rarely have. The roadmap communicates *sequence and decision points*, not day-by-day timelines. When someone asks for a Gantt chart, the answer is usually "here is the milestone list; I can rough date-bracket the milestones but the intra-milestone timing is a team-level question."
- A backlog. A backlog is a bag of tickets ordered by "next up." A roadmap is a *narrative* — "here is the outcome we are chasing, here is the sequence, here are the decision points, here is what we cut if we have to cut."
- A staffing spreadsheet. Staffing is one section of the roadmap, not the whole thing. Staff-driven roadmaps ("we have five engineers, what can they do this quarter?") produce plans that are legible to your director but not to your product partner or your platform-team peer.
- A slide deck. Slides are for the review meeting the roadmap is *presented at*. The load-bearing artefact is the writing.

The senior discipline is to notice when you are producing one of the *not-roadmaps* — a backlog, a Gantt, a staffing spreadsheet, a slide deck — and stop. Write the roadmap first; the other artefacts are extractions from it.

## §2 — The four questions the roadmap answers

A commitable ML initiative roadmap answers four questions, in this order:

### 2.1 What outcome are we committing to?

The outcome is the *thing that will be different in the world* when the initiative is done. It is *not* the model, the pipeline, the dashboard. It is the observable end-state.

- Bad: "Ship a new ranker."
- Better: "Improve search-result quality on the mobile surface, measured by a 3 % lift in the top-1 CTR guardrail, with no regression on session latency at p99."
- Best: "By end of Q3, the mobile-search top-1 CTR is ≥ 3 % above the Q1 baseline, at ≤ p99 500 ms end-to-end latency, gated by a shadow-then-canary rollout with SLO signoff. Downside: if the lift is < 1 %, we hold the launch and post a 'lessons learned' note that funds the next iteration."

The outcome-first framing is what makes the roadmap commitable. The reviewer is not committing to your work — they are committing to a *state of the world*. Everything else in the roadmap defends the path to that state.

Three tests for the outcome statement:

- **Falsifiability.** Can you tell, at the end, whether the outcome was met? "Improve quality" fails; "top-1 CTR ≥ 3 % lift" passes.
- **Boundedness.** Is the outcome time-bounded? "By end of Q3" beats "eventually."
- **Downside symmetry.** What does the roadmap do if the outcome is *not* met? A plan without a stated downside is a plan whose failure mode is silent. Name it.

### 2.2 What is the sequence, and why this sequence?

The sequence is the ordered list of milestones the team will hit on the way to the outcome. Each milestone is a *deliverable* — a shipped piece of work — and each milestone has a *rationale*: why it comes here in the sequence and not earlier or later.

For a typical multi-quarter ML initiative, the sequence has these milestone kinds:

- **Baseline lock.** The frozen offline eval harness (mod-305 chapter 01), the frozen production baseline model, the "we agree this is what we are trying to beat" moment. Every initiative starts here. Skipping it is the most common source of a mid-quarter "wait, what were we comparing against?" incident.
- **First-pass model.** The simplest thing that could plausibly work. Often deliberately not the model you expect to ship — the point is to *find out what a working end-to-end pipeline reveals* before investing in the sophisticated model.
- **Data and infra unblock.** The upstream dependencies — a new feature-store view, a labelling pipeline, a training-cluster capacity ask — that block progress if they slip. These are surfaced early because their slippage is often the initiative's dominant risk.
- **Sophisticated model.** The version that is expected to hit the outcome. Trained against the baseline lock, evaluated against the harness, staged behind the same rollout gate the baseline was.
- **Shadow deployment.** The model is shipped to production but its predictions are not consumed. Shadow catches training/serving skew (mod-302), infrastructure regressions, and cost anomalies before any user is exposed.
- **Canary rollout.** The model is exposed to a small fraction of traffic behind an experiment (mod-306), the guardrails are watched, the ramp plan is followed.
- **Full rollout.** The model is at 100 %; the baseline is retired; the runbook (mod-307) is registered.
- **Handoff and retraining.** The initiative closes with a *steady-state hand-off* — the retraining cadence (mod-307 chapter 05), the on-call ownership, the review-cadence policy (mod-309 chapter 03), the SLO document, the model card (mod-309 chapter 01).

The *why this sequence* is answered in one paragraph per milestone: "Baseline lock in week 1 because without it we cannot claim any lift. First-pass model in weeks 2–3 because it forces the pipeline gaps to surface before we invest in the sophisticated model. Data-and-infra unblock in weeks 3–6 in parallel because the feature-view work depends on the labelling pipeline landing first. Sophisticated model in Q2 because that's when the labelled data volume clears the minimum-for-fine-tuning threshold. Shadow through the end of Q2 because we want two full weekly cycles of skew data before we open canary." The paragraph is not padding — it is what makes the sequence defensible against "why not the reverse?"

### 2.3 What are the decision points?

Every non-trivial ML initiative has moments where the team has to *choose*, and the choice depends on evidence that isn't in hand yet. The roadmap names those moments in advance, with the evidence each decision will use. Naming decision points ahead of time is the difference between a plan that adapts as evidence arrives and a plan that gets *renegotiated* every time the team learns something.

Typical ML-initiative decision points:

- **Baseline-lock decision.** After the baseline lock, does the current production model already meet the target on the frozen harness? If yes, the initiative is scoped to a *maintenance* milestone rather than a build milestone.
- **First-pass-model decision.** After the first-pass model, does the simplest approach clear the outcome threshold? If yes, the sophisticated-model milestone is pruned or de-scoped. If no, what does the gap tell us about which sophisticated approach to invest in?
- **Data-availability decision.** After the labelling pipeline reports its first month of yield, is the labelled-data volume on track? If not, we choose between (a) extending the labelling timeline, (b) switching to a lower-supervision approach (weak labels, distillation), (c) narrowing the scope so the labels we do have suffice.
- **Prompt-vs-fine-tune decision.** For LLM-augmented initiatives, after the prompt-engineering iteration on the eval harness saturates, do we invest in fine-tuning or stay on the prompt path? mod-304 chapter 01 is the technical treatment of the tradeoff; the roadmap decision point is *when* the team decides.
- **Shadow-to-canary decision.** After shadow, is training/serving skew within budget? Are the cost projections holding? Is the p99 latency budget met? Any of these being no is a "hold the canary" trigger and a *decision*, not an emergency.
- **Ramp-step decision.** During canary, at each ramp step (mod-306 chapter 01), does the guardrail-metric picture support the next ramp? If not, we roll back, hold, or extend the current step and gather more data.
- **Cutline decision.** If the initiative is under-staffed (someone leaves, a peer team's dependency slips), what gets cut? The cutlines are named *now*, not at the moment of pain.

The decision point is written in the format: **at <milestone>, using <evidence>, we will decide <branch A> vs. <branch B> vs. <cut>**. Three properties matter:

- **The trigger is a milestone, not a date.** "When the first-pass model report lands" beats "in week 6." The initiative can slip a week without invalidating the decision structure.
- **The evidence is named.** Not "we'll look at the numbers" but "the top-1 CTR lift on the frozen harness, with the paired-bootstrap CI shape from mod-305 chapter 01." The reviewer can check whether the evidence will actually support the decision.
- **The branches are named.** Not "we'll figure it out" but "sophisticated model A vs. sophisticated model B vs. narrower-scope-with-baseline-only." A decision point without named branches is a wish for future clarity.

### 2.4 What are the dependencies, cost, and cutlines?

The final section of the roadmap makes the plan *executable* by naming what the team needs and what the team gives up when it can't have it.

- **Dependencies.** Every peer-team commitment the plan requires, with the *specific* deliverable, the *named* owner, the *earliest date* the deliverable must be ready by to avoid slipping this initiative. mod-308's paved-road inventory is the source. Each dependency has a *risk column* — what happens if it slips — and a *contingency* — the fallback the initiative takes if it does. "Feature-store view `user_active_session_v3` from ml-platform team, owner <name>, earliest date 2026-05-01. Risk: canary rollout slips by the delta. Contingency: retrain on the v2 view and ship a v3 upgrade patch in Q4."
- **Cost.** Compute (GPU-months, CPU-hours, storage), vendor (API tokens, licences), and headcount (engineer-quarters). The numbers are estimated with a *range*, not a single value. mod-307 chapter 03's cost-budgeting discipline is the toolkit. Total cost lands in the initiative's finance-review packet.
- **Headcount.** Named engineers, with the fraction of their time this initiative expects to consume, over each quarter. A senior tech lead expects to consume 20–40 % of their own time on the initiative (the rest being reviews, mentorship, on-call, roadmap upkeep — the L30 job the other chapters name). If the roadmap needs 100 % of the tech lead's time, either the initiative is too large for one lead or the lead role is under-staffed.
- **Cutlines.** The ordered list of what gets cut if the team is short — one engineer down for a quarter, a dependency slipping a month, a product deadline pulling in. The cutlines are named *now*, in the roadmap, so they can be exercised without a re-negotiation. "If we lose one engineer for Q2, we cut milestone M3 (sophisticated model B) and hold at the baseline-plus-first-pass. If the feature-store dependency slips 4 weeks, we cut the canary in Q2 and slide it to Q3." A roadmap without cutlines is a roadmap that fails as soon as reality diverges.

## §3 — The template you actually ship

Below is a minimum viable ML initiative roadmap, in the shape that combines the four questions of §2 into a single artefact. This is the file that lives at `docs/roadmap-<initiative>-<year>.md` in the team's repo and is the artefact leadership commits to.

```markdown
# Roadmap — <initiative name> — <year / half>

**Version.** <n>
**Owner.** <senior ML engineer name>
**Sponsor.** <named director or lead PM>
**Reviewers.** <named peers on platform, product, data, governance>
**Status.** draft | in review | committed | in progress | delivered | cut
**Created.** <date>
**Last updated.** <date>
**Next review.** <date>

## 1. Outcome

<One paragraph. Falsifiable. Bounded. Downside named. See §2.1 for the tests.>

## 2. Non-goals

<Explicit list of what this initiative deliberately does NOT deliver. Scoping is 30 % of the work. A roadmap with no non-goals is a roadmap that will be asked to do everything.>

## 3. Sequence

For each milestone, name the deliverable, the target quarter (or half), the owner, and *why this sequence*.

- **M0 — Baseline lock.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.
- **M1 — First-pass model.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.
- **M2 — Data / infra unblock.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.
- **M3 — Sophisticated model.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.
- **M4 — Shadow.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.
- **M5 — Canary.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.
- **M6 — Full rollout.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.
- **M7 — Handoff / retraining.** <deliverable>. <quarter>. <owner>. Why here: <rationale>.

## 4. Decision points

For each: at <milestone>, using <evidence>, we will decide <branch A> vs. <branch B> vs. <cut>.

- **D1 — Baseline-lock decision.** <at M0, using <baseline eval report>, we will decide build vs. maintenance>.
- **D2 — First-pass decision.** <at M1, using <first-pass eval report>, we will decide sophisticated model A vs. B vs. de-scope>.
- **D3 — Data-availability decision.** <at end of month 2 of labelling, using <labelling-yield report>, we will decide extend timeline vs. lower-supervision vs. narrower scope>.
- **D4 — Prompt-vs-fine-tune decision.** <if LLM-augmented; at M3, using <prompt-iteration eval saturation curve>, we will decide fine-tune vs. stay-on-prompt>.
- **D5 — Shadow-to-canary decision.** <at end of M4, using <skew report, cost projection, latency report>, we will decide open canary vs. hold vs. abort>.
- **D6 — Ramp-step decisions.** <at each ramp step in M5, using <guardrail-metric report>, we will decide advance vs. hold vs. rollback>.
- **D7 — Cutline decisions.** <continuous; the ordered cutline list is §7.>

## 5. Dependencies

For each: dependency, owner, earliest-date, risk-if-slips, contingency.

- **Dep 1 — <platform primitive>.** Owner: <name on peer team>. Earliest date: <YYYY-MM-DD>. Risk: <what slips if this slips>. Contingency: <the fallback>.
- ...

## 6. Cost and headcount

- **Compute.** <GPU-months, CPU-hours, storage> — range, not point estimate.
- **Vendor.** <API tokens, licences>.
- **Headcount.** <named engineers × fraction of quarter × how many quarters>.
- **Total cost estimate.** <bounded range with the assumptions listed>.

## 7. Cutlines

Ordered list. If we lose <resource>, we cut <milestone> and produce the <fallback>.

- **Cut 1 — <resource loss>.** Cut: <milestone or scope>. Fallback: <what we still deliver>.
- ...

## 8. Risks

Three-to-five risks worth naming. For each, the mitigation.

- **Risk 1 — <risk>.** Mitigation: <the specific action>.
- ...

## 9. Open questions

Explicitly stated. Answerable. The reviewer can respond.

## 10. References

Links to the tradeoff docs, the RFC decisions, the prior roadmaps, the eval reports, the mod-308 paved-road inventory the dependencies reference.

## 11. Decision log

Populated as the roadmap is reviewed and as it evolves. Every decision-point outcome, every cutline exercised, every material change to sequence or scope.
```

Three properties of this template that are worth naming:

- **Outcome and non-goals come first.** Reviewers decide whether to read the rest in the first 200 words. Bury the outcome and they bounce.
- **Sequence and decision points are separated.** The sequence is the plan; the decision points are the places the plan branches. Keeping them separated makes the plan legible to a reviewer whose job is to check whether the *branches* are correct, not just whether the sequence is fast enough.
- **Cutlines are explicit.** The most common failure mode of an ML roadmap is that it is optimistic about resources and silent about what happens when the optimism is wrong. Cutlines force the plan to hold under adversarial staffing.

## §4 — ML-specific scoping traps

Four traps recur specifically in ML-initiative roadmaps that don't hit software-initiative roadmaps as often. The senior discipline is to catch them in your own draft before a reviewer does.

### 4.1 The "we'll figure out the eval later" trap

The most expensive trap. The roadmap commits to a model, a rollout, and a launch date — but leaves the evaluation harness (mod-305 chapter 01) as "we'll build it as we go." The result: the team is one week from launch when someone asks "wait, how do we know this is better than the baseline?" and the answer is a two-week eval-harness build that pushes the launch into the next quarter.

The fix: **milestone M0 (baseline lock) always includes the frozen evaluation harness**, with the primary metric chosen in advance, the guardrail metrics named, the CI shape declared, and the harness code merged. If the roadmap does not treat the harness as a milestone deliverable, the roadmap is not commitable.

### 4.2 The "model uncertainty as project uncertainty" trap

ML roadmaps confuse two different kinds of uncertainty:

- **Model uncertainty.** Will the sophisticated model beat the baseline? Nobody knows until the training runs. This is inherent to modelling and is what the decision points of §2.3 exist to structure.
- **Project uncertainty.** Will the data land on time? Will the platform primitives be ready? Will the on-call rotation absorb the launch? This is manageable — dependencies, cost estimates, cutlines all address it.

The trap is treating the second as if it were the first. "We can't predict when this will ship because ML is inherently uncertain" is a smokescreen for project management. The senior discipline is to *separate the two uncertainties* — model uncertainty gets decision points, project uncertainty gets dates, dependencies, and cutlines.

### 4.3 The "hero-project" trap

The roadmap commits the team to an ambitious, novel, single-track effort — a "moon shot." The narrative sounds great. In practice, the hero project consumes all the team's slack, blocks other maintenance work, and if it slips (as ambitious projects often do), the team has nothing to show for the quarter.

The fix: **at least 30 % of the team's quarterly capacity is reserved for maintenance, retraining, incident response, on-call, and the paved-road contribute-back work of mod-308**. The hero project is one initiative, not the initiative. The roadmap that reserves the maintenance slice is the roadmap that survives a bad quarter.

Larson's *[Systems of Engineering Management](https://lethain.com/elegant-puzzle/)* names this discipline as *systems, not heroes*: the sustainable version of a senior engineer's leadership is producing durable systems (of standards, of reviews, of maintenance), not a heroic quarter followed by burnout.

### 4.4 The "over-committed platform-dependency" trap

The roadmap counts on a peer platform team's Q2 deliverable. The platform team's roadmap counts on your team's Q2 feedback. Neither team has told the other, in writing, what they are counting on. Q2 arrives; the deliverable is late (or the wrong shape); both teams are surprised.

The fix: **every dependency in §5 has been confirmed with the named owner in writing** — a Slack acknowledgement is a floor, an email is better, a signed-off doc is best. mod-308 chapter 04's hand-off contract is the artefact the confirmation goes into. A dependency the peer team has not confirmed is a *risk*, not a *dependency* — treat it in §8, not §5.

## §5 — The pre-commit review

The roadmap is not written in isolation and shipped to the reviewer cold. Between draft and commit, the senior runs a *pre-commit review* — a small, invited-audience walk of the roadmap with the peers whose sign-off matters. The pre-commit is the moment where the roadmap catches its own weaknesses before the leadership-review meeting where it will be defended.

Three pieces of the pre-commit:

- **The peer walk (before the sponsor review).** Trade the roadmap with a senior on a peer team — platform, product, data, another ML team. The peer's job is not to approve; it is to *stress-test*. "If I were the director, I would ask you these three questions — how would you answer them?" This is the same discipline as mod-308 chapter 03's *sponsor conversation*, applied to the roadmap.
- **The staff-engineer or manager walk.** Trade with your manager or the staff engineer above you. They are the ones who will be sitting next to you in the leadership review defending the plan. They surface the org-context questions ("this cuts across the search team; have you talked to their lead? this touches next year's headcount plan; is finance aware?") that a peer reviewer will miss.
- **The sponsor pre-review.** The final round before the leadership-review meeting. The sponsor is briefed on the plan, sees the objections you've heard, sees your responses. The sponsor walks into the leadership review already knowing the answers.

The pre-commit is where the *cutlines* section usually gets sharpened. A first-draft cutlines section reads as aspirational; peer reviewers stress the assumptions and force the fallbacks to be specific.

## §6 — After commit: keeping the roadmap alive

The roadmap is committed. Now the harder discipline: keeping it *current* as reality diverges.

Three practices:

- **The weekly roadmap update.** A one-paragraph note appended to the decision-log section every week. What advanced, what slipped, what decision-point evidence landed, what cutlines were exercised (or came close to being exercised). The weekly note is the artefact the sponsor reads to know whether to trust the roadmap without asking. mod-308 chapter 04's *quarterly consumer report* discipline is the equivalent for peer-team feedback; the weekly roadmap update is the same discipline turned inward.
- **The decision-point close-out.** Every time the plan hits a decision point, the outcome is recorded in the decision log with the evidence used and the branch chosen. Decision points that come and go silently are decision points that later look like "we drifted into this" rather than "we chose this." The close-out is what makes the drift visible.
- **The mid-half review.** At the halfway point of the initiative (mid-quarter for a one-quarter initiative, end-of-Q2 for a two-quarter initiative), the roadmap is re-defended in a short review. What is on track, what has slipped, what has been cut, what evidence has landed at decision points, and — the most important question — *is the outcome still the right outcome*? Half of the mid-half reviews find that the world has shifted and the outcome needs re-framing; half find that the outcome is still right and the plan just needs an adjustment. Both outcomes are healthy; the pathological outcome is not running the mid-half review at all.

Roadmaps that die are almost always roadmaps that were committed and then not maintained. The commit is the beginning of the work, not the end.

## §7 — When the roadmap changes

Roadmaps change. New evidence arrives from a decision point; a peer team's deliverable slips; the product roadmap adjusts; the director asks for a scope change. The senior discipline is to *update the artefact*, not to run the initiative from a mental model that has diverged from the written plan.

Three change categories, with the discipline for each:

- **Adjustments within the plan.** A milestone slides by two weeks; a decision-point outcome shifts the sequence; a cutline is exercised. Recorded in the decision log; the roadmap is updated in place. No re-review — the sponsor is notified in the weekly update.
- **Scope changes within the outcome.** The outcome stays; the sequence changes materially. A new milestone is added; a sub-outcome is added or removed; a dependency is added or dropped. Recorded in the decision log; the sponsor is asked for a light re-approval — often a Slack thread or a 30-min sync.
- **Outcome changes.** The outcome itself moves — the target metric changes, the launch surface changes, the deadline changes materially, the initiative pivots (e.g., from "ship a fine-tuned model" to "ship a prompt-and-retrieval configuration"). This is a *re-commit*, not an update. The roadmap goes through the pre-commit review of §5 again. The old outcome is recorded in the decision log; the new outcome is dated and re-signed. Attempting to run an outcome change as an "adjustment" is the failure mode that eventually costs the sponsor's trust.

The discipline is to be *honest about the change category*. It is tempting to frame an outcome change as a scope change to avoid the re-commit; the cost is that the sponsor discovers the drift later and correctly asks why they were not told. Better to over-classify (call a scope change an outcome change and get the light re-commit) than to under-classify.

## Summary

A multi-quarter ML initiative roadmap is a decision artefact between the senior tech lead and the leadership who commit resources to the initiative. It answers four questions: what outcome are we committing to (falsifiable, bounded, downside named), what is the sequence and why this sequence (with the ML-specific milestone kinds — baseline lock, first-pass, data/infra unblock, sophisticated model, shadow, canary, full rollout, handoff), what are the decision points (at <milestone>, using <evidence>, decide <A> vs. <B> vs. <cut>), and what are the dependencies, cost, and cutlines. The template (§3) makes the roadmap commitable — outcome first, non-goals explicit, sequence and decision points separated, cutlines named. Four ML-specific traps ("we'll figure out the eval later," model-vs-project uncertainty confusion, hero project, over-committed platform dependency) recur; the senior catches them in their own draft. A pre-commit review (§5) with peers, staff engineers, and the sponsor sharpens the plan before the leadership-review meeting; a weekly update, decision-point close-outs, and a mid-half review (§6) keep the roadmap alive after commit. Roadmap changes are classified honestly (adjustment, scope change, outcome change), and outcome changes go back through the pre-commit review as a re-commit. The roadmap is the artefact leadership commits to; the discipline is producing one that survives the questions a director will ask cold.
