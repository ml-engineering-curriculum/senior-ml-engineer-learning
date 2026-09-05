# Resources for mod-310-technical-leadership

Curated external references. These are the primary and authoritative sources the chapters and exercises cite or lean on. Every URL is publicly reachable at time of authoring; where a book or paper's canonical URL is behind a paywall, an author-hosted preprint or publisher landing page is provided instead.

The senior-technical-leadership canon is unusual in that its most-cited texts are books, blogs, and industry essays rather than peer-reviewed papers. That's a feature, not a bug: the practices this module trains (roadmaps, reviews, mentorship, interview loops) are experiential disciplines refined in industry rather than empirically-studied phenomena, with a few load-bearing exceptions (Schmidt & Hunter for interview validity, Fagan and Wiegers for peer-review foundations, Kahneman for decision hygiene). Where empirical evidence exists — most heavily on the interviewing side — the papers are called out inline.

## The load-bearing frames (all chapters)

- **Will Larson, *[An Elegant Puzzle: Systems of Engineering Management](https://lethain.com/elegant-puzzle/)*** (Stripe Press, 2019). The load-bearing single-book reference for senior-and-staff engineering-management judgement — sizing teams, running planning, building roadmaps, holding the bar. Chapters *Sizing Engineering Teams*, *Product Management with Just Enough Process*, and *Systems, Not Heroes* are the ones this module leans on most. The [full text is free online](https://lethain.com/elegant-puzzle/).
- **Will Larson, *[Staff Engineer: Leadership Beyond the Management Track](https://staffeng.com/)*** (2021). The archetype catalogue (*tech lead*, *architect*, *solver*, *right hand*) this module names in the introduction; the *tech lead* archetype is the one mod-310 trains against. The [full text and companion essays](https://staffeng.com/) are free online.
- **Camille Fournier, *[The Manager's Path](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)*** (O'Reilly, 2017). The load-bearing external reference on the L20 → L30 → management transition. The *Tech Lead* and *Mentorship* chapters are the ones this module inherits framing from — the roadmap discipline as the load-bearing shift in the tech-lead role, the mentor-vs-manager line, the sponsorship-vs-mentorship distinction.
- **Lara Hogan, *[Resilient Management](https://larahogan.me/management/)*** (A Book Apart, 2019). The conversational-discipline companion to Fournier. Hogan's *[Questions for our first 1:1](https://larahogan.me/blog/first-one-on-one-questions/)* and *[The Feedback Equation](https://larahogan.me/blog/feedback-equation/)* essays are the specific references for the mentorship chapter's conversation-shape guidance.
- **Tanya Reilly, *[The Staff Engineer's Path](https://www.oreilly.com/library/view/the-staff-engineers/9781098118723/)*** (O'Reilly, 2022). The more-recent companion to StaffEng and *Elegant Puzzle*, with heavier practical scaffolding on the *big picture, execution, and leveling up* triad. Chapters on *Meaningful Availability* and *Reviews* compose cleanly with this module's chapters 02–03.
- **Kim Scott, *[Radical Candor](https://www.radicalcandor.com/)*** (St. Martin's Press, 2017). The *care personally / challenge directly* frame that structures the feedback discipline in the mentorship chapter and — with more caution — the debrief and pushback discipline in the code-review and design-review chapter.
- **Andy Grove, *[High Output Management](https://www.penguinrandomhouse.com/books/281469/high-output-management-by-andy-grove/)*** (Random House, 1983 / re-released 1995). The classical management text on leverage — the underlying framing behind chapter 03's "mentorship and standards work *scale the senior*" thesis and chapter 02's "the senior reviewer's leverage is the team's ability to review at bar, not the senior's own review throughput." Older than most of this list, and still the reference.

## Chapter 01 — Roadmap and scoping

### Primary references

- **Will Larson, *[Sizing Engineering Teams](https://lethain.com/sizing-engineering-teams/)*** (essay). The load-bearing framing on team size vs. initiative scope — the input to chapter 01 §4.3's hero-project trap and §2.4's cost-and-headcount discipline.
- **Will Larson, *[Systems, not heroes](https://lethain.com/systems-not-heroes/)*** (essay from *Elegant Puzzle*). The reference for the 30 % maintenance-reservation discipline in §4.3.
- **Camille Fournier, *[The Manager's Path — Tech Lead chapter](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)*** (duplicated from the load-bearing frames). Names the roadmap as the load-bearing shift from IC to tech-lead altitude.
- **Yonatan Zunger, *[The Friendly Design Doc](https://medium.com/@yonatanzunger/the-friendly-design-doc-2c96b8ba0842)*** (essay). Reference for writing an artefact *to* a specific reviewer at their altitude; the roadmap chapter inherits this discipline from mod-308 chapter 03 and mod-309 chapter 01.
- **Rachel Potvin, *[How to Write a Good Design Document](https://www.designdocsforengineers.com/)*** (2022). Structural-quality discipline the roadmap template §3 shares with a good design doc — summary-first, motivation-before-proposal, alternatives-considered.
- **Sarah Drasner, *[Engineering Management for the Rest of Us](https://engmanagement.dev/)*** (2022). Adjacent framing on planning as a communication artefact rather than an execution artefact. Chapter 3 (*Structuring Your Team*) composes with chapter 01 §2.4's headcount discipline.
- **Julia Ferraioli, *[Engineering strategy resources](https://juliaferraioli.com/engineering-strategy-resources/)*** (curated list). A regularly-updated public list of the essays and papers that make up the current engineering-strategy canon; a useful pointer beyond this module's core.

### Roadmap templates and industry references

- **Basecamp / 37signals, *[Shape Up](https://basecamp.com/shapeup)*** (Ryan Singer, 2019). The alternative to Gantt-chart planning that chapter 01 §1 argues *against as a shape*. The book is free online; useful to read even if you disagree with the specific cycle length, because the *shape appetite → betting table → hill chart* mental model is the industry-facing reference for scoping under uncertainty.
- **Google Cloud, *[Site Reliability Engineering: Managing Roadmap and Project Prioritization](https://sre.google/workbook/managing-load/)*** (essay). Adjacent framing on planning under production-load constraints; the load-bearing reference for chapter 01 §4.3's maintenance-reservation percentage.
- **Etsy Engineering Blog, *[Immutable documentation](https://www.etsy.com/codeascraft/etsys-experiment-with-immutable-documentation)*** (2018). Reference for treating decision-log entries in the roadmap's §11 as first-class engineering artefacts.
- **Marty Cagan, *[INSPIRED: How to Create Tech Products Customers Love](https://svpg.com/inspired-how-to-create-products-customers-love/)*** (Wiley, 2018). Product-side reference for the outcome-vs.-output distinction the roadmap chapter §2.1 requires (the outcome is the state of the world, not the model or the ship).
- **John Doerr, *[Measure What Matters](https://www.whatmatters.com/the-book)*** (Portfolio, 2018). The OKR canon; useful as the corporate-planning framing the ML roadmap outcome integrates with (the roadmap's outcome is usually one leaf on a team-level or org-level OKR tree).

### Adjacent practices this chapter composes with

- **Chip Huyen, *[Designing Machine Learning Systems](https://huyenchip.com/books/)*** (O'Reilly, 2022). Chapters 5 (evaluation) and 10 (monitoring) inform the ML-specific milestone kinds of §2.2 — baseline lock, first-pass, sophisticated model, shadow, canary, full rollout, handoff.
- **Andriy Burkov, *[Machine Learning Engineering](http://www.mlebook.com/)*** (True Positive, 2020). The load-bearing reference on the ML-project lifecycle; the milestone-kinds vocabulary of §2.2 draws from Burkov's chapters on modelling, deployment, and ops.
- **Nithya Sambasivan et al. (Google), *["Everyone wants to do the model work, not the data work": Data Cascades in High-Stakes AI](https://research.google/pubs/pub49953/)*** (CHI 2021). Empirical reference for the "eval-later trap" of §4.1 and for why the M0 baseline-lock milestone matters.

## Chapter 02 — Code review and design review at bar

### Primary references — code review

- **[Google Engineering Practices — Code Review Developer Guide](https://google.github.io/eng-practices/review/)** and its paired *[Reviewer](https://google.github.io/eng-practices/review/reviewer/)* and *[CL Author](https://google.github.io/eng-practices/review/developer/)* handbooks. The canonical published treatment of what a code review is and what a reviewer is looking for. Specific essays this module leans on: *[what to look for](https://google.github.io/eng-practices/review/reviewer/looking-for.html)*, *[speed of code reviews](https://google.github.io/eng-practices/review/reviewer/speed.html)*, *[how to write comments](https://google.github.io/eng-practices/review/reviewer/comments.html)*, *[handling pushback](https://google.github.io/eng-practices/review/reviewer/pushback.html)*, and *[navigating a CL](https://google.github.io/eng-practices/review/reviewer/navigate.html)*.
- **Karl E. Wiegers, *[Peer Reviews in Software: A Practical Guide](https://www.pearson.com/en-us/subject-catalog/p/peer-reviews-in-software-a-practical-guide/P200000000646)*** (Addison-Wesley, 2001). The foundational treatment of the peer review as an engineering discipline. Wiegers' shorter essay *[Humanizing Peer Reviews](https://www.processimpact.com/articles/humanizing_reviews.pdf)* is the reference for chapter 02 §6's review-retro discipline.
- **Michael Fagan, *[Design and Code Inspections to Reduce Errors in Program Development](https://www.mfagan.com/pdfs/ibmfagan.pdf)*** (IBM Systems Journal, 1976). The ancestor of all modern code-review practice — the paper that first framed the review as a structured process with defined roles and a defect-detection focus.
- **[Conventional Comments](https://conventionalcomments.org/)** (community spec). The lightweight discipline for classifying review comments (`issue:`, `suggestion:`, `nit:`, `question:`, `thought:`, `praise:`) that chapter 02 §2 recommends. Free, portable, no tooling required.
- **Microsoft, *[Engineering Fundamentals — Code Reviews](https://microsoft.github.io/code-with-engineering-playbook/code-reviews/)*** (playbook). A more recent, industry-facing sibling of the Google guide; useful contrast on tooling and cadence.
- **GitHub, *[About pull request reviews](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)*** (docs). The current-state platform mechanics the review discipline runs on.
- **SmartBear, *[State of Code Review](https://smartbear.com/state-of-code-review/)*** (annual survey). Longitudinal industry data on review practices — cadence, comment volume, defect-detection rates. Useful for benchmarking your team's practice.

### Primary references — design review

- **Yonatan Zunger, *[The Friendly Design Doc](https://medium.com/@yonatanzunger/the-friendly-design-doc-2c96b8ba0842)*** (duplicated from chapter 01 references). The reference on writing the doc to the reviewer; chapter 02 §5 leans on this for the design-doc contents list.
- **Rachel Potvin, *[How to Write a Good Design Document](https://www.designdocsforengineers.com/)*** (duplicated from chapter 01). Structural-quality discipline; §5's design-doc section list is compatible with Potvin's template.
- **Malte Ubl / Google, *[Design Docs at Google](https://www.industrialempathy.com/posts/design-docs-at-google/)*** (2020). The published corporate treatment of design-doc culture at Google — the reference for §5's *"a design review with no substantive comments is a failure of the review, not a success of the design"* norm.
- **Squarespace Engineering, *[The Anatomy of a Design Doc](https://engineering.squarespace.com/blog/2019/the-anatomy-of-a-design-doc)*** (2019). A widely-copied engineering-org design-doc template that composes cleanly with chapter 02 §5's list.
- **Michael Nygard, *[Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)*** (2011). The ADR (Architecture Decision Record) format — the lighter-weight cousin of the design doc, useful for smaller decisions that come out of a design review as follow-ups.
- **Gergely Orosz, *[Software Engineering RFCs](https://www.pragmaticengineer.com/rfcs-and-design-docs/)*** (Pragmatic Engineer, 2022). The industry-facing reference on RFC culture, cross-referenced from mod-308 chapter 03. Chapter 02 §5's design-doc discipline and mod-308's RFC discipline are two altitudes of the same underlying practice.

### ML-specific review references

- **Papers with Code, *[ML Reproducibility Checklist](https://paperswithcode.com/rc2022)*** (2022). The canonical form for the reproducibility discipline chapter 02 §3.3 checks against. The team's shorter version lives in the standards library.
- **Sculley et al. (Google), *[Hidden Technical Debt in Machine Learning Systems](https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html)*** (NIPS 2015). The reference for why ML PRs need a broader checklist than generic backend PRs — the *CACE* (change anything, change everything) property that makes chapter 02 §3.1's skew discipline load-bearing.
- **Eric Breck et al. (Google), *[The ML Test Score: A Rubric for ML Production Readiness](https://research.google/pubs/pub46555/)*** (2017). The rubric adjacent to chapter 02 §3's checklist; the two compose (a PR that passes the reviewer's checklist and the ML Test Score is a PR at bar).

## Chapter 03 — Mentorship and the standards library

### Mentorship references

- **Camille Fournier, *[The Manager's Path — Mentorship chapter](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)*** (duplicated from the load-bearing frames). The reference for the mentor-vs-manager line (§1) and the sponsorship-vs-mentorship distinction.
- **Lara Hogan, *[Resilient Management](https://larahogan.me/management/)*** and *[Questions for our first 1:1](https://larahogan.me/blog/first-one-on-one-questions/)* (duplicated from the load-bearing frames). The reference for the mentorship-conversation discipline.
- **Lara Hogan, *[The Feedback Equation](https://larahogan.me/blog/feedback-equation/)*** (essay). The reference for delivering hard-mentorship feedback in a way that lands rather than defends (§4).
- **Lily Herman, *[What's the Difference Between a Mentor and a Sponsor?](https://www.themuse.com/advice/whats-the-difference-between-a-mentor-and-a-sponsor)*** (The Muse). The tight external reference on the mentorship-vs-sponsorship distinction (§1).
- **Kim Scott, *[Radical Candor](https://www.radicalcandor.com/)*** (duplicated from the load-bearing frames). The *care personally / challenge directly* frame that mentorship feedback lives in.
- **Will Larson, *[Delegate to the right task](https://lethain.com/using-strategy-for-alignment/)*** (essay). Reference for chapter 03 §3 phase 3's ownership-transfer discipline — delegating the decision, not the outcome.
- **[StaffEng, *Make delegation stick*](https://staffeng.com/guides/writing-engineering-strategy)** (guide). The staff-engineer-side treatment of the same delegation discipline.
- **Sheryl Sandberg, *[Lean In](https://leanin.org/book)*** (Knopf, 2013). The corporate-facing reference on mentorship-and-sponsorship that popularised the "get me a sponsor, not another mentor" framing that Fournier and Herman crystallise.
- **Julie Zhuo, *[The Making of a Manager](https://www.juliezhuo.com/book/manager.html)*** (Portfolio, 2019). Adjacent framing on the first-year-of-management transition — useful as the *other side* of the mentor/manager line (§1) for a senior IC who mentors mid-levels heading toward management.

### Standards library and paved-road references

- **Netflix Technology Blog, *[Full-Cycle Developers at Netflix — Operate What You Build](https://netflixtechblog.com/full-cycle-developers-at-netflix-a08c31f83249)*** (2018). The load-bearing external reference on paved-road-plus-standards as a leverage strategy. Duplicated from mod-308.
- **Matthew Skelton, Manuel Pais, *[Team Topologies](https://teamtopologies.com/book)*** (IT Revolution Press, 2019). The framework for stream-aligned vs. platform vs. enabling vs. complicated-subsystem teams. Chapter 03 §8's composition of standards library + paved road + reviews is an instance of the *enabling* interaction mode applied at senior-IC scale. The [key concepts](https://teamtopologies.com/key-concepts) page is a free summary.
- **[Google Style Guides](https://github.com/google/styleguide)** (open source). The public standards library for programming style — the shape (short, per-topic, opinionated, versioned) that a team's ML-standards library should adopt (§5–6).
- **[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)** (open source). A canonical shape for a per-language standards library that other libraries reference. The template for a standards-library entry that composes cleanly.
- **Etsy Engineering Blog, *[Immutable documentation](https://www.etsy.com/codeascraft/etsys-experiment-with-immutable-documentation)*** (duplicated from chapter 01 references). Example of treating standards as a first-class engineering artefact.
- **Camille Fournier, *[Building a Platform Team](https://skamille.medium.com/building-a-platform-team-e08ea8b23a68)*** (essay, 2020). The platform-team-side treatment of the consumer relationship — the mirror of chapter 03 §8's discussion of how a standard becomes a paved-road primitive.
- **Camille Fournier, *[Bar-raising internal platforms](https://skamille.medium.com/bar-raising-internal-platforms-d5b8a72c8a1e)*** (essay, 2022). The evolution question — when does a standard need to graduate from documentation to enforcement in code? Chapter 03 §8's compounding loop is a specific instance.
- **Evan Bottcher, *[What I Talk About When I Talk About Platforms](https://martinfowler.com/articles/talk-about-platforms.html)*** (martinfowler.com, 2018). The reference for the *platform-as-a-product* discipline the standards library composes with.
- **Google Cloud Vertex AI *[Model Card Toolkit](https://github.com/tensorflow/model-card-toolkit)*** (open source). A concrete example of the "forkable template" pattern chapter 03 §7 recommends for cross-team adoption.
- **[HashiCorp practitioner-facing engineering blog](https://www.hashicorp.com/blog?content=Engineering)** (ongoing). A public example of an org that treats its opinionated defaults (Terraform modules, Consul patterns, etc.) as a standards library other orgs adopt.

### Adjacent practices — writing that scales

- **Google, *[Technical Writing courses](https://developers.google.com/tech-writing)*** (free curriculum). Reference for the writing-quality discipline the standards library depends on — short, opinionated, adoptable.
- **William Zinsser, *[On Writing Well](https://www.harpercollins.com/products/on-writing-well-william-zinsser)*** (Harper Perennial, 1976 / 30th anniversary edition 2006). The classical text on non-fiction writing craft; the reference for the "cite more, write shorter" discipline of a good library entry.

## Chapter 04 — Rubric-based ML interviews and calibration

### Empirical foundations

- **Frank L. Schmidt & John E. Hunter, *[The Validity and Utility of Selection Methods in Personnel Psychology: Practical and Theoretical Implications of 85 Years of Research Findings](https://mavweb.mnsu.edu/howard/Schmidt%20and%20Hunter%201998%20Validity%20and%20Utility%20Psychological%20Bulletin.pdf)*** (Psychological Bulletin, 1998). The meta-analysis behind the "structured interviews have ~0.51 predictive validity vs. unstructured at ~0.20" claim in chapter 04 §1. The single most-cited empirical reference on interviewing methodology.
- **Julia Levashina, Christopher J. Hartwell, Frederick P. Morgeson, Michael A. Campion, *[The Structured Employment Interview: Narrative and Quantitative Review of the Research Literature](https://onlinelibrary.wiley.com/doi/abs/10.1111/peps.12052)*** (Personnel Psychology, 2014). The 21st-century update to Schmidt & Hunter, replicating the structured-interview advantage and documenting the specific mechanisms (rubrics, question standardisation, panel consensus) that drive it.
- **Amber N. Culbertson, Kenneth D. Locke, Peggy J. Yeager, *[Structured Interviewing](https://journals.aom.org/doi/10.5465/annals.2013.0016)*** (Academy of Management Annals, 2016). Review of structured-interview methodology with a focus on bias reduction.
- **Patricia C. Smith & L. M. Kendall, *[Retranslation of expectations: an approach to the construction of unambiguous anchors for rating scales](https://doi.apa.org/doiLanding?doi=10.1037%2Fh0047343)*** (Journal of Applied Psychology, 1963). The foundational paper on behaviourally-anchored rating scales (BARS) — the format the four-part rubric of §2 uses.
- **Michael A. McDaniel et al., *[The Validity of Employment Interviews: A Comprehensive Review and Meta-Analysis](https://psycnet.apa.org/record/1994-40571-001)*** (Journal of Applied Psychology, 1994). Earlier meta-analysis that Schmidt & Hunter's work extends; useful for the specific comparison of *structured behavioural* vs. *structured situational* interview variants.

### Corporate references

- **Laszlo Bock, *[Work Rules!](https://www.workrules.io/)*** (Twelve, 2015). Bock's chapter 5 (*Why We Hire Only the Best*) is the corporate-scale companion to the Schmidt & Hunter research — Google's public case study on abandoning brainteaser interviews and moving to structured, rubric-driven loops. Chapter 6 covers score-against-outcome calibration.
- **[Google re:Work — Guide to Structured Interviewing](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/introduction/)** (public guide). The current Google corporate reference. Specific sub-guides this module leans on: *[Design your questions](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/design-your-questions/)*, *[Use a scoring rubric](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/use-a-scoring-rubric/)*, and *[Reduce interviewer bias](https://rework.withgoogle.com/guides/hire-use-structured-interviewing/steps/reduce-interviewer-bias/)*.
- **Aline Lerner (interviewing.io), *[Anatomy of a Great Interview](https://interviewing.io/blog/hiring-loops-vs-interviews)*** and the *[Machine Learning Engineer interview guide](https://interviewing.io/guides/machine-learning-engineer-interview)* (ongoing). The industry-facing reference on loop mechanics, question design, and calibration; one of the most rigorous public sources on ML-interview practice.
- **[interviewing.io — How to Shadow Interviews](https://interviewing.io/blog/how-to-shadow-interviews)** (blog). The load-bearing reference for the shadow-and-lead calibration mechanism of §6 and the mentee-training arc of §8.
- **Gayle Laakmann McDowell, *[Cracking the Coding Interview](https://www.crackingthecodinginterview.com/)*** (CareerCup, 2015). The candidate-facing canonical reference. Useful for the interviewer as a source of *what your candidates prepared against* — knowing the preparation landscape sharpens the question-design discipline of §4.
- **Rich Moy et al., *[Textio, Interviewer.io, Karat](https://karat.com/blog/)* case studies** (various). Industry-facing case studies on interview-loop design at scale from vendors and platforms who see many loops.

### Bias, fairness, and legal defensibility

- **[SIOP — Society for Industrial and Organizational Psychology, *Principles for the Validation and Use of Personnel Selection Procedures*](https://www.siop.org/Portals/84/PDFs/Principles.pdf)** (5th edition, 2018). The canonical I/O-psychology reference behind chapter 04 §7's behaviourally-anchored rating-scale discipline.
- **[EEOC — Section 6: Lawful Employment Decisions](https://www.eeoc.gov/laws/guidance/section-6-lawful-employment-decisions)** (guidance). U.S. federal reference for the discipline of contemporaneous, auditable interview records. Consult your legal team's guidance for your jurisdiction; the 1-year retention window mentioned in chapter 04 §7 is the private-employer default in the U.S. and may differ elsewhere.
- **[EEOC — Uniform Guidelines on Employee Selection Procedures (29 CFR Part 1607)](https://www.eeoc.gov/uniform-guidelines-employee-selection-procedures)** (federal regulation). The regulatory framework that requires selection procedures (including interviews) to be job-related and defensible against disparate-impact analysis.
- **U.S. Department of Labor, *[Federal Contractor OFCCP guidance](https://www.dol.gov/agencies/ofccp/faqs/employee-selection)*** (guidance). Reference for federal contractors' more-stringent record-retention window.
- **Daniel Kahneman, Olivier Sibony, Cass R. Sunstein, *[Noise: A Flaw in Human Judgment](https://us.macmillan.com/books/9780316451383/noise/)*** (Little, Brown Spark, 2021). The load-bearing conceptual reference for the calibration discipline of §6 — *noise* (interviewer-to-interviewer variance on the same candidate) is one of the most-studied and most-underinvested-in interviewing practices. The *decision hygiene* chapter is the specific reference.
- **Daniel Kahneman, *[Thinking, Fast and Slow](https://us.macmillan.com/books/9780374533557/thinkingfastandslow)*** (Farrar, Straus & Giroux, 2011). Kahneman's earlier reference on cognitive biases; useful background for understanding *why* structured interviewing reduces bias (short-circuiting the fast-thinking heuristics that drive unstructured judgement).
- **Iris Bohnet, *[What Works: Gender Equality by Design](https://www.hup.harvard.edu/books/9780674089037)*** (Harvard University Press, 2016). Reference on the specific mechanisms (structured interviews, standardised rubrics, blinded review) that reduce demographic bias in evaluation processes.

### Adjacent practices — hiring at senior levels

- **Adam Grant, *[Give and Take](https://www.adamgrant.net/book/give-and-take/)*** (Viking, 2013). Reference for the *reciprocity styles* framing that shows up in senior-hire collaboration signal (§2.4's *collaboration, mentorship, and cross-team* dimension).
- **Peter Cappelli, *[Why Good People Can't Get Jobs](https://www.pennpress.org/9781613630136/why-good-people-cant-get-jobs/)*** (Wharton Digital Press, 2012). The reference on how loops that over-specify requirements or use unstructured methods systematically fail to hire even strong candidates — the empirical case for keeping the rubric focused on job-representative signals.
- **Katie Womersley, *[Why we let our engineers spend all their time on hiring for months](https://leaddev.com/hiring/why-we-let-our-engineers-spend-all-their-time-hiring-for-months)*** (LeadDev, 2020). Reference for the loop-lead role and the org-investment cost of running a rubric-driven loop well.
- **Cate Huston, *[Sr Engineer Interview Loops](https://catehuston.com/blog/2019/12/09/senior-engineer-interview-loop/)*** (blog). A practitioner-side treatment of specifically senior-engineer loop design, adjacent to chapter 04 §3's four-round shape.

## Cross-references inside this module

- Chapter 01 (`01-roadmap-and-scoping.md`) — the roadmap as decision artefact; four questions (outcome, sequence, decision points, dependencies + cost + cutlines); ML-specific milestone kinds; template; four scoping traps; pre-commit review; keeping the roadmap alive.
- Chapter 02 (`02-code-review-and-design-review-at-bar.md`) — three purposes of review; block-vs-nudge classification with Conventional Comments; ML-specific reviewer checklist (skew, eval, reproducibility, cost, rollout, model card, review composition); ML block-vs-nudge rules of thumb; design review at bar; mentoring a mid-level to review; escalation; five failure modes.
- Chapter 03 (`03-mentorship-and-standards-library.md`) — mentorship vs. sponsorship vs. supervision; who to mentor; four-phase arc; leading indicators; standards library; what goes in the library; writing standards other teams adopt; composition with paved road and reviews; five failure modes.
- Chapter 04 (`04-rubric-based-ml-interviews.md`) — empirical case for structured interviews; four-part ML rubric with BARS anchors; four-round loop shape; question design; debrief mechanics; calibration; bias / fairness / legal-defensibility; mentee training; five failure modes.

## Cross-references to other modules

- [mod-301 chapter 01 (what changes at senior)](../mod-301-senior-ml-role-scope/01-what-changes-at-senior.md) — the L20-vs-L30 shift the outcome-first framing (chapter 01 §2.1) and the modelling-vs-data-vs-evaluation reframe (chapter 04 §2.1 anchor 5) depend on.
- [mod-301 chapter 03 (verbs that define seniority)](../mod-301-senior-ml-role-scope/03-verbs-that-define-seniority.md) — the *own / drive / design / influence* vocabulary that anchors chapter 04 §2.3's *judgement* dimension.
- [mod-301 chapter 05 (self-assessment and growth plan)](../mod-301-senior-ml-role-scope/05-self-assessment-and-growth-plan.md) — the vocabulary a mentee brings to chapter 03 §3 phase 1's framing conversation.
- [mod-302](../mod-302-ml-systems-architecture/) — the training/serving-skew and architecture-decision vocabulary chapter 02 §3.1 checks against and chapter 04 §2.2 tests for.
- [mod-303](../mod-303-advanced-modeling/) — the advanced-modelling vocabulary chapter 04 §2.1's modelling-depth rubric anchors reference.
- [mod-304 chapter 01–03 (LLM integration)](../mod-304-production-llm-integration/) — the prompt-vs-fine-tune decision point chapter 01 §2.3 and the LLM-augmented question archetype chapter 04 §4 test for.
- [mod-305 chapter 01 (offline eval harness shape)](../mod-305-advanced-evaluation/01-offline-eval-harness-shape.md) — the eval-harness discipline chapter 02 §3.2 checks against and chapter 04 §2.1–2.2 test for.
- [mod-305 chapter 02 (slice and adversarial guardrails)](../mod-305-advanced-evaluation/02-slice-and-adversarial-guardrails.md) — the guardrail-slice coverage chapter 02 §4's block rule requires.
- [mod-305 chapter 04 (ML Test Score / production-readiness)](../mod-305-advanced-evaluation/04-ml-test-score-production-readiness.md) — the rubric-adjacent-to-review-checklist tool.
- [mod-306 chapter 01 (experimentation ramp plans)](../mod-306-experimentation-at-scale/) — the rollout-gate vocabulary chapter 02 §3.5 and chapter 04 §2.2 depend on; the ramp-step decision-point vocabulary chapter 01 §2.3 uses.
- [mod-307 chapter 01 (SRE fundamentals for ML)](../mod-307-ml-reliability-slos/01-sre-fundamentals-for-ml.md) — the SLO document the roadmap chapter 01 §5 dependencies reference and the review chapter 02 §3.5 checks against.
- [mod-307 chapter 03 (cost and quota)](../mod-307-ml-reliability-slos/) — the cost-and-budget discipline chapter 01 §2.4 and chapter 02 §3.4 lean on.
- [mod-307 chapter 04 (incident response)](../mod-307-ml-reliability-slos/04-ml-incident-response-and-postmortems.md) — the on-call-lead skill loop of chapter 03 §3 phase 2.
- [mod-307 chapter 05 (retraining as a first-class deploy)](../mod-307-ml-reliability-slos/05-retraining-as-a-first-class-deploy.md) — the retraining-cadence decision point chapter 01 §2.2 M7 (handoff) references.
- [mod-308 chapter 01 (paved-road consumption)](../mod-308-platform-collaboration/01-paved-road-consumption.md) — the paved-road-inventory chapter 01 §5 dependencies cite and chapter 03 §8's compounding loop lands into.
- [mod-308 chapter 03 (contribute-back RFCs)](../mod-308-platform-collaboration/03-contribute-back-rfcs.md) — the RFC discipline chapter 03 §3 phase 2's RFC-loop skill uses; the *sponsor conversation* pattern chapter 01 §5 inherits.
- [mod-308 chapter 04 (hand-off contracts to specialists)](../mod-308-platform-collaboration/04-handoff-contracts-to-specialists.md) — the six-section hand-off contract the roadmap chapter 01 §5 dependencies borrow shape from.
- [mod-309 chapter 01 (model cards and data statements)](../mod-309-responsible-ai-governance/01-model-cards-and-data-statements.md) — the model-card artefact chapter 02 §3.6 checks against and chapter 01 §2.2 M7 (handoff) references.
- [mod-309 chapter 03 (fairness, robustness, safety cadence)](../mod-309-responsible-ai-governance/03-fairness-robustness-safety-review-cadence.md) — the review-cadence policy chapter 01 §2.2 M7 (handoff) references.

## Peer specialist tracks (referenced from chapter 03 §7 and chapter 04 §8)

Consult each track's own README for the specialist-side delegation vocabulary. The interview loop of chapter 04 also uses these tracks as *cross-loop referral targets* — a candidate who is a better fit for a specialist track than a generalist Senior ML Engineer role should be referred, not rejected.

- **`ai-eval-engineer-learning`** / **`model-evaluation-engineer-learning`** — the peer tracks for evaluation-platform and LLM-eval depth; a candidate whose modelling-depth signal skews heavily toward evaluation depth may be a better fit here.
- **`ai-governance-analyst-learning`** — the peer track for governance and compliance depth; a candidate whose collaboration signal skews cross-functional and regulated-industry may be a better fit here.
- **`ai-infra-mlops-learning`** — the peer track for MLOps / registry / paved-road-platform depth; a candidate whose systems-reasoning signal skews toward the platform side may be a better fit here.
- **`ai-infra-ml-platform-learning`** — the peer track for platform-engineering-of-ML depth; adjacent to MLOps but with a stronger platform-team framing.
- **`ai-infra-security-learning`** — the peer track for ML/AI security and adversarial-ML depth.
- **`training-pipeline-engineer-learning`** — the peer track for distributed-training / MFU / parallelism depth; a candidate whose modelling depth skews toward training-systems performance may be a better fit here.

<!-- needs-research: verify the peer-track repo URLs once the ai-infra-curriculum org page is finalised. The track slugs above are stable but the exact URL prefix is org-dependent. -->
