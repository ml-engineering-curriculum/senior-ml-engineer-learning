# Self-assessment and growth plan

## Motivation

The rest of this track is nine modules and three projects, roughly ~200 hours. Working them in strict order is the *default* schedule — it is not necessarily the *right* schedule for you. A senior engineer already coming from a strong architecture background does not need to spend the first sixteen hours on mod-302; a senior engineer promoted from a modelling-heavy role needs to spend twice as much on mod-307. The point of this chapter is to give you a **calibrated self-assessment** so you can prioritise honestly, and to give you a **growth plan** artefact you will actually run against.

Exercise-03 produces that plan. This chapter gives you the rubric.

## Core concept — the L30 competency map

The nine following modules (mod-302 → mod-310) are not nine independent skills. They cluster into five **L30 competencies**, each of which is a legitimate axis of senior scope. The map below is the working reference.

| # | L30 competency | The question you should be able to answer at bar | Owning modules |
|---|---|---|---|
| **A** | Architecture — take a business problem to a defensible ML system | *"Show me the ML system RFC for the last thing you took to production. What alternatives did you consider? Why not stream? Why not fine-tune?"* | mod-302 (systems architecture), mod-303 (advanced modelling regime choice), mod-304 (LLM integration decision) |
| **B** | Evaluation & experimentation — run the programme a team lives by | *"Walk me through your evaluation harness. What slice caught the last regression? What was your last A/B, and what was the guardrail metric?"* | mod-305 (advanced evaluation), mod-306 (experimentation at scale) |
| **C** | Reliability & cost — own the SLOs and incident response | *"What's your model-freshness SLO? Your inference cost budget? Walk me through the last regression incident you commanded."* | mod-307 (ML reliability & SLOs) |
| **D** | Cross-team & governance — own the contracts on the boundary | *"Which peer platform teams do you consume from? What's the last contribute-back RFC you wrote? Show me the review packet for a shipped model."* | mod-308 (platform collaboration), mod-309 (responsible AI & governance) |
| **E** | Technical leadership — grow the team | *"Walk me through the roadmap you set for your team. Show me a mid-level engineer whose PR you reviewed at bar. Show me an interview rubric you authored."* | mod-310 (technical leadership) |

The five competencies map back onto the five ownership axes from chapter 02 (Development / Systems / People / Process / Influence), with the *Development* axis handled by the L20 prerequisite `ml-engineer-learning` and not re-taught here.

## The five-point rubric for self-assessment

For each competency, rate yourself on the following five-point rubric. Be honest — the point is to spend the next 200 hours on your gap, not to feel good on paper.

- **0 — Novice.** You do not have a working mental model. You cannot describe the artefact you would produce.
- **1 — Aware.** You can read a well-formed artefact from someone else and follow the argument, but you have not produced one at bar.
- **2 — Emerging.** You have produced one, and it went through review. It needed rework.
- **3 — Solid.** You produce these regularly. They pass review largely as authored.
- **4 — Excellent.** You mentor others in this competency. Your artefacts are used as templates by peers.

At L30 you are aiming for **3 across all five** and **4 on at least one**. A "3-3-3-3-3" senior is a well-shaped senior; a "4-2-3-1-4" senior is a lopsided one who needs to invest in reliability (C) and cross-team (D). Both shapes are real. Neither is a failure mode. The point is to know where you sit.

Below L30 you will see 2s and 1s scattered across the row — that is the signal for what to prioritise. Above L30 (staff-scope) you will see 4s across the row and the growth plan for that role belongs to a different track (`staff-ml-engineer-learning`).

## Turning the self-assessment into a growth plan

The self-assessment tells you *where you are*. The growth plan tells you *what you will do about it, in the artefacts of this track*. A good growth plan has four parts.

### 1. The competency ranking

Rank the five competencies by (target level − current level). Break ties by *cost of not investing* — a 1 on Reliability is more expensive to not fix than a 1 on Leadership if you are about to be on call.

### 2. The module allocation

For every gap of ≥1 point, allocate a specific module and a specific artefact from this track. The mapping in the competency table above is your default. If your gap is on Competency A (Architecture) and you already scored a 3 on mod-303's material coming in, spend the module reading budget on mod-302 and skip the mod-303 warm-ups.

Represent the plan concretely — hours per module and the artefact you will produce, not just "study more." Example:

- **Competency B gap (2 → 3):** mod-305 (12h), mod-306 (12h), lab-05-01 (LLM-judge harness lab), project-301 RFC #2 (evaluation-heavy design). Produced artefact: a review packet template you will use for the rest of your career.

### 3. The stretch project

At least one of the three projects (project-301, project-302, project-303) is your **stretch project**. Pick the one whose artefact you are most nervous to author in front of a reviewer. That is the one that closes the gap fastest.

- Weak on Architecture (A)? project-301 (three senior-level RFCs).
- Weak on LLM integration (part of A) and evaluation (B)? project-302 (LLM-augmented ML feature, end-to-end).
- Weak on Leadership (E) and Cross-team (D)? project-303 (tech-lead simulation).

Choosing the stretch project is not the same as choosing the *only* project — the other two are also required. It is the one you approach first, or slow down on, or ask for review on twice.

### 4. The review cadence

A growth plan you never re-check is a wish list. Schedule at minimum a **mid-track review** (after mod-306 or thereabouts, ~100 hours in) where you re-run the five-point rubric and adjust the allocation. If you have a manager or mentor, this is a good conversation to bring them into. If you do not, run it against yourself and journal the delta.

## A concrete example — the growth plan for a lopsided senior

The rubric produces a rating vector like `A=3, B=2, C=1, D=3, E=2`. The growth plan writes itself:

- **Rank:** C (target 3, gap 2) → B (gap 1) → E (gap 1) → A/D no gap this cycle.
- **Module allocation:**
  - Competency C: mod-307 in full (12h). Focus on SLO authoring and cost budgets, since the incident-command craft is best learned on your real system. Read Google SRE Chapter 2–6 in parallel.
  - Competency B: mod-305 warmups, then the full mod-306 (A/B testing, SRM, CUPED). Kohavi/Tang/Xu is your reference book.
  - Competency E: mod-310. Author one review rubric and mentor one mid-level engineer through a real change during the track.
  - Competency A: read mod-302 & mod-304 chapters, skip warmups, run the exercises only if the module rubric surprises you.
  - Competency D: read mod-308 & mod-309, run only the exercises that touch cross-team artefacts you have not authored (contribute-back RFC, review packet).
- **Stretch project:** project-301 (three RFCs) is a comfort project. project-303 (tech-lead simulation) is uncomfortable. Pick project-303 as the stretch. Do it second — do not do it under stress.
- **Cadence:** re-rate at 100 hours. If Competency C is now a 3, the plan for the second half is different.

## A concrete example — the growth plan for a strong-across senior

The rubric produces `A=3, B=3, C=3, D=3, E=3`. The growth plan is not "coast" — it is:

- Where in the table are you *closest to a 4*? That is your investment target. If it is B (Evaluation), your stretch project is project-302 and your artefact is a shared review-packet template your organisation will adopt.
- Where is the module material *newest to your organisation*? A "3" on Reliability in a team that has never seen an ML SLO is worth an artefact even if you personally do not need the material — that is a Leadership (E) contribution.
- The track is still worth running end-to-end. The gap for a strong-across senior is not on any single axis — it is on **producing artefacts that raise the org's bar**, which is the L30 → L40 stretch and the point of mod-310 and project-303.

## The artefact this module produces

By the end of the module you should have:

- A **self-assessment sheet** with your five competency ratings and one sentence of evidence per rating (produced in exercise-03).
- A **growth plan** with module allocation, stretch project pick, and review cadence (produced in exercise-03).
- Optionally a **mentor / manager conversation note** if you have someone to run the plan by.

You will re-open the self-assessment sheet at least once more, at the mid-track review. Keep it under version control — you will want the delta.

## Summary

The nine following modules cluster into five L30 competencies: Architecture, Evaluation & Experimentation, Reliability & Cost, Cross-team & Governance, Technical Leadership. Rate yourself 0–4 on each, aim for 3 across and 4 on at least one, and turn the gap into a growth plan with module allocation, a stretch project pick, and a mid-track review cadence. Exercise-03 walks you through producing the artefact.
