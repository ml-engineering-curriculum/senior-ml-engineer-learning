# exercise-01: Multi-quarter ML initiative roadmap

**Estimated effort:** 3 hours

## Objective

Author the **full multi-quarter ML initiative roadmap** for a specific initiative, at the shape and depth chapter 01 §3's template requires — every one of the ten sections filled or explicitly marked `n/a`, every claim carrying a supporting reference, every decision point and cutline named, and the artefact ready to walk into a leadership review.

The output is a Markdown document (`roadmap-<initiative>-<year>.md`) that could live in the team's repo, be linked from the director's staffing plan, and be defended cold against the questions of chapter 01 §5. A mock pre-commit review (self- or peer-run) closes the loop.

This is the L30 tell that separates "I have a sprint plan" from "leadership committed to a plan I authored." An L20 writes tickets; an L30 writes roadmaps that leadership signs.

## Prerequisites

- Read chapter 01 (`01-roadmap-and-scoping.md`) end-to-end — the four questions the roadmap answers, the template, the four ML-specific scoping traps, the pre-commit review, and the post-commit maintenance discipline.
- Skim [mod-301 chapter 01](../../mod-301-senior-ml-role-scope/01-what-changes-at-senior.md) — the L20-vs-L30 shift the outcome-first framing depends on.
- Skim [mod-305 chapter 01](../../mod-305-advanced-evaluation/01-offline-eval-harness-shape.md) — the eval-harness discipline milestone M0 (baseline lock) commits to.
- Skim [mod-306 chapter 01](../../mod-306-experimentation-at-scale/) — the ramp-plan vocabulary the M5 (canary) milestone lives in.
- Skim [mod-307 chapter 01 §6](../../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md) and [chapter 03](../../mod-307-ml-reliability-slos/) — the SLO doc and cost-budget discipline the roadmap's dependencies and cost sections point at.
- Skim [mod-308 chapter 01 §7](../../mod-308-platform-collaboration/01-paved-road-consumption.md) — the paved-road inventory the dependencies section cites.
- Skim Will Larson's *[Sizing Engineering Teams](https://lethain.com/sizing-engineering-teams/)* and *[Systems of Engineering Management](https://lethain.com/elegant-puzzle/)* — the load-bearing external framing on roadmap-as-artefact and hero-project caution.

## Pick your initiative

Pick **one** of the following, or bring your own (I5, with a preamble on scale, current state, deadlines, and dependencies):

- **I1 — Search relevance rebuild.** Rebuild the search ranker for a high-traffic consumer surface. Two-to-three quarters. Current baseline: gradient-boosted classical model; the initiative evaluates neural, LLM-augmented reranking, and a hybrid. Rollout is behind an A/B experiment; blast radius is the primary revenue metric.
- **I2 — Fraud model platform migration.** Migrate the fraud-detection model family from a legacy in-house training platform to the paved-road ML platform. Three-to-four quarters. Regulatory constraints (model-risk-management sign-off). Downstream review workflow depends on the model's scoring API remaining stable.
- **I3 — LLM-augmented ticket triage rollout.** Roll out a new LLM-augmented ticket triage system replacing a classifier-plus-heuristics baseline. Two quarters. Includes prompt-vs-fine-tune decision, retrieval quality, safety-classifier composition, and cost budgets.
- **I4 — Personalisation ranker refactor.** Refactor a large personalisation ranker to consume a new feature store, adopt online learning, and switch from batch to streaming inference. Three-to-four quarters. Multiple peer-team dependencies (feature store team, streaming platform team, experimentation team).
- **I5 — Bring-your-own.** A real or planned initiative from your team. Anonymise anything sensitive; add a preamble covering: scale, current baseline, business context, deadline pressure, staffing, key dependencies, and current state of the roadmap.

## Steps

### 1. Scope frame (≈ 15 min)

Before the roadmap: write one paragraph on *who commits to this initiative* and *what the leadership review looks like*. Named director/PM sponsor. Peer reviewers on platform, product, data, governance. Frequency of leadership review (weekly, bi-weekly, monthly). Format (written update, spoken review, both). This paragraph goes in the roadmap's frontmatter metadata.

### 2. Section 1 — Outcome (≈ 20 min)

Write the outcome statement (chapter 01 §2.1). One paragraph. Falsifiable. Bounded. Downside named.

Discipline: run the three tests explicitly in a private note next to the outcome — falsifiability ("can I tell at the end whether this was met?"), boundedness ("is this time-bounded?"), downside symmetry ("what does the roadmap do if the outcome is not met?"). If any of the three fails, rewrite the outcome.

### 3. Section 2 — Non-goals (≈ 15 min)

Author at least three explicit non-goals. Scoping is 30% of the roadmap. A roadmap with fewer non-goals than goals is a roadmap that will be asked to do everything.

Discipline: for each non-goal, name the *reason* it's out of scope. "We are not addressing the ranker's cold-start problem this initiative because the cold-start user population is <2% of traffic and the incremental complexity would double the initiative's headcount."

### 4. Section 3 — Sequence (≈ 30 min)

Author the milestone list, using the eight ML-specific milestone kinds from chapter 01 §2.2 (M0 baseline lock, M1 first-pass, M2 data/infra unblock, M3 sophisticated model, M4 shadow, M5 canary, M6 full rollout, M7 handoff / retraining) as the default scaffold. Some initiatives collapse or reorder milestones — a maintenance-only initiative might have no M3, an LLM-augmented initiative might insert a "prompt vs. fine-tune" branch after M1.

For each milestone: **deliverable, target quarter, owner, and one paragraph on why this sequence** (why here, not earlier or later).

Discipline: a milestone without a *why here* is a milestone that will get renegotiated the first time evidence lands. Force each milestone's rationale.

### 5. Section 4 — Decision points (≈ 30 min)

Author the decision points (chapter 01 §2.3). At least four, in the format **at <milestone>, using <evidence>, we will decide <branch A> vs. <branch B> vs. <cut>**.

Typical decision points to consider:

- The baseline-lock decision (D1): does the current production model already meet the target?
- The first-pass decision (D2): does the simplest approach clear the outcome threshold?
- The data-availability decision (D3): is the labelling-yield on track?
- The prompt-vs-fine-tune decision (D4): after prompt iteration saturates, do we invest in fine-tuning?
- The shadow-to-canary decision (D5): is the skew report / cost projection / latency budget clean?
- The ramp-step decisions (D6): at each ramp step, does the guardrail-metric picture support advancing?
- The cutline decisions (D7): the ordered cutline list in §7.

Discipline: each decision point has a *trigger* (a milestone, not a date), *evidence* (specific, named — not "we'll look at the numbers"), and *branches* (named alternatives, not "we'll figure it out").

### 6. Section 5 — Dependencies (≈ 25 min)

Author the dependency list (chapter 01 §2.4 and the trap of §4.4). For each dependency: **deliverable, owner on peer team, earliest-date, risk-if-slips, contingency**.

Cover at least three dependencies, drawing from the mod-308 paved-road inventory: feature-store view, model-registry entry, training-cluster capacity, evaluation-harness update, experimentation-platform ramp-step support, serving-platform primitive, review-packet sign-off from mod-309 governance.

Discipline: a dependency the peer team has not confirmed in writing is a *risk*, not a *dependency*. Move any unconfirmed items to §8 (risks). This is exercise-hard — most first-draft roadmaps have several undocumented "we assumed X team would do Y" items.

### 7. Section 6 — Cost and headcount (≈ 20 min)

Author the cost and headcount section. Estimate ranges, not point estimates.

- **Compute.** GPU-months, CPU-hours, storage — with the assumptions listed. mod-307 chapter 03 is the source for the cost-modelling discipline.
- **Vendor.** API tokens, licences, third-party data.
- **Headcount.** Named engineers × fraction of quarter × how many quarters. Yourself included — with the 20–40% cap chapter 01 §2.4 names.

Discipline: if the roadmap needs 100% of the tech lead's time, either the initiative is too large for one lead or the lead role is under-staffed. Flag the tension explicitly.

### 8. Section 7 — Cutlines (≈ 20 min)

Author the ordered cutline list (chapter 01 §2.4). At least three cutlines, each in the format **if we lose <resource>, we cut <milestone or scope> and produce the <fallback>**.

Typical cutline scenarios:

- Losing one engineer for one quarter.
- A dependency slipping four weeks.
- The product deadline pulling in one month.
- The primary metric's baseline moving during the initiative (invalidating the outcome delta).

Discipline: the cutlines are named *now*, not at the moment of pain. A roadmap without cutlines is a roadmap that fails as soon as reality diverges.

### 9. Section 8 — Risks (≈ 15 min)

Author three-to-five risks with mitigations. Each risk is specific and each mitigation is a concrete action, not "we will monitor closely."

Include at least one risk from each category:

- **Model risk.** The sophisticated model may not beat the baseline (chapter 01 §4.2).
- **Data risk.** The labelling pipeline may under-yield.
- **Platform risk.** A dependency may slip.
- **Adoption risk.** The rollout may face product-side objections at canary.
- **Organisational risk.** Team capacity may shift (headcount, re-org, on-call load).

### 10. Sections 9–11 — Open questions, references, decision log (≈ 15 min)

- **Open questions.** State as *answerable* questions. "Does the LLM path need a fine-tune, or is prompt-plus-retrieval enough?" is answerable; "how do we handle LLM concerns?" is not.
- **References.** Links to the tradeoff docs, the RFC decisions, prior roadmaps, eval reports, mod-308 paved-road inventory. Every citation is a specific link.
- **Decision log.** Empty at draft. This is the section that will grow as decision points close and as the roadmap is maintained (chapter 01 §6).

### 11. Walk the four ML-specific traps against your draft (≈ 15 min)

Chapter 01 §4 named four traps:

- **Eval-later trap.** Does M0 include a *frozen evaluation harness* with primary metric declared, guardrails named, CI shape declared, and harness code merged? If not, add M0 detail.
- **Model-vs-project uncertainty confusion.** Are model uncertainties in the decision points (§5) and project uncertainties in the dependencies and cutlines (§§5, 7)? If any project uncertainty ("we don't know when the platform primitive lands") is framed as decision-point ambiguity, re-classify.
- **Hero-project trap.** Does the initiative consume more than 70% of the team's quarterly capacity? Is there a named 30% reservation for maintenance, retraining, incident response, on-call, and mod-308 contribute-back work? If not, add the reservation and cut initiative scope to fit.
- **Over-committed platform-dependency trap.** Has every §5 dependency been confirmed with the named owner in writing? If not, move to §8 (risks) until confirmation lands.

Every hit is a change to the roadmap *before* the pre-commit review, not after.

### 12. Pre-commit review (≈ 20 min)

Either self-review with the reviewer hat on, or trade with a peer. The reviewer walks with the four director-level questions from chapter 01 §5:

- **Why this sequence, not that one?** For each milestone that could plausibly be re-ordered, is the *why here* strong enough to defend?
- **What breaks if we pull Q3 into Q2?** Which milestones become impossible? Which cutlines get exercised?
- **What is the cheapest thing we could cut?** Does the cutline order match the reviewer's intuition on cheapness?
- **Where are the hidden dependencies?** Any assumption about a peer team's capacity or a data source's availability that isn't in §5?

The reviewer produces marginal notes; the notes are folded into the roadmap or acknowledged as follow-ups in an appendix.

## Deliverable

One Markdown document — `roadmap-<initiative>-<year>.md`, following the chapter 01 §3 template, 4–8 pages — plus a short review-note appendix that lists what the mock reviewer said and how you handled each note.

## Acceptance criteria

- [ ] Frontmatter metadata filled: version, owner, sponsor, reviewers, status, created/last-updated dates, next-review date.
- [ ] Section 1 (outcome) is one paragraph and passes the three tests (falsifiability, boundedness, downside symmetry) — with the private test-walk noted.
- [ ] Section 2 (non-goals) has at least three explicit non-goals, each with a reason.
- [ ] Section 3 (sequence) has milestones with deliverable + quarter + owner + *why here* rationale for each. At least M0 (baseline lock) is present; the rest are drawn from chapter 01 §2.2's kinds as the initiative shape requires.
- [ ] Section 4 (decision points) has at least four decision points, each with trigger (milestone), evidence (specific), and named branches.
- [ ] Section 5 (dependencies) has at least three dependencies with deliverable + owner + earliest-date + risk-if-slips + contingency. Every dependency has been confirmed in writing or been moved to §8.
- [ ] Section 6 (cost and headcount) gives ranges (not point estimates), includes the tech-lead's own time-fraction, and flags any 100%-tech-lead tension.
- [ ] Section 7 (cutlines) has at least three cutlines in the "if we lose X, we cut Y and produce Z" format.
- [ ] Section 8 (risks) has 3–5 risks with concrete mitigations, covering at least model, data, platform, adoption, and organisational categories.
- [ ] Section 9 (open questions) states answerable questions, not handwaves.
- [ ] Section 10 (references) cites the source docs, eval reports, and paved-road inventory.
- [ ] Section 11 (decision log) exists (empty at draft).
- [ ] The four-trap walk (step 11) is documented — either "none hit" with the walk noted, or the hits and their fixes named.
- [ ] The pre-commit-review appendix names at least one reviewer note per director-level question, with the response.

## Stretch goals

- **The dependency confirmation round.** For each §5 dependency, draft the actual Slack / email / doc-comment you would send to the peer-team owner to confirm the deliverable and date. Confirmation-in-writing is what turns a §5 line into a real commitment.
- **The mid-half re-defence.** Compose the mid-half-review update (chapter 01 §6) as if it were four to six weeks into the initiative. What advanced, what slipped, what decision-point evidence landed, what cutlines came close to being exercised. This forces the roadmap to be usable as a *maintenance artefact*, not just a launch artefact.
- **The re-commit scenario.** Assume a scope change lands — the product team wants the outcome deadline pulled in by one quarter. Author the *re-commit* (chapter 01 §7) — the revised outcome, the resequenced milestones, the exercised cutlines, the re-negotiated dependencies. The re-commit is what tests whether the roadmap composes with change.
- **The RFC spawn.** Pick a decision point that plausibly resolves into a contribute-back RFC (a platform-primitive change your team would drive). Draft the RFC opening (chapter 03 §3 of mod-308) as a follow-on artefact.
- **The paired review packet.** Attach the roadmap as document 1 of a leadership review packet, with the SLO doc (mod-307 chapter 01 §6), the eval-harness contract (mod-305 chapter 01), and the mod-309 review packet as the other components. The roadmap composes with these; the exercise produces the composed packet.
- **The narrative one-pager.** Extract a one-page executive summary from the roadmap — the outcome, the top three milestones, the top three risks, the top cutline. This is the artefact that gets read in the leadership review's first three minutes; the roadmap is the artefact that answers the follow-up questions.
