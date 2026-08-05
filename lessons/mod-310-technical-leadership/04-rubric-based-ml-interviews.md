# Rubric-based ML interviews and calibration on bar signals

## Motivation

At L20 you *take* interviews. Sometimes you're pulled in as an extra pair of hands on a loop; you ask a question you know the answer to; you write "hire" or "no hire" in the feedback form.

At L30 you *run* interviews. You design the loop, you write the rubric, you calibrate the panel, you sit on the debrief that decides whether the candidate joins the team. If the loop hires poorly, the team pays for years — a bad ML hire compounds through PRs, design decisions, and the standards library. If the loop hires *unfairly* — inconsistently, arbitrarily, biased toward a shape of candidate that doesn't reflect the job — the team pays legally, culturally, and in reputation with future candidates.

Two shifts define the L30 interviewing discipline:

- **The interview is a *rubric-driven* structured assessment**, not an open-ended chat. Structured interviews are one of the most-studied topics in industrial-organisational psychology, and the empirical finding is stark: **structured, rubric-driven interviews predict job performance about 2× as well as unstructured ones and reduce demographic bias substantially** ([Schmidt & Hunter, 1998](https://mavweb.mnsu.edu/howard/Schmidt%20and%20Hunter%201998%20Validity%20and%20Utility%20Psychological%20Bulletin.pdf); [Levashina et al., 2014](https://onlinelibrary.wiley.com/doi/abs/10.1111/peps.12052)). The senior discipline is to bring that finding into practice, not to run interviews on gut.
- **The loop is *calibrated* against real bar signals**, not against each interviewer's private taste. Calibration is the practice of comparing signals across interviewers, across candidates, and against actual on-the-job performance — and adjusting rubrics, questions, and scoring accordingly. Uncalibrated loops drift; calibrated loops hold their bar over years.

Two primary external references anchor the chapter:

- **Google's [*Work Rules!*](https://www.workrules.io/) (Laszlo Bock, 2015)** and the [Google re:Work *Guide to Structured Interviewing*](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/introduction/) — the load-bearing external treatment of structured interviewing at scale, including the internal research that led Google to abandon unstructured "brainteaser" interviews. Bock's chapter 5 (*Why We Hire Only the Best*) is the corporate-scale companion to the psychology literature.
- **Aline Lerner and interviewing.io's [*Anatomy of a Great Interview*](https://interviewing.io/blog/hiring-loops-vs-interviews)** — the load-bearing external treatment of *loop design*, calibration, and the practical mechanics of running a rubric-driven interview well. The interviewing.io blog is one of the most rigorous public sources on interview mechanics.

The technique is not ML-specific in its shape — the same structured-interview discipline works for backend, frontend, and infrastructure loops. What this chapter installs is the *ML-specific rubric content* — the modelling, systems, reliability, and judgement questions that separate an L20 ML engineer from an L30 candidate — and the *ML-specific calibration signals* that let a panel converge on a bar over time.

## §1 — Why rubric-based, and why structured

Three properties of a rubric-driven structured interview are what make it more predictive and less biased than an unstructured chat:

- **The question set is the same across candidates.** Different interviewers may ask *different* questions in a loop, but two candidates going through the same loop see comparable questions. Comparability is what makes cross-candidate scoring meaningful; a loop where every candidate sees a different question is a loop that cannot calibrate.
- **The evaluation dimensions are named in advance.** Before the interview, the interviewer has a written list of dimensions ("modelling depth," "systems reasoning," "collaboration signal") and the observable behaviours that would score high, medium, or low on each. After the interview, the interviewer scores against those dimensions — not against an overall "gut feel."
- **The scoring is done *before* the debrief.** Every interviewer submits their scored rubric before the debrief meeting. This prevents the loudest voice in the debrief from anchoring everyone else — a well-documented phenomenon called *groupthink cascade* in the meta-analyses.

The [Schmidt & Hunter (1998)](https://mavweb.mnsu.edu/howard/Schmidt%20and%20Hunter%201998%20Validity%20and%20Utility%20Psychological%20Bulletin.pdf) meta-analysis of 85 years of hiring-methodology research is the load-bearing evidence. Unstructured interviews have a predictive validity of ~0.20 (weakly correlated with performance); structured interviews score ~0.51. The methodological rigor at the top of that scale (work-sample tests, structured interviews with behaviourally-anchored ratings) is what an ML hiring loop should aspire to.

Structured interviewing is *not* the same as scripted interviewing. The interviewer still probes, follows up, and adapts to the candidate — but within the frame of the rubric. Bock's chapter puts it: *"structure is the scaffolding, not the cage."*

The **anti-goals** matter as much as the goals:

- **No brainteasers.** "How many golf balls fit in a school bus?" was rigorously studied by Google itself and shown to be uncorrelated with job performance. Bock's book documents the abandonment; the [Google re:Work guide](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/introduction/) is the current corporate position.
- **No trivia gauntlets.** "What's the time complexity of TF-IDF?" isn't a modelling depth signal; it's a memory signal. If the trivia is load-bearing on the job, ask the deeper question that would reveal whether the candidate could *find* the answer on the job.
- **No unstructured "culture fit" gate.** *Culture fit* as an unstructured gate is where bias concentrates. If the loop needs to assess collaboration or communication, name it as a rubric dimension with observable behaviours; don't leave it as a private judgement.

## §2 — The four-part ML rubric

Below is a minimum viable ML-interview rubric for a Senior ML Engineer loop. It has four dimensions; each dimension has a behaviourally-anchored scale from "1 — no signal / concerning" through "3 — meets bar" to "5 — clearly above bar." The scale is deliberately named in behaviours, not in points — a *behaviourally-anchored rating scale* (BARS), the well-studied format from I/O psychology ([Smith & Kendall, 1963](https://doi.apa.org/doiLanding?doi=10.1037%2Fh0047343)).

### 2.1 Modelling depth

**What it measures.** Can the candidate reason about model choice, loss design, evaluation, and failure modes at the depth a senior ML engineer needs? Includes ML foundations (bias/variance, regularisation, cross-validation), applied modelling (classical + neural + LLM-augmented), and — the load-bearing L30 signal — *the ability to notice when a modelling problem is really a data problem or an evaluation problem*.

**Behavioural anchors.**

- **1 — Concerning.** Candidate proposes a model choice without discussing evaluation. Confuses correlation with causation without hedging. Cannot articulate what could go wrong with their approach.
- **2 — Below bar.** Candidate discusses model choice reasonably but is shallow on evaluation. Doesn't ask about data volume, class balance, or slice matrix. Names one metric without discussing tradeoffs.
- **3 — Meets bar.** Candidate proposes a model choice with rationale, discusses evaluation approach including metric choice and CI, names 1–2 failure modes that would be worth checking. Comfortable with classical baselines and at least one neural approach for the class of problem.
- **4 — Above bar.** Candidate proposes model choice and *baseline* to beat; discusses evaluation with slice matrix; names training/serving skew or data-drift as failure modes; discusses tradeoffs between competing model choices with cost/latency/quality/interpretability altitude.
- **5 — Clearly above bar.** All of the above, plus notices when the problem is really a data problem, a labelling problem, or an evaluation problem (mod-301 chapter 01's L20-vs-L30 example). Frames the choice in terms of decision points and evidence to gather.

**Sample question stems.** "Walk me through how you would build a <specific ML system>." "Here's an ML system that started degrading; how would you debug?" "You have a model with 95% accuracy on aggregate but a user complaint from one slice; what would you do?"

### 2.2 Systems and production reasoning

**What it measures.** Can the candidate reason about the *system around the model* — training/serving skew (mod-302), evaluation harness (mod-305), rollout gates (mod-306), SLOs and cost budgets (mod-307), the paved-road primitives they consume (mod-308)? This is where the L20-vs-L30 gap shows up most clearly.

**Behavioural anchors.**

- **1 — Concerning.** Candidate treats the model as the deliverable; is unaware of feature stores, model registries, or shadow deployments. Cannot articulate how a model would be tested in production.
- **2 — Below bar.** Candidate names some production concepts (offline eval, canary) but doesn't reason about them in depth. Cannot compose SLOs with dependency SLOs.
- **3 — Meets bar.** Candidate reasons about training/serving skew as a first-class risk; knows to run shadow before canary; can propose a rollout plan with guardrail metrics; understands cost surface.
- **4 — Above bar.** Candidate proposes an eval harness with primary metric, guardrails, slice matrix, CI shape (mod-305); reasons about the retraining cadence and its interaction with product cycles (mod-307); names the on-call runbook as an artefact of the change.
- **5 — Clearly above bar.** All of the above, plus reasons about hand-off contracts (mod-308 chapter 04) with peer teams (platform, training-pipeline, evaluation, governance); connects the systems reasoning back to the roadmap altitude (chapter 01 of this module).

**Sample question stems.** "How would you design the offline evaluation for this model?" "Walk me through the rollout from a merged PR to 100% traffic." "What does the on-call runbook look like for this model six months after launch?"

### 2.3 Judgement, tradeoffs, and communication

**What it measures.** Can the candidate reason about tradeoffs under ambiguity, make and defend judgement calls, and communicate at the altitude of the reviewer they are talking to? This is the *senior-verb* dimension (mod-301 chapter 03) — *own*, *drive*, *design*, *influence*.

**Behavioural anchors.**

- **1 — Concerning.** Candidate defers all tradeoff questions to "it depends" without engaging. Cannot articulate a decision they own, or a decision they got wrong. Communication is opaque; the interviewer has to reverse-engineer intent.
- **2 — Below bar.** Candidate discusses tradeoffs at surface level. Reasons in isolation ("I would use approach X") without engaging alternatives. Communication is workable but not crisp.
- **3 — Meets bar.** Candidate discusses tradeoffs with named alternatives; articulates decisions they've owned; distinguishes what was their call from what was the team's; communicates clearly.
- **4 — Above bar.** Candidate frames tradeoffs at the altitude of the reviewer (product-vs-technical, technical-vs-governance); adjusts vocabulary to the audience; names a decision they got wrong and what they learned.
- **5 — Clearly above bar.** All of the above, plus surfaces the *unasked* question ("you're framing this as a model problem, but I think it's actually an evaluation problem — is that a valid reframe?"); shows evidence of running RFCs, design reviews, or roadmaps at the L30 altitude of this module.

**Sample question stems.** "Tell me about a technical decision you owned that turned out to be wrong." "Two engineers on your team disagree on model choice X vs. Y; walk me through how you'd get to a decision." "You're presenting this system to a director; what do you cover in the first three minutes?"

### 2.4 Collaboration, mentorship, and cross-team signal

**What it measures.** Does the candidate operate as a collaborator, mentor, and cross-team partner in the shape a senior ML engineer needs to? This is the *culture add* dimension, kept explicit and behavioural to avoid it becoming a smokescreen for cultural bias.

**Behavioural anchors.**

- **1 — Concerning.** Candidate frames engineering as an individual sport; dismisses code review or design review; disparages past colleagues or teams; treats junior engineers as annoyances.
- **2 — Below bar.** Candidate mentions collaboration but doesn't have concrete examples of mentoring, running reviews, or working across teams.
- **3 — Meets bar.** Candidate has concrete examples of collaboration on a project, mentoring a peer, or working across a team boundary. Speaks respectfully about past colleagues.
- **4 — Above bar.** Candidate has led design reviews or run RFCs (mod-308 chapter 03); has mentored a junior or mid-level to a specific outcome (chapter 03 of this module); has partnered with a peer team on a hand-off (mod-308 chapter 04).
- **5 — Clearly above bar.** All of the above, plus has *scaled themselves* through a standards library, a paved-road contribute-back, or a mentorship arc that closed with the mentee themselves mentoring others.

**Sample question stems.** "Tell me about a time you mentored someone through a growth arc." "Tell me about a cross-team disagreement and how you resolved it." "What's a piece of engineering culture on your current team you'd bring with you, and one you'd leave behind?"

## §3 — The four-round loop shape

A Senior ML Engineer loop typically has four rounds, each covering one or two of the rubric dimensions. Four rounds is the sweet spot between *enough signal* (each round is 60–75 minutes; less than four rounds is thin evidence for a senior hire) and *not exhausting the candidate* (more than four rounds is a signal-to-fatigue tradeoff that starts favouring fatigue).

- **Round 1 — Modelling depth (technical, 75 min).** A concrete ML system to design or debug. Working through data → model → evaluation → failure modes. Rubric dimension: modelling depth (§2.1); some signal into systems reasoning (§2.2).
- **Round 2 — Systems and production reasoning (technical, 75 min).** A concrete ML system to design at the production altitude — rollout plan, SLOs, hand-offs, retraining. Rubric dimension: systems reasoning (§2.2); some signal into modelling depth.
- **Round 3 — Judgement and communication (behavioural, 60 min).** Past work, decisions owned, tradeoffs made, reviews run. Rubric dimension: judgement (§2.3); some signal into collaboration.
- **Round 4 — Collaboration, mentorship, cross-team (behavioural, 60 min).** Past mentorship arcs, cross-team partnerships, culture examples. Rubric dimension: collaboration (§2.4); some signal into judgement.

Two rounds are technical (deep dives into what the candidate can do); two are behavioural (deep dives into how the candidate operates). Each round is owned by one interviewer, who scores against their primary rubric dimension and takes secondary signal on the adjacent dimension.

A hiring-manager conversation (30 min) often bookends the loop but is not scored — it's a two-way conversation about scope, growth, and expectations. The hiring manager may sit in the debrief but with a specific role (context and hire recommendation, not evaluation).

## §4 — Writing the interview questions

Question design is where most loops leak signal. Three properties of a well-designed question:

- **It admits multiple valid answers.** A question with one right answer is a trivia question. A question with multiple valid answers reveals the candidate's *reasoning* — how they choose, what tradeoffs they weigh, what they'd probe further.
- **It probes at multiple depths.** The interviewer can drill deeper based on the candidate's answer. A candidate who nails the surface question is asked the follow-up that tests the next depth; a candidate who struggles at the surface is scaffolded rather than moved on.
- **It's job-representative.** The question resembles work the candidate would actually do. "Design an eval harness for this recommendation model" is job-representative; "solve this graph puzzle on the whiteboard" is not, for most ML roles.

The [Google re:Work *interview question design*](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/design-your-questions/) template is the canonical shape:

- **Question.** The specific prompt the interviewer opens with.
- **Follow-up probes.** The next 3–5 questions the interviewer uses to drill into depth.
- **Scoring anchors.** What a 3 (meets bar), 4 (above bar), and 5 (clearly above bar) answer looks like *for this question*.
- **Common pitfalls.** Things candidates say that sound good but reveal shallow understanding.

Each question in the loop is documented at this depth. The document lives in the standards library (chapter 03), owned by the interview-loop lead, versioned, and updated based on calibration signal (§6).

Two ML-specific question archetypes worth naming, both from the *[interviewing.io ML-interview guide](https://interviewing.io/guides/machine-learning-engineer-interview)*:

- **The end-to-end system design.** "Design an ML system for <specific product surface>." Covers data → model → training pipeline → serving → evaluation → rollout → monitoring. A senior candidate hits all of these; a mid-level candidate stalls at the modelling section without reaching production.
- **The debugging deep-dive.** "This ML system started regressing; here's the offline eval, here's the online metric, here's the alert; what do you do?" Signal into the L30 discipline of separating model, data, and evaluation problems (mod-301 chapter 01).

## §5 — The debrief

The debrief is the meeting where the loop makes its decision. Two properties of a debrief that works:

- **Every interviewer submits their scored rubric *before* the debrief.** This is non-negotiable. A debrief where the first score comes out in the room is a debrief that will anchor on the loudest voice. Submitting in advance forces each interviewer to commit privately, and the debrief becomes a discussion of the *reasoning* behind the scores, not a live consensus-forming exercise.
- **The debrief is *facilitated*, not chaired by seniority.** The facilitator's job is to walk each rubric dimension across all interviewers, ensure every interviewer speaks, and note where the panel converges and diverges. The most-senior interviewer is *not* the facilitator by default — that flatters seniority at the cost of calibration.

A typical debrief runs 45–60 minutes for a four-round loop. The structure:

- **Facilitator walks the rubric.** For each of the four dimensions: "Here's what each interviewer scored, in one word. Interviewer X, walk your reasoning; Interviewer Y, walk yours; Interviewer Z, add any signal I've missed."
- **Panel discusses points of divergence.** Where interviewers disagreed materially (e.g., one scored 4 on modelling depth, another scored 2), the panel discusses *what specifically* the interviewers observed. Often the divergence resolves — one interviewer heard signal the other missed. Sometimes it doesn't — and the honest recording is "the panel disagreed on modelling depth signal."
- **Hire recommendation, per interviewer, then panel.** Each interviewer states their hire/no-hire recommendation, with the primary rubric evidence. The facilitator captures the panel's consensus (or explicit disagreement) and drafts a recommendation memo.
- **Hire recommendation goes to the hiring manager (or hiring committee).** In small orgs, the panel's recommendation is the decision. In larger orgs, a separate hiring committee reviews the packet.

The **hire recommendation memo** is the artefact the loop produces. It contains:

- Panel members and their rubric scores.
- Points of consensus and divergence per dimension.
- Overall recommendation (strong hire, hire, no hire, strong no hire).
- Any *conditional* recommendations (e.g., "hire with strong onboarding plan on production systems").
- Notable observations that fall outside the rubric — often useful for onboarding.

## §6 — Calibrating on real bar signals

Calibration is the practice of making sure the loop's bar matches the *actual bar* the team is trying to hire against. Uncalibrated loops drift — bar gets tighter or looser as different interviewers rotate through, or as the panel loses institutional memory of what "meets bar" means.

Three calibration mechanisms:

- **Shadow-and-lead rotation for new interviewers.** Every new interviewer shadows 2–3 loops before they lead a round; leads 2–3 rounds with a shadowing calibrator before their scores count; then their scores count independently. This is the same *shadow / co-pilot / pilot* structure chapter 03 §6 used for reviewer training and chapter 03 §3's mentorship arc. [interviewing.io's shadow-interview guide](https://interviewing.io/blog/how-to-shadow-interviews) is the load-bearing reference.
- **Score-against-outcome calibration.** For candidates who joined, six months and twelve months into their tenure, the hiring panel revisits the loop's scores against actual on-the-job performance. Where the loop *scored high* on a candidate who *underperformed*, what did the loop miss? Where the loop *scored low* on a candidate who *outperformed*, what signal was hidden? These retros feed the rubric and the question design. Bock's *Work Rules!* chapter 6 is the reference on Google's version of this practice.
- **Cross-interviewer score-distribution audits.** Quarterly, the loop lead looks at each interviewer's score distribution — are they scoring higher than the panel average, lower, more variance, less? An interviewer who scores 4.5 on every candidate is providing no signal; an interviewer who scores 2 on every candidate is a bar-drifter in the other direction. The audit is not adversarial — it's a calibration conversation, and often the interviewer's rubric interpretation needs to align with the panel's.

Calibration is the discipline that makes the rubric *useful*. A rubric that everyone interprets differently is a rubric-shaped ornament on an unstructured interview. Calibration is what turns the rubric into a shared vocabulary.

The load-bearing conceptual reference is Kahneman's *[Noise: A Flaw in Human Judgment](https://us.macmillan.com/books/9780316451383/noise/)* (2021), specifically the *decision hygiene* chapter. Noise reduction (interviewer-to-interviewer variance on the same candidate) is one of the most-studied and most-underinvested-in interviewing practices; calibration is the tool.

## §7 — Bias, fairness, and legal-defensibility discipline

Structured interviewing is not only more predictive — it is more *defensible* against bias and legal claims. Unstructured interviews are one of the primary vectors for demographic bias in hiring; structured interviews measurably reduce it ([Levashina et al., 2014](https://onlinelibrary.wiley.com/doi/abs/10.1111/peps.12052); [Culbertson et al., 2016](https://journals.aom.org/doi/10.5465/annals.2013.0016)).

Three disciplines the senior interviewer runs:

- **Behaviourally-anchored, not trait-anchored.** Every rubric dimension is anchored in *behaviours* ("proposed a model choice with rationale"), not *traits* ("smart," "curious," "leadership"). Trait-anchored rubrics leave interpretation to the interviewer's implicit bias; behavioural anchors constrain it. The [SIOP *Principles for the Validation and Use of Personnel Selection Procedures*](https://www.siop.org/Portals/84/PDFs/Principles.pdf) is the canonical reference.
- **No proxy signals for demographics.** The rubric does not ask about (or reward) signals that correlate with demographic categories without being job-relevant. Prestigious school, "cultural fit," communication style — all common vectors for unintentional bias. If a signal is worth scoring, it goes in the rubric behaviourally.
- **Feedback in writing, contemporaneous, and auditable.** Every interviewer's rubric and notes are written up within 24 hours of the interview, filed in the applicant-tracking system, and retained for the legally-required window (in the US, [EEOC guidance](https://www.eeoc.gov/laws/guidance/section-6-lawful-employment-decisions) suggests 1 year for private employers, longer for federal contractors — check your legal team's guidance). This protects the candidate (fair review), the company (legal defensibility), and future calibration (a signal-retention practice).

The senior interviewer's discipline is that the loop is *legally auditable* — every hire and no-hire decision can be reconstructed from the artefacts, and the artefacts show a rubric-consistent rationale. This is not just a legal-cover exercise; it is the discipline that produces a fair loop.

## §8 — Mentoring a mid-level to interview at bar

The interview loop, like the review process (chapter 02) and the standards library (chapter 03), is another axis where the senior scales themselves by mentoring others. A mid-level engineer trained to interview at bar is what makes the hiring loop's throughput exceed the senior's own interview capacity.

The training arc mirrors the shadow/co-pilot/pilot pattern of chapter 03 §3:

- **Shadow (2–3 loops).** The mid-level sits in on interviews as a silent second, writing their own private rubric alongside the lead interviewer. After each interview, the two compare rubrics and calibrate.
- **Co-pilot (2–3 loops).** The mid-level leads part of the interview (often the technical warm-up) with the lead interviewer taking the deeper question. Rubrics are still calibrated post-interview.
- **Pilot with observation (2–3 loops).** The mid-level leads the full interview; the lead sits in silently and observes only. Post-interview calibration.
- **Independent.** The mid-level's scores count independently. The lead is available for calibration questions but not routinely present.

The training arc has a legible target: the mid-level runs a full interview loop round independently, submits a rubric-scored writeup within 24 hours, and their scores calibrate to within one point of the panel's on the same candidate. When that target is met, the mid-level is interviewing at bar — and the loop's capacity has doubled.

Two failure modes to guard against in the training arc:

- **Rubber-stamp shadowing.** The shadow sits silently, agrees with the lead, and never surfaces their own signal. The fix: the calibrator *asks* the shadow's rubric first, before revealing their own — the shadow's independent read is the signal.
- **Off-book piloting.** The mid-level runs the interview but off the rubric, back to unstructured. The fix: rubric compliance is part of the calibration; deviations are flagged and discussed.

## §9 — Failure modes to catch in your own loop

Five failure modes recur in ML interview loops. The senior discipline is to notice them and address them in the calibration retros of §6.

- **The one-question loop.** Every round is a variation of the same question — modelling design. The candidate is a strong systems reasoner but the loop never surfaces the signal. The fix: the four-round structure of §3 is not optional; drop any round only with an explicit rationale.
- **The Ivy-League gauntlet.** Rounds are pitched at the level of a research interview when the job is production ML. Candidates who could excel at the job are rejected because they can't derive backprop on the whiteboard. The fix: job-representative questions (§4); rubric alignment to the actual work; regular calibration against on-the-job performance.
- **The unstructured behavioural round.** The "tell me about yourself" round with no rubric, no anchors, no scoring discipline. The fix: behavioural rounds are rubric-scored (§2.3, §2.4); behavioural anchors are as tight as technical anchors.
- **The debrief hijack.** The most-senior interviewer speaks first in the debrief and everyone else agrees. Groupthink cascade. The fix: rubrics submitted before the debrief (§5); facilitator is not by-seniority; every interviewer speaks on every dimension.
- **The stale rubric.** The rubric hasn't been updated in two years; the role has evolved; the questions no longer match the work. The fix: quarterly rubric review; feedback from onboarded hires' six-month check-ins; retire questions that don't calibrate.

Each of these is fixable with the mechanisms of §§5–7 — rubric discipline, submitted-in-advance scoring, facilitator-led debrief, quarterly calibration retros. The senior's job is to *run those mechanisms*, not to be the sole holder of the bar.

## Summary

Rubric-based, structured ML interviews predict job performance about 2× as well as unstructured interviews and reduce demographic bias measurably (§1). The four-part ML rubric (§2) covers modelling depth, systems and production reasoning, judgement and communication, and collaboration and mentorship — each dimension behaviourally anchored on a 1–5 scale. The four-round loop shape (§3) has two technical rounds (modelling depth, systems reasoning) and two behavioural rounds (judgement, collaboration), each 60–75 minutes with one rubric-owning interviewer. Questions are designed with multiple valid answers, multiple depths of probe, and job-representativeness (§4). The debrief (§5) requires scored rubrics submitted before the meeting, a facilitator not chosen by seniority, and a written hire-recommendation memo. Calibration (§6) runs three mechanisms — shadow-and-lead rotation for new interviewers, six/twelve-month score-against-outcome retros, and quarterly cross-interviewer score-distribution audits — so the bar does not drift. Bias, fairness, and legal-defensibility discipline (§7) requires behaviourally-anchored rubrics, no proxy signals for demographics, and contemporaneous auditable feedback. Mentoring a mid-level to interview at bar (§8) uses the same shadow / co-pilot / pilot arc as reviewer training and general mentorship. Five failure modes (§9) — one-question loop, Ivy-League gauntlet, unstructured behavioural round, debrief hijack, stale rubric — each map to a specific mechanism to fix. The senior's job is to *run the loop's mechanisms* so the bar holds without depending on any single interviewer's private judgement.
