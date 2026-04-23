# Case Study — GRACE Validator Built Using GRACE (Self-Hosted)

> Companion file: `../04-markup-system/markup-validators.md` is the general topic page on GRACE markup validators. This file is the **case study** viewpoint of post 3904 — what it means that the tool was bootstrapped on itself, and what that signals about GRACE's maturity.

## What

A colleague from the GRACE author's training cohort built a **validator for GRACE-style markup**, and did so by **using GRACE to manage the agents that wrote the validator** `[empirical]` (post 3904). The tool is a ~1650-line Python program whose job is to algorithmically check whether AI agents, when they produce code, are keeping the GRACE in-source documentation elements intact rather than silently dropping them `[empirical]` (post 3904).

The colleague added the validator module to the project's autotest suite, so it runs as part of ordinary CI alongside regular tests `[empirical]` (post 3904).

The case is notable for three reasons, each a strand of the same story:

1. **Self-hosted bootstrap.** GRACE was used to build a GRACE tool. The methodology proved capable of directing the agents that built a piece of infrastructure *for* the methodology.
2. **Algorithmic, not LLM.** The validator performs deterministic, token-free, instant syntactic checks — no LLM call in the hot path.
3. **Complement, not substitute.** The author's own default is to run an LLM review concurrently with code review; the colleague's validator does not replace that, it runs alongside as a mechanical floor `[author-claim]` (post 3904).

## Why

### Why this case is load-bearing for GRACE

Most GRACE case studies (e.g. Kirill's 132k-LOC migration, Alexey's 300k-LOC project, Vlad's CRM, Mikhail's bank RAG) are about *applying* GRACE to a target problem. Post 3904 is different: it is a case of **GRACE building a piece of GRACE's own tooling** `[empirical]` (post 3904). That is a methodology-maturity signal — if the methodology cannot be used to build its own infrastructure cleanly, it is not ready for general use.

### Why an algorithmic validator is necessary at all

The embedded-in-code documentation that GRACE relies on (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS, RATIONALE / AAG) only works if agents actually preserve it across edits. If an autonomous or swarm agent quietly drops a PURPOSE field during a refactor, the semantic scaffold is silently degraded and later agents begin to drift `[empirical]` (post 3904; context on anti-loop protection: post 3695).

An LLM reviewer can catch this, but at a real token and latency cost. An algorithmic validator catches the **mechanical floor** — "did the markup survive?" — instantaneously, with no tokens, deterministically `[author-claim]` (post 3904).

### Why it is feasible at all — the Dyck-Language property

Building a parser is usually a project. Building a GRACE-markup parser is not — the markup system is deliberately **Dyck-Language-compatible** via paired START-END tags in comments, so it can be walked in a single pass without an AST `[research-backed]` (post 3948, referencing arxiv.org/abs/2105.11115 for native Dyck-Language support in transformers; applied-parsing note: post 3622). The same structural property that lets Anton's MCP read contracts out of legacy code without a compiler (post 3622) is what lets the colleague's validator check them without one.

Put differently: the validator is only 1650 Python lines because GRACE's markup is 1650-Python-lines-cheap to parse. If the markup were YAML — not Dyck-compatible — the same validator would either be much larger or would have to lean on a third-party parser `[research-backed]` (post 3948).

### Why it is complementary to LLM review, not a replacement

An algorithmic validator can confirm that a PURPOSE field *exists* and is *well-formed*. It cannot confirm that a PURPOSE field *truthfully describes the function*. That judgment is semantic and requires an LLM reviewer (or a human) `[author-claim]` (post 3904). This is why the author keeps LLM review in the loop even after the algorithmic validator is in place `[author-claim]` (post 3904).

## How

### The setup, as reported

The sources describe the case with the following specifics:

- **Who**: a colleague from the GRACE author's training cohort `[empirical]` (post 3904).
- **What was built**: a validator for GRACE markup — i.e., a tool that parses source files and checks whether the GRACE in-source documentation elements are present and well-formed `[empirical]` (post 3904).
- **How it was built**: by "vibe-coding" through GRACE — GRACE was used to manage the agents that wrote the validator `[empirical]` (post 3904). The author's phrasing: `"через GRACE написал валидатор разметки самого GRACE"` — "via GRACE wrote a validator of GRACE's own markup."
- **Target of validation**: other agents' output. The purpose is to check that agents "do not miss important documentation elements" in the code they emit `[empirical]` (post 3904).
- **Size**: on the order of **1650 lines of Python** `[empirical]` (post 3904).
- **Complexity**: the author notes the syntactic-parsing algorithms are "logically non-trivial" — although simpler than a full AST-based parser because paired tags make single-pass parsing viable `[author-claim]` (post 3904).
- **Integration**: the colleague added the validator module to the project's **autotest suite**, so it runs as part of CI `[empirical]` (post 3904).
- **Working status**: "it works" `[empirical]` (post 3904).

The sources do not specify: the validator's name, repository URL, license, exact rule set, tag taxonomy, error-reporting format, or whether it is public. `"the sources do not specify."`

### What "using GRACE to build GRACE tooling" actually means

The case is worth unpacking because the self-hosting is the part that carries the methodology signal.

The author does not give a step-by-step account, but the GRACE workflow as documented elsewhere gives the shape:

- **Phase 1 — architectural graph**: model the validator as a graph of main classes and modules — file reader, tag scanner, rule engine, reporter — with business-process nodes for the validation rules themselves (`../08-workflow-and-phases/top-down-pipeline.md`).
- **Phase 2 — module contracts**: derive contracts per module — parser, rule set, report writer — expanding intent before writing code.
- **Phase 3 — function contracts + code**: generate function bodies with PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS fields *in the validator's own source*, placed before each declaration per the GRACE placement rule (post 3878; see `../03-contracts/contract-fields.md`).

In other words: the validator's source is itself GRACE-annotated, which means the validator can in principle check its own markup — a closed loop.

The sources do not confirm that the colleague's validator does in fact validate itself; `"the sources do not specify."`

### The trade-off the case clarifies

Post 3904 is also the canonical reference for the algorithmic-vs-LLM trade-off in GRACE review:

| Check | Algorithmic validator (~1650 Py lines) | LLM reviewer |
|---|---|---|
| Markup present and balanced? | yes | redundant `[author-claim]` (post 3904) |
| Required fields populated? | yes | redundant `[author-claim]` (post 3904) |
| PURPOSE *truthful*? | no | yes `[author-claim]` (post 3904) |
| KEYWORDS semantically right? | no | yes `[author-claim]` (post 3904) |
| Architectural intent preserved? | no | yes `[author-claim]` (post 3890, post 3904) |
| Tokens per run | ~0 | real tokens `[author-claim]` (post 3904) |
| Latency per run | ~0 | real latency `[author-claim]` (post 3904) |
| Integrates into autotests? | yes | possible but expensive `[empirical]` (post 3904) |

The colleague's posture — module-in-autotests — occupies the left column `[empirical]` (post 3904). The author's posture — LLM review concurrent with code review — occupies the right column `[author-claim]` (post 3904). They are meant to be **run together**, not chosen between `[author-claim]` (post 3904).

The author's summary of the combined posture: for projects that embed documentation in code at GRACE scale, "this is a rather good practice" `[author-claim]` (post 3904).

### Relation to the autonomous-agent story

The validator case is especially meaningful when read alongside the autonomous-agent posts:

- Autonomous / swarm agents can burn billions of tokens looping on failed fixes `[author-claim]` (post 3695; see `../07-autonomous-agents/anti-loop-protection.md`).
- Protections against that are **built into the test framework** — counters, forced-context injection, knowledge-base-style regression memory (post 3695).
- The colleague's validator slots into exactly that layer: it is one more deterministic autotest in the suite that an autonomous agent must pass, and it specifically guards against the failure mode where the agent silently drops GRACE markup during a loop `[empirical]` (post 3904).

So the validator case is not just "a tool someone built." It is evidence that GRACE's story about autonomous-agent infrastructure — **complete autotests + rich logs + algorithmic gates + LLM review** — is buildable on top of GRACE itself.

## Evidence

- **Primary source — post 3904** (`grace_matches.md`). All directly-stated facts in this file come from this post:
  - A colleague from the GRACE training cohort built a validator for GRACE markup using GRACE `[empirical]`.
  - The tool is ~1650 Python lines with non-trivial syntactic-parsing algorithms `[empirical]` + `[author-claim]` on complexity.
  - The module was added to the project's autotest suite `[empirical]`.
  - Trade-off vs LLM review: algorithmic validation is instant and token-free but cannot catch semantic errors `[author-claim]`.
  - The author's default remains LLM review concurrent with code review `[author-claim]`.
  - Author's verdict for in-source-documentation workflows: "rather good practice" `[author-claim]`.
- **Post 3948** (`grace_matches.md`). Formal basis for why a 1650-line Python parser is enough: paired START-END tags give GRACE markup Dyck-Language compatibility, so it parses in one pass without chain-of-thought `[research-backed]`. Referenced paper: arxiv.org/abs/2105.11115 (Shunyu Yao et al. on native Dyck-Language support in transformers).
- **Post 3622** (`grace_matches.md`). Anton's contract-only MCP — independent demonstration that GRACE's START-END overlay allows AST-free tooling. `read_contracts` returns contract IDs + text + implementation line ranges; `get_contracts_by_contract_path` walks PATH-header graphs. The same structural property the colleague's validator relies on `[empirical]`.
- **Post 3878** (`grace_matches.md`). Canonical contract-field list (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS) and placement rule (before declaration). Specifies the surface the validator has to check `[empirical]`.
- **Post 3890** (`grace_matches.md`). RATIONALE / AAG intent sections and the "Captain Obvious" anti-pattern — an example of the semantic failure class only an LLM reviewer can catch, motivating the complementary-not-substitute posture `[research-backed]`.
- **Post 3695** (`grace_matches.md`). Anti-loop protection for autonomous agents — the infrastructure layer into which the validator naturally slots `[author-claim]`.

What the sources do not provide about the validator specifically: name, URL, repository, license, exact rule set, tag taxonomy, test count, or authorship beyond "a colleague from training." `"the sources do not specify."`

## See also

- `../04-markup-system/markup-validators.md` — the general topic page on GRACE markup validators (T24 counterpart).
- `../04-markup-system/start-end-tags.md` — the paired-tag convention the validator relies on.
- `../04-markup-system/xml-like-markup.md` — the broader markup system being validated.
- `../04-markup-system/in-source-documentation.md` — why GRACE embeds docs in code at all.
- `../04-markup-system/markup-compression.md` — where compression is safe vs harmful; a semantic judgment only LLM review can make.
- `../03-contracts/contract-fields.md` — the field set the validator checks.
- `../03-contracts/rationale-and-aag.md` — architectural intent; outside algorithmic reach.
- `../03-contracts/wenyan-prompting.md` — KEYWORDS/LINKS compression whose correctness is only partly syntactic.
- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis for single-pass parseability.
- `../06-testing/tests-in-self-correction-loop.md` — how autotest verdicts (including validator verdicts) feed the debug loop.
- `../07-autonomous-agents/anti-loop-protection.md` — the autonomous-agent layer the validator hardens.
- `../09-tooling/mcp-scepticism.md` — why custom MCPs tend to fail but local CI validators do not.
- `./kirill-132k-loc.md` — large-codebase GRACE adoption, where algorithmic markup gates earn their keep at scale.
- `./alexey-300k-loc.md` — largest reported GRACE codebase; a scale at which markup discipline must be enforced mechanically.
