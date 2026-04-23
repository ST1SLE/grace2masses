# Mikhail Evdokimov — Bank RAG on GRACE Principles

## What

A reported production case in which **Mikhail Evdokimov** — CPO of ALGA Group International and 1st Deputy Chairman of the Board of O!Bank — built a Retrieval-Augmented Generation (RAG) system for bank documents on top of the author's GRACE methodology. Instead of applying GRACE to source code (its canonical domain), Mikhail applied GRACE's graph-first structural approach and contract-style semantic compression to complex enterprise textual documents. [empirical] (post 3839)

The case is notable for three reasons:

1. It is one of the few **publicly attributed** GRACE deployments outside code. [author-claim] (post 3839)
2. The author reports **at least three private bank cases** with similar approaches, suggesting a pattern inside Russian banking rather than an isolated experiment. [empirical] (post 3839)
3. It is the canonical evidence used by the author to argue that GRACE generalizes beyond code to any "complex text" problem. [author-claim] (post 3839)

> The GRACE build plan treats this case as the evidence anchor for the broader "GRACE for RAG documents" pattern (see [`../10-beyond-code/grace-for-rag-documents.md`](../10-beyond-code/grace-for-rag-documents.md) for the general extension; this file focuses on the case itself — who, what, and what evidence exists).

## Why

### Why this case matters for the KB

GRACE was refined against a code-generation problem: large AI-generated applications (800–1000+ LOC) collapsing past a critical mass threshold because AI agents lose navigation in dense, interconnected logic. [author-claim] The mechanism — sparse attention collapse past ~4k tokens, random-attention fallback, KV-cache freezing — is a **transformer-internal** phenomenon, not a code-specific one. [research-backed] (most_important.md §"Почему без семантического графа")

If the collapse is transformer-internal, then any sufficiently long and interconnected text should suffer the same failure mode, and any methodology that fixes it for code should fix it for text. The Evdokimov case is the first **third-party, non-code, enterprise-grade** deployment the author cites as empirical confirmation of this prediction. [author-claim] (post 3839)

### Why bank documents specifically

The sources do not specify the exact document categories Mikhail's RAG ingests. The sources do not specify the regulatory, contractual, or operational mix. What the author does state is that bank documents are a natural fit because they are:

- long (multi-hundred- to thousand-page corpora — the sources do not specify exact lengths);
- internally cross-referenced (documents cite each other, regulations reference laws, policies reference procedures);
- semantically dense (little filler; high information density per page);
- high-stakes (wrong retrieval or misinterpretation has compliance and financial consequences).

These are the same qualitative properties that GRACE addresses in code: length exceeding the dense-attention window, heavy cross-referencing, and low tolerance for hallucination. [author-claim] (post 3839; inferred from the general GRACE argument in most_important.md)

### Why at least three bank cases

The author reports that Mikhail's is not the first such approach — at least two more **private bank cases** exist with "similar approaches." [empirical] (post 3839) The sources do not specify which banks, which teams, or how similar "similar" is. What the author claims is that the pattern — graph + per-document semantic squeeze — has been independently re-discovered or adapted in Russian banking.

Implication for the reader: this is not a single datapoint that could be an outlier; it is at minimum three. [empirical] (post 3839)

## How

### The architecture Mikhail reportedly built

The sources give a compact description. The system has two architectural layers, isomorphic to GRACE's code-level design:

1. **A main map expressed as a graph.** This plays the role that GRACE's top-level architectural graph plays in code: navigation anchors for sparse-attention agents, hierarchical decomposition of the corpus, and edges between conceptually related documents. [empirical] (post 3839)
2. **A per-document "semantic squeeze"** (Russian: "семантическая выжимка"), explicitly described as **analogous to a code contract**. [empirical] (post 3839) In the code case, a contract is a compact pre-function annotation containing PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS (see [`../03-contracts/contract-fields.md`](../03-contracts/contract-fields.md)). By analogy, a document-level squeeze is a compact header containing the document's purpose, its inputs/scope, its outputs/conclusions, keywords for semantic retrieval, and links to related documents.

The sources do not specify the exact field names Mikhail uses, the exact markup format, or whether he preserves GRACE's XML-like START-END tag convention for the document squeezes. The sources do not specify the underlying vector store, retrieval pipeline, LLM choice, or reranker configuration. The sources do not specify metrics.

What the sources DO specify explicitly:

- **Graph as main map**: yes, graph-structured. [empirical] (post 3839)
- **Per-document analog of code contracts**: yes, named "semantic squeeze" and explicitly described as similar to a code contract. [empirical] (post 3839)
- **Author did not audit the implementation in detail**: "I did not study Mikhail's implementation in detail." [author-claim] (post 3839)

### The adaptation rule

The author states the general adaptation principle around this case:

> "GRACE is not just about code, but a general methodology for agent work with complex texts. My templates are simply standardized for code, but they can be adapted to other contexts." [author-claim] (post 3839)

This means: the **structure** of GRACE (graph-first, per-node semantic compression, KEYWORDS + LINKS hooks, Dyck-compatible markup) transfers directly. The **templates** (specific field names, language choices, file conventions) require redesign per domain. The sources do not specify which of GRACE's templates Mikhail kept verbatim and which he rewrote.

See [`../10-beyond-code/grace-for-rag-documents.md`](../10-beyond-code/grace-for-rag-documents.md) for the general pattern and the template-adaptation logic.

### Why the squeeze is the load-bearing innovation (inferred from GRACE principles)

In the code case, the author's research-referenced claim is that:

- **PURPOSE** (in-source intent statement) is **load-bearing**: the agent can skip DESCRIPTION entirely, but not PURPOSE — omitting it causes measurable degradation. [research-backed] (post 3889, arxiv 2601.16661v1)
- **Shorter is better**: long descriptions are either neutral or harmful; aggressive compression via "wenyan-style" KEYWORDS yields up to +10% HumanEval. [research-backed] (post 3892, ShortenDoc paper — dl.acm.org/doi/10.1145/3735636)
- **Adjacency matters**: if description is separated from content by MCP hops or file switches, degradation follows. [research-backed] (post 3889)
- **Architectural intent improves bug-fix accuracy up to 3×** in code; attention analysis shows agents directly attend to intent tokens. [research-backed] (post 3890, ResearchGate publication 400340807)

These results are about code, but the mechanism (token-level attention over compact, intent-revealing anchors) is transformer-internal and not code-specific. The author's argument — unstated here but implicit — is that the per-document semantic squeeze transfers these gains to document RAG. The sources do not specify whether Mikhail measured these effects empirically in the bank deployment.

### What the squeeze replaces

In a vanilla document-RAG stack, a document enters the vector store as chunks. Retrieval depends on semantic similarity of the chunk embeddings to the query. This fails for:

- **exact citations** (regulation paragraph numbers, policy clause IDs);
- **precise facts** (dates, monetary thresholds, signatory names);
- **cross-document relationships** (policy X references regulation Y, which amends law Z). [author-claim] (post 3732)

The per-document squeeze, combined with the main-map graph, replaces the missing structure:

- KEYWORDS-style hooks become **semantic-search enrichment points** — in the code case, the author reports 90% of Cursor semantic-search hits landing on KEYWORDS/LINKS enrichment rather than raw code. [empirical] (post 3819) By analogy, document-level KEYWORDS and LINKS should catch semantic queries that would miss bare chunks.
- LINKS between document squeezes replace a classical document-graph database, as they do for Call Graph in code. [author-claim] (post 3732)
- The main-map graph gives the agent sparse-attention anchors for navigating long RAG contexts — the same role the code-level semantic graph plays past 4k tokens. [research-backed] (most_important.md §"Почему без семантического графа")

For details on the hybrid-RAG infrastructure argument this rests on, see [`../09-tooling/hybrid-rag-infra.md`](../09-tooling/hybrid-rag-infra.md) and [`../02-semantic-graph/graph-rag-vs-vector-rag.md`](../02-semantic-graph/graph-rag-vs-vector-rag.md).

### Implementation gaps the sources leave open

For a reader trying to replicate Mikhail's design, the following are **not specified in the sources** and cannot be inferred without inventing:

- the exact field set in the per-document squeeze;
- the markup format (XML-like? Markdown? custom?);
- the RAG infrastructure (which vector DB, graph DB if any, reranker, LLM);
- the graph's schema (node types, edge types, cardinality);
- the ingestion pipeline (who writes the squeeze — human, AI, hybrid?);
- the evaluation methodology;
- the specific document classes covered;
- the scale (documents, tokens, query volume).

For these, the reference article [habr.com/ru/articles/1020548/](https://habr.com/ru/articles/1020548/) is cited but the sources in this KB do not include its contents. [author-claim] (post 3839)

## Evidence

### Primary source

**Post 3839** (2026-04-08 10:46 UTC): the single post announcing and characterizing Mikhail's implementation. Full text attributes the case, names the two organizations (ALGA Group International, O!Bank), describes the architecture ("main map is also a graph, and for individual documents there is a semantic squeeze, similar to a code contract"), notes the author did not audit it in detail, reports at least two additional private bank cases, generalizes GRACE to complex texts beyond code, and links to the Habr article. [empirical] (post 3839)

### Referenced external article

- [habr.com/ru/articles/1020548/](https://habr.com/ru/articles/1020548/) — Mikhail's own write-up. Linked from post 3839. The sources in this KB do not quote or summarize the article's contents; only its existence and URL are cited. [vendor-doc-adjacent] (post 3839)

### Author's positions used to contextualize the case

- **GRACE generalizes beyond code**: "GRACE is not only about code, but a general methodology for agent work with complex texts." [author-claim] (post 3839) Re-stated in [`../00-foundations/what-is-grace.md`](../00-foundations/what-is-grace.md) as a founding principle.
- **Methodology-as-graph litmus test**: any methodology that can be graphed is operable by AI; one that cannot is text entropy. [author-claim] (most_important.md §"Методология как граф") Indirectly supports the decision to port GRACE structurally rather than just linguistically.
- **At least three similar bank cases exist**: "this is not the first such approach in banks. There are at least two more private cases." [empirical] (post 3839) The sources do not name them.

### Research anchors that motivate the design (all code-domain originally; transfer is the author's claim, not the papers')

- **In-source documentation benchmarks** — arxiv.org/abs/2601.16661v1 — PURPOSE is load-bearing; adjacency matters; English > other languages; -95% to +435% effect size depending on structure. [research-backed] (post 3889)
- **ShortenDoc** — dl.acm.org/doi/10.1145/3735636 — 40% compression without quality loss; ~10% HumanEval gain. [research-backed] (post 3892)
- **Architectural intent and bug-fix accuracy** — researchgate.net/publication/400340807 — intent comments improve bug-fix accuracy up to 3×. [research-backed] (post 3890)
- **Native Dyck-Language support in transformers** — arxiv.org/abs/2105.11115 — transformers natively parse paired-bracket formats; the basis for XML-like markup in both code contracts and (by analogy) document squeezes. [research-backed] (post 3948)

The author does not cite Mikhail's deployment as having measured these effects; these are the mechanistic reasons the author argues the transfer should work. [author-claim]

### What is NOT evidence

- The sources do not specify any quantitative retrieval-quality metric from Mikhail's deployment.
- The sources do not specify the production status (pilot? production? scale?).
- The sources do not specify the LLM vendor or model.
- The sources do not specify whether squeezes are authored by humans, AI, or a mix.
- The sources do not specify what the "main map" graph schema looks like.

Readers who want those details must go to the Habr article (habr.com/ru/articles/1020548/), which the sources cite but do not reproduce.

## Cross-reference: how this case appears in other KB entries

- **[`../10-beyond-code/grace-for-rag-documents.md`](../10-beyond-code/grace-for-rag-documents.md)** — the **general pattern** (graph + semantic squeeze for document RAG), with Mikhail as the anchor case. This file is the case write-up; that file is the pattern write-up.
- **[`../00-foundations/what-is-grace.md`](../00-foundations/what-is-grace.md)** — GRACE's beyond-code positioning; Mikhail's case is the illustrative example.
- **[`../02-semantic-graph/graph-as-backbone.md`](../02-semantic-graph/graph-as-backbone.md)** — the graph-first principle, from which the "main map is also a graph" decision follows.
- **[`../02-semantic-graph/graph-rag-vs-vector-rag.md`](../02-semantic-graph/graph-rag-vs-vector-rag.md)** — the hybrid RAG argument (vector-only is insufficient; LINKS enrichment replaces a separate graph DB for many cases).
- **[`../03-contracts/contracts-overview.md`](../03-contracts/contracts-overview.md)** — the code contract concept that the "semantic squeeze" is the document-domain analog of.
- **[`../03-contracts/contract-fields.md`](../03-contracts/contract-fields.md)** — the canonical field set that would presumably be adapted per document class.
- **[`../03-contracts/wenyan-prompting.md`](../03-contracts/wenyan-prompting.md)** — the compression technique behind KEYWORDS/LINKS that presumably drives the per-document squeeze.

## See also

- [`../10-beyond-code/grace-for-rag-documents.md`](../10-beyond-code/grace-for-rag-documents.md) — general pattern for document-RAG with GRACE
- [`../10-beyond-code/general-complex-text.md`](../10-beyond-code/general-complex-text.md) — broader generalization of GRACE to non-code methodologies
- [`../00-foundations/what-is-grace.md`](../00-foundations/what-is-grace.md) — GRACE scope and positioning
- [`../02-semantic-graph/graph-as-backbone.md`](../02-semantic-graph/graph-as-backbone.md) — why a graph is mandatory as the main map
- [`../02-semantic-graph/graph-rag-vs-vector-rag.md`](../02-semantic-graph/graph-rag-vs-vector-rag.md) — hybrid retrieval argument
- [`../03-contracts/contracts-overview.md`](../03-contracts/contracts-overview.md) — code-contract concept that the per-document squeeze is analogous to
- [`../03-contracts/contract-fields.md`](../03-contracts/contract-fields.md) — canonical contract fields available for per-domain adaptation
- [`../03-contracts/wenyan-prompting.md`](../03-contracts/wenyan-prompting.md) — compression technique relevant to the squeeze
- [`../09-tooling/hybrid-rag-infra.md`](../09-tooling/hybrid-rag-infra.md) — sqlite-vec and hybrid-RAG infrastructure choices
- [`../14-research-references/annotated-bibliography.md`](../14-research-references/annotated-bibliography.md) — bibliographic entries for the research papers cited above
