# Prompt, RAG, fine-tune, hybrid: picking the pattern

## Motivation

Product walks into the room and says "we want the assistant to summarise the customer's ticket history and suggest a next action." Six ML teams in the same building will produce six different architectures for that sentence:

- A prompt-only call to a hosted frontier model, with the ticket history stuffed into the context window.
- A retrieval-augmented pipeline that indexes the ticket history plus the knowledge base, then prompts the LLM against the top-k retrievals.
- A fine-tuned smaller model trained on labelled ticket-summarisation examples.
- A hybrid where a classical classifier chooses the next action from a fixed taxonomy and an LLM writes the summary.
- A router that sends easy cases to a cheap classifier and hard cases to a frontier LLM.
- A batch pipeline that pre-computes summaries offline and stores them, with an LLM used only for cache misses.

All six can plausibly ship. Only two or three will still be alive in six months once cost, latency, evaluation, and drift bills come in. At L20 the question is "how do I call the LLM?" At L30 the question is **which of prompt / RAG / fine-tune / hybrid is the right pattern for this feature, and can I defend it against the alternatives?**

This chapter is the decision map. Chapters 02–06 drill into the sub-decisions each pattern carries. The escalation question — "should this feature even live on my team, or does it belong to a specialist track?" — is chapter 06.

## What "LLM-augmented ML feature" actually means

The senior ML engineer at L30 is almost never building a foundation model. You are not pre-training GPT. You are not fine-tuning a 70B model on eight nodes of H100s — that is the `fine-tuning-engineer-learning` track's territory (chapter 06). You are not owning the vector store or the retrieval eval harness at the depth `rag-engineer-learning` does. You are not designing the agent tool loop at the depth `llm-application-developer-learning` does.

What you *are* doing is deciding when an LLM belongs in your product feature at all, wiring it into an ML system that already has classical components (rankers, classifiers, feature stores, evaluation harnesses), and owning the cost / latency / reliability envelope. That is a distinct job. It uses the LLM as a *component*, not as a research artefact.

Every decision in this module falls out of that framing. When you read "prompt as code" in chapter 02, the "code" is the code your ML repo already lives in. When you read "guardrails" in chapter 04, the guardrails are wired into the same serving stack that hosts your existing classifier.

## The four patterns you should know cold

You should be able to describe each in one paragraph and name one production-shaped reference for each. Chapter references say where you go to drill in.

### 1. Prompt-only

A hosted LLM is called with a carefully authored prompt and (increasingly often) a JSON schema or tool specification. No external retrieval, no fine-tuning, no gradient step. The bet: the pre-trained model already knows enough, and the domain adaptation the feature needs fits inside the prompt.

Prompt-only is the right pattern more often than L20s expect and less often than "AI-first" product managers hope. It ships in days. It has almost no infrastructure surface area beyond the LLM API client, an eval harness, and a prompt registry. Failure modes are (a) prompt regression when the base model version changes underneath you, (b) prompt drift as the team edits without discipline, (c) cost blowup at scale because a naive prompt is expensive per call, and (d) hallucination when the model does not in fact know the domain. Chapter 02 is where you learn to treat the prompt as a versioned engineering artefact.

References. OpenAI's [prompt-engineering guide](https://platform.openai.com/docs/guides/prompt-engineering) and Anthropic's [prompt-engineering documentation](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview) are the vendor-authoritative starting points. Google Cloud's [prompt-design guide](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/introduction-prompt-design) covers the same territory for Vertex AI.

### 2. Retrieval-augmented generation (RAG)

The LLM is called with a prompt *and* a set of retrieved passages from an external corpus that the LLM was not trained on: your knowledge base, your product catalogue, the customer's own history. The bet: the model needs facts the base pre-training did not (or could not) memorise, and putting them in-context is cheaper and safer than putting them in the weights.

RAG is the right pattern whenever the answer depends on data that is (a) proprietary, (b) fresh, or (c) too voluminous to fit in a prompt. It is a substantially bigger build than prompt-only: chunking strategy, embedding model, vector store, retriever eval, citation format, groundedness evaluation. Failure modes are retrieval failures (the right passage was never retrieved), grounding failures (the passage was retrieved but the LLM ignored or contradicted it), and evaluation-loop failures (you cannot separate retrieval quality from generation quality when both regress at once).

The senior ML engineer's altitude on RAG is "when is this the pattern, and what does the hand-off to `rag-engineer-learning` look like?" — not "how do I chunk PDFs?" Lewis et al.'s [*Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*](https://arxiv.org/abs/2005.11401) (NeurIPS 2020) is the canonical academic reference. Anthropic's [contextual retrieval](https://www.anthropic.com/news/contextual-retrieval) writeup is a modern production-oriented tutorial. Databricks's [RAG evaluation guide](https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG) covers the eval side.

### 3. Fine-tuning

The model's weights are updated on a domain-specific dataset. In practice at L30 this almost always means one of: supervised fine-tuning (SFT) on labelled input/output pairs, instruction tuning on a curated instruction dataset, preference tuning (RLHF, DPO), or parameter-efficient fine-tuning (LoRA / QLoRA) that updates only a small adapter. The bet: prompting cannot make the model do what you need — the behaviour has to be baked into the weights.

Fine-tuning is the right pattern less often than L20s reach for it and *specifically* when: (a) the desired behaviour is a stable format or persona the model refuses to hold via prompting; (b) inference cost matters enough that a smaller fine-tuned model is cheaper than a larger prompted one; (c) the domain is far enough from pre-training that in-context examples are insufficient; (d) you have (or can create) enough labelled examples — typically thousands to tens of thousands, not dozens. Failure modes are catastrophic forgetting, over-fitting to the fine-tune distribution, evaluation regressions on capabilities you did not fine-tune for, and the infrastructure tax of running a training pipeline you did not need before.

At L30 this is almost always a hand-off to `fine-tuning-engineer-learning`. Your job is to (a) recognise when fine-tuning is warranted, (b) author the delegation contract (chapter 06), (c) consume the resulting artefact, and (d) evaluate it in your own harness. Hu et al.'s [*LoRA: Low-Rank Adaptation of Large Language Models*](https://arxiv.org/abs/2106.09685) (ICLR 2022) is the canonical LoRA reference; Ouyang et al.'s [InstructGPT paper](https://arxiv.org/abs/2203.02155) is the canonical instruction-tuning + RLHF reference; Rafailov et al.'s [DPO paper](https://arxiv.org/abs/2305.18290) is the canonical direct preference optimisation reference.

### 4. Hybrid classical + LLM

A classical ML component (a classifier, a ranker, a retriever, a rules engine) is composed with an LLM component (a prompt, a RAG pipeline, a fine-tuned generator) in the same feature. The bet: neither pattern alone gives the right cost / accuracy / latency trade-off, but their composition does.

The hybrid patterns worth knowing at L30 are drilled in chapter 03: **router** (cheap model decides, expensive model handles), **LLM as feature extractor** (LLM turns unstructured input into structured features that feed a classical model), **classical scorer + LLM presenter** (classical model decides *what*, LLM writes *how*), **LLM as verifier / fallback** (classical model handles the golden path, LLM catches the tail), and **offline enrichment** (LLM is called in a batch pipeline, results are cached and served like any other feature).

The hybrid patterns are where L30 differentiates from L20 most clearly. An L20 reaches for "LLM everything." An L30 recognises that a fraud model, a ranker, or a search index probably does not want an LLM in the golden path at all — but almost certainly wants an LLM *somewhere* in the system.

## The pattern-selection rubric

Six questions, in order. The first that forces a pattern pins it; later questions can add sub-decisions but cannot override.

### Q1 — Does the feature need generated text, generated structure, or a decision from a fixed taxonomy?

If the feature outputs a decision from a fixed, small taxonomy — one of ten policy categories, a fraud verdict, a ranking of items — and no natural-language explanation is required, an LLM is a bad default. A classical classifier is faster, cheaper, more evaluable, and does not hallucinate a class that does not exist. Move to the modeling regimes in [mod-303]. LLM may still show up in the *hybrid* pattern (chapter 03) as a router input or an explanation generator, but it is not the primary model.

If the feature needs *generated content* — a summary, a draft reply, an extracted structured object, a rewritten passage — an LLM is on the table. Keep walking.

### Q2 — Does the answer depend on data the base model was never trained on?

If the answer depends on proprietary, fresh, or user-specific data — the customer's own ticket history, your product catalogue as of this morning, an internal wiki — **the model needs that data at inference time**. Two answers: (a) put it in the prompt if it fits (prompt-only, chapter 02); (b) retrieve it if it does not (RAG, hand-off to `rag-engineer-learning`).

The line between "fits in the prompt" and "needs RAG" is not the token limit — it is *how much of the total corpus a single query might need*. If every query needs one small user-scoped record (last ten tickets), stuff it. If any query might need to search across a million passages, that is RAG.

If the answer depends only on general knowledge the base model already has, keep walking.

### Q3 — Can prompting alone produce the required behaviour reliably?

Reliably means: at your eval harness's per-slice pass rate threshold, on a representative eval set that includes adversarial and tail cases. "It worked in three examples I typed by hand" is not reliably.

If prompting reliably produces the behaviour, **stop and ship prompting**. This is the L30 tell: the temptation to fine-tune something that a prompt already handles is real, and it costs quarters. Chapter 02 makes prompting reliable enough to trust the answer here.

If prompting cannot get the behaviour above the pass-rate threshold, and you have already exhausted prompt-engineering techniques (few-shot examples, structured output, chain-of-thought, tool use, model-tier upgrade), fine-tuning is on the table (Q4).

### Q4 — Would fine-tuning solve a problem prompting demonstrably cannot?

Fine-tune when *both* are true:

- Prompting has been genuinely exhausted at the pass-rate threshold, with a paper trail from your eval harness showing which sub-slices are the ones that fail.
- Either the format / persona / style is one the base model refuses to hold, or a smaller fine-tuned model is materially cheaper at inference than a prompted larger one, or you need to remove behaviours the base model exhibits.

If yes, this is a hand-off to `fine-tuning-engineer-learning` (chapter 06). Your team almost certainly does not run the fine-tuning stack.

If no — if the honest answer is "prompting is 90 % there and we could ship if we invested in the eval harness and prompt-engineered harder" — then fine-tuning is a distraction. Ship prompting.

### Q5 — Would the classical + LLM composition beat either alone on cost, latency, or accuracy?

The five hybrid shapes (chapter 03):

- **Router.** Most queries are easy; a small model handles them; a bigger model catches the hard tail.
- **LLM as feature extractor.** Convert unstructured input into structured features that a classical model consumes.
- **Classical decides, LLM presents.** A classifier / ranker owns the decision; the LLM writes the human-facing explanation or draft.
- **LLM verifier / fallback.** Classical model handles the golden path; LLM verifies or handles the tail.
- **Offline enrichment.** LLM generates content in batch; results are cached and served with classical latency.

If any of these hybrid shapes lifts cost, latency, or accuracy materially above the LLM-only or classical-only baseline, they are on the table. In production this is more common than pure prompt-only or pure RAG, and it is where the ML engineer's altitude matters most — a specialist LLM engineer will over-index on the LLM pipeline; a specialist classical engineer will under-index on the LLM component. The L30 owns the composition.

### Q6 — Is any of this actually your team's job, or is it a specialist track's?

The senior read is that not every LLM-augmented feature should be built by the ML team. Some of them are the deliverable of a peer specialist track:

- **Deep RAG** with citation-quality evaluation, embedding-model selection, and vector-store operations belongs in `rag-engineer-learning`.
- **Fine-tuning at scale** (SFT, RLHF, DPO, distributed training) belongs in `fine-tuning-engineer-learning`.
- **Agentic tool loops** with multi-step planning, tool selection, and error recovery belong in `llm-application-developer-learning`.
- **Applied NLP pipelines at scale** — industrial-strength NER, classification, extraction over enormous document corpora — belong in `applied-ai-engineer-learning`.

Chapter 06 is entirely about the escalation contract. The L30 job is not to build all of these — it is to know which ones your team should build and which ones to hand off to (or consume from) a specialist track.

## The map, at a glance

```
   Q1: needs generated content?
     └── no  ──► classical model; LLM at most in a hybrid role (ch 03)
     └── yes ──►
        Q2: needs proprietary / fresh / user data?
          └── yes ──► put in prompt if it fits, else RAG (hand-off to rag-engineer)
          └── no  ──►
             Q3: prompting alone gets above the pass-rate threshold?
               └── yes ──► ship prompt-only (ch 02); consider hybrid (ch 03)
               └── no  ──►
                  Q4: does fine-tuning solve what prompting cannot?
                    └── yes ──► hand-off to fine-tuning-engineer (ch 06)
                    └── no  ──► return to Q3; the answer is more prompt work
   Q5: does a classical + LLM composition beat pure? ──► hybrid (ch 03)
   Q6: is this your team's job at all? ──────────────► escalate (ch 06)
```

## A worked example — "next-action assistant for support agents"

**Business problem.** "When an agent opens a ticket, show a suggested next action from a fixed taxonomy of eight, plus a one-paragraph summary of the customer's recent history."

Two outputs — a decision (next action) and a generated summary. Q1 says the *decision* is a classification problem (eight classes, no generation needed); the *summary* is generation. So the shape is already hybrid.

- **Decision side.** Fixed taxonomy, hundreds of thousands of labelled historical (ticket, next-action) pairs from resolved cases. This is a classical classifier (mod-303 chapter 01 rubric). Fast, cheap, calibrated, evaluable. An LLM here would be worse on every dimension.
- **Summary side.** Q2 — summary depends on the customer's own history (proprietary + fresh). The history for one customer is small (last ten tickets), so it fits in a prompt; RAG is unnecessary for this slice. Q3 — a prompt with the history stuffed into context can produce a summary reliably; test on a labelled eval set and confirm. Q4 — no need to fine-tune.

So the pattern is: **classical classifier + prompt-only LLM summariser, composed as a hybrid**. Cost / latency guardrails (chapter 04) go on the LLM branch: token cap on the history slice, timeout, degradation path to "no summary shown" if the LLM slice is slow or errors. Observability (chapter 05) logs the classifier decision, the LLM prompt, the LLM output, and the token count per call. Prompt registry (chapter 02) versions the summariser prompt like any other artefact in the repo. No hand-off (Q6) — this is squarely on the ML team.

Change the problem shape and the answer changes. If the summary needs to cite the exact wiki article that resolved a similar past ticket, Q2 flips to "search a million-passage corpus" and RAG is now on the table — hand-off to `rag-engineer-learning`. If product wants the summary to sound in the company's brand voice and the base model refuses, Q4 may flip and fine-tuning becomes a candidate — hand-off to `fine-tuning-engineer-learning`.

## Three failure modes the rubric catches

- **"Fine-tune because we haven't tried harder prompts."** A team spends a quarter labelling a fine-tune dataset when two more days of prompt engineering and structured outputs would have shipped the feature. Q3 catches this if you enforce the exhaustion clause honestly.
- **"LLM in the golden path because it's cool."** A team wraps an LLM around a classification problem that a calibrated classifier already solved. The result is 10× the latency, 100× the cost, occasional hallucinated classes outside the taxonomy, and no measurable quality lift. Q1 catches this.
- **"Built it here because we didn't know a specialist track existed."** A team builds a bespoke RAG stack over a documentation corpus, complete with a vector database they now operate, without engaging `rag-engineer-learning`. Six months later they discover the peer track's paved-road RAG platform. Q6 catches this — chapter 06 is where you learn to make the call.

## Summary

At L30 the LLM question is not "how do I call the API" — it is "which of prompt / RAG / fine-tune / hybrid fits this feature." A six-question rubric — needs generation, needs proprietary data, prompting sufficient, fine-tuning warranted, hybrid lifts the trade-off, specialist territory — pins the pattern to the problem shape rather than to fashion. Prompt-only is the underrated default; RAG belongs when the answer depends on data the base model never saw; fine-tuning is a hand-off, not something you casually pick up; hybrid is the pattern L30s are the best in the room at composing. Chapter 02 makes prompting reliable enough to trust; chapter 03 walks the hybrid shapes; chapters 04–05 wire the guardrails and observability; chapter 06 owns the delegation contract.
