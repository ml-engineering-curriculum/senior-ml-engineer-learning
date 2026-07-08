# exercise-03: Training/Serving Skew — The Architectural Fix

**Estimated effort:** 3 hours

## Objective

Take a training/serving skew incident, classify it against the chapter-04 five-category taxonomy, and design the **architectural** fix — the change to the system that prevents the *class* of skew from recurring, not just the instance. The deliverable is a short remediation memo plus a completed parity checklist that the on-call team can attach to the incident post-mortem.

The exercise is deliberately about the altitude difference: L20 fixes an instance of skew ("added a missing conversion in the SQL"). L30 removes the *possibility* of a whole class of skew ("added a point-in-time-join CI test and made all training-set builds go through the primitive"). Every senior ML engineer has to be able to do this on demand.

## Prerequisites

- Read chapter 04 (`04-training-serving-skew.md`) in full — the five categories and the parity checklist are the load-bearing tools.
- Skim chapter 03 (`03-feature-store-and-model-registry.md`) so you can name whether feature-substrate quality is upstream of the fix.
- Skim chapter 05 (`05-retraining-and-rollout.md`) so you can name where the fix hooks into rollout gates.

## Pick a scenario

Pick **one**. Both bring-your-own and case-scenario options are supported. Every scenario is a real *category* of skew (not always a specific bug) so that the fix can be architectural.

### Option A — bring your own

An incident you have real access to. Anonymise as needed. Include a one-paragraph "what actually happened" writeup at the top: symptom, timeline, immediate root cause, immediate fix already applied.

### Option B — pick a case scenario

- **Scenario B1 — Rank model regressed after a schema migration.** The upstream `users` table gained a new column `country_of_residence`. The offline training pipeline picked it up automatically (SELECT *). The online serving path used a hand-written protobuf and did not know about the new column, defaulting it to empty. Users in every country now look like `""` at serving time. Offline metrics are unchanged; online CTR dropped 4%.
- **Scenario B2 — Fraud model's precision collapsed on a Wednesday.** Investigation: a feature `merchant_avg_txn_amount_30d` is computed offline against the warehouse (which has ~2 hour ingest lag) and online against a Redis feature service (which has ~30 second ingest lag). A large-merchant onboarding event on Tuesday night changed the merchant mix. Offline features show the old distribution; online features show the new one. Precision decayed as the merchant mix drifted through Wednesday.
- **Scenario B3 — Recommender's AUC was too good to be true.** Offline AUC is 0.94, online CTR is 30% below baseline. Investigation: the training set was built by joining today's feature snapshot to two-week-old labels. The features include `user_last_purchased_days_ago`, which is trivially predictive when it is computed as-of-today against a two-week-old label — the label event is now visible in the feature.
- **Scenario B4 — Session model works on desktop, fails on mobile.** The model was trained on ~90% desktop traffic (the analytics team's default filter, unnoticed). At serving time, ~60% of traffic is mobile. Mobile users have systematically shorter session times and different click patterns. The model's expected input distribution does not match the served input distribution.
- **Scenario B5 — Streaming feature model has a mysterious quality drop every night at 2 a.m.** Investigation: a batch job that materialises "yesterday's aggregates" into the feature store runs at 2 a.m. UTC and briefly evicts entries from the online cache. During the cache warm-up (~10 minutes) the model receives stale or null features. The feature histogram monitors do not catch it because the daily averages hide the 10-minute outage.

## Steps

### 1. Reconstruct the incident (≈ 30 min)

If you are using Option A, write it up. If you are using Option B, expand the scenario into a full timeline: what the symptom was, what the on-call team saw, what the immediate patch was (or would have been). Cite specific timestamps or ordering. This is what makes the fix defensible.

### 2. Classify the skew (≈ 30 min)

Walk the chapter-04 five categories:

- Feature-computation skew
- Point-in-time / label leakage skew
- Data-source skew
- Timing skew (offline pretending to be online)
- Distribution / population skew

For each, answer **yes / no / partial contributor** with one sentence citing the evidence. Skew incidents commonly involve more than one category; do not stop at the first hit. If you find yourself using more than 3 sentences per category to defend the answer, you are over-diagnosing — the exercise is to identify categories, not to invent them.

### 3. Complete the parity checklist (≈ 30 min)

Fill in the chapter-04 parity checklist (10 items) for the *current, incident-state* system. Every "no" is a place the system was vulnerable. Every "yes with a caveat" is a place your fix needs to strengthen.

Do this before designing the fix. The checklist is what forces you to look at the whole system, not just the box that broke.

### 4. Design the architectural fix (≈ 45 min)

Now the load-bearing section. For each category you identified in step 2, name **the architectural change** — not the code patch — that makes that class of skew fail in CI or at gate rather than in production. Concrete moves you can reach for:

- Move a feature transformation into a shared library or feature-service definition (chapter 03).
- Introduce a point-in-time-join primitive as the *only* supported way to build a training set; assert this in CI.
- Add a CI test that computes a feature both offline and online on a shared fixture and asserts equality.
- Add a per-feature freshness monitor to the online store.
- Add a per-feature distribution monitor comparing training-time to serving-time; page on breach.
- Add a promotion gate (chapter 03, chapter 05) that refuses to promote if the required feature version is not present in the online store.
- Change the training-set construction to *simulate* the serving-time lag of every feature.
- Register the training population contract in the model registry (chapter 03); alert on serving-time drift from the registered population.

Each fix names the class it removes, the class it does not, and the operating cost (compute, on-call surface, developer friction). If a fix would remove a class only in v2 and not v1, say so and note the interim mitigation.

### 5. Sketch the *specific* fix for the incident (≈ 15 min)

Distinct from step 4 — the specific code or configuration change that closes the current incident. This section should be short, ideally under 100 words. The point is to show you know the difference between the point fix and the class fix, and that you did *both*.

### 6. Rollout, gate, and monitor (≈ 15 min)

Cross-reference into chapter 05:

- Which chapter-05 promotion gate should carry the class-level fix (evaluation gate, shadow gate, canary abort condition)?
- Which monitor (feature-level, prediction-level, ground-truth backfill) would have caught this earlier?
- What is the rollback path if the architectural fix itself introduces a regression?

## Deliverable

A single Markdown document `skew-incident-<scenario>.md`, roughly 2 pages, structured as:

- Header — scenario, date, author.
- **1. Incident narrative** — timeline and symptoms.
- **2. Skew classification** — 5-category walk.
- **3. Parity checklist** — 10 items, current state.
- **4. Architectural fix** — the class-level changes.
- **5. Specific fix** — the instance-level patch (short).
- **6. Gates and monitors** — chapter-05 hooks.

## Acceptance criteria

- [ ] The incident is reconstructed with a concrete timeline (or, for Option A, real events).
- [ ] All five categories are classified with evidence, not left as "yes/no" without explanation.
- [ ] The parity checklist is completed for the incident-state system, with at least three items answered "no" or "at risk" (a checklist with all "yes" answers is either a wrong classification or a system that did not have the incident).
- [ ] The architectural fix in step 4 is *distinct* from the specific fix in step 5. Both are present.
- [ ] Every architectural fix names the class of skew it removes and at least one operating cost it adds.
- [ ] Gates and monitors from chapter 05 are named specifically — not "add monitoring."
- [ ] The memo is *at most two pages* excluding the parity checklist.

## Stretch goals

- **Pair the memo with a real fixture.** If you can, write the specific CI test (fixture + assertion) that would have caught the incident in the training pipeline. This is where the class-level fix becomes concrete code.
- **Post-mortem-quality writeup.** Convert the memo into a full incident post-mortem in your organisation's template. This is a preview of the reliability craft in [mod-307].
- **Two-incident contrast.** Do the exercise on a second scenario and compare which architectural moves recur. The recurring moves (point-in-time primitive, per-feature monitors, promotion gates) are your default checklist for future systems — save them to a personal template you reach for at every RFC review.
- **Feed into the RFC.** Reuse this memo's step-3 parity checklist as the source of section 6 (data and feature contracts) in the RFC you will produce in the paired project (`project-301-ml-system-design-portfolio`).
