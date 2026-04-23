# Wenyan Prompting — Literary-Chinese Compression for Contracts

## What

**Wenyan** (文言) is the term for classical / literary Chinese, a register of the language famed for its extreme brevity — a few characters carry what would take a paragraph of modern prose. The author adopts it as the label for a specific compression discipline inside GRACE: **reducing in-source documentation to the minimum set of key concepts a modern LLM can still fully decode**, and nothing more. [author-claim] (post 3819)

In a GRACE contract, the **KEYWORDS** and **LINKS** sections are the prototypical wenyan units. They are short lists of architectural patterns, task classes, or cross-module references — not sentences, not explanations. [author-claim] (post 3819; post 3878)

Typical compression vs. full natural-language prose: **~1:15** in tokens. [author-claim, reproducible demo] (post 3819)

Wenyan prompting is the compression technique that makes this ratio safe — the decoding side has been validated on real LLMs (Grok, Cursor's search backend, the author's Qwen/Gemini experiments) and by an independent research paper (ShortenDoc). [author-claim + research-backed] (posts 3819, 3892)

---

## Why

### Token budget is not infinite, and neither is attention

Injecting documentation into every module (GRACE's core move; see `../04-markup-system/in-source-documentation.md`) costs tokens at every read. Without compression, an agent working across many modules exhausts its long-context budget long before it has seen the code. The author's trainees who ignored compression "over-invested tokens in documentation injection and broke agent navigation as a side-effect". [author-claim] (post 3819)

### Compressed keywords become RAG hooks

In Cursor (chunk-based semantic search over 100–200 line windows; see `../09-tooling/cursor-limitations.md`), the author reports that roughly **90% of semantic-search hits** land on KEYWORDS / LINKS — the wenyan "enrichment points" of each chunk. [empirical] (post 3819)

The mechanism is straightforward: compressed, domain-salient tokens have a much higher vector-similarity signal per token than surrounding prose, so they dominate the retrieval score for any chunk they appear in. Related modules become discoverable **without a direct call appearing in code**, and without an external graph DB. [author-claim] (post 3819)

### Full natural-language prose degrades GPT

Wenyan compression is not only an economy measure; the opposite is actively harmful. Independent research (ShortenDoc, see below) shows that "full prepositional phrases with connectives" in docstrings cause measurable degradation, while "mutilating" the same text down to key concepts raises HumanEval by approximately 10 percentage points. [research-backed] (post 3892, citing dl.acm.org/doi/10.1145/3735636)

The in-source-documentation benchmark reinforces this: shorter comments are either neutral or positive, longer ones are either neutral or negative, never reliably better. [research-backed] (post 3889, citing arxiv.org/abs/2601.16661v1)

### Wenyan is decodable because the LLM already invented it

The author's recurring framing: **"GPT already invented a secret transformer language; you just don't speak it yet."** [author-claim] (post 3892)

Grounding: base-coder and SFT training rewarded the model for mapping short, tag-like inputs to rich program semantics. Task-classification tags in SFT datasets, Python docstring conventions, and the long tail of compressed architectural-pattern mentions in open-source code all trained the model to expand a compressed phrase back into its full meaning on its own. GRACE leans on that existing capability rather than re-training. [author-claim] (posts 3819, 3878, 3892)

Corollary: a compression that works on one modern LLM almost always works across vendors, because they share the training-data distribution. The author observes the same effect on Qwen and Gemini, not only OpenAI models. [author-claim] (post 3819; corroborated by post 3892's multi-vendor experiments)

---

## How

### The two sections that carry wenyan in a function contract

From the canonical Python example (post 3878; reproduced in `contract-fields.md`):

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

The `KEYWORDS:[Saver, FileWrite]` line is the wenyan unit. Two tokens name the role (a *Saver*) and the action class (a *FileWrite*). A LINKS section, where present, plays the same role at cross-module scope. [author-claim] (post 3878)

Note: in the example above the other fields (PURPOSE, INPUTS, OUTPUTS) are only lightly compressed — wenyan is applied in **degrees**, with KEYWORDS / LINKS the most aggressive and PURPOSE kept readable.

### Two reproductions to run before accepting a compression

The author's personal gate for adding a compressed form to GRACE uses two independent checks. [author-claim] (post 3892)

1. **Logit-prediction check.** Feed the model the compressed form as the preceding context and observe whether it predicts the downstream code tokens with correct logits — i.e. low perplexity on the implementation given the keywords alone.
2. **Blind-gloss test.** Without showing the model the code or any instructions, ask it to describe in plain words what a given compressed phrase ("a phrase in the secret transformer language", in the author's joking register) is likely to mean. If the paraphrase matches the intended meaning, the compression is safe.

Only phrases that pass **both** checks are accepted into the author's GRACE templates. [author-claim] (post 3892)

### Grok compressed-contract demo

The author ran a deliberate ablation: feed a weaker-tier Grok model **only the module contract**, with the code stripped away, and ask it to describe the hidden meaning. Grok reconstructed the intent successfully — a demonstration that wenyan-compressed contracts stay decodable by a separate agent at roughly 1:15 token compression vs. a full-prose alternative. [author-claim, reproducible demo] (post 3819)

The post draws an analogy to Confucian texts: dense literary Chinese is decodable by a reader trained on the tradition; wenyan-GRACE is decodable by an LLM trained on code plus natural language.

### Operational heuristics

- **Never expand keywords to sentences.** A `KEYWORDS:[Saver, FileWrite]` is strictly better than a sentence that names both concepts inline. [research-backed + author-claim] (posts 3889, 3892)
- **Prefer concept names the model has actually seen.** Architectural-pattern names (e.g. `Saver`, `Repository`, `Builder`), common task-class labels, and library nouns work because they appeared verbatim in SFT data. Invented vocabulary will not decode. [author-claim] (post 3878)
- **English only.** Benchmarks on in-source documentation confirm English outperforms other languages and "sovereign AI" non-English comments cause degradation. This applies to wenyan units as strongly as anywhere else. [research-backed] (post 3889)
- **Verify before shipping.** Run the logit + blind-gloss pair on any new compression vocabulary; don't guess. [author-claim] (post 3892)
- **Compress, then stop.** Wenyan is a floor, not a race; ShortenDoc shows 40% compression is the research-validated safe level for docstrings (see below). Over-compressing past the point of decodability is exactly the failure mode the two-test gate exists to catch. [research-backed] (post 3892)

### Why compressed contracts reach across modules without code calls

KEYWORDS and LINKS, because they are salient, short, and vector-retrievable, let an agent **navigate to a related module** even when the call graph contains no direct reference between the two modules. The relevant chunks surface together because they share wenyan tokens. [author-claim] (post 3819)

This is the mechanism that lets GRACE substitute vector retrieval over enriched chunks for a classical Call Graph or a graph DB — see `../02-semantic-graph/graph-rag-vs-vector-rag.md`. [author-claim] (post 3732)

---

## Evidence

### Primary posts

- **Post 3819** (2026-04-05) — Canonical post on wenyan prompting as a GRACE practice. Establishes KEYWORDS / LINKS as the wenyan sections; ~1:15 compression ratio vs. full prose; Grok successfully decoded a compressed contract with the code removed; Cursor trainees report ~90% of semantic-search hits land on wenyan enrichment points; without the technique agents over-spend tokens on documentation injection and lose navigation. [empirical + author-claim]
- **Post 3892** (2026-04-13) — **ShortenDoc** paper (dl.acm.org/doi/10.1145/3735636). Proves effectiveness of wenyan-analog for code generation: full prepositional phrases with connectives cause degradation; "mutilating" text down to key concepts yields ~+10% on HumanEval; docstrings can be compressed ~40% with no quality loss, sometimes with small gains. Authors validated via **perplexity** — the model must still correctly predict downstream tokens from the compressed form. The author reproduces with two checks: **logit prediction** and **blind-gloss paraphrase**. The post coins the "secret transformer language" phrasing. [research-backed + author-claim]

### Supporting posts

- **Post 3878** (2026-04-12) — Places the technique in the field specification: KEYWORDS appear on the canonical Python example; SFT datasets routinely contained task-classification tags, which is why such keywords decode reliably. [author-claim] (see `contract-fields.md`)
- **Post 3889** (2026-04-12) — In-source documentation benchmark (arxiv.org/abs/2601.16661v1). Empirical backing for the "shorter is better" side of wenyan: long descriptions are either neutral or harmful; English outperforms other languages; adjacency matters. Effect sizes: -95% to +435%. [research-backed]
- **Post 3890** (2026-04-12) — Architectural-intent comments (researchgate.net/publication/400340807) — covered fully in `rationale-and-aag.md`; relevant here because RATIONALE / AAG sections are written in the same compressed register.
- **Post 3732** (2026-03-26) — Hybrid / Graph RAG. LINKS-style enrichment acts as a Call Graph substitute without needing a graph DB; `sqlite-vec` for exact-fact retrieval (github.com/asg017/sqlite-vec). [author-claim]

### Research papers cited in this file

- **ShortenDoc** — dl.acm.org/doi/10.1145/3735636. Neural network for docstring compression. Proves 40% compression without quality loss, ~+10% HumanEval gain when reducing to key concepts, validated via perplexity. (post 3892)
- **In-source documentation benchmark** — arxiv.org/abs/2601.16661v1. Proves short > long, English > other languages, adjacency matters. (post 3889)
- **Impact of Code Comments on Automated Bug-Fixing** — researchgate.net/publication/400340807. Architectural-intent → 3× bug-fix accuracy; attention analysis. (post 3890)

Full annotated list: `../14-research-references/annotated-bibliography.md`.

### Reproducibility notes

- Grok decoding demo (post 3819): feed an LLM **only** a GRACE module contract, no code, and ask it to reconstruct intent. The author performed this on "not the smartest Grok" and it worked; the same test is suggested as a sanity check for any team adopting the technique.
- Author's two-part validator (post 3892): logit prediction on compressed form + blind-gloss paraphrase. Described as the personal gate for adding a new compression vocabulary to GRACE templates.

---

## See also

- `contract-fields.md` — the full field spec, where KEYWORDS and LINKS sit.
- `rationale-and-aag.md` — RATIONALE / AAG sections; same compressed register, at module scope.
- `contracts-overview.md` — why contracts exist and why their form is tuned to GPT.
- `ai-contracts-vs-dbc.md` — why wenyan contracts look nothing like Hoare logic.
- `language-specific/python.md` — Python-specific placement of the compressed fields.
- `language-specific/csharp.md` — C# XML Documentation + `region` carries the same wenyan units, with KEYWORDS/LINKS richer than the `summary` tag.
- `../02-semantic-graph/graph-rag-vs-vector-rag.md` — LINKS as Call Graph substitute; KEYWORDS-driven vector retrieval.
- `../04-markup-system/in-source-documentation.md` — lineage and positioning versus classical Documentation-as-Code.
- `../04-markup-system/markup-compression.md` — broader empirical compression bounds across the markup system.
- `../09-tooling/cursor-limitations.md` — why wenyan matters specifically for chunk-based Cursor search.
- `../14-research-references/annotated-bibliography.md` — ShortenDoc and in-source-docs benchmark entries.
