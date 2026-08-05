# exercise-04: Misuse and adversarial threat model

**Estimated effort:** 2 hours

## Objective

Author the **misuse-and-adversarial threat model** for a specific deployment — deployment description, attacker profiles, two-to-five domain-specific adversarial threats, two-to-five misuse patterns, a full mitigation matrix mapping every threat and misuse to a specific control in the deployed system, and a test-anchor for every mitigation. Close the loop by running a peer review with an ML/AI security stand-in.

The output is a threat-model document (`threat-model-<feature>-v<n>.md`) that a governance analyst signs, that composes with the model card (chapter 01), the lineage audit (chapter 02), and the review packet (chapter 03), and that the retraining cadence renews.

This is the L30 tell that separates "we thought about security" from "we authored a threat model that names the two-to-five deployment-realistic threats deeply, maps mitigations to specific controls, anchors each to a test, and is peer-reviewed by the security specialist track." An L20 team ships a launch review with a slide called "Security Considerations"; an L30 team ships a signed threat model with a mitigation matrix and an audit trail.

## Prerequisites

- Read chapter 04 (`04-misuse-and-adversarial-threat-modeling.md`) — the five-question ML frame, the four adversarial categories, the five misuse patterns, the seven mitigation landing sites, the test-anchor discipline, the threat-model artefact, and the five failure modes.
- Skim [MITRE ATLAS](https://atlas.mitre.org/) — enough to have specific tactic and technique IDs (AML.TXXXX) in hand when authoring adversarial threats.
- Skim the [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/) for LLM-augmented deployments — LLM01 through LLM10 give you a named starting checklist.
- Skim [NIST AI 100-2e2023](https://csrc.nist.gov/publications/detail/ai/100-2e2023/final) — sections 3–6 for the taxonomy vocabulary.
- Skim Shostack's [*Threat Modeling*](https://shostack.org/books/threat-modeling-book) or his four-question summary — the general threat-modelling discipline the ML variant composes with.
- If your deployment is LLM-augmented, skim [Greshake et al. 2023 (*Not what you've signed up for*)](https://arxiv.org/abs/2302.12173) for indirect prompt injection through retrieval sources.

## Pick your deployment

Use the same deployment as exercises 01–03, or pick a fresh one from the D1–D5 list in exercise 01. Deployments with a plausible adversarial pressure (D2 fraud, D3 LLM-triage with prompt-injection surface, D4 perception with physical-world attack surface) or a plausible misuse pressure (D1 rec-sys with feedback-loop harm, D3 LLM-triage with off-label reuse) will exercise the threat-model discipline most fully.

## Steps

### 1. Deployment description and attacker profiles (≈ 20 min)

Two subsections of the threat-model doc:

- **Deployment description.** Two-to-three paragraphs. What the deployment does, whom it serves, what decisions it enables, what data it consumes and produces. Point at the model card (§1, §2) and the lineage graph. This is the *what are we building* answer.
- **Attacker profiles.** Two-to-five profiles. For each: capability (query access, training-time access, physical access, insider), motivation (financial, adversarial-competition, ideological, criminal, curious), resource level (nation-state, organised group, individual, script-kiddie), access pattern (bulk vs. targeted, sustained vs. burst, coordinated vs. lone). Skip profiles that are not realistic for the deployment — no imagined nation-state attacker against a rec-sys unless the platform is politically consequential; no bored-teenager attacker against a fraud model whose reward is money.

The attacker profiles are load-bearing. §3 and §4 will reference them.

### 2. Adversarial threats (≈ 30 min)

Pick **two-to-four** adversarial threats *realistic for the deployment* from the four categories in chapter 04 §2. Do NOT try to cover all four categories shallowly; go deep on two-to-four. For each:

- **Threat name and category.** From the four (evasion, poisoning, extraction / membership inference, abuse at scale). Cite the MITRE ATLAS tactic and technique ID where applicable (e.g., `AML.T0044 – Full ML Model Access`, `AML.T0043 – Craft Adversarial Data`, `AML.T0047 – ML-Enabled Product or Service`).
- **Scenario.** A concrete paragraph — who does what to what, over how long, with what expected effect. "An attacker sends crafted transactions to evade the fraud model over a 30-minute burst" is a scenario; "adversarial examples are a concern" is not.
- **Attacker profile it depends on.** Which of the profiles from §1 can execute this.
- **Prerequisites.** What the attacker needs — access, capability, time, resources. If prerequisites are implausible for the profile, the threat is un-actionable.
- **Detection surface.** What signal — if any — the deployment currently emits that would surface the attack.
- **Blast radius.** If successful, what is compromised — single decision, a batch, a subpopulation, the whole model, the training data.

### 3. Misuse patterns (≈ 25 min)

Pick **two-to-four** misuse patterns *realistic for the deployment* from the five in chapter 04 §3 (off-label use, automation without oversight, feedback-loop harm, aggregation-based harm, cross-model composition). Same structure as adversarial threats — concrete scenario, prerequisites, detection surface, blast radius — but the "attacker" is often a benign downstream user or team.

Discipline notes:
- The misuse pattern *cannot be dismissed with the model card*. If the entire mitigation is "the model card says do not use for this," that is not a mitigation — that is a disclosure. Section 5 must name an *enforcement*.
- Feedback-loop harm and aggregation-based harm are the two most likely to be missed by a team new to threat modelling; force yourself to consider both.

### 4. Author the mitigation matrix (≈ 20 min)

Draft a matrix — rows are the threats and misuse patterns from §2 and §3, columns are the seven landing sites from chapter 04 §5. Cells are `owned / partial / not-covered / not-applicable`, with the specific control named:

| Threat / Misuse | Card §2 scope | Card §8/§9 disclosure | Feature pipeline | Training pipeline | Inference path | Monitoring SLI | Incident-response taxonomy |
|---|---|---|---|---|---|---|---|

At least one row must have a cell filled in each column (across the matrix), so the exercise touches every landing site. Rows without a cell in *any* column are un-mitigated — those are F0 blocking findings.

For every `not-covered` cell where the threat / misuse is realistic, this is a *finding* — either a mitigation to add or an accepted-risk disclosure (F3) with governance signoff.

### 5. Anchor every mitigation to a test (≈ 20 min)

For each `owned` or `partial` cell in the matrix, name the specific test that would catch the mitigation's failure. Test types from chapter 04 §6:

- **Safety slice.** From exercise 03's cycle-1 sweep. Cite the specific slice.
- **Robustness slice.** Same.
- **Monitoring SLI.** From mod-307 chapter 02. Cite the specific SLI + alert.
- **Red-team drill.** Cite the drill scenario and the schedule.
- **Poisoning / backdoor scan.** Cite the scan pipeline and the retraining-loop hook.

Every mitigation must anchor to at least one test. Mitigations without a test are *silent-rot* candidates — either add a test or move the mitigation to accepted-risk with disclosure.

### 6. Findings ladder (≈ 10 min)

Fold the matrix's un-mitigated cells and the un-tested mitigations into the F0 / F1 / F2 / F3 ladder from chapter 03 §5.4. At least one F1 and one F3 finding is expected honestly. F3 findings require a mock governance signoff.

### 7. Peer or mock ML/AI-security review (≈ 15 min)

Either self-review with the security-peer hat on or trade with a peer taking that role. The reviewer walks:

- Are the two-to-four adversarial threats *deployment-realistic* and *specific*, not shopping-list generic?
- Are the misuse patterns *not* dismissed with model-card disclosure alone?
- Does the mitigation matrix cover every threat in at least one column?
- Does every mitigation anchor to a test?
- Are the attacker profiles realistic (or are we imagining nation-states where they do not exist)?

Bring the review notes back into the threat model.

## Deliverable

One Markdown document — `threat-model-<feature>-v<n>.md`, following the chapter 04 §7 template, 4–8 pages — plus a diff-style update to the model card (from exercise 01) with the threat-model's residual-risk disclosures folded into §8 and §9.

## Acceptance criteria

- [ ] Deployment description points at the model card and the lineage graph.
- [ ] At least two attacker profiles, each with capability, motivation, resource level, and access pattern.
- [ ] At least two adversarial threats named, each with MITRE ATLAS ID where applicable, a concrete scenario, prerequisites, detection surface, and blast radius.
- [ ] At least two misuse patterns named (not just adversarial), each with a plausible mechanism, prerequisites, detection surface, and blast radius.
- [ ] Mitigation matrix covers every threat / misuse row in at least one column.
- [ ] At least one row has a filled cell in each of the seven landing-site columns — the matrix exercises every column somewhere.
- [ ] Every misuse pattern has an enforcement in the matrix, not just a card disclosure.
- [ ] Every `owned` / `partial` mitigation cell cites a specific test (safety slice ID, SLI + alert, drill scenario, scan pipeline).
- [ ] Every `not-covered` cell is either a finding on the ladder or an explicit `not-applicable` with justification.
- [ ] Findings ladder includes at least one F1 (roadmap) and one F3 (accepted risk with mock governance signoff).
- [ ] The threat-model doc is signed (mock ML/AI-security-peer and governance-analyst signoffs acceptable).
- [ ] Model card §8 and §9 updated with the threat-model's residual-risk disclosures.

## Stretch goals

- **The prompt-injection deep-dive** (for LLM-augmented deployments). Pick indirect prompt injection through a retrieval source (Greshake et al. 2023) and threat-model it to depth. Include: the specific retrieval sources that are indirect-injection surfaces, the delimiter / instruction-mixing shape the attacker uses, the safety-classifier layer's coverage, the escape scenarios where the classifier does not catch the injection, and the incident-response runbook path.
- **The physical-world adversarial patch deep-dive** (for perception deployments). Pick the specific adversarial-perception attack most realistic for the sensor and operating context (Eykholt et al. 2018 for road-sign, Athalye et al. 2018 for 3D printable). Enumerate the operating conditions under which the attack is feasible, the adversarial-training regime that would mitigate, the run-time input-anomaly detector that would surface, and the failure mode if all three fail.
- **The feedback-loop harm long-run walk** (for recommender / ranking deployments). Simulate — in narrative — how the model's outputs, over 6–12 months, shape the training distribution the next retraining reads. Name the specific slice (chapter 03) that would surface the pathology. This is the aggregation-based-harm scenario that a single-cycle safety review will not catch.
- **The red-team drill design.** For the highest-severity threat in your matrix, author the specific red-team drill — the scenario, the "attackers" (roles played by team members or security peers), the deployment's defences engaged, the pass criteria, and the schedule. This is the mod-307 chapter 04 gameday discipline applied to a threat-model entry.
- **Compose with the incident-response taxonomy.** For each threat in §2, name which mod-307 chapter 04 incident-response taxonomy entry the materialised attack would fall under. If any threat has no matching taxonomy entry, that is a finding — the incident-response taxonomy needs an *Adversarial abuse* extension.
- **Compose with the lineage audit.** For each threat in §2, note whether it is enabled or worsened by any of the lineage findings from exercise 02 (a T5 log drift with sensitive data is a privacy-attack accelerator; a T2 bypass increases poisoning risk). Cross-reference explicitly.
- **The peer-track hand-off contract.** If any threat requires depth beyond your altitude — advanced adversarial-robustness certification, formal verification, cryptographic differential-privacy tuning — draft the [mod-308 chapter 04](../../mod-308-platform-collaboration/04-handoff-contracts-to-specialists.md) six-section hand-off contract for the `ai-infra-security-learning` engagement. The hand-off is the mechanism by which the ML team stays scoped to authoring-and-driving and the security specialist owns the depth.
- **The composed review-packet dry run.** Take all four exercises' artefacts (model card, lineage audit, review packet, threat model) and produce a two-page executive summary that a VP-engineering could read in five minutes to approve the deployment. The dry run is what tests whether the artefacts *compose* — if the summary is impossible to write coherently, one of the artefacts is drifting from the others.
