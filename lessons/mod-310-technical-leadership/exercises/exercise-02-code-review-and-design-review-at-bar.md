# exercise-02: Code review and design review at bar

**Estimated effort:** 3 hours

## Objective

Perform a **full senior-level ML code review** on a chosen (or peer-authored) ML pull request, plus a **full design-doc review** on a chosen (or peer-authored) ML design doc, using chapter 02's rubric — the ML-specific checklist of §3, the block-vs-nudge classification of §2 and §4, the design-review discipline of §5, and the mentorship framing of §6. Close the loop with a written review retro (§7 of this exercise) that identifies patterns worth encoding in the standards library (chapter 03).

The output is two review artefacts (one code-review comment set + one design-review comment set) plus a one-page review retro. Together they are the evidence that you can *run reviews at bar and hold the bar consistently*, not just spot the occasional bug.

This is the L30 tell that separates "I leave good comments on PRs" from "I run our team's reviews and hold the bar predictably." An L20 reviews to catch bugs; an L30 reviews to catch bugs *and* hold the bar *and* teach the author *and* train the team.

## Prerequisites

- Read chapter 02 (`02-code-review-and-design-review-at-bar.md`) end-to-end — the three purposes of review, block-vs-nudge classification, the ML-specific checklist, the rules of thumb, design-review discipline, reviewer training, escalation, and the five failure modes of the senior reviewer's own practice.
- Skim [mod-302](../../mod-302-ml-systems-architecture/) chapters on training/serving skew — the checklist's §3.1 rests on this vocabulary.
- Skim [mod-305 chapter 01](../../mod-305-advanced-evaluation/01-offline-eval-harness-shape.md) and [chapter 02](../../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) — the eval harness discipline the checklist's §3.2 checks against.
- Skim [mod-306 chapter 01](../../mod-306-experimentation-at-scale/) — the rollout-gate vocabulary the checklist's §3.5 checks against.
- Skim [mod-307 chapter 01 §6](../../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md) and [chapter 03](../../mod-307-ml-reliability-slos/) — the SLO doc and cost-budget discipline the checklist's §3.4–5 point at.
- Skim [mod-309 chapter 01](../../mod-309-responsible-ai-governance/01-model-cards-and-data-statements.md) — the review-packet discipline the checklist's §3.6 checks against.
- Read the [Google Engineering Practices — Code Reviewer's Guide](https://google.github.io/eng-practices/review/reviewer/) — specifically the *[looking for](https://google.github.io/eng-practices/review/reviewer/looking-for.html)*, *[speed](https://google.github.io/eng-practices/review/reviewer/speed.html)*, and *[pushback](https://google.github.io/eng-practices/review/reviewer/pushback.html)* essays.
- Read the [Conventional Comments](https://conventionalcomments.org/) format — the block-vs-nudge classification the review comments use.

## Pick your review targets

You need one **PR-shaped artefact** and one **design-doc-shaped artefact**. Options:

- **PR sources.** Real: a recent PR from your team you were not the primary reviewer on (with author's permission if it's not public); an open-source ML PR from a project you know (e.g., a Hugging Face model repo, a scikit-learn PR, a Fairlearn PR). Synthesized: a peer's exercise 02 output from another module (e.g., mod-305 exercise 01's eval-harness PR, mod-307 exercise 04's retraining-as-deploy PR). Or, the fallback: write a **deliberately-flawed PR** based on the failure modes in chapter 02 §3 — see the P-appendix below.
- **Design doc sources.** Real: a recent design doc from your team, a public ML design doc (Uber's [Michelangelo posts](https://eng.uber.com/scaling-michelangelo/), Airbnb's [Bighead posts](https://medium.com/airbnb-engineering/bighead-airbnb-s-end-to-end-machine-learning-platform-e2c9f9b31735)), a mod-302 exercise doc, or a mod-308 exercise 02 tradeoff doc. Synthesized: your own mod-310 exercise 01 roadmap, treated as a doc submitted for peer review.

If your PR / doc source is not real, use these **P-appendix flawed-artefact seeds** — pick one PR seed and one doc seed:

- **P1 — PR that changes feature-computation code without a skew test.** Adds a normalisation step in the training feature pipeline, forgets to touch the serving feature pipeline. Eval harness output shows a lift; no skew test in the PR. (Should trigger a block on §3.1 and §3.2.)
- **P2 — PR that promotes a model without updating the model card.** The training script trained a new version; the promotion PR bumps the model-version pointer and merges a benchmark file. No mod-309 model card update; no fairness slice re-run. (Should trigger a block on §3.6.)
- **P3 — PR that changes the loss function without pinned seeds or data snapshot.** The change looks correct; the reported metric is better; but the training run's seed and data snapshot are unrecorded in the PR description. (Should trigger a block on §3.3.)
- **P4 — PR that adds an LLM-call to the serving path without cost or rollout gate discussion.** Adds a re-ranker call to a production request path; PR description says "small quality lift"; no cost projection, no shadow deployment, no rollout plan. (Should trigger blocks on §3.4 and §3.5.)
- **D1 — Design doc missing the evaluation plan.** Proposes an LLM-augmented ranker; walks the modelling and serving in detail; leaves "evaluation TBD" in the eval section. (Should trigger a design-review send-back.)
- **D2 — Single-alternative design doc.** Proposes a new model architecture; alternatives-considered section says "we considered the current baseline and it isn't good enough." (Should trigger a design-review comment on §5's alternatives discipline.)
- **D3 — Design doc that under-scopes rollout.** Modelling and evaluation sections are strong; rollout section is "we'll run an A/B test." No shadow, no ramp, no guardrails, no rollback trigger. (Should trigger a design-review comment on §3.5.)

## Steps

### 1. Setup and orientation (≈ 15 min)

Load the PR and the design doc. Read each once through before commenting. Note in your private working doc:

- The change's *scope* (files touched, LOC, area of the system).
- The change's *intent* (from PR description or doc summary).
- Your first-pass sense of what the load-bearing questions will be (before you start scoring against the rubric).

The read-through-first discipline is what prevents the reviewer from anchoring on the first bug they see; it also matches the [Google reviewer's-guide navigation approach](https://google.github.io/eng-practices/review/reviewer/navigate.html).

### 2. Code review pass 1 — correctness (≈ 25 min)

Comment on correctness bugs first. This is the review's *floor* — a change that fails the correctness bar does not merge (chapter 02 §1 and §4).

For each comment:

- Use the Conventional Comments format: `issue:` for blocks, `question:` for clarifications, `nit:` for style.
- Name the *file:line* location and the *specific* concern.
- If you're claiming a block, name the *reason* — what fails if this ships as-is.

### 3. Code review pass 2 — the ML-specific checklist (≈ 30 min)

Walk chapter 02 §3's checklist against the PR. For each of the seven areas — training/serving skew, eval harness discipline, reproducibility, cost surface, rollout gate, model card / review packet, review's own composition — comment on the PR's state. Use the block-vs-nudge rules of thumb (§4) to classify.

Discipline: your review should end with roughly 1–3 blocks, 3–8 nudges, 1–2 pieces of `praise:`. If you land 15 blocks, the PR is either significantly under-baked or you're using blocks where nudges would work — reflect and re-classify. If you land zero blocks on a PR with real defects (from a P-appendix seed), you missed a checklist item — re-run.

### 4. Code review pass 3 — teaching (≈ 15 min)

Look at the review comments you've written. For each *block*, does the comment tell the author *why* — with a citation to a chapter, a standard, a paved-road primitive, or a runnable example? A block without a *why* teaches only that this reviewer blocks; a block with a *why* teaches the pattern.

Add teaching to any block missing it. For 1–2 comments, add a `thought:` prefix note pointing at the next-level pattern the author might explore ("thought: once this ships, the same pattern applies to feature-view Y; happy to pair on that later").

Aim: every block-quality comment on the final PR carries a rule + reason + fix + optional pattern reference.

### 5. Design-doc review (≈ 45 min)

Move to the design doc. Read it once through, then comment. The design-doc review is different from the code review in two important ways (chapter 02 §5):

- The **decision surface is broader** — approve, approve-with-revisions, reject, or send-back-for-rescope.
- The **facilitation matters** — you're not the only reviewer; your comments should compose with what other reviewers will find.

Comment on:

- **The problem statement.** Is the outcome named in falsifiable, bounded, downside-explicit terms (chapter 01 §2.1)?
- **The evaluation plan.** Primary metric, guardrails, slices, CI shape (mod-305 chapter 01)?
- **The data plan.** Sources, labelling, volume, freshness (mod-309 chapter 02)?
- **The modelling plan.** Algorithm choice with alternatives considered (mod-303)?
- **The serving plan.** Skew mitigation, SLO contract (mod-302 + mod-307)?
- **The rollout plan.** Shadow, canary, ramp, guardrails, rollback trigger (mod-306)?
- **Alternatives considered.** At least three including "keep the baseline" (mod-308 chapter 03 §5)?
- **Risks and dependencies.**

For each: use the same block-vs-nudge discipline as the code review. Design-review "blocks" mean "this design cannot be approved as-written" — often manifesting as *send back for revision* rather than *no*.

At the end, produce a **facilitation summary** — the 3–5 questions you would open the synchronous design review meeting with, based on the comments. This is the artefact that makes the meeting productive rather than approval-theatre.

### 6. Review retro (≈ 25 min)

Now step out of reviewer-mode and into retro-mode. Walk your own review artefacts against the failure modes of chapter 02 §8:

- **Bottleneck.** How long would this review take a real author to unblock on? If your comment volume + comment depth would produce a multi-day back-and-forth, was any of that avoidable?
- **Rubber stamp.** Did you cover every checklist item, or did you skim any?
- **Teaching-by-fixing.** Did any comment rewrite the fix rather than name the pattern?
- **Re-litigating.** Did any comment argue a standard you'd previously argued (or would repeatedly)? These are standards-library candidates (chapter 03).
- **Exhausted / narrow.** Which comments felt effortful because you're the only person on the team who catches them? These are also standards-library candidates.

Produce a short (half-page) retro document with:

- **Block tally.** How many blocks, of what type (correctness / skew / eval / reproducibility / cost / rollout / model card).
- **Nudge tally.** How many nudges, of what type.
- **Praise count.** Praise is easy to skip; it matters. If you left zero praise on a solid PR, reflect.
- **Teaching moves.** Which 1–3 comments would you highlight as good teaching moves the team's mid-levels should learn from?
- **Standards-library candidates.** Which 1–3 comment patterns should become library entries so you don't re-argue them next PR?
- **Reviewer-training opportunities.** If a mid-level shadowed this review (chapter 02 §6), which 1–2 comments would you walk them through as calibration examples?

### 7. Mock reviewer-training walk (≈ 15 min, optional)

If you have a peer available, run the shadow-review calibration of chapter 02 §6:

- Trade one of your reviews with a peer. They walk their would-be comments; you walk yours; you compare.
- Notice where you would have blocked and they would have nudged, or vice versa. Discuss the *why*.
- Note any standards-library candidates that came out of the calibration.

If you don't have a peer, do the same walk against the P-appendix "expected blocks" list — did you catch the blocks the seed was designed to test?

## Deliverable

Three artefacts:

- **`code-review-<pr-slug>.md`** — the full set of code-review comments, in Conventional Comments format, one comment per file:line with the block-vs-nudge classification.
- **`design-review-<doc-slug>.md`** — the full set of design-doc review comments, plus the facilitation-summary section with 3–5 opening questions for the meeting.
- **`review-retro-<date>.md`** — the half-page retro (§6) with block tally, nudge tally, praise count, teaching moves, standards-library candidates, and reviewer-training opportunities.

## Acceptance criteria

- [ ] Every code-review comment uses the Conventional Comments format (`issue:`, `suggestion:`, `nit:`, `question:`, `thought:`, `praise:`), with the block-vs-nudge classification explicit.
- [ ] Every block-quality comment (chapter 02 §4) states the rule, cites the standard (or the chapter/section), and either points at the fix or names the pattern of the fix.
- [ ] The code review covers all seven areas of the ML-specific checklist (chapter 02 §3): training/serving skew, eval harness, reproducibility, cost surface, rollout gate, model card / review packet, review composition. Areas where there is nothing to say are explicitly marked `n/a` in the retro.
- [ ] The code review ends with roughly 1–3 blocks, 3–8 nudges, 1–2 pieces of `praise:` (or a written note in the retro explaining any material deviation from this shape).
- [ ] The design-doc review comments cover the design-doc contents of chapter 02 §5 — problem statement, evaluation plan, data plan, modelling plan, serving plan, rollout plan, alternatives considered, risks, dependencies. Areas where a section is absent from the doc are flagged as `issue:`.
- [ ] The design-doc review ends with a facilitation summary — 3–5 opening questions for the synchronous design review meeting.
- [ ] The retro tallies blocks, nudges, and praise by category.
- [ ] The retro names at least one teaching move to highlight for mid-levels.
- [ ] The retro names at least one standards-library candidate (a pattern that should be encoded in the library so it doesn't have to be re-argued).
- [ ] The retro walks the five failure modes of chapter 02 §8 and honestly names any hit.

## Stretch goals

- **Time the review.** Record how long each pass took. A senior review of a 300-LOC PR typically takes 45–90 min for a substantial change; if yours was much less, the risk is rubber-stamp; if much more, the risk is bottleneck. Reflect in the retro.
- **The escalation call.** For one comment where you would plausibly disagree with the author, walk the escalation path (chapter 02 §7). Would this go to the tech lead? The standards library? The governance-analyst peer? Draft the escalation message.
- **The library entry.** For one of the standards-library candidates from step 6, draft the actual library entry (chapter 03 §6's format — rule, reason, enforcement, owner, next-review date). This is the compound move — a review that improves the library is worth more than one that just fixes the PR.
- **The mentee calibration.** Write up the shadow-review walk (step 7 or an imagined version) as a mentee-facing artefact — the 1–2 comments you'd highlight, the reasoning walk, the calibration takeaway. This is what chapter 03 §3's *skill loops* produce.
- **The bot-augmented review.** For one of the ML-specific checklist items (§3.3 reproducibility, §3.2 eval harness), sketch what a CI check could enforce automatically. Some review load compresses into automation; the reviewer's judgement lands on the things the bot can't check. mod-308 chapter 03's contribute-back path applies if the CI check becomes a paved-road primitive.
- **The re-review.** Assume the author responds to your review with 3–4 pushbacks — 2 legitimate, 2 not. Draft your responses (chapter 02 §7's *handling pushback* discipline). Which of your original blocks stand? Which downgrade to nudge? Which flip?
