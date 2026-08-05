# Designing an experiment: power, MDE, primary and guardrail metrics, ramp

## Motivation

The offline harness (mod-305 chapter 01) shipped a `HarnessDecision` with `passes_release_gate=True`. Shadow was clean. Canary walked its ladder with no rollback. The candidate is now sitting in front of the last gate the release ever crosses — the A/B test. This is the stage that turns "does not blow up under partial exposure" into "is measurably better for users." Every serious ML org has this stage. Not every ML org runs it defensibly.

The way an A/B stage stops being defensible is boring and repeatable. The team ships the experiment without a power calculation, runs it for a week because a week feels like the right amount of time, reads a flat result as "no effect," and either ships anyway ("the offline lift looks real") or throws the candidate out ("A/B didn't confirm"). Both readings are wrong for the same reason — the experiment could not have detected the effect the team cared about because it never had the sample size to do so. Kohavi, Tang, and Xu's [*Trustworthy Online Controlled Experiments*](https://experimentguide.com/) calls this class of failure *inconclusive by design*.

This chapter is the discipline that keeps an A/B test from being inconclusive by design. Power and Minimum Detectable Effect (MDE) up front. Primary and guardrail metrics chosen before the treatment exists. A ramp plan that names every step, its duration, and its go/no-go criteria. Chapters 02 (trust diagnostics), 03 (variance reduction), and 04 (sequential testing) are the machinery you deploy *inside* a well-designed experiment. This chapter is where you make sure the experiment is well-designed in the first place.

## What an experiment is, at senior altitude

At L20 an experiment is a treatment column added to an events table and a p-value at the bottom of a notebook. At L30 an experiment is a **contract**:

- A **hypothesis** — one sentence, testable, tied to a product outcome.
- A **primary metric** — one, agreed in advance, matching the hypothesis.
- A **set of guardrail metrics** — the things that must not regress even if the primary lifts.
- An **MDE and a power calculation** — the smallest effect the experiment is instrumented to detect, and the probability it will detect it.
- A **randomisation unit and traffic-split plan** — who gets which arm, and how they stay in it.
- A **ramp plan** — how exposure grows over time and what advances it.
- A **stopping rule** — when the experiment ends and how the decision is made.

If any of the seven is missing, the experiment is a script producing a number, not an evidence-generating instrument. The rest of this module (and mod-305 chapter 03) assumes the seven are present.

## §1 — The hypothesis and the primary metric

The hypothesis is the sentence a stakeholder can read and either agree or disagree with. Not "the new ranker is better" but "the new ranker will improve conversion-per-session for the new-visitor slice by ≥ 1 % relative, holding p95 latency and refund-rate flat." The hypothesis names the population, the direction, the size, and the boundary conditions.

The primary metric is *derived from* the hypothesis, not the other way around. The rule from mod-305 chapter 01 applies unchanged: **the primary metric is chosen before the candidate exists**. Metrics-shopping — running the experiment and then choosing the metric that lifted — is what turns a 5 % false-positive rate into a 30 % false-positive rate when the team routinely picks the best of six candidate metrics. Andrew Gelman's [*The garden of forking paths*](http://www.stat.columbia.edu/~gelman/research/unpublished/p_hacking.pdf) is the statistical-methodology reference; Kohavi et al.'s [*A/B Testing Intuition Busters*](https://exp-platform.com/Documents/2022-A_BTestingIntuitionBusters.pdf) covers the industrial manifestation.

Good primary metrics share three properties:

- **Directly tied to the product outcome.** Not clicks when the business cares about revenue; not sessions when the business cares about retention. Chapter 06 of Chip Huyen's [*Designing Machine Learning Systems*](https://huyenchip.com/books/) and chapter 7 of Kohavi/Tang/Xu are the classical treatments.
- **Measurable within the experiment window.** A "30-day retention" primary metric on a two-week experiment is a metric the experiment cannot observe end-to-end. Either extend the window or pick a proxy the team has *validated* against the long-horizon metric on prior experiments.
- **Sensitive to the intervention.** A metric with variance so large that no realistic sample size detects the change is a metric that will always report "no effect." Chapter 03 (variance reduction) is a partial answer; picking a better metric is often the better one.

The one-metric rule is not a religious prohibition on secondary metrics — it is a discipline about *which metric the ship/don't-ship decision is bound to*. Secondary metrics are recorded; the decision is bound to the primary.

## §2 — Guardrail metrics: the things that must not regress

Guardrail metrics are the "the primary metric may lift but not at any cost" list. A senior team keeps a small, stable set — usually four to seven — that guard the outcomes the org has already decided it will not trade against a primary lift. Common shapes:

- **Latency percentiles** — p50, p95, p99 on the request path the treatment touches. Mod-307 (reliability) is the vocabulary.
- **Error rate / crash rate.** Any candidate that lifts the primary metric while doubling the 5xx rate is not a release.
- **Cost per event.** For ML-augmented features, per-request cost and per-day cost envelope. Mod-304 chapter 04 is where the envelope is set.
- **Business counter-metrics.** Refund rate for a purchase experiment, complaint rate for a moderation experiment, unsubscribe rate for a notification experiment.
- **Fairness / worst-group** — the slice matrix from mod-305 chapter 02, re-enforced online.
- **Safety triggers** — for LLM-augmented features, refusal rate and safety-classifier trigger rate.

Guardrails carry **asymmetric thresholds**. A latency guardrail that says "p99 must not increase by more than 3 %" is a *cap*, not a lift target. The experiment does not need to *improve* p99 to pass; it needs to *not regress it beyond the cap*. Statistically, guardrails use a *non-inferiority* test (is the treatment worse by more than the cap, at some confidence?) not a superiority test. The `passes_release_gate` boolean from mod-305 chapter 01 becomes online: primary lift *and* every guardrail non-inferior.

The guardrail list is a **repo file**, reviewed like code, versioned, and the same across experiments in a given surface area. Adding a guardrail during an experiment is a red flag; removing one is a bigger one. Kohavi/Tang/Xu chapter 6 ("Metrics for Experimentation") is the reference for building the guardrail set.

## §3 — Power, α, β, and Minimum Detectable Effect

Every experiment has four numbers the design has to name:

- **α (significance level).** The probability of falsely declaring an effect when there is none. Usually 0.05 (two-sided) — a convention, not a truth.
- **β (Type II error rate).** The probability of missing an effect that is there. `1 − β` is the *power*. Usually a power of 0.8 is the industry floor; high-stakes experiments target 0.9 or higher.
- **σ (standard deviation of the metric per unit).** Measured from historical data on the metric.
- **Δ (Minimum Detectable Effect, MDE).** The smallest true effect the experiment is instrumented to catch at the chosen α and power.

The four are related by a formula every experimenter should have muscle memory for. For a two-sample two-sided test on a mean with equal-sized arms and randomisation unit ≡ analysis unit:

```
n_per_arm  ≈  ((z_{1 − α/2} + z_{1 − β})² * 2 * σ²) / Δ²
```

Numerically, with α = 0.05 two-sided and power 0.8, `(z_{1 − α/2} + z_{1 − β})² ≈ (1.96 + 0.84)² ≈ 7.84`. So `n_per_arm ≈ 16 σ² / Δ²` (a useful back-of-envelope). For a proportion metric like conversion rate `p`, substitute `σ² = p(1 − p)`.

The four numbers are load-bearing in both directions:

- **Given α, β, σ, choose Δ → get n.** "The smallest effect we care to detect is 1 % relative to a 4 % baseline conversion; how many users per arm?" (`p = 0.04`, `Δ = 0.0004`, `σ² = 0.04 * 0.96 ≈ 0.0384`, `n_per_arm ≈ 16 * 0.0384 / 0.0004² ≈ 3.8 M`.)
- **Given α, β, σ, n → get Δ.** "We have 200 K users per arm per week; what is the smallest effect our experiment can detect if it runs a week?" — the *achievable* MDE.

A worked skeleton in Python (using `statsmodels` for readability; every stats library has an equivalent):

```python
from statsmodels.stats.power import NormalIndPower, tt_ind_solve_power

# Proportion metric: baseline 4%, target lift 1% relative → absolute 0.04 → 0.0404
p, lift_rel = 0.04, 0.01
delta = p * lift_rel
sigma = (p * (1 - p)) ** 0.5

analysis = NormalIndPower()
n_per_arm = analysis.solve_power(
    effect_size=delta / sigma,     # Cohen's d
    alpha=0.05,
    power=0.8,
    alternative="two-sided",
)
print(f"n per arm: {n_per_arm:,.0f}")
```

The MDE calculation is the single most-skipped step in an underpowered experiment. Kohavi et al.'s [*Controlled experiments on the web: survey and practical guide*](https://www.researchgate.net/publication/220208696_Controlled_experiments_on_the_web_survey_and_practical_guide) walks through it in the applied setting.

### Reading a flat experiment honestly

An experiment whose primary metric CI includes zero has three interpretations, and the design decides which one you get to make:

- **The effect is truly ~zero.** Your CI is narrow (upper bound below the MDE). Ship or kill on the guardrails and the operational trade.
- **The effect exists but the experiment was too small.** Your CI is wide (upper bound well above the MDE). "Flat" is uninformative — you needed a larger experiment.
- **The effect exists but you measured the wrong metric.** The primary was insensitive; secondary signals moved. Re-scope the hypothesis, do not metric-shop.

A team that runs a flat experiment and concludes "the candidate does nothing" without checking which of the three they are in ships the wrong decision routinely. Kohavi, Deng, Vermeer's [*A/B Testing Intuition Busters*](https://exp-platform.com/Documents/2022-A_BTestingIntuitionBusters.pdf) catalogues the mistake.

## §4 — Randomisation unit vs. analysis unit

The randomisation unit is what you split — a user, a session, a device, a session-scoped cookie, an account, a page-view. The analysis unit is what you compute the metric over — often but not always the same thing.

The rule: **the randomisation unit must be at least as coarse as the analysis unit**, or the standard errors reported by the naive analysis are wrong. If you randomise by user but analyse per-session, sessions from the same user are correlated; the naive standard error underestimates the true variance and the reported p-value understates the noise. Deng, Knoblich, Lu's [*Applying the Delta Method in Metric Analytics: A Practical Guide with Novel Ideas*](https://www.exp-platform.com/Documents/2018KDDDeltaMethod.pdf) is the industrial treatment; the fix is either to aggregate to the randomisation unit before analysis (per-user metric averaged) or to use the delta method to compute a properly clustered standard error.

Common randomisation units and when to use each:

- **Per-request.** Stateless per-request decision, no session-level or downstream effect. Fine for a classifier decision that never persists.
- **Per-session.** Session-scoped experience. Common for search rankers when session continuity matters.
- **Per-user (persistent ID).** The default for user-facing product experiments. Guarantees consistency across sessions.
- **Per-account / per-workspace / per-organisation.** B2B or multi-tenant SaaS. Randomising per-user inside an account causes interference (chapter 02) — one user sees the treatment, complains to their admin, poisons the control users' perception.
- **Per-cluster (geographic, social, catalogue).** When network / marketplace interference (chapter 02) makes user-level randomisation invalid. Ugander, Karrer, Backstrom, Kleinberg's [*Graph cluster randomization: network exposure to multiple universes*](https://arxiv.org/abs/1305.6979) is the reference.

Once chosen, the randomisation unit is **stable across the experiment**. A user assigned to treatment on day one is in treatment on day fourteen, on every device, in every session. Bucket assignment is a deterministic function of `hash(user_id, experiment_id, salt)` — no server-side state, no chance of re-bucketing on a cache miss.

## §5 — The ramp plan

The ramp plan is the traffic schedule. It is *not* the canary from mod-305 chapter 03 — that stage answered "does this candidate blow up under partial exposure." The ramp is inside the A/B stage, and its job is to trade off blast radius against statistical power.

A standard shape:

```
1 %  → 24 h   trust + guardrail check (SRM, error rate, latency)
5 %  → 24 h   guardrail check, first metric direction read
10 % → 72 h   business-cycle exposure across weekend/weekday
50 % → full run to power (typically 1–4 weeks)
```

Every step has three predefined pieces:

- **Duration.** In wall-clock time, not "until it looks fine." At minimum a full business day; more common one to two weeks at the full-power step because weekly seasonality is a first-class source of variance (Kohavi/Tang/Xu chapter 3 discusses running experiments for at least one full weekly cycle).
- **Advance criterion.** What the guardrails have to show for the ramp to go to the next step. Trust diagnostics (chapter 02) live here — SRM, cross-arm error-rate parity, guardrail non-inferiority.
- **Rollback criterion.** The auto-rollback threshold if a guardrail goes red. Same shape as mod-305 chapter 03's canary auto-rollback.

Two design choices that shape the ramp:

- **Split ratio.** 50/50 is the most statistically efficient (minimum variance for fixed total N). Uneven splits (95/5, 90/10) sacrifice power for blast-radius safety; a 95/5 experiment needs roughly 4× the total N to reach the same power as a 50/50 experiment (Kohavi/Tang/Xu chapter 15 has the math). Uneven splits are appropriate when the treatment is riskier or when treatment traffic is expensive (an LLM-augmented feature); 50/50 is the default when it is safe.
- **Fixed-horizon vs. sequential monitoring.** A fixed-horizon experiment declares "we will run for N users and read the result once." A sequentially-monitored experiment reads the metric continuously and can stop early. Chapter 04 covers when each is valid. For this chapter's ramp, the default is fixed-horizon at the full-power step: the ramp *advances* on guardrails but the *primary-metric decision* is read once at the end.

### The 50 %-to-100 % question

The ramp usually stops at 50 %. Moving from 50 % to 100 % is not part of the *experiment* — it is the rollout. The confusion appears when a team reads a green result at 50 % and, without a decision meeting, ramps to 100 % "since we already know." Two things go wrong when you do this: the change of exposure alters the balance of guardrails you have been monitoring (novelty effects, chapter 02, kick in when the treatment reaches users who were previously in control), and the rollout ceases to be reversible in one flag flip. Discipline: 50 % → decision meeting → rollout, with the same kill-switch discipline as chapter 03 of mod-305.

## §6 — Stopping rule and the ship/don't-ship decision

The stopping rule is written *before* the experiment starts. A workable rule for a fixed-horizon design:

- **Ship** if the primary-metric CI is above zero at the α agreed in advance, *and* every guardrail is non-inferior at its threshold, *and* every slice from the mod-305 slice matrix is non-inferior (or better).
- **Don't ship** if the primary-metric CI includes zero and the upper bound is below the MDE (a true null) or if any guardrail is red.
- **Iterate** if the primary-metric CI includes zero and the upper bound is above the MDE (the experiment was underpowered — extend or redesign, do not conclude).
- **Rollback and investigate** if a trust diagnostic (chapter 02) fired during the run; the experiment is invalid until the diagnostic is explained.

Every ship/don't-ship decision is recorded — the experiment ID, the design (MDE, α, power, ramp), the observed lift and CI on each metric, the guardrail readings, the trust-diagnostic status, the decision, the decision-maker. The record is what the postmortem calendar (mod-305 chapter 03) reads a month later to check "did offline predict online" and to grow the harness.

## §7 — What this chapter deliberately does not cover

The design contract this chapter installs is the **skeleton** — hypothesis, metrics, MDE, randomisation, ramp, stopping rule. The chapters ahead deepen specific slots:

- **Chapter 02** owns the trust diagnostics — SRM, cross-arm parity, interference, novelty — that decide whether an experiment is *readable at all*.
- **Chapter 03** owns variance reduction (CUPED / CUPAC). A CUPED-adjusted metric has a smaller σ and therefore a smaller MDE at fixed N; the design contract picks up variance reduction as an input.
- **Chapter 04** owns sequential testing — when it is valid to peek at an experiment early. If sequential testing is in play, the stopping rule of §6 gets a different mathematical shape.
- **Chapter 05** owns causal inference when an A/B test is not possible. The design contract of this chapter is what you write when you *can* run the experiment; chapter 05 is what you reach for when you cannot.

## Summary

An A/B experiment is a contract with seven load-bearing pieces: a hypothesis, a primary metric, a guardrail set, a power/MDE/α design, a randomisation unit and traffic-split plan, a ramp plan, and a stopping rule. Power is not a courtesy — it is the reason a flat result is a null result rather than an inconclusive one. Guardrails carry non-inferiority thresholds and gate the release alongside the primary. The randomisation unit is at least as coarse as the analysis unit or the standard errors are wrong. The ramp trades blast radius for power in named steps; the stopping rule is written before the experiment starts, not tuned to the observed results. Chapters 02–04 deepen specific slots inside this skeleton; chapter 05 is what you reach for when the experiment cannot be run at all.
