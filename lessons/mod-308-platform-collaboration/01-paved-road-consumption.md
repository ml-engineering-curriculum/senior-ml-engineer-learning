# Consuming the paved road: feature stores, model registries, and training clusters

## Motivation

At L20 the platform is a set of *tools you had to install* — a feature store because someone told you to, a model registry because the deploy script won't run without it, a training cluster because the notebook died on your laptop. At L30 the platform is a *contract you are a party to*. You know what it guarantees, what it explicitly does not, what its failure modes are, who is on-call for each surface, and — the load-bearing part — what feedback the platform team needs from you in order to keep improving the road you drive on.

The phrase "paved road" comes from Netflix ("[the paved road is the path we recommend, we support, and we make easiest](https://netflixtechblog.com/full-cycle-developers-at-netflix-a08c31f83249)") and has become the standard vocabulary for an internal platform team's *supported* offering. Camille Fournier's *[Building a Platform Team](https://skamille.medium.com/building-a-platform-team-e08ea8b23a68)* and the [Team Topologies](https://teamtopologies.com/book) framing of a *platform team* as one that reduces the cognitive load of *stream-aligned teams* are the two references worth reading before this chapter.

This chapter is about the *consumer's* side of that relationship — how a senior ML engineer consumes the paved road *idiomatically* and turns their consumption into feedback the platform team can use. Chapter 02 is about when to *leave* the paved road. Chapter 03 is about how to *pave a new lane* on it. Chapter 04 is about the specialist tracks that hand off *underneath* it.

The claim of this chapter: the ML engineer's altitude is not "know how the feature store is implemented," it is "consume the feature store idiomatically, notice when the road is missing a lane, and give the platform team the feedback that gets the lane paved." The rest of the module builds on this altitude.

## §1 — What "the paved road" actually is

The paved road is the set of platform primitives that a stream-aligned team is *expected* to use, that come with support, on-call, migration paths, and a roadmap. It is contrasted with:

- The **unpaved road** — things you can technically do, but at your own risk, without on-call, migration, or support. Notebook-only training, ad-hoc S3 buckets for model artefacts, hand-rolled inference containers behind an internal ALB. If the unpaved road breaks, you own it.
- The **new road under construction** — a beta or preview offering, opt-in, with partial support. The platform team wants users but explicitly does not commit to backwards-compatibility.

The paved road for a mature ML platform typically includes:

- A **feature store** (offline + online) with a declared schema, versioning, freshness SLOs, and a training-serving-consistency guarantee. Uber's [Michelangelo](https://www.uber.com/blog/michelangelo-machine-learning-platform/), Airbnb's [Zipline / Chronon](https://airbnb.io/projects/chronon/), and open-source [Feast](https://docs.feast.dev/) are the reference implementations to skim.
- A **model registry** with model artefacts, lineage, evaluation metadata, and a lifecycle state machine (Staging → Production → Archived). MLflow's [Model Registry](https://mlflow.org/docs/latest/model-registry.html) is the reference open-source implementation; SageMaker's [Model Registry](https://docs.aws.amazon.com/sagemaker/latest/dg/model-registry.html) and Vertex AI's [Model Registry](https://cloud.google.com/vertex-ai/docs/model-registry/introduction) are the cloud equivalents.
- A **training platform** with cluster provisioning, distributed-training primitives, checkpoint management, and quota. Ray, Kubeflow Training Operator, or a hosted equivalent (SageMaker Training, Vertex AI Training). See Ray's [training documentation](https://docs.ray.io/en/latest/train/train.html) and the Kubeflow Training Operator [reference](https://www.kubeflow.org/docs/components/training/).
- A **serving platform** with autoscaling, canary deploys, model rollout controls, and observability. KServe, Seldon, Triton, or a hosted equivalent (SageMaker Endpoints, Vertex AI Endpoints).
- A **workflow orchestrator** — Airflow, Prefect, Dagster, Kubeflow Pipelines, Flyte, or a hosted equivalent — for DAG-shaped training / evaluation / promotion pipelines.
- An **experimentation platform** — the A/B and canary layer. mod-306 depth on the experimentation-platform side; here it is a paved-road primitive to consume.
- An **eval harness** — the offline evaluation infrastructure. mod-305 depth on the eval side; here it is a primitive.
- **Cost and observability** — the metrics, dashboards, tracing, and cost-attribution surface. mod-307 depth on the SLO side; here it is a primitive.

The paved road is *not* the individual tools — it is the *integration* between them, the deploy pipeline that carries a model from `git push` through evaluation, registry, canary, and production without asking the ML engineer to hand-wire anything. The value of the paved road is *not saving you the code* — it is *saving you the on-call, migration, and evolution* work that would otherwise sit on your team forever.

## §2 — Why the paved road exists (the platform team's job)

The reason the paved road exists is not "the platform team enjoys building it." It exists because *someone* has to answer the following questions across an org, and the alternative is every ML team answering them independently and inconsistently:

- **How do we get training-serving consistency?** If the offline features are computed from Spark on the warehouse and the online features are computed from Flink on Kafka, drift is inevitable unless the transformation is declared once and executed by both. The feature store's job is to be that declaration and enforcement point. Uber's Michelangelo paper is explicit that this was the *core* motivation for the platform.
- **How do we know what is deployed?** If every ML team's model artefacts live in per-team S3 buckets, there is no way to answer "what model is in production right now" without asking every team. The model registry's job is to be the org-wide answer to that question.
- **How do we roll something back?** If the deploy path is per-team, so is the rollback path. The platform's job is to make rollback a *one-action property* (mod-307 §5 §4) at org scale.
- **How do we not run out of GPUs?** GPU allocation is expensive; unattended jobs and lost checkpoints burn quota. The training platform's job is to be the quota-enforcement, priority-queue, and checkpoint-recovery layer no individual team has the incentive to build.
- **How do we compare offerings?** If team A's ranking model and team B's ranking model report metrics against different eval slices, no one can decide where to invest. The eval harness's job is to make the metrics comparable.
- **How do we survive re-orgs?** Teams re-form. If the tooling is per-team, re-orgs orphan systems. The platform's tooling survives the re-org because it is not owned by the team that consumes it.

Naming these motivations out loud matters because it tells you what *feedback* the platform team can actually act on. Feedback of the form "the feature store is annoying to use" is un-actionable. Feedback of the form "the online feature-fetch p99 breaches our per-request latency budget for feature F for 4 % of requests, and the fallback path we would have to hand-roll costs us 20 hours of engineering per team-quarter" is actionable — it names an SLO gap and quantifies the cost of *not* fixing it. That form of feedback is the load-bearing artefact of this module.

## §3 — Idiomatic consumption: the four disciplines

An ML engineer's team can consume a platform primitive in the *idiomatic* way or in *any of a hundred non-idiomatic ways*. The non-idiomatic ways all superficially work. They just make you a bad citizen — a heavy support burden, a migration blocker, a source of "we can't change the platform because we'd break this team." The four disciplines that keep a team idiomatic:

### 3.1 — Use the declared API, not the implementation

Every platform primitive has a *declared* consumer API (the SDK, the CLI, the YAML manifest schema) and an *implementation* underneath (the specific Kubernetes controller, the specific S3 bucket layout, the specific database table). The idiomatic consumer uses the declared API only. The non-idiomatic consumer reaches around the API — `boto3.client('s3').get_object` against the model artefact bucket instead of `registry.load('model-name', 'production')`.

The consequence of reaching around the API is that when the platform team migrates the implementation (from S3 to GCS, from bare Postgres to a managed model-metadata service, from one feature-store SDK version to the next), your team is the migration blocker. Everyone else moved because they used the API; you did not because you reached around it. The platform team is either forced to preserve the underlying implementation for you (bad for them) or force you to migrate on emergency timelines (bad for you). Neither outcome is a good relationship.

The senior discipline is to *ask* when the declared API is missing what you need instead of reaching around it. That "ask" is the feedback loop of §5; sometimes it becomes the contribute-back RFC of chapter 03.

### 3.2 — Version everything the platform lets you version

Every mature platform primitive supports versioning of the things whose change is a correctness issue: the feature-view schema, the model artefact, the prompt template, the eval slice definition, the pipeline DAG. Idiomatic consumption pins the version at *read time* and *promotes* the version explicitly. Non-idiomatic consumption reads "the latest" without pinning, or worse, mutates the artefact in place.

The visible failure mode of the non-idiomatic pattern is that a rollback is impossible — the "previous version" the runtime pointed at got overwritten. The invisible failure mode is that reproducibility is impossible — the training run's inputs cannot be reconstructed six weeks later because the feature-view schema silently changed underneath. mod-302 chapter 05 (training/serving skew) and mod-307 chapter 05 (retraining as deploy) both leaned on the versioning discipline; here it is a *consumption* discipline, not just a modelling one.

### 3.3 — Emit the standard telemetry, register the standard artefacts

Every mature platform primitive expects you to emit a small set of standard telemetry — the feature-fetch latency spans, the prediction-request tags, the model-load status, the pipeline stage timings — and to register a small set of standard artefacts — the model, the eval report, the release memo, the runbook link, the SLO document (mod-307 chapter 01 §6). The idiomatic consumer emits and registers *all of them* even for the ones they do not personally use.

The reason to emit them anyway is that the *aggregate* of the org's telemetry and artefacts is what makes the platform's own SLOs and roadmap possible. If half the teams do not emit the standard tracing spans, the platform team cannot see which feature-store operations dominate their tail latency and cannot prioritise the right optimisation. If half the teams do not register the standard release memo, the incident-review process cannot correlate an outage against recent deploys.

The senior discipline is to treat "emit the standard telemetry" and "register the standard artefacts" as part of *shipping*, not as optional hygiene. Chapter 01 §6 of mod-307 authored the SLO document; the platform expects it in a specific place with specific frontmatter so its own dashboards can find it.

### 3.4 — Land your dependencies on top of the primitive, not underneath

When your team builds ML tooling *on top of* a platform primitive — a per-team eval wrapper, a per-team retraining scheduler, a per-team feature-view generator — the idiomatic pattern is for your wrapper to *delegate* to the primitive, not to *replace* it. Your wrapper adds team-specific opinions on top; the primitive still owns the API, the versioning, the telemetry, the on-call.

The non-idiomatic pattern is a per-team wrapper that either bypasses the primitive for "performance reasons" or that shadows the primitive's API surface enough that the platform team's SDK changes silently break the wrapper. The former makes you the migration blocker; the latter makes every platform release a per-team firefight.

Sam Newman's [*Building Microservices*](https://samnewman.io/books/building_microservices_2nd_edition/) has a chapter on the "shared library" problem that is directly analogous — a shared library that reaches into internal state, rather than consuming a stable API, is a coupling point that is invisible until it breaks. The senior discipline is to structure the team's platform-adjacent code so a platform-team SDK bump is a *dependency version bump* on your side, not a re-architecture.

## §4 — The consumer's contract: SLOs and support

The paved road comes with an SLO. The senior ML engineer knows what the SLO *is* — for each primitive the team depends on — and treats the number as a real number, not a "the platform is up" reassurance.

The three questions to answer for every paved-road primitive the team consumes:

- **What is the SLO?** Availability, latency, freshness, cost — expressed as SLI + target + window, in the vocabulary of mod-307 chapter 01. If the platform team has not published the SLO, that is a feedback item (§5). If the SLO is 99 % availability over 28 days, your team's SLO for a service that *depends* on this primitive cannot be tighter than the composition (mod-307 chapter 01 §5 dependency SLIs).
- **What is the escalation path?** Which Slack channel or ticket queue, what response-time expectation, what after-hours pager for P0. The senior discipline is to *have used* the escalation path at least once for a non-P0 before the first P0 — so the on-call knows the channel exists and the platform team knows your team by name.
- **What is the deprecation policy?** When the platform team deprecates the version of the SDK you are on, what is the notice period, the migration guide, the sunset date? [Google's API deprecation policy](https://cloud.google.com/terms/deprecation) is the industry-standard reference for what a *published* deprecation policy looks like; internal platforms typically follow a lighter version of the same discipline.

The senior discipline is to treat these three answers as *part of the platform primitive*, not as "nice to have." If the answers are missing, that is a *maturity gap* in the platform, not "well, they haven't gotten around to it." Feedback on missing SLOs and missing deprecation policies is often *more* useful to the platform team than feedback on the API shape, because the missing SLO is a symptom the platform team is under-invested in the reliability layer, and the ML team's feedback is often the trigger that unblocks that investment.

## §5 — The consumer's other job: feedback that improves the road

The load-bearing shift from L20 to L30 is that a consumer's job is not only *to consume* — it is also *to feed back*. The platform team cannot see what its consumers see. The consumers must tell them, in a form the platform team can act on. Three forms of feedback, in ascending order of formality:

### 5.1 — Bug reports and pain points

The lightest form: a Slack message or a ticket in the platform team's queue. Idiomatic form:

- **Concrete reproduction.** The exact SDK call, the exact input, the exact observed vs. expected behaviour. Not "the feature store is broken."
- **Frequency and blast radius.** Is this affecting one prediction, one team, one region, or the fleet? Is it a P0 (production degradation) or a P3 (annoying dev-loop paper cut)?
- **The workaround you tried.** So the platform team knows what already did not work and can skip suggesting it.

The senior discipline: file the ticket even for small-blast-radius issues. The platform team's prioritisation depends on ticket count; ten teams each silently working around the same paper cut looks like "nobody cares about that" to the platform team, when it is actually "everybody quietly hates that."

### 5.2 — Aggregated pain: the quarterly consumer report

A ticket per bug is high-noise for the platform team's roadmap. A quarterly report from a *lead consumer team* — the one this module trains you to be — is high signal.

Idiomatic form:

- **Two to five load-bearing pain points.** Each named as an SLO gap, a workflow gap, or a maturity gap.
- **For each, the *cost to your team*.** In hours-per-quarter, in dollars, in blocked features, in on-call load. A pain point without a cost is a "please" ticket; a pain point with a cost is a business case.
- **For each, the *proposed direction*.** Not a full RFC (that is chapter 03), but a directional sketch — "we think a per-feature-view freshness SLO could be added to the platform SDK" is enough for the platform team to decide whether to invite a formal RFC.
- **A ranking.** Which of the pain points is most important to *your* team? The platform team will negotiate against everyone else's ranking; yours is one input, not a decree.

The senior discipline is that this report is *authored*, not extracted from Slack. It goes to the platform team's staff engineer / TL / product manager on a schedule you agreed with them, not "when you get to it."

### 5.3 — The RFC and the contribute-back PR

The heaviest form: an RFC proposing a specific change to the platform, and often a PR implementing it. Chapter 03 is entirely about this. What to note here is that the RFC does not appear from nowhere — it is the *promotion* of a quarterly-report item that both sides agreed was worth an RFC.

The precondition to a successful RFC is that the platform team already agrees the *problem* is real. If your first interaction with the platform team is "here is my RFC," you are asking them to evaluate both the problem and the solution at once. If your first interaction is a ticket, escalating to a quarterly report, escalating to "we think there is an RFC here," the RFC is the third round of a conversation both sides have been building context on.

## §6 — Anti-patterns the paved road catches

Some patterns look reasonable at the individual-team altitude and are recognisable to the platform team as recurring anti-patterns. A senior ML engineer notices themselves reaching for them and stops.

- **The per-team feature store.** "The org feature store doesn't support X, so we built our own for our team." The moment two features on the team need to share a feature-view, or two teams need to share a feature, this becomes a coordination problem the platform was built to solve. Idiomatic move: *escalate* the gap (§5) and, if urgent, land the interim on-team solution in a way that migrates cleanly (chapter 02 §5).
- **The per-team model registry.** "We use MLflow but we point at our own artefact bucket because the org registry is slow." Same failure mode; now nobody can answer "what is in production" across teams.
- **The parallel eval harness.** "The org harness doesn't do X so we scored our model with our own harness in a notebook." mod-305 chapter 01's warning about the un-registered eval notebook lives here; the harness is a platform primitive, and every team's out-of-band scores are un-comparable.
- **The direct-to-S3 model deploy.** Bypassing the deploy pipeline for "a quick fix." No canary, no rollback, no incident review; the change is invisible to the incident-response taxonomy of mod-307 chapter 04.
- **The Terraform-only serving stack.** Rolling your own KServe / Triton / GKE cluster because the platform serving stack "doesn't fit our shape." Sometimes right (chapter 02), often wrong; when wrong, becomes a permanent per-team on-call burden that no re-org survives.

The senior discipline is to notice the pattern *early* and force the conversation with the platform team before the on-team workaround becomes the on-team dependency. Chapter 02 is the framework for that conversation.

## §7 — Worked example: the paved-road inventory

The artefact that anchors this chapter is a *paved-road inventory* — a table your team maintains that names, for every platform primitive the team consumes:

```markdown
# Paved-road inventory — recommendation-serving team

| Primitive | Version | Owner | SLO | Escalation | Deprecation | Idiomatic? | Notes |
|---|---|---|---|---|---|---|---|
| Feature store (Chronon) | v3.2 | ml-platform | 99.9% avail 28d, p99 fetch < 20 ms | #ml-platform-oncall | 6-month notice | ✓ | Two custom feature-views land through the platform DSL. |
| Model registry (MLflow) | v2.14 | ml-platform | 99.95% avail 28d | #ml-platform-oncall | 6-month notice | ✓ | Registry is source-of-truth for production model version. |
| Training platform (Ray on K8s) | v2.30 | infra-platform | Best-effort; no formal SLO ✗ | #infra-platform | Unpublished ✗ | Partially | We use `ray.train` idiomatically; we set our own quota policy on top. |
| Serving (KServe) | v0.12 | ml-platform | 99.99% avail 28d, p99 predict + fetch < 80 ms | #ml-platform-oncall | 12-month notice | ✓ | Auto-scale from platform. |
| Orchestrator (Airflow) | 2.9 | data-platform | 99.5% avail 28d | #data-platform | 3-month notice | ✓ | Retraining DAG runs here nightly. |
| Experimentation platform | v1.8 | growth-platform | 99.9% avail 28d, SRM monitoring live | #growth-eng | 6-month notice | ✓ | mod-306 flow. |
| Eval harness (internal) | v0.6 | ml-platform | Best-effort; no formal SLO ✗ | #ml-platform-oncall | 6-month notice | ✓ | mod-305 flow. Team-specific slice extension merged upstream 2026-Q1. |
| Cost / observability | Grafana + platform cost service | infra-platform | Grafana 99.9%, cost service best-effort | #infra-platform | Grafana published; cost service unpublished ✗ | ✓ | mod-307 flow. Cost service missing SLO is a feedback item. |
```

Three properties of this inventory that are worth naming:

- **Every "✗" is a feedback item.** The training platform's missing SLO, its unpublished deprecation policy, the cost service's missing SLO — each is a quarterly-report line-item.
- **"Idiomatic?" is a self-audit.** The training platform is only "Partially" because the team layered a per-team quota policy on top; the entry names *why* and links to the follow-up conversation with the platform team.
- **It survives re-orgs.** When the team's manager changes, or the platform teams re-org, the inventory tells the incoming manager what the team depends on, what the SLOs are, and where the gaps are — without an archaeological dig.

Exercise 01 authors this inventory for a specific team. Chapter 02's build-vs-adopt decision uses it as an input. Chapter 03's RFC uses it as evidence.

## Summary

The paved road is the set of platform primitives a stream-aligned ML team is expected to consume — feature store, model registry, training platform, serving platform, orchestrator, experimentation platform, eval harness, observability. The paved road exists because someone has to answer the org-scale questions (training-serving consistency, what's in production, rollback, quota, comparability, re-org survival) once, not once per team. Idiomatic consumption has four disciplines: use the declared API not the implementation; version everything the platform lets you version; emit the standard telemetry and register the standard artefacts; land team-specific tooling *on top of* the primitive, not underneath. The consumer's contract with the platform includes SLOs, escalation, and deprecation policy — the senior ML engineer knows the number for each. The consumer's *other* job is feedback: bug reports, quarterly consumer reports with named costs and proposed directions, and — the topic of chapter 03 — RFCs. Anti-patterns (per-team feature store, per-team registry, parallel eval, direct-to-S3 deploys, roll-your-own serving) are the recurring failure modes the paved road exists to prevent. The paved-road inventory (§7) is the artefact this chapter's exercise authors and the rest of the module reuses.
