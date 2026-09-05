# exercise-04: Rubric-based Senior ML Engineer interview loop

**Estimated effort:** 3 hours

## Objective

Design the **full four-round Senior ML Engineer interview loop** — the rubric with behaviourally-anchored 1–5 anchors on every dimension, four job-representative questions documented at the [Google re:Work *interview question design*](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/design-your-questions/) depth (question, probes, scoring anchors, common pitfalls), the debrief facilitation script, the calibration-retro cadence, and the mentee-training plan for a mid-level growing into an interviewer role at bar.

The output is a single interview-loop packet (`interview-loop-senior-ml-engineer-<year>.md`) plus one companion artefact — the hire-recommendation memo template your debriefs will output against. Together they are the evidence that you can *run a loop that hires at bar and calibrates over time*, not just conduct a strong single interview.

This is the L30 tell that separates "I get pulled into interviews and write good feedback" from "I own the interview loop that decides who joins the team, and I train the panel that runs it with me." A team without this artefact hires on the loudest voice; a team with it hires against a rubric everyone can see.

## Prerequisites

- Read chapter 04 (`04-rubric-based-ml-interviews.md`) end-to-end — the empirical case for structured rubric-based interviews, the four-part ML rubric with behavioural anchors, the four-round loop shape, question design, debrief mechanics, the three calibration mechanisms, bias/fairness/legal-defensibility discipline, the mentee-training arc, and the five failure modes.
- Skim chapter 02 §6 and chapter 03 §3 of this module — the *shadow / co-pilot / pilot / independent* pattern the interviewer-training arc reuses, so the mentee-training plan (§7 below) composes with the two other mentee-training arcs the module installs.
- Skim [mod-301 chapter 03](../../mod-301-senior-ml-role-scope/03-verbs-that-define-seniority.md) — the seniority-verb vocabulary that anchors the *judgement and communication* rubric dimension in observable behaviours.
- Read the [Google re:Work *Guide to Structured Interviewing*](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/introduction/) end-to-end — the canonical corporate treatment the rubric and loop shape derive from.
- Read [Schmidt & Hunter, *The Validity and Utility of Selection Methods in Personnel Psychology* (1998)](https://mavweb.mnsu.edu/howard/Schmidt%20and%20Hunter%201998%20Validity%20and%20Utility%20Psychological%20Bulletin.pdf) — the meta-analysis behind the "structured predicts ~2× as well as unstructured" claim. At least the abstract and the validity-coefficients table.
- Skim [Aline Lerner / interviewing.io, *Anatomy of a Great Interview*](https://interviewing.io/blog/hiring-loops-vs-interviews) and the [ML-engineer interview guide](https://interviewing.io/guides/machine-learning-engineer-interview) — the industry-facing reference for loop design and ML-specific question archetypes.
- Skim [Laszlo Bock, *Work Rules!*](https://www.workrules.io/), chapter 5 (*Why We Hire Only the Best*) — the case study Google publicly published on abandoning unstructured brainteaser interviews.
- Skim [Daniel Kahneman et al., *Noise: A Flaw in Human Judgment*](https://us.macmillan.com/books/9780316451383/noise/) — the *decision hygiene* / calibration framing the rubric-audit mechanisms of chapter 04 §6 sit inside.
- Skim the [SIOP *Principles for the Validation and Use of Personnel Selection Procedures*](https://www.siop.org/Portals/84/PDFs/Principles.pdf) — the I/O-psychology reference behind the *behaviourally-anchored rating scale* discipline chapter 04 §7 requires.

## Pick your loop context

You are designing the loop for a **Senior ML Engineer (L30-equivalent)** hire. Pick one of the following team contexts, or bring your own (L5 with a preamble on the real team, its stage, its regulatory context, and the specific senior gap the hire is meant to close). The context sharpens what the rubric emphasises and what a "meets bar" answer sounds like.

- **L1 — The classical-ML product team.** ~10 engineers on a mature consumer-facing surface (search, ranking, personalisation) with a paved-road ML platform. The Senior ML Engineer will lead a multi-quarter initiative, run reviews, and mentor two mid-levels. Modelling depth skews classical; systems reasoning skews production-platform integration; the hiring bar leans toward the *judgement and communication* and *collaboration* dimensions because the technical foundations are well-served by the platform.
- **L2 — The LLM-augmented deployment team.** ~6 engineers building an LLM-augmented product surface (retrieval-augmented generation, LLM-assisted triage, generative UX) with heavy dependencies on external LLM vendors. The Senior ML Engineer will own prompt-vs-fine-tune decisions, safety-classifier composition, and cost budgets. Modelling depth skews toward the prompt/eval/retrieval altitude; systems reasoning skews toward vendor-integration and cost/latency SLOs; the *judgement* dimension leans on cost/quality/latency tradeoffs.
- **L3 — The regulated-ML team.** ~8 engineers on a fraud, credit, healthcare, or safety-critical model family under [SR 11-7](https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm)-style model-risk-management or [EU AI Act](https://artificialintelligenceact.eu/) obligations. The Senior ML Engineer will lead a platform migration or model rebuild through the governance gates. Modelling depth is important but *evaluation depth* (slice matrices, protected-attribute analysis) is even more so; systems reasoning includes governance-review composition; the *collaboration* dimension leans heavily on cross-functional partnership with legal, risk, and compliance.
- **L4 — The pre-platform stream-aligned team.** ~4 engineers on a team where no ML platform exists yet — everything from feature computation to serving is bespoke. The Senior ML Engineer will *build the paved road* (mod-308) alongside shipping product ML. Modelling depth matters, but the *systems reasoning* dimension is dominant because the candidate will be architecting the primitives, not consuming them; *judgement* leans heavily on build-vs-adopt decisions; *collaboration* focuses on stakeholder alignment across a small team.
- **L5 — Bring-your-own.** Your actual team's Senior ML Engineer opening (current, planned, or one you might open in the next year). Anonymise as needed. Preamble covers: team size and composition, stage of ML maturity, regulatory context, deadline pressure, the specific senior gap the hire fills, and any known constraints (comp band, remote/onsite, timezone).

## Steps

### 1. Loop-context frame (≈ 15 min)

Before the rubric: write one paragraph on *the specific gap the hire fills* and *what the loop must reliably discriminate*. Which of the four rubric dimensions matter most for this team? Which failure modes of the hire would be most expensive (e.g., "a hire who cannot mentor is expensive because we have two mid-levels who need a mentor," or "a hire who cannot reason about vendor cost is expensive because we're on a $2M/year vendor budget with no ceiling")? This paragraph anchors every scoring decision that follows.

Discipline: the frame should name *at least one* dimension where the bar is set *higher than the default* for this hire, and *at least one* where the bar is set *at the default* (i.e., not everything is above bar — that is a wish list, not a loop).

### 2. Author the four-part rubric with behavioural anchors (≈ 40 min)

Adapt the four-dimension rubric of chapter 04 §2 for your loop context. For each of the four dimensions — **modelling depth**, **systems and production reasoning**, **judgement, tradeoffs, and communication**, **collaboration, mentorship, and cross-team signal** — author:

- **What this dimension measures**, in one paragraph, in the vocabulary of your team context. Not the generic definition from chapter 04 — the *team-specific* definition.
- **Behavioural anchors at each level 1–5.** Every anchor is *behavioural* ("proposes a model choice with rationale and names the failure mode they would monitor"), not *trait-anchored* ("smart," "curious," "senior-feeling"). See chapter 04 §7 and [SIOP *Principles*](https://www.siop.org/Portals/84/PDFs/Principles.pdf) for the BARS discipline.
- **Sample question stems** appropriate to the dimension and your team context. 3–5 stems each; these seed the questions you'll fully author in step 3.

Discipline: each anchor should be readable by a new interviewer who has never met the candidate. If the anchor requires interpretation ("the candidate showed real senior maturity"), rewrite it in observable behaviours ("the candidate proposed a decision, defended it, and named a specific case in which they'd revise it").

### 3. Fully document four interview questions (≈ 45 min)

Chapter 04 §3 names the four-round shape: two technical rounds (modelling depth, systems reasoning), two behavioural rounds (judgement, collaboration). Author **one anchor question per round** — four questions total — at the depth the [Google re:Work question template](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/design-your-questions/) requires:

- **Question.** The specific opening prompt the interviewer uses. Concrete, job-representative (chapter 04 §4), admits multiple valid answers.
- **Follow-up probes.** 3–5 next-level questions the interviewer uses to drill into depth. Each probe should test *the next layer* — a strong candidate answers the opening quickly and the probes are where the discrimination happens.
- **Scoring anchors for this specific question.** What a 3 (meets bar), 4 (above bar), and 5 (clearly above bar) answer looks like. These are *question-specific* anchors — they compose with the dimension-level BARS anchors of step 2 but are more concrete.
- **Common pitfalls.** Things candidates say that sound good but reveal shallow understanding. Chapter 04 §4 names this section as the "reads-well, tests-badly" corrective.
- **Interviewer-side notes.** The context or vocabulary the interviewer should have loaded before asking. If the question depends on a system diagram, the diagram is here. If the probes reference a specific paper or paved-road primitive, the reference is here.

At least *one* of the four questions must be an **end-to-end system design** or a **debugging deep-dive** (chapter 04 §4's two ML-specific archetypes). If your context is L3 (regulated-ML) or L2 (LLM-augmented), *one* question must probe the domain-specific senior signal — model-risk-management review composition (L3) or prompt-vs-fine-tune / cost-quality tradeoffs (L2).

Discipline: a question with only one right answer is a trivia question — rewrite it. A question the interviewer cannot answer confidently themselves is a red flag — either drop it or first author the answer key (the *5 — clearly above bar* answer written out) as a calibration exercise.

### 4. Debrief structure and hire-recommendation memo template (≈ 30 min)

Chapter 04 §5 describes the debrief. Author your loop's specific debrief protocol as a runnable script the facilitator would work from:

- **Pre-debrief.** Every interviewer submits their scored rubric via the applicant-tracking system before the debrief. Deadline: 24 hours after the interview (chapter 04 §7's contemporaneous-feedback discipline). No scores may be revealed in the ATS thread before the debrief.
- **Debrief agenda.** 45–60 minutes, structured as:
  - 5 min — facilitator orients (candidate context, roles at the table, meeting norms).
  - 25 min — facilitator walks the rubric dimension-by-dimension. Each interviewer speaks on the dimension they primarily owned, then adjacent-signal interviewers add.
  - 10 min — points-of-divergence discussion. Where interviewers materially disagreed, the panel unpacks *what specifically* each observed.
  - 10 min — hire recommendation, per interviewer, then panel consensus (or explicit disagreement).
- **Facilitator selection.** Explicit: not the most-senior interviewer. Name how the facilitator is picked (rotation, hiring-loop lead, a designated debrief facilitator).
- **Hire-recommendation memo template.** Author the actual template the debrief outputs — a Markdown / doc template with fields for: candidate name and role, panel members and rubric scores per dimension, points of consensus and divergence, overall recommendation (strong hire / hire / no hire / strong no hire), any conditional recommendations (e.g., "hire with a strong onboarding plan on production systems"), notable observations outside the rubric useful for onboarding, and the *evidence trail* (links to each interviewer's ATS scoresheet).

The template is the artefact the loop *outputs* against every candidate. A loop without this artefact cannot calibrate against outcome (§6) — there is nothing to compare six-month performance to.

### 5. Bias, fairness, and legal-defensibility checklist (≈ 15 min)

Author the loop's bias-and-defensibility checklist. This is a short (half-page) checklist the loop lead runs against the loop's artefacts *every quarter*. Chapter 04 §7 names the three disciplines:

- **Behaviourally-anchored, not trait-anchored.** Every anchor in step 2's rubric is observable behaviour. The checklist item is: *audit the anchors quarterly; flag any that have drifted into traits.*
- **No proxy signals for demographics.** The rubric does not reward prestigious school, "cultural fit," or communication style *unless* those are named as job-relevant behaviours with observable anchors. The checklist item is: *audit each question and anchor for proxy signals; flag anything that correlates with demographic categories without job relevance.*
- **Contemporaneous, auditable feedback.** Every interviewer's rubric and notes are written up within 24 hours, filed in the ATS, and retained for the [legally-required window](https://www.eeoc.gov/laws/guidance/section-6-lawful-employment-decisions) (in the U.S., typically 1 year for private employers; longer for federal contractors — check your legal team's guidance for your jurisdiction). The checklist item is: *audit ATS compliance quarterly; flag any interviewer or round with missing or late feedback.*

Discipline: the checklist should be *runnable* — a new loop lead should be able to pick it up and execute against it without institutional context. Name who runs each check, who receives the flags, and what the remediation is when a flag fires.

### 6. Calibration mechanisms and cadence (≈ 25 min)

Chapter 04 §6 names three calibration mechanisms. For your loop, design the specific cadence and artefacts of each:

- **Shadow-and-lead rotation for new interviewers.** Every new interviewer shadows N loops, then leads M loops with a shadowing calibrator, then their scores count. Pick specific N and M with rationale for your team's throughput. Name the artefact — the *shadow calibration note* — that the shadow and lead compare after each interview.
- **Score-against-outcome calibration.** At the 6-month and 12-month marks after a candidate joins, the hiring panel revisits the loop's scores against actual on-the-job performance. Name the artefact — the *outcome retro note* — and the specific questions it answers (what did we score high on that underperformed? what did we score low on that outperformed? what signal was hidden?). Name the cadence — likely quarterly rollup across the batch of candidates who hit the 6-month or 12-month mark.
- **Cross-interviewer score-distribution audit.** Quarterly, the loop lead pulls each interviewer's score distribution. Name the specific views (score mean, variance, per-dimension mean, positive/negative-recommendation split) and the intervention protocol (calibration conversation? shadow rotation refresh? de-list from the loop until re-calibrated?).

Discipline: the calibration cadence must be *self-sustaining*. If any mechanism requires the tech lead to remember to run it manually every quarter, it will lapse. Name the mechanism that ensures it runs (a calendar recurrence, a quarterly review agenda item, an ATS-automated trigger).

### 7. Mentee-training plan for a mid-level to interview at bar (≈ 25 min)

Chapter 04 §8 names the four-stage training arc — *shadow / co-pilot / pilot with observation / independent* — that mirrors the reviewer-training arc of chapter 02 §6 and the general mentorship arc of chapter 03 §3.

Pick one mid-level engineer on your team (real or role-played from exercise 03) and author the plan that gets them interviewing at bar within 6 months:

- **Shadow phase.** How many loops (2–3 typical), which rounds they'll shadow, how the private-rubric-then-compare artefact works, and what the calibration conversation looks like after each interview.
- **Co-pilot phase.** How many loops, which portion of the interview they lead (chapter 04 §8 suggests the technical warm-up first), how rubric calibration happens post-interview.
- **Pilot-with-observation phase.** How many loops, the lead-then-observe protocol, the post-interview calibration.
- **Independent.** The specific target signal — chapter 04 §8's default is "mid-level runs a full round independently, submits a rubric-scored writeup within 24 hours, and calibrates within one point of the panel's on the same candidate." Adopt this or set your own; either way, make it observable.

Address chapter 04 §8's two failure modes explicitly in the plan:

- **Rubber-stamp shadowing.** How you prevent it — the calibrator asks the shadow's rubric *first* before revealing their own.
- **Off-book piloting.** How you catch it — rubric compliance is part of the calibration; deviations are flagged and discussed.

Discipline: the plan should be plausible in 6 months for a mid-level who is willing and capable. If your plan takes 12 months, either the mid-level is not the right pick or the loop's throughput is too low to give them enough reps — flag either.

### 8. Reflection and failure-mode walk (≈ 15 min)

Walk your loop packet against the five failure modes of chapter 04 §9:

- **The one-question loop.** Do the four rounds cover the four rubric dimensions with the primary/secondary split the shape requires, or does the loop over-index on modelling depth?
- **The Ivy-League gauntlet.** Are the technical questions job-representative, or do they favour academic research signal over production ML signal?
- **The unstructured behavioural round.** Do the behavioural rounds have BARS anchors as tight as the technical rounds, or is one of them a "tell me about yourself" chat?
- **The debrief hijack.** Does the debrief structure (§4) enforce pre-submitted rubrics and a non-by-seniority facilitator?
- **The stale rubric.** Does the calibration cadence (§6) include a rubric-review moment where questions and anchors get retired or updated based on outcome retros?

For each: honest name of any hit, and the specific fix. If none hit, name the *closest miss* — the failure mode you'd be most at risk of drifting into over the next year, and the early-warning signal that would tell you it's happening.

## Deliverable

Two artefacts:

- **`interview-loop-senior-ml-engineer-<year>.md`** — 5–8 pages, containing the loop-context frame (§1), the four-part rubric with behavioural anchors (§2), the four fully documented questions (§3), the debrief structure and facilitator protocol (§4), the bias-and-defensibility checklist (§5), the calibration mechanisms with cadences (§6), the mentee-training plan (§7), and the failure-mode reflection (§8).
- **`hire-recommendation-memo-template.md`** — 1–2 pages, the template the debrief outputs against every candidate (from §4).

The two are versioned together and owned by the interview-loop lead; the packet lives in the standards library (chapter 03 §5–6) as the team's authoritative interview reference.

## Acceptance criteria

- [ ] The loop-context frame (§1) names the specific gap the hire fills and identifies at least one dimension where the bar is set *higher than the default* and at least one where the bar is set *at the default*.
- [ ] The rubric has behavioural anchors for all four dimensions at all five levels (1–5). Every anchor is observable behaviour, not a trait; a new interviewer could read the anchor cold and apply it to a candidate's answer.
- [ ] Each of the four fully documented questions has: question text, 3–5 follow-up probes, question-specific scoring anchors at levels 3/4/5, common pitfalls, and interviewer-side notes.
- [ ] At least one of the four questions is an end-to-end system design or debugging deep-dive (chapter 04 §4's ML-specific archetypes). If context is L3 or L2, at least one question probes the domain-specific senior signal.
- [ ] The debrief protocol (§4) requires rubrics submitted before the meeting, names a facilitator who is not by-seniority, and includes the four-block agenda (orient / dimension walk / divergence / recommendation).
- [ ] The hire-recommendation memo template is a runnable Markdown template with all required fields (candidate, panel, rubric scores per dimension, consensus/divergence, overall recommendation, conditional recommendations, onboarding-useful observations, evidence trail).
- [ ] The bias-and-defensibility checklist (§5) covers all three chapter 04 §7 disciplines — behaviourally-anchored anchors, no-proxy-for-demographics, contemporaneous auditable feedback — and names the owner, cadence, and remediation for each.
- [ ] The calibration cadence (§6) specifies numeric N/M for shadow-and-lead, defines the outcome-retro artefact and cadence, and defines the cross-interviewer score-distribution audit with the specific views. Every mechanism has a self-sustaining trigger (calendar / agenda item / ATS trigger) named.
- [ ] The mentee-training plan (§7) has all four phases (shadow / co-pilot / pilot with observation / independent) with round counts, artefacts, and an observable independence target. Both failure modes (rubber-stamp shadowing, off-book piloting) are explicitly addressed.
- [ ] The failure-mode walk (§8) honestly names any of chapter 04 §9's five modes the loop is closest to, with the specific fix or the early-warning signal.

## Stretch goals

- **The answer key.** For each of the four questions, author the *5 — clearly above bar* answer as a written response. This forces you to have prepared an answer key you could hand a new interviewer as a calibration exercise. If you find you cannot write the *5* answer, the question probably tests something you cannot reliably score — rewrite the question.
- **The candidate-facing prep note.** Chapter 04 §1 names *fairness to the candidate* as one of the reasons for structured interviews. Draft the 1-page prep note candidates receive before the loop — what the rounds are, what they should expect, what the rubric dimensions are (at a high level), what materials they can bring. Radical transparency in loop shape reduces bias against candidates who don't have inside knowledge of the process.
- **The rubric-vs-baseline diff.** If you started from chapter 04 §2's example rubric, produce a *diff* showing what you changed for your team context and *why*. This is the artefact that lets a future loop-lead understand the reasoning, and it composes with the standards-library entry the loop packet becomes (chapter 03 §6).
- **The escalation guide.** For scenarios where the debrief cannot reach consensus — panel splits 2/2, or the hiring manager disagrees with the panel — write the escalation protocol. Who decides? On what evidence? What is the deadline before the candidate is disadvantaged by the delay? This is the interview-loop analogue of chapter 02 §7's *when to escalate*.
- **The paired-track hand-off.** Some Senior ML Engineer candidates are actually a better fit for a paired specialist track (`ai-eval-engineer-learning`, `ai-infra-mlops-learning`, `ai-governance-analyst-learning`, `training-pipeline-engineer-learning`, or the AI-infra platform tracks). Author the *cross-loop referral protocol* — what signal in the interview tells the panel to raise "we should refer this candidate to the peer track's loop," and what the referral note looks like. Referral is not rejection; it is a service to both the candidate and the peer team.
- **The calibration-retro dry run.** Take three synthetic candidate scoresheets (author them yourself against your rubric — one strong-hire, one no-hire, one borderline) and walk the debrief protocol against them. Note where the anchors were easy to apply and where they were ambiguous. Anchors that were ambiguous in the dry run will be ambiguous in the real debrief — sharpen them before shipping the loop.
- **The one-page loop diagram.** Extract a one-page visual (or written) representation of the loop — the four rounds, their dimensions, their questions, the debrief, the hire-recommendation memo, the calibration mechanisms, the mentee-training arc. This is the artefact the hiring committee, the recruiter, and the candidate-facing prep note all reference; a loop without a one-page representation is a loop that gets described inconsistently.
- **The rubric-drift retrospective.** Assume 12 months have passed and 15 candidates have been through the loop. Author the *rubric-drift retro* — the questions that stopped discriminating, the anchors that turned out to be ambiguous, the dimensions where the panel converged too easily (a sign the anchors are too permissive), and the dimensions where the panel diverged too widely (a sign the anchors are too vague). This is the artefact that keeps the loop honest over years.
