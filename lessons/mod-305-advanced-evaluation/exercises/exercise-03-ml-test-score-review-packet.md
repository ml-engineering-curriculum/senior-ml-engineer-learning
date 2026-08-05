# exercise-03: ML Test Score Review Packet

**Estimated effort:** 3 hours

## Objective

Score a chosen ML system against the Google ML Test Score rubric (28 tests × 4 categories, from Breck et al. 2017) and produce a **review packet** — the per-test scores, the evidence link for every score, the overall weakest-link score, and a three-item next-quarter investment list. The packet is what a senior ML engineer holds a release conversation to; the exercise is what makes you fluent in using it.

You are not just filling in a spreadsheet. You are producing the artefact that a peer reviewer, a governance reviewer, or an incident postmortem would consult. The value of the exercise is the *disagreements* about what specific test evidence looks like, and the specific investments the review makes visible.

This is the L30 tell that separates "our system feels ready" from "our system scores 2.5 on the ML Test Score, here are the four monitoring tests scored 0 that will move us to 3.5 by end of quarter."

## Prerequisites

- Read chapter 04 (`04-ml-test-score-production-readiness.md`) — the 28 tests, the weakest-link scoring rule, the score-to-maturity map, and the "conversation prompt, not compliance form" framing are load-bearing.
- Read Breck, Cai, Nielsen, Salib, Sculley's [*The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction*](https://research.google/pubs/the-ml-test-score-a-rubric-for-ml-production-readiness-and-technical-debt-reduction/) (IEEE BigData 2017) directly. It is short.
- Skim chapters 01, 02, 03 — several tests are passed by the harness and the promotion path you (may have) designed in exercises 01–02.
- Skim [mod-307](../../mod-307-ml-reliability-slos/) for the monitoring depth some tests refer to and [mod-309](../../mod-309-responsible-ai-governance/) for the fairness disaggregation depth model tests 4 and 7 refer to.

## Pick your system

Pick **one** system to score. Ideally one you have real access to; otherwise a well-documented reference system.

- **S1 — Your own system.** A production model you own or contribute to. Anonymise anything sensitive. If you cannot cite public references for every test, you can cite internal artefact IDs or descriptions.
- **S2 — The paired-project's system.** If you have a `project-302-llm-augmented-ml-feature` in flight, score it as of today.
- **S3 — An open-source reference system.** For example, one of TensorFlow Extended's [tutorial pipelines](https://www.tensorflow.org/tfx/tutorials), or a public deployed model you can inspect the code of. Note this may score low on tests that depend on non-public monitoring evidence.
- **S4 — A composite hypothetical.** Use the chapter 01 worked example (support-ticket triage + LLM summariser), and *design* the system around the exercise-01 harness and exercise-02 promotion plan. Then score the composite honestly, penalising things you have not designed (e.g. Monitoring Test 3 training–serving skew).

## Steps

### 1. Assemble the evidence pack (≈ 30 min)

For each of the 28 tests, gather (or note the absence of) evidence *before* you score. Evidence types:

- Code reference — file path, module, function name.
- CI job reference — the pipeline that runs the test on every PR / nightly / weekly.
- Runbook reference — for tests where a human action is required.
- Config reference — a schema file, an eval config, a slice matrix.
- Postmortem or incident reference — an incident that would have been caught by the test.

If you cannot find evidence for a test, note that too. Chapter 04's *Failure mode 1 — score inflation* is precisely the pattern where evidence is invented after the score; do the evidence first.

### 2. Score each test with the rule that "no evidence, no score" (≈ 45 min)

Fill in the table for each of the four categories. For each test:

- **0** — no evidence, or the evidence is a comment that says "TODO."
- **0.5** — a manual, documented process runs when a human clicks a button; evidence is a runbook or a recorded manual-run history.
- **1** — the test runs automatically (CI, cron, monitoring alerts) with no human in the loop.

The Google paper's exact test-list wording is authoritative; use chapter 04's paraphrases as a reviewer's translation. Where the wording is ambiguous, note the interpretation you used — a peer might interpret differently, and the point of the exercise is to make the interpretation visible.

### 3. Compute the weakest-link score (≈ 5 min)

```
category_score(k) = sum of test scores in category k         # ∈ [0, 7]
overall_score    = 2 * min(data, model, infra, monitoring)   # ∈ [0, 14]
```

Map to the paper's interpretation band (research prototype / not fully tested / first pass / reasonably tested / strong / exceptional).

If your minimum category is 4 and the other categories are 6 or 7, the overall score is 8 — the *weakest-link* rule caps you at 2 × 4. The point is not to lift 6 to 7; it is to lift the minimum. Say what the minimum is.

### 4. Write the three-item next-quarter investment list (≈ 30 min)

The three tests you would invest in *next quarter*, ordered by how much they lift the weakest-link category. For each:

- The test's number and reviewer-paraphrased name.
- The current score (0 or 0.5).
- What it takes to raise it to 1 — a specific piece of infrastructure work, a specific tool adoption, a specific runbook to write.
- The estimated effort (a person-week bucket is enough).
- What incident the investment would have prevented, if any (chapter 04's "postmortem or incident reference" evidence type).

Three is deliberate. A list of ten is a wish list; a list of three is a commitment.

### 5. Failure-mode audit (≈ 15 min)

Chapter 04 flags two failure modes — score inflation, and checklist-shopping (the test's evidence exists but does not actually catch the failure it was meant to catch). Go back to your evidence pack:

- For each test scored 1, ask: if the thing the test guards against happened tonight, would this evidence catch it? If not, downgrade to 0.5 or 0 with a note.
- For each test scored 0.5, ask: has the manual step actually been executed in the last quarter? If not, downgrade to 0.

Recompute the weakest-link score after the audit. If it dropped, that is not a bug in the exercise — it is the *point*.

### 6. Compose the review packet (≈ 45 min)

Assemble the deliverable in one document:

- **Header.** System name, reviewer, review date, overall score, weakest-link category.
- **Score table.** Four sections, seven tests each, with score, evidence link / description, and interpretation notes.
- **Failure-mode audit.** The downgrades, with reasoning.
- **Next-quarter investment list.** The three items.
- **Cross-references.** For each of chapters 01–03 and mod-307 / mod-309, one line naming the strongest evidence link into the score. E.g. "Chapter 01 harness's `HarnessDecision` object is the evidence for Model Test 6 (per-slice quality)."

## Deliverable

A single Markdown document `ml-test-score-review-<system>-<date>.md` following the structure above. Two to three pages is a normal length; longer is fine if the evidence pack is substantial, shorter is a sign the audit was not thorough.

## Acceptance criteria

- [ ] All 28 tests are scored — no test left blank, no test left "N/A" without a defended reason.
- [ ] Every score has an evidence link or an explicit "no evidence" note. Chapter 04 §*Failure mode 1* is respected: no evidence, no score above 0.
- [ ] The weakest-link score is computed correctly (2 × minimum category, not average category).
- [ ] The overall score maps to the paper's interpretation band; the mapping is stated in one sentence.
- [ ] The next-quarter investment list is exactly three items, each with a specific piece of work, an effort estimate, and (where possible) an incident story.
- [ ] The failure-mode audit downgraded at least one score, or the deliverable states explicitly why every score survived the audit.
- [ ] Cross-references to at least three chapters in this module and two other modules (mod-307, mod-309, mod-306, mod-302) are named.

## Stretch goals

- **Peer swap.** Trade packets with a peer. Reviewer runs the "would this evidence actually catch the failure" audit against the peer's system and downgrades independently. Compare the two audits.
- **Score a system twice, six months apart.** For a system you have longitudinal access to, re-score at the end of the quarter after executing the investment list. The delta *is* the review packet's value.
- **Extend the rubric.** The Google 2017 rubric predates LLM-augmented systems. Author three additional tests you would add for a system that includes LLM components — for example, "the prompt bundle version is logged with every prediction" (mod-304 chapter 02), "an LLM-as-judge calibration set is re-scored on every base-model bump" (chapter 05 of this module), "vendor status is monitored as an SLI." Slot them into the rubric and explain why they belong in their chosen category.
- **Regulator-facing packet.** For a regulated deployment, extend the packet with mod-309 fairness-disaggregation evidence and NIST AI RMF crosswalk. This is heavier work than the base exercise and belongs on the paired project rather than a two-hour sitting.
- **Feed into the paired project.** Attach the review packet to [`project-302-llm-augmented-ml-feature`](../../../projects/project-302-llm-augmented-ml-feature/) or [`project-301-ml-system-design-portfolio`](../../../projects/project-301-ml-system-design-portfolio/). The packet is often the artefact a portfolio RFC review is graded against.
