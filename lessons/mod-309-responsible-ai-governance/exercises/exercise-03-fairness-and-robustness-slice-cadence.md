# exercise-03: Fairness, robustness, and safety slice cadence

**Estimated effort:** 2 hours

## Objective

Design the **review program** for a specific deployment — pick the always-on slice set, the periodic-review sweep, the cadence, and the findings-ladder policy — then produce a **first-cycle review packet** that walks through the shape a real cycle would produce. The output is a cadence-decision doc, a fairness-metric decision doc, and a first-cycle review packet.

The exercise output would be usable by a governance analyst as the *ongoing evaluation* layer of the review packet: chapter 01's model card claims are re-verified on the schedule this exercise designs, on the slices this exercise selects.

This is the L30 tell that separates "we ran a fairness eval once" from "we run a review program that a governance analyst schedules against." An L20 team's fairness answer is a notebook; an L30 team's is a signed packet on a defensible cadence with a findings ladder that closes items on schedule.

## Prerequisites

- Read chapter 03 (`03-fairness-robustness-safety-review-cadence.md`) — the three review dimensions, the metric-choice discipline, the always-on / periodic-review split, the cadence-choice inputs, the four scoping moves, and the review-packet template.
- Skim chapter 01 §7 (model-card cadence policy) — the cadence in the model card and the cadence in this exercise's cadence-decision doc must be *the same policy*.
- Skim [mod-305 chapter 02 (slice and adversarial guardrails)](../../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) — the always-on layer *is* that chapter's harness; you are inheriting it, not re-designing it.
- Skim [Fairlearn's user guide](https://fairlearn.org/main/user_guide/) or [AIF360's tutorial](https://aif360.res.ibm.com/) — enough to have concrete metric names in hand when authoring the fairness-metric decision doc.
- Skim the [NIST AI RMF Playbook](https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook) — the *Measure* and *Manage* functions where cadence discipline lives.

## Pick your deployment

Use the same deployment as exercises 01 and 02, or pick a fresh one from the D1–D5 list in exercise 01. Deployments with a plausible policy-relevant group breakdown (D1 rec-sys with user demographics, D2 fraud with regional differences, D3 LLM-triage with user-language subpopulations, D4 perception with sensor-condition subpopulations) will exercise the fairness-metric decision most fully.

## Steps

### 1. Risk-posture and cadence-input framing (≈ 15 min)

Write a short section that names, for this deployment:

- **Risk posture.** Which of chapter 03 §4's four tiers (minimal, standard, elevated, high) the deployment lives in. Cite the reason — internal AI-use policy, EU AI Act tier, SR 11-7 tier, safety-critical designation.
- **Retraining cadence.** How often the model is promoted (from mod-307 chapter 05 planning if you have it; otherwise a defensible cadence).
- **Population shift rate.** Fast (adversarial pressure, seasonal traffic, region rollouts), medium, or slow. mod-307 chapter 02's distribution-drift SLI would surface fast shifts.
- **Incident history.** Any prior fairness / safety incidents in a recent window, and their heightened-cadence implications.

These four inputs feed the cadence choice in step 4. The framing goes in the cadence-decision doc.

### 2. Author the fairness-metric decision doc (≈ 20 min)

Draft a short doc — `fairness-metric-decision-<feature>.md`, 1–2 pages — that names:

- **The fairness question the deployment asks.** In plain language, what disparate treatment or disparate impact the deployment must guard against. Barocas et al.'s relational-reasoning discipline.
- **The metrics considered.** At least three of demographic parity, equal opportunity, equalised odds, calibration by group.
- **The metric chosen.** Named. With a paragraph justifying against the deployment's ethical structure.
- **The metric(s) not chosen.** With a paragraph naming *why not* — usually citing an impossibility result (Chouldechova 2017, Kleinberg-Mullainathan-Raghavan 2016) or a mismatch with the use case.
- **Intersectional slice choices.** Which factor pairs are stratified on (Buolamwini-Gebru intersectional discipline); which are not, and why.
- **Group-set choice.** Which groups the review stratifies on and the reason the set stops there (Kearns et al. 2018 subgroup-fairness intractability is the reason it must stop somewhere).

The doc feeds two artefacts: the model card §4 (metrics) and §8 (ethical considerations) both cite it.

### 3. Author the always-on slice set (≈ 15 min)

Enumerate the always-on slice set for the deployment (chapter 03 §3.1). Rules:

- **10–15 slices maximum.** More than that is noise.
- **Each slice ties to a specific product or policy risk.** No slice without a stated purpose.
- **Includes at least two per-group slices**, at least two intersectional slices, at least two robustness slices (per-corruption or per-shift-regime), and at least one safety slice (for LLM-augmented systems, an off-policy or refusal-vs.-over-refusal slice; for classical systems, a high-consequence-error slice).
- **Each slice has a delta rule against baseline** (chapter 03 §3.1's ratcheted-not-gated discipline). Note the specific "no worse than baseline − X %" tolerance.

The list feeds mod-305 chapter 02's harness. If the exercise is being done in a training context without an existing harness, note that the harness would need to be extended — this is one of the F1 roadmap items in the packet later.

### 4. Author the cadence policy (≈ 15 min)

Using the four inputs from step 1 and the default table from chapter 03 §4, name:

- **Fairness review cadence.** With justification.
- **Robustness review cadence.** With justification.
- **Safety review cadence.** With justification.
- **Out-of-cycle triggers.** What events pull a review forward. At minimum: a new region deployment, a new data source, a new regulation, an incident from mod-307 chapter 04 whose root cause touches a fairness / robustness / safety slice, or a governance-flagged concern.
- **Defence of any deviation from the default table.** If the deployment is at "elevated risk" but you chose a slower cadence than chapter 03 §4's default, name the specific mitigation that lets you defend the slower cadence to the reviewer.

The cadence goes in the model card (chapter 01 §7) *and* in this cadence-decision doc; both must reference the same policy.

### 5. Author the periodic-review sweep for cycle 1 (≈ 20 min)

The periodic-review sweep is the *deeper* slice set that runs on cadence, not on every candidate. For cycle 1:

- **20–50 slices.** Broader than the always-on set.
- **A rotation policy** naming which of the slices are *deep-dived* this cycle (chapter 03 §5.1). Rotation ensures every slice is deep-dived at least once per year (or per the policy cycle).
- **Any red-team eval set** (chapter 03 §1.3) for the safety review — the specific corpus, its size, its source, its refresh cadence.
- **Any distribution-shift replay** for the robustness review — which corruption or shift regime, on what baseline.
- **Sample-size and CI shape** — how many examples per slice, what CI method, what sample-size floor below which a slice reports "under-powered" rather than a metric.

The list is not exhaustive; it is enough for a cycle-1 dry run. Cycle 2's list will be a superset — this is the moving-window discipline.

### 6. Produce the first-cycle review packet (≈ 25 min)

Following the chapter 03 §6 skeleton, author a first-cycle review packet — `review-packet-<feature>-<yyyy-qN>.md` — with:

- **Summary.** 2–3 sentences, invented but plausible: overall verdict, count of findings by severity, any recommended cadence change.
- **Always-on layer window aggregate.** Invented but plausible aggregate — which slices trended flat, which trended down, which candidates were blocked and why.
- **Periodic-review results.** For each slice in cycle 1's list, a plausible result with CI. At least three intersectional results.
- **Deep-dive narratives.** For each rotation-pulled slice, a paragraph on error modes and root causes.
- **Findings.** At least one at each of F0, F1, F2, F3 (or an explicit "no F0 this cycle" with a defence). Each finding: named, owner, target close date, blocking rationale (for F0), governance signoff (for F3).
- **Cadence assessment.** Was cycle 1's cadence right? Any recommendation.
- **Model-card updates required.** For each finding that changes a card claim, name the section and the update.
- **Signoff.** ML team owner, governance analyst, product policy (mock signoffs are acceptable).

The packet is 3–6 pages. If it is longer, the review is being asked to consume too much; prune.

### 7. Compose against the model card (≈ 10 min)

Update the model card (from exercise 01) with:

- **Section 4** — the fairness metric chosen (from step 2's decision doc).
- **Section 7** — the cycle-1 quantitative results (a subset — the load-bearing numbers).
- **Section 8** — the F3 accepted risks the cycle 1 packet named.
- **Section 9** — the F1 / F2 roadmap / backlog items as caveats.

Every card update cites the review-packet cycle that produced it.

## Deliverable

Three linked Markdown documents:

- `fairness-metric-decision-<feature>.md` — the metric-choice doc, 1–2 pages.
- `cadence-decision-<feature>.md` — the risk posture, the always-on list, the periodic-review list, the cadence policy, the out-of-cycle triggers, 3–4 pages.
- `review-packet-<feature>-<yyyy-qN>.md` — the cycle-1 review packet, 3–6 pages.

Plus a diff-style update to the model card (from exercise 01) with the fairness-metric choice, the cycle-1 quantitative results, and the findings-ladder cross-references.

## Acceptance criteria

- [ ] Risk-posture framing names the tier and cites the reason (policy, regulation, safety designation).
- [ ] Fairness-metric decision doc names at least three metrics considered, one chosen, and at least one *not* chosen with an impossibility-result citation.
- [ ] Fairness-metric decision doc names intersectional-slice choices and defends the group-set boundary.
- [ ] Always-on slice list has 10–15 slices, each with a stated product / policy risk purpose and a baseline-delta rule.
- [ ] Always-on list includes at least two per-group, two intersectional, two robustness, and one safety slice.
- [ ] Cadence policy names the cadence for each of fairness / robustness / safety, with justification for each.
- [ ] Cadence policy names the out-of-cycle triggers.
- [ ] Any deviation from chapter 03 §4's default table is defended.
- [ ] Periodic-review sweep for cycle 1 has 20–50 slices with a rotation policy, CI shape, and sample-size discipline.
- [ ] Review packet cycle 1 has at least one finding at each of F0 / F1 / F2 / F3 (or explicit "none at F0" with defence).
- [ ] Review packet's model-card-updates list feeds back into the card.
- [ ] Model card §4, §7, §8, §9 updated from the review packet.
- [ ] All three docs are signed (mock signoffs acceptable).

## Stretch goals

- **The findings-ladder policy doc.** Chapter 03 §5.4 named the four-tier ladder in shape; write the *policy* — how findings are triaged in-review, who authorises an F1 → F2 downgrade, who authorises F3 acceptance, and the escalation-if-stale rules. The policy lives with the cadence-decision doc.
- **Compose with the mod-306 online slice diagnostics.** For a deployment with an active experimentation program, the offline slice matrix here has an online twin — the [mod-306 chapter 02](../../mod-306-experimentation-at-scale/02-srm-and-trust-diagnostics.md) SRM-and-diagnostic layer. Draft a short section on how the offline and online slice sets are aligned (or on the specific reason they diverge — sometimes the online audit picks up slices offline data cannot resolve).
- **The rotation-policy pull for cycle 2 and cycle 3.** The rotation policy claim is only credible if you can walk it forward. Author the specific slice list for cycle 2 and cycle 3, showing that every slice in your always-on set will be deep-dived within one year.
- **The incident-cycle drill.** Pick one mod-307 chapter 04 incident-response scenario whose root cause hits a slice in your set. Walk through how the incident triggers an out-of-cycle review, what the review adds to the F0 pile, and how the packet cycle changes.
- **The regulatory-audit walk.** Imagine you have received a regulator's request for the deployment's fairness evidence for the last 12 months. Which review-packet cycles do you point at? Are there gaps in the record (missing cycles, missing signoffs)? This is the audit-readiness discipline that turns cadence from a rhythm into a *contract*.
- **The impossible-metric worked example.** Take Chouldechova 2017's calibration-vs.-error-rate-balance impossibility. On your deployment, describe what the impossibility looks like *concretely* — pick two groups with different base rates, sketch what the numbers would show, and walk what you would report in the review packet's §3.2 metric-results section.
