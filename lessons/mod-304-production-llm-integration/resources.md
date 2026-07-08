# Resources for mod-304-production-llm-integration

Curated external references. These are the primary and authoritative sources cited or leaned on by the chapters and exercises above. Every URL is publicly reachable at the time of authoring.

## Vendor documentation (all chapters)

The vendor-authoritative docs are the ground truth for pricing, rate limits, model versions, and API surface. Consult them directly rather than relying on second-hand summaries — point-in-time numbers on these pages change often.

- **Anthropic — API documentation.** <https://docs.anthropic.com/en/api/overview>
- **Anthropic — pricing.** <https://www.anthropic.com/pricing>
- **Anthropic — models overview and deprecations.** <https://docs.anthropic.com/en/docs/about-claude/models/overview>
- **Anthropic — rate limits.** <https://docs.anthropic.com/en/api/rate-limits>
- **OpenAI — API reference.** <https://platform.openai.com/docs/api-reference>
- **OpenAI — pricing.** <https://openai.com/api/pricing/>
- **OpenAI — model list and deprecations.** <https://platform.openai.com/docs/models>
- **OpenAI — rate limits.** <https://platform.openai.com/docs/guides/rate-limits>
- **Google — Gemini API pricing.** <https://ai.google.dev/pricing>
- **Google — Gemini API rate limits.** <https://ai.google.dev/gemini-api/docs/rate-limits>
- **Google Cloud — Vertex AI generative-AI documentation.** <https://cloud.google.com/vertex-ai/generative-ai/docs>
- **AWS Bedrock — pricing.** <https://aws.amazon.com/bedrock/pricing/>
- **AWS Bedrock — quotas and limits.** <https://docs.aws.amazon.com/bedrock/latest/userguide/quotas.html>

## Prompt-only, prompts as engineering artefacts (chapters 01, 02, exercises 01–02)

- **Anthropic — prompt engineering overview.** <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview>
- **OpenAI — prompt engineering guide.** <https://platform.openai.com/docs/guides/prompt-engineering>
- **Google Cloud — introduction to prompt design.** <https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/introduction-prompt-design>
- **Anthropic — structured output via tool use.** <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview>
- **OpenAI — structured outputs.** <https://platform.openai.com/docs/guides/structured-outputs>
- **Reynolds and McDonell, *Prompt Programming for Large Language Models*** (2021) — early framing of prompts as programs. <https://arxiv.org/abs/2102.07350>
- **Wei, Wang, Schuurmans, Bosma, Ichter, Xia, Chi, Le, Zhou, *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*** (NeurIPS 2022) — the canonical CoT reference. <https://arxiv.org/abs/2201.11903>
- **Kojima, Gu, Reid, Matsuo, Iwasawa, *Large Language Models are Zero-Shot Reasoners*** (NeurIPS 2022) — "let's think step by step" as a prompt-engineering baseline. <https://arxiv.org/abs/2205.11916>
- **Wang, Wei, Schuurmans, Le, Chi, Narang, Chowdhery, Zhou, *Self-Consistency Improves Chain of Thought Reasoning in Language Models*** (ICLR 2023). <https://arxiv.org/abs/2203.11171>
- **Schulhoff, Ilie, Balepur, Kahadze, Liu, Si, Li, Gupta, Han, Schulhoff, Dulepet, Vidyadhara, Ki, Agrawal, Pham, Kroiz, Li, Tao, Srivastava, Da Costa, Gupta, Rogers, Goncearenco, Sarli, Galynker, Peskoff, Carpuat, White, Anadkat, Hoyle, Resnik, *The Prompt Report: A Systematic Survey of Prompting Techniques*** (2024) — comprehensive taxonomy of prompting techniques. <https://arxiv.org/abs/2406.06608>
- **Anthropic — prompt caching.** <https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching>
- **OpenAI — prompt caching.** <https://platform.openai.com/docs/guides/prompt-caching>
- **Google — Gemini API context caching.** <https://ai.google.dev/gemini-api/docs/caching>

## Retrieval-augmented generation (chapter 01, hand-off framing)

Deep RAG lives in `rag-engineer-learning`. These references let the senior ML engineer set up the delegation contract with vocabulary they can defend.

- **Lewis, Perez, Piktus, Petroni, Karpukhin, Goyal, Küttler, Lewis, Yih, Rocktäschel, Riedel, Kiela, *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*** (NeurIPS 2020) — the canonical academic reference. <https://arxiv.org/abs/2005.11401>
- **Gao, Xiong, Gao, Jia, Pan, Bi, Dai, Sun, Wang, Wang, *Retrieval-Augmented Generation for Large Language Models: A Survey*** (2024) — modern survey covering the industrial-production shape. <https://arxiv.org/abs/2312.10997>
- **Anthropic — contextual retrieval.** <https://www.anthropic.com/news/contextual-retrieval>
- **Databricks — RAG evaluation and LLM-as-judge best practices.** <https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG>

## Fine-tuning and preference tuning (chapter 01, hand-off framing)

Deep fine-tuning lives in `fine-tuning-engineer-learning`. Same shape — the references here are enough to author the delegation contract, not to build the pipeline.

- **Hu, Shen, Wallis, Allen-Zhu, Li, Wang, Wang, Chen, *LoRA: Low-Rank Adaptation of Large Language Models*** (ICLR 2022) — the reference for parameter-efficient fine-tuning. <https://arxiv.org/abs/2106.09685>
- **Dettmers, Pagnoni, Holtzman, Zettlemoyer, *QLoRA: Efficient Finetuning of Quantized LLMs*** (NeurIPS 2023). <https://arxiv.org/abs/2305.14314>
- **Ouyang, Wu, Jiang, Almeida, Wainwright, Mishkin, Zhang, Agarwal, Slama, Ray, Schulman, Hilton, Kelton, Miller, Simens, Askell, Welinder, Christiano, Leike, Lowe, *Training language models to follow instructions with human feedback (InstructGPT)*** (NeurIPS 2022) — the canonical instruction-tuning + RLHF reference. <https://arxiv.org/abs/2203.02155>
- **Rafailov, Sharma, Mitchell, Ermon, Manning, Finn, *Direct Preference Optimization: Your Language Model is Secretly a Reward Model (DPO)*** (NeurIPS 2023). <https://arxiv.org/abs/2305.18290>
- **Anthropic — fine-tuning on Bedrock.** <https://docs.aws.amazon.com/bedrock/latest/userguide/model-customization-fine-tune.html>
- **OpenAI — fine-tuning guide.** <https://platform.openai.com/docs/guides/fine-tuning>

## Hybrid classical + LLM patterns (chapter 03, exercise 03)

- **Anthropic — orchestration and agent patterns.** <https://www.anthropic.com/research/building-effective-agents>
- **Ma, Achiam, Bansal, Bavarian, Belgum, Bello, Berdine, Bernadett-Shapiro, Berner, Bogdonoff, Boiko, Boyd, Brakman, Brockman, Brooks, Brundage, Button, Cai, Campbell, ..., et al., *GPT-4 Technical Report*** (2023) — architectural context for hybrid systems' expensive-model tier. <https://arxiv.org/abs/2303.08774>
- **Ong, Almahairi, Wu, Chiang, Wu, Gonzalez, Kadous, Stoica, *RouteLLM: Learning to Route LLMs with Preference Data*** (2024) — a modern reference for LLM routing between tiers. <https://arxiv.org/abs/2406.18665>
- **Bang, Ma, Ohkuma, *A Systematic Study on Retrieval + Reranking for LLMs*** (see: <https://arxiv.org/abs/2504.07717>) — retrieval + rerank composition patterns. <!-- needs-research: verify the primary citation for LLM-as-reranker in production; the composition pattern is well-known but a single canonical paper is not obvious. -->
- **Zilliz — GPTCache** (open-source semantic caching for LLM applications). <https://github.com/zilliztech/GPTCache>

## Cost, latency, and reliability (chapter 04, exercise 04)

- **Michael T. Nygard, *Release It! Second Edition*** (Pragmatic Bookshelf, 2018) — the canonical reference on the circuit breaker, bulkhead, and other resilience patterns for hosted-vendor dependencies. <https://pragprog.com/titles/mnee2/release-it-second-edition/>
- **Gil Tene, *How NOT to Measure Latency*** (InfoQ) — the reference on coordinated-omission bias in latency measurement. <https://www.infoq.com/presentations/latency-response-time/>
- **Anthropic — message batches API.** <https://docs.anthropic.com/en/docs/build-with-claude/batch-processing>
- **OpenAI — batch API.** <https://platform.openai.com/docs/guides/batch>
- **Google — Gemini batch API.** <https://ai.google.dev/gemini-api/docs/batch-mode>
- **AWS Bedrock — batch inference.** <https://docs.aws.amazon.com/bedrock/latest/userguide/batch-inference.html>
- **Google SRE — the four golden signals** (SRE Book, chapter 6). <https://sre.google/sre-book/monitoring-distributed-systems/#xref_monitoring_golden-signals>
- **Google SRE — alerting on SLOs.** <https://sre.google/workbook/alerting-on-slos/>

## Observability (chapter 05)

- **OpenTelemetry — generative AI semantic conventions.** The vendor-neutral standard for LLM span attributes; every observability tool is converging on it. <https://opentelemetry.io/docs/specs/semconv/gen-ai/>
- **OpenTelemetry — instrumentation overview.** <https://opentelemetry.io/docs/concepts/instrumentation/>
- **Prometheus — naming and labels best practices.** The canonical reference on why unbounded metric labels destroy metric backends. <https://prometheus.io/docs/practices/naming/>
- **Google SRE Workbook — multi-window multi-burn-rate alerts.** <https://sre.google/workbook/alerting-on-slos/#6-multiwindow-multi-burn-rate-alerts>
- **Anthropic — usage and cost API.** <https://docs.anthropic.com/en/api/usage-cost-api>
- **OpenAI — usage endpoints.** <https://platform.openai.com/docs/api-reference/usage>
- **OpenAI — moderation guide.** <https://platform.openai.com/docs/guides/moderation>
- **Langfuse — open-source LLM observability.** <https://langfuse.com/>
- **Traceloop — OpenLLMetry (OpenTelemetry-based instrumentation for LLMs).** <https://www.traceloop.com/openllmetry>
- **Arize AI — Phoenix (open-source LLM tracing / eval).** <https://phoenix.arize.com/>
- **Helicone — LLM observability.** <https://www.helicone.ai/>

## Vendor status and incident surfaces (chapters 04, 05)

Wire these into on-call runbooks — a vendor incident is often the first-cause of your error-rate SLO trip.

- **Anthropic — status page.** <https://status.anthropic.com/>
- **OpenAI — status page.** <https://status.openai.com/>
- **Google Cloud — status dashboard.** <https://status.cloud.google.com/>
- **AWS — health dashboard.** <https://health.aws.amazon.com/>

## Delegation and specialist tracks (chapter 06)

The escalation chapter refers to peer tracks in the wider AICG curriculum ecosystem. Read the READMEs of each so the hand-off contracts are concrete.

- Peer LLM specialist tracks (consume-from targets): `llm-application-developer-learning`, `rag-engineer-learning`, `fine-tuning-engineer-learning`, `training-pipeline-engineer-learning`.
- Peer evaluation tracks: `model-evaluation-engineer-learning`, `ai-eval-engineer-learning`.
- Peer platform tracks: `ai-infra-ml-platform-learning`, `ai-infra-mlops-learning`.
- Governance / security counterparts: `ai-governance-analyst-learning`, `ai-infra-security-learning`.
- Higher-level tracks: `staff-ml-engineer-learning`, `principal-ml-engineer-learning`.

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->

## Canonical texts (all chapters)

- **Chip Huyen, *Designing Machine Learning Systems*** (O'Reilly, 2022) — chapter 8 (monitoring) is the classical framing that chapter 05 extends for LLM-augmented systems. <https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/> · <https://huyenchip.com/books/>
- **Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (eds.), *Site Reliability Engineering*** (O'Reilly, 2016) — the SLO / SLI vocabulary chapters 04–05 lean on. Freely available online. <https://sre.google/sre-book/table-of-contents/>
- **Betsy Beyer, Niall Richard Murphy, David K. Rensin, Kent Kawahara, Stephen Thorne (eds.), *The Site Reliability Workbook*** (O'Reilly, 2018) — companion volume; alerting-on-SLOs chapter is directly relevant to chapter 05. <https://sre.google/workbook/table-of-contents/>

## Cross-references inside this module

- Chapter 01 (`01-prompt-rag-finetune-hybrid-decision.md`) is the pattern-selection rubric; every other chapter drills into one of its branches.
- Chapter 02 (`02-prompts-as-engineering-artifacts.md`) feeds the versioning discipline chapter 05 relies on (`prompt_version` as a first-class metric label).
- Chapter 03 (`03-hybrid-classical-plus-llm-patterns.md`) feeds the composition ownership framing chapter 06 leans on ("you delegate the how, not the what").
- Chapter 04 (`04-cost-and-latency-guardrails.md`) sets the envelope; chapter 05 is how you see it slip.
- Chapter 05 (`05-observability-for-llm-augmented-systems.md`) sets the observability contract that chapter 06's hand-off document formalises with a peer track.
- Chapter 06 (`06-delegation-contract-to-llm-specialists.md`) is the LLM-specific application of the mod-303 chapter 08 escalation rubric.

## Cross-references to other modules

- mod-302 chapter 01 (system shape) and chapter 02 (batch / streaming / online) are the vocabulary chapter 06's problem-statement and interface fields draw on.
- mod-303 chapter 08 is the general escalation rubric; chapter 06 in this module is its LLM-specific specialisation.
- mod-305 covers advanced evaluation and how the eval harness relates to release qualification, feeding chapter 06's evaluation-hand-off field.
- mod-307 covers ML SLOs, cost budgets, and incident response — the SLI framing chapter 05's alerting section extends.
- mod-308 covers peer-platform collaboration; chapter 06 is a specific case of the delegation discipline it teaches.
- mod-309 covers responsible-AI review, model cards, and data-lineage — inputs to chapter 06's data-hand-off field for regulated deployments.
