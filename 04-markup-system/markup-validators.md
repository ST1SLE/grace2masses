# Markup Validators

## What

A **markup validator** in the GRACE context is a deterministic, algorithmic tool that parses source files and checks whether their GRACE-style in-source markup (contracts, START-END tags, PURPOSE/INPUTS/OUTPUTS/KEYWORDS fields, etc.) is present, well-formed, and complete — without invoking an LLM.

The canonical example is a validator built by a colleague of the GRACE author: **~1650 lines of Python**, constructed by using GRACE itself to guide the agents that wrote it, and used to verify that other agents do not drop important documentation elements when generating or modifying code `[empirical]` (post 3904).

## Why

Why build an algorithmic validator at all when an LLM reviewer could be asked to do the same thing?

- **Speed and cost.** An algorithmic validator is effectively instantaneous and consumes **no tokens** — unlike an LLM review call `[author-claim]` (post 3904).
- **Determinism.** Syntactic checks always return the same verdict on the same input, so they can be wired into a CI-style autotest suite without flake `[author-claim]` (post 3904).
- **Agent discipline.** The point of embedding documentation *in code* is that agents must not skip or truncate the markup; a mechanical validator enforces that floor `[author-claim]` (post 3904).
- **Autonomous-agent safety.** Autonomous/swarm agents can happily emit code that "looks right" but silently drops PURPOSE or LINKS sections. Algorithmic gates catch that at test time rather than at review time `[empirical]` (post 3904).

Why GRACE makes this feasible at all:

- GRACE's markup uses paired **START-END tags** in comments, giving it **Dyck-Language compatibility** — which means the grammar is simpler than a full AST and can be parsed in a single pass by ordinary algorithms `[research-backed]` (post 3948). The same property is what makes Anton's contract-only MCP work without a compiler (post 3622).
- Because of this, writing a validator is a tractable engineering task, not a research project — the author notes that the colleague's parsing algorithms "are logically non-trivial" but still doable in ~1650 Python lines `[empirical]` (post 3904).

Why it is not a replacement for LLM review:

- An algorithmic validator **cannot catch semantic errors** — e.g., a PURPOSE field that is syntactically valid but lies about what the function does, or KEYWORDS that are present but wrong `[author-claim]` (post 3904).
- The author's *default* remains **LLM review run concurrently with code review**; the algorithmic validator is a complement, not a substitute `[author-claim]` (post 3904).

## How

### The reported implementation (post 3904)

What the sources state about the colleague's validator:

- **Author/provenance**: built by a colleague from the GRACE author's training cohort `[empirical]` (post 3904).
- **Bootstrapping**: the validator itself was "vibe-coded" using GRACE — i.e., GRACE was used to manage the agents that wrote GRACE's own validator `[empirical]` (post 3904).
- **Stated purpose**: check that agents do not omit important documentation elements from their output `[empirical]` (post 3904).
- **Size**: on the order of **1650 Python lines** `[empirical]` (post 3904).
- **Complexity**: syntactic-parsing algorithms described as "logically non-trivial" (although simpler than a full AST-based parser, because paired tags + Dyck-compatibility make single-pass parsing viable) `[author-claim]` (post 3904).
- **Integration**: the colleague added the module to the project's **autotest suite** — markup validation runs alongside ordinary tests `[empirical]` (post 3904).
- **Working status**: "it works" `[empirical]` (post 3904).

The sources do not specify: the validator's name, repository URL, license, exact rule set, tag taxonomy, error formats, or whether it is public. `"the sources do not specify."`

### Why single-pass parsing is enough

The markup system that a validator has to read is designed to be Dyck-compatible:

- Paired START/END tags around logical blocks in code or documentation give **Dyck-3-style bracket structure** `[research-backed]` (post 3948). Example pattern shown in post 3948: `( [ [ ] { } ] ( ) { ( ) } ) [ ]`.
- That structure is natively parseable by TC⁰ algorithms, i.e. by ordinary finite-state / stack-based code without chain-of-thought and without a compiler front-end `[research-backed]` (post 3948, referencing arxiv.org/abs/2105.11115).
- Anton independently exploited the same property to build a **contract-only MCP** that reads "only contracts and function declarations" out of legacy source files without parsing the AST — `read_contracts` returns contract IDs + text + implementation line ranges, and `get_contracts_by_contract_path` walks PATH-header graphs `[empirical]` (post 3622).
- A validator is the dual of that MCP: instead of *returning* the bracketed structure, it *checks* that the bracketed structure is present and well-formed.

See `./start-end-tags.md` for the markup conventions the validator is checking, and `../01-transformer-foundations/tc0-and-dyck-language.md` for the formal basis.

### What an algorithmic validator can check (inferable from sources)

The sources describe GRACE markup well enough to indicate what a syntactic validator can be expected to enforce:

- Paired START/END tag balance around annotated blocks `[research-backed]` (post 3948).
- Presence of required fields per contract, e.g. **PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS** `[empirical]` (post 3878 — Python contract example; see `../03-contracts/contract-fields.md`).
- Placement rule: contract block appears **before** the function/method declaration (GRACE's placement rule, critical for the anchor vector at the function-name token) `[author-claim]` (post 3878).
- Presence of **RATIONALE / AAG** sections in module-level contracts `[author-claim]` (post 3890 — intent-level annotations; see `../03-contracts/rationale-and-aag.md`).
- Basic well-formedness of KEYWORDS/LINKS syntax (e.g., the `[Keyword1, Keyword2]` list form shown in post 3878).

The sources do not enumerate the colleague's exact rule set; the items above are what a validator *can* check given the public markup conventions. `"the sources do not specify."`

### What an algorithmic validator cannot check

The following failure modes require an LLM reviewer (or a human):

- Whether PURPOSE **accurately describes** what the code actually does (semantic, not syntactic).
- Whether KEYWORDS are the **right** architectural-pattern labels for the block (wenyan-style compression only works if the hooks are meaningful — see `../03-contracts/wenyan-prompting.md`).
- Whether LINKS point to the **actually-related** modules rather than plausible-looking names.
- Whether RATIONALE reflects the **real** architectural reasoning vs. "Captain Obvious" restatement of the code `[author-claim]` (post 3890).
- Whether compression is **too aggressive** relative to the benchmarked sweet spot (~40% with no quality loss per ShortenDoc) `[research-backed]` (post 3892, dl.acm.org/doi/10.1145/3735636).

This is exactly why the author keeps LLM review in the pipeline (post 3904).

### Recommended usage pattern

The two mechanisms are complementary, not alternative:

| Check | Algorithmic validator | LLM review |
|---|---|---|
| Markup present at all? | yes | redundant |
| Tags balanced? | yes | redundant |
| Required fields populated? | yes | redundant |
| PURPOSE truthful? | no | yes |
| KEYWORDS semantically right? | no | yes |
| Architectural intent preserved? | no | yes |
| Cost per run | ~zero (tokens), ~zero (time) | real tokens, real latency |
| Fits in autotests? | yes | possible but expensive |

- The colleague's default: **include the validator module in autotests** `[empirical]` (post 3904).
- The author's default: **run LLM review concurrently with code review** `[author-claim]` (post 3904).
- Combined posture: "both approaches have their place" `[author-claim]` (post 3904). For projects that embed documentation in code at GRACE scale, the author notes this combined posture is "not a bad practice" (post 3904).

### Building your own validator — what the sources imply

The sources do not give a step-by-step build guide, but they constrain the design space:

- Use the language the rest of the agent stack is in — in the reported case, Python. `"the sources do not specify"` why Python specifically; the reported validator is 1650 Python lines (post 3904).
- Rely on the START/END tag structure rather than the host language's AST — that is the whole point of Dyck-compatible markup (post 3948).
- Start with the minimal rule set and expand as real agent-output failure modes appear — the colleague's validator grew from real agent-supervision needs, not from a spec `[empirical]` (post 3904).
- Keep it deterministic and keep it in CI. It exists to **catch dropped markup** before a semantic reviewer spends tokens on it (post 3904).

### Relation to other GRACE mechanisms

- Validators complement the **self-correction loop** (see `../06-testing/tests-in-self-correction-loop.md`): they run as autotests, so their verdicts feed the same debug feedback channel as ordinary test failures.
- Validators are one example of the broader pattern in GRACE: **tooling built on START-END markup** rather than on AST. Anton's contract-only MCP (post 3622) is another; both exploit Dyck-compatibility to avoid writing a compiler (see `./start-end-tags.md`).
- The argument that custom MCPs are usually a bad idea (see `../09-tooling/mcp-scepticism.md`) does **not** apply here — a validator is a local CI tool, not an LLM-facing MCP. It is never exposed to the agent's tool surface, so it cannot be silently ignored.
- The author notes that embedding documentation in code works only if the markup discipline is actually upheld; an algorithmic validator is the mechanical component of that discipline (post 3904), while LLM review and the in-source doc standards (post 3878; see `./in-source-documentation.md`) provide the semantic component.

## Evidence

- **Primary source — post 3904** (`grace_matches.md`): full description of the colleague's GRACE-markup validator. Specific facts cited: built via GRACE (self-hosted), ~1650 Python lines, non-trivial syntactic-parsing algorithms, included in autotests, functioning. Trade-off with LLM review: instant + token-free + deterministic on one side, cannot catch semantic errors on the other. Author's default: LLM review in parallel with code review. Author's verdict: for in-code-documentation workflows, the combined approach is "not a bad practice."
- **Post 3948** (`grace_matches.md`): formal basis for why algorithmic parsing of GRACE markup is feasible — TC⁰ / Dyck Language; paired START-END tags in Python comments make markup parseable in one pass. References arxiv.org/abs/2105.11115 (Shunyu Yao et al. on native Dyck Language support in transformers).
- **Post 3622** (`grace_matches.md`): Anton's contract-only MCP — independent confirmation that START-END markup enables AST-free tooling (`read_contracts`, `get_contracts_by_contract_path`). Same structural property a validator relies on.
- **Post 3878** (`grace_matches.md`): contract field definitions (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS) and placement rule (before declaration); the surface a validator checks.
- **Post 3890** (`grace_matches.md`): RATIONALE / AAG in module contracts and the "Captain Obvious" anti-pattern — a concrete semantic failure class that the LLM reviewer, not the algorithmic validator, has to catch.
- **Post 3892** (`grace_matches.md`): ShortenDoc paper (dl.acm.org/doi/10.1145/3735636) on 40% compression without quality loss — informs the semantic side of review.

The sources do not provide a URL, repository, or name for the specific validator described in post 3904. `"the sources do not specify."`

## See also

- `./start-end-tags.md` — the paired-tag convention the validator relies on.
- `./xml-like-markup.md` — the broader markup system being validated.
- `./in-source-documentation.md` — why GRACE puts docs in code in the first place.
- `./markup-compression.md` — when aggressive compression is safe vs. harmful; an area only LLM review can check.
- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal reason single-pass parsing works.
- `../03-contracts/contract-fields.md` — the field set a validator typically checks.
- `../03-contracts/rationale-and-aag.md` — semantic content the validator cannot judge.
- `../03-contracts/wenyan-prompting.md` — KEYWORDS/LINKS compression whose correctness is only partly syntactic.
- `../06-testing/tests-in-self-correction-loop.md` — how validator output joins the broader test feedback channel.
- `../09-tooling/mcp-scepticism.md` — why custom MCPs usually fail but local CI validators do not.
- `../11-case-studies/grace-validator-self-hosted.md` — the same post 3904 case study viewed as a tooling-maturity signal.
