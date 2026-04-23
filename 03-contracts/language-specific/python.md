# Python Contract Placement

## What

GRACE's Python convention places the contract fields (PURPOSE, INPUTS, OUTPUTS, KEYWORDS) as **line comments immediately BEFORE the `def` statement**, while the Python docstring remains **below** the `def` as an optional SFT-hint complement [author-claim] (post 3878).

This deviates from the default Python convention where the docstring alone (below `def`) carries function documentation. [author-claim] (post 3878).

In addition, GRACE recommends wrapping Python code blocks with **paired START-END tags inside comments** to make Python code Dyck-Language-compatible in a single parse pass. [author-claim] (post 3948).

## Why

### Python comments are not Dyck-friendly on their own

Python uses `#` for single-line comments with no native paired-close construct. [author-claim] (post 3948).

Discrete logic accessible to a transformer without chain-of-thought is limited to TC⁰, which natively parses **Dyck Language** (paired-bracket languages). [research-backed] (post 3948, citing arxiv 2105.11115 — Shunyu Yao et al.).

Dyck-compatible formats include **XML, HTML, JSON, and programming languages with paired brackets**; YAML is NOT Dyck-compatible. [research-backed] (post 3948).

Because bare Python comments lack a paired close, the author states: *"if you do not add paired tags in comments in Python, as I do in GRACE, it is much harder for GPT to parse it."* [author-claim] (post 3948).

Adding paired **START-END** tags in Python comments — the GRACE pattern — makes the file Dyck-compatible so GPT can *"parse the semantics in one pass without degradation."* [author-claim] (post 3948).

### Causal reading and the anchor vector around the function-name token

The author's justification for placing PURPOSE/INPUTS/OUTPUTS/KEYWORDS **before** `def` (not inside the docstring below) is that cross-language in-source-documentation conventions standardly place the description **before** the declaration, and this position is *"of critical significance for the formation of the anchor vector behind the tokens of the function/method name"* under GPT's causal reading. [author-claim] (post 3878).

In plain terms: GPT reads left-to-right, so documentation that arrives *before* the function-name token contributes to the vector attached to that name token; documentation that arrives *after* cannot.

### The docstring still has value below `def`

Sparse attention typically covers roughly the last ~1000 recent tokens, so a docstring sitting immediately below `def` still falls inside the window during generation and can act as a **SFT-hint** — OpenAI has publicly stated that it trains ChatGPT on Python docstrings, so the format is deeply ingrained in the model. [research-backed] (post 3892).

Therefore GRACE keeps **both layers**:
- Line-comment contract fields **before** `def` — form the anchor vector at the name token (load-bearing). [author-claim] (post 3878).
- Docstring **below** `def` — contributes as a recent-token SFT-hint; optional but not harmful if concise. [author-claim] (posts 3878, 3892).

### Why the PURPOSE field carries the most weight

Research on in-source documentation benchmarks (arxiv 2601.16661v1) shows that the agent can skip the DESCRIPTION entirely but **cannot skip PURPOSE** — the latter is critical for alignment, while long descriptions are at best neutral and often degrade the agent. [research-backed] (post 3889).

This finding matches Anthropic's observation of **Purpose Breakdown Structure**: GPT always builds a hierarchical goal decomposition from global to local, and verbalizing that purpose explicitly is better than leaving it implicit. [research-backed, via author-claim] (post 3878).

### Why KEYWORDS instead of long prose

KEYWORDS (and LINKS) sections are GRACE's instance of **wenyan prompting** (literary-Chinese-style compression): roughly 1:15 token compression vs full prose. [author-claim] (post 3819).

The ShortenDoc paper (dl.acm 10.1145/3735636) independently proved that aggressive compression of Python docstrings — keeping only key concepts — yields 40% compression with no quality loss and ~10% HumanEval improvement, validated via perplexity. [research-backed] (post 3892).

Full prepositional phrases degrade GPT; compressed concept lists do not. [research-backed] (post 3892).

## How

### Canonical example (verbatim from post 3878)

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

Source: post 3878 (reproduced exactly; the source post uses Russian glosses — the author notes elsewhere that English is the best language for code comments and that "sovereign AI" non-English comments cause degradation [research-backed] (post 3889), so teams building for production should translate descriptions to English while preserving the placement pattern).

### Placement rules

| Rule | Location | Rationale |
|---|---|---|
| PURPOSE field | Line comment **before** `def` | Anchor vector at name token (post 3878) |
| INPUTS / OUTPUTS | Line comments **before** `def` | Same; plus standard practice for in-source docs (post 3878) |
| KEYWORDS | Line comment **before** `def` | wenyan-prompting hook for RAG/search agents (posts 3819, 3892) |
| Docstring | **Below** `def` (triple-quoted) | SFT-hint; covered by ~1000-token sparse-attention window (post 3892) |
| Module-level and block-level START-END tags | Paired `#`-comments wrapping logical blocks | Dyck compatibility in one parse pass (post 3948) |

### Field-format conventions

The canonical example shows the field grammar:

- `PURPOSE:` — one concise sentence describing the intent. Critical per benchmarks (post 3889). Keep short (post 3889, 3892).
- `INPUTS:` — one bullet per argument of shape `- <type> - <name>: <brief description>` (post 3878).
- `OUTPUTS:` — one bullet of shape `- <type> - <description>` (post 3878).
- `KEYWORDS:` — `[Keyword1, Keyword2]` list of architectural patterns and task classifications (post 3878). These are the wenyan/enrichment hooks used by vector search and Cursor's semantic search; Cursor-based empirical observation is that ~90% of semantic-search hits land on KEYWORDS/LINKS enrichment points. [empirical] (post 3819).

LINKS (cross-module references) and RATIONALE/AAG (architectural intent) belong primarily in **module contracts** rather than on every function, but they follow the same line-comment-above-declaration rule when they do appear at function level. [author-claim] (posts 3878, 3890). See `../contract-fields.md` and `../rationale-and-aag.md`.

### START-END tags

Add paired `# START ...` / `# END ...` comments around logical blocks (modules, classes, regions) so the file is Dyck-parsable in one pass. [author-claim] (post 3948).

The sources do not specify the exact literal spelling of the START/END tokens in Python; the post describes the *pattern* ("paired tags in comments") rather than prescribing an exact lexeme.

This same pattern also enables AST-free tooling — e.g., Anton's contract-only MCP walks paired tags to return only contracts plus function declarations without invoking a full Python parser. [author-claim] (post 3622). See `../../04-markup-system/start-end-tags.md`.

### What to avoid

- **Docstring-only documentation (no line comments above `def`)** — loses the anchor-vector effect at the function-name token. [author-claim] (post 3878).
- **Long prose descriptions** — neutral at best, degradation-causing at worst. [research-backed] (posts 3889, 3892).
- **"Captain Obvious" comments** restating what the code already says — useless and potentially harmful. [author-claim] (post 3890).
- **Non-English comments** ("sovereign AI") — benchmarks show degradation. [research-backed] (post 3889).
- **Plain Python files with no START-END tags relied on for large contexts** — Python's `#` alone is not Dyck-friendly and GPT struggles to parse large files semantically in one pass. [author-claim] (post 3948).

### Mental model

For each Python function, GRACE attaches **two complementary layers of documentation**:

1. **Above `def`** (line comments, before declaration): contract fields — load-bearing, forms the anchor vector, cannot be skipped semantically by GPT during causal read.
2. **Below `def`** (docstring): SFT-hint — the format GPT was trained on, falls inside the ~1000-token sparse-attention window during generation.

Both layers are cheap; the upper layer is load-bearing and the lower layer is a training-distribution concession.

## Evidence

### Sources

- **Post 3878** — introduces the GRACE Python convention. Contains the canonical `save_config` example verbatim and explicitly states *"For Python I may deviate somewhat from the usual description in docstrings, but this is done precisely because there are larger cross-language conventions about how to make in-source documentation, and there the description goes before the function declaration, which for the AI's causal reading is of critical significance for forming the anchor vector behind the tokens of the function/method name."* [author-claim translated from Russian].
- **Post 3892** — ShortenDoc paper (dl.acm 10.1145/3735636): 40% compression of docstrings, ~10% HumanEval gain; also notes docstring can act as SFT-hint and that ~1000 recent tokens fall in sparse-attention coverage. [research-backed].
- **Post 3948** — TC⁰ and Dyck Language argument (arxiv 2105.11115, Shunyu Yao et al.): Dyck-compatible formats (XML/HTML/JSON/bracketed languages) are native to transformers; Python's `#` comments are not Dyck-friendly alone; paired START-END tags in comments solve this in one pass. [research-backed].
- **Post 3889** — In-source documentation benchmarks (arxiv 2601.16661v1): PURPOSE critical, shorter is better, English > other languages, adjacency matters; swings from -95% to +435%. [research-backed].
- **Post 3890** — Architectural-intent comments can triple bug-fix accuracy (researchgate.net/publication/400340807). [research-backed].
- **Post 3819** — wenyan prompting; Grok reconstructs meaning from KEYWORDS/LINKS alone; Cursor empirical ~90% semantic-search hits land on KEYWORDS/LINKS. [author-claim + empirical].
- **Post 3622** — contract-only MCP (AST-free) depends on START-END tags. [author-claim].

### Research references

- Shunyu Yao et al., *Dyck Language in Transformers* — arxiv.org/abs/2105.11115 (post 3948).
- *ShortenDoc* — dl.acm.org/doi/10.1145/3735636 (post 3892).
- In-source documentation benchmarks — arxiv.org/abs/2601.16661v1 (post 3889).
- *On the Impact of Code Comments for Automated Bug-Fixing* — researchgate.net/publication/400340807 (post 3890).

See `../../14-research-references/annotated-bibliography.md` for full annotations.

## See also

- `../contract-fields.md` — full field specification (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS, RATIONALE/AAG) shared across languages.
- `../contracts-overview.md` — why contracts are central to GRACE.
- `../ai-contracts-vs-dbc.md` — distinction from classical Design by Contract.
- `../wenyan-prompting.md` — compression rationale for KEYWORDS.
- `../rationale-and-aag.md` — architectural-intent sections (module-level).
- `./csharp.md` — C# XML-Doc mapping of the same pattern.
- `../../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis for paired-tag requirement.
- `../../01-transformer-foundations/sparse-attention-and-kv.md` — why the ~1000-token window argument for docstrings works.
- `../../04-markup-system/start-end-tags.md` — paired-tag markup in detail.
- `../../04-markup-system/xml-like-markup.md` — broader markup rules.
- `../../13-antipatterns/all-antipatterns.md` — long summary-style descriptions, non-English comments, Captain-Obvious comments.
