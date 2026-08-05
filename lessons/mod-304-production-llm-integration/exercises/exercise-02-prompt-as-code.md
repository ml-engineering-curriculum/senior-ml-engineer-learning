# exercise-02: Prompt As Code

**Estimated effort:** 3 hours

## Objective

Take a "scratch-note" prompt — the kind that lives in a notebook, a Slack thread, or a copy-pasted `system_prompt = "..."` string — and turn it into a **versioned prompt bundle** that ships to production with the same discipline as any other feature in your ML repo. That means: source of truth in the repo, an explicit version pinned to a specific model version, deterministic tests in CI, a per-slice evaluation set with a defined pass-rate threshold, and a staged rollout plan.

You are not writing a new prompt; you are installing the five disciplines from chapter 02 (`02-prompts-as-engineering-artifacts.md`) around one. The deliverable is a small PR-shaped deliverable — a folder of files, plus a one-page rollout memo — that a reviewer could actually merge.

This is the L30 tell that separates "we're using an LLM" from "the LLM feature is production-quality." An L20 ships the prompt; an L30 ships the bundle and the discipline around it.

## Prerequisites

- Read chapter 02 (`02-prompts-as-engineering-artifacts.md`) — the five disciplines and the prompt-bundle file layout are load-bearing.
- Skim chapter 04 (`04-cost-and-latency-guardrails.md`) so your bundle carries a `max_tokens` and a timeout that were chosen deliberately.
- Skim chapter 05 (`05-observability-for-llm-augmented-systems.md`) so your bundle emits a `fingerprint` you can attribute production quality regressions to.
- Optional: skim [mod-305] for the eval-harness shape this exercise leans on.
- Access to at least one hosted LLM API (Anthropic, OpenAI, Vertex, or Bedrock) with pinned model-version identifiers.

## The scratch prompt you will promote

Pick **one** of the two starter scenarios below (or bring your own — see stretch goals).

### Scenario A — the ticket summariser (chapter 01's worked example)

**Feature.** On ticket open, produce a ≤ 120-word summary of the customer's last ten interactions and identify one of eight next-action taxonomy values.

**Starting state.** The team currently uses this string, pasted into serving code:

```python
system_prompt = """You are a helpful assistant. Summarise this customer's ticket history
and suggest a next action from: refund, escalate, ask_for_order_id, resend_link,
apologise_and_wait, close_as_duplicate, close_as_resolved, other."""
```

The team has no eval set beyond ten hand-typed inputs, no versioning, no test in CI, and no idea which base model version it was validated against. Traffic is currently 100 rps peak; the plan is to grow it 5× over the next quarter.

### Scenario B — the review-summariser (batch enrichment)

**Feature.** Nightly, produce a ≤ 60-word summary of the last 30 days of reviews for each product with ≥ 5 new reviews. Runs off the request path (chapter 03 shape 5). Results feed the product-detail page.

**Starting state.** A `notebooks/review_summariser.ipynb` cell that concatenates reviews into a prompt with the instruction "summarise these reviews focusing on what customers most consistently mention." No structured output; whatever the LLM writes is what lands on the page. Two teams already report having "improved the prompt" in different branches.

## Steps

### 1. Author the prompt bundle in the repo (≈ 45 min)

Following chapter 02 section 1, produce this folder in your working repo (or a fresh sandbox repo):

```
prompts/
  <feature_name>/
    v1.md               # the prompt bundle — template with named slots
    v1.schema.json      # the structured-output schema (JSON schema)
    v1.tests.yaml       # deterministic tests and golden examples
    v1.eval.yaml        # eval-set reference + per-slice pass-rate thresholds
    CHANGELOG.md        # human-authored rationale — start with v1 entry
    README.md           # what this prompt does, how to run it locally
```

The prompt bundle must, at minimum:

- Be a Markdown file with named slots (`{customer_history}`, `{ticket_body}`, or `{reviews_last_30d}`, depending on scenario).
- Declare a **structured output schema** — a JSON schema the LLM output must conform to. Even scenario B benefits from a schema (e.g. `{"summary": "string", "top_themes": ["string"]}`). Chapter 02: "constraining eliminates a whole class of 'output was valid JSON except when it wasn't' incidents."
- Pin a **specific model version identifier**, not a family alias (Anthropic's [model overview](https://docs.anthropic.com/en/docs/about-claude/models/overview) and OpenAI's [model list](https://platform.openai.com/docs/models) publish specific version strings). Chapter 02: `<prompt_template_version, model_id, model_version, decoding_config_hash>`.
- Set explicit `max_tokens`, `temperature`, and any `stop` sequences deliberately — write a one-line comment in the bundle explaining why each number was chosen.

### 2. Wire a `PromptBundle` loader and fingerprint (≈ 30 min)

Implement (or sketch, if this exercise is on paper) the `PromptBundle` dataclass from chapter 02 with a stable `fingerprint` — a hash over the entire bundle (`template + schema + model + model_version + temperature + max_tokens + stop + tools + guardrails`). The fingerprint is what production logs later join on to attribute a quality regression to a specific bundle version.

Serving-side code loads the bundle by `(name, version)`, renders the slots, calls the LLM, validates the response against the schema, and logs the fingerprint on every call. Chapter 05 explains why this attribution is the difference between a five-minute regression debug and a week-long one.

### 3. Write deterministic tests (≈ 30 min)

Chapter 02 section 3 lists the five test categories. Author a `v1.tests.yaml` (or a `test_v1.py`) that covers, at minimum:

- **Template renders.** No missing slots for every valid input shape.
- **Structured output validates.** A stored (real or synthetic) LLM response parses against the schema.
- **Golden examples.** Three to five hand-curated inputs where the *shape* of the expected output is asserted — for scenario A, "top-level `next_action` is one of the eight taxonomy values"; for scenario B, "output has a non-empty `summary` string of ≤ 60 words." Do not assert exact strings.
- **Guardrail tests.** If the prompt is required to strip PII before rendering (chapter 02: "if it must strip PII before rendering, a test should confirm the PII does not leak into the rendered template"), a test confirms it.
- **Regression fixtures.** One or two synthetic inputs that break naive prompts — an empty ticket history, a review body containing prompt-injection text like "ignore prior instructions and…" — with the expected sane behaviour asserted.

Tests should run in seconds and not depend on live LLM calls (mock the LLM response or use stored fixtures).

### 4. Sketch a per-slice evaluation set (≈ 45 min)

Chapter 02 section 4: "hundreds to low thousands of examples per slice. Not ten."

Design (do not necessarily populate) the eval set:

- **Total size.** How big does it need to be for the per-slice metric's confidence interval to be tight enough to gate a release?
- **Slices.** For scenario A: ticket topic, customer tier, region / language, ticket length. For scenario B: product category, review-count bucket, presence of adversarial content, language. Name every slice you would insist on having represented before shipping.
- **Metric.** Which of the four evaluation shapes from chapter 02 (deterministic checks, reference-based metrics, LLM-as-judge, human labels) applies to which slice? A structured-output field like `next_action` is a deterministic check; a summary's quality is LLM-as-judge on a rubric, calibrated by a small human-labelled slice.
- **Pass-rate threshold.** Per slice, not aggregate. "The Spanish-language slice pass rate must be ≥ 90 %" is a threshold you can gate on; "the aggregate quality is high" is not.

The `v1.eval.yaml` names the eval set (reference, not the data itself), the slices, the metrics, and the thresholds. It does not need to have every row populated to be a passing deliverable — the shape and thresholds are what the reviewer cares about.

### 5. The rollout memo (≈ 30 min)

One page, in the same repo (or in the folder alongside the bundle). It answers:

- **Shadow evaluation.** How you will run `v1` in shadow against real production inputs before any live rollout, and what "shadow diff acceptable" means (chapter 02 section 5).
- **Staged rollout schedule.** 1 % → 10 % → 50 % → 100 %, with the gate at each stage (regression on which metrics bounces the flag back).
- **Rollback.** How you disable `v1` and re-enable the previous scratch behaviour (or the "no summary" degradation from chapter 04) if the metrics regress.
- **Base-model-version alerting.** How you will be notified if the vendor deprecates or silently updates the pinned model version. Anthropic's [deprecations page](https://docs.anthropic.com/en/docs/about-claude/models/overview) and OpenAI's [deprecations page](https://platform.openai.com/docs/deprecations) are the sources.
- **Observability tags.** Which of the chapter 05 signal families are wired from day one — at minimum, `prompt_version` and `model_version` as first-class metric labels, and per-request token counts + dollar cost.

## Deliverable

A single folder `prompts/<feature_name>/` containing the six files from step 1, plus a `rollout.md` (or a `README.md` section) from step 5. If your team does not have a repo to commit to, produce the same folder as a zip / gist.

Total, this is probably ~200 lines of prompt / schema / tests / eval spec plus a one-page memo. It is deliberately small — the exercise is the *discipline*, not the volume.

## Acceptance criteria

- [ ] The bundle folder exists with all six files present (bundle, schema, tests, eval spec, changelog, README).
- [ ] The prompt file has named slots and a structured-output schema; the schema is a valid JSON schema (or vendor structured-output spec) that the prompt tells the LLM to conform to.
- [ ] The model version is pinned to a specific version identifier from the vendor's model list, not a family alias like `gpt-4o` or `claude-sonnet-4-5`.
- [ ] `max_tokens`, `temperature`, and any `stop` sequences are set to deliberate numbers with a one-line justification each.
- [ ] The `PromptBundle`-shaped loader exists (in code or in pseudocode) and computes a stable `fingerprint` hash over the whole bundle.
- [ ] Deterministic tests cover at least: template rendering, schema validation, three golden examples, one guardrail test, one regression fixture.
- [ ] The eval spec names slices, per-slice metrics, per-slice pass-rate thresholds, and the total size / target confidence interval — even if the rows are not yet populated.
- [ ] The rollout memo covers shadow evaluation, staged rollout schedule, rollback, base-model-version alerting, and the day-one observability tags (`prompt_version`, `model_version`, tokens, cost).
- [ ] Nothing in the bundle depends on live LLM calls to test (mocks / fixtures used for the deterministic layer).

## Stretch goals

- **Actually run the eval.** Populate ~50 rows on the smallest slice (scenario B's "product category = electronics" is a tractable choice) and produce a pass-rate number with a confidence interval. Then bump the prompt (`v1` → `v2` with one deliberate change — a tighter instruction, an added few-shot example) and produce the same number for `v2`. Chapter 02 section 4: "if you cannot separate the new prompt from the old on the eval set, you are guessing."
- **Wire prompt caching.** Restructure the bundle so the static prefix (system instructions, few-shot examples) is separated from the dynamic suffix (the specific input), and enable Anthropic's [prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) or OpenAI's [prompt caching](https://platform.openai.com/docs/guides/prompt-caching). Measure the per-call cost delta on a small sample. Chapter 04 section on prompt caching.
- **Bring-your-own bundle.** Repeat the exercise against an actual production prompt from work. Anonymise as needed. Note whether promoting the scratch prompt to a bundle would have caught a real past incident.
- **Feed into the paired project.** Reuse the bundle folder as the "prompt component" of `project-302-llm-augmented-ml-feature`. That project needs a bundle exactly this shaped.
