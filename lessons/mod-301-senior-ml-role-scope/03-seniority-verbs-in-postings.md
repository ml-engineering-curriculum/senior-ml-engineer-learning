# Seniority verbs in postings

## Motivation

Every senior ML engineer will read hundreds of job postings across their career — for their own moves, for calibration, and for hiring rubrics they help author (mod-310). Most of them look interchangeable at a glance: buzzwords, the same tech stack, the same "5+ years of experience." The signal that separates a genuine senior posting from a mid-level posting with senior title-inflation is not in the skills bullets — it is in the **verbs**.

This chapter teaches you to read a posting for those verbs and to extract from it the seniority scope the hiring team actually wants. Exercise-01 walks you through the audit on five real postings.

## The core idea — verbs are the seniority signal

A level-20 ML Engineer posting overwhelmingly uses **build-altitude verbs**: *build, develop, implement, deploy, train, tune, optimise, monitor, write, contribute*. These verbs are indifferent to scope — one engineer can perform every one of them alone on a well-scoped task.

A level-30 Senior ML Engineer posting layers **lead-altitude verbs** on top: *own, lead, mentor, guide, set the roadmap for, partner with, drive, evangelise, raise the bar for, define the standard for, architect, champion, be the technical lead for*. These verbs presuppose scope beyond the individual — you cannot *set a roadmap* without a team to run it, you cannot *raise the bar* without a bar and reviewees, you cannot *partner with* without a peer team on the other side of the boundary.

A posting whose title says "Senior" but whose bullets never use a lead-altitude verb is almost always mid-level work with senior compensation gated by tenure. A posting whose bullets are dominated by lead-altitude verbs but that also asks you to "own the roadmap for a team of ML engineers" is often a *staff*-scope role mistitled as senior. Reading the verbs is how you tell.

## The six verb families

For the rest of this track, we use six verb families to classify a bullet:

| Family | Representative verbs | Altitude signalled | Reads to |
|---|---|---|---|
| **Build** | build, develop, implement, code, write, train, tune, deploy, integrate, containerise | L20 build-altitude (necessary at every level, not sufficient for senior) | You will be shipping code. |
| **Operate** | monitor, maintain, on-call, respond, debug, tune, harden, ship | L20/L30 operating altitude | You will be on the pager. |
| **Own** | own, be responsible for, be accountable for, drive to completion, be the tech lead for | L30 scope | A named system or programme is yours. |
| **Lead** | lead, guide, coordinate, orchestrate, run (a review / a programme), drive alignment | L30 people/process scope | Other engineers work on this with you and to your bar. |
| **Mentor / raise the bar** | mentor, coach, grow, teach, review at bar, raise the bar for, uplevel | L30 people scope | You will be graded on other engineers' output. |
| **Set direction** | set the roadmap for, define the strategy for, chart the direction of, evangelise, decide the standard for | L30 (team scope) → L40 (org scope) — read the qualifier | You have written or spoken decision authority for the scope named. |

A well-written senior posting will have bullets from at least four of the six families. A senior-titled posting that is only Build + Operate is almost always mid-level in disguise.

## Reading a bullet — a worked example

Take a hypothetical bullet:

> Build and deploy production ML models that power personalisation across the app.

This bullet is Build + Operate + no scope qualifier. It is *not* a seniority signal. A mid-level engineer will do this bullet exactly as it reads. Two candidates for the same posting could interpret it identically and both be right.

Now the same posting adds:

> Own the technical direction of the personalisation ranking system end-to-end, partner with the ML platform team on feature-store adoption, and raise the bar for evaluation rigour across the ranking pod.

Three seniority signals appear: *own the technical direction* (Own + Set direction, scope = "the ranking system end-to-end"), *partner with* (Lead, scope = "ML platform team"), *raise the bar for* (Mentor / raise the bar, scope = "the ranking pod"). This is a genuine L30 posting. A candidate who reads only the first bullet and interprets the role as build-altitude will underscope their interview prep and, if they land the role, will underdeliver in the first quarter.

## Scope qualifiers — the second half of every senior verb

A senior verb is only interpretable with its **scope qualifier**. "Own" is meaningless; "own a component" is L20; "own the ranking system end-to-end" is L30; "own the recommendations portfolio across teams" is L40. Every time you find a lead-altitude verb in a posting, mark the scope qualifier attached to it. The set of scope qualifiers in a posting is the second-most reliable seniority signal after the verbs themselves.

Useful qualifier classifications:

- **Component scope** ("a feature", "a service", "a pipeline stage") → L20 with senior title.
- **System scope** ("the ranker", "the fraud model system", "the retrieval stack for X", "the ML system behind Y") → L30.
- **Programme scope** ("the evaluation programme", "the incident response process", "the roadmap for the pod") → L30 (senior tech-lead) or L40 (staff), depending on the number of teams named.
- **Portfolio / cross-team scope** ("ML across the personalisation org", "the shared ML infra for N teams", "the multi-team roadmap") → L40+.

## Anti-patterns — what disguises the real level

Real postings are noisy. The following patterns hide the actual level and are worth calling out explicitly:

- **Senior-titled but component-scoped.** All the bullets read as build-altitude on a single well-scoped component. The company is using "senior" as tenure. If your goal is L30 growth, this role will not stretch you. If your goal is L30 compensation with L20 responsibility, this may be exactly what you want — be honest with yourself.
- **Staff-scoped but titled Senior.** Multiple team names in the scope qualifiers, "roadmap across the org," "set the ML strategy for the platform." This role is L40 work; a genuine L30 will get eaten by it. Either they are hiring on stretch, or the level in this company is deflated by one step.
- **"Mentor" without scope.** The bullet says "mentor junior engineers" with no ownership or roadmap context. Often means "you'll answer their questions on Slack." Not a real Mentor-family signal unless paired with an Own or Lead signal.
- **"Full-stack ML" grab-bag.** The posting asks for build + operate + evaluate + train + fine-tune + deploy + monitor + secure + govern. This is either a startup where you actually will do all of them (in which case the L30 shape is unstable but real), or a lazy JD copy-paste (in which case you can only tell in the interview loop). Add a note and interrogate in the loop.
- **"Set direction" attached to a codebase, not a decision surface.** Real Set-direction bullets attach to a *roadmap*, a *standard*, or an *architecture*. If the bullet says "set direction for our codebase," it usually means "you write most of the code" — a Build-family bullet in senior clothing.

## A rubric for auditing a posting

Given a real posting, work through this rubric in order — this is exactly the rubric exercise-01 asks you to apply.

1. **List every requirement bullet** verbatim (both required and preferred). Do not paraphrase.
2. **Classify each bullet** into one of the six verb families. A bullet may fall into two families (e.g. "own and mentor" is Own + Mentor); classify both.
3. **Extract the scope qualifier** for every Own / Lead / Mentor / Set-direction bullet. If the qualifier is missing, mark `<no-scope>`.
4. **Count** bullets per family. Note the ratio of Build/Operate bullets to Own/Lead/Mentor/Set-direction bullets. Genuine senior postings have at least four Lead-family bullets and at least half as many Lead-family bullets as Build-family ones.
5. **Classify the posting**: L20-in-disguise, L30, or L40-mistitled. Cite the bullets that drove the classification.
6. **Extract the "senior verb signature"** — a compressed sentence of the form: `Own <system scope>, lead <programme>, mentor <cohort>, partner with <peer team>, set direction for <decision surface>.` If you cannot form this sentence from the posting, the posting is not a senior posting.

## Concrete example — the compressed senior verb signature

From a well-formed senior ML engineer posting for a recommendations team, the extracted signature might read:

> Own the recommendations ranking system end-to-end, lead the evaluation programme for the ranking pod, mentor two mid-level ML engineers, partner with the ML platform team on feature-store adoption, set direction for the retraining roadmap through H2.

This one sentence tells you what you would be graded on. It is the artefact exercise-01 produces for each posting.

## Summary

Seniority in an ML engineering posting is carried by verbs and scope qualifiers, not by tech-stack bullets. Six verb families are the working taxonomy: Build, Operate, Own, Lead, Mentor / raise the bar, Set direction. A real senior posting has Lead-family verbs attached to system-scope qualifiers. A senior-titled posting whose bullets are entirely Build + Operate is mid-level in disguise; one whose scope qualifiers name multiple teams is staff-scope mistitled. The compressed senior verb signature — one sentence of Own, Lead, Mentor, Partner, Set direction — is the artefact you extract from every posting. Exercise-01 has you run this rubric on five real postings.
