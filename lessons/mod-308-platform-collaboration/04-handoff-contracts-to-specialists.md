# Hand-off contracts: training-pipeline-engineer and model-evaluation-engineer

## Motivation

The paved road of chapter 01, the build-vs-adopt calculus of chapter 02, and the RFC discipline of chapter 03 all handle the *peer platform team* relationship — teams whose job is to build tooling every ML team uses. This chapter is about a different relationship: the *specialist track* relationship, where the peer team's job is deep expertise in a narrow surface that your ML team needs occasionally.

The two specialist tracks the AICG curriculum names as peer-of-senior-ML-engineer are:

- **`training-pipeline-engineer`** — depth on distributed training, GPU cluster ops, checkpoint management, mixed-precision numerics, throughput / MFU optimisation, and the kernel-level performance work that gets a training run from "runs" to "runs efficiently at scale."
- **`model-evaluation-engineer`** (with adjacent `ai-eval-engineer` for LLM-eval depth) — depth on evaluation-platform build (LLM-judge harnesses, agentic-eval infrastructure, calibrated scoring across slices), the eval-harness *engineering* that mod-305 consumes.

The AICG requirements catalogue puts both at 0.19 and 0.12 frequency in the sampled 26 Senior ML Engineer postings — below the 0.30 in-scope threshold. That is *not* a signal these specialties don't matter; it is a signal that the senior ML engineer does not build depth in them, but *hands off* to them when depth is needed. The hand-off is where a lot of value leaks — a bad hand-off makes the specialist re-derive the context the ML team already has, or delivers depth that solves the wrong problem, or gets stuck in a review loop because the interface between the two sides was not designed.

The claim of this chapter: the hand-off is *designed*, not improvised. A designed hand-off has a written contract (what the ML team gives the specialist, what the specialist gives back, on what cadence, against what acceptance criteria), a shared vocabulary that bridges the two teams' altitudes, and a review discipline that catches drift early. This chapter authors the two hand-off contracts — one for training-pipeline-engineers, one for model-evaluation-engineers — that a senior ML engineer's team maintains.

## §1 — Why the hand-off is the hard part

Two failure modes are common enough at the ML-team ↔ specialist-team boundary that they are worth naming:

- **The "throw it over the wall" hand-off.** The ML team sends a stack trace, a slow training run, or a "eval numbers look weird" ticket. The specialist team asks questions, gets partial answers over days, finally has enough context to diagnose. Elapsed time: weeks; specialist calendar time: 60 % context-recovery, 40 % actual work.
- **The "solve the wrong problem" hand-off.** The ML team over-specifies the fix ("please add speculative decoding to our LLM serving path") and the specialist implements it. Weeks later, the underlying problem (a naïve batching config wasting 40 % of GPU time) is un-addressed. The specialist's depth was consumed on the wrong lever.

Both are hand-off design failures, not specialist-team failures. A hand-off *contract* prevents them by naming what context is transferred at what altitude and what the acceptance criterion is before the specialist starts work.

The load-bearing external reference is Michael Nygard's *[Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)* — the ADR format is a hand-off document, meant to be read by an audience the author does not directly work with. Team Topologies' [chapter on interaction modes](https://teamtopologies.com/key-concepts) names *facilitating* and *collaborating* as the two interaction modes that fit specialist-team engagements, and specifies that both require a bounded time window and an explicit expected end-state.

## §2 — The shape of a hand-off contract

The hand-off contract is a short document — one to two pages — that the ML team's TL and the specialist team's TL sign before work begins. Every contract has the same six sections:

- **Scope.** What problem is being handed off, in the ML team's vocabulary. What is *in* scope; what is deliberately *out* of scope. The scope statement is short (two paragraphs); the out-of-scope list is what prevents scope creep on the specialist side.
- **Context artefacts.** The specific documents, dashboards, code links, telemetry queries, and prior tickets the specialist needs to read to be productive. Not "our repo" — the specific files. This is where the "throw it over the wall" failure is prevented; the ML team does the context-package assembly work once, not the specialist doing it repeatedly.
- **The interface.** The specific artefact the specialist produces at completion. A PR against a specific branch? A design doc reviewed by both teams? A configuration change to a specific file? An updated eval harness? Named at contract time, not "we'll figure it out."
- **The acceptance criteria.** The bar for "done." Measurable — a specific number on a specific metric, or a specific test passing, or a specific milestone met. Not "the specialist thinks it's good enough."
- **The cadence.** How often the two sides sync during the engagement. Weekly is typical for a multi-week engagement; daily standup-adjacent for tight-timeline work. The cadence includes the *format* — a running doc, a Slack channel, a standing meeting — and the *escalation path* if the cadence is missed.
- **The end-state and hand-back.** What the ML team owns *after* the specialist is done. The specialist is *not* on-call for the delivered work permanently; the hand-back is the moment the ML team takes ownership. Named artefacts at hand-back: docs, runbooks, tests, benchmark baselines.

The six sections are *always* there. Different contracts fill them in with different specifics; the invariant is that all six are answered before work begins.

## §3 — The training-pipeline-engineer hand-off

The training-pipeline-engineer specialty owns the depth mod-303 declared as *awareness-only* for the ML engineering ladder: distributed-training primitives (DDP, FSDP, ZeRO-1/2/3, tensor / pipeline / sequence parallelism), collective-communication tuning (NCCL topology, all-reduce vs. all-gather choice), checkpoint discipline (async, sharded, verified restore), throughput / MFU measurement, mixed-precision numerics (fp16 / bf16 / fp8 loss scaling and stability), and kernel-level performance for the fastest 5–10 % of models. Reference material: PyTorch's [FSDP documentation](https://pytorch.org/docs/stable/fsdp.html), NVIDIA's [collective-communication guide](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/), Microsoft's [DeepSpeed / ZeRO papers](https://arxiv.org/abs/1910.02054), and the mod-303 chapters that already handed off to this track for depth.

Common hand-off triggers from the ML-team side:

- **Training is slow.** The team's model trains, but MFU is 15 % and the run is blocking downstream deadlines.
- **Training does not converge at scale.** The single-node numerics are stable, the multi-node numerics diverge.
- **Checkpoints are unreliable.** Restarts fail, checkpoints corrupt, mid-run failures cost days.
- **New parallelism regime.** The model outgrew data-parallelism; the team needs to introduce tensor or pipeline parallelism and does not have the depth in-house.
- **New numerical regime.** Moving from fp16 to bf16 or fp8, moving from FSDP to a new sharding shape.

### 3.1 — The training-pipeline-engineer contract skeleton

```markdown
# Hand-off contract: <specific training problem>

## Scope
- **In scope:** <e.g., "reduce end-to-end wall-clock training time of the ranking-v4 candidate from 72 hours to under 24 hours at current data volume, on the current 128-A100 quota">
- **Out of scope:** <e.g., "model architecture changes; changes to the ranking model's loss; changes to the training data pipeline upstream of the training job">

## Context artefacts
- Model repository: <link + commit SHA>
- Current training config: <link>
- Baseline MFU / throughput dashboard: <link>
- Historical training-time telemetry: <query>
- Prior attempts at speed-up: <linked tickets / PRs>
- On-call runbook for the current pipeline: <link>
- Related architecture doc: <link — mod-302 pattern>

## Interface
- **Deliverable:** a PR against `<repo>@<branch>` implementing the training-pipeline changes, plus:
  - a benchmark table (before / after) covering wall-clock time, MFU, per-GPU memory, checkpoint size, and convergence-parity check.
  - an ADR documenting the parallelism / checkpointing / numerics choices.
  - a runbook update for the new operational shape.

## Acceptance criteria
- Wall-clock training time ≤ 24 hours on 128 A100 quota. **Measured**, not projected.
- MFU ≥ 40 % end-to-end.
- Convergence parity: eval-loss curve within stated tolerance of the pre-change baseline on the fixed eval slice.
- Checkpoint restart tested at least twice under simulated failure.
- No regression on downstream serving path (the model artefact loads and serves unchanged).

## Cadence
- Weekly 30-min sync (Wednesday 15:00 UTC).
- Running doc: <link>.
- Escalation: if a weekly sync is missed twice consecutively without notice, escalate to <ML team TL> and <training-pipeline-engineer team TL>.

## End-state and hand-back
- ML team owns the training job after hand-back.
- Training-pipeline engineer is on-call for the *change* for 2 weeks after ship; then on-call transfers to ML team.
- Hand-back artefacts: PR merged, ADR merged, runbook merged, benchmark saved as regression baseline for future retrains.
- Regression: if MFU degrades > 20 % on a subsequent retrain, ML team pings the same specialist for a lightweight diagnostic (not a re-engagement).
```

Three properties of this contract worth naming:

- **The acceptance criterion is *measured*, not projected.** "Should be roughly 3× faster" is not acceptance; "wall-clock ≤ 24 hours, measured" is. The specialist is on the hook for the measurement, not for the intent.
- **Convergence parity is a first-class criterion.** A parallelism or numerics change that speeds training up 3× at the cost of a silent 2-point AUC regression on the eval slice is *not* a successful hand-off. The mod-303 eval-slice discipline is what defines "parity"; mod-305 chapter 01 is where the harness lives.
- **On-call transfer is explicit.** The specialist is on-call for the change during the burn-in window (2 weeks is a common default; adjust to the change's blast radius) and then hands the on-call to the ML team. Without an explicit transfer, the specialist is on-call forever by default — which no specialist team can sustain across the fleet of ML teams they support.

### 3.2 — The most common training-pipeline hand-off mistakes

Three that recur:

- **Over-specifying the fix.** "Please add speculative decoding to our serving path" — when the underlying problem may be a batching config, not a decoding technique. The ML team's role is to name the *problem* (training wall-clock, MFU, memory ceiling); the specialist's role is to pick the *lever*. If the ML team names the lever, the specialist's depth is wasted.
- **Skipping the baseline.** Without a signed pre-change baseline (MFU, wall-clock, eval-loss curve), the "after" numbers are unverifiable. The ML team's context-package includes the baseline; if the baseline is missing, the specialist re-generates it and the hand-off is 2× longer.
- **Missing the convergence-parity check.** A speed-up that quietly degrades quality is a regression the ML team will discover in production. The convergence-parity check is the specialist's insurance that the speed-up is honest.

## §4 — The model-evaluation-engineer hand-off

The model-evaluation-engineer specialty owns the depth mod-305 hands off for: LLM-judge harness engineering (rubric authoring, judge-vs-judge agreement, judge calibration), agentic-eval harness engineering (multi-turn tool-use scoring), evaluation-platform infrastructure (offline harness at fleet scale, slice-machinery, adversarial-suite management), calibrated scoring across slice matrices where the number of slices is large. Reference material: Anthropic's [*A statistical approach to model evaluations*](https://www.anthropic.com/research/statistical-approach-to-model-evaluations), Amanda Askell et al.'s [*Constitutional AI*](https://arxiv.org/abs/2212.08073) work on LLM-judge harnesses, and the OpenAI Evals framework as an open-source instance.

Common hand-off triggers from the ML-team side:

- **New evaluation surface.** The team is adding a feature (an LLM-augmented capability, an agent, a new safety classifier) that needs an eval harness the mod-305 primitives don't yet cover.
- **Judge harness needs re-calibration.** The team's LLM-judge scores drifted; the judge and the human reviewers disagree more than they used to.
- **New adversarial suite.** A red-team pass or an incident revealed a class of failures not covered by the current adversarial suite.
- **Eval throughput ceiling.** The harness runs slowly enough that the team's canary decision is delayed; the eval-platform team can shard, cache, or parallelise at the platform altitude.
- **Slice-matrix explosion.** The team's slice matrix (from mod-305 chapter 02) grew past what the current harness can compute without spraying alerts.

### 4.1 — The model-evaluation-engineer contract skeleton

```markdown
# Hand-off contract: <specific evaluation problem>

## Scope
- **In scope:** <e.g., "author and ship an LLM-judge harness for the ticket-triage classifier, with rubrics for routing correctness and answer-completeness, agreement-with-human ≥ 0.85">
- **Out of scope:** <e.g., "the ticket-triage model itself; the deployment path; the SLO document">

## Context artefacts
- Feature spec: <link — mod-305 chapter 04 release memo, or its predecessor>
- Current eval harness code: <link + commit SHA>
- Human-labelled reference dataset: <link + row count + slice coverage>
- Historical scoring runs: <query or dashboard>
- Related judge harness(es) already in production: <link — for consistency>
- Prior tickets on this surface: <linked>

## Interface
- **Deliverable:** a PR against `<eval-harness repo>@<branch>` adding the new harness, plus:
  - a judge-rubric document (mod-305 chapter 04 pattern).
  - a judge-vs-human agreement report on the reference dataset, per-slice.
  - a judge-vs-judge agreement report (if a second judge is used).
  - the harness registered in the model-registry adjacency so promotion gates read from it.

## Acceptance criteria
- Judge-human agreement on the reference dataset ≥ 0.85 (Cohen's κ or equivalent).
- Per-slice judge-human agreement no less than 0.10 below the aggregate, for every business-critical slice.
- Judge inference cost per prediction ≤ <$X> at fleet scale.
- Harness runs to completion on the full offline eval set within <Y hours>.
- The harness is added to the promotion gate and gates a subsequent staging or canary decision as intended.

## Cadence
- Weekly 30-min sync.
- Running doc: <link>.
- Escalation: same shape as §3.1.

## End-state and hand-back
- ML team owns the harness after hand-back.
- Model-evaluation engineer is on-call for the *harness* for 2 weeks post-ship (judge-drift diagnostics, cost-runaway diagnostics); then hands to ML team.
- Hand-back artefacts: PR merged, rubric merged, agreement report saved as regression baseline, runbook merged.
- Regression: if judge-human agreement drops below 0.80 in a subsequent monthly re-check, ML team pings the same specialist.
```

Three properties of this contract worth naming:

- **The agreement metric is measurable and pre-registered.** The specialist is not delivering "a judge harness" — they are delivering a judge harness whose *agreement* with humans passes a specific bar. Without the bar, the harness could ship with poor calibration and pass silently.
- **The cost line is explicit.** LLM judges have unbounded upside cost; mod-304 chapter 04's cost / latency guardrails apply here. The contract includes the cost budget so the specialist's design accounts for it, not so the ML team discovers the cost after the fact.
- **The promotion-gate integration is part of the deliverable.** A harness that produces numbers but is not wired into the mod-305 promotion path does not actually change the team's decisions. The wiring is the deliverable, not "nice to have."

### 4.2 — The most common eval hand-off mistakes

Three that recur:

- **Missing the reference dataset.** The specialist cannot calibrate a judge without a human-labelled reference. If the ML team hands off the problem without the reference, the specialist has to first commission (or scrape together) a reference and the hand-off doubles in length. mod-305 chapter 01 §4 authored the discipline; the reference dataset is part of the context-package, not part of the specialist's work.
- **No per-slice agreement.** A judge whose aggregate agreement is 0.90 but whose logged-out-user slice agreement is 0.55 is a bad harness for the team's slice matrix. Per-slice agreement is a non-negotiable acceptance criterion; without it, the harness ships plausibly and fails invisibly.
- **Ship-and-abandon.** The harness ships and no one re-calibrates it. Six months later, the LLM judge model has drifted (the platform team upgraded the base LLM), and the harness's agreement collapsed without anyone noticing. The hand-back's *regression* clause is what prevents this — a monthly re-check on the reference dataset, with a threshold that triggers a lightweight re-engagement.

## §5 — Cross-team vocabulary: the "altitude" translation

Both hand-offs share a common failure mode: the ML team and the specialist team think in different vocabularies, and the words that mean specific things in one team are ambiguous in the other. Three examples:

- **"Throughput."** For the ML team it usually means predictions-per-second on the serving path. For the training-pipeline engineer it means tokens-or-samples-per-second-per-GPU during training. When a hand-off doc says "we need to improve throughput," both interpretations are valid and neither is disambiguated. The senior discipline is to *always* qualify the noun: "training throughput in tokens/sec/GPU" or "serving throughput in predictions/sec/replica."
- **"Eval."** For the ML team it usually means the offline harness that scores a candidate. For the model-evaluation engineer it can mean the harness, the framework the harness runs on, the judge harness, or the agentic-eval harness. Qualify: "the offline classification harness on the ticket-triage v3 candidate."
- **"Latency."** For the ML team it usually means per-request p99 including feature-fetch. For the training-pipeline engineer it can mean per-step wall-clock during training. For the eval engineer it can mean per-scoring-run duration. Qualify: "serving p99 including feature-fetch."

The contract skeletons in §3 and §4 use the qualified vocabulary. The senior ML engineer's job in the hand-off is to *translate* their team's shorthand into the specialist's precise vocabulary before the specialist has to guess.

## §6 — When to *not* hand off

A hand-off has a cost — context-transfer, cadence overhead, review latency. Some problems are cheaper for the ML team to absorb than to hand off. Three tests:

- **The problem is one-off and small.** A one-time training-config tuning that saves 10 % is often faster to attempt in-team than to hand off; a hand-off would take longer than the fix.
- **The team has partial depth.** A team with an engineer who has some GPU-perf experience can absorb the small-and-medium levers without a specialist engagement. The specialist is reserved for what requires their depth.
- **The problem is going away.** A pipeline shape that will be deprecated next quarter does not benefit from a specialist engagement to speed it up. Bend (chapter 02 §1) or wait.

The senior discipline is to *not* hand off when the ML team can absorb it and *do* hand off when specialist depth is the load-bearing lever. Over-handing-off wastes specialist capacity the org needs for the load-bearing problems; under-handing-off leaves the team stuck on levers it cannot pull.

## §7 — Program-level: the hand-off portfolio

At the individual-engagement altitude, this chapter is about writing a good contract. At the *program* altitude — the senior ML engineer looking at the team's specialist engagements over a year — the discipline is a portfolio view:

- **How many specialist engagements did the team run this year?** For each, the outcome (delivered, delivered late, cancelled, in flight).
- **What was the average time-to-value?** From contract-signed to hand-back. A trend of increasing time-to-value is a signal the specialist tracks are over-subscribed or the ML team's context-packages are degrading.
- **What was the on-call regression rate?** How often did a delivered engagement's regression clause fire (the harness lost calibration, the training MFU degraded)? A high regression rate is a signal the burn-in window is too short or the hand-back artefacts were incomplete.
- **Which specialty saved the most engineer-hours this year?** The team's story to leadership about the value of the specialist tracks is quantified in this metric.

The program-level view is what a senior ML engineer brings to a manager 1:1, a staff-level review, or the org's platform-investment discussion. Without it, the specialist-track relationship reads as a black box and is undervalued at investment time.

## Summary

The hand-off between a senior ML engineer's team and the specialist tracks (training-pipeline-engineer, model-evaluation-engineer) is *designed*, not improvised. A hand-off contract has six sections: scope (in and out), context artefacts, the interface (the specific deliverable), the acceptance criteria (measured, not projected), the cadence (with escalation), and the end-state and hand-back (with on-call transfer and regression clause). The training-pipeline hand-off (§3) targets training-wall-clock / MFU / numerics / checkpoint / parallelism problems with parity-preserving acceptance criteria and an on-call burn-in. The model-evaluation hand-off (§4) targets harness-authoring, judge-calibration, adversarial-suite, and eval-throughput problems with agreement-vs-human as the load-bearing acceptance criterion and cost budgets baked in. Cross-team vocabulary (§5) is a common failure mode; the ML engineer's job is to translate their team's shorthand into the specialist's precise vocabulary in the contract. Not every problem warrants a hand-off (§6); over-handing-off wastes specialist capacity, under-handing-off leaves the team stuck. At program altitude (§7) the portfolio view — engagements per year, time-to-value, regression rate, engineer-hours saved — is what makes the specialist relationship visible to leadership and defensible at investment time.
