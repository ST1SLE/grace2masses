---
name: grace-legacy-contracts
description: "Bootstrap GRACE function-level contracts on poorly-documented brownfield Python. Scan target .py files, synthesize PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS contract blocks from existing code and docstrings, enforce wenyan compression + English-only PURPOSE + flush-above-def placement, show unified diffs, and write after per-file confirmation. Use when a Python project has no GRACE contracts and needs the semantic anchors retrofitted for graph-RAG retrieval and sparse-attention navigation."
---

Retrofit GRACE function-level contracts on brownfield Python. This is the AST-free bootstrap path: scrape existing code, synthesize contract blocks, and write them in-source as `#` comments immediately above each `def`. The goal is to create semantic anchors the graph-RAG layer and the transformer's sparse attention can both use.

## Prerequisites

- Target must be a Python project (`.py` files present).
- User has authorized edit-in-place mode; this skill modifies source files after per-file confirmation.
- If `docs/knowledge-graph.xml` exists (from `grace-init` or `grace-plan`), read it first so LINKS fields can reference `M-xxx` and `V-M-xxx` IDs. If it doesn't exist, LINKS fields reference file paths instead.

## Contract block shape

Insert this block as plain `#` comments IMMEDIATELY above each `def` — no blank line between the block and the `def` signature:

```python
# PURPOSE: <one sentence, English only, wenyan-compressed, verb-first>
# INPUTS: <param: type — semantic hint>, ...
# OUTPUTS: <type — semantic hint>
# KEYWORDS: <comma-separated semantic tokens for RAG retrieval>
# LINKS: <M-xxx or V-M-xxx references, or ../path/file.py references>
def target_function(...):
    ...
```

**Placement rule (load-bearing).** The contract block MUST sit on the lines immediately preceding the `def`, with no blank line between. That contiguity is the causal-read anchor: transformer attention near the `def` token picks up the contract without a retrieval round-trip. An existing docstring inside the body is fine (it matches SFT-era expectations), but it is NOT the primary contract location.

## Field rules

- **PURPOSE**: one line, English only (no non-ASCII letters), verb-initial where possible, wenyan-compressed (drop articles and filler: "Compute total", not "This function computes the total"). No trailing period. Soft ceiling ~80 chars.
- **INPUTS / OUTPUTS**: type + one-fragment semantic hint. Omit INPUTS if signature is `()`. Omit OUTPUTS if return is `None` and no side effect is surfaced via the return path.
- **KEYWORDS**: 3–8 comma-separated tokens, lowercase, no duplication of the function name. Target the semantic surface RAG queries will hit.
- **LINKS**: when `docs/knowledge-graph.xml` exists, use `M-xxx` / `V-M-xxx` IDs. Otherwise cite dependency file paths or call-site paths. Never leave empty — write `LINKS: none` if truly standalone.

## Process

### Phase 1 — Scan

Walk the target directory. For each `.py` file, find every `def` (including `async def` and methods inside classes) whose immediately preceding lines do NOT form a valid contract block. A valid block has at minimum a `# PURPOSE:` line on the line directly above the `def`.

Skip:
- Auto-generated files (e.g., `_pb2.py`, `migrations/`).
- Files matching the project's existing exclusion patterns (`.gitignore` hints).

### Phase 2 — Synthesize per function

For each targeted `def`:

1. Read the function body (up to ~50 lines).
2. Read the existing docstring if present (first triple-quoted string).
3. Scan the file for up to 3 call sites referencing this function (`name(` text search is sufficient; an AST pass is optional).
4. Draft **PURPOSE** from the docstring's first line or, if absent, from body behavior.
5. Derive **INPUTS / OUTPUTS** from type hints first; fall back to docstring text; fall back to `<unknown>` with a TODO marker the user will need to resolve.
6. Draft **KEYWORDS** from tokens in the function name, domain terms in the body, and prominent type names.
7. Populate **LINKS** from `docs/knowledge-graph.xml` if present, else from the imports actually used inside the function body.

### Phase 3 — Lint each draft

Before proposing a change, validate:
- PURPOSE is a single line, English-only (reject if any non-ASCII letters appear), verb-initial if possible, no trailing period, ≤ 80 chars.
- KEYWORDS are lowercase, no duplicates, count between 3 and 8.
- No blank line between the last line of the contract block and the `def` line.
- All five fields are present (LINKS may be `none`).

If lint fails, fix the draft before proposing. Do not propose broken contracts and ask the user to fix.

### Phase 4 — Show diff + confirm per file

Present a unified diff of the proposed changes for the file. Wait for explicit user confirmation (y / n / skip) before writing. Default action is no-op if the user is ambiguous.

### Phase 5 — Write

After confirmation, apply the edits. Preserve file indentation, line endings (LF vs CRLF), and trailing newline.

### Phase 6 — Report

After all files processed, emit a short report:
- Files scanned / modified / skipped.
- Functions: contracted / already-contracted / skipped.
- PURPOSE lint failures that required the user to intervene (likely non-English text or multi-line descriptions).
- Suggested next step: `grace-ldd-retrofit` for logging retrofit, or `grace-verification` if planning artifacts exist.

## Legacy scraping pattern (Anton, post 3622)

Files that already use the START-END marker pattern (`# START: block_name` / `# END: block_name` bracketing legacy regions, as described in `kb/03-contracts/legacy-contract-recovery.md`) are treated as pre-anchored: do not remove or alter the markers. Place the function's contract block above the `def` normally, even when the `def` sits inside a START-END region. Do not nest contract blocks inside START-END regions as if they were alternative anchors — they complement each other, they do not replace each other.

## Anti-patterns (reject automatically)

- Contract block separated from `def` by a blank line → close the gap.
- Multi-line PURPOSE → collapse to one line; if it genuinely cannot fit in ~80 chars, the function is probably doing two things.
- Non-English PURPOSE → stop and surface it to the user. Do NOT translate silently; the KB's English-only rule is about disambiguating the agent's retrieval surface, and a silent translation introduces drift.
- Field reshuffling (KEYWORDS above PURPOSE, LINKS before INPUTS, etc.) → use the canonical field order exactly: PURPOSE → INPUTS → OUTPUTS → KEYWORDS → LINKS.
- Wrapping the contract block in triple-quotes or moving it inside the function body → it belongs as `#` comments above the `def`.

## Important

- Do NOT generate or refactor code during this phase. This skill only adds contract comment blocks.
- Every file edit requires explicit per-file confirmation.
- If the project has existing classical-TDD assertion tests, leave them alone. Contract retrofit is orthogonal to the testing philosophy discussion.

## Citations

- Contract field spec: `kb/03-contracts/contract-fields.md`
- Python placement rule: `kb/03-contracts/language-specific/python.md`
- Wenyan compression rationale and targets: `kb/03-contracts/wenyan-prompting.md`
- Legacy START-END scraping (post 3622): `kb/03-contracts/legacy-contract-recovery.md`
- Markup compression ceilings and English-only rule: `kb/04-markup-system/markup-compression.md`
- Why contracts are the second load-bearing layer: `kb/03-contracts/contracts-overview.md`
