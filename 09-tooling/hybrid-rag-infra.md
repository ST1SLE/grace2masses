# Hybrid RAG Infrastructure — Storage Decisions

## What

This file is the **infrastructure / storage** side of the retrieval discussion in GRACE: what to put the data **into** so that a retrieval pipeline actually works for large AI-generated codebases. It covers three decisions from post 3732:

1. **Pure vector stores are insufficient** for exact facts, exact citations, and precise numbers.
2. **sqlite-vec** (github.com/asg017/sqlite-vec) is the author's recommended combination of SQL exact-match and vector similarity in a single store.
3. **A separate graph database is usually not required** if chunks are pre-enriched with GRACE's KEYWORDS/LINKS contract fields — the enrichment stands in for a Call Graph.

> **Scope note**: this file focuses on **infra/storage choices**. The broader retrieval-strategy debate (why Graph-RAG over pure Vector-RAG, what KEYWORDS/LINKS do semantically, the 90% Cursor-hit observation) is covered in `../02-semantic-graph/graph-rag-vs-vector-rag.md`. Cross-link rather than duplicate. [editorial]

The governing source is **post 3732**. The sources do not specify benchmark numbers comparing these stores; the argument is structural + empirical.

## Why

### Vector-only stores have three structural failure modes

From post 3732, vectors do well on **semantic similarity** — "new Qwen models even confidently find analogies of a chunk with some abstraction via vectors." [empirical] But three kinds of retrieval are structurally broken in a pure-vector setup:

- **Exact concept links** — precise, named relations between entities.
- **Exact citations** — reproducing a literal passage verbatim.
- **Exact facts / numbers** — "precise numbers are practically impossible to store for AI in a vector" (post 3732). [author-claim]

This is not a tuning problem; it is what an embedding is — a lossy semantic compression. For any KB or codebase where the AI agent will need to quote a line, cite a clause, or surface a specific dollar amount, a pure-vector store will fail. Post 3732's summary: "if your RAG relies only on vectorization of chunks, you won't get far." [author-claim]

### A dedicated graph DB adds cost that KEYWORDS/LINKS can usually avoid

Post 3732 weighs a dedicated graph database explicitly and decides against making it the default:

> "graph DBs are not so critical yet, because embeddings store concept relations reasonably well **if you enrich chunks with LINKS-like concepts before vectorization**." [author-claim]

Operationally, a graph DB means: extra infrastructure, extra schema, staleness concerns when code changes, and another data source for the AI agent to possibly distrust (cross-link `mcp-scepticism.md` once written — agents tend to ignore custom non-training-distribution data layers, per posts 3820, 3821). The cheaper route, and the one GRACE defaults to: push the graph edges into the chunk itself via contract markup, then embed.

### Code has a specific simplification: LINKS usually suffices

For **code specifically**, the LINKS section in a GRACE contract typically encodes enough cross-module structure to replace a separate Call Graph (post 3732). [author-claim] Combined with KEYWORDS (architectural patterns, task classifications — see `../03-contracts/contract-fields.md`), the vector search naturally walks the implicit graph because the enrichment is embedded.

This is why GRACE's tooling recommendations stop one level short of graph databases for code: the contract markup is already doing the graph's job.

## How

### Store selection by query type

From post 3732, the practical decision tree:

- **"What chunk is about this idea?"** — vector search alone works (similarity query).
- **"What modules relate to this concept even without direct calls?"** — vector search over KEYWORDS/LINKS-enriched chunks works (post 3819: Cursor hits enrichment points ~90% of the time [empirical]; see `../02-semantic-graph/graph-rag-vs-vector-rag.md`).
- **"What exact number / quote / ID?"** — **requires SQL or another exact-match layer**; vector alone misfires (post 3732). [author-claim]
- **"What does the Call Graph look like between these modules?"** — LINKS in contracts usually enough; reach for a graph DB only when it is not (post 3732). [author-claim]

### sqlite-vec — the recommended hybrid store

Post 3732 names **sqlite-vec** (github.com/asg017/sqlite-vec) as the concrete hybrid solution:

> "for precise 'table-style facts' without integrating vectors with an SQL base you can just blunder, since storing precise numbers for AI in a vector is practically impossible. There are good hybrid solutions like sqlite-vec, I wrote about it in the channel earlier. There you can combine SQL search and vector search." [author-claim]

The relevant property for GRACE pipelines: **one store, both access modes**. Vector similarity walks the semantic graph implicit in KEYWORDS/LINKS; SQL exact-match pulls the literal value when that is what the query needs. No network hop between two systems, no sync problem.

The sources do not specify operational details (index type, dimension, throughput, etc.) — only the recommendation and the GitHub reference. For anything beyond the recommendation itself, the sources defer to the project's own documentation. [editorial]

### When chunks are not enriched — and why GRACE avoids that state

A pure-vector RAG over raw code chunks (no KEYWORDS/LINKS, no contract pre-processing) lands in the failure modes above: no concept-link surfacing, no Call-Graph walk, no exact-fact retrieval. The author's remedy is not "add a graph database" — it is "fix the chunking" (post 3732 + post 3819). [author-claim]

GRACE's enrichment happens at contract-authoring time, not at index time:

- **PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS** are written into the source file by the AI author of the code (see `../03-contracts/contract-fields.md`, `../03-contracts/contracts-overview.md`).
- Chunks are embedded **with the contract comments already present**, so the KEYWORDS/LINKS tokens are inside the embedded region.
- Post 3819's observation — ~90% of Cursor semantic searches land on these enrichment points [empirical] — is the empirical confirmation that this chunking strategy actually concentrates signal on the enrichment lines.

This matters for infra choice: the question "do I need a graph DB?" reduces to "are my chunks enriched?" If yes, post 3732 says you probably do not. If no, the fix is to enrich — not to add a graph DB on top of un-enriched chunks.

### When a graph DB might be warranted

Post 3732 is cautious rather than prescriptive on this threshold. Its verdict is **"graph DBs are not so critical yet"** with the qualifier that enrichment is what makes them unnecessary (post 3732). [author-claim]

What the sources do **not** specify:

- A LOC threshold at which a graph DB is required.
- A benchmark number showing graph-DB superiority.
- A specific graph-DB product recommendation.

A reasonable reading — but one not stated in the sources — is that if LINKS/KEYWORDS enrichment is infeasible (e.g., a legacy codebase with no contracts, where retrofitting is prohibitive), a graph DB can earn its keep. For the target GRACE case — AI-authored code where contracts are written as part of generation — the sources' position is that the graph DB is avoidable. [author-claim, inferential]

### Pipeline sketch (infra view)

Assembling a hybrid store the GRACE way, drawing on post 3732's prescription and the contract conventions in posts 3878 and 3819:

1. **Author contracts with KEYWORDS/LINKS** during code generation. The graph edges go into the source file as comments before embedding (post 3878). [author-claim]
2. **Chunk with contract boundaries respected** — a function's contract + body travel together so the enrichment is inside the embedded chunk.
3. **Store in sqlite-vec** so the same row has both the vector and the exact text/ID/number columns (post 3732). [author-claim]
4. **Retrieval**: vector-similarity for navigation; SQL-lookup by key when the query demands a literal value (post 3732). [author-claim]
5. **Skip the graph DB by default**; reserve it for the edge case where LINKS cannot encode the relation you need (post 3732). [author-claim]

The point is that every one of these steps is **inside the vector/SQL store** or **inside the source file**. Nothing lives in a separate graph service unless forced.

### Relationship to the rest of GRACE tooling

- **Against MCP-on-documentation** (posts 3820, 3821, 3834; to be covered in `mcp-scepticism.md` — T45). If the retrieval layer is sqlite-vec plus enriched chunks, the temptation to build a custom MCP that serves documentation separately is reduced; the agent reads code files where the contracts already live. Post 3820 argues that agents trained on FIM will load whole modules anyway, so the store's job is to surface the right module. [author-claim]
- **Supports Cursor's chunked reads** (post 3819; `cursor-limitations.md`). Cursor's `read_file` returns 100–200-line windows; a hybrid store that lets the agent jump from a KEYWORDS hit into the specific function body minimises wasted reads. [empirical]
- **Supports agent swarms** (`../07-autonomous-agents/`). Multi-agent orchestration needs fast, deterministic retrieval; sqlite-vec's single-store property avoids coordination bugs between separate vector and graph services. [author-claim, inferential]

### What the sources explicitly do not say

It is worth stating the silence, because it bounds what can be asserted:

- No benchmark numbers for sqlite-vec vs alternatives. The recommendation is endorsement, not measurement.
- No explicit list of "bad" vector stores. The argument is against **vector-only**, not against any specific product.
- No claim that sqlite-vec is the only hybrid option — just the one the author recommends and has written about before (post 3732).
- No specification of when exactly a graph DB crosses into "necessary." The language is "not so critical yet."
- No benchmark of enrichment strategies against each other (KEYWORDS alone vs LINKS alone vs both). Post 3819's 90% figure covers the combined effect.

If an engineer needs these numbers, the sources do not specify them — they must be produced empirically on the target project.

## Evidence

### Primary

- **Post 3732 (2026-03-26)** — the governing source for every infrastructure claim in this file. Articulates hybrid RAG's advantage, names the three vector-only failure modes, recommends **sqlite-vec** (github.com/asg017/sqlite-vec), and gives the "graph DB not so critical yet" position. Closes with "if your RAG relies only on vectorization of chunks, you won't get far." [author-claim, with the Qwen-analogy observation as empirical]

### Supporting

- **Post 3819 (2026-04-05)** — on wenyan-prompting compression in KEYWORDS/LINKS. Provides the ~90% Cursor-semantic-search hit rate on enrichment points [empirical] — the operational evidence that contract enrichment converts a flat vector store into Graph-RAG behaviour without a graph DB. Cross-link in `../03-contracts/wenyan-prompting.md`.
- **Post 3878 (2026-04-12)** — canonical GRACE contract fields (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS) and the rule that description sits before declaration for causal-read anchor formation. This is what "enriched chunks" concretely contain. [author-claim]
- **Post 3820 (2026-04-05)** — LLM training-pipeline argument for **why** external documentation stores (including custom docs-MCPs and graph DBs) conflict with training distribution. Supports the "stay inside the source file" infra posture. [author-claim]
- **Post 3821 (2026-04-05)** — Sergei Muravyev doc-MCP discussion. Reinforces that the agent's real task is "find modules," not "understand class" — a store optimised for module location + exact-fact retrieval fits this shape; a separate docs-MCP misdefines the task. [author-claim]
- **Post 3834 (2026-04-07)** — the "kulibin CLI" failure case, supporting the general infrastructural preference for training-distribution-standard tools over DIY layers (relevant by analogy to custom graph-DB stacks). [empirical, author-claim]
- **Post 3839 (2026-04-08)** — Mikhail Evdokimov's bank RAG: graph-shaped main map + per-document semantic squeeze. An outside-code example of the same hybrid pattern; demonstrates the pattern is not code-specific. [empirical]

### External reference (from sources)

- **sqlite-vec** — github.com/asg017/sqlite-vec. Referenced directly in post 3732 as the SQL + vector hybrid store of choice. The sources do not specify version, benchmark, or integration details beyond the reference. [vendor-tool]

### Negative / boundary

- The sources do not cite a research paper comparing Graph-RAG, Vector-RAG, and hybrid stores head-to-head. Post 3732's argument is structural (what vectors can/cannot hold) plus empirical (the Qwen analogy observation, author's own experience with GRACE chunks). [editorial]
- The sources do not prescribe a LOC, query-type, or data-volume threshold at which a graph DB becomes necessary. "Not so critical yet" is a default, not a rule. [editorial]
- No claim about query latency, index size, or throughput is in the sources for any of the recommended tools. [editorial]

## See also

- `../02-semantic-graph/graph-rag-vs-vector-rag.md` — the retrieval-strategy debate (why hybrid wins, what KEYWORDS/LINKS replace, the 90% Cursor hit observation). This file is the strategic companion; the file you are reading is its infra-side counterpart.
- `../02-semantic-graph/graph-as-backbone.md` — why a graph exists at all in GRACE; the upstream rationale that makes LINKS meaningful.
- `../03-contracts/contract-fields.md` — canonical definition of KEYWORDS and LINKS and the placement rule that makes enrichment effective.
- `../03-contracts/wenyan-prompting.md` — compression of KEYWORDS/LINKS that makes them effective anchor tokens in embedding space.
- `../09-tooling/cursor-limitations.md` — Cursor's chunked `read_file` and why semantic-enrichment hits are load-bearing for agent navigation.
- `../09-tooling/mcp-scepticism.md` — broader argument against custom data layers that sit outside the training distribution; grounds the "prefer SQL + vector in one store" posture.
- `../10-beyond-code/grace-for-rag-documents.md` — Mikhail Evdokimov's document-RAG application of the same hybrid pattern.
- `../14-research-references/annotated-bibliography.md` — the cited external references; note that post 3732 is not paper-backed.
