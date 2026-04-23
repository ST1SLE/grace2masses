# GRACE Glossary

> Master glossary of every GRACE term, acronym, and concept found in the source material. Entries are sorted alphabetically; related acronyms are grouped under their expansion. Each entry cites its source and links to the kb/ file that covers the concept in depth.

## A

**AAG** — Architectural-intent section in a module contract; appears alongside `RATIONALE` in GRACE module contracts to expose architectural reasoning to the AI agent. Sources do not specify the full expansion of the acronym; referenced in post 3890. Cross-ref: [03-contracts/rationale-and-aag.md](03-contracts/rationale-and-aag.md)

**AI-contracts** — Contracts written around code specifically for LLM consumption; closer in spirit to Python docstrings and Spec-Driven Development than to classical Design by Contract (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"). Cross-ref: [03-contracts/ai-contracts-vs-dbc.md](03-contracts/ai-contracts-vs-dbc.md)

**AI-Friendly (library rating)** — Attribute GRACE's Kilo Code integration scans for when rating libraries for agent-based development (practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"). Cross-ref: [09-tooling/kilo-code-open-code.md](09-tooling/kilo-code-open-code.md)

**Alignment anchor** — Role of the `PURPOSE` field in an AI contract; anchors the model to goal-achievement patterns rewarded during Reinforcement Learning (post 3878, post 3889). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**Anchor** — Token/section serving as a navigation point inside a large code/log context; the "table of contents" for sparse attention past 4k tokens (most_important.md §"Без семантического графа"). Cross-ref: [02-semantic-graph/hierarchy-and-anchors.md](02-semantic-graph/hierarchy-and-anchors.md)

**Anti-Loop Protection** — Design-time mechanism preventing autonomous agents from burning tokens in fix-loops; combines attempt counters with forced context injections (post 3695). Cross-ref: [07-autonomous-agents/anti-loop-protection.md](07-autonomous-agents/anti-loop-protection.md)

**Architect mode / Code mode / Debug mode** — Kilo Code's phase-aware UI modes, each with associated prompts; maps directly onto GRACE's top-down pipeline (practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"). Cross-ref: [08-workflow-and-phases/architect-code-debug-modes.md](08-workflow-and-phases/architect-code-debug-modes.md)

**Architectural graph** — Phase-1 output of the GRACE top-down pipeline: the AI models the app as a graph of main classes/modules plus business-process modeling (most_important.md §"Top-down логика"). Cross-ref: [08-workflow-and-phases/top-down-pipeline.md](08-workflow-and-phases/top-down-pipeline.md)

**AST-free parsing** — Reading or mutating code without a full compiler/parser by relying on paired START-END tags; powers Anton's contract-only MCP pattern (post 3622). Cross-ref: [04-markup-system/start-end-tags.md](04-markup-system/start-end-tags.md)

**Attention-head correlations** — Raw origin of graphs inside GPT; correlations in attention-head tables act as edges between token vectors — graphs are "emergent" in attention heads (most_important.md §"Граф как основа навигации"). Cross-ref: [02-semantic-graph/graph-as-backbone.md](02-semantic-graph/graph-as-backbone.md)

## B

**Base coder (training phase)** — First phase of LLM coding training: ~80% code, ~20% natural language; the 20% is for task-understanding, NOT documentation (post 3820). Cross-ref: [01-transformer-foundations/llm-training-pipeline.md](01-transformer-foundations/llm-training-pipeline.md)

**Bash (standard toolset)** — Bash is in the standard LLM toolset — reliable inside long trajectories; author recommends bash CLI wrappers over DIY MCPs (post 3834). Cross-ref: [06-testing/cli-not-custom-mcp.md](06-testing/cli-not-custom-mcp.md)

**Belief state** — AI's verbalized hypothesis about how code works at the moment of writing; logged alongside observable state so later analysis can compare belief to ground truth (post 3693). Cross-ref: [05-logging-ldd/belief-state-logging.md](05-logging-ldd/belief-state-logging.md)

## C

**Call Graph proxy** — Role of the `LINKS` section in contracts; replaces a classical Call Graph for many needs without requiring a graph database (post 3732, post 3819). Cross-ref: [02-semantic-graph/graph-rag-vs-vector-rag.md](02-semantic-graph/graph-rag-vs-vector-rag.md)

**"Captain Obvious" comments** — Comments that restate what the code already says; useless and can be harmful; LLM reads code natively (post 3890). Cross-ref: [13-antipatterns/all-antipatterns.md](13-antipatterns/all-antipatterns.md)

**Causal reading** — GPT reads in one direction, "not finishing" content on the way; consequence: placement BEFORE a function/method declaration is critical for the anchor-vector formation around the name token (post 3878). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**Chain-of-thought (absence)** — GPT without chain-of-thought is limited to TC⁰ discrete logic (post 3948). Cross-ref: [01-transformer-foundations/tc0-and-dyck-language.md](01-transformer-foundations/tc0-and-dyck-language.md)

**Claude Code greedy reading** — Subagent prompt in Claude Code instructing retrieval of everything loosely related to the task (post 3821). Cross-ref: [09-tooling/mcp-scepticism.md](09-tooling/mcp-scepticism.md)

**Claude Code grace-marketplace plugin** — Alexey Chendemerov's Claude Code plugin implementing GRACE ideas: graph on code, START-END markup, contracts (github.com/osovv/grace-marketplace; practical_applications.md §"Плагин для Claude Code"). Cross-ref: [09-tooling/claude-code-grace-plugin.md](09-tooling/claude-code-grace-plugin.md)

**Cline** — Former core of Kilo Code; became a "sinking Titanic" after team departed to OpenAI; Gemini compatibility degraded; Aider-patch legacy introduced bugs (post 3793). Cross-ref: [09-tooling/kilo-code-open-code.md](09-tooling/kilo-code-open-code.md)

**CodeT5+** — Google's fine-tuned model for contract recovery from legacy code; used JML to recover contracts at 90–97% quality (contracts_as_anchor.md §"Научная опора"; dl.acm.org/doi/10.1145/3689484.3690738). Cross-ref: [03-contracts/legacy-contract-recovery.md](03-contracts/legacy-contract-recovery.md)

**Contract (AI)** — Function/module specification placed in-source near code; acts as the AI's semantic shield during modification (contracts_as_anchor.md §"Почему контрактное программирование"). Cross-ref: [03-contracts/contracts-overview.md](03-contracts/contracts-overview.md)

**Contract-only MCP** — Anton's pattern (post 3622): a custom MCP returning only contracts + declarations + implementation line ranges; particularly useful for legacy code where raw human source seeds bad patterns. Cross-ref: [03-contracts/legacy-contract-recovery.md](03-contracts/legacy-contract-recovery.md)

**Context 7 (integration)** — Example-lookup integration supported by the Kilo Code version of GRACE (practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"). Cross-ref: [09-tooling/kilo-code-open-code.md](09-tooling/kilo-code-open-code.md)

**Critical-mass threshold** — 5,000–10,000 LOC of interconnected logic past which AI loses understanding; not the whole-app size but a "noodle-code" pocket (post 3654). Cross-ref: [00-foundations/why-grace-exists.md](00-foundations/why-grace-exists.md)

**Cursor** — AI-IDE whose RAG reads code/logs in chunks of 100–200 lines via `read_file`; only the first ~100 lines are guaranteed read; without semantic markers the AI is "blinded" (xml_mapping_anchors.md §"Почему семантическая разметка"). Cross-ref: [09-tooling/cursor-limitations.md](09-tooling/cursor-limitations.md)

## D

**DbC** — Design by Contract (Eiffel/Hoare tradition); classical pre/post/invariant formal verification; LLM vendors burned strict test-like conditions out of training because they caused overfitting (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"). Cross-ref: [03-contracts/ai-contracts-vs-dbc.md](03-contracts/ai-contracts-vs-dbc.md)

**Dense attention** — n² attention matrix fitting ~4k tokens in Nvidia GPU memory (~16M attention values); above 4k, sparse attention takes over (most_important.md §"Почему без семантического графа"). Cross-ref: [01-transformer-foundations/sparse-attention-and-kv.md](01-transformer-foundations/sparse-attention-and-kv.md)

**DESCRIPTION (field)** — Optional narrative description of a function; GRACE permits skipping it — `PURPOSE` is critical, `DESCRIPTION` is not (post 3889). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**Developer-agent / Tester-agent split** — Multi-agent testing pattern: an AI developer writes a Markdown guide for an AI tester on how to test; on failure, tester-agent cuts failing data to XML and reports back (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy"). Cross-ref: [06-testing/agent-based-testing.md](06-testing/agent-based-testing.md)

**Diff-tool (Anthropic)** — New diff instrument Anthropic offered for Claude; supported in Kilo Code; author adapted GRACE code edits to it and made it work with Gemini via prompts + GRACE markup (practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"). Cross-ref: [09-tooling/kilo-code-open-code.md](09-tooling/kilo-code-open-code.md)

**Documentation-as-Code / In-source documentation** — Enterprise IT standard for embedding documentation directly in source code; GRACE is an Enterprise-level variant of this lineage (post 3878). Cross-ref: [04-markup-system/in-source-documentation.md](04-markup-system/in-source-documentation.md)

**Docstring** — Python function-level documentation string; LLM contracts are closer to docstrings than to classical DbC (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"). In GRACE Python, docstring stays BELOW `def` as an SFT-hint complement; sparse attention covers ~1000 recent tokens, so it still contributes (post 3878, post 3892). Cross-ref: [03-contracts/language-specific/python.md](03-contracts/language-specific/python.md)

**Drift (from original goals)** — Accumulated deviation of generated code from its spec, which GRACE's top-down hierarchy "practically completely eliminates" (most_important.md §"Top-down логика"). Cross-ref: [08-workflow-and-phases/top-down-pipeline.md](08-workflow-and-phases/top-down-pipeline.md)

**Dyck Language** — Paired-bracket languages formalized by Walter von Dyck; natively supported by TC⁰ and by transformers (Shunyu Yao et al., arxiv 2105.11115). Example: `( [ [ ] { } ] ( ) { ( ) } ) [ ]`. XML, HTML, JSON, and bracket-based programming languages are Dyck-compatible; YAML is not (post 3948). Cross-ref: [01-transformer-foundations/tc0-and-dyck-language.md](01-transformer-foundations/tc0-and-dyck-language.md)

**Dyck-3** — Example Dyck-language variant with three bracket types; illustration in post 3948. Cross-ref: [01-transformer-foundations/tc0-and-dyck-language.md](01-transformer-foundations/tc0-and-dyck-language.md)

## E

**Emergent graph** — Graphs implicit in attention-head tables; if you don't build one explicitly, GPT builds its own — often wrong — and freezes it in the KV cache (most_important.md §"Граф как основа навигации"). Cross-ref: [02-semantic-graph/graph-as-backbone.md](02-semantic-graph/graph-as-backbone.md)

**Enrichment points** — KEYWORDS/LINKS hooks in a contract that vector search lands on 90% of the time in Cursor trainees' observations (post 3819). Cross-ref: [03-contracts/wenyan-prompting.md](03-contracts/wenyan-prompting.md)

**Enterprise-level variant (of in-source docs)** — Author's framing of GRACE markup: an Enterprise-level variant of Documentation-as-Code / In-source documentation (post 3878). Cross-ref: [04-markup-system/in-source-documentation.md](04-markup-system/in-source-documentation.md)

**ERP-style logic (1C, SAP)** — Business-logic domains too fuzzy for strict assertion-equality; LDD unlocks auto-testing for these (testing_and_autonomous_agents.md §"Адаптация GRACE"). Cross-ref: [05-logging-ldd/log-driven-development.md](05-logging-ldd/log-driven-development.md)

**ETL rewrite (agent-based)** — Representative complex-case for agent-based testing; classical pytest autotests often fail to launch with AI on such flows (testing_and_autonomous_agents.md §"Agent-based testing"). Cross-ref: [06-testing/agent-based-testing.md](06-testing/agent-based-testing.md)

## F

**Feedback layer** — Modern location of tests in the AI-era workflow: tests moved from the analytical phase to the self-correction loop on debug (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"). Cross-ref: [06-testing/tests-in-self-correction-loop.md](06-testing/tests-in-self-correction-loop.md)

**Few-shot** — Training/prompting regime of in-context examples; relevant to the autonomous-loop danger: failed fixes become few-shots that make the model more confident about the failed approach (post 3695). Cross-ref: [07-autonomous-agents/anti-loop-protection.md](07-autonomous-agents/anti-loop-protection.md)

**FIM** — Fill-In-the-Middle; second phase of LLM coding training: complete and total ban on documentation; the goal is "how correct is this fragment vs the rest" (post 3820). Explains why agents are biased to read code and ignore external docs. Cross-ref: [01-transformer-foundations/llm-training-pipeline.md](01-transformer-foundations/llm-training-pipeline.md)

**Flag-style conclusions** — Modern test-framework practice of flooding the model with structured flag-rich log output instead of binary `TEST FAIL` (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"). Cross-ref: [06-testing/tests-in-self-correction-loop.md](06-testing/tests-in-self-correction-loop.md)

**Forced context** — Deliberate injection of information into an agent's context rather than relying on the agent to self-query via Tools; critical when an agent is looping or stuck (post 3695, post 3693). Cross-ref: [07-autonomous-agents/forced-context.md](07-autonomous-agents/forced-context.md)

**Forced skill tracing** — Reliable skill-loading pattern: one skill explicitly names the next skill to load, forming a deterministic chain; in Open Code, strengthened by explicitly requiring the `skill` tool call (post 3809). Cross-ref: [09-tooling/skills-system.md](09-tooling/skills-system.md)

**Function contract** — Phase-3 GRACE artifact: a local refinement of overall app intent at the function/method site (most_important.md §"Top-down логика"). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

## G

**GRACE** — Graph-RAG Anchored Code Engineering; methodology for AI-generated code at large scale (author's formal introduction; most_important.md §"Формальное введение в GRACE"). Positioned as "something like RUP for the AI era". Cross-ref: [00-foundations/what-is-grace.md](00-foundations/what-is-grace.md)

**Graph DB (optional)** — Not strictly required if chunks are pre-enriched with LINKS concepts before vectorization — GRACE's approach (post 3732). Cross-ref: [09-tooling/hybrid-rag-infra.md](09-tooling/hybrid-rag-infra.md)

**Graph RAG** — Hybrid RAG combining graph-based retrieval with vector similarity; more effective than pure vector RAG (post 3732). Cross-ref: [02-semantic-graph/graph-rag-vs-vector-rag.md](02-semantic-graph/graph-rag-vs-vector-rag.md)

**Greedy reading** — RAG-agent strategy (e.g., Claude Code subagent prompts) of pulling everything loosely related to the task — the FIM-trained default; preferred for AI-native code over contract-only reading (post 3622, post 3821). Cross-ref: [09-tooling/mcp-scepticism.md](09-tooling/mcp-scepticism.md)

**Ground truth (in LDD)** — Observable/correct outcome against which the AI compares its own `belief state` (post 3693). Cross-ref: [05-logging-ldd/belief-state-logging.md](05-logging-ldd/belief-state-logging.md)

## H

**Hoare logic** — `{P}C{Q}` (precondition, command, postcondition) formal verification system; classical DbC's mathematical basis (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"). Cross-ref: [03-contracts/ai-contracts-vs-dbc.md](03-contracts/ai-contracts-vs-dbc.md)

**HumanEval** — Simple-function code-generation benchmark; ShortenDoc's ~10% improvement metric (post 3892). Cross-ref: [03-contracts/wenyan-prompting.md](03-contracts/wenyan-prompting.md)

**Hybrid RAG** — Combination of exact-match SQL / keyword retrieval with vector similarity; the real answer to RAG limitations (post 3732). Cross-ref: [02-semantic-graph/graph-rag-vs-vector-rag.md](02-semantic-graph/graph-rag-vs-vector-rag.md)

## I

**IDE 2.0** — Vision of an AI-native IDE with a semantic-graph-of-anchor-tokens at the center; non-programmer can work at contract level (class/function/variable concepts only) (most_important.md §"IDE 2.0"). Cross-ref: [02-semantic-graph/hierarchy-and-anchors.md](02-semantic-graph/hierarchy-and-anchors.md)

**In-app AI console** — Agent with a console embedded inside the application; API wired to Tools; can run tests more sophisticated than pytest scripts (testing_and_autonomous_agents.md §"Почему обычные автотесты"). Cross-ref: [06-testing/in-app-ai-console.md](06-testing/in-app-ai-console.md)

**Injection of docs into code** — GRACE's approach to curing "semantic incompleteness" of bare code — embed docs IN code rather than beside (post 3820). Cross-ref: [04-markup-system/in-source-documentation.md](04-markup-system/in-source-documentation.md)

**In-source documentation** — See `Documentation-as-Code`. Cross-ref: [04-markup-system/in-source-documentation.md](04-markup-system/in-source-documentation.md)

**INPUTS (contract field)** — Canonical field listing the function's inputs as `- type - name: description` lines (post 3878). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**Isolation tests** — Small-scope tests used in the RL phase of LLM training (SWE-Bench analog) to fix/enhance code (post 3820). Cross-ref: [01-transformer-foundations/llm-training-pipeline.md](01-transformer-foundations/llm-training-pipeline.md)

## J

**JML** — Java Modeling Language; formal spec language for annotating Java code in the Design-by-Contract paradigm; used by CodeT5+ for legacy contract recovery (contracts_as_anchor.md §"Научная опора"). Cross-ref: [03-contracts/legacy-contract-recovery.md](03-contracts/legacy-contract-recovery.md)

**JSON (in long context)** — Degrades inside large contexts due to brace-counting across long spans and false correlations from `{}`; OpenAI's GPT-4.1 Prompting Guide explicitly advises against (xml_mapping_anchors.md §"XML vs JSON"). Still Dyck-compatible on small contexts (post 3948). Cross-ref: [01-transformer-foundations/xml-vs-json-vs-yaml.md](01-transformer-foundations/xml-vs-json-vs-yaml.md)

## K

**Kilo Code** — VS Code extension migrated from Cline core to Open Code core; has explicit Architect / Code / Debug modes; $100-for-bug-reports reward program (post 3793; practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"). Cross-ref: [09-tooling/kilo-code-open-code.md](09-tooling/kilo-code-open-code.md)

**KEYWORDS (contract field)** — Compressed list `[Keyword1, Keyword2]` of architectural patterns and task classifications; hooks for RAG; wenyan-style compression (post 3819, post 3878). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**Kimi paper** — Source referenced by the author for the claim that top-1000 MCP servers are in LLM training; sources do not specify URL — cite as "per Kimi paper" (post 3834). Cross-ref: [14-research-references/annotated-bibliography.md](14-research-references/annotated-bibliography.md)

**Knowledge Base (in tests)** — Regression memory of previous fixes; on regression, relevant KB entries are forced into the agent's context to prevent repeat-failure loops (post 3695). Cross-ref: [07-autonomous-agents/knowledge-base-in-tests.md](07-autonomous-agents/knowledge-base-in-tests.md)

**kulibin** (кулибин) — Russian colloquial for a DIY garage engineer; author's pejorative for home-grown tools (custom CLIs, custom MCPs) that LLMs weren't trained on and therefore ignore or hallucinate around (post 3834). Cross-ref: [09-tooling/mcp-scepticism.md](09-tooling/mcp-scepticism.md)

**KV cache** — Transformer's cached key/value states; can freeze wrong early graph branches — author's quip: "GPT becomes stubborn as a donkey" (most_important.md §"Граф как основа навигации"). Cross-ref: [01-transformer-foundations/sparse-attention-and-kv.md](01-transformer-foundations/sparse-attention-and-kv.md)

## L

**LDD** — Log Driven Development; author's methodology where the AI reads structured logs and makes a semantic judgment about correctness from the execution trajectory, instead of relying on strict `assert` equality (testing_and_autonomous_agents.md §"Адаптация GRACE"). Cross-ref: [05-logging-ldd/log-driven-development.md](05-logging-ldd/log-driven-development.md)

**Legacy contract recovery** — Practice of injecting contracts into old human-written code; CodeT5+/JML approach achieves 90–97% quality; Anton's contract-only MCP pattern is particularly suitable for legacy (contracts_as_anchor.md §"Научная опора"; post 3622). Cross-ref: [03-contracts/legacy-contract-recovery.md](03-contracts/legacy-contract-recovery.md)

**LINKS (contract field)** — Cross-module reference section in a contract; acts as a Call Graph proxy, enabling semantic module discovery even without direct call-graph tooling (post 3732, post 3819). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**Litmus test (methodology-as-graph)** — If a methodology can't be represented as a graph, it's text entropy to the AI — useful for spotting "water" in paid courses because GPT drops filler nodes (most_important.md §"Методология как граф"). Cross-ref: [02-semantic-graph/hierarchy-and-anchors.md](02-semantic-graph/hierarchy-and-anchors.md)

**Literary Chinese compression** — See `Wenyan prompting`. Cross-ref: [03-contracts/wenyan-prompting.md](03-contracts/wenyan-prompting.md)

**Logit analysis** — Author's technique for observing model degradation — checks whether the model correctly predicts tokens from compressed semantics (xml_mapping_anchors.md §"XML vs JSON"; post 3892). Cross-ref: [03-contracts/wenyan-prompting.md](03-contracts/wenyan-prompting.md)

**Log-to-code correlation** — Inject function ID + block ID tokens into every log line; without this, AI cannot bind logs back to code — Gemini attention collapses past ~5000 lines even with 1M context (post 3693). Cross-ref: [05-logging-ldd/log-to-code-correlation.md](05-logging-ldd/log-to-code-correlation.md)

**LOFT (log practices)** — Logging practices mentioned by the author as the basis he adapts for AI log localization needs (xml_mapping_anchors.md §"Почему семантическая разметка"). Cross-ref: [05-logging-ldd/log-to-code-correlation.md](05-logging-ldd/log-to-code-correlation.md)

## M

**MANDATORY MODE / MANDATORY PROTOCOL** — Trigger words inserted into skill descriptions to signal "this is a critical rule, not optional Knowledge Base"; mitigates LLM's tendency to skip skill-loading it thinks it already knows (post 3809). Cross-ref: [09-tooling/skills-system.md](09-tooling/skills-system.md)

**MCP (Model Context Protocol)** — LLM tool-integration protocol; LLMs weren't trained on custom MCPs and often ignore or hallucinate around them; top-1000 MCP servers ARE in training per Kimi paper (post 3820, post 3821, post 3834). Cross-ref: [09-tooling/mcp-scepticism.md](09-tooling/mcp-scepticism.md)

**MCS (Microsoft Consulting Services)** — Vendor pattern: MCS does first reference cases and helps small partners enter giant accounts (Gazprom, Boeing), then steps back (post 3708). Cross-ref: [12-economics-strategy/vendor-strategy.md](12-economics-strategy/vendor-strategy.md)

**Module contract** — Phase-2 GRACE artifact: per-module contract expanding intent; holds RATIONALE/AAG architectural sections (most_important.md §"Top-down логика"; post 3890). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

## N

**Noodle-code** (code-лапша) — Author's term for a tangled logic pocket inside a codebase; the critical-mass threshold (5–10k LOC) applies to such pockets, not to whole-app size (post 3654). Cross-ref: [00-foundations/why-grace-exists.md](00-foundations/why-grace-exists.md)

## O

**Open Code** — Core now underlying Kilo Code; ~2× more token-efficient than Cline core; direct Gemini API via VPN without OpenRouter proxy; supports single-agent ↔ multi-agent switch with the same prompts; supports dynamic skills via forced tracing (post 3793, post 3809). Cross-ref: [09-tooling/kilo-code-open-code.md](09-tooling/kilo-code-open-code.md)

**OpenAI GPT-4.1 Prompting Guide** — OpenAI cookbook article explicitly advising against JSON in long context; references Lee et al. "Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?" (cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters; xml_mapping_anchors.md §"XML vs JSON"). Cross-ref: [01-transformer-foundations/xml-vs-json-vs-yaml.md](01-transformer-foundations/xml-vs-json-vs-yaml.md)

**Orchestrator (in swarm)** — Coordinator role in multi-agent setups; example: 16-subagent Claude Opus swarm with orchestrator consolidating work (post 3859); also the gpt-5.4(xh) orchestrator in Kirill's 132k-LOC project (post 3620). Cross-ref: [07-autonomous-agents/swarm-vs-single-agent.md](07-autonomous-agents/swarm-vs-single-agent.md)

**OUTPUTS (contract field)** — Canonical field listing outputs as `- type - description` lines (post 3878). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**Overfitting (to tests)** — LLM tendency to collapse solution space to "whatever passes tests" when tests are in context alongside the contract; reason vendors burned strict test-like conditions out of training (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"; testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'"). Cross-ref: [06-testing/classical-tests-antipattern.md](06-testing/classical-tests-antipattern.md)

## P

**PATH header** — Header in a contract that declares the graph of related contracts; consumed by `get_contracts_by_contract_path` in Anton's contract-only MCP (post 3622). Cross-ref: [04-markup-system/start-end-tags.md](04-markup-system/start-end-tags.md)

**PBS** — Purpose Breakdown Structure; Anthropic-attributed concept of hierarchical goal decomposition; GPT always builds this decomposition from global goals down to local ones — verbalizing is better than leaving implicit (post 3878). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**PCAM** — Author's companion methodology for building agents; compatible with GRACE but not required by GRACE (most_important.md §"Формальное введение в GRACE"). Sources do not specify the full expansion of the acronym. Cross-ref: [00-foundations/what-is-grace.md](00-foundations/what-is-grace.md)

**Pet-project (survivorship bias)** — Small pet-project tutorials don't scale to production; "vibe coding" works until it doesn't (post 3654). Cross-ref: [00-foundations/why-grace-exists.md](00-foundations/why-grace-exists.md)

**Programmer 2.0** — Author's term for the practitioner who works "more with contracts than with code" (contracts_as_anchor.md §"Почему контрактное программирование"). Cross-ref: [03-contracts/contracts-overview.md](03-contracts/contracts-overview.md)

**PURPOSE (contract field)** — Goals/intent field; load-bearing; proven critical in benchmark work (post 3889). Cross-ref: [03-contracts/contract-fields.md](03-contracts/contract-fields.md)

**pytest** — "Reliable like a rock" in LLM training; standard tool — author removed his custom "kulibin CLI" after the LLM ignored it, kept pytest alone (post 3834). Cross-ref: [06-testing/cli-not-custom-mcp.md](06-testing/cli-not-custom-mcp.md)

## Q

**Qwen 3.5-27B** — Small Language Model; SWE-Bench Verified 72.4%; ran GRACE cleanly, more disciplined with in-source docs than larger LLMs; failed on Gradio framework specifics but fine for Python/React base (post 3902). Cross-ref: [11-case-studies/qwen-slm.md](11-case-studies/qwen-slm.md)

## R

**Random attention** — Fallback attention regime when no explicit graph exists; inefficient, used less in modern GPT (most_important.md §"Почему без семантического графа"). Cross-ref: [01-transformer-foundations/sparse-attention-and-kv.md](01-transformer-foundations/sparse-attention-and-kv.md)

**RATIONALE / AAG** — Sections in the module contract exposing architectural intent; research shows comments with architectural intent improve bug-fix accuracy up to 3× (post 3890). Cross-ref: [03-contracts/rationale-and-aag.md](03-contracts/rationale-and-aag.md)

**read_contracts (tool)** — Custom MCP tool in Anton's contract-only pattern: returns contract IDs + text + implementation line ranges (post 3622). Cross-ref: [04-markup-system/start-end-tags.md](04-markup-system/start-end-tags.md)

**read_file (Cursor)** — Cursor's chunked-read tool; 100–200 lines; only first ~100 guaranteed (xml_mapping_anchors.md §"Почему семантическая разметка"). Cross-ref: [09-tooling/cursor-limitations.md](09-tooling/cursor-limitations.md)

**Reflection (GPT)** — Author considers GPT reflection a reliable evaluation technique, but only when (1) GPT created something via the prompt/graph rather than merely observing it, and (2) you trust the OUTPUT, not the explanation (most_important.md §"Методология как граф"). Cross-ref: [02-semantic-graph/hierarchy-and-anchors.md](02-semantic-graph/hierarchy-and-anchors.md)

**region (C# directive)** — C# paired directive that functions as START-END tags for sparse attention inside an XML Documentation file (post 3960). Cross-ref: [03-contracts/language-specific/csharp.md](03-contracts/language-specific/csharp.md)

**RL** — Reinforcement Learning; fourth phase of LLM coding training (SWE-Bench analog): fix/enhance via isolation tests; no documentation (post 3820). RL rewards "goal achievement" — foundation of why PURPOSE is an alignment anchor (post 3878). Cross-ref: [01-transformer-foundations/llm-training-pipeline.md](01-transformer-foundations/llm-training-pipeline.md)

**RUP (for the AI era)** — Rational Unified Process analog; author's positioning of GRACE's scope (most_important.md §"Формальное введение в GRACE"). Cross-ref: [00-foundations/what-is-grace.md](00-foundations/what-is-grace.md)

## S

**SDD** — Spec-Driven Development; philosophical lineage of AI-contracts (contracts as "micro-specs" rather than validators) (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"). Cross-ref: [03-contracts/ai-contracts-vs-dbc.md](03-contracts/ai-contracts-vs-dbc.md)

**Self-correction loop** — Modern location of tests: during debug, with binary and detailed signals; tests help the model fix specific bugs rather than help it understand the task (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"). Cross-ref: [06-testing/tests-in-self-correction-loop.md](06-testing/tests-in-self-correction-loop.md)

**Semantic density** — Graphs compress ~50× vs source tokens and also denoise information (most_important.md §"Граф как основа навигации"). Cross-ref: [02-semantic-graph/graph-as-backbone.md](02-semantic-graph/graph-as-backbone.md)

**Semantic incompleteness** — Bare code's issue for AI: missing business/architectural context — cured by embedding docs IN code (post 3820). Cross-ref: [04-markup-system/in-source-documentation.md](04-markup-system/in-source-documentation.md)

**Semantic monolith** — Fused code+log object formed when log-to-code correlation tokens are injected — for GPT, log and code "merge into a single semantic monolith" (post 3693). Cross-ref: [05-logging-ldd/log-to-code-correlation.md](05-logging-ldd/log-to-code-correlation.md)

**Semantic noise (from tests)** — Cross-module test calls burn sparse-attention budget and can outweigh the benefits of tests (testing_and_autonomous_agents.md §"Почему обычные автотесты"). Cross-ref: [06-testing/classical-tests-antipattern.md](06-testing/classical-tests-antipattern.md)

**Semantic scaffold (of code)** — Author's name for the GRACE technique previously NDA-locked at major IT corps, made public for the first time (most_important.md §"Формальное введение в GRACE"). Cross-ref: [00-foundations/what-is-grace.md](00-foundations/what-is-grace.md)

**Semantic shield** — Role of contracts for AI during modification: AI always cross-checks edits against the contract (contracts_as_anchor.md §"Почему контрактное программирование"). Cross-ref: [03-contracts/contracts-overview.md](03-contracts/contracts-overview.md)

**Semantic squeeze** — Per-document compressed summary, analogous to a code contract; used by Mikhail Evdokimov's bank RAG on top of a graph main map (post 3839). Cross-ref: [10-beyond-code/grace-for-rag-documents.md](10-beyond-code/grace-for-rag-documents.md)

**SFT** — Supervised Fine-Tuning; third phase of LLM coding training: task-code pairs where the task is ~2–3 paragraphs and code is ~200–300 lines; strict Tools-chain training on popular agents/MCPs with a ban on DIY (post 3820). Cross-ref: [01-transformer-foundations/llm-training-pipeline.md](01-transformer-foundations/llm-training-pipeline.md)

**SFT-hint** — Role a descriptive docstring can still play as a supplementary hint placed BELOW the `def` in GRACE Python, because sparse attention still covers ~1000 recent tokens (post 3892, post 3878). Cross-ref: [03-contracts/language-specific/python.md](03-contracts/language-specific/python.md)

**ShortenDoc** — Small neural network + paper (dl.acm.org/doi/10.1145/3735636); proves 40% compression of docstrings without quality loss (sometimes improvement); ~10% HumanEval gain; full prepositional phrases cause degradation (post 3892). Cross-ref: [03-contracts/wenyan-prompting.md](03-contracts/wenyan-prompting.md)

**Shunyu Yao et al.** — Authors who proved native transformer Dyck-Language support; arxiv 2105.11115 (post 3948). Cross-ref: [01-transformer-foundations/tc0-and-dyck-language.md](01-transformer-foundations/tc0-and-dyck-language.md)

**skill (tool)** — Tool call in Open Code that can be explicitly required to force skill loading; strengthens skill reliability (post 3809). Cross-ref: [09-tooling/skills-system.md](09-tooling/skills-system.md)

**Skills system** — Modular, loadable prompt-fragment system; risk is LLM skipping "I already know" loads; mitigated by MANDATORY MODE / MANDATORY PROTOCOL and forced tracing (post 3809). Cross-ref: [09-tooling/skills-system.md](09-tooling/skills-system.md)

**SLM** — Small Language Model; Qwen 3.5-27B is the case study — 72.4% SWE-Bench Verified, runs GRACE cleanly (post 3902). Cross-ref: [11-case-studies/qwen-slm.md](11-case-studies/qwen-slm.md)

**Sliding window** — 100–200 token neighbor window used in sparse attention; stitched together via semantic graph anchors (most_important.md §"Почему без семантического графа"). Cross-ref: [01-transformer-foundations/sparse-attention-and-kv.md](01-transformer-foundations/sparse-attention-and-kv.md)

**Smoke tests** — "Did it crash?" tests that form the majority of real-world autotests and don't catch real bugs (testing_and_autonomous_agents.md §"Почему обычные автотесты"). Cross-ref: [06-testing/classical-tests-antipattern.md](06-testing/classical-tests-antipattern.md)

**"Sovereign AI" comments** — Derogatory label for non-English comments injected by "LLM nationalists" advocating sovereign AI; benchmarks show English comments are best, other languages cause degradation (post 3889). Cross-ref: [13-antipatterns/all-antipatterns.md](13-antipatterns/all-antipatterns.md)

**Sparse attention** — Attention mode engaged past ~4k tokens; uses 100–200-token sliding windows stitched via a semantic graph (most_important.md §"Почему без семантического графа"). Cross-ref: [01-transformer-foundations/sparse-attention-and-kv.md](01-transformer-foundations/sparse-attention-and-kv.md)

**Spec (TZ)** — Top of the GRACE hierarchy: TZ (Russian техническое задание) → design docs → module contracts → function contracts → code (most_important.md §"IDE 2.0"). Cross-ref: [02-semantic-graph/hierarchy-and-anchors.md](02-semantic-graph/hierarchy-and-anchors.md)

**sqlite-vec** — Hybrid SQL + vector-search library (github.com/asg017/sqlite-vec); author-recommended for precise facts where pure vector search fails (post 3732). Cross-ref: [09-tooling/hybrid-rag-infra.md](09-tooling/hybrid-rag-infra.md)

**START-END tags** — Paired markup tags enabling AST-free parsing and Dyck compatibility; work inside any language with comment syntax; powers Anton's contract-only MCP pattern (post 3622, post 3948). Cross-ref: [04-markup-system/start-end-tags.md](04-markup-system/start-end-tags.md)

**Summary tag (C#)** — C# XML Documentation tag; author notes it is sub-optimal for AI — single-line summary equivalent to long; AI reads code fine (post 3960). Cross-ref: [03-contracts/language-specific/csharp.md](03-contracts/language-specific/csharp.md)

**Survivorship bias (in pet-projects)** — Tutorials on small pet-projects don't scale; hidden failure mode when claiming small results will extrapolate (post 3654). Cross-ref: [00-foundations/why-grace-exists.md](00-foundations/why-grace-exists.md)

**Swarm (of agents)** — Multi-agent configuration; 16-subagent Claude Opus swarm example (post 3859); Open Code supports single-agent ↔ multi-agent swap without changing prompts (post 3793). Cross-ref: [07-autonomous-agents/swarm-vs-single-agent.md](07-autonomous-agents/swarm-vs-single-agent.md)

**SWE-Bench** — Benchmark/training regime for fix-via-test-signal; RL phase trains the model to read log fragments and emit patches (post 3820). Qwen 3.5-27B scored 72.4% on SWE-Bench Verified (post 3902). Cross-ref: [01-transformer-foundations/llm-training-pipeline.md](01-transformer-foundations/llm-training-pipeline.md)

## T

**Table of contents (for sparse attention)** — Metaphor for the role of the semantic graph — it stitches sliding windows together for GPT on large contexts (most_important.md §"Почему без семантического графа"). Cross-ref: [02-semantic-graph/graph-as-backbone.md](02-semantic-graph/graph-as-backbone.md)

**TC⁰** — Complexity class of discrete logic accessible to GPT without chain-of-thought; natively supports Dyck Language (post 3948). Cross-ref: [01-transformer-foundations/tc0-and-dyck-language.md](01-transformer-foundations/tc0-and-dyck-language.md)

**TDD (classical)** — Test-Driven Development — often an antipattern for AI; tests in the contract context collapse the solution space and cause overfitting (testing_and_autonomous_agents.md §"Адаптация GRACE"). Cross-ref: [06-testing/classical-tests-antipattern.md](06-testing/classical-tests-antipattern.md)

**Technical debt (AI)** — Accumulated loss of AI understanding after 1–2 working modules; main reason most AI-code efforts stall (most_important.md §"Формальное введение в GRACE"; post 3654). Cross-ref: [00-foundations/why-grace-exists.md](00-foundations/why-grace-exists.md)

**TEST FAIL (binary signal)** — Anti-practice: modern test frameworks should emit flag-rich structured logs, not a single binary verdict (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"). Cross-ref: [06-testing/tests-in-self-correction-loop.md](06-testing/tests-in-self-correction-loop.md)

**Top-down pipeline** — GRACE's three-phase workflow: architectural graph → module contracts → function contracts + code; assisted top-down because AI tends to jump to code on its own (most_important.md §"Top-down логика"). Cross-ref: [08-workflow-and-phases/top-down-pipeline.md](08-workflow-and-phases/top-down-pipeline.md)

**Top-1000 MCP servers** — MCP servers that ARE in LLM training (per Kimi paper) — Asana/JIRA-tier work; custom MCPs do not (post 3834). Cross-ref: [09-tooling/mcp-scepticism.md](09-tooling/mcp-scepticism.md)

**Trajectory (execution)** — The running log of what a program did; LDD has the AI make semantic judgments from it rather than from strict assert equality (testing_and_autonomous_agents.md §"Адаптация GRACE"). Cross-ref: [05-logging-ldd/log-driven-development.md](05-logging-ldd/log-driven-development.md)

**Trigger words** — See `MANDATORY MODE / MANDATORY PROTOCOL`. Cross-ref: [09-tooling/skills-system.md](09-tooling/skills-system.md)

**TZ (техническое задание)** — Russian for "technical task" / spec; top of the GRACE hierarchy (most_important.md §"IDE 2.0"). Cross-ref: [02-semantic-graph/hierarchy-and-anchors.md](02-semantic-graph/hierarchy-and-anchors.md)

## V

**Vector RAG (limits)** — Pure vector search handles semantic similarity well but fails for exact citations, exact facts, precise numbers (post 3732). Cross-ref: [02-semantic-graph/graph-rag-vs-vector-rag.md](02-semantic-graph/graph-rag-vs-vector-rag.md)

**Vendor-agnosticism** — GRACE's observed effect: good semantic scaffolding makes models feel interchangeable — "you don't feel a difference between models" (Vlad CRM case, practical_applications.md §"Пример CRM"). Cross-ref: [00-foundations/core-principles.md](00-foundations/core-principles.md)

**"Vibe coding"** — Informal AI-coding with minimal method; works at pet-project scale but not at production scale; "vibe coding works until it doesn't" (post 3654). Cross-ref: [13-antipatterns/all-antipatterns.md](13-antipatterns/all-antipatterns.md)

## W

**Walter von Dyck** — German mathematician who defined the paired-bracket languages now called Dyck Languages (post 3948). Cross-ref: [01-transformer-foundations/tc0-and-dyck-language.md](01-transformer-foundations/tc0-and-dyck-language.md)

**Wenyan prompting** (文言) — Literary-Chinese-style extreme-brevity prompting; KEYWORDS/LINKS sections ARE wenyan prompting; ~1:15 token compression vs full prose; Grok reconstructed hidden meaning from compressed contract alone (post 3819; post 3892). Cross-ref: [03-contracts/wenyan-prompting.md](03-contracts/wenyan-prompting.md)

## X

**XML** — Dyck-compatible markup; favored over JSON in long contexts (vendor-endorsed by OpenAI + Google) (xml_mapping_anchors.md §"XML vs JSON"; post 3948). Cross-ref: [04-markup-system/xml-like-markup.md](04-markup-system/xml-like-markup.md)

**XML Documentation (C#)** — Standard C# documentation format; highly compatible with GRACE; `region` directives work as paired START-END tags; IDE + auto-generator support (post 3960). Cross-ref: [03-contracts/language-specific/csharp.md](03-contracts/language-specific/csharp.md)

**XML-like markup** — Author-preferred lightweight variant of XML: fewer angle brackets, simpler tags; paired START/END tags for Dyck compatibility; tags name semantic roles, not implementation details (xml_mapping_anchors.md; post 3948, post 3960). Cross-ref: [04-markup-system/xml-like-markup.md](04-markup-system/xml-like-markup.md)

## Y

**YAML** — NOT Dyck-compatible; Google banned YAML libraries in Gemini's Code Execution sandbox to prevent degradation; author recommends converting YAML to Dyck-compatible formats (post 3948). Cross-ref: [01-transformer-foundations/xml-vs-json-vs-yaml.md](01-transformer-foundations/xml-vs-json-vs-yaml.md)

## Z

**Zero-doc (as embedded architecture)** — GRACE's outcome: with a graph + module contract + function contracts, classical documentation becomes unnecessary because architecture and business model are embedded in code (most_important.md §"Top-down логика"). Cross-ref: [03-contracts/contracts-overview.md](03-contracts/contracts-overview.md)

**"Zombie mode" (agent)** — Author's term for an agent on a long trajectory with self-check as top priority; it will not let external operators easily verify results mid-flight (post 3927). Cross-ref: [07-autonomous-agents/swarm-vs-single-agent.md](07-autonomous-agents/swarm-vs-single-agent.md)
