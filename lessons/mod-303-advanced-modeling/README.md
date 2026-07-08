# mod-303-advanced-modeling: Advanced Modeling — Multi-Task, Multi-Modal, Self-Supervised, Calibrated

**Estimated effort:** 14 hours

At L20 the modeling question was "which model fits this table?" At L30 the question is different: **which regime fits this problem?** Multi-task, multi-modal, self-supervised / transfer, ensembling / distillation, or the special-case cold-start / long-tail / label-scarce shape. On top of whichever regime you picked, downstream decisions may need **calibrated probabilities** and per-input **uncertainty**. And some advanced-modeling problems are not yours to build at all — they belong to a peer specialist track.

This module teaches the regimes, the two cross-cutting concerns (calibration, uncertainty), the special-case distribution shapes (cold-start, long-tail, label-scarce), and the senior-altitude judgement of when to escalate to a specialist. It is the "modeling brain" companion to [`mod-302-ml-systems-architecture`](../mod-302-ml-systems-architecture/) (the "systems brain") — you use both when a new business problem lands.

## Learning objectives

- Choose the right modeling regime for the problem shape: multi-task, multi-modal, self-supervised, transfer, ensembling.
- Calibrate model outputs and quantify uncertainty when downstream decisions depend on probability.
- Handle cold-start, long-tail, and label-scarce regimes with the modeling toolbox appropriate to each.
- Know when to escalate to a specialist track (fine-tuning, NLP, RAG, training-pipeline) rather than build in-house.

## Chapters

1. [`01-choosing-the-modeling-regime.md`](01-choosing-the-modeling-regime.md) — the five advanced regimes and the six-question rubric that pins each one to a problem shape.
2. [`02-multi-task-learning.md`](02-multi-task-learning.md) — MTL architectures (hard sharing, MMoE), loss balancing, negative-transfer diagnostics, per-task baseline table.
3. [`03-multi-modal-modeling.md`](03-multi-modal-modeling.md) — fusion altitude (late / middle / early), missing-modality behaviour, the multi-modal + multi-task composite.
4. [`04-self-supervised-transfer-and-ensembling.md`](04-self-supervised-transfer-and-ensembling.md) — SSL objective families, the transfer decision (fine-tune vs. linear-probe vs. frozen), catastrophic forgetting, ensembling and distillation.
5. [`05-calibration.md`](05-calibration.md) — discrimination vs. calibration, reliability diagrams and ECE, Platt / isotonic / temperature / Beta scaling, per-slice calibration.
6. [`06-uncertainty-quantification.md`](06-uncertainty-quantification.md) — aleatoric vs. epistemic, deep ensembles, MC dropout, conformal prediction and coverage guarantees.
7. [`07-cold-start-long-tail-label-scarce.md`](07-cold-start-long-tail-label-scarce.md) — content-based cold-start, class-balanced and decoupled long-tail training, transfer / weak supervision / active learning for label-scarce.
8. [`08-when-to-escalate-to-a-specialist-track.md`](08-when-to-escalate-to-a-specialist-track.md) — the escalation rubric and the hand-off contract with LLM-application, RAG, fine-tuning, training-pipeline, and evaluation peer tracks.

## Exercises

- [`exercises/exercise-01-modeling-regime-decision-rubric.md`](exercises/exercise-01-modeling-regime-decision-rubric.md) — apply the chapter-01 rubric to three business problems and defend the regime choice (including "escalate to specialist" as a valid answer).
- [`exercises/exercise-02-multi-task-learning-in-practice.md`](exercises/exercise-02-multi-task-learning-in-practice.md) — design an MTL model on a real dataset, produce the single-task-baseline table, and diagnose negative transfer.
- [`exercises/exercise-03-calibration-and-uncertainty.md`](exercises/exercise-03-calibration-and-uncertainty.md) — plot reliability diagrams and per-slice ECE, fit temperature and isotonic calibrators, produce a conformal prediction interval on the same model.
- [`exercises/exercise-04-cold-start-and-long-tail-tactics.md`](exercises/exercise-04-cold-start-and-long-tail-tactics.md) — design a cold-start-plus-long-tail modeling plan for a realistic recommender or classifier and specify the evaluation slices.

## Labs & quizzes

- `labs/` — reserved for a longer-form modeling lab in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — papers, surveys, platform documentation — are catalogued in [`resources.md`](resources.md). Chapters cite Caruana's *Multitask Learning*, Ruder's MTL overview, Ma et al.'s MMoE, Baltrušaitis et al.'s multi-modal survey, Devlin et al.'s BERT, Chen et al.'s SimCLR, Radford et al.'s CLIP, Hinton et al.'s distillation paper, Guo et al. on modern calibration, Lakshminarayanan et al. on deep ensembles, Angelopoulos and Bates on conformal prediction, Zhang et al.'s long-tailed survey, Ratner et al. on Snorkel, and the peer-track READMEs for the escalation vocabulary.

## How this module hands off

- The regime choice (chapter 01) and per-task baselines (chapter 02) feed the RFC skeleton from mod-302 chapter 06 and the paired project `project-301-ml-system-design-portfolio`.
- Calibration (chapter 05), uncertainty (chapter 06), and the per-slice evaluation discipline in chapters 03 and 07 hand off to mod-305 (advanced evaluation) — the release harness that enforces them at every retrain.
- Cold-start and long-tail evaluation slices (chapter 07) hand off to mod-305 (evaluation) and mod-307 (reliability — abstain rate and cold-start coverage are SLIs).
- The escalation rubric (chapter 08) hands off to mod-308 (platform / specialist collaboration) and to the paired project `project-302-llm-augmented-ml-feature`, which is the natural place to exercise a hand-off to the LLM-application specialist track.
