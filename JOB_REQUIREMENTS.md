# Job Requirements — Senior Machine Learning Engineer

**Role level:** 30 (senior tech-lead altitude, ML engineering ladder)
**Track:** `senior-ml-engineer-learning`
**Research window:** 2026-04-05 → 2026-07-04 (last 90 days)
**Today:** 2026-07-04

This file documents the requirements catalogue used to seed the Senior Machine Learning Engineer curriculum. Raw normalized data lives in [`.aicg/job-requirements.json`](.aicg/job-requirements.json); the planned curriculum lives in [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json).

## Status — bootstrap session, postings deferred

<!-- needs-research: collect ≥25 distinct in-window postings titled "Senior Machine Learning Engineer" / "Sr. Machine Learning Engineer" / "Senior ML Engineer" (NOT Machine Learning Engineer, NOT Staff, NOT Principal, NOT Platform, NOT MLOps, NOT NLP-only, NOT LLM-only, NOT Research Scientist, NOT Data Scientist) and re-validate every requirement against the live evidence. -->

This packet was authored in a bootstrap session **without WebSearch / WebFetch permissions**. Both the parent agent and a delegated research subagent were denied web-tool access. Per the project rules ("Do not invent facts, incidents, or salary figures. Cite sources."), the `postings` array in `.aicg/job-requirements.json` is intentionally empty — the curriculum cannot honestly claim to have analysed 25 live postings when none were fetched.

The autonomous research loop is expected to fill that gap on its next cycle. To make that loop deterministic, this document instead grounds the requirements catalogue in **authoritative public references** that publish what the role is hired against (canonical books, official cloud certifications, academic MLOps papers, and public engineering career frameworks). Every requirement below cites at least one such reference, and every requirement is shaped so that posting-frequency evidence can be added underneath it without restructure.

## Methodology

1. Sourced the canonical task domains for a senior-tier Machine Learning Engineer from public references — see `authoritative_references` in `.aicg/job-requirements.json`:
   - *Designing Machine Learning Systems* — Chip Huyen (O'Reilly, 2022)
   - *Machine Learning Engineering* — Andriy Burkov (2020)
   - Google Developers, *Rules of Machine Learning*
   - Google Cloud, *MLOps: Continuous delivery and automation pipelines in machine learning*
   - Breck et al., *The ML Test Score* (Google, 2017)
   - Sculley et al., *Hidden Technical Debt in Machine Learning Systems* (NeurIPS 2015)
   - Kreuzberger et al., *Machine Learning Operations (MLOps)* (arXiv:2205.02302)
   - Kohavi, Tang, Xu, *Trustworthy Online Controlled Experiments* (Cambridge, 2020)
   - AWS Certified Machine Learning Engineer – Associate exam guide
   - Google Cloud Professional Machine Learning Engineer exam guide
   - DeepLearning.AI / Coursera *ML Engineering for Production* specialization
   - `engineeringladders.com` and `progression.fyi` public engineering-career frameworks
   - Google Engineering Practices Documentation (code review)
   - Official PyTorch and MLflow documentation
2. Applied the seniority-differentiation heuristic: for every requirement, articulated **what a level-30 senior owns that a level-20 mid-level does not** (roadmap, RFC, review-bar, mentorship, delegation contract). This is captured in the `seniority_signal` field of every requirement.
3. Applied the **ownership rule**: assign coverage to the lowest-level role that genuinely requires the skill; higher levels add depth, architecture, or leadership framing. Peer specialist tracks own depth in their domain and are linked, not duplicated.
4. Flagged everything that has not yet been validated against in-window postings so the next cycle can demote any requirement whose evidence stays empty.

## Requirement themes → curriculum ownership

The table below lists each requirement theme, its planned owner per the level hierarchy, and the curriculum coverage path. **Freq** is intentionally blank for this cycle — it will be backfilled by the next research pass.

| # | Theme | Freq | Owner role | Coverage |
|---|---|---|---|---|
| 1 | Senior role scope (own end-to-end, mentor, set roadmap) | <!-- needs-research --> | `senior-ml-engineer` (this) | [`mod-301-senior-ml-role-scope`](lessons/mod-301-senior-ml-role-scope) |
| 2 | ML systems architecture (batch/streaming, feature stores, training-serving skew) | <!-- needs-research --> | `senior-ml-engineer` | [`mod-302-ml-systems-architecture`](lessons/mod-302-ml-systems-architecture) |
| 3 | Advanced modeling tactics (multi-task, multi-modal, self-supervised, calibration) | <!-- needs-research --> | `senior-ml-engineer` | [`mod-303-advanced-modeling`](lessons/mod-303-advanced-modeling) |
| 4 | Production LLM integration (RAG, prompt-as-engineering, LLM-as-judge, cost guardrails) | <!-- needs-research --> | `senior-ml-engineer` (integration) → peer specialists for depth | [`mod-304-production-llm-integration`](lessons/mod-304-production-llm-integration); depth at `llm-application-developer`, `rag-engineer`, `fine-tuning-engineer`, `nlp-engineer` |
| 5 | Advanced evaluation (offline harness, shadow/canary, LLM-judge, error playbooks) | <!-- needs-research --> | `senior-ml-engineer` | [`mod-305-advanced-evaluation`](lessons/mod-305-advanced-evaluation) |
| 6 | Experimentation at scale (A/B, SRM, CUPED, interference, causal inference) | <!-- needs-research --> | `senior-ml-engineer` | [`mod-306-experimentation-at-scale`](lessons/mod-306-experimentation-at-scale) |
| 7 | ML reliability, SLOs, cost budgets, incident response | <!-- needs-research --> | `senior-ml-engineer` | [`mod-307-ml-reliability-slos`](lessons/mod-307-ml-reliability-slos) |
| 8 | Cross-team collaboration with platform/MLOps/training tracks | <!-- needs-research --> | `senior-ml-engineer` | [`mod-308-platform-collaboration`](lessons/mod-308-platform-collaboration) |
| 9 | Responsible-AI review packets, model cards, data lineage | <!-- needs-research --> | `senior-ml-engineer` (packet-author) → `ai-governance-analyst` / `ai-infra-security-learning` for depth | [`mod-309-responsible-ai-governance`](lessons/mod-309-responsible-ai-governance) |
| 10 | Technical leadership — roadmap, RFC, code review at bar, mentorship, hiring | <!-- needs-research --> | `senior-ml-engineer` | [`mod-310-technical-leadership`](lessons/mod-310-technical-leadership) |
| 11 | ML practitioner workflow foundations (data → model → deploy → monitor) | n/a — prerequisite | `ml-engineer` (level 20) | Listed in [`PREREQUISITES.md`](PREREQUISITES.md); not re-taught. Depth link: [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) |
| 12 | Self-service ML platform build (feature stores, multi-tenant compute) | n/a — peer track | `ai-infra-ml-platform-learning` (level 30) | The senior ML engineer is a paved-road consumer; linked out |
| 13 | Deep MLOps automation / CI-CD-for-ML platform | n/a — peer track | `ai-infra-mlops-learning` | Consumer contract in mod-308; linked out for depth |
| 14 | Distributed training / GPU cluster ops / NCCL tuning | n/a — out of scope | `training-pipeline-engineer`, `ai-infra-performance-learning` | Awareness-only in mod-303; depth linked out |
| 15 | LLM fine-tuning depth (LoRA/QLoRA, DPO/RLHF, dataset engineering) | n/a — peer specialist | `fine-tuning-engineer` | Delegation contract in mod-304; depth linked out |
| 16 | Eval-platform build (LLM-judge harness, agent eval) | n/a — peer specialist | `model-evaluation-engineer`, `ai-eval-engineer` | Consumer contract in mod-305; depth linked out |
| 17 | ML/AI security depth (data poisoning, model extraction, adversarial inputs) | n/a — higher level | `ai-infra-security-learning` (level 35) | Awareness in mod-309; depth owned upstream |
| 18 | Governance / compliance review depth | n/a — peer track | `ai-governance-analyst` | Awareness in mod-309; depth owned upstream |

## Seniority differentiation — what changes from level 20

The single most important signal in a Senior Machine Learning Engineer posting is the **verb**. The level-20 ML Engineer posting says "build", "train", "deploy". The level-30 senior posting says "**own**", "**lead**", "**mentor**", "**set the technical direction for**", "**partner with**", "**raise the bar for**". Every requirement in this catalogue is shaped around that shift and captured in the `seniority_signal` field of the JSON. When the autonomous research loop backfills the postings array, it should verify that each requirement's phrasing on the live evidence matches this seniority shift — a "familiarity with" bullet does not count as level-30 evidence for a requirement.

Framework references used to cross-check the seniority scope:

- [Engineering Ladders](https://www.engineeringladders.com/) — public five-axis framework (Development, Systems, People, Process, Influence).
- [progression.fyi](https://www.progression.fyi/) — collected public engineering-career frameworks from Monzo, Meta, GitLab, Buffer, Basecamp, and others.
- [Google Engineering Practices](https://google.github.io/eng-practices/) — code-review and design-doc standards a senior engineer is expected to run.

## Posting evidence

<!-- needs-research: populate with the ≥25 in-window postings sampled next cycle. Use the same table shape as ai-infra-senior-engineer-learning/JOB_REQUIREMENTS.md. -->

No postings were sampled this cycle. See the **Status** section above for the reason. The next autonomous research cycle should fan out across `job-boards.greenhouse.io`, `jobs.lever.co`, `jobs.ashbyhq.com`, and major employer ATS roots. A recommended employer coverage set:

- **AI-native**: OpenAI, Anthropic, Scale AI, Cohere, Character.AI, Perplexity, Databricks, Weights & Biases, Cresta, Adept, Together AI.
- **Consumer tech**: Airbnb, DoorDash, Instacart, Pinterest, Reddit, Discord, Roblox, Duolingo, Spotify, Netflix.
- **Fintech**: Stripe, Ramp, Brex, Mercury, Plaid, Robinhood, SoFi.
- **Retail / adtech**: Wayfair, Walmart Global Tech, Target, The Trade Desk, Teads, Nextdoor.
- **Health / bio**: Tempus, Recursion, Flatiron, Hims, Ro, Oscar, Verily.
- **Traditional**: GM, Ford, JPMorgan Chase, Capital One, American Express, Comcast.

For each posting, capture employer, exact title, URL, `date_observed`, `date_posted_estimate` (or `estimated:2026-MM`), location, 5–10 verbatim required-skill bullets, 2–6 preferred-qualification bullets, salary range when published, and one short representative quote. **Filter out**: mid-level (plain "Machine Learning Engineer"), Staff/Principal, Platform, MLOps, Research Scientist, Data Scientist, NLP-only, and LLM-only titles — those belong to other tracks.

## Ownership map — quick reference for next cycle

When backfilling postings, use this ownership decision to keep the curriculum from drifting into peer territory:

- **Senior ML Engineer (this track, level 30)** owns the *architecture, evaluation program, reliability, cross-team contracts, and technical leadership* around production ML systems. Build-altitude fundamentals are inherited from level 20.
- **ML Engineer (level 20)** owns the build-altitude workflow — this track links to it rather than re-teaching.
- **Staff / Principal ML Engineer** (levels 40 / 50) own multi-team architecture and org-wide strategy.
- **AI Infra ML Platform / MLOps** peer tracks own the paved-road platforms the senior ML engineer consumes.
- **Training Pipeline Engineer** owns distributed training infrastructure.
- **Model Evaluation / AI Evaluation Engineer** own evaluation-platform depth.
- **NLP / RAG / LLM Application / Fine-Tuning / Applied AI** engineers own LLM specialisations.
- **AI Infra Security** (level 35) owns ML/AI security depth.
- **AI Governance Analyst** owns governance and compliance depth.

## Conclusion

<!-- needs-research: re-run on the next autonomous cycle with web tools enabled, populate `postings` in .aicg/job-requirements.json, backfill the Freq column above, and demote any requirement whose evidence_post_ids stays empty. -->

The curriculum plan in [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json) is structured so that requirement frequencies can be added underneath each module without restructure. The themes themselves are grounded in cited public references, not invented, and every requirement carries an explicit seniority-signal statement so the next research cycle can validate that the live postings actually differentiate this role from a level-20 ML Engineer.
