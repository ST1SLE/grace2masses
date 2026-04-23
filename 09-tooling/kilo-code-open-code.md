# Kilo Code — Migration from Cline to Open Code Core

## What

**Kilo Code** is a VS Code extension used by the author of GRACE as one of the operational substrates for running the methodology. In April 2026 (post 3793, 2026-04-01), Kilo Code shipped a new version that migrated its internal engine from the **Cline** core to the **Open Code** core. The author tested the new version and recommended it for production use. [author-claim] (post 3793)

The reference announcement from the vendor (cited in the source) is at:

- **blog.kilo.ai/p/we-completely-rebuilt-the-kilo-vs-code-extension** — the Kilo Code blog post explaining the Cline-to-Open-Code rebuild. [vendor-doc] (post 3793)

For GRACE specifically, this migration is load-bearing because the Open Code core is the substrate on which:

1. The **Architect / Code / Debug mode UI** runs (covered in `../08-workflow-and-phases/architect-code-debug-modes.md`). [author-claim] (post 3793; `practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
2. **Multi-agent ↔ single-agent switching without prompt rewrites** is available. [author-claim] (post 3793)
3. **Dynamic skills** (to which the author migrated all of GRACE) work reliably when combined with forced tracing. [author-claim] (post 3809)

This file covers the migration itself: why the Cline core became untenable, what the Open Code core changes, and what the author's empirical observations were on the new version.

## Why

### The Cline core had become a "sinking Titanic"

The author's reason for migrating GRACE off the Cline-cored Kilo Code is explicit. After the Cline team departed to OpenAI, bug-fixing on the Cline core and its forks stagnated. Three concrete symptoms are called out in the source:

- **Bugs accumulated in a mountain**, with no one to fix them. [author-claim] (post 3793)
- **Full compatibility with newer Gemini versions was lost.** For a GRACE user whose Architect-mode workflow depends on large Gemini context windows, this was particularly costly. [author-claim] (post 3793)
- **Legacy Aider-patch code carried a lot of bugs.** The author mentions this as a separate degradation source beyond the Gemini-compat regression. [author-claim] (post 3793)

The author's phrasing, translated:

> "The migration is driven by the fact that Cline and its forks became a sinking Titanic after its team left for OpenAI. Bugs piled up in a mountain and no one was fixing them. Effectively even full compatibility with new Gemini versions was lost. Of the crooked legacy such as Aider patches with a heap of glitches we are not even speaking." [author-claim] (post 3793)

The practical consequence for GRACE is that a GRACE user running on Cline-based Kilo Code was accumulating friction against newer model versions — the opposite of the vendor-agnosticism GRACE is designed to enable (cross-link `../00-foundations/core-principles.md`, `../11-case-studies/vlad-crm-and-contracts.md`). [author-claim, inference from the author's vendor-agnosticism principle] (post 3793; `practical_applications_output.md` §"Пример CRM")

### What the Open Code core changes

The author observes four properties of the Open Code core that mattered for GRACE:

1. **Stability on full-cycle automated tasks.** The author's own test bed is a set of automated training tasks that exercise a full development cycle; these are compact but end-to-end. On the Open Code core they run **more stably** than on the previous Cline-based version. [empirical — author's own tests] (post 3793)
2. **~2× more token-efficient per task.** From the author's tests, the Open Code core uses roughly half the tokens of the previous core to accomplish the same tasks — "and that is where the speed comes from." [empirical — author's own tests] (post 3793)
3. **Direct Gemini API via VPN, no OpenRouter proxy.** Gemini became both stable and directly callable via API through a VPN, with no need to route through OpenRouter. The author reports "impressive speed" as a consequence. [author-claim] (post 3793)
4. **Single-agent ↔ multi-agent switch without prompt changes.** The Open Code core is "flexible: you can switch from multi-agent to single-agent without changing prompts, depending on the task." [author-claim] (post 3793)

Point (4) is called out separately because it is the **main evolutionary gain** of the migration in the author's framing: swarm support. [author-claim] (post 3793)

### Why swarm support is the main benefit

The author's sentence: "swarms of agents are the main benefit of the development here, but the Open Code core is flexible." [author-claim] (post 3793)

Two reasons this matters for GRACE:

- **GRACE's autonomous-agent layer is explicitly designed for swarms.** Anti-loop protection, forced context, and knowledge-base-in-tests (covered in the `../07-autonomous-agents/` directory) were all adapted specifically for autonomous and swarm agents — see `testing_and_autonomous_agents_output.md` §"Адаптация GRACE". [author-claim] (`testing_and_autonomous_agents_output.md` §"Адаптация GRACE")
- **Being able to swap single-agent and multi-agent modes on the same prompts** lets a GRACE user scale up to a swarm when the task justifies it (e.g., large refactors, Architect-phase modelling) and back down when a single focused agent is better (e.g., targeted debug). The cost of that switch is a UI click, not a prompt rewrite. [author-claim, inference] (post 3793; cross-link `../07-autonomous-agents/swarm-vs-single-agent.md`)

A concrete example of swarm capability that ran on this substrate: the 16-subagent Claude Opus swarm that generated a book from video training material, with an orchestrator consolidating the output. [author-claim] (post 3859; cross-link `../07-autonomous-agents/swarm-vs-single-agent.md`)

### Why token-efficiency matters for GRACE

GRACE is already biased toward **few, large prompts**. An Architect-mode design session typically runs ~20 prompts generating 300–400k tokens of design context (cross-link `../08-workflow-and-phases/large-rare-prompts-billing.md`). [author-claim] (`practical_applications_output.md` §"Большие, но редкие запросы")

At that scale, a 2× token-efficiency improvement at the core level compounds with the methodology's own token discipline (wenyan prompting, adjacency rules, semantic-density graphs, etc.). The author does not state a specific total savings number — but the direction is additive, not redundant. [author-claim, inference from GRACE's token discipline] (post 3793; posts 3819, 3892)

### Why the $100-for-a-bug offer is worth flagging

The vendor (Kilo Code) publicly offered **$100 in tokens for a found bug** at the time of the migration post. The author mentions this as a vendor signal: the new core is considered ready enough to bounty-hunt, not a beta being dumped on users. [vendor-doc] (post 3793)

## How

### The migration path the author followed

The sequence is documented across two consecutive posts:

1. **Post 3793 (2026-04-01)** — migrate GRACE onto the new Kilo Code (Open Code core). This is the "core replacement" step. [author-claim] (post 3793)
2. **Post 3809 (2026-04-03)** — migrate all of GRACE onto **dynamic skills** on the Open Code core. This is the "prompt architecture" step that follows. [author-claim] (post 3809)

The author's phrasing from post 3809:

> "I've moved all of GRACE to dynamic skills on the Open Code core (which is also in the new Kilo Code)." [author-claim] (post 3809)

That means a practitioner following the author's trajectory today enters at **step 2** — dynamic skills — with the Open Code core already assumed. Step 1 is historical at this point, but the trade-offs that drove it (Cline-core decay, Gemini compat loss, Aider-patch bugs) are still the right reasons to prefer Open-Code-cored environments going forward. [author-claim, inference] (posts 3793, 3809)

### What specifically was ported across the migration

The author's sentence: "I ported all the best of the prompting for Gemini CLI and Cursor into Kilo Code." [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

Four concrete adaptations the source enumerates (from `practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code"):

1. **Semantic-search prompts for the AI agent**, tested on a large codebase. The author notes Kilo Code's chunk approach differs significantly from Cursor's. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
2. **Runtime-version control**: the agent scans the environment's versions, rates libraries by AI-friendliness, and integrates with **Context 7** for example retrieval. [author-claim + vendor-doc on Context 7] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
3. **Knowledge-base creation in Kilo Code's graph DB, by the AI itself**, with a split rule on what goes into XML/MD and what goes into the graph database. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
4. **Adaptation of code edits to the new diff tool** Anthropic proposed for Claude and that Kilo Code supports, which also works well with Gemini through GRACE prompts and markup. [author-claim + vendor-doc on Anthropic's diff tool] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

The sources do **not** specify the exact prompt text for any of these four. [sources do not specify]

### How the migration affects multi-agent work

The Open Code core's ability to switch between single-agent and multi-agent execution **without rewriting prompts** is the structural enabler for GRACE's autonomous-agent layer. The author pairs this with mandatory autonomous-agent protections (cross-link `../07-autonomous-agents/`):

- **Anti-loop protection** — attempt counters plus forced context injection when a loop is detected. [author-claim] (post 3695)
- **Forced context** — pushing critical log lines, prior fixes, and constraints into the agent's context rather than hoping it calls the right Tool. [author-claim] (posts 3695, 3693)
- **Knowledge-base in tests** — regression memory so repeat failures inject prior-fix knowledge into the agent. [author-claim] (post 3695)

Without these protections, the same swarm capability that makes Open Code attractive becomes the mechanism by which billions of tokens are burned in a fix loop. The author is explicit: "a swarm, or even a single autonomous agent, can dumbly hang and burn even a billion tokens" without anti-loop protection. [author-claim] (post 3695)

### Dynamic skills on the Open Code core

The second migration (post 3809) moved GRACE onto dynamic skills. The author notes that skills carry a scientifically-noted risk: **the LLM can decide "I already know" and simply skip loading the skill**. That is called out as a serious risk for agent stability, and only dilettantes drop skills in without protection. [author-claim + research-backed framing] (post 3809)

Two mitigations the author uses, both compatible with the Open Code core:

1. **Trigger words — MANDATORY MODE / MANDATORY PROTOCOL.** These signal to the LLM that the skill encodes critical rules, not optional knowledge-base material. [author-claim] (post 3809)
2. **Forced tracing.** The author's preferred method. One skill explicitly names — by name — the next skill to load. This forms a reliable loading chain: the agent does not have to "discover" which skill to load next; the currently-loaded skill tells it. [author-claim] (post 3809)

In the Open Code core specifically, the author can strengthen this by explicitly requiring the `skill` tool call, which is visible in the author's screenshots in post 3809 (screenshot not transcribed in the source text). [author-claim, vendor-doc on Open Code's `skill` tool] (post 3809)

With trace-based loading, dynamic skills are "no less reliable than trace-based dynamic prompts." [author-claim] (post 3809)

(Cross-link `./skills-system.md` for the full skill-system treatment.)

### What this migration does NOT change

Several GRACE invariants are independent of the core-switch; they apply equally on Cline-cored and Open-Code-cored setups:

- **Graph first, contracts second, code third.** The top-down pipeline is methodology, not tooling. (Cross-link `../08-workflow-and-phases/top-down-pipeline.md`.) [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **In-source documentation, not external docs.** The FIM-trained agent's bias to read code, not external documentation, does not depend on the IDE. (Cross-link `../04-markup-system/in-source-documentation.md`.) [author-claim] (posts 3820, 3821, 3878)
- **LDD + log-to-code correlation for debug.** Debug-mode behaviour rides on log heuristics that are prompt-level, not engine-level. (Cross-link `../05-logging-ldd/log-to-code-correlation.md`.) [author-claim] (post 3693)
- **XML-like / Dyck-compatible markup.** Format choice is driven by the transformer, not the IDE. (Cross-link `../01-transformer-foundations/tc0-and-dyck-language.md`, `../04-markup-system/xml-like-markup.md`.) [research-backed + vendor-doc] (post 3948; `xml_mapping_anchors_output.md`)

The migration changes the **substrate** on which these invariants run; it does not change the invariants. [author-claim, inference] (posts 3793, 3809; `most_important_output.md`)

### Vendor economics context

A tangential but connected note from the sources: the billing model that fits GRACE on Kilo Code best is **Google Gemini Code Assist**, at roughly $20–$40/month for ~2000 requests/day (vendor-documented at developers.google.com/gemini-code-assist/resources/quotas). Even a disciplined GRACE practitioner struggles to use 1000 requests/day because GRACE's Architect-style workflow batches design work into ~20 large prompts that together generate 300–400k tokens. [author-claim + vendor-doc] (`practical_applications_output.md` §"Большие, но редкие запросы")

This economics angle matters to the migration because the Open Code core's **direct Gemini API via VPN** (no OpenRouter proxy) makes the Gemini Code Assist path the natural one. [author-claim, inference] (post 3793; `practical_applications_output.md` §"Большие, но редкие запросы")

(Cross-link `../08-workflow-and-phases/large-rare-prompts-billing.md` for the detailed treatment.)

### What the sources do NOT specify

Do not invent these:

- **The exact version numbers** of Kilo Code, Open Code, Cline, or any other engine mentioned. The source text does not include them. [sources do not specify]
- **The specific bugs** that accumulated in Cline or the exact Aider-patch bugs called out. The source uses summary language ("mountain of bugs", "crooked legacy"), not an enumerated list. [sources do not specify]
- **The exact token-count numbers** behind the "~2× more efficient" claim. The author cites his own tests without reproducing the raw numbers. [sources do not specify]
- **The exact shape of the $100-for-a-bug offer** (what counts as a bug, how it is paid, deadlines). The source only states that the offer exists. [sources do not specify]
- **Benchmark comparisons** between Open Code and any non-Cline core. The source compares only Open Code vs the previous Cline-based version. [sources do not specify]
- **Whether Kilo Code offers modes other than Architect / Code / Debug.** The sources name only those three. [sources do not specify]
- **Any screenshot content** beyond what the source text transcribes. The author references a screenshot in post 3809 showing the `skill` tool call requirement, but the screenshot content itself is not in the source text. [sources do not specify]

## Evidence

### Primary source — post 3793 (2026-04-01)

The core migration claim bundle. All items below are from post 3793 unless otherwise marked.

**Vendor / tool facts:**

- Kilo Code is a VS Code extension. [vendor-doc]
- Kilo Code migrated from the **Cline** core to the **Open Code** core. [vendor-doc]
- Kilo Code offers **$100 in tokens** for a reported bug on the new version. [vendor-doc]
- Reference link in the source: **blog.kilo.ai/p/we-completely-rebuilt-the-kilo-vs-code-extension**. [vendor-doc]

**Author-claims on the Cline core's decline:**

- The Cline team left for OpenAI. [author-claim]
- Bugs accumulated in a "mountain" with no one to fix them. [author-claim]
- Full compatibility with newer Gemini versions was lost. [author-claim]
- Aider-patch legacy carried many bugs. [author-claim]

**Empirical / author-claim observations on the Open Code core:**

- The new version is "noticeably more reliable" than the previous Kilo Code on the author's automated training tasks (compact but full-cycle). [empirical — author's own tests]
- Gemini is **stable and directly callable via API over VPN**, no OpenRouter proxy, at "impressive speed". [author-claim]
- Open Code is **roughly 2× more token-efficient per task** than the previous core ("which is where the speed comes from"). [empirical — author's own tests]
- Open Code is **flexible: switch from multi-agent to single-agent without changing prompts**. [author-claim]
- **Swarms of agents are the main evolutionary benefit** of the new core. [author-claim]
- The author migrated GRACE onto the new Kilo Code. [author-claim]

Author's summary recommendation in the source:

> "I tested the new version of Kilo Code for VS Code, to which it updates by default, and I recommend it." [author-claim] (post 3793)

### Primary source — post 3809 (2026-04-03)

The follow-up migration: dynamic skills on the Open Code core.

- The author moved **all of GRACE** to dynamic skills on the Open Code core (also used by new Kilo Code). [author-claim]
- The risk: an LLM may skip loading a skill because it believes it "already knows" — this is a scientifically-noted risk. [author-claim + research-backed framing]
- Mitigation 1 — trigger words: **MANDATORY MODE** / **MANDATORY PROTOCOL**. [author-claim]
- Mitigation 2 — **forced tracing**: one skill names by name the next skill to load, forming a reliable chain. [author-claim]
- In Open Code specifically, the loading can be strengthened by **explicitly requiring the `skill` tool call**. [author-claim + vendor-doc on Open Code `skill` tool]
- With trace-based loading, dynamic skills are "no less reliable than trace-based dynamic prompts." [author-claim]

### Supporting — `practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code"

The upstream rationale for choosing Kilo Code's interface organisation over Cursor:

- Architect / Code / Debug modes with associated prompts make GRACE's phase structure visually explicit. [author-claim]
- The author ported the best of his Gemini CLI and Cursor prompting into Kilo Code. [author-claim]
- Four concrete adaptations: semantic-search prompting for the vector index; runtime-version control with Context 7 integration; AI-built knowledge base in Kilo Code's graph DB with a split between XML/MD and graph-DB storage; adaptation of code edits to the new Anthropic diff tool Kilo Code supports. [author-claim + vendor-doc on Context 7 and Anthropic diff tool]

(Detailed treatment: `../08-workflow-and-phases/architect-code-debug-modes.md`.)

### Supporting — post 3695 (autonomous-agent protections)

The protections that become non-negotiable once swarm capability is on the table:

- Anti-Loop Protection: attempt counters plus context injection. Without it, a swarm or a single autonomous agent can burn a billion tokens. [author-claim] (post 3695)
- Forced-context injection as a general principle; relying on the agent to call Tools when confused is naive, especially if looping. [author-claim] (post 3695)
- Knowledge base of prior fixes, force-injected on regression. [author-claim] (post 3695)

(Detailed treatment: `../07-autonomous-agents/anti-loop-protection.md`, `../07-autonomous-agents/forced-context.md`, `../07-autonomous-agents/knowledge-base-in-tests.md`.)

### Supporting — post 3859 (swarm example)

A concrete swarm output the author approved of:

- 16 Claude Opus subagents wrote a book from video training material; an orchestrator consolidated their work. The author reviewed it and considered the swarm's grasp of his concepts "not bad" — "something the AI even explained a bit more simply." [author-claim] (post 3859)

### Supporting — post 3793 economics

The Open Code core's **direct Gemini API** path is the one that pairs well with Google Gemini Code Assist flat-rate billing. Source for the billing detail:

- Gemini Code Assist at $20–$40/month, ~2000 requests/day. [vendor-doc] (`practical_applications_output.md` §"Большие, но редкие запросы")
- Link: developers.google.com/gemini-code-assist/resources/quotas. [vendor-doc]
- GRACE-style work rarely exceeds 1000 requests/day even in heavy sessions. [author-claim]

(Detailed treatment: `../08-workflow-and-phases/large-rare-prompts-billing.md`.)

### Supporting — post 3859 and the case-study line

GRACE on Kilo Code (pre-migration) has a large-codebase case study from Kirill: 132,666 prod LOC + 50,026 test LOC on a pre-stage project, migrated to Kilo Code using GPT-5.4(xh) as orchestrator and GPT-5.4(m) as workers, 20+ hours of autonomous run. That case is on the **earlier** Kilo Code (Cline-based). (Cross-link `../11-case-studies/kirill-132k-loc.md`.) [empirical — practitioner case] (post 3620)

That case exists as a data point for the level of autonomy Kilo Code could already reach pre-migration; the Open Code core migration is an improvement on that base. [author-claim, inference] (posts 3620, 3793)

### What is NOT in the sources (do not invent)

- No specific **version strings** for Kilo Code, Cline, Open Code, or Gemini.
- No **enumerated list of the Aider-patch bugs** or the Gemini-compat breakages.
- No **raw token counts** behind the "~2× more efficient" figure.
- No **terms-and-conditions detail** on Kilo Code's $100-for-a-bug offer.
- No **direct URL for Open Code's own repo** (the cited URL is the Kilo Code blog post, not the Open Code project).
- No **named non-Cline cores** for comparison other than Open Code itself.

## See also

- `../08-workflow-and-phases/architect-code-debug-modes.md` — the Architect / Code / Debug mode UI that rides on the Open Code core.
- `../08-workflow-and-phases/top-down-pipeline.md` — the three-phase GRACE pipeline the mode UI maps onto.
- `../08-workflow-and-phases/large-rare-prompts-billing.md` — the "few, large, rare" prompt economics; why direct Gemini API (one of the Open Code core's wins) aligns with Gemini Code Assist billing.
- `./skills-system.md` — dynamic skills on the Open Code core; MANDATORY MODE / MANDATORY PROTOCOL trigger words and forced-tracing loading.
- `./claude-code-grace-plugin.md` — an alternative operational substrate for GRACE (Claude Code plugin at osovv/grace-marketplace), for readers not using Kilo Code.
- `./cursor-limitations.md` — why Cursor fits GRACE worse than Kilo Code: chunked `read_file` reads (100–200 lines; only first ~100 guaranteed) and no explicit per-phase mode UI.
- `../07-autonomous-agents/swarm-vs-single-agent.md` — the single-agent ↔ multi-agent swap that the Open Code core enables without prompt rewrites.
- `../07-autonomous-agents/anti-loop-protection.md` — mandatory protection once swarm capability is used.
- `../07-autonomous-agents/forced-context.md` — forced-context injection, a companion protection to anti-loop.
- `../07-autonomous-agents/knowledge-base-in-tests.md` — regression memory for the autonomous layer.
- `../05-logging-ldd/log-driven-development.md` — LDD as the substance of Debug-mode work on this substrate.
- `../05-logging-ldd/log-to-code-correlation.md` — the log heuristic every Open-Code-based Debug run relies on.
- `../04-markup-system/in-source-documentation.md` — the in-source documentation GRACE runs on regardless of core.
- `../11-case-studies/kirill-132k-loc.md` — a large-codebase Kilo Code case study (on the pre-migration, Cline-based version) that contextualises the level of autonomy Kilo Code already reached before the Open Code migration.
- `../00-foundations/core-principles.md` — vendor-agnosticism principle; the reason the author migrated to a core that does not lock out newer Gemini versions.
