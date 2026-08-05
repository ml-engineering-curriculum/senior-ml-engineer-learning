# exercise-04: Causal Inference When A/B is Impossible

**Estimated effort:** 3 hours

## Objective

For a scenario where an A/B test is *not* available, pick the appropriate observational causal-inference method — propensity score matching / IPW, difference-in-differences, instrumental variables, or regression discontinuity — spec the estimator + diagnostics, and write a release memo that names the estimand, the estimator, the identifying assumption, and the sensitivity analysis. The point of the exercise is *not* to run the shipping decision on observational data at the same confidence level as an A/B — it is to prove you can pick the right tool, defend the assumption load, and name what the framing costs.

This is the L30 tell that separates "we can't A/B this so we can't measure it" from "we can't A/B this, so here is the observational estimate with an explicit assumption load and the shape of decision it justifies." An L20 gives up when randomisation is unavailable; an L30 knows which fallback to reach for and how far to trust it.

## Prerequisites

- Read chapter 05 (`05-causal-inference-when-ab-is-impossible.md`) — the four methods (PSM, DID, IV, RDD), the assumption load each carries, and the "rescue" options *before* reaching for causal inference.
- Skim Cunningham's [*Causal Inference: The Mixtape*](https://mixtape.scunning.com/) chapters on PSM and DID for the applied statistical treatment.
- Skim [mod-309](../../mod-309-responsible-ai-governance/) if your scenario is a regulator- or fairness-visible decision.

## Pick your scenario

Pick **one**:

- **S1 — Pricing / plan change.** Product wants to launch a new pricing tier in Germany on Tuesday and everywhere else in three months. Executive decided; no randomisation is possible. The question: what is the causal effect on conversion, revenue-per-user, and churn? (DID / synthetic control natural fit.)
- **S2 — Fair-lending model change.** A credit-underwriting model changes its decision threshold. Regulator will not allow randomising which applicants get the lower-threshold model in production. Historical data has the old and new thresholds applied on comparable applicant populations. The question: does the new threshold cause disparate impact on protected classes? (PSM / IPW on matched cohorts + regulator-visible sensitivity analysis.)
- **S3 — Model-triggered intervention (fraud review).** A fraud model above score 0.7 triggers a manual review; below does not. The question: does the manual review cause a reduction in downstream chargebacks vs. no review? (RDD around the 0.7 threshold.)
- **S4 — Encouragement design (feature-adoption email).** The team sent an educational email to a randomly-chosen half of eligible users about a new feature. Only some users read the email and try the feature. The question: what is the causal effect of *using* the feature (not just *receiving the email*) on retention? (IV — assignment to email is the instrument for feature use.)
- **S5 — Bring-your-own.** A real scenario at your team where A/B is not available. Preamble with why (regulatory, operational, historical) and the data you have access to.

## Steps

### 1. First: interrogate whether A/B is *really* impossible (≈ 20 min)

Chapter 05 §8 lists three rescues that sometimes turn an "impossible" A/B into a possible one:

- Randomise the *decision*, not the *treatment*.
- Randomise the *rollout order* across markets.
- Randomise the *default* and use IV on the encouragement.

For your scenario, walk each rescue and write one paragraph on whether it is viable. If a rescue is viable, note that the observational analysis in the rest of the exercise is a *fallback* if the rescue is refused. This step is deliberately first: a senior ML engineer's default is to push back on "impossible" once before accepting an observational-only analysis.

### 2. State the estimand in potential-outcomes notation (≈ 20 min)

Write the estimand explicitly. Examples:

- ATE — `E[Y(1) − Y(0)]` — the average effect if everyone got the treatment.
- ATT — `E[Y(1) − Y(0) | T = 1]` — the average effect on the actually-treated.
- CACE / LATE — the effect on the compliers (relevant for IV / encouragement designs).

Name the population, the treatment indicator, and the outcome. If your outcome is a rate, name numerator and denominator. If your outcome is over a time window, name the window.

### 3. Pick a method and defend the choice (≈ 30 min)

For your scenario, pick one of {PSM, DID, IV, RDD, synthetic control}. Write two paragraphs:

- **Why this method fits.** What is the identifying assumption in your setting, and what makes it *plausible* (a specific institutional or domain fact, not "usually reasonable")?
- **Why not the others.** Rule out at least two of the alternatives with one sentence each.

If the fit is not clean — e.g., you would like DID but your two comparison groups have visibly diverging pre-trends — say so and either (a) switch to synthetic control, or (b) accept that the observational estimate carries an assumption you know is stretched and adjust your recommendation accordingly.

### 4. Spec the estimator (≈ 30 min)

Write the estimator step-by-step in a way an ML engineer could code from. Include:

- **The data query.** What columns, what filters, what time window, what unit of analysis?
- **The pre-processing.** Any winsorisation, feature construction, or covariate selection.
- **The estimator formula.** For PSM/IPW: the propensity model, the matching or weighting scheme. For DID: the two-way fixed-effects regression (or the Callaway–Sant'Anna staggered variant if the rollout is staggered). For IV: the first stage, the second stage, the exact 2SLS setup. For RDD: the running variable, the bandwidth choice (Imbens–Kalyanaraman is a defensible default), the local-linear regression.
- **The SE and CI.** Cluster-robust SEs where the analysis unit is at the individual level but assignment is at the group level; bootstrap for PSM / matched analyses.

### 5. Spec the diagnostics (≈ 30 min)

Every method has a diagnostic; every diagnostic is *mandatory*. For your chosen method:

- **PSM / IPW** — covariate-balance table before and after matching (standardised mean differences per covariate, all below 0.10). Overlap / common-support plot. E-value or Rosenbaum bounds sensitivity analysis.
- **DID** — pre-treatment parallel-trends plot with multiple pre-periods. Placebo test on a fake pre-treatment date. Multiple control-group comparison.
- **IV** — first-stage F-statistic (target > 10). Exclusion-restriction argument (a paragraph). If multiple instruments, Sargan-Hansen overidentification test.
- **RDD** — McCrary density test on the running variable. Covariate-continuity checks at the threshold. Bandwidth-sensitivity report at 2–3 bandwidths.
- **Synthetic control** — pre-treatment fit quality (RMSPE ratio pre vs. post). Placebo tests on donor units.

If any diagnostic fails, the estimate is not shippable *at the strength of an A/B result*. State how you would communicate a partial or fragile result.

### 6. Write the release memo (≈ 30 min)

One to two pages, structured as:

- **Context.** The decision the memo informs. Why A/B was not available (and why the rescue paths from step 1 do not apply, if they don't).
- **Estimand.** The precise causal quantity, in potential-outcomes notation and in plain English.
- **Estimator.** The method, the data, the formula, the SEs.
- **Assumption load.** The identifying assumption, spelled out; the plausibility argument; a paragraph naming *what would have to be true* in the world for the estimate to be trustworthy.
- **Diagnostics.** The mandatory-check results.
- **Sensitivity.** The E-value / Rosenbaum bounds / placebo-test result. The recommendation is *bounded* by the sensitivity — an effect that vanishes under mild unmeasured confounding is not the same evidence as one that survives strong sensitivity.
- **Recommendation.** What decision the memo supports. Explicitly *weaker* than an A/B — e.g., "the observational estimate is consistent with a positive effect of size Δ ± ε, but the identification assumption cannot be tested, so we recommend the launch proceed if operational risk is otherwise acceptable; a 90-day post-launch holdback measurement would tighten the evidence."

## Deliverable

A folder `causal-analysis/` containing:

- `release-memo.md` — the memo (one to two pages).
- `estimator.py` (or pseudocode) — the estimator with the diagnostic functions.
- `diagnostics.md` — the rendered diagnostic outputs (tables, plots described in text if not rendered).
- (If you actually ran the analysis on data) A notebook or script that reproduces the numbers.

## Acceptance criteria

- [ ] Step 1's interrogation names at least one rescue path and either uses it or defends why it is refused.
- [ ] The estimand is stated in potential-outcomes notation *and* in plain English, with the population, treatment, and outcome named.
- [ ] The method choice is defended against at least two alternatives.
- [ ] The estimator spec is concrete enough that an ML engineer could code from it — data query, formula, SE choice.
- [ ] The mandatory diagnostics for the chosen method are specified with numerical thresholds (e.g., "standardised mean difference below 0.10 for every covariate," "first-stage F > 10").
- [ ] A sensitivity analysis is named — E-value, Rosenbaum bounds, placebo test, bandwidth sweep, whatever fits the method.
- [ ] The recommendation is explicitly weaker than an A/B result and names the assumption load.
- [ ] The memo is at most two pages.

## Stretch goals

- **Run the analysis on real data.** For S1 or S2, pull anonymised historical data and produce the point estimate, CI, and diagnostic tables. Report the assumption load with numbers rather than just prose.
- **Staggered DID.** If your scenario is a staggered rollout, implement both the classical two-way fixed-effects DID and the Callaway–Sant'Anna estimator, compare the point estimates, and comment on the direction and magnitude of the bias TWFE introduces.
- **Synthetic control comparison.** For S1, produce both a DID estimate and a synthetic-control estimate. Report the pre-treatment fit RMSPE ratio for the synthetic control and comment on the consistency.
- **Fairness disaggregation.** For S2, run the estimator on protected-class sub-populations. Present a fairness-disaggregation table alongside the aggregate estimate; hand off to mod-309 for the review packet.
- **Handoff to mod-309.** For S2, write the "regulator-visible summary" section of the memo — a one-paragraph statement a fair-lending review can consume without a causal-inference primer.
- **Post-launch measurement plan.** Author a 30-day / 90-day / 180-day post-launch measurement plan. For each measurement, name the estimator, the data cut, and the update that will be published. This is where an observational-initial + holdback-follow-up strategy becomes a defensible release path.
