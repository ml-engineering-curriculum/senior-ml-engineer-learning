# exercise-03: Variance Reduction with CUPED

**Estimated effort:** 3 hours

## Objective

Implement **CUPED** on a real or synthetic experiment and *measure* the variance reduction against the naive difference-in-means analysis. Rewrite exercise-01's MDE calculation with the CUPED-adjusted σ and quantify the ramp-duration change the reduction buys. Optionally extend to CUPAC and compare the two.

This is the L30 tell that separates "we used the platform's CUPED because it was on by default" from "we can quantify what variance reduction costs and delivers, defend the assumptions, and know when to invest in CUPAC vs. staying with CUPED." An L20 leans on the framework's defaults; an L30 understands what the framework is doing and when it stops helping.

## Prerequisites

- Read chapter 03 (`03-variance-reduction-cuped-cupac.md`) — the CUPED formula, the `1 − ρ²` shrink identity, and the validity conditions are load-bearing.
- Read (or re-skim) chapter 01 (`01-experiment-design-power-mde-and-ramp.md`) — the `n ≈ 16 σ² / Δ²` calculation you will rewrite is here.
- Skim Deng, Xu, Kohavi, Walker's CUPED paper for the original derivation; it is short and worth the read.

## Pick your data

Pick **one**:

- **D1 — Synthetic.** Simulate an experiment with `N = 200 000` users, 50/50 split, and:
  - a pre-experiment covariate `X_i` distributed `Normal(μ, σ_x²)`.
  - an in-experiment outcome `Y_i = β * X_i + τ * T_i + ε_i` with `ε_i` independent noise.
  - Vary `β` (and hence `ρ = Corr(Y, X)`) across three configurations: `ρ ≈ 0.3`, `0.6`, `0.85`. Simulate the treatment effect `τ = +2%` of `mean(Y)`.
- **D2 — Public dataset with a natural pre/post split.** Use a public dataset where you can construct a pre-experiment period and an experiment period on the same units. The Criteo Uplift dataset or a synthetic-A/B test corpus works.
- **D3 — Bring-your-own.** An anonymised experiment log where you have per-user metrics *before* and *during* the experiment. Anonymise IDs; keep only the metric columns.

D1 is recommended: it lets you verify the theoretical `1 − ρ²` shrink identity empirically.

## Steps

### 1. Implement the naive baseline (≈ 20 min)

Write `naive_ate(y, t) -> (estimate, se, ci_low, ci_high)`:

- Estimate = `mean(y[t==1]) − mean(y[t==0])`.
- SE = `sqrt(var(y[t==1]) / n_t + var(y[t==0]) / n_c)`.
- 95 % CI = estimate ± 1.96 * SE.

Run against each of your data variants and report the point estimate, SE, and CI.

### 2. Implement CUPED (≈ 30 min)

Write `cuped_adjust(y, x) -> y_adjusted` following chapter 03 §2:

- Compute `θ = Cov(y, x) / Var(x)` on the *pooled* data.
- Return `y − θ * (x − mean(x))`.

Write `cuped_ate(y, x, t) -> (estimate, se, ci_low, ci_high)`:

- Compute `y_adj = cuped_adjust(y, x)`.
- Run `naive_ate(y_adj, t)`.

Verify empirically that `Var(y_adj) ≈ Var(y) * (1 − Corr(y, x)²)`. Report the observed `ρ` and the observed shrink factor. On D1, the shrink should closely match the theory; a 5 % discrepancy is normal at N = 200 K.

### 3. Measure the SE / CI reduction (≈ 20 min)

For each data variant, produce a small table:

```
scenario       ρ     var_naive    var_cuped    shrink    se_naive    se_cuped    ci_width_reduction
low corr       0.3    ...
mid corr       0.6    ...
high corr      0.85   ...
```

Confirm the empirical shrink matches `1 − ρ²` and the SE reduction matches `sqrt(1 − ρ²)`.

### 4. Rewrite the MDE calculation with the CUPED-adjusted σ (≈ 30 min)

Load exercise-01's `n_per_arm ≈ 16 * σ² / Δ²` calculation. Rewrite it substituting `σ_cuped = σ * sqrt(1 − ρ²)`.

Report:

- The naive `n_per_arm` at your chosen α, power, Δ.
- The CUPED `n_per_arm` at the same α, power, Δ.
- The ratio (naive / CUPED) — this is your sample-size savings.
- The corresponding *ramp-duration* savings at your feature's traffic rate.

For a `ρ = 0.6` metric, expect roughly `1 / (1 − 0.36) ≈ 1.56×` the sample savings — a two-week experiment becomes a nine-day experiment.

### 5. Explore CUPED validity edges (≈ 30 min)

Simulate or find:

- **Missing-covariate case.** A cohort with no pre-experiment data (new users). Implement one of chapter 03 §2's remedies: impute `X = mean(X_existing)` OR run CUPED only on the existing-user sub-population and report the raw metric on the rest. Report the resulting variance reduction on each sub-population.
- **Post-treatment-contaminated covariate.** Deliberately let `X` be measured *during* the experiment on a variable the treatment might influence. Show that the resulting CUPED estimate is biased (the point estimate shifts, not just the SE). This is the "collider" failure mode chapter 03 warns about.

Write a short paragraph on each, in the deliverable.

### 6. (Optional) Extend to CUPAC (≈ 30 min)

If time permits, extend the adjustment from CUPED to CUPAC:

- Fit a small regression `f(X_i) = ŷ_i` where `X_i` is *multiple* pre-experiment covariates (2–5 features): pre-metric, tenure, prior-week engagement, plan tier.
- Use `xgboost`, `sklearn`'s `RandomForestRegressor`, or a linear regression with interactions.
- Fit `f` with out-of-fold predictions to avoid over-fit bias.
- Compute `y_cupac = y − f(X) + mean(f(X))` and run the ATE analysis.
- Compare `Var(y_cupac)` to `Var(y_cuped)`. If your R² on `f` exceeds ρ², CUPAC wins; if not, the linear CUPED was already capturing the signal.

Report the R² of `f`, the empirical shrink, and the SE reduction.

### 7. Write the memo (≈ 20 min)

One page, structured as:

- **Data description** — the source, N, the covariate.
- **Results table** — naive vs. CUPED (and CUPAC if implemented) SE, CI width, shrink, and MDE.
- **Ramp-duration savings** — the calculation from step 4.
- **Validity edges** — the two paragraphs from step 5 (missing covariate, collider).
- **Recommendation** — for a team like yours, at your feature's ρ, which technique to make the default. Cite chapter 03 §2 "start with CUPED; move to CUPAC when the residual variance is expensive enough."

## Deliverable

A folder `cuped-analysis/` (or a Markdown doc with sections inlined) containing:

- `naive_ate.py`, `cuped.py` (and `cupac.py` if implemented) with clean function signatures.
- Notebook or script that reproduces the results table.
- `results.md` — the memo.
- (Optional) `cupac_model.pkl` and a note on how it was fit.

## Acceptance criteria

- [ ] `naive_ate` and `cuped_ate` are implemented with equivalent signatures; both return point estimate, SE, and 95 % CI.
- [ ] `cuped_adjust` computes θ on the pooled data (not per-arm) and returns a mean-shifted adjusted metric.
- [ ] Empirical variance shrink is reported and matches `1 − ρ²` within a few percent on the synthetic case.
- [ ] The MDE calculation is rewritten with `σ_cuped` and the sample-size savings ratio is reported.
- [ ] The ramp-duration savings is stated in wall-clock time at the feature's traffic rate.
- [ ] The missing-covariate edge case is explored with a specific remedy and its cost quantified.
- [ ] The collider edge case is demonstrated: the CUPED estimate on a treatment-contaminated covariate is *biased*, not just less efficient.
- [ ] The memo is at most one page.

## Stretch goals

- **CUPAC comparison.** Full CUPAC implementation and side-by-side comparison of variance reduction and pipeline cost. Report the fitted R² and reason about whether the ~5–10 % additional shrink is worth the pipeline debt.
- **Stratification + CUPED composition.** Stratify by an axis (country, platform, tenure bucket), estimate the within-stratum treatment effect, and combine with CUPED. Report the additional shrink beyond CUPED alone.
- **Winsorisation.** For a heavy-tailed metric (simulate a Pareto-tailed outcome), winsorise at the 99th percentile and re-run CUPED. Report the variance-reduction contribution of each step.
- **Delta-method ratio metric.** Choose a ratio metric (e.g., click-through rate = clicks / impressions), implement the delta-method estimator per Deng, Knoblich, Lu, and compare to the naive per-user-ratio t-test. Report the SE difference.
- **Plug back to exercise-01.** Update exercise-01's experiment plan with the CUPED-adjusted σ and rewrite the ramp plan to reflect the shorter horizon.
- **Regression tests on the adjustment.** Author a test suite for the CUPED implementation — a synthetic input with known θ should produce a known output — following the mod-305 chapter 04 ML Test Score discipline.
