# exercise-03: Self-Assessment and Growth Plan

**Estimated effort:** 3 hours

## Objective

Produce two artefacts you will carry across the rest of the track:

1. A **self-assessment sheet** rating you 0–4 on each of the five L30 competencies (Architecture; Evaluation & Experimentation; Reliability & Cost; Cross-team & Governance; Technical Leadership), with one sentence of evidence per rating.
2. A **growth plan** that turns the self-assessment into a module allocation, a stretch-project pick, and a review cadence for the rest of the ~200-hour track.

You will re-open both artefacts at the mid-track review, and again at the end. Keep them under version control — you will want the delta as evidence of the growth you actually did.

## Prerequisites

- Read `05-self-assessment-and-growth-plan.md`.
- Ideally: have completed exercise-01 (verb audit) and exercise-02 (hand-off map). Both feed the self-assessment. The verb audit gives you a calibration for what L30 sounds like in the wild; the hand-off map surfaces which contracts you have not yet had to author.
- Skim the READMEs of mod-302 through mod-310 so you know what you are rating yourself against. You do not need to read the chapters — the READMEs are enough to expose the module's scope.

## Steps

### 1. Rate yourself on the five competencies (≈ 45 min)

For each competency below, apply the five-point rubric from `05-self-assessment-and-growth-plan.md` and write **one sentence of evidence** justifying the rating. If you cannot write the evidence sentence, you do not have the rating — drop it by one and note why.

For each row, use the shape below:

```
Competency A — Architecture (mod-302 / mod-303 / mod-304)
Target level: 3
Current level: __ (0–4)
Evidence: <one specific sentence — an RFC you authored, a system you own, a review you passed. "I read Chip Huyen" is not evidence.>
```

Do all five competencies:

- **A — Architecture** (mod-302 systems architecture, mod-303 advanced modelling regime choice, mod-304 LLM integration decision).
- **B — Evaluation & Experimentation** (mod-305 advanced evaluation, mod-306 experimentation at scale).
- **C — Reliability & Cost** (mod-307 ML reliability & SLOs).
- **D — Cross-team & Governance** (mod-308 platform collaboration, mod-309 responsible AI & governance).
- **E — Technical Leadership** (mod-310 technical leadership).

Two calibration reminders:

- The rubric is *artefact-based*, not *knowledge-based*. "I know what CUPED is" is a 1. "I have run three A/B tests with CUPED variance reduction and one of them exposed sample-ratio mismatch that I diagnosed and fixed" is a 3.
- At L30 you are aiming for **3 across the row** and **4 on at least one competency**. Below the 3 target is a gap you will invest in; the 4 target is your area of leverage. Realistic senior engineers are lopsided; the point is to name the shape.

### 2. Rank the gaps and note cost of inaction (≈ 20 min)

Compute per-competency gap = target − current. List the competencies in descending order of gap. For each competency with gap ≥ 1, add a **cost-of-inaction** sentence: what breaks in the next 6 months if you do not close this gap? Examples:

- "Reliability gap: if I do not close this before the next quarterly on-call rotation, I will be an incident commander for a regression I cannot debug."
- "Leadership gap: if I do not close this, the mid-level engineer I'm supposed to mentor will not grow this cycle, and I will be graded down on People at review."

Cost of inaction is what breaks tie in the ranking. A 2-point gap on Reliability with an on-call rotation coming up outranks a 2-point gap on Leadership with no near-term mentorship expectation.

### 3. Allocate the modules and hours (≈ 45 min)

Using the module hours from `CURRICULUM.md` (the total is ~122 hours across mod-302 through mod-310), allocate them to the growth plan. Two rules:

- **Do not skip modules with a gap of ≥ 2.** Read them in full, do all the exercises, do the lab, do the quiz.
- **Reallocate time from modules where you are already at target.** If you scored a 3 on Competency A coming in and mod-302 has 16 planned hours, you can compress to ~8 (skim chapters, run the RFC exercise but skip the warm-ups). Use the reclaimed hours on the gap modules.

Produce a concrete allocation table:

| Module | Planned hours | Your allocation | Reason |
|---|---|---|---|
| mod-302 | 16 | 16 | Gap of 2 on Architecture. Read in full. |
| mod-303 | 14 | 6 | At target. Skim, focus on cold-start section only. |
| ... | | | |

The sum of "your allocation" does not have to equal 122 — the gap should determine total investment. But mark whether you are spending more or fewer hours than planned and why.

### 4. Pick the stretch project (≈ 20 min)

Read the three project READMEs briefly (`projects/project-301-*`, `projects/project-302-*`, `projects/project-303-*`) and pick one as the stretch project — the artefact you are most nervous to author in front of a reviewer.

Default heuristic (from chapter 05):

- Gap on Architecture → project-301 (three senior-level RFCs).
- Gap on LLM integration + Evaluation → project-302 (LLM-augmented ML feature, end-to-end).
- Gap on Leadership + Cross-team → project-303 (tech-lead simulation).

Record:

- Which project you chose and why (one paragraph).
- Which of the other two projects will you run *first* (as a warm-up) and which will you run *last* (as a capstone). Do not run the stretch project first or last unless you have a specific reason.
- One reviewer or mentor you will ask to review the stretch project's deliverables. If you do not have one, name the artefact you will share publicly (a blog post, a portfolio README, a GitHub issue in the paired solutions repo) to force yourself to hold a public bar.

### 5. Set the review cadence (≈ 15 min)

Write the review cadence you will actually hold yourself to. At minimum:

- A **mid-track review** at ~100 hours in (roughly after mod-306). Re-run the five-point rubric. Update the allocation for the second half.
- An **end-of-track review** after all three projects are shipped. Re-run the rubric a third time. Compare the three snapshots. Which competency moved the most? Which moved less than you expected? What is the next-cycle plan (staff scope, deeper specialism, or lateral into a peer track)?

Optional:

- A **weekly journal note** — one line per module completed: "did mod-3XX exercise-N, updated my rating on competency Y from 2 → 2.5." This is the finest-grained data. It is optional because compliance is hard; if you have shown weekly-journal discipline before, do it.
- A **manager or mentor check-in** at each formal review. Even if the plan does not change, articulating it out loud tends to expose a hidden mis-rating.

### 6. Reflect on the shape of the plan (≈ 15 min)

Close with a short reflection (3–5 sentences) answering:

- What shape does your growth plan take — even across all five competencies, or heavily weighted to one or two?
- What is the *risk* in this plan? (For example: "I am spending 40% of my hours on Reliability, which means if I under-invest in Architecture I will not be able to answer the architecture question in a promotion loop.")
- What would you have to *learn about yourself* mid-track to change the plan? (For example: "If my Reliability rating jumps to 3 after mod-307 alone, I will reallocate the remaining hours to Leadership.")

## Acceptance criteria

Submit a single Markdown document `growth-plan.md` containing:

- [ ] Five self-assessment rows: competency, target, current, one sentence of evidence.
- [ ] A ranked gap list with cost-of-inaction sentences for every competency with a gap of ≥ 1.
- [ ] A module-allocation table across mod-302 → mod-310, with reasons.
- [ ] A stretch-project pick with a paragraph of reasoning, an ordering of the other two projects, and a reviewer or public-artefact commitment.
- [ ] A review cadence with at least the mid-track and end-of-track reviews.
- [ ] A 3–5-sentence closing reflection on the shape of the plan and its risks.

The document should fit under 4 pages. If it is longer, you are over-planning — get moving.

## Stretch goals

- **Bring in exercise-02's map.** If you produced the hand-off map in exercise-02, tie each contract on the map to a competency. Which contracts are you weakest on? Do they cluster on a specific competency (usually Cross-team & Governance, sometimes Reliability)? Does that change the ranking in your growth plan?
- **Sanity-check against a posting.** If exercise-01 surfaced a genuine L30 posting you would actually apply for, hold your rating vector next to that posting's compressed senior verb signature. Which slots in the signature would you be under-qualified for today? Would the growth plan close the gap by the end of the track? If not, revise.
- **Publish the plan.** Put the growth plan in a personal GitHub repo (private is fine) and commit the updated snapshots at each review. The delta between the three snapshots is the artefact you will hand to your promotion committee or your next hiring manager — evidence of calibrated self-directed growth is the point of the L30 → L40 stretch and is what mod-310 asks you to be able to induce in the people you mentor.
