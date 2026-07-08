# exercise-02: Role Hand-Off Map

**Estimated effort:** 3 hours

## Objective

Draw the **hand-off map** for a specific real (or realistic) production ML system that you own or would own as a Senior ML Engineer. The output is a written artefact that, for every neighbouring track (peer platform, peer specialist, review counterpart), states what you own, what they own, the interface between you, and one concrete failure mode you will guard against.

The point of the exercise is to turn chapter 04 (`04-hand-off-contracts.md`) from a reference table into a living artefact for a system you can defend in a design review. Exercise-03 will use this map as an input to your growth plan.

## Prerequisites

- Read `04-hand-off-contracts.md`.
- Have chapter 02 (`02-altitude-and-ownership.md`) fresh — the ownership axes underlie the hand-off contracts.
- Pick the target ML system before starting. Two options:
  - **Preferred:** an ML system you currently own or have shipped (search ranker, fraud model, recommender, forecasting service, LLM-augmented feature, etc.). This makes the exercise concrete and portable back to work.
  - **Fallback:** a hypothetical but well-scoped system. Reasonable defaults: (a) a ranking model for a marketplace search page, (b) a fraud-detection model at a fintech, (c) an LLM-augmented content summariser plugged into an existing product. Whichever you pick, write one paragraph describing it before starting — reviewers cannot audit a map without the underlying system.

## Steps

### 1. Describe the system (≈ 20 min)

At the top of the artefact, write a short **system spec**:

- One-paragraph description of the ML system and the product it lives in.
- The team boundary — which team owns the system today, roughly what size, roughly what other functions sit around it.
- The current lifecycle stage — greenfield, in-production for a quarter, mature, legacy.
- One sentence on why the system exists and what the primary product metric is.

If you are describing a real system, redact anything sensitive but keep the shape.

### 2. Enumerate the neighbouring tracks (≈ 20 min)

For your system, list every peer track from the three categories in chapter 04 that is **actually relevant** to this system. Not every system touches every track — a pure tabular fraud model probably has no LLM specialist neighbour; an LLM-summariser probably has no training-pipeline neighbour. Be honest about which contracts exist.

For each track, mark whether it is:

- **Peer platform track** (paved-road consumption).
- **Peer specialist track** (delegation contract).
- **Review counterpart** (packet authorship + sign-off).

### 3. Fill the contract table (≈ 90 min)

For every neighbouring track you enumerated, produce a row of the following table:

| Peer track | Category | You own (L30) | They own | Interface artefact | One realistic failure mode | Guardrail |
|---|---|---|---|---|---|---|

- **You own (L30)** — what stays inside your ownership boundary as the Senior ML Engineer for this specific system. Do not paste from chapter 04 — restate in the language of your system. If the row of chapter 04 says "feature spec, feature freshness SLO," yours should say something like "the spec for the `search_query_ctr_1d` feature and its 30-minute freshness SLO."
- **They own** — the concrete deliverables from the peer team.
- **Interface artefact** — the concrete document or protocol carrying the contract for this system (e.g. `feature_spec.yaml` in the shared feature registry, the retrieval-quality SLO in `retrieval_slo.md`, the review packet stored at `packets/summariser_v3.md`).
- **One realistic failure mode** — a specific way this hand-off can fail. Not a generic "communication breakdown" — a concrete failure. Example: "The feature store publishes `search_query_ctr_1d` on a 1h freshness SLA, but the retraining trigger assumes 15m freshness — silent training/serving skew."
- **Guardrail** — a concrete mitigation. A dashboard, a test, an SLO alert, a review-cadence artefact, a paved-road pattern. Not "communicate more."

Do at least eight rows total, with representation from all three categories. If your system has fewer than eight relevant neighbours, add rows for the neighbours you would need if the system moved to the next lifecycle stage (greenfield → in-production, in-production → mature, mature → legacy) — that surfaces the contracts you have not yet had to design.

### 4. Draw the boundary picture (≈ 30 min)

Add a diagram to the artefact. Any format is fine — Mermaid in Markdown, a hand-drawn photo, a whiteboard screenshot, an ASCII sketch. Requirements:

- Your ML system sits in the middle, clearly labelled.
- Every neighbouring track from step 2 is a box around it.
- The **arrow direction** matters: draw arrows *from* the party who authors the interface artefact *to* the party who reviews or consumes it. So the feature-store paved road has an arrow *from* the ML platform team *to* you (you consume it). The review packet has an arrow *from* you *to* the responsible-AI reviewer (you author it).
- Annotate each arrow with the interface artefact from your table (e.g. `feature_spec.yaml`, `review_packet.md`).

The picture is the artefact you will pin to a design-review slide when you defend the system's scope in front of a director.

### 5. Reflect on scope-creep and scope-punt (≈ 20 min)

Close the artefact with a short reflection (2–4 paragraphs) answering:

- **Which contract are you most likely to violate by scope creep?** (Where would you catch yourself building the paved road instead of consuming it, or diving into the specialism instead of writing a good spec?) Name the neighbour and one honest reason.
- **Which contract are you most likely to violate by scope punt?** (Where would you catch yourself throwing an under-specified problem at the peer team and being surprised by what came back, or rubber-stamping a review packet?) Name the neighbour and one honest reason.
- **What one change to your growth plan** does this exercise imply? For example: "I need to invest in mod-308 because my instinct is to build the shadow platform," or "I need to invest in mod-309 because I have never authored a threat-model draft before handing to security review."

The reflection is the load-bearing part of the artefact — the table is context, the reflection is the calibration.

## Acceptance criteria

Submit a single Markdown document `hand-off-map.md` containing:

- [ ] The system spec (one paragraph + boundary metadata).
- [ ] The list of relevant neighbouring tracks with category tags.
- [ ] The contract table with at least eight rows, covering all three neighbour categories.
- [ ] The boundary diagram with correctly-directed arrows annotated by interface artefact.
- [ ] The 2–4-paragraph reflection naming your most likely scope-creep contract, most likely scope-punt contract, and one implied growth-plan change.

## Stretch goals

- **Cross-check against a real posting.** Take one of the L30 postings you audited in exercise-01. If you were the hire, would this hand-off map cover the contracts implied by the posting? Which contracts would need to be added or reshaped?
- **Contract-versus-reality gap.** For a system you actually own today, mark each row in the contract table as "clean" (contract holds), "informal" (contract exists but not written down), or "broken" (contract does not exist and it hurts). The distribution of informal / broken cells is a candidate list for RFC work in mod-308.
- **Author one contribute-back RFC skeleton.** For a paved-road contract you rated informal or broken, sketch the outline of a contribute-back RFC to the platform team. Do not write the full RFC — this is mod-308's job — but the outline is enough to demonstrate the muscle.
