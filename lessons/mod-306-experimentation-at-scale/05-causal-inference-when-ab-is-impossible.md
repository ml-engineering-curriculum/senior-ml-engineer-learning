# Causal inference when an A/B test is impossible

## Motivation

Chapters 01–04 assumed the experiment could be run. Randomise, ramp, monitor, decide. The whole module's power comes from the *randomisation*: assign users to arms independently of anything about them, and the difference in means is an unbiased estimate of the causal effect.

Sometimes randomisation is unavailable. The three shapes that recur across ML teams:

- **Ethical or regulatory prohibitions.** Randomising which loan applicants get a lower rate is a disparate-impact question a regulator will not sign; randomising which patients receive a treatment recommendation is a clinical-trial question with its own governance regime.
- **Operational impossibility.** A pricing change, a policy change, a currency change, a legal ToS change — you cannot expose one arm of users to different prices at scale without market-integrity concerns. A geographic rollout — the feature launches in Germany on Tuesday, everywhere else in three months — is a decision the business made, not an experiment the ML team designs.
- **Retrospective questions.** "Did the ranker upgrade we shipped last February actually cause the revenue lift?" — no one randomised control users in February; the question is asked in August against the historical data.

In each case the question is still *causal*: what would have happened *counterfactually* if the intervention had not occurred? An A/B test answers it by constructing the counterfactual through randomisation. When you cannot randomise, you have to construct the counterfactual some other way. That is the province of **causal inference** as it is practised in econometrics, epidemiology, and — increasingly — in industry data-science teams.

This chapter is the working knowledge a senior ML engineer needs to *frame* an observational question causally, to pick the right tool for the counterfactual, and — most importantly — to know when the framing is not strong enough to justify a shipping decision. The deep textbooks (Angrist and Pischke's [*Mostly Harmless Econometrics*](https://www.mostlyharmlesseconometrics.com/), Imbens and Rubin's [*Causal Inference for Statistics, Social, and Biomedical Sciences*](https://www.cambridge.org/core/books/causal-inference-for-statistics-social-and-biomedical-sciences/71126BE90C58F1A431FE9B2DD07938AB), Cunningham's [*Causal Inference: The Mixtape*](https://mixtape.scunning.com/), Pearl's [*Causality*](http://bayes.cs.ucla.edu/BOOK-2K/)) are the reference; this chapter is the L30's *route map* into them.

## §1 — What causal inference is (in one paragraph)

Every causal question can be written in **potential outcomes** notation (Rubin's counterfactual framework). For each unit *i*, let `Y_i(1)` be the outcome if treated and `Y_i(0)` be the outcome if not treated. The unit-level causal effect is `Y_i(1) − Y_i(0)`. **You never observe both** — a unit is either treated or not; the other potential outcome is counterfactual. The estimand of interest is usually the *Average Treatment Effect* `E[Y(1) − Y(0)]` or the *Average Treatment Effect on the Treated* `E[Y(1) − Y(0) | T = 1]`.

Randomised experiments identify these estimands directly because `T` is independent of `(Y(1), Y(0))`. Observational studies do not — and the whole job of a causal-inference method is to *emulate* the identifying assumption of randomisation under some weaker, defensible substitute.

The four substitutes this chapter covers are: **matching / weighting on observables** (§2), **difference-in-differences** (§3), **instrumental variables** (§4), and **regression discontinuity** (§5). Each rests on a specific *assumption* about the world; each fails silently when the assumption fails. A senior ML engineer's job is not to invent the method — it is to *recognise which assumption is plausible in a given setting* and to demand the diagnostics that would break the analysis if the assumption is violated.

## §2 — Propensity Score Matching (PSM) and inverse-probability weighting

### The assumption

**Conditional ignorability** (also called unconfoundedness): conditional on observed covariates `X`, treatment is as-good-as-random. Formally, `(Y(1), Y(0)) ⊥ T | X`. Equivalently, "we have measured every confounder — every variable that affects both `T` and `Y`."

This assumption is *strong*. It is untestable directly (you cannot check independence with an unobserved variable). Its plausibility is a domain argument: are there plausible confounders you have not measured? For most product-analytics settings the answer is yes, and the assumption is a stretch. For some settings — a retrospective analysis of a training-data policy where the covariates *are* everything the treatment mechanism could have used — the assumption is defensible.

### The method

Rosenbaum and Rubin's [*The Central Role of the Propensity Score in Observational Studies for Causal Effects*](https://academic.oup.com/biomet/article-abstract/70/1/41/240879) (Biometrika 1983) is the origin. The **propensity score** is `e(X) = P(T = 1 | X)` — a scalar summary of everything you know that might have driven treatment. Two units with the same propensity score have (under the ignorability assumption) the same treatment probability, so *among* those units, treatment is as-good-as-random and their outcome difference is an unbiased estimate of the local causal effect.

Two practical implementations:

- **Propensity Score Matching (PSM).** Fit `e(X)` (a logistic regression, a gradient-boosted classifier). For each treated unit, find the control unit(s) with the closest `e(X)` and pair them. Compute the outcome difference on the matched pairs. `MatchIt` (R) and `causalinference` (Python) are common libraries.
- **Inverse Probability Weighting (IPW / IPTW).** Fit `e(X)`. Weight each unit by `1 / e(X)` if treated and `1 / (1 − e(X))` if control. The weighted difference in means is an unbiased estimate of the ATE under ignorability. More statistically efficient than matching in many settings; more sensitive to extreme weights.

### The diagnostics

- **Covariate balance.** After matching or weighting, the covariate distribution *should* be identical (up to noise) in treated and control. Report standardised mean differences per covariate; a common rule of thumb is that any absolute standardised difference above 0.10 is a red flag.
- **Overlap / common support.** For every covariate value that appears in treatment, there should be control units with similar values (and vice versa). If treated units cluster at `e(X) ≈ 0.95` and no control unit is above `e(X) = 0.6`, you have no counterfactual for those treated units and PSM extrapolates rather than matches.
- **Sensitivity analysis.** Because ignorability is untestable, run a sensitivity analysis (Rosenbaum bounds or the E-value; VanderWeele and Ding's [*Sensitivity Analysis in Observational Research: Introducing the E-Value*](https://www.acpjournals.org/doi/10.7326/M16-2607)): how strong would an *unmeasured* confounder have to be to explain away the estimated effect? If the answer is "one much smaller than covariates already in the model," the estimate is fragile.

### When PSM/IPW is appropriate for an ML question

- **Retrospective analysis of a launched change**, where the covariates you have plausibly explain who did and did not get exposed.
- **A quasi-experiment** where the assignment mechanism is *known* and depends on observables (e.g., a rules-based rollout to eligible users; a targeted campaign whose targeting rules are recorded).
- **Never**: a setting where the treatment was chosen by the user (self-selection), because the user's private information — motivation, expected utility from the treatment — is unobserved by definition. This is one of the hardest sources of confounding to argue away.

## §3 — Difference-in-Differences (DID)

### The assumption

**Parallel trends**: in the absence of treatment, the treated and control groups' outcomes would have moved in parallel over time. Any deviation from parallel that appears at treatment time is attributed to the treatment. Card and Krueger's [*Minimum Wages and Employment: A Case Study of the Fast-Food Industry in New Jersey and Pennsylvania*](https://davidcard.berkeley.edu/papers/njmin-aer.pdf) (AER 1994) is the canonical exemplar: New Jersey raised its minimum wage, Pennsylvania did not, and the *change* in employment in NJ from before to after is compared to the *change* in employment in PA over the same window. If the two states' employment moved in parallel *before* the minimum wage change, the divergence *after* is causally attributable to the wage change.

### The method

Let `Y_gt` be the outcome for group `g ∈ {treated, control}` at time `t ∈ {pre, post}`. The DID estimator is:

```
τ_DID = (Y_treated,post − Y_treated,pre) − (Y_control,post − Y_control,pre)
```

Equivalently, a two-way fixed-effects regression:

```
Y_it = α_i + γ_t + τ * (T_i * Post_t) + ε_it
```

where `α_i` is a unit fixed effect, `γ_t` is a time fixed effect, and `τ` is the coefficient of interest.

### The diagnostics

- **Pre-treatment parallel-trends plot.** Plot treated and control outcomes over multiple pre-treatment periods. If they moved in parallel, the assumption is plausible. If they did not, DID is compromised — a divergence that started *before* treatment is not caused by treatment.
- **Placebo tests.** Fit the DID on a *fake* treatment date before the real one. If the "effect" at the fake date is small and non-significant, the design is credible. If it is not, the design has a lurking trend the DID is picking up.
- **Multiple control groups.** Test the DID against several plausible controls; if the estimate is consistent across them, that is corroborating evidence.

### The modern DID complication

Callaway and Sant'Anna's [*Difference-in-Differences with Multiple Time Periods*](https://arxiv.org/abs/1803.09015) (Journal of Econometrics 2021) and Goodman-Bacon's [*Difference-in-Differences with Variation in Treatment Timing*](https://arxiv.org/abs/1908.05481) — plus a broader "staggered DID" literature that emerged since 2019 — showed that classical two-way fixed effects DID is *biased* when treatment is rolled out at different times to different units (which is the common case in industry rollouts). The modern estimators (Callaway–Sant'Anna, Sun–Abraham, de Chaisemartin–D'Haultfœuille) are what a senior ML engineer should reach for on staggered rollouts. `did` (R) and `differences` (Python) are implementations.

### When DID is appropriate for an ML question

- **A geographic rollout.** The feature launches in Germany on Tuesday, everywhere else in three months. Compare Germany-vs.-elsewhere on the outcome, before-vs.-after.
- **A model version cutover.** The old model retired on a date; the new model started; compare the *change* in metric before and after cutover, against a control group that was on a *different* system that did not change.
- **Never**: when the treatment was rolled out based on a trend the control did not experience. If the rollout targeted the segment whose engagement was already climbing, DID attributes the pre-existing climb to the treatment.

## §4 — Instrumental Variables (IV)

### The assumption

There exists a variable `Z` (the *instrument*) that:

- Affects `T` (the treatment).
- Affects `Y` (the outcome) **only through `T`** (the exclusion restriction).
- Is independent of any unobserved confounder of `T` and `Y`.

If such a `Z` exists, the two-stage least squares (2SLS) estimator gives an unbiased estimate of the causal effect of `T` on `Y` even when `T` is confounded. Angrist and Pischke's chapter on IV is the reference; Angrist, Imbens, Rubin's [*Identification of Causal Effects Using Instrumental Variables*](https://www.jstor.org/stable/2291629) (JASA 1996) is the modern statistical statement.

### The method

Two-stage:

- **First stage.** Regress `T` on `Z` (and any controls). Predict `T̂`.
- **Second stage.** Regress `Y` on `T̂` (and the same controls). The coefficient on `T̂` is the IV estimate of the causal effect.

### Where it appears in ML

- **A/B tests with imperfect compliance.** Users are assigned to treatment, but only some actually see the treatment (they close the app before it renders, they are in a network region that skipped the fetch). Assignment is a valid instrument for exposure. The IV estimator here is the *Complier Average Causal Effect* (CACE).
- **Encouragement designs.** You send an email prompting some users to try a feature; users who receive the email are more likely to try; you cannot randomise "trying the feature" but you can randomise "receiving the email." The email is an instrument.
- **Natural experiments.** A weather event, a competitor outage, a random regulatory change that pushes some users to the treatment without directly affecting the outcome. Rare in industry but powerful when present.

### The diagnostics

- **First-stage F-statistic.** If the instrument is *weak* (barely predicts `T`), the IV estimator's variance blows up and the point estimate can be badly biased. A conventional threshold is `F > 10` for a single instrument.
- **Exclusion restriction argument.** The instrument cannot affect the outcome except through the treatment. This is *untestable* — the argument is a domain one, and if the reader is not convinced, the IV estimate is not credible.
- **Sargan-Hansen test** (with multiple instruments): tests overidentification; not a definitive proof of exclusion but a useful check.

### The senior read on IV

IV is powerful when a valid instrument exists. Valid instruments are rare and almost never discovered by looking through the covariates for something correlated with treatment; they are usually a specific quirk of the world (a randomised email, a random natural event, an assignment mechanism you have direct knowledge of). If you find yourself hunting for an instrument in a bag of features, you are probably looking at PSM territory, not IV territory.

## §5 — Regression Discontinuity Design (RDD)

### The assumption

Assignment to treatment is a discontinuous function of a *running variable* — a threshold. Units just above the threshold get the treatment; units just below do not. If the running variable is *not* manipulable at the boundary (nobody can nudge their own value to land on the right side), then units in a narrow band around the threshold are as-good-as-randomised.

Imbens and Lemieux's [*Regression Discontinuity Designs: A Guide to Practice*](https://scholar.harvard.edu/imbens/publications/regression-discontinuity-designs-guide-practice) (JoE 2008) is the applied reference; Lee and Lemieux's [*Regression Discontinuity Designs in Economics*](https://www.aeaweb.org/articles?id=10.1257/jel.48.2.281) (JEL 2010) is a broader survey.

### Where it appears in ML

- **A credit-approval rule** — applicants above a credit-score threshold get the loan; applicants just below do not. RDD estimates the causal effect of receiving the loan on subsequent outcomes.
- **A tier eligibility rule** — spending above a threshold unlocks a premium tier; users just above vs. just below are approximately comparable.
- **A model-triggered intervention** — a fraud model above score 0.7 triggers a review; below does not. RDD on the review's outcome.

### The diagnostics

- **Density-of-running-variable test.** McCrary's [*Manipulation of the Running Variable in the Regression Discontinuity Design: A Density Test*](https://eml.berkeley.edu/~jmccrary/DCdensity/) (JoE 2008): a discontinuous *density* at the threshold suggests manipulation, invalidating RDD.
- **Covariate continuity at the threshold.** Non-treatment covariates should be continuous across the threshold. If they jump, something else is different at the threshold.
- **Bandwidth sensitivity.** RDD estimates are local to the threshold. Reporting estimates at multiple bandwidths (Imbens and Kalyanaraman optimal bandwidth is a standard choice) tells you whether the result is fragile to the choice.

### When RDD is appropriate for an ML question

Whenever a **rule** — not an experiment — determines who gets a treatment based on a continuous score, and the rule is enforced sharply. Model-triggered interventions (fraud review, ML-gated moderation) are frequently RDD candidates.

## §6 — Synthetic control (a modern favourite)

For a single treated unit (e.g., a country where a feature launched), Abadie, Diamond, Hainmueller's [*Synthetic Control Methods for Comparative Case Studies*](https://economics.mit.edu/sites/default/files/publications/synthetic-control-methods.pdf) (JASA 2010) construct a *synthetic control* — a weighted combination of untreated units that matches the treated unit's pre-treatment trajectory. The counterfactual is the synthetic control's post-treatment trajectory.

Synthetic control has become the standard tool for geographic rollouts with one or a few treated units where classical DID's parallel-trends assumption is hard to argue. `Synth` (R) and `SparseSC` (Python) are common libraries.

Newer variants — Athey, Bayati, Doudchenko, Imbens, Khosravi's [*Matrix Completion Methods for Causal Panel Data Models*](https://arxiv.org/abs/1710.10251); Ben-Michael, Feller, Rothstein's [*The Augmented Synthetic Control Method*](https://arxiv.org/abs/1811.04170) — extend the framework.

## §7 — What the framing gives up, honestly

Every method in this chapter is a *substitute* for randomisation, and every substitute weakens the claim you can make. A senior ML engineer names the weakening explicitly in the release conversation:

- **PSM / IPW** assumes no unobserved confounder. If a decision hinges on the estimate, an E-value sensitivity analysis is mandatory.
- **DID** assumes parallel trends. Pre-treatment plots and placebo tests are mandatory.
- **IV** assumes the exclusion restriction. A domain argument is mandatory, and the instrument's strength must be checked.
- **RDD** assumes non-manipulation at the threshold. A McCrary density test and covariate-continuity check are mandatory.
- **Synthetic control** assumes the pre-treatment fit is good; the post-treatment gap is causal only if the synthetic control would have continued matching absent treatment.

None of these estimates should be treated with the same confidence as an A/B result. The written framing on a release memo is different: "the A/B showed a lift of X ± Y with p < 0.05" is a *statistical* statement; "the DID estimate is Z ± W under the parallel-trends assumption, corroborated by placebo test A and Callaway–Sant'Anna staggered estimate B" is a *methodological* statement with an explicit assumption load.

## §8 — When to reach for causal inference vs. designing a better experiment

The senior read: **before reaching for causal inference, look harder at whether an experiment is truly impossible.** Many "impossible to randomise" claims are actually "we did not build the platform to randomise on this dimension" claims. Options that sometimes rescue an A/B:

- **Randomise the *decision*, not the *treatment*.** You cannot randomise which patients receive a treatment, but you can randomise which clinicians see the decision support tool.
- **Randomise the *rollout order*.** A geographic rollout that would launch in Germany first can, without much cost, be extended to launch in a randomly-chosen half of markets on the same date. The other half becomes the control for a DID or synthetic-control analysis with the *randomised* order strengthening the identification.
- **Randomise the *default*.** Instead of randomising the feature, randomise whether the feature is on by default; observe uptake, use IV on the encouragement.

Causal inference is the fallback when *none* of these rescues is available. It is not the first tool a senior ML engineer reaches for; it is the tool of last resort when the world genuinely denies the randomisation.

## §9 — What to bring back to the module

- **Chapter 01 (design)** is about the primary metric, MDE, and stopping rule of an *experiment*. When the experiment is impossible, the primary metric and the estimand (ATE, ATT, CACE, LATE) are still named up front; the "stopping rule" becomes the estimator + the sensitivity analysis.
- **Chapter 02 (trust)** applies unchanged in spirit: the equivalent of SRM is *balance* — is the treated group representative of the population you claim to be estimating on. The equivalent of a novelty check is a *time-heterogeneity check* — is the estimated effect stable across pre/post windows.
- **Chapter 03 (variance reduction)** is closely related to observational adjustment: CUPAC is regression on covariates to shrink variance under randomisation; PSM/IPW is regression on covariates to *remove* confounding without randomisation. The tooling is similar; the guarantee is not.
- **Chapter 04 (sequential testing)** has less direct application: observational causal estimates are usually one-shot analyses on retrospective data, not continuous-monitoring settings.

## Summary

When randomisation is unavailable — for regulatory, operational, or historical reasons — the counterfactual has to be constructed differently. Propensity score matching and IPW assume conditional ignorability; DID assumes parallel trends; IV assumes an exclusion restriction on a variable that predicts treatment but not the outcome directly; RDD assumes a sharp threshold with non-manipulation; synthetic control matches a treated unit to a weighted average of untreated units on the pre-treatment trajectory. Every method is a *substitute* for randomisation, weakens the strength of the claim, and demands its own diagnostic (balance tables, parallel-trends plots, first-stage F, McCrary density, pre-treatment fit). Before reaching for causal inference, a senior ML engineer looks hard at whether a smarter design can rescue an A/B — randomise the decision, the rollout order, or the default. When the rescue is unavailable, causal inference is the tool; the release memo names the estimand, the estimator, the assumption load, and the sensitivity analysis explicitly, and the decision is made with the weakened-guarantee shape acknowledged.
