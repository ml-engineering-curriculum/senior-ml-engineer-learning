# exercise-01: Seniority Verb Audit

**Estimated effort:** 2 hours

## Objective

Read five real "Senior Machine Learning Engineer" job postings and audit them for the seniority-defining verbs, using the rubric from chapter 03 (`03-seniority-verbs-in-postings.md`). Produce a per-posting classification (L20-in-disguise, L30, L40-mistitled) plus a compressed "senior verb signature" for every posting you classify as L30.

The point of the exercise is to train your ear for what a genuine L30 posting sounds like, and to give you a repeatable rubric you can rerun the next time you are considering a job change or authoring a JD (mod-310).

## Prerequisites

- Read `03-seniority-verbs-in-postings.md`.
- Optional but recommended: read `02-altitude-and-ownership.md` first, so the L20 / L30 / L40 ownership axes are fresh.

## Sourcing the postings

Sample **five** in-window postings you can cite by URL. Ground rules:

- Titles must contain "Senior" and "Machine Learning Engineer" (or the abbreviations "Sr." / "ML Engineer"). Exclude titles that also contain "Staff", "Principal", "Platform", "MLOps", "Research", "Data Scientist", "NLP", "LLM", or "Applied AI" — those are peer or higher tracks.
- Prefer variety: at least one AI-native employer, at least one consumer-tech employer, at least one finance / fintech, and at least one from a domain you would not normally look at (health, retail, adtech, industrial). Variety exposes the range of what "senior" means across the industry.
- Prefer ATS-hosted postings (`greenhouse.io`, `lever.co`, `ashbyhq.com`, employer careers pages) — those tend to preserve the requirement bullets faithfully.
- Capture, for each posting: employer, exact title, URL, date observed, and the full text of the required and preferred bullets.

If you already have a job-postings corpus (for example a snapshot from `.aicg/job-requirements.json` on this or a peer track), you may sample from it — but re-verify the URL is still live so your writeup can be cited.

## Steps

### 1. Capture the postings (≈ 30 min)

For each posting, produce a short header block:

```
Posting #N
Employer:
Exact title:
URL:
Date observed: YYYY-MM-DD
Location / remote?:
Represents: (AI-native / consumer / fintech / other — pick one)
```

Below the header, paste the **verbatim** requirement bullets. Do not paraphrase. Split them into "required" and "preferred" if the posting does. If a bullet is a paragraph, break it into individual clauses so each clause can be classified independently.

### 2. Apply the six-family rubric to every bullet (≈ 45 min)

For each bullet, mark:

- **Family** (one or two of Build / Operate / Own / Lead / Mentor / Set-direction).
- **Scope qualifier** (component / system / programme / cross-team / `<no-scope>`).
- **Comment** — one sentence if there is anything unusual (see the anti-patterns in chapter 03).

A simple Markdown table works:

| # | Bullet (verbatim) | Family | Scope qualifier | Comment |
|---|---|---|---|---|
| 1 | Build and deploy… | Build | component | Standard L20 build bullet. |
| 2 | Own the technical direction… | Own + Set-direction | system | Genuine L30 signal. |

### 3. Compute the counts and ratio (≈ 15 min)

For each posting, produce a short summary:

- Number of bullets in Build/Operate.
- Number of bullets in Own/Lead/Mentor/Set-direction (call this the "lead-family count").
- Ratio of lead-family bullets to build-family bullets.
- Distinct scope qualifiers that appear (component / system / programme / cross-team).

### 4. Classify the posting (≈ 15 min)

Using the counts and the anti-patterns in chapter 03, assign one of:

- **L20-in-disguise** — senior-titled, bullets almost entirely Build/Operate, scope qualifiers component-level or missing.
- **L30 (genuine senior)** — at least four distinct lead-family bullets, scope qualifiers include system-level and at least one programme-level or cross-team.
- **L40-mistitled** — scope qualifiers include multi-team or portfolio, "roadmap across the org" or similar language.
- **Ambiguous** — the posting is a grab-bag or a copy-paste. Note why.

Cite the specific bullets that drove the classification. A one-word classification with no evidence is a fail.

### 5. Extract the senior verb signature (≈ 15 min)

For every posting classified as L30, produce the compressed one-sentence signature from chapter 03:

> Own <system scope>, lead <programme>, mentor <cohort>, partner with <peer team>, set direction for <decision surface>.

If a slot is not filled by the posting, mark it `<missing>`. A well-formed L30 posting will fill at least four of the five slots. Note which slots are missing across your five postings — if `mentor` and `set direction` are the most-frequently-missing slots in your sample, that is a signal about how the industry currently under-specifies senior-scope people/process expectations, and worth flagging in your writeup.

For postings you classify L20-in-disguise or L40-mistitled, skip the signature and note in one sentence which slot(s) the posting misuses (e.g. "uses Own with a component-scope qualifier — reads as L20").

## Acceptance criteria

Submit a single Markdown document `verb-audit.md` (or similar) containing:

- [ ] Five posting header blocks with URL, date observed, and the verbatim bullets.
- [ ] A six-family classification table for every posting.
- [ ] Counts, ratio, and distinct scope qualifiers per posting.
- [ ] A classification per posting (L20-in-disguise / L30 / L40-mistitled / ambiguous) with cited bullets.
- [ ] A compressed senior verb signature for every L30 posting; `<missing>` markers where slots are unfilled.
- [ ] A short (1–2 paragraph) reflection at the end: what surprised you across the five postings? Which verb families does the industry under-specify? Would you apply to any of these five given your growth plan (chapter 05, exercise-03)? Why or why not?

The writeup does not need to be long. It needs to be honest, cite bullets, and land the rubric.

## Stretch goals

- **Cross-track comparison.** Also sample one posting titled "Staff Machine Learning Engineer" and one titled "Machine Learning Engineer" (no seniority qualifier). Apply the same rubric. Does the verb distribution on the Staff posting cleanly separate from the Senior one, or is the industry noisy? Does the mid-level posting have any lead-family bullets at all?
- **Employer-shape read.** Group your five postings by employer type (AI-native / consumer / fintech / other). Is there a shape difference in what "senior" means across employer types? For example — do AI-native employers weight Set-direction bullets more heavily than fintech ones? Write one paragraph on the pattern.
- **JD authorship.** Draft your own L30 JD for the ML system on your current team, using the six-family taxonomy and hitting at least four of the five signature slots. Compare it against the five you audited. This is a preview of the JD-authorship craft that mod-310 covers.
