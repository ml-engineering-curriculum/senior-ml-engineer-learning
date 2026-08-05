# exercise-02: Cost budgets and quotas — enforce, don't just report

**Estimated effort:** 3 hours

## Objective

Author the **enforced** cost budget for the ML feature you chose in exercise-01 — the budget hierarchy, the four enforcement primitives (rate-limits and quotas, circuit breakers, priority queues, kill switches), the degradation path when a budget is stressed, the runaway detection thresholds, and the cost-attribution instrumentation the enforcement reads from. The output is a Markdown design document plus a small worked example (config snippet, pseudocode, or IaC skeleton) of at least one primitive.

This is the L30 tell that separates "we have a cost dashboard" from "we have a cost ceiling." An L20 reports cost after the fact; an L30 blocks the request that would exceed the ceiling at the moment it is made.

## Prerequisites

- Read chapter 03 (`03-cost-budgets-and-quotas.md`) — the report-vs-enforce distinction, the budget hierarchy, the four enforcement primitives, the cost instrumentation shape.
- Skim [mod-304 chapter 04](../../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) — the per-request cost envelope for LLM-augmented features, whose per-request lever you inherit if you picked an LLM-augmented feature.
- Skim [Google *Handling overload*](https://sre.google/sre-book/handling-overload/) and the [FinOps Framework](https://www.finops.org/framework/) — the industry references for quota-based reliability and cloud cost management.
- Bring the feature and stakeholder context from exercise-01 (or draft an equivalent preamble if you jumped straight to this exercise).

## Steps

### 1. Map the cost surface (≈ 20 min)

For the chosen feature, enumerate the *five* cost lines from chapter 03 §1 that apply:

- **Serving compute.** Per-hour GPU / CPU cost, instance types, replica counts. Rough monthly total.
- **Training compute.** Retraining pipeline cost per run, cadence, monthly total.
- **Data-plane storage / I/O.** Warehouse queries for training-data assembly, feature-store storage, streaming pipeline cost. Monthly total.
- **Vendor API calls.** LLM providers, embedding APIs, third-party ML APIs. If none, say so explicitly (and defend it — a hidden vendor dependency is a common failure mode).
- **Data-labelling and human review.** Human-annotator time, RLHF budget, manual-review queues. If none, say so.

Sum to a monthly total. This is the *baseline* the budget is designed against. If a line item is estimated rather than measured, mark it explicitly — those are the ones most likely to blow the budget.

### 2. Author the budget hierarchy (≈ 30 min)

Chapter 03 §3 authored the four-level hierarchy. For your feature, name:

- **Feature budget.** Monthly cap. Soft cap (80 %) alert threshold. Hard cap threshold. Justify the number — cite the baseline from §1 plus headroom for growth.
- **Team budget.** The team's aggregate. Explain why your feature's slot is what it is inside the team budget.
- **Org budget.** The org's total ML spend line as a reference (an estimate is fine).
- **Emergency reserve.** Named. Even if the org does not officially have one, sketch what "we blow the budget for a real reason" looks like — who authorises the overage, what the postmortem obligation is.

Every level names an owner (specific role, not "the team") and a policy-on-breach.

### 3. Enforcement primitives — pick and design (≈ 60 min)

For each of the four primitives from chapter 03 §4, describe *if and how* it applies to your feature. Not every primitive is needed for every feature; the design decision is what is worth naming.

**A. Rate limits and quotas.**

- **Per-tenant quota.** If your feature has multiple tenants, per-tenant N requests/second and M requests/day. Enforcement layer (ingress, service, feature-level). Naming for the reason label emitted on rate-limited requests.
- **Per-model quota.** For a feature calling multiple models (a router, a fallback chain), per-model quotas that mirror the vendor-side or infrastructure-side limits.
- **Per-request cost cap.** For LLM-augmented features especially — the p50 and p99 cost envelope; the degradation for a request that would exceed the cap.

**B. Circuit breakers.**

Specifically for *runaway detection* — a bug-driven spend rate that is 5× or 10× normal for a sustained window. Name:

- The metric the breaker watches (`spend_dollars_per_minute` per feature).
- The threshold (baseline rate × factor).
- The window (chapter 03 §4 uses 3 consecutive 1-minute windows).
- The action (open the breaker for K minutes, route all traffic through the fallback path).
- The escalation (page the on-call).

Provide **pseudocode or config**. Not just prose. A concrete threshold and a concrete action.

**C. Priority queues and load shedding.**

For any component of the feature that is batch or offline (retraining, backfills, offline enrichment), name the priority tier for each and the shedding rule. Chapter 03 §4 uses a 4-tier scheme; adapt to your feature.

**D. Kill switches.**

The nuclear-option flag. Name:

- The flag name and location (feature-flag service, config-store key, ENV var).
- Who is authorised to flip it (on-call, IC, VP-eng-only?).
- The mechanical rehearsal cadence (chapter 03 §4 — quarterly against a canary tenant).
- The audit log — where every flip lands.

### 4. Degradation path (≈ 20 min)

For your feature specifically:

- **What the user sees when the feature is under budget stress.** A cached response? A fallback rule-based system? A downgraded model tier? An explicit "we could not use ML for this request" annotation? The user-facing behaviour is *designed*, not accidental.
- **What the *downstream* consumer sees.** If a schema field is normally populated by your model and the fallback cannot populate it, what does the consumer see?
- **The SLI on the degradation path itself.** A rising rule-routed-rate is a leading indicator that either the budget needs a lift or a model has regressed. Name the SLI and its alert threshold.

### 5. Runaway detection thresholds (≈ 15 min)

Distinct from the budget cap. Chapter 03 §4 emphasises this — the budget cap answers "we spent enough this month," the runaway detector answers "we are spending 100× the expected rate right now."

Name:

- The `baseline_dollars_per_minute` value used for detection (measured over what window; from what historical run).
- The multiplier that trips the runaway breaker (5× is a common default; justify).
- The consecutive-window count required (3× 1-minute windows is common; justify).
- The response — kill switch trip, escalation, incident category (chapter 04 §1 Category E).

### 6. Instrumentation (≈ 20 min)

Chapter 03 §5 named the four instrumentation properties. For your feature, name:

- The `cost_units` metric emitted per request, its labels (feature, tenant, model tier), and the metrics backend it flows to.
- The `spend_dollars_current_window` gauge — how it is aggregated, where it is stored, how fast it is available to the enforcement code.
- The "would-have-cost" counter — active while a kill switch is open, so the postmortem can quantify avoided damage.
- The vendor-side reporting (AWS Cost Anomaly Detection, GCP Budget Alerts, Anthropic / OpenAI usage API) as the complementary reporting layer.

### 7. Composed cost-budget section of the SLO document (≈ 15 min)

Chapter 03 §6 sketched the composed cost-budget section of the SLO document. Assemble your document's version — the header from exercise-01, extended with the sections you built here (§2 hierarchy, §3 primitives, §4 degradation, §5 runaway detection, §6 instrumentation).

The result is a *self-contained* cost-budget document that a reviewer can read once and understand what will happen at 80 %, 100 %, and 500 % of budget.

## Deliverable

A single Markdown document, 3–5 pages, titled `cost-budget-<feature>.md`, structured as:

- Header — feature name, owner, finance stakeholder, last reviewed, next review.
- Cost surface (§1) — the five lines and monthly baseline.
- Budget hierarchy (§2) — feature / team / org / emergency, with owners.
- Enforcement primitives (§3) — rate limits, circuit breaker (with pseudocode / config), priority queues, kill switches.
- Degradation path (§4) — what the user and downstream see under stress.
- Runaway detection (§5) — thresholds, window, action.
- Instrumentation (§6) — metrics, gauges, would-have-cost accounting.
- Composed cost-budget document section (§7).

Plus at least *one* concrete artefact: a config snippet, a pseudocode block, or an IaC skeleton for at least one enforcement primitive (the circuit breaker is the canonical one).

## Acceptance criteria

- [ ] The cost surface (§1) covers all five lines from chapter 03 §1 — or explicitly names the ones that do not apply and why.
- [ ] The budget hierarchy has a named owner at every level, not "the team."
- [ ] At least one enforcement primitive has a concrete config or pseudocode artefact — not just prose.
- [ ] The rate-limit / quota strategy names *where* the enforcement happens (ingress, service, feature) and *why* that layer.
- [ ] The circuit breaker names a specific baseline rate, a specific multiplier, and a specific window — not "when spend gets too high."
- [ ] The kill switch names a specific flag, a specific authorisation policy, and a specific rehearsal cadence.
- [ ] The degradation path names *what the user sees* — not just "we turn the feature off."
- [ ] Runaway detection is distinct from budget cap; both mechanisms exist and are documented separately.
- [ ] Instrumentation names the specific metric names, labels, and backend — not "we emit cost metrics."
- [ ] The composed document is written to be *executed*, not archived — a reviewer can read it once and understand the whole enforcement story.

## Stretch goals

- **Per-tenant fairness under budget stress.** When the budget is depleting, are tenants shed proportionally, or is a specific tier (free vs. paid, small vs. large) protected? Design and justify. Google's [*Fair sharing*](https://sre.google/sre-book/handling-overload/) discussion is a reference.
- **Multi-region / multi-cell budget composition.** If the feature runs in multiple regions or cells, is the budget global or per-region? What happens when one region's cell blows the budget while others are underspending? Design the redistribution or the isolation.
- **Cost-attribution to slices.** Which slice of traffic (region × device × logged-in-state × tenant tier) consumes what fraction of the budget? Extend the instrumentation to answer this. Feed into the reliability review.
- **The reliability-cost curve.** Sketch — with numbers, even estimated — the marginal cost of one more nine of reliability (99.9 % → 99.99 %) for your feature. Google's [*Embracing Risk*](https://sre.google/sre-book/embracing-risk/) chapter is the framing. Justifies why you are *not* over-investing.
- **Feed the paired project.** Extend the SLO document from exercise-01 with this cost-budget section, then attach the combined document to a paired project (e.g., `project-302-llm-augmented-ml-feature`). Two documents, one reliability posture.
- **Cost-runaway gameday scenario.** Write the injection script that would simulate a 20× spend rate for your feature — a runaway retry loop, a token-explosion bug, a cache-miss cascade. Use it in the incident-drill exercise (exercise-03) to exercise the circuit breaker and kill switch.
