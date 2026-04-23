# General Complex Text — Methodology-as-Graph Beyond Code

## What

GRACE is not a code-only methodology. The author's framing in post 3839 is explicit: **"GRACE is not only about code — it is a general methodology for an agent working with complex texts. The templates are simply standardized for code, but they can be adapted to other contexts."** [author-claim] (grace_matches.md post 3839)

Two anchors in the sources support the generalization outside software:

1. The author has applied the **same graph-first approach** outside software — specifically to **construction-work planning**, formalised as a methodology graph that feeds a Microsoft Project schedule. [author-claim] (most_important.md §"Методология как граф")
2. The author promotes a **methodology-as-graph litmus test**: any methodology that can be rendered as a connected graph is operable by an AI agent; any methodology that cannot be rendered as a graph is, to the model, **"text entropy"** (текстовая энтропия) — not a method at all. [author-claim] (most_important.md §"Методология как граф")

This file is the companion to `./grace-for-rag-documents.md` (T48), which treats the document-RAG instance of the same generalization. Where T48 covers the banking case (Evdokimov's RAG), T49 covers the **principle** — why graph-first scaffolding works on any complex text, with the construction-planning example and the litmus test as the two load-bearing pieces of evidence.

## Why

### The constraints are properties of the transformer, not of code

GRACE was designed around three operational constraints that belong to the model, not to source code [research-backed] (most_important.md §"Почему без семантического графа"; cross-link T06, `../01-transformer-foundations/sparse-attention-and-kv.md`):

- Dense attention runs out at roughly 4k tokens; beyond that, sliding-window sparse attention takes over (100–200 token neighbourhoods).
- Sparse attention can only stitch those windows together through a **semantic graph of anchors**, the "table of contents" that lets the model move across long material.
- Without an explicit anchor graph, the model falls back to random attention or fabricates an internal graph from a partial causal read and **freezes the wrong version in the KV cache** — "GPT becomes stubborn as a donkey" about a wrong branch once the cache is populated (most_important.md §"Граф как основа навигации") [author-claim].

None of those facts refer to source code. They refer to token sequences, regardless of whether the tokens are Python, a construction-work plan, a legal clause, or a methodology writeup. The same hybrid-RAG argument applies: pure-vector retrieval cannot handle exact facts, named entities, or precise numbers (post 3732), which matters in any domain where citations are load-bearing [author-claim] (grace_matches.md post 3732; cross-link T12, `../02-semantic-graph/graph-rag-vs-vector-rag.md`).

Because the constraints are model-level, the mitigation — **graph-first scaffolding of anchor tokens** — is model-level too. GRACE's top-down hierarchy (spec → contract → artefact) is the concrete instantiation of that mitigation. The artefact type can swap out without the hierarchy's shape changing.

### Why "methodology" is the interesting unit of generalization

The author's generalization argument does not stop at "long documents." It runs at the level of **methodology** itself — the body of structured knowledge that a human teacher, consultant, or domain expert uses to organise work [author-claim] (most_important.md §"Методология как граф").

The key claim (most_important.md §"Методология как граф") [author-claim]:

> With the arrival of AI there is a very important point: if a methodology cannot be represented as a graph, then for the AI it is not a methodology — it is just text entropy.

This is a **litmus test**, not a soft guideline. The proposition has two directions:

- **Operable direction.** If your methodology *can* be rendered as a connected graph of nodes and edges, AI can act on it. The graph is what the model's attention navigates; the nodes become the anchor tokens that survive past the 4k-token dense-attention boundary.
- **Inoperable direction.** If your methodology *cannot* be rendered as a graph, you are not teaching the agent a method — you are loading text without semantic correlations into its context. Without correlations there are no attention edges; without edges the material has no structural hook for sparse attention to latch onto. The material becomes entropy relative to the model's processing.

This is why graph-first is not a cosmetic choice: it is the test for whether what you are giving the agent is a method or noise. Cross-link T05 Principle 7 (`../00-foundations/core-principles.md`) and T11 (`../02-semantic-graph/hierarchy-and-anchors.md`), which the author frames as the in-code version of the same argument.

### A useful side-effect: graphing a methodology exposes its "water"

The litmus test has an exploitable side-effect. When the author graphs the content of third-party methodologies — specifically paid courses by coaches — the result is a diagnostic (most_important.md §"Методология как граф") [author-claim / empirical]:

> It is funny to build such graphs over parts of the courses of modern coaches. Often out of teaching that cost a pile of money and time, 5–6 nodes remain in the graph. GPT simply flags the rest as "water, platitudes, and old material." Since there is no semantic value, GPT just drops it from the methodologist's graph.

The author gives that filtering its own label: graphs over methodology are a **litmus test for whether you are being taken in** ("хороший лакмус — разводят вас или нет"). [author-claim] (most_important.md §"Методология как граф")

Why this works as a filter [author-claim]:

- **GPT does not include filler in graphs.** Filler produces no attention correlations — no node-to-node edges in the induced graph. So when GPT is asked to construct a methodology graph, material with no correlations has nothing to attach to and gets dropped.
- The test relies on the **construction act**, not on commentary. You are not asking GPT to rate the course; you are asking it to build a graph from the course's text. What survives is the signal.

This is specifically the reason graphing a methodology is cheaper and more reliable than asking an LLM to review it narratively. Prose review is exactly the post-hoc-explanation mode the author warns against (see "Hard caveats" below). Graph construction is a constructive act and its output is a structured artefact that you can inspect directly.

## How

### The construction-planning case: identical shape, different artefact

The single concrete domain example in the sources outside software is **construction-work planning** (most_important.md §"Методология как граф") [author-claim]:

> I ran training today for a builder on planning work from the design estimate (ПСД) to a schedule in Microsoft Project. Before that, I had formalised the methodology of construction-work planning as a methodology graph. I built and use an analogous graph for the methodology of AI-assisted development.

The artefact flow the author describes in that training parallels GRACE's flow in code point-for-point:

| Layer in construction-planning methodology | Corresponding layer in GRACE for code |
|---|---|
| **ПСД** (project/design estimate — the target spec for the build) | **TZ** / spec at the top of the semantic-artifact hierarchy |
| **Methodology graph of construction-work planning** (the author's own artefact) | **Architectural graph** of classes/modules + business-process mapping |
| **Microsoft Project schedule** (the downstream operable artefact) | **Code** — the downstream operable artefact |

The sources do not describe the detailed nodes/edges of the construction graph, nor do they specify that graph's representation format (whether it lives in markup, a graph DB, or a tool-specific file). The only facts on record: (i) the graph exists; (ii) the author uses it to teach builders to move from ПСД to an MS Project schedule; (iii) the shape is analogous to the AI-development methodology graph [author-claim]. Do not invent the node list.

What the parallel establishes is the **pattern**, not the specific build:

1. **Start from the spec.** In construction, that is the design estimate; in code, that is the TZ; in documents (Evdokimov's bank RAG, T48), that is the corpus-level main map. The spec is a graph-shaped artefact in all three cases.
2. **Pass through the methodology graph.** In construction, it is the planning-methodology graph. In code, it is the architectural graph + module/function contracts. In documents, it is the main map + per-document semantic squeezes.
3. **Emit a schedulable / buildable / retrievable artefact at the leaf.** In construction, the MS Project schedule. In code, the generated code. In documents, the retrieved chunks + their squeezes.

All three are instances of the same shape: **spec → graph → operable artefact**. That is the GRACE shape. Cross-link T11 (`../02-semantic-graph/hierarchy-and-anchors.md`) for the code-level formalisation of the same hierarchy, and T48 (`./grace-for-rag-documents.md`) for the document-level instance.

### Operating the litmus test

Practical procedure for the methodology-as-graph test — grounded in what the sources actually describe, not invented [author-claim] (most_important.md §"Методология как граф"):

1. **Prepare the source material.** Any text body claiming to be a methodology: a course, a manual, an internal standard, a consulting deck's transcript.
2. **Ask GPT to construct a methodology graph from that material.** Nodes and edges — what concepts, tools, steps, and relationships appear in the material. The prompt operates GPT in **construction mode**, which is the mode the author validates as reliable (see caveats below).
3. **Inspect the resulting graph.** Count nodes. Examine which parts of the source material have no nodes — that is the filler GPT dropped. In the author's reported experience with third-party courses, typical surviving node counts are **5–6** even for expensive, long courses (most_important.md §"Методология как граф") [author-claim].
4. **Decide.** A methodology that reduces to a handful of nodes is either genuinely compact (rare, and diagnosable on its own merits) or, more commonly in the author's reports, padded content with little semantic substance. A methodology that produces a dense, connected graph is one GPT can attach attention to — therefore one an agent can operate on.

The test is **deliberately unsophisticated**: you are not measuring something subtle, you are observing whether the construction act has anything to construct on. That lack of sophistication is the point — it is why the test can be run cheaply and at scale.

### Hard caveats on GPT reflection as evaluation

The litmus test depends on GPT's structural judgement, so the author's two conditions for reliable GPT reflection apply directly (most_important.md §"Методология как граф") [author-claim]:

1. **GPT must have created something via the prompt (graph), not merely observed it.** "Is this methodology good?" is observation and is not reliable. "Build a methodology graph from this material" is creation and is reliable.
2. **Trust the OUTPUT, not the explanation.** The nodes GPT kept, the edges GPT drew — those are the evidence. If you then ask GPT "why did you drop those sections?", the answer is post-hoc rationalisation, which can be a plausible hallucination. Do not rate on the rationalisation.

These are the same two caveats the author applies to GPT reflection elsewhere in the methodology — for example, when judging whether a belief-state log interpretation is trustworthy (cross-link T28, `../05-logging-ldd/belief-state-logging.md`) [author-claim]. They are not ad-hoc for this test.

### What counts as "graphable"

The sources do not give a formal definition. What they do commit to is the direction: graph-ability requires **correlations between concepts** that survive the model's attention process. If GPT constructs the graph and repeatedly finds no hook for a passage, that passage is not graphable — and therefore is entropy to the agent (most_important.md §"Методология как граф") [author-claim].

Practical implications that the sources support [author-claim]:

- **A methodology that only lists maxims** ("always think first, deliver value, iterate") lacks relations between nodes; it will produce a star or a flat list, not a graph, and the author's observation is that GPT will collapse it.
- **A methodology that defines steps and their inputs/outputs/prerequisites** produces edges natively; the construction-planning methodology is in this category because its steps have predecessor/successor relationships (that is exactly what a Microsoft Project schedule is — a DAG of tasks with dependencies).
- **A methodology that references external frameworks or tooling** can produce edges into those references, which count as correlations. The AI-development methodology graph inherits edges into concepts like Kilo Code modes, sparse attention, KV cache, etc., which is why it graphs densely (cross-link T11).

The boundary the sources do **not** draw: they do not define graphability strictly, they do not specify a minimum node count that qualifies as "a real methodology," and they do not formalise what counts as an edge. The author's framing is pragmatic and uses GPT itself as the oracle.

### How this generalization interacts with the rest of GRACE

Graphing a methodology is upstream of, not a replacement for, the rest of GRACE's machinery. The generalization says the **graph-first principle** holds outside code. The mechanics of what goes into the graph's leaves still vary by domain:

- For **code**, the leaves are contracts over functions and classes (cross-link T13, T15).
- For **documents**, the leaves are semantic squeezes over documents (cross-link T48).
- For **a work-planning methodology**, the sources do not specify the leaf format — only that the graph exists and produces an MS Project schedule. The author's wording in post 3839 (**"templates are standardized for code, but they can be adapted to other contexts"**) is the only explicit guidance the sources give about template evolution across domains [author-claim].

What transfers cleanly across all three:

- **PURPOSE-style fields** — a node-level "what is this for" is domain-agnostic and is the load-bearing field in code contracts per post 3889 (cross-link T15, T48) [research-backed] (grace_matches.md post 3889).
- **KEYWORDS / LINKS** — correlation hooks for retrieval are domain-agnostic (cross-link T16) [author-claim] (grace_matches.md post 3819).
- **Placement before the artefact** — anchor formation rules are about causal reading order, which is model-level (cross-link T15, post 3878) [research-backed] (grace_matches.md post 3878).
- **Wenyan compression** — compressed anchor carriers generalise to any complex text, not only code (cross-link T16) [research-backed] (grace_matches.md post 3892).

What does **not** transfer cleanly:

- **Type signatures (INPUTS/OUTPUTS in the Python sense)** do not map to planning steps or legal clauses; these are the parts of the template the author's "adaptation is required" comment most plausibly refers to [author-claim] (grace_matches.md post 3839).
- **Language-specific START-END tag placement** (cross-link T22) is a code-markup concern; the sources do not describe its analog in a construction-planning methodology or a document corpus.
- **FIM/SFT pretraining-distribution arguments** (cross-link T08) are about why AI trusts code over external docs; they do not extend to non-code artefacts in a straight line and the sources do not formalise how they apply to, say, a schedule.

## Evidence

### Primary source

- **`most_important.md` §"Методология как граф"** — the single governing passage for this file. Named content on:
  - The author's construction-work planning training (ПСД → Microsoft Project) and the methodology graph of construction-work planning he built for it.
  - The statement that an analogous methodology graph is built and used for AI-assisted development.
  - The load-bearing claim: **"if a methodology cannot be represented as a graph, then for the AI it is not a methodology — it is just text entropy."**
  - The litmus-test application to paid courses: typical 5–6 surviving nodes; GPT drops "water, platitudes, old material."
  - The reliability caveats on GPT reflection: must be creation-not-observation; trust output-not-explanation.
  - All [author-claim] except where noted otherwise.

### Supporting

- **Post 3839 (grace_matches.md, 2026-04-08)** — explicit statement that "GRACE is not only about code; it is a general methodology for an agent working with complex texts. Templates are standardized for code, but they can be adapted for other contexts." Anchors the generalization claim at framework level. [author-claim] Cross-link T48 (`./grace-for-rag-documents.md`) for the bank-RAG instance.
- **`most_important.md` §"Почему без семантического графа"** — transformer-level justification (4k-token boundary, sparse attention, sliding-window stitching through a semantic graph, random-attention fallback). Cross-link T06 (`../01-transformer-foundations/sparse-attention-and-kv.md`). Establishes that the graph-first principle is model-level, not code-level. [research-backed]
- **`most_important.md` §"Граф как основа навигации"** — KV-cache freezing of wrong branches if an explicit graph is not provided; "if you cannot build graphs in GPT, you do not know how to use GPT." Cross-link T10 (`../02-semantic-graph/graph-as-backbone.md`). [author-claim]
- **`most_important.md` §"Top-down логика"** and **§"IDE 2.0"** — the hierarchy (TZ → design → contracts → code) that the construction-planning hierarchy parallels. Cross-link T11 (`../02-semantic-graph/hierarchy-and-anchors.md`) and T39 (`../08-workflow-and-phases/top-down-pipeline.md`). [author-claim]
- **Post 3732 (grace_matches.md, 2026-03-26)** — hybrid Graph-RAG argument; why vector-only fails on exact facts. Extends the argument for graph-first retrieval into non-code domains. Cross-link T12 (`../02-semantic-graph/graph-rag-vs-vector-rag.md`). [author-claim]
- **Post 3878 (grace_matches.md, 2026-04-12)** — canonical GRACE contract fields (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS) and the placement rule (description before declaration). Supports the "what transfers" discussion. Cross-link T15 (`../03-contracts/contract-fields.md`). [research-backed]
- **Post 3889 (grace_matches.md, 2026-04-14)** — PURPOSE is load-bearing; short is better than long; English beats non-English; adjacency matters. Domain-agnostic benchmarks that back the "PURPOSE survives the port" claim. Cross-link T25 (`../04-markup-system/markup-compression.md`). [research-backed]
- **Post 3819 (grace_matches.md, 2026-04-05)** — wenyan-compressed KEYWORDS/LINKS as correlation carriers; 1:15 compression without semantic loss. Backs the "KEYWORDS/LINKS transfer" claim. Cross-link T16 (`../03-contracts/wenyan-prompting.md`). [author-claim]

### Claim-type summary

| Claim | Type | Source |
|---|---|---|
| Author has applied graph-first approach to construction-work planning (ПСД → MS Project schedule, via a methodology graph) | [author-claim] | most_important.md §"Методология как граф" |
| Methodology-as-graph test: graphable methodology is AI-operable; non-graphable is "text entropy" | [author-claim] | most_important.md §"Методология как граф" |
| Graphing a methodology exposes "water" because GPT drops filler nodes (no attention correlations → no graph nodes) | [author-claim / empirical] | most_important.md §"Методология как граф" |
| Paid courses typically reduce to 5–6 surviving graph nodes in the author's reported runs | [author-claim / empirical] | most_important.md §"Методология как граф" |
| GPT reflection reliable only when GPT created via prompt/graph (not mere observation) | [author-claim] | most_important.md §"Методология как граф" |
| Trust GPT's output artefact, not its post-hoc explanation | [author-claim] | most_important.md §"Методология как граф" |
| GRACE is a general methodology for complex text, not code-only; templates must be adapted to other contexts | [author-claim] | grace_matches.md post 3839 |
| Sparse-attention constraint past 4k tokens is model-level, not code-specific | [research-backed] | most_important.md §"Почему без семантического графа" |
| Hybrid Graph-RAG is required for exact facts in any domain | [author-claim] | grace_matches.md post 3732 |

### What the sources do not specify

- The sources do not specify the node list, edge list, or representation format of the construction-planning methodology graph. Only its existence and its role as upstream to the MS Project schedule.
- The sources do not name the third-party coaches whose courses the author tested with the litmus test. Only the general claim of "5–6 nodes" surviving.
- The sources do not specify a minimum node count that qualifies as a "real" methodology under the litmus test — no formal threshold.
- The sources do not give a formal definition of "graphable." The author's practical test uses GPT itself as the oracle.
- The sources do not describe further non-code domains beyond construction-work planning (for the methodology graph) and document-RAG (covered in T48). Adjacent applications are not enumerated.
- The sources do not quantify how often the litmus test's verdict would be wrong — there are no false-positive or false-negative statistics for methodology graphing.

## See also

- `./grace-for-rag-documents.md` — T48: the document-RAG companion; Evdokimov's banking case is the production instance of "GRACE for complex text." Together with this file, that file forms the "beyond-code" pair.
- `../02-semantic-graph/hierarchy-and-anchors.md` — T11: the code-level version of the same top-down hierarchy argument, including the methodology-as-graph litmus test and the GPT-reflection caveats.
- `../02-semantic-graph/graph-as-backbone.md` — T10: why graphs are the persistent scaffolding; what happens without them (KV-cache freezing wrong branches).
- `../02-semantic-graph/graph-rag-vs-vector-rag.md` — T12: hybrid-retrieval argument; why vector-only RAG fails on exact facts in any domain.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — T06: the 4k-token boundary and sliding-window mechanics that make the graph-first principle model-level rather than code-specific.
- `../00-foundations/core-principles.md` — T05: Principle 1 "Graph first, contracts second, code third" and Principle 7 "if the methodology cannot be expressed as a graph, it's noise" — where the generalization is stated as a first principle.
- `../00-foundations/what-is-grace.md` — T03: top-level positioning noting that GRACE generalizes beyond code.
- `../03-contracts/contract-fields.md` — T15: the contract fields (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS) that partly transfer to other domains.
- `../03-contracts/wenyan-prompting.md` — T16: compression discipline that survives the port to non-code material.
- `../05-logging-ldd/belief-state-logging.md` — T28: the other place the GPT-reflection caveats (create-not-observe; output-not-explanation) appear inside GRACE.
- `../08-workflow-and-phases/top-down-pipeline.md` — T39: the three-phase operational walk; the upstream pattern the construction-planning flow parallels.
- `../14-research-references/annotated-bibliography.md` — for the research citations indirectly referenced via posts 3878, 3889, 3819, 3732.
