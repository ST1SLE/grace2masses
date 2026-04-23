# Legacy Contract Recovery

## What

Legacy contract recovery is the practice of adding GRACE-style contracts (PURPOSE, INPUTS, OUTPUTS, critical invariants, etc.) to old code bases that were written by humans without any AI-oriented documentation. The goal is to give an AI agent enough semantic scaffolding to modify that code safely, without the agent acting "like an elephant in a china shop" during refactoring [empirical] (`contracts_as_anchor.md` §"Научная опора на восстановление контрактов в legacy").

The task has two forks:

1. Generate contracts automatically from the existing implementation (research-backed by the CodeT5+ paper below).
2. Expose only those recovered contracts to downstream AI agents — not the original human code — so the agent does not learn from "ugly" legacy patterns [empirical] (post 3622).

## Why

Contracts are the semantic shield an AI relies on when modifying code [author-claim] (`contracts_as_anchor.md` §"Почему контрактное программирование"). For brand-new, 100%-AI-generated code, the contract is written by the AI-author at generation time and lives next to the code from day one. For legacy code, no such contracts exist — yet the same AI must now modify it. Without recovered contracts, the agent has no semantic anchors and reverts to random attention past 4k tokens (cross-link `../01-transformer-foundations/sparse-attention-and-kv.md`).

There is a second, subtler reason specific to legacy: human-written code frequently contains patterns the author labels "рука-лицо" (face-palm) [author-claim] (post 3622). If that raw code is fed back to the agent during modification, the agent will treat it as a few-shot example and propagate the bad patterns and rudiments. Recovering a clean contract layer and hiding the raw implementation behind it breaks that feedback loop [author-claim] (post 3622).

The legacy recovery problem sits at the intersection of:

- classical Design-by-Contract's desire to retrofit pre/post/invariants onto old code, and
- GRACE's modern need for AI-readable semantic anchors inside every module.

Note however that AI contracts are **not** classical DbC (cross-link `./ai-contracts-vs-dbc.md`) — the recovered form is closer to a compressed docstring / micro-spec than to formal Hoare triples.

## How

### 1. Automated recovery via CodeT5+ (research-backed)

A cited research result uses a fine-tuned **Google CodeT5+** network to recover contracts from existing Java code, written in the **JML (Java Modeling Language)** formalism [research-backed] (`contracts_as_anchor.md` §"Научная опора на восстановление контрактов в legacy"; dl.acm.org/doi/10.1145/3689484.3690738).

- JML is a formal specification language for annotating Java code in the Design-by-Contract paradigm [research-backed] (`contracts_as_anchor.md` §"Научная опора").
- The fine-tuned CodeT5+ reconstructs contracts at **90–97% quality**, depending on the class of test used to evaluate [research-backed] (`contracts_as_anchor.md` §"Научная опора"; dl.acm.org/doi/10.1145/3689484.3690738).
- The recovered contracts focus on **critical input/output conditions and invariants** rather than the full richness of a hand-authored spec [research-backed] (`contracts_as_anchor.md` §"Научная опора").
- The author notes these contracts are "довольно примитивные" (rather primitive) compared to what modern GRACE uses for fresh AI-generated code, but that recovering critical I/O conditions and invariants is already significant and almost eliminates the chance that an AI will destroy the code during edits [author-claim] (`contracts_as_anchor.md` §"Научная опора").

### 2. Empirical recovery by the author (LLM-reviewer evaluation)

The author reports positive results applying the same idea to his own legacy-code workflow, but with a crucial methodological difference [empirical] (`contracts_as_anchor.md` §"Научная опора"):

- Quality was evaluated by **another LLM acting as reviewer**, not by running formal JML tests.
- The author did not publish numeric accuracy figures for his own runs; the 90–97% figure belongs to the CodeT5+ paper, not to the author's own measurements [research-backed / author-claim distinction] (`contracts_as_anchor.md` §"Научная опора").
- Despite the less rigorous evaluation, the author's takeaway is that contract injection into legacy code is "довольно позитивный" (quite positive) in practice — "we are close to total adoption of contract programming" [author-claim] (`contracts_as_anchor.md` §"Научная опора").

### 3. Anton's contract-only MCP pattern (specific to legacy)

When contracts have been injected into legacy code, a colleague (Anton) built an MCP that exploits this layer [empirical] (post 3622):

- Instead of the native `read_file` tool, a custom `read_contracts` tool returns:
  - the identifiers of the contracts in a file,
  - the text of each contract,
  - the line ranges of the implementation each contract covers.
- A companion tool such as `get_contracts_by_contract_path` walks a graph of related contracts via `PATH`-header links, if present.
- Result: the LLM never has to "guess where the implementation of this piece of code lives" [empirical] (post 3622).
- Context is isolated to only the semantically-relevant contracts; tokens saved by hiding implementation compensate for the tokens spent on contracts [empirical] (post 3622).

This MCP works because GRACE's **START-END tag markup** makes parsing possible without a full AST — a simple algorithm can scan paired tags directly [empirical] (post 3622; cross-link `../04-markup-system/start-end-tags.md`).

#### Why this pattern is good for legacy and bad for AI-native code

The author is explicit that contract-only exposure is a legacy-specific tactic, not a universal one [author-claim] (post 3622):

- **For legacy code**: human code is often ugly; exposing it seeds bad patterns into the agent's few-shots. Expose contracts only.
- **For AI-native code**: do the opposite. Use **greedy reading** — pull the whole module (2–4 modules is usually fine, within a ~20k-token budget) [empirical] (post 3622). AI-written modules already act as few-shots that tune the agent to the current framework versions. AI tends to keep modules small — typically under 1000 lines — so the greedy reading cost is bounded [empirical] (post 3622).

This distinction — contract-only for legacy, greedy reading for AI-native — is the critical design decision when choosing a modification strategy.

### 4. Recommended recovery workflow (synthesized)

The sources do not specify a step-by-step recipe, but combining the points above:

1. Decide on a contract format appropriate to the target language and LLM (cross-link `./contract-fields.md`, `./language-specific/python.md`, `./language-specific/csharp.md`).
2. Either fine-tune / use an existing model in the CodeT5+-style pipeline, or use a general-purpose LLM to draft contracts from the implementation [research-backed for CodeT5+; author-claim for general LLM].
3. Review draft contracts with a second LLM reviewer and/or humans before committing [empirical] (`contracts_as_anchor.md` §"Научная опора").
4. Wire downstream agents to **read contracts first, implementation only when strictly necessary** — e.g., via Anton's contract-only MCP [empirical] (post 3622).
5. Keep START-END tags around every contract/implementation block so simple tooling can parse them without an AST [empirical] (post 3622).

## Evidence

- **CodeT5+ / JML contract recovery** — dl.acm.org/doi/10.1145/3689484.3690738 — 90-97% contract recovery quality on legacy Java, focused on critical I/O conditions and invariants [research-backed] (`contracts_as_anchor.md` §"Научная опора").
- **Author's empirical legacy experience** — evaluated via an LLM reviewer rather than formal tests; results "quite positive" [author-claim / empirical] (`contracts_as_anchor.md` §"Научная опора").
- **Anton's contract-only MCP pattern** — working example of leveraging recovered contracts on legacy code via START-END tag parsing; custom `read_contracts` + `get_contracts_by_contract_path` tools [empirical] (post 3622).
- **Legacy vs AI-native modification strategies** — contract-only for legacy, greedy reading for AI-native [author-claim / empirical] (post 3622).
- **START-END tags enable AST-free parsing** — foundation for any recovery tooling [empirical] (post 3622; cross-link `../04-markup-system/start-end-tags.md`).

## See also

- `./contracts-overview.md` — why contracts are the semantic shield in GRACE.
- `./ai-contracts-vs-dbc.md` — why recovered contracts should not imitate classical Hoare-logic DbC.
- `./contract-fields.md` — the PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS fields a recovered contract should carry.
- `./language-specific/python.md` — placement rules for Python legacy recovery.
- `./language-specific/csharp.md` — C# XML Documentation as a recovery target.
- `../04-markup-system/start-end-tags.md` — the paired-tag mechanism that makes contract-only MCPs possible.
- `../04-markup-system/xml-like-markup.md` — broader markup rationale.
- `../09-tooling/mcp-scepticism.md` — why most custom MCPs fail but Anton's contract-only MCP is a working exception.
- `../14-research-references/annotated-bibliography.md` — full bibliographic entry for the CodeT5+ paper.
