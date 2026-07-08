# Hand-off contracts to peer tracks

## Motivation

The single most common failure mode for a newly-promoted senior ML engineer is **scope creep**: the person who owns everything in their production ML system starts owning things that a peer track should own — the feature-store platform, the LLM fine-tuning run, the evaluation harness, the compliance sign-off. Every hour spent there is an hour not spent on the architecture, roadmap, and mentorship that L30 is actually graded on.

The remedy is to make the **hand-off contracts** to peer specialist and peer platform tracks explicit. This chapter names those tracks, describes the contract in each direction, and gives you a hand-off map you will reuse in exercise-02.

## The two categories of neighbour

Every peer track that shows up around the Senior ML Engineer role falls into one of two categories, and the shape of the contract is different for each.

- **Peer platform tracks.** These teams build the paved road you consume — feature stores, model registries, training clusters, deployment platforms, evaluation harnesses. Your default posture is **consumer on the paved road**; you contribute back via RFCs when the paved road does not fit; you do *not* build a shadow platform inside your team. Owners: `ai-infra-ml-platform-learning`, `ai-infra-mlops-learning`, `training-pipeline-engineer-learning`, `model-evaluation-engineer-learning`, `ai-eval-engineer-learning`.
- **Peer specialist tracks.** These are engineers with deeper depth in a domain than you — LLM application, RAG, NLP, fine-tuning, applied AI. Your default posture is **integrator with a delegation contract**; you own the integration boundary, they own the depth. You do *not* deep-dive into their specialism as a matter of course; you know enough to write a good spec and receive a good deliverable. Owners: `llm-application-developer-learning`, `rag-engineer-learning`, `nlp-engineer-learning`, `fine-tuning-engineer-learning`, `applied-ai-engineer-learning`.

A third category — the **review counterparts** who sign off on the artefacts you author — is close enough to peer that it belongs here too: `ai-infra-security-learning` (ML/AI security depth) and `ai-governance-analyst-learning` (governance and compliance depth). You author the review packet (mod-309), they sign it.

## The hand-off map

The table below is the working reference for the Senior ML Engineer's hand-off contracts. For every peer track:

- **You own** — what stays inside your ownership boundary as the Senior ML Engineer.
- **They own** — what belongs on the other side of the contract.
- **Interface** — the concrete artefact or protocol that carries the contract.
- **Module** — which of the following modules teaches the contract in depth.

### Peer platform tracks (paved-road consumption)

| Peer track | You own (L30) | They own | Interface | Module |
|---|---|---|---|---|
| `ai-infra-ml-platform-learning` | Feature spec, feature freshness SLO, consumer of the feature store, contribute-back RFC when the paved road is missing something | The feature store product itself, its SLA, its multi-tenant compute | Feature spec doc + platform SLA + shared feature registry entry | mod-302 (architecture), mod-308 (collaboration) |
| `ai-infra-mlops-learning` | Deployment spec, canary/shadow plan, retraining trigger definition, roll-back plan | The CI/CD-for-ML platform, the shared model registry, the retraining orchestration primitives | Deployment RFC + registry entry + on-call runbook | mod-302, mod-307, mod-308 |
| `training-pipeline-engineer-learning` | Training job spec, dataset lineage, resource envelope, priority tier | The distributed-training platform, GPU cluster ops, NCCL tuning, cost & scheduling | Training job spec + cost budget + on-call escalation path | mod-303 (awareness), mod-308 |
| `model-evaluation-engineer-learning` / `ai-eval-engineer-learning` | Slice definitions, holdout data, adversarial cases, LLM-judge rubric, review-packet template | The eval harness itself, the judge platform, the shared eval infra | Slice spec + rubric + shared eval-harness entry | mod-305, mod-308 |

### Peer specialist tracks (delegation contract)

| Peer track | You own (L30) | They own | Interface | Module |
|---|---|---|---|---|
| `llm-application-developer-learning` | The LLM-integration decision (prompt vs. RAG vs. fine-tune vs. hybrid), the cost/latency guardrails, the integration surface, the eval gates | Prompt engineering depth, agentic flows, LLM app framework depth | Integration spec + prompt version registry + eval gate | mod-304 |
| `rag-engineer-learning` | RAG-adoption decision, retrieval quality gates, the eval slice set for retrieval | Retrieval architecture depth, chunking/embedding decisions, retrieval infra | Retrieval spec + retrieval-quality SLO | mod-304 |
| `nlp-engineer-learning` | NLP problem framing, downstream task metric, integration into the wider ML system | Model choice and fine-tuning depth for NLP, tokeniser and preprocessing depth | Task spec + eval rubric | mod-303, mod-304 |
| `fine-tuning-engineer-learning` | Fine-tune-vs-not decision, dataset envelope, eval gates, deployment surface | LoRA/QLoRA/DPO/RLHF depth, dataset engineering depth, training loop specifics | Fine-tune spec + dataset spec + eval gate | mod-303, mod-304 |
| `applied-ai-engineer-learning` | System integration around applied AI features, the product-level metric | Product-facing AI feature depth, prototype-to-production applied AI craft | Feature spec + integration boundary | mod-304 |

### Review counterparts (packet authorship + sign-off)

| Peer track | You own (L30) | They own | Interface | Module |
|---|---|---|---|---|
| `ai-infra-security-learning` (L35) | Data-lineage & PII inventory, threat-model draft for realistic misuse, mitigation plan | Deep ML/AI security review, data poisoning / model extraction / adversarial input depth, final sign-off | Review packet section + threat model doc + sign-off | mod-309 |
| `ai-governance-analyst-learning` | Model card, data statement, fairness/robustness/safety slice results, review-cadence artefact | Governance framework, compliance / regulatory sign-off, policy authorship | Review packet section + governance sign-off | mod-309 |

## How to use the hand-off map — a worked example

**Situation:** the product team asks your ML team to add an LLM-based summariser to the ranking result page.

Without the hand-off map, the L20 instinct is: "I'll pick a model, wire up a prompt, ship it, add a monitor." This works for a prototype and fails for a product.

With the hand-off map, the L30 sequence is:

1. **Frame the integration decision.** Prompt-only, RAG, fine-tune, or hybrid? (mod-304). This decision is yours. Write it up.
2. **Delegate the depth.** If the answer is RAG, the retrieval architecture goes to `rag-engineer-learning` scope — you write the retrieval spec (quality bar, freshness, corpus scope) and receive the retrieval implementation. If the answer is fine-tuning, the fine-tune goes to `fine-tuning-engineer-learning` scope — you write the fine-tune spec (dataset envelope, eval gates, roll-out surface) and receive the trained model. If the answer is prompt-only, the deep prompt engineering goes to `llm-application-developer-learning` scope.
3. **Consume the paved-road platforms.** The eval harness for the summariser is authored on top of the shared eval infra (`model-evaluation-engineer-learning`). The deployment goes through the shared model registry (`ai-infra-mlops-learning`). You do *not* build a bespoke eval framework inside your team.
4. **Author the review packet.** Model card, data statement, threat model draft. Hand to `ai-infra-security-learning` and `ai-governance-analyst-learning` for review + sign-off.
5. **Own the system.** SLOs, cost budget, incident response, retraining/re-eval cadence, the integration boundary. You are the tech lead for the summariser system, not for any of the specialisms it composes.

The exercise is: the L30 owns the decision and the integration and the SLOs. The specialists own the depth. The platform teams own the paved road.

## The three anti-patterns of hand-off failure

Three failure modes are worth naming so you can catch them in yourself and in your team's ML system RFCs.

- **The "I'll just build it" anti-pattern.** The senior spins up a shadow feature store, a bespoke eval harness, or a hand-rolled deployment pipeline inside their team because the paved road "doesn't quite fit." Six months later they own an unmaintained internal platform. Fix: file a contribute-back RFC (mod-308) instead. If the paved road genuinely does not fit and cannot be extended, escalate to L40+ for a platform decision, do not silently fork.
- **The "I'll just delegate" anti-pattern.** The senior throws an under-specified problem at a peer specialist ("build me the RAG system") and is surprised when what comes back does not fit their production integration constraints. Fix: the delegation contract must include the eval gates, the SLO envelope, and the integration surface. The specialist owns the depth; you own the boundary.
- **The "I'll just sign it" anti-pattern.** The senior treats the responsible-AI review as a rubber stamp — no threat model draft, no slice results, no data statement — and hands the whole packet to the security or governance reviewer. This is not a hand-off; it is a punt. Fix: you author the packet; they review and sign it (mod-309).

## Summary

At L30 your hand-off contracts to peer tracks are load-bearing artefacts of your job, not the edge of it. Peer *platform* tracks give you the paved road you consume — your default posture is consumer, and your escalation is a contribute-back RFC. Peer *specialist* tracks give you depth you integrate — your default posture is integrator with a delegation contract that names eval gates, SLOs, and the integration surface. Review counterparts sign off packets that *you* author. Exercise-02 has you draw this hand-off map for a specific ML system.
