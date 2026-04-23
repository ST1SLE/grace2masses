# START-END Tags

## What

**START-END tags** are paired markup tags placed around logical blocks of code or documentation. In GRACE, they are inserted via the host language's comment syntax so they are invisible to the compiler but visible to the LLM and to simple text-processing tools [author-claim] (post 3948). They behave like opening/closing brackets: every `START` has a matching `END`. This makes them act as a **paired-bracket structure** over otherwise free-form text.

In practice the tags name a semantic role (e.g., a contract section, a module boundary, a logical block) and are wrapped in whatever the host language accepts as a comment — line comments in Python, `region` directives in C#, and so on (post 3948; post 3960).

## Why

GRACE introduces START-END tags for **two distinct reasons** (post 3948; post 3622). Both reasons are load-bearing — neither alone would justify the discipline.

### Reason 1 — Dyck-Language compatibility for one-pass parsing

Without chain-of-thought, a transformer is limited to TC⁰ discrete logic [research-backed] (post 3948). TC⁰ natively supports **Dyck Language** — the class of paired-bracket languages formulated by Walter von Dyck; every opening bracket must have a matching closing bracket, otherwise the structure is no longer parseable by loop-free TC⁰ circuits (post 3948). Shunyu Yao and co-authors proved native Dyck-Language support specifically for transformers (arxiv 2105.11115) (post 3948).

XML, HTML, JSON, and bracket-based programming languages are Dyck-compatible and therefore parseable by GPT in a single pass [research-backed] (post 3948). YAML is **not** Dyck-compatible [research-backed] (post 3948). Python's line comments, on their own, are not Dyck-friendly either — a plain `# ...` has no closing partner.

The author's statement (post 3948):

> "In the case of Python, one can build a Dyck-Language-compatible variant by adding paired START-END tags in comments, as I do in GRACE. Then GPT can parse the semantics in one pass without degradation."

So the first function of START-END tags is to **lift a non-Dyck host language into a Dyck-Language-compatible form** for the LLM. Once they are present, the LLM can parse the semantic structure without degradation, and without needing chain-of-thought to compensate for an unbalanced grammar [author-claim] (post 3948).

### Reason 2 — AST-free parsing by simple algorithms

The second reason appears in post 3622, describing Anton's MCP experiment on GRACE-marked code. In that post the author writes explicitly that START-END tags are inserted "not only for sparse attention, but also so that code can be read and modified by simple algorithms, without AST parsing" [author-claim] (post 3622).

This matters because:

- Conventional tooling on code requires a language-specific parser (AST) — one per language, often version-dependent.
- A paired-bracket markup overlay (START/END) can be walked with a trivial bracket-matcher, written in any language, and applied uniformly across host languages [author-claim] (post 3622).
- Tools built on this overlay — readers, editors, contract-extractors, validators — avoid the cost and fragility of full compilation pipelines [author-claim] (post 3622; post 3904 for the validator case).

Put together: Reason 1 makes the **LLM** see the structure cleanly; Reason 2 makes **tooling** see the same structure cleanly, without pulling in a compiler.

## How

### Placement pattern

START-END tags are placed inside the host language's comment syntax, wrapping a logical block — most often a contract section, a module boundary, or a block that tooling needs to locate without AST walking (post 3622; post 3948).

In Python specifically, the author notes that adding paired START-END tags in comments is what makes a Python file Dyck-Language-compatible (post 3948):

> "In Python, one can build a Dyck-Language-compatible variant by adding paired START-END tags in comments, as I do in GRACE."

In C#, the built-in `region` directive already behaves as a paired START/END delimiter and is **directly usable for sparse-attention paired tagging** [author-claim] (post 3960).

The sources do not specify an exhaustive tag vocabulary for every language — only the pattern (paired, in-comment, wrapping semantic blocks).

### Anton's contract-only MCP (post 3622)

The canonical tooling example of AST-free parsing is Anton's MCP server, reported in the comments under post 3620 and summarized in post 3622. Anton's own description (post 3622, quoted from the comment):

> "To access contracts in a file, instead of the native `read_file` tool I use a custom `read_contracts`, which returns the identifiers of contracts in the file, their text, and the implementation line ranges for the functions of those contracts. This saves a lot of tokens, extracts something like an outline of the file, and simplifies access to the needed parts of the file, because the LLM no longer has to guess where the implementations of the relevant code lie. And with tools like `get_contracts_by_contract_path`, one can already return a graph of related contracts that are read from the PATH header of the contract, if one exists. This opens up semantic contract search and isolates the context to only the necessary related code, which helps a lot."

Breakdown of what the two tools do, paraphrasing the quoted comment (post 3622):

- **`read_contracts`** — instead of the default `read_file`, returns a structured record: contract IDs present in the file, the contract text, and the line ranges of the function implementations bound to each contract [empirical] (post 3622).
- **`get_contracts_by_contract_path`** — walks a graph of related contracts via the `PATH` header present in a contract (when the header exists) [empirical] (post 3622).

These tools exist because START-END tags let a simple parser locate contract blocks, extract their text, and compute implementation line ranges without running a real Python/C#/etc. AST [author-claim] (post 3622).

The author's editorial note on the same pattern (post 3622):

- The approach is "particularly effective for legacy code", because human-written legacy is often "hand-to-face" — exposing it to the LLM raw seeds bad patterns and rudiments [author-claim] (post 3622).
- For **AI-native** code, the author prefers **greedy reading** (read the whole module) instead of contract-only extraction, because the existing AI-written code acts as few-shot anchoring for the LLM, tuning it onto current framework versions [author-claim] (post 3622).

So the tools are first-class for legacy contract injection; for AI-native code, the same START-END tags exist for Dyck compatibility and are used without the contract-only MCP.

### Language-agnosticism

Because START-END tags live inside comments, they **work in any language that has a comment syntax** [author-claim] (post 3948; post 3960). The specific syntactic carrier varies — `#` in Python, `//` or XML-doc / `region` in C# (post 3960) — but the overlay structure is identical.

This is important operationally: the same contract-processing tools, validators, and MCPs can apply across languages without rewriting per-language AST walkers (post 3622 for the MCP case; post 3904 for the validator case — a 1650-line Python validator over GRACE markup, built by a colleague using GRACE itself).

### Interaction with sparse attention

Paired tags are not *only* a Dyck-parsing trick — they also serve as **sparse-attention anchors** in long contexts (post 3622; post 3960). The same `region`-style paired delimiters in C# that give Dyck compatibility also give the attention heads stable bracket boundaries to anchor on [author-claim] (post 3960). The author explicitly lists both purposes side by side:

> "For paired tags for sparse attention there is `region`." — post 3960 on C#.
> "START-END markup tags are not only for sparse attention, but also so the code can be read and modified by simple algorithms without AST parsing." — post 3622 on the MCP.

## Evidence

- **Post 3622** — Anton's contract-only MCP. Primary source for the AST-free parsing purpose and for the `read_contracts` / `get_contracts_by_contract_path` tools. Contains the quoted author description of why START-END exists beyond sparse attention. Also contains the legacy-vs-AI-native distinction (greedy reading for AI-native code).
- **Post 3948** — TC⁰ and Dyck Language. Primary source for the Dyck-compatibility purpose. Cites Shunyu Yao et al. (arxiv 2105.11115) as formal proof that transformers natively support Dyck-Language parsing. States explicitly that adding paired START-END tags in Python comments makes Python one-pass parseable by GPT without degradation. Also records Google's YAML ban in Gemini's Code Execution sandbox as a parallel data point on non-Dyck format degradation.
- **Post 3960** — C# XML Documentation. Confirms that `region` directives function as paired-tag anchors for sparse attention, and notes high generation stability because the pattern is in the LLM's training distribution.
- **Post 3904** — GRACE-markup validator. A colleague built a 1650-line Python validator for GRACE markup, using GRACE itself to manage the agents that wrote it; the validator is included in autotests. This is an empirical witness that simple algorithmic parsing over START-END markup is tractable in practice [empirical] (post 3904).
- **Shunyu Yao et al., arxiv 2105.11115** — cited by post 3948 as the formal proof of native Dyck-Language support in transformers. Full bibliographic details beyond the arxiv ID are not specified in the sources.
- **Google's YAML ban in Gemini Code Execution sandbox** — cited by post 3948 as a vendor-level confirmation that non-Dyck formats degrade LLM output; the sandbox disables YAML libraries from Python to prevent the model from degrading while attempting to use a non-Dyck format [vendor-doc] (post 3948).

## See also

- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis (TC⁰, Walter von Dyck, Shunyu Yao proof) for why paired brackets matter.
- `../01-transformer-foundations/xml-vs-json-vs-yaml.md` — format comparison grounded in the same Dyck-compatibility argument.
- `../04-markup-system/xml-like-markup.md` — the broader markup family of which START-END tags are one component.
- `../04-markup-system/in-source-documentation.md` — enterprise lineage (In-source documentation / Documentation as Code) that START-END tags sit within.
- `../04-markup-system/markup-validators.md` — the algorithmic validator built on top of START-END structure (post 3904).
- `../03-contracts/language-specific/python.md` — concrete Python usage: START-END tags in `#` comments to make Python files Dyck-compatible.
- `../03-contracts/language-specific/csharp.md` — C# `region` directives as the native paired-tag carrier.
- `../03-contracts/legacy-contract-recovery.md` — the legacy-code context where Anton's contract-only MCP (post 3622) is the preferred reading strategy.
- `../09-tooling/mcp-scepticism.md` — how Anton's contract-only MCP is one of the few MCP exceptions that works in the LLM's training distribution.
