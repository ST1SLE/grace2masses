# Core Principles of GRACE

## What

GRACE (Graph-RAG Anchored Code Engineering) is driven by seven first principles that distinguish it from classical software-engineering methodologies and from naive "vibe coding". These principles are not stylistic preferences; each one is grounded either in transformer internals, in LLM training pipelines, or in hard-won empirical observation across 200+ practitioners trained on the methodology [author-claim] (most_important.md §"Формальное введение"). If you ignore any of them, the methodology silently breaks at scale.

The seven principles:

1. Graph first, contracts second, code third (strict top-down).
2. Documentation lives IN code, not beside it.
3. Follow LLM's own patterns, not human standards.
4. Format choices are not cosmetic (XML-like, Dyck-compatible).
5. Testing moves to the self-correction loop, with LDD replacing strict asserts.
6. Vendor-agnosticism through semantic scaffolding.
7. If the methodology cannot be expressed as a graph, it is noise to the AI.

## Why

Each principle resolves a concrete failure mode observed when AI-generated code crosses the 5,000–10,000 LOC interconnected-logic threshold where GPT loses understanding (post 3654) [empirical]. Principles 1 and 7 address navigation collapse in sparse attention past ~4k tokens (most_important.md §"Почему без семантического графа") [research-backed]. Principle 2 follows from the LLM training pipeline, where FIM pretraining totally banned documentation and SFT used task-code pairs — so agents ignore external docs and trust code (post 3820) [research-backed]. Principle 3 stems from vendors having "burned" strict test-like DbC conditions out of training via overfitting avoidance (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC") [author-claim]. Principle 4 is backed by the TC⁰/Dyck-Language result (post 3948) [research-backed]. Principle 5 prevents overfitting-to-tests and semantic noise (testing_and_autonomous_agents.md §"Почему обычные автотесты") [author-claim]. Principle 6 is empirically grounded in the Vlad CRM case where multi-model switching was seamless under GRACE discipline (practical_applications.md §"Пример CRM") [empirical].

Without these principles, AI-generated apps stall after 1–2 working modules due to technical debt the AI itself injects (most_important.md §"Формальное введение") [author-claim].

## How

### Principle 1 — Graph first, contracts second, code third

Strict top-down pipeline: architectural graph of main classes/modules (with business-process modelling mapped into the same graph), then module contracts, then function contracts, then code (most_important.md §"Top-down логика и переход к контрактам") [author-claim]. The AI must be **actively assisted** through top-down; left alone, it jumps straight to code [empirical]. Function contracts are **local refinements** of overall application intent at that call-site, not spontaneous creativity (most_important.md §"Top-down логика"). Changes flow top-down: edit the spec → derived documents update → contracts update → code regenerates.

Why it exists: graphs are **emergent in attention heads** (correlations in attention tables = edges between token vectors) (most_important.md §"Граф как основа навигации") [research-backed]. If you do not build an explicit graph, GPT builds one anyway — often wrong — and then **freezes it in the KV cache**, becoming "stubborn as a donkey" (most_important.md §"Граф как основа навигации") [author-claim]. Graph semantic density reaches ~50× token compression vs. source (most_important.md §"Граф как основа навигации") [author-claim].

See also:
- `../02-semantic-graph/graph-as-backbone.md` — graph as persistent scaffolding.
- `../02-semantic-graph/hierarchy-and-anchors.md` — spec → contract → code hierarchy.
- `../08-workflow-and-phases/top-down-pipeline.md` — the three phases operationally.

### Principle 2 — Documentation lives IN code, not beside it

GRACE is an **Enterprise-level variant** of "In-source documentation" / "Documentation as Code" (post 3878) [author-claim]. Every mature IDE has built-in support for one or more such formats; not using them is "improper" (post 3878) [author-claim]. GRACE differs in **what** to put inside the format, not the format itself — the classical convention describes "what the function does"; GRACE emphasises **purpose/intent** and **correlation hooks** (KEYWORDS, LINKS) (post 3878) [author-claim].

Why it exists: the four-phase LLM training pipeline (post 3820) systematically penalises external documentation [research-backed]:
1. Base coder: ~80% code, ~20% natural language for task-understanding — NOT documentation.
2. FIM (Fill-In-the-Middle): **complete and total ban on documentation**; goal is fragment-correctness.
3. SFT: task-code pairs; task ~2–3 paragraphs, code ~200–300 lines; strict standard-Tools training; DIY banned.
4. RL (SWE-Bench analog): fix/enhance via isolation tests; no documentation.

Consequences (post 3820):
- External docs systems conflict with the training distribution → agents ignore them [empirical].
- If FORCED to use external docs, agents hallucinate "plausibly" (post 3820) [author-claim].
- Code is **not** harder for AI than docs; Python/Rust are "like English" to the model — the issue with bare code is only **semantic incompleteness**, solved by embedding docs IN code (post 3820) [author-claim].
- Adjacency matters empirically: docs separated from code by file-read or MCP-hop cause degradation (post 3889) [research-backed].

Research backing: in-source documentation benchmarks (arxiv 2601.16661v1, post 3889) show swings from **-95% degradation on bad structure** to **+435% improvement on good structure** [research-backed].

See also:
- `../01-transformer-foundations/llm-training-pipeline.md` — why AI ignores external docs.
- `../04-markup-system/in-source-documentation.md` — enterprise lineage.
- `../04-markup-system/markup-compression.md` — empirical bounds.

### Principle 3 — Follow LLM's own patterns, not human standards

"Do not invent your own format. Extract the target model's own contract patterns from its SFT data and follow those: sit on the patterns the AI already knows." (contracts_as_anchor.md §"Почему контрактное программирование" + §"Почему AI-contracts не равны классическому DbC") [author-claim].

Why it exists:
- Classical Design-by-Contract (Eiffel, JML) uses Hoare logic `{P}C{Q}` for formal verification (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC") [research-backed].
- LLM vendors **burned strict test-like conditions out of training** because they caused overfitting (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC") [author-claim].
- AI-contracts are closer to **Python docstrings** and **Spec-Driven Development** — "micro-specs" not validators (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC") [author-claim].
- OpenAI trains ChatGPT on docstrings directly (post 3892) [author-claim].
- When the AI generates a contract itself, it reproduces task wordings close to the SFT distribution it saw (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC") [author-claim].

The same principle applies to tools: custom MCP servers that "were not in training" are silently ignored or hallucinated around (post 3820, 3821, 3834) [empirical]. Top-1000 MCP servers are in training per the Kimi paper; Asana/JIRA-tier works, DIY does not (post 3834) [research-backed]. Bash is a standard tool and reliable; pytest is reliable "like a rock"; an author-built "kulibin CLI" was ignored by agents and had to be removed (post 3834) [empirical]. *Kulibin* (Russian) = DIY/garage engineering — pejorative for home-grown tools LLMs were never trained on.

See also:
- `../03-contracts/ai-contracts-vs-dbc.md` — the DbC distinction in full.
- `../09-tooling/mcp-scepticism.md` — when MCPs work and when they don't.
- `../06-testing/cli-not-custom-mcp.md` — standard tools over custom.

### Principle 4 — Format choices are not cosmetic (XML-like, Dyck-compatible)

XML-like markup, not JSON or YAML, in long contexts. This is vendor-endorsed by OpenAI and Google (xml_mapping_anchors.md §"Почему автор предпочитает XML") [vendor-doc]:
- OpenAI's GPT-4.1 Prompting Guide explicitly advises **against JSON in long context** (cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters) [vendor-doc].
- That guide references Lee et al. "Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?" (xml_mapping_anchors.md §"Почему автор предпочитает XML") [research-backed].
- Google also endorses XML-like markup (xml_mapping_anchors.md §"Почему автор предпочитает XML") [vendor-doc].

Why it exists — the formal layer (post 3948) [research-backed]:
- Without chain-of-thought, GPT is limited to **TC⁰** discrete logic.
- TC⁰ natively supports **Dyck Language** (paired-bracket languages, Walter von Dyck). Example: `( [ [ ] { } ] ( ) { ( ) } ) [ ]`.
- Shunyu Yao et al. proved native transformer Dyck support (arxiv.org/abs/2105.11115).
- **Dyck-compatible**: XML, HTML, JSON, bracket-based programming languages.
- **NOT Dyck-compatible**: YAML. Google **banned YAML libraries** in Gemini's Code Execution sandbox to prevent degradation (post 3948).
- Python comments alone are not Dyck-friendly; adding paired START-END tags in comments (GRACE pattern) makes them parsable in a single pass (post 3948).

Empirical JSON degradation: author observes via logit analysis that JSON causes slower convergence and false correlations from `{}` across long spans; the AI starts "counting braces" and drifts; universal across vendors including Qwen (xml_mapping_anchors.md §"Почему автор предпочитает XML") [empirical].

Paired START-END tags additionally enable **AST-free parsing** — tooling can read and edit code without a full compiler (post 3622) [author-claim].

See also:
- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis for XML preference.
- `../01-transformer-foundations/xml-vs-json-vs-yaml.md` — format choice, empirical & formal.
- `../04-markup-system/xml-like-markup.md` — rules for XML-like markup.
- `../04-markup-system/start-end-tags.md` — paired-tag markup.

### Principle 5 — Testing moves to the self-correction loop, with LDD replacing strict asserts

Classical TDD is "designed for humans, not models" (testing_and_autonomous_agents.md §"Почему обычные автотесты") [author-claim]. GRACE replaces it with two mechanics:

**LDD (Log Driven Development)**: instead of strict `assert` equality, the AI **reads structured logs and makes a semantic judgement** about correctness from the execution trajectory (testing_and_autonomous_agents.md §"Адаптация GRACE") [author-claim]. This is more reliable than strict equality for fuzzy business logic; it **unlocks auto-testing for ERP-style code** (1C, SAP) where automation is historically sparse (testing_and_autonomous_agents.md §"Адаптация GRACE") [empirical].

**Tests relocated to self-correction loop**: modern trend moves tests from the analytical phase to the **self-correction loop** during debug (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop") [research-backed]. Tests now help the model **fix a specific bug**, not understand the task. Both binary and detailed signals matter — `TEST FAIL` alone is bad practice; modern frameworks flood the model with "flag"-style conclusions easy for GPT to parse. SWE-Bench-style RL explicitly trains the model to read log fragments and emit patches.

Why it exists:
- Tests are "just more code" from the AI's perspective, and unusually hostile: heavy cross-module referencing burns sparse-attention budget (testing_and_autonomous_agents.md §"Почему обычные автотесты", §"Еще один пост про 'тесты как шум'") [author-claim].
- LLMs **overfit to test conditions**: if contracts and tests share context, solution space collapses to "whatever passes tests" (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'") [empirical].
- Most autotests are smoke tests (`did it crash?`) and don't catch real bugs (testing_and_autonomous_agents.md §"Почему обычные автотесты") [author-claim].
- LLM-written tests are low quality; training saw "tests+code" pairs but no meta-reasoning about tests → "LLM is a self-taught parrot when it comes to tests" (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop") [author-claim].

Concrete LDD mechanics (post 3693) [author-claim/empirical]:
- **log-to-code correlation**: inject function ID + block ID tokens into every log line; without this Gemini attention collapses past ~5000 lines even with 1M context.
- **belief-state logging**: when the AI writes code it verbalises its hypothesis about how the code works; corrections are trivial because "what I believed" vs "ground truth" is visible in the log.
- **forced context injection**: critical log lines pushed into context rather than relying on the agent to fetch them (post 3693, 3695).

See also:
- `../06-testing/classical-tests-antipattern.md` — why classical TDD hurts AI.
- `../06-testing/tests-in-self-correction-loop.md` — tests relocated to debug.
- `../05-logging-ldd/log-driven-development.md` — LDD overview.
- `../05-logging-ldd/log-to-code-correlation.md` — the critical heuristic.
- `../05-logging-ldd/belief-state-logging.md` — AI's hypothesis-driven logs.

### Principle 6 — Vendor-agnosticism through semantic scaffolding

Strong semantic scaffolding eliminates per-vendor sensitivity. In the Vlad CRM case (practical_applications.md §"Пример CRM"), a colleague with SAP background built a custom CRM for 5 security companies managing 1000+ personnel per shift (Next.js + shadcn + Supabase) while rapidly switching across **Gemini Flash, Gemini Pro, Claude Opus 4.6, GLM-5, Kimi K2.5** [empirical]. His reported observation: GRACE markup + specs made the models feel interchangeable — "you don't feel any particular difference between models; all generate equally well" (practical_applications.md §"Пример CRM") [empirical].

Why it exists: if you observe high LLM sensitivity during development, the author's claim is that **your pipeline is weak, not the model** (practical_applications.md §"Пример CRM") [author-claim]. Strong prompt frameworks tightly constrain the LLM's working channel, so per-vendor idiosyncrasies are smoothed over; frameworks also mask individual LLM weaknesses.

Corollary: when you switch LLMs and see a sharp quality delta, the first suspicion should be your project/methodology, not the model.

See also:
- `../11-case-studies/vlad-crm-and-contracts.md` — Vlad CRM case in full.
- `../12-economics-strategy/vendor-strategy.md` — why big labs won't own this layer.

### Principle 7 — If the methodology cannot be expressed as a graph, it is noise to the AI (litmus test)

"If a methodology cannot be represented as a graph, then for AI it is not a methodology but textual entropy." (most_important.md §"Методология как граф") [author-claim]. This is both a design rule and a diagnostic:
- **Design rule**: GRACE itself is represented as a methodology graph; construction-work planning was represented the same way (MS Project case, most_important.md §"Методология как граф") [empirical].
- **Diagnostic**: graphing a methodology exposes "water" — GPT drops filler nodes. Useful for detecting padding in paid courses: often only 5–6 graph nodes remain, and "GPT doesn't include water in the graph" (most_important.md §"Методология как граф") [author-claim].

Why it exists: GPT builds graphs from attention-head correlations regardless; content that cannot be graphed simply does not propagate through the model's internal structure. This is the same mechanism that makes Principle 1 work — so 7 is Principle 1's extension from code to methodology itself.

A complementary rule on reflection: GPT reflection is a reliable evaluation signal but only when (1) GPT created something via prompt/graph rather than merely observing it, and (2) you trust OUTPUT, not EXPLANATION (most_important.md §"Методология как граф") [author-claim].

See also:
- `../02-semantic-graph/hierarchy-and-anchors.md` — the litmus test in depth.
- `../10-beyond-code/general-complex-text.md` — generalisation beyond code.
- `../10-beyond-code/grace-for-rag-documents.md` — document-RAG extension (bank RAG case).

## Evidence

- **Formal introduction** (most_important.md §"Формальное введение") — GRACE name, target scope (800–1000+ LOC), refinement across 200+ practitioners, "only empirically working parts kept" [author-claim].
- **Top-down pipeline** (most_important.md §"Top-down логика и переход к контрактам") — the three-phase model; AI must be assisted through top-down [author-claim].
- **Graph as navigation base** (most_important.md §"Граф как основа навигации" + §"Почему без семантического графа") — graphs are emergent in attention heads; 4k boundary, sliding windows, random-attention fallback [research-backed].
- **Methodology as graph** (most_important.md §"Методология как граф") — litmus test [author-claim].
- **Contracts as shield** (contracts_as_anchor.md §"Почему контрактное программирование") — central principle; "Programmer 2.0 works more with contracts than with code" [author-claim].
- **AI-contracts ≠ classical DbC** (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC") — Hoare logic vs docstrings; vendors burned test-like conditions out of training [author-claim].
- **Vlad CRM multi-model swap** (practical_applications.md §"Пример CRM") — empirical backing for Principle 6 [empirical].
- **OpenAI GPT-4.1 Prompting Guide** (xml_mapping_anchors.md §"Почему автор предпочитает XML") — cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters; recommends XML-like over JSON in long context [vendor-doc].
- **TC⁰ / Dyck Language** (post 3948) — arxiv.org/abs/2105.11115 (Shunyu Yao et al.); mathematical basis for XML/JSON preference over YAML [research-backed].
- **Critical-mass threshold** (post 3654) — 5,000–10,000 LOC of interconnected logic is where AI loses understanding; can be a "noodle-code" pocket in a 100k-LOC app [empirical].
- **LLM training pipeline** (post 3820) — four phases; FIM bans docs; SFT bans DIY tools; RL has no docs [research-backed].
- **Sergei Muravyev MCP discussion** (post 3821) — agent's primary task is "find modules" not "understand class"; FIM-bias toward loading whole modules [author-claim].
- **kulibin-CLI removal** (post 3834) — author replaced custom CLI with standard pytest after LLM ignored/hallucinated around it; Kimi paper on top-1000 MCPs being in training [empirical / research-backed].
- **Log-to-code correlation, belief state, forced context** (post 3693) — Gemini attention collapse past ~5000 log lines without function/block ID tokens; log + code become "a single semantic monolith" with correlation [empirical].
- **Anti-loop / forced context for autonomous agents** (post 3695) — failed fixes become few-shots that reinforce wrong approach; attempt counters + context injections required [author-claim].
- **In-source-documentation benchmarks** (post 3889) — arxiv.org/abs/2601.16661v1; PURPOSE critical; English > other languages; adjacency matters; -95% to +435% [research-backed].
- **Bug-fix architectural intent** (post 3890) — researchgate.net/publication/400340807; architectural intent → 3× bug-fix accuracy; attention analysis confirms [research-backed].
- **ShortenDoc** (post 3892) — dl.acm.org/doi/10.1145/3735636; 40% compression without quality loss; ~10% HumanEval gain [research-backed].
- **In-source documentation lineage** (post 3878) — GRACE as Enterprise variant of "In-source documentation" / "Documentation as Code"; Anthropic Purpose Breakdown Structure; concrete Python example [author-claim / vendor-doc for PBS attribution].
- **Classical tests as antipattern** (testing_and_autonomous_agents.md §"Почему обычные автотесты", §"Еще один пост про 'тесты как шум'") — tests are code; cross-module calls burn sparse-attention budget; overfitting [author-claim].
- **Self-correction-loop tests** (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop") — SWE-Bench RL; binary + detailed signals; flag-rich logs [research-backed].

## See also

- `./what-is-grace.md` — name, positioning, scope.
- `./why-grace-exists.md` — the problem that motivates these principles.
- `../GLOSSARY.md` — definitions of all terms referenced above.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — transformer basis for Principles 1, 2, 4.
- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis for Principle 4.
- `../01-transformer-foundations/llm-training-pipeline.md` — training-pipeline basis for Principles 2, 3.
- `../02-semantic-graph/graph-as-backbone.md` — Principle 1 deep dive.
- `../02-semantic-graph/hierarchy-and-anchors.md` — Principles 1 and 7 deep dive.
- `../03-contracts/contracts-overview.md` — Principle 3 deep dive.
- `../03-contracts/ai-contracts-vs-dbc.md` — Principle 3 detailed distinction.
- `../04-markup-system/xml-like-markup.md` — Principle 4 application.
- `../04-markup-system/in-source-documentation.md` — Principle 2 lineage.
- `../05-logging-ldd/log-driven-development.md` — Principle 5 (LDD).
- `../06-testing/classical-tests-antipattern.md` — Principle 5 negation.
- `../06-testing/tests-in-self-correction-loop.md` — Principle 5 relocation.
- `../07-autonomous-agents/anti-loop-protection.md` — Principle 5 extension for autonomous agents.
- `../08-workflow-and-phases/top-down-pipeline.md` — Principle 1 operationalised.
- `../09-tooling/mcp-scepticism.md` — Principle 3 for tools.
- `../10-beyond-code/general-complex-text.md` — Principle 7 generalisation.
- `../11-case-studies/vlad-crm-and-contracts.md` — Principle 6 evidence.
- `../13-antipatterns/all-antipatterns.md` — antipatterns violating these principles.
- `../14-research-references/annotated-bibliography.md` — the cited papers in full.
