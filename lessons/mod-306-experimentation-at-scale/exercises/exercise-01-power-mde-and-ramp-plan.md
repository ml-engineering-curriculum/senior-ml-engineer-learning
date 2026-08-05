# exercise-01: Power, MDE, and Ramp Plan

**Estimated effort:** 3 hours

## Objective

Design the **seven-piece experiment contract** for a specific candidate model, at senior altitude. The contract is a memo — hypothesis, primary metric, guardrail set, power / MDE calculation, randomisation unit and traffic split, ramp plan with predefined step gates, and a written stopping rule — such that an on-call could execute the ramp and a reviewer six months later could reconstruct the ship / don't-ship decision. This exercise is where "we ran an A/B" becomes "we ran an A/B whose design would have been signed by a peer L30 *before* the treatment existed."

This is the L30 tell that separates "the experiment was inconclusive" from "the experiment was under-powered by design." An L20 reports a p-value; an L30 reports a decision under a contract everyone agreed to in advance.

## Prerequisites

- Read chapter 01 (`01-experiment-design-power-mde-and-ramp.md`) — the seven-piece contract, the `n ≈ 16 σ² / Δ²` back-of-envelope, and the three interpretations of a flat result are load-bearing.
- Skim chapter 02 (`02-trust-diagnostics-srm-interference-novelty.md`) — the ramp step gates in this exercise depend on SRM and cross-arm parity checks from chapter 02.
- Skim [mod-305 chapter 03](../../mod-305-advanced-evaluation/03-shadow-and-canary-online-promotion.md) — this exercise is the A/B stage of that promotion path.
- Skim [mod-307](../../mod-307-ml-reliability-slos/) for the SLI vocabulary the guardrails should name.

## Pick your candidate

Pick **one** of the following, or bring your own (F5, with a one-paragraph "context I already know" preamble):

- **F1 — Search-ranker replacement.** New two-tower retriever + gradient-boosted reranker replacing lexical BM25 + hand-tuned rules on a shopping-site search. Traffic ~1 M queries/day. Offline NDCG@10 lift is 0.03 with a CI clear of zero; the harness gate is green.
- **F2 — LLM-augmented ticket triage classifier.** Fine-tuned smaller LLM replacing a rule-based triage layer for support-ticket routing. ~5 K tickets/day. Offline macro-F1 up 3 points; latency budget met offline.
- **F3 — Recommender re-ranker on the homepage feed.** Neural re-ranker on top of the existing candidate generator. ~10 M sessions/day. Offline offline-metric (nDCG or MRR) shows a small lift; business owner cares about click-through and time-to-first-click, but the *ship criterion* is 7-day-retained-active-users.
- **F4 — Fraud-detection threshold change.** New calibration (mod-303 chapter 05) shifts the decision threshold on an existing fraud model. Not a model swap — a *threshold* swap. Business cares about false-positive rate on legitimate purchases while holding fraud catch-rate constant.
- **F5 — Bring-your-own.** A production or planned experiment on a candidate you have access to. Anonymise anything sensitive; add a preamble covering traffic scale, baseline metric level, and what the release conversation is about.

## Steps

### 1. Write the one-sentence hypothesis (≈ 15 min)

Draft the hypothesis so a product owner and a peer engineer can both agree or disagree with it. Name the population, the direction, the size, and the boundary conditions. Rewrite it twice before you keep it. If the hypothesis has "and" or "or" in it, split into two experiments or narrow one down.

### 2. Choose the primary and guardrail metrics (≈ 30 min)

- **Primary.** One metric, matched to the downstream decision. State *why* — cite chapter 01 §1's three properties (tied to outcome, measurable within window, sensitive). If the outcome you care about is 30-day retention and the experiment window is two weeks, name the proxy you will use and cite the historical correlation between proxy and long-horizon outcome (or acknowledge you do not have that history and what that costs).
- **Guardrails.** A stable set of four to seven, each with:
  - Metric name and definition (a function in the repo, not a query).
  - Threshold direction (cap on regression, floor on non-inferiority).
  - Numerical threshold — the actual number, not "should not regress much."
  - Source of the threshold — mod-307 SLO, mod-304 chapter 04 cost envelope, product agreement, regulator floor.
  - What happens if the guardrail fires — auto-rollback vs. page + human decide.

### 3. Compute the power / MDE (≈ 45 min)

Pick two of the four (α, power, Δ, n) as fixed and solve for the other two.

- **Design mode.** Given α = 0.05 two-sided, power = 0.8, and Δ as the smallest business-relevant effect, solve for `n_per_arm`. Report the answer, the assumed σ, and the source of σ (a historical query on the metric on the current production system on comparable users). If you use a proportion metric, use `σ² = p(1 − p)`.
- **Sanity mode.** Given the traffic available in the ramp horizon you plan to run (e.g., 4 weeks × 50 % of daily traffic per arm), what is the *achievable* MDE? Report it and compare to Δ.

Include a worked calculation in the memo — the numbers, the formula, the answer — not just the result. `statsmodels.stats.power` or a Bayesian equivalent is fine; a hand calculation is also fine if the arithmetic is shown.

If the achievable MDE is larger than Δ, name the three options:

- Extend the ramp horizon.
- Increase the split fraction toward 50/50 (if not already).
- Reduce σ via chapter 03 (variance reduction) — write down what the CUPED-adjusted σ would need to be to bring the achievable MDE below Δ; that becomes the input to exercise-03.

### 4. Randomisation unit and traffic split (≈ 20 min)

- **Randomisation unit.** Per-request, per-session, per-user, per-account, per-cluster? Defend the choice in a paragraph against the alternative unit above it in the hierarchy (chapter 01 §4). If the analysis unit is finer than the randomisation unit (per-session metric with per-user randomisation), note the delta-method / per-user-aggregation adjustment.
- **Traffic split.** 50/50 unless there is a specific safety reason to use a skewed split. If you pick 90/10 or 95/5, defend against the ~4× loss in power (Kohavi/Tang/Xu chapter 15) and state whether the resulting achievable MDE is still below Δ.

### 5. Ramp plan (≈ 30 min)

Author the ramp as a table with columns *step, percentage, duration, advance criterion, rollback criterion, on-call action*. At least four steps. Cover:

- **1 %** — trust step. SRM + guardrail sanity. No metric-direction read.
- **5–10 %** — guardrail hold. First metric-direction glance (informational, not decisional).
- **25–50 %** — expansion step. Cross-a-full-business-cycle exposure.
- **Full-power step (usually 50 %)** — runs to the sample size the power calculation demanded. This is the step whose stopping rule §6 governs.

Every step's *advance criterion* refers to specific numbers on specific SLIs; every step's *rollback criterion* has a threshold and a hold-time. The 50 %-to-100 % transition is out of scope for this exercise (it is a rollout decision after the experiment concludes).

### 6. Stopping rule and decision matrix (≈ 20 min)

Write the stopping rule *before* the experiment starts. A worked matrix (adapted from chapter 01 §6):

- **Ship** — primary CI above zero at α; every guardrail non-inferior; every mod-305 slice non-inferior.
- **Don't ship** — primary CI includes zero AND upper bound below the MDE (true null); OR any guardrail red.
- **Iterate** — primary CI includes zero AND upper bound above the MDE (under-powered). Extend, redesign, or reduce σ.
- **Rollback and investigate** — a trust diagnostic (chapter 02) fires; experiment is invalid until explained.

Name the decision maker and the review meeting on the calendar. "The eng team reads it on Slack" is not a decision process.

### 7. Failure-mode rehearsal (≈ 10 min)

Two paragraphs — one for each of chapter 01's named failure modes: metric-shopping and under-powered "flat result." For each, state where in *your* contract the discipline sits that prevents the mode from being a live risk.

## Deliverable

A single Markdown memo, one to two pages, titled `experiment-plan-<candidate>.md`, structured as:

- Header — candidate ID, baseline ID, target start date, author, stakeholders.
- Hypothesis (one sentence).
- Primary metric + guardrail table.
- Power / MDE calculation (worked).
- Randomisation unit and traffic split.
- Ramp plan (as a table).
- Stopping rule and decision matrix.
- Failure-mode rehearsal.

Two pages hard cap. Reviewers hate experiment plans they cannot read in ten minutes.

## Acceptance criteria

- [ ] The hypothesis is one sentence, testable, and names population + direction + size + boundary conditions.
- [ ] The primary metric is a single metric, matched to the downstream decision, with a stated *why*.
- [ ] The guardrail table lists at least four metrics, each with a definition, a threshold direction, a numerical threshold, and a source.
- [ ] The power / MDE calculation is shown numerically — Δ, α, power, σ, `n_per_arm` — with the source of σ named.
- [ ] The randomisation unit is at least as coarse as the analysis unit, or the delta-method / aggregation fix is named.
- [ ] The ramp plan has at least four named steps with percentages, durations, and per-step advance + rollback criteria.
- [ ] The stopping rule distinguishes *ship / don't-ship / iterate / rollback* and is written before observed data.
- [ ] The memo names two failure modes and the contract element that prevents each.
- [ ] Nothing in the memo picks a metric "because it lifted" — chapter 01 §1's "metric chosen before the candidate exists" rule is respected.
- [ ] The memo is at most two pages.

## Stretch goals

- **Sequential-testing variant.** Rewrite the stopping rule under a Group Sequential (Pocock or O'Brien-Fleming) or mSPRT framing from chapter 04. Compare the achievable-MDE-at-horizon and the peeking cost.
- **CUPED plug-in.** Reference exercise-03: state the CUPED-adjusted σ the experiment would use, and rewrite the MDE / `n_per_arm` calculation with the reduced σ. Compare the ramp duration change.
- **Feed the paired project.** Attach this contract as the deployment-experimentation section of [`project-302-llm-augmented-ml-feature`](../../../projects/project-302-llm-augmented-ml-feature/) or another paired project. The plan is meant to be executed, not archived.
- **Guardrail SLI cross-link.** For every guardrail, cite the specific mod-307 SLI it maps to. Any guardrail that does not map to an SLI is a candidate to remove or a candidate SLI to author.
- **Peer review.** Trade plans with a peer. Reviewer walks the seven-piece contract and lists which pieces are load-bearing vs. under-specified. Gaps become follow-up work.
