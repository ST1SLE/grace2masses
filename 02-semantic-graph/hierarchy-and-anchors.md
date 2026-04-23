# Hierarchy and Anchors — Spec → Contract → Code

## What

GRACE organises every AI-generated application as a **top-down hierarchy of semantic artifacts**, not as a flat pile of source files. The hierarchy is (top to bottom) [author-claim]:

1. **TZ** — Russian "техническое задание", i.e. the specification / statement of work.
2. **Design documents** — derived from the TZ; architecture decisions, business-process modelling.
3. **Module contracts** — one per module; expand the design's intent at module scope.
4. **Function / method contracts** — local refinements of app-level intent at the call site.
5. **Code** — generated from the contracts, not from free creativity.

The artefacts are connected as a **semantic graph of anchor tokens** (most_important.md §"IDE 2.0"). Anchors are the token spans the transformer latches onto when navigating long contexts — concretely, the names, KEYWORDS, LINKS and PURPOSE fields that appear in the GRACE markup layer (post 3819; post 3878). Each level acts as the "table of contents" for the level below it, feeding sparse attention the skeleton it needs once the dense 4k-token window is exhausted (most_important.md §"Почему без семантического графа") [research-backed].

This file covers three things that ride on top of that hierarchy:

- how changes **propagate** through it top-down;
- the **IDE 2.0** vision that falls out of it;
- two diagnostic techniques enabled by it — the **methodology-as-graph litmus test** and **GPT reflection as evaluation**.

## Why

### The hierarchy replaces classical documentation

In the base Fill-In-the-Middle pretraining regime, documentation was almost absent as context; Supervised Fine-Tuning pairs were "task → code" with ~2–3 paragraphs of task and ~200–300 lines of code (most_important.md §"Top-down"; cross-link T08) [author-claim]. As a consequence the model has a strong pull toward reading code itself, and a weak disposition toward reading external documentation (most_important.md §"Top-down"). Once the **graph + module contract + function contracts** sit inside the code, "classical documentation becomes simply unnecessary — architecture and business model are already embedded in code in an AI-legible form" (most_important.md §"Top-down") [author-claim].

### The hierarchy prevents drift

GRACE explicitly **assists** GPT through the top-down pass, because left to itself "AI has a huge pull to jump straight to code" (most_important.md §"Top-down") [author-claim]. Walking down the hierarchy means each function contract is "a refinement of the general intent of the application at this specific site, not spontaneous AI creativity" (most_important.md §"Top-down") [author-claim]. The author claims this "sharply raises generation quality and almost entirely removes drift from the initial goals" (most_important.md §"Top-down") [author-claim].

### The hierarchy is the only thing sparse attention can navigate

Past 4k tokens, transformer attention switches to 100–200-token sliding windows; those windows are stitched together only through a semantic graph (most_important.md §"Почему без семантического графа"; cross-link T06) [research-backed]. Without an explicit top-level anchor structure, GPT falls back to random attention or, worse, fabricates an internal graph from a partial causal read and freezes it in the KV cache, becoming "stubborn as a donkey" about that wrong branch (most_important.md §"Граф как основа навигации") [author-claim / empirical]. The TZ-to-code hierarchy is what keeps the anchor graph both correct and persistent.

## How

### Top-down propagation of change

Changes always start at the highest affected level and cascade downward. The canonical example (most_important.md §"IDE 2.0") [author-claim]:

> If you change the TZ, the derived documents change, then the contracts for functionality change, then the code is regenerated there.

Practical consequences:

- **The AI, not the human, regenerates the lower layers.** The human edits the top-most artefact that actually changed; the tooling re-runs generation against the updated upstream artefact.
- **No edits at code level without first reconciling the contract.** The contract is the AI's "semantic shield" — if the code edit contradicts the contract, the agent should either update the contract first (propagating upward from a provably-local concern) or refuse the edit (cross-link T13, `../03-contracts/contracts-overview.md`).
- **Function contracts are NOT authored in isolation.** Phase 3 of the GRACE pipeline presupposes that the graph and module contract are already present in context when a function contract is written (most_important.md §"Top-down"; cross-link T39).

The AI has to be **assisted** through this pipeline. On its own it skips to code; GRACE enforces the three-phase walk (most_important.md §"Top-down"; cross-link T39) [author-claim]:

1. Phase 1 — architectural graph (classes/modules + business-process mapping).
2. Phase 2 — module contracts.
3. Phase 3 — function contracts + code.

### IDE 2.0 — the interface that exposes the hierarchy

Vlad prototyped an "IDE 2.0" in Claude from the author's Vision brief (most_important.md §"IDE 2.0") [author-claim]. The stated properties:

- **Core object is the semantic graph of anchor tokens**, organised by level of the hierarchy — not a filesystem tree, not a project explorer of `.py` files.
- **Top of the graph is the TZ; bottom is the function/class contract** that generates the code.
- **Code is generated, not edited directly**, from the contract at the bottom of the branch.
- **A regular builder — a non-programmer — can operate in such an IDE**, because they never need to step below the contract layer. The contract is "just a text order for functions and classes" (most_important.md §"IDE 2.0") [author-claim].
- **Minimum literacy required** to program in IDE 2.0: the concepts **class / function / variables** (most_important.md §"IDE 2.0") [author-claim]. Everything else is handled by navigating the hierarchy.

The author predicts (most_important.md §"IDE 2.0") [author-claim]:

> It is just a question of time before some start-up bursts into the market with this and feasts on the wreckage of old IDEs — simply because even pure code-generation quality from GPT will be an order of magnitude higher when you speak with it in its own language of anchor graphs.

Two implications:

- **Cursor and PyCharm are not IDE 2.0.** They still put code files at the centre and treat contracts as decoration (most_important.md §"IDE 2.0") [author-claim].
- **Kilo Code's Architect / Code / Debug modes** are the closest currently-shipping UI approximation of the hierarchy's phases (cross-link T40, `../08-workflow-and-phases/architect-code-debug-modes.md`) (practical_applications.md §"GRACE + PCAM") [author-claim].

### Methodology-as-graph — litmus test for substance

The hierarchy argument generalises beyond code (most_important.md §"Методология как граф") [author-claim]. The author has built methodology graphs for two different domains: AI-assisted development (his own GRACE), and planning of construction works from the design estimate to a Microsoft Project schedule (most_important.md §"Методология как граф") [author-claim].

The load-bearing claim (most_important.md §"Методология как граф") [author-claim]:

> A very important point with the arrival of AI: if a methodology cannot be represented as a graph, then for the AI it is not a methodology — it is just text entropy.

The test works in both directions:

- **For your own methodology.** If the discipline you are trying to operationalise for agents cannot be rendered as a connected graph of nodes, you are loading noise into the model, not a method. Cross-link T49 (`../10-beyond-code/general-complex-text.md`).
- **For other people's methodologies — "are they running a con?"** (most_important.md §"Методология как граф") [author-claim]. The author claims that when he graphs the content of expensive courses by a third-party coach, "often from a course costing a pile of money and time, only 5–6 nodes remain in the graph" — GPT, asked to synthesise a methodology graph, simply marks the rest "water, platitudes and old stuff" with "no semantic value" and drops it from the methodologist's graph.

That filtering is the mechanism: **GPT does not include filler in graphs**, because filler produces no attention correlations. So graphing a body of teaching is a cheap, automatic filter for empty content [author-claim].

### GPT reflection as an evaluation tool — with hard caveats

The methodology-as-graph test depends on trusting GPT's judgement of semantic value. The author is explicit that GPT reflection is "extremely reliable as an evaluation of prompting effectiveness" **only under two conditions** (most_important.md §"Методология как граф") [author-claim]:

1. **GPT must have created something via the prompt (graph), not merely observed it.** In other words, you do not ask GPT "is this methodology good?" — you ask it to turn the methodology into a graph. The reflection rides on the act of construction, not on commentary.
2. **Trust the OUTPUT, not the explanation.** The reliable signal is the artefact GPT produced (which nodes it kept, which it dropped), not the prose it generates if you then ask it why. Post-hoc explanations from GPT can be plausible-sounding hallucinations; the structure of what it built cannot.

Both caveats also apply to the "is this code correct?" case where GPT inspects a diff or a trajectory: the act must be creation (e.g. writing a fix, or building a belief-state graph against logs) and the rated artefact must be the fix itself, not the commentary (cross-link T28, `../05-logging-ldd/belief-state-logging.md`).

## Evidence

### Sources cited in this file

- `most_important.md` §"Top-down логика и переход к контрактам" — three-phase top-down pipeline; why AI must be assisted; zero-doc consequence; FIM/docs argument.
- `most_important.md` §"IDE 2.0 и иерархия: ТЗ → контракты → код" — Vlad's prototype; semantic-graph-of-anchor-tokens core; non-programmer at contract level; minimum literacy (class/function/variables); prediction about old IDEs.
- `most_important.md` §"Методология как граф, а не текстовая каша" — MS Project methodology graph; litmus test for coach content; GPT drops filler nodes; two caveats on reflection (create-not-observe; output-not-explanation).

### Supporting cross-references

- Anchor-graph mechanics and sparse attention: see T06 (`../01-transformer-foundations/sparse-attention-and-kv.md`).
- KEYWORDS / LINKS as concrete anchor-token carriers inside contracts: post 3819 (wenyan prompting; cross-link T16).
- PURPOSE / INPUTS / OUTPUTS placement that makes the function-level anchors load-bearing: post 3878 (cross-link T15).
- Explicit graph prevents KV-cache lock-in on wrong branches: most_important.md §"Граф как основа навигации" (cross-link T10).

### Claim-type summary

| Claim | Type | Source |
|---|---|---|
| Hierarchy TZ → design → module contract → function contract → code | [author-claim] | most_important.md §"Top-down", §"IDE 2.0" |
| Top-down edit propagation (change TZ → regenerate code at the leaf) | [author-claim] | most_important.md §"IDE 2.0" |
| AI must be assisted through the three phases | [author-claim] | most_important.md §"Top-down" |
| Graph + contracts replace classical documentation | [author-claim] | most_important.md §"Top-down" |
| FIM pretraining barely used docs → AI pulls toward code | [author-claim] | most_important.md §"Top-down" (cross-link T08) |
| IDE 2.0 centres on semantic graph of anchor tokens | [author-claim] | most_important.md §"IDE 2.0" |
| Non-programmer can operate at contract level (class/function/variables only) | [author-claim] | most_important.md §"IDE 2.0" |
| Methodology-as-graph litmus test | [author-claim] | most_important.md §"Методология как граф" |
| "If a methodology cannot be graphed, for AI it is text entropy" | [author-claim] | most_important.md §"Методология как граф" |
| GPT drops filler nodes; useful for detecting "water" in paid courses | [author-claim / empirical] | most_important.md §"Методология как граф" |
| GPT reflection reliable only when GPT created via prompt/graph | [author-claim] | most_important.md §"Методология как граф" |
| Trust GPT's output artefact, not its post-hoc explanation | [author-claim] | most_important.md §"Методология как граф" |
| Sparse attention past 4k tokens needs anchor graph | [research-backed] | most_important.md §"Почему без семантического графа" (cross-link T06) |

### What the sources do not specify

- The sources do not specify a concrete IDE 2.0 product name, URL, or ship date — only that Vlad built a Claude prototype from the author's Vision brief.
- The sources do not specify quantitative metrics for drift reduction under the three-phase pipeline — only the author's qualitative claim of "sharply raises generation quality and almost entirely removes drift".
- The sources do not specify the exact graph format used for the MS Project construction-planning methodology — only that the author built one.
- The sources do not specify which third-party courses were tested with the litmus test — only the general claim that "5–6 nodes remain" from paid courses.

## See also

- `./graph-as-backbone.md` — T10: why graphs are the persistent scaffolding at all and what happens without them.
- `./graph-rag-vs-vector-rag.md` — T12: hybrid retrieval on top of the anchor graph.
- `../00-foundations/core-principles.md` — T05: Principle 1 "Graph first, contracts second, code third" and Principle 7 "if the methodology cannot be expressed as a graph, it's noise".
- `../01-transformer-foundations/sparse-attention-and-kv.md` — T06: the mechanical reason the hierarchy's anchors are load-bearing past 4k tokens.
- `../03-contracts/contracts-overview.md` — T13: contracts as the "semantic shield" at the bottom of the hierarchy.
- `../03-contracts/contract-fields.md` — T15: PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS fields that materialise the anchor tokens inside code.
- `../03-contracts/wenyan-prompting.md` — T16: KEYWORDS and LINKS as compressed anchor carriers.
- `../08-workflow-and-phases/top-down-pipeline.md` — T39: the three-phase operational walk.
- `../08-workflow-and-phases/architect-code-debug-modes.md` — T40: Kilo Code's UI mapping of the hierarchy's phases.
- `../10-beyond-code/general-complex-text.md` — T49: the methodology-as-graph argument generalised outside code.
