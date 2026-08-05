# exercise-03: Mentorship plan for a mid-level engineer

**Estimated effort:** 3 hours

## Objective

Author the **full written mentorship plan** for a specific mid-level engineer's growth arc, at the shape and depth chapter 03 §3 requires — the framing conversation, the skill-specific loops for a chosen quarter, the ownership-transfer plan, the wind-down triggers, and the calibration checkpoints. Draft the shared mentorship doc and the first-conversation agenda so the mentorship could actually start on Monday.

Plus: author a **companion candidate entry** for the team's standards library (chapter 03 §6) — one library entry that would encode a piece of judgement you'd otherwise have to re-transmit to every mentee.

The output is two artefacts: `mentorship-plan-<mentee-name>-<quarter>.md` and `standards-<topic>.md`. Together they demonstrate that you can *scale yourself through both a person and an artefact* — the two load-bearing forms of L30 leverage that chapter 03 names.

This is the L30 tell that separates "I answer questions in Slack when asked" from "I have a written, structured mentorship arc for a specific engineer and a standards library that outlasts my involvement." An L20 helps; an L30 scales.

## Prerequisites

- Read chapter 03 (`03-mentorship-and-standards-library.md`) end-to-end — mentorship vs. sponsorship vs. supervision, who to mentor, the four-phase arc, leading indicators, the standards library, what goes in it, how it composes with the paved road and the review process, and the five failure modes.
- Skim [mod-301 chapter 05](../../mod-301-senior-ml-role-scope/05-self-assessment-and-growth-plan.md) — the self-assessment vocabulary that structures the mentee-side goal framing.
- Skim chapter 02 §6 of this module (`02-code-review-and-design-review-at-bar.md`) — the reviewer-training loop is the most common first skill loop for a mid-level mentee.
- Skim Camille Fournier's *[The Manager's Path](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)* — the mentorship chapter for the manager-vs-mentor distinction; the tech-lead chapter for the arc framing.
- Skim Lara Hogan's *[Questions for our first 1:1](https://larahogan.me/blog/first-one-on-one-questions/)* and *[The Feedback Equation](https://larahogan.me/blog/feedback-equation/)* — the conversational discipline.
- (Optional but valuable) Skim Kim Scott's *[Radical Candor](https://www.radicalcandor.com/)* — specifically the *care personally / challenge directly* frame that mentorship feedback lives in.

## Pick your mentee

Pick **one** of the following scenarios, or bring your own (M5 with a preamble on the real mentee's context, anonymised as needed):

- **M1 — The "ready-to-jump" mid-level.** Mid-level ML engineer, two years on your team, ships at bar on individual PRs. Their manager and yours have both signalled they should be growing toward L30-adjacent responsibilities in the next 12–18 months. They've never authored an RFC, never facilitated a design review, and haven't led an initiative. The mentorship's directional goal: get them running RFCs and design reviews by end of year.
- **M2 — The "strong IC, weak reviewer" mid-level.** Mid-level ML engineer, one year on your team, individual ML depth is strong but their code-review comments are inconsistent — sometimes shallow, sometimes over-blocking. They've been asked to be a required reviewer on the team's ML PRs and are struggling. The mentorship's directional goal: get them reviewing at bar (chapter 02's rubric) within two quarters.
- **M3 — The "cross-team senior-adjacent" mid-level.** Mid-level ML engineer on a peer team that lacks a Senior ML Engineer in your domain. Their manager and yours have agreed you'll be a cross-team mentor. They ship at bar; the challenge is that their team's engineering-culture standards differ from yours (weaker eval harness discipline, no standards library), and they're navigating whose bar to hold. The mentorship's directional goal: help them build the *first version* of an ML-standards library on their team.
- **M4 — The junior-just-past-onboarding.** Junior engineer, six months in, has landed their first PRs but is still on scaffolded work. Their manager asked you to mentor them through their first year. The mentorship's directional goal: get them to solo-owning a small feature end-to-end (design → PR → rollout → runbook) by end of year.
- **M5 — Bring-your-own.** A real mentee (current or planned). Anonymise; add a preamble covering the mentee's tenure, their manager, the specific growth target both of you have agreed on, and any known constraints (life-events, workload, org context).

## Steps

### 1. The framing conversation prep (≈ 25 min)

Draft the *first-conversation agenda* (chapter 03 §3, phase 1). The agenda covers:

- **Growth-target framing.** Not "senior engineer" — the specific behaviours the mentee will grow into. Draft 2–3 candidate framings the first conversation will pick between (e.g., "author RFCs that ship," "run design reviews," "lead the Q3 initiative").
- **Evidence question.** How will you both know it's working? A promo packet? A specific artefact ("I authored the roadmap for X")? A role ("I'm the tech lead of Y next year")? Draft 2–3 candidate evidence shapes.
- **Cadence proposal.** 30 min every two weeks is baseline. Adjust based on the mentee's tenure and the arc's intensity. Draft your proposed cadence with rationale.
- **Out-of-scope statement.** What this mentorship deliberately does not cover — performance management, compensation, complaints about their manager. Write this in the specific words you'd use in the conversation.
- **Shared-doc location.** Where does the mentorship doc live? Draft the doc's opening template.

Discipline: the first-conversation agenda is not a monologue — it's a set of *questions and proposals*, so the mentee co-owns the framing. If the agenda reads as "here's what we'll do," rewrite it as "here's what I'm proposing; how does this land with you?"

### 2. Mentorship-doc opening (≈ 15 min)

Draft the shared mentorship doc's opening section — the framing, the target, the evidence, the cadence, the out-of-scope, and a placeholder for the *skill-loop history*. This is the doc the mentee owns; the mentor contributes.

The opening section should be filled with the *drafts* from step 1, marked as "for discussion in first conversation," so the mentee can push back before anything is committed.

### 3. Skill-loop selection and design (≈ 45 min)

Chapter 03 §3 phase 2 lists typical L20→L30-adjacent skill loops: the RFC loop, the design-review-facilitation loop, the eval-harness loop, the on-call-lead loop, the reviewer loop, the interview-panel loop.

For your chosen mentee scenario, pick **two skill loops** to run in the next quarter. For each loop, author:

- **Loop name and target.** "The reviewer loop — mentee runs ML PR reviews at bar (chapter 02 §3 checklist, block-vs-nudge classification) without prompting."
- **Loop starting move.** The first activity — usually a shadow round with heavy scaffold. E.g., "Mentee shadows the mentor's next three ML PR reviews; mentee writes their own would-be comments in a private thread before the mentor comments on the PR; mentor + mentee walk the deltas."
- **Iteration structure.** How the loop escalates. E.g., "After three shadow rounds, mentee co-reviews one PR (they leave half the comments; mentor leaves the other half). After two co-reviews, mentee leads the review (mentor is silent second). After three led reviews with clean calibration, the loop closes."
- **Close criteria.** How you'll know the loop is done. Specific and observable — "mentee's next four consecutive ML PR reviews carry all seven checklist areas without prompting, with block-vs-nudge classification matching mine on 80%+ of comments."
- **Time estimate.** How many weeks. Loops that stretch beyond 8–10 weeks usually indicate an unclear target or an under-invested mentee.

Two properties of well-designed loops:

- **Loops nest inside real work.** Don't invent synthetic exercises — the loop happens on the team's real PRs, real design reviews, real incidents. Synthetic loops teach the mentee that mentorship is a game separate from work.
- **Loops have a close.** A loop without a close criterion is a loop that becomes supervision. If you cannot articulate how you'll know the loop is done, redesign it.

### 4. Ownership-transfer plan (≈ 20 min)

Chapter 03 §3 phase 3 is the *ownership-transfer* phase. For your chosen mentee, name one *real* piece of work (in the current or next quarter) you would transfer ownership of once the skill loops close. This is not a hypothetical — pick a concrete artefact or responsibility:

- An RFC you were going to author, now owned by the mentee.
- A design review you were going to facilitate, now facilitated by the mentee.
- An interview loop round you were going to lead, now led by the mentee.
- A standards library entry you were going to write, now co-authored by the mentee.

Write the ownership-transfer plan in the shape:

- **The work being transferred.** Specific artefact or responsibility, with the deliverable.
- **The transfer trigger.** What has to happen before the transfer (which loop must close first, what calibration signal must land).
- **The mentor's role after transfer.** Silent backup? Named consultant? Officially uninvolved? The chapter 03 §3 principle is "delegate the decision, not the outcome" — but the mentor's continuing role should still be named.
- **The failure plan.** What happens if the mentee falters. Is there a fallback owner? A timeline for you to re-engage? Silent failures are the biggest risk of ownership-transfer.

### 5. Wind-down triggers and calibration checkpoints (≈ 15 min)

Chapter 03 §3 phase 4 is the *wind-down*. Author:

- **Wind-down triggers.** The signals that indicate the mentorship's specific arc has run its course. Chapter 03 §3 names two: the mentee is doing the work without the loop, and the conversations have become recurring without new content. Adapt to your scenario. Include a target end date range (e.g., "the mentorship's arc closes when the two skill loops of §3 close and the ownership-transfer of §4 has been demonstrated at bar; expected timeline: 4–6 months").
- **Calibration checkpoints.** Regular (monthly? quarterly?) explicit moments where you and the mentee review the arc together against the leading indicators of chapter 03 §4. What's working, what's not, what to change.

### 6. First-conversation agenda (≈ 15 min)

Now write the actual **agenda for the first mentorship conversation**, at a level of detail you would walk into the meeting with. 30 min, structured as:

- **First 5 min — set the mode.** Non-transactional opening; check on the mentee's context and workload before diving into arc-framing.
- **Next 15 min — walk the arc drafts.** Growth target options, evidence options, cadence proposal, out-of-scope statement. Ask the mentee's reaction to each; adjust in-conversation.
- **Next 5 min — surface the mentee's own priorities.** "What are you working on this quarter that you'd want to be part of this? What's a growth area you were going to raise if I didn't?" The mentee should leave feeling their agency is respected.
- **Last 5 min — commit to the mentorship doc.** Where it lives; what goes in the opening section; the next conversation's date.

The first-conversation agenda is a script (bullet-point, not word-for-word) that lives in the mentor's own private notes. It is *not* shared with the mentee before the meeting — it's the mentor's preparation.

### 7. Standards-library candidate entry (≈ 30 min)

Chapter 03 §5–8 names the standards library as the other L30 leverage tool. For your mentee, identify one specific piece of judgement you would otherwise have to re-transmit to every mentee — a rule about how to author eval-harness reports, a naming convention for feature-store views, a template for RFC opening sections, a runbook contents checklist.

Author the full library entry, following chapter 03 §6's format:

- **Title.** One-line, descriptive.
- **The rule.** One-to-two sentences. The invariant the library holds.
- **The reason.** One paragraph. Why this rule exists — the failure mode it prevents or the coordination it enables.
- **The enforcement.** How the rule is enforced — review-time check (link to chapter 02 §3 area), CI check (script or paved-road primitive), template (with a link).
- **The cross-references.** Related library entries, related chapters, related paved-road primitives.
- **Metadata.** Owner name, last-updated date, next-review date (6–12 months out).

Discipline: the entry should be *short* (half a page to one page). A four-page library entry is a chapter, not a library entry. Write shorter, cite more.

### 8. Reflection (≈ 15 min)

Write a short (half-page) reflection on the mentorship plan and the library entry:

- **Which of chapter 03 §9's five failure modes am I closest to?** Over-scoped mentorship, un-owned standard, un-cited standard, mentorship-as-supervision drift, heroic un-adoptable standard. Name any that you can see risk of in your own plan.
- **What would tell me to end the mentorship early?** Beyond the wind-down triggers — what signals from the mentee, or from the org context, would trigger you to honestly close the arc before its natural close?
- **What is the *sponsorship* move that would compose with this mentorship?** Sponsorship (chapter 03 §1) is speaking up for the mentee in rooms they aren't in. What's the specific room and moment where you'd sponsor them in the coming quarter?

## Deliverable

Two artefacts:

- **`mentorship-plan-<mentee-name>-<quarter>.md`** — 3–5 pages, containing the framing (§1–2), the two skill loops (§3), the ownership-transfer plan (§4), the wind-down triggers and calibration checkpoints (§5), the first-conversation agenda (§6), and the reflection (§8).
- **`standards-<topic>.md`** — half-page to one page, the standards-library entry (§7).

## Acceptance criteria

- [ ] The mentorship plan names a specific mentee scenario (M1–M5) and specific growth target (not "grow into a senior engineer" but the specific behaviours).
- [ ] The framing section has candidate growth-target framings, evidence shapes, cadence proposal, out-of-scope statement, and shared-doc opening — all drafted at a level the first conversation could actually walk them.
- [ ] Two skill loops are designed with loop name, starting move, iteration structure, close criteria, and time estimate. Each loop nests in real work, not synthetic exercises.
- [ ] The ownership-transfer plan names a *specific* real piece of work (not a hypothetical), the transfer trigger, the mentor's post-transfer role, and the failure plan.
- [ ] Wind-down triggers are specific and observable, with a target end-date range.
- [ ] Calibration checkpoints are on a stated cadence.
- [ ] The first-conversation agenda is minute-level structured (30 min broken into blocks) and includes the mentee's-own-priorities block.
- [ ] The standards-library entry follows chapter 03 §6's format (rule, reason, enforcement, cross-references, metadata with owner + last-updated + next-review).
- [ ] The library entry is half-page to one-page. Longer entries are broken up or shortened.
- [ ] The reflection names any failure-mode risks (chapter 03 §9), any early-end signals, and one specific sponsorship move.
- [ ] The mentorship arc, from framing to wind-down, is plausible in a 4–6-month window for the chosen scenario. Arcs that exceed 12 months without a wind-down trigger are re-scoped.

## Stretch goals

- **The paired mentor.** For scenario M3 (cross-team mentor), draft the *manager-to-manager conversation* both managers need to have to sanction the cross-team mentorship. What are the boundaries; what signals do they escalate to each other; what does the quarterly check-in look like?
- **The retrospective wind-down doc.** Assume the mentorship ran for 6 months and closed successfully. Author the *closing conversation* — the review of what worked, what didn't, and the mentee's next mentorship (with someone else, or as a mentor themselves). The wind-down as an artefact.
- **The sponsorship packet.** For the sponsorship move you named in §8's reflection, draft the actual message you would send (in whatever forum — a promo committee, a project-staffing meeting, a team-lead alignment) that speaks up for the mentee. Sponsorship is what turns mentorship into org-visible growth.
- **The library entry adoption plan.** For the library entry from §7, draft the *adoption plan* — how you'll cite it in reviews for a month to establish the citation habit, how you'll surface it at the team's next standards review, whether you'll propose it to a peer team's guild forum. A library entry that isn't adopted is just a document.
- **The failure-mode role-play.** Assume 3 months in, the mentee starts showing one of chapter 03 §4's failure signals (avoiding conversations, or arguing back at every piece of feedback). Draft the *addressing conversation* — the specific words you would use, in the moment, to name the drift and offer to change or end the arc.
- **The mentorship-of-mentors move.** Once your mentee closes their arc, draft the *hand-off* — how you would sponsor them as a mentor to a more-junior engineer, what shape of first-mentorship you would recommend, and what your own role becomes (available for peer consultation but not central to their new arc). This is the recursive move that turns mentorship into a culture.
- **The measurement retrospective.** Six months after the plan starts, assume you're revisiting it. Which leading indicators (chapter 03 §4) landed? Which didn't? What would you change about the arc? This exercise, run on your own past mentorships, is one of the highest-value calibration moves an L30 mentor can make.
