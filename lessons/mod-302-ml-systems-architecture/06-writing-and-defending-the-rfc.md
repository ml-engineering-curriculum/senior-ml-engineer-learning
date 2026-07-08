# Writing and defending an ML system RFC

## Motivation

Everything in chapters 01–05 lands in a document. At L20 you may have written design docs; at L30 you are the *owner* of the ML system RFC — the artefact that (a) forces the team's decisions to be legible and defensible, (b) lets a reviewer who was not in the room raise the objection you missed, and (c) becomes the reference the team, the on-call, and the next engineer reads six months from now.

An RFC is not paperwork. It is the way a senior engineer *thinks in public*. This chapter gives you a standard RFC skeleton, tells you what each section is for, names the sections most under-written by juniors, and prepares you to defend the RFC in an architecture review — including how to hear "no" without churn.

## Why an RFC (as opposed to "just build it and iterate")

Two situations force the RFC:

- **Cost of reversal.** The four load-bearing decisions in chapter 01 (serving posture, feature substrate, model artefact contract, change management) are expensive to reverse. An RFC front-loads the cost of the reversal at review time, when it is cheap to change your mind, rather than at month six when it is not.
- **Cost of implicit knowledge.** The decisions have context (constraints, alternatives considered, non-goals) that lives in the reviewer's head. If the context is not written, it evaporates. The person who inherits the system in a year has no way to know why you chose batch, only that you did.

The RFC is the mechanism by which both costs are paid up front and once. Google's [*Engineering Practices*](https://google.github.io/eng-practices/) documentation is the industry-canonical description of the design-review discipline; the ["Design Docs at Google"](https://www.industrialempathy.com/posts/design-docs-at-google/) essay by Malte Ubl documents the specific shape.

## The RFC skeleton

Every ML system RFC an L30 signs contains, in order, at minimum:

### 1. Header

- **Title.** Descriptive, includes the system name (e.g., "RFC-042: Checkout Fraud Risk Model — Architecture v1").
- **Status.** Draft / In review / Approved / Superseded. Approved RFCs are immutable; changes are new RFCs that supersede.
- **Author(s), reviewers, approvers.** Explicit list. Approvers include the ML team's tech lead, the peer platform team's point of contact (`ai-infra-mlops-learning`, `ai-infra-ml-platform-learning` — see [mod-308]), and any review counterpart required (`ai-infra-security-learning`, `ai-governance-analyst-learning` — see [mod-309]).
- **Date.** Absolute (not "Q3").
- **Related documents.** Product spec, prior RFCs superseded, mod-305 evaluation plan.

### 2. Context and problem statement

One page maximum. What is the business problem? Who is the consumer? What are the current constraints (data available, latency SLO, compliance requirements, budget)? What is *not* the problem?

The reviewer's first question is always "why are we doing this at all?" This section answers it in the reviewer's language, not yours.

### 3. Non-goals

*Before* the proposal. The reviewer needs to know what you have deliberately excluded, so they do not spend their review time asking why the RFC does not cover it.

Concrete non-goal statements read like:

- "We are not building real-time streaming in v1. Freshness budget is daily; streaming can be re-evaluated when the business SLA changes."
- "We are not building a bespoke feature store. We are consuming the peer platform team's paved-road Feast deployment."
- "We are not addressing model explainability in v1. That is scoped for RFC-043."

If the reviewer objects to a non-goal, the discussion happens up front — cheaply — instead of after implementation.

### 4. Proposed architecture

The system as one clean picture. In practice: a diagram plus prose covering the four chapter-01 decisions:

- **Serving posture** (chapter 02) — batch / online / streaming; why.
- **Feature substrate** (chapter 03) — feature store or ad-hoc; the parity discipline.
- **Model artefact contract** (chapter 03) — registry choice, promotion contract.
- **Change management** (chapter 05) — retraining trigger, rollout strategy, abort conditions, rollback path.

Every decision is named against the chapter-01 constraints (freshness budget, latency SLO, volume, operability envelope). "We chose batch because Q1 answered 'days'" is what makes the decision defensible.

### 5. Alternatives considered — and rejected

The section juniors most consistently under-write. At L30 it is non-optional and it is *specific*: at least two other shapes you seriously considered, with a paragraph each on why they were rejected.

Read the "streaming because Kafka is cool" and "cron script forever" anti-patterns in chapter 02 as the shape of what belongs here. If you cannot describe a rejected shape without strawmanning it, you did not seriously consider it.

Alternatives keep you honest, and they arm the reviewer. The reviewer who asks "did you consider X?" and reads "yes, here is what we rejected and why" spends their time on your real risks, not on hypothetical ones you already thought about.

### 6. Data contract and feature contract

- What data does the system consume? Source-of-truth systems, refresh cadence, expected schemas, data owners.
- What features does the model consume? Names, types, expected value ranges, freshness, source. If a feature store is in use, the feature-service or feature-view names.
- What is the training population contract (filters, sampling)?
- What labels does the training pipeline consume? Latency-to-label, source, quality assumptions.

Chapter 04's five skew categories are avoided or accepted here.

### 7. Evaluation plan

- Offline metrics and their gates (mod-305 owns the slice discipline).
- Online metrics and their guardrails.
- The pre-registered abort conditions from chapter 05.

Reviewers cannot approve promotion without knowing how "good enough" is defined.

### 8. Reliability and operations

- SLOs and error budgets (see [mod-307]).
- On-call rotation and runbook location.
- Cost budget (compute, storage, external API dollars if any).
- Kill switch and rollback path.

Reviewers cannot approve a system without knowing who is on the hook when it breaks.

### 9. Rollout plan

Concrete milestones, not "iteratively." Shadow start date, canary %, ramp schedule, 100% target date. Named owner per milestone.

### 10. Risks and open questions

Every material risk, with mitigations or an "accept and monitor" statement. Every open question with a named owner and a date by which it will close.

Under-writing risks is the surest way to be sent back for another round. Reviewers know the risks are there; the only question is whether you know they are there.

### 11. Timeline and staffing

Not a Gantt chart — a paragraph. Who is on the team? Which peer platform team is on the delegation contract? What is the ETA to shadow, canary, and 100%?

### 12. Appendices

- Reference to the model card / data card once produced (mod-305, mod-309).
- Reference to the reproducibility plan.
- Reference to the review-packet template (mod-309).

## What the reviewer is actually reading for

Understanding the reviewer's mental model makes the RFC dramatically easier to write. A good L30/L40 reviewer is checking, in order:

1. **Is the problem worth solving, in this shape, now?** They read sections 2 and 3.
2. **Is the architecture defensible against the constraints?** They read section 4 with a copy of section 2 next to it. Any decision not pinned to a constraint gets a comment.
3. **What did the author reject, and why?** Section 5. If it is missing or weak, the review stalls.
4. **What breaks, and how do you know?** Sections 6, 7, 8, 10. The reviewer imagines being paged in six months and asks whether the RFC gives them enough context to survive.
5. **How does this fail cleanly?** Section 9 (rollback, kill switch, abort conditions).
6. **What does the team not sign up for?** Section 3 (non-goals) again — reviewers re-read this at the end.

If any of the six questions cannot be answered by the RFC in front of them, the reviewer asks in the review — and asking in the review is expensive. Writing so the reviewer does not have to ask is the L30 craft.

## Defending the RFC in an architecture review

The review meeting is *not* the moment to explain the RFC. If the reviewers cannot follow the RFC without you narrating it, the RFC is not done. The meeting is the moment to answer objections.

Three defence patterns worth practising:

### 1. "Why not X?"

The reviewer asks about an alternative you did not include in section 5. Two right answers:

- **You considered it and it lost.** "We considered it. Rejected because of the p99 latency budget in section 4 and the point-in-time constraint in section 6. I'll add it to section 5 with those reasons in the next revision." Then you actually add it.
- **You didn't consider it.** "I didn't consider that. Give me a day — I'll add it to section 5 with a full analysis or a reasoned rejection." Then you do.

The wrong answer is defending a decision under pressure. If the reviewer's question surfaces a real gap, take the gap.

### 2. "This is over-engineered / under-engineered."

The reviewer's frame ("over" or "under") is a hypothesis. Your job is to check it against the constraint in section 2, not to defend the choice on its own merits. If the constraint supports the architecture, walk the reviewer through the pinning; if it does not, revise.

### 3. "What about failure mode Y?"

Reviewers who have operated systems for years reach for failure modes juniors do not. Two right answers:

- **You handled it.** Point to the relevant subsection (parity check, monitoring, abort condition).
- **You did not handle it.** Say so. Add it to risks with a mitigation or an accepted-risk statement. This is not a loss for you — it is what the review is for.

An L30 sensitive to reviews is not chasing "no changes required." They are chasing "no *undiscussed* risks remain."

### The one thing not to do

Do not treat objections as personal. The RFC is a shared artefact; the reviewer's job is to raise objections; the review is the cheapest place they can happen. Every objection heard now is a page not sent to on-call at 3 a.m.

## Non-goals and superseding — the two things juniors miss

Two sections are worth calling out one more time because they are the most consistently missed by newly-L30 authors and are the most consistently load-bearing later.

### Non-goals bound the system for the team, for months

A non-goal in the RFC is what you point to when the product team asks for scope-creep two months in. "That's a non-goal in RFC-042; if we want to change it, we file RFC-042.1." Without written non-goals, every request is a fresh negotiation and the team's roadmap fills with unplanned work.

### Superseding preserves institutional memory

RFCs that go stale in place become "the old doc no one trusts." RFCs that are *explicitly superseded* by a new RFC preserve the decision history — you can walk from the current design back to the original constraints, one supersession at a time. Google's design-doc practice ([Ubl, 2020](https://www.industrialempathy.com/posts/design-docs-at-google/)) treats this as a first-class concept.

## A short RFC template you can steal

Feel free to reuse the skeleton below as the starting point for the RFC exercise in the paired project (`project-301-ml-system-design-portfolio`). Sections are mandatory even if some are single-line:

```markdown
# RFC-<NNN>: <System Name> — <Version>

**Status:** Draft
**Authors:** <you>
**Reviewers:** <peer L30/L40, peer platform TL, review counterparts>
**Approvers:** <ML team TL, peer platform TL>
**Date:** <YYYY-MM-DD>

## 1. Context
## 2. Problem statement
## 3. Non-goals
## 4. Proposed architecture
### 4.1 Serving posture
### 4.2 Feature substrate
### 4.3 Model artefact contract
### 4.4 Change management
## 5. Alternatives considered
### 5.1 <Alternative A> — rejected because …
### 5.2 <Alternative B> — rejected because …
## 6. Data and feature contracts
## 7. Evaluation plan
## 8. Reliability and operations
## 9. Rollout plan
## 10. Risks and open questions
## 11. Timeline and staffing
## 12. Appendices
```

The template's job is not to be a genius — it is to make sure no load-bearing section is missing. Genius comes from the specific answers, not the outline.

## Summary

An ML system RFC is the artefact where the four load-bearing decisions from chapter 01 — serving posture, feature substrate, model artefact contract, change management — are written down, defended against alternatives, bound by non-goals, and made operable by an evaluation plan, reliability plan, and rollout plan. At L30 you do not just author it — you defend it in review, hear "no" without churn, and treat every objection as a page not sent. The exercises in this module produce the components of the RFC; the paired project (`project-301-ml-system-design-portfolio`) is where you assemble a full one for a real business problem and defend it.
