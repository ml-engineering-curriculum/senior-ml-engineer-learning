# When to escalate to a specialist track

## Motivation

The most senior-altitude question in this module is not "which regime fits this problem?" — it is "**should this problem be my team's, or does it belong to a specialist track?**"

At L20 the answer is usually "yes, build it here." Every ML problem looks buildable and the muscle you are training is the modeling one. At L30 the muscle you are training is different: you are training the judgement that says *this piece is core to my team, this piece is a specialist's job, this piece is a platform's job.* Getting that judgement wrong costs quarters. A ranking team that decides to build its own fine-tuned LLM in-house without the specialist stack usually ships a mediocre one late; a ranking team that outsources a straightforward tabular ranker to a "LLM everything" specialist usually gets a worse product at higher cost.

The AICG curriculum ecosystem has explicit specialist tracks for the exact places where in-house work most often goes wrong: LLM prompting, RAG, LLM fine-tuning, distributed training pipelines, applied AI, ML platform. This chapter is the escalation rubric — how to recognise when you are entering another track's territory, what the hand-off contract looks like, and what stays on your team when you delegate.

## The peer specialist tracks — the map

Reachable from senior ML engineer at L30, each has its own paved road, its own vocabulary, and its own artefacts. The senior ML engineer's job is to know which is which.

### `llm-application-developer-learning`

**Territory.** Prompt design, prompt orchestration, tool use, agents, retrieval-augmented generation on top of hosted LLMs, guardrail engineering, LLM evaluation harnesses, LLM observability.

**Consume from.** If your ML system's answer is "call an LLM with a well-designed prompt and a small tool loop," this is the track that should own it (or that you should hand off to). You provide the domain problem and the evaluation harness; they provide the prompt / agent / harness engineering.

**Escalate when.** The load-bearing work is prompt engineering, tool-use orchestration, or agent design — not model training. If you are debugging why the LLM ignored your instructions on 5 % of inputs, that is llm-application territory.

### `rag-engineer-learning`

**Territory.** Retrieval-augmented generation systems specifically — chunking strategies, embedding models, vector stores, retrieval evaluation, groundedness metrics, ranker + LLM composition, citation quality.

**Consume from.** If your system's answer is "index a corpus and let an LLM answer questions grounded in it," this is the track. RAG has its own evaluation vocabulary (context recall, groundedness, faithfulness) and its own infrastructure stack (vector DBs, embedding pipelines, retrievers).

**Escalate when.** The load-bearing quality driver is *retrieval* — not classification, not ranking, not fine-tuning. A knowledge-base assistant, a documentation search, a compliance-question answering system are RAG-first problems.

### `fine-tuning-engineer-learning`

**Territory.** Supervised fine-tuning (SFT), instruction tuning, RLHF, DPO, parameter-efficient fine-tuning (LoRA, QLoRA) at production scale, evaluation of fine-tuned LLMs.

**Consume from.** If the answer to your problem needs a fine-tuned LLM — not a pre-trained-and-prompted one — this is the track.

**Escalate when.** You need to change the model's behaviour in ways prompting cannot achieve, or you need to run a fine-tune large enough to require the specialist stack (distributed training, RLHF pipelines, preference-data collection). A small in-house LoRA on a 1B model may still live on your team; a full SFT on a 70B model with RLHF almost certainly should not.

### `training-pipeline-engineer-learning`

**Territory.** Distributed training infrastructure — data parallel, tensor parallel, pipeline parallel; large-scale data loaders; checkpointing; distributed optimisation; hardware efficiency.

**Consume from.** You do not build training infrastructure at senior ML engineer altitude. You consume the paved-road training pipeline this track owns.

**Escalate when.** Your training run needs multi-node, multi-GPU orchestration. Your throughput is bottlenecked by data loading, not model compute. You need bespoke optimiser sharding.

### `applied-ai-engineer-learning`

**Territory.** Applied NLP, applied CV, applied speech in production — the specialist tracks for narrow-modality ML at scale.

**Escalate when.** The problem is squarely inside one modality's specialist territory (industrial-strength NER over legal documents, a large-scale image search system, an on-device speech pipeline) and the paved-road version of that specialist stack does the job better than a generalist ML build.

### `model-evaluation-engineer-learning` and `ai-eval-engineer-learning`

**Territory.** Evaluation infrastructure, evaluation harness design, offline and online evaluation, human-in-the-loop evaluation, LLM-as-judge evaluation, statistical significance.

**Consume from.** Once your system is beyond a certain scale, evaluation stops being a folder in your training repo and becomes a platform. These tracks own that platform.

**Escalate when.** You are building your third bespoke evaluation harness in a year. When the eval harness has its own on-call. When cross-team evaluation reuse would materially accelerate the whole org.

### `ai-infra-ml-platform-learning`, `ai-infra-mlops-learning`

**Territory.** The paved-road ML platform — feature stores, model registries, training pipelines, serving stack, monitoring. This is the [mod-308] peer platform relationship in [mod-302].

**Consume from.** If you are writing custom infrastructure that a peer platform team could provide, you are almost certainly on the wrong side of the escalation. Consume the platform; contribute back what is missing.

### `ai-governance-analyst-learning`, `ai-infra-security-learning`

**Territory.** Governance, compliance, model cards, risk assessments, red-teaming, secure serving.

**Escalate when.** The system is regulated, high-risk, or subject to formal audit. See [mod-309] (responsible AI governance) for the connection.

### `staff-ml-engineer-learning`, `principal-ml-engineer-learning`

**Territory.** Above you. Multi-quarter roadmap ownership, cross-team modeling strategy, hiring bar.

**Escalate when.** The scope is a program not a project, and you need someone who can arbitrate priorities across multiple L30 teams.

## The escalation decision rubric

Five questions. If the first two are "yes, this is another track's territory," escalate.

### Q1 — Is the load-bearing skill in another track's title?

Is the hard problem *prompt design*? RAG. Is it *fine-tuning at scale*? Fine-tuning. Is it *distributed training*? Training pipeline. Is it *the evaluation platform*? Evaluation.

If the load-bearing skill is in the name of a peer track, that is the strongest single signal. It is not a proof — sometimes an ML team is the right owner even for work adjacent to a peer track — but it is where the reviewer's first question should land.

### Q2 — Does the paved-road version of that specialist stack exist in-org?

If a paved-road RAG stack exists, and you build your own, you are duplicating work and taking on debt. If it does not exist yet, the escalation shape is: bring the requirement to the specialist track's roadmap conversation, not roll your own.

### Q3 — Is the ownership cost worth it to your team?

Even when a piece of work is buildable in-house, ask: what does owning it on-call look like for the next twelve months? Is the on-call surface expanding your team into a new competence you want, or into one you do not?

Sometimes the right answer is "we can build it, but the operational cost is not one we want to carry, so we consume the specialist's service."

### Q4 — Where does the evaluation live?

If the evaluation harness for this piece is in another track (LLM eval harnesses in the evaluation track; RAG groundedness metrics in the RAG track), consume both the build and the evaluation from that track. Splitting build from eval across teams is a reliable way to ship a system that no one can measure.

### Q5 — What is the hand-off contract?

If you do escalate, the L30 job is to *make the hand-off concrete*:

- **Problem statement.** What are you asking them to solve? Bring the mod-302 chapter 01 kind of clarity — prediction unit, consumer, freshness, volume, downstream cost.
- **Data.** What data do you own and can share? What data do they need from you?
- **Evaluation.** Who owns the eval harness? What are the acceptance criteria and their thresholds?
- **Interface.** REST endpoint, batched job, in-process library, streaming topic? See mod-302 chapter 02.
- **On-call.** Who pages who when the specialist's system misbehaves and it manifests as your system's outage?

That checklist is the hand-off contract. It is what turns "we escalated" from a shrug into a signed-and-defensible agreement.

## What stays on the ML team even after escalation

Delegation is not abdication. When your team consumes an LLM through the llm-application track, you still own:

- **The problem statement.** Only your team knows what the business is trying to do.
- **The evaluation slices.** Only your team knows which failure modes are unacceptable in your product.
- **The integration.** How the specialist's system plugs into your ranker, your batch pipeline, your monitoring.
- **The rollback.** If the specialist's system degrades, your team is the one that switches to the fallback.
- **The user-facing metric.** The specialist optimises their own metrics; your team owns the metric the business cares about.

The pattern is: **you delegate the how, not the what.**

## A worked example — a "product-question answering" feature

**Business problem.** "When users ask a question on a product page ('does this fit a 6-foot person?'), give them a helpful answer using the product description, reviews, and Q&A history."

Naive read: "That's an NLP / ML problem, my team owns it." Rubric walk:

- Q1 — load-bearing skill? *RAG.* The corpus is (product description + reviews + prior Q&A). The retrieval + grounded generation is the whole system. → RAG-track territory.
- Q2 — paved-road RAG stack? Yes, the RAG-engineer track has one.
- Q3 — ownership cost? Not one your team wants; you'd be building embedding pipelines, vector DBs, groundedness eval, LLM prompt engineering — a whole stack you do not otherwise use.
- Q4 — evaluation? RAG-specific (context recall, groundedness, citation quality). The RAG track has the harness.
- Q5 — hand-off contract? You own the problem, the acceptance criteria, the product surface, the fallback ("no answer available" is fine); they own the retrieval + generation stack.

→ Escalate. Your team's deliverable is the problem statement, the acceptance criteria, the integration, and the fallback. The RAG track's deliverable is the running system that meets the acceptance criteria.

## The reverse mistake — escalating what you should own

The opposite failure is real too. A team hears "the product wants better ranking" and says "let's ask the LLM track to build an LLM-based ranker." That is almost always the wrong move if the problem is a tabular ranking problem with plenty of labelled data.

Rubric on that reverse case:

- Q1 — load-bearing skill? *Tabular ranking on interaction data.* That is squarely senior ML engineer territory.
- Q2 — paved road? Yes, the ML platform's ranker training and serving stack.
- Q3 — ownership cost? Aligned with your team's existing competence.
- Q4 — evaluation? Ranking-evaluation harnesses your team already owns.

→ Own it. Do not escalate a problem that is core to your team's mandate just because a fashionable specialist track exists.

## Three failure modes to catch in review

- **Building the paved-road piece in-house.** The team builds a bespoke RAG stack because "the specialist track's version is not quite what we want." Six months later they own a RAG stack no one else uses and no one on the ML team wants to be on-call for. Fix: work with the specialist track to extend the paved road; do not fork it.
- **Escalating without the hand-off contract.** The team hands "figure out an LLM feature" to the LLM-application track without a problem statement, without acceptance criteria, without a fallback. Six months later there is a demo of something adjacent to what you asked for, and no one can say whether it should ship. Fix: the hand-off contract from Q5 is mandatory.
- **Escalating the ownership of the user-facing metric.** The team says "the LLM track owns this metric now." The LLM track will optimise for LLM-track metrics; the business metric drifts. Fix: the user-facing metric always stays with the team that owns the business surface.

## Summary

The most senior-altitude question in advanced modeling is what to build and what to hand off. The peer specialist tracks — LLM application, RAG, fine-tuning, training pipeline, applied AI, evaluation, platform, governance — exist precisely to own the pieces senior ML engineers most often build badly in-house. Escalation is pinned by five questions: load-bearing skill, paved-road availability, ownership cost, evaluation location, hand-off contract. Delegating the how does not mean delegating the what — the problem statement, evaluation, integration, fallback, and business metric stay with the ML team even after escalation. The reverse mistake (escalating a problem core to your team's mandate) is just as expensive.

Exercise-01 in this module is a decision rubric across all five regimes plus the escalation question — the compact application of everything in the module.
