# exercise-01: Paved-road consumption audit

**Estimated effort:** 3 hours

## Objective

Author the **paved-road inventory** for a specific ML team, then self-audit the team's consumption of each primitive against the four disciplines of idiomatic consumption from chapter 01 §3. Package the resulting pain points as a **quarterly consumer report** that a platform team's staff engineer / TL / PM could actually act on.

The output is two Markdown documents: `paved-road-inventory-<team>.md` and `quarterly-consumer-report-<team>-<yyyy-qN>.md`. Together they are what a senior ML engineer's team maintains as a living record of the platform relationship — the input to chapter 02's build-vs-adopt decisions and the raw material for chapter 03's RFCs.

This is the L30 tell that separates "we use the platform" from "we are a lead consumer team giving the platform team the feedback that shapes their roadmap." An L20 files bug tickets when things break; an L30 authors a quarterly report that the platform-team PM cites at planning time.

## Prerequisites

- Read chapter 01 (`01-paved-road-consumption.md`) — the paved-road concept, the four disciplines of idiomatic consumption, the consumer's contract (SLO / escalation / deprecation), the three forms of feedback, and the paved-road inventory skeleton in §7.
- Skim chapter 02 §1 — the four paths available when the paved road doesn't fit; this exercise catalogues the situations, chapter-02 exercise picks the response.
- Skim [mod-307 chapter 01 §5](../../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md) — dependency-SLI composition, so your inventory's SLO column is a real SLI + target + window, not "the platform says so."
- Have access to (or invent, if you are working from a bring-your-own team) the platform primitives your team consumes — feature store, model registry, training platform, serving platform, orchestrator, experimentation platform, eval harness, cost / observability.

## Pick your team

Pick **one** of the following, or bring your own (T5, with a one-paragraph "team scope" preamble):

- **T1 — Recommendation-serving team on a high-traffic surface.** ~10 M sessions/day. Depends on: feature store (rich, sub-hour freshness needs), model registry, serving platform (hard p99), orchestrator (nightly retraining DAG), experimentation platform (weekly ramps), eval harness (mod-305 flow), observability. Historically a lead-consumer team; platform team has heard from you before.
- **T2 — Fraud-detection team on transaction path.** ~1 M transactions/day. Depends on: feature store (moderate-freshness needs, but with adversarial input drift), model registry, serving platform (hard low-p99), orchestrator (monthly retraining), experimentation platform (used cautiously), eval harness. Growing consumer; platform team knows you but not intimately.
- **T3 — LLM-augmented ticket-triage team.** ~5 K tickets/day. Depends on: feature store (limited use — most inputs are the ticket text), model registry (LLM prompt versions and safety classifiers), serving platform (LLM vendor call, latency dominated by vendor), orchestrator (event-driven retraining), eval harness (LLM-judge harness — a stretch on the platform's existing surface). Newer consumer; the LLM adjacency stretches the platform's existing coverage.
- **T4 — Ad-conversion team on the auction path.** ~50 M requests/day. Depends on: feature store (hard sub-15-min freshness), model registry (daily retrain cadence), serving platform (very hard p99 in the tens-of-ms), orchestrator (daily), experimentation platform (heavy — dozens of concurrent experiments), eval harness. Historically a heavy consumer with dedicated platform-team relationships.
- **T5 — Bring-your-own.** A real or planned team you have access to. Anonymise anything sensitive; add a preamble covering scale, model surfaces, platform-primitive dependencies, current platform relationship posture.

## Steps

### 1. Enumerate the primitives (≈ 20 min)

For your team, list every platform primitive it consumes. Use the taxonomy from chapter 01 §1 as the checklist:

- Feature store (offline + online)
- Model registry
- Training platform / cluster
- Serving platform
- Workflow orchestrator
- Experimentation platform
- Eval harness
- Cost / observability

Some teams do not use every primitive; some use additional primitives (a vector store, a labelling platform, a promotion / release-management tool). Adapt the taxonomy to your team's actual surface, and note any *absence* as a possible feedback item — "we don't use the org's experimentation platform because we did a build-in-house three years ago" is a real observation with real implications.

### 2. Author the inventory table (≈ 40 min)

For each primitive, fill in the columns from chapter 01 §7:

- **Primitive.** Name and short version identifier.
- **Version.** The specific SDK / platform version your team is pinned to. If you are on `latest`, note that — it's a red flag.
- **Owner.** The specific team (not "the platform"). If you don't know, that's a *feedback item* — an unattributed primitive is a support risk.
- **SLO.** The specific SLI + target + window the platform team publishes. If unpublished, mark ✗ and note; this is a §5.2 line item.
- **Escalation.** The specific Slack channel / ticket queue / pager. If unknown, mark ✗.
- **Deprecation.** The specific notice period the platform commits to. If unpublished, mark ✗.
- **Idiomatic?** Your self-audit against the four disciplines of chapter 01 §3. ✓ / Partially / ✗, plus a short note.
- **Notes.** Anything else worth recording — known gaps, in-flight RFCs, prior migrations.

The table should cover at least 6 primitives and at most ~12 (larger teams may have more). Aim for one paragraph or less per primitive of narrative around the row where the row's ✓ / ✗ needs context.

### 3. Idiomatic-consumption self-audit (≈ 40 min)

For each primitive marked "Partially" or "✗" in the *Idiomatic?* column, write a short (2–5 sentence) paragraph naming *which* of the four disciplines from chapter 01 §3 is violated, and why:

- **Declared API vs. implementation.** Are we reaching around the SDK?
- **Versioning.** Are we pinning at read time, promoting explicitly, or reading "latest"?
- **Standard telemetry and artefacts.** Are we emitting the platform's standard tracing / metrics? Registering the standard artefacts (SLO doc, release memo, runbook link)?
- **Land-on-top-not-underneath.** Have we built a wrapper that delegates to the primitive, or one that shadows / bypasses it?

At least one primitive must be self-audited to "Partially" or "✗" for the exercise to have teeth. If your team is genuinely idiomatic across the board, describe the *closest* discipline to slipping — the primitive that would slip first under scale or turnover — and write the self-audit paragraph for that one.

Name at least one *concrete cost* the non-idiomatic consumption imposes — the migration you know you'll owe next time the SDK bumps, the on-call load, the reproducibility gap.

### 4. The three-form feedback triage (≈ 20 min)

Every ✗ or "Partially" cell in your inventory is a *feedback item*. Triage each into one of the three feedback forms from chapter 01 §5:

- **Bug report / ticket** — small blast radius, concrete reproduction, immediate fix.
- **Quarterly consumer report line-item** — recurring pain, aggregate cost, no obvious immediate fix.
- **RFC candidate** — structural gap in the platform, worth a specific proposed change.

You are not writing the tickets or the RFCs yet (RFC is exercise 03). You are naming which of the three forms each item is *destined* for and why.

### 5. The quarterly consumer report (≈ 40 min)

Author a separate document, `quarterly-consumer-report-<team>-<yyyy-qN>.md`. Structure (chapter 01 §5.2):

```markdown
# Quarterly consumer report — <team> — <yyyy-qN>

## Reporting party
- Team: <team>
- Author: <you, role>
- Reviewers on your team: <TL, manager>
- Delivery date: <date>
- Recipient: <platform-team PM / TL / staff engineer, named>

## Executive summary
<Two or three sentences. The most important item this quarter, the aggregate engineer-hours-of-workaround cost, and the one thing you're asking the platform team to consider.>

## Load-bearing pain points (2–5)
For each:
- **Name.** Short, in the platform team's vocabulary.
- **Symptom.** What we observe.
- **Cost.** Engineer-hours per quarter, dollars, blocked features, on-call load. A concrete number, not "significant."
- **Proposed direction.** One paragraph. Not an RFC — a directional sketch.
- **Priority (ours).** Our team's ranking of this item, 1–5.
- **Related tickets.** Links to prior filings.

## Successes and continuities
<Two or three sentences on what is working well. This exists so the report is not purely complaint-driven — the platform team needs to know what to preserve.>

## Asks
- **RFC invitations.** Which of the pain points do we think warrant an RFC? Named.
- **Deprecation feedback.** Any deprecations that we cannot meet on the platform's proposed schedule.
- **Roadmap awareness.** Anything on your published roadmap that we want to signal we're depending on.
```

The report is 2–4 pages. Every pain point has a *number* attached to its cost — an engineer-week estimate, a dollar figure, a P95 tail latency measurement. If you can't estimate the cost, note the estimation method that would produce one and flag as an open item.

### 6. Cross-check against the SLO document (≈ 20 min)

Open (or draft) the mod-307 chapter 01 §6 SLO document for your team's flagship feature. For each dependency SLI it names, confirm the inventory has a matching row with a matching SLO. Any mismatch is either an out-of-date SLO doc, an out-of-date inventory, or a missing platform commitment. Note the mismatches — they are feedback items on top of the ones from §3.

### 7. Peer review (≈ 20 min)

If this exercise is being done in a group, trade the inventory and the quarterly report with a peer. The reviewer walks:

- Is every ✗ cell a real feedback item, or is the team just underinformed?
- Is every self-audit paragraph in §3 concrete about *which* discipline is violated, not "the code is bad"?
- Is every quarterly-report cost estimate defensible against a "how did you get that number" challenge?
- Does the "successes and continuities" section exist and read as honest?

The peer's marginal notes are added to both documents as "open items — 2026-Qn peer review."

## Deliverable

Two Markdown documents:

- `paved-road-inventory-<team>.md` (2–4 pages). Sections: preamble (team scope, date, reviewers), inventory table (§2), idiomatic-consumption self-audit paragraphs (§3), feedback triage (§4), SLO-doc cross-check summary (§6).
- `quarterly-consumer-report-<team>-<yyyy-qN>.md` (2–4 pages). Structure per §5.

Both are written as *living documents* — with a review cadence, an owner, and a next-review date. They should be usable by an incoming team member on day one to understand the team's platform relationship.

## Acceptance criteria

- [ ] Inventory covers at least 6 platform primitives, with all 8 columns (§2) filled or explicitly noted as missing.
- [ ] Every SLO, escalation, and deprecation cell is either the specific published value, or ✗ + a feedback item.
- [ ] At least one primitive is self-audited to "Partially" or "✗" against the four disciplines, with a concrete cost named.
- [ ] Every ✗ or Partially item is triaged into one of the three feedback forms (§4).
- [ ] The quarterly consumer report names 2–5 load-bearing pain points, each with a *quantified* cost.
- [ ] Every pain point has a proposed direction — one paragraph, directional.
- [ ] The report includes a successes-and-continuities section (§5, so the report is not purely complaint-driven).
- [ ] The RFC-invitation ask is explicit — at least one pain point is nominated for RFC treatment (feeds exercise 03).
- [ ] The SLO-doc cross-check identified any mismatches (or noted "no mismatches — SLO doc reviewed <date>").
- [ ] Both documents name an owner, reviewers, and a next-review date.

## Stretch goals

- **Historical pain-point tracking.** For each pain point in this quarter's report, note whether it appeared in prior quarters' reports (or whether you are inferring from tickets). Recurring pain points warrant heavier escalation — either the RFC has stalled, or the platform team's response is not landing.
- **The composition-SLI table.** For the top three dependency chains (your feature → primitive A → primitive B), compute the composed SLO the [mod-307 chapter 01 §5 way](../../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md). If the composition undercuts your feature's SLO, that is a first-class feedback item — the platform's SLOs are structurally under-provisioning your feature.
- **Sponsor-conversation pre-work.** For the pain point you'd most like to RFC (feeds exercise 03), draft the two-paragraph sponsor-conversation pitch from chapter 03 §2. Not the full RFC — the pitch a sponsor can react to.
- **Cross-team check.** Compare inventories with a peer team's audit. Which pain points appear in both? Cross-team-visible pain points are the strongest candidates for RFC treatment because the platform-team's payoff on the fix is fleet-scale.
- **Peer-track hand-off preview.** For any pain point whose fix would require depth from `training-pipeline-engineer` or `model-evaluation-engineer`, note it in an appendix; that appendix feeds chapter 04's hand-off contracts if / when the specialist engagement is scoped.
- **Feed the paired project.** Attach the inventory as the "platform dependencies" appendix of a paired project (e.g., a mod-302 architecture doc or a mod-307 SLO doc). The inventory should be *cited*, not archived.
