# Sequential testing and the cost of peeking

## Motivation

Every experiment has the same tension. The business wants to ship the winners fast and kill the losers faster. The statistics wants a fixed sample size and one look at the data. The team, in practice, splits the difference — they peek at the dashboard every day, and if the p-value dips below 0.05 they call it a win.

That last habit is the one that quietly destroys the false-positive guarantee the experiment was designed for. Under a fixed-horizon design with α = 0.05, the probability of *ever* declaring significance while continuously monitoring is not 0.05 — it climbs toward 1 as the experiment runs. Johari, Koomen, Pekelis, Walsh's [*Peeking at A/B Tests: Why It Matters, and What to Do About It*](https://arxiv.org/abs/1512.04922) (KDD 2017) is the modern industrial paper on the problem: a naive experimenter who monitors continuously and stops on first significance ships a false positive at rates of **20–50 %** rather than the intended 5 %.

Two responses. **Wait longer** — commit to a fixed horizon, do not look at significance until the end. The team hates it, and the business often hates it more. Or **change the statistical framing** — use *always-valid inference* (sequential tests, confidence sequences) that hold their false-positive guarantee under continuous monitoring. This chapter is the working knowledge a senior ML engineer needs to reach for the second option correctly: when it applies, what the cost is, and how it composes with the design work from chapters 01–03.

## §1 — Why peeking breaks fixed-horizon tests

A fixed-horizon t-test's guarantee is: *under the null (no effect), the p-value at the pre-specified horizon is uniform on [0, 1], so `P(p < 0.05) = 0.05`*. Nothing about that guarantee survives *repeated* looks. If you look 100 times over the course of the experiment and stop the moment any one look shows `p < 0.05`, you have effectively taken the *minimum* p-value across 100 correlated tests. The distribution of that minimum is not uniform — it is concentrated near zero.

The intuition: for a null-effect experiment, the running estimated effect is a Brownian-motion-like walk around zero. Given enough looks, it *will* wander into significance territory purely by chance. The naive analyst declares that wander a win.

The rate at which the false-positive probability inflates depends on the looking schedule, the metric variance, and the horizon. Two rules of thumb from the Johari et al. paper:

- **10 looks over an experiment inflates a nominal α = 0.05 to roughly 0.15–0.20** for typical metrics.
- **Continuous monitoring inflates it toward the classical *law of the iterated logarithm* — the maximum grows like `sqrt(2 * log log t)`**, and given long enough, any α-threshold is crossed almost surely.

The consequence for practice is stark: **a fixed-horizon experiment must be analysed at the fixed horizon, once.** If you cannot commit to that, you need a framing whose guarantee survives peeking.

## §2 — What "always-valid" means

An always-valid confidence sequence is a sequence of intervals `(L_t, U_t)` such that:

```
P(θ ∈ (L_t, U_t) for all t simultaneously) ≥ 1 − α
```

Compare to a fixed-horizon CI, which is `P(θ ∈ (L, U)) ≥ 1 − α` at the *single* pre-specified time. The always-valid version bakes the peeking discipline in: you can look at the interval at any point during the experiment, stop when you like, report the interval as of the stopping time, and the coverage guarantee holds.

Equivalently, an **always-valid p-value** `p_t` is a sequence such that under the null, `P(inf_t p_t ≤ α) ≤ α`. You can stop the first time `p_t ≤ α` and your false-positive rate is still ≤ α.

The core tool that makes this work is the **martingale**. Under the null, likelihood-ratio-based statistics have a martingale structure; Ville's inequality bounds the probability that a non-negative martingale ever exceeds a threshold. Wald's [*Sequential Analysis*](https://archive.org/details/sequentialanalys00wald_0) (1947) is the origin. Modern industrial treatments — Johari, Pekelis, Walsh's [*Always Valid Inference: Continuous Monitoring of A/B Tests*](https://arxiv.org/abs/2103.14043) — turn the machinery into practical estimators.

## §3 — Three practical always-valid families

### 3a — Group Sequential Tests (GST)

Look at the data a fixed, small number of times (say 4 or 5), at pre-registered fractions of the horizon. At each look, compare the test statistic to a *look-specific* boundary that is wider than the nominal α threshold; if the statistic crosses, stop and declare significance. The boundaries are picked so that the *overall* false-positive rate across all looks is ≤ α.

- **Pocock boundaries** — every look uses the same (wider) threshold; simplest to reason about.
- **O'Brien-Fleming boundaries** — very conservative early, closer to nominal α late; the most common industrial choice because it lets long-running experiments enjoy near-nominal α at the end while still enabling early stopping for dramatic effects.
- **Alpha-spending functions** (Lan-DeMets) — a continuous generalisation; you allocate an "α budget" over the experiment and each look consumes some of it.

GSTs suit programs where **experiments have a scheduled review cadence** (a weekly experiment review) and the "look" is naturally discrete. Kohavi/Tang/Xu chapter 17 has a walkthrough. The R packages `gsDesign` and `ldbounds` are the classical references; every enterprise experimentation platform (Split.io, Optimizely, Statsig, Eppo, LinkedIn's XLNT) implements at least the O'Brien-Fleming variant.

### 3b — mSPRT (mixture Sequential Probability Ratio Test)

Wald's SPRT tests a *point null* against a *point alternative* — but an experimenter usually does not know the alternative in advance. The **mixture SPRT** places a prior on the effect size and integrates. The result is an always-valid test that does not require pre-specifying the effect: you can peek at every user, and the resulting p-value's minimum under the null is still ≤ α.

Johari, Pekelis, Walsh's paper is the modern reference; Optimizely productionised the mSPRT under the name *Stats Engine*.

The trade-off: mSPRTs are more conservative than a fixed-horizon test of the same size — you need somewhat more sample for the same MDE if you never end up stopping early. The gain is that you can stop early when the effect is dramatic, and you can peek without penalty.

### 3c — Confidence sequences via betting / e-values

The newest wing of the always-valid literature — Waudby-Smith and Ramdas's [*Estimating means of bounded random variables by betting*](https://arxiv.org/abs/2010.09686) — reframes always-valid inference as a betting game. The estimator is a *confidence sequence*, tight around the true mean, and valid under continuous monitoring. Especially useful for bounded metrics (proportions, capped revenue) where the intervals are notably tighter than mSPRT.

The senior read: this is the direction the field is moving. If you are picking a library today, prefer one that supports both mSPRT and betting-based sequences. Existing implementations include `confseq` (Waudby-Smith / Ramdas) and integration into some experimentation platforms.

## §4 — Bayesian framings (a partial answer)

Bayesian A/B testing — quoting a posterior probability that treatment exceeds control — is often marketed as immune to peeking. The truth is more nuanced. Deng, Lu, Chen's [*Continuous Monitoring of A/B Tests without Pain: Optional Stopping in Bayesian Testing*](https://arxiv.org/abs/1602.05549) shows that Bayesian posteriors *are* well-defined under continuous monitoring — they are always the posterior given the data seen so far — but making a *decision* from a posterior with an implicit loss function still has calibration properties that depend on the prior and the stopping rule.

Practical take: Bayesian A/B is useful when the team is comfortable specifying priors and thinks in loss functions rather than p-values. It is not a free pass on peeking discipline; it is a *different* framing where the peeking question is expressed as "what is the posterior expected loss of stopping now" rather than "has the α been spent." Both framings are valid; both require care.

## §5 — When to use sequential testing (and when not to)

Use it when:

- **The experiment has a large potential MDE and stakes are high for early stopping.** A dramatic treatment can be shipped in a week instead of six; a dramatic regression can be killed in a day instead of running out the horizon.
- **The team has a strong culture of peeking regardless.** If people are going to look daily, do them the favour of giving them a framing where looking is valid.
- **The metric has heavy tails and effects that grow over exposure.** Sequential tests naturally accumulate evidence; fixed-horizon tests can underestimate because the metric has not stabilised.

Do not use it when:

- **You need the maximum statistical power at a fixed sample size.** A fixed-horizon test is strictly more powerful than a sequential one at the same total N if you never end up peeking. If your program's culture supports "one look at horizon," fixed-horizon is a smaller-N experiment for the same MDE.
- **You have novelty concerns (chapter 02 §3).** Sequential tests can crown a winner early on a metric that has not yet reached steady state. Combine sequential testing with a minimum horizon (usually at least two weeks) that lets novelty decay before the "keep going / stop" decision is made.
- **You do not have the tooling to compute always-valid p-values correctly.** A hand-rolled sequential test is a pipeline that must be tested — its false-positive coverage should be validated against simulated null data before it is trusted (Miller and Hosanagar's [*Simulation-based statistical testing in journalism*](https://arxiv.org/abs/1701.05038) is one reference on empirical validation of statistical procedures).

## §6 — Composition with the rest of the module

- **With chapter 01 (design).** Sequential testing changes the *stopping rule*, not the *MDE*. Chapter 01's MDE calculation still applies — with the caveat that the effective MDE at any given horizon is slightly larger under sequential testing than under a fixed-horizon test of the same total N. Design accordingly.
- **With chapter 02 (trust).** Sequential testing *does not* fix SRM, interference, or novelty. A confidence sequence over an SRM-poisoned experiment is a valid interval around a *biased* estimator, which is the wrong tool. The trust diagnostics run every day of a sequential experiment, and any red trust check suppresses the read (same policy as chapter 02 §4).
- **With chapter 03 (variance reduction).** CUPED and sequential testing compose. A sequential test on a CUPED-adjusted metric enjoys both benefits.
- **With chapter 05 (causal inference).** Sequential inference has observational analogues (sequential estimators for causal effects in observational data), but the failure modes of observational causal inference (unmeasured confounding, selection bias) dominate the peeking question in that setting.

## §7 — The peeking rule in practice

A senior ML engineer's working policy on peeking:

- **Trust diagnostics (chapter 02) are always visible.** SRM, cross-arm error parity, ramp-step guardrails. Suppression rules from chapter 02 §4 apply.
- **Primary and guardrail metrics are visible with confidence intervals.** Whether you interpret the CI at the current time or only at the horizon is a matter of the framing. **If the framing is fixed-horizon, the CI is "as of the horizon we agreed to" and looking early is for guardrail sanity, not for stopping**. If the framing is sequential, the CI is always-valid and looking early *is* the framing.
- **The stopping rule is written before the experiment starts.** For fixed-horizon: "we will read at N = X, alpha = 0.05, ship if the primary CI is above zero and all guardrails non-inferior." For sequential: "we will monitor continuously with a Pocock / O'Brien-Fleming / mSPRT boundary at alpha = 0.05; stop when the sequential p is below 0.05 for a hold-time of Y hours; otherwise run to horizon Z."
- **Deviations from the pre-registered stopping rule are recorded.** "We stopped early because" is a legitimate business decision — it just needs to be recorded as an override rather than presented as a statistical significance claim.

## §8 — Two failure modes sequential testing catches (and one it doesn't)

- **"We stopped at the first significance and shipped a false positive."** The classical peeking failure. Sequential tests are exactly the fix: always-valid p-values guarantee that stopping on first significance keeps the false-positive rate at α.
- **"We ran to horizon and shipped a false positive because we metric-shopped."** Not a peeking failure — a metric-selection failure. Sequential testing does not fix this; only the discipline of primary metric chosen in advance (chapter 01 §1) does.
- **"We shipped a null-result experiment as inconclusive when it was actually a real 0.2 % effect that needed 4× the sample."** Not a peeking failure either — an under-powered design. Sequential testing does not fix this; the design's MDE (chapter 01 §3) does.

The senior read: sequential testing is the right tool for exactly one problem — false-positive control under continuous monitoring. Attempting to make it fix other problems is how a team ends up with three overlapping statistical framings and no consistent decision rule.

## Summary

Peeking at a fixed-horizon experiment silently inflates the false-positive rate to many times its nominal α. The two responses are to *stop peeking* (commit to the fixed horizon) or to *change the framing* to always-valid inference. Group Sequential Tests (Pocock, O'Brien-Fleming), the mixture SPRT (Optimizely-style Stats Engine), and betting-based confidence sequences are the three families that give you always-valid guarantees. Bayesian framings are a legitimate alternative when the team thinks in posteriors and loss functions rather than p-values, but they are not a free pass. Sequential testing composes with variance reduction (chapter 03) but does not fix trust failures (chapter 02) or under-powered designs (chapter 01); its one job is false-positive control under peeking, and it is the right tool for exactly that one job. Chapter 05 is the last piece of the module: what to do when the experiment cannot be run at all.
