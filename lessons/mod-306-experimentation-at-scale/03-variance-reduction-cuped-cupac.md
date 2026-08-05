# Variance reduction: CUPED, CUPAC, and stratification

## Motivation

Chapter 01 gave the power formula. In its most common form the sample size an experiment needs to detect an effect of size Δ at α = 0.05, power = 0.8 is roughly `n_per_arm ≈ 16 σ² / Δ²`. Two of the three terms are fixed by the world: Δ is the size of the effect you care about (a business decision); α and power are conventions the org agreed on. **σ is the one term you can shrink.**

Every 1 % reduction in σ² translates to a 1 % reduction in the sample size an experiment needs — or, at the same sample size, to a 1 % reduction in the MDE the experiment can detect. Real variance-reduction techniques deliver reductions of **30 %–70 %** on high-variance metrics like revenue-per-user, session length, or engagement time, which is the difference between an experiment that needs a month of traffic and one that reads in a week. Deng, Xu, Kohavi, Walker's [*Improving the sensitivity of online controlled experiments by utilizing pre-experiment data*](https://exp-platform.com/Documents/2013-02-CUPED-ImprovingSensitivityOfControlledExperiments.pdf) (WSDM 2013) — the paper that introduced **CUPED** — reports variance reductions of 45 %–50 % on Bing metrics. Xie and Aurisset's [*Improving the sensitivity of online controlled experiments: Case studies at Netflix*](https://netflixtechblog.com/reducing-variance-in-online-experiments-by-utilizing-post-experiment-data-4c60ea11b025) reports similar magnitudes.

This chapter is the senior ML engineer's working knowledge of variance reduction — what it is, when it applies, what it costs, and where the two most common industrial techniques (CUPED and its regression-adjustment cousin CUPAC) sit in the design. Chapter 04 covers *sequential testing* — a different framing that trades statistical form (fixed-horizon vs. always-valid) for the ability to peek. This chapter and chapter 04 are the two levers a senior ML engineer reaches for when a naive design is under-powered.

## §1 — Why variance is the lever

The randomisation controls for *systematic* bias between arms. What it does not control for is *user-level heterogeneity within* an arm. Two users in the treatment arm can have wildly different values of the metric — one is a heavy spender, one is a lurker — and the wildness is *unrelated to the treatment*. All the metric's variance is passed through to the estimated treatment effect's standard error.

The core insight of variance reduction is that a *lot* of that user-level variance is *predictable from pre-experiment data*. A user who spent $500 in the month before the experiment is likely to spend $500 in the month during the experiment — with or without the treatment. If we could *subtract off* the predictable part, what remained would be the noise plus the treatment signal — a much cleaner ratio.

The formal statement: variance reduction is a *covariate adjustment*. Any covariate `X` that (a) predicts the metric `Y` and (b) is *independent of the treatment assignment* — which any pre-experiment covariate is, by construction, if randomisation is honest — can be used to shrink the variance of the treatment-effect estimate.

## §2 — CUPED: Controlled experiments Using Pre-Experiment Data

### The method

CUPED starts from the outcome metric `Y_i` (the value for user *i* in the experiment window) and a *pre-experiment covariate* `X_i` (the same metric, or a related one, measured on user *i* before the experiment started). The **CUPED-adjusted metric** is:

```
Y_i_cuped = Y_i − θ * (X_i − mean(X))
```

where `θ = Cov(Y, X) / Var(X)` — the OLS slope of `Y` regressed on `X` on the pooled data. The mean shift keeps the adjusted metric's expectation equal to `Y`'s expectation, so the *estimated treatment effect* on `Y_cuped` is the same in expectation as on `Y`. What changes is the *variance*:

```
Var(Y_cuped) = Var(Y) * (1 − ρ²)
```

where `ρ = Corr(Y, X)`. If the pre-experiment covariate correlates with the outcome at `ρ = 0.7`, CUPED shrinks the variance by `1 − 0.49 = 51 %`. This is the source of the 45 %–50 % reductions Deng et al. reported: at Bing scale, the previous-period value of a metric is heavily correlated with the current-period value.

### The recipe

- **Pick the covariate `X`.** The canonical choice is the *same metric* measured on the same user over the pre-experiment period (usually the four weeks before the experiment starts). Other choices — user tenure, prior engagement, prior spend — work if they correlate with the outcome.
- **Compute `θ` on the pooled data** (control + treatment). Using the pooled slope keeps `θ` from being biased by the treatment.
- **Compute `Y_cuped` for every user, in both arms.** Then run the *same* difference-in-means analysis on `Y_cuped` that you would have run on `Y`. The reported point estimate is unbiased; the reported CI is narrower.

A minimal worked example in Python:

```python
import numpy as np

def cuped(y, x):
    """Return the CUPED-adjusted outcome vector.

    y: array of shape (n,) — the metric observed during the experiment window
    x: array of shape (n,) — the same metric (or a related covariate) on the
       pre-experiment window, pooled across arms
    """
    theta = np.cov(y, x, ddof=1)[0, 1] / np.var(x, ddof=1)
    return y - theta * (x - x.mean())

# In practice, `x` is the pre-experiment metric per unit; the exact function
# depends on the metric (mean spend, session length, etc.). The estimator
# lift on cuped-adjusted y has variance approximately Var(y) * (1 - Corr(y, x)^2).
```

### Where CUPED is valid

- **`X` must be measured *before* random assignment**, or at least be logically independent of the treatment. A covariate influenced by the treatment (a "collider") introduces bias.
- **The covariate is the same across arms.** No arm-specific transformations of `X` before computing `θ`.
- **`θ` is estimated on pooled data**, not per-arm.
- **The randomisation unit and the analysis unit are the same**, or the covariate is aggregated to the randomisation unit before the adjustment. (If you randomise per user but adjust per session, you re-introduce the mismatch chapter 01 §4 warned about.)
- **New users have no pre-experiment data.** For new users, `X` is missing; either impute (a defensible choice is `X = mean(X_existing_users)`, which reduces `θ`'s effectiveness on the missing subpopulation to zero but does not bias the estimate) or run CUPED only on the sub-population where `X` exists and report the raw metric on the rest. Deng et al. describe both.

### Where CUPED is not enough

- **A user with no pre-experiment history contributes noise CUPED cannot remove.** For a product with a large new-user cohort (an onboarding experiment), CUPED's variance reduction is smaller than on a returning-user base.
- **When the treatment changes the *shape* of the metric** (not the mean but the tail), a linear covariate adjustment does not remove the tail-driven variance. Winsorisation (capping the metric at, say, the 99th percentile) is the complementary lever; Kohavi/Tang/Xu chapter 22 discusses combining winsorisation with CUPED.
- **When the covariate is not scalar or linear.** A pre-experiment metric that predicts `Y` non-linearly is not fully captured by a linear `θ`. The generalisation is CUPAC (§3).

## §3 — CUPAC: regression-adjustment with multiple predictors

CUPED uses one covariate and a linear slope. CUPAC ("CUPED with Additional Covariates," popularised in industry by DoorDash's engineering write-ups) generalises to *any regression*: predict `Y` from a set of pre-experiment covariates using a model (linear regression, gradient-boosted trees, a neural net — anything), and subtract the *model's prediction* from `Y`:

```
Y_cupac_i = Y_i − f(X_i) + mean(f(X))
```

where `f` is a model fit on *pre-experiment* data (or on pooled data with the treatment as one of the covariates, taking care to avoid regularisation bias — see below). The variance reduction becomes:

```
Var(Y_cupac) = Var(Y) * (1 − R²)
```

where `R²` is the out-of-sample R² of `f` predicting `Y`. Any predictor that lifts `R²` above `ρ²` (CUPED's single-covariate `R²`) improves CUPED.

Poyarkov, Drutsa, Khalyavin, Gusev, Serdyukov's [*Boosted Decision Tree Regression Adjustment for Variance Reduction in Online Controlled Experiments*](https://research.yandex.com/publications/121) (KDD 2016) is the canonical academic reference for the boosted-tree variant; Doordash's [*Improving Experimental Power Through Control Using Predictions as Covariate*](https://doordash.engineering/2020/06/08/improving-experimental-power-through-control-using-predictions-as-covariate-cupac/) is the widely-cited industry write-up under the CUPAC name.

### The recipe

- **Freeze pre-experiment data.** Snapshot every covariate at experiment-start time; anything computed later can leak treatment-side information.
- **Fit `f` on pre-experiment data**, or fit on the pooled in-experiment data with the treatment indicator included as a covariate — Chernozhukov et al.'s [*Double/Debiased Machine Learning for Treatment and Structural Parameters*](https://arxiv.org/abs/1608.00060) is the reference for how to do the latter without regularisation bias.
- **Compute `f(X_i)` for every user in the experiment**, in both arms.
- **Analyse the adjusted metric.** Difference-in-means on `Y − f(X)` with a standard error that accounts for the estimation of `f` (out-of-fold predictions to avoid the "the model saw this user in training" bias).

### Why CUPAC over CUPED

- **Higher variance reduction when non-linear predictors are available.** Boosted trees on 20 pre-experiment features can lift `R²` from 0.4 (CUPED's linear one-covariate) to 0.7+, which shrinks `1 − R²` correspondingly.
- **Handles heterogeneous populations.** A single θ is a single slope; a tree ensemble can learn different slopes per sub-population.

### Why sometimes CUPED anyway

- **Operational cost.** CUPED is a one-line adjustment. CUPAC is a training pipeline that has to be maintained, monitored, and re-fit as the covariate distribution drifts.
- **Auditability.** Regulated deployments (mod-309) may prefer the *transparent* linear adjustment over the black-box predictor.
- **Diminishing returns.** If a single-covariate CUPED already gives you 45 %, moving to 55 % via CUPAC is often not worth the pipeline debt.

The senior read: start with CUPED. Move to CUPAC only when the residual variance is expensive enough to justify the pipeline, and when the pipeline can be *tested* (the fitted `f` needs its own regression tests — mod-305 chapter 04's ML Test Score applies).

## §4 — Stratification and post-stratification

A complementary variance-reduction lever is **stratification**: partition the population into strata (by country, device, plan tier, tenure bucket) *before* randomisation, and randomise separately inside each stratum. The treatment-effect estimate is a weighted average of within-stratum effects, and the variance is smaller because within-stratum users are more homogeneous.

Two flavours:

- **Stratified randomisation.** Randomise inside each stratum before the experiment starts. Requires the strata to be knowable at assignment time.
- **Post-stratification.** Analyse *as if* you had stratified — compute the within-stratum effect and re-weight — after randomisation. Cheaper (no design change) but slightly less efficient when strata are large.

Miratrix, Sekhon, Yu's [*Adjusting Treatment Effect Estimates by Post-Stratification in Randomized Experiments*](https://arxiv.org/abs/1109.6402) is the modern statistical treatment. In experimentation platforms, stratified randomisation is common on major axes (country, platform) and post-stratification is common everywhere else.

CUPED and stratification compose. A CUPED-adjusted metric analysed with post-stratification typically outperforms either alone, especially when the covariate correlates strongly with the outcome only *within* a stratum.

## §5 — Winsorisation and trimming (variance from the tail)

Many product metrics have long tails — revenue, session time, watch time — where a single user's outlier value dominates the mean. Trimming or capping the metric reduces the variance the analysis has to fight through. Two common shapes:

- **Winsorisation.** Cap the metric at the pth percentile (usually 99 or 99.5) on the *pooled* pre-experiment distribution. A user whose true spend is $50 000 contributes $500 (the 99.5th percentile cap) to the estimator.
- **Trimming.** Drop the top p % outright. Statistically cleaner in some senses; loses the tail information entirely.

Both are done *before* the treatment-effect calculation and are applied identically in both arms. Kohavi/Tang/Xu chapter 22 discusses the trade-off. The rule: **cap on the pooled pre-experiment distribution, not on the in-experiment observations**, or you re-introduce a treatment-dependent cap that biases the estimator.

Winsorisation and CUPED compose: winsorise first, then CUPED on the winsorised metric. In heavy-tail domains (marketplace GMV, ad revenue) the combined reduction can exceed either alone.

## §6 — Delta method for ratio metrics

Some primary metrics are *ratios* — click-through rate = clicks / impressions, revenue-per-session = revenue / sessions. If the numerator and denominator are both random per user, the ratio's variance is not the sum of the pieces. The **delta method** gives the correct asymptotic variance:

```
Var(N / D) ≈ (μ_N / μ_D)² * ( Var(N) / μ_N² − 2 * Cov(N, D) / (μ_N * μ_D) + Var(D) / μ_D² )
```

Every mature experimentation platform implements the delta method for ratio metrics. Deng, Knoblich, Lu's [*Applying the Delta Method in Metric Analytics: A Practical Guide with Novel Ideas*](https://exp-platform.com/Documents/2018KDDDeltaMethod.pdf) is the industrial reference.

The mistake to avoid: computing per-user ratio (`click_rate_per_user = clicks_i / impressions_i`) and running t-test on it. This under-uses the data (users with 1 impression are weighted equally to users with 1000) and biases the estimate when the denominator is heteroskedastic. Delta method on `N` and `D` at the arm level is the correct estimator.

## §7 — What variance reduction does not do

Variance reduction shrinks the *standard error*, which shrinks the *confidence interval*, which shrinks the MDE. It does **not**:

- Fix a broken randomisation (SRM — chapter 02). A CUPED-adjusted metric on an SRM-poisoned experiment is still an invalid experiment.
- Fix interference (chapter 02 §2). A cluster-randomisation design plus CUPED at the cluster level is the composed pattern; CUPED on a user-level analysis of a marketplace experiment does not fix the SUTVA violation.
- Justify peeking at the metric before the experiment ends. That is a *sequential-testing* question, covered in chapter 04.
- Change the *sign* or *bias* of the treatment-effect estimate. It only shrinks the SE around the estimate.

Variance reduction is a *multiplier* on a *sound* design. Chapters 01 and 02 make the design sound; this chapter makes it sensitive.

## §8 — What to wire into the platform

- **CUPED, by default, on every metric with a pre-experiment analogue.** The platform stores a rolling window of per-user metrics for exactly this reason.
- **Post-stratification on the standard axes** (country, platform, tenure bucket) with no analyst action.
- **Winsorisation on tail-heavy metrics** at a pre-registered percentile.
- **Delta-method SEs on every ratio metric**, with the analyst using the platform's helper — not rolling their own t-test on per-user ratios.
- **Regression tests on the adjustment pipeline.** A CUPED implementation whose θ estimation drifts (a bug in the pre-experiment aggregation) silently inflates or deflates the reported CI. The pipeline needs the same test discipline the offline harness has (mod-305 chapter 04).
- **A pre-experiment simulation.** For the design step in chapter 01, the MDE calculator should let the analyst say "assume 40 % variance reduction from CUPED" and see the resulting sample size — so the ramp plan reflects the sensitivity the experiment will actually have.

## §9 — Where this fits with the rest of the module

- **Chapter 01 (design).** The MDE calculation uses `σ_adjusted` — the CUPED-adjusted metric's SD, not the raw SD. Variance reduction feeds the design as an input assumption.
- **Chapter 02 (trust).** Variance reduction *does not* fix trust failures. SRM still blocks the read.
- **Chapter 04 (sequential testing).** A sequential test with a CUPED-adjusted metric has a smaller MDE at any given horizon. The two levers compose; both address the same "the experiment is under-powered" problem from different angles.
- **Chapter 05 (causal inference).** When A/B is impossible, the observational analogues (regression adjustment, propensity-score weighting) are *related in spirit* to CUPAC — regression on covariates to remove predictable variation before estimating an effect. The difference is that in observational settings, unmeasured confounders bias the estimate; in A/B, randomisation guarantees unbiasedness and covariate adjustment only shrinks the SE.

## Summary

Variance reduction is the lever that turns an under-powered experiment into a readable one without changing the design's other assumptions. CUPED — subtract off `θ * (X − mean(X))` where `X` is a pre-experiment covariate and `θ` is the pooled OLS slope — routinely halves variance on the metrics that matter most. CUPAC generalises CUPED to arbitrary regression models, trading pipeline debt for larger variance reductions. Stratification and post-stratification compose with CUPED; winsorisation compounds with both on tail-heavy metrics; the delta method is the correct estimator for ratio metrics. Every technique in this chapter is a multiplier on a *sound* design — it shrinks the CI on a trustworthy estimator but does not fix a broken randomisation, an SUTVA violation, or a novelty effect. Chapter 04 is the other big lever: not "reduce the variance" but "change the statistical framing so peeking is valid."
