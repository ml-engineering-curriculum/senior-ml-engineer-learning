# Prompts as engineering artifacts

## Motivation

There is a specific pattern of failure that shows up in almost every team that starts using LLMs in production:

- The prompt lives in a Slack message, a notebook, and a copy-pasted `system_prompt = "..."` string in the serving code — three copies that have quietly diverged.
- The last time it changed, no one wrote down what changed or why, and the eval score afterwards was "seemed better on the ten examples I typed."
- The base model version was silently upgraded by the vendor and the prompt's behaviour changed with it. No one has an alert wired.
- The rollout of a "small edit" broke a live customer flow because the change touched a JSON output schema the downstream parser depended on.
- There is no per-slice pass rate — the team's only evidence of prompt quality is a hand-chosen "vibe check" of ten inputs.

Every one of these failures is what would be unacceptable for a *feature* — you version features, test them, gate them behind evaluation, deploy them via CI, page on their regressions. **Prompts are features.** They live in the serving path. They influence quality more than most classical model changes do. The senior read is: prompts are engineering artefacts, and the same discipline that keeps your ranker's code from rotting should keep your prompts from rotting.

This chapter is the discipline. Chapter 04 handles cost and latency; chapter 05 handles observability at runtime. This chapter handles the *authoring / versioning / evaluation / deployment* loop that surrounds the prompt itself.

## What a prompt actually is, at senior altitude

A prompt in production is rarely one string. It is a bundle:

- A **template** with named slots — `{customer_history}`, `{ticket_body}`, `{knowledge_base_passage}`.
- A **system instruction** that sets role, tone, and hard constraints.
- **Few-shot examples** demonstrating the desired input → output mapping.
- A **structured output schema** — JSON schema, Pydantic model, or vendor "structured output" spec — the LLM must conform to.
- **Tool / function specifications** if the pattern uses tool use.
- **Model configuration** — model ID and version, temperature, top-p, max tokens, stop sequences, seed if the vendor supports it.
- **Guardrails** — content-safety filters, refusal instructions, input pre-processing rules (e.g. "strip PII before rendering the template").

That bundle is the artefact. Version it as a unit. Evaluate it as a unit. Roll it out as a unit. If any of those parts is scattered across a codebase such that a reader has to grep to reconstruct "what was the prompt on 2026-06-15?", you do not have a versioned prompt — you have a decayed one.

## The five disciplines

Treating a prompt as code means five disciplines from the ML repo travel unchanged:

1. **Source of truth in the repo.** One file, one place, reviewed via PR.
2. **Versioning.** Explicit prompt version identifiers, pinned to specific model version, testable side by side with the previous version.
3. **Tests.** Deterministic property tests plus a golden-example test suite. Not a substitute for evaluation, but a floor.
4. **Evaluation.** A per-slice, statistically meaningful evaluation set with a defined pass-rate threshold. Runs on every PR that touches the prompt bundle and on every base-model version bump.
5. **Deployment.** Rolled out via the same feature-flag / staged-rollout discipline your ranker or classifier is.

The rest of this chapter walks each one.

## 1. Source of truth in the repo

The prompt bundle lives in the ML repo, next to the code that renders and calls it. Not in a notebook. Not in a doc. Not in a "prompt library" a product manager edits from a UI without a review. If someone can change what your LLM says in production without a PR that touches your repo, you do not own the prompt.

A workable file layout, adapted from the shape most production ML repos already have:

```
prompts/
  next_action_summariser/
    v3.md                 # the current prompt bundle, human-editable
    v3.schema.json        # the structured output schema
    v3.tests.yaml         # deterministic tests and golden examples
    v3.eval.yaml          # eval set reference + pass-rate thresholds
    CHANGELOG.md          # human-authored rationale per version
    README.md             # what this prompt does, how to run it locally
  ...
```

Version numbers are integer-bumped on breaking changes to the prompt or the schema; a minor edit that does not change the output schema can be a suffix (`v3a`, `v3b`) — pick a convention and stick with it. The point is not the convention; the point is that the prompt-loading code in serving can name a version, and rollback is `git revert` + a redeploy.

The prompt itself is a Markdown file rather than a Python string. That is deliberate — it lets you review a prompt like a document (with headings, examples, and rationale), diff it cleanly in PRs, and let non-engineer collaborators (product, ops, safety) read and comment. The rendering code loads it, substitutes the slots, and calls the LLM.

Anthropic's [prompt-engineering guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview), OpenAI's [prompt-engineering guide](https://platform.openai.com/docs/guides/prompt-engineering), and Google's [Vertex prompt-design guide](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/introduction-prompt-design) each cover the authoring craft — few-shot, structured output, chain-of-thought, XML tags, delimiters. Consult them for the technique; the *versioning* discipline this chapter installs is orthogonal.

## 2. Versioning, and the base-model version is part of it

The prompt version is a compound: `<prompt_template_version, model_id, model_version, decoding_config_hash>`. When any component changes, the compound version changes. The reason is a rule most teams learn the hard way: **the same prompt on a "silently updated" base model can produce systematically different outputs.** Vendors advertise that model IDs like `claude-sonnet-4-5` are pinned, but the point-in-time behaviour of a model can shift with policy updates, safety fine-tunes, and infrastructure changes. Pin the version you can pin (Anthropic's [model overview](https://docs.anthropic.com/en/docs/about-claude/models/overview) and OpenAI's [model documentation](https://platform.openai.com/docs/models) publish specific version identifiers you should use, not the "latest alias") and be alerted when it changes.

Concretely, the prompt-loading code returns something like:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class PromptBundle:
    name: str                      # "next_action_summariser"
    version: str                   # "v3"
    template: str                  # rendered template text
    schema: dict                   # JSON schema for structured output
    model: str                     # e.g. "claude-sonnet-4-5-20250929"
    temperature: float
    max_tokens: int
    stop: list[str] | None
    tools: list[dict] | None       # tool/function specs
    guardrails: dict               # e.g. {"pii_strip": True}

    @property
    def fingerprint(self) -> str:
        """Stable hash used for logging and eval attribution."""
        ...
```

Every LLM call is logged with the `fingerprint`. That is how chapter 05 attributes production quality regressions to specific prompt versions. Without this, an incident review cannot answer "which prompt was live at 03:00 last night?"

## 3. Tests — the deterministic floor

Prompt evaluation (next section) is what tells you the prompt is *good*. Tests are what tells you the prompt is *not broken*. They run in CI in seconds and they catch the failures that would embarrass you if they shipped:

- **Template renders.** No missing slots, no unclosed tags, no accidental trailing whitespace that changes tokenisation.
- **Structured output validates.** Sample a stored (real or synthetic) LLM response and confirm it parses against the schema. This catches schema drift when someone edits the schema without updating the prompt or vice versa.
- **Golden examples.** A handful of small, hand-curated inputs whose *shape* of expected output is known — a summary containing the customer name, a classification whose top choice is one of the taxonomy values. Golden tests should be about **invariants**, not exact strings. "The output JSON contains a `next_action` field whose value is in the taxonomy" is a good golden test. "The summary is exactly this paragraph" is a bad one (LLMs are non-deterministic).
- **Guardrail tests.** If the prompt is required to refuse harmful inputs, a small set of red-team examples should confirm refusal. If it must strip PII before rendering, a test should confirm the PII does not leak into the rendered template.
- **Regression fixtures.** Every time an incident produces an input that broke the prompt in production, the input goes into the fixture set with the correct behaviour asserted. This is the same "bug fix comes with a regression test" discipline you already use.

Tests do **not** replace evaluation. A test suite of ten golden examples cannot tell you whether the prompt is right on 95 % of inputs. It can tell you the prompt has not been broken since the last commit.

## 4. Evaluation — the per-slice pass rate

Evaluation is what tells you whether the prompt is *good enough to ship*. It is the load-bearing artefact for the pattern-decision rubric in chapter 01 — Q3 ("does prompting reliably solve this?") has no meaning without an eval set.

At senior altitude the eval discipline follows [mod-305]'s pattern:

- **A statistically meaningful eval set.** Big enough for the per-slice metrics you care about to have narrow confidence intervals. Rule of thumb: hundreds to low thousands of examples per slice. Not ten.
- **Labelled slices.** The eval set is stratified across the slices the launch has to hold on — customer type, ticket topic, product line, region, language. A prompt that averages 92 % but fails at 60 % on the Spanish-language slice is not a shippable prompt.
- **A pass metric with a threshold.** LLM outputs are open-ended, so pick a metric that fits the task:
  - **Deterministic checks** for structured output — is the JSON valid, is the field's value in the taxonomy, is the number in range?
  - **Reference-based metrics** when there are ground-truth references — exact match / F1 for extraction, ROUGE / METEOR for summarisation. These have known weaknesses (ROUGE rewards lexical overlap even when meaning diverges) — use as a crude signal, not a truth.
  - **LLM-as-judge** for open-ended quality — a second (usually stronger, or at least differently-trained) LLM judges the output against a rubric. The judge itself needs its own eval (are its verdicts correlated with human ones on a held-out set?). Zheng et al.'s [*Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*](https://arxiv.org/abs/2306.05685) is the canonical reference.
  - **Human labels** on a sampled slice, especially for the safety-relevant tail. This is expensive; use it for the acceptance gate and for calibrating the LLM-judge.
- **Confidence intervals**, not point estimates. A prompt that scores 91 % on 100 examples cannot be distinguished from 88 % on statistical grounds. If you cannot separate the new prompt from the old on the eval set, you are guessing.
- **Adversarial and tail slices.** Include inputs that broke past prompts, inputs that are edge cases in your product, and inputs the safety team owns. A production regression that traces back to a slice you did not evaluate is a preventable failure.

The eval harness runs on every PR that touches the prompt bundle. It also runs on a schedule, unattended, to catch base-model drift and eval-set drift. Regressions block merge.

## 5. Deployment — staged rollout, not a swap

A prompt change ships the way a model change ships. That means:

- **Feature-flagged.** The serving path can select `v2` or `v3` based on a flag. That flag defaults off for new prompts; a rollout enables it for a percentage of traffic.
- **Staged.** 1 % → 10 % → 50 % → 100 %, gated at each stage on live evaluation, error rate, and cost/latency (chapter 04). Regressions on any of those bounce the flag back.
- **Shadow-evaluated first.** Before any live-traffic rollout, run the new prompt in shadow: real production inputs, new prompt output, no user impact, comparison against the old prompt. This is the same "shadow deployment" pattern from [mod-302] and [mod-305].
- **Rollbackable.** The old prompt is still in the repo. Rollback is a flag flip, not a re-deploy.

The concrete rollout gate looks like: shadow diff shows less than *X* % rate of "materially different" outputs on the eval slice; live 1 % rollout shows no regression on the flagged metrics for a full traffic cycle; then 10 %, etc. Any escalation stops the rollout automatically.

## Making the vendor SDKs cooperate

Modern LLM SDKs give you three things you should turn on by default in production:

- **Structured outputs.** OpenAI's [structured outputs](https://platform.openai.com/docs/guides/structured-outputs), Anthropic's [tool use with output schema](https://docs.anthropic.com/en/docs/build-with-claude/tool-use), and vendor JSON-mode features constrain the output to a schema. Constraining eliminates a whole class of "output was valid JSON except when it wasn't" incidents. Use them, and put the schema in the versioned prompt bundle so the schema and the prompt cannot drift apart.
- **Prompt caching.** Anthropic's [prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching), OpenAI's [prompt caching](https://platform.openai.com/docs/guides/prompt-caching), and equivalents let you cache the deterministic prefix of a prompt (system instructions, tool specs, long few-shot examples) so subsequent calls with the same prefix cost less and answer faster. Layout the prompt with the static bits first, dynamic bits last, and the cache almost pays for itself in most workloads. Chapter 04 revisits this in the cost discussion.
- **Batch inference** for offline / async workloads. Vendors offer 50 %-off (or similar) pricing for asynchronous batch jobs — see OpenAI's [batch API](https://platform.openai.com/docs/guides/batch) and Anthropic's [message batches API](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing). If your feature can tolerate hours of latency (offline enrichment, chapter 03), batching drops the cost line item on its own.

None of these substitute for the discipline in sections 1–5 — they are levers you pull *inside* that discipline.

## The prompt registry — one artefact your platform team may own

Once you have more than a handful of prompts in the repo, a prompt registry becomes worth building: a service that keeps versioned prompt bundles, exposes them to serving with a `get_bundle(name, version)` API, and logs which version served which request. Vendors ship one — see, for example, [Vertex AI's prompt management](https://cloud.google.com/vertex-ai/generative-ai/docs/prompt-management) and [Amazon Bedrock's prompt management](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-management.html) — and OSS tools like [LangSmith](https://docs.smith.langchain.com/) provide a similar layer.

At senior altitude, the decision is not "should we build this?" — it is:

- Is this best consumed as a paved-road service from the platform team (`ai-infra-mlops-learning` — see [mod-308])?
- Or is the registry small enough that the "repo file plus a fingerprint" pattern above is fine for now?

Small teams start with the repo pattern. Larger orgs converge on a service. Do not build a bespoke registry when the platform team already offers one.

## A worked example — v2 → v3 of the ticket summariser

Starting state: `next_action_summariser/v2` is live on 100 % of traffic. Product wants the summary to include the last relevant knowledge-base article. That change is more than a prompt tweak — it touches the input schema (adds a `top_kb_article` slot) and the output schema (adds a `cited_article_id` field).

Workflow:

1. **Author `v3` as a new bundle.** New Markdown file, new JSON schema, new eval file. Do not overwrite v2.
2. **Update golden and regression tests** for the new schema. If the parser downstream doesn't yet handle `cited_article_id`, gate it behind a feature-flag on the parser side.
3. **Run eval.** The eval set has a "kb-relevant" slice added for the new capability. Confirm the new prompt does not regress on the existing slices.
4. **Ship the parser change and the prompt change independently.** Parser goes first, defensively handling both v2 and v3 outputs. Then the prompt version can be swapped without a coordinated deploy.
5. **Shadow-evaluate v3 against v2.** For a day of real production inputs, log both outputs, diff them, spot-check the diffs.
6. **Staged rollout.** 1 % → 10 % → 50 % → 100 %, gated on the flagged metrics.
7. **Retire v2** — do not delete it from the repo (rollback), but stop routing traffic to it.

None of the steps are LLM-specific. They are the disciplines you already use for model rollouts (see [mod-302] chapter 05 and [mod-305]) applied to the prompt.

## Three failure modes to catch in review

- **"The prompt is in a Google Doc."** Any prompt that changes without a repo commit is not versioned. Ask where the prompt lives; if the answer is anything other than a specific file path in the ML repo, the discipline is missing.
- **"We tested it on ten examples."** Ten examples cannot separate a 90 %-pass-rate prompt from an 80 %-pass-rate one. Ask for the eval set size, the slice coverage, and the confidence interval. If the team cannot show one, evaluation is missing.
- **"We didn't pin the model version."** The prompt is called with `claude-sonnet-4-5` (an alias) or `gpt-4o` (a family). Silent version updates change behaviour. Pin the specific version identifier and alert on updates.

## Summary

Prompts in production are engineering artefacts. Treat them like features: one source of truth in the repo, explicit compound versioning (template + model + decoding), deterministic tests as a floor, per-slice evaluation with confidence intervals as the acceptance gate, staged rollout with rollback. Modern vendor SDKs give you structured outputs, prompt caching, and batch pricing — use them inside the discipline, not as a substitute for it. When the prompt count grows beyond a handful, consume a paved-road prompt registry from the platform team rather than building bespoke. Chapter 03 picks up the hybrid patterns; chapters 04–05 wire the cost / latency guardrails and observability that the prompt version is instrumented against.
