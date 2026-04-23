# Claude Code GRACE Plugin (osovv/grace-marketplace)

## What

A third-party **Claude Code plugin that implements GRACE ideas**, authored by **Alexey Chendemerov** ("Алексей Чендемеров" in the source) and published on GitHub at **`github.com/osovv/grace-marketplace`**. [author-claim] The plugin is external to the GRACE author's own toolchain — the author flags that he himself **normally implements GRACE without plugins, using prompt frameworks instead**. (practical_applications.md §"Плагин для Claude Code")

The post is short and serves one narrow purpose in the corpus: it is **the only explicit GRACE endorsement of a Claude-Code-native plugin in the sources**, and it pins down which GRACE elements the author treats as the plugin's load-bearing core. (practical_applications.md §"Плагин для Claude Code")

What the author publicly signs off on — and what he explicitly does not — is the entire substance of this entry. The sources do not specify any further technical detail about the plugin beyond the three confirmed core ideas and the GitHub link.

## Why

### Why this entry exists at all

GRACE as the author practices it is **prompt-framework-shaped**, not plugin-shaped. [author-claim] The exact source line is: "Обычно я реализую методологию без плагинов фреймворками промптов." — i.e. "I usually implement the methodology *without* plugins, with prompt frameworks." (practical_applications.md §"Плагин для Claude Code")

That framing matters because it tells the reader two things at once:

1. **Plugins are not the canonical GRACE delivery vehicle.** [author-claim] The default delivery vehicle is a body of prompts layered into an agent workflow (e.g., Kilo Code / Open Code, Cursor, Gemini CLI — see T40, T42). (practical_applications.md §"Плагин для Claude Code"; practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code" for the prompt-framework-in-IDE baseline.)
2. **A plugin implementation is nonetheless possible, and one such plugin passes the author's core-ideas check.** [author-claim] The existence of osovv/grace-marketplace is the author's proof-by-example that the methodology can be crystallized into a Claude-Code-native package by a third party. (practical_applications.md §"Плагин для Claude Code")

### Why the endorsement is narrow (and why that matters)

The author is explicit about the **scope of his validation**:

- **[author-claim]** "Реализацию внимательно не проверял" — "I did not carefully check the implementation." (practical_applications.md §"Плагин для Claude Code")
- **[author-claim]** "но основные идеи правильные" — "but the core ideas are right." (practical_applications.md §"Плагин для Claude Code")

Practically this means the post is **not a code audit and not a functional certification**. It is specifically an endorsement of the plugin's **conceptual alignment** with GRACE's core machinery. A reader who adopts the plugin for production should treat it the way the author did: verify the implementation themselves.

This narrow framing is consistent with how GRACE endorses tooling elsewhere in the corpus:

- In T42 (Kilo Code / Open Code migration), the author endorses a particular agent core after hands-on integration work of his own. [author-claim]
- Here, he endorses only the **ideas** the plugin is built around — not the code that realizes them. [author-claim]

The distinction is the whole point of listing this plugin in the KB: the reader is being told what the plugin *claims to be*, not that it has been shown to work at GRACE-scale projects.

### Why the three named elements are the ones checked

The author names exactly three GRACE elements he verified as "correctly present" in the plugin's design: **graph over code, START-END markup, contracts**. (practical_applications.md §"Плагин для Claude Code") That set is not arbitrary — it maps directly onto what GRACE treats as its three irreducible primitives elsewhere in the KB:

- **Graph over code** → the semantic scaffold (see T10 `graph-as-backbone.md`). [author-claim]
- **START-END markup** → Dyck-compatible paired-tag markup (see T07 `tc0-and-dyck-language.md` and T22 `start-end-tags.md`). [author-claim]
- **Contracts** → the contract-first design layer (see T13 `contracts-overview.md`). [author-claim]

So the endorsement can be read as a **minimal conformance test**: a plugin that gets those three right is at least structurally GRACE-shaped, regardless of whatever else it does.

## How

### What the source actually states

The full post is short enough to reproduce its load-bearing claims verbatim. [author-claim] The content elements relevant to this KB entry are:

1. **Who built it**: Alexey Chendemerov. (practical_applications.md §"Плагин для Claude Code")
2. **Based on what**: "по моей методологии GRACE" — "based on my GRACE methodology." (practical_applications.md §"Плагин для Claude Code")
3. **Target host**: Claude Code. (practical_applications.md §"Плагин для Claude Code")
4. **Audit status**: "Реализацию внимательно не проверял" — implementation not carefully audited. (practical_applications.md §"Плагин для Claude Code")
5. **Ideas confirmed right**: "граф на код, START-END разметки и контракты" — "graph over code, START-END markup, and contracts." (practical_applications.md §"Плагин для Claude Code")
6. **Author's own default**: "Обычно я реализую методологию без плагинов фреймворками промптов." — normally uses prompt frameworks, not plugins. (practical_applications.md §"Плагин для Claude Code")
7. **Reference**: `https://github.com/osovv/grace-marketplace`. (practical_applications.md §"Плагин для Claude Code")

That is the entirety of the source material on this plugin in this corpus.

### What the reader should take away

**Treat this as a conformance pointer, not a deployment recommendation.** [author-claim] The source endorses the plugin's conceptual alignment with GRACE, not its readiness. Adoption should be a two-step process:

1. **Conformance check on the ideas.** Verify that the plugin actually implements, at the code level, the three items the author named as right (graph over code, START-END markup, contracts). The author did not do this check on behalf of the reader — he only checked that the plugin *claims* to implement them. (practical_applications.md §"Плагин для Claude Code")
2. **Implementation quality check.** Everything past the "core ideas are right" statement is the reader's responsibility. The sources do not specify test coverage, stability, maintenance cadence, or completeness of the plugin's feature set.

### Plugin vs. prompt-framework: when to prefer which

The author's default is **prompt framework over plugin**. [author-claim] (practical_applications.md §"Плагин для Claude Code") The sources do not spell out an exhaustive decision matrix, but the surrounding posts make the implicit argument:

- **Prompt frameworks** (the author's default) are portable across agent cores — the same prompts can target Kilo Code / Open Code, Cursor, Gemini CLI, Claude Code. (practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code" implies this portability by describing how the author moved "all the best from prompting for Gemini CLI and Cursor" into Kilo Code.)
- **Plugins** are host-specific: the plugin under discussion here is tied specifically to Claude Code. [author-claim] (practical_applications.md §"Плагин для Claude Code")

So reaching for a plugin is reasonable when the team has committed to a specific agent host (in this case Claude Code) and wants packaged delivery; reaching for a prompt framework is the portable default. The sources do not specify trade-offs beyond this general framing.

### What the plugin is *not* claimed to do

The source does **not** claim any of the following:

- that the plugin is the author's official or endorsed reference implementation — it is explicitly a third-party artifact;
- that the plugin has been tested at the 100k–300k LOC project scale discussed elsewhere (T52 `alexey-300k-loc.md`, T50 `kirill-132k-loc.md`);
- that the plugin covers the full GRACE methodology beyond the three named primitives — e.g. LDD logging (T26–T29), anti-loop protection (T35), hybrid RAG (T47);
- that it is production-ready, maintained, or benchmarked.

For any of those claims the sources do not specify; readers who need those guarantees will need to evaluate the plugin themselves. (practical_applications.md §"Плагин для Claude Code")

### What to cross-check if adopting the plugin

The following KB entries collectively define what a "correct" implementation of the three confirmed-right elements looks like according to GRACE. A plugin-level audit should map plugin behaviour onto each of these:

- **Graph over code** → see T10 `graph-as-backbone.md`; the graph should be an **explicit semantic scaffold**, not a latent fallback reconstructed by the model. [author-claim] (cross-ref, this KB)
- **START-END markup** → see T22 `start-end-tags.md`; the tags should be **paired, Dyck-compatible, AST-free parseable**, and usable across the plugin's code-reading path. [research-backed for the Dyck basis via T07; author-claim for the AST-free requirement] (cross-ref, this KB)
- **Contracts** → see T13 `contracts-overview.md` and T15 `contract-fields.md`; contracts should be placed **inside code** (not in external docs), use the canonical field set (PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS), and sit before the declaration — see T19 `python.md`. [author-claim + research-backed per T15 sources] (cross-ref, this KB)

If the plugin deviates on any of these, the "core ideas are right" endorsement does not transfer automatically — the author's confirmation is of the ideas, not of any particular implementation choice underneath them. [author-claim] (practical_applications.md §"Плагин для Claude Code")

### Upstream snapshot (directly inspected; not from source posts)

> Facts in this subsection come from **direct inspection of the upstream repository** cloned locally at `../../grace-marketplace/` (sibling of `kb/`, cloned 2026-04-23). They are separate from the author's endorsement above, which predates v3.10.0 and was of the plugin's **ideas**, not any specific version's implementation. Treat this subsection as a vendor-doc summary of the current package contents. [vendor-doc]

**Repository**: `https://github.com/osovv/grace-marketplace` · **Version**: v3.10.0 · **License**: MIT · **Author**: osovv (Alexey Chendemerov per source post).

**Install via Claude Code marketplace**:

```
/plugin marketplace add osovv/grace-marketplace
/plugin install grace@grace-marketplace
```

**Install via OpenPackage**:

```
opkg install gh@osovv/grace-marketplace
```

**Install via git (Agent-Skills-compatible hosts)**:

```
git clone https://github.com/osovv/grace-marketplace
cp -r grace-marketplace/skills/grace/grace-* /path/to/agent/skills/
```

**Optional CLI** (bun runtime, used by the `grace-cli` skill):

```
bun add -g @osovv/grace-cli
grace lint --path /path/to/project
```

#### Skills included (14, per `.claude-plugin/marketplace.json`)

Each skill is a folder under `skills/grace/` with a `SKILL.md` carrying YAML frontmatter (`name`, `description`). The Claude Code harness auto-loads a skill when its description matches user intent; none of these are slash commands. Descriptions below are abridged from each skill's frontmatter. [vendor-doc]

| Skill | When it triggers |
|---|---|
| `grace-init` | Bootstrap GRACE structure in a new project — creates `docs/`, `AGENTS.md`, and XML templates. |
| `grace-plan` | Architectural planning after `requirements.xml` and `technology.xml` exist — produces `development-plan.xml`, `verification-plan.xml`, `knowledge-graph.xml`. |
| `grace-execute` | Sequential execution of the development plan with controller-managed context packets, scoped reviews, and level-based verification. |
| `grace-multiagent-execute` | Parallel-safe wave execution with synchronization, batched shared-artifact sync, and selectable safety profiles. |
| `grace-verification` | Design and enforce tests, execution traces, and log-driven verification; maintain `verification-plan.xml`. |
| `grace-reviewer` | Scoped gate reviews during execution; autonomy-readiness preflights; full integrity audits at phase boundaries. |
| `grace-fix` | Debug via semantic navigation of graph / verification plan / semantic blocks, then targeted fix. |
| `grace-refresh` | Sync shared artifacts with codebase after waves, refactors, or suspected drift. |
| `grace-refactor` | Rename, move, split, merge, or extract modules while keeping contracts / graph / verification / markup synchronized. |
| `grace-status` | Overview of artifacts, codebase metrics, graph health, verification coverage, next actions. |
| `grace-ask` | Q&A grounded in all GRACE artifacts; navigates the knowledge graph; cites sources. |
| `grace-explainer` | Methodology reference — explains GRACE principles, markup, graphs, contracts, testing, tag conventions. |
| `grace-setup-subagents` | Scaffold worker/reviewer agent files for Claude Code / OpenCode / Codex / other shells. |
| `grace-cli` | Operate the optional `grace` CLI (`lint`, `status`, `module`, `verification`, `file show`). |

#### XML artifacts the plugin scaffolds under the target project's `docs/`

Confirmed against `skills/grace/grace-init/SKILL.md`:

- `docs/requirements.xml` — product intent and scope.
- `docs/technology.xml` — stack and tooling choices.
- `docs/development-plan.xml` — modules and implementation order.
- `docs/verification-plan.xml` — tests and verification scenarios.
- `docs/knowledge-graph.xml` — module map with `M-xxx` IDs and CrossLinks.
- `docs/operational-packets.xml` — execution templates.

This XML-artifact scaffold sits **on top of** the inline `# PURPOSE: / # INPUTS: / # OUTPUTS: / # KEYWORDS: / # LINKS:` contract pattern documented from the source posts (see `../03-contracts/contract-fields.md` [T15]). It is the plugin's opinionated interpretation of GRACE, not a replacement for the inline-contract pattern. [vendor-doc]

#### Module taxonomy used by `grace-plan`

`ENTRY_POINT / CORE_LOGIC / DATA_LAYER / UI_COMPONENT / UTILITY / INTEGRATION` — from `skills/grace/grace-plan/SKILL.md`. [vendor-doc] The source posts do not define this six-way taxonomy; it is plugin-specific.

#### Where the plugin's scope exceeds the author's audit

The author's endorsement (see §Why above) covered three primitives: **graph over code**, **START-END markup**, **contracts**. The plugin at v3.10.0 additionally ships: the six-artifact XML scaffold; module / verification ID schemes (`M-xxx`, `V-M-xxx`); parallel-wave multi-agent execution; a lint CLI; drift-detection skills; subagent scaffolding for non-Claude-Code shells. These additions **inherit no endorsement from the source posts** — treat them on their own merits. [author-claim]

#### Local clone

Cloned at project-relative path: `../../grace-marketplace/` (sibling of `kb/`, cloned 2026-04-23). Read-only reference — the canonical source is `https://github.com/osovv/grace-marketplace`; `git pull` for updates.

### Local supplement for brownfield Python (this KB's deployment, not upstream)

> Facts in this subsection describe how this specific KB's deployment extends the plugin for brownfield-Python use. They are **not from source posts**, and are **not endorsed by the plugin author** — they are specific to this project's layout. Treat as a deployment reference, not a generic recommendation. [kb-local]

The plugin assumes **greenfield**: `grace-init` → `grace-plan` → `grace-execute` walks from `requirements.xml` forward to code. Two GRACE concepts that are load-bearing for **brownfield** (existing code, poorly-logged) Python use are not implemented by the plugin at the in-source level:

- **Function-level contract retrofit** — placing `# PURPOSE / # INPUTS / # OUTPUTS / # KEYWORDS / # LINKS` comment blocks flush above existing `def`s per `../03-contracts/language-specific/python.md` [T19], using Anton's AST-free START-END scraping pattern per `../03-contracts/legacy-contract-recovery.md` [T16, source post 3622].
- **LDD correlation-token retrofit** — injecting `[ModuleName][function_name][BLOCK_NAME]` prefixes on existing `logger.*` calls, belief-state lines at critical branches, and `@forced-context` markers on load-bearing lines per `../05-logging-ldd/log-to-code-correlation.md` [T26], `../05-logging-ldd/belief-state-logging.md` [T27], `../05-logging-ldd/forced-context-injection.md` [T28].

This KB's deployment adds two project-local Claude Code skills covering those gaps at `.claude/skills/` (sibling of `kb/`):

| Skill | Location | Purpose |
|---|---|---|
| `grace-legacy-contracts` | `../../.claude/skills/grace-legacy-contracts/SKILL.md` | Scan brownfield Python; synthesize the 5-field contract block; lint wenyan compression + English-only PURPOSE + flush-above-def placement; edit-in-place with per-file confirmation. |
| `grace-ldd-retrofit` | `../../.claude/skills/grace-ldd-retrofit/SKILL.md` | Audit existing logging; inject correlation tokens + belief-state + `@forced-context`; stop-and-ask if target uses `print` / bespoke logger instead of `logging.getLogger()`. |

The same `.claude/skills/` directory holds relative symlinks to each of the 14 osovv skills under `../../grace-marketplace/plugins/grace/skills/grace/`, so the full skill set (14 plugin + 2 supplement, 16 total) travels with the project tree without requiring `/plugin install` on target machines.

The gap analysis that motivated these two skills — 52 KB concepts mapped against the 14 plugin skills, yielding ~24 COVERED / ~25 PARTIAL / 14 MISSING / 1 DIVERGENT — is persisted at `../_plugin-audit.md`. The 14 MISSING gaps split into **Bucket A** (filled by `grace-legacy-contracts`), **Bucket B** (filled by `grace-ldd-retrofit`), and **Bucket C** (deferred until swarm-scale usage demands them). [kb-local]

## Evidence

### Primary source

- **practical_applications.md §"Плагин для Claude Code"** — the entire short post. Key lines reproduced:
  - "Алексей Чендемеров по моей методологии GRACE сделал плагин для Claude Code." [author-claim]
  - "Реализацию внимательно не проверял, но основные идеи правильные как граф на код, START-END разметки и контракты." [author-claim]
  - "Обычно я реализую методологию без плагинов фреймворками промптов." [author-claim]
  - Reference URL: `https://github.com/osovv/grace-marketplace`. [author-claim, as URL source]

### What the corpus does and does not say about the plugin

**Said:**
- Who built it (Alexey Chendemerov). [author-claim]
- Host agent (Claude Code). [author-claim]
- Three conceptually correct elements (graph over code, START-END markup, contracts). [author-claim]
- GitHub repository URL. [author-claim]
- The author's own default (prompt frameworks, not plugins). [author-claim]
- The scope of the author's check (ideas, not implementation). [author-claim]

**Not said (the sources do not specify):**
- internal architecture of the plugin;
- the plugin's dependency set, license, or packaging;
- whether the plugin supports LDD logging, hybrid RAG, anti-loop protection, or any GRACE element beyond the three named primitives;
- case studies or LOC-scale evidence that the plugin has been used on large codebases;
- maintenance status, issue backlog, or test coverage;
- whether the plugin is distributed as a Claude Code marketplace entry, a standalone repo, or both (the URL `osovv/grace-marketplace` suggests marketplace-style distribution but the post does not elaborate);
- compatibility with other agent cores (Kilo Code / Open Code, Cursor, Gemini CLI);
- performance or token-efficiency claims.

### Supporting cross-references in the corpus

These are **context-setting**, not claims about the plugin itself:

- **practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"** — establishes that the author's default GRACE delivery vehicle is a **prompt framework layered into an agent IDE**, with Kilo Code / Open Code as his current choice. [author-claim] Used here only to make the "plugins are not canonical" point in §Why concrete. See T40 (`architect-code-debug-modes.md`) and T42 (`kilo-code-open-code.md`) for full treatment.
- **practical_applications.md §"Пример CRM"** — Vlad's case study of LLM-interchangeability under GRACE markup. [author-claim] Used here only to reinforce that the author's usual lens is **markup + specs as the portable layer** — which is why the three conceptual pillars (graph / markup / contracts) are what he checks when endorsing any third-party implementation. See T51 (`vlad-crm-and-contracts.md`).

Neither cross-reference speaks to osovv/grace-marketplace specifically; both are included only because they explain why the author's endorsement is shaped the way it is.

### Direct upstream inspection (2026-04-23 clone)

Independent of the primary source post, the following facts come from reading the upstream repository at `github.com/osovv/grace-marketplace` (cloned locally at `../../grace-marketplace/`):

- **Marketplace manifest** `.claude-plugin/marketplace.json` — v3.10.0, declares 14 skills, schema `anthropic.com/claude-code/marketplace.schema.json`. [vendor-doc]
- **Plugin manifest** `plugins/grace/.claude-plugin/plugin.json` — v3.10.0, MIT licensed. [vendor-doc]
- **Repository CLAUDE.md** — states the repo "packages and distributes GRACE skills" so coding agents can initialize artifacts, plan architecture, design verification, execute sequentially or in parallel-safe waves, and inspect health / drift / integrity. Attributes the methodology to Vladimir Ivanov. [vendor-doc]
- **Skills tree** `skills/grace/` — 14 subdirectories (`grace-ask`, `grace-cli`, `grace-execute`, `grace-explainer`, `grace-fix`, `grace-init`, `grace-multiagent-execute`, `grace-plan`, `grace-refactor`, `grace-refresh`, `grace-reviewer`, `grace-setup-subagents`, `grace-status`, `grace-verification`), each with a `SKILL.md` carrying YAML frontmatter `name` + `description`. [vendor-doc]
- **`grace-plan/SKILL.md`** (127 lines) — codifies the six-way module taxonomy, contract fields (PURPOSE / SCOPE / DEPENDS / LINKS), knowledge-graph ID scheme (`M-xxx`), and verification-ref scheme (`V-M-xxx`) summarized in the Upstream snapshot subsection above. [vendor-doc]
- **`grace-init/SKILL.md`** — confirms the six-artifact `docs/*.xml` scaffold listed above. [vendor-doc]
- **Canonical-vs-packaged split** per upstream `CLAUDE.md` — `skills/grace/*` is the canonical source; `plugins/grace/skills/grace/*` is a packaged mirror for marketplace distribution. A validation script `scripts/validate-marketplace.ts` enforces parity. [vendor-doc]

These facts are **not from the source post** and do not change what the author claims in the primary-source endorsement. They are included here as a direct-inspection reference for readers choosing whether to adopt the plugin.

## See also

- `../_plugin-audit.md` — 2026-04-23 coverage audit mapping the 14 plugin skills against the KB's 52 load-bearing concepts; source of the supplement-layer motivation in §How. [kb-local]
- `../02-semantic-graph/graph-as-backbone.md` — the semantic-graph primitive one of the three the author confirmed the plugin implements. [T10]
- `../04-markup-system/start-end-tags.md` — the START-END paired-tag markup the author names as correctly implemented. [T22]
- `../04-markup-system/xml-like-markup.md` — broader context on why paired-tag markup is the favored format. [T21]
- `../01-transformer-foundations/tc0-and-dyck-language.md` — the formal Dyck-Language basis for why START-END markup matters. [T07]
- `../03-contracts/contracts-overview.md` — the contract primitive, the third of the three elements the author confirmed. [T13]
- `../03-contracts/contract-fields.md` — canonical contract field specification against which an implementation can be checked. [T15]
- `../03-contracts/language-specific/python.md` — placement rules (description before `def`) any Claude-Code-native plugin should respect. [T19]
- `../09-tooling/kilo-code-open-code.md` — the author's own non-plugin delivery path (Kilo Code / Open Code prompt framework), included as the contrasting default. [T42]
- `../08-workflow-and-phases/architect-code-debug-modes.md` — Architect / Code / Debug phase model the plugin's feature surface would have to map onto for full GRACE coverage. [T40]
- `../11-case-studies/vlad-crm-and-contracts.md` — real case showing the markup + contracts layer doing the heavy lifting that such a plugin packages. [T51]
