# Canonical Example — Python (Reference Convention)

## What

This chapter pins **one concrete spelling** of GRACE markup in Python — a single ~90-LOC file that exercises every markup level GRACE uses (module contract, module map, per-symbol contracts, paired in-function blocks). It exists so that authors and tooling have a fixed reference instead of inferring tag names from prose.

The original posts (3622, 3878, 3948) describe the **pattern** — paired tags inside comments, contract fields above declarations — but do not prescribe an exact tag vocabulary. This chapter selects one vocabulary, shows it in a real file, and explains where each rule comes from. Teams may diverge as long as the underlying constraints (paired, in-comment, before declaration) are preserved; what matters is that **one project pins one spelling**.

Status: **reference convention** distilled from a brownfield retrofit (a 7-checkpoint GRACE migration, 2026-04-26, 113 Python files / 374 contract blocks). Not a normative GRACE upstream prescription.

## Why a fixed example matters

Without a canonical example, three failure modes recur in adoption:

1. **Tag drift inside the same project** — `# START_CONTRACT` here, `# BEGIN_CONTRACT` there, `# REGION` elsewhere. Breaks AST-free tooling (post 3622): a paired-bracket walker has to know the tokens it is matching.
2. **Field-format drift** — INPUTS as a YAML-ish block here, as a numbered list there. Breaks contract extractors and degrades LLM parsing (post 3948).
3. **Markup-level confusion** — agents wrap individual statements with `START_BLOCK_*` (overhead) or skip module-level contracts entirely (loss of the anchor vector). Adjacency matters (post 3878).

A locked example resolves all three by demonstration.

## The example

The file below is the LDD logging helper from a production codebase. It is reproduced verbatim because it is short, real, tested, and exercises every markup level. Russian glosses have been removed in the source comments per the English-only recommendation (post 3889).

```python
# START_MODULE_CONTRACT
#   PURPOSE: Lightweight LDD logging helper that emits canonical GRACE markers
#            ``[Module][function][BLOCK_NAME]`` and belief-state pairs around
#            state-machine transitions. Wraps stdlib logging — does not
#            replace it, so existing logger.info/.warning/.error calls keep working.
#   SCOPE:   Used by all backend modules for verification-grade observability.
#   DEPENDS: stdlib logging only.
#   LINKS:   docs/verification-plan.xml, docs/development-plan.xml M-SHARED.
#   ROLE:    RUNTIME
#   MAP_MODE: EXPORTS
# END_MODULE_CONTRACT
#
# START_MODULE_MAP
#   GraceLogger        - thin wrapper around logging.Logger with .block() and .belief()
#   get_grace_logger   - factory that returns a GraceLogger bound to a module name
# END_MODULE_MAP

from __future__ import annotations

import logging
from typing import Any, Optional


# START_CONTRACT: GraceLogger
#   PURPOSE: Emit GRACE-format log lines bound to a module label, so downstream
#            agents (and the verification plan) can grep by [Module][fn][BLOCK].
#   INPUTS:  module: str — module label like "CoreApi" or "PaymentWorker"
#            base:   logging.Logger | None — optional underlying logger; defaults
#                    to logging.getLogger(module).
#   OUTPUTS: GraceLogger instance.
#   SIDE_EFFECTS: none on construction; .block()/.belief() emit through stdlib logging.
#   LINKS:   docs/verification-plan.xml GlobalPolicy/log-format
# END_CONTRACT: GraceLogger
class GraceLogger:
    def __init__(self, module: str, base: Optional[logging.Logger] = None) -> None:
        self._module = module
        self._logger = base if base is not None else logging.getLogger(module)

    # START_CONTRACT: GraceLogger.block
    #   PURPOSE: Emit a block-marker log line at INFO level.
    #   INPUTS:  fn: str  — function or step label
    #            blk: str — BLOCK_NAME (UPPER_SNAKE)
    #            msg: str — short human-readable message (optional)
    #            **fields — extra structured key=value pairs appended to the line
    #   OUTPUTS: None
    #   SIDE_EFFECTS: emits through underlying stdlib logger at INFO
    # END_CONTRACT: GraceLogger.block
    def block(self, fn: str, blk: str, msg: str = "", **fields: Any) -> None:
        # START_BLOCK_LDD_EMIT
        prefix = f"[{self._module}][{fn}][{blk}]"
        body = msg.strip()
        extras = " ".join(f"{k}={v}" for k, v in fields.items()) if fields else ""
        line = " ".join(part for part in (prefix, body, extras) if part)
        self._logger.info(line)
        # END_BLOCK_LDD_EMIT

    # START_CONTRACT: GraceLogger.belief
    #   PURPOSE: Emit a belief-state log line at INFO comparing the agent's hypothesis
    #            against the observed runtime value. Required at state-machine
    #            boundaries.
    #   INPUTS:  fn: str
    #            blk: str
    #            belief: Any — the hypothesis (typically expected status enum value)
    #            actual: Any — the runtime value
    #            **fields — extra key=value annotations
    #   OUTPUTS: None
    #   SIDE_EFFECTS: emits through underlying stdlib logger at INFO; never raises.
    #   NOTE: Comparison is by str() so enum/string mixes still work cleanly.
    # END_CONTRACT: GraceLogger.belief
    def belief(self, fn: str, blk: str, belief: Any, actual: Any, **fields: Any) -> None:
        # START_BLOCK_LDD_BELIEF
        status = "MATCH" if str(belief) == str(actual) else "MISMATCH"
        prefix = f"[{self._module}][{fn}][{blk}]"
        core = f"BELIEF: {belief} ACTUAL: {actual} STATUS: {status}"
        extras = " ".join(f"{k}={v}" for k, v in fields.items()) if fields else ""
        line = " ".join(part for part in (prefix, core, extras) if part)
        self._logger.info(line)
        # END_BLOCK_LDD_BELIEF


# START_CONTRACT: get_grace_logger
#   PURPOSE: Factory for module-bound GraceLogger; preferred entry point so
#            callers do not need to import the class directly.
#   INPUTS:  module: str
#            base:   logging.Logger | None
#   OUTPUTS: GraceLogger
#   SIDE_EFFECTS: none
# END_CONTRACT: get_grace_logger
def get_grace_logger(module: str, base: Optional[logging.Logger] = None) -> GraceLogger:
    return GraceLogger(module, base)
```

## Tag vocabulary used

Every paired tag in the file is one of five shapes. This is the entire vocabulary — there is no sixth.

| Tag pair | Wraps | Position | Source rule |
|---|---|---|---|
| `# START_MODULE_CONTRACT` / `# END_MODULE_CONTRACT` | Module-level header (PURPOSE, SCOPE, DEPENDS, LINKS, ROLE, MAP_MODE) | Top of file, before any code | Module contract is the highest-leverage anchor (post 3890); its presence at file head feeds the anchor vector for every symbol below (post 3878). |
| `# START_MODULE_MAP` / `# END_MODULE_MAP` | Inventory of public exports with one-line summaries | Immediately after `END_MODULE_CONTRACT` | Map gives the LLM a fast outline without reading the body; `MAP_MODE: EXPORTS` declares the listing scope. Companion to module contract. |
| `# START_CONTRACT: <name>` / `# END_CONTRACT: <name>` | Per-symbol contract fields (PURPOSE, INPUTS, OUTPUTS, SIDE_EFFECTS, LINKS, NOTE) | Immediately above each public `def` / `class` | Anchor vector for the symbol (post 3878). The trailing `<name>` lets a bracket-walker tie open and close even when nesting depth grows. |
| `# START_BLOCK_<NAME>` / `# END_BLOCK_<NAME>` | A logical block inside a function body (e.g., emit, transaction window) | Inside `def`, paired around the block | Lifts Python into Dyck-Language compatibility (post 3948); also marks where LDD log markers fire. `<NAME>` is `UPPER_SNAKE`. |
| `[Module][function][BLOCK_NAME]` | (Not markup — the runtime log line shape) | At runtime, inside log strings | Verification-grade pattern. The same tokens used in `START_BLOCK_*` appear as the third bracket of the runtime line, so log greps line up with source greps. See `../05-logging-ldd/log-driven-development.md`. |

### Field grammar inside contracts

```
PURPOSE: <one concise sentence>
INPUTS:  <name>: <type> — <brief description>           (one bullet per input)
OUTPUTS: <type> — <brief description>                   (or `None`)
SIDE_EFFECTS: <one line; "none" if pure>
LINKS:   <comma-separated paths into the GRACE substrate>
NOTE:    <optional caveat>
```

Two-space indent inside a `# ` comment. No more, no less. Field names are `UPPER_SNAKE`. Field bodies are short — the canonical example fits every field on one to three lines.

`KEYWORDS` and `RATIONALE/AAG` from the broader contract spec are not present in this particular file because the symbols here are runtime infrastructure, not domain logic; see `../03-contracts/contract-fields.md` for when they apply.

## Why this exact shape — five rationale notes

Each rule on the page maps back to an existing GRACE chapter. Nothing in the example is novel; the example only fixes a spelling.

1. **Module contract first.** Anchor vector for everything below it (post 3878). See `../03-contracts/contracts-overview.md`.
2. **Module map second.** Outline-style summary the LLM can scan without reading the body — companion to the module contract. The `MAP_MODE: EXPORTS` line is what lets a contract-extractor tool know whether the listing is exhaustive.
3. **Per-symbol contract immediately above `def`/`class`.** Causal-read anchor at the name token (post 3878). Docstring may still appear below `def` as an SFT-hint (post 3892); this example omits docstrings because the contract above already carries the same information and a docstring would duplicate it.
4. **Paired in-function blocks for state-changing regions.** Dyck-Language compatibility (post 3948); also a hook for AST-free tooling (post 3622) and the LDD log-line correlate.
5. **Tag tokens are `UPPER_SNAKE` and the trailing identifier matches START to END.** A bracket-walker can pair them with no language-specific parser. See `../04-markup-system/start-end-tags.md`.

## What this is NOT

- **Not the only valid spelling.** A team adopting GRACE may pick `# REGION_CONTRACT_BEGIN` / `# REGION_CONTRACT_END`, or `# <<CONTRACT` / `# CONTRACT>>`, etc. The constraints are: in-comment, paired, balanced. Pick one and stick to it project-wide. Mixed spellings inside one project break the AST-free walker.
- **Not the maximum field set.** `KEYWORDS`, `LINKS`, `RATIONALE`, `AAG` are valid additions for domain code; this example happens to be runtime infrastructure where they would not pull weight. See `../03-contracts/contract-fields.md`.
- **Not a replacement for docstrings.** Below-`def` docstrings still act as SFT-hints inside the recent-token sparse-attention window (post 3892); they remain optional but are not forbidden. This particular file omits them only because there is no domain-side description to add beyond what the contract already states.

## Adopting the convention into a fresh project

Three steps:

1. **Copy this file into your project as the seed.** It compiles, runs, and emits the canonical log shape. Use it as the LDD logger or strip it down to a placeholder — either way the markup is now in the codebase as a self-evident reference.
2. **Pin the tag spelling in your `AGENTS.md` / `CLAUDE.md` / `GRACE.md`** (whichever you use) by linking to this file. New contributors and agents read one example, not five different conventions.
3. **Run a one-shot brackets validator** as a CI check: walk every `*.py` file, verify each `# START_<X>` has a matching `# END_<X>` with the same `<X>`. ~50 lines of Python; see `markup-validators.md` and post 3904 for the reference 1650-LOC validator built on the same paired-bracket overlay.

For brownfield adoption (existing codebase, no markup yet) see `../08-workflow-and-phases/brownfield-adoption.md` (forthcoming).

## See also

- `./start-end-tags.md` — the formal Dyck/AST-free argument behind paired tags.
- `./xml-like-markup.md` — broader markup family.
- `./in-source-documentation.md` — enterprise lineage.
- `./markup-validators.md` — algorithmic validation on top of paired markup.
- `../03-contracts/language-specific/python.md` — the original Python placement rules; this file is the worked-out example of those rules.
- `../03-contracts/contract-fields.md` — full field spec (PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS / RATIONALE / AAG).
- `../05-logging-ldd/log-driven-development.md` — why the runtime log shape mirrors the `START_BLOCK_*` markup.
- `../06-testing/agent-based-testing.md` — log-trajectory tests on top of the `[Module][fn][BLOCK]` markers.
