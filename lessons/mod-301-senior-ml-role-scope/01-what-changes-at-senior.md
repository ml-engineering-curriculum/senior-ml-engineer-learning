# What changes at Senior

## Motivation

If you have been shipping ML features for a few years, "senior" often sounds like "the same job, but you're expected to know more." That framing is wrong, and it is the reason a lot of ML engineers stall between L20 and L30. The senior tech-lead altitude is a **different job**, not a heavier version of the mid-level one. The units of work change, the artefacts you are graded on change, and what a good day looks like changes.

This chapter names the shift so you can measure yourself against it for the rest of the track.

## The three-line summary

At level 20 (mid-level ML Engineer, owned by [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning)) the unit of work is the **model or feature**: you take a well-scoped problem, build the model, and put it in production.

At level 30 (Senior ML Engineer, this track) the unit of work is the **system a team lives with**: you own the architecture, the evaluation programme, the reliability contract, and the cross-team hand-offs for a production ML system for as long as it exists.

At level 40 (Staff ML Engineer) the unit of work is the **portfolio across multiple teams**: you own the shared platforms, the multi-team roadmap, and the technical strategy the L30s inherit and execute against.

Every subsequent chapter drills into where the L30 line sits.

## Core concepts — what "senior" is actually made of

Four load-bearing shifts separate L30 from L20. All of them are visible in postings, in performance rubrics, and in the artefacts a senior engineer is graded on.

### 1. From "the model works" to "the system holds"

At L20 the deliverable is the trained, deployed, monitored model. At L30 the deliverable is the **system that keeps the model working for months**: training/serving skew mitigated at architectural altitude (mod-302), evaluation gates a team can rerun (mod-305), SLOs and cost budgets an incident commander can page on (mod-307), and a hand-off contract for the peer platform team paved-road you consume (mod-308). The rubric of *Hidden Technical Debt in Machine Learning Systems* (Sculley et al., NeurIPS 2015) is a good pocket check — most of a mature ML system's cost is the glue around the model, and owning that glue is the L30 shift.

### 2. From "I built X" to "the team ships X, at bar"

At L20 you are graded on your own throughput and your own PRs. At L30 you are graded on the **team's throughput and the team's bar**. Concretely: the design docs that come out of your team, the review comments that keep bad ML into production, the retros you run after a regression, the mentorship notes you leave on a mid-level engineer's PR. Google's *Engineering Practices* code-review guide is the canonical description of the bar you are now enforcing on other people's changes rather than merely receiving on your own.

### 3. From "my problem" to "who owns which contract"

At L20 you are usually inside one team's boundary. At L30 you are on the boundary. You have a **hand-off contract** with the ML platform team (they own the feature store; you consume it — mod-308), with the training-pipeline team (they own the cluster; you request runs), with the model-evaluation team (they own the harness; you author slices), with the LLM specialists (they own model choice / fine-tuning; you own integration — mod-304), and with the responsible-AI reviewers (they own the sign-off; you author the packet — mod-309). Chapter 04 formalises those contracts.

### 4. From "I decided" to "the decision is written down and defensible"

At L20 you can decide by intuition and let the code speak. At L30 the decisions have to survive being read by someone who was not in the room: a technical RFC with alternatives and non-goals (mod-302), a production-readiness review packet (mod-305), a roadmap that a director can defend against a business partner (mod-310). "It works on my model card" is not a level-30 answer.

## A concrete example — the same task at two altitudes

**Task:** "We are getting complaints that our search ranker's quality dropped this quarter."

**L20 framing.** Pull the recent metrics. Look at CTR by slice. Notice the drop is concentrated in one query cluster. Retrain with more recent data. Ship. Write a short summary and close the ticket.

**L30 framing.** Ask two questions first: *is this a model problem, a data problem, or a labelling problem?* and *what is the SLO we're actually violating?* Then:

- Pull the offline evaluation harness, rerun on the last four weeks of data, compare against the frozen baseline — is the regression reproducible offline, or only in production? If only production, we have a training/serving skew or a data-source drift, not a model regression (mod-302, mod-305).
- Check the retraining cadence and whether the last retrain was gated by the evaluation harness or bypassed (mod-307).
- If the ranker consumes an LLM re-ranker, check the prompt version and the LLM provider's model-version drift (mod-304).
- Draft a short RFC with two candidate fixes, their cost/latency implications, and the roll-out plan (canary → shadow → 100%). Circulate to the team.
- Assign the fix to a mid-level engineer, review their PR at bar, sign the review packet, and note the missing SLO for query-cluster-level quality so the next incident does not need this same investigation.

The L20 framing is not wrong — it is just one branch of the L30 framing. The senior is the person who owns the whole tree.

## What this module gives you

This module does not teach any of the above depth — the following nine modules do. What this module gives you is:

- A shared vocabulary for the shift (this chapter and chapter 02).
- A tool for reading job postings and calibrating what any given company means by "senior" (chapter 03 and exercise-01).
- A map of where your ownership ends and a peer track's begins, so you can spot when you are drifting into someone else's scope (chapter 04 and exercise-02).
- A self-assessment against the following nine modules so you can prioritise the rest of the track for yourself, not by module order (chapter 05 and exercise-03).

## Summary

Senior ML Engineer is not "mid-level plus experience." It is a role whose unit of work is a **system, not a model**; whose grade is **the team's output, not the individual's**; whose deliverables include **written decisions defensible to non-participants**; and whose day is spent as much on **the hand-off contracts around the system** as inside them. The next four chapters put that into operable form.
