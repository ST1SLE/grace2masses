# Nested AGENTS.md — Root + Per-Module Onboarding

## What

The convention of placing one **root `AGENTS.md`** at the project root plus one **smaller `AGENTS.md` in each module directory**. Both files are agent-onboarding documents read at session start; together they form a two-tier navigation surface that scales with project size.

This pattern was developed inside non-GRACE methodologies (early dev-workflow KBs, spec-driven-workflow projects). It is **fully GRACE-compatible** and complements the substrate XMLs cleanly: the XML artifacts are the *machine-readable* graph; the `AGENTS.md` files are the *human-and-agent-readable* navigation overlay. Each tier carries different content and is read in different cadences.

The chapter ships:

- A pinned root `AGENTS.md` template (~150–200 lines).
- A pinned module `AGENTS.md` template (~50–80 lines).
- Section conventions, with rationale per section.
- Adoption guidance — when to add nested files, how to keep them in sync, what to delete.

## Why nested instead of flat

Three forces push toward nesting:

1. **Context-budget pressure.** A single root file at 600 LOC is read into the agent's context every time the agent starts a task. If the project has 7 modules and each module has 80 lines of module-specific guidance, putting all 7 into the root makes the root unwieldy and forces the agent to read 560 lines of *other modules' guidance* even when working on one module.
2. **Locality of constraint.** Module-specific constraints (e.g., "this module MUST NOT call external APIs directly — delegate to a worker") only matter when working in that module. They are noise in the root.
3. **Adjacency for the agent.** Agents that read the directory tree to understand a task path naturally encounter `AGENTS.md` next to the source they are about to edit. Adjacency is the same principle that motivates in-source contracts (`../03-contracts/contracts-overview.md`); nested AGENTS.md applies it at module scope.

The root file becomes the *navigator* (always read), and the module file becomes the *local brief* (read when the work is in this module).

## What goes where — division of labor

| Section | Root | Module |
|---|---|---|
| Project name + one-line purpose | yes | yes (module-scoped) |
| Authoritative spec pointer (e.g., PDD path) | yes | reference, not pointer |
| Tech stack table | yes (full) | subset relevant to module |
| Module map (table of all modules) | yes | no |
| Cross-cutting constraints (INV-* / shared invariants) | yes | reference by ID, do not duplicate |
| State-machine catalog | yes (top-level list) | only the transitions this module owns |
| Development methodology (GRACE substrate, LDD, verification) | yes | brief reference + module-local fixture path |
| Workflow skills table | yes | no |
| Environment workarounds | yes | only if module-specific |
| General rules (cross-cutting style) | yes | no |
| Module scope ("this module is responsible for") | no | yes |
| Module constraints ("this module MUST NOT") | no | yes |
| Module testing fixtures | brief mention | full detail |
| Migration / adoption history | yes | no |

The rule: **never duplicate**. If the root says it, the module references it by section name or ID. Duplication breeds drift.

## Root `AGENTS.md` template

The root file is the navigator. It must be small enough to fit comfortably in every context-load, but rich enough to brief a fresh agent in one read. Target: 150–200 lines. The pinned section list:

```markdown
# <Project Name>

<One-paragraph project description: what the system does, who its users are.
Link to the authoritative product/design spec.>

**Authoritative design doc:** `docs/<spec-filename>.md`
**GRACE artifacts:** `docs/{requirements,technology,development-plan,verification-plan,knowledge-graph,operational-packets}.xml`

## Tech Stack

- **Backend:** <list>
- **Frontend:** <list>
- **Data:** <list>
- **Infra:** <list>
- **Testing:** <runners> (+ GRACE LDD log assertions via `<your-pkg>.grace.testing.GraceLogCapture`)

## Module Map

| Directory | Module Tag | GRACE ID | Purpose |
|---|---|---|---|
| `<path>/` | `[<short>]` | M-<NAME> | <one-line purpose> |
<!-- one row per module -->

## Cross-Cutting Constraints

These invariants apply to ALL modules. See <spec-anchor> for full definitions.

- **INV-NNN — <short name>:** <one-line constraint>
<!-- one bullet per invariant -->

## State Machines

State machines are defined in <spec-anchor>. Reference them by section:

- §X.1 — <Machine Name> (<state list>)
<!-- one bullet per machine -->

## Development Methodology: GRACE

This project uses GRACE (Graph-RAG Anchored Code Engineering). Full reference: the
`grace:grace-explainer` skill plus the artifacts under `docs/`.

### Substrate (in-source)

<3-5 lines: every public function/class carries a contract; mirror the canonical
example file; declarative classes get module-only contracts.>

### Log-Driven Development (LDD)

<3-5 lines: state-machine transitions emit `[Module][fn][BLOCK]` markers and BELIEF/ACTUAL/STATUS.
Use `<your-pkg>.grace.logging.get_grace_logger`. Required markers per module live in
`docs/verification-plan.xml`.>

### Verification

<3-5 lines: tests opt into LDD assertions with the `grace_logs` fixture. Old RED/GREEN
discipline retired (if applicable). Existing pytest assertions still work.>

### Workflow skills (from the `grace` plugin)

| Skill | When |
|---|---|
| `grace:grace-plan` | Designing a new module / phase / data flow before code |
| `grace:grace-execute` | Sequential implementation with controller-managed packets |
| `grace:grace-multiagent-execute` | Parallel implementation waves (use carefully) |
| `grace:grace-verification` | Adding tests / log markers / scenarios |
| `grace:grace-refactor` | Renaming / moving / splitting modules |
| `grace:grace-fix` | Debugging via graph navigation |
| `grace:grace-refresh` | Sync GRACE artifacts with code after changes |
| `grace:grace-status` | Health check + suggested next action |
| `grace:grace-ask` | Q&A grounded in project artifacts |
| `grace:grace-reviewer` | Pre-merge integrity review |

## Environment Workarounds

<Project-specific oddities the agent must know up-front: broken Pythons, port collisions,
SSL quirks, Docker-vs-host differences. Each one in its own subsection.>

## General Rules

- <One-line cross-cutting style rules (timestamps in UTC, prices as integers in subunits, bilingual fields, etc.)>
- New code is GRACE-substrate from the start: contracts before functions, LDD markers at state transitions, paired START/END markup.
```

Section rationale — why each one earns its place at the root:

- **Module Map** — the agent's table of contents. Without it, the agent guesses module locations from filename heuristics, which fails on monorepo layouts.
- **Cross-Cutting Constraints** — invariants that are everyone's problem. Putting them in the root once means every module's local file can reference them by ID instead of restating them.
- **State Machines** — the catalog. Per-machine detail lives in the canonical spec; the root just enumerates them so an agent reading "transitioning Order to PAID" knows where the rules live.
- **Workflow skills table** — the menu of available `grace-*` skills. Without this, the agent picks generic tools when a specialized one exists.
- **Environment Workarounds** — the section that turns "this code does not run on my machine" into "this code does not run on my machine because of X, fix is Y." Examples: a broken stdlib in a system Python, container vs. host port mismatches, SSL libraries missing in a CI image.

## Module `AGENTS.md` template

The module file is the local brief. Target: 50–80 lines. Smaller is better.

```markdown
# <Module Name>

<One-paragraph module description: what this module owns, who its callers are.>

**<Spec> sections:** <comma-separated section refs the module owns>

## Tech Stack

- <subset of the root tech stack relevant to this module>

## Scope

This module is responsible for:
- <bullet>
- <bullet>

## Constraints

- **<Constraint name>:** <one-line constraint specific to this module>
<!-- module-specific rules go here; reference cross-cutting INVs by ID without restating -->

## Key Files

- `<path>` — <one-line purpose>
<!-- the 3-7 files an agent should know about before editing this module -->

## Testing

- **Framework:** <runner>
- **Canonical runner:** `<exact command line>`
- **Test files:** `<naming convention>`
- **Fixtures:** `<grace_logs / DB session / Redis fake / etc.>`
- **Methodology:** GRACE — pytest plus optional LDD log assertions via the `grace_logs` fixture. Required markers per `docs/verification-plan.xml` V-M-<NAME>.

## This Module MUST NOT

- <one-line negative rule>
- <one-line negative rule>
```

Section rationale:

- **Scope** — the affirmative case. What this module is responsible for. Specific enough to disambiguate it from siblings.
- **Constraints** — module-specific rules. Latency SLAs, write-only / read-only restrictions, dependency direction.
- **Key Files** — the 3-7 files most likely to be touched. NOT a directory listing; just the load-bearing entry points.
- **Testing** — the exact `pytest` / `vitest` command, the fixture wiring, the convention for test-file placement. Saves the agent one round-trip per task.
- **This Module MUST NOT** — the negative space. Reading the affirmative scope tells the agent what to do; the MUST NOT section tells the agent what is *deliberately* not its job. Particularly load-bearing in monorepos where similar work happens in multiple modules.

## Negative-space sections — why they earn their length

The "MUST NOT" pattern recurs because it carries information that affirmative descriptions cannot:

- "This module MUST NOT call external payment APIs directly — delegate to the payment worker." A reader of the affirmative scope sees "handles checkout" and wonders if a YuKassa call belongs here. The negative rule answers in one line.
- "This module MUST NOT import from core-api, payment-worker, or sms-worker (dependency flows one way: services → shared)." This is dependency-direction discipline; the agent's pull is to do whatever import works, and the rule pre-empts that pull.
- "This module MUST NOT serve static frontend files in production (Nginx handles this)." Carves out an obvious-looking responsibility that actually belongs to infrastructure.

A good MUST NOT section is 3–6 bullets. Longer means the module is over-scoped and probably wants to be split. Empty means either the module's responsibilities are crystal-clear (rare) or the negative space has not been articulated yet (common).

## Adoption guidance

Adding the nested pattern to a project that does not have it.

### Greenfield

Start at module-creation time. Every `grace:grace-plan` for a new module produces both:

- An `<M-...>` block in `development-plan.xml` with full contract.
- An `<module>/AGENTS.md` from the template above.

The two are companions: the XML carries machine-readable contract; the markdown carries human/agent navigation. Drift between them is what `grace:grace-refresh` catches.

### Brownfield

Two phases:

1. **Bootstrap (during CP1 of a GRACE migration).** Leave the existing root `AGENTS.md` (or whatever the project's onboarding file is called) alone. Substrate XMLs land first.
2. **Sync (during CP7).** Rewrite root `AGENTS.md` to match the template above, replacing whatever predecessor-methodology section it had with the GRACE methodology section. Surgical-edit each module `AGENTS.md` to add a "Methodology: GRACE" line under Testing and to remove obsolete RED/GREEN or spec-driven-workflow lines.

Surgical edits beat full rewrites at CP7. Module `AGENTS.md` files often contain hard-won project-specific knowledge (the broken Python interpreter, the Redis fixture quirk, the CORS dance for multi-worktree dev). Preserving that content while replacing the methodology section is the cheapest path forward.

### When NOT to nest

Three cases where flat is fine:

- **Tiny project** (one module). Nest is over-engineering; one root `AGENTS.md` is enough.
- **Generated / vendored module** that should not be hand-edited. Adding `AGENTS.md` invites edits to a directory whose content is rebuilt on every codegen run.
- **Test-fixture or migration directory.** These are utilities, not modules. The root file's references are sufficient.

The threshold for nesting: the project has at least 3 distinct domains (e.g., 2 services + 1 frontend), and at least one module has > 5 module-specific rules a reader needs to know.

## Maintenance

Two practices keep the nested files true.

1. **Treat module `AGENTS.md` as substrate.** It MUST be updated in the same commit that changes the module's scope, constraints, or fixtures. A grep for "Scope" or "MUST NOT" in the staged diff flags the file as needing review.
2. **`grace-refresh` reads the nested files.** When the sync skill compares substrate to code, it should also confirm: every module in `development-plan.xml` has an `AGENTS.md` at its declared path; root `AGENTS.md` Module Map row exists for every `<M-...>`.

Both practices are cheap; neither requires custom tooling beyond what `grace-refresh` already does.

## Worked example — 7-module monorepo

A Python/FastAPI + React project at GRACE adoption time:

- `AGENTS.md` (root) — 178 lines.
- `services/core-api/AGENTS.md` — 82 lines.
- `services/payment-worker/AGENTS.md` — 51 lines.
- `services/sms-worker/AGENTS.md` — 65 lines.
- `web/customer/AGENTS.md` — 54 lines.
- `web/admin/AGENTS.md` — 50 lines.
- `packages/shared/AGENTS.md` — 49 lines.
- `database/AGENTS.md` — 77 lines.

Total: 8 files, 606 lines. The root is the largest single file because it carries the cross-cutting content (module map, INV catalog, state-machine list, methodology section). Module files average ~62 lines each.

Properties:

- The root is read at the start of every session (~178 lines load).
- A module file is read when the agent enters the module's source path (~60 lines load on top of the root).
- Cross-cutting constraints are referenced by ID (`INV-002`, `INV-013`) in module files; the canonical body lives only in root.
- Each module file ends with a "MUST NOT" section that articulates 2–4 module-specific exclusions.
- Every module file points at the test runner command line for that module — saves one shell round-trip.

Adoption cost: at CP1 of the GRACE migration the root file already existed (predecessor-methodology content). At CP7, it was rewritten in place; module files received surgical edits that replaced the predecessor's TDD-section bullets with GRACE pointers. Total wall-clock: under one focused agent session.

## See also

- `./brownfield-adoption.md` — when in the migration sequence the nested files are populated (CP7).
- `./top-down-pipeline.md` — the wider project-shape pattern this nesting fits inside.
- `../04-markup-system/canonical-example-python.md` — the in-source contract format that AGENTS.md sits next to.
- `../04-markup-system/substrate-xml-schemas.md` — the machine-readable counterpart of the human-readable AGENTS.md tier.
- `../03-contracts/contracts-overview.md` — adjacency principle that motivates per-module navigation files.
- `../09-tooling/cli-not-custom-mcp.md` — why adjacency-based navigation beats custom-MCP-based navigation.
- `../11-case-studies/` — production projects that ship this pattern.
