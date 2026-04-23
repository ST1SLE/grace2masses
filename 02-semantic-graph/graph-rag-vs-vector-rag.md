# Graph-RAG vs Vector-RAG — Hybrid Retrieval

## What

Two retrieval strategies that sit behind any RAG pipeline — whether for code, documents, or a knowledge base:

- **Vector RAG** — chunks of source text are embedded into vectors, then similarity-searched at query time. The classical pattern: "embed everything, cosine-search, feed top-k to the LLM" (post 3732). [empirical]
- **Graph RAG / hybrid RAG** — combines vector similarity with explicit relations (call graph, cross-references, SQL facts, semantic links). Does not necessarily require a dedicated graph database: if chunks are pre-enriched with cross-references, vector search alone can stand in for a call graph (post 3732). [empirical]

GRACE's position, distilled in post 3732: **pure vector RAG "won't get you far"** (не уедет). Hybrid RAG is the real answer, and GRACE's `KEYWORDS` + `LINKS` contract sections are the enrichment layer that lets vector search behave like a Graph-RAG without the operational cost of a separate graph DB. [author-claim]

## Why

### Vector search is strong — until you need facts

Vector embeddings are good at **semantic similarity**. Modern models go further: "new Qwen models even confidently find analogies of a chunk with some abstraction via vectors" (post 3732). [empirical]

But three failure modes are structural, not fixable by better embeddings (post 3732):

1. **Exact concept links** — precise, named relations between entities. Embeddings blur them.
2. **Exact citations** — reproducing a literal passage by reference. Embeddings approximate.
3. **Exact facts / numbers** — "precise numbers are practically impossible to store for AI in a vector" (post 3732). [author-claim]

The failure is not a tuning problem. It is what embeddings are: lossy semantic compressions. Asking a vector to hold "a quote" or "a dollar amount" is asking it to be something else.

### A real Call Graph is heavy; often unnecessary

You could bolt a dedicated graph DB onto the pipeline. The author's claim is that you usually do not have to (post 3732): "graph DBs are not so critical yet, because embeddings store concept relations reasonably well **if you enrich chunks with LINKS-like concepts before vectorization**." [author-claim]

This matters operationally — a graph DB adds infra, schema, staleness concerns, and another place for AI agents to get confused (see `../09-tooling/mcp-scepticism.md` for why agents distrust custom data layers in general). GRACE's preference is to push the structure **into the chunk itself** via contract markup, so vector search naturally walks the graph.

### Tables, IDs, numbers — use SQL, not vectors

For "exact table-style facts" vectors alone will misfire (post 3732). The author's recommended combination: **SQL search + vector search in the same store**, via `sqlite-vec` (github.com/asg017/sqlite-vec). [author-claim] This covers the precise-fact case without giving up similarity search.

## How

### The rule of thumb

From post 3732, in practice:

- Use vectors for **similarity** and **analogy** — "what chunk is about this idea?"
- Use structured retrieval (SQL, graph, or enriched chunk fields) for **exact facts, exact citations, exact numbers**.
- Enrich chunks before vectorization so that concept-level edges survive into embedding space.
- Reach for a real graph DB only when LINKS-style enrichment is not enough — "not so critical yet" (post 3732). [author-claim]

### GRACE's cheap Graph-RAG: KEYWORDS + LINKS

GRACE does not require a graph database to get Graph-RAG behavior. Instead, the contract sections carry the edges:

- **KEYWORDS** — architectural patterns and task classifications. These are the hooks for semantic search (see `../03-contracts/wenyan-prompting.md`). Post 3732: GRACE includes "such things in the markup so that vector search works as a replacement for a Call Graph and typically triggers even without a graph DB." [author-claim]
- **LINKS** — cross-module references. Act as a Call-Graph proxy at the contract level (see `../03-contracts/contract-fields.md`).

Post 3819 corroborates with a Cursor-side observation: "semantic search in 90% of cases lands on these 'points of semantic enrichment'" — i.e., the KEYWORDS/LINKS lines are what the vector index actually hits. [empirical] That is Graph-RAG behavior produced by a flat vector store — because the chunk itself encodes relations.

A concrete knock-on effect from post 3819: "keywords hook very well onto agents' search tools — they can immediately find related modules in the code by meaning, even when there are no direct calls and the link is not obvious in code." [author-claim] This is the payoff: the vector index surfaces architecturally-related modules, not just lexically-similar ones.

### When the chunk does not have the fact

If the question is "what is the exact tax rate in row X?" or "cite the clause verbatim," no amount of enrichment saves a pure-vector setup. Post 3732's fix:

- Use **sqlite-vec** (github.com/asg017/sqlite-vec) — it puts vector similarity and SQL exact-match in the same store. [vendor-tool reference from the post]
- Query strategy: vector-search to locate the candidate chunk, then SQL-lookup by key to pull the exact value. The vector does the navigation; SQL does the citation.

Post 3732 explicitly: "for precise 'table-style facts' without integrating vectors with an SQL base you can just blunder, since storing precise numbers for AI in a vector is practically impossible. There are good hybrid solutions like sqlite-vec, I wrote about it in the channel earlier. There you can combine SQL search and vector search." [author-claim]

### When a graph DB is actually warranted

The source is cautious rather than prescriptive. Post 3732 is the only source that weighs this, and its verdict is **"graph DBs are not so critical yet"** when chunks are enriched. [author-claim] The sources do not specify a formal threshold at which a graph DB becomes warranted. A reasonable reading is: if LINKS/KEYWORDS enrichment is infeasible, or the domain relations are not expressible as "a list of names in a contract section," a graph DB starts to earn its keep — but the sources do not pin this down.

### Pipeline sketch

A hybrid RAG assembled in the GRACE style — based on post 3732's prescription and GRACE's markup conventions from posts 3819 and 3878:

1. **Chunking** — respect contract boundaries (a module contract, a function contract, a document section).
2. **Enrichment** — before embedding, append/include KEYWORDS and LINKS from the GRACE contract. These carry the graph edges into the vector (post 3819). [empirical]
3. **Dual index** — store chunks both as vectors (for similarity) and in SQL-indexed tables (for exact-match on IDs, numbers, quoted passages). `sqlite-vec` keeps them in one store (post 3732). [author-claim]
4. **Retrieval** — vector-similarity to navigate, SQL-lookup for exact facts. Use LINKS in retrieved chunks to one-hop to related modules without needing a Call Graph.
5. **Reserve a graph DB** — for cases where the LINKS encoding is insufficient. Post 3732 does not prescribe when that is; the author's empirical default is "not yet." [author-claim]

### The failure mode of pure-vector RAG

Post 3732 closes with the blunt summary: "in any case, if your RAG relies only on vectorization of chunks, you won't get far." [author-claim] The concrete pathologies seen in practice from the same post:

- Abstract-level analogies: fine.
- "What exact number did we agree on?": fails.
- "Which module calls this function?": works if LINKS is in the chunk; fails otherwise.
- "Quote the spec verbatim": fails without SQL/exact-match.

This is not a defect in any specific vector store — it is a property of compressing semantics into a fixed-size vector.

## Evidence

### Primary

- **Post 3732 (2026-03-26)** — the governing source. Lists the advantage of hybrid RAG, identifies vector-RAG's three failure modes (exact concept links, exact citations, exact facts), positions GRACE's KEYWORDS/LINKS as Call-Graph replacement, recommends `sqlite-vec` for precise-fact cases, and closes with "only vectors → you won't get far." All claims in this file about what vectors can/can't do and when a graph DB is needed trace back here. [author-claim, with Qwen-analogy observation as empirical]
- **sqlite-vec** — github.com/asg017/sqlite-vec. Referenced directly in post 3732 as the SQL + vector hybrid store of choice. [vendor-tool]

### Supporting

- **Post 3819 (2026-04-05)** — on wenyan-style compression of KEYWORDS/LINKS. Provides the empirical observation that in Cursor "semantic search in 90% of cases lands on these 'points of semantic enrichment'." [empirical] This is the operational evidence that chunk enrichment converts a vector index into a Graph-RAG approximation.
- **Post 3878 (2026-04-12)** — canonical GRACE contract fields including KEYWORDS; grounds what "LINKS-like concepts" actually look like in code. [author-claim]
- **Post 3839 (2026-04-08)** — Mikhail Evdokimov's bank RAG uses a graph-shaped main map plus per-document semantic squeeze; an example of the same hybrid idea applied outside code. [empirical] See `../10-beyond-code/grace-for-rag-documents.md`.

### Negative / boundary

- **Post 3732** also bounds the claim: hybrid RAG is necessary precisely because pure-vector fails on facts, but LINKS-in-chunks is usually enough — "graph DBs not so critical yet." The claim is not "graph DBs are useless"; it is "enriched chunks defer the need." [author-claim]
- The sources do not specify concrete benchmark numbers for Graph-RAG vs Vector-RAG, nor do they cite a research paper comparing them head-to-head. The argument in post 3732 is structural + empirical, not paper-backed.

## See also

- `../02-semantic-graph/graph-as-backbone.md` — why a graph exists at all in GRACE; the upstream rationale that makes LINKS meaningful.
- `../02-semantic-graph/hierarchy-and-anchors.md` — the spec → contract → code hierarchy whose edges LINKS encodes.
- `../03-contracts/contract-fields.md` — canonical definition of KEYWORDS and LINKS contract sections.
- `../03-contracts/wenyan-prompting.md` — how KEYWORDS/LINKS are compressed (the ~1:15 ratio; 90% Cursor semantic-search hit rate).
- `../09-tooling/hybrid-rag-infra.md` — storage-level decisions: sqlite-vec, when a graph DB is warranted, how to wire enrichment into ingestion.
- `../09-tooling/cursor-limitations.md` — why Cursor's chunked `read_file` makes semantic-enrichment hits the critical path for agent navigation.
- `../10-beyond-code/grace-for-rag-documents.md` — the same hybrid pattern applied to enterprise documents (Mikhail Evdokimov, post 3839).
- `../14-research-references/annotated-bibliography.md` — the sources used; note that post 3732 is not paper-backed, so no entry is cited there for this topic.
