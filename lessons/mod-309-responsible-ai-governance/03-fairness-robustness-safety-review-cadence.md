# Fairness, robustness, and safety slices on a review cadence

## Motivation

Fairness, robustness, and safety slice reviews fail in two symmetric ways. On one side, the team runs them **once**, at launch, and never again — the launch review passes, the model is redeployed monthly, and the reviewer has no idea whether the slice numbers still hold six months later. On the other side, the team runs them **every day**, on every candidate, exhaustively — the eval-set explodes, the harness takes six hours, engineers stop looking at the results, and the roadmap stalls under the weight of its own diligence.

The senior discipline is a **review cadence** that separates *always-on guardrails* (mod-305 chapter 02's slice pass rates, run on every candidate) from *periodic review slices* (fairness / robustness / safety questions that need a deeper look, run on a schedule that matches their signal). The cadence lets the team ship *and* stay accountable: the always-on layer catches the fast regressions, the periodic layer answers the questions that only surface after enough traffic accumulates.

This chapter is the *review-program* layer on top of mod-305 chapter 02's slice-and-guardrail vocabulary. Where mod-305 named the slices you gate promotions with, this chapter names the slices you *revisit*, the cadence you revisit them on, and the escape hatch for scoping reviews down so the roadmap does not stall on completeness that is not paying its way.

Primary references:

- **Barocas, Hardt, Narayanan — [*Fairness and Machine Learning*](https://fairmlbook.org/)** (fairmlbook.org, in progress). The canonical textbook on fairness metrics, their impossibility results, and the choice of metric as a value-laden decision.
- **Mehrabi et al., 2019 — [*A Survey on Bias and Fairness in Machine Learning*](https://arxiv.org/abs/1908.09635).** The lay of the land for the fairness-metric literature.
- **Hendrycks & Dietterich, 2019 — [*Benchmarking Neural Network Robustness to Common Corruptions and Perturbations*](https://arxiv.org/abs/1903.12261).** The reference framing for robustness-slice testing under distribution shift.
- **Weidinger et al. (DeepMind), 2021 — [*Ethical and social risks of harm from Language Models*](https://arxiv.org/abs/2112.04359).** The reference for the safety-risk taxonomy chapter 04's threat model composes with.
- **[NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework)** — the *Measure* and *Manage* functions where the review cadence lives.

## §1 — The three review dimensions

The review program has three dimensions, each answering a different question and requiring a different toolkit. The senior discipline is to name which of the three a slice belongs to, because the *right* fairness slice is a wrong answer to a safety question.

### 1.1 — Fairness slices

**Question.** Does model performance vary in ways that produce policy-relevant harm across protected or sensitive groups?

**Primitive.** A metric computed per-group and compared across groups. Which metric — demographic parity, equal opportunity, equalised odds, calibration by group, disparate impact — is a *value-laden choice*, not a technical one. Barocas et al.'s [chapter on relational reasoning](https://fairmlbook.org/pdf/fairmlbook.pdf) is the reference for the choice.

**Tool.** The [Fairlearn](https://fairlearn.org/) library and the IBM [AIF360](https://aif360.res.ibm.com/) toolkit both implement the standard fairness metrics; both surface the load-bearing tension between them.

**Impossibility.** Chouldechova, 2017 ([*Fair prediction with disparate impact*](https://arxiv.org/abs/1610.07524)) and Kleinberg-Mullainathan-Raghavan, 2016 ([*Inherent Trade-Offs in the Fair Determination of Risk Scores*](https://arxiv.org/abs/1609.05807)) established that except in trivial cases, calibration and error-rate balance cannot be simultaneously satisfied across groups whose base rates differ. Any fairness review reports the metric it *chose*, the metric it *did not*, and *why*.

### 1.2 — Robustness slices

**Question.** How does the model behave under distribution shift — inputs the training distribution did not cover, adversarial perturbations, upstream data-quality regressions, time drift?

**Primitive.** A metric computed against the primary evaluation set and re-computed against a *shifted* eval set — corrupted inputs, out-of-distribution inputs, adversarially perturbed inputs, temporal-shift replays. The delta is the robustness signal.

**Tool.** [Hendrycks & Dietterich](https://arxiv.org/abs/1903.12261)'s ImageNet-C style corruption suite for vision, [CheckList](https://github.com/marcotcr/checklist) for NLP behavioural testing, HELM's [robustness scenario](https://crfm.stanford.edu/helm/) for LLM regime.

**Distinction from fairness.** Robustness slices can *look* like fairness slices (a per-region slice, a per-device slice) but are asked as *"does performance degrade under shift"* rather than *"does performance vary across a policy-protected group."* Same slice, different question, different metric.

### 1.3 — Safety slices

**Question.** Does the model produce outputs that constitute one of the harm categories the deployment's risk posture names? For LLM-augmented systems: harmful content, PII leakage, misinformation, security-sensitive advice, refusal-vs.-over-refusal balance. For classical ML: high-stakes false positives / negatives, over-confident wrong predictions, adversarial-input triggered failures.

**Primitive.** A *red-team eval set* — inputs designed to *induce* the failure mode, with either a classifier-based check or a human review determining pass/fail. The safety slice is the fraction of the red-team set the model handles correctly, per category.

**Tool.** The [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/), MITRE ATLAS mitigation catalogue, [Perez et al. red-teaming](https://arxiv.org/abs/2202.03286), and the [HarmBench](https://arxiv.org/abs/2402.04249) taxonomy are external references. Internally, most teams maintain a **taxonomy of unacceptable outputs** for their specific deployment — the set of harm categories the policy team, legal, and product agree the model must not produce.

**Coupling with chapter 04.** Safety slices are the *evaluation* side of the *threat model* chapter 04 authors. A well-formed safety slice matrix is one-to-one with the threat-model attacks: every attack has a slice, every slice traces to an attack.

## §2 — Choosing the metric: a value-laden decision

The single most common failure mode in fairness slice reviews is *unnamed metric choice*. The team runs "the fairness metric," reports a number, and never names *which* metric and why. Because different metrics are mutually incompatible, the number is not interpretable without the choice.

Four metrics recur across senior-ML fairness reviews, and the discipline is to know which of the four the deployment requires and why:

- **Demographic parity** (also *statistical parity*). `P(prediction = positive | group = A) == P(prediction = positive | group = B)`. Appropriate when the intervention is meant to *increase representation* regardless of ground-truth base-rate differences (some scholarship-allocation and outreach-decision cases). Inappropriate when ground-truth base rates legitimately differ and the deployment is a diagnostic (a medical test with base-rate differences by group).
- **Equal opportunity.** `P(prediction = positive | group, true label = positive) == P(prediction = positive | group', true label = positive)`. True-positive rates match across groups; false-positive rates may differ. Hardt et al. 2016 ([*Equality of Opportunity in Supervised Learning*](https://arxiv.org/abs/1610.02413)) is the paper.
- **Equalised odds.** Both true-positive and false-positive rates match across groups. Stricter than equal opportunity.
- **Calibration by group.** `P(true label = positive | predicted probability = p, group) == p` for every group. Chouldechova 2017 showed this cannot be simultaneously satisfied with error-rate balance when base rates differ.

The senior discipline is a **fairness metric decision doc** that names the deployment's fairness question, the metrics considered, the metric chosen, and the metrics that could not be simultaneously satisfied. The doc is one of the model-card's audit hooks (chapter 01 §3) — the model card cites the doc for its fairness claims.

Two adjacent notes:

- **Intersectional fairness.** Per-group metrics can *pass* on gender-slice and on race-slice separately and *fail* on the (gender × race) intersection. Buolamwini & Gebru 2018's [*Gender Shades*](http://gendershades.org/) result is the reference — the paper stratified commercial face-classification error rates by (skin tone × gender) and found intersectional gaps hidden by unitary reports. mod-305 chapter 02's intersectional-slice discipline is the eval-harness expression of this; the fairness review is where the intersection choice gets named and defended.
- **Group definition is not free.** Which groups the review stratifies on is itself a choice — the [Kearns et al. 2018 subgroup fairness](https://arxiv.org/abs/1711.05144) result showed that guaranteeing fairness across all identifiable subgroups is intractable. The senior discipline is to name the *finite set* of groups the review stratifies on, and to name the reason the set stops there (the groups regulation names, the groups incident history flagged, the groups that appear in the data at analysable frequency).

## §3 — The always-on layer vs. the periodic-review layer

The distinction that makes the cadence work is between *always-on* slice guardrails and *periodic-review* slice sweeps.

### 3.1 — Always-on guardrails (every candidate)

These are the slices that run on **every** candidate model, in the mod-305 chapter 02 harness, as promotion-gating guardrails. Properties:

- **Small set.** Ten to fifteen slices maximum. Anything larger becomes noise and gets ignored by the engineers watching it.
- **Direct product-risk mapping.** Each slice must correspond to a business or policy risk the team has explicitly signed up to guard against.
- **Ratcheted, not gated absolutely.** The primary rule is *no regression* against the promoted baseline, not *pass an absolute threshold* — mod-305 chapter 02's baseline-delta discipline. Absolute thresholds hide silent drift; relative regressions catch it.
- **Reported in the release memo.** Every candidate's per-slice deltas ship in the release memo (mod-305 chapter 04), regardless of whether they blocked promotion.

The always-on layer is what makes the *periodic review* free to ask the harder questions. When the always-on layer is trustworthy, the reviewers do not need to re-verify the guardrails on every review — they focus on the deeper slices instead.

### 3.2 — Periodic review sweeps (on cadence)

These are the slices that run on a schedule — monthly, quarterly, or event-triggered. Properties:

- **Larger and more expensive.** Twenty to fifty slices, potentially with human-in-the-loop rating (safety), with a red-team eval set, or with distribution-shift replays that are not part of the always-on harness.
- **Report against absolute standards, not just baseline delta.** The always-on layer catches regressions; the periodic layer answers "is this deployment *fair enough*, *robust enough*, *safe enough* in its own right." Both matter — the always-on layer alone can drift downward through a series of individually-acceptable regressions.
- **Produce a review packet, not just numbers.** The review packet includes the metric results, the slice-methodology commentary, findings, open items, and a signoff. The packet composes into the model card's next revision (chapter 01) and the governance-analyst hand-off (chapter 01 §6).
- **Triggerable out of schedule.** A production incident (mod-307 chapter 04), a new region deployment, a new data source, or a governance-flagged concern can trigger an out-of-cycle review that pulls the schedule forward.

Both layers are load-bearing. Teams that skip the always-on layer discover regressions at the next review, six weeks late. Teams that skip the periodic review layer discover the slow-drift-to-broken-fairness only when an incident forces the question.

## §4 — Cadence choice: matching signal to schedule

The cadence question is "how often is the periodic review worth running." A cadence too tight burns engineering hours on noise; too loose lets slow drift accumulate to policy-visible failure. Four inputs drive the choice:

- **Retraining frequency.** A model retrained weekly needs faster periodic review than one retrained annually. The default rule: at minimum, one full periodic review per retraining cadence, plus at least one *between* retrainings to catch drift not driven by the retrain.
- **Population shift rate.** A deployment whose serving population's distribution changes fast (new region rollouts, seasonal traffic, adversarial pressure in fraud / spam / abuse) needs faster cadence. mod-307 chapter 02's distribution-drift SLI is the *signal* that would compress the cadence.
- **Risk classification.** The EU AI Act's [risk tiers](https://artificialintelligenceact.eu/) (unacceptable, high-risk, limited-risk, minimal-risk) and the NIST AI RMF's severity mapping are the standard external references. Higher-risk deployments compress the cadence; lower-risk ones expand it. The internal AI-use policy usually maps to these tiers.
- **Incident history.** A deployment that has had a fairness or safety incident in the last N periodic cycles is under a *heightened* cadence until the review packet shows stable pass rates for M cycles. This is analogous to the SR 11-7 model-risk-management "elevated monitoring" posture.

A reasonable default cadence table for a mature senior-ML program:

| Deployment risk posture | Fairness review | Robustness review | Safety review |
|---|---|---|---|
| Minimal-risk (internal-only tooling, no user impact) | Annual | Annual | On event only |
| Standard-risk (user-facing but reversible, no protected-class impact) | Semi-annual | Semi-annual | Semi-annual |
| Elevated-risk (protected-class impact, or high-consequence decisions) | Quarterly | Quarterly | Monthly |
| High-risk (regulated decision, safety-critical) | Monthly | Monthly | Continuous with monthly deep review |

The table is a *default*; the specific deployment's cadence is defended in the model card §7. A team that argues for a slower cadence than the risk-posture default is arguing to the reviewer; the argument must be explicit.

## §5 — Scoping without stalling: the four moves

The core roadmap tension is that the review program *takes time* — engineering to author the slices, run the harness, review the outputs, close the findings. Uncontrolled, it displaces feature work; over-controlled, it silently degrades the deployment's accountability. Four moves keep the program tractable without letting it stall the roadmap.

### 5.1 — Rotate deep-dive slices

Not every slice is examined in depth every cycle. A **rotation policy** picks a handful of slices per cycle for deep-dive treatment (per-slice error mode narrative, human-review sampling, root-cause investigation on regressions). The other slices continue to be *measured* but not *narrated*. Rotation ensures every slice is deep-dived once per year (or per whatever the policy cycle is), without the entire matrix consuming a cycle each time.

The rotation policy is the fairness-review-program's version of the code-review triage discipline: not every diff gets the same attention, but every diff eventually gets attention on schedule.

### 5.2 — Cap the always-on layer

The always-on slice count is fixed. Adding a slice to the always-on layer means removing one — because always-on is a *budget*, not a wish list. This is what keeps the release harness fast enough that engineers still look at its output.

The candidate for eviction is usually the slice whose regressions have been consistent-false-alarms or whose signal is subsumed by another slice. The eviction decision is made *in the review*, not in the hallway.

### 5.3 — Make the review a *day*, not a *quarter*

A well-run periodic review is a *day of work* for the program owner, not a *quarter*. The way it becomes a day: the harness has already run (the always-on layer + the scheduled periodic sweeps produce the numbers automatically), the eval sets are versioned and re-used, the review-packet template is filled in from the previous cycle with the deltas highlighted, and the review meeting is the last two hours where the humans discuss the findings.

If the periodic review consumes a week, the harness or the packet template is not carrying its weight; the fix is investment in the automation, not stretching the cadence.

### 5.4 — The findings ladder

Every review produces findings. Uncontrolled, findings pile up and the program becomes cynical ("we know it fails on slice X, we've known for a year"). The discipline is a **findings ladder**:

- **F0 — Blocking.** Findings that block the next promotion until closed. Bounded number per cycle; more than 2 F0s is a signal the program has *not* been running frequently enough.
- **F1 — Roadmap.** Findings that go on the team's roadmap for the next cycle. Owners and dates assigned in-review.
- **F2 — Backlog.** Findings that are on the backlog with an explicit "we will not work this in the next N cycles" note. The reviewer signs off on F2 status, and the review re-visits F2 findings every K cycles.
- **F3 — Accepted risk.** Findings that the deployment will not fix; the risk is *disclosed* in the model card §8 and §9 and reviewed by governance. F3 requires an explicit governance signoff, not a team decision.

The ladder is what keeps the program *actionable*. A finding that lives in F1 for four cycles without progress gets escalated to F0; a finding that lives in F2 for four cycles gets re-triaged. The ladder is one of the artefacts the review packet includes.

## §6 — The review packet: what ships at each cadence

The **fairness / robustness / safety review packet** is the artefact each periodic cycle produces. Skeleton:

```markdown
# Review packet — <feature-name> — <yyyy-qN or yyyy-mm>

**Cycle window.** <start> → <end>
**Review type.** Fairness / Robustness / Safety (mark all that apply)
**Model versions in scope.** <list of registry versions promoted or in canary during the window>
**Packet owner.** <name, role>
**Reviewers.** <governance analyst, senior-ML on team, product policy>

## 1. Summary
<2–3 sentences: overall verdict, count of F0 / F1 / F2 findings, any recommended cadence change.>

## 2. Always-on layer status (window aggregate)
<Table: slice × metric × delta, aggregated over the window; note any release blocked on this axis and why.>

## 3. Periodic-review results
### 3.1 Slices reviewed this cycle
<List of slices in scope, with rationale (rotation-policy pull, event-triggered, or new-slice-first-review).>

### 3.2 Metric results
<Table with CIs; per-slice, per-group, per-intersection where relevant.>

### 3.3 Deep-dive narratives
<For each rotation-pulled slice, a paragraph on error modes, root causes, mitigations attempted.>

## 4. Findings
### 4.1 F0 (blocking)
<Named finding, owner, target close date, blocking rationale.>

### 4.2 F1 (roadmap)
<Same fields; assigned to a roadmap cycle.>

### 4.3 F2 (backlog)
<Same fields; explicit "will not work in the next N cycles" note.>

### 4.4 F3 (accepted risk)
<Same fields; explicit governance signoff link.>

## 5. Cadence assessment
<Was this cycle's cadence right, too fast, too slow? Any recommendation for the next cycle. Any events that would trigger an out-of-cycle review.>

## 6. Model-card updates required
<For each finding that changes a model-card claim (§2 intended use, §7 quantitative analysis, §8 ethical considerations, §9 caveats), name the section and the update.>

## 7. Signoff
- ML team owner: <name, date>
- Governance analyst: <name, date>
- Product policy (if applicable): <name, date>
```

Three properties of the packet that matter:

- **The packet is short.** Two to six pages for a standard-risk deployment; ten pages for a high-risk one. If the packet is longer, the review is being asked to consume too much — invest in the automation, not the reviewer's endurance.
- **The packet composes with the model card.** Section 6's model-card-updates list is the direct feed into the card's next revision (chapter 01 §7).
- **The packet is signed.** The signoff line is not a formality — it is what makes the packet the artefact governance can quote back if the deployment's behaviour is later challenged.

## §7 — Handling the impossibility results: how to review when the metrics conflict

Chapter §2 flagged that fairness metrics are pairwise incompatible in the general case. The consequence for the review program: **the review's job is not to make the incompatible metrics both pass — it is to make the choice explicit and defensible.**

Three operational moves:

- **Publish the choice.** The model card §4 (metrics) and §8 (ethical considerations) name the fairness metric the team optimises for, the metric it does not, and why. The publication itself is what makes the deployment defensible — an unstated choice reads to a reviewer as an unmade choice.
- **Report both.** Even when the team optimises for calibration, the equal-opportunity numbers are still reported in the review packet. The reviewer can see the trade-off and confirm the team is *not* silently drifting on the un-optimised metric.
- **Justify the choice against the deployment.** Barocas et al.'s [chapter on relational reasoning](https://fairmlbook.org/pdf/fairmlbook.pdf) is the reference for tying the metric choice to the *use case* — a diagnostic model has a different ethical structure from an outreach model has a different structure from a lending model. The senior discipline is to write the justification, not to reach for the metric that gives the best numbers.

Impossibility results are not a defeat — they are a constraint that forces the review program to think about *values*, not just *numbers*. The review that does this well is one that produces a legible policy artefact; the review that does this poorly is one that gets challenged and has no answer.

## §8 — Composing with the rest of the module

The review program does not stand alone. Chapters 01, 02, and 04 each connect:

- **Model card (chapter 01).** The model card §7 (quantitative analyses) reports the current cycle's slice numbers; §8 (ethical considerations) reflects the F3 accepted risks; §9 (caveats) reflects the F1 / F2 roadmap items. Every cycle updates the card.
- **Lineage and PII (chapter 02).** The lineage audit's findings can trigger a fairness or safety review — a discovered PII exposure in a prediction log, for instance, is a safety finding. The review program is one of the audit's *downstream consumers*.
- **Threat model (chapter 04).** Safety slices are the eval-side expression of the threat model's misuse patterns. Every threat-model attack has a corresponding safety slice; a threat-model update adds or updates a safety slice in the next review.
- **mod-305 chapter 02.** The always-on layer *is* mod-305 chapter 02's slice-and-adversarial-guardrail harness, running on every candidate. The periodic layer is the review program's addition on top.
- **mod-306 chapter 02.** The experimentation platform's slice-level diagnostics for A/B experiments are the *online* twin of the offline review program — the same slice matrix, applied against production traffic. SRM and slice-diagnostic disciplines from mod-306 chapter 02 transfer.
- **mod-307 chapter 04.** A production incident whose root cause hits a fairness or safety slice triggers an out-of-cycle review; the postmortem's action items feed the F0 / F1 ladder.

## Summary

The review program has three dimensions — fairness (does performance vary across policy-relevant groups), robustness (does performance degrade under shift), safety (does the model produce prohibited outputs) — each with a different metric family, a different tool, and a different question. Fairness-metric choice is *value-laden* — demographic parity, equal opportunity, equalised odds, and calibration by group are pairwise incompatible in the general case, and the senior discipline is a metric decision doc that names the choice and the trade-off, not a metric selection that reaches for the best numbers. The cadence separates an *always-on layer* (small slice set, every candidate, promotion-gating, mod-305 chapter 02's harness) from a *periodic-review layer* (larger, deeper, on schedule, packet-producing). Cadence choice is driven by retraining frequency, population shift rate, risk classification, and incident history — high-risk deployments compress to monthly, minimal-risk expand to annual, with a defended default in the model card. Scoping keeps the program tractable via four moves: rotate deep-dive slices, cap the always-on layer, make the review a day not a quarter, and manage findings on a four-tier ladder (F0 blocking, F1 roadmap, F2 backlog, F3 accepted risk). The **review packet** is the artefact each cycle ships — summary, always-on status, periodic results, findings ladder, cadence assessment, model-card updates, signoff — short enough that reviewers actually read it. The impossibility results in fairness are handled by publishing the choice, reporting both metrics, and justifying the choice against the deployment; the review program's job is legibility of the trade-off, not the impossible reconciliation. The program composes with the model card (chapter 01), the lineage audit (chapter 02), the threat model (chapter 04), the mod-305 harness, the mod-306 experimentation-slice diagnostics, and the mod-307 incident-response taxonomy.
