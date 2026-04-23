# Top-Down Pipeline — The Three GRACE Phases

## What

GRACE's generation workflow is a **strict top-down pipeline of three phases**, executed in order, where each later phase refines the output of the earlier one:

1. **Phase 1 — Architectural graph.** The AI models the application as a graph of the main classes/modules. Business-process modelling of the task and the mapping of business processes onto architecture are placed in the **same graph**. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
2. **Phase 2 — Module contracts.** Using the graph, the AI starts from the **module contract** and inside it expands the module's detailed intent. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
3. **Phase 3 — Function contracts + code.** When the AI is already writing classes and methods with contracts, GPT leans on the ready structure from the graph plus the refinement from the module contract. Therefore the **function contract is effectively a refinement of the application's overall intent at that specific site, not spontaneous AI creativity**. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")

The load-bearing rule on top of the ordering: for the transition to contract generation the AI **must be assisted**, because left on its own GPT will not structure its thinking top-down. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")

The phrase the author uses for the pipeline is "top-down logic and transition to contracts" (top-down логика и переход к контрактам). The post it comes from is a continuation of the formal GRACE introduction post and sits alongside the graph-as-navigation and IDE-2.0 posts in `most_important_output.md`. [source identifier] (`most_important_output.md` §"Top-down логика и переход к контрактам")

### One-line summary

> Graph first. Module contract second. Function contract + code third. The AI has to be walked through this order — on its own it jumps straight to code.
> ([author-claim] — paraphrase of `most_important_output.md` §"Top-down логика и переход к контрактам")

### Outcome properties

Two properties of the end-state are stated verbatim in the source and used throughout this file:

- **Near-elimination of drift from original goals.** Running the three phases in order "sharply raises code-generation quality and almost completely eliminates drift from original goals." [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **Zero-doc state.** "Graph + module contract + function contracts 100% replaces ordinary documentation. Documentation simply becomes unnecessary — both architecture and business model are embedded directly in the code in a form understandable for the AI." [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")

## Why

### Why the order is not negotiable

The author's position is that the top-down order is not a stylistic choice but a **correctness constraint**, for two explicit reasons stated in the source:

1. **Drift elimination.** Executing the three phases in order "sharply raises code-generation quality and almost completely eliminates drift from original goals" (сильно повышает качество генерации кода и практически полностью убирает дрейф от изначальных целей). [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
2. **Zero-doc state.** After the pipeline runs, the graph + module contracts + function contracts **replace classical documentation entirely** for any agent that later edits the code. The architecture and the business model are already embedded in the code in a form GPT reads. Documentation "simply becomes unnecessary." [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")

Both consequences depend on the phases running **in order**. If function contracts are written before a module contract exists, there is nothing for them to refine; they become spontaneous AI output and the drift they were supposed to contain is re-introduced. [author-claim — direct inference from the "refinement, not creativity" formulation] (`most_important_output.md` §"Top-down логика и переход к контрактам")

### Why the AI has to be **assisted** through the pipeline

The source is explicit that the agent will not run this pipeline by itself:

> "For the transition to contract generation the AI must be assisted in order to help GPT structure its thinking correctly. Here top-down analytics from the AI is required. In my GRACE framework I force the AI to go through this in 3 stages."
> ([author-claim] — `most_important_output.md` §"Top-down логика и переход к контрактам")

Two mechanisms from the broader GRACE source material explain **why** unassisted GPT fails to run the pipeline:

- **FIM training bias toward code.** The base pretraining regime (FIM — Fill-In-the-Middle) placed a "complete and total ban on documentation" and tuned the model to produce code against surrounding code. The model therefore has a strong pull toward code generation and away from structured upstream artefacts like graphs and contracts. [author-claim] (post 3820)
- **FIM's downstream bias toward module-level context.** The same training regime causes the agent's primary navigational task to be "find modules," not "understand class" — which is compatible with Phase-1 graph thinking, but only when the graph exists to navigate. Without an explicit graph, the agent fills in its own implicit one (see below). [author-claim] (post 3821)

Put the two together: unaided GPT will produce working code for small scopes but will skip the graph and the module contract on the way there. That shortcut is what GRACE's phase discipline blocks. [author-claim — synthesis of `most_important_output.md` §"Top-down логика и переход к контрактам" and post 3820]

### Why a graph at the top — not a spec or a doc

The graph-as-top-of-pipeline is justified by GRACE's broader position on transformer internals:

- **Graphs are emergent in attention heads.** Correlations in the heads' tables form edges between token vectors. If the author does not build a graph, GPT **builds one anyway**, often wrong, and freezes it in the KV cache. Explicitly constructing the graph prevents that. [author-claim] (`most_important_output.md` §"Граф как основа навигации")
- **Sparse attention past ~4k tokens needs anchors.** The top of the graph plays the role of a "table of contents" for GPT over long contexts. Without it, sparse attention falls back to random attention, which is inefficient. [author-claim, transformer-internals] (`most_important_output.md` §"Без семантического графа")
- **Semantic density.** Graph form compresses source by up to ~50× in token count and denoises it at the same time. That is what makes the graph usable as a Phase-1 anchor rather than another wall of prose. [author-claim] (`most_important_output.md` §"Граф как основа навигации")

In short: the graph sits at the top not because it is "the design document" in the human sense but because it is the only artefact shape the transformer can hold stably over a long generation trajectory. [author-claim — synthesis of `most_important_output.md` §"Граф как основа навигации" + §"Без семантического графа"]

### Why business-process modelling is inside the same graph

The source is explicit that business-process description and its links to architecture do not live in a separate document — they are imported into the Phase-1 graph itself:

> "The AI also transfers into this graph the business modelling of the task, the description of business processes, and their relationships with architecture."
> ([author-claim] — `most_important_output.md` §"Top-down логика и переход к контрактам")

The design implication: the Phase-1 graph is simultaneously the architectural graph and the business-model graph. Mapping between "what the business needs" and "which module covers it" is an intra-graph edge, not a cross-document link. [author-claim — direct reading of the source] (`most_important_output.md` §"Top-down логика и переход к контрактам")

This matters for Phase 2 and Phase 3 consumption. When the AI writes a module contract, the architectural intent AND the business intent of that module are both already in the graph the contract is refining. When the AI then writes a function contract, both intents are present one level down as well. [author-claim — direct reading of the source] (`most_important_output.md` §"Top-down логика и переход к контрактам")

### Why contracts — not more documentation

The source gives the training-distribution reason GRACE uses contracts rather than external docs at Phases 2 and 3:

> "The base training of most modern models as FIM practically did not use code documentation as context. Therefore the AI has a huge tendency to rely specifically on code, not on documentation. The presence of contracts physically does not let the agent 'ignore' the description of sense and intent."
> ([author-claim] — `most_important_output.md` §"Top-down логика и переход к контрактам")

Contracts live inside the code, therefore the FIM-trained agent reads them as part of reading the code. An external doc is in a different place and different distribution, and is usually not read. See `../03-contracts/contracts-overview.md` and `../01-transformer-foundations/llm-training-pipeline.md` for the full argument. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам"; post 3820)

### Why Phase-3 function contracts are "refinement, not creativity"

The source formulates Phase 3 very precisely:

> "When the AI is already writing classes and methods with contracts, GPT leans on the ready structure from the graph + refinement from the module contract. Therefore the function contract is in essence a **refinement of the application's overall intent at this specific location**, not spontaneous AI creativity."
> ([author-claim] — `most_important_output.md` §"Top-down логика и переход к контрактам")

Two things to extract from that formulation:

- **"Overall intent"** lives at the top of the pipeline — in the Phase-1 graph and (to a lesser extent) in the Phase-2 module contract.
- **"At this specific location"** means the function contract is the local instance of that overall intent at the site of one function. It is not a fresh piece of reasoning; it is the projection of the upstream intent onto the local scope.

This is what "near-elimination of drift" mechanically means in GRACE: every function contract is traceable up the pipeline to a module contract, which is traceable up to the graph. There is no floating function-level intent with no anchor. [author-claim — direct reading of the source] (`most_important_output.md` §"Top-down логика и переход к контрактам")

## How

### Single-shot generation vs phase-gated generation

The "must be assisted" rule can be reframed in prompt-engineering terms as: do **not** ask the model to produce the graph, the contracts, and the code in one prompt. Produce the graph first, inspect it, then produce the module contracts from the graph, inspect them, then produce the function contracts + code from the module contracts.

The economics of this approach are covered in detail in `./large-rare-prompts-billing.md`, but for the pipeline discussion two points are relevant:

- **Architect-phase prompt counts are low.** A typical Architect-mode session is "~20 prompts generating ~300–400k tokens of design context." This is Phase 1 and Phase 2 combined. The prompt count is small because each prompt carries a large amount of upstream context and produces a large structured artefact. [author-claim] (`practical_applications_output.md` §"Большие, но редкие запросы")
- **Daily request budgets rarely saturate.** Even a full working day of GRACE generation rarely exceeds 1000 requests. This is why flat-rate tariffs (Gemini Code Assist, $20–$40/month for ~2000 requests/day) suit GRACE better than per-token API pricing. [author-claim, vendor-doc] (`practical_applications_output.md` §"Большие, но редкие запросы")

The implication for the pipeline itself is that phase-gated prompting is not a performance cost. It does not multiply the prompt count by 3×; it structures what each prompt produces so that each prompt is much larger and much more anchored than an equivalent single-shot attempt. [author-claim — direct inference from the prompt-count and token-volume numbers] (`practical_applications_output.md` §"Большие, но редкие запросы")

### Concrete phase-by-phase walkthrough

The source describes the three phases in order; what follows is that description, expanded only with detail the source itself supplies.

#### Phase 1 — Architectural graph

- **Input.** The task/specification (TZ) at whatever level the user supplies it.
- **Produced artefact.** A graph whose nodes are the main classes/modules of the application and whose edges are their relationships. Into the **same** graph the AI transfers the business-process modelling of the task and the links between business processes and architecture. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **Why this is upstream.** Past ~4k tokens the top of the graph acts as the sparse-attention anchor system for everything the agent does afterwards. If it is missing, GPT will construct an implicit and usually wrong graph and freeze it in the KV cache. [author-claim, transformer-internals] (`most_important_output.md` §"Граф как основа навигации", §"Без семантического графа")
- **What not to do.** Do not defer the graph to "after we have some code." The IDE-2.0 post in the same source file places the graph at the very top of the hierarchy (TZ → derived docs → contracts → code), and the rest of the pipeline updates top-down from edits to the graph. [author-claim] (`most_important_output.md` §"IDE 2.0")
- **Two distinct graph surfaces in Phase 1.** The source places both the architectural structure (classes/modules) and the business-process modelling (processes + links to architecture) inside one graph. This is a deliberate compression: the agent sees the business reason and the technical structure in the same anchor surface, which is what allows the downstream contracts to refine both intents simultaneously. [author-claim — direct reading] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **Methodology-as-graph litmus test.** The graph form is also a methodology test: if something cannot be expressed as a graph, for GPT it is "text entropy," not methodology. This applies to external methodologies pulled into the project (e.g. planning methodologies imported from other domains) — they have to be graph-expressible to be operable. [author-claim] (`most_important_output.md` §"Методология как граф")

See `../02-semantic-graph/graph-as-backbone.md` and `../02-semantic-graph/hierarchy-and-anchors.md` for the full graph-layer treatment.

#### Phase 2 — Module contracts

- **Input.** The Phase-1 graph.
- **Produced artefact.** For each module (= each node in the graph that corresponds to a module/class), a **module contract** in which the detailed intent of that module is expanded. The source's phrasing: "the AI starts from the module contract and in it expands the detailed intent of this module." [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **What 'intent' means here.** The sources emphasise **architectural intent** at the module level — rationale for why the module exists, which architectural patterns it uses, and how it slots into the graph. This is the RATIONALE / AAG section of the module contract, covered in `../03-contracts/rationale-and-aag.md`. [author-claim] (post 3890; `most_important_output.md` §"Top-down логика и переход к контрактам")
- **Module-contract fields that carry intent upstream.** Beyond RATIONALE/AAG, the module contract uses KEYWORDS and LINKS as compact carriers of pattern-level and relationship-level intent. These are the fields that make the contract wenyan-compressed (~1:15 vs prose) and that the semantic-search infrastructure "lands on" in 90% of agent queries, per Cursor-trained practitioners reporting to the author. [author-claim, empirical] (post 3819)
- **Why this is not the last phase.** A module contract is still too coarse to generate code from directly. It is an intent layer, not an instruction layer. The instruction layer is the function contract in Phase 3.

See `../03-contracts/contracts-overview.md` for the broader theory of why contracts sit where they do.

#### Phase 3 — Function contracts + code

- **Input.** The module contract (Phase 2), itself anchored in the architectural graph (Phase 1).
- **Produced artefact.** For each function/method: a function contract in front of the declaration, followed by the code. The function contract is a **local refinement** of the upstream intent at this site; the code is generated against that contract plus the upstream context. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **What the function contract contains.** GRACE's canonical fields at the function level are PURPOSE, INPUTS, OUTPUTS, KEYWORDS, and LINKS. The concrete placement rule is that these go **before** the function declaration so the anchor vector forms around the function-name token during causal reading. [author-claim, research-backed] (post 3878; post 3890)
- **What the function contract is not.** It is not a place for the AI to invent new intent. The source's formulation is explicit: "contract of a function is in essence a refinement of the overall intent of the application in this place, **not spontaneous creativity of the AI**." [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **Why PURPOSE is the load-bearing field at this phase.** The arxiv 2601.16661v1 benchmarks (post 3889) found that the agent can skip the DESCRIPTION field entirely and still work, but skipping PURPOSE produces measurable degradation. Anthropic's Purpose Breakdown Structure observation — that GPT always builds hierarchical goal decomposition from global to local — is the mechanical reason: the function-level PURPOSE is the most local alignment anchor the agent has, and the whole pipeline upstream exists to produce a sensible PURPOSE at this site. [research-backed] (post 3889; post 3878)
- **Downstream of function-contract writing: code.** Once the contract is written in front of the declaration, code generation is anchored against the contract plus the upstream module contract plus the graph. The canonical Python example (post 3878) places PURPOSE / INPUTS / OUTPUTS / KEYWORDS as line comments before `def`, with a docstring below `def` as an SFT-hint complement covered by sparse attention's ~1000-token trailing window. [author-claim] (post 3878; post 3892)

See `../03-contracts/contract-fields.md` for the full field specification and `../03-contracts/language-specific/python.md` / `../03-contracts/language-specific/csharp.md` for language-specific placement rules.

### The "AI must be assisted" mechanism

The source states the requirement but does not prescribe a single mechanism. What it does describe — across the broader GRACE material — is the category of mechanisms that work:

- **Phase gating at the prompt level.** GRACE is implemented as a framework of prompts that walks the AI through the three phases one at a time, rather than a single prompt that asks for "the whole thing." [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам"; post 3793)
- **Phase-aware UI modes.** Kilo Code's Architect / Code / Debug mode UI is the closest tool-level expression of the three phases in the source material. Architect mode hosts Phase 1 and Phase 2; Code mode hosts Phase 3; Debug mode is downstream of the pipeline. See `./architect-code-debug-modes.md`. [vendor-doc, author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
- **Dynamic skills with forced tracing.** The author moved GRACE onto Open Code's dynamic-skills system, where the skills load each other by explicit name — precisely because without forced loading the LLM may decide "I already know" and skip a skill. For phase gating, this means one phase's skill explicitly loads the next phase's skill. [author-claim, research-mention] (post 3809; see `../09-tooling/skills-system.md`)

The sources **do not** specify a single canonical ordering of prompts or skill names for the three phases. Do not invent them. [sources do not specify]

### What breaks if you skip a phase

The sources do not list the failure modes as a table, but they are reconstructable from the surrounding GRACE material:

- **Skipping Phase 1 (no explicit graph).** GPT builds an implicit graph anyway during generation, using correlations in its attention heads. That implicit graph is usually wrong at scale and then gets frozen in the KV cache, making GPT "stubborn as a donkey" about the wrong structure. Past ~4k tokens, without the explicit top-of-graph as a sparse-attention anchor system, the agent falls back to random attention — inefficient and used less in modern GPT. [author-claim, transformer-internals] (`most_important_output.md` §"Граф как основа навигации", §"Без семантического графа")
- **Skipping Phase 2 (jumping from graph to function-level code).** Without a module contract expanding the intent of each module, function-level work has no proximate intent anchor to refine — only the distant graph. The source's formulation ("function contract = refinement of overall intent **at this place**") implicitly assumes the module contract as the local-enough intent layer. [author-claim — direct inference from the "refinement" formulation] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **Skipping Phase 3's contract-before-code discipline (writing code without a local function contract).** The agent lacks the anchor-vector formation around the function-name token that in-source research shows is load-bearing. The ShortenDoc paper plus the arxiv 2601.16661v1 benchmarks (referenced in posts 3892 and 3889) give the mechanical reason: PURPOSE adjacency matters, and docs split from code degrade the agent. [research-backed] (post 3889; post 3892)
- **Skipping the "must be assisted" step (single-shot prompting for the whole app).** This is the default behaviour the source warns against. Unassisted GPT jumps to code, which re-introduces exactly the technical debt GRACE exists to prevent. Most AI-code efforts stall after 1–2 working modules precisely because of this shortcut. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам"; post 3654)

### Empirical grounding for the pipeline's value

Several case studies in the source material bear on the pipeline's downstream payoff:

- **Kirill's 132k-LOC project (post 3620).** After applying GRACE markup + modernised logging on top of an existing pre-stage codebase (1494 files, 132,666 prod LOC, 50,026 test LOC), the agents read **fewer unnecessary files**, produced **zero rollbacks** across follow-up iterations, and hallucinated "noticeably less." Two latent bugs were also surfaced during the markup annotation. [empirical — practitioner report] (post 3620)
- **Vlad's CRM project (practical_applications_output.md §"Пример CRM").** With GRACE markup in place, Vlad reported that switching between Gemini Flash / Pro, Claude Opus 4.6, GLM-5, and Kimi K2.5 made no noticeable difference to generation quality: "you don't feel a particular difference between models — all generate equally well." The author's interpretation: high LLM sensitivity signals pipeline weakness, not model weakness. Since the pipeline in question is GRACE's three-phase top-down pipeline, this is evidence that the pipeline is the thing doing the stabilisation, not the model. [empirical — practitioner report; author-claim for the interpretation] (`practical_applications_output.md` §"Пример CRM")
- **Vlad's contracts project (post 3693).** Same practitioner, different project — 282 contracts, 1049 autotests. Applied GRACE contracts well but lacked GRACE logging. Reading failed "because tests without logs = vodka without beer." This case shows the pipeline is necessary but not sufficient: the debug layer below Phase 3 also has to exist. It does not weaken the pipeline's correctness claim; it qualifies the scope of what the pipeline alone delivers. [empirical — practitioner report] (post 3693)
- **Alexey's 300k+ LOC project (post 3838).** Built with GRACE. At US developer rates, equivalent traditional work would cost $7M+. This is a ceiling observation: the three-phase pipeline scales to the 300k-LOC regime in at least one documented case. [empirical — practitioner report] (post 3838)
- **Qwen 3.5-27B SLM (post 3902).** Ran GRACE cleanly and — notably — more **disciplined** with in-source docs than larger LLMs. Fails only on framework-specific gaps that 27B weights cannot hold (Gradio knowledge absent). Since GRACE's pipeline is prompt-driven and not model-specific, this is evidence that the pipeline does not depend on frontier-model behaviour. [empirical — practitioner report] (post 3902)

All five of these are practitioner reports, so the evidence category is [empirical], not [research-backed]. The research backing for specific **contract-field** claims is separate; see the field-level files linked below.

### The zero-doc outcome

The source explicitly describes the end-state the pipeline produces:

> "For agents that later edit such code, the presence of a graph, a module contract and function contracts 100% replaces ordinary documentation. It simply becomes unnecessary — both the architecture and the business model are already embedded directly in the code in a form understandable for the AI."
> ([author-claim] — `most_important_output.md` §"Top-down логика и переход к контрактам")

The consequence is that downstream edits do not need an external spec or doc-MCP to find intent. The agent reads the code, which includes the function contract, which is the local refinement of the module contract, which is the local refinement of the graph. The intent is reachable via the same reading path as the implementation. [author-claim — direct reading of the source] (`most_important_output.md` §"Top-down логика и переход к контрактам")

This is the same outcome the posts on external-documentation systems argue against: maintaining docs outside the code conflicts with the FIM-trained agent's "read the code" habit, so external docs are either ignored or hallucinated around. Embedding intent in the code via the three-phase pipeline side-steps that failure mode. [author-claim] (post 3820; post 3821)

### What is upstream and what is downstream

The source pins the pipeline inside a broader hierarchy, given in the IDE-2.0 post of the same file:

> "At the top of the IDE-2.0 is usually the TZ [spec], and at the lower level are already the contracts of functions/classes against which code is generated. If you change the TZ, the documents derived from it change, then the contracts for the functionality change, then the code there is regenerated."
> ([author-claim] — `most_important_output.md` §"IDE 2.0")

Mapping this onto the three phases:

- **Upstream of Phase 1.** The TZ (specification) and any derived design documents. When the TZ changes, the Phase-1 graph should be regenerated, which will in turn regenerate Phase-2 module contracts, which will regenerate Phase-3 function contracts and code. [author-claim] (`most_important_output.md` §"IDE 2.0")
- **Downstream of Phase 3.** Debug / self-correction over the generated code. GRACE's debug layer is LDD-based (log-driven), with log-to-code correlation tokens and belief-state logging. See `../05-logging-ldd/log-driven-development.md` and `./architect-code-debug-modes.md`. [author-claim] (post 3693; `testing_and_autonomous_agents_output.md` §"Тесты как часть self-correction loop")

So the three phases described here are neither the whole workflow nor self-contained; they are the **generation** portion of it. TZ → design docs sit above, and debug/test sit below.

### Propagation direction — top-down only

The IDE-2.0 post gives the canonical propagation direction of edits through the hierarchy: **top-down only**. A change to the TZ flows downward into the derived documents, then into contracts, then into the regenerated code. The source does not describe code-level changes flowing upward into contracts or graph. [author-claim] (`most_important_output.md` §"IDE 2.0")

This matches the "function contract = local refinement of overall intent" framing. If the local refinement drifts away from what the upstream graph and module contract specify, the expected repair path in GRACE is to **re-regenerate the downstream artefact from the current upstream**, not to patch the code and hope the upstream stays consistent. The author's wording — that top-down execution "almost completely eliminates drift from original goals" — only holds if the direction of regeneration is also top-down. [author-claim — direct inference from the propagation-direction formulation] (`most_important_output.md` §"Top-down логика и переход к контрактам"; §"IDE 2.0")

### The IDE-2.0 ergonomic consequence

One ergonomic consequence the source flags: once the pipeline is running top-down with graph + module contracts + function contracts as the operator surface, the operator no longer has to work at code level at all. The IDE-2.0 post phrases this as:

> "Strictly speaking, even an ordinary builder could work in this, because it is not obligatory to go below the level of contracts — it is simply the text of an order for functions and classes. In essence, to program with AI it will be enough to have concepts about what a class, a function, and variables are."
> ([author-claim] — `most_important_output.md` §"IDE 2.0")

The three-phase pipeline is therefore **also** the operator-surface model: Phase 1 (graph) and Phase 2 (module contracts) are the levels a non-programmer can work at meaningfully, with Phase 3 (function contracts + code) handled by the agent from the upstream artefacts. [author-claim] (`most_important_output.md` §"IDE 2.0")

### How the pipeline composes with the rest of GRACE

The three-phase pipeline is not a standalone doctrine; it is the **workflow surface** on which the rest of GRACE sits. A compact inventory of the dependencies as they appear in the source material:

- **XML-like markup at every phase.** The format in which the graph, the module contract, and the function contract are expressed is XML-like, not JSON and not YAML. The formal justification is Dyck-Language compatibility (TC⁰-parseable paired brackets) and the practical one is the OpenAI GPT-4.1 guide's explicit warning against JSON in long context. [research-backed, vendor-doc] (post 3948; `xml_mapping_anchors_output.md` §"XML vs JSON")
- **START-END paired tags in contract blocks.** The tags that delimit contract sections (and, in Python, the comment-block contract itself) are paired tags that give the transformer Dyck compatibility and also allow AST-free parsing tools (Anton's contract-only MCP in post 3622 is the concrete example). [author-claim] (post 3622; post 3948)
- **Wenyan-style compression in KEYWORDS/LINKS.** The compression used in the KEYWORDS and LINKS fields of Phase-2 and Phase-3 contracts is wenyan-style (~1:15 vs prose). This is what makes the contracts cheap enough to carry at scale without blowing the token budget. [research-backed] (post 3819; post 3892)
- **Log-to-code correlation downstream.** Phase-3 code emits logs with function-ID + block-ID tokens embedded. During debug, the agent reads logs against the contracts that generated the code — the logs merge with code into "a single semantic monolith" for GPT only because both carry the same identifier tokens. [author-claim] (post 3693)
- **Belief-state logging for self-correction.** When the AI writes Phase-3 code, it also writes log lines that verbalise its belief about how the code works. During debug the agent compares "what I believed" vs ground truth. This is only possible because the belief at generation time was anchored in the function contract, which was anchored in the module contract, which was anchored in the graph. [author-claim] (post 3693)
- **Forced context for autonomous runs.** At swarm scale, the agent cannot be trusted to fetch the relevant contract or graph on its own. Critical upstream context must be **pushed** into the agent's context. The three-phase pipeline only remains stable under autonomous operation if this mechanism is in place. [author-claim] (post 3695; post 3693)

The short version: GRACE's format choices (XML-like, Dyck-compatible, wenyan-compressed), its markup choices (paired START-END, PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS fields), its logging doctrine (log-to-code correlation, belief state), and its autonomy mechanics (forced context, anti-loop protection) are all specialisations for making the three-phase pipeline survive in realistic conditions. The pipeline is the spine; the rest are ribs. [author-claim — synthesis]

### When the pipeline is overkill

The sources acknowledge that for small enough scope the three-phase pipeline is unnecessary:

- **Small isolated tasks.** "While you are doing a small simple isolated task, you will not get a big benefit from contracts in code. The AI will do this easily anyway." [author-claim] (post 3654)
- **Under 800–1000 LOC.** The GRACE positioning statement says the methodology "makes sense for code from 800–1000 lines. For very small applications it is rather redundant." [author-claim] (`most_important_output.md` §"Формальное введение в GRACE")

For scopes under this threshold the pipeline's ceremony is not justified by the drift it prevents. The drift the pipeline exists to prevent only becomes a problem at the "5,000–10,000 LOC of interconnected logic" critical-mass threshold — which can be a local noodle-code pocket inside an otherwise larger codebase, not necessarily the whole application. [author-claim] (post 3654)

For larger scopes — specifically for 100%-AI-generated apps at 800–1000+ LOC scale, and especially for the 100k+-LOC range the author sees on training — the pipeline is not optional. This is the regime GRACE is positioned for. [author-claim] (`most_important_output.md` §"Формальное введение в GRACE"; post 3837)

### What the sources do **not** specify for this pipeline

To respect the no-invention rule, the following items are called out as not specified in the source for this file's scope:

- **Exact prompt text** for each of the three phases. The source says the author's framework "forces the AI to go through this in 3 stages" but does not reproduce the stage prompts verbatim. [sources do not specify]
- **Exact notation of the Phase-1 graph** (DOT, Mermaid, custom DSL, XML-like blocks in a markdown doc, etc.). The source describes the graph conceptually and functionally, not as a single concrete syntax. [sources do not specify]
- **Exact trigger conditions** for re-running Phase 1 vs Phase 2 vs Phase 3 after a change. The IDE-2.0 post describes the propagation-direction (top-down re-generation) but does not enumerate when each phase is mechanically re-run. [sources do not specify]
- **Per-phase review / acceptance criteria** (automatic validator, human review, LLM-reviewer). The sources mention GRACE-markup validators and LLM review in other contexts (posts 3904, 3892), but those are not specifically pinned to Phase 1, 2, or 3. [sources do not specify]

## Evidence

### Primary source

- **`most_important_output.md` §"Top-down логика и переход к контрактам"** — the only place in the source material that names the three phases explicitly, states the "must be assisted" requirement, and states the zero-doc outcome. All claims about phase numbering, ordering, what each phase produces, and drift elimination flow from this section. [author-claim]

### Supporting sources (same file)

- **`most_important_output.md` §"Граф как основа навигации"** — justifies why Phase 1 sits at the top (emergent graphs in attention heads; implicit graph will be built anyway if none is supplied). [author-claim]
- **`most_important_output.md` §"Без семантического графа"** — justifies why the Phase-1 graph is a hard requirement past ~4k tokens (sparse attention needs anchors). [author-claim, transformer-internals]
- **`most_important_output.md` §"IDE 2.0"** — places the three phases inside the broader hierarchy TZ → design docs → contracts → code and describes top-down propagation of edits. [author-claim]
- **`most_important_output.md` §"Методология как граф"** — the litmus test that a methodology that cannot be expressed as a graph is "text entropy" to the AI, which is why Phase 1 is a graph and not prose. [author-claim]

### Supporting sources (other files)

- **`testing_and_autonomous_agents_output.md` §"Тесты как часть self-correction loop"** — places the debug loop downstream of the three-phase pipeline. [author-claim]
- **`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code"** — describes Architect / Code / Debug as the mode UI that externalises the three phases at the tool level. [vendor-doc, author-claim]
- **`practical_applications_output.md` §"Большие, но редкие запросы"** — describes the economics of Phase-1/Phase-2 prompts (few, large, ~300–400k tokens per design session, ~20 prompts). [author-claim]

### Supporting sources (grace_matches.md)

- **Post 3820** — training-pipeline argument for why contracts (and not external docs) sit at Phases 2 and 3. FIM banned documentation during base training; the agent reads code, not docs. [author-claim]
- **Post 3821** — the agent's primary navigational task is "find modules," not "understand class" — which is why the Phase-1 graph's top-level nodes are modules/classes and why module contracts are produced before function contracts. [author-claim]
- **Post 3878** — canonical contract fields (PURPOSE, INPUTS, OUTPUTS, KEYWORDS) and the placement-before-declaration rule relevant to Phase-3 function contracts. [author-claim]
- **Post 3890** — RATIONALE/AAG sections and architectural intent at module level (Phase 2), with up-to-3×-bug-fix-accuracy evidence. [research-backed]
- **Post 3889** — PURPOSE is load-bearing at the contract level, shorter is better, English matters, adjacency matters — practical constraints on how Phase-3 contracts should be written. [research-backed]
- **Post 3793** — GRACE's migration to Open Code core, which supports the mode UI that externalises the phases. [author-claim]
- **Post 3809** — dynamic skills + forced tracing as the mechanism that keeps the AI on the pipeline instead of skipping phases. [author-claim, research-mention]

### Claim-type summary

- **Phase-1/2/3 structure, ordering, and content**: [author-claim] — sourced from `most_important_output.md` §"Top-down логика и переход к контрактам". No research paper in the sources pins the three-phase structure itself.
- **Transformer-level reasoning for why Phase 1 is a graph**: [author-claim, transformer-internals] — sourced from `most_important_output.md` §"Граф как основа навигации" and §"Без семантического графа".
- **Training-pipeline reasoning for why contracts (Phases 2–3) are in code, not external docs**: [author-claim] — sourced from post 3820.
- **Architectural-intent research backing for Phase-2 module-contract RATIONALE/AAG sections**: [research-backed] — post 3890.
- **Contract-field research (PURPOSE load-bearing, shorter better, English matters)**: [research-backed] — posts 3889 and 3892.
- **Empirical evidence for the pipeline at scale**: [empirical] — practitioner reports in posts 3620, 3693, 3838, 3902 and `practical_applications_output.md` §"Пример CRM".
- **Economics of phase-gated prompting**: [author-claim] — `practical_applications_output.md` §"Большие, но редкие запросы". The vendor link developers.google.com/gemini-code-assist/resources/quotas in that section is the only [vendor-doc] in this file's scope.

No claim in this file rests on a research paper or finding not stated in the six source files. Where research papers are mentioned (arxiv 2601.16661v1, dl.acm 10.1145/3735636, researchgate 400340807, arxiv 2105.11115), they are the papers cited by the respective source posts. No URL is invented. No finding is extrapolated beyond what the posts themselves report. [author-claim — meta-statement about this file's compliance with the PLAN.md no-invention rule]

## See also

- `./architect-code-debug-modes.md` — the Kilo Code UI's mapping of the three phases + downstream debug.
- `./large-rare-prompts-billing.md` — why Phase-1/Phase-2 prompts are few and large, and the billing model this implies.
- `../00-foundations/core-principles.md` — "Graph first, contracts second, code third" as a first principle.
- `../02-semantic-graph/graph-as-backbone.md` — full treatment of why the graph is the backbone of Phase 1.
- `../02-semantic-graph/hierarchy-and-anchors.md` — TZ → design docs → module contracts → function contracts → code hierarchy.
- `../03-contracts/contracts-overview.md` — why contracts are central to Phases 2 and 3.
- `../03-contracts/contract-fields.md` — the canonical PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS fields for function contracts.
- `../03-contracts/rationale-and-aag.md` — RATIONALE/AAG sections of module contracts at Phase 2.
- `../03-contracts/language-specific/python.md` and `../03-contracts/language-specific/csharp.md` — per-language placement rules for Phase-3 function contracts.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — why Phase-1 anchors are structurally necessary past ~4k tokens.
- `../01-transformer-foundations/llm-training-pipeline.md` — FIM / SFT / RL training regime that makes contracts-in-code the only reliable intent surface.
- `../05-logging-ldd/log-driven-development.md` — the debug-layer feedback mechanism downstream of Phase 3.
- `../07-autonomous-agents/forced-context.md` — the stabilisation mechanism that keeps autonomous agents on the pipeline.
- `../09-tooling/skills-system.md` — forced-tracing skills as the substrate for phase gating in Open Code / Kilo Code.
