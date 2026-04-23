# RATIONALE / AAG — Architectural Intent in Module Contracts

## What

`RATIONALE` and `AAG` are paired section names used in GRACE **module contracts** to expose **architectural intent** — the *why* behind how a module is structured — so that both humans and AI agents can see the reasoning that produced the current design rather than only the surface result [author-claim] (post 3890).

- `RATIONALE` is the section in a module contract that exposes the architectural reasoning behind the module's design [author-claim] (post 3890).
- `AAG` appears alongside `RATIONALE` in the same role (module-level architectural-intent section). The sources do not expand the acronym — post 3890 writes the pair as `RATIONALE\AAG` without explaining what the letters stand for. The sources do not specify [author-claim] (post 3890).
- Both sections live in **module contracts** — not in function contracts — because they describe architecture-scale intent that spans multiple functions and classes (post 3890).

These sections sit next to — and complement — references to named architectural patterns, which GRACE also uses for the same purpose: pointing the AI at the design reasoning so it can reconstruct broken logic from intent, not only from code [author-claim] (post 3890).

## Why

The motivation for `RATIONALE`/`AAG` sits on two legs: a research finding and an empirical observation the author made earlier on his own.

**The research finding (post 3890).** A paper on the impact of code comments for automated bug-fixing — cited in the sources as `researchgate.net/publication/400340807_On_the_Impact_of_Code_Comments_for_Automated_Bug-Fixing_An_Empirical_Study` — reports that when in-source documentation reveals the **architectural intent** of the code, automated bug-fix accuracy improves **up to 3×** [research-backed] (post 3890). The same paper's attention-analysis shows that the AI agent literally attends to the intent-description tokens when it reconstructs the logic of a broken method [research-backed] (post 3890). In other words, the tokens that carry *why* a piece of code exists are load-bearing for the agent's repair behaviour; they are not decoration.

**The empirical discovery (post 3890).** The author states that he had already found the same effect empirically well before the paper was published, which is why GRACE "strongly keeps architectural intent in focus" through both pattern references and `RATIONALE`/`AAG` sections in the module contract [author-claim] (post 3890). The paper validates a pattern the framework had already adopted; GRACE did not add these sections *because of* the paper — it added them because fix-quality was observably higher when intent was present.

**What breaks without it.** Contracts that describe *what* a module does but not *why* it is structured the way it is leave the agent to guess at architectural reasoning on every edit. When it guesses wrong — and sparse attention freezes that wrong branch in the KV cache — the agent will "logically" fix a symptom in a way that violates the design it cannot see. Intent-level attention targets are the mechanism by which the agent actually recovers the original logic during a fix (post 3890). Drop the targets and you drop the recovery mechanism.

The same post 3890 makes the complementary negative point: comments written in the spirit of "Captain Obvious" — restating what the code itself already says clearly — are **not what the agent needs**. The LLM reads code as a first-class language; it does not need a prose retelling of code it can already parse (post 3890; reinforced by post 3820 which states that for AI "Python or Rust is the same kind of language as English, so it reads them as easily as Shakespeare's sonnets"). What the agent lacks, and what `RATIONALE`/`AAG` is there to supply, is the **architectural reasoning that is not present in the code itself**.

This aligns with the broader GRACE stance on comment content: PURPOSE-level alignment (goal, intent, reasoning) is load-bearing; long narrative DESCRIPTION is mostly neutral or harmful (see `contract-fields.md` and post 3889) [research-backed] (post 3889). `RATIONALE`/`AAG` is the module-level analogue of PURPOSE — it carries architectural goals instead of function-level goals.

## How

**Placement.** `RATIONALE` and `AAG` are **module-contract sections**, not function-contract sections (post 3890). They describe decisions at architecture scope (patterns, layering, coupling choices, trade-offs made during design) rather than per-function pre/post conditions. Function contracts carry PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS instead (see `contract-fields.md` and the canonical example in post 3878).

**Partnered with pattern references.** GRACE keeps architectural intent visible in **two complementary ways** (post 3890):

1. **References to named architectural patterns** — concise names of the patterns the module implements or relies on. Names work as wenyan-style hooks: short, token-cheap, high-information, useful for semantic search and for cross-module correlation (cross-link wenyan prompting in `wenyan-prompting.md`).
2. **`RATIONALE`/`AAG` sections** — prose (kept short — see post 3892 on ShortenDoc compression) explaining the reasoning behind the architectural choices.

Both go **in the module contract**, at the place the agent reads when it opens the module. The point is to make architectural intent load into the attention window together with the code, not sit in a separate documentation store the agent will refuse to consult (see post 3820 on why external documentation fails against FIM-trained agents).

**What to put in it.** The sources describe the role abstractly — "architectural intent", "architectural reasoning" (post 3890) — and demonstrate by negative example ("Captain Obvious" is out). Concretely, based on what the cited paper measures (bug-fix reconstruction) and how the author frames it:

- Decisions that are *not* visible from the code alone: why this decomposition, why this boundary, why this pattern over a similar one, which invariants must hold across the module, what this module is deliberately *not* doing.
- Pattern names and short notes on why each pattern was chosen.
- Constraints that shape valid modifications (e.g., "this module must not know about X").

The sources do not specify an exact field schema for `RATIONALE`/`AAG` beyond the naming. The exact content template is not spelled out in the sources available; the constraint given is the role (architectural intent) and the bounds (short, English — see post 3889 on language and length).

**Keep it short.** Compression advice from `markup-compression.md` and `wenyan-prompting.md` applies. Long descriptions are "either neutral or cause degradation" (post 3889) [research-backed]; 40% compression without quality loss is demonstrated for docstrings (post 3892) [research-backed]. `RATIONALE`/`AAG` is not an architecture decision record essay — it is a semantic anchor for attention.

**Language.** English. Post 3889 shows cross-vendor degradation when in-source docs are written in non-English languages [research-backed] (post 3889). This warning is phrased bluntly in the source as a critique of "LLM-nationalists" pushing "sovereign AI" ideas straight into code (post 3889).

**Adjacency.** Keep the section next to the code it describes. Post 3889 proves that when an in-source description is separated from the code it documents by token distance — because documentation lives in a separate file, or is fetched through an MCP hop — the agent degrades [research-backed] (post 3889). `RATIONALE`/`AAG` earns its effect precisely because it sits inside the module contract *in the code file*, not beside it.

**What the agent actually does with it.** Per the attention analysis cited in post 3890, when the agent works on a broken method it **reads attention over the intent tokens** to reconstruct the correct logic. The mechanism is concrete: intent tokens become retrieval targets that the attention heads consult while producing the fix. This is why "up to 3×" is measured in bug-fix accuracy rather than in some softer metric — the section physically participates in the model's repair process (post 3890) [research-backed].

**What the agent does *without* it.** Without an architectural-intent section, the agent still produces a graph of attention correlations — GRACE's general claim that "if you don't build a graph, GPT builds one anyway, often wrong, and freezes it in the KV cache" applies here too [author-claim] (`most_important.md` §"Граф как основа навигации"). For `RATIONALE`/`AAG` specifically this means: the agent will invent its own architectural rationale from whatever tokens it has, get it wrong on non-obvious designs, and then defend the invented rationale across the KV cache freeze. The correct intent must be supplied as load-bearing tokens, not left to emerge.

### Example (structural sketch)

The sources do not provide a verbatim `RATIONALE`/`AAG` code example the way post 3878 provides a verbatim PURPOSE/INPUTS/OUTPUTS/KEYWORDS example for function contracts. Any concrete template below would be an invention on top of the sources. The sources do not specify the exact field structure [author-claim] (post 3890).

What the sources **do** fix structurally:

- lives in module contract (post 3890);
- carries architectural reasoning (post 3890);
- paired with named-pattern references (post 3890);
- short (by extension from posts 3889, 3892);
- English (by extension from post 3889);
- adjacent to the module code (by extension from post 3889).

### Anti-pattern: "Captain Obvious"

Post 3890 calls out the comment style where the documentation simply restates what the code already says ("Captain Obvious" comments) as **useless, and potentially harmful**, on the grounds that the LLM reads code fluently and gets no attention signal from a prose retelling (post 3890). This lands consistently with post 3820's "for AI, Python or Rust is the same kind of language as English" [author-claim] (post 3820) and post 3889's finding that long narrative descriptions are neutral-or-negative [research-backed] (post 3889).

Concretely, this rules out comments like:
- "This function returns the user." on a function named `get_user`.
- "Loops over items and adds them up." next to an obvious summation.
- A `RATIONALE` that restates the KEYWORDS instead of explaining *why* those patterns were chosen.

The mental test: if the line's content can be reproduced by reading the code for two seconds, it burns tokens and contributes nothing to attention. If the line states something that is **not** inferrable from the code (intent, pattern choice, trade-off, invariant), it is doing the `RATIONALE`/`AAG` job.

### Relation to other contract sections

- **PURPOSE** (function contract) is to a single function what `RATIONALE`/`AAG` (module contract) is to the module's architecture: a verbalized goal/intent statement the model can attend to (posts 3878, 3889, 3890).
- **KEYWORDS** (function or module) can reference architectural patterns by name; this is the short wenyan-style hook half of the "intent in focus" story (post 3890; post 3819 on wenyan prompting).
- **LINKS** provide cross-module references; a `RATIONALE` may name peer modules referenced via LINKS when the reasoning hinges on their existence (post 3878; cross-link `contract-fields.md`).
- Classical DbC pre/post/invariants are a different thing — they describe behaviour boundaries, not reasoning (see `ai-contracts-vs-dbc.md`). `RATIONALE`/`AAG` is intent-layer, not verification-layer.

### Notes on the `AAG` acronym

Post 3890 writes the pair as `RATIONALE\AAG` and does not expand `AAG`. No other source in the 6 GRACE-build files expands it either. The sources do not specify what `AAG` stands for. The agent should treat it as the section name used in GRACE module contracts for architectural-intent content, interchangeable with `RATIONALE` in that role, pending a future source that defines the expansion.

## Evidence

**Primary source — post 3890** (`grace_matches.md`):

> "Следующая важная работа о встроенной документации в код. Она говорит о том, что в документации в духе 'Капитан Очевидность' в реале LLM не нуждается, т.к. прекрасно читает код.
>
> Однако, если встроенная в код документация раскрывает архитектурный замысел, точность исправления багов улучшается до 3 (!) раз. Анализ внимания (attention) показал, что ИИ-агент опирается именно на токены описания замысла для восстановления логики сломанного метода. На самом деле я давно это обнаружил эмпирически, поэтому у меня GRACE сильно держит в фокусе раскрытие архитектурного замысла как через ссылки на архитектурные паттерны, так и через RATIONALE\\AAG секции в контракте самого модуля.
>
> https://www.researchgate.net/publication/400340807_On_the_Impact_of_Code_Comments_for_Automated_Bug-Fixing_An_Empirical_Study"

This single post contains every fact in this file's core claim:

- architectural-intent comments raise bug-fix accuracy **up to 3×** [research-backed];
- attention-analysis mechanism — the agent attends to intent tokens during method reconstruction [research-backed];
- the author's empirical discovery predates the paper [author-claim];
- GRACE uses **pattern references + `RATIONALE`/`AAG` sections in the module contract** [author-claim];
- "Captain Obvious" comments are useless [author-claim, reinforced by research-backed findings in post 3889];
- paper URL: `researchgate.net/publication/400340807`.

**Supporting sources:**

- Post 3889 (`grace_matches.md`) — PURPOSE criticality, shorter > longer, English > other, adjacency matters; **-95% degradation / +435% improvement** bench swings [research-backed]. Grounds the "keep it short, keep it English, keep it adjacent" guidance.
- Post 3892 (`grace_matches.md`) — ShortenDoc paper (`dl.acm.org/doi/10.1145/3735636`): 40% docstring compression without quality loss; ~10% HumanEval gain under compression [research-backed]. Supports aggressive brevity.
- Post 3878 (`grace_matches.md`) — canonical in-source-docs posture: PURPOSE is alignment, not exposition; Anthropic's Purpose Breakdown Structure; placement before declaration critical for causal-read anchor vector. Underpins the "intent as attention target" argument that `RATIONALE`/`AAG` extends to module scope.
- Post 3820 (`grace_matches.md`) — "for AI, Python or Rust is the same kind of language as English" [author-claim]; justifies why restating code is wasteful.
- `most_important.md` §"Граф как основа навигации" — "if you don't build a graph, GPT builds one anyway, often wrong, and freezes it in KV cache" [author-claim]; generalizes to "supply the intent or the model will invent a wrong one."

**Citation fidelity notes:**

- Paper URL is quoted verbatim from post 3890: `researchgate.net/publication/400340807_On_the_Impact_of_Code_Comments_for_Automated_Bug-Fixing_An_Empirical_Study`. The paper's authors, venue, and arXiv ID are not given in the source.
- The "up to 3×" figure is quoted verbatim from post 3890 ("точность исправления багов улучшается до 3 (!) раз"). The source does not give a lower bound or the benchmark used beyond the paper's own framing as an empirical study.
- The `AAG` acronym is not expanded anywhere in the six source files. Not inventing an expansion.

## See also

- `contract-fields.md` — canonical contract sections (PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS/RATIONALE/AAG); function-level analogue of this file.
- `contracts-overview.md` — why contracts are central (semantic shield for AI during modification).
- `ai-contracts-vs-dbc.md` — why AI-contracts are not classical Design-by-Contract; `RATIONALE`/`AAG` is intent-layer, not verification-layer.
- `wenyan-prompting.md` — compression technique behind the "pattern reference" half of the intent pair.
- `../04-markup-system/markup-compression.md` — empirical bounds on description length (post 3889: -95% to +435%; post 3892: 40% compression at no cost).
- `../04-markup-system/in-source-documentation.md` — enterprise lineage; PURPOSE over DESCRIPTION.
- `../02-semantic-graph/graph-as-backbone.md` — "if you don't build the graph, the model builds a wrong one and freezes it in KV cache"; structural reason intent must be supplied, not inferred.
- `../13-antipatterns/all-antipatterns.md` — "Captain Obvious" comments, long summary-style descriptions (both sourced from posts 3890, 3889).
- `../14-research-references/annotated-bibliography.md` — full entry for `researchgate.net/publication/400340807` (post 3890) and `arxiv.org/abs/2601.16661v1` (post 3889).
