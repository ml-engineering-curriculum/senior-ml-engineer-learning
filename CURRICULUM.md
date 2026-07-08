# Senior Machine Learning Engineer Curriculum

**Role level:** 30 (senior tech-lead altitude, ML engineering ladder)
**Status:** planned — modules and projects below are the planned scope authored from [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json). Lessons and projects will be drafted by subsequent autonomous content cycles.

## Overview

This track lifts a mid-career Machine Learning Engineer to senior altitude. It does **not** re-teach the build-altitude workflow that the level-20 [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) track owns. Instead it adds the architecture, evaluation program, reliability, cross-team collaboration, responsible-AI review authoring, and technical-leadership scope that separates a senior tech-lead ML engineer from a mid-level.

Total planned commitment: **122 hours** across 10 modules + **95 hours** across 3 projects = **~217 hours**.

## Ownership rule

Following the project-wide ownership rule, this curriculum:

- **Inherits** from `ml-engineer-learning` (level 20) — the build-altitude ML practitioner workflow is a prerequisite, not a module.
- **Owns** ML systems architecture, advanced modeling regimes, production LLM integration, advanced evaluation and experimentation, ML reliability + SLOs, cross-team platform collaboration, responsible-AI review authoring, and technical leadership at senior scope.
- **Defers up** to `staff-ml-engineer-learning` / `principal-ml-engineer-learning` for multi-team architecture and org-wide strategy.
- **Defers sideways** to peer specialist tracks for depth in their domain — `nlp-engineer`, `rag-engineer`, `llm-application-developer`, `fine-tuning-engineer`, `model-evaluation-engineer`, `ai-eval-engineer`, `training-pipeline-engineer`, `applied-ai-engineer`, `ai-infra-mlops-learning`, `ai-infra-ml-platform-learning`.
- **Links out** to `ai-infra-security-learning` (level 35) for ML/AI security depth and to `ai-governance-analyst` for governance depth.

See [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) for the requirements-to-coverage map and the cited public references the catalogue is grounded in.

## Module Plan

| Module | Title | Hours | Status |
|---|---|---|---|
| mod-301-senior-ml-role-scope | The Senior ML Engineer Role: Scope, Verbs, and Hand-off Contracts | 10 | planned |
| mod-302-ml-systems-architecture | ML Systems Architecture at Senior Altitude | 16 | planned |
| mod-303-advanced-modeling | Advanced Modeling: Multi-Task, Multi-Modal, Self-Supervised, Calibrated | 14 | planned |
| mod-304-production-llm-integration | Production LLM Integration for ML Systems | 14 | planned |
| mod-305-advanced-evaluation | Advanced Evaluation & Production-Readiness Reviews | 12 | planned |
| mod-306-experimentation-at-scale | Experimentation at Scale — A/B, Variance Reduction, Causal Framing | 12 | planned |
| mod-307-ml-reliability-slos | ML Reliability: SLOs, Cost Budgets, and Incident Response | 12 | planned |
| mod-308-platform-collaboration | Cross-Team Collaboration with Platform, MLOps, and Training Tracks | 10 | planned |
| mod-309-responsible-ai-governance | Responsible AI: Review Packets, Model Cards, and Data Lineage | 10 | planned |
| mod-310-technical-leadership | Technical Leadership for a Senior ML Engineer | 12 | planned |

## Project Plan

| Project | Title | Hours | Status |
|---|---|---|---|
| project-301-ml-system-design-portfolio | ML System Design Portfolio — Three Senior-Level RFCs | 30 | planned |
| project-302-llm-augmented-ml-feature | Ship an LLM-Augmented ML Feature End-to-End | 35 | planned |
| project-303-tech-lead-simulation | Tech-Lead Simulation: Lead a Multi-Team ML Initiative | 30 | planned |

## Module summaries

### mod-301 — Senior ML Engineer Role Scope
The seniority-defining verbs (own, lead, mentor, set roadmap, partner with, raise the bar for), the hand-off contracts to peer specialist and peer platform tracks, and a self-assessment that names the areas of the following nine modules the learner most needs to grow into.

### mod-302 — ML Systems Architecture
Batch vs. streaming, online vs. offline features, feature store adoption, model registry as source of truth, training/serving skew mitigation at architectural altitude, retraining cadence and data-flywheel design, and RFC authoring plus defence.

### mod-303 — Advanced Modeling
Choosing the modeling regime for the problem shape (multi-task, multi-modal, self-supervised, transfer, ensembling), calibration and uncertainty quantification, cold-start and long-tail tactics, and knowing when to escalate to a specialist track.

### mod-304 — Production LLM Integration
Prompt vs. RAG vs. fine-tune vs. hybrid decisions, treating prompts as engineering artifacts (versioned, tested, deployed like code), cost/latency guardrails, and the delegation contract to the LLM specialist tracks.

### mod-305 — Advanced Evaluation
Offline harness with slice/holdout/adversarial guardrails, shadow and canary deploy paths, the Google ML Test Score rubric applied to production-readiness reviews, and LLM-as-judge when and how.

### mod-306 — Experimentation at Scale
A/B test design (power, MDE, primary vs. guardrail, ramp plan), sample-ratio-mismatch diagnostics, variance reduction (CUPED / CUPAC), interference and novelty effects, sequential testing, and causal-inference framings when an A/B test is impossible.

### mod-307 — ML Reliability & SLOs
ML-specific SLIs/SLOs (prediction freshness, quality drift, retraining SLA), cost budgets and quotas that engineering can enforce, incident response for model regressions, and retraining as a first-class deploy.

### mod-308 — Platform Collaboration
Idiomatic paved-road consumption of peer platform teams' work (feature stores, model registries, training clusters), build-vs-adopt calls at team-scope, contribute-back RFCs, and hand-off contracts to training-pipeline and model-evaluation specialists.

### mod-309 — Responsible AI & Governance
Model cards, data statements, data lineage and PII audit, review-cadence fairness/robustness/safety slices, and threat modelling for the deployed system's realistic misuse patterns.

### mod-310 — Technical Leadership
Roadmap and scoping for a multi-quarter ML initiative, code review and design review at bar, mentorship of mid-level engineers, and rubric-based ML interview authoring for hiring loops.

## Assessment

Each module ships **1 quiz** plus the exercises and (where present) a lab listed in [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json). Each project ships a portfolio-grade README and an explicit assessment rubric covering the seniority signals — architecture defence, review-packet quality, delegation contract clarity, mentorship artifact, and roadmap defensibility.

## Where to go after this curriculum

- **`staff-ml-engineer-learning`** — next-up role on the ML engineering ladder; cross-team architecture and organisational strategy.
- **`principal-ml-engineer-learning`** — org-wide technical leadership.
- **`senior-agentic-ai-engineer-learning`** — adjacent senior track for teams whose ML systems are agentic.
- **`ai-infra-ml-platform-learning`** — peer at level 30 for engineers who want to move to the platform side.
- **`ai-infra-security-learning`** — for deep ML/AI security ownership at level 35.

<!-- needs-research: backfill industry-frequency evidence into JOB_REQUIREMENTS.md once the autonomous research loop runs with WebSearch / WebFetch permissions; demote any module or exercise whose underlying requirement does not show up in ≥3 in-window Senior ML Engineer postings. -->
