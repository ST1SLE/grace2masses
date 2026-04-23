# GRACE Knowledge Base — Build Plan & Task Checklist

> **Authority**: this file is the single source of truth for the KB build. Every agent reads it before starting. Every task's status is tracked here.

## 1. Goal

Build a comprehensive, fact-based, verified knowledge base on the GRACE methodology (Graph-RAG Anchored Code Engineering) from the source material in `../output_files/`.

- **Audience**: a working engineer who is comfortable with prompting LLMs but has never seen GRACE.
- **Language**: English (sources are Russian — translate faithfully, preserve technical terms).
- **Quality bar**: every claim is cited to its source; nothing invented; "what / why / how" covered for every concept.

## 2. Source material

All source files live in `/home/p3tal/Projects/personal/grace-posts/output_files/`:

| File | Content |
|---|---|
| `most_important_output.md` | 6 foundational GRACE posts (formal intro, top-down logic, graphs, IDE 2.0, etc.) |
| `contracts_as_anchor_output.md` | 3 contract-focused posts (shield, legacy recovery, DbC contrast) |
| `practical_applications_output.md` | 4 operational posts (Claude Code plugin, Kilo Code, billing, Vlad CRM) |
| `testing_and_autonomous_agents_output.md` | 5 posts on testing philosophy + autonomous agents |
| `xml_mapping_anchors_output.md` | 2 posts on XML-like markup and log anchors |
| `grace_matches.md` | **29 recent posts (2026-03-14 → 2026-04-23)** — the richest source, containing all new research citations, concrete contract-field spec, Dyck-Language argument, case studies, and autonomous-agent mechanics |

**Every agent must read all 6 source files before writing their assigned task.**

## 3. Output structure

```
kb/
├── PLAN.md                            ← this file
├── README.md                          [T01 — final pass]
├── INDEX.md                           [final pass, sequential]
├── GLOSSARY.md                        [T02]
│
├── 00-foundations/                    [T03–T05]
├── 01-transformer-foundations/        [T06–T09]
├── 02-semantic-graph/                 [T10–T12]
├── 03-contracts/                      [T13–T20]
│   └── language-specific/
├── 04-markup-system/                  [T21–T25]
├── 05-logging-ldd/                    [T26–T29]
├── 06-testing/                        [T30–T34]
├── 07-autonomous-agents/              [T35–T38]
├── 08-workflow-and-phases/            [T39–T41]
├── 09-tooling/                        [T42–T47]
├── 10-beyond-code/                    [T48–T49]
├── 11-case-studies/                   [T50–T55]
├── 12-economics-strategy/             [T56–T58]
├── 13-antipatterns/                   [T59]
└── 14-research-references/            [T60]
```

## 4. Rules for every agent

These apply to every single task:

1. **Read all 6 source files before writing.** No exceptions. The sources are dense and interconnected; skipping any one produces gaps.
2. **Cite every factual claim** to its source. Format:
   - Post from `grace_matches.md`: `(post 3693)` — use the post number from the `## Post 3693` headers.
   - Named content from the other 5 files: `(most_important.md §"Top-down…")` or the closest short heading label.
3. **Label claim type** at the sentence or paragraph level:
   - `[empirical]` — author's or community's observed behavior.
   - `[research-backed]` — supported by a cited paper, OpenAI guide, or similar.
   - `[author-claim]` — the author asserts it without external evidence.
   - `[vendor-doc]` — from an official vendor publication (OpenAI cookbook, Google docs).
4. **No invention.** If the sources do not specify something, write `"the sources do not specify."` Do NOT invent:
   - research paper URLs, titles, authors, or findings not present in the source text;
   - post numbers or dates;
   - benchmark numbers;
   - technical internals not stated in sources.
5. **Preserve Russian source terms** where helpful (e.g., "kulibin" for DIY tools) with a brief English gloss.
6. **File skeleton** (every file uses this):
   ```
   # <Title>

   ## What
   <concise definition / what this concept is>

   ## Why
   <why GRACE uses this; what problem it solves; what breaks without it>

   ## How
   <concrete mechanics / rules / examples>

   ## Evidence
   <citations to posts/files; research papers; empirical case studies>

   ## See also
   <relative links to other kb/ files — use relative paths like `../01-transformer-foundations/sparse-attention-and-kv.md`>
   ```
7. **Size cap**: ~300–500 lines per file. If content exceeds this, flag at the top of the file with a `> **Split candidate**: reason.`
8. **Do not write to any file outside your assigned output path.** Do not edit other agents' files.
9. **When you finish**: mark your task `- [x]` in this PLAN.md. Find your task ID (e.g. `T07`) on its line and toggle the checkbox. Use an Edit with unique `old_string` (include the task ID + full line text so no collision is possible). Then verify with Read.

## 5. Agent prompt skeleton (for whoever launches agents)

```
You are completing task <T##> in the GRACE knowledge base.

1. Read /home/p3tal/Projects/personal/grace-posts/kb/PLAN.md in full (rules,
   structure, your task's key-facts list, skeleton, citation conventions).

2. Read all 6 source files in /home/p3tal/Projects/personal/grace-posts/output_files/.
   Do not skip any.

3. Write your assigned output file at the path given in the PLAN.

4. After writing, Edit PLAN.md to change your task's `- [ ]` to `- [x]`.
   Match by the unique task ID in your line (e.g. "T07").
   Verify with Read that the edit landed.

5. Hard rules: cite every claim; label claim type; do not invent; English; ~300-500 lines.
```

## 6. Execution waves

- **Wave 1** (pilot, foundations + transformer grounding): T02–T09 — 8 tasks, parallel.
- **Wave 2** (core machinery: graph, contracts, markup): T10–T25 — 16 tasks, parallel.
- **Wave 3** (operational: logging, testing, autonomous, workflow): T26–T41 — 16 tasks, parallel.
- **Wave 4A** (tooling + beyond-code): T42–T49 — 8 tasks, parallel.
- **Wave 4B** (cases + economics + antipatterns + research): T50–T60 — 11 tasks, parallel.
- **Final pass** (sequential, single agent): T01 README, INDEX, link/duplication sweep.

Waves exist so early files can establish shared vocabulary before later files reference them. Within a wave, all tasks are independent.

---

## 7. Task checklist

Each task entry lists the **output file**, a **one-line scope**, and **required facts** (what the file MUST cover — not exhaustive but non-negotiable). Agents may add further content from the sources.

### Wave 1 — Foundations & transformer grounding

- [x] **T02** `kb/GLOSSARY.md` — Master glossary of every GRACE term and acronym.
  - Required entries (non-exhaustive; add any other terms found in sources):
    - **GRACE** — Graph-RAG Anchored Code Engineering.
    - **PCAM** — author's companion methodology for building agents.
    - **LDD** — Log Driven Development.
    - **FIM** — Fill-In-the-Middle (training regime).
    - **SFT** — Supervised Fine-Tuning.
    - **RL** — Reinforcement Learning (SWE-Bench-style).
    - **SWE-Bench** — benchmark/training regime for fix-via-test-signal.
    - **PBS** — Purpose Breakdown Structure (Anthropic-attributed; hierarchical goal decomposition).
    - **AAG** — (contract section name; sources use alongside RATIONALE; describe as "architectural-intent section in module contract").
    - **RATIONALE** — contract section exposing architectural intent.
    - **Wenyan prompting** — literary-Chinese-style compression (KEYWORDS/LINKS style).
    - **Dyck Language** — paired-bracket languages (Walter von Dyck); natively parsed by TC⁰.
    - **TC⁰** — complexity class of discrete logic accessible to GPT without chain-of-thought.
    - **KV cache** — transformer's cached key/value states (can freeze wrong early branches).
    - **Sparse attention** — attention mode past ~4k tokens, using sliding windows.
    - **Random attention** — fallback when no explicit graph (inefficient).
    - **Sliding window** — 100–200 token neighbor window in sparse attention.
    - **Belief state** — AI's verbalized hypothesis about how code works; used in logging.
    - **Log-to-code correlation** — function/block identifier tokens injected into log lines.
    - **Anti-loop protection** — counters + context injection to prevent autonomous-agent loops.
    - **Forced context** — deliberate injection of context into agent (vs. relying on self-query).
    - **MANDATORY MODE / MANDATORY PROTOCOL** — skill-system trigger words.
    - **Architect / Code / Debug modes** — Kilo Code's phase-aware UI modes.
    - **SDD** — Spec-Driven Development.
    - **DbC** — Design by Contract (Eiffel/Hoare tradition).
    - **Hoare logic** — `{P}C{Q}` formal verification system.
    - **Documentation-as-Code** / **In-source documentation** — enterprise IT standard GRACE extends.
    - **JML** — Java Modeling Language (formal contract spec).
    - **kulibin** — Russian colloquial for DIY garage engineering; author's pejorative for home-grown tools that LLMs weren't trained on.
    - **START-END tags** — paired markup tags enabling AST-free parsing and Dyck compatibility.
    - **Greedy reading** — RAG-agent strategy (e.g., Claude Code) of pulling all loosely-related modules.
    - **Few-shot** — training/prompting regime of in-context examples.
  - For each term: definition + cross-reference to the file(s) that explain it in depth.
  - Sources: all 6 files; `grace_matches.md` is especially rich.

- [x] **T03** `kb/00-foundations/what-is-grace.md` — Name, positioning, scope.
  - Required facts:
    - Full expansion: **Graph-RAG Anchored Code Engineering** (not "Graph Anchored"; the RAG is load-bearing).
    - Positioning: "something like RUP for the AI era".
    - Compatibility with PCAM; PCAM not required.
    - Refined over months with 200+ practitioners; only empirically-working parts kept.
    - Target: 100%-AI-generated apps at 800-1000+ LOC scale; overkill for small scripts.
    - Public exposure of "semantic scaffold of code" technique (author claims previously NDA-locked at major IT corps).
    - GRACE generalizes beyond code (see `10-beyond-code/`).
  - Sources: `most_important.md` primarily (formal intro post); `grace_matches.md` for reconfirmations.

- [x] **T04** `kb/00-foundations/why-grace-exists.md` — The problem.
  - Required facts:
    - Most AI-code efforts stall after 1-2 working modules due to accumulated technical debt (author observation).
    - **Critical-mass threshold**: 5,000–10,000 LOC of interconnected logic is where AI loses understanding (post 3654). This is NOT the whole-app size — can be a "noodle-code" pocket in a 100k-LOC app.
    - Two root causes:
      1. Sparse-attention collapse past ~4k tokens (transformer internals).
      2. RAG agents read chunked, small windows — blind without semantic anchors.
    - Survivorship bias: tutorials on small pet-projects don't scale.
    - "Vibe coding" works until it doesn't.
  - Sources: `most_important.md` + post 3654 + posts on sparse attention.

- [x] **T05** `kb/00-foundations/core-principles.md` — First principles.
  - Required facts:
    - Principle 1: **Graph first, contracts second, code third** (strict top-down).
    - Principle 2: **Documentation lives IN code, not beside it** (lineage: Documentation-as-Code / In-source documentation; post 3878).
    - Principle 3: **Follow LLM's own patterns**, not human standards (contracts from AI's SFT, not Eiffel).
    - Principle 4: **Format choices are not cosmetic** (XML-like, Dyck-compatible).
    - Principle 5: **Testing moves to self-correction loop**, with LDD replacing strict asserts.
    - Principle 6: **Vendor-agnosticism through semantic scaffolding** (Vlad CRM observation).
    - Principle 7: **If the methodology can't be expressed as a graph, it's noise to the AI** (litmus test).
  - Sources: synthesize across all files.

- [x] **T06** `kb/01-transformer-foundations/sparse-attention-and-kv.md` — Why long code contexts collapse.
  - Required facts:
    - Dense attention matrix grows n²; ~4k tokens fits in Nvidia GPU memory (≈16M attention values).
    - Past 4k tokens: **sparse attention** using 100–200-token sliding windows.
    - Sliding windows stitched together via semantic graph anchors = "table of contents".
    - Without anchors: fallback to **random attention** — inefficient, used less in modern GPT.
    - **KV cache** freezes early wrong branches ("GPT becomes stubborn as a donkey").
    - The ~4k boundary is a hardware constraint of the GPU working memory for attention matrices.
  - Sources: `most_important.md` §"Без семантического графа"; `grace_matches.md` post 3693 (log experiments).

- [x] **T07** `kb/01-transformer-foundations/tc0-and-dyck-language.md` — Formal basis for XML preference.
  - Required facts:
    - Without chain-of-thought, GPT is limited to **TC⁰** discrete logic.
    - TC⁰ natively supports **Dyck Language** (paired-bracket languages; Walter von Dyck).
    - Example: `( [ [ ] { } ] ( ) { ( ) } ) [ ]`.
    - Shunyu Yao et al. proved native transformer Dyck-Language support (arxiv 2105.11115).
    - Dyck-compatible formats: **XML, HTML, JSON, bracket-based programming languages**.
    - **NOT Dyck-compatible**: YAML.
    - Google **banned YAML libraries** in Gemini's Code Execution sandbox to prevent degradation.
    - Python's comment syntax is not Dyck-friendly alone; adding START-END paired tags in comments (GRACE pattern) makes it parsable in one pass.
  - Source: post 3948.

- [x] **T08** `kb/01-transformer-foundations/llm-training-pipeline.md` — Why AI ignores external docs.
  - Required facts — four-phase LLM coding training (post 3820):
    1. **Base coder**: ~80% code, ~20% natural language. The 20% is for task-understanding, NOT documentation.
    2. **FIM (Fill-In-the-Middle)**: complete and total ban on documentation; goal is "how correct is this fragment vs the rest?".
    3. **SFT**: task-code pairs; task is ~2–3 paragraphs, code is ~200–300 lines. Strict Tools-chain training on popular agents/MCPs; ban on DIY.
    4. **RL** (SWE-Bench analog): fix/enhance via isolation tests; no documentation.
  - Implications:
    - External docs systems conflict with training distribution → agents ignore them.
    - If FORCED to use external docs, agents hallucinate "plausibly" (post 3820).
    - Top-1000 MCP servers ARE in training (per Kimi paper) — Asana/JIRA-tier works; DIY MCPs don't (post 3834).
    - Bash is in the standard toolset — CLI wrappers work where custom MCPs don't.
    - Code is NOT harder for AI than docs; Python/Rust are "like English" to the model. Issue with bare code is only **semantic incompleteness**, solved by embedding docs IN code.
  - Sources: posts 3820, 3821, 3834.

- [x] **T09** `kb/01-transformer-foundations/xml-vs-json-vs-yaml.md` — Format choice, empirical & formal.
  - Required facts:
    - **OpenAI GPT-4.1 Prompting Guide explicitly advises against JSON in long context** (cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters).
    - That guide references Lee paper *"Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?"*.
    - JSON degradation mechanics: brace-counting across long spans, false correlations from `{}`, slower convergence.
    - Author observes effect via logit analysis; universal across vendors (Qwen too).
    - Google also endorses XML-like markup.
    - Formal layer: XML is Dyck-compatible (cross-link T07); YAML is not.
    - **Recommendation**: convert YAML to Dyck-compatible formats; author notes YAML often inserted by "second-role" vendor engineers by incompetence.
  - Sources: `xml_mapping_anchors.md` §"XML vs JSON"; post 3948.

### Wave 2 — Core machinery

- [x] **T10** `kb/02-semantic-graph/graph-as-backbone.md` — Graph as persistent scaffolding.
  - Required facts:
    - Graphs are **emergent in attention heads** (attention-table correlations = edges between token vectors).
    - If you don't build a graph, GPT builds one anyway — often wrong — and freezes it in KV cache.
    - Graph top = navigation anchors for sparse attention past 4k tokens.
    - Semantic density: ~50× token compression vs source; also denoises.
    - Without an explicit graph, GPT falls back to random attention (inefficient).
    - "If you can't build graphs in GPT, you don't actually know how to use GPT" (author claim).
  - Sources: `most_important.md` §"Граф как основа навигации" + "Почему без семантического графа".

- [x] **T11** `kb/02-semantic-graph/hierarchy-and-anchors.md` — Spec → contract → code hierarchy.
  - Required facts:
    - Hierarchy: TZ (spec) → design docs → module contracts → function contracts → code.
    - Changes propagate top-down: edit TZ → derived docs update → contracts update → code regenerates.
    - **IDE 2.0 vision**: semantic-graph-of-anchor-tokens at the center; non-programmer can work at contract level (class/function/variable concepts only).
    - **Methodology-as-graph litmus test**: if a methodology can't be graphed, it's noise to AI; useful for detecting "water" in paid courses (GPT drops filler nodes).
    - GPT reflection is reliable evaluation, but only when: (1) GPT created something via prompt/graph, not just observed; (2) you trust OUTPUT, not explanation.
  - Sources: `most_important.md` §"Top-down логика", §"IDE 2.0", §"Методология как граф".

- [x] **T12** `kb/02-semantic-graph/graph-rag-vs-vector-rag.md` — Hybrid retrieval.
  - Required facts:
    - Vector search handles **semantic similarity** well; new Qwen models even find abstraction-level analogies.
    - Vector search FAILS for **exact citations, exact facts, precise numbers**.
    - Hybrid RAG is the real answer.
    - Graph DB not strictly required if chunks are pre-enriched with LINKS concepts (GRACE's approach).
    - LINKS sections replace classic Call Graph without needing a graph DB.
    - **sqlite-vec** combines SQL + vector search — recommended for precise facts (github.com/asg017/sqlite-vec).
    - Vector alone → "you won't go far".
  - Source: post 3732.

- [x] **T13** `kb/03-contracts/contracts-overview.md` — Why contracts are central.
  - Required facts:
    - Contracts = **semantic shield** for AI during modification (it always cross-checks edits against the contract).
    - Written by the AI-author of the code. Humans may not need them; AI does.
    - "Programmer 2.0 will work more with contracts than with code" (author).
    - Graph + module contract + function contracts → **classical documentation becomes unnecessary** (architecture and business model embedded in code).
    - Base FIM pretraining barely used docs → AI heavily trusts code; contracts, sitting in code, cannot be ignored.
  - Sources: `contracts_as_anchor.md` §"Почему контрактное программирование".

- [x] **T14** `kb/03-contracts/ai-contracts-vs-dbc.md` — Distinction from classical DbC.
  - Required facts:
    - Classical DbC (Eiffel, JML): Hoare logic `{P}C{Q}` — formal pre/post/invariant for verification.
    - LLM vendors **burned strict test-like conditions** out of training because they caused overfitting.
    - AI-contracts are closer in spirit to **Python docstrings** and **Spec-Driven Development** — "micro-specs" not validators.
    - Common with classical DbC: location (in/next to code) and intent (requirements on algorithm).
    - Form is fundamentally different — tuned to GPT.
    - **Do not invent your own format**. Extract the target model's own contract patterns from its SFT data and follow those: "sit on the patterns the AI already knows".
  - Source: `contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC".

- [x] **T15** `kb/03-contracts/contract-fields.md` — Full field specification.
  - Required facts — canonical fields and their semantics:
    - **PURPOSE**: goals/intent. Load-bearing; proven critical (post 3889). Anthropic's Purpose Breakdown Structure: GPT always builds hierarchical goal decomposition from global → local; verbalizing is better than implicit.
    - **INPUTS**: `- type - name: description` per input (example in post 3878).
    - **OUTPUTS**: `- type - description`.
    - **KEYWORDS**: `[Keyword1, Keyword2]` — architectural patterns, task classifications. Hooks for RAG. (Cross-link T16 wenyan prompting.)
    - **LINKS**: cross-module references. Acts as Call Graph proxy; enables semantic module discovery.
    - **RATIONALE / AAG**: architectural intent in module contracts. (Cross-link T17.)
  - **Placement rule**: description goes BEFORE function/method declaration. Critical for causal-read anchor vector formation around the name token. (Post 3878.)
  - Python docstring can still exist BELOW declaration as SFT hint; sparse attention covers ~1000 recent tokens, so both layers contribute.
  - Concrete Python example (verbatim from post 3878):
    ```python
    # PURPOSE:Сохранение параметров в JSON файл.
    # INPUTS:
    # - dict - config: Словарь параметров
    # - str - config_path: Путь к файлу (по умолчанию "config.json")
    # OUTPUTS:
    # - bool - Статус успеха операции
    # KEYWORDS:[Saver, FileWrite]
    def save_config(config: dict, config_path: str = "config.json") -> bool:
        """Сохраняет переданный словарь параметров в JSON файл."""
    ```
  - Sources: posts 3878, 3889, 3890, 3892; `most_important.md` §"Top-down…".

- [x] **T16** `kb/03-contracts/wenyan-prompting.md` — Compression technique.
  - Required facts:
    - Term "wenyan" (文言): literary Chinese's extreme brevity.
    - KEYWORDS and LINKS sections ARE wenyan prompting (post 3819).
    - ~1:15 token compression vs full prose.
    - Grok reconstructed hidden meaning from compressed contract alone (no code); author's Cursor trainees observe 90% of semantic-search hits land on these "enrichment points".
    - Compressed keywords enable finding related modules without direct calls in code.
    - **ShortenDoc paper** (dl.acm 10.1145/3735636, post 3892):
      - 40% compression of docstrings without quality loss (sometimes improvement).
      - Full prepositional phrases cause degradation.
      - "Mutilating" text by keeping only key concepts: +~10% HumanEval.
      - Verified via perplexity.
    - Author's two-part validation before accepting a compression: logit prediction check + asking GPT to describe what the compressed phrase means (blind).
    - Phrase: "GPT already invented a secret transformer language; you just don't speak it yet" (author).
  - Sources: posts 3819, 3892.

- [x] **T17** `kb/03-contracts/rationale-and-aag.md` — Architectural intent.
  - Required facts:
    - RATIONALE / AAG sections live in **module contracts**, exposing architectural reasoning.
    - Research (ResearchGate pub 400340807, post 3890): code comments with architectural intent **improve bug-fix accuracy up to 3×**.
    - Attention-analysis: the AI agent literally attends to intent tokens when reconstructing broken logic.
    - Author's empirical discovery of the same effect predates the paper.
    - GRACE uses architectural-pattern references + RATIONALE/AAG sections to keep this in focus.
    - "Captain Obvious" comments (restating what code already says) are useless and can be harmful (post 3890).
  - Source: post 3890.

- [x] **T18** `kb/03-contracts/legacy-contract-recovery.md` — Adding contracts to old code.
  - Required facts:
    - **Google CodeT5+** fine-tuned for contract recovery (dl.acm 10.1145/3689484.3690738).
    - Used JML (Java Modeling Language) formal spec.
    - Recovered contracts at 90-97% quality depending on test class.
    - Focus: critical input/output conditions + invariants.
    - Author's empirical success with legacy contract injection (quality evaluated via LLM-reviewer, not formal tests).
    - Anton's contract-only MCP pattern (post 3622) is particularly useful for legacy: read only contracts + declarations, avoid exposing "ugly" human code that seeds bad patterns.
    - For AI-native code, prefer **greedy reading** (full modules) over contract-only.
  - Sources: `contracts_as_anchor.md` §"Научная опора"; post 3622.

- [x] **T19** `kb/03-contracts/language-specific/python.md` — Python contract placement.
  - Required facts:
    - Python comments (`#`) are not Dyck-friendly alone; add paired START-END tags in comments (Dyck-compatible in one pass; post 3948).
    - GRACE places PURPOSE/INPUTS/OUTPUTS/KEYWORDS **before** the `def`, as line comments.
    - Docstring stays BELOW `def`, as SFT-hint complement.
    - Author deviates from standard docstring convention because cross-language in-source standards place description BEFORE declaration, critical for causal-read anchor vector at the function-name token.
    - Full example reproduced from post 3878 (see T15).
  - Sources: posts 3878, 3892, 3948.

- [x] **T20** `kb/03-contracts/language-specific/csharp.md` — C# XML Doc mapping.
  - Required facts:
    - **XML Documentation for C#** highly compatible with GRACE.
    - `region` directives work as paired START-END tags for sparse attention.
    - IDE integration + automatic documentation generators support it out of the box — useful for search agents.
    - High generation stability: the format is familiar to LLMs.
    - **Summary** tag sub-optimal for AI — single-line summary equivalent to long; AI reads code fine.
    - Richer anchors for AI: **PURPOSE** + **KEYWORDS** + **LINKS** — fewer tokens, higher semantic value.
  - Source: post 3960.

- [x] **T21** `kb/04-markup-system/xml-like-markup.md` — Rules for XML-like markup.
  - Required facts:
    - XML and XML-like markup favored over JSON in long contexts (vendor-endorsed by OpenAI + Google).
    - Author-specific preference for lightweight XML-like (not full XML): fewer angle brackets, simpler tags.
    - Paired tags (START/END) give Dyck compatibility (cross-link T07, T22).
    - Tags should name semantic roles (not implementation details).
  - Sources: `xml_mapping_anchors.md`; posts 3948, 3960.

- [x] **T22** `kb/04-markup-system/start-end-tags.md` — Paired-tag markup.
  - Required facts:
    - Paired START/END tags around logical blocks in code or documentation.
    - **Two purposes**:
      1. Dyck compatibility — GPT parses in one pass without degradation.
      2. Enables AST-free parsing by simple algorithms (Anton's MCP approach, post 3622).
    - Allows building tooling that reads/edits code without a full compiler/parser.
    - Works in any language with comment syntax.
    - Powers the contract-only MCP pattern where `read_contracts` returns contract IDs + text + implementation line ranges, and `get_contracts_by_contract_path` walks PATH-header graphs (post 3622).
  - Sources: posts 3622, 3948.

- [x] **T23** `kb/04-markup-system/in-source-documentation.md` — Enterprise lineage.
  - Required facts:
    - GRACE-style markup is an **Enterprise-level variant** of "In-source documentation" / "Documentation as Code".
    - Every mature IDE has built-in support for one or more formats — "not using them is improper".
    - GRACE differs in **what** to put inside the format, not the format itself.
    - Classical in-source docs emphasize "description of function"; GRACE emphasizes **purpose/intent** and **correlation hooks** (KEYWORDS, LINKS).
    - Why: RL training rewards "goal achievement" → PURPOSE is an alignment anchor, not exposition.
  - Source: post 3878.

- [x] **T24** `kb/04-markup-system/markup-validators.md` — Validating GRACE markup.
  - Required facts:
    - A colleague built a GRACE-markup validator **using GRACE itself** (post 3904).
    - 1650 Python lines.
    - Complex syntactic-parsing algorithms (though grammar is simpler than full AST).
    - Included in autotests.
    - Trade-off vs LLM review: instant, no tokens, algorithmic — but can't catch semantic errors.
    - Author's default: LLM review concurrent with code review.
    - Both approaches have their place.
  - Source: post 3904.

- [x] **T25** `kb/04-markup-system/markup-compression.md` — Empirical bounds on compression.
  - Required facts:
    - In-source docs benchmarks (arxiv 2601.16661v1, post 3889):
      - **PURPOSE critical**: agent can skip DESCRIPTION entirely; not PURPOSE.
      - **Shorter is better**: long descriptions either neutral or harmful.
      - **English > other languages**: "sovereign AI" non-English comments cause degradation.
      - **Adjacency matters**: docs split from code by file-read or MCP-hop → degradation.
    - Effect sizes: **-95% degradation on bad structure, up to +435% improvement on good structure**.
    - ShortenDoc (T16) cross-link: 40% compression with no quality loss.
    - Practical heuristic: aggressively minimize, verify with logit or blind LLM-gloss test.
  - Sources: posts 3889, 3892.

### Wave 3 — Operational layer

- [x] **T26** `kb/05-logging-ldd/log-driven-development.md` — Overview.
  - Required facts:
    - LDD: instead of strict `assert` equality, the AI **reads structured logs and makes a semantic judgment** about correctness from the execution trajectory.
    - Why it exists: LLMs overfit to tests; asserts are brittle; trajectory analysis generalizes.
    - **Unlocks auto-testing for ERP-style logic** (1C, SAP) where business logic is too fuzzy for strict equality.
    - Foundation for the self-correction loop (cross-link T31).
  - Sources: `testing_and_autonomous_agents.md` §"Адаптация GRACE", §"Почему обычные автотесты".

- [x] **T27** `kb/05-logging-ldd/log-to-code-correlation.md` — The critical heuristic.
  - Required facts:
    - **Inject function ID + block ID tokens into every log line**.
    - Without this, AI cannot bind log back to code; Gemini attention collapses past ~5000 lines even with 1M context (post 3693).
    - With this: log + code merge into a single semantic monolith for GPT.
    - This is NOT optional — it is "archimportant" per author.
  - Source: post 3693.

- [x] **T28** `kb/05-logging-ldd/belief-state-logging.md` — AI's hypothesis-driven logs.
  - Required facts:
    - When the AI writes code, it verbalizes its **belief state** about how that code works.
    - Log lines record belief alongside observable state.
    - During analysis, AI compares "what I believed" vs "ground truth" — corrections are easy.
    - "Not always clear to humans, but GPT understands the concept perfectly" (author).
  - Source: post 3693.

- [x] **T29** `kb/05-logging-ldd/forced-context-injection.md` — Pushing log lines into context.
  - Required facts:
    - Some log lines are too important to trust the agent to fetch — **inject them forcibly** into the agent's context.
    - Category markup in logs aids filtering.
    - Essential for autonomous agents (cross-link T35, T36).
    - Assuming "an agent will call Tools when confused" is naive — especially if looping.
  - Sources: posts 3693, 3695.

- [x] **T30** `kb/06-testing/classical-tests-antipattern.md` — Why classical TDD hurts AI.
  - Required facts:
    - Tests are "just more code" from the AI's perspective, and unusually hostile: they do heavy cross-module referencing, which burns sparse-attention budget.
    - LLMs **overfit to test conditions** — contract-in-context + tests-in-context collapses solution space to "whatever passes tests".
    - Classical TDD is designed for humans, not models.
    - Most autotests are smoke tests (`did it crash?`) and don't catch real bugs anyway.
    - LLM-written tests are low quality: training saw "tests+code" pairs but no meta-reasoning about tests.
    - Quote: "LLM is a self-taught parrot when it comes to tests".
  - Sources: `testing_and_autonomous_agents.md` §"Почему обычные автотесты", §"Еще один пост про 'тесты как шум'".

- [x] **T31** `kb/06-testing/tests-in-self-correction-loop.md` — Tests relocated to debug.
  - Required facts:
    - Modern trend: tests moved from the **analytical phase** to the **self-correction loop** during debug.
    - New job of tests: "help the model fix a specific bug", not "help the model understand the task".
    - Both **binary and detailed signals** matter; `TEST FAIL` alone is bad practice.
    - Modern frameworks flood the model with "flag"-style conclusions (structured, easy for GPT to parse).
    - SWE-Bench-style RL trains the model to read log fragments and emit patches.
  - Source: `testing_and_autonomous_agents.md` §"Тесты как часть self-correction loop".

- [x] **T32** `kb/06-testing/agent-based-testing.md` — Developer-agent / tester-agent split.
  - Required facts:
    - Complex cases (e.g., ETL rewrite): the AI-developer writes a **Markdown guide** for an AI-tester on how to test.
    - If the same testing discipline were code, it would be huge modules; as Markdown for an agent, it's compact.
    - On failure: tester-agent **cuts failing data to XML**, sends a detailed report to the developer-agent.
    - Developer-agent updates code, manuals, and tests.
    - Moral: classical autotests for complex ETL often fail to launch at all with AI; agent-based is more reliable.
    - LLMs will often produce code that passes autotests but doesn't work in reality — agent testing catches this.
  - Source: `testing_and_autonomous_agents.md` §"Agent-based testing при переписывании legacy".

- [x] **T33** `kb/06-testing/in-app-ai-console.md` — Embedded AI tester.
  - Required facts:
    - Install an **AI agent WITH a console inside your application**, with the API wired to Tools.
    - The agent can run tests much more sophisticated than pytest scripts and produce proper reports.
    - Author claim: takes ~2 hours to build; people are just lazy about it.
    - GUI-based AI testing is slow and expensive; CLI (pytest-backed) is the pragmatic fallback.
  - Source: `testing_and_autonomous_agents.md` §"Почему обычные автотесты".

- [x] **T34** `kb/06-testing/cli-not-custom-mcp.md` — Reliability of standard tools.
  - Required facts:
    - Bash is in the **standard LLM toolset** → reliable inside long trajectories.
    - **pytest is reliable "like a rock"** — in training.
    - Author tried a "kulibin CLI" alongside pytest — the LLM ignored it or hallucinated around it. Removed.
    - Top-1000 MCPs are in training (per Kimi paper) → Asana/JIRA MCPs likely work; custom ones likely don't.
    - Moral: AI dislikes DIY tooling because it wasn't trained on it; for MCPs it can silently refuse to use them.
  - Source: post 3834.

- [x] **T35** `kb/07-autonomous-agents/anti-loop-protection.md` — Preventing autonomous loops.
  - Required facts:
    - Failed fixes become **few-shots** for the model → it becomes more confident about the same failed approach.
    - A swarm or single autonomous agent can **burn billions of tokens** looping.
    - Protections: **attempt counters** + **context injections** (force sanity back into context).
    - Must be designed INTO the test framework from the start.
    - Regular autotests are **not compatible** with autonomous agents without these protections.
  - Source: post 3695.

- [x] **T36** `kb/07-autonomous-agents/forced-context.md` — Stabilization mechanism.
  - Required facts:
    - **Forced context**: deliberately push information into an agent's context rather than hoping it calls the right Tool.
    - Particularly important when the agent is looping or stuck.
    - Examples: key log lines, prior-fix knowledge, critical constraints.
    - Naive assumption that "the agent will call Tools to gather needed info" fails in practice.
  - Sources: posts 3695, 3693.

- [x] **T37** `kb/07-autonomous-agents/knowledge-base-in-tests.md` — Regression memory.
  - Required facts:
    - Tests integrated with a **knowledge base of previous fixes**.
    - On regression, force the relevant KB entries into the agent's context.
    - Prevents repeat-failure loops by reminding the agent of the past.
    - Part of anti-loop protection infrastructure.
  - Source: post 3695.

- [x] **T38** `kb/07-autonomous-agents/swarm-vs-single-agent.md` — Scaling modes.
  - Required facts:
    - Open Code (Kilo Code core) supports multi-agent and single-agent with the **same prompts** — switch without rewriting.
    - 16-subagent Claude Opus swarm produced a book from video training material (post 3859); orchestrator consolidated.
    - Swarm architecture best for Architect-phase work; single agent better for focused debug.
    - Forced context + anti-loop protection become non-negotiable at swarm scale.
  - Sources: posts 3793, 3859.

- [x] **T39** `kb/08-workflow-and-phases/top-down-pipeline.md` — The three phases.
  - Required facts:
    - **Phase 1 — Architectural graph**: AI models app as graph of main classes/modules; business-process modeling and its mapping to architecture are in the same graph.
    - **Phase 2 — Module contracts**: from graph, AI produces contract per module, expanding intent.
    - **Phase 3 — Function contracts + code**: contracts of functions/methods are **local refinements** of overall app intent at that site, not spontaneous AI creativity.
    - AI must be **assisted** through top-down — on its own it jumps to code.
    - Result: near-elimination of drift from original goals; zero-doc (architecture+business embedded).
  - Sources: `most_important.md` §"Top-down логика и переход к контрактам".

- [x] **T40** `kb/08-workflow-and-phases/architect-code-debug-modes.md` — Kilo Code mode mapping.
  - Required facts:
    - Kilo Code UI has explicit **Architect / Code / Debug** modes with associated prompts.
    - Maps directly onto GRACE's phase model — makes the methodology visually explicit.
    - Cursor can also hold a RAG of generation prompts but lacks explicit mode UI → GRACE less ergonomic there.
    - Architect mode is the upstream: few, large, high-stakes prompts.
    - Debug mode uses LDD + self-correction.
  - Source: `practical_applications.md` §"GRACE + PCAM и интерфейс Kilo Code".

- [x] **T41** `kb/08-workflow-and-phases/large-rare-prompts-billing.md` — Economics of the workflow.
  - Required facts:
    - GRACE favors **few, large, structured prompts** over many small ones.
    - In Architect mode, an entire design session is typically **~20 prompts generating 300–400k tokens** of design context.
    - Reaching even 1000 requests/day is hard with GRACE discipline.
    - **Google Gemini Code Assist** ($20–$40/mo flat, ~2000 requests/day) fits better than per-token API billing.
    - Link: developers.google.com/gemini-code-assist/resources/quotas.
  - Source: `practical_applications.md` §"Большие, но редкие запросы".

### Wave 4A — Tooling + beyond-code

- [x] **T42** `kb/09-tooling/kilo-code-open-code.md` — Kilo Code migration.
  - Required facts:
    - Kilo Code migrated from **Cline** to **Open Code** core.
    - Cline was "a sinking Titanic" after team departed to OpenAI; Gemini compatibility degraded; Aider-patch legacy introduced bugs.
    - Open Code: ~2× more token-efficient; direct Gemini API via VPN (no OpenRouter proxy); faster.
    - Flexible: swap single-agent ↔ multi-agent without changing prompts.
    - Swarm support is the main evolutionary gain.
    - Kilo Code offers $100 in tokens for bug reports.
    - Author migrated GRACE to Open Code (post 3793) then to dynamic skills (post 3809).
    - Reference: blog.kilo.ai/p/we-completely-rebuilt-the-kilo-vs-code-extension.
  - Sources: posts 3793, 3809.

- [x] **T43** `kb/09-tooling/claude-code-grace-plugin.md` — osovv grace-marketplace.
  - Required facts:
    - Alexey Chendemerov built a Claude Code plugin implementing GRACE ideas.
    - Author didn't audit in detail but confirmed the core ideas are right: **graph on code, START-END markup, contracts**.
    - GitHub: github.com/osovv/grace-marketplace.
    - Author normally implements GRACE via prompt frameworks, not plugins.
  - Source: `practical_applications.md` §"Плагин для Claude Code".

- [x] **T44** `kb/09-tooling/cursor-limitations.md` — Cursor's chunked reads.
  - Required facts:
    - Cursor reads code/logs in chunks of 100–200 lines via `read_file`.
    - Only the first ~100 lines are **guaranteed** to be read.
    - Beyond that: text or semantic search.
    - Without semantic markers, the AI is "blinded" in Cursor.
    - With GRACE markers, ~90% of Cursor's semantic searches land on the enrichment points (KEYWORDS/LINKS).
    - For very long logs, copy to Gemini's 1M-token window (still needs markers).
  - Sources: `xml_mapping_anchors.md` §"Почему семантическая разметка"; post 3819.

- [x] **T45** `kb/09-tooling/mcp-scepticism.md` — When MCPs work and when they don't.
  - Required facts:
    - LLMs **weren't trained on custom MCPs**; they often ignore or hallucinate around them.
    - Exception — the **top-1000 MCP servers** are in training (per Kimi paper): Asana/JIRA-tier work.
    - **Anton's contract-only MCP** (post 3622) is a working exception: legacy code, returns only contracts + declarations, strongly typed around GRACE markup.
    - Sergei Muravyev's doc-MCP discussion (post 3821): agent's **primary task is "find modules"**, not "understand class" — an MCP that serves docs misdefines the task.
    - FIM training biases agents to load whole modules (Claude Code has "greedy reading" prompt for subagents).
    - Custom MCPs compete with $10B training supercluster work — they lose.
    - Reference: github.com/comol/cursor_rules_1c.
  - Sources: posts 3622, 3820, 3821, 3834.

- [x] **T46** `kb/09-tooling/skills-system.md` — Making skills actually load.
  - Required facts:
    - Risk: **LLM can decide "I already know" and skip loading a skill**. Scientifically-noted danger.
    - Only dilettantes drop skills in blindly.
    - Mitigations:
      - **MANDATORY MODE** / **MANDATORY PROTOCOL** trigger words to signal "this is critical, not optional KB".
      - **Forced tracing**: one skill explicitly names the next skill to load (by name), forming a reliable chain.
    - In **Open Code**, you can strengthen this by explicitly requiring the `skill` tool call.
    - With trace-based loading, dynamic skills are as reliable as trace-based dynamic prompts.
  - Source: post 3809.

- [x] **T47** `kb/09-tooling/hybrid-rag-infra.md` — Storage decisions.
  - Required facts:
    - Pure vector search is insufficient for exact facts, numbers, citations.
    - **sqlite-vec**: combines SQL exact-match with vector similarity — recommended (github.com/asg017/sqlite-vec).
    - Graph DB not strictly required if chunks are enriched.
    - GRACE's KEYWORDS/LINKS enrichment can replace a separate Call Graph for many needs.
    - For code specifically, the LINKS pattern in contracts is usually enough.
  - Source: post 3732.

- [x] **T48** `kb/10-beyond-code/grace-for-rag-documents.md` — Document-RAG extension.
  - Required facts:
    - **Mikhail Evdokimov** (CPO ALGA Group International; 1st Deputy Chairman of O!Bank) built a RAG system on GRACE principles for documents (post 3839).
    - Structure: graph-based main map + per-document **semantic squeeze** (analog of code contracts).
    - Author notes at least **three private bank cases** with similar approaches.
    - Point: GRACE is a **general methodology for complex text**, not only code.
    - Templates are standardized for code; adaptation for other domains requires template evolution.
    - Reference: habr.com/ru/articles/1020548/.
  - Source: post 3839.

- [x] **T49** `kb/10-beyond-code/general-complex-text.md` — Methodology generalization.
  - Required facts:
    - Author has used the same graph-first approach for **construction work planning** (MS Project methodology graph).
    - Methodology-as-graph argument (cross-link T11).
    - Any methodology that can be graphed is operable by AI; one that can't is "text entropy" to it.
    - A useful side-effect: graphing a methodology exposes "water" — GPT drops filler nodes.
  - Source: `most_important.md` §"Методология как граф".

### Wave 4B — Cases, economics, antipatterns, research

- [x] **T50** `kb/11-case-studies/kirill-132k-loc.md` — Large codebase markup adoption.
  - Required facts (post 3620):
    - Project: pre-stage product.
    - 1494 files, 11,481 lines of docs.
    - **132,666 prod LOC + 50,026 test LOC** (no comments); 11,771 prod comments + 1,761 test comments.
    - Migrated to Kilo Code with GPT-5.4(xh) orchestrator + GPT-5.4(m) workers.
    - **20+ hours continuous autonomous run** after initial prompt setup.
    - Two latent bugs surfaced during markup annotation.
    - After switch: agents read **fewer unnecessary files**; zero rollbacks required; noticeably less hallucination.
    - Per-file tokens increased but total is lower because fewer files read.
    - Logs also modernized — standard logging "obsolete".
  - Source: post 3620.

- [x] **T51** `kb/11-case-studies/vlad-crm-and-contracts.md` — Two Vlad cases.
  - Required facts:
    - **Case A — CRM** (practical_applications): colleague Vlad, ex-SAP; CRM for 5 security companies, 1000+ personnel per shift; weapons+docs+shift tracking; Next.js + shadcn + Supabase. Multi-model switching (Gemini Flash/Pro, Claude Opus 4.6, GLM-5, Kimi K2.5). Reported observation: GRACE markup + specs made models feel interchangeable — "you don't feel a difference between models, all generate equally well". Author takeaway: if you feel high LLM sensitivity, your pipeline is weak, not the model.
    - **Case B — Contracts project** (post 3693): same colleague, different project. **282 contracts, 1049 autotests**. Applied GRACE contracts well but **lacked GRACE logging**. Agent rightly reflected: "tests without logs = vodka without beer". Motivating post for log-to-code correlation + belief-state logging.
  - Sources: `practical_applications.md` §"Пример CRM"; post 3693.

- [x] **T52** `kb/11-case-studies/alexey-300k-loc.md` — Largest reported.
  - Required facts (post 3838):
    - **300,000+ LOC**.
    - Built with GRACE.
    - At US developer rates, such a project would cost **$7M+** to build traditionally.
    - Signals the efficiency ceiling achievable with current AI-native workflow.
  - Source: post 3838.

- [x] **T53** `kb/11-case-studies/mikhail-bank-rag.md` — Bank RAG.
  - Required facts:
    - Mikhail Evdokimov — CPO ALGA Group International; 1st Deputy Chair of O!Bank.
    - Built a RAG system using GRACE-style graph + semantic-squeeze per document.
    - At least three private bank cases exist with similar approaches.
    - Reference: habr.com/ru/articles/1020548/.
    - Implication: GRACE works for complex enterprise documents, not only code.
  - Source: post 3839.

- [x] **T54** `kb/11-case-studies/qwen-slm.md` — SLM viability.
  - Required facts (post 3902):
    - Model: **Qwen 3.5-27B**, SWE-Bench Verified **72.4%** (previously unthinkable for 27B params).
    - Ran GRACE cleanly — more **disciplined** with in-source docs than larger LLMs.
    - Failed on one training example: Gradio framework knowledge absent (expected at 27B weights; base Python/React OK).
    - Good for: **tests, bug reports, bug fixing on existing code**.
    - Author-claim: SLMs are now working tools; find the right niche.
    - Reference: huggingface.co/Qwen/Qwen3.5-27B.
  - Source: post 3902.

- [x] **T55** `kb/11-case-studies/grace-validator-self-hosted.md` — Tool built using GRACE.
  - Required facts (post 3904):
    - Colleague built a GRACE-markup validator **using GRACE** to manage agents.
    - ~1650 Python lines.
    - Performs algorithmic validation of markup in an agent's output.
    - Added to project's autotest suite.
    - Complementary to LLM-based review — instant, token-free, deterministic.
    - Author's default remains LLM review in parallel with code review.
  - Source: post 3904.

- [x] **T56** `kb/12-economics-strategy/vendor-strategy.md` — Why big labs won't build this.
  - Required facts (post 3708):
    - Big vendors (Anthropic, Google, Microsoft) target $1B+/year products.
    - Under **$100M/year**: "seeds", not business. Explicitly closed by MS for MS Project Server.
    - Ecosystem model: vendors rely on partners; directly competing with partners kills the ecosystem.
    - **MCS (Microsoft Consulting Services)** pattern: vendor does first reference cases, helps small partners enter giant accounts (e.g., Gazprom, Boeing), then steps back.
    - Author prediction: no full-pipeline GRACE-tier framework from Anthropic/Google/MS; they'll absorb techniques into training distribution instead.
    - Gemini (via free-tier bot) has already "absorbed" author's methodology into Gemini's understanding of the author's prompts.
  - Source: post 3708.

- [x] **T57** `kb/12-economics-strategy/project-size-trends.md` — Scale trajectory.
  - Required facts (post 3837):
    - Current 100%-AI-made apps: ~**100,000 LOC** (author's training-data observation across dozens of projects/month).
    - Users trend: engineers who **don't code in target frameworks** themselves — they manage design/coding/testing agents.
    - Apps are **vertical/niche** solutions for specific industry cases; largely invisible to those searching horizontal markets.
    - 1-2y projection: **400-500K LOC** per operator managing a bot swarm.
  - Source: post 3837.

- [x] **T58** `kb/12-economics-strategy/division-of-labor.md` — Designers vs users.
  - Required facts (post 3804):
    - Testing whether one contract-section works across all tests: **4–8 hours per iteration**.
    - Individual developers don't have time to build full frameworks + keep up with current LLM research.
    - One paper properly digested = ~1 person-day.
    - Efficient division: **framework designers** (deep experimentation) vs **framework users** (adaptation + application).
    - Users must take "experiment experience", not just the framework — adaptation is always needed.
  - Source: post 3804.

- [x] **T59** `kb/13-antipatterns/all-antipatterns.md` — Consolidated antipattern catalog.
  - Each antipattern a subsection: `## NN — Name` → What it is / Why wrong / What to do instead / Source refs.
  - Required subsections:
    - **Vibe-coding-at-scale** — pet-project habits don't scale; 5-10k LOC collapse threshold (post 3654).
    - **Classical TDD with AI** — overfitting, semantic noise, imported anthropomorphism.
    - **External documentation systems** — MCP/graph-DB docs AI was never trained to use (post 3820, 3821).
    - **Custom MCP tools ("kulibin")** — DIY Tools/MCPs that LLM ignores (post 3834).
    - **YAML configuration** — not Dyck-compatible; banned by Google in Gemini sandbox (post 3948).
    - **JSON in long context** — OpenAI explicitly warns against.
    - **"Sovereign AI" non-English comments** — benchmarks show degradation (post 3889).
    - **"Captain Obvious" comments** — restating what code says; useless/harmful (post 3890).
    - **Long summary-style descriptions** — no benefit over short; sometimes degradation (post 3889, 3892).
    - **TEST FAIL binary signals** — modern frameworks provide flag-rich logs.
    - **Assuming the agent will self-rescue via Tools** — especially when looping (post 3695).
    - **Skills without forced loading** — LLM will skip "I already know" skills (post 3809).
    - **Using strict assertions as primary truth** — overfits; LDD is more reliable for business logic.
  - Sources: across all files.

- [x] **T60** `kb/14-research-references/annotated-bibliography.md` — Cited works with annotations.
  - Each entry follows:
    ```
    ## <Title>
    - **Authors**: <if given in sources, else "not specified in sources">
    - **Venue / URL**: <given URL>
    - **What it proves**:
    - **How GRACE uses it**:
    - **Source post**: (post ####)
    ```
  - Required entries (at minimum):
    - **OpenAI GPT-4.1 Prompting Guide** — cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters — endorses XML-like over JSON in long context (xml_mapping_anchors.md).
    - **Lee et al.: "Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?"** — referenced from OpenAI guide (xml_mapping_anchors.md); sources do not specify arxiv ID.
    - **Shunyu Yao et al. on Dyck Language in Transformers** — arxiv.org/abs/2105.11115 — native Dyck support proof; underpins XML/HTML/JSON native processing and YAML incompatibility (post 3948).
    - **CodeT5+ / JML contract recovery** — dl.acm.org/doi/10.1145/3689484.3690738 — 90-97% contract recovery quality on legacy Java (contracts_as_anchor.md).
    - **In-source documentation benchmarks** — arxiv.org/abs/2601.16661v1 — PURPOSE critical, shorter > longer, English > other, adjacency matters; -95% to +435% swings (post 3889).
    - **ShortenDoc** — dl.acm.org/doi/10.1145/3735636 — 40% compression without quality loss, ~10% HumanEval gain (post 3892).
    - **Impact of Code Comments on Automated Bug-Fixing** — researchgate.net/publication/400340807 — architectural intent → 3× bug-fix accuracy; attention analysis (post 3890).
    - **Kimi paper on MCP training distribution** — referenced in post 3834; sources do not specify URL; cite as "per Kimi paper".
  - Sources: all 6 files.

### Final pass — sequential

> After all T02-T60 are `- [x]`, one agent completes these in order:

- [x] **T01** `kb/README.md` — Entry-point and navigation.
  - Brief "What is GRACE" (one paragraph, link to `00-foundations/what-is-grace.md`).
  - How this KB is organized (directory overview).
  - Reading paths by audience:
    - "I just want the core idea" → `00-foundations/` + `02-semantic-graph/graph-as-backbone.md` + `03-contracts/contracts-overview.md`.
    - "I need to implement this today" → `08-workflow-and-phases/` + `09-tooling/` + `03-contracts/contract-fields.md`.
    - "I want the research backing" → `14-research-references/` + `01-transformer-foundations/`.
    - "I'm debugging agents that don't scale" → `05-logging-ldd/` + `07-autonomous-agents/` + `06-testing/`.
  - How the KB was built (methodology note; link to PLAN.md).

- [x] **INDEX** `kb/INDEX.md` — Full cross-reference topic map.
  - Topic → file(s) map for every indexed concept.
  - Reverse-lookup from source posts: post# → files that cite it.
  - Research papers → files that cite them.

- [x] **SWEEP** Final sweep (no file output — edits).
  - Link-check: every `See also` link resolves.
  - Duplication check: no fact owned by two files without cross-reference.
  - Consistency: claim-type labels used uniformly.
  - Citation coverage: every section has at least one citation.
  - Typo/translation pass.

---

## 8. How to mark a task complete

When an agent completes its task, it edits PLAN.md to change the task's checkbox. Example transformation for T07:

**Before**:
```
- [ ] **T07** `kb/01-transformer-foundations/tc0-and-dyck-language.md` — Formal basis for XML preference.
```

**After**:
```
- [x] **T07** `kb/01-transformer-foundations/tc0-and-dyck-language.md` — Formal basis for XML preference.
```

Use the Edit tool with a `old_string` that includes the task ID (`T07`) so the match is unique. Verify with Read.

If two agents race and one Edit fails because the file changed, retry. The task ID is unique so the target line is always identifiable.

---

## 9. Status log

Append a one-line note here after each wave completes. Main Claude maintains this.

- 2026-04-23 — kb/ directory + PLAN.md created. Wave 1 launch pending.
