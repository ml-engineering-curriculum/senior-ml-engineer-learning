# LLM-as-judge in the harness: when it belongs and how it fails

## Motivation

A lot of the outputs an L30 ML engineer ships in 2026 do not have a programmatic metric that correlates with quality. There is no exact-match answer for a customer-support summary. There is no BLEU-4 that maps cleanly to "is this a good product blurb." There is no F1 for "did the assistant explain the policy in a way the user could actually act on." For years the only credible metric on outputs of this shape was **human evaluation** — labelled by an annotator pool, expensive, slow, and impossible to run on every PR.

**LLM-as-judge** — an LLM prompted to score other model outputs — is the pattern that lets a team put a metric on these outputs *inside* the harness in chapter 01. Zheng, Chiang, Sheng, Zhuang, Wu, Zhuang, Lin, Li, Li, Xing, Zhang, Gonzalez, Stoica's [*Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*](https://arxiv.org/abs/2306.05685) (NeurIPS 2023) is the canonical modern reference, showing that a strong LLM judge can approximate human preference on many open-ended tasks. Databricks's [*LLM auto-eval best practices for RAG applications*](https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG) applies the pattern to production RAG.

Used well, LLM-as-judge is what makes a summariser or a draft-reply generator continuously evaluable — the harness runs on every PR, drift shows up in nightly runs, chapter 03's shadow can score its own outputs. Used badly, LLM-as-judge is a metric that quietly optimises the model for verbosity, style, or same-family favouritism instead of quality.

This chapter is when to bring LLM-as-judge into the harness at all, how to compose it with the disciplines from chapters 01–04, and — most importantly — the four failure modes to gate against.

## What LLM-as-judge is (three shapes)

Three common shapes, each answering a different question:

- **Pointwise scoring.** The judge is shown one output and a rubric and returns a score (e.g. 1–5 on each of {relevance, factuality, clarity, format}). Cheapest per call; noisiest per call.
- **Pairwise comparison.** The judge is shown two outputs A and B (same input) and returns "A is better," "B is better," or "tie." Higher signal per call at 2× cost; the shape Zheng et al. showed correlates well with human preference on MT-Bench.
- **Reference-based scoring.** The judge is shown the output *and* a reference (a gold answer, a retrieved passage the output must be grounded in, a policy the output must obey) and returns an alignment score. This is the shape RAG groundedness evals typically use (Databricks reference above).

Each shape has different bias exposure (§ *failure modes*) and different calibration requirements (§ *calibrating against human labels*). Pick the shape that maps to the underlying question the harness is trying to answer, not the shape that is cheapest to run.

## When LLM-as-judge belongs in the harness

Four conditions, all of which have to be true.

### 1. The output is free-text or free-structure, and no programmatic metric captures quality

If a programmatic metric captures quality — exact match, schema conformance, extractive-answer F1, ROUGE-L when the task is genuinely extractive — use it. It is deterministic, free, and immune to every failure mode below. LLM-as-judge is the tool for outputs where "correct" is a rubric, not a match: draft replies, summaries, explanations, marketing blurbs, plausibility judgments.

### 2. The rubric can be written down clearly

If the team cannot articulate what "good" means in a short rubric — three to seven bullet points on relevance, factuality, safety, format, style, groundedness — then a judge cannot either. LLM-as-judge inherits the rubric author's ambiguity; ambiguous rubric produces ambiguous scores, which produce a metric that measures nothing in particular. A useful test: give the rubric to two humans and ask each to score a shared set of 20 examples. If the two humans do not agree at usable inter-rater agreement, the judge is not going to save you.

### 3. There is (or can be) a human-labelled calibration set

You need a set of examples with human labels — 100–500 per slice as a working range — that the judge is *calibrated against*. The judge's alignment with the human labels on that set is the credibility of the judge's scores on unlabelled data. Without the calibration set, the judge's scores are unfalsifiable — a critical failure mode of many production LLM-eval stacks.

### 4. The stakes match the judge's precision

A judge appropriate for filtering candidates in a nightly regression pass is not necessarily appropriate for gating a fairness slice review. Chapter 04's *Test 7 — inclusion considerations* asks for a human review for high-stakes decisions; LLM-as-judge is a scalable filter that surfaces the cases the humans see, not a replacement for the humans on that decision.

If any of the four is not true, LLM-as-judge is either the wrong tool or a premature tool. The right response is not "run it anyway"; it is "do human eval on a smaller set for now, spend the effort on the rubric or the calibration set, come back."

## The four failure modes the harness has to gate against

The failure modes below are ones the literature has *documented* on real judges. Every LLM judge in production has some quantity of each. The harness's job is to know how much.

### 1. Position bias (pairwise)

Zheng et al. and Wang, Li, Wang, Li, Wu, Zhang, Xiao, Ren, Cao, Chang's [*Large Language Models are not Fair Evaluators*](https://arxiv.org/abs/2305.17926) documented a systematic preference for the first or second position in pairwise comparison. Some judges consistently prefer position A; some prefer position B; both are bias.

**Mitigation.** Run the pairwise comparison in both orders (A then B, then B then A) and only count the pair as a win when both orders agree. Ties otherwise. This doubles the cost and roughly halves the effective sample size, but it removes the confound. Zheng et al. call this *swap consistency*.

### 2. Verbosity bias

Judges prefer longer answers, often independent of quality. Zheng et al. document this on MT-Bench; Wu, Liu, Zhang, Wu, Li, Liu, Yang, Peng, Guo, Rong, Xie, Yang's [*Style Over Substance: Evaluation Biases for Large Language Models*](https://arxiv.org/abs/2307.03025) generalise it as a style-bias family.

**Mitigation.** For pairwise, control length in the rubric ("prefer the answer that is more correct and complete; do not reward or penalise length as such"). For pointwise, log per-example token counts and add a length-conditioned validation slice — the calibration-set exercise (below) should be re-run stratified by output length to verify the judge is not simply length-scoring.

### 3. Self-preference / self-enhancement bias

Panickssery, Bowman, Feng's [*LLM Evaluators Recognize and Favor Their Own Generations*](https://arxiv.org/abs/2404.13076) documented that a judge from model family X shows a small but real preference for outputs generated by the same family. Zheng et al. discussed the same class as *self-enhancement bias*.

**Mitigation.** Do not judge model outputs with a judge from the same family as one of the candidates. Or, if that is unavoidable, run the judgment with two judges (from different families or different provider stacks) and require both to agree for the pair to count. Report the by-judge breakdown, not just the aggregated result — a single-judge score for a same-family candidate is a metric you cannot defend in a review.

### 4. Style-over-substance and rubric drift

Judges reward polished formatting, confident tone, and cliché structure. Style-over-substance (Wu et al. above) and rubric drift (the same prompt scoring differently as vendor base-model versions change under it) are the two ways a judge score can move without model quality moving.

**Mitigation.** The judge is itself a **prompt bundle** in the sense of mod-304 chapter 02 — versioned, pinned to a specific base-model version, tested, changelog. When the vendor's base-model version bumps (mod-304 chapter 02), the judge's calibration against the human-labelled set must be re-run before scores from the new bump are comparable to scores from the old one. Chapter 04's *Test 4 — considerations for staleness* applies to the judge exactly as much as to the model under review.

## Calibrating the judge against human labels

The single most important discipline. Without it, LLM-as-judge is theatre.

### The calibration set

100–500 examples per slice (chapter 02) with labels from at least two humans. The label is whatever the judge is being asked to produce — a rubric score, a preference, a groundedness verdict. Inter-annotator agreement (Cohen's κ, Krippendorff's α) is measured *between the humans* first; if humans do not agree at a usable κ, the judge cannot be evaluated meaningfully and the rubric needs work.

### The alignment metric

For pointwise scoring: Pearson or Spearman correlation between judge score and human score, per slice. For pairwise: agreement rate between judge preference and human majority preference, per slice, with the position-bias mitigation applied. For reference-based: agreement rate on the binary groundedness (or the score bucket, if numeric), per slice.

Report the agreement number *before* trusting any harness score derived from the judge. A judge with Spearman ρ = 0.55 against humans on business-critical slice is not the same tool as a judge with ρ = 0.85 — the first is a directional signal, the second is a metric you can gate a release on.

### Re-calibrate on every change

The judge's calibration is not permanent. Re-run the alignment measurement:

- On every judge-prompt change (rubric edit, few-shot change).
- On every judge base-model version bump.
- On every new business-critical slice added to the harness.
- On a nightly cadence, on a sampled subset of the calibration set, as a drift canary.

If calibration decays below the release threshold, the harness stops trusting the judge — either the calibration is refreshed, the judge is re-prompted, or the judge is temporarily replaced with human eval until the gap is closed.

## Composing LLM-as-judge with the rest of the harness

- **The judge produces one metric among many.** LLM-as-judge scores go into chapter 01's `HarnessDecision` as one entry — often the primary quality metric for a generation task, sometimes a guardrail. Chapter 02's slice matrix, calibration guardrails, and adversarial suite still apply, unchanged.
- **The judge is versioned like a prompt.** Judge model, judge prompt version, judge base-model version identifier, judge decoding config — all fingerprinted, logged with every eval run, and the fingerprint becomes part of the eval-config version.
- **Judge cost is a real cost.** Running a judge on 10 000 eval examples per PR is not free. Sample sizes are chosen for statistical power (chapter 01 §4), stratified across slices, and cached where the inputs are deterministic. Budget the judge into the harness the same way any other expensive test in CI is budgeted.
- **Human eval remains the ground truth on the calibration set.** The judge is a scalable extension of human eval, not a replacement. High-stakes launches — regulated deployments, fairness-critical releases, first-time features — still put human eval in the critical path even when the judge is used elsewhere.

## Two failure modes the framing catches

- **"The judge score is our metric."** A team ships an LLM-augmented feature, adopts LLM-as-judge, and gates releases on the judge score alone. Six months in, the model has been optimised toward the *judge*, not the user — verbose, formatted in the judge's preferred structure, saying nothing new. The mitigation: keep programmatic metrics where they apply, keep human eval in the loop for the calibration set, and treat the judge as one signal among many, never the only signal.
- **"The judge is a frontier model; we trust it."** The team picks a strong LLM as the judge and skips the calibration step, on the theory that the judge is capable enough not to need calibration. Six months in, one of the failure modes above quietly biases every release. The rule: no calibration set, no score. This is the least popular rule in the LLM-eval world and the one that most reliably separates "an LLM-as-judge that works" from "an LLM-as-judge that generates plausible numbers."

## Where this hands off

- **mod-304 chapter 02** (prompts as engineering artefacts) is where the judge itself lives — the judge *is* a prompt bundle with all the versioning discipline that chapter installs.
- **mod-309** (responsible AI) is where the human-eval discipline for high-stakes launches is authored; LLM-as-judge is a scalable adjunct to human eval, not a substitute for it.
- **Peer track — `ai-eval-engineer-learning`** is the specialist track for evaluation systems at depth: judge-prompt design at scale, judge calibration methodology, industrial judge harnesses. When the harness's judge needs is deeper than "wire an off-the-shelf pattern," this is the delegation target (mod-304 chapter 06 has the delegation contract shape).
- **Peer track — `model-evaluation-engineer-learning`** covers the broader evaluation-engineering stack (benchmark curation, held-out design, adversarial construction, dataset governance) that a mature harness eventually needs from a specialist.

## Summary

LLM-as-judge is the pattern that puts a metric on free-text and free-structure outputs inside the harness — pointwise, pairwise, or reference-based. It belongs when the rubric is clear, a human-labelled calibration set exists, and the stakes match the judge's precision. It fails via position bias, verbosity bias, self-preference, and rubric drift, each of which the harness has to explicitly gate against — swap consistency, length control, cross-family judging, and versioned judge bundles calibrated on every change. A judge without a calibration set produces numbers you cannot defend; a judge with calibration is one metric in the `HarnessDecision`, not the only one. The discipline is not "LLM-as-judge as the metric" — it is "LLM-as-judge inside the harness we already had, held to the same rubric chapter 04 holds everything else to."
