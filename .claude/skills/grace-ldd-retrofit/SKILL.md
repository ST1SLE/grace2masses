---
name: grace-ldd-retrofit
description: "Retrofit Log-Driven Development on brownfield Python with poor existing logging. Audit existing logger.* calls, inject [ModuleName][function_name][BLOCK_NAME] correlation-token prefixes, add belief-state log lines at critical branches, and mark the load-bearing lines with @forced-context so a debugging harness can push them into the agent's next-turn context. Use when a Python project has scattered or prefix-less logging that cannot be traced back to specific code blocks, blocking autonomous-agent debugging and trajectory reading."
---

Retrofit LDD (Log-Driven Development) on brownfield Python. Adds the three mechanisms the KB identifies as archimportant for agent-readable trajectories: correlation tokens, belief-state log lines at critical branches, and forced-context markers on load-bearing lines.

## Prerequisites

- Target must be a Python project.
- Target uses `logging.getLogger()` or a compatible interface that accepts string-prefixed messages. If the target uses `print`, a bespoke logger, or structlog-style bound context exclusively, STOP and ask the user how to proceed — do not migrate loggers silently.
- User has authorized edit-in-place; this skill modifies source files after per-file confirmation.

## Core pattern: correlation token

Every log line must carry a three-part prefix so trajectory logs can be deterministically sliced by module, function, and logical block:

```
[ModuleName][function_name][BLOCK_NAME]: message
```

- `ModuleName` — file module name, CamelCase (e.g., file `order_processor.py` → `OrderProcessor`).
- `function_name` — the enclosing `def`'s symbol name exactly, snake_case.
- `BLOCK_NAME` — logical block label, UPPER_SNAKE_CASE, unique within the function. Examples: `VALIDATE_INPUT`, `FETCH_USER`, `COMMIT_TX`, `ON_ERROR`, `ON_TIMEOUT`.

This is not a convention borrowed from structured logging. It's a string-prefix convention specifically so a plain `grep` over logs lets an agent localize failures without understanding a schema.

## Process

### Phase 1 — Audit existing log calls

Find every `logger.debug|info|warning|error|critical|exception(...)` call. Categorize each:
- **missing-prefix**: first positional arg does not begin with `[`.
- **partial-prefix**: starts with `[` but does not match the three-bracket pattern.
- **compliant**: matches `[Module][func][BLOCK]:` shape.

### Phase 2 — Plan correlation-token injection

For each non-compliant call, derive the three tokens:

- **Module**: file name stripped of `.py`, converted to CamelCase.
- **Function**: enclosing `def`'s symbol. If the call sits at module level (no enclosing `def`), use `__module__`.
- **Block**: derived by heuristics, in order:
  1. Nearest preceding `# START: <name>` marker (Anton's pattern, `kb/04-markup-system/start-end-tags.md`) → use `<name>` uppercased.
  2. Nearest structural context: `try:` → `TRY_<n>`; `except <Type>:` → `EXCEPT_<TYPE>`; `if <cond>:` → `BRANCH_<short_excerpt>` (truncated excerpt of the predicate).
  3. Position in function: `ENTER` for the first log, `EXIT` for the final log, `STEP_<n>` for the sequence in between.

Each BLOCK_NAME must be unique within its function. If a heuristic collides, append a numeric suffix (`TRY_1`, `TRY_2`).

### Phase 3 — Inject belief-state log lines at critical branches

A critical branch is:
- Any `if`/`elif`/`else` where at least one arm performs an external effect (file write, network call, DB commit, subprocess, state mutation of a shared object).
- Any `except` block.
- Any `return` that ends the function early on a validation failure.

Before each such branch performs its action, insert:

```python
logger.info("[Module][func][BLOCK]: belief=<plain-English hypothesis about current state>")
```

The belief-state line states what the code ASSUMES is true at this point. Examples:
- `belief=request passed schema validation`
- `belief=user is authenticated and has write permission on resource_id=%s`
- `belief=db connection alive and in autocommit mode`

This lets a debugging agent see the author's mental model and catch false assumptions when logs diverge from belief.

### Phase 4 — Mark critical log lines for forced-context injection

Load-bearing log lines — the ones that must be surfaced into an agent's context regardless of log volume — get an inline comment marker:

```python
logger.error("[Module][func][ON_ERROR]: belief=tx will rollback; error=%s", e)  # @forced-context
```

The `@forced-context` marker is the convention for a debugging / test harness to identify which lines to push into the agent's next-turn context. Mark:
- All `logger.error` and `logger.critical` calls.
- Belief-state lines at the highest-risk branches (branches touching persistence, network, or financial state).
- Entry / exit lines for the top-level handler of each ENTRY_POINT module (per `docs/knowledge-graph.xml` if it exists, else judged from the file — `main()`, FastAPI routes, Celery tasks, etc.).

Do NOT mark every line. Target: 5–15 % of log lines marked. Over-marking defeats the purpose by filling the forced-context budget with noise.

### Phase 5 — Show diff + confirm per file

Present a unified diff for each modified file. Wait for explicit user confirmation (y / n / skip). Default is no-op if ambiguous.

### Phase 6 — Write

Apply the edits on confirmation. Preserve:
- Indentation.
- Existing `%s`/`%d` lazy-formatting placeholders (do not convert to f-strings — lazy formatting is cheaper when the log level is filtered out).
- Line endings and trailing newline.

### Phase 7 — Report

- Files scanned / modified / skipped.
- Log calls: already-compliant / prefix-injected / belief-state added / forced-context marked.
- Functions touched.
- Suggested next step: run the target's existing test suite or the agent workflow under LDD; observe whether trajectory logs now allow localization of any failures that previously needed step-through debugging.

## Field guide

### BLOCK_NAME discipline

BLOCK_NAMEs must be:
- UPPER_SNAKE_CASE.
- Unique within the enclosing function.
- Semantic — prefer verb-ish (`VALIDATE_INPUT`, `COMMIT_TX`) or state-ish (`ON_TIMEOUT`, `ON_NOT_FOUND`) over positional (`BLOCK_1`, `STEP_2`). Only fall back to positional when a better label genuinely cannot be derived.

### Belief-state phrasing

Good (informative, self-contained, referenceable against runtime state):
- `belief=order.status == 'PAID' so ship can proceed`
- `belief=cache miss, will fetch from upstream`

Bad:
- `belief=True` (not informative).
- `belief=see line 42` (breaks when code moves).
- `belief=trying now` (states action, not hypothesis).

### Forced-context scope

Target is 5–15 % of total log lines marked `@forced-context`, concentrated at:
- Error-path terminals.
- State-transition log lines (`belief=` lines at branches with external effect).
- Entry and exit of top-level handlers.

## Anti-patterns (reject automatically)

- Replacing `logger.info(...)` with `print(...)` under any circumstance.
- Migrating to `logger.bind(...)` / `structlog.contextvars` instead of the prefix pattern. Structured context is agent-hostile for trajectory reading; the prefix pattern is a plain-text slicing convention that `grep` can exploit.
- Splitting prefix and message across two log calls. The full `[Module][func][BLOCK]: message` must be a single string argument.
- Stripping lazy-formatting placeholders (`%s`) and rewriting them as eager f-strings. Keep the original formatting style unless it is broken.

## Important

- This skill does not introduce a new logging library, reconfigure handlers, or change log levels. It only adds prefix tokens, belief-state lines, and forced-context markers to existing calls.
- If the target's existing log handler does NOT preserve the message string verbatim (e.g., a JSON-structured handler that drops the message into a field but mangles the brackets), surface the issue to the user and recommend switching the handler before the retrofit is useful.
- If `docs/knowledge-graph.xml` exists with ENTRY_POINT module IDs, consult it to decide which modules get entry/exit forced-context marking. Otherwise use the judgment heuristic above.

## Citations

- Correlation-token pattern: `kb/05-logging-ldd/log-to-code-correlation.md`
- Belief-state logging: `kb/05-logging-ldd/belief-state-logging.md`
- Forced-context injection: `kb/05-logging-ldd/forced-context-injection.md`
- LDD overview and rationale: `kb/05-logging-ldd/log-driven-development.md`
- START-END block markers (source of BLOCK_NAME when present): `kb/04-markup-system/start-end-tags.md` and `kb/03-contracts/legacy-contract-recovery.md` (post 3622)
