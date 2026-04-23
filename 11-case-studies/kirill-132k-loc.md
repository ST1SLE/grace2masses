# Kirill's 132k-LOC Pre-Stage Project — GRACE Markup Adoption

## What

This case study documents Kirill Shutov's empirical experience rolling out GRACE-style semantic markup and Log Driven Development (LDD) concepts onto an existing pre-stage (pre-production) product. [empirical] (post 3620). The colleague reported the experience in the author's Telegram chat; the author then re-shared it as a public post along with scale context. [empirical] (post 3620)

At a glance, the case covers:
- A real, non-toy codebase at "pre-stage contour" — i.e. a project already sitting behind the developer's production-like pipeline, not a fresh pet-project. [empirical] (post 3620)
- A multi-week reading phase followed by a structured rollout through Kilo Code orchestration. [empirical] (post 3620)
- Measurable behavioral changes in the AI agents after markup coverage. [empirical] (post 3620)

## Why

This case is a load-bearing data point for GRACE because it shows the methodology operating at a scale well beyond the "small example" regime that survivorship-bias tutorials favor. [author-claim] (post 3654; most_important.md §"Top-down логика")

Specifically:

- GRACE explicitly targets **800–1000+ LOC apps** and is described as meaningful for medium-to-large Enterprise code. [author-claim] (most_important.md §"Формальное введение")
- The author's **critical-mass threshold** claim is that AI understanding starts to collapse around 5,000–10,000 LOC of interconnected logic even in much larger apps. [author-claim] (post 3654)
- Kirill's project is one to two orders of magnitude larger than that threshold, so the reported stability is a direct empirical check on the methodology's scaling story. [empirical] (post 3620)
- The case also validates the author's claim that **standard logging becomes obsolete** once GRACE-style anchors and belief-state log lines are in place. [empirical] (post 3620; cross-ref post 3693)

Without this kind of case study, the GRACE scaling claims would rest only on the author's own projects and smaller trainee work. The Kirill case provides a third-party empirical anchor. [empirical] (post 3620)

## How

### Scale of the project

The post gives precise counts [empirical] (post 3620):

- **Files in the project**: 1494
- **Lines of documentation**: 11,481
- **Lines of code without documentation**:
  - production code: **132,666 LOC**
  - test code: **50,026 LOC**
- **Lines with documentation in code** (comment lines):
  - production code comments: **11,771**
  - test code comments: **1,761**

Total production + test code is ~182,692 LOC. [empirical] (post 3620, arithmetic on reported counts)

The author explicitly framed this as "not some mini-project, but a project at the pre-stage contour". [empirical] (post 3620)

### Preparation phase

Kirill's team spent about **two weeks** reading the author's GRACE posts and related material before touching the code. [empirical] (post 3620)

They then [empirical] (post 3620):
1. Pulled the entire post/context corpus into an AI as analysis input.
2. Had the AI analyze the methodology against the concrete project ("the victim").
3. Produced a carefully-structured prompt describing exactly what had to change and how.

This matches the GRACE doctrine of **few, large, structured prompts** over many small ones (cross-link economics), and of assisting the AI through top-down analysis rather than letting it jump straight to code. [author-claim] (practical_applications.md §"Большие, но редкие запросы"; most_important.md §"Top-down логика")

### Tooling stack

The rollout was executed via **Kilo Code** with its orchestration mode [empirical] (post 3620):

- Orchestrator / architect role: **GPT-5.4(xh)**
- Worker agents: **GPT-5.4(m)**

The choice of Kilo Code is consistent with the author's separate observation that Kilo Code's Architect/Code/Debug mode UI maps cleanly onto GRACE phases. [author-claim] (practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code")

### The 20+ hour autonomous run

After iteratively refining the initial prompt, the team launched what Kirill describes as "**20+ hours of uninterrupted work**" by the agent roster on an initial markup rollout pass. [empirical] (post 3620)

Follow-up behavior [empirical] (post 3620):
- After the long run ended, they studied the output and ran "several additional iterations of corrections and refinements."
- Early in the process it became clear that "AI agents understand very well what markup is, what it is for, and what is required of it."

This is consistent with the author's design premise that **LLMs already know GRACE-style contract patterns from their SFT corpus** — the format sits on patterns the AI was already trained on rather than on a DIY format. [author-claim] (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC")

### Two latent bugs found during markup

During markup coverage the agents surfaced "a couple of small bugs that we did not even know about." [empirical] (post 3620)

This is a side-effect GRACE predicts: forcing the AI to articulate **PURPOSE**, **LINKS**, and related intent fields over existing code is itself a semantic review pass. [author-claim] (post 3878; post 3890 on intent tokens)

### Post-switch observations

After two days of day-to-day use with the new markup in place, Kirill reported [empirical] (post 3620):

- Agents "collect the context they need faster"
- Agents "understand better what is required of them"
- During this working period, **zero reverts/rollbacks** of agent changes were required
- The agents "hallucinate less"
- Per-file token cost went **up**, because each file now carries more in-source documentation
- Total token cost per task went **down**, because agents read **fewer unnecessary files**

This confirms the author's broader claim that the right loss function is not "minimize tokens per file" but "minimize total tokens to solve the task" — which is achieved by making each file semantically self-descriptive so that greedy-reading agents do fewer irrelevant fan-outs. [author-claim] (post 3821 on FIM/find-modules framing; post 3819 on 90% of semantic searches landing on enrichment points)

### Log modernization

In parallel with markup, the team also **modernized the logging**. [empirical] (post 3620)

Kirill's framing — paraphrased from the post — is that "**standard logging is a relic of the past**, where an error or some information is simply printed out", and that new logs must let the AI see not only errors but "all information about how the project works and all its processes in a traceable form." [empirical] (post 3620)

This matches the author's Log Driven Development doctrine [author-claim] (testing_and_autonomous_agents.md §"Адаптация GRACE"; post 3693):
- **log-to-code correlation** (function/block identifier tokens injected into log lines, cross-link to T27)
- **belief-state logging** (agent's verbalized hypothesis about how code works sits in the log, cross-link to T28)

The sources do not specify which concrete logging framework or format Kirill adopted. [empirical] — the post only describes the direction of the change, not the implementation.

### Summary of outcome

Kirill concludes [empirical] (post 3620):
- "Agent-oriented development became better and more effective"
- Even with larger per-file token footprints, "the agents seem to read fewer files that they do not need"
- Team intent: "continue to develop AI-oriented development rather than trying to make AIs work like a human"

## Evidence

### Primary source

- **Post 3620** — 2026-03-14 04:32 UTC — [t.me/turboproject/3620](https://t.me/turboproject/3620). Kirill's full write-up is re-shared in the chat (link `t.me/c/1467914348/117751` in the post). The post also contains the scale block at the bottom with the 1494 / 11,481 / 132,666 / 50,026 / 11,771 / 1,761 figures. [empirical]

### Supporting / related posts

- **Post 3622** — Anton's follow-up in the same thread, proposing a contract-only MCP pattern on legacy code that uses GRACE's START-END markup without AST parsing. Anton's pattern is especially relevant to projects like Kirill's where legacy code is being retrofit. [empirical] (post 3622)
- **Post 3654** — Critical-mass threshold (5,000–10,000 interconnected LOC as the collapse zone) that Kirill's project clears by orders of magnitude. [author-claim] (post 3654)
- **Post 3693** — The full GRACE logging doctrine (log-to-code correlation, belief state, forced context injection) that Kirill's log modernization maps onto. [author-claim] + [empirical] (post 3693)
- **Post 3819** — 90% of semantic searches landing on KEYWORDS/LINKS enrichment points; consistent with Kirill's observation that agents read fewer unnecessary files. [empirical] (post 3819)
- **Post 3878** — Rationale for in-source documentation (PURPOSE-first, before the declaration) that Kirill's markup pass applied. [author-claim] (post 3878)
- **Post 3889** — In-source documentation benchmarks (PURPOSE critical, shorter > longer, -95% to +435% swings) — the scientific underpinning for the observed "less hallucination" effect. [research-backed] (post 3889)

### What the sources do not specify

The following details are not given in post 3620 and are therefore not stated in this file:
- The exact programming language(s) of the 132,666 production LOC.
- The exact domain / industry of the "pre-stage product".
- The specific logging library or log format adopted after modernization.
- Exact commit counts, ticket counts, or elapsed calendar time beyond "two weeks of study" + "20+ hours autonomous" + "a couple of days of post-rollout use".
- Whether the GPT-5.4(xh) and GPT-5.4(m) are internal Kilo Code model slugs or a third-party provider — the sources do not specify. [empirical] (post 3620)

## See also

- `../00-foundations/why-grace-exists.md` — the critical-mass threshold this case clears (post 3654).
- `../00-foundations/core-principles.md` — top-down / in-code-docs / graph-first principles reflected in Kirill's rollout.
- `../02-semantic-graph/graph-as-backbone.md` — why markup + anchors reduce unnecessary file reads.
- `../03-contracts/contracts-overview.md` — contracts as semantic shield for AI during modification.
- `../03-contracts/contract-fields.md` — PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS — fields Kirill's rollout applied.
- `../04-markup-system/start-end-tags.md` — paired tags that made Anton's contract-only MCP possible on this class of project (post 3622).
- `../05-logging-ldd/log-driven-development.md` — LDD overview.
- `../05-logging-ldd/log-to-code-correlation.md` — the "archimportant" heuristic behind Kirill's log modernization.
- `../05-logging-ldd/belief-state-logging.md` — belief-state logging (post 3693).
- `../08-workflow-and-phases/architect-code-debug-modes.md` — Kilo Code mode UI Kirill's team used for orchestration.
- `../09-tooling/kilo-code-open-code.md` — Kilo Code / Open Code migration context.
- `../11-case-studies/alexey-300k-loc.md` — the larger, 300k+ LOC case that sits above Kirill's in the scale ladder (post 3838).
- `../11-case-studies/vlad-crm-and-contracts.md` — Vlad's 282-contracts / 1049-autotest case where markup was applied but logging was missing — useful contrast with Kirill, who modernized both (post 3693).
