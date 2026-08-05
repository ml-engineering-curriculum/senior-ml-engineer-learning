# exercise-03: ML incident response drill — runbook, rollback, postmortem

**Estimated effort:** 3 hours

## Objective

Run a **scripted ML incident drill** against a chosen scenario, executing the runbook, hitting the rollback, and writing a full blameless postmortem in the chapter 04 template. The output is *three* artefacts — the runbook you wrote (or extended), the timeline of the drill as it happened, and the postmortem with action items.

This is the L30 tell that separates "we know incident response in theory" from "we have practised it, our runbooks work, our rollback machinery has been exercised, and our postmortems produce actionable follow-ups." An L20 can respond to an incident; an L30 has built the system that responds *reliably* and learns from every incident.

The exercise can be run as a tabletop (walk the scenario on paper), as a real drill against a staging environment if you have one, or as a mixed-mode (mechanical rollback in staging, everything else on paper). All three modes produce the same three artefacts.

## Prerequisites

- Read chapter 04 (`04-ml-incident-response-and-postmortems.md`) — the ML incident taxonomy, runbooks as decision trees, the four rollback surfaces, the postmortem template.
- Skim chapter 02 (`02-ml-specific-slis-and-slos.md`) — the SLIs that fire during the scenarios below.
- Skim chapter 05 (`05-retraining-as-a-first-class-deploy.md`) if you pick the promotion-rollback scenario.
- Read Google's [*Managing Incidents*](https://sre.google/sre-book/managing-incidents/) chapter and [*Postmortem Culture*](https://sre.google/sre-book/postmortem-culture/) chapter, and John Allspaw's [*The Infinite Hows*](https://queue.acm.org/detail.cfm?id=2353017) — the classical blameless postmortem canon.
- Bring the feature and SLO document context from exercise-01; the drill will name specific SLIs against your document.

## Pick your scenario

Pick **one** (or run two for the stretch goal):

- **S1 — Feature-pipeline lateness (Category B).** A batch pipeline that produces the `user_recent_clicks_30m` feature-view runs 6 hours late because an upstream Airflow DAG got stuck waiting for a schema-changed table. The freshness SLI is burning; the quality-drift SLI has started ticking; some fraction of predictions are using stale feature values.
- **S2 — Silent calibration drift (Category D).** A customer-support team reports "our fraud model is letting through obvious cases this week." No SLI has fired. Manual investigation shows a subtle upstream-feature calibration change that is pushing scores below the manual-review threshold. Diagnose, roll back, and figure out why the SLI missed it.
- **S3 — Cost runaway (Category E).** A prompt-template deploy last night included a stray "explain your reasoning in detail" instruction. Overnight, the LLM-augmented feature has spent 8× its daily budget by 6 a.m. The circuit breaker did not trip because the runaway detector threshold is set at 10×. Contain the incident, roll back the prompt, and postmortem the enforcement gap.
- **S4 — Bad promotion (Category C).** A retrained fraud model was promoted last Tuesday. By Friday, the quality-drift SLI has burned 60 % of its 28-day error budget. Diagnose whether it is the model, the feature-views, or a downstream schema change; roll back to the previous champion; postmortem the offline / online gap.
- **S5 — Bring-your-own.** A scenario from a real incident (yours or a documented public one). Anonymise; include a preamble covering the initial symptom, the time to detect, and the eventual root cause. Use the exercise to see whether *your* runbook + rollback machinery would have caught it faster.

## Steps

### 1. Author (or extend) the runbook for this scenario (≈ 45 min)

Chapter 04 §3 defines the runbook shape — a decision tree with dashboard links and action leaves. For your scenario:

- **Which SLI is the entry signal?** Point at your exercise-01 SLO document. If the entry signal is *not* an SLI (S2), that is important — chapter 04 §1 Category D — and the runbook has to explicitly cover the "customer complaint arrives" entry path.
- **Draft the decision tree.** Chapter 04 §3 shows the anatomy. Aim for 5–8 nodes; leaves are actions or escalations, not observations.
- **Link the dashboards.** Specific PromQL / SQL queries or Grafana panel URLs (real or placeholder). Not "check the freshness dashboard" — the specific query.
- **Name the SME the runbook escalates to.** The producer-team on-call, the model owner, the feature-store owner.

### 2. Set the drill up (≈ 20 min)

Whether tabletop or real:

- Pick a *start time* for the drill. Write down the initial state — no SLI has fired, or one has just fired.
- Assign roles: Incident Commander, Operations, Communications (chapter 04 §2). If you are drilling solo, play IC and rotate roles yourself, or invite a peer.
- Define an *injection*: for tabletop, the fact pattern that triggers the incident (in S1: "the freshness SLI just went red on `user_recent_clicks_30m` with 6 h staleness"). For a real drill, the mechanism (in S1: stop the DAG in a staging environment; watch the SLI actually fire).
- Set an incident-response *timebox*: 30–60 minutes. The IC's decision cadence (every 15 minutes) is exercised inside it.

### 3. Execute the drill and record the timeline (≈ 45 min)

Walk the scenario, following the runbook. Record a *literal* timeline — the same shape as the postmortem template (chapter 04 §6):

- HH:MM — First signal (SLI alert / customer report / anomaly). What was the exact signal?
- HH:MM — Incident declared. IC assigned.
- HH:MM — Runbook node 1 executed. Result: `_____`. Next node: `_____`.
- HH:MM — Runbook node 2 executed. Result: `_____`. Next node: `_____`.
- HH:MM — Root cause hypothesised. Confirmation attempted: `_____`.
- HH:MM — Mitigation action decided. Rollback surface: `_____` (one of: model, feature-view, prompt, schema). Chapter 04 §4.
- HH:MM — Rollback executed. Time to complete: `_____`.
- HH:MM — SLI recovers to baseline (or, in tabletop, "would recover").
- HH:MM — Incident closed.

If the runbook does *not* have a node for what you actually need to check, that is important — write it into the timeline as "improvised" and open a follow-up action to add the node. This is exactly the shape gamedays are designed to surface (chapter 04 §5).

### 4. Note what worked and what did not (≈ 15 min)

Distinct from the postmortem itself, before you compose the final artefact, list:

- **What went well.** SLI fired quickly. Rollback took less than 60 seconds. Runbook was accurate. Comms cadence was maintained.
- **What went badly.** Runbook missed a node. Rollback machinery required a manual step. Dashboards were slow. Ambiguity about who paged whom.
- **Where you got lucky.** Chapter 04 §6's "where we got lucky" section. Was there something that *would have gone wrong if the specifics were different*?

### 5. Write the postmortem (≈ 30 min)

Using the chapter 04 §6 template, produce the postmortem. Cover:

- Summary, impact, timeline, root cause (proximate + underlying), what went well, what went badly, where we got lucky, action items.
- **Action items in all four categories** — prevent, detect, mitigate, practise. At least one in each.
- Include the ML-specific fields — category label (§1), SLI coverage question, rollback-rehearsed question, training-serving skew question (for S4), label-availability question.
- Cross-references — link to the runbook you wrote (§1), the SLO document you would update (exercise-01), the cost-budget document you would update (exercise-02) if relevant.

Publish. Yes, even if it is a drill. The postmortem is the artefact that matters; the drill is the mechanism that produces it.

## Deliverable

Three artefacts, delivered together:

- **`runbook-<sli-or-category>.md`** — the runbook decision tree you wrote or extended. 1–2 pages.
- **`drill-timeline-<scenario>.md`** — the drill's literal timeline as it happened, with role assignments and per-node results. 1 page.
- **`postmortem-<scenario>-drill.md`** — the full postmortem in the chapter 04 §6 template, 2–3 pages, with action items in all four categories.

Deliver as a small folder or a single Markdown with clear headers.

## Acceptance criteria

- [ ] The runbook is a *decision tree* with numbered nodes and dashboard links, not a wall of prose.
- [ ] Every runbook node ends in either an action or an escalation, not an observation.
- [ ] Every runbook node points at a specific dashboard, query, or SLI — not "check the dashboards."
- [ ] The timeline records real (or simulated-real) minutes-and-seconds, not "next we did X."
- [ ] The timeline explicitly notes any node the runbook was *missing* (an improvised diagnostic step), and that gap appears as an action item in the postmortem.
- [ ] The postmortem's root-cause section names *both* a proximate cause and an underlying condition.
- [ ] Action items exist in all four categories — prevent, detect, mitigate, practise — with a specific owner and due date each.
- [ ] The postmortem cross-references at least the SLO document (exercise-01) and either the cost-budget document (exercise-02) or the retraining-plan document (exercise-04).
- [ ] The ML-specific fields (category, SLI coverage, rollback rehearsal, training-serving skew, label availability) are all filled in — even if the answer is "N/A because ...".
- [ ] The postmortem is blameless — individual actions are contextualised, the question is "what system property allowed this," not "who did this."

## Stretch goals

- **Run two scenarios back-to-back.** Compare where the runbook and rollback machinery held up in both vs. only one. Patterns across scenarios are the *program-level* signal for the reliability roadmap.
- **Time the mean-time-to-mitigation.** Distinct from the whole incident duration. For a real drill in staging: the wall-clock time from "SLI fires" to "SLI back to baseline." Compare to your SLO's error-budget consumption rate — is the MTM small enough that a slow-burn budget will survive an incident of this length?
- **Author the runbook gap into the platform team's queue.** Chapter 04 §4's rollback machinery requires platform investment (feature-view rollback CLI, atomic flag propagation). Any gap found in the drill that the model team cannot fix alone → a specific ticket for the platform team with a specific ask. Sets up the delegation contract mod-308 covers.
- **Simulate the wrong on-call being paged.** In a real org the paged engineer often does not have the context for the specific SLI. Extend the drill: the on-call has to hand off to the SME while the incident is live. Observe how much extra time the handoff costs and whether the runbook covers it.
- **Composite scenario — S3 + S4.** A retraining promotion also introduced a prompt regression (both C and E fire). Which do you diagnose first? The senior discipline is to mitigate the fastest-rollback first (usually the prompt) while diagnosing the second, not to diagnose serially.
- **Feed the gameday calendar.** File the postmortem's "practise" action items on a real gameday calendar. A postmortem action that says "add a new node to the runbook" and no gameday exercises it will silently rot.
