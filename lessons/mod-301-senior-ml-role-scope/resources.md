# Resources for mod-301-senior-ml-role-scope

Curated external references. These are the primary and authoritative sources used to ground the chapters and exercises above. Every URL is publicly reachable at the time of authoring.

## Engineering career frameworks (seniority scope)

- **Engineering Ladders** — public five-axis engineering career framework (Development, Systems, People, Process, Influence). Used throughout chapter 02 (`02-altitude-and-ownership.md`). <https://www.engineeringladders.com/>
- **progression.fyi** — a collected catalogue of public engineering-career frameworks from Monzo, Meta, GitLab, Buffer, Basecamp, and others. Cross-check the L20 / L30 / L40 shape against multiple companies' rubrics. <https://www.progression.fyi/>
- **Rent the Runway engineering career ladder** (public) — a well-annotated example ladder that names the "senior tech-lead" altitude explicitly and describes the ownership shift. <https://dresscode.renttherunway.com/blog/ladder>
- **Kickstarter engineering ladder** (public) — a smaller-org ladder with a clean L3 (senior) → L4 (staff) description of the shift from team-scope to portfolio-scope. <https://github.com/kickstarter/engineering-ladder>
- **Circle CI engineering ladder** — another public reference used at progression.fyi; useful for cross-checking that the "senior owns a system, staff owns a portfolio" pattern is stable across the industry. <https://boards.greenhouse.io/circleci> (careers site linking the public ladder)

## Code review, design review, and process (chapter 01, chapter 02)

- **Google Engineering Practices Documentation** — the canonical, public description of the code-review and design-doc bar a senior engineer is expected to hold. Chapter 01 refers to this as the "bar" you are enforcing at L30. <https://google.github.io/eng-practices/>
- **Google, *Code Review Developer Guide*** — the "reviewer" side of the practices doc; useful when calibrating whether you are running reviews at L30 bar. <https://google.github.io/eng-practices/review/>

## ML systems, architecture, and the "system holds" mindset (chapters 01, 04)

- **Chip Huyen, *Designing Machine Learning Systems*** (O'Reilly, 2022) — the canonical text for the architecture altitude this track operates at. Publisher page: <https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/> · Author page: <https://huyenchip.com/books/>.
- **Andriy Burkov, *Machine Learning Engineering*** (2020) — build-altitude companion; the L20 material this track inherits and does not re-teach. <https://www.mlebook.com/>
- **Google Developers, *Rules of Machine Learning*** — 43 heuristics that senior ML engineers routinely quote in review. Chapter 01 recommends being able to quote these. <https://developers.google.com/machine-learning/guides/rules-of-ml>
- **Sculley et al., *Hidden Technical Debt in Machine Learning Systems*** (NeurIPS 2015) — the paper chapter 01 leans on to name the L30 shift ("most of a mature ML system's cost is the glue around the model"). <https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html>
- **Breck et al., *The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction*** (Google, 2017) — production-readiness rubric used later in mod-305; worth a first read at L30 orientation. <https://research.google/pubs/pub46555/>

## MLOps and hand-off contracts to peer platform tracks (chapter 04)

- **Google Cloud, *MLOps: Continuous delivery and automation pipelines in machine learning*** — the canonical description of the "paved-road" platform that L30 engineers consume rather than build. <https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning>
- **Kreuzberger, Kühl, Hirschl, *Machine Learning Operations (MLOps): Overview, Definition, and Architecture*** — academic MLOps survey; useful vocabulary for the hand-off contracts to `ai-infra-mlops-learning`. <https://arxiv.org/abs/2205.02302>
- **MLflow official documentation** — model registry as source of truth; the "interface artefact" for the deployment contract in chapter 04. <https://mlflow.org/docs/latest/index.html>

## Reliability, SLOs, and incident command (chapters 02, 05)

- **Google, *Site Reliability Engineering*** (free book) — the canonical text for SLIs / SLOs / error budgets. Referenced in chapter 05's growth-plan example for Competency C. <https://sre.google/sre-book/table-of-contents/>
- **Google, *The Site Reliability Workbook*** — companion volume with worked examples of SLO authorship. <https://sre.google/workbook/table-of-contents/>

## Experimentation and evaluation (Competency B, chapter 05)

- **Ron Kohavi, Diane Tang, Ya Xu, *Trustworthy Online Controlled Experiments*** (Cambridge, 2020) — the reference book for mod-306; cited in the chapter 05 growth-plan example. Publisher page: <https://www.cambridge.org/core/books/trustworthy-online-controlled-experiments/D97B26382EB0EB2DC2019A7A7B518F59> · Companion site: <https://experimentguide.com/>.
- **DeepLearning.AI / Coursera, *Machine Learning Engineering for Production (MLOps) Specialization*** — a well-scoped applied companion to the L30 material on evaluation programmes and monitoring. <https://www.coursera.org/specializations/machine-learning-engineering-for-production-mlops>

## Cloud certifications (real-world "what the role is hired against")

Cited for the practical requirements catalogue, not as a curriculum substitute. Both track publishers publish exam guides that name what senior ML engineers are commonly expected to be fluent in.

- **AWS Certified Machine Learning Engineer – Associate — exam guide.** <https://aws.amazon.com/certification/certified-machine-learning-engineer-associate/>
- **Google Cloud, *Professional Machine Learning Engineer* — exam guide.** <https://cloud.google.com/learn/certification/machine-learning-engineer>

## Job postings (grounding for chapter 03 and exercise-01)

Postings evidence for this track is deferred to the next autonomous research cycle (see `JOB_REQUIREMENTS.md` `Status` section). When you sample postings for exercise-01, prefer the following ATS roots — they preserve requirement bullets verbatim and are the sources the research pass will canonicalise against.

- **Greenhouse-hosted ATS roots.** <https://boards.greenhouse.io/>
- **Lever-hosted ATS roots.** <https://jobs.lever.co/>
- **Ashby-hosted ATS roots.** <https://jobs.ashbyhq.com/>

Employer coverage guidance (AI-native, consumer tech, fintech, retail/adtech, health/bio, traditional) is in `JOB_REQUIREMENTS.md` — reuse that guidance when sourcing postings for exercise-01.

## Peer-track linkage

Read the READMEs of the peer tracks referenced in chapter 04 (`04-hand-off-contracts.md`) at least once during this module so the hand-off contracts are concrete. Track slugs:

- Peer platform: `ai-infra-ml-platform-learning`, `ai-infra-mlops-learning`, `training-pipeline-engineer-learning`, `model-evaluation-engineer-learning`, `ai-eval-engineer-learning`.
- Peer specialist: `llm-application-developer-learning`, `rag-engineer-learning`, `nlp-engineer-learning`, `fine-tuning-engineer-learning`, `applied-ai-engineer-learning`.
- Review counterparts: `ai-infra-security-learning` (L35), `ai-governance-analyst-learning`.
- Prerequisite: `ml-engineer-learning` (L20).
- Higher-level: `staff-ml-engineer-learning`, `principal-ml-engineer-learning`.

<!-- needs-research: verify the public URLs for each peer-track repo once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->
