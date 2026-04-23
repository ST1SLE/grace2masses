# Swarm vs Single Agent

## What

GRACE supports two scaling modes for autonomous execution on top of the same prompt material: a **single-agent** configuration and a **multi-agent swarm** configuration. The two modes share identical prompts and methodology; the switch is an operational choice, not a rewrite. [author-claim] (post 3793)

- **Single agent** — one autonomous agent executes the whole trajectory end-to-end.
- **Multi-agent swarm** — multiple subagents execute in parallel under an **orchestrator** that consolidates their outputs. [author-claim] (post 3859)

Open Code (the core that Kilo Code migrated to) is the concrete runtime the author validates this duality on:

> "The Open Code core is … flexible: you can switch from multi-agent to a single agent without changing prompts, depending on the task." [author-claim] (post 3793)

This file is about the operational question: **when is the swarm the right shape for the problem, and when is a single agent?** The protections that must be in place before you dare run a swarm autonomously are treated in sibling files (`anti-loop-protection.md`, `forced-context.md`, `knowledge-base-in-tests.md`).

## Why

### Why the duality exists at all

In the old Kilo Code on the Cline core, swarm support was effectively broken along with the rest of the stack ("Cline and its forks became a sinking Titanic after the team left to OpenAI" — post 3793). [author-claim] (post 3793) The move to the Open Code core was partly motivated by the fact that **swarm support is the main evolutionary gain** of that rebuild. [author-claim] (post 3793) The prompt-compatibility guarantee — that single-agent prompts also work in swarm mode — is what makes the duality practical: you can pick the shape per task without maintaining two methodologies.

### Why you would want swarm mode

- **Parallel breadth of output.** A 16-subagent Claude Opus swarm, run against GRACE methodology on video training material, produced a book-length methodological text — the orchestrator consolidated the subagents' outputs into a single consistent document. [empirical] (post 3859) The author: "In reality it was written by a swarm of 16 subagents of Claude Opus. They wrote based on the video training. Then the orchestrator consolidated the swarm's work into a book." [author-claim, empirical] (post 3859)
- **Quality of methodological reconstruction at swarm scale.** The same post reports that the swarm output "captured my concepts quite well, and some things the AI even explained in a simpler way." [empirical] (post 3859) i.e., breadth did not cost semantic fidelity in that case.
- **Contrast with no-swarm, narrow context.** The author contrasts this with a single-trajectory attempt (Denis's book, derived from Telegram posts only) where "hallucinations or incorrect understandings by the LLM [are] quite often present, because the AI did not see the holistic context." [empirical] (post 3859) The implication the author draws is not "swarms are always better" but that the **shape of context delivered to the agents** is what matters — a swarm with an orchestrator can, by design, hold a broader and more consistent picture of the source than a single linear pass.

### Why you would want a single agent instead

The sources do not give a categorical answer; they give positional guidance tied to the GRACE phase model (see `../08-workflow-and-phases/architect-code-debug-modes.md`):

- **Architect-phase work** is the natural home of swarms: few, large, structured prompts that generate hundreds of thousands of tokens of design context (`../08-workflow-and-phases/large-rare-prompts-billing.md`). A 16-subagent swarm consolidating a book is that shape at maximum. [author-claim] (post 3793, post 3859; cross-reference to Architect-mode framing)
- **Focused debug work** — single-failure investigation, LDD cycles around one failing trajectory — does not benefit from parallelism in the same way. The self-correction loop is inherently sequential on a single failure trace, and swarm overhead adds coordination cost without adding insight. The sources do not specify an exact boundary, but the consistent framing of "few, large prompts" in the Architect phase versus "feedback layer" in Debug (see `../06-testing/tests-in-self-correction-loop.md`) implies that per-failure Debug work is the single-agent side of the dial. [author-claim, inferential — see note under Evidence] (posts 3793, 3859; tests-in-self-correction-loop cross-reference)

### Why the duality makes methodology portable

Because Open Code preserves prompt semantics across modes, the same GRACE scaffolding — graph, module contracts, function contracts, START-END markup, LINKS/KEYWORDS sections — is legible to both a lone agent and a swarm. You do not have to invent a second methodology for multi-agent work. [author-claim] (post 3793) In practice this is one of the reasons the author moved GRACE onto Open Code's dynamic skills in the first place: the same skill trace (see `../09-tooling/skills-system.md`) can be loaded by one agent or by each subagent. [author-claim] (post 3809 — supporting context)

### Why protections get non-negotiable at swarm scale

When you go from one agent to many, the cost of the autonomous-loop failure mode compounds:

- A single autonomous agent that loops can "burn up to a billion tokens" (post 3695). [author-claim] (post 3695 — cross-reference, `anti-loop-protection.md`)
- A swarm that loops multiplies that by every subagent that is also looping, and adds orchestrator-level loops on top.
- The author calls out swarms and single autonomous agents together on this risk: "your swarm or even a single autonomous agent can just dumbly hang and burn up to a billion tokens." [author-claim] (post 3695)

Therefore **forced context** (`forced-context.md`) and **anti-loop protection** (`anti-loop-protection.md`) are not optional conveniences in a swarm — they are load-bearing. The recommendation "these must be designed into the test framework from the start" (post 3695) applies with particular force when the framework is going to be consumed by many subagents in parallel. [author-claim] (post 3695)

## How

### Operational rule: pick the shape from the phase

Mapped onto GRACE's phase model:

- **Architect / large design-generation work** → swarm-friendly. Precedent: 16 Claude Opus subagents writing a book, orchestrator consolidating. [empirical] (post 3859)
- **Code / module generation** → sources do not specify a categorical preference; either mode works because prompts are identical. The choice is operational (parallel breadth vs. simpler state). [author-claim] (post 3793)
- **Debug / self-correction loop on a specific failure** → single-agent by default, because the loop is one failing trajectory. [inferential — derived from author-claims in post 3793 and post 3695 and from `../06-testing/tests-in-self-correction-loop.md`; the sources do not specify "use a single agent in Debug" as an explicit rule.]

### Operational rule: prompts do not change between modes

In Open Code, the same prompt set can be fed to either shape. This is the explicit selling point the author uses to justify the migration from Cline to Open Code. [author-claim] (post 3793)

> "You can switch from multi-agent to a single agent, without changing prompts, depending on the task." [author-claim] (post 3793)

What this means practically:

- Your GRACE module-contract and function-contract templates are written once.
- Your skill trace (see `../09-tooling/skills-system.md`) is written once.
- The shape decision — swarm vs single — is a runtime configuration, not a content rewrite.

### Operational rule: an orchestrator is required above the swarm

The only swarm pattern the sources describe explicitly involves an **orchestrator** that consolidates subagent output:

> "In reality it was written by a swarm of 16 subagents of Claude Opus. They wrote based on the video training. Then the orchestrator consolidated the swarm's work into a book." [author-claim, empirical] (post 3859)

The sources do not specify the orchestrator's implementation details (how it dispatches tasks, how it merges outputs, what prompt it runs, how conflicts are resolved). **The sources do not specify.** What they do specify is that the orchestrator's role is not optional — raw swarm output without consolidation is not the deliverable in the described case.

Contrast with a single-trajectory approach without an orchestrator, where the author observed more hallucinations "because the AI did not see the holistic context" (post 3859). [empirical] (post 3859) The lesson the author draws is that a swarm without an orchestrator to stitch the context back together loses the very advantage the swarm was supposed to provide.

### Operational rule: protections must exist before you run the swarm autonomously

Before dispatching a multi-agent swarm against a GRACE project in autonomous mode, the following must already be in place in the framework:

1. **Forced context** — key log lines, prior-fix KB entries, and critical constraints are pushed into each subagent's context rather than waited on. See `forced-context.md` (T36). [author-claim] (post 3695)
2. **Anti-loop protection** — attempt counters plus context injections built into the test framework itself. See `anti-loop-protection.md` (T35). [author-claim] (post 3695)
3. **Knowledge base of prior fixes** — regression memory that anti-loop injections draw from. See `knowledge-base-in-tests.md` (T37). [author-claim] (post 3695)
4. **AI-friendly logs** with log-to-code correlation, so a subagent reading a failing trajectory can actually bind the log back to the code it concerns. See `../05-logging-ldd/log-to-code-correlation.md` (T27) and `../05-logging-ldd/belief-state-logging.md` (T28). [author-claim] (post 3693)

The author is explicit that **regular autotests are not compatible with autonomous agents** without this stack (post 3695), and swarm scale is where the incompatibility becomes the most expensive. [author-claim] (post 3695, cross-reference `../06-testing/classical-tests-antipattern.md`)

### Operational rule: broader context delivery beats raw parallelism

The Denis-vs-orchestrated-swarm contrast in post 3859 is not "more agents = better" but **"more coherent context delivered to each agent = better"**:

- The orchestrated swarm reportedly captured the author's concepts well. [empirical] (post 3859)
- The single linear pass (Denis) hallucinated and misunderstood because it lacked "the holistic context." [empirical] (post 3859)

The operational reading: use a swarm when its architecture can deliver broader and more consistent context per subagent than a single agent can hold; not when it simply runs more subagents in parallel against the same narrow view.

### Operational rule: prompt-framework discipline is what makes models interchangeable

A GRACE-strong pipeline makes models nearly fungible — the Vlad CRM case (Case A, `../11-case-studies/vlad-crm-and-contracts.md`) reports that under GRACE markup and specs, Gemini Flash, Gemini Pro, Claude Opus, GLM-5, and Kimi "all generated equally well," and the author's summary rule is that high sensitivity to model choice signals a weak pipeline, not a strong model. [empirical, author-claim] (practical_applications.md §"Пример CRM") The swarm/single-agent duality in Open Code inherits the same property: you should not be retuning prompts per shape, and if your prompts stop working when you change shape, the shape is not your problem — the pipeline is. [author-claim] (post 3793, practical_applications.md §"Пример CRM" cross-reference)

## Evidence

Primary sources for this file:

- **Post 3793** (2026-04-01) — Kilo Code migration from Cline to Open Code. Specifies:
  - Open Code as ~2× more token-efficient than Cline core.
  - Direct Gemini API access via VPN (no OpenRouter proxy).
  - Explicit statement that swarm support is the main evolutionary gain.
  - Explicit statement that Open Code lets you switch from multi-agent to single agent **without changing prompts**, depending on task.
- **Post 3859** (2026-04-10) — The two-books comparison. Specifies:
  - The 16-subagent Claude Opus swarm that produced a book from video-training material.
  - The role of the orchestrator in consolidating swarm output.
  - The contrast with a narrower single-pass book (Denis's) that hallucinated due to lack of holistic context.
  - The author's verification that the orchestrated-swarm book captured GRACE concepts well.

Supporting / adjacent posts (used only for cross-referencing, not for primary claims in this file):

- **Post 3695** — anti-loop protection, forced context injections, KB-of-previous-fixes integration; scope of the "billion tokens" cost model that makes protections non-negotiable at swarm scale. Treated fully in `anti-loop-protection.md`, `forced-context.md`, `knowledge-base-in-tests.md`.
- **Post 3693** — log-to-code correlation and belief-state logging; the reading surface any swarm relies on.
- **Post 3809** — GRACE migrated onto Open Code's dynamic skills with forced skill-tracing. Relevant as the reliability mechanism that makes the same skill chain usable by both a lone agent and every subagent.
- **practical_applications.md §"Пример CRM"** — the Vlad CRM observation that a strong GRACE pipeline makes models interchangeable; used here as supporting evidence for the "prompt-framework discipline beats shape choice" rule.
- **practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"** — Kilo Code's Architect/Code/Debug mode UI; the phase mapping used above to place swarm vs single agent by phase. Treated fully in `../08-workflow-and-phases/architect-code-debug-modes.md`.

Classifications used:

- **[author-claim]** — framework statements by the author (prompt-compatibility across modes, swarm support as main evolutionary gain, protections being non-negotiable at swarm scale).
- **[empirical]** — the author's own verified observations: the 16-subagent swarm-produced book captured his concepts; the Denis book showed context-starvation hallucinations. Both are the author's observations of external artifacts, not controlled experiments.
- **[inferential]** — the "Debug is single-agent by default" reading is inferred from the combination of the single-failure-trace logic of self-correction loops and the author's Architect-first framing of swarms. The sources do not specify a categorical "use single agent in Debug" rule; this is noted in-place above.

The sources do not specify:

- Orchestrator implementation details (dispatch logic, merge strategy, conflict resolution).
- The exact boundary (in task size, subagent count, or LOC) at which swarm beats single-agent or vice versa.
- Benchmark numbers comparing swarm vs single-agent on the same GRACE task.
- Whether the 16-subagent count is a tuned number or an incidental one for the book experiment.

Where the text above makes a claim beyond what the sources specify, it is explicitly flagged (see "inferential" notes under **How** and in this section).

## See also

- `anti-loop-protection.md` — counters + context injections without which swarm mode burns tokens at catastrophic scale.
- `forced-context.md` — push-over-pull principle for stabilizing autonomous agents at any shape.
- `knowledge-base-in-tests.md` — regression memory that forced injections draw from; becomes especially important when multiple subagents are running against the same failure class.
- `../05-logging-ldd/log-to-code-correlation.md` — the logging heuristic any swarm subagent relies on to bind logs back to code.
- `../05-logging-ldd/belief-state-logging.md` — AI-authored logs that let subagents (and the orchestrator) reason about what the code believes vs. what happened.
- `../06-testing/tests-in-self-correction-loop.md` — tests as Debug-phase feedback; why this phase is the single-agent-by-default side of the dial.
- `../06-testing/classical-tests-antipattern.md` — why classical TDD is hostile to autonomous agents regardless of shape.
- `../08-workflow-and-phases/architect-code-debug-modes.md` — Kilo Code's Architect/Code/Debug UI, the phase map that motivates the shape choice here.
- `../08-workflow-and-phases/large-rare-prompts-billing.md` — few, large prompts as the economics of Architect mode (the natural home of swarms).
- `../09-tooling/kilo-code-open-code.md` — the Cline → Open Code migration, ~2× token efficiency, and prompt-compatibility across shapes.
- `../09-tooling/skills-system.md` — skill tracing that lets the same prompt stack load reliably under any shape.
- `../11-case-studies/vlad-crm-and-contracts.md` — empirical evidence that a strong GRACE pipeline makes models fungible, which is what lets shape be a runtime decision.
- `../13-antipatterns/all-antipatterns.md` — entries on "Assuming the agent will self-rescue via Tools" and on running autonomous agents against classical TDD suites.
