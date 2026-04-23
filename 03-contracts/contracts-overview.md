# Contracts Overview — Why Contracts Are Central to GRACE

## What

In GRACE, a **contract** is an AI-oriented micro-specification attached to code — typically placed before a function, method, module, or class — that declares intent (PURPOSE), inputs/outputs, keywords, and cross-links. Contracts are NOT the formal pre/post/invariant predicates of classical Design-by-Contract; they are compressed semantic descriptors tuned to how LLMs actually read and modify code. [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

Contracts sit at the third level of GRACE's hierarchy — below the architectural graph and above the code itself — and are the primary artifact a so-called "Programmer 2.0" interacts with during AI-generated software development. [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование"; most_important.md §"IDE 2.0")

## Why

### Contracts as a semantic shield during modification

The central purpose of a contract is to act as a **semantic shield** for the AI when it modifies existing code. During every edit the AI-agent cross-checks its proposed change against the contract before committing to it. [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

Source text (direct paraphrase):

> "Contracts — the SEMANTIC SHIELD of the AI from errors during code modification; it will always verify its edit against the contracts." [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

Without this shield, AI agents drift from original intent during modification and introduce silent regressions — a failure mode the author reports observing repeatedly in student projects where 5,000–10,000 LOC of interconnected logic is the collapse threshold. [empirical] (post 3654)

### Written by the AI-author, for the AI-reader — not for humans

A defining property of GRACE contracts: **they are written by the AI that authored the code, and the primary consumer is another AI (or the same agent on a later turn)**. Humans may or may not need them; the AI needs them critically. [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

Direct translation of the source:

> "Contracts are something like a specification for functions. Writing contracts should be done by the AI-author of the code. You may not need them, but the AI critically does." [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

This is a load-bearing inversion of the traditional documentation audience model. It explains why GRACE rejects classical Docstrings-for-humans conventions (too verbose, wrong emphasis) and instead favors compressed "wenyan-prompting" markers such as KEYWORDS and LINKS. [author-claim] (post 3819; see `wenyan-prompting.md` for mechanics)

### Programmer 2.0 works with contracts, not code

In GRACE's forward-looking IDE model, the primary working object for the engineer is the contract layer — not the code:

> "Programmer 2.0 will work more with contracts than with code." [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

The IDE-2.0 vision (prototyped by colleague Vlad at the author's request) places a **semantic graph of anchor tokens** at the center of the interface, with specs at the top, module contracts in the middle, and generated code at the bottom. A non-programmer needs only the concepts of *class*, *function*, and *variable* to operate at the contract level; code generation becomes a derived step. [author-claim] (most_important.md §"IDE 2.0")

### Graph + module contracts + function contracts replace traditional documentation

Once the graph, module contracts, and function contracts are in place, classical separate documentation becomes unnecessary — architecture and business model are **embedded directly in the code** in a form the AI natively parses. [author-claim] (most_important.md §"Top-down логика и переход к контрактам")

Direct translation:

> "For agents that later edit such code, the presence of a graph, module contract, and function contracts replaces ordinary documentation 100%. It simply becomes unnecessary — both architecture and business model are already embedded directly in the code in a form the AI understands." [author-claim] (most_important.md §"Top-down логика и переход к контрактам")

This is a structural consequence of placing top-down intent into the code itself, not a stylistic preference. Side documentation drifts out of sync with code; documentation that lives inside the code cannot drift without being visible on the next read. [author-claim] (post 3889 shows the degradation cost of out-of-sync side docs — up to −95% on tasks)

### Why FIM training makes contracts-in-code unignorable

The most training-grounded argument for why contracts must live INSIDE the code — not in a parallel system — is rooted in how modern LLMs were trained:

1. **Base coder training**: ~80% code, ~20% natural language; the 20% is task-understanding text, NOT documentation-as-reference. [research-backed] (post 3820)
2. **FIM (Fill-In-the-Middle)**: a "complete and total ban on documentation for code"; training objective is "how correct is this fragment versus the rest?". [research-backed] (post 3820)
3. **SFT**: task-code pairs where the task is 2–3 paragraphs and the code is 200–300 lines; strict Tools-chain training on popular agents/MCPs with a hard ban on DIY tooling. [research-backed] (post 3820)
4. **RL (SWE-Bench-style)**: isolation-test-driven bug fixing; no documentation. [research-backed] (post 3820)

Direct translation of the key implication:

> "Base training of most modern models as FIM practically did not use code documentation as context. Therefore the AI has a huge tendency to lean on code, not documentation." [research-backed] (most_important.md §"Top-down логика и переход к контрактам"; post 3820)

The consequence is twofold:

- **External documentation systems (MCP for docs, graph DBs of docs, sidecar .md files) fight the training distribution.** Agents either ignore them entirely or — if forced — hallucinate "plausibly" because they were never trained to consume docs from those locations. [author-claim] (posts 3820, 3821)
- **Contracts placed IN the code cannot be ignored by the AI.** Because FIM and base coder training pushed the model to trust code over everything else, and contracts are now syntactically part of the code (as comments or language-native documentation directives immediately preceding the declaration), the AI reads them with the same priority as the code itself. [author-claim] (most_important.md §"Top-down логика и переход к контрактам"; post 3820)

This is why GRACE insists that contracts be inside the source file, immediately adjacent to the declaration they describe — not in a separate docs system, MCP, or database. Adjacency is not a convenience; it is a training-distribution alignment choice. [research-backed] (post 3889 proves that separating docs from code by file-read or MCP-hop causes measured degradation)

## How

### Contracts as one layer of the three-layer hierarchy

GRACE enforces a strict top-down progression:

1. **Architectural graph** — AI models the app as a graph of main classes/modules; business-process modeling is mapped into the same graph.
2. **Module contract** — derived from the graph; expresses the detailed intent of each module, including an architectural-rationale section (RATIONALE/AAG).
3. **Function contract + code** — the function-level contract is a **local refinement** of the overall app intent at that call site, not AI improvisation. Code generation follows from the contract. [author-claim] (most_important.md §"Top-down логика и переход к контрактам")

Direct translation:

> "The contract of a function is, in essence, a refinement of the app's overall intent at this particular place — not spontaneous AI creativity." [author-claim] (most_important.md §"Top-down логика и переход к контрактам")

Assistance is required: left alone, an AI jumps straight to code generation. GRACE mandates that agents be guided through each phase so that the contract genuinely is derived from graph-level intent rather than retrofitted after the code exists. [author-claim] (most_important.md §"Top-down логика и переход к контрактам")

### Canonical contract fields

Contracts in GRACE use a small, canonical set of fields:

- **PURPOSE** — goal/intent of the function or module. Load-bearing; demonstrated critical by benchmark (post 3889 shows agents can skip DESCRIPTION but not PURPOSE). [research-backed] (post 3889)
- **INPUTS / OUTPUTS** — typed parameter and return description. [author-claim] (post 3878)
- **KEYWORDS** — compressed architectural-pattern / task-classification tags acting as RAG hooks and wenyan-prompting anchors. [author-claim] (posts 3819, 3878)
- **LINKS** — cross-module references acting as a Call Graph proxy. [author-claim] (post 3732)
- **RATIONALE / AAG** — architectural intent, present in module contracts; the research citation for this field shows architectural-intent comments improve automated bug-fix accuracy up to 3×. [research-backed] (post 3890)

Full field specification is in `contract-fields.md`; placement rules and a working Python example live in `language-specific/python.md`.

### Why contracts do not eliminate the need for skilled prompting

Testing whether a single contract section works correctly across all tests takes **4–8 hours per iteration**. This is why the author makes a point that individual developers rarely have time to build complete frameworks plus keep up with LLM research — the efficient division is between framework designers (who run these experiments) and framework users (who adapt and apply). Contracts themselves are cheap to write but expensive to *validate* as effective. [author-claim] (post 3804)

### Contracts complement, rather than replace, other GRACE layers

A CRM case study (colleague Vlad, Next.js + shadcn + Supabase, 5 security companies with 1000+ personnel per shift) demonstrated that well-applied semantic markup + specs + contracts made the app largely **model-agnostic** — the developer "couldn't feel a difference" between Gemini Flash/Pro, Claude Opus 4.6, GLM-5, and Kimi K2.5. Model sensitivity is a symptom of a weak pipeline, not of the model. [empirical] (practical_applications.md §"Пример CRM")

A separate Vlad project (contracts-focused: **282 contracts, 1049 autotests**) showed the complementary failure mode: **contracts applied correctly, logging absent**. The agent itself reflected to the author that "tests without logs are like vodka without beer." Contracts are necessary but not sufficient — logging (see `05-logging-ldd/`) is the paired discipline. [empirical] (post 3693)

### Contracts for legacy vs native AI-generated code

For legacy code, contract recovery is feasible: a Google CodeT5+ fine-tune achieved **90–97% contract-recovery quality** on legacy Java using JML spec (dl.acm 10.1145/3689484.3690738). The author reports comparable empirical success on his own legacy projects, evaluated via LLM-reviewer rather than formal tests. [research-backed] (contracts_as_anchor.md §"Научная опора")

Anton's contract-only MCP pattern is particularly useful for legacy: the MCP returns only contract identifiers + declarations + implementation line ranges, hiding the "rough human code" from the agent to prevent it from tiling bad patterns into new code. For AI-native code, the author recommends the opposite — **greedy reading** of full modules — because the native code itself serves as few-shot context that pins the agent to current framework versions. [empirical] (post 3622)

See `legacy-contract-recovery.md` for the full legacy flow.

### What contracts are NOT

Contracts are not a revival of Eiffel-style DbC. Classical DbC uses Hoare-logic pre/post/invariant triples `{P}C{Q}` for formal verification. LLM vendors **burned out** strict test-like conditions from training because they caused overfitting — the model began "fitting to the test" instead of generalizing. [research-backed] (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC")

Direct translation of the key passage:

> "In the case of LLMs, it [strict pre/post conditions] turned out to be bad practice. If you show the AI strict test-like conditions during training, it starts fitting solutions specifically to them — and they don't generalize. Therefore the LLM vendors factually burned the test-like examples out of training with napalm." [research-backed] (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC")

GRACE contracts are closer in spirit to Python docstrings and Spec-Driven Development micro-specs, not validators. Commonalities with classical DbC: **location** (inside/next to the code) and **intent** (requirements on the algorithm). Form is fundamentally different and tuned to what the target model's SFT already contains. [author-claim] (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC")

See `ai-contracts-vs-dbc.md` for the full contrast.

### Do not invent your own contract format

Equally important, contract syntax must NOT be freely designed. GRACE extracts the target model's own contract patterns from its SFT distribution and builds on those:

> "Want predictability from the AI? Sit on the patterns that the AI itself already knows." [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

This means the "right" contract form for Claude differs from the "right" form for Qwen, and both differ from the right form for GPT-4 — the author's canonical GRACE templates were derived by probing SFT outputs and keeping what the model spontaneously produces, not by designing a clean notation from scratch. [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

## Evidence

Primary sources:

- `contracts_as_anchor.md §"Почему контрактное программирование становится центральным"` — the semantic-shield argument; the "Programmer 2.0 works with contracts not code" claim; the instruction to extract contracts from target LLM's own patterns rather than invent. [author-claim]
- `most_important.md §"Top-down логика и переход к контрактам"` — the claim that graph + module contract + function contracts replace traditional documentation; the FIM-training argument that base pretraining practically did not use documentation, pushing AI to trust code over docs. [author-claim / research-backed]
- `most_important.md §"IDE 2.0"` — the Programmer 2.0 vision with the contract layer at the center of the IDE. [author-claim]

Training-pipeline grounding:

- Post 3820 — four-phase LLM coding-training walkthrough (Base → FIM → SFT → RL) with documentation banned from FIM and RL stages. [research-backed]
- Post 3821 — the complementary discussion with Sergei Muravyev: external doc MCPs misdefine the agent's task; the real task is "find modules", not "understand class"; FIM training biases agents to load whole modules. [author-claim / research-backed]

Research backing for in-source placement:

- Post 3889 — benchmark work on in-source documentation (arxiv 2601.16661v1): PURPOSE critical, shorter-is-better, English-better-than-other-languages, adjacency-matters; effect sizes span −95% degradation to +435% improvement. [research-backed]
- Post 3890 — architectural-intent comments improve automated bug-fix accuracy up to 3× (researchgate 400340807); attention analysis shows the agent attends to intent tokens. [research-backed]
- Post 3892 — ShortenDoc (dl.acm 10.1145/3735636): 40% compression with no quality loss; supports the "contracts should be compressed, not verbose" rule. [research-backed]

Empirical case evidence:

- Post 3620 — Kirill's 132,666-LOC production project: after GRACE markup + contracts were rolled in via Kilo Code, agents read fewer unnecessary files, hallucinated less, and required zero rollbacks during autonomous runs. Two latent bugs surfaced during markup annotation. [empirical]
- Post 3693 — Vlad's 282-contract, 1049-test project: contracts applied correctly but logging absent; agent self-reflected that "tests without logs are like vodka without beer" — motivating post for the contracts-plus-logging pairing. [empirical]
- Post 3838 — Alexey's 300,000+ LOC project built with GRACE (estimated $7M+ at US developer rates if built traditionally). [empirical]
- Practical_applications.md §"Пример CRM" — Vlad's CRM: multi-model interchangeability under GRACE contracts + markup. [empirical]

Legacy-contract-recovery research:

- `contracts_as_anchor.md §"Научная опора"` — Google CodeT5+ fine-tuned on JML for contract recovery at 90–97% quality (dl.acm 10.1145/3689484.3690738). [research-backed]

## See also

- `../02-semantic-graph/graph-as-backbone.md` — the graph layer that sits above contracts and drives their structure.
- `../02-semantic-graph/hierarchy-and-anchors.md` — full spec → contract → code hierarchy; IDE-2.0 vision.
- `./ai-contracts-vs-dbc.md` — contrast with Eiffel/JML classical DbC; why Hoare-logic conditions were burned out of LLM training.
- `./contract-fields.md` — canonical PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS/RATIONALE specification with concrete example.
- `./wenyan-prompting.md` — the compression technique behind KEYWORDS and LINKS sections.
- `./rationale-and-aag.md` — architectural intent section in module contracts; 3× bug-fix accuracy research.
- `./legacy-contract-recovery.md` — adding contracts to old code; CodeT5+ and Anton's contract-only MCP.
- `./language-specific/python.md` — Python placement rules and full example from post 3878.
- `./language-specific/csharp.md` — C# XML-documentation mapping and `region` directives.
- `../01-transformer-foundations/llm-training-pipeline.md` — the FIM/SFT/RL stages that force trust-code-over-docs.
- `../05-logging-ldd/log-driven-development.md` — the paired discipline that makes contracts-plus-logs the working combination.
- `../11-case-studies/vlad-crm-and-contracts.md` — the two Vlad cases where contract discipline met (and missed) logging discipline.
