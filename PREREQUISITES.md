# Prerequisites — Senior Machine Learning Engineer Track

**Role level:** 30 (senior tech-lead altitude)

This track is a **level-up** curriculum, not a from-scratch one. It assumes fluency in the build-altitude ML practitioner workflow and adds architecture, evaluation, reliability, cross-team collaboration, and technical-leadership scope on top. Do not start here without the prerequisites below in hand.

## Assumed foundation (level 20)

The full [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) curriculum is the primary prerequisite. Concretely, you should already be able to:

- Frame a business problem as a regression, classification, ranking, or retrieval task.
- Ingest, profile, clean, and validate data; engineer features for tabular, text, and image data.
- Train and tune classical models (regression, trees, gradient-boosted ensembles) with correct cross-validation and hyperparameter search.
- Write a PyTorch training loop, build an MLP / CNN / small Transformer encoder, and apply regularisation and mixed-precision training.
- Track experiments with MLflow, version datasets and models, and produce reproducible runs.
- Pick metrics correctly for the task, run leakage-resistant validation, and conduct error analysis.
- Package a model behind FastAPI, containerise with Docker, reason about batch vs. online inference, and load-test the service.
- Instrument the ML-specific monitoring signals (data drift, concept drift) and design a retraining trigger.
- Walk through an ML system design from business goal to blueprint at a working-level altitude.

If any of the above is shaky, complete `ml-engineer-learning` first. This track will assume — and reinforce, but not re-teach — every one of them.

## Assumed engineering craft

Owned by [`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning):

- Git workflow, code review basics, README hygiene, Linux command-line fluency.
- Basic scripting, project structure, and testing practices.

Owned by [`ai-infra-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-engineer-learning) at the same level (peer, level 20):

- Working familiarity with Docker, cloud object storage, and one managed model-serving path.
- Enough Kubernetes vocabulary to consume a paved-road deployment rather than build one.

## Assumed collaboration context

You should have shipped at least one ML feature or system in a team setting, meaning you have:

- Written a design doc or RFC that a reviewer read and pushed back on.
- Been in a code review as reviewer, not just author.
- Handed off a system to on-call or received one you did not write.

Without any of these lived experiences, the leadership-scope modules (mod-301, mod-308, mod-310) and the tech-lead simulation project (project-303) will feel abstract.

## Recommended reading before you start

- Chip Huyen, *Designing Machine Learning Systems* (O'Reilly, 2022) — the canonical text for the architecture altitude this track operates at.
- Andriy Burkov, *Machine Learning Engineering* (2020) — the build-altitude companion to reinforce.
- Google Developers, *[Rules of Machine Learning](https://developers.google.com/machine-learning/guides/rules-of-ml)* — treat as heuristics you can quote in reviews.
- Kohavi, Tang, Xu, *Trustworthy Online Controlled Experiments* (Cambridge, 2020) — mod-306 is grounded in this book.

## Recommended tooling access

- MLflow (or equivalent) running locally.
- PyTorch and scikit-learn.
- Docker.
- An LLM API key you can budget against (mod-304, project-302) — or a local inference stack such as Ollama.

## Peer specialist tracks worth knowing exist

The senior ML engineer routinely delegates to and collaborates with these owners. You do not need to master them, but you should be able to read the shape of their contract:

- `nlp-engineer-learning`, `rag-engineer-learning`, `llm-application-developer-learning`, `fine-tuning-engineer-learning` — LLM specialisations.
- `model-evaluation-engineer-learning`, `ai-eval-engineer-learning` — evaluation-platform depth.
- `training-pipeline-engineer-learning` — distributed training infrastructure.
- `ai-infra-ml-platform-learning`, `ai-infra-mlops-learning` — the paved-road ML platforms this role consumes.
- `ai-infra-security-learning`, `ai-governance-analyst-learning` — the review counterparts for the responsible-AI packets you'll be authoring in mod-309.
