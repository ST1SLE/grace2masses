# Cursor Limitations

## What

Cursor is a RAG-agent-style AI IDE. When it reads a file (code or log), it does not stream the entire file into the model's context. Instead, it reads the file in **chunks of roughly 100-200 lines** via its internal `read_file` tool, and only the **first ~100 lines are guaranteed** to be read on a given call. Anything beyond that first chunk must be surfaced through an additional text search or semantic search pass against the file. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

Because of this chunked, bounded read behavior, a Cursor agent is effectively **"blinded"** on any file — code or log — that does not carry semantic markers the agent can search for. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

Within GRACE, this constraint is one of the direct, concrete operational reasons the methodology pushes so hard on in-source semantic markup (KEYWORDS, LINKS, START-END tags, per-function contract headers): those markers are what makes Cursor's chunked, search-based reading actually land on the relevant section of a file. [empirical] (post 3819; `xml_mapping_anchors.md` §"Почему семантическая разметка")

## Why

### The mechanism behind the blinding

Cursor's `read_file` returns a window. On a file of any real size, that window covers only a fraction of the content, with only the opening ~100 lines treated as guaranteed. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка") For everything past that guaranteed prefix, the agent has to fall back on **text search or semantic search** against the remainder of the file to locate the region it needs. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

If the file has no semantic anchors — no KEYWORDS, no LINKS, no START-END paired tags, no per-function contract header — there is nothing for those searches to latch onto. The agent's search either returns nothing useful, or returns arbitrary matches that don't correspond to the section the agent actually needed. At that point the AI cannot find the relevant parts of the log or code and effectively operates on a partial, misleading view of the file. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

This is the Cursor-specific expression of the more general GRACE claim that **RAG agents read in small chunked windows and are blind without semantic anchors** — the methodology-level framing shows up here as a concrete tool limitation. [author-claim] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

### Why logs are hit especially hard

The sources call out logs specifically. Cursor "cannot normally read a log longer than 100 lines" unless the log carries semantic markup. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка") Long traces are exactly where the chunked-read limit bites, and the author frames this as the motivating case for log-markup practice: the claim is that without markers the log becomes unreachable to the Cursor agent in any useful sense. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

### Even the 1M-token escape hatch is not enough on its own

The natural reflex for large logs is to escape Cursor's chunked reads by pasting the log into a model with a massive context window — Gemini's ~1M-token window is the canonical example in the sources. The sources explicitly warn that this is **not a free fix**: Gemini's 1M-token window still does not help if the log lacks semantic markup, because to that model an unmarked log is "chaotic text entropy". [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

The implication is that the underlying requirement — **semantic markers** — is invariant across tools. Switching IDE does not remove it; it only changes where the bottleneck shows up. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

### GRACE markup is what keeps Cursor useful

The counterpart to the "blinded" claim is an empirical payoff statistic the author reports from trainees who work in Cursor: when code or logs are marked up in the GRACE style, **~90% of Cursor's semantic searches land directly on the "enrichment points"** — the KEYWORDS and LINKS sections in particular. [empirical] (post 3819) These KEYWORDS/LINKS sections behave as "chunk enrichment" for the vector RAG that Cursor uses, giving semantic search strong hooks to latch onto. [empirical] (post 3819)

So the Cursor limitation is not positioned in the sources as a reason to avoid Cursor — it is positioned as a reason that GRACE-style markup is effectively mandatory there. Without it, Cursor's chunked-read model goes blind; with it, the same chunked-read model lands where it needs to 9 times out of 10. [empirical] (post 3819; `xml_mapping_anchors.md` §"Почему семантическая разметка")

## How

### The chunked-read pattern in practice

Operationally, a Cursor agent working on a file follows roughly this path:

1. `read_file` is called on the target file. The first ~100 lines come back guaranteed; past that, the read is chunked in ~100-200-line windows. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")
2. If the code/log section the agent needs falls outside that initial window, the agent switches to **text search or semantic search** on the file to locate the section. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")
3. Those searches only succeed if the file contains anchors the search can match. Without GRACE-style markers, the searches either fail to find the region or return unrelated matches, and the agent loses the actual context. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

### GRACE's mitigation: density of anchors

GRACE addresses this by ensuring every meaningful section of code or log carries structured, searchable anchors. Two mechanisms in particular carry the weight in the Cursor context:

- **KEYWORDS and LINKS sections** in contract headers — compressed "wenyan-style" keyword lists and cross-module references that are dense with the exact tokens a semantic search will hit. [empirical] (post 3819) The author notes that Cursor trainees observe ~90% of semantic searches "landing" exactly on these enrichment points. [empirical] (post 3819)
- **Paired START-END tags** in comments (and equivalents like C#'s `region`) that let a simple (non-AST) text search identify bounded regions. The sources place this primarily in the Dyck-language / markup rationale, but the practical effect is the same: a searchable, bounded anchor a chunked reader can locate. [author-claim] (post 3948) (see `../04-markup-system/start-end-tags.md` for the broader rationale)

### Very long logs: copy to Gemini, but keep markup

The sources give one operational escape route for logs that are too long to navigate inside Cursor even with markup: **copy the log out of Cursor and into Gemini's 1M-token window for whole-log analysis**. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")

Two caveats are stated explicitly:

- The log **still needs semantic markers** — Gemini cannot make sense of an unmarked log either; without anchors it is "chaotic text entropy" even at 1M tokens. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка")
- The author teaches log markup practices based on **LOFT** with modifications tuned for AI — specifically, modifications aimed at letting the AI localize the needed region of the log. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка") (The sources do not elaborate further on what the LOFT-plus-AI-modifications look like beyond that reference.)

### The reactions that motivated the claim

The pattern the author reports is that Cursor agents running Claude 3.7 and Gemini Pro produce "loud enthusiasm" in response to GRACE-style semantic markup in **logs** specifically — not only in code. [empirical] (`xml_mapping_anchors.md` §"Почему семантическая разметка") This is presented as direct, repeated observation of the markup-vs-no-markup gap inside Cursor, and is the anecdote the §"Почему семантическая разметка" section opens with.

## Evidence

- **`xml_mapping_anchors.md` §"Почему семантическая разметка"** — primary source for the core claims in this file: Cursor's `read_file` reads in 100-200-line chunks; only the first ~100 lines are guaranteed; past that the AI must use text or semantic search; without semantic markup the AI is "blinded" on code and logs; large logs can be handed off to Gemini's 1M-token window but still need markers; the author teaches log markup based on LOFT with AI-specific modifications. [empirical]
- **Post 3819 (2026-04-05)** — empirical statistic that ~90% of Cursor semantic searches land on enrichment points (KEYWORDS/LINKS) when GRACE-style markup is present; framing of KEYWORDS/LINKS as "wenyan prompting" acting as chunk enrichment for vector RAG; example of Grok reconstructing a module's hidden meaning from contract-only input with ~1:15 compression. [empirical]
- **Post 3948 (2026-04-19)** — formal basis for why START-END tags in comments (Python) give Dyck-language compatibility and thus enable one-pass semantic parsing — indirectly relevant to why Cursor's search-based chunked reading finds these markers reliably. [research-backed] (Shunyu Yao et al., arxiv 2105.11115)

No benchmark numbers, vendor documents, or paper URLs are reported in the sources for the specific Cursor chunk-size claim itself — it is an author-empirical observation. The 90% hit-rate figure is likewise author-empirical, reported from trainees working in Cursor. [empirical]

## See also

- `../04-markup-system/start-end-tags.md` — paired START-END tag mechanics that make anchors searchable without an AST.
- `../04-markup-system/xml-like-markup.md` — the broader markup format choice that powers the anchors Cursor's search hits on.
- `../03-contracts/contract-fields.md` — the KEYWORDS and LINKS fields whose density drives the ~90% semantic-search hit rate reported for Cursor.
- `../03-contracts/wenyan-prompting.md` — compression technique behind KEYWORDS and LINKS; the same post (3819) is the origin of the Cursor hit-rate statistic.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — the transformer-level analog of the same "blind without anchors" problem that manifests here as a tool-level limitation.
- `../08-workflow-and-phases/architect-code-debug-modes.md` — Kilo Code's explicit Architect/Code/Debug UI modes are contrasted by the sources against Cursor's lack of explicit mode UI, making GRACE "less ergonomic" in Cursor even though the underlying mechanics can be carried via a prompt RAG.
- `../09-tooling/kilo-code-open-code.md` — the Kilo Code / Open Code stack the author migrated GRACE to, which avoids the chunked-read bottleneck differently.
- `../05-logging-ldd/log-to-code-correlation.md` — why the same markup argument extends to logs, with the log-to-code correlation heuristic.
