# exercise-01: Offline Harness With Slices And Guardrails

**Estimated effort:** 3 hours

## Objective

Design an **offline evaluation harness** for a chosen feature that a team could actually run on every candidate model. The harness is a program with four inputs (candidate, baseline, versioned eval config, statistical shape) and one output (a `HarnessDecision` object). It has a slice matrix drawn from chapter 02, an append-only adversarial suite, and a small set of guardrail metrics beside the primary quality metric. The deliverable is a spec — the eval config as a file, the harness entry-point signature, a sketch of the decision object, and a short design memo — such that a peer L30 could code it in a sprint.

You are not (necessarily) writing the harness code end-to-end in this exercise. You are proving you can *design* one that survives a review by chapter 04's ML Test Score rubric.

This is the L30 tell that separates "we compare models with a notebook" from "the team's harness produced the decision that gated the release." An L20 ships a comparison; an L30 ships the program the comparison runs inside.

## Prerequisites

- Read chapter 01 (`01-offline-eval-harness-shape.md`) — the four inputs, the decision object, and the paired statistical shape are load-bearing.
- Read chapter 02 (`02-slice-and-adversarial-guardrails.md`) — the three-tier slice matrix and the adversarial-suite growth discipline show up in the deliverable.
- Skim chapter 04 (`04-ml-test-score-production-readiness.md`) — you will be graded on ML Test Score evidence in the acceptance criteria.
- Optional: skim [mod-303 chapter 05 (calibration)](../../mod-303-advanced-modeling/05-calibration.md) for the calibration guardrail vocabulary and [mod-304 chapter 04](../../mod-304-production-llm-integration/04-cost-and-latency-guardrails.md) if the feature is LLM-augmented.

## Pick your feature

Pick **one** of the following, or bring your own (F4, with a short "context I already know" preamble):

- **F1 — Support-ticket triage classifier.** Classify a new ticket into one of twelve next-action classes (mod-304 chapter 01's worked example). ~2 M labelled historical pairs, ~500 rps peak, p99 ≤ 1.5 s from panel open. New candidate is a fine-tuned smaller model claiming a 3-point macro-F1 lift over the current XGBoost baseline.
- **F2 — Search ranker.** NDCG@10 on a shopping-site search feature. New candidate is a two-tower retriever + gradient-boosted reranker; baseline is a lexical BM25 + hand-tuned rules. Business-critical slices include enterprise catalogue, new-visitor session, and mobile-app queries.
- **F3 — LLM-augmented summariser.** The nightly review-summariser from mod-304 chapter 02 (batch enrichment). New candidate bundles a prompt update *and* a base-model version bump. Primary quality metric is a rubric-scored quality (chapter 05 candidate).
- **F4 — Bring-your-own.** A production or planned feature you have access to. Anonymise anything sensitive and add a one-paragraph context preamble covering the primary metric, the eval-set shape today, and any known adversarial cases.

## Steps

### 1. Name the primary metric and the baseline, before anything else (≈ 15 min)

Pin down, in two paragraphs total:

- The primary quality metric. One metric, chosen because it maps to the downstream decision the feature drives. Cite chapter 01's rule that this is picked *before* the candidate exists.
- The baseline. What is currently in production (if any)? If nothing, what is the reference model — the majority class, a linear model, a well-prompted frontier LLM, an existing rules engine? Why is this a defensible floor?

If you find yourself listing three primary metrics, stop and pick one. The metric-shopping pattern chapter 01 warns about begins here.

### 2. Author the versioned eval config (≈ 30 min)

Produce a YAML (or JSON) file — `eval_config.v1.yaml` — that a harness could load. At minimum it names:

- **The eval sets.** Primary eval set (name, version, row count, label source), plus one or more slice eval sets and the adversarial suite. Each entry has a fingerprint or hash mechanism; the eval config's *version* is bumped whenever any entry changes.
- **The slice matrix.** For F1/F2/F3/F4, at least three business-critical slices, at least one fairness / regulator-visible slice (if applicable — if not, say so and defend the omission), and at least one error-concentration slice. Each slice: name, filter expression, minimum row count, and pass-rate gate.
- **The guardrail set.** At minimum: calibration, worst-group metric, cost per prediction, and offline latency. LLM-augmented candidates add refusal / abstention rate. Each guardrail: metric, threshold, source of the threshold (chapter 02 rules, product agreement, regulator floor).
- **The adversarial suite reference.** A pointer to `adversarial_suite.v14.yaml` or similar. You do not have to author dozens of adversarial cases — three worked cases with the incident number, expected behaviour, and rationale is enough for this exercise.
- **The statistical shape.** Which paired test the harness runs (bootstrap on per-row deltas, McNemar, paired normal), the CI level (95 % is standard), and the rule that a lift's CI must clear zero.

The YAML does not have to be executable in a specific tool. It has to be readable by a peer and version-bumpable via PR.

### 3. Sketch the `HarnessDecision` object and the entry-point signature (≈ 30 min)

Chapter 01 has a sketch; adapt it to your feature. Produce:

- A Python (or pseudo-code) dataclass for `SliceResult`, `GuardrailResult`, and `HarnessDecision`, with the fields you need. If your primary metric is not scalar (e.g. NDCG@10 with a per-query score), the `SliceResult` shape has to accommodate it.
- The harness's entry-point signature — the function `harness(candidate_id, baseline_id, eval_config_path) -> HarnessDecision`. Note what artefacts each input resolves to (a model-registry ID, an eval-set path, a bundle fingerprint).
- The `passes_release_gate` computation. Which gates are mandatory (any red fails the release) and which are advisory (recorded, not blocking)?

### 4. Design the three-tier slice matrix in detail (≈ 30 min)

For the slice matrix from step 2, for each named slice:

- **Business-critical slices** — why is this slice load-bearing? Which product owner will notice a regression? What is the minimum row count that gives a per-slice CI narrower than half the release threshold on the primary metric?
- **Fairness / regulator-visible slices** — cite the regulation or the internal policy that names the slice. Which fairness metric is the load-bearing one (equal opportunity, calibration disaggregation, worst-group precision)? Mod-309's review packet is the interface; this exercise is where you enforce it in the harness.
- **Error-concentration slices** — how did you discover it (manual error analysis, automated slice-finder tool, cohort dashboard)? What is the story that promotes it from "an interesting concentration" to "a gated slice"?

If any slice's minimum row count is not achievable on today's eval set, name the collection plan.

### 5. Adversarial suite — three worked cases with growth policy (≈ 15 min)

Author three adversarial cases in the format chapter 02 describes:

```yaml
- id: adv_042
  incident: INC-2026-08-14
  category: safety_prompt_injection      # or historical_regression, invariance, calibration, ood
  input: ...
  expected_behaviour: ...
  added_by: {author}, {date}
  notes: ...
```

Then, in one paragraph, name the *growth policy*: who is authorised to add cases, what is the review before a case is added, when (if ever) is a case removed, and what happens when a case flakes (i.e. the model sometimes-passes-sometimes-fails on the case).

### 6. One-page design memo (≈ 30 min)

Wrap the deliverable in a memo (one page, no more) that a peer could read cold. Cover:

- **What the harness is for.** One paragraph: the feature, the primary metric, the release gate.
- **What is *not* in the harness.** Latency and cost past the offline floor; online product KPI; user-behaviour effects. Cite chapter 01's "the harness's job is to name these gaps" — say what the online path (chapter 03) picks them up as.
- **Cadence.** PR-blocking / nightly / weekly split (chapter 01 §*Cadence*).
- **Failure modes the harness catches.** Name two — pick from the ones in chapter 01 §*Two failure modes* and chapter 02 §*Two failure modes*, or add a domain-specific one.
- **Cross-references to the ML Test Score.** For each of chapter 04's four categories, name one specific test the harness passes automatically (e.g. Model Test 6 — slice quality — is passed by the slice matrix in this exercise).

## Deliverable

A folder `harness/` (or a single Markdown document with the sections inlined) containing:

- `eval_config.v1.yaml` (or `.json`).
- `harness_types.py` (or a pseudo-code equivalent) with `SliceResult`, `GuardrailResult`, `HarnessDecision`, and the entry-point signature.
- `adversarial_suite.v1.yaml` with the three worked cases.
- `README.md` — the one-page design memo.

If you want to go further (see stretch goals), a runnable prototype that ingests a small synthetic dataset and produces a real `HarnessDecision` is a natural extension.

## Acceptance criteria

- [ ] The primary metric is named in one sentence, chosen before the candidate exists, and defended by the downstream decision it drives.
- [ ] The baseline is named and is either the current production model or a defensible reference floor.
- [ ] The eval config is a file, versioned, and every eval set has a fingerprint or hash mechanism.
- [ ] The slice matrix has at least three business-critical slices, at least one fairness / regulator-visible slice (or a defended omission), and at least one error-concentration slice, each with a minimum row count and a pass-rate gate.
- [ ] Guardrail metrics include, at minimum, calibration, worst-group, cost, and offline latency; each has a threshold with a stated source.
- [ ] The `HarnessDecision` object sketch includes per-slice and per-guardrail results and a composable `passes_release_gate`.
- [ ] The paired statistical shape is named — bootstrap on per-row deltas, McNemar, or paired normal — and the CI-clears-zero rule is stated.
- [ ] The adversarial suite has at least three worked cases with a growth policy paragraph.
- [ ] The one-page memo names two failure modes the harness catches and cites at least four ML Test Score tests the harness passes.
- [ ] Nothing in the deliverable references an online metric the offline harness cannot see without saying so; chapter 01's "the harness's job is to name these gaps" is respected.

## Stretch goals

- **Runnable prototype.** Implement the harness against a small (public or synthetic) dataset and produce a real `HarnessDecision` object serialised to JSON. Score the same model twice with different eval-set versions to prove the fingerprint machinery works.
- **Fairness deep-dive.** For F1/F2/F4, choose a fairness metric from Barocas, Hardt, and Narayanan's [*Fairness and Machine Learning*](https://fairmlbook.org/), justify it against your domain, and wire it as a guardrail. Include a per-slice reliability diagram in the decision object for classifiers with a downstream threshold decision.
- **Slice-discovery pass.** Run a slice-finder (Chung et al., or a manual sort-by-confidence-and-segment) on a real error set for the feature. Promote one discovered slice to Tier 1 or 2 with a written promotion argument.
- **Peer review.** Trade harness designs with a peer. Reviewer runs chapter 04's rubric against the harness and lists which of the 28 tests it demonstrably passes, which it partially passes, and which it does not. The gaps become follow-up work.
- **Feed into the paired project.** Reuse the harness spec as the evaluation section of [`project-302-llm-augmented-ml-feature`](../../../projects/project-302-llm-augmented-ml-feature/). The harness *is* the release gate the project's LLM-augmented feature has to survive.
