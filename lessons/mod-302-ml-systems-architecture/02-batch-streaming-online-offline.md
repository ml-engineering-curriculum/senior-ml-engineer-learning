# Batch, streaming, online — and the offline/online feature axis

## Motivation

"Batch vs. streaming" is the most common question a senior ML engineer answers in an architecture review, and the one juniors most often answer wrong. The wrong answer is expensive in both directions: pick batch when you needed online and the launch dies at go-live; pick online when the business would tolerate a nightly job and you spend the next two years paying to run infrastructure you did not need.

This chapter gives you the decision rubric, names the two axes people conflate (serving posture vs. feature freshness), and shows the failure modes that follow when you get either one wrong.

## The two axes people conflate

Everyone talks about "batch" and "streaming" as if there were one axis. There are two — and reasoning about them separately is where the L30 clarity comes from.

### Axis 1 — Serving posture: how predictions reach the consumer

- **Batch prediction.** A scheduled job (Airflow, Dagster, cron) scores every relevant entity on a cadence and writes predictions to a durable store (a warehouse table, a Redis lookup, a document index). Consumers read the precomputed prediction. Latency at read time is dominated by the store lookup, not the model.
- **Online (request-time) prediction.** A model server is called synchronously in the request path — a REST/gRPC call, an in-process invocation — and returns a prediction against the current request's features. Latency at read time is dominated by the model call.
- **Streaming prediction.** Continuous scoring against arriving events (Kafka, Kinesis, Flink, Beam). A prediction is produced per event, often written back to an online store or emitted to a downstream stream. Latency is the event-to-prediction delay.

A single system can use more than one. A common shape: streaming updates aggregated features into an online store, an online model server reads those features at request time, and a batch job produces daily reports off the same predictions.

### Axis 2 — Feature freshness: how current the features are

Independent of how predictions reach the consumer, the features the model consumes have their own freshness axis:

- **Offline features.** Computed against historical windows in the warehouse — "the user's 30-day purchase count as of last midnight." Cheap, dense, stable, but stale.
- **Online features (request-time).** Computed inline from the request payload or a low-latency lookup — "the item ID in this request; the user's session-cart total right now."
- **Streaming features.** Continuously updated aggregates fed by an event stream — "clicks in the last 5 minutes," "5-second EWMA of transaction amount for this merchant."

A single feature vector routinely mixes all three. The serving posture and the feature freshness are **independent choices** — you can have a batch scoring job that consumes streaming-computed features (unusual), or an online prediction path that consumes only offline features (very common in v1 systems). The two axes are two knobs.

## The freshness–cost–complexity triangle

For every choice on both axes there is a triangle:

- **Freshness** — how current the feature or prediction is.
- **Cost** — dollars per month for compute, storage, and on-call.
- **Complexity** — number of moving parts, failure modes, on-call surface.

You can improve two at the expense of the third. You cannot improve all three. The senior job is to make the trade-off *deliberate* and *defensible*, not to pretend it does not exist.

Rough orders of magnitude — treat as directional, not benchmark:

| Choice | Freshness | Cost | Complexity |
|---|---|---|---|
| Batch predictions off offline features | Hours to days | Low | Low |
| Online predictions off offline features | Milliseconds (prediction) / hours (features) | Medium | Medium |
| Online predictions off streaming features | Milliseconds (prediction) / seconds (features) | High | High |
| Streaming predictions on streaming features | Seconds end-to-end | Highest | Highest |

The point of this table is not that streaming is "worse" — it is that when a reviewer asks "why streaming here?" the L30 answer is a business constraint on freshness, not a technology preference.

## The decision rubric

Six questions, in order. If any one forces you to a decision, stop there — the later questions cannot override.

### Q1 — What is the consumer's freshness budget?

If a stale prediction is useful for hours (recommendation-list refresh, weekly risk report, daily churn dashboard), **batch is the default**. Any online or streaming shape here is a cost tax. Google Cloud's [*MLOps: Continuous delivery and automation pipelines in machine learning*](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) opens with this framing: match the pipeline to the freshness the business actually needs.

If a stale prediction is useless (checkout-time fraud scoring, ad ranking on impression, next-word suggestion), the answer is online. If the underlying signal itself moves in seconds (session-level anomaly detection, feed real-time re-ranking), you are in streaming.

### Q2 — Are the features available at request time, or do they require historical aggregation?

Some features only make sense computed over a window (e.g., "clicks in the last 24 h", "median transaction size for this merchant over 30 days"). Those cannot be computed inside the request path — you have to precompute and cache. That does not force streaming; it forces you to precompute somewhere. If daily precompute is fresh enough, batch precompute suffices.

Other features are trivially available at request time — the item ID, the request payload, a look-up on the current user's plan. Those can be online-only.

If your feature list mixes both (it almost always does), you land on a hybrid: aggregates precomputed batch or streaming into a store, request-time features read from the request, both concatenated at serving time.

### Q3 — What is the prediction volume and traffic shape?

Batch prediction shines when you predict for a large, stable population on a cadence — nightly scoring for 200 M users. It is a poor fit when only 0.1% of the population is ever queried, and the query pattern is unpredictable — you spend compute predicting for the 99.9% you never look up.

Online prediction inverts this: you pay per request, but never pay for entities never queried. The break-even is roughly at the fraction of the population queried in a fraction of the batch cadence. Batch beats online when that fraction is high; online beats batch when it is low.

Streaming's cost model is dominated by throughput and state size (the amount of feature state you have to keep hot per key). Kafka's own [*Kafka Streams* documentation](https://kafka.apache.org/documentation/streams/) is the canonical reference for the state-management trade-offs.

### Q4 — What is the latency SLO in the request path?

If the model call sits in a p99 <100 ms budget, online serving is a constrained problem — you have to pick a model whose inference fits inside the budget minus your feature-fetch time minus your network and serialisation overhead. If the budget is p99 <10 ms, you are almost certainly precomputing predictions (i.e., batch predictions cached in an online store), not doing online inference at all. If the budget is p99 <1 s, most reasonable models fit.

If there is no user in the request path (queue consumer, offline pipeline), there is no latency SLO — batch or streaming is fine.

### Q5 — What is the operability envelope?

Streaming adds a category of failure that batch does not have: **event-time correctness** (late-arriving events, out-of-order events, watermarks). The Google [*Dataflow Model* paper (Akidau et al., VLDB 2015)](https://research.google/pubs/pub43864/) and Tyler Akidau's [*The Dataflow Model / Streaming 101 & 102*](https://www.oreilly.com/radar/the-world-beyond-batch-streaming-101/) essays are the canonical treatment of what breaks and how you reason about it.

If your team is not staffed to own Kafka/Flink/Beam on-call — including partitioning, backpressure, checkpointing, replay — do not choose streaming because "it's more modern." Choose it because the freshness in Q1 forces you to. The `ai-infra-mlops-learning` peer platform track (see [mod-308]) is the place to escalate if you need streaming but do not have the platform.

### Q6 — Is there a cheaper posture that still meets Q1–Q5?

The default L30 posture on a new system is: **the least sophisticated shape that meets the freshness and latency SLO**. Not because sophistication is bad, but because every axis you add is a permanent tax on the team.

- If daily is fresh enough, do not build hourly.
- If pre-computed batch scores fit an online store within the latency budget, prefer that to synchronous online inference.
- If a materialised view fed by a scheduled query fits, prefer that to Flink.

Chapter 06 (the RFC) requires you to name the alternatives you rejected. Q6 is where that list comes from.

## Worked example — the same "personalised recommendations" problem, three shapes

Consider the same business ask — "personalised product recommendations on the home page for logged-in users" — under three different constraints:

**Constraint A — freshness measured in days.** The homepage refreshes recommendations every 24 hours; users tolerate a day-old list; volume ~10 M active users/day. Rubric answers: Q1 says days → batch. Q2 says features are 30-day aggregates → precomputed. Q3 says 10 M actives out of 100 M users → batch coverage is fine. Q4 says the read is a fast key lookup, not a model call → online store hit. → **Batch predictions off offline features, precomputed nightly into a key-value store served at request time.**

**Constraint B — freshness measured in the current session.** A user's clicks in the last 5 minutes should influence the next list; homepage revisit is common inside a session; volume similar. Rubric answers: Q1 says minutes → not batch. Q2 says session aggregates are needed → cannot precompute nightly; needs a fast-moving feature. Q4 says p99 <300 ms → online inference is possible. → **Online prediction with hybrid features: 30-day aggregates precomputed batch into an online store, session-level features streamed from a Kafka topic into the same store, both read at request time.**

**Constraint C — freshness measured in seconds, and the ranker itself is the reactive signal.** A live-events product where the ranker must react to a burst of clicks on a new SKU inside 10 seconds. Rubric answers: Q1 says seconds; Q4 says request-path budget is <200 ms; Q5 says the platform must own Flink/Beam. → **Streaming prediction: Flink job consumes click events, updates SKU-level features, streams updated scores to the ranker's cache, ranker reads from cache with millisecond latency.**

Same product ask, three architectures. The rubric is what makes the choice defensible.

## Two failure modes the rubric catches

### The "streaming because Kafka is cool" anti-pattern

An RFC lands with Kafka + Flink + a bespoke online feature store for a system whose consumer is a nightly report. The rubric answers say Q1 → days, Q3 → full-population batch, Q4 → no request path. The reviewer's job is to send the RFC back with those three answers written on it. The team can rebuild it against the real constraints and ship a batch job in two weeks instead of six months.

### The "cron script forever" anti-pattern

An RFC lands with a Python script scheduled by cron because "we don't need Kafka." Six months later the business needs session-level freshness, and the script's data model — reading yesterday's warehouse partition — cannot express it. The team either bolts on a shadow streaming pipeline (worst of both worlds) or does a from-scratch rewrite.

The rubric catches this too. Q1 in the *current* review may say "days," but Q6 should ask whether the freshness budget is likely to tighten inside 12 months. If it is, at minimum design the storage layer so predictions can be produced by more than one job. Do not paint yourself into a batch-only corner if the roadmap already knows it will need online.

## The offline/online axis in one paragraph

Independent of everything above, every feature the model consumes has to be reachable at **training time** (offline, historical) and **serving time** (online, current) with the same semantics. That is the offline/online feature axis and it is what chapter 03 (feature store) and chapter 04 (training/serving skew) are entirely about. The most common architectural failure in a mixed batch/streaming/online system is that offline and online paths compute "the same" feature differently. This chapter has told you when to reach for streaming; the next chapter tells you where to put the feature code so that skew does not eat you alive.

## Summary

Serving posture (batch / online / streaming) and feature freshness (offline / online / streaming) are two axes, not one. Pick each one with the six-question rubric — freshness budget, feature availability, volume shape, latency SLO, operability envelope, cheapest sufficient posture. Streaming is the answer when Q1 forces it, not when Kafka is fashionable. Batch is the answer when Q1 tolerates it, not when the team is scared of Kafka. Exercise-01 is a real batch-vs-streaming decision writeup on a real business problem; chapter 03 picks up the offline/online feature axis.
