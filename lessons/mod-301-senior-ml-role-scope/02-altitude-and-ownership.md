# Altitude and ownership: L20 vs. L30 vs. L40

## Motivation

Chapter 01 named the shift in prose. This chapter turns it into a **side-by-side ownership table** you can point at during career conversations, calibrations, and scope arguments. It is deliberately opinionated. Real companies vary — some call an L30 "senior", some "IC3", some "L4"; some call an L40 "staff", some "principal", some "senior 2" — but the *shape* of the ladder in the tables below is stable across the public engineering-career frameworks catalogued at [progression.fyi](https://www.progression.fyi/) and [engineeringladders.com](https://www.engineeringladders.com/), and across the seniority differentiation the *Google Engineering Practices* documentation implicitly assumes for design and review authors.

If your company's rubric differs, treat this chapter as a reference framework and translate.

## Core concept — the five ownership axes

Every published engineering ladder — Monzo, Meta, GitLab, Buffer, Basecamp, Google, and the composite at engineeringladders.com — walks the same five axes:

- **Development** — the code and models you personally produce.
- **Systems** — the architectural scope you are responsible for.
- **People** — mentorship, review, and hiring.
- **Process** — the practices your team follows (RFCs, on-call, retros, evaluation gates).
- **Influence** — how far your written and spoken opinions travel.

The L20 → L30 → L40 shift is a shift in **which axes you are graded on**, not just how well you perform on each.

## The ownership matrix

The table below is the working reference for the rest of this track. Rows are the five axes; columns are the three levels around this track.

| Axis | L20 — ML Engineer (build altitude) | L30 — Senior ML Engineer (this track) | L40 — Staff ML Engineer |
|---|---|---|---|
| **Development** | Builds features and models under a scoped task. Ships PRs at team standard. Writes tests. | Owns the reference implementation of the team's ML system. Sets the bar the team's PRs are reviewed against. Personally writes the load-bearing components (retraining glue, eval harness, canary logic). | Reviews rather than authors most code. Writes the seed for a shared platform component when a team-scope tool needs to become a multi-team one. |
| **Systems** | Owns *a model* end-to-end. Understands the surrounding system enough to be a good citizen. | Owns *a production ML system* end-to-end: architecture, retraining cadence, eval, SLOs, cost budget, incident response. Authors the RFC and defends it. | Owns *a portfolio across teams*: shared platforms, cross-team architecture, org-wide ML system quality bar. |
| **People** | Learns from a senior. Reviews peers on request. May onboard an intern. | Mentors mid-level engineers on the team as a primary responsibility. Runs interviews and gives calibrated hire/no-hire signals. Sets the review bar. | Grows senior engineers into staff. Runs interview loops. Owns a hiring rubric across teams. |
| **Process** | Follows the team's process. Uses the eval harness, the retraining cadence, the on-call rota. | Owns the team's ML process: evaluation gates (mod-305), retraining SLA (mod-307), review-packet template (mod-309), roadmap cadence (mod-310). Fixes broken process by authoring a new one. | Owns cross-team process: shared eval standards, incident postmortem norms, ML review governance. Represents the org externally. |
| **Influence** | Influences their own PRs. May speak up in team design reviews. | Influences the team's roadmap. Their RFCs are read by adjacent teams. They are the person the eng manager escalates ML scope calls to. | Influences the org's direction. Their strategy docs are read by directors and VPs. They set multi-quarter technical direction. |

Two calibration notes on the table:

1. **The L20 → L30 line is more visible than the L30 → L40 line.** At L20 vs. L30 the axis that changes hardest is *Systems* (from model-scope to system-scope) and *Process* (from consumer to owner). At L30 vs. L40 the axis that changes hardest is *Systems* (from team-scope to portfolio) and *Influence* (from team roadmap to org direction). If you are honestly at L30 on Systems + Process + People but only L20 on Influence, you are a **strong senior**, not a proto-staff.
2. **You do not have to peak on every axis.** Real senior engineers are lopsided. Some are heavier on Systems + Development (the "principal-eng archetype at senior scope"), some heavier on People + Process (the "tech-lead archetype"). The self-assessment in chapter 05 and exercise-03 uses these axes so you can see your own shape.

## Concrete example — the same ownership boundary, three levels

**Scenario:** the team owns a recommendations model. A production regression is discovered on Monday morning.

- **L20** gets pulled into the incident, does the root-cause work, ships the fix, and writes the retro. The eng manager and the senior guide the framing.
- **L30** is the incident commander for the ML side. They decide whether this violates the SLO, whether to roll back, who to page, what to communicate to the product partners. They author the retro's structural fixes (a missing slice in the eval harness, a missing SLO on freshness, a broken hand-off with the feature-store team). They assign one of those fixes to an L20 and review the PR.
- **L40** is not in the incident channel by default. They are pulled in if the retro's structural fix requires a change to the shared eval platform or if this regression pattern is showing up across multiple teams. Their output is a cross-team RFC, not this specific fix.

## Concrete example — where the ownership shift shows up in the artefacts

You can predict a lot about someone's real level by asking, "show me an artefact you're proud of from the last quarter." What they show you tells you their altitude.

- **L20 artefact:** a well-written model card for a shipped model, or a clean PR that fixes a subtle training bug.
- **L30 artefact:** an ML system RFC that got merged, plus the review packet for the deploy it authorised, plus a retro they wrote for a regression it caught.
- **L40 artefact:** a multi-quarter roadmap that got funded, plus the platform-team RFC for the shared component it commissioned, plus a hiring rubric they revised.

If you are aiming at L30 and your best artefact from last quarter is on the L20 list, mod-302 and mod-310 are load-bearing for you. Chapter 05 turns this into a self-assessment.

## The hand-off boundary is part of ownership

A hand-off is not a *loss* of ownership. It is a *shape* of ownership. At L30 you own:

- Everything **inside** your production ML system.
- The **contract** with everything outside it — what you consume from the peer platform tracks, what you delegate to the peer specialist tracks, and what you commission from the ML platform team when the paved road does not cover you.

That contract is a first-class L30 artefact. Chapter 04 formalises it.

## Summary

The senior ML engineer's ownership is best described on five axes (Development, Systems, People, Process, Influence). L30 moves you from *model-scope + team-consumer* to *system-scope + team-owner*. L40 moves you from *team-scope* to *portfolio-scope*. Not every L30 is strong on every axis — real seniors are lopsided, and the self-assessment in chapter 05 respects that. The hand-off contracts to peer tracks (chapter 04) are part of the ownership, not the edge of it.
