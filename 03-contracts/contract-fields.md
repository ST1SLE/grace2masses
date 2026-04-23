# Contract Fields — Full Specification

## What

A GRACE contract is a structured block of in-source documentation placed **before** a function, method, or module declaration. It is written in English, uses lightweight markup (line comments plus short labelled sections), and expresses the AI-author's intent in a form the LLM can load into its attention window alongside the code itself. [author-claim] (post 3878)

The canonical field set used for a function-level contract comprises:

- **PURPOSE** — goal/intent of the function (load-bearing).
- **INPUTS** — per-parameter triples of `type - name: description`.
- **OUTPUTS** — per-return-value pairs of `type - description`.
- **KEYWORDS** — a compact list of architectural patterns or task tags.
- **LINKS** — cross-module references (Call Graph proxy).
- **RATIONALE / AAG** — architectural intent (used principally in *module* contracts; cross-link `rationale-and-aag.md`).

The Python docstring, if present, stays **below** the declaration as an SFT-style hint. [author-claim] (post 3878; post 3892)

---

## Why

### PURPOSE is load-bearing

An agent can skip a long DESCRIPTION without measurable damage; it cannot skip PURPOSE without degrading. [research-backed] (post 3889, citing arxiv.org/abs/2601.16661v1)

Reinforcement-Learning training rewards the model for **"goal achievement"**. PURPOSE is therefore not exposition; it is an alignment anchor that maps the function onto the same optimisation axis the weights were tuned against. [author-claim] (post 3878)

Anthropic has shown that GPT models always construct a **Purpose Breakdown Structure (PBS)**: a hierarchical decomposition of goals from global intent down to the current local step. Verbalising that decomposition in the contract is better than leaving it implicit. [author-claim, referencing Anthropic] (post 3878)

### INPUTS / OUTPUTS formalise the micro-spec

Classic in-source documentation already describes parameters and return values; this part of GRACE is deliberately conventional. The `- type - name: description` form is compact, aligns with Dyck-Language-friendly bracketed/paired reading (cross-link `../01-transformer-foundations/tc0-and-dyck-language.md`), and matches how LLMs saw task→code pairs during SFT. [author-claim] (post 3878)

### KEYWORDS are "wenyan" anchors

KEYWORDS (and LINKS) are **wenyan prompting** — extreme literary-Chinese-style compression. A contract in this form compresses roughly **1:15** versus full prose while remaining fully decodable by a modern LLM; Grok reconstructed the hidden meaning of a GRACE contract with no code attached, matching the extraction behaviour seen for Confucian texts. [author-claim, with reproducible demo] (post 3819)

Keywords double as RAG hooks: in Cursor's semantic search, about **90%** of hits land on exactly these enrichment points, per the author's trainees. [empirical] (post 3819)

SFT datasets routinely included task-classification tags, so KEYWORDS ride a distribution the model already knows. [author-claim] (post 3878)

Cross-link to the compression theory and ShortenDoc evidence: `wenyan-prompting.md`.

### LINKS act as a Call Graph proxy

Cross-module references inside the contract let the agent discover related modules **without an actual graph DB and without the calls appearing in code**. Vector retrieval over the chunk plus LINKS is usually enough to replace an explicit Call Graph for code-scale needs. [author-claim] (post 3732)

### RATIONALE / AAG expose architectural intent

Architectural-intent comments raise automated bug-fix accuracy **up to 3×**; attention analysis shows the AI literally attends to intent tokens when reconstructing the broken logic. [research-backed] (post 3890, citing researchgate.net/publication/400340807)

Without RATIONALE / AAG, a model patching broken code has no token-level handle on *why* the method exists, only on *what* it does — and "what" is already visible in code. "Captain Obvious" comments (restating code) add no value and can be harmful. [author-claim + research-backed] (post 3890)

Full treatment in `rationale-and-aag.md`.

### Placement matters — before the declaration

Cross-language in-source documentation standards place the description **before** the declaration. This is not a style preference: in a causal (left-to-right) read, the tokens of PURPOSE/INPUTS/OUTPUTS/KEYWORDS feed the anchor-vector that the model assembles around the function-name token at the moment of the `def` / signature. If the description comes after, that anchor is already frozen. [author-claim] (post 3878)

Short descriptions, **tightly adjacent** to the code, were the best-performing configuration in the in-source-docs benchmark; separating description from code (via file-hop or MCP-hop) caused measurable degradation. [research-backed] (post 3889)

Effect sizes in the same benchmark: **-95% degradation** when structure is broken, **up to +435% improvement** when structure is correct. [research-backed] (post 3889)

### Why a Python docstring may still sit below the declaration

Sparse attention's sliding window typically covers the **last ~1000 tokens**, so a docstring *below* the function signature can still be read during generation of the body. It operates as an SFT-style hint (docstrings are explicitly used in ChatGPT training). [research-backed] (post 3892, citing dl.acm.org/doi/10.1145/3735636)

Both layers contribute — the contract above anchors long-range navigation; the docstring below nudges local generation. [author-claim] (post 3892)

Language-specific placement details: `language-specific/python.md`, `language-specific/csharp.md`.

---

## How

### Field-by-field reference

#### PURPOSE

- **Content**: one sentence naming the goal. Not a description of mechanism. Ask "*what is this function for?*", not "*what does it do?*".
- **Position**: first field in the contract.
- **Compression rule**: aggressive. Long summary-style text is at best neutral, at worst degrading. [research-backed] (post 3889)
- **Language**: English. Non-English ("sovereign AI") comments cause degradation in published benchmarks. [research-backed] (post 3889)
- **Omission policy**: omitting DESCRIPTION is acceptable; omitting PURPOSE is not. [research-backed] (post 3889)

#### INPUTS

- **Content**: one line per parameter, in the form `- type - name: description`.
- **Position**: immediately below PURPOSE.
- **Description constraint**: short. Full prepositional phrases degrade the LLM; "mutilated" text keeping only key concepts performs better on HumanEval. [research-backed] (post 3892)

#### OUTPUTS

- **Content**: one line per return value, in the form `- type - description`.
- **Position**: after INPUTS.
- **Note**: the name slot is omitted versus INPUTS because many languages have unnamed returns; keep the two forms consistent. (post 3878)

#### KEYWORDS

- **Content**: `KEYWORDS:[Keyword1, Keyword2, …]` — short tags naming architectural patterns and/or task classifications.
- **Purpose**: (1) give the model correlation hooks to similar functions/modules it saw during training; (2) act as RAG-anchor tokens that semantic search reliably lands on. [author-claim + empirical] (posts 3878, 3819)
- **Validation rule**: the author accepts a compression only if (a) logit prediction still recovers the expected tokens, and (b) a blind LLM gloss reproduces the intended meaning without seeing the code. Both checks must pass. [author-claim] (post 3892)
- **"GPT already invented a secret transformer language; you just don't speak it yet."** — (post 3892)

#### LINKS

- **Content**: a compact list of cross-module / cross-contract references.
- **Purpose**: serves as a Call Graph proxy. Lets the agent reach related modules through vector search even when no direct call exists in the code. [author-claim] (post 3732)
- **Infrastructure note**: a graph DB is not strictly required if chunks are pre-enriched with LINKS. GRACE's contracts plus LINKS plus a vector store (e.g., sqlite-vec, github.com/asg017/sqlite-vec) handle most code-scale retrieval. [author-claim] (post 3732)

Cross-link: `../02-semantic-graph/graph-rag-vs-vector-rag.md`.

#### RATIONALE / AAG

- **Content**: the architectural reasoning behind the module — why this component exists, what pattern it instantiates, what trade-offs the author accepted.
- **Position**: in the **module** contract, not normally at function level.
- **Evidence**: architectural-intent comments deliver up to **3× bug-fix accuracy**; attention studies confirm the model actively reads those tokens during repair. [research-backed] (post 3890)
- **Anti-pattern**: "Captain Obvious" comments that merely restate code. No benefit; sometimes harmful. [research-backed] (post 3890)

Full treatment: `rationale-and-aag.md`.

### Placement rule

Contract block sits **immediately before** the declaration. No blank line in between. No docstring above the contract. In Python this means line comments starting with `#` sit directly atop `def`; in C# the `///` / `region` block sits atop the method. (post 3878; post 3960)

Language-specific mechanics (paired START-END tags in comments for Dyck compatibility; `region` directives in C#): `language-specific/python.md`, `language-specific/csharp.md`, and `../04-markup-system/start-end-tags.md`.

### Canonical Python example

Reproduced verbatim from post 3878:

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

[author-claim, illustrative demo — the Russian text above is from the source post and shows the field layout; in production, fields must be in English per the benchmark finding in post 3889] (posts 3878, 3889)

Notes on the example:

- **Position** — PURPOSE/INPUTS/OUTPUTS/KEYWORDS precede `def`. The docstring inside the function is the SFT-hint layer (post 3892).
- **KEYWORDS** — `[Saver, FileWrite]` tags both the role ("Saver") and the concrete operation class ("FileWrite"). Two tokens, rich correlation surface.
- **No LINKS in this micro-example** — LINKS are more commonly populated on functions that coordinate across modules and on module contracts. (post 3878)
- **No RATIONALE** — this is a function-level contract; RATIONALE normally lives on the module contract above. (post 3890)

### Minimal compliance checklist

1. PURPOSE present, one line, English, goal-oriented. (post 3889)
2. INPUTS with `- type - name: description` per parameter. (post 3878)
3. OUTPUTS with `- type - description`. (post 3878)
4. KEYWORDS as `[Keyword1, …]`, compressed, validated via logit + blind-gloss. (posts 3878, 3892)
5. LINKS populated if the function or module reaches across modules. (post 3732)
6. RATIONALE / AAG populated on module contracts. (post 3890)
7. Entire block tightly adjacent to the declaration — no interleaving file reads, no MCP hops. (post 3889)
8. Placement: before declaration (causal-read anchor). (post 3878)
9. Language: English throughout. (post 3889)
10. No "Captain Obvious" restatement of code. (post 3890)

### Validation

Two approaches are used in practice:

- **LLM review** — the author's default, run concurrently with code review. Catches semantic errors; costs tokens. [author-claim] (post 3904)
- **Algorithmic validator** — a colleague built a GRACE-markup validator (~1650 Python lines) using GRACE itself, wired into autotests. Instant, deterministic, token-free; cannot catch semantic errors. [empirical] (post 3904)

Both approaches have their place. Cross-link: `../04-markup-system/markup-validators.md`.

### Infrastructure interactions

- **Anton's contract-only MCP** (post 3622) reads contracts and declarations via the paired START-END tags, using the exact field layout above. It returns contract IDs, text, and implementation line ranges, and can walk a PATH-header graph via a `get_contracts_by_contract_path` tool. Useful specifically for *legacy* code; for AI-native code, prefer greedy reading. [empirical] (post 3622)
- **Cursor semantic search** — in the author's observations, roughly 90% of semantic-search hits land on KEYWORDS / LINKS tokens, confirming that this is where retrieval actually "grabs". [empirical] (post 3819)

---

## Evidence

### Primary posts

- **Post 3878** (2026-04-12) — "In-source documentation / Documentation as Code" framing; canonical Python example; PURPOSE as RL-alignment anchor; Purpose Breakdown Structure (Anthropic); placement-before-declaration rationale; KEYWORDS as task-classification tags from SFT.
- **Post 3889** (2026-04-12) — Benchmarks of in-source documentation (arxiv.org/abs/2601.16661v1). Proves: PURPOSE critical, agent can skip DESCRIPTION; shorter is better; English outperforms other languages; adjacency matters; -95% to +435% metric swings.
- **Post 3890** (2026-04-12) — Architectural-intent comments and bug-fixing (researchgate.net/publication/400340807). Up to 3× bug-fix accuracy; attention analysis; "Captain Obvious" warning; RATIONALE/AAG justification.
- **Post 3892** (2026-04-13) — ShortenDoc (dl.acm.org/doi/10.1145/3735636): 40% docstring compression without quality loss; full prepositional phrases degrade; "mutilated" text raises HumanEval ~10%. Perplexity / logit validation protocol. Docstring below declaration works as SFT hint because sparse attention covers ~1000 recent tokens. "Secret transformer language" phrase.

### Supporting posts

- **Post 3819** — wenyan-prompting; ~1:15 compression; 90% of Cursor semantic-search hits land on KEYWORDS/LINKS; Grok reconstructed contract meaning from compressed form with no code.
- **Post 3732** — Hybrid / Graph RAG; LINKS as Call Graph substitute; sqlite-vec reference (github.com/asg017/sqlite-vec).
- **Post 3622** — Anton's contract-only MCP, operating on contract structure alone.
- **Post 3904** — Algorithmic GRACE-markup validator (~1650 LOC Python) built *using* GRACE.
- **Post 3948** — TC⁰ / Dyck Language / Shunyu Yao (arxiv.org/abs/2105.11115): XML and paired-tag formats are natively parsable by transformers; relevant to the paired-tag infrastructure the contract rides on.
- **Post 3960** — C# XML Documentation maps cleanly to this field set via `region` directives; summary tags are sub-optimal, KEYWORDS/LINKS are richer anchors.
- **Most-important file**, §"Top-down логика и переход к контрактам" — contracts are the output of the top-down workflow: graph → module contract → function contract → code; contracts replace classical documentation because the architecture and business model live inside the code. (`most_important.md`)

### Research papers cited in this file

- **In-source documentation benchmark** — arxiv.org/abs/2601.16661v1 (post 3889).
- **Code-comment architectural-intent study** — researchgate.net/publication/400340807 (post 3890).
- **ShortenDoc** — dl.acm.org/doi/10.1145/3735636 (post 3892).
- **Shunyu Yao et al. on Dyck Language in transformers** — arxiv.org/abs/2105.11115 (post 3948).

Full annotated list: `../14-research-references/annotated-bibliography.md`.

---

## See also

- `contracts-overview.md` — why contracts are the semantic shield for AI modification.
- `ai-contracts-vs-dbc.md` — how AI-contracts differ from Hoare-logic DbC (Eiffel / JML).
- `wenyan-prompting.md` — compression theory behind KEYWORDS / LINKS.
- `rationale-and-aag.md` — module-contract architectural-intent section.
- `legacy-contract-recovery.md` — adding contracts to pre-existing code.
- `language-specific/python.md` — Python-specific placement (comments-before-def + docstring).
- `language-specific/csharp.md` — C# XML Documentation mapping.
- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis for paired-tag / XML-like structure.
- `../02-semantic-graph/graph-rag-vs-vector-rag.md` — LINKS as Call Graph substitute.
- `../04-markup-system/start-end-tags.md` — the paired-tag infrastructure this block sits inside.
- `../04-markup-system/in-source-documentation.md` — lineage and positioning versus classical Documentation-as-Code.
- `../04-markup-system/markup-validators.md` — validating the field set algorithmically or via LLM review.
- `../04-markup-system/markup-compression.md` — empirical compression bounds.
- `../14-research-references/annotated-bibliography.md` — full paper list.
