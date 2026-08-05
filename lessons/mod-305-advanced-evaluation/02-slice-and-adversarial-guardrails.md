# Slices, adversarial suites, and guardrails

## Motivation

Aggregate metrics lie. Not always — but often enough that a senior ML engineer treats an aggregate number as the *start* of an evaluation conversation, not the end.

A ranker whose NDCG is up 0.5 points overall may have gotten there by lifting logged-in users (the majority slice) while regressing new-visitor sessions (the slice product cares about most). A classifier whose F1 is unchanged may have quietly shifted its error concentration from one demographic to another. A summariser whose ROUGE score is flat may be silently emitting a new class of hallucination on customer-facing tickets — a class the aggregate never sees. Buolamwini and Gebru's [*Gender Shades*](http://proceedings.mlr.press/v81/buolamwini18a/buolamwini18a.pdf) is a canonical demonstration for face classification; the same shape shows up wherever a model with high aggregate accuracy has sub-populations it is bad at. Sagawa et al.'s [*Distributionally Robust Neural Networks for Group Shifts*](https://arxiv.org/abs/1911.08731) framed the same failure mode as worst-group generalisation.

The harness from chapter 01 is the frame. This chapter is what goes inside the frame — the **slice matrix** that catches unequal quality across sub-populations, and the **adversarial suite** that catches the model's known-bad behaviour before it re-ships. Both are grown, not spawned; a team ships a first version on day one, then grows it every incident, every launch review, every fairness audit.

## Two failure classes, two mechanisms

- **Slice failures.** The candidate is worse for a sub-population that matters — a customer segment, a regulator-visible group, a language, a locale, a device class. The aggregate metric hides it because the sub-population is small or because gains elsewhere out-weigh the regression. The mechanism to catch it is **per-slice pass rates** — the harness computes the primary metric *per slice* and gates each slice independently.
- **Regression failures.** The candidate re-introduces a specific bad behaviour the model has seen before — the fraud model that mislabels a specific known-benign merchant, the summariser that emits the marketing team's forbidden phrase, the recommender that surfaces a policy-violating category. The mechanism to catch it is an **adversarial / regression suite** — an accumulating list of known-bad cases that must be handled correctly, gated with a hard pass rate that never drops.

Neither mechanism replaces the primary eval set. They are *additional* gates the harness in chapter 01 composes with an `AND`.

## Slices, in three tiers

Not every slice is worth gating. If the harness gates on 300 slices, the gate is meaningless — some slice will always regress by noise. The discipline is to pick the slices that *actually* map to a business, safety, or regulatory decision, and to explicitly name the ones you are not gating.

### 1. Business-critical slices — the ones the product manager can name

These are the slices where a regression is a business problem. The product manager can name them without help. Examples that show up across domains:

- **The high-value segment.** Enterprise customers, top-quintile spenders, top-decile advertisers.
- **The onboarding funnel.** New users; users still in their first week; the trial-to-paid conversion cohort.
- **The high-recall-required segment.** Fraud events; safety-content moderation; a regulated compliance case.
- **The primary language / locale / device.** English-mobile is not English-desktop is not French-mobile.

A slice belongs on this tier when a product owner will notice the regression *and* when the size of the slice is large enough to compute the primary metric with usable statistics. Chapter 01's bootstrap CI applies per-slice; a slice with 50 rows carries a CI so wide that any decision on it is noise. State the minimum row count in the eval config (a rough rule of thumb is that a per-slice CI of the primary metric should be no more than half the release threshold; sample sizes below that require raising the row count, aggregating slices, or accepting that the slice is a *directional* signal only).

### 2. Fairness / regulator-visible slices — even when they are small

Some slices belong on the gate even when they are small — because a regulator, a policy team, or a moral responsibility says so. Demographic disaggregation (where allowed and where the labels exist), a protected-class slice, a jurisdictional slice (EU vs US when the two are subject to different rules), a language slice a specific market cares about. Chouldechova's [*Fair prediction with disparate impact*](https://arxiv.org/abs/1610.07524) and Hardt, Price, Srebro's [*Equality of Opportunity in Supervised Learning*](https://arxiv.org/abs/1610.02413) name the disaggregation metrics; Barocas, Hardt, and Narayanan's [*Fairness and Machine Learning*](https://fairmlbook.org/) is the modern textbook synthesis.

The senior read: a slice's inclusion here is a *policy* call, not a statistics call. Mod-309 (Responsible AI) is where the review packet is authored and where the slice list is signed off. The evaluation harness is where it is *enforced*. If you find yourself picking slices for the harness that mod-309's review packet does not know about, one of the two is out of date.

### 3. Error-concentration slices — discovered from the data

The third tier is not on the product manager's or the regulator's list — the harness *discovers* it from the errors of the current model. The old-school name is error analysis; the modern name is slice discovery, and there are tools:

- **Manual error analysis.** Sort errors by confidence, by segment, by feature value; look for concentrations. Andrew Ng's *Machine Learning Yearning* (freely available) is the canonical treatment. Cheap, high-signal, painful only because engineers avoid it.
- **Automated slice discovery.** Chung, Kraska, Polyzotis, Tae, Whang's [*Slice Finder: Automated Data Slicing for Model Validation*](https://arxiv.org/abs/1807.06068) and Eyuboglu, Varma, Saab, Delbrouck, Lee-Messer, Dunnmon, Zou, Ré's [*Domino: Discovering Systematic Errors with Cross-Modal Embeddings*](https://arxiv.org/abs/2203.14960) are two representative approaches; both search for feature-conjunction slices where the model under-performs.
- **Cohort-based dashboards.** Product-analytics tooling often exposes cohort views that surface the same slices — new-device cohort, new-country cohort, new-plan cohort.

An error-concentration slice that has a plausible business or user-experience story gets promoted to Tier 1 or Tier 2 and gated. A slice that has no story is worth an investigation but not a permanent gate.

### The slice matrix

The slice list is not a flat list; it is a matrix — some slices are *dimensions* (device × locale × plan × loyalty tier), and the interesting slices are often intersections. Every intersection is a slice; not every intersection needs a gate; the ones that do are named in the eval config with row counts and pass thresholds.

A workable eval-config shape:

```yaml
slices:
  overall:
    min_rows: 5000
    gate: primary_metric >= baseline
  business_critical:
    - name: enterprise_customers
      filter: {plan: enterprise}
      min_rows: 800
      gate: primary_metric >= baseline * 0.99   # tolerate no drop
    - name: new_visitors
      filter: {tenure_days: {"$lte": 7}}
      min_rows: 1200
      gate: primary_metric_ci_low >= baseline_ci_low
  fairness:
    - name: language_es
      filter: {locale: es}
      min_rows: 400
      gate: worst_group_metric >= 0.95 * overall
  adversarial:
    include: regression_suite_v14.yaml
```

The details vary by team; the two things that stay constant are **the eval config is a file in the repo** (chapter 04 tests 4 and 6 — feature-engineering code and data are versioned) and **each gate is a boolean expression a reviewer can read**.

## Adversarial suites — the regression bank

The adversarial / regression suite is a separate eval set with a different discipline: **it is grown from real incidents, and every case must be handled correctly**.

The categories worth carrying, and the entry point where each grows:

### 1. Historical-incident regressions

Every production incident where the model got a specific case wrong becomes an adversarial suite entry. The support ticket the model triaged wrong; the fraud event the model missed; the summarised message that leaked PII; the hallucinated policy the LLM invented. Post-incident, the case is added — with the expected correct behaviour, the incident ticket number, and the date. Chapter 04's *Test 5 — A simple model is not a better production model* has a companion here: every case where a simple hand-written rule would have caught what the model missed goes in the suite.

### 2. Safety and refusal cases (LLM-augmented systems)

For LLM-augmented features (mod-304), a safety suite is table stakes: prompt-injection attempts, jailbreak attempts, protected-class prompts, requests for restricted content. The OWASP [*Top 10 for Large Language Model Applications*](https://owasp.org/www-project-top-10-for-large-language-model-applications/) is a starting taxonomy; the NIST [*AI Risk Management Framework* Generative AI Profile](https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/NIST_AI_600-1.pdf) lists risk categories that also map to test cases.

### 3. Invariance and counterfactual cases

Behaviour that must not change when an irrelevant feature changes. Ribeiro, Wu, Guestrin, Singh's [*Beyond Accuracy: Behavioral Testing of NLP Models with CheckList*](https://aclanthology.org/2020.acl-main.442/) is the canonical modern reference — invariance tests (input paraphrase leaves output unchanged), directional expectation tests (adding a negation flips the output), minimum-functionality tests (specific narrow capabilities the model must have). CheckList is written for NLP but the pattern is general — a fraud model's decision should not flip when the currency-symbol formatting changes; a ranker's top result should not flip when the client-side tracking parameters change.

### 4. Calibration cases

For models where a downstream decision depends on the probability's value (mod-303 chapter 05 on calibration): a set of examples with known base rates, gated on ECE-per-slice and reliability-diagram-per-slice, not just aggregate ECE.

### 5. Out-of-distribution and drift-canary cases

Cases the deploy is *not* claiming to handle, gated on refusal behaviour or "no confident answer." A support ticket in a language the classifier was not trained on should return "no confident class" — not the argmax of a training-language distribution. A cold-start slice (mod-303 chapter 07) belongs here.

### The growth discipline

The adversarial suite is *append-only*. Cases enter with an incident number, a reviewer, and an expected behaviour; they exit only via an explicit review that the case no longer represents a real failure (e.g. a policy changed and the "wrong" behaviour is now correct). The gate is unambiguous: 100 % of cases must pass, or the harness fails. Not 95 %. Not "well the newly-added ones failed but the old ones passed."

The suite grows by roughly one case per shipped incident. In a team of ML engineers with an active release cadence, that means the adversarial suite ends the year with dozens to hundreds of cases — small, expensive to run per case, and *the most reliable of all the harness signals* because every entry has a real-incident story behind it.

## Guardrails — metrics that are not the primary metric but must not regress

Beyond slices and adversarial cases, a small number of guardrail metrics gate the harness *in addition to* the primary metric. Reilly Grant and D. Sculley et al.'s [*The ML Test Score: A Rubric for ML Production Readiness*](https://research.google/pubs/the-ml-test-score-a-rubric-for-ml-production-readiness-and-technical-debt-reduction/) (chapter 04) treats several of these as tests. The common ones:

- **Calibration.** ECE overall and per-slice, gated at a threshold the release conversation has agreed on. A model that lifts AUC by trashing calibration is not a release (mod-303 chapter 05 has the vocabulary).
- **Fairness disaggregation.** A per-slice worst-group metric — worst-group precision, worst-group recall, worst-group AUC — with a floor. Sagawa et al.'s worst-group generalisation framing.
- **Refusal / abstention rate.** For LLM-augmented systems, the rate at which the model refuses when it should refuse and answers when it should answer. Regressions in either direction fail the guardrail.
- **Cost per prediction.** For candidates where cost is an operational constraint, a per-request cost estimate against baseline. Mod-304 chapter 04 is the vocabulary; chapter 03 (this module) is where the online path picks it up as a live SLI.
- **Latency proxy.** Offline latency measured on a representative machine — a floor, not a ceiling, because the online path (chapter 03) is authoritative for the p99. Offline latency is caught here specifically to prevent a candidate that would blow the latency budget from ever reaching shadow.
- **Robustness proxies.** Accuracy on invariance-perturbed inputs, worst-case across a small set of common perturbations. Hendrycks and Dietterich's [*Benchmarking Neural Network Robustness to Common Corruptions and Perturbations*](https://arxiv.org/abs/1903.12261) is the canonical robustness benchmark for vision; the general pattern is domain-specific.

Each guardrail is a `GuardrailResult` in the decision object (chapter 01) — a metric, a threshold, and a boolean pass. `passes_release_gate` is the AND across all guardrails; a single guardrail failing blocks the release exactly like a slice failure would.

## Two failure modes the slice-and-guardrail machinery catches

- **Metric-shopping via slicing.** A team, given the pressure to ship, computes the primary metric across many slices and reports the ones that improved. The harness prevents this because the slice list is fixed *in the eval config, in advance*. Adding a new slice mid-release requires a config PR reviewed by someone other than the release author.
- **"The aggregate looks fine."** The candidate's aggregate primary metric is up; the release ships; a week later product-analytics reports the new-visitor funnel is silently down. The slice gate on the new-visitor slice would have caught it. This is the modal reason a release conversation should distrust an aggregate lift without a slice matrix under it.

## What lives here vs. what lives in mod-309

Slice enforcement is chapter 02's job; slice *policy* — which protected classes must be tracked, which fairness metric is the load-bearing one for a regulated deployment, what the review packet says — is [mod-309]'s job. The two hand off at the eval config: mod-309 writes the policy, this module wires it into the harness gate. If the eval config's slice list does not match mod-309's review packet, one of the two is silently out of date and the harness is enforcing something the reviewer would not sign.

## Summary

Aggregate metrics hide slice-level regressions; hard-fought bug fixes get silently un-fixed unless the case is in an adversarial suite. The harness enforces both — the slice matrix (business-critical + fairness + error-concentration slices, each with a row-count minimum and a per-slice pass rate) and the adversarial / regression suite (append-only, one incident per case, 100 % pass rate required). Alongside these, a small set of guardrail metrics — calibration, worst-group, refusal, cost, offline latency, robustness — must not regress. Every gate is a boolean in the decision object; `passes_release_gate` is the AND across all of them. Chapter 03 picks up where this chapter stops: an offline `passes_release_gate=True` is the *precondition* for an online promotion, not the promotion itself.
