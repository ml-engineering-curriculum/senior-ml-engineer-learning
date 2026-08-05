# exercise-04: LLM-as-Judge — When And How

**Estimated effort:** 3 hours

## Objective

Design an **LLM-as-judge** for a chosen LLM-augmented feature — the judge shape (pointwise, pairwise, reference-based), the judge prompt bundle, the human-labelled calibration set, the failure-mode gates, and the way the judge's score flows into the offline harness from chapter 01. *Or* — defend a decision *not* to bring LLM-as-judge in, with programmatic metrics and human eval as the alternative.

Both outcomes are passing. Chapter 05's core message is that LLM-as-judge without a calibration set is theatre; the exercise is where you either build the discipline or explain why the discipline is not (yet) worth the cost.

The deliverable is a short design memo — a page or two — that a peer L30 could sign, and that could be attached to a paired-project's release review.

## Prerequisites

- Read chapter 05 (`05-llm-as-judge-when-and-how.md`) — the four conditions LLM-as-judge belongs, the three shapes, the four failure modes, and the calibration discipline are load-bearing.
- Read [Zheng, Chiang, Sheng, Zhuang, Wu, Zhuang, Lin, Li, Li, Xing, Zhang, Gonzalez, Stoica, *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*](https://arxiv.org/abs/2306.05685) (NeurIPS 2023). It is short and the canonical modern reference.
- Skim chapter 01 (harness shape) — the judge's score becomes an entry in `HarnessDecision`.
- Skim [mod-304 chapter 02](../../mod-304-production-llm-integration/02-prompts-as-engineering-artifacts.md) — the judge itself is a prompt bundle.
- Skim [mod-309](../../mod-309-responsible-ai-governance/) — human eval on the calibration set connects to the responsible-AI review packet.

## Pick your feature

Pick **one** LLM-augmented feature. Ideally one you have real access to; otherwise a scenario:

- **J1 — Customer-support summariser.** Nightly (batch) summariser of the last 30 days of tickets per customer. The primary quality metric today is a rubric-scored quality; you have no programmatic metric that captures "the summary is useful to the reading agent."
- **J2 — Draft-reply generator.** In-agent tool: for an inbound ticket, generate a first-draft reply in brand voice. Quality dimensions include factuality (against the customer's ticket history), brand-voice alignment, and safety.
- **J3 — RAG doc-Q&A assistant.** User-facing docs-site Q&A. Quality dimensions include groundedness (in cited passages), relevance to the question, and answer completeness. This case is Databricks's [*LLM auto-eval best practices for RAG applications*](https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG) territory.
- **J4 — Product-blurb generator.** For newly-uploaded merchant SKUs, generate a short marketing blurb. Quality dimensions include factuality (against structured attributes), style, and length.
- **J5 — Bring-your-own.** A production or planned LLM-augmented feature you have access to. Anonymise anything sensitive.

## Steps

### 1. Decide whether LLM-as-judge belongs at all (≈ 30 min)

Walk the four conditions from chapter 05:

- **Free-text / free-structure output where no programmatic metric captures quality.** Say yes / no with a sentence naming the programmatic metric you tried or ruled out. If exact match, schema conformance, or a well-known metric would work, use *that* — not LLM-as-judge.
- **Rubric can be written down clearly.** Write the rubric. Three to seven bullets. Show it to a peer or a product owner; if they cannot use it to score two examples the same way, the rubric is not ready and neither is the judge.
- **A human-labelled calibration set exists (or can be built).** Name the plan. 100–500 examples per business-critical slice, two humans per example, inter-annotator agreement measured. Where do the humans come from — the team, an internal annotator pool, a vendor, existing labelled data?
- **Stakes match the judge's precision.** What decision does the judge score gate — a PR block, a shadow entry, a canary gate, or a full release? For high-stakes gates, chapter 05's rule is human-in-the-loop on the calibration set even when a judge is used elsewhere.

If any of the four is not true, the answer is not automatically "no judge" — it may be "not yet." Say what would have to be true first.

Getting to "no judge, use programmatic metric + human eval instead, here is why" is a passing outcome. That is the point of chapter 05 §*When LLM-as-judge belongs*.

### 2. If yes: choose the judge shape and author the prompt bundle (≈ 45 min)

- **Shape.** Pointwise (score per output), pairwise (A vs B), or reference-based (score against a reference — a passage, a gold answer, a policy)? Defend the choice against the other two. Zheng et al. showed pairwise correlates better with human preference than pointwise on MT-Bench; but pairwise is 2× cost, and reference-based is what RAG groundedness typically needs.
- **The judge prompt bundle.** Following mod-304 chapter 02, produce (or sketch) a prompt bundle for the judge — a `judge/v1.md` with a rubric section, a decision section, and structured output; a schema; the pinned judge base-model version; the decoding config. The bundle has a fingerprint that goes into every harness log line.
- **Length-control clause.** For pointwise and pairwise, include a clause in the rubric that explicitly instructs the judge not to reward length as such. This is the chapter 05 §*Failure mode 2 (verbosity)* gate.
- **Position control.** For pairwise, note the swap-consistency contract — every pair is scored in both orders; only pairs where both orders agree are counted. This is the chapter 05 §*Failure mode 1 (position)* gate.
- **Judge family choice.** Cite Panickssery et al. and pick a judge from a *different family* from the candidate-under-review. If you cannot, name the two-judge-agree fallback plan.

### 3. Design the calibration set (≈ 30 min)

- **Sampling.** How is the calibration set drawn — from production traffic, from an existing eval set, from a curated seed? Which slices from chapter 02 are represented? Row counts per slice.
- **Human labelling.** How many humans per example (chapter 05 says ≥ 2). Which agreement metric — Cohen's κ, Krippendorff's α — do you compute *between the humans* first? What is the acceptable floor (a common bar is κ ≥ 0.6, though task-dependent)?
- **The alignment metric.** How you measure judge alignment with the humans — Pearson / Spearman for pointwise, agreement rate for pairwise, agreement rate for reference-based binary. Per slice. State a *release threshold* — the judge's alignment must be at or above this before the judge's score is trusted in the harness.
- **The re-calibration cadence.** When must the calibration be re-run? Chapter 05 says: judge-prompt change, judge base-model version bump, new business-critical slice, and a nightly sampled drift canary.

### 4. Wire the judge into the offline harness (≈ 20 min)

- Which entry in `HarnessDecision` does the judge's score become? Primary quality metric, one of the guardrails, or something else?
- What is the paired statistical treatment (chapter 01 §4)? Bootstrap on per-example judge deltas is a defensible default; note the CI-clears-zero rule.
- What happens to the harness when the judge's calibration decays below the release threshold? Chapter 05: the harness stops trusting the judge — either the calibration is refreshed, the judge is re-prompted, or the judge is temporarily replaced with human eval. State the exact fallback for your feature.
- What programmatic metrics *stay in the harness* alongside the judge score? Chapter 05: keep programmatic metrics where they apply. Name at least one.

### 5. Failure-mode audit (≈ 15 min)

For each of chapter 05's four failure modes — position bias, verbosity bias, self-preference, style-over-substance / rubric drift — write one sentence naming how your design gates against it. If a gate is not in place, say so and name the reason.

### 6. Write the deliverable memo (≈ 20 min)

Assemble the memo:

- Header — feature, candidate under review, judge decision (use / don't use / not yet).
- Section 1 — the four-conditions walk from step 1. This is the *why*.
- Section 2 — the judge shape and prompt bundle (if applicable).
- Section 3 — the calibration set design.
- Section 4 — the harness integration.
- Section 5 — the failure-mode audit table.
- Section 6 — one paragraph on what the memo *does not* cover (e.g. specialist-track hand-off to `ai-eval-engineer-learning` if the judge design gets more elaborate).

## Deliverable

A single Markdown memo `llm-as-judge-<feature>.md`, roughly one to two pages. If you conclude no-judge, the memo is shorter — the four-condition walk plus the alternative plan (programmatic metric + human eval on the calibration set) is the deliverable.

## Acceptance criteria

- [ ] The four-conditions walk from chapter 05 is explicit; each condition has a yes / no and a defended sentence.
- [ ] If yes, the judge shape is chosen and defended against the other two shapes.
- [ ] If yes, the judge prompt bundle is fingerprinted, pins a specific judge base-model version, and follows mod-304 chapter 02's bundle discipline.
- [ ] If yes, the calibration set has a sampling plan, at least two humans per example, an inter-annotator-agreement measurement, an alignment metric per slice, and a re-calibration cadence.
- [ ] If yes, the harness integration names the `HarnessDecision` entry, the paired statistical treatment, the calibration-decay fallback, and at least one programmatic metric that stays alongside.
- [ ] If no, the alternative plan (programmatic metric + human eval on the calibration set, or "not yet") is named and defended, and the memo says what would have to be true to revisit.
- [ ] The failure-mode audit covers all four of chapter 05's failure modes with an explicit gate or an explicit gap.
- [ ] Where the judge is from the same family as one of the candidates, the two-judge-agree fallback is named. Otherwise the different-family choice is stated.
- [ ] Length-control clause and swap-consistency contract are explicit in the judge design.

## Stretch goals

- **Two-judge cross-validation.** Design a second judge from a different family and run the pairwise agreement between the two judges as a live diagnostic. When they disagree, that pair goes to the human queue.
- **Prometheus / RewardBench comparison.** If you have implementation appetite, compare your judge against the open-source [Prometheus](https://arxiv.org/abs/2310.08491) judge (Kim et al. 2023) or measure it on [RewardBench](https://arxiv.org/abs/2403.13787) (Lambert et al. 2024). This is an *ai-eval-engineer* depth exercise — appropriate as a stretch, not required.
- **Bias sanity checks.** Author three worked adversarial cases against your judge — a length-controlled pair (same content, different length), a self-preference pair (same-family vs cross-family output), a position-swap pair (A/B vs B/A). Run them and report the judge's behaviour. If the judge fails on your synthetic cases, your production judge would too.
- **Delegation to a specialist track.** If the judge design gets more elaborate than a senior ML engineer's day-to-day (industrial-scale judge harness, judge fine-tuning, dataset governance at policy depth), author a delegation contract to `ai-eval-engineer-learning` in the shape of [mod-304 chapter 06](../../mod-304-production-llm-integration/06-delegation-contract-to-llm-specialists.md).
- **Feed into the paired project.** Attach the memo to [`project-302-llm-augmented-ml-feature`](../../../projects/project-302-llm-augmented-ml-feature/) as the evaluation-metric section for the LLM-augmented branch. The harness from exercise-01 + the promotion plan from exercise-02 + this memo is a coherent release story.
