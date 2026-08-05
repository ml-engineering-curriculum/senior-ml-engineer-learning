# Job Requirements — Senior Machine Learning Engineer

**Role level:** 30 (senior tech-lead altitude, ML engineering ladder)
**Track:** `senior-ml-engineer-learning`
**Research window:** 2026-05-07 → 2026-08-05 (last 90 days)
**Today:** 2026-08-05

This file documents the requirements catalogue used to keep the Senior Machine Learning Engineer curriculum honest to the market. Raw normalized data lives in [`.aicg/job-requirements.json`](.aicg/job-requirements.json); the current curriculum lives in [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json); any *proposed* additions live in [`.aicg/curriculum-plan-delta.json`](.aicg/curriculum-plan-delta.json).

## Status — evidence-backfilled

The prior bootstrap cycle authored the requirement catalogue against authoritative public references (Chip Huyen, Burkov, Google, Kohavi, PyTorch, MLflow, engineering-ladder frameworks) and left the `postings` array empty. This cycle fans out across three research segments (AI-native, consumer tech, and fintech / healthtech / adtech / traditional enterprise), samples **26 distinct in-window Senior ML Engineer postings**, and backfills the `Freq` column below with real counts. Every requirement now cites at least three specific `p*` posting IDs from `.aicg/job-requirements.json`. The requirement themes hold — none are demoted — and no net-new theme reaches the ≥ 0.30 add-in-scope threshold. **The curriculum-plan delta this cycle is intentionally empty.**

## Methodology

1. Fanned out to three parallel research subagents on 2026-08-05, each covering an employer segment:
   - **AI-native / infra**: OpenAI, Anthropic, Scale AI, Cohere, Databricks, Groq, Together AI, Cresta, xAI, Mistral, Hugging Face, Runway, Pika, ElevenLabs — 9 verified postings.
   - **Consumer tech**: Airbnb, DoorDash, Instacart, Pinterest, Reddit, Discord, Roblox, Duolingo, Spotify, Netflix, Uber, Meta, Google, Apple, Amazon, Etsy, Yelp, Wayfair, Nextdoor — 9 verified postings.
   - **Fintech / healthtech / adtech / traditional enterprise**: Stripe, Ramp, Brex, Mercury, Plaid, Robinhood, SoFi, Chime, Capital One, American Express, JPMC, Tempus, Recursion, Flatiron, Hims, Oscar, Verily, Komodo, Trade Desk, Teads, GM, Ford, Comcast, Walmart, Target, Costco — 8 verified postings.
2. **Inclusion rule** applied consistently across all three segments: title must contain `Senior`, `Sr.`, or `II` (when clearly senior-band, e.g., Instacart's `Senior Machine Learning Engineer II`) and the role must be an ML engineering role — not Staff / Principal (above), Platform / MLOps / MLE Platform (peer track), Research Scientist / Applied Scientist / Data Scientist (different discipline). LLM-only or NLP-only specialist titles were dropped when scope was clearly a specialist track owned by a peer curriculum. Unverifiable postings (403s, redirects to search index, rotated listings) were dropped rather than fabricated.
3. Each posting was extracted **verbatim** — required-skill bullets, preferred qualifications, salary range, and one representative quote showing senior-differentiating verbs. See `postings` in `.aicg/job-requirements.json`.
4. Tallied every requirement theme against the 26 postings; recorded frequency in the table below and `evidence_post_ids` in the JSON.
5. Applied the **continuity bias**: proposed net-new content only when (a) a theme cites ≥ 3 postings, (b) frequency ≥ 0.30, AND (c) no existing module or exercise can be incrementally extended. Nothing cleared all three gates this cycle.
6. Retained the seniority-differentiation heuristic: every requirement's `seniority_signal` field names what the level-30 senior owns that a level-20 mid-level does not.

## Requirement themes → curriculum ownership → frequency

| # | Theme | Freq | Owner role | Coverage |
|---|---|---|---|---|
| 1 | Senior role scope (own end-to-end, mentor, set roadmap) | 0.96 | `senior-ml-engineer` (this) | [`mod-301-senior-ml-role-scope`](lessons/mod-301-senior-ml-role-scope) |
| 2 | ML systems architecture (batch/streaming, distributed systems, feature stores, training/serving skew) | 0.85 | `senior-ml-engineer` | [`mod-302-ml-systems-architecture`](lessons/mod-302-ml-systems-architecture) |
| 3 | Advanced modeling tactics (transformer, multi-task, calibration, ranking/recsys, embeddings) | 0.85 | `senior-ml-engineer` | [`mod-303-advanced-modeling`](lessons/mod-303-advanced-modeling) |
| 4 | Production LLM integration (RAG, prompt-as-engineering, LLM-as-judge, cost/latency guardrails) | 0.65 | `senior-ml-engineer` (integration) → peer specialists for depth | [`mod-304-production-llm-integration`](lessons/mod-304-production-llm-integration); depth at `llm-application-developer`, `rag-engineer`, `fine-tuning-engineer`, `nlp-engineer` |
| 5 | Advanced evaluation (offline harness, shadow/canary, precision/recall/F1, LLM-judge, error playbooks) | 0.50 | `senior-ml-engineer` | [`mod-305-advanced-evaluation`](lessons/mod-305-advanced-evaluation) |
| 6 | Experimentation at scale (A/B, causal inference, counterfactual, longitudinal statistical methods) | 0.35 | `senior-ml-engineer` | [`mod-306-experimentation-at-scale`](lessons/mod-306-experimentation-at-scale) |
| 7 | ML reliability, SLOs, cost budgets, incident response, real-time serving | 0.46 | `senior-ml-engineer` | [`mod-307-ml-reliability-slos`](lessons/mod-307-ml-reliability-slos) |
| 8 | Cross-team collaboration with platform / MLOps / product / MRM | 0.77 | `senior-ml-engineer` | [`mod-308-platform-collaboration`](lessons/mod-308-platform-collaboration) |
| 9 | Responsible-AI: review packets, bias mitigation, fairness, safety-critical framing, MRM partnership | 0.35 | `senior-ml-engineer` (packet-author) → `ai-governance-analyst` / `ai-infra-security-learning` for depth | [`mod-309-responsible-ai-governance`](lessons/mod-309-responsible-ai-governance) |
| 10 | Technical leadership — roadmap, RFC, code review at bar, mentorship, influence without authority | 0.62 | `senior-ml-engineer` | [`mod-310-technical-leadership`](lessons/mod-310-technical-leadership) |
| 11 | ML practitioner workflow foundations (data → model → deploy → monitor) | 1.00 (prerequisite) | `ml-engineer` (level 20) | Listed in [`PREREQUISITES.md`](PREREQUISITES.md); not re-taught. Depth link: [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) |
| 12 | MLOps toolchain consumption (Airflow / Ray / MLflow / Triton / SageMaker / Vertex AI) | 0.31 | `ai-infra-ml-platform-learning` (peer, level 30) | Consumer contract in [`mod-308`](lessons/mod-308-platform-collaboration); build depth linked out |
| 13 | Deep MLOps automation / CI-CD-for-ML platform build | n/a — peer track | `ai-infra-mlops-learning` | Consumer contract in mod-308; linked out for depth |
| 14 | Distributed training / GPU cluster ops / NCCL / speculative decoding / QAT / kernel-level optimization | 0.19 | `training-pipeline-engineer`, `ai-infra-performance-learning` | Awareness-only in mod-303; depth linked out. Concentrated in AI-native / inference-shop postings where the depth work is the role. |
| 15 | LLM fine-tuning depth (LoRA/QLoRA, DPO/RLHF, dataset engineering) | 0.27 | `fine-tuning-engineer` | Delegation contract in mod-304; depth linked out. Below threshold. |
| 16 | Agentic AI systems (agent orchestration, tool-use, HITL for high-trust actions) | 0.19 | `senior-agentic-ai-engineer` (level 40) | Delegation contract in mod-304; peer depth track linked out. Emerging surface — re-evaluate next cycle. |
| 17 | Eval-platform build (LLM-judge harness, agent eval infrastructure) | 0.12 | `model-evaluation-engineer`, `ai-eval-engineer` | Consumer contract in mod-305; depth linked out |
| 18 | ML/AI security depth (data poisoning, model extraction, adversarial ML) | 0.12 | `ai-infra-security-learning` (level 35) | Awareness in mod-309; depth owned upstream |
| 19 | AI coding-assistant fluency (Cursor / Copilot / Codex / interactive AI tooling) | 0.12 | `senior-ml-engineer` | Below threshold; on-the-job / short-training scope, not a role-specific learning module |
| 20 | Governance / compliance review depth | n/a — peer track | `ai-governance-analyst` | Awareness in mod-309; depth owned upstream |

## Seniority differentiation — what changes from level 20

The single most important signal in a Senior Machine Learning Engineer posting is the **verb**. The level-20 posting says "build", "train", "deploy". The level-30 senior posting says "**own**", "**lead**", "**mentor**", "**set the technical direction for**", "**partner with**", "**raise the bar for**". Verbs that surfaced verbatim across the sampled 26 postings:

- **own** — "Own the full ML lifecycle" (Reddit, p010), "Own the complete modeling lifecycle" (DoorDash, p016), "Own the model serving stack" (Together AI, p002), "Own the end-to-end pipeline for safety-critical ML perception models" (GM, p023), "Own deployment, monitoring, and iteration of agent systems" (Scale AI, p003).
- **lead** — "Lead pre-training and post-training efforts" (Groq, p006), "Lead the design and development of Cresta's next-generation AI Agents" (Cresta, p007/p009), "Lead ASR quality improvement efforts" (Cresta Voice, p008), "Lead research and development of pCTR and conversion prediction models" (Instacart, p015), "developing and leading advanced machine learning initiatives" (DoorDash, p016).
- **architect / set direction** — "Architect and own the ML foundations for commerce discovery" (Discord, p017), "setting architectural direction, and driving alignment in ambiguity" (Stripe, p019), "developing and executing a vision for the evolution of the machine learning technology stack within Ads" (Pinterest Monetization, p014), "machine learning architectural design" (Capital One, p020).
- **mentor / raise the bar** — "Provide technical leadership and mentorship to team members, fostering an environment of continuous learning and innovation" (Groq, p005), "influence cross-functional decisions and raise the engineering bar" (Cresta, p007/p009), "mentoring and elevating engineers, elevating AI/ML awareness and posture within organizations" (Stripe, p019), "Experience mentoring junior engineers or shaping technical direction for a modeling team" (Instacart, p015), "Technical leadership experience, including mentoring junior engineers and leading major feature development from concept to launch" (GM, p023).
- **partner / influence** — "Partner closely with teams across Pinterest to experiment and improve ML models" (Pinterest, p013), "Led cross-functional initiatives and excel at influencing decision-making without authority" (Flatiron, p024), "Serve as a cross-functional representative and advocate for machine learning techniques across engineering and product organizations" (Scale AI, p004).

At least one such verb appears in 25 of 26 postings (0.96). The single posting without an obvious verb (Nextdoor p018) is a scraped skill-tag listing where the source didn't expose the responsibilities section; the underlying role is titled `Senior` and paid at the senior band. `mod-301`'s seniority-verb audit exercise remains well-grounded — every posting is a live example.

Framework references still cross-check the seniority scope:

- [Engineering Ladders](https://www.engineeringladders.com/) — public five-axis framework (Development, Systems, People, Process, Influence).
- [progression.fyi](https://www.progression.fyi/) — collected public engineering-career frameworks from Monzo, Meta, GitLab, Buffer, Basecamp, and others.
- [Google Engineering Practices](https://google.github.io/eng-practices/) — code-review and design-doc standards a senior engineer is expected to run.

## Posting evidence — all 26 in-window postings

| ID | Employer | Title | Segment | Location | Salary range | URL |
|---|---|---|---|---|---|---|
| p001 | Databricks | Senior Machine Learning Engineer - GenAI Platform | ai-native | San Francisco | $166K-$225K | [link](https://www.databricks.com/company/careers/engineering/genai-senior-machine-learning-engineer-platform--6954585002) |
| p002 | Together AI | Senior Machine Learning Engineer, Voice AI | ai-native | San Francisco | $200K-$260K + eq | [link](https://job-boards.greenhouse.io/togetherai/jobs/5088817007) |
| p003 | Scale AI | Senior/Staff Machine Learning Engineer, General Agents, Enterprise GenAI | ai-native | SF / NY | $264.8K-$331K | [link](https://job-boards.greenhouse.io/scaleai/jobs/4658162005) |
| p004 | Scale AI | Senior Machine Learning Engineer, Public Sector | ai-native | Washington, DC | $225.75K-$282.45K | [link](https://job-boards.greenhouse.io/scaleai/jobs/4281519005) |
| p005 | Groq | Senior Machine Learning Engineer: Post Training & Speculative Decoding | ai-native | Palo Alto / Remote | $175.9K-$307.8K + eq | [link](https://jobs.uncorkcapital.com/companies/groq/jobs/45929251-senior-machine-learning-engineer-post-training-speculative-decoding) |
| p006 | Groq | Senior Machine Learning Engineer: Post Training & Speculative Decoding (Toronto) | ai-native | Toronto | — | [link](https://builtin.com/job/machine-learning-engineer-post-training/6584234) |
| p007 | Cresta | Senior Machine Learning Engineer | ai-native | US Remote | $205K-$270K + eq | [link](https://job-boards.greenhouse.io/cresta/jobs/4741283008) |
| p008 | Cresta | Senior Machine Learning Engineer, Voice Experience | ai-native | UK Remote | — | [link](https://simplify.jobs/p/f8c53120-8392-47f5-af93-ba274787ff37/Senior-Machine-Learning-Engineer) |
| p009 | Cresta | Senior Machine Learning Engineer (Canada) | ai-native | Canada Remote | — | [link](https://job-boards.greenhouse.io/cresta/jobs/4249943008) |
| p010 | Reddit | Senior Machine Learning Engineer | consumer-tech | US Remote | $216.7K-$303.4K | [link](https://job-boards.greenhouse.io/reddit/jobs/6960831) |
| p011 | Reddit | Senior Machine Learning Engineer, GenAI Security | consumer-tech | US Remote | $216.7K-$303.4K | [link](https://job-boards.greenhouse.io/reddit/jobs/7891887) |
| p012 | Reddit | Senior Machine Learning Engineer, Safety | consumer-tech | US Remote | $216.7K-$303.4K | [link](https://job-boards.greenhouse.io/reddit/jobs/7729096) |
| p013 | Pinterest | Sr. Machine Learning Engineer, Core Engineering | consumer-tech | SF / Palo Alto / Seattle | $189.7K-$332K | [link](https://www.pinterestcareers.com/jobs/6121464/sr-machine-learning-engineer-core-engineering/) |
| p014 | Pinterest | Sr. Machine Learning Engineer, Monetization Engineering | consumer-tech | SF / Palo Alto / Seattle | $189.7K-$332K | [link](https://www.pinterestcareers.com/jobs/6121551/sr-machine-learning-engineer-monetization-engineering/) |
| p015 | Instacart | Senior Machine Learning Engineer II | consumer-tech | US Remote | $201K-$253.5K | [link](https://jobs.generalcatalyst.com/companies/instacart/jobs/73350816-senior-machine-learning-engineer-ii) |
| p016 | DoorDash | Senior Machine Learning Engineer - Growth | consumer-tech | Toronto | I5 $149.5K-$187K / I6 $185K-$231.5K CAD | [link](https://jobs.khoslaventures.com/companies/doordash/jobs/63065670-senior-machine-learning-engineer-growth) |
| p017 | Discord | Senior Software Engineer, Machine Learning (Commerce) | consumer-tech | San Francisco | $220K-$247.5K + eq | [link](https://jobs.accel.com/companies/discord-2/jobs/70486110-senior-software-engineer-machine-learning-commerce) |
| p018 | Nextdoor | Senior Machine Learning Engineer - Notifications/Feed | consumer-tech | US Remote | $170K-$334K | [link](https://jobright.ai/jobs/info/67ed8ccd75b1acffa5a92b07) |
| p019 | Stripe | Senior Machine Learning Engineer, Stripe Assistant | fintech | Seattle | $212K-$318K | [link](https://stripe.com/jobs/listing/senior-machine-learning-engineer-stripe-assistant/6894964) |
| p020 | Capital One | Senior Machine Learning Engineer (AI Foundations) | fintech | McLean / NYC | $161.8K-$201.4K | [link](https://www.capitalonecareers.com/job/mclean/senior-machine-learning-engineer-ai-foundations/1732/96207278784) |
| p021 | SoFi | Senior Machine Learning Engineer | fintech | SF Hybrid | — | [link](https://builtin.com/job/senior-machine-learning-engineer/7269518) |
| p022 | Target | Sr Machine Learning Engineer - Marketing and Corporate Systems (ML Ops) | retail-enterprise | Brooklyn Park, MN | $98K-$176K | [link](https://corporate.target.com/jobs/w01/00/sr-machine-learning-engineer-marketing-and-corporate-systems-ml-ops) |
| p023 | General Motors | Senior Machine Learning Engineer – Perception & Embodied AI | traditional-enterprise | Mountain View | $170.6K-$261.3K | [link](https://search-careers.gm.com/en/jobs/jr-202606689/senior-machine-learning-engineer-perception-embodied-ai/) |
| p024 | Flatiron Health | Senior Machine Learning Engineer, AI Incubator | healthtech | NYC Hybrid | $139.2K-$208.8K | [link](https://www.builtinnyc.com/job/senior-machine-learning-engineer-ai-incubator/3986687) |
| p025 | Flatiron Health | Senior Machine Learning Engineer, Predictive Modeling & Applied AI | healthtech | NYC | $163.2K-$224.4K | [link](https://techjobsforgood.com/jobs/34662/) |
| p026 | Teads | Senior Machine Learning Engineer - Open Application | adtech | Ljubljana | — | [link](https://job-boards.eu.greenhouse.io/teads1/jobs/4808749101) |

## Postings dropped and why (transparency)

The three research subagents surfaced additional candidate listings that were dropped rather than fabricated:

- Chime Senior Software Engineer, ML Platform — excluded (Platform title; peer track).
- Ramp Staff ML Engineer, Mercury Sr ML Ops Engineer — excluded (Staff/MLOps titles).
- Robinhood Senior ML Engineer (Agentic / AI R&D) — Greenhouse detail pages redirected to the general careers index; verbatim details unavailable.
- Recursion Senior ML Engineer / LLM — echojobs.io returned 403.
- Walmart Global Tech Senior ML Engineer roles — listed WD IDs returned 404 on `careers.walmart.com` (postings appear rotated).
- Tempus AI — no active Senior ML Engineer titles currently posted on public boards (only Staff / Translational Scientist).
- Komodo Health, Goldman Sachs, The Trade Desk, JPMorgan, Amex, Comcast, Costco, Verily — search results did not resolve to a directly fetchable Senior ML Engineer detail page within the time budget.

The 26 verified postings still comfortably exceed the ≥ 25 target and span all three segments.

## Delta this cycle

See [`.aicg/curriculum-plan-delta.json`](.aicg/curriculum-plan-delta.json). **Intentionally empty.** No requirement theme cleared all three continuity-bias gates:

- **Ranking / recommender systems** — 7/26 postings (~0.27), just under the 0.30 gate. The `modeling-regime-decision-rubric` exercise in `mod-303-advanced-modeling` already covers when a ranking regime is the right choice; the mod-303 exercises can teach ranking-specific patterns without a new module. Re-evaluate next cycle if frequency crosses 0.30.
- **Agentic AI systems / tool-use orchestration** — 5/26 postings (~0.19), below the 0.30 gate. Delegation contract already covered in `mod-304`. Depth is owned by `senior-agentic-ai-engineer-learning` at level 40. Re-evaluate next cycle.
- **GPU profiling / speculative decoding / kernel optimization** — 5/26 postings (~0.19). Concentrated in AI-native / inference-shop roles where the *role itself* is the deep specialty. Owned by `training-pipeline-engineer` and `ai-infra-performance-learning`. Not in scope for the ML engineering ladder.
- **AI coding assistant fluency (Cursor / Copilot / Codex)** — 3/26 postings (~0.12). Emerging preferred-qual in 2026 postings. On-the-job scope, not a role-specific learning module. Re-evaluate next cycle.
- **LoRA / QLoRA / RLHF fine-tuning depth** — 6/26 postings (~0.23), always as an "ability to reach for" rather than the core skill. Owned by `fine-tuning-engineer`; delegation contract already in `mod-304`.

## Ownership map — quick reference

- **Senior ML Engineer (this track, level 30)** owns the *architecture, evaluation program, reliability, cross-team contracts, and technical leadership* around production ML systems. Build-altitude fundamentals are inherited from level 20.
- **ML Engineer (level 20)** owns the build-altitude workflow — this track links to it rather than re-teaching.
- **Staff / Principal ML Engineer** (levels 40 / 48) own multi-team architecture and org-wide strategy.
- **AI Infra ML Platform / MLOps** peer tracks own the paved-road platforms the senior ML engineer consumes.
- **Training Pipeline Engineer / AI Infra Performance** own distributed training and inference-engine depth (concentrated in AI-native postings this cycle).
- **Model Evaluation / AI Evaluation Engineer** own evaluation-platform depth.
- **NLP / RAG / LLM Application / Fine-Tuning / Applied AI / Senior Agentic AI** engineers own LLM and agent specialisations.
- **AI Infra Security** (level 35) owns ML/AI security depth.
- **AI Governance Analyst** owns governance and compliance depth.

## Conclusion

The 26 in-window postings validate every one of the 10 existing modules. No module needs demotion (all cleared the ≥ 0.30 in-scope frequency threshold; several — role scope, cross-team, architecture, modeling — cleared 0.65 or higher). No net-new theme cleared all three continuity-bias gates. The curriculum-plan delta for this cycle is empty by design — the market has not shifted enough in the last 90 days to justify additions, and the emerging surfaces (agents, ranking, AI coding tools) either sit under existing modules' exercises or are peer-track depth. Next research cycle should re-check ranking and agent-integration frequencies specifically — if either crosses 0.30, add an exercise to the corresponding existing module rather than a new module.
