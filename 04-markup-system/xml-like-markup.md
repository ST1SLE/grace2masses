# XML-like Markup

## What

**XML-like markup** is GRACE's baseline format for structuring in-source documentation, contracts, and agent-facing prompt content. It is a **lightweight variant** of XML — paired angle-bracket tags that name semantic roles, nested according to Dyck-Language rules — but without the full XML ceremony (namespaces, attributes, DTDs, entity encoding).

The author explicitly aligns GRACE with prompting patterns used "by Google promoters" and by OpenAI guidance, while keeping tags simpler than full XML documents. [author-claim] (`xml_mapping_anchors.md` §"Почему автор предпочитает XML-like разметку")

> Note: this file covers the **rules of XML-like markup design** in GRACE. The empirical and formal comparison of XML vs JSON vs YAML lives in `../01-transformer-foundations/xml-vs-json-vs-yaml.md`. The specific paired-tag mechanism (START/END) that enables AST-free parsing lives in `./start-end-tags.md`.

## Why

Four reinforcing reasons make XML-like the default markup in GRACE:

### 1. Vendor endorsement — OpenAI + Google

OpenAI's official GPT-4.1 Prompting Guide **explicitly advises against JSON in long contexts** and recommends XML or XML-like markup instead. [vendor-doc] (`xml_mapping_anchors.md` §"Почему автор предпочитает XML-like разметку"; cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters)

That guide references the Lee paper "Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?" as the empirical basis. [vendor-doc] (`xml_mapping_anchors.md`; sources do not specify the full arxiv ID)

Google's prompting teams also stand behind XML-like markup. [author-claim] (`xml_mapping_anchors.md` §"Почему автор предпочитает XML-like разметку")

The author frames this as a vendor alignment signal: "for ordinary mortals it's time to follow the vendor's guidance, not argue with it." [author-claim] (`xml_mapping_anchors.md`)

### 2. Formal basis — Dyck-Language compatibility

Without chain-of-thought, GPT is limited to **TC⁰** discrete logic, and TC⁰ natively supports **Dyck Language** (paired-bracket languages formulated by Walter von Dyck). [research-backed] (post 3948)

XML, HTML, JSON, and bracket-based programming languages are **Dyck-compatible** formats — GPT can parse them in a single pass without degradation. YAML is **not** Dyck-compatible. [research-backed] (post 3948)

This means XML-like markup sits on the most mathematically favorable format class for transformer parsing. See `../01-transformer-foundations/tc0-and-dyck-language.md` for the full proof chain.

### 3. Empirical — JSON brace-counting collapse in long context

Through logit analysis, the author observes the mechanism of JSON degradation in long contexts: the AI effectively "counts braces" and loses track at scale, while the `{}` literals also trigger spurious correlations that pull attention away from the real payload. [empirical] (`xml_mapping_anchors.md` §"Почему автор предпочитает XML-like разметку")

The same effect appears in Qwen models — it is a **universal transformer reaction**, not a single vendor's quirk. [empirical] (`xml_mapping_anchors.md`)

Convergence on JSON is also slower than on XML or XML-like markup at comparable payload size. [empirical] (`xml_mapping_anchors.md`)

### 4. Markup lets the RAG agent find anchors past 4k tokens

Past ~4k tokens, GPT switches to sparse attention via 100–200-token sliding windows stitched together by semantic anchors. [research-backed] (`most_important.md` §"Почему без семантического графа большие контексты разваливаются")

Without XML-like tags the agent falls back to random attention or chunked heuristics (Cursor reads 100–200-line chunks via `read_file` and only the first ~100 lines are **guaranteed** to be read). [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка логов важна")

With XML-like tags ~90% of Cursor's semantic searches land on the enrichment points (KEYWORDS / LINKS). [empirical] (post 3819)

Without markup, the log or code is "blinded" for the agent — just **"chaotic text entropy"** even to Gemini's 1M-token window. [author-claim] (`xml_mapping_anchors.md`)

## How

GRACE's rules for designing XML-like markup.

### Rule 1 — Favor XML or XML-like over JSON in long context

Hard rule from the vendors: do not use JSON as the primary structure holding long-context payload. [vendor-doc] (OpenAI GPT-4.1 Prompting Guide; `xml_mapping_anchors.md`)

JSON inside **short** spans (e.g., a single API payload inside one attention window) is fine; the problem is JSON wrapping hundreds or thousands of lines of content. [empirical] (`xml_mapping_anchors.md`)

If you find someone pushing JSON as the backbone of a long-context RAG system, it is a legitimate question whether that was a good choice. [author-claim] (`xml_mapping_anchors.md`)

### Rule 2 — Convert YAML to a Dyck-compatible format

YAML is not Dyck-compatible; Google **banned YAML libraries** in Gemini's Code Execution sandbox to prevent Gemini from degrading when trying to use YAML as a common format. [research-backed] [vendor-doc] (post 3948)

The author recommends: **if something is stored in YAML, rework it into a Dyck-Language-compatible format.** YAML frequently gets inserted by "second-role" vendor engineers out of incompetence, not by deliberate design. [author-claim] (post 3948)

### Rule 3 — Use lightweight XML-like, not full XML

The author uses **XML-like** — not strict XML. Characteristics:

- Paired start/end tags naming semantic roles.
- Simpler than full XML — fewer attributes, no namespaces, no DTD discipline.
- In Python, paired `START-END` tags are embedded inside `#` comments because Python comments alone are not Dyck-friendly; adding paired tags makes them parsable in one pass. [author-claim] [research-backed] (post 3948)

The goal is the **Dyck-compatibility and semantic-role naming**, not ceremonial XML compliance. [author-claim] (post 3948; `xml_mapping_anchors.md`)

### Rule 4 — Tags name semantic roles, not implementation details

The author's contract markup uses role-bearing uppercase tags / section names such as **PURPOSE**, **INPUTS**, **OUTPUTS**, **KEYWORDS**, **LINKS**, **RATIONALE / AAG**. [empirical] (post 3878, post 3890)

These are **alignment anchors** for the agent — PURPOSE is not exposition, it is the hook that RL-trained models (trained to "achieve goals") bind to. [author-claim] [research-backed] (post 3878, post 3889)

"Captain Obvious" tags that restate what code already says are useless and can be harmful; what matters is architectural intent and correlation hooks. [research-backed] (post 3890)

See `../03-contracts/contract-fields.md` for the full field specification.

### Rule 5 — Paired tags for Dyck compatibility and AST-free parsing

Every logical block should be wrapped in **paired start / end tags** (START/END-style). This serves **two purposes**:

1. **Dyck compatibility** — the agent parses the block in one transformer pass without degradation. [research-backed] (post 3948)
2. **AST-free parsing** — simple algorithms can read and edit code without a full compiler or AST parser. Anton's contract-only MCP is built on exactly this — `read_contracts` returns contract IDs + text + implementation line ranges, and `get_contracts_by_contract_path` walks PATH-header graphs, all without AST. [empirical] (post 3622)

The contract-only MCP pattern works **particularly well for legacy code** (human-authored, often "face-palm" quality — better not to feed the agent bad patterns wholesale); for AI-native code the preferred strategy is "greedy reading" of whole modules. [empirical] (post 3622)

See `./start-end-tags.md` for the full specification of the paired-tag mechanism.

### Rule 6 — Place the markup BEFORE the declaration

Across languages, cross-language in-source-documentation conventions put descriptions **before** the function / method declaration. This is critical for **causal-read anchor vector formation** around the name token — the agent reads forward, and the markup seeds the anchor the agent will attach to when referencing the function later. [author-claim] (post 3878)

In Python, this means **PURPOSE / INPUTS / OUTPUTS / KEYWORDS go ABOVE the `def`**, as line comments. A docstring can still exist BELOW the `def` as an SFT-hint layer — sparse attention covers ~1000 recent tokens, so both layers contribute. [author-claim] [research-backed] (post 3878, post 3892)

Concrete Python example (verbatim from post 3878):

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
    ...
```

See `../03-contracts/language-specific/python.md` for the Python-specific placement rules.

### Rule 7 — Reuse the language's built-in markup where it is already Dyck-compatible

"Every respectable environment has a built-in format; not using it is improper." [author-claim] (post 3878)

GRACE differs from classical in-source documentation in **what** to put inside the format, not in the format itself. [author-claim] (post 3878)

Worked example — C#: **XML Documentation for C#** has very high compatibility with GRACE. `region` directives work as paired start-end tags for sparse attention; the format is a familiar standard for LLMs (high generation stability); IDE integration + automatic documentation generators support it out of the box, which helps search agents. [author-claim] (post 3960)

The author notes that the classical `Summary` tag is sub-optimal for AI — a one-line summary is equivalent to a long one, since the AI can read the code itself. What actually carries semantic weight for AI is **PURPOSE + KEYWORDS + LINKS** — fewer tokens, higher semantic value. [author-claim] (post 3960)

### Rule 8 — Keep the markup short; verify compression before adopting

Long textual summaries either neutralize or degrade — short semantic hooks are strictly better. [research-backed] (post 3889, post 3892)

The ShortenDoc paper shows **40% compression of docstrings without quality loss** (sometimes +~10% HumanEval); full prepositional phrases cause degradation; "mutilating" text by keeping only key concepts helps. [research-backed] (post 3892; dl.acm.org/doi/10.1145/3735636)

Before adopting a compressed tag or keyword in GRACE, the author runs two validations:

1. **Logit prediction check** — does the model correctly predict downstream tokens from the compressed form?
2. **Blind LLM gloss test** — without showing the model the code or instructions, ask it to describe what the compressed phrase means. If both pass, the compression goes in. [empirical] (post 3892)

Context: "GPT already invented a secret transformer language; you just don't speak it yet." [author-claim] (post 3892)

See `./markup-compression.md` for the full compression story, and `../03-contracts/wenyan-prompting.md` for the KEYWORDS/LINKS compression technique.

### Rule 9 — English-only inside the markup

The in-source-documentation benchmark paper (post 3889; arxiv.org/abs/2601.16661v1) shows that **English is the best language for code comments**; other ("sovereign AI") languages cause agent degradation. [research-backed] (post 3889)

Effect sizes in the paper: from **–95% degradation on bad structure** to **up to +435% improvement on good structure**. [research-backed] (post 3889)

### Rule 10 — Adjacency matters

Code descriptions must be **"maximally pressed against" the code**. If the description is split from the code across a file-read boundary or an MCP hop, the agent degrades — the attention anchor is lost. [research-backed] (post 3889)

Practical consequence: **keep GRACE markup in-source, adjacent to the declaration it describes.** Do not externalize it to a separate documentation file or an MCP documentation server.

This is also why **external documentation systems** (MCP docs, doc graph DBs) fight against the training distribution. Base FIM pretraining contained a **total ban on documentation**; SFT and RL contained no documentation. If the agent is forced to use external docs, it hallucinates "plausibly"; left alone, it will just read the code. [research-backed] [empirical] (post 3820, post 3821)

See `../04-markup-system/in-source-documentation.md` for the lineage and `../09-tooling/mcp-scepticism.md` for when external MCP docs do and do not work.

## Evidence

### Vendor documents

- **OpenAI GPT-4.1 Prompting Guide** — cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters — endorses XML-like over JSON in long context. [vendor-doc] (`xml_mapping_anchors.md`)
- **Lee paper referenced by the OpenAI guide**: "Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?" — sources do not specify the full arxiv ID. [research-backed] (`xml_mapping_anchors.md`)

### Research

- **Shunyu Yao et al. on Dyck Language in Transformers** — arxiv.org/abs/2105.11115 — native Dyck support proof for transformers; underpins XML / HTML / JSON native processing and YAML incompatibility. [research-backed] (post 3948)
- **In-source documentation benchmarks** — arxiv.org/abs/2601.16661v1 — PURPOSE critical, shorter > longer, English > other, adjacency matters; –95% to +435% swings. [research-backed] (post 3889)
- **ShortenDoc** — dl.acm.org/doi/10.1145/3735636 — 40% docstring compression without quality loss, ~10% HumanEval gain. [research-backed] (post 3892)
- **Architectural-intent comments → 3× bug-fix accuracy** — researchgate.net/publication/400340807 — attention analysis shows the agent literally attends to intent tokens. [research-backed] (post 3890)

### Empirical observations

- **Grok reconstructed hidden meaning from a KEYWORDS/LINKS-compressed contract alone**, without the code. Compression ratio ~1:15 vs expanded prose. [empirical] (post 3819)
- **90% of Cursor semantic searches land on KEYWORDS/LINKS enrichment points** — Cursor trainees' reported observation. [empirical] (post 3819)
- **Google banned YAML libraries** in Gemini's Code Execution sandbox to prevent YAML-induced LLM degradation. [research-backed] [vendor-doc] (post 3948)
- **Qwen shows the same JSON-degradation effects via logit analysis** — universal, not single-vendor. [empirical] (`xml_mapping_anchors.md`)

### Posts cited

- Post 3619 / 3620 — case study showing concrete pre-stage adoption of GRACE markup (1494 files, 132k prod LOC).
- Post 3622 — Anton's contract-only MCP built on START-END markup.
- Post 3819 — wenyan prompting via KEYWORDS/LINKS.
- Post 3878 — PURPOSE / INPUTS / OUTPUTS / KEYWORDS placement and Python example.
- Post 3889 — in-source documentation benchmark study.
- Post 3890 — architectural intent → 3× bug-fix accuracy.
- Post 3892 — ShortenDoc 40% compression result.
- Post 3948 — TC⁰ and Dyck Language formal argument; YAML non-compatibility; Google sandbox YAML ban.
- Post 3960 — C# XML Documentation mapping to GRACE.

## See also

- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis for XML / HTML / JSON native transformer support and YAML incompatibility.
- `../01-transformer-foundations/xml-vs-json-vs-yaml.md` — format-choice debate (empirical + formal layers, OpenAI + Google guidance).
- `../01-transformer-foundations/sparse-attention-and-kv.md` — why markup anchors matter past 4k tokens.
- `../01-transformer-foundations/llm-training-pipeline.md` — why the training distribution rewards in-source markup, not external docs.
- `./start-end-tags.md` — paired start / end tag mechanism; AST-free parsing; Anton's contract-only MCP pattern.
- `./in-source-documentation.md` — Enterprise-level lineage of in-source documentation; GRACE as a specialization.
- `./markup-compression.md` — empirical compression bounds; PURPOSE critical; English > other; adjacency matters.
- `./markup-validators.md` — validating GRACE markup algorithmically (1650-line Python validator built with GRACE).
- `../03-contracts/contract-fields.md` — full spec for PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS / RATIONALE.
- `../03-contracts/wenyan-prompting.md` — compression technique behind KEYWORDS and LINKS.
- `../03-contracts/language-specific/python.md` — Python-specific placement rules.
- `../03-contracts/language-specific/csharp.md` — C# XML Documentation mapping.
- `../09-tooling/mcp-scepticism.md` — when custom MCPs and doc-MCPs work and when they don't.
- `../13-antipatterns/all-antipatterns.md` — YAML configuration, JSON in long context, "sovereign AI" non-English comments, long summary-style descriptions.
