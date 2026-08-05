# Mentoring mid-levels and running the standards library

## Motivation

Chapter 02 named one specific mentorship arc — training a mid-level to run reviews. This chapter zooms out. Mentorship is one of the two load-bearing forms of L30 leverage that don't show up on the sprint board:

- **Mentorship** turns a mid-level engineer into someone who ships the initiative at the senior's bar — without the senior having to hold their hand on every PR.
- **The standards library** turns the senior's judgement calls into *written*, *searchable*, *cite-able* artefacts other engineers (and other teams) can adopt.

Both share the same underlying discipline: **encode the senior's non-obvious judgements into artefacts and relationships that outlast the senior's presence**. A senior who is the sole holder of "here's how we think about training/serving skew" is a bottleneck; a senior whose team's ML-standards page names the invariants, whose mid-level has been mentored into carrying them, and whose paved-road-adjacent library has been adopted by three other teams is a leverage multiplier.

Camille Fournier's *[The Manager's Path](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)* names this transition — the mentorship chapter is the reference for the mid-level growth arc; the tech-lead chapter is the reference for the standards-and-culture arc. Lara Hogan's *[Resilient Management](https://larahogan.me/management/)* and her essay *[Questions for our first 1:1](https://larahogan.me/blog/first-one-on-one-questions/)* are the reference on the *conversational* discipline of mentorship. Larson's *[StaffEng](https://staffeng.com/)* on the *tech lead* and *architect* archetypes describes what a senior-scaled-into-standards-work looks like from the L40 side.

This chapter has two halves. §§1–4 install the mentorship discipline: who to mentor, what to mentor them into, how the conversations run, how you tell whether it is working. §§5–8 install the standards-library discipline: what a standards library is, what it holds, how the standards get written and adopted, and how the library composes with the paved road (mod-308) and the review process (chapter 02).

## §1 — What mentorship is (and is not)

Mentorship at L30 is a specific relationship with a specific goal. Two properties define it:

- **The mentor has a *directional stake* in the mentee's growth.** You aren't just an occasional sounding board — you are trying to move a specific person from where they are to where they need to be. That directionality is what makes the mentorship an engineering-org accountability, not just a friendly favour.
- **The mentee is *not* a report.** You are not on the mentee's performance-review chain. You are not doing the mentee's manager's job. The manager owns compensation, ratings, and performance-management; the mentor owns skill growth, taste development, and career navigation. Fournier's *Manager's Path* is emphatic on this line: a mentor who blurs into management is a mentor whose feedback becomes filtered through performance-review politics.

Mentorship is *not*:

- **Pair programming with the label "mentorship."** Pair programming is a technique; it may be *part* of a mentorship, but a pairing relationship without a growth arc is not mentorship.
- **Office-hours availability.** "My door is always open" is a floor, not a mentorship. Mentorship is scheduled, structured, and has an arc.
- **Doing the work for the mentee.** Every "let me just take this one" is a moment of un-mentorship. The mentee grows by doing, not by watching you do.
- **The same as sponsorship.** Sponsorship is *speaking up for the mentee in rooms they aren't in* — proposing them for a project, naming them in a promo packet, insisting they lead the next review. Mentorship and sponsorship overlap but are not the same; a senior who mentors but never sponsors is only doing half the job. Lily Herman's *[Mentorship vs. Sponsorship](https://www.themuse.com/advice/whats-the-difference-between-a-mentor-and-a-sponsor)* is the tight external reference on the distinction.

## §2 — Who to mentor

At L30, you don't have infinite mentorship capacity — one or two mentees at a time is the sustainable range, with a third possible when one is winding down. The senior discipline is to pick mentees where the *return on your time* is highest, and to be honest with yourself about who benefits and who doesn't.

Three good default mentees:

- **The mid-level on your team who ships at bar but is stuck on the *next* seniority axis.** They can execute; they need to learn to design, or to review, or to write RFCs, or to run a design meeting. This is the *most common* L30 mentee. mod-301 chapter 05's self-assessment vocabulary is the shared framing.
- **The junior on your team you inherited.** They need scaffolding on the basics — how a PR gets reviewed, what a design doc looks like, how the eval harness is used. The mentorship is heavier but the growth is faster.
- **A cross-team mid-level whose team lacks a senior in your domain.** This is *cross-team mentorship* — you're not on their team's manager chain, but their manager and yours have agreed you'll help fill the gap. Common in orgs where ML seniority is thin. The relationship is looser and needs a written expectation up-front.

Three mentees to be honest about *not* taking on unless the fit is right:

- **The mentee whose manager is disengaged.** The manager owns performance; if the manager isn't paying attention, the mentorship absorbs work the manager should be doing. Fine to help the mentee, but escalate to your manager and theirs — it's a management gap, not a mentorship gap.
- **The mentee who wants career validation, not skill growth.** Some engineers seek out mentors as social proof (or as ammunition for a promo case). The signal is that the mentee argues back at *every* piece of feedback rather than trying it. Mentorship is not a debate club; if the mentee is not open to being changed, the mentorship is not working.
- **The mentee whose growth is blocked by something you can't help with.** Compensation gap, wrong team fit, life circumstance. The most valuable thing you can do is name it and refer them to the right person — their manager, HR, an EAP. Mentorship is not counselling.

## §3 — The mentorship arc

A useful mentorship has a legible shape. Below is the four-phase arc that works for most L20 → L30-adjacent growth trajectories. The phases are approximate — a mentee might spend a quarter or a year in each — but the *order* is stable.

### Phase 1 — Framing (weeks 1–2)

The first two conversations are about setting the arc up. Cover:

- **What are you trying to grow into?** Not "senior engineer" — the specific behaviours: "I want to run design reviews," "I want to write RFCs that ship," "I want to lead an initiative next year." The mentor's job is to make the goal concrete enough to be aimed at.
- **What's the evidence you'll use to know it's working?** A promo packet is one shape; a specific artefact ("I want to author the roadmap for X") is another; a role ("I want to be the tech lead of Y") is another. Name the evidence now so the mentorship isn't drifting.
- **What's the cadence?** 30 min every two weeks is the standard baseline. More frequent for a heavier arc; less frequent for a lighter one. Consistent cadence matters more than length.
- **What's *out of scope* for this mentorship?** Performance management, compensation, complaints about their manager. If the mentee needs to talk about those, they need a different conversation partner.
- **Where do notes live?** A shared doc — even a private one, even a bullet-list one — is what turns the mentorship from a series of conversations into an artefact the mentee can look back on. The mentee owns the doc; the mentor contributes.

### Phase 2 — Skill-specific loops (months 1–6)

Once the arc is framed, the mentorship becomes a series of *loops* — pick a skill, work on it deliberately for a few weeks, notice the growth, move to the next skill.

Typical L20→L30-adjacent skill loops for an ML engineer:

- **The RFC loop.** The mentee writes an RFC (mod-308 chapter 03). The mentor reviews it, in-doc, before it circulates. Second RFC follows a month later. By the fourth RFC, the mentor's comments have shifted from "you need to add an alternatives section" to "the alternatives section could be stronger by considering X."
- **The design-review-facilitation loop.** The mentee facilitates a design review (chapter 02 §5) with the mentor as a silent observer. The mentee reads the mentor's private notes after; next design review, the mentee tries the correction.
- **The eval-harness loop.** The mentee owns the harness for one initiative end-to-end (mod-305). The mentor is the reviewer.
- **The on-call-lead loop.** The mentee shadows the mentor on the next incident (mod-307 chapter 04); leads the next; runs the postmortem for the third.
- **The reviewer loop.** From chapter 02 §6 — the shadow-review to rotating-ownership arc.
- **The interview-panel loop.** From chapter 04 — shadow, co-interview, lead.

Each loop has a *starting move* (mentee does the thing with heavy scaffold), an *iteration* (mentee tries again with less scaffold), and a *close* (the mentee owns the skill without prompting). The mentor's discipline is to *notice when the loop has closed* and move on. A loop that stays open past its natural end is a mentorship that has quietly become supervision.

### Phase 3 — Ownership transfer (months 6–18)

At some point, the mentee starts owning things the mentor used to own. This is the phase where mentorship becomes hardest to run well — because it means *letting go*.

Two disciplines here:

- **Delegate real work, not fake work.** The mentee owns a real deliverable — an RFC that will actually ship, a design review that will actually decide, an interview loop that will actually hire. Fake ownership ("run this design review but I'll actually make the call") teaches nothing and erodes the mentee's trust.
- **Delegate the *decision*, not the *outcome*.** The mentee gets the authority to decide; the mentor stays informed and available. If the mentee's decision is defensible, you back it — even if you would have decided differently. Delegation with a veto is not delegation.

Larson's *[Delegate to the right task](https://lethain.com/using-strategy-for-alignment/)* and the [StaffEng *make delegation stick*](https://staffeng.com/guides/writing-engineering-strategy) essays are the load-bearing external references on this.

### Phase 4 — The mentorship winds down (month 12+)

Mentorships end. Some end because the mentee is ready to be someone else's mentor; some end because the mentee moves teams; some end because the arc has run its course and the relationship becomes peer-to-peer.

Two signals the mentorship should wind down:

- **The mentee is doing the work you set the loops up for, without the loop.** They write RFCs; they run reviews; they lead initiatives; they mentor others. The loops closed successfully. Time to formally note the wind-down — a conversation with the mentee, a note in the shared doc, an offer to stay in touch as peers.
- **The conversations are recurring without new content.** Same problems, same advice, no growth. If this persists for a couple of months, the mentorship has become supervision, and the honest move is to end it and refer the mentee to a different mentor or to their manager.

The wind-down is not a failure. A one-year mentorship that ends with the mentee having grown into an L30-adjacent role is a success; a five-year mentorship that never ends because it is a comfort routine is not.

## §4 — Tell whether the mentorship is working

Mentorships are hard to measure. The mentee's promotion is a lagging, coarse, noisy signal. Three leading indicators that the mentorship is on track:

- **The mentee's PRs and design docs need less scaffolding from you.** You can see it in the review comments — the ML-specific checks from chapter 02 §3 that used to require prompting are now present unprompted.
- **The mentee is quoted by their peers.** Other mid-levels start citing the mentee's opinion, referring PRs to them, asking them to review design docs. This is the sign that the mentee's judgement has become externally-visible.
- **The mentee is teaching other people.** They mentor a more junior engineer, or they run the review retro, or they write a section of the standards library. Mentorship growth eventually turns into mentorship *giving*.

Three failure signals:

- **The mentee starts avoiding conversations.** Reschedules, cancels, ghosts. Address it directly — "our conversations don't seem to be landing lately; is there something to change or should we pause?" — but be prepared for the answer to be "the mentorship isn't for me."
- **The mentee argues back at every piece of feedback.** A healthy mentorship has some argument; a mentorship where the mentee treats every piece of feedback as adversarial is one where growth has stalled.
- **You dread the conversations.** Mentors have finite emotional bandwidth. If you dread the recurring conversation, the mentorship is not sustainable for you — end it honestly rather than half-mentoring for months.

Lara Hogan's *[Feedback Equation](https://larahogan.me/blog/feedback-equation/)* is the reference on delivering hard-mentorship feedback in a way that lands rather than defends.

## §5 — The standards library: what it is

Every senior ML engineer accumulates *judgements* — "we always run a paired-bootstrap CI on the primary metric," "we never merge without a skew test on a feature-touching PR," "our shadow deployment always runs for at least two weekly cycles before canary." Left in the senior's head, these judgements have to be re-transmitted on every PR and re-invented by every new team.

A **standards library** is the artefact that makes the judgements *legible*, *reviewable*, and *adoptable*. It sits between the paved road (which is code) and the review process (which is a conversation):

- The paved road (mod-308 chapter 01) is *code* — an SDK, a manifest, a scheduler. It enforces standards by making the non-standard path harder.
- The standards library is *writing* — a set of short, opinionated documents naming the invariant, its reason, and its enforcement. It codifies the standard even where no code exists yet.
- The review process (chapter 02) is *conversation* — humans applying judgement in the moment. It picks up standards not yet in code or writing, and feeds the observations back into the library.

The standards library's discipline shows up strongest when a *second* team wants to adopt what your team does. Without a library, the answer is "well, come to our Slack and ask." With a library, the answer is a link — and the second team can adopt the standard in a week.

External references worth naming:

- **Google's [`google/styleguide`](https://github.com/google/styleguide)** — a public standards library for programming style. The shape (short, per-topic, opinionated, versioned) is the shape a team's ML-standards library should adopt.
- **Netflix's [*Paved Road* and *Full Cycle Developers* essays](https://netflixtechblog.com/full-cycle-developers-at-netflix-a08c31f83249)** — the load-bearing external reference on paved-road-plus-standards as a leverage strategy.
- **Etsy's engineering blog on their [technical documentation culture](https://www.etsy.com/codeascraft/etsys-experiment-with-immutable-documentation)** — an example of a company that treats standards documents as a first-class engineering artefact.
- **The [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)** — a canonical shape for a per-language standards library that other Rust libraries reference.

## §6 — What goes in the library

The library is opinionated but not exhaustive. Cover the *high-leverage* standards — the ones that catch expensive failure modes or that unlock other teams' adoption. Don't try to codify everything; the maintenance cost swamps the value.

For an ML team, the load-bearing sections are:

- **The ML-PR checklist.** The one from chapter 02 §3 (training/serving skew, eval harness, reproducibility, cost surface, rollout gate, model card). One paragraph per item, with a citation to the deeper reference.
- **The eval-harness contract.** What every offline eval report contains — primary metric, CI shape, guardrail metrics, slice matrix, baseline reference, harness version. mod-305 chapter 01 is the technical treatment; the library entry is the *one-page authoritative form* the team writes to.
- **The rollout playbook.** What shadow, canary, ramp, and full rollout mean on this team; the guardrail-metric thresholds; the rollback triggers. mod-306 chapter 01 is the technical treatment; the library entry is the *team-specific* form.
- **The runbook template.** What a runbook contains, at what depth, updated on what cadence. mod-307 chapter 04 is the treatment; the library entry is the *team's* template.
- **The model-card template.** The team's version of the mod-309 chapter 01 template — with the sections the team's regulatory context requires.
- **The RFC template.** The team's version of the mod-308 chapter 03 template.
- **The design-doc template.** The team's version of chapter 02 §5's design-doc contents.
- **The "when to escalate" guide.** From chapter 02 §7 — the team-specific version of what escalates to whom.
- **The naming conventions.** Feature names, metric names, model-version schemes, run-URI schemes. Deceptively load-bearing — a team without naming conventions spends 10 % of its review comments on naming.
- **The reproducibility contract.** Seeds, snapshot pinning, environment capture. Chapter 02 §3.3 is the checklist; the library entry is the enforcing document.

Two properties of a good library entry:

- **It states the standard, its reason, and its enforcement.** "Every ML PR must attach an eval-harness output. Reason: without it, the reviewer cannot verify the primary-metric claim (chapter 02 §3.2). Enforcement: chapter 02 §4's block rule; a CI check flags PRs missing the eval-harness artefact."
- **It is dated and owned.** "Last updated 2026-04-14 by <name>. Next review 2026-10-14." Standards without ownership drift into contradiction; the ownership fix is the same as the model-card cadence fix (mod-309 chapter 01 §7).

## §7 — Writing the standards that other teams adopt

Standards are easier to write than to *have adopted*. The senior discipline is that a standard is not adopted until it has *changed behaviour* — not just been written into a document. Three practices for adoption:

- **Write the standard where the reader already is.** If the team's engineers open the wiki, the library lives on the wiki. If they open the repo, it lives in `docs/`. The library that lives in a location the readers don't visit is a library nobody adopts. This is the *[docs where the code is](https://blog.rescale.com/technical-writing-for-engineers)* discipline.
- **Cite the standard in reviews.** Every PR-review comment that says "please add the eval-harness output" should link to the library entry that says why. Over time, the citations become the *canonical reference* and the standards-library entries become the shared vocabulary of the team's reviews. This is the compounding move — every review is either applying or improving the library.
- **Deprecate the standards that don't work.** A library entry that is routinely violated without pushback is a signal that the standard has drifted from practice. Two responses: fix the practice (re-emphasise the standard) or fix the standard (retire or amend it). Both are healthy; leaving the contradiction in place is not.

For **cross-team adoption**, the discipline shifts. Other teams won't adopt your team's standards by osmosis — they adopt them when the standard makes their life easier or when a peer team's standard becomes an industry consensus.

- **Package the standard as a template.** Not "here's how we do it" but "here's a template you can fork." The [Google Model Card Toolkit](https://github.com/tensorflow/model-card-toolkit)'s forkable Jupyter template is the shape.
- **Point at the standard from your paved road.** If your team's paved-road primitive (mod-308 chapter 01) refers to a library entry as the *canonical* contract ("the runbook must follow this template"), the primitive's other consumers inherit the standard without extra work.
- **Present the standard at the cross-team forum.** Every mature org has a forum — an ML guild, an architecture review, a monthly platform review — where cross-team standards get discussed. Bring the library entry, walk it, take the feedback, incorporate the feedback, and now the standard is *shared*, not just yours.
- **Accept the standard changes when other teams adopt it.** A standard that survives adoption unchanged is a standard nobody actually adopted — they just nodded and kept doing what they were doing. Adoption always adds nuance. Absorb the nuance into the library.

## §8 — Composing the library with the paved road and reviews

The three tools (paved road, standards library, reviews) form a compounding loop:

- **A standard is written in the library.** Its reason is documented; its enforcement is a review rule; its citation is available for reviewers.
- **Over time, if the standard is important enough, it becomes a paved-road primitive.** A `chronon-eval-report` CLI is more enforceable than a library entry that says "run these ten commands manually." mod-308 chapter 03's contribute-back-RFC discipline is the path from library-entry to paved-road.
- **The paved-road primitive frees the review from re-checking the standard by hand.** The reviewer trusts the primitive; the library entry becomes documentation of *why the primitive exists*. The review conversation moves up an altitude — it stops being about "did you run the eval harness" and starts being about "did the eval-harness output tell us what we needed to know?"

This is the *Team Topologies* pattern applied at senior-IC scale ([Skelton & Pais, 2019](https://teamtopologies.com/book)). Stream-aligned teams (your ML team) consume platform teams' paved roads; the *enabling* work — writing standards, mentoring mid-levels, contributing back to the platform — is what makes the platform improve for everyone.

The senior discipline is to *know which tool* fits each new standard:

- If the standard is *new and not yet clear*, a library entry is the right first move. Cheap; iterable; teaches the reviewer.
- If the standard is *established but not enforced*, the review process is the right enforcement — with the library entry as the citation.
- If the standard is *important enough to be always-on*, contribute-back to the paved road (mod-308 chapter 03) is the right long-term move. Expensive; slow; but takes the standard off the review's plate forever.

Reaching for the paved road too early codifies a standard that turns out to be wrong. Reaching for it too late leaves the review as the enforcement layer for standards that everyone agrees on and nobody wants to re-check by hand.

## §9 — Failure modes

Five failure modes recur in the senior's mentorship and standards work. The discipline is to catch them in your own practice.

- **The over-scoped mentorship.** You take on three or four mentees; each conversation is 15 min; none of them go deep. The fix: one or two active mentees at a time; the rest are polite decline or referral.
- **The un-owned standard.** The library entry has no name, no date, no next-review. It rots quietly. The fix: every entry has an owner and a next-review date; the entries missing them get archived or claimed at the next quarterly standards review.
- **The un-cited standard.** The library exists but reviewers don't cite it; authors don't read it. The fix: cite it religiously in reviews for a month; if the citation habit doesn't stick, the library entry is in the wrong place or written in the wrong voice.
- **The mentorship-as-supervision drift.** The mentee has been in "phase 2" for two years; the loops don't close. Either the mentee is not actually growing or you have taken on supervision the manager should own. Name it and address it.
- **The heroic standard.** You write a comprehensive, exhaustive, un-adoptable standard. Other teams nod politely and don't adopt. The fix: shorter, opinionated, adoptable-in-a-week, and shipped with a template.

The unifying discipline is that mentorship and standards work *scale you*. They fail when they become you-shaped work that only you can do — the exact opposite of leverage.

## Summary

Mentorship and the standards library are the two L30 forms of leverage that make the team's bar survive without the senior having to hold it PR-by-PR. Mentorship is a directional-stake relationship with a mentee who is not a report (§1), scoped to one or two active mentees (§2), running through a four-phase arc — framing, skill-specific loops, ownership transfer, wind-down (§3) — with legible leading indicators of growth and specific failure signals (§4). The standards library encodes the senior's non-obvious judgements into short, opinionated, dated, owned documents (§§5–6), and cross-team adoption requires the standard to be templated, cited in reviews, and presented at the cross-team forum (§7). The library composes with the paved road (mod-308) and the review process (chapter 02) as a compounding loop — new standards start as library entries, get enforced in reviews, and eventually contribute back to the paved road when they are important enough to be always-on (§8). Five failure modes (§9) — over-scoped mentorship, un-owned standard, un-cited standard, mentorship-as-supervision drift, heroic un-adoptable standard — share the underlying fix that mentorship and standards work *scale the senior*, and fail when they become you-shaped work that no one else can carry.
