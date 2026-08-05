# ML incident response and postmortems

## Motivation

Chapters 01–03 built the *upstream* discipline — SLIs, SLOs, error budgets, cost budgets. This chapter is what happens when one of them fires and a real user is experiencing the failure right now.

Classical SRE has a well-developed incident-response playbook: Incident Commander, Operations Lead, Communications Lead, structured status updates, timeboxed decision windows, a blameless postmortem, action items with owners and due dates. Google's [*Managing Incidents*](https://sre.google/sre-book/managing-incidents/) and the [PagerDuty Incident Response documentation](https://response.pagerduty.com/) are the reference implementations; Etsy's [*Debriefing Facilitation Guide*](https://extfiles.etsy.com/DebriefingFacilitationGuide.pdf) and John Allspaw's [*The Infinite Hows*](https://queue.acm.org/detail.cfm?id=2353017) are the load-bearing reads on blameless retrospectives.

The classical playbook works for ML incidents too. The gaps this chapter has to fill:

- **The ML failure taxonomy is different.** "500 error" and "high latency" cover a fraction of ML incidents. The rest — silent drift, feature-pipeline breakage, calibration collapse, prompt injection, training-serving skew — do not have Runbook entries in a classical SRE library.
- **Detection is different.** Classical incidents are self-announcing (alerts fire; users complain; PagerDuty pages). ML incidents are often *silent* until a customer complaint or a downstream signal surfaces them — the availability SLI is 100 %, the latency SLI is green, and the model has been silently miscalibrated for six days.
- **Rollback is different.** "Roll back the deploy" for a classical service is a code-version flip. For an ML system the "deploy" is a *model artefact*, a *feature-pipeline config*, a *prompt version*, and a *training dataset* — any of which could be the cause and each of which has a different rollback mechanism.
- **The postmortem shape is different.** Classical postmortems ask "what code was buggy?" ML postmortems have to ask "what code, what data, what model, what feature, what upstream schema?" — and often the answer is *all five interacted*.

This chapter authors the ML-specific extension of the incident-response playbook. Roles and cadence are unchanged from the classical playbook; runbooks, taxonomy, rollback machinery, and the postmortem template are ML-adapted.

## §1 — The ML incident taxonomy

A senior ML team keeps a small taxonomy of incident categories, each with a *typical detection signal*, a *first-move runbook*, and a *rollback mechanism*. The taxonomy is not exhaustive — every real program grows it — but it is stable enough to be a repo artefact.

### Category A — Serving path degradation (classical)

- **Detection.** Availability, latency, or error-rate SLI (chapter 01 §5) burns.
- **Common causes.** Deploy-shape regressions, downstream dependency degradation (LLM vendor, feature store, embedding service), infrastructure incidents (cluster autoscale gap, cert expiry, load-balancer misconfiguration).
- **First move.** Standard classical SRE playbook — check the deploy history, check the dependency dashboards, check the recent config changes.
- **Rollback.** Deploy revert; feature-flag flip; scale-out.

Nothing here is ML-specific. Every mature team has this playbook in place; this category is included so that the taxonomy is complete and not because the ML angle is interesting.

### Category B — Feature-pipeline breakage

- **Detection.** Prediction freshness SLI (chapter 02 §1) burns, or feature-fetch fallback-rate SLI (chapter 01 §5) burns.
- **Common causes.** Upstream schema change (a producer added a nullable column with no default, and the feature-store consumer is now emitting nulls); Kafka consumer position drift; batch pipeline hang on a specific partition; feature-store rollout that changed a computation; a bug in the feature engineering code that produces a plausible-looking but wrong value.
- **First move.** For each burnt SLI, identify the specific feature(s) affected. Compare the current value distribution to the last-known-good distribution (this is why chapter 02's PSI SLI is a *diagnostic tool*, not just a monitoring line). Check the upstream producer's SLI and its recent deploy history.
- **Rollback.** Not a code rollback — a *feature-view rollback*. Either point the serving path at the last-known-good version of the feature view (feature stores like [Feast](https://docs.feast.dev/) and [Tecton](https://docs.tecton.ai/) support this) or roll back the upstream producer.

This category is where the *runbook* gets its main workout. Feature-pipeline incidents are the most common ML incident in mature programs — often the majority.

### Category C — Model quality regression

- **Detection.** Quality-drift SLI (chapter 02 §2) burns; calibration SLI (chapter 02 §3) burns; a customer complaint routes through the ML on-call.
- **Common causes.** A retrained model was promoted with a regression the offline harness missed; the input distribution shifted enough that the current champion no longer covers it; the ground-truth pipeline is emitting biased labels (label leakage, delayed labels arriving in the wrong window); training-serving skew from a feature-engineering discrepancy (mod-302 chapter 04).
- **First move.** Check what changed *recently*: was a new model promoted? Was a new feature added? Was there an upstream schema change? Then check the distribution-drift SLI to see if inputs shifted. Then check the label-pipeline SLI. The order is *most-actionable-first*: rolling back a promotion is fast; retraining is not.
- **Rollback.** Roll back to the previous champion in the model registry (chapter 05 §3). One flag flip; the *rehearsed* rollback the retraining-as-deploy chapter builds.

The rule for this category: **the first move is a rollback if you have a reversible cause and it is safe; only then do you investigate.** Investigating a live quality regression while it degrades user experience is what turns a 30-minute incident into a 6-hour one. Google's [*Managing Incidents*](https://sre.google/sre-book/managing-incidents/) chapter formalises this as *mitigate first, root-cause second*.

### Category D — Silent drift without an SLI breach

- **Detection.** Usually a *customer complaint* or a downstream-team signal — "your recommendations look wrong today," "the fraud model is letting through obvious cases." No SLI has fired.
- **Common causes.** The drift is real but slower than the SLI's window (the calibration SLI has a 7-day rolling window and the shift is subtle enough to stay within the window's tolerance); the SLI is on the *wrong metric* for the failure (a global-average metric hiding a slice regression); the SLI has an unmet dependency (labels are late, so the quality SLI is stale).
- **First move.** Take the customer complaint as a signal to *manually* run the drift diagnostics — per-slice quality metrics, per-feature PSI, per-slice calibration — that the automated SLI aggregates. If manual diagnostics confirm the drift, the incident is upgraded and the missing SLI becomes a follow-up action item.
- **Rollback.** Same as Category C — model rollback or feature-view rollback, depending on the diagnosis.

This category deserves its own entry in the taxonomy because it is the *most* common way a mature program discovers its SLI menu is incomplete. Every incident in Category D grows the SLI menu (chapter 02) by one entry.

### Category E — Cost incident (runaway, budget-blown)

- **Detection.** Cost budget alert (chapter 03 §6); runaway detector (chapter 03 §4) trips; anomaly on the cost line item.
- **Common causes.** Prompt regression that causes N× the output tokens; a retry loop with no jitter that hammers a paid vendor; a feature-engineering bug that fans out one input into 100 downstream vendor calls; a legitimate traffic spike that the budget did not anticipate.
- **First move.** If a circuit breaker or kill switch has already tripped (chapter 03 §4), confirm the fallback path is behaving; if not, trip it manually.
- **Rollback.** Whatever caused the spend spike — a recent deploy, a config change, a prompt version. Then re-open the kill switch on the fallback path and observe.

Cost incidents *always* trigger a postmortem, even when the resolution is fast, because the shape of the failure is often a design gap in the enforcement layer (chapter 03) rather than a code bug.

### Category F — Adversarial / abuse incident

- **Detection.** A pattern of anomalous inputs that trip safety classifiers, prompt-injection attempts (mod-304 chapter 05), fraud model queries that show adversarial evasion, or a spike in a fairness-slice SLI.
- **Common causes.** A new attack pattern the model has not seen (adversarial adaptation); prompt injection through a user-controlled field; a coordinated abuse campaign; a data-poisoning attempt through a feedback loop (mod-302 chapter 05 §Part 2).
- **First move.** Restrict / block the abusing pattern (rate limit, IP block, tenant block); if that is not immediate, kill switch the affected feature; escalate to the security or trust-and-safety team.
- **Rollback.** Not usually a rollback — an *enforcement change*. A stricter rate-limit, a more conservative safety classifier threshold, an added input validator.

Category F is where mod-307 and mod-309 (Responsible AI) overlap; the security-facing depth is in mod-309, but the on-call posture is here.

## §2 — Roles and cadence — what transfers from classical SRE

The classical incident-response role structure works unchanged for ML:

- **Incident Commander (IC).** Coordinates the response, makes the mitigate-first decisions, decides when to escalate. Not a technical role — a *decision* role.
- **Operations Lead (Ops).** The engineer executing the mitigation — running the rollback, tripping the kill switch, checking the diagnostics.
- **Communications Lead (Comms).** Writes the status updates for internal and external audiences on a fixed cadence (usually every 30 minutes for a severity-1, every 60 for a severity-2).
- **Subject Matter Experts (SMEs).** ML-specific — the model owner, the feature-pipeline owner, the labelling pipeline owner. Called into the incident by the IC.

The [PagerDuty Incident Response documentation](https://response.pagerduty.com/) is the standard reference; the shape is well-covered elsewhere. The ML-specific adaptation is *which SMEs* the IC knows to call in. The taxonomy of §1 is what tells the IC "for a Category B incident, page the feature-pipeline owner; for a Category C, page the model owner."

Two shape-preservers worth naming:

- **Severity levels.** Sev-1 (user-facing, revenue-impacting, or safety-visible), Sev-2 (degraded but not blocking), Sev-3 (internal-only impact). The classical severity ladder works; the *criteria* just include ML-specific failure modes ("50 %+ of predictions taking a fallback feature value" is a Sev-1 even if the availability SLI is 100 %).
- **Timeboxes.** The IC operates on a fixed decision cadence — every 15 minutes, review status and decide next action. Prevents the "let's investigate for two hours before deciding whether to rollback" anti-pattern.

## §3 — Runbooks — the ML-specific document

A runbook is the checklist the on-call reads *while* the incident is active. Classical runbooks are checklists per alert; ML runbooks are checklists per *SLI-breach category*.

The load-bearing property of an ML runbook: **it is a decision tree, not a wall of text.** Each node is a question; each answer routes to the next node or an action.

### Anatomy of an ML runbook

```
Runbook: prediction-freshness SLI burning

1. WHICH feature-view is out of TTL?
   - Check the freshness-per-view dashboard (link)
   - Identify the specific feature-view(s) with age > TTL
   → GOTO 2 with the specific feature-view name

2. Is the upstream producer's SLI green?
   - Check producer dashboard for the specific feature-view (link)
   - If red → ACTION: escalate to producer's on-call. Continue diagnosis in parallel.
   - If green → GOTO 3

3. Is the consumer pipeline running behind?
   - Check consumer lag metric (link)
   - If lag > 5 min → ACTION: increase consumer replica count; check for recent config change.
   - If lag < 5 min → GOTO 4

4. Is the feature-store's write path healthy?
   - Check feature-store write success rate (link)
   - If unhealthy → ACTION: engage feature-store on-call.
   - If healthy → GOTO 5

5. Is a recent feature-engineering deploy the cause?
   - Check deploy history for the feature-view's compute job (link)
   - If a deploy in the last 24 h touches this view → ACTION: revert. Verify the revert restores freshness.
   - If no relevant deploy → GOTO 6

6. UNKNOWN. Escalate to model owner, ML-platform SME, and IC.
   - Snapshot the current SLI dashboard and paste the link in the incident channel.
   - Document the diagnostics run and the specific value differences observed.
```

Two properties of the runbook worth naming:

- **Every node names the dashboard or query it needs.** Not "check the freshness dashboard" — the specific link, the specific PromQL, the specific Grafana panel URL. The on-call is not being asked to compose diagnostics under pressure; they are being asked to *execute* pre-composed diagnostics.
- **Every leaf is an action, not just an observation.** "Producer's SLI is red" is not an action; "escalate to producer on-call" is. Every leaf that ends in "unknown" is a follow-up action item on the postmortem: the runbook needs a new node for this cause.

Runbooks are living documents. Every incident closes with either "the existing runbook covered it" (rare) or "the runbook needs a new node" (common). The nodes accumulate; the runbook grows; the mean-time-to-resolution shortens.

### The runbook per incident category

Each of Category A through F has its own runbook. A senior ML team's runbook library, at minimum:

- `runbooks/serving-availability.md` — Category A serving degradation.
- `runbooks/feature-freshness.md` — Category B feature-pipeline lateness (worked above).
- `runbooks/quality-drift.md` — Category C quality-drift SLI breach.
- `runbooks/calibration-drift.md` — Category C calibration drift specifically.
- `runbooks/silent-quality-report.md` — Category D customer-complaint-triggered.
- `runbooks/cost-runaway.md` — Category E cost incident.
- `runbooks/abuse-pattern.md` — Category F.
- `runbooks/retraining-pipeline-failure.md` — Chapter 05.

Every runbook is *practiced* — a periodic gameday (see §5) where the on-call walks the runbook against a simulated incident, and the runbook's gaps are surfaced without a real user experiencing them.

## §4 — Rollback machinery — the "one action" property

The senior discipline is that every rollback is *one action*. Not "run the deploy pipeline in reverse"; not "page the ML engineer"; not "read a runbook to figure out which knob to turn." One flag, one config, one CLI command.

The mechanisms for a mature ML system:

- **Model version pin.** The serving path reads a "current champion model version" flag from a config store (LaunchDarkly, Flipt, Consul KV, a database row). Rollback = update the flag. The previous champion artefact is retained in the model registry; the flag flip is the rollback.
- **Feature-view version pin.** Same shape at the feature-view level. The serving path resolves feature values through a versioned view; the version is a flag. Rollback = change the version.
- **Prompt version pin.** For LLM-augmented features (mod-304), the prompt is a versioned artefact in the same registry, and the current prompt version is a flag. Rollback = change the flag.
- **Feature kill switch.** For the LLM-augmented feature specifically, a flag that switches from "call the LLM path" to "serve the deterministic fallback." Chapter 03's kill-switch discussion.
- **Traffic-split kill.** For a canary or A/B in-progress (mod-305 chapter 03, mod-306 chapter 01), a flag that immediately routes 100 % of traffic to the control arm. The canary auto-rollback controller (mod-305 chapter 03 §2) automates this on SLI breach; the manual kill is what the on-call uses when the automation is not fast enough.

All five mechanisms live behind the *same* flag store. That is important: the on-call has one system to look at during an incident, not five. The [Google Feature Flags at Scale](https://cloud.google.com/blog/products/devops-sre/feature-flags-at-scale-google) writeup covers the operational shape; [LaunchDarkly](https://launchdarkly.com/), [Flipt](https://www.flipt.io/), [Unleash](https://www.getunleash.io/), [ConfigCat](https://configcat.com/) are common vendor implementations.

### The rehearsal property

A rollback that has never been tested is not a rollback. Every quarter — some mature programs monthly — the team runs a *rollback drill*:

- On a canary tenant (or a small staging fraction), simulate the incident that would trigger the rollback.
- The on-call executes the rollback against production tooling.
- The time-to-mitigation is measured. The gaps are turned into runbook nodes or tooling improvements.

Google's [*Disaster Recovery*](https://sre.google/workbook/disaster-recovery/) chapter of the Workbook covers the discipline; Etsy's [*How Complex Systems Fail*](https://how.complexsystems.fail/) (a summary of Richard Cook's classic paper) is the theory.

A rollback drill's typical output is *not* a passing check — it is a list of gaps. "The model-version flag update took 4 minutes to propagate to all serving replicas — investigate." "The feature-view rollback required editing a YAML by hand and re-deploying — build a CLI." Every gap becomes a maintenance ticket, and the drill catalogue is the input to the reliability roadmap.

## §5 — Gamedays and drills — practising the runbook

The runbook and rollback machinery are *dead* documents until they are practised. Two shapes of practice:

- **Gameday.** A scheduled exercise where the ML team simulates an incident against a real system (usually a staging tenant, sometimes a small production fraction with the model owner's consent). Someone plays the "attacker" — introducing the failure — and the on-call plays the response. Netflix's [*Chaos Monkey*](https://netflix.github.io/chaosmonkey/) and the broader [Chaos Engineering](https://principlesofchaos.org/) discipline are the mature version; John Allspaw and Jesse Robbins's ["Fault Injection in Production"](https://queue.acm.org/detail.cfm?id=2353017) is the classical reference.
- **Tabletop exercise.** A meeting where the ML team walks a hypothetical incident on a whiteboard. Cheaper than a gameday; useful for exercising the *coordination* between on-calls, ICs, and SMEs. Especially valuable for Category D (silent drift) and Category F (abuse), where the real thing is rare and expensive to simulate.

ML-specific gameday scenarios worth having in the rotation:

- **Kafka lag on the recent-events feature-pipeline.** Simulates Category B; exercises the freshness SLI and the feature-view rollback.
- **A promoted model with a specific slice regression.** Simulates Category C; exercises the model rollback and the slice-level diagnostic runbook.
- **Prompt injection through a user-controlled field.** Simulates Category F; exercises the input-validation and kill-switch pathways.
- **A vendor outage on the LLM provider path.** Simulates Category A + E jointly; exercises the fallback path and the cost-shifting behaviour.
- **A retrained model that "passes offline but tanks online."** Simulates Category C's most common shape — the offline/online gap — and exercises the canary rollback (chapter 05).

The gameday is *not* about proving the system works. It is about *finding the gaps* that only become visible when you exercise the system. Every gameday produces a punch list. That punch list is the reliability roadmap.

## §6 — The ML postmortem template

The postmortem is where the incident becomes learning. The classical shape from Google's [postmortem chapter](https://sre.google/sre-book/postmortem-culture/) and John Allspaw's [*The infinite hows*](https://queue.acm.org/detail.cfm?id=2353017) transfers directly; the ML-specific fields are additions.

The load-bearing property is that the postmortem is *blameless*: the goal is understanding, not blame. Individual actions during the incident are contextualised; the question is not "who did this" but "what system property allowed this."

### The template

```markdown
# Postmortem: <incident title>

- **Date of incident:** YYYY-MM-DD
- **Incident duration:** HH:MM — HH:MM UTC (X hours Y minutes)
- **Severity:** Sev-1 / Sev-2 / Sev-3
- **Author:** <name>
- **Reviewers:** <names, including one non-team engineer for external perspective>

## Summary
One paragraph. What happened, what the user impact was, how it was mitigated. Written for
someone who was not on the incident and wants the story in 30 seconds.

## Impact
- Users affected: N tenants; K% of MAU; specific slices most affected.
- Business impact: dollar amount if quantifiable, else qualitative.
- SLIs breached: list, with the amount of error budget consumed.
- Cost impact: dollar amount above the expected spend for the period.

## Timeline
- HH:MM — First signal (SLI alert / customer report / anomaly).
- HH:MM — Incident declared. IC assigned.
- HH:MM — Category identified (A / B / C / D / E / F).
- HH:MM — Mitigation action (rollback / kill switch / other). Details.
- HH:MM — SLI recovers to baseline.
- HH:MM — Incident closed.

Every entry is factual and links to the underlying artefact (Slack message, PR, config
change, alert). The timeline is a *record*, not an interpretation.

## Root cause
The chain of events that produced the incident. For ML incidents, the root cause is
almost always at least *two* things — a proximate cause and an underlying condition
that let the proximate cause reach a user. Both are documented.

- **Proximate cause.** What broke.
- **Underlying condition.** Why the system was fragile to this proximate cause.

## What went well
Even in a bad incident, some things go well. Naming them is important:
- The SLI fired quickly.
- The rollback took less than 60 seconds.
- The runbook's node 3 was accurate.

## What went badly
- No SLI covered this failure mode; detection was via customer complaint at HH:MM.
- The runbook did not have a node for this cause; the on-call improvised.
- The model rollback was rehearsed but the feature-view rollback was not; the on-call
  had to edit YAML by hand.

## Where we got lucky
Named separately from "what went well." Luck is a signal that the system was fragile
in a way that could bite next time:
- The label-pipeline SLI would have caught this in 6 hours; the customer caught it in 2.
  Next time we might not be so lucky.
- The affected slice was 4% of traffic; if this had hit the primary slice, the impact
  would have been 20× larger.

## Action items
Each action item has an owner, a due date, and a category:

| # | Description | Owner | Due | Category |
|---|-------------|-------|-----|----------|
| 1 | Add per-slice quality SLI on business-critical slice | @jane | 2026-08-30 | prevent |
| 2 | Feature-view rollback CLI (currently manual YAML edit) | @rahul | 2026-09-15 | mitigate |
| 3 | Runbook node for "upstream schema change with null tolerance" | @arya | 2026-08-19 | detect |
| 4 | Gameday: rehearse feature-view rollback | @jane | 2026-09-30 | practise |

The four categories are load-bearing:
- **prevent** — makes the failure impossible or much less likely.
- **detect** — makes the next failure of this shape surface faster.
- **mitigate** — makes the next failure of this shape recover faster.
- **practise** — makes the human response better under pressure.

A postmortem with no "prevent" action item is a postmortem that has accepted the failure
will happen again. Sometimes that is the right decision; when it is, it should be
explicit, not implicit.

## Cross-references
- Runbook updated: <link>
- SLI added / modified: <link to SLO doc PR>
- Relevant prior incident (if any): <link>
- Relevant chapter of mod-307 for the vocabulary used in this postmortem: <link>
```

### The ML-specific fields

- **The category label from §1.** Every postmortem lists the category. The categories form a distribution over time — a team where 80 % of incidents are Category B has a feature-pipeline reliability problem, not a model reliability problem, and the reliability roadmap should reflect that.
- **The "SLI coverage" question.** Did an SLI fire? Was the SLI the primary detection signal, or was it a customer report? For every incident where an SLI *should have* caught it but didn't, the postmortem includes an action item to author or fix the SLI.
- **The "rollback rehearsed" question.** Was the specific rollback mechanism used in this incident practised in a prior gameday? If not, that is an action item.
- **The "training-serving skew" question.** For Category C incidents, was the incident downstream of a mismatch between how a feature was computed in training vs. in serving? Mod-302 chapter 04 covers the design; this question surfaces the ones that slipped through.
- **The "labels available" question.** For quality regressions, were labels arriving in the window they were expected to? A label-pipeline lateness incident often causes what looks like a quality regression because the SLI is comparing today's predictions against last week's labels.

### Publishing and reviewing

The postmortem is:

- **Published within a week.** A postmortem that takes a month to write is a postmortem that has lost the details. Google's [postmortem chapter](https://sre.google/sre-book/postmortem-culture/) recommends a draft within 24 hours; a week for the reviewed final is the practical cadence.
- **Reviewed in a scheduled forum.** Every mature team has a monthly incident-review meeting. Postmortems are read; action items are tracked; patterns across incidents are named.
- **Cross-linked in the runbook.** When a runbook grows a new node because of an incident, the node links back to the postmortem so future on-calls understand *why* the step exists.

The postmortem is the *primary* learning artefact of the reliability program. A team that has an incident and does not write a postmortem is a team that will have the same incident again in six months.

## §7 — Where this chapter hands off

- **Chapter 05 (Retraining as a first-class deploy)** owns the rollback machinery for the retraining pipeline specifically — the model-registry state machine, the promotion gates, the canary shape for retrains.
- **Mod-305 chapter 03 (Shadow and canary online promotion)** is where the auto-rollback controller for a canary is authored; this chapter's manual rollback discipline is the fallback when the controller is not fast enough or the failure mode is outside its scope.
- **Mod-304 chapter 05 (Observability for LLM-augmented systems)** is the observability layer this chapter's runbooks read from for LLM-specific incidents.
- **Mod-309 (Responsible AI Governance)** is where fairness incidents and adversarial incidents get their security-facing review; this chapter's Category F is the *on-call* posture, mod-309 is the *policy* posture.
- **Mod-308 (Platform collaboration)** is where the multi-team incident-response contract is authored — who pages whom, on which SLI, under which severity.

## Summary

Classical incident response (roles, cadence, blameless postmortem) works unchanged for ML; the extension is a domain-specific incident taxonomy (six categories from serving degradation through feature-pipeline breakage, model quality regression, silent drift, cost runaway, and abuse), a set of runbooks that are decision trees named against those categories, a rollback machinery whose primitives (model-version flag, feature-view flag, prompt flag, kill switch, traffic-split kill) are each one-action, and a postmortem template that adds ML-specific fields (category, SLI coverage, rollback rehearsal, training-serving skew, label availability) and demands action items in four categories (prevent, detect, mitigate, practise). The whole system is *practised* — gamedays and tabletops exercise the runbooks and the rollback machinery before the real incident arrives. Chapter 05 zooms into the retraining pipeline as its own reliability-critical deploy path.
