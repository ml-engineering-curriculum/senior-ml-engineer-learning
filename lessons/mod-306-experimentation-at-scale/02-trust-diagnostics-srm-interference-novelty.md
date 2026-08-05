# Trust diagnostics: SRM, interference, and novelty

## Motivation

Chapter 01 designed an experiment that *could* answer the question — the right primary metric, the guardrail set, the MDE, the ramp. This chapter is the discipline that decides whether an experiment's numbers are *readable at all*.

The single most expensive failure mode in an experimentation program is the invisible one: an experiment that appears to have shipped a 2 % lift, actually shipped a bug in the bucketing service, and taught the team the wrong lesson about what works on their product. Ron Kohavi's [*Trustworthy Online Controlled Experiments*](https://experimentguide.com/) and Fabijan et al.'s [*Diagnosing Sample Ratio Mismatch in A/B Testing: A Lightweight Approach*](https://www.microsoft.com/en-us/research/publication/diagnosing-sample-ratio-mismatch-in-a-b-testing-a-lightweight-approach/) both single this out as the *load-bearing* trust check — the one an experimentation platform must run automatically on every experiment, block on when it fires, and treat as a signal that the experiment's other numbers are, until proven otherwise, unreliable.

Three failure modes account for most of the readability problems in mature programs. Each has a diagnostic; each has a mitigation:

- **Sample Ratio Mismatch (SRM)** — the split you designed is not the split you observed. The bucketing is broken, the log pipeline is dropping treatment users, or the analysis is filtering unevenly.
- **Interference (SUTVA violation)** — the treatment on one unit affects the outcome of another. Marketplace effects, social effects, budget cannibalisation.
- **Novelty and primacy** — the observed effect is the *reaction to change*, not the treatment's steady-state impact. Users click the new thing because it is new; the effect fades after a week.

None of the three is a statistics problem the analysis fixes after the fact. Each is a *design* problem the experiment platform diagnoses and either mitigates upstream or refuses to release on. This chapter is how a senior ML engineer builds the diagnostic layer that makes the experiment trustworthy.

## §1 — Sample Ratio Mismatch (SRM)

### What SRM is

Every experiment declares a design split — 50/50, 90/10, 95/5. Every experiment observes an *actual* count of units in each arm after randomisation. SRM is the statistical claim: the observed split diverges from the design split by more than random chance can explain.

The standard test is a **chi-squared goodness-of-fit** on the arm counts against the design ratio. For a 50/50 experiment with `n_control` and `n_treatment` observed:

```python
from scipy.stats import chisquare

expected_ratio = [0.5, 0.5]
observed = [n_control, n_treatment]
expected = [sum(observed) * r for r in expected_ratio]
chi2, p = chisquare(observed, expected)

if p < 0.001:
    raise SRMDetected(p, observed, expected)
```

The threshold is typically **p < 0.001** or **p < 0.0001** — much stricter than the α of the primary-metric analysis, because SRM is a *pre-condition* for reading the metric, not a hypothesis test on its own. Fabijan et al. and the Microsoft ExP platform documentation both use p < 0.0005 as a common floor. The point of the strict threshold is not statistical purity — it is that a genuine 50/50 randomisation across a large-N experiment produces a p above 0.001 by chance on the order of 0.1 % of experiments; a real bucketing bug produces p far below 10⁻⁶.

### Why SRM matters

**When SRM fires, every other number in the experiment is suspect.** A 50/50 experiment whose observed split is 50.3/49.7 has *nothing wrong with the metric* — it is well within noise. A 50/50 experiment whose observed split is 51.5/48.5 with tens of millions of users has an *upstream problem*: whatever mechanism is producing that skew is almost certainly also skewing *which* users end up in each arm, which is exactly the property randomisation was supposed to guarantee. Kohavi et al.'s [*Diagnosing Sample Ratio Mismatch in A/B Testing*](https://www.microsoft.com/en-us/research/publication/diagnosing-sample-ratio-mismatch-in-a-b-testing-a-lightweight-approach/) documents case studies where an SRM led to a *reported* 10 %+ lift that vanished when the bucketing bug was fixed.

The rule: **SRM fires → experiment is invalid → root-cause the SRM before reading any other metric.** Not "we saw SRM but the primary lift is huge so we shipped anyway." Not "we saw SRM but the guardrails look fine." SRM is a *reason* the other numbers are wrong, not a datum to be weighed against them.

### Why SRM appears (the taxonomy)

Every mature program collects a taxonomy of SRM causes. Fabijan et al. list common ones; the taxonomy that ends up in an internal runbook usually includes:

- **Bucketing bugs.** The hash function is not uniform; the salt collides with another experiment; the bucketing service uses different randomness on different code paths.
- **Log-pipeline drops.** The event pipeline drops some treatment events (a treatment code path throws a swallowed exception before logging; a downstream schema mismatch drops one arm's rows).
- **Filter asymmetry.** The analysis filter (`WHERE session_length > 0`) evaluates differently for the two arms because the treatment changes what "session_length > 0" means.
- **Redirect / opt-out leakage.** Users in the treatment arm are more (or less) likely to close the page, uninstall the app, or hit a redirect that removes them from the eligible set.
- **Bot / crawler asymmetry.** The treatment changes page structure enough that a bot's crawl pattern differs; bot traffic is systematically over- or under-represented in one arm.
- **Trigger analysis.** The experiment "triggers" only when a user encounters the treated surface; the trigger condition itself depends on user behaviour that the treatment influences.

The senior read: SRM is almost never a randomisation problem in the randomiser itself (bucketing services are usually well-tested); it is almost always a problem in the *event collection or analysis path* around the randomisation. That is where the investigation starts.

### Wiring SRM into the platform

- **Automatic on every experiment.** Not "we check when the analyst runs the notebook." Every experiment reports SRM at every ramp step, and the ramp cannot advance if SRM has fired.
- **Reported alongside every metric.** The experiment scorecard has SRM at the top of the page, not in an appendix.
- **A runbook for triage.** When SRM fires, a fixed sequence — count events per arm, check the bucketing service telemetry, check the log-pipeline drop rate per arm, check the trigger condition — that reduces the mean-time-to-cause. The runbook is a repo artefact; every incident that finds a new class of SRM adds a step.

## §2 — Interference (SUTVA violations)

### What SUTVA is and where it breaks

The Stable Unit Treatment Value Assumption (SUTVA) says: the outcome for a unit depends only on that unit's treatment, not on any other unit's treatment. Every off-the-shelf A/B analysis assumes it. Rubin's [*Comment on "Randomization Analysis of Experimental Data: The Fisher Randomization Test"*](https://www.jstor.org/stable/2287653) is the original statement; every experimentation textbook restates it.

SUTVA breaks whenever the *treated units affect the control units through some shared mechanism*. The three canonical shapes in an ML/product context:

- **Marketplace / two-sided platforms.** In a ride-share, a hotel-booking site, or an ad auction, the treatment on one side of the market changes the supply seen by the other side. Blake and Coey's [*Why Marketplace Experimentation Is Harder to Design and Analyze*](https://sites.google.com/site/danielkcoey/marketplace-experimentation) documents the failure mode: a treatment that steals demand from control listings shows a large positive effect on treatment units and a negative effect on control units the analysis does not see.
- **Network / social effects.** A social-graph experiment that shows one user's feed differently changes what that user shares, which changes what their friends (some of whom are in control) see. Ugander, Karrer, Backstrom, Kleinberg's [*Graph cluster randomization: network exposure to multiple universes*](https://arxiv.org/abs/1305.6979) is the classical treatment.
- **Shared-budget cannibalisation.** Two experiments (or a treatment and a control) share a scarce resource — an inventory quota, an ad budget, a rate-limited vendor call. A treatment that consumes more of the shared budget starves the control.

### What interference does to your numbers

The standard difference-in-means estimator is *biased* under interference — usually in a way that *inflates* the apparent treatment effect (treatment cannibalises control, so treatment−control overstates the true effect the intervention would have at 100 % rollout). Blake and Coey's estimation on eBay data showed marketplace interference could account for a large fraction of the naive lift on a rollout. The senior read: **if SUTVA is broken, the number your analysis reports is not the number you would see at 100 % rollout, and the *direction* of the bias depends on the mechanism.**

### Diagnosing interference

Interference is not a p-value; it is a design consideration. The diagnostic is a *checklist*:

- **Do the treated and control units share a resource, market, or graph?** If so, interference is *possible*.
- **Does the treatment change the resource / market / graph state?** If so, interference is *likely*.
- **Is the treatment large enough (a big feature, a pricing change, a ranking change) that the second-order effect on non-treated units is material?** If so, interference is *material* and the analysis needs to account for it.

Two useful empirical checks:

- **Ramp comparison.** Compare the observed lift at 1 %, 5 %, 25 %, 50 %. If the treatment effect *shrinks* as exposure grows, interference is a plausible explanation: at 1 % exposure the control is barely affected; at 50 % the control is heavily cannibalised, and the "gap" collapses.
- **Placebo tests.** Run an A/A test (both arms identical) on the same infrastructure at the same time. Any observed "effect" is a lower bound on the noise-plus-interference floor.

### Mitigating interference by design

- **Cluster randomisation.** Randomise at a unit that contains the interference — a city (for ride-share), a market (for search-and-buy), a friend-of-friend cluster (for social). The Ugander et al. paper is the canonical reference. Cluster randomisation costs statistical power (the effective sample size is the number of clusters, not the number of units), so pick the coarsest unit *sufficient* to enclose the interference, not coarser.
- **Switchback experiments.** Alternate the entire system between treatment and control on a time-window schedule (e.g., 30-minute windows in a city). Every user experiences both arms; the analysis attributes to the window. Common in marketplaces where cluster sizes are small. Sneider, Tang, Zhu's [*Experiment Rigor for Switchback Experiment Analysis*](https://engineeringblog.yelp.com/2015/09/experiment-rigor.html) and DoorDash's [switchback engineering write-ups](https://careersatdoordash.com/blog/switchback-tests-and-randomized-experimentation-under-network-effects-at-doordash/) are industry-oriented treatments.
- **Ego-network / two-hop randomisation.** For social features, treat all users in a chosen ego-network the same. Bakshy, Eckles, Bernstein's [*Designing and Deploying Online Field Experiments*](https://arxiv.org/abs/1409.3174) discusses several designs Facebook has used.
- **Budget isolation.** For shared-budget experiments, allocate a separate budget slice to the treatment so the two arms cannot cannibalise. Common in ad-platform experimentation.

The senior read: **interference is a designed-for problem, not a corrected-for one.** The correction (an unbiased estimator under known interference) exists in the literature but is fragile and rarely applied at the same statistical power as a well-designed cluster randomisation. Design the experiment so SUTVA holds (or holds approximately) at the randomisation unit, then analyse as usual.

## §3 — Novelty and primacy effects

### What they are

**Novelty effect**: users engage with the treatment because it is *new*, not because it is *better*. The measured lift is a decaying transient. The steady-state effect is smaller — sometimes zero, sometimes negative.

**Primacy effect** (the opposite): users engage *less* with the treatment because it is unfamiliar. The measured effect is a decaying negative transient. The steady state is neutral or positive.

Both are covered in Kohavi/Tang/Xu chapter 22 and are named across the industry literature (e.g., Microsoft's ExP posts, Google's *Overlapping Experiment Infrastructure*). The mechanism is the same: the user population's response to a change is not stationary over the experiment window.

### Why they matter

An experiment run for two days on a treatment with a novelty effect can report a 4 % lift that is actually 0 % at steady state. The team ships; a month later engagement is flat vs. pre-launch; the "success" is a phantom. Novelty is one of the most common causes of "the offline win became an online null a month later."

### Diagnosing novelty and primacy

- **Effect over time.** Plot the treatment effect on the primary metric per day (or per week). A stationary treatment produces a roughly flat effect line (with confidence bands). A novelty-affected treatment produces a *decaying* effect line — day 1 shows a large lift, day 7 shows less, day 14 shows very little.
- **Cohort by exposure recency.** Split treated users into "first-week exposure" vs. "second-week exposure" vs. "third-week exposure" and compare their metrics *at the same exposure age*. If the older cohorts' effects are smaller than the newer cohorts', that is the novelty pattern; if the newer cohorts underperform, that is the primacy pattern.
- **Returning-user analysis.** Compute the effect only on users who were exposed at least N days ago and are still active. If this "well-adapted" cohort shows a much smaller effect than the fresh cohort, novelty is likely.

### Mitigating novelty

- **Run long enough.** Extend the experiment window to a duration where the day-over-day effect has stabilised. Kohavi/Tang/Xu recommend at least two weeks for most consumer experiments, longer for anything that changes UX substantially.
- **Report the steady-state estimate separately.** If your experiment is 14 days long, report the primary-metric effect on the last 7 days as the "steady-state" estimate alongside the full-window estimate. If they diverge materially, name the divergence.
- **Pre-registered decay model.** For treatments where novelty is expected (a new visual, a new UX pattern), pre-register a decay model (e.g., exponential decay to a plateau) and fit it. The plateau is the steady-state estimate.
- **Hold-back cohorts.** After launch, retain a small hold-back cohort in the control arm for weeks or months. The lift measured against the hold-back at 30 or 90 days is the steady-state estimate. Long-hold-back cohorts are how mature programs answer "does this launch still work" a year later.

### The complementary trap — over-diagnosing novelty

A team burned by novelty once tends to over-attribute *any* decay to novelty. But an effect that shrinks over the experiment window is also the shape of an *underpowered stopping rule* (the trend is noise), an *interference effect* (chapter §2), or a *policy change* that landed midway through the experiment. The diagnostic is not "does the effect line slope downward" — it is "does the effect *plateau* at a value materially below the initial reading, and does the plateau replicate on the returning-user cohort." Anything else is a hypothesis, not a diagnosis.

## §4 — What a trust scorecard looks like

Every experiment produces a scorecard. The trust section sits *above* the primary and guardrail metrics because reading the primary is only meaningful once trust is green. A workable shape:

```
Experiment: exp_2026_08_new_ranker_v3
Status: SHIPPING DECISION — DO NOT READ, TRUST FAILURES BELOW

Trust diagnostics
  SRM (chi-squared, design 50/50, threshold p<0.001)
    observed:  n_c=8_412_311, n_t=8_487_004
    p = 3.2e-8                             FAIL
    → runbook link, on-call paged
  A/A parallel (identical arms, 10% traffic)
    primary lift 0.02 [-0.4, 0.4] pp       PASS
  Cross-arm error-rate parity (5xx per 1k req)
    control 0.14, treatment 0.17           PASS (within tolerance)
  Trigger analysis
    trigger rate control 92.1%, treatment 91.8%  PASS
  Novelty check (day-1 vs day-7 primary lift)
    day-1 +2.4 pp, day-7 +2.1 pp           PASS
  Interference check (per-ramp-step lift)
    1% → +2.3, 5% → +2.4, 10% → +2.3       PASS
Primary metric (SUPPRESSED until trust green)
Guardrail metrics (SUPPRESSED until trust green)
```

The suppression is deliberate: a scorecard that shows a green primary next to a red SRM invites the team to read the primary anyway. Hiding the readings until trust is green is a *policy* an experimentation platform enforces, not a suggestion the analyst can override.

## §5 — Where each diagnostic hands off

- **SRM → the experiment platform team.** Fixing SRM usually requires changes to the bucketing service or the event pipeline. The senior ML engineer's job is to *know* the diagnostic exists, to *demand* it on every experiment, and to *refuse to ship* on an experiment where it fired.
- **Interference → the experimental design.** The senior ML engineer's job is to *notice* when the feature is on a shared-resource or graph surface (marketplace, social, budgeted) and to *push for the right randomisation unit* before the experiment starts. Cluster randomisation is a decision made in the design meeting, not applied after the fact.
- **Novelty → the experiment duration and reporting.** The senior ML engineer's job is to *pre-register* the steady-state read and the decay model, and to *build the hold-back cohort* into the rollout plan for anything UX-substantial.

## §6 — Two failure modes the diagnostics catch

- **"The lift is real."** SRM at p = 1e-10 in a 50/50 experiment; primary lift +4 %. Team ships; a week later the bucketing bug is found; the actual effect is +0.1 %. The ship was noise, and now the team has learned the wrong lesson about the ranker. SRM at the top of the scorecard, with the primary metric *suppressed* until trust is green, is what would have caught this.
- **"The lift is a launch honeymoon."** New feature ships with a large day-1 lift; steady-state effect at day-30 is a small loss. The experiment window was too short to see the plateau; no hold-back cohort was retained. A novelty check on day-1 vs. day-7 and a 5 % hold-back through day-30 would have caught it. This one is common enough that mature programs treat any UX-changing experiment without a hold-back plan as a design defect.

## Summary

An experiment's primary metric is only readable once the trust diagnostics are green. Sample Ratio Mismatch is the load-bearing check every experimentation platform must run on every experiment, block on when it fires, and treat as a reason the other numbers are wrong. Interference (SUTVA violations) is a *designed-for* problem — cluster or switchback randomisation is the mitigation for marketplace, network, and shared-budget surfaces. Novelty and primacy effects are diagnosed with day-over-day effect plots, exposure-cohort splits, and returning-user analyses; the mitigation is a longer window, a steady-state re-read, and a hold-back cohort in the rollout. Every experiment produces a scorecard where the trust section sits above (and, if red, suppresses) the metric readings. Chapter 03 is where variance reduction lowers the MDE inside a *trustworthy* experiment; chapter 04 covers when it is valid to peek at an experiment early; chapter 05 covers what to do when the experiment cannot be run at all.
