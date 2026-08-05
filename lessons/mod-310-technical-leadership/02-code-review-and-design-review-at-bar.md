# Running code review and design review at bar

## Motivation

At L20 you *receive* reviews. A senior comments on your PR; you fix or push back; the PR merges. The rubric is one-sided — you learn what your team's bar is by having it applied to your work.

At L30 you *run* reviews. You are the senior comment on someone else's PR. You chair the design review that decides whether an approach ships. You are the reviewer of last resort — the person the mid-level engineer tags when they are stuck between two designs, the person the on-call escalates to at 2 AM when a fix looks urgent but the change is architecturally load-bearing. And you are training a mid-level engineer to run reviews too, so the team's bar survives your absence.

Two shifts define the L30 review discipline:

- **You know what blocks vs. what nudges.** Every review comment sits on a spectrum from "this is a suggestion" to "this cannot merge." Getting that classification right is what separates a reviewer who is respected from one who is either a bottleneck (blocks too much) or a rubber stamp (blocks too little). The senior discipline is *knowing when the comment is a block, and stating it as one — clearly, briefly, and with the reason*.
- **You review with the intent of raising the team's bar, not just this PR.** A mid-level engineer's PR is an opportunity to teach — a *specific* concept, a *specific* pattern, a *specific* piece of the team's engineering culture. The comment that fixes the PR but does not teach is a comment that will be re-earned on the next PR. The comment that teaches, one PR at a time, is what compounds into a team that reviews at bar even when you are not looking.

Two primary external references anchor the chapter:

- **[Google's Engineering Practices — Code Review Developer Guide](https://google.github.io/eng-practices/review/)** and its paired *[Reviewer](https://google.github.io/eng-practices/review/reviewer/)* and *[CL Author](https://google.github.io/eng-practices/review/developer/)* handbooks. The canonical published treatment of what a code review is, what a reviewer is looking for, and how to hold the bar without becoming a bottleneck. Every subsequent code-review guide (Microsoft's *[Engineering Fundamentals — Code Review](https://microsoft.github.io/code-with-engineering-playbook/code-reviews/)*, GitHub's *[About pull request reviews](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)*, the [SmartBear *State of Code Review*](https://smartbear.com/state-of-code-review/) surveys) is a derivative or elaboration of this shape.
- **Karl E. Wiegers, *[Peer Reviews in Software: A Practical Guide](https://www.processimpact.com/pubs.shtml#reviews)*** and the follow-on Wiegers & Wiley *[Peer Reviews in Software](https://www.pearson.com/en-us/subject-catalog/p/peer-reviews-in-software-a-practical-guide/P200000000646)* — the foundational treatment of the review as an *engineering discipline*, not just a courtesy. The Fagan-inspection origin ([Michael Fagan, IBM, 1976](https://www.mfagan.com/pdfs/ibmfagan.pdf)) is the ancestor of all modern code-review practice.

This chapter uses those references but keeps the focus tight on the *ML-specific* shape of the review — what a reviewer of an ML PR is looking for that a reviewer of a generic backend PR would not, and what a reviewer of an ML design doc is looking for that a reviewer of a service-architecture doc would not.

## §1 — What a review is for

The senior reviewer holds three purposes in mind at once. When the purposes conflict, the discipline is to name which one is winning and why.

- **Correctness.** The change does what it claims. It does not introduce bugs. Its unit tests, integration tests, and — for ML — its eval-harness runs (mod-305) pass. Its rollout plan (mod-306) will detect regression before it hurts. This is the *floor* the review is holding against. A change that fails the correctness bar does not merge.
- **Bar-holding.** The change matches the team's standards — for structure, naming, testing, telemetry, review packet, on-call ownership. Bar is what the paved road (mod-308 chapter 01) and the standards library (chapter 03 of this module) codify. Changes that are individually correct but violate the standard weaken the standard for the next PR; the reviewer's job is to hold the line.
- **Teaching.** The review is one of the highest-bandwidth teaching moments the senior has with a mid-level engineer. A comment that fixes the PR while explaining *why* — with a link, an example, or a "here's how our team thinks about this" — trains the author to do it right unprompted the next time. A comment that only says "please change this" leaves the teaching for the reviewer to repeat.

The three purposes are related but *not* the same:

- A correct change can miss the bar (works but not up to standard) — the review nudges it toward the standard.
- A bar-holding review can teach or not teach — the discipline is to teach.
- A teaching moment can happen without a bar violation — a "here's a pattern you might find useful" comment on a solid PR is teaching without blocking.

The **[Google reviewer's guide's *speed of code reviews* essay](https://google.github.io/eng-practices/review/reviewer/speed.html)** is the load-bearing external treatment of the tradeoff — reviews that are too slow hurt the team's velocity, but reviews that are too fast miss the bar and leave the teaching undone. The senior discipline is to keep first-response times short (one business day is the industry-standard ceiling) and to make each response substantive.

## §2 — What blocks vs. what nudges

Every review comment sits on a spectrum from *block* (this cannot merge) to *nudge* (a suggestion for now or later). Getting the classification right is the load-bearing skill.

The Conventional Comments format ([conventionalcomments.org](https://conventionalcomments.org/)) is a lightweight discipline that names the classification in the comment itself. The categories:

- **`issue` (block).** A defect the reviewer believes must be fixed. "The eval-harness call is missing the paired-bootstrap CI shape; the primary-metric claim in the PR description won't be defensible without it." Blocks the PR.
- **`suggestion` (nudge).** A change the reviewer would make but is not blocking on. "You could factor the loss-function normalisation into a helper; not blocking." Does not block.
- **`nitpick` (nudge, minor).** A trivial preference — a naming choice, a whitespace. Never blocks. Explicit `nit:` prefix is a service to the author.
- **`question` (clarify).** The reviewer needs to understand something before they can classify their concern. Blocks only until the answer clarifies whether the underlying concern is a `issue` or a `suggestion`.
- **`thought` (share).** A related observation the reviewer wants to record — often for follow-up work, sometimes for the team's shared understanding. Does not block.
- **`praise` (encourage).** A specific thing the author did well. Does not block; costs almost nothing to write; changes the tone of the review.

Three properties of the classification discipline:

- **Every comment has a category.** If the author reads a comment and cannot tell whether it is blocking or optional, the review has failed. The category prefix (`issue:`, `nit:`, `question:`) is the fix. Culture matters more than tooling — a team where "please consider" is understood as blocking and "you must fix" is understood as optional has an unwritten culture that a new engineer cannot learn from the comments.
- **A block has a reason.** "This does not meet the bar" is not enough. The block-quality comment names the *rule*, cites the *standard*, and *ideally* points at the fix. "The offline eval-harness is missing the guardrail-metric slice for the `region=EU` cut; our standard (link) requires guardrails on any protected slice, and the failure mode is undetected regression on that slice. Fix: add the slice to the harness config."
- **A block is not personal.** The block is about the change, not the author. "This PR is missing X" is fine; "you always miss X" is not. The Google reviewer's guide's *[handling pushback](https://google.github.io/eng-practices/review/reviewer/pushback.html)* essay is the reference on holding a block professionally when the author disagrees.

The senior discipline is that most reviews land 1–3 blocks, 3–8 nudges, and 1–2 pieces of praise. A review with zero blocks on a substantial PR is either an excellent PR or a rubber-stamp review; a review with ten blocks is often a review that should have been a design conversation before the code was written. Both extremes are worth reflecting on.

## §3 — ML-specific things to look for in a code review

Beyond the generic checks — correctness, readability, tests, structure — an ML PR has a set of specific things a senior reviewer scans for. The list below is not exhaustive; it is the ML-specific delta on top of a general code-review checklist.

### 3.1 Training/serving skew surface

Any change that touches feature computation, feature-store access, or model input assembly is a training/serving skew risk (mod-302). The reviewer scans:

- Is the *same code path* computing the feature at training time and at serving time? If they diverge, is the divergence covered by an explicit skew test?
- Are the *feature versions* pinned? Silent bumping of a shared library that changes tokenisation, normalisation, or aggregation is the classical source of overnight regressions.
- Are the *point-in-time* semantics preserved? Features computed at serving time but not at training time (or vice versa) create leakage.

A PR that changes feature code without touching (or without acknowledging) the skew surface is a PR that goes back for a rewrite or a review-packet update.

### 3.2 Evaluation harness discipline

Any PR that changes the model, the loss, the training data, or the serving pipeline needs to compose with the offline eval harness (mod-305). The reviewer scans:

- Is the *primary metric declared in advance* (mod-305 chapter 01) and reported with a CI?
- Are the *guardrail metrics* run, and are the slice cuts the team requires (mod-305 chapter 02) present?
- Is the *baseline lock* — the frozen model, the frozen data snapshot — referenced by URI/hash?
- If the PR introduces a new dataset or a new slice, is that composition-friendly with the existing harness, or is it a one-off script that will be forgotten by the next PR?

Two red flags: an ML PR without an eval-harness output attached (or referenced), and an ML PR whose eval output shows the primary metric moving without the guardrails being reported. Both are `issue`-level blocks.

### 3.3 Reproducibility and seeds

Non-determinism is the enemy of the review. The reviewer scans:

- Is the random seed *pinned* and *logged*? If the training was run with the wrong seed, the review cannot verify the number.
- Is the *data snapshot* pinned and hashed? Live data flow is fine at serving time; at training time it means the review cannot reproduce.
- Is the *training command* recorded — command line, environment, library versions — in a way the reviewer could run themselves?

The [Papers with Code *ML Reproducibility Checklist*](https://paperswithcode.com/rc2022) is the canonical form; the team's shorter version lives in the standards library (chapter 03).

### 3.4 Cost surface

Every ML change has a cost profile that generic backend changes don't. The reviewer scans:

- Is the *serving cost* named? "This adds a re-ranker call at request time; estimated added cost ~$X per million requests."
- Is the *training cost* named? "This training run consumes ~N GPU-hours; the retraining cadence (mod-307 chapter 05) will consume ~M per week."
- Is the *quota headroom* considered? An ML PR that pushes a vendor-API-token budget past 80 % without an ask is a `issue`-level block — it is priming an outage.

mod-307 chapter 03's cost-and-quota discipline lives here in the review.

### 3.5 Rollout gate and runbook

Any change that will be exposed to production traffic needs to name the gate and the runbook (mod-307). The reviewer scans:

- Is the *rollout plan* named — shadow, canary, ramp steps, guardrails, rollback trigger? mod-306 chapter 01 is the source.
- Is the *runbook* updated for the change? Reference to the on-call playbook, alerts, dashboards.
- Is the *SLO document* still accurate? A change that shifts the p99-latency budget by 20 % should update the SLO doc, not silently consume the error budget.

An ML PR whose only sign of production readiness is "we'll watch the dashboards for a day" is not ready to merge.

### 3.6 Model card and review packet

For a change that promotes a model version, the review packet (mod-309 chapter 01) is a co-artefact. The reviewer scans:

- Is the *model card version* updated?
- Are the *evaluation-harness reports* re-referenced?
- Are the *fairness / robustness / safety slices* re-run and any regressions flagged?
- Is the *cadence policy* still valid — has anything triggered a re-review outside the model cadence?

A promotion PR whose model card is unchanged from the last version is a block, not a nudge — the promotion cannot be reviewed without the artefact it produces.

### 3.7 The review's own composition

The senior reviewer also checks the review packet the PR itself carries — a description, a test plan, the eval-harness output, the shadow-run trace, the mod-309 packet update. A PR whose description is one line is a PR whose reviewer will have to reverse-engineer the intent — that is *not* something the reviewer should just absorb. It is a comment: "Please expand the description; a reviewer should not have to reverse-engineer intent from the diff."

## §4 — What blocks vs. what nudges — the ML rules of thumb

The four categories above have a rough tendency toward blocking or nudging. The rules of thumb, with the reasoning:

- **Correctness bugs (any code) → block.** Always. This is the floor.
- **Training/serving skew surface without a skew test → block.** The failure mode is silent, expensive, and hard to detect once shipped.
- **Missing eval-harness output on a model-changing PR → block.** No way to review the claim.
- **Missing primary metric declaration or missing CI shape → block.** The claim is not defensible.
- **Missing guardrail-slice coverage on a protected slice → block.** Fairness/reliability regressions in silence.
- **Non-pinned seeds or non-pinned data on a training PR → block.** Reproducibility floor.
- **Cost surface not named on a serving change → nudge unless projected cost exceeds team budget → block.** The team's cost SLI catches the drift eventually, but the reviewer is the cheaper catch.
- **Rollout gate not named → block on any traffic-exposed change. Nudge on a pure offline change.**
- **Model card / review packet not updated on a promotion PR → block.**
- **Naming, style, factoring, minor refactors → nudge.** Almost never block. Style disagreements should be resolved in the standards library (chapter 03) once and not re-litigated per PR.
- **Suggestions for follow-up work → nudge, `thought:` prefix.** Do not block on future work that is not this PR's job.

The senior discipline is to be *predictable* about the blocks. A team that knows which failure modes will block reliably has authors who avoid those failure modes proactively. A team where the blocks feel arbitrary — this reviewer blocks on X, that one on Y — has authors who lose trust in the review.

## §5 — Design review at bar

Code review holds a specific change to the bar. **Design review** holds the *approach* — the architecture, the algorithm choice, the data plan — to the bar *before* the code is written. The senior discipline is that a well-run design review saves ten to a hundred hours of code-review pain.

A design doc for an ML change carries specific sections beyond the generic engineering design doc:

- **The problem statement, in outcome terms** — the same discipline as the roadmap outcome (chapter 01 §2.1).
- **The evaluation plan** — the harness, the metrics, the slices, the CI shape.
- **The data plan** — the training data source, the labelling strategy, the volume expectation, the freshness contract.
- **The modelling plan** — the algorithm choice, with alternatives considered. mod-303's advanced-modelling vocabulary lives here.
- **The serving plan** — the online architecture, the training/serving skew mitigation, the SLO contract.
- **The rollout plan** — shadow, canary, ramp, guardrails, rollback trigger.
- **The alternatives** — including "keep the baseline." A single-alternative design doc is advocacy, not design (same lesson as mod-308 chapter 03 §5).
- **The risks and mitigations.**
- **The dependencies** — feature-store views, platform primitives, peer-team deliverables.

The design review meeting itself is a facilitated conversation. Two failure modes to guard against:

- **The design review that becomes an approval theatre.** Everyone shows up; the author presents the slides; no one asks the hard questions; the doc is "approved" and the author writes the code. The senior discipline is to *plant the hard questions in advance* — a pre-review conversation with a peer, a pre-circulated set of reviewer questions, a designated "devil's advocate" for the meeting. Google's [design-doc culture](https://www.industrialempathy.com/posts/design-docs-at-google/) has an internal norm that a design review with no substantive comments is treated as a *failure of the review*, not a success of the design.
- **The design review that becomes a re-write meeting.** The doc is 40 % done; the reviewers spend the meeting trying to fill in the missing sections; the author leaves with more work than they came in with. The senior discipline is to *refuse to review* an under-baked doc. "Come back when the alternatives section is written; I can't review the design against alternatives that aren't on the page." This is not rudeness — it is respect for everyone's time.

The design review's decision surface is broader than a code review's: **approve as written, approve with named revisions, reject with rationale, or send back for a re-scope**. The design-review outcome is recorded in the doc's decision-log (the same shape as the RFC decision-log of mod-308 chapter 03) so a reader six months later can reconstruct the choice.

## §6 — Mentoring a mid-level to run reviews too

The senior discipline is not to *be* the review — it is to *make the team a reviewing team*. A mid-level engineer running reviews at bar is what makes the senior's leverage exceed the senior's throughput.

Three practices for training a mid-level into a reviewer:

- **Shadow reviewing.** The mid-level does a full review on a PR, in a private thread with the senior, before commenting on the PR. The senior reads the mid-level's would-be comments, adds their own, and the two calibrate. After a few rounds, the mid-level's comments start matching the senior's; the mid-level begins commenting on PRs directly. Shadow reviewing is the code-review equivalent of the [ATLAS *pilot / co-pilot* structure for interview training](https://interviewing.io/blog/how-to-shadow-interviews) that chapter 04 will reuse.
- **Rotating review ownership.** Each week, one PR is *owned* by the mid-level — they are the reviewer of record, and the senior is a silent second reviewer for backup only. Over time, the number of PRs owned by the mid-level grows. This is the practice that eventually retires the senior from being the reviewer of last resort.
- **The review retro.** Once a month, the team looks at the last four weeks of PRs and asks: what did we block that we should have nudged? What did we nudge that we should have blocked? What did we miss? What did we teach well? The retro is the artefact that makes the team's bar improve *as a team*, not as a series of individual reviewers. Wiegers' *[Humanizing Peer Reviews](https://www.processimpact.com/articles/humanizing_reviews.pdf)* is the reference on running review retros that don't turn into blame sessions.

The reviewer-training arc has a legible target: the mid-level's reviews cover the checklist in §3, the block-vs-nudge classification in §2, and the design-review discipline in §5, without prompting. When that target is met, the mid-level is running reviews at bar — and the senior's own bar-holding has scaled from one person to two. Chapter 03 develops the mentorship arc more broadly; this chapter's slice of it is the reviewer-training piece.

## §7 — When to escalate

A reviewer of last resort is not the same as a reviewer of every disagreement. Most review disagreements are resolved between the author and the reviewer. Sometimes they escalate — and the senior's discipline is to know when.

Escalation is appropriate when:

- **The disagreement is about the standard, not the change.** "The author and I disagree on whether we should ever land ML PRs without a paired-bootstrap CI." This is a standards-library conversation (chapter 03), not a PR conversation. Take it out of the PR thread; land the PR either way once the standards question has been decided elsewhere.
- **The disagreement is about scope.** "The author thinks the missing dependency is a follow-up; I think it's a block." Ask the tech lead of the initiative or the senior on the initiative's roadmap (chapter 01). Their call.
- **The disagreement is about risk classification.** "I think this is a governance-visible change; the author disagrees." mod-309 escalation path; the governance-analyst peer track's opinion decides.
- **The disagreement is about bar.** "The author's PR is at their bar; I think it's below the team's bar." This is the hardest — the senior discipline is to hold the line professionally, cite the specific standard, and, if the author still disagrees, escalate to another senior or the tech lead. Never merge below-bar work in exchange for peace; the cost is paid on the next PR when the author (correctly) infers that the bar drifted.

The Google *[handling pushback](https://google.github.io/eng-practices/review/reviewer/pushback.html)* essay is the load-bearing reference: the reviewer's job is to hold the bar, but the reviewer is also accountable to the team's velocity. Neither wins outright; the discipline is to know when to yield on a nudge and when to stand on a block.

## §8 — Failure modes to catch in your own reviewing

Five failure modes recur in the senior reviewer's own practice. The discipline is to notice them in yourself before the team does.

- **The bottleneck reviewer.** Your reviews are thorough, but they land after two days. The team's velocity slows around you. The fix: shorter response time (1 business day for the first response, even if the substantive review takes longer); more delegation (chapter 06); fewer "I'll get to it" queues.
- **The rubber-stamp reviewer.** Your reviews land fast but shallow. Bugs pass through. Standards drift. The fix: block the calendar for review time (30 min/day is the industry-standard floor); use the ML-specific checklist of §3; do fewer reviews if that's what quality requires.
- **The teaching-by-fixing reviewer.** You end up rewriting the PR yourself in review comments. The author learns nothing except that you'll fix it. The fix: comment on the *pattern* the fix would use, not the fix itself; let the author land the fix; let the second-round review be shorter because the author has understood the pattern.
- **The re-litigating reviewer.** Every PR re-argues the same standards questions. The team's culture erodes because the standards are not written down. The fix: land the standards in the standards library (chapter 03); refuse to re-litigate at PR time; point at the library.
- **The exhausted reviewer.** You are the only reviewer for a class of PRs. Every PR of that shape comes to you. Your throughput is a hard ceiling on the team's throughput of that shape. The fix: train mid-levels (§6); split the review ownership; document the checklist so more people can pick it up.

The five failure modes have the same underlying fix: **the senior reviewer's leverage is the team's ability to review at bar, not the senior's own review throughput**. Every review is a marginal step toward or away from that leverage.

## Summary

Running code review and design review at bar is what separates the senior from the mid-level whose PRs merely land: the review is *for* correctness, bar-holding, and teaching, held in that priority when they conflict; every comment carries an explicit *block* vs. *nudge* classification (§2) so the author knows what to act on; the ML-specific checklist (§3) covers training/serving skew, eval harness discipline, reproducibility, cost surface, rollout gate, and model-card / review-packet integrity; the rules of thumb (§4) make the blocks predictable — correctness bugs, unmitigated skew surface, missing eval-harness output, missing primary-metric CI, missing guardrail-slice coverage, non-pinned seeds/data, missing rollout gate on a traffic-exposed change, missing model-card update on a promotion PR. Design review (§5) holds the *approach* to bar before the code is written — the load-bearing artefact is the design doc's alternatives-considered section — and the design-review meeting is facilitated to avoid approval-theatre and re-write-meeting failure modes. The senior mentors a mid-level to review too (§6) via shadow reviewing, rotating review ownership, and the monthly review retro — the target is the mid-level covering the checklist and the block-vs-nudge discipline without prompting. Escalation (§7) is appropriate for standards, scope, risk classification, and bar disagreements the author-reviewer pair cannot resolve. Five failure modes (§8) — bottleneck, rubber-stamp, teaching-by-fixing, re-litigating, exhausted — recur in the senior's own practice; the shared fix is that leverage lives in the *team's* review capacity, not the individual's.
