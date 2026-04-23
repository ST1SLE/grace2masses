# Architect / Code / Debug Modes — Mapping GRACE Phases to Kilo Code

## What

**Kilo Code** (the VS Code extension built on the **Open Code** core) exposes three explicit, separately-prompted UI modes:

1. **Architect** — design / modelling phase.
2. **Code** — implementation phase.
3. **Debug** — self-correction / debugging phase.

Each mode has its own associated system prompt, and the user switches between them explicitly in the IDE UI. [vendor-doc — Kilo Code UI] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

From the author's perspective, this three-mode UI maps **directly** onto GRACE's top-down phase model (Architectural graph → Module/function contracts → Code → Debug with LDD). The modes make the phase structure of GRACE **visually explicit** in the IDE, rather than leaving it implicit in a single chat window. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

The author's summary phrase:

> "My GRACE and PCAM fit the Kilo Code interface organisation better than Cursor. Although in Cursor you can also keep a RAG-base of code-generation prompts, the fact that Kilo Code in the UI carves out the **Architect, Code, Debug** modes with associated prompts at minimum makes GRACE **more visually understandable** in the UI on the phase-structure of development."
> ([author-claim] — `practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

Main source: **`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code"**. Supporting sources: post 3793 (migration to Open Code core), post 3809 (dynamic skills), and the phase-description posts referenced in `./top-down-pipeline.md`.

## Why

### The problem the mode UI solves

GRACE is fundamentally a **phase-aware** methodology: the agent must be kept on top-down rails (graph → module contract → function contract → code), and debug behaviour (LDD + self-correction) is different from design behaviour. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")

In a single-mode IDE (one chat, one system prompt, regardless of where the user is in the lifecycle), there is no structural signal telling the agent "we are designing now, not coding" or "we are debugging now, not designing." The methodology's phase structure exists only in the user's head and in whatever prompts they remember to paste. The author's observation is that on a single-mode IDE the AI tends to **jump to code** on its own — it has to be actively assisted through the top-down pipeline. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")

Kilo Code's Architect / Code / Debug modes give the methodology a visible place to live in the UI. Each mode carries a different system prompt, so when the user is in Architect mode the model is primed for design-level behaviour, and when the user is in Debug mode the model is primed for self-correction with LDD. The phase structure is no longer just the user's discipline — it is scaffolded by the tool. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

### Why Cursor is a worse fit for GRACE than Kilo Code

The author's explicit comparison:

- **Cursor** can hold a RAG-base of generation prompts. That is useful, but the IDE does not expose distinct lifecycle modes. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
- **Kilo Code** exposes Architect / Code / Debug as first-class UI modes, with associated prompts. This makes GRACE's phase structure ergonomically reachable: the user clicks into the right mode instead of manually paging through a prompt library. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

The conclusion the author draws is **not** that Cursor cannot do GRACE — it can (and the author previously ran GRACE in Cursor and in Gemini CLI, see migration note in post 3793). The conclusion is that the methodology **fits** the Kilo Code UI better, because the UI's phase modes mirror GRACE's phase model. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code"; post 3793)

### Why making the methodology visually explicit matters

GRACE emphasises that the agent should be given **structural signals**, not just text. XML-like markup (cross-link `../04-markup-system/xml-like-markup.md`), paired START/END tags (cross-link `../04-markup-system/start-end-tags.md`), and explicit per-mode prompts all serve the same purpose: they give the model anchor tokens for "which phase are we in" and "which behaviour is expected now." The Kilo Code mode UI is, at the IDE level, the same pattern applied to the development lifecycle. [author-claim — inference from the author's consistent position on structural signals] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code"; `xml_mapping_anchors_output.md`)

### What each mode is for in GRACE terms

The author maps the modes onto GRACE phases as follows:

- **Architect mode** — upstream: few, large, high-stakes prompts. This is where GRACE's Phase-1 architectural-graph work and Phase-2 module-contract work happen. The sources are explicit that design-phase prompts are large and rare, not many and small (see the economics note below). [author-claim] (`practical_applications_output.md` §"Большие, но редкие запросы")
- **Code mode** — implementation: the agent generates classes, methods, and function-level contracts downstream from the architectural graph and module contracts. GRACE's Phase-3 work. [author-claim — mapping onto `most_important_output.md` §"Top-down логика и переход к контрактам"]
- **Debug mode** — self-correction: the agent uses LDD (Log-Driven Development), log-to-code correlation, and tests-in-the-self-correction-loop to converge on working code. [author-claim] (`testing_and_autonomous_agents_output.md` §"Тесты как часть self-correction loop"; `practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

### The upstream economics the Architect mode implies

Architect-mode prompts are **few and large**. From the same source document (`practical_applications_output.md` §"Большие, но редкие запросы"):

- A GRACE Architect-mode design session is typically in the order of **~20 prompts** that together generate **~300–400 thousand tokens** of design context. [author-claim] (`practical_applications_output.md` §"Большие, но редкие запросы")
- Even reaching 1000 requests/day is hard when working GRACE-style. [author-claim] (`practical_applications_output.md` §"Большие, но редкие запросы")
- The concrete billing consequence: Google Gemini Code Assist (flat $20–$40/month, ~2000 requests/day) tends to fit better than per-token API pricing. The relevant link in the source: developers.google.com/gemini-code-assist/resources/quotas. [vendor-doc] (`practical_applications_output.md` §"Большие, но редкие запросы")

(That economics angle is covered in detail in `./large-rare-prompts-billing.md`. This file only notes the link between "Architect mode = upstream, rare, large prompts" and the resulting billing model.)

## How

### Using the three modes in a GRACE workflow

The canonical flow, per the author's phase model plus Kilo Code's mode UI:

1. **Architect mode** → produce the architectural graph (main classes / modules and their relationships; business-process modelling mapped onto architecture). Then produce per-module contracts from the graph. Prompts in this phase are few, large, and high-stakes. (See `./top-down-pipeline.md` for the phase-model detail.) [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам"; `practical_applications_output.md` §"Большие, но редкие запросы")
2. **Code mode** → produce function/method contracts and their code, anchored in the module contract, which is in turn anchored in the architectural graph. The function contract here is a **local refinement** of the app's overall intent at that site, not spontaneous AI creativity. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
3. **Debug mode** → when something breaks, enter the self-correction loop. Read structured logs (with log-to-code correlation tokens — see `../05-logging-ldd/log-to-code-correlation.md`), make semantic judgments about the trajectory (LDD — see `../05-logging-ldd/log-driven-development.md`), apply fixes, repeat. [author-claim] (`testing_and_autonomous_agents_output.md` §"Адаптация GRACE", §"Тесты как часть self-correction loop"; post 3693)

The sources do **not** specify the exact per-mode prompt content the author uses inside Kilo Code. The source document says that the author "moved the best of the prompting for Gemini CLI and Cursor into Kilo Code, and also supported Kilo Code-specific features" — which implies per-mode prompt sets exist — but the source does not reproduce those prompts verbatim. Do not invent them. [author-claim — sources do not specify the prompts themselves] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

### What the author explicitly ported into Kilo Code

From `practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code", the author lists four concrete adaptations made when migrating GRACE to Kilo Code:

1. **Semantic-search prompts for the AI agent** to correctly query vector search — tested on a large codebase. The author notes the approach to chunks differs significantly from Cursor. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
2. **Runtime-version control**: the agent scans environment versions, rates libraries by AI-friendliness, and integrates with **Context 7** for example retrieval. [author-claim, vendor-doc mention of Context 7] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
3. **Knowledge-base creation in Kilo Code's graph DB, by the AI itself**, with a split rule: what goes in XML/MD vs what goes in the graph database. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
4. **Adaptation of code edits to the new diff tool** Anthropic proposed for Claude and that Kilo Code supports, but which also works well with Gemini through GRACE prompts and markup. [author-claim, vendor-doc mention of Anthropic's diff tool] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

None of these four items is a per-mode prompt by itself; they are infrastructure pieces that each mode relies on. The sources do not specify which piece lives in which mode. [sources do not specify]

### Open Code core and mode ergonomics

Kilo Code's mode UI sits on top of the **Open Code** core, which Kilo Code migrated to from the Cline core. The migration matters for how the modes work in practice:

- **Same prompts, multi-agent ↔ single-agent switch.** Open Code lets the user switch between multi-agent and single-agent execution without rewriting prompts. This applies inside each mode. [author-claim] (post 3793; cross-link `../07-autonomous-agents/swarm-vs-single-agent.md`)
- **~2× token efficiency vs Cline.** The author's own tests on automatic training tasks. [empirical — author's own tests] (post 3793)
- **Direct Gemini API via VPN, no OpenRouter proxy.** Relevant because Architect-mode prompts are large (300–400k-token sessions), so direct-API speed matters for iterability. [author-claim] (post 3793)

Reference in the source: blog.kilo.ai/p/we-completely-rebuilt-the-kilo-vs-code-extension (post 3793).

### Dynamic skills as the per-mode prompting substrate

The author subsequently moved all of GRACE onto **dynamic skills** on the Open Code core (which is also what new Kilo Code uses). This is the substrate the per-mode prompts ride on. [author-claim] (post 3809)

Two caveats the author flags about skills, both relevant inside an Architect/Code/Debug UI:

- **The LLM may decide "I already know" and skip loading a skill.** This is a scientifically-noted risk; only dilettantes drop skills in without protection. [author-claim + research-backed framing] (post 3809)
- **Mitigations:** trigger words such as **MANDATORY MODE** / **MANDATORY PROTOCOL** signal "this is a critical rule, not optional KB"; and **forced tracing** — one skill explicitly names the next skill by name, forming a reliable chain. In Open Code the author can strengthen this by explicitly requiring the `skill` tool call. With trace-based loading, dynamic skills are as reliable as dynamic prompts on tracing. [author-claim] (post 3809)

The consequence for the mode UI is that when the user enters, say, Debug mode, the associated skill(s) need to actually load — the mitigations above are what make that reliable.

### What each mode inherits from GRACE's invariants

Regardless of mode, the same GRACE invariants apply:

- **Graph first, contracts second, code third.** Architect mode is where the graph and module contracts are produced; Code mode assumes they exist; Debug mode uses them for navigation. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- **In-source documentation, not external docs.** The same contract-in-code placement rule applies across all three modes. (Cross-link `../04-markup-system/in-source-documentation.md`.) [author-claim] (post 3878)
- **LDD + log-to-code correlation when logs are involved.** Debug mode is the primary consumer, but any mode that reads logs relies on these heuristics. (Cross-link `../05-logging-ldd/log-to-code-correlation.md`, `../05-logging-ldd/belief-state-logging.md`.) [author-claim] (post 3693)
- **Autonomous-agent protections if swarms are used.** Anti-loop protection, forced context, knowledge base of prior fixes. These apply inside any mode that runs long trajectories (typically Debug, sometimes Code). [author-claim] (posts 3695, 3693)

### What Cursor can (and can't) do for comparison

The author acknowledges Cursor remains capable:

- Cursor **can** hold a RAG-base of generation prompts. So a disciplined user can simulate mode-switching by selecting the right prompt for the current phase. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")
- Cursor **does not** expose explicit mode-separated UI with per-mode prompts. So the phase structure remains implicit in the user's workflow, not scaffolded by the IDE. [author-claim] (`practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code")

Separately, Cursor has a concrete read-size constraint that matters in all GRACE phases: `read_file` reads code and logs in ~100–200-line chunks, with only the first ~100 lines guaranteed — beyond that the agent falls back on text or semantic search. GRACE markup is what keeps the agent oriented in that setting. (Cross-link `../09-tooling/cursor-limitations.md`.) [author-claim] (`xml_mapping_anchors_output.md` §"Почему семантическая разметка логов важна"; post 3819)

### What the sources do NOT specify

Do not invent these:

- The **exact per-mode prompt text** the author puts into Kilo Code's Architect / Code / Debug modes. The source describes the migration and its adaptations, not the prompt bodies. [sources do not specify]
- **Whether a formally-named "Architect mode" skill is one skill or a chain.** The author describes dynamic skills and the forced-tracing pattern (post 3809) but does not enumerate which skills are loaded by which mode in Kilo Code. [sources do not specify]
- **Any benchmark numbers for "mode UI vs single-chat UI."** The source argues fit, not measured uplift. [sources do not specify]
- **Whether Kilo Code offers modes other than Architect / Code / Debug.** The source names only those three. [sources do not specify]

## Evidence

### Primary source — `practical_applications_output.md` §"GRACE + PCAM и интерфейс Kilo Code"

The canonical claim:

> "My GRACE and PCAM fit the Kilo Code interface organisation better than Cursor. Although in Cursor you can also keep a RAG-base of code-generation prompts, the fact that Kilo Code in the UI carves out the Architect, Code, Debug modes with associated prompts at minimum makes GRACE more visually understandable in the UI on the phase-structure of development."

Claim types:

- "Kilo Code UI has Architect / Code / Debug modes with associated prompts" — **[vendor-doc — Kilo Code UI behaviour]**.
- "Maps onto GRACE phases; makes the methodology visually explicit in the UI" — **[author-claim]**.
- "Cursor can hold a RAG-base of generation prompts but lacks explicit mode UI" — **[author-claim]** with [vendor-doc] grounding on Cursor's capabilities.

### Primary source — `practical_applications_output.md` §"Большие, но редкие запросы"

Economics of Architect-mode prompts (few, large, high-stakes):

- ~20 prompts per Architect-mode design session, generating ~300–400k tokens of design context. [author-claim]
- Even 1000 requests/day is hard in GRACE-style work. [author-claim]
- Google Gemini Code Assist ($20–$40/month, ~2000 requests/day) fits better than per-token API billing. [vendor-doc + author-claim]
- Link: developers.google.com/gemini-code-assist/resources/quotas.

(Detailed treatment of this economics in `./large-rare-prompts-billing.md`.)

### Supporting — post 3793 (Open Code core migration)

The Kilo Code mode UI the author praises runs on the **Open Code** core that Kilo Code migrated to from the Cline core. Key properties that affect the mode UI:

- "Kilo Code migrated from Cline to Open Code. Cline had become a 'sinking Titanic' after its team moved to OpenAI; Gemini compatibility degraded; Aider-patch legacy introduced bugs." [author-claim]
- Open Code is ~2× more token-efficient on the author's tests. [empirical — author tests]
- Direct Gemini API via VPN (no OpenRouter proxy); faster. [author-claim]
- Flexible: single-agent ↔ multi-agent swap without changing prompts. [author-claim]
- Swarm support is the main evolutionary gain. [author-claim]
- Kilo Code offers $100 in tokens for bug reports. [vendor-doc]
- Reference: blog.kilo.ai/p/we-completely-rebuilt-the-kilo-vs-code-extension.

### Supporting — post 3809 (dynamic skills as the per-mode substrate)

The per-mode prompting in Kilo Code rides on dynamic skills on the Open Code core:

- Author moved GRACE onto dynamic skills on Open Code (and thus new Kilo Code). [author-claim]
- Risk: LLM can skip a skill it thinks it "already knows" — scientifically-noted. [author-claim + research-backed framing]
- Mitigations: MANDATORY MODE / MANDATORY PROTOCOL trigger words; forced tracing where one skill names the next by name. [author-claim]
- In Open Code, the author can strengthen skill loading by explicitly requiring the `skill` tool call. [author-claim]

(Cross-link `../09-tooling/skills-system.md` for the full treatment.)

### Supporting — GRACE's phase model (context for the mapping)

The Architect / Code / Debug modes map onto a three-phase model that exists independently of any IDE:

- Phase 1 — architectural graph. [author-claim] (`most_important_output.md` §"Top-down логика и переход к контрактам")
- Phase 2 — module contracts. [author-claim] (same)
- Phase 3 — function contracts + code (local refinements of overall app intent, not spontaneous AI creativity). [author-claim] (same)
- The AI must be actively assisted through top-down — otherwise it jumps straight to code. [author-claim] (same)

Debug-phase behaviour (LDD, log-to-code correlation, tests-in-self-correction-loop) is documented in:

- `../05-logging-ldd/log-driven-development.md` (`testing_and_autonomous_agents_output.md` §"Адаптация GRACE").
- `../05-logging-ldd/log-to-code-correlation.md` (post 3693).
- `../06-testing/tests-in-self-correction-loop.md` (`testing_and_autonomous_agents_output.md` §"Тесты как часть self-correction loop").

### Supporting — Kilo Code plugin ecosystem (context)

Not all GRACE implementations ride on Kilo Code. The author also notes:

- Alexey Chendemerov built a Claude Code plugin implementing GRACE ideas (github.com/osovv/grace-marketplace). The author confirms the core ideas are right — graph on code, START-END markup, contracts — but notes that he normally implements GRACE via prompt frameworks, not plugins. [author-claim] (`practical_applications_output.md` §"Плагин для Claude Code")

The relevance to this file: mode-style UI is one operational substrate for GRACE; plugins are another; CLI-with-skills is yet another. Kilo Code's mode UI is called out as a **good fit** for GRACE, not the **only** substrate. [author-claim, inference]

### What is NOT in the sources (do not invent)

- The **exact text** of any Architect / Code / Debug prompt the author uses.
- Any **benchmark** comparing Kilo Code mode UI against Cursor single-chat for GRACE work.
- Any **screenshot** or other visual evidence from the source that is not transcribed in the source text.
- **A named list of additional Kilo Code modes** beyond Architect / Code / Debug (if more exist, sources do not name them).
- A claim that **only** Kilo Code supports GRACE — the sources are clear that Cursor, Gemini CLI, and Claude Code (via plugin) have also been used; the Kilo Code claim is specifically about fit to the phase model.

## See also

- `./top-down-pipeline.md` — the three-phase GRACE pipeline (graph → module contracts → function contracts + code) that the Architect / Code / Debug modes map onto.
- `./large-rare-prompts-billing.md` — the "few, large, high-stakes prompts" economics implied by Architect mode; detailed treatment of Gemini Code Assist pricing and the 300–400k-token design session.
- `../05-logging-ldd/log-driven-development.md` — LDD as the substance of Debug mode.
- `../05-logging-ldd/log-to-code-correlation.md` — the log heuristic Debug mode relies on.
- `../05-logging-ldd/belief-state-logging.md` — the second log heuristic Debug mode relies on.
- `../05-logging-ldd/forced-context-injection.md` — pushing log lines into the agent's context in Debug mode.
- `../06-testing/tests-in-self-correction-loop.md` — how tests live inside the self-correction loop used in Debug mode.
- `../06-testing/classical-tests-antipattern.md` — why the Debug-mode view of tests differs from classical TDD.
- `../07-autonomous-agents/anti-loop-protection.md` — protections for Debug-mode behaviour when agents are autonomous.
- `../07-autonomous-agents/forced-context.md` — forced-context stabilization mechanism.
- `../07-autonomous-agents/swarm-vs-single-agent.md` — single-agent vs multi-agent execution; Open Code allows switching without changing prompts.
- `../09-tooling/kilo-code-open-code.md` — the Kilo Code migration from Cline to Open Code that the mode UI rides on.
- `../09-tooling/cursor-limitations.md` — why Cursor is a worse fit in practice (chunked reads, no explicit mode UI).
- `../09-tooling/skills-system.md` — dynamic skills as the prompting substrate for each mode; MANDATORY MODE / MANDATORY PROTOCOL and forced tracing.
- `../09-tooling/claude-code-grace-plugin.md` — an alternative substrate: Claude Code plugin implementing GRACE (osovv/grace-marketplace).
- `../03-contracts/contracts-overview.md` — contracts as the semantic shield produced in Architect mode and consumed in Code and Debug modes.
- `../03-contracts/contract-fields.md` — the canonical PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS fields that each mode operates on.
- `../04-markup-system/in-source-documentation.md` — in-source documentation as GRACE's documentation substrate, independent of mode.
