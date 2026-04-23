# Division of Labor: Framework Designers vs Framework Users

## What

GRACE is built on an economic asymmetry: the person who *designs* a prompt/contract/testing framework for LLMs is doing fundamentally different work from the person who *uses* that framework to ship product code. GRACE's author explicitly argues that these two roles should remain separate because each requires a dedicated time budget that individual working developers cannot afford to allocate simultaneously [author-claim] (post 3804).

The claim rests on two measured cost anchors that the author surfaces from his own experimentation:

1. **One framework-section validation iteration = 4–8 hours.** Verifying whether a single contract section actually works across the full test suite of a project takes roughly four to eight hours per iteration (post 3804) [empirical].
2. **One research paper, properly digested = ~1 person-day.** Reading a current LLM-internals paper and extracting something actually applicable from it takes "at minimum one person-day per publication" (post 3804) [empirical].

Users, by contrast, are expected to **take the experimental experience** (the conclusions, the heuristics, the distilled rules) from the designer rather than re-deriving the framework themselves — and then adapt it to their project. Adaptation is always required; taking the framework verbatim is not the intended workflow (post 3804) [author-claim].

## Why

The division exists because the two activities compete for the same scarce resource: continuous, undistracted attention on LLM internals and agent behavior.

- **Working developers have no slack for this.** "An ordinary developer simply does not have time to spend a long time poking at agents and meticulously picking all sorts of injections of documentation into code" (post 3804) [author-claim]. Their day job is the product, not the framework.
- **Framework validation compounds.** Because a single contract-section test cycle is 4–8 hours and many sections must be tested against many tests, the full framework-level experimentation budget is measured in person-weeks to person-months — not something a product team can absorb while also meeting delivery commitments (post 3804) [author-claim].
- **Research velocity is non-trivial.** Staying current on "how the LLM works inside" — the material that decides *which* contract sections matter and in what form — requires reading papers at ~1 person-day per publication. The author has built GRACE explicitly around recent research findings (e.g., ShortenDoc, in-source-doc benchmarks, bug-fix attention studies), each of which represents this kind of digestion cost (post 3892, post 3889, post 3890) [research-backed].
- **Without the split, both sides break.** A developer trying to be their own framework designer either short-changes the product (missed deadlines) or short-changes the framework (ships rules that weren't actually validated). Neither outcome is acceptable for 100%-AI-generated applications at Enterprise scale, which is GRACE's target (most_important.md §"Формальное введение", post 3804) [author-claim].

The same reasoning undergirds why GRACE is stabilized over months with ~200+ practitioners as a single centralized methodology rather than distributed ad-hoc (most_important.md §"Формальное введение") [empirical] — centralization is how the designer cost gets amortized across many users.

## How

### The designer's job

The framework designer runs experiments that users do not:

- **Section-by-section contract validation** — each contract field (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS, RATIONALE/AAG) is tested for whether it holds up across a full test suite; this is the 4–8-hour-per-section cost (post 3804) [empirical]. The canonical contract fields came out of this kind of iteration (post 3878, post 3889) [research-backed].
- **Logit and blind-gloss verification of compression.** The author's own method for accepting any new compressed form: (1) check that a logit prediction still triggers correctly on the compressed phrase; (2) ask a blind GPT to describe what the compressed phrase means and see if it recovers the intent (post 3892) [empirical]. This is inherently experimental work and requires tooling the average developer will not have.
- **Keeping pace with research.** GRACE incorporates findings from specific papers (arxiv 2601.16661v1 on in-source-doc effects, ACM 10.1145/3735636 on ShortenDoc compression, ResearchGate 400340807 on bug-fix attention, arxiv 2105.11115 on Dyck-language support, ACM 10.1145/3689484.3690738 on CodeT5+ contract recovery). Each one is a person-day of reading plus additional validation time before integration (posts 3889, 3890, 3892, 3948, contracts_as_anchor.md §"Научная опора") [research-backed].
- **Cross-model and cross-vendor validation.** The author observes JSON-degradation effects on Qwen as well as OpenAI-family models, treating the phenomenon as a universal GPT response rather than a per-vendor artifact (xml_mapping_anchors.md §"XML vs JSON") [empirical]. This kind of multi-model validation is another thing framework users cannot afford to redo.

### The user's job

Users take the framework and adapt it:

- **Take the experiment experience, not just the framework.** The load-bearing thing a user is supposed to carry away is the set of conclusions — the "which heuristics actually work and why" — not a verbatim copy of the rules (post 3804) [author-claim].
- **Adapt to the project's language, framework, and context.** Adaptation is always needed (post 3804) [author-claim]. Examples from the GRACE corpus:
  - Vlad adapted GRACE to a Next.js + shadcn + Supabase CRM for a security-firm consortium, switching between Gemini Flash/Pro, Claude Opus 4.6, GLM-5, and Kimi K2.5 models (practical_applications.md §"Пример CRM") [empirical].
  - Mikhail Evdokimov adapted GRACE's graph + semantic-squeeze approach from code into a document-RAG system at a bank (post 3839) [empirical].
  - Kirill Shutov studied the GRACE posts for two weeks, consolidated them into a prompt, and then ran Kilo Code with GPT-5.4(xh) + GPT-5.4(m) against a 132k-LOC codebase (post 3620) [empirical]. That is classic adaptation: absorb the experience, convert it to a project-specific prompt, then execute.
- **Manage agents rather than write all the code.** Many GRACE users are "engineers managing design/coding/testing agents" rather than engineers coding themselves (post 3837) [empirical] — i.e., they specialize in the user-side job and do not attempt to also be the designer.

### The "uncritical copying" failure mode

Taking the framework without the experiment experience is the failure mode the author warns against. Two concrete examples from the sources:

- Trying to reuse classical TDD patterns with AI: the design assumption (tests help the model understand the task) was already disproved in research the framework designer tracks, but a user who only copies rules will not know that and will hurt their agent (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'") [author-claim].
- Building a custom MCP for code documentation: a user can copy the idea "expose contracts via MCP" but without the designer's context (Kimi paper on MCP training distribution, FIM-training-driven "find-modules" bias) they will miss why the agent will ignore it (posts 3820, 3821, 3834) [research-backed].

### Why this is economically stable

The author draws an analogy to the MCS / ecosystem model from classical IT vendors: a small number of specialists do the deep R&D and reference cases, and partners/users do the per-project adaptation at scale. Frameworks like GRACE are the small-specialist output; individual projects at 100k–300k LOC are the partner-scale adaptation (post 3708, post 3837, post 3838) [author-claim]. This matches the sources' observation that (a) large vendors deliberately do NOT build full-pipeline frameworks in this segment (post 3708) [author-claim], leaving room for designers like the GRACE author, and (b) users are reaching 100k–500k LOC projects because they are NOT spending their day re-validating contract sections (post 3837) [empirical].

## Evidence

- **Primary**: post 3804. Defines both cost anchors (4–8 hours per contract-section iteration; ~1 person-day per paper) and the "framework designers vs framework users" split. Explicitly states that users must carry the experiment experience and adapt; taking the framework wholesale is not the intended workflow.
- **Supporting — designer cost**:
  - post 3892: ShortenDoc paper; the author's additional validation protocol (logit check + blind-gloss test) — an example of the kind of work that sits behind a single GRACE rule.
  - post 3889: in-source-doc benchmarks (arxiv 2601.16661v1) — the research that justifies PURPOSE, short descriptions, English-only, and adjacency. Each claim rests on a tested effect.
  - post 3890: ResearchGate 400340807 on architectural-intent comments yielding up to 3× bug-fix accuracy — the research that justifies RATIONALE/AAG.
  - post 3948: Yao et al. (arxiv 2105.11115) Dyck-language support — underpins the XML/START-END choice.
  - contracts_as_anchor.md §"Научная опора" (ACM 10.1145/3689484.3690738): CodeT5+ contract recovery at 90–97% quality — underpins legacy-contract recovery practice.
- **Supporting — user adaptation**:
  - post 3620 (Kirill, 132k prod LOC + 50k test LOC): classic user adaptation — absorb posts, build a custom prompt, execute on a real project.
  - practical_applications.md §"Пример CRM" (Vlad): adaptation across multiple LLMs on a vertical-CRM project.
  - post 3839 (Mikhail Evdokimov, O!Bank): adaptation of GRACE's graph + semantic-squeeze pattern to document RAG.
  - post 3838 (Alexey, 300k+ LOC): shows the ceiling a well-adapted user can reach.
  - post 3837: describes the emerging user archetype — engineers who do not code the target framework themselves but manage agent swarms.
- **Supporting — ecosystem analog**:
  - post 3708: explains why large vendors do not build full-pipeline frameworks in this segment, leaving the designer niche for specialists.
- **Supporting — why users cannot self-serve this work**:
  - most_important.md §"Формальное введение": GRACE stabilized over months with 200+ practitioners; only empirically-working parts were kept — this is the amortization of designer cost across a community.
  - testing_and_autonomous_agents.md §"Почему обычные автотесты": the kind of counter-intuitive finding (classical TDD harms AI agents) that a user will not discover on their own in a reasonable time.

## See also

- [../00-foundations/what-is-grace.md](../00-foundations/what-is-grace.md) — positioning as "RUP for the AI era," which frames why a methodology-level designer role exists at all.
- [../12-economics-strategy/vendor-strategy.md](./vendor-strategy.md) — why the big vendors do not compete in this designer niche.
- [../12-economics-strategy/project-size-trends.md](./project-size-trends.md) — the user-side scaling observation (100k → 400–500k LOC).
- [../11-case-studies/kirill-132k-loc.md](../11-case-studies/kirill-132k-loc.md) — concrete user-side adaptation case.
- [../11-case-studies/vlad-crm-and-contracts.md](../11-case-studies/vlad-crm-and-contracts.md) — user-side adaptation across multiple LLMs.
- [../11-case-studies/alexey-300k-loc.md](../11-case-studies/alexey-300k-loc.md) — ceiling of a well-adapted user.
- [../11-case-studies/mikhail-bank-rag.md](../11-case-studies/mikhail-bank-rag.md) — cross-domain user adaptation (code → documents).
- [../14-research-references/annotated-bibliography.md](../14-research-references/annotated-bibliography.md) — the specific papers whose per-publication digestion cost is part of the designer budget.
- [../03-contracts/contract-fields.md](../03-contracts/contract-fields.md) — the contract-section artifact that is actually being validated in the 4–8-hour iteration cycle.
- [../04-markup-system/markup-compression.md](../04-markup-system/markup-compression.md) — example of the empirical heuristics the designer derives and users receive as already-validated rules.
