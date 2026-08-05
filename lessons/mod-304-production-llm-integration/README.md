# mod-304-production-llm-integration: Production LLM Integration for ML Systems

**Estimated effort:** 14 hours

At L20 the LLM question was "how do I call the API?" At L30 the question is different: **which of prompt / RAG / fine-tune / hybrid is the right pattern for this feature, and can I defend it against the alternatives?** This module teaches the pattern decision, the discipline that keeps prompts from rotting once they are in production, the hybrid classical + LLM shapes that show up in real ML systems, the cost / latency guardrails an LLM-augmented feature has to carry, the observability that lets you see the envelope slip, and the delegation contract to the four LLM specialist tracks (`llm-application-developer-learning`, `rag-engineer-learning`, `fine-tuning-engineer-learning`, `training-pipeline-engineer-learning`) when depth is needed.

It is the "LLM brain" companion to [`mod-303-advanced-modeling`](../mod-303-advanced-modeling/) (the "modeling brain") and [`mod-302-ml-systems-architecture`](../mod-302-ml-systems-architecture/) (the "systems brain"). You use all three when a new LLM-adjacent product feature lands.

## Learning objectives

- Decide between prompt, RAG, fine-tune, and hybrid classical+LLM patterns for a real product feature.
- Treat prompts as engineering artifacts: versioned, tested, evaluated, and deployed like code.
- Design cost and latency guardrails for LLM-augmented systems and instrument the observability to see them slip.
- Author the delegation contract to `fine-tuning-engineer` / `rag-engineer` / `llm-application-developer` when depth is needed.

## Chapters

1. [`01-prompt-rag-finetune-hybrid-decision.md`](01-prompt-rag-finetune-hybrid-decision.md) — the four patterns and the six-question rubric that pins each one to a problem shape.
2. [`02-prompts-as-engineering-artifacts.md`](02-prompts-as-engineering-artifacts.md) — the prompt bundle, versioning against the pinned model version, deterministic tests, per-slice evaluation, staged rollout.
3. [`03-hybrid-classical-plus-llm-patterns.md`](03-hybrid-classical-plus-llm-patterns.md) — router, LLM as feature extractor, classical decides + LLM presents, LLM verifier / fallback, offline enrichment.
4. [`04-cost-and-latency-guardrails.md`](04-cost-and-latency-guardrails.md) — the five-axis envelope (per-request cost, per-request latency, concurrency, aggregate spend, degradation path), levers on each, timeouts and circuit breakers.
5. [`05-observability-for-llm-augmented-systems.md`](05-observability-for-llm-augmented-systems.md) — the five signal families, OpenTelemetry gen-ai conventions, the `prompt_version` axis, quality signals from schema conformance to offline replay, cost as an SLI, alert design.
6. [`06-delegation-contract-to-llm-specialists.md`](06-delegation-contract-to-llm-specialists.md) — the four peer tracks, the six-question delegation rubric, the eight-field hand-off contract, what stays with your team even after escalation.

## Exercises

- [`exercises/exercise-01-prompt-vs-rag-vs-finetune-decision.md`](exercises/exercise-01-prompt-vs-rag-vs-finetune-decision.md) — apply the chapter 01 rubric to three business problems and defend the pattern choice, including "escalate to a specialist track" as a valid answer.
- [`exercises/exercise-02-prompt-as-code.md`](exercises/exercise-02-prompt-as-code.md) — turn a scratch-note prompt into a versioned bundle with deterministic tests, a per-slice eval set, and a staged rollout plan.
- [`exercises/exercise-03-hybrid-classical-plus-llm-feature.md`](exercises/exercise-03-hybrid-classical-plus-llm-feature.md) — design a hybrid feature end-to-end, naming the hybrid shape, the composition contract, and the two-layer evaluation shape.
- [`exercises/exercise-04-llm-cost-and-latency-guardrails.md`](exercises/exercise-04-llm-cost-and-latency-guardrails.md) — sketch the five-axis envelope and instrument the observability that catches it slipping, ending in a one-page guardrails design doc.

## Labs & quizzes

- `labs/` — reserved for a longer-form LLM-integration lab in a future authoring cycle.
- `quizzes/` — reserved for a knowledge check in a future authoring cycle.

## Resources

External references — vendor documentation, primary papers, production writeups — are catalogued in [`resources.md`](resources.md). Chapters cite Lewis et al. on RAG, Hu et al. on LoRA, Ouyang et al. on InstructGPT, Rafailov et al. on DPO, Zheng et al. on LLM-as-judge, Anthropic and OpenAI prompt-engineering documentation, OpenTelemetry's gen-ai semantic conventions, the SRE Book on golden signals and alerting, Nygard's *Release It!* on circuit breakers, and the four peer-track READMEs for the delegation vocabulary.

## How this module hands off

- The pattern decision (chapter 01) and hybrid shape (chapter 03) feed the RFC skeleton from mod-302 chapter 06 and the paired project `project-302-llm-augmented-ml-feature`.
- The prompt-versioning and evaluation discipline (chapter 02) hands off to mod-305 (advanced evaluation) — the release harness that enforces the pass-rate gate at every prompt change and every base-model bump.
- The cost / latency envelope (chapter 04) and the observability (chapter 05) hand off to mod-307 (reliability, SLOs) — cost is a first-class SLI alongside latency and error rate.
- The delegation contract (chapter 06) hands off to mod-308 (platform / peer collaboration) and to the paired project `project-302-llm-augmented-ml-feature`, which is the natural place to exercise a hand-off to one of the four LLM specialist tracks.
