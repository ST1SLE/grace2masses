# Bringing GRACE to an Existing Codebase

## What

A practical playbook for adopting GRACE in a project that already has running code, tests, CI, a process layer (TDD / OpenSpec / RFCs / spec-kit / etc.), and a development team that cannot stop work to wait for a methodology change.

The greenfield path (chapters `04`–`07`) assumes you write a contract above every new symbol from day one. Brownfield does not have that luxury. This chapter answers: *given a codebase that exists already, how do you land GRACE in it without breaking flow, without rewriting the world, and without throwing away the audit trail of whatever workflow preceded it?*

## Why a separate chapter

Three brownfield realities push back on the greenfield writeup:

1. **Mass retrofit cost.** A medium project has hundreds of public symbols. Hand-writing contracts is weeks of work; an undirected agent run is unsafe (broken signatures, drift on shared types, accidentally rewritten business logic). You need a *strategy*, not a sweep.
2. **Process-layer inertia.** Most teams already have a ceremony in place — TDD pyramid, RFC docs, spec-driven workflows, ticket-template PRs. GRACE replaces the *artifact* layer (substrate, LDD, verification) but is mostly orthogonal to ceremony. You need a clear *retire / keep / coexist* decision per ceremony.
3. **Reversibility pressure.** Greenfield mistakes are cheap; brownfield mistakes can corrupt a working codebase. Every checkpoint must be a single revertable commit, and the migration must run on a branch that is not the integration branch.

The patterns below are distilled from a real 7-checkpoint brownfield retrofit (see *Worked example* at the end). Numbers from that retrofit appear as empirical anchors only — your project's numbers will differ.

## How — the seven-checkpoint pattern

Each checkpoint is a single commit. Each commit is independently revertable. Sequence matters: substrate before contracts, contracts before logging, logging before tests, code before docs.

```
CP0   preflight        — clean working tree; archive plan saved
CP1   substrate        — docs/*.xml + LDD logger + smoke test
CP2   mothball         — retire predecessor process layer (move, do not delete)
CP3   contracts (BE)   — backend mass retrofit by parallel subagents
CP4   contracts (FE)   — frontend mass retrofit by parallel subagents
CP5   LDD wire-up      — emit log markers at state-machine boundaries
CP6   LDD test fixture — log-trajectory assertion helper + smoke tests
CP7   docs sync        — root + per-module AGENTS, CLAUDE/README, grace-refresh
```

CP3 and CP4 are the only steps that touch large amounts of code. Everything else is small and easy to revert.

### CP0 — preflight

Three things, in order:

1. **Working tree clean.** `git status` empty, no stashes, no pending merges. The migration touches many files; mixing it with an unrelated change is the #1 way to get a botched revert later.
2. **Branch.** Create `grace-migration` (or equivalent) off the integration branch. Never run the migration on `main`/`master`.
3. **Worktree hygiene.** If your team uses parallel worktrees, prune stale ones (`git worktree prune`). Subagent-driven retrofits dislike old worktrees that share files with the active one.

No commit at CP0; it's preflight only.

### CP1 — substrate

Drop in the GRACE artifacts the rest of the migration depends on. One commit; small; easy to review.

What lands:

- **Six XML artifacts in `docs/`** — `requirements.xml`, `technology.xml`, `development-plan.xml`, `verification-plan.xml`, `knowledge-graph.xml`, `operational-packets.xml`. Populate from your existing PDD / RFCs / architecture docs. **Do not duplicate** large product specs into `requirements.xml`; let it be an *index* of `§X-anchor → existing-doc#section`. See `04-markup-system/` for the schema and `02-semantic-graph/` for the graph artifacts.
- **An LDD logger module** in your shared package (~50–100 LOC). Smoke-test it: `import` works, one call emits the canonical `[Module][fn][BLOCK]` line, one belief call emits `BELIEF: x ACTUAL: y STATUS: MATCH|MISMATCH`. See `04-markup-system/canonical-example-python.md` for the worked example.
- **No production code touches yet.** Existing call sites still use whatever logger they used. The new logger is additive.

Decision flagged here: **leave the root `AGENTS.md` (or equivalent root agent-onboarding file) alone for now.** Rewriting it before contracts exist forces you to describe a methodology that is not yet in the code. The right time for that rewrite is CP7.

### CP2 — mothball the predecessor

If your project has an existing process layer (TDD harness, OpenSpec change directory, RFC index, spec-kit folder, custom orchestrator scripts, manual-test scenarios that no agent reads anymore), this is the checkpoint to retire it.

**Move, do not delete.** The predecessor encodes decision history. Throwing it away erases context that future incident analysis will want.

Recommended layout:

```
docs/.archive/<predecessor-name>/   ← prose + plans
scripts/.archived/                  ← retired automation
.archive/legacy-skills/             ← retired agent skill files
```

Two operational hazards to watch:

1. **Discovery races.** If your agent harness auto-discovers skills/commands in directories like `.claude/`, do not rename folders *inside* that directory — the harness can re-discover the renamed folder as a new namespace. Move the files *out* of the discovery path entirely.
2. **Allowlist/permission files.** If your harness writes a permissions file every time you run a command, do not try to clean stale entries out of it during the migration — the harness will race your edits. Note the residue in the migration log and clean it by hand later. Allowlist noise is harmless; chasing it during a migration is not.

Keep the predecessor's scripts that are still useful as plain utilities (e.g., dev-environment bootstrap, port-collision handling, container up/down). Retire only the *workflow* scripts.

Output: one commit, `chore(grace): mothball <predecessor> process layer`.

### CP3 / CP4 — contract retrofit

These are the heavy checkpoints. Each is one commit, comments-only — no signatures, no behavior, no imports changed. The only safe way to do them at scale is **parallel subagents with non-overlapping write scopes**.

**Light-retrofit policy.** Do not attempt to contract every line of the codebase. The targets are:

| Target | Action |
|---|---|
| Public function / public class | Full contract (PURPOSE / INPUTS / OUTPUTS / SIDE_EFFECTS / LINKS) |
| Module file | MODULE_CONTRACT + MODULE_MAP at top |
| Declarative class (ORM model, DTO/DataClass, Pydantic schema, type-only) | MODULE_CONTRACT only (`MAP_MODE: TYPES`); skip per-symbol contracts |
| Private helper (`_foo`, `__bar`) | Skip |
| Auto-generated code (gRPC stubs, OpenAPI clients, codegen output) | Skip; document the generator instead |
| Migration version files (immutable historical snapshots) | Skip |
| Test files | Skip during retrofit; contract them when you next touch them |

This policy yields ~3–4 contracts per file on logic-heavy modules and 1 contract per file on declarative modules. Empirical anchor: a backend with ~110 source files lands at ~370 contracts in a single CP3 run.

**Parallel-subagent orchestration.** Split the codebase into non-overlapping write scopes (one per module / one per top-level package). Spawn one subagent per scope with:

- Read access to the whole repo (so contracts can mention cross-module concepts)
- Write access only to its assigned paths
- A short brief: target paths, light-retrofit policy, format example link, INV constraints to mention
- A required `ast.parse()` (Python) or `tsc --noEmit` (TypeScript) self-check before reporting done

A reasonable scope split is one subagent per architectural module. Empirical anchor: 7 backend subagents finished CP3 in roughly the wall-clock time of a long lunch; 2 frontend subagents finished CP4 in roughly an hour.

**Handling the harness.** If you run this in autonomous mode, set explicit safety rules in the harness brief: never push, never `git reset --hard`, never edit secrets, halt on N consecutive subagent failures. See *Autonomy safety rules* below for the concrete list.

**What to capture in the migration log.** Subagents will surface real bugs and design concerns while reading every public symbol. Capture each in the migration log under "concerns flagged for human review". Do not act on them in CP3/CP4 — those checkpoints are comments-only by contract. Acting on a flagged bug mid-retrofit makes the commit no longer revertable as a pure substrate change.

### CP5 — LDD wire-up

Light wiring: pick the state-machine boundaries and atomic-transaction commits that already matter in your domain (the same boundaries your incident retros keep coming back to), and emit `[Module][fn][BLOCK]` markers and BELIEF/ACTUAL pairs there.

Per touched file, expect:

- 2 added lines at top (`from <pkg>.grace.logging import get_grace_logger` + `_grace_log = get_grace_logger("<MODULE_LABEL>")`)
- 1–3 emission lines at the strategic boundaries

Empirical anchor: a backend retrofit landed LDD on 8 functions across 7 files for ~40 lines of new code total. CP5 is *small on purpose* — the goal is to anchor LDD where it is most informative, not to instrument every function.

Existing logger calls (`logger.info`, `.warning`, `.error`) stay — the new logger is additive. See `05-logging-ldd/log-driven-development.md` for which boundaries deserve a marker.

### CP6 — LDD test fixture

Ship the test-side helper that lets new tests assert on log trajectories, plus a smoke-test file proving the helper works.

Two artifacts:

1. **`GraceLogCapture`** — a context manager that installs a logging handler, captures emitted lines, and exposes `assert_trajectory(*expected)` and `beliefs(status="MATCH"|"MISMATCH")` helpers. See `06-testing/` for the reference implementation.
2. **A pytest fixture** named `grace_logs` (or equivalent for your test runner) wrapping the context manager. Place a `conftest.py` (or test runner equivalent) in each test root that wants the fixture.

Tests that opt into log-trajectory assertions tag the file with a `# GRACE-LDD` header comment so reviewers know the file's verification model differs from classical assertion-based tests.

Existing tests untouched. The fixture is opt-in.

### CP7 — docs sync

The last checkpoint pulls everything together at the documentation layer.

What lands:

1. **Root `AGENTS.md` (or `CLAUDE.md` / `GRACE.md`) rewrite.** Replace the predecessor-methodology section with a GRACE section: substrate pointers, LDD pointers, verification skill table, INV constraints, migration-history pointers to the `.archive/` paths from CP2.
2. **Per-module `AGENTS.md` surgical edits.** If you adopt the nested AGENTS.md pattern (see `08-workflow-and-phases/nested-agents-md.md`), each module's local AGENTS.md gets a small edit replacing TDD-era guidance with a pointer to the verification markers that module owns.
3. **README** — first-paragraph methodology mention, quickstart, architecture pointer.
4. **`grace-refresh` run.** If you have the `grace-*` plugin installed, run `grace-refresh` to confirm: substrate ↔ code in sync, no orphan CrossLinks, no missing module declarations. Fix any trivial drifts inline (status flags on phase migration steps, removed "(planned)" placeholders that are now real).

Output: one commit, `docs(grace): AGENTS/CLAUDE/README updated for GRACE`.

## Decision matrix — what to retire vs. keep

GRACE is the substrate. Most other things in your project remain.

| Predecessor element | Keep / Retire / Adapt |
|---|---|
| TDD strict assertion equality | **Adapt.** Existing assertion tests still run. New behavior tests use log-trajectory assertions. See `06-testing/`. |
| Spec-driven workflow (OpenSpec / spec-kit / RFC index) | **Retire.** GRACE substrate (XML artifacts) replaces the role. Move predecessor to `docs/.archive/`. |
| 2-phase TDD (RED/GREEN cycles) | **Retire.** LDD replaces it. |
| Phase plans / orchestration scripts | **Retire** if they encoded the predecessor's workflow. **Keep** dev-utility scripts (env setup, container up/down, port collision avoidance). |
| Manual-test scenario docs that no agent reads | **Retire.** Move to `.archive/`. Replace with verification scenarios in `docs/verification-plan.xml`. |
| INV-* invariants (auth, atomicity, PII) | **Keep.** GRACE explicitly carries them — every contract LINKS field can reference them, and the verification plan grades against them. |
| Per-module `AGENTS.md` files | **Keep and adapt.** See `nested-agents-md.md`. |
| ORM models / DTOs / type-only modules | **Keep code as-is; add MODULE_CONTRACT only.** No per-symbol contracts on type-only declarations. |
| CI / linters / formatters | **Keep.** Add a paired-bracket markup validator as a new check. See `04-markup-system/markup-validators.md`. |
| Code review checklists | **Keep and adapt.** Add "every new public symbol carries a contract" and "log markers at state-machine boundaries". |

## Autonomy safety rules

If you run any portion of the migration in `--dangerously-skip-permissions` (or your harness equivalent), put these rules in the harness brief and the subagent briefs verbatim:

- **No remote pushes** at any checkpoint. The whole migration stays on a local branch until the user reviews it.
- **No `git reset --hard`, no `git push --force`, no `git checkout --`** without explicit user approval.
- **No edits to secrets** (`.env`, credentials, signing keys), CI/CD pipeline files, or anything outside the project root.
- **Halt-and-log on N consecutive subagent failures.** Recommended N=3. Failure mode: write a status report to the migration log, exit cleanly, do not retry blindly.
- **Comments-only on retrofit checkpoints.** CP3 and CP4 must not change signatures, behavior, or imports. Subagents that flag a real bug record it in the migration log and move on; the bug fix is a separate commit after the migration.
- **Self-check before report.** Subagent reports "done" only after running an AST parse (or type-check) over its written paths and confirming clean output.
- **Migration log is append-only during the run.** All decisions, deviations, and skipped items go in. Honesty about skipped work is more valuable than a false "all clean" report.

## Reversibility checklist

After every checkpoint, confirm:

- [ ] One commit landed; no half-checkpoint state
- [ ] `git revert <sha>` would cleanly undo this checkpoint without disturbing later ones
- [ ] Smoke test from this checkpoint still passes (substrate imports, LDD logger emits, test fixture runs)
- [ ] Migration log updated with what landed, what was skipped, and why

If any of these fail, fix before starting the next checkpoint. Do not stack two non-revertable checkpoints.

## Common pitfalls

1. **Mixing the migration with unrelated work.** Keep the migration branch single-purpose. If you find a bug while contracting, log it in the migration log and fix it on a separate branch after.
2. **Trying to contract auto-generated files.** They will be regenerated and your contracts will be lost. Document the generator instead.
3. **Translating contracts on the fly.** Pick English (the LLM training-distribution recommendation, see `03-contracts/language-specific/python.md`) and write contracts in English from CP3 onward. Do not mix languages.
4. **Skipping the smoke test at CP1.** If the LDD logger does not emit the canonical line shape on day one, every later checkpoint inherits the bug.
5. **Rewriting the root `AGENTS.md` at CP1.** Do it at CP7. The root file should describe what is in the codebase, and at CP1 the codebase does not yet have GRACE substrate spread across it.
6. **Letting the harness auto-mutate inside `.claude/` or its equivalent.** See CP2's discovery-race note. Move retired skill/command files *out* of the discovery path, not just to a hidden subfolder.
7. **Treating "concerns flagged for human review" as bugs to fix immediately.** They are not. They are deferred-action items. Fix them on separate commits after the migration.

## Worked example

A 2026-04-26 retrofit of a Python/FastAPI + React/TypeScript monorepo executed all seven checkpoints in `--dangerously-skip-permissions` mode while the user was offline. End state:

- 7 commits on the migration branch, each independently revertable
- ~110 Python files contracted (~370 contract blocks) in CP3 via 7 parallel subagents on non-overlapping module scopes
- ~120 TypeScript files contracted (~290 contract blocks) in CP4 via 2 parallel subagents
- 8 boundary functions LDD-wired in CP5 (~40 LOC of new code total)
- 11 GRACE smoke tests passing in CP6
- 9 AGENTS.md files + new CLAUDE.md + expanded README updated in CP7
- Predecessor process layer (97 archived spec-driven changes, 77 specs, 10 phase plans, 5 manual-test scenarios, 3 workflow scripts, 4 retired skill files) preserved under `.archive/` paths
- 7 design concerns flagged during retrofit, none blocking, all logged for a follow-up review pass

The autonomous run produced no false "all clean" reports — every skipped step (a permissions-file trim that raced with the harness, 8 immutable migration version files left untouched, private helpers skipped per the light-retrofit policy) is recorded in the migration log.

This is one data point. Your numbers will differ; the *shape* of the playbook should not.

## See also

- `./top-down-pipeline.md` — overall GRACE workflow shape (substrate first, contracts second, logging third).
- `./architect-code-debug-modes.md` — the three operating modes a contracted codebase enables.
- `./nested-agents-md.md` — root + per-module agent-onboarding pattern (forthcoming).
- `../04-markup-system/canonical-example-python.md` — the contract format you retrofit toward.
- `../04-markup-system/markup-validators.md` — CI gate on paired-bracket markup integrity.
- `../05-logging-ldd/log-driven-development.md` — what the LDD wire-up at CP5 is actually emitting.
- `../06-testing/agent-based-testing.md` — log-trajectory test approach the CP6 fixture supports.
- `../09-tooling/cli-not-custom-mcp.md` — why the migration relies on `git`, `ast`, simple bracket walkers rather than custom MCP servers.
- `../13-antipatterns/all-antipatterns.md` — patterns to avoid; many of them are tempting during a brownfield rush.
