# Shadow and canary: how an offline win earns online promotion

## Motivation

The candidate passed the harness. Every slice green, every guardrail green, primary metric lift with a CI comfortably clear of zero. Ship it? At L20 the answer is "yes." At L30 the answer is "no, not yet — the online path has three things to check that the offline harness cannot see."

Chapter 01 named the offline / online divergence in the abstract. Chapter 03 is where the online path is authored so that the divergence *cannot ship as a surprise*. Nothing gets a full rollout on offline signal alone — every candidate walks a promotion path where an offline win has to earn each subsequent stage. The stages, in the order every mature ML org runs them:

```
offline harness  ──►  shadow  ──►  canary  ──►  A/B or interleaving  ──►  full rollout
   (chapter 01)     (this ch, §1)  (§2)          (§3, mod-306 depth)      (§4)
```

Every stage has a **question it is uniquely qualified to answer** and a **failure mode it uniquely catches**. Running fewer stages does not save time; it saves time until an incident, at which point the incident is more expensive than every stage combined.

This chapter walks each stage: what it is, what it uniquely proves, what it uniquely misses, and how the harness decision from chapter 01 is the precondition for entering it.

## Prerequisite: the harness decision, again

The offline harness (chapter 01) produced a `HarnessDecision` with `passes_release_gate=True`. **That is a precondition of shadow, not a substitute for it.** If the harness decision is not green, the candidate does not enter shadow. If someone wants to override that rule — because "the failing slice does not matter" or "the latency guardrail is a false alarm" — the override is a signed acknowledgement, filed with the release, that a specific gate was consciously waived. Overrides are visible in the release record. They are not a way to make a red gate green by pretending it was not red.

Chapter 04 (ML Test Score) formalises this rule as *Test 25 — Model quality is sufficient on all important data slices*: if you cannot show the gate went green, you have not passed the test.

## §1 — Shadow deploy

### What it is

The candidate model runs **alongside** the current production model. Every production request is routed to both. The current model's response is what the user sees. The candidate model's response is discarded — logged, compared against the current model's response and against ground truth when ground truth is available, but never returned to the user. Google's [SRE Workbook](https://sre.google/workbook/canarying-releases/) and Chip Huyen's [*Designing Machine Learning Systems*](https://huyenchip.com/books/) both cover this as *dark launch* or *mirroring*.

The candidate is running on production **inputs**, on production **hardware**, with production **latency budgets**, at production **volume** — everything except the "what the user sees" bit is exactly what it would be at 100 % rollout.

### What shadow uniquely proves

- **Latency and cost under real load.** Offline latency (chapter 02 guardrails) is a floor; production latency is the truth. Coordinated omission, cache-warmth, cold-start behaviour, actual p99 tail — all live-visible in shadow. Gil Tene's [*How NOT to Measure Latency*](https://www.infoq.com/presentations/latency-response-time/) is the reference on why offline latency measurements systematically under-report.
- **Error rates and the failure taxonomy.** Real inputs surface real edge cases. The candidate that had 0 % error on the eval set may have 0.3 % error on a shape of input the eval set did not contain. Timeouts, 500s, schema-validation failures, degraded-response paths.
- **Distribution shift, made concrete.** A per-slice histogram of production inputs vs. the eval-set slice distribution shows exactly *how far* the eval distribution is from today's. If the eval distribution said 8 % of traffic was new-visitor slice and the production traffic shows 22 %, the candidate's offline pass on new-visitor slice deserves a much closer look.
- **Serving-path correctness.** The candidate is loaded through the actual model-registry code path, the actual feature-fetch pipeline, the actual downstream schema; anything that "works in the notebook" but fails in serving is a shadow-first defect.

### What shadow cannot prove

Shadow reveals nothing about **user behaviour**. The candidate's response never reaches a user, so no click, no conversion, no downstream action changes. If your feature's value is in how users respond — the recommender, the ranker, the classifier that gates a UX flow, the summariser whose summary is read — shadow proves the pipes work but *not* that the model is better for the product. That is what canary (§2) and A/B (§3) are for.

Two special cases where shadow is *even more useful than usual*:

- **When ground truth is available on the request path.** For a classifier where the label lands within seconds or hours (an ad click, a purchase decision, a support-ticket resolution), shadow lets you compute an *online* primary metric on the same requests both models handled. This is the strongest signal short of a live A/B.
- **When the request path is expensive.** For LLM-augmented systems (mod-304), shadow is the first place per-request cost at production scale is visible. Aggregate cost, cost-per-slice, cost tail — all live under real query patterns for the first time.

### Setting shadow up honestly

Two anti-patterns kill the shadow signal:

- **Sampling shadow to save cost.** If shadow only runs on 10 % of requests, the tail of the p99 and the tail of the cost distribution are systematically under-represented. Run shadow at 100 % if the cost budget allows; if it does not, sample stratified across the slice matrix (chapter 02) so at minimum every business-critical and fairness slice is fully covered.
- **Comparing shadow to a stale eval baseline.** The comparison is shadow-candidate vs. the *currently-serving production* model on the *same* request, not vs. the harness baseline from three months ago. The pair is what unlocks the paired statistics from chapter 01 online.

### Exit criteria — what a shadow run needs to show before canary

- Latency: candidate p50/p95/p99 within the SLO for the deploy (mod-307 covers SLOs). "Within" is a specific number, not "roughly."
- Error rate: candidate error rate ≤ baseline within CI on shadow traffic.
- Cost: candidate per-request cost within the target envelope (mod-304 chapter 04).
- Serving-path correctness: no schema validation failures, no downstream-consumer errors, no unexpected null responses.
- Where ground truth lands: online-primary-metric lift within the offline CI, or within an agreed-in-advance tolerance.

Any of the five failing means shadow is not passed. Chapter 04's *Test 20 — Model quality is sufficient on production data* is asking exactly this question.

## §2 — Canary

### What it is

The candidate is exposed to a **small fraction** of live traffic — 1 %, 5 %, 10 % — and its responses go to the users routed to it. The rest of traffic sees the production model. Traffic split can be random (per-request), sticky (per-user), or slice-selected (only enterprise traffic, only English, only a specific region). Google's [SRE Workbook — Canarying Releases](https://sre.google/workbook/canarying-releases/) is the canonical reference on the pattern.

Canary is the first stage where **user behaviour actually changes for someone**. The stakes are correspondingly higher: a bad canary means real users have a worse experience for as long as the canary runs. Every canary carries an *auto-rollback* condition. Every canary has a maximum duration.

### What canary uniquely proves

- **The candidate does not blow up under partial production exposure.** New failure modes — an unusual request shape, a downstream consumer that cannot parse the candidate's slightly different output, a rate-limit interaction with a vendor — surface only when the candidate is *serving* users, not shadowing.
- **First real signal on feedback-loop behaviour.** For a recommender or a ranker, canary is the earliest moment the candidate's decision affects the *next* observation (the click that changes the next impression, the item shown that changes the next query). Shadow was blind to this.
- **First look at product-KPI direction.** Canary is a small-sample A/B: for a canary at 5 % of traffic over a day, you have very roughly 5 % of the daily conversion, engagement, or downstream-KPI data. Not statistically decisive (that is the full A/B in §3), but directional. If the canary is *dramatically* worse on the product KPI, that is a rollback signal, not a "wait for the full A/B" signal.

### Guardrails and auto-rollback

Every canary runs behind an *auto-rollback controller*. The controller watches a small set of signals — the guardrails — and if any of them crosses a threshold for longer than a hold-time, traffic is instantly routed back to the baseline and the on-call is paged. Sculley et al.'s [*Hidden Technical Debt in Machine Learning Systems*](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf) names this as one of the load-bearing pieces of production ML infrastructure; Michael Nygard's [*Release It!*](https://pragprog.com/titles/mnee2/release-it-second-edition/) generalises the pattern from the resilience literature.

The rollback signals are a superset of shadow's exit criteria and add:

- **Product KPI direction.** A dramatic direction change on the primary product KPI (conversion, engagement, error rate) is a rollback signal. The threshold is agreed in advance, not eyeballed live.
- **Business-critical slice regression.** The slice matrix from chapter 02 is monitored live on canary traffic. A dramatic per-slice regression is a rollback signal.
- **Safety and refusal-rate signal (LLM-augmented).** A large shift in refusal rate, safety-classifier trigger rate, or downstream-moderation queue volume is a rollback signal.
- **Cost overrun.** Candidate per-request cost exceeding the envelope by more than a small tolerance (mod-304 chapter 04).
- **Vendor-side outage or degradation.** A candidate that depends on a vendor whose status page is red should not be exercising a canary — chapter 04's *Test 18 — Model rollback is fast* is essentially "you can turn this off in one action."

### Traffic-split choices — random, sticky, slice-selected

- **Random per-request.** Simple, unbiased, but gives a single user a mix of models across requests. For a recommender, this is behaviourally strange and confounds session-level signals. Fine for stateless per-request features (a classifier decision that does not persist).
- **Sticky per-user.** Every user in the canary bucket sees the candidate consistently. This preserves session-level signal and makes the canary a small-sample A/B. Standard for user-facing product features.
- **Slice-selected.** Canary exposes only a specific slice — one region, one plan tier, one locale. Reduces blast radius when the release is *specifically* aimed at improving that slice; not a substitute for the full canary before broader rollout.

The choice affects statistical inference. Mod-306 (experimentation) covers sticky-vs-random and the variance-reduction techniques that go with each. For this chapter, the rule is: **decide the split before starting the canary, name it in the release plan, do not change it midway.**

### The escalation ladder inside canary

Canary is not one stage but several. A common ladder:

- 1 % of traffic, 30 minutes → check guardrails.
- 5 % of traffic, 2 hours → check guardrails + first-look at product KPI.
- 10 % of traffic, 24 hours → business-day exposure across time zones.
- 25 % of traffic, 24–72 hours → enough sample for a per-slice product-KPI look.

Every step is gated on the previous step's guardrails staying green. The times shorten for teams with high-volume products and lengthen for regulated deployments. The point is that the ladder is *predefined*: nobody negotiates "let's just go straight to 50 %" mid-release.

## §3 — A/B testing and interleaving

Canary at 25 % that has stayed green for a business cycle is not the same thing as *proven better*. The candidate might still be flat on the primary product KPI at usable statistical power. The stage that answers "is the candidate actually better" is the A/B test.

### A/B testing — the gold standard for user-facing releases

A/B testing is a randomised controlled experiment: users are split into treatment (candidate) and control (baseline) at some ratio (50/50, 90/10, 5/95), assigned stably, and observed for a period long enough to reach the statistical power the release conversation agreed on. Chapelle, Manavoglu, Rosales's [*Simple and Scalable Response Prediction for Display Advertising*](https://olivier.chapelle.cc/pub/tist_predictor.pdf) is the canonical modern reference for the industrial-scale variant; Ron Kohavi, Diane Tang, Ya Xu's [*Trustworthy Online Controlled Experiments*](https://experimentguide.com/) is the modern textbook.

Full depth on A/B testing lives in [mod-306] (experimentation at scale). For this chapter, the load-bearing pieces:

- **Metric definition upfront.** Primary product KPI, guardrail metrics, and any secondary KPIs are *listed in advance*, before the experiment starts. No metrics-shopping post hoc.
- **Statistical power.** Sample size is picked to detect the smallest effect that would matter to the business, at the significance level and power agreed in advance. Small experiments detect nothing except very large effects; a "flat" A/B may just be under-powered.
- **Guardrails on the harness slices.** The offline slice matrix (chapter 02) is the online slice matrix: the release is a win when the primary metric is up *and* no business-critical slice is regressed *and* no fairness slice is regressed.
- **Interaction effects.** If multiple experiments are running simultaneously (as they always are in a mature product), the mutual-exclusion, layering, or CUPED variance-reduction machinery (mod-306) is what makes the results interpretable.

### Interleaving — the ranker-specific accelerator

For search and recommender rankers, **interleaving** is a technique that mixes the two rankers' outputs into one ranked list per query and observes user preference at the click level. Radlinski, Kurup, Joachims's [*How Does Clickthrough Data Reflect Retrieval Quality?*](https://www.cs.cornell.edu/people/tj/publications/radlinski_etal_08b.pdf) is the classical reference; Chapelle, Joachims, Radlinski, Yue's [*Large-scale Validation and Analysis of Interleaved Search Evaluation*](https://www.cs.cornell.edu/people/tj/publications/chapelle_etal_12a.pdf) is the industrial validation.

The reason interleaving matters for the promotion path: for rankers, it typically achieves the same statistical power as a classical A/B on 10–100× less traffic. That means candidates that would otherwise be under-powered can be evaluated meaningfully. Interleaving does not replace A/B — it accelerates the phase of the promotion where "which of these two rankers do users prefer" is the question. Full A/B still runs before rollout to confirm the product-KPI story.

## §4 — Full rollout, and the state to preserve

A green A/B is the release. The last stage is the rollout to 100 % of traffic, and the state that has to survive it:

- **The baseline record.** The prior production model, the prior harness decision, the prior A/B result. Not "we deleted the old model to save disk"; the prior model artefact is retained in the registry so a rollback is a *one-action* redeploy.
- **Kill switch and rollback rehearsal.** The team can turn the new model off in one action. Not "we would have to re-run the deploy pipeline." Not "the on-call would have to page the ML engineer." One flag, one config value, one action. Chapter 04's *Test 18* is exactly this.
- **The postmortem calendar.** A week or two after rollout, a look at the online metrics against the offline predictions. Where did offline and online agree? Where did they disagree? Every gap is a candidate improvement to the harness — a slice to add, an adversarial case to grow, a metric to add to the harness. This is how the harness gets better over time.

Chapter 01 said the harness produces a decision object. Every full rollout produces a postmortem entry. Both live in the release history.

## Two failure modes the promotion path catches

- **"The offline win is real; the online serving pipe is broken."** Candidate passes the harness, gets shipped straight to 100 % on offline signal alone, and 4 % of requests fail with a schema-validation error because the feature-fetch code path in serving isn't quite the same as the offline evaluation code path. Shadow (§1) catches this before a user sees it. Every team learns this one at least once.
- **"The offline win was on the wrong metric."** Candidate lifts NDCG offline; canary and A/B show engagement flat and downstream conversion down. The candidate is what the offline harness asked for; the offline harness was aimed at the wrong target. Rollback fast, add engagement / conversion signals to the online part of the promotion path in future, revisit the primary metric in the harness.

## Where this hands off

- **mod-306 (experimentation at scale)** is where A/B testing is done at senior depth — CUPED, sequential testing, mutual exclusion, multi-armed rollout. This chapter's altitude is "here is the promotion ladder"; mod-306's altitude is "here is how the A/B stage is done rigorously."
- **mod-307 (ML reliability, SLOs)** owns the runbook the on-call reads when a canary rolls back. The SLI vocabulary in mod-307 is what the auto-rollback controller thresholds are named against.
- **mod-309 (responsible AI)** owns the review that the slice matrix is enforced in the online promotion path, not just offline. The slice gate lives in *both* places.
- **mod-304 chapter 04** owns the cost / latency envelope the shadow and canary stages are measuring against.

## Summary

An offline harness pass is a *precondition* of promotion, not a substitute for it. Every candidate walks a fixed ladder — shadow (does the serving pipe work at production load), canary (does the candidate not blow up under partial exposure and are guardrails green), A/B or interleaving (is the candidate actually better on the product KPI), full rollout (with a rehearsed rollback). Each stage has predefined entry and exit criteria; each stage has an auto-rollback contract with metrics named in advance; each stage is designed to catch a failure mode the previous stage cannot see. Chapter 04 (ML Test Score) is the review packet that says the whole ladder is fit for purpose; mod-306 is where the A/B stage is done at experimentation-track depth; mod-307 is where the SLIs the rollback thresholds are named against live.
