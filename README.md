# GRACE Knowledge Base

A fact-based, cited knowledge base on **GRACE** — **Graph-RAG Anchored Code Engineering** — a methodology for generating and maintaining production code with LLM agents at scale.

## What is GRACE

GRACE is the name an author on the Russian Telegram channel *turboproject* gave to the full-pipeline methodology he distilled from several months of experimentation with 200+ practitioners in his training programs. The name is load-bearing: GRACE pairs an explicit **semantic graph** over code with a **retrieval-augmented** workflow tuned to how transformer attention actually works past ~4k tokens. The goal is to keep a 100%-AI-generated codebase coherent up to current ceilings of ~100,000 LOC (and a projected 400-500k LOC per operator managing a bot swarm within 1-2 years). The author positions it as "something like RUP for the AI era". For the full introduction, see [`00-foundations/what-is-grace.md`](00-foundations/what-is-grace.md).

## Applying GRACE with Claude Code

This KB documents the methodology's *why*. The canonical packaged *how* is the **`osovv/grace-marketplace`** Claude Code plugin (v3.10.0, MIT) — 14 skills covering the full lifecycle: `grace-init` / `grace-plan` / `grace-execute` / `grace-multiagent-execute` / `grace-verification` / `grace-reviewer` / `grace-fix` / `grace-refresh` / `grace-refactor` / `grace-status` / `grace-ask` / `grace-explainer` / `grace-setup-subagents` / `grace-cli`.

Install:

```
/plugin marketplace add osovv/grace-marketplace
/plugin install grace@grace-marketplace
```

A local clone of the plugin repo lives at `../grace-marketplace/` (sibling of `kb/`). For the full skill inventory, artifact scaffold, install variants, and a note on where the plugin's scope exceeds the KB author's explicit audit, see [`09-tooling/claude-code-grace-plugin.md`](09-tooling/claude-code-grace-plugin.md).

## How this KB is organized

Each chapter has its own directory. Every file follows the `What / Why / How / Evidence / See also` skeleton, with every factual claim cited to a source post or file and labeled by claim type (`[empirical]`, `[research-backed]`, `[author-claim]`, `[vendor-doc]`).

| Chapter | Purpose |
|---|---|
| [`00-foundations/`](00-foundations/) | What GRACE is, why it exists, and its first principles. Start here for orientation. |
| [`01-transformer-foundations/`](01-transformer-foundations/) | The transformer internals GRACE is designed around: sparse attention, KV cache, TC-zero / Dyck Language, LLM coder-training pipeline, XML vs JSON vs YAML. |
| [`02-semantic-graph/`](02-semantic-graph/) | The graph as persistent scaffolding: anchors for sparse attention, spec-to-code hierarchy, hybrid graph-plus-vector RAG. |
| [`03-contracts/`](03-contracts/) | AI-oriented micro-specifications attached to code — field spec (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS, RATIONALE/AAG), wenyan-style compression, legacy recovery, and language-specific (Python, C#) placement rules. |
| [`04-markup-system/`](04-markup-system/) | XML-like markup rules, paired START/END tags for Dyck compatibility, in-source-documentation lineage, validators, and empirical compression bounds. |
| [`05-logging-ldd/`](05-logging-ldd/) | Log-Driven Development: belief-state logging, log-to-code correlation tokens, forced context injection. |
| [`06-testing/`](06-testing/) | Why classical TDD hurts AI; tests relocated to the self-correction loop; developer/tester-agent split; in-app AI console; CLI (bash/pytest) over custom MCPs. |
| [`07-autonomous-agents/`](07-autonomous-agents/) | Anti-loop protection, forced context, regression-memory knowledge bases, and swarm-vs-single-agent trade-offs. |
| [`08-workflow-and-phases/`](08-workflow-and-phases/) | The three GRACE phases (Architectural graph, Module contracts, Function contracts + code), Architect/Code/Debug mode mapping, and prompt-billing economics. |
| [`09-tooling/`](09-tooling/) | Concrete tools: Kilo Code / Open Code, the osovv/grace-marketplace Claude Code plugin, Cursor's chunked-read limits, MCP scepticism, reliable skill-loading, hybrid-RAG storage (sqlite-vec). |
| [`10-beyond-code/`](10-beyond-code/) | Extending GRACE beyond code: document-RAG (bank cases), methodology generalization. |
| [`11-case-studies/`](11-case-studies/) | Reported real-world projects: Kirill's 132k LOC migration, Vlad's CRM and contracts project, Alexey's 300k LOC codebase, Mikhail's bank RAG, Qwen SLM viability, the self-hosted GRACE-markup validator. |
| [`12-economics-strategy/`](12-economics-strategy/) | Why big labs will not build a full-pipeline GRACE framework, project-size trends, and the framework-designer-vs-user division of labor. |
| [`13-antipatterns/`](13-antipatterns/) | Consolidated catalog of what breaks at scale: vibe-coding, classical TDD with AI, external docs systems, kulibin MCPs, YAML configuration, JSON in long context, "Captain Obvious" comments, and more. |
| [`14-research-references/`](14-research-references/) | Annotated bibliography of every research paper, vendor guide, and external reference cited across the KB. |
| [`GLOSSARY.md`](GLOSSARY.md) | Master glossary of GRACE terms, acronyms, and Russian source terms (PCAM, LDD, FIM, SFT, RL, PBS, RATIONALE/AAG, Dyck Language, TC-zero, KV cache, kulibin, wenyan, etc.). |
| [`INDEX.md`](INDEX.md) | Full cross-reference topic map, reverse lookup from source posts, and research-paper-to-file map. |

## Reading paths by audience

Pick the lane that matches why you opened this KB. Each lane lists files in reading order.

### "I just want the core idea"

1. [`00-foundations/what-is-grace.md`](00-foundations/what-is-grace.md) — name, positioning, scope.
2. [`00-foundations/why-grace-exists.md`](00-foundations/why-grace-exists.md) — the problem GRACE solves (the 5,000-10,000 LOC collapse threshold).
3. [`00-foundations/core-principles.md`](00-foundations/core-principles.md) — the seven first principles.
4. [`02-semantic-graph/graph-as-backbone.md`](02-semantic-graph/graph-as-backbone.md) — why the graph is not optional.
5. [`03-contracts/contracts-overview.md`](03-contracts/contracts-overview.md) — why contracts are the second load-bearing layer.

### "I need to implement this today"

1. [`08-workflow-and-phases/top-down-pipeline.md`](08-workflow-and-phases/top-down-pipeline.md) — the three phases in order.
2. [`08-workflow-and-phases/architect-code-debug-modes.md`](08-workflow-and-phases/architect-code-debug-modes.md) — mapping onto Kilo Code modes.
3. [`08-workflow-and-phases/large-rare-prompts-billing.md`](08-workflow-and-phases/large-rare-prompts-billing.md) — prompt economics (few-large-structured prompts, ~20 prompts generating 300-400k tokens per design session).
4. [`03-contracts/contract-fields.md`](03-contracts/contract-fields.md) — the canonical field spec with a concrete Python example.
5. [`09-tooling/`](09-tooling/) — concrete tool choices (Kilo Code / Open Code, Claude Code plugin, skill-system reliability, hybrid-RAG storage).

### "I want the research backing"

1. [`14-research-references/annotated-bibliography.md`](14-research-references/annotated-bibliography.md) — every cited paper, vendor guide, and URL, with what each proves and how GRACE uses it.
2. [`01-transformer-foundations/sparse-attention-and-kv.md`](01-transformer-foundations/sparse-attention-and-kv.md) — the ~4k-token dense-attention boundary and sparse-attention sliding windows.
3. [`01-transformer-foundations/tc0-and-dyck-language.md`](01-transformer-foundations/tc0-and-dyck-language.md) — the formal basis for preferring XML over YAML.
4. [`01-transformer-foundations/llm-training-pipeline.md`](01-transformer-foundations/llm-training-pipeline.md) — the four-phase LLM coder training (Base coder, FIM, SFT, RL) and why external docs are ignored.
5. [`01-transformer-foundations/xml-vs-json-vs-yaml.md`](01-transformer-foundations/xml-vs-json-vs-yaml.md) — OpenAI's and Google's format guidance combined with the Dyck-Language argument.

### "I'm debugging agents that don't scale"

1. [`05-logging-ldd/log-driven-development.md`](05-logging-ldd/log-driven-development.md) — reading trajectories instead of asserting equality.
2. [`05-logging-ldd/log-to-code-correlation.md`](05-logging-ldd/log-to-code-correlation.md) — the archimportant heuristic: inject function-ID and block-ID tokens into every log line.
3. [`05-logging-ldd/belief-state-logging.md`](05-logging-ldd/belief-state-logging.md) — verbalizing the AI's hypothesis about its own code.
4. [`05-logging-ldd/forced-context-injection.md`](05-logging-ldd/forced-context-injection.md) — pushing critical log lines into context rather than hoping the agent fetches them.
5. [`07-autonomous-agents/anti-loop-protection.md`](07-autonomous-agents/anti-loop-protection.md) — attempt counters and context injection to stop swarms burning billions of tokens.
6. [`07-autonomous-agents/forced-context.md`](07-autonomous-agents/forced-context.md) — the stabilization mechanism.
7. [`07-autonomous-agents/knowledge-base-in-tests.md`](07-autonomous-agents/knowledge-base-in-tests.md) — regression memory wired into the test harness.
8. [`06-testing/classical-tests-antipattern.md`](06-testing/classical-tests-antipattern.md) — why classical TDD actively hurts AI agents.
9. [`06-testing/tests-in-self-correction-loop.md`](06-testing/tests-in-self-correction-loop.md) — tests as a bug-fix signal, not as a requirements artifact.

## Methodology note

This KB was built mechanically from a fixed set of source files using a task-graph-and-waves build plan. Every file follows the `What / Why / How / Evidence / See also` skeleton; every factual claim is cited to a specific source post (for example `(post 3693)`) or to a named section of a source file (for example `(most_important.md §"Top-down logika")`) and carries a claim-type label. Nothing in this KB is invented: where the sources do not specify a detail, the file records "the sources do not specify" rather than guess. For the full build plan — task list, claim-type conventions, execution waves, and completion tracking — see [`PLAN.md`](PLAN.md).

## Source attribution

This knowledge base was compiled from Russian-language posts published on the Telegram channel **turboproject** (author anonymous on this repo). The original posts are retained verbatim in `../output_files/` and include:

- `most_important_output.md` — six foundational GRACE posts (formal introduction, top-down logic, graph-as-navigation, IDE 2.0, methodology-as-graph, etc.).
- `contracts_as_anchor_output.md` — three contract-focused posts (contracts as semantic shield, legacy recovery, contrast with classical DbC).
- `practical_applications_output.md` — four operational posts (Claude Code plugin, Kilo Code workflow, billing, Vlad's CRM case).
- `testing_and_autonomous_agents_output.md` — five posts on testing philosophy and autonomous agents.
- `xml_mapping_anchors_output.md` — two posts on XML-like markup and log anchors.
- `grace_matches.md` — 29 recent posts (2026-03-14 to 2026-04-23), the richest source, containing all new research citations, the concrete contract-field specification, the Dyck-Language argument, the case studies, and autonomous-agent mechanics.

Russian technical terms are preserved where they carry meaning the English gloss would lose (for example **kulibin** for DIY garage engineering, **wenyan** for literary-Chinese compression). See [`GLOSSARY.md`](GLOSSARY.md) for the full list.
