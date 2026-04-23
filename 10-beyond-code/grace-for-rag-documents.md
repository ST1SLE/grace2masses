# GRACE for Document RAG

## What

GRACE principles transfer outside code. The governing evidence is a production case: **Mikhail Evdokimov** — **CPO of ALGA Group International** and **1st Deputy Chairman of the Management Board of O!Bank** — built a RAG system for enterprise documents on top of GRACE. [empirical] (grace_matches.md post 3839)

The shape of Evdokimov's system is the GRACE shape, translated from code to documents (post 3839):

- A **graph-based main map** plays the role of the top-level semantic scaffold — the same navigational role the architectural graph plays over a codebase.
- Per document, a **semantic squeeze** (семантическая выжимка) sits in front of the full text — a deliberate analog of the **module/function contract** that sits in front of a code block in GRACE.

The author's framing in post 3839 is explicit: "GRACE is not only about code — it is a general methodology for an agent working with complex texts. The templates are simply standardized for code, but they can be adapted to other contexts." [author-claim] (grace_matches.md post 3839)

The public reference for Evdokimov's write-up: **habr.com/ru/articles/1020548/**. [empirical] (grace_matches.md post 3839)

## Why

### The operating constraints do not change when you leave code

Documents hit the same transformer and RAG-agent constraints that GRACE was designed around — the constraints are properties of the model, not of the source-type:

- **Sparse-attention collapse past ~4k tokens.** Any long document (a contract bundle, a regulation, a compliance package) crosses the same boundary where the ~4k dense-attention window runs out and sliding-window sparse attention takes over, needing anchors to stitch across. See `../01-transformer-foundations/sparse-attention-and-kv.md`. [research-backed] (most_important.md §"Почему без семантического графа")
- **Chunk-based retrieval agents** read documents the same way they read code — in windows, without an inherent understanding of cross-document structure. Without semantic anchors they are "blinded" in long documents just as they are blinded in long logs or long files. See `../09-tooling/cursor-limitations.md`. [empirical] (xml_mapping_anchors.md §"Почему семантическая разметка")
- **Pure-vector RAG misses exact facts**, which is exactly the thing that matters in a financial/legal document (clause citations, precise numbers, named parties). See `../02-semantic-graph/graph-rag-vs-vector-rag.md`. [author-claim] (grace_matches.md post 3732)

Under these constraints, whatever works for code should work for documents — and whatever fails for code (unstructured chunks, YAML scaffolding, external docs systems the model was never trained on) will fail for documents too. The sources do not specify any additional transformer-internal effect that would make document-RAG fundamentally different from code-RAG.

### Why the Evdokimov case matters specifically

Three things make post 3839 the anchor for this file rather than a side-note:

1. **It is not a toy.** Evdokimov's titles — CPO of a group holding and 1st Deputy Chairman of a bank — put this in a production enterprise context, not a pet project. [empirical] (grace_matches.md post 3839)
2. **It is not isolated.** The author notes explicitly that "this is not the first such approach in banks. At minimum, there exist two further private cases" — i.e., **at least three private bank cases** of GRACE-style document RAG are known to the author. [empirical] (grace_matches.md post 3839)
3. **The author confirms the generalization point at the same time.** Post 3839 is the post where the author states on the record that GRACE is a general methodology for complex text and its code-specific templates are a convenience, not a limit. [author-claim] (grace_matches.md post 3839)

The sources do not specify further technical details of Evdokimov's implementation (exact markup format, storage stack, chunking rules, retrieval pipeline). The author writes: "I did not study Mikhail's implementation in detail." [author-claim] (grace_matches.md post 3839) The habr.com reference is offered for readers who want the implementation; inside this KB, that article is not quoted.

### What generalization actually requires

Applying GRACE to documents is not lift-and-shift. The author's wording in post 3839: templates are **standardized for code**; adapting to other contexts requires **evolving the templates**. [author-claim] (grace_matches.md post 3839)

The sources do not enumerate which template fields survive the port and which have to change. A conservative reading, cross-referenced to the contract-field specification (`../03-contracts/contract-fields.md`):

- **PURPOSE** is the most generally-applicable field — a document-level "what is this for" works the same way as a function-level PURPOSE. PURPOSE is the load-bearing field per post 3889. [research-backed] (grace_matches.md post 3889)
- **KEYWORDS** and **LINKS** transfer directly as RAG-enrichment hooks — they give the vector index architectural traction (see `../02-semantic-graph/graph-rag-vs-vector-rag.md`). [author-claim] (grace_matches.md post 3732)
- **INPUTS / OUTPUTS** — typed-argument fields do not map cleanly to a regulation or a clause. These are the ones that the author's "template evolution" comment most plausibly refers to. The sources do not specify a replacement, so any concrete design is outside what the sources back.

## How

### The hybrid pattern, instantiated for documents

Evdokimov's system, as described in post 3839, instantiates the same hybrid-RAG pattern GRACE uses over code (cross-link `../02-semantic-graph/graph-rag-vs-vector-rag.md`):

1. **Graph-based main map** — the top layer, playing the role of architectural graph. It is what the sparse-attention sliding windows anchor to when the agent crosses 4k-token boundaries. The sources do not specify whether this graph is stored in a graph DB, materialized as markup, or both. [empirical] (grace_matches.md post 3839)
2. **Per-document semantic squeeze** — the document-level analog of a module contract: a compressed, agent-facing summary that sits in front of the full document. Per post 3839 this is explicitly described as "similar to a contract on code." [empirical] (grace_matches.md post 3839)
3. **The full document body** — sits under the squeeze, the way code sits under a contract in the code case.

The pattern is the same top-down cascade as in code (graph → contract → body). See `../02-semantic-graph/hierarchy-and-anchors.md` for the code-side version. The sources do not specify whether Evdokimov's system adds additional tiers (a per-section squeeze under a per-document squeeze, for example) — post 3839 stops at document level.

### What "semantic squeeze" inherits from the code contract

Per post 3839, the squeeze is an **analog** of a code contract. The author does not specify the exact field set, but the analogy implies at least the properties that make a code contract a semantic shield:

- It sits **in front of** the artifact it describes — the same placement rule GRACE uses for code contracts (description before the declaration). [research-backed] (grace_matches.md post 3878)
- It is **compressed** — contracts at module/function level are wenyan-compressed summaries, not long prose. See `../03-contracts/wenyan-prompting.md`. The sources do not specify Evdokimov's compression ratio. [author-claim] (grace_matches.md post 3819)
- It is **written for the AI** — the audience is an LLM agent navigating a large corpus, not a human reader. [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование")

### What changes vs the code templates

Post 3839's only explicit guidance on adaptation: "templates are standardized for code; they can be adapted for other contexts." [author-claim] (grace_matches.md post 3839) The post does not enumerate differences. Readers who want specifics should consult Evdokimov's habr.com article directly — the sources in this KB do not reproduce its technical detail.

What the author **does** re-assert in post 3839 and elsewhere: the principles that survive the port are the GRACE principles — graph-first scaffolding, contracts/squeeze as semantic shield, hybrid rather than pure-vector retrieval. Anything framework-specific to code (type signatures, method-name causal-anchor rules, language-specific START-END tag placement) is the part that needs evolution. The sources do not define the line precisely.

### Other document-domain confirmations in the sources

The sources do not contain further production document-RAG case studies beyond Evdokimov's and the author's reference to at least two further private bank cases (post 3839, author-claim; specifics not disclosed). Adjacent generalizations that the sources do support:

- The author has applied the same graph-first methodology to **construction-work planning** (МS Project methodology graph). See `../10-beyond-code/general-complex-text.md`. [author-claim] (most_important.md §"Методология как граф")
- **Methodology-as-graph litmus test.** Any methodology should be expressible as a graph to be AI-operable; one that is not is "text entropy" to the model (see `../02-semantic-graph/hierarchy-and-anchors.md`). This generalizes the graph-first principle beyond code. [author-claim] (most_important.md §"Методология как граф")

These are supporting arguments for the generalization claim, not additional document-RAG case studies.

## Evidence

### Primary

- **Post 3839 (2026-04-08)** — the governing source. Names Mikhail Evdokimov (CPO ALGA Group International; 1st Deputy Chairman of O!Bank). Describes the RAG system's structure: graph-based main map + per-document semantic squeeze analogous to a code contract. Notes at least two further private bank cases with similar approaches. States the generalization point explicitly: **"GRACE is not only about code; it is a general methodology for an agent working with complex texts. Templates are standardized for code, but they can be adapted for other contexts."** References habr.com/ru/articles/1020548/ for the write-up. All claims in this file about Evdokimov's system trace back here. [empirical on the case; author-claim on the generalization]

### Supporting

- **habr.com/ru/articles/1020548/** — Evdokimov's own write-up of the RAG system, referenced in post 3839. The sources inside this KB do not quote from it; implementation-level questions should be directed there. [external reference]
- **Post 3732 (2026-03-26)** — establishes why vector-only RAG fails on exact facts and why hybrid Graph-RAG is the real answer. Supports the structural argument that document-RAG hits the same constraints as code-RAG. [author-claim] See `../02-semantic-graph/graph-rag-vs-vector-rag.md`.
- **Post 3878 (2026-04-12)** — canonical GRACE contract field set (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS). Grounds the claim that a "semantic squeeze" is an analog of a code contract. [author-claim] See `../03-contracts/contract-fields.md`.
- **Post 3889 (2026-04-14)** — PURPOSE is the critical contract field; short is better than long; English beats non-English; adjacency matters. Supports the "what survives the port" discussion above. [research-backed] See `../04-markup-system/markup-compression.md`.
- **`most_important.md` §"Методология как граф"** — any methodology expressible as a graph is AI-operable; one that cannot be graphed is entropy. Supports the generalization-beyond-code claim at principle level. [author-claim] See `../10-beyond-code/general-complex-text.md`.

### Boundaries

- The sources do not specify the exact markup format, storage stack, chunking rules, embedding model, or retrieval pipeline used in Evdokimov's system. The author states explicitly: **"I did not study Mikhail's implementation in detail."** (grace_matches.md post 3839)
- The sources do not enumerate the specific template-level changes required when porting GRACE from code to documents — only that some template evolution is required.
- The sources do not name or describe the two additional private bank cases beyond Evdokimov's. The only claim is that they exist.
- The sources do not provide benchmarks, hit-rate numbers, or latency figures for document-RAG built on GRACE.

## See also

- `../00-foundations/what-is-grace.md` — the top-level positioning of GRACE as a general methodology for complex text, with the banking case cited there as one piece of evidence for generalization.
- `../10-beyond-code/general-complex-text.md` — the companion file covering the construction-planning application and the methodology-as-graph litmus test; together with this file it forms the "beyond-code" pair.
- `../02-semantic-graph/graph-as-backbone.md` — why a graph is the persistent scaffolding in GRACE; the upstream principle the Evdokimov case re-instantiates.
- `../02-semantic-graph/hierarchy-and-anchors.md` — spec → contract → code hierarchy. The document analog (main map → per-document squeeze → document body) is the same cascade.
- `../02-semantic-graph/graph-rag-vs-vector-rag.md` — why hybrid RAG is needed; the structural argument that applies to documents as much as to code.
- `../03-contracts/contracts-overview.md` — what a code contract is; the thing Evdokimov's "semantic squeeze" is an analog of.
- `../03-contracts/contract-fields.md` — canonical field set (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS); starting point for considering which fields survive the port to documents.
- `../03-contracts/wenyan-prompting.md` — the compression discipline that "squeeze" implies.
- `../11-case-studies/mikhail-bank-rag.md` — case-study treatment of the same Evdokimov system, focused on the case rather than on the generalization argument.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — the sparse-attention boundary that makes the document case structurally identical to the code case.
- `../14-research-references/annotated-bibliography.md` — for the research citations referenced above (posts 3732, 3878, 3889 are not paper-backed; the habr.com article is the only external reference for this file).
