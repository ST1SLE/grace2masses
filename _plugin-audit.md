# osovv Plugin ↔ KB Coverage Audit

Compiled 2026-04-23 during the decision between adopting `osovv/grace-marketplace` v3.10.0 as-is, supplementing it, or building a custom layer.

**Chosen path: plugin + thin supplement** — adopt all 14 osovv skills (symlinked into `.claude/skills/` for portability) and add 2 custom skills filling brownfield-Python-specific HARD gaps.

This audit is decision-focused: the MISSING bucket is enumerated in full, PARTIAL and COVERED are summarized. It is a snapshot of plugin v3.10.0; re-audit when the plugin version changes.

## Summary

| Category   | Rough count | Meaning                                                         |
|:-----------|-------:|:----------------------------------------------------------------|
| COVERED    | ~24    | Plugin skill implements the concept with matching semantics     |
| PARTIAL    | ~25    | Plugin skill touches the concept but misses nuance or subrule   |
| MISSING    | 14     | Concept not implemented; requires custom skill or manual work   |
| DIVERGENT  | 1      | Documented for awareness; no action needed                      |

The rough counts are from the full sweep conducted 2026-04-23 (52 load-bearing operational concepts across the 14 KB chapters, mapped against the 14 plugin skills). The 14 MISSING and 1 DIVERGENT items are enumerated exactly below; PARTIAL and COVERED are summarized because the MISSING bucket is the actionable surface.

## MISSING — the 14 gaps

Split into three buckets by how the supplement layer responds.

### Bucket A — filled by new skill `grace-legacy-contracts`

1. **AST-free legacy contract bootstrap via START-END scraping** — `kb/03-contracts/legacy-contract-recovery.md` (source post 3622). Plugin assumes greenfield: `grace-init` and `grace-plan` generate `docs/*.xml` from requirements, not from existing code. No path exists for "the code is already here, contract it now."
2. **Wenyan compression of contract-field content** — `kb/03-contracts/wenyan-prompting.md`. Plugin's contract text is verbose English with no compression lint. The KB reports ~1:15 compression target and 90 % of Cursor-style semantic hits landing on KEYWORDS / LINKS enrichment points when compression is applied.
3. **PURPOSE English-only enforcement** — `kb/04-markup-system/markup-compression.md`. Plugin does not validate language; the KB's English-only rule is about stabilizing the retrieval surface across agents trained predominantly on English tokens.
4. **Python placement rule: contract as `#` comments flush above `def`, no blank line** — `kb/03-contracts/language-specific/python.md`. Plugin stores contracts in XML planning artifacts (`docs/knowledge-graph.xml` et al.), not in-source. The KB's placement rule is load-bearing for sparse-attention anchoring near the `def` token.
5. **Full five-field function contract (PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS)** — `kb/03-contracts/contract-fields.md`. Plugin's MODULE_CONTRACT covers four fields at module level (PURPOSE / SCOPE / DEPENDS / LINKS); there is no equivalent at function level, and INPUTS / OUTPUTS / KEYWORDS are missing even at module level.

### Bucket B — filled by new skill `grace-ldd-retrofit`

6. **`[Module][function][BLOCK]` correlation-token prefix on every log line** — `kb/05-logging-ldd/log-to-code-correlation.md`. Plugin's `grace-plan` mentions "log or trace anchors" abstractly; no concrete prefix convention, no retrofit path.
7. **Belief-state log lines at critical branches** — `kb/05-logging-ldd/belief-state-logging.md`. Not referenced in any plugin skill.
8. **Forced-context injection markers on critical log lines** — `kb/05-logging-ldd/forced-context-injection.md`. Not referenced; forced-context as an orchestration concept exists but the log-line-level marker convention is absent.
9. **LDD as a trajectory-reading philosophy over assertion-based testing** — `kb/05-logging-ldd/log-driven-development.md`. Plugin's verification-ref + test artifacts align with LDD intent but do not implement the log-trajectory-reading workflow — you can have complete plugin coverage and still be stuck asserting equality instead of reading trajectories.

### Bucket C — deferred (activate when swarm scenarios demand them)

Per the 2026-04-23 use-case decision (brownfield Python, solo, task-dependent swarm usage), these five concepts are not blocking and are postponed.

10. **Anti-loop attempt counters + context injection for runaway agents** — `kb/07-autonomous-agents/anti-loop-protection.md`. Matters when swarm iterations burn tokens; a solo single agent self-notices loops through a shorter feedback cycle.
11. **Regression-KB wired into the test harness** — `kb/07-autonomous-agents/knowledge-base-in-tests.md`. Matters at swarm scale; solo projects accumulate less regression memory in shorter timescales.
12. **Forced-context injection at the agent-orchestration layer** — `kb/07-autonomous-agents/forced-context.md` (distinct from the log-line-level marker in Bucket B #8). Orchestration-level; solo single-agent covers it through the session context window.
13. **Swarm-vs-single-agent trade-off explicit sizing** — `kb/07-autonomous-agents/`. No swarm today → no sizing decision.
14. **Hybrid graph-plus-vector RAG store (sqlite-vec)** — `kb/09-tooling/` (hybrid-rag-storage entry). Plugin's `docs/knowledge-graph.xml` + `grace-ask` cover retrieval adequately at project sizes we're targeting. Upgrade path if the graph gets too big for pure-XML scans.

## DIVERGENT

- **Classical assertion-based TDD.** The KB argues it actively hurts AI agents (`kb/06-testing/classical-tests-antipattern.md`). The plugin neither enforces nor forbids classical TDD; its `grace-verification` produces test artifacts that can be read either way. Not a conflict in practice — flagged so future custom skills do not inadvertently bake in classical TDD discipline.

## COVERED (load-bearing subset)

The plugin implements these KB-load-bearing concepts cleanly:

- Semantic graph as persistent scaffolding → `grace-init` / `grace-plan` / `grace-refresh` produce and maintain `docs/knowledge-graph.xml`.
- Three-phase GRACE workflow (Architectural graph → Module contracts → Function contracts and code) → `grace-init` → `grace-plan` → `grace-execute`.
- XML artifact scaffold with six canonical files (requirements, technology, development-plan, verification-plan, knowledge-graph, operational-packets) → generated by `grace-init`.
- ID schemes (`M-xxx`, `V-M-xxx`, `DF-xxx`, `Phase-N`, `step-N`) → codified in `grace-plan`.
- Module taxonomy (ENTRY_POINT / CORE_LOGIC / DATA_LAYER / UI_COMPONENT / UTILITY / INTEGRATION) → codified in `grace-plan`.
- Verification-aware planning with critical flows and log-trace evidence hooks → `grace-plan` + `grace-verification`.
- Reviewer ↔ fixer ↔ refresher loop → `grace-reviewer` / `grace-fix` / `grace-refresh`.
- Parallel-safe execution waves → `grace-multiagent-execute`.
- Project-health inspection → `grace-status`.
- Lint profile for execution readiness → `grace lint --profile autonomous` (via `@osovv/grace-cli`).
- CLI + marketplace distribution → `@osovv/grace-cli`.
- Skill-based distribution (vs bespoke-MCP) → the plugin itself is the Claude Code-native counter-example to the "kulibin MCP" antipattern.
- Contract-first design (no code until MODULE_CONTRACTs exist) → enforced in `grace-plan`.

(The full 24-item COVERED list abbreviated here; the above is the practically load-bearing subset.)

## PARTIAL (summary)

PARTIAL coverage mostly clusters where the plugin uses a mechanism without enforcing the KB's stricter rule:

- Function-level contract fields (PARTIAL at module level → MISSING at function level — see Bucket A #5).
- Wenyan compression (PARTIAL at artifact level → MISSING at in-source level — Bucket A #2).
- Dyck-language XML preference (PARTIAL — the plugin uses XML artifacts, but does not enforce Dyck compatibility beyond syntactic XML validity).
- LLM-training-pipeline rationale (PARTIAL — described in `grace-explainer`'s text, not operationalized as a constraint on skill prompts).
- Log-anchor conventions (PARTIAL at plan level via `grace-plan`'s "log or trace anchors" language → MISSING at retrofit level — Bucket B #6–#8).

## Decision record

- **Path chosen**: B — plugin + thin supplement.
- **Custom skills added**: `grace-legacy-contracts` + `grace-ldd-retrofit`.
- **Install layout**: `.claude/skills/` project-local. All 14 osovv skills relative-symlinked in from `../../grace-marketplace/plugins/grace/skills/grace/`. 2 custom skills written directly into `.claude/skills/grace-legacy-contracts/` and `.claude/skills/grace-ldd-retrofit/`.
- **Portability**: project is not a git repo; moved as a unit via tar / rsync / zip. Symlinks are relative, so moving the project dir keeps them resolvable as long as `grace-marketplace/` travels alongside `.claude/`.
- **Rationale**: Plugin covers approximately 60 % of the KB's load-bearing operational content; the remaining 40 % splits between out-of-scope content (case studies, economics), soft-educational content (transformer-internals rationale), and HARD gaps. Buckets A + B are the HARD-gap set that specifically matters for brownfield Python with poor logging (our stated use case). Bucket C is deferred until swarm scenarios demand them.
- **Source-of-truth rules**:
  - Rationale: `kb/` — cite specific files and source post numbers.
  - Already-implemented operational contracts: `grace-marketplace/plugins/grace/skills/grace/*/SKILL.md` — do not duplicate.
  - Brownfield-Python HARD-gap fills: `.claude/skills/grace-legacy-contracts/SKILL.md` + `.claude/skills/grace-ldd-retrofit/SKILL.md`.

## See also

- [`09-tooling/claude-code-grace-plugin.md`](09-tooling/claude-code-grace-plugin.md) — plugin entry in the KB.
- [`README.md`](README.md) — "Applying GRACE with Claude Code" section.
