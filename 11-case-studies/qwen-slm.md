# Case study — Qwen 3.5-27B: GRACE on a Small Language Model

## What

A practical trial of running the GRACE methodology on a **Small Language Model (SLM)** rather than on a frontier Large Language Model. The model under test was **Qwen 3.5-27B** (27 billion parameters), reported to score **72.4% on SWE-Bench Verified** — a number the author flags as "previously unthinkable" for a model of that size only six months earlier [empirical] (post 3902).

The case was observed during one of the author's training sessions, where GRACE was exercised end-to-end on Qwen 3.5-27B to see whether the methodology — normally validated on much larger frontier models — would survive a shrink to SLM scale [empirical] (post 3902).

## Why

Most GRACE posts concern Claude Opus / Gemini Pro / GPT-5.4-tier models. A working engineer looking at GRACE naturally asks: **does this still work on smaller, cheaper, locally hostable models?** The Qwen 3.5-27B run answers that question and also gives a blueprint for where SLMs fit in a GRACE-style pipeline [author-claim] (post 3902).

Two broader points are made through this case:

1. **SLMs are becoming working tools** rather than toys — the author states that small networks are becoming real instruments for developers, and that the practical job is finding them the right niche [author-claim] (post 3902).
2. **GRACE's value scales downward**, not only upward: because the methodology embeds intent and navigation directly in code, the less the model "knows" from its own weights, the more it benefits from explicit scaffolding. In this run the SLM was observed to be **more disciplined with in-source documentation than larger LLMs** [empirical] (post 3902).

## How

### The model

- Qwen 3.5-27B, open weights, 27 billion parameters [empirical] (post 3902).
- Reported benchmark: **SWE-Bench Verified 72.4%** [empirical] (post 3902).
- The author treats that benchmark as credible for this IQ level, not "painted" / cherry-picked: "SWE-Bench Verified is definitely not 'painted' there for such IQ" [author-claim] (post 3902).
- Reference: huggingface.co/Qwen/Qwen3.5-27B (post 3902).

### What went well

- The SLM **handled a complex set of instructions** given to it under GRACE cleanly [empirical] (post 3902).
- It produced **in-source documentation** (GRACE-style contract markup embedded in code) with high discipline — in the author's phrasing, **even more disciplined than larger LLMs** on this specific behavior [empirical] (post 3902).
- Practical implication: the "follow LLM's own patterns" core principle (see `../00-foundations/core-principles.md`) applies to SLMs too; the PURPOSE / INPUTS / OUTPUTS / KEYWORDS scaffolding from `../03-contracts/contract-fields.md` is honored by Qwen 3.5-27B in generated code [empirical] (post 3902).

### What failed — and why that was expected

- One training example did **not** complete successfully: the SLM did not know how to program **Gradio** (a Python UI framework) precisely [empirical] (post 3902).
- The author frames this as **normal and expected**: specialized framework knowledge cannot physically fit in 27B parameters — such a model holds only base development skills, mostly **Python** and some **React** [author-claim] (post 3902).
- In other words, the failure is a **knowledge-coverage failure**, not a methodology failure. GRACE markup was applied correctly; the model simply lacked the factual weight-stored Gradio API knowledge.
- This matches the broader GRACE argument that model selection is a niche-fitting problem (cross-link to `../12-economics-strategy/vendor-strategy.md`): SLMs occupy a different niche than frontier models, not an inferior version of the same niche.

### The suggested niche for SLMs in GRACE workflows

Based on this observation, the author's recommendation [author-claim] (post 3902):

- **Not recommended** for unconstrained greenfield coding in specialized frameworks — weight capacity is simply too small to hold niche framework knowledge, and you will hit "edge" cases like the Gradio failure.
- **Recommended** for:
  - **Writing tests** — the SLM can produce proper, disciplined test code.
  - **Producing bug reports** — normal, well-structured bug reports are within SLM capability.
  - **Fixing bugs in existing code** — when the surrounding code already exists and supplies the framework context as in-context few-shots (see `../03-contracts/ai-contracts-vs-dbc.md` on SFT-aligned contracts), the SLM can act on it.

The SWE-Bench Verified number directly supports the last use case: SWE-Bench rewards exactly "read the repo + logs + tests → emit a patch" behavior (cross-link to `../06-testing/tests-in-self-correction-loop.md` for the self-correction loop where SWE-Bench-style RL trains patch-emission, and to `../01-transformer-foundations/llm-training-pipeline.md` §RL for the SWE-Bench training regime).

### Why GRACE helps an SLM specifically

The sources do not enumerate this explicitly, but the case's logic follows GRACE's first principles:

- GRACE's central mechanism — a **semantic graph + contracts in source code** (see `../02-semantic-graph/graph-as-backbone.md` and `../03-contracts/contracts-overview.md`) — acts as an **external memory substitute**. Where the frontier model might "recall" a framework from weights, the SLM cannot; but contracts, KEYWORDS, and LINKS give the SLM the same anchor scaffolding an LLM would have built internally, **without requiring that the knowledge live in weights** [author-claim / extrapolation from post 3902 + post 3878 + post 3693].
- Sparse-attention navigation (see `../01-transformer-foundations/sparse-attention-and-kv.md`) is a hardware-bound constraint; it applies to SLMs and LLMs alike, so the GRACE top-down graph anchors help both.
- The belief-state / log-to-code correlation mechanics (see `../05-logging-ldd/log-to-code-correlation.md` and `../05-logging-ldd/belief-state-logging.md`) are model-agnostic — they work because they give *any* model a stronger token-level correlation between code and runtime.

## Evidence

- Source post: **post 3902** (2026-04-14 19:38 UTC, `grace_matches.md`).
- Verbatim claim: Qwen 3.5-27B scored **72.4% on SWE-Bench Verified** [empirical] (post 3902).
- Verbatim claim: the SLM produced in-source documentation **"по дисциплине даже лучше LLM"** — "more disciplined than LLMs" [empirical] (post 3902).
- Verbatim claim: the Gradio failure was expected because **27B weights cannot hold private-framework knowledge, only base Python / React** [author-claim] (post 3902).
- Verbatim claim: SLMs "are becoming working tools; the key is finding them the right niche" [author-claim] (post 3902).
- Reference (vendor link as stated by the author): huggingface.co/Qwen/Qwen3.5-27B [vendor-doc] (post 3902).

Notes on what the sources do NOT specify:

- The sources do not specify the exact training-example application beyond "it used Gradio" — no full task description is given.
- The sources do not specify exact token counts, latency, or cost comparisons for the SLM run.
- The sources do not specify which GRACE components (Architect/Code/Debug modes, LDD, agent-based testing) were and were not used in this particular SLM run.
- The sources do not specify benchmark methodology for the 72.4% SWE-Bench Verified number beyond the vendor figure.

## See also

- `../00-foundations/core-principles.md` — "Follow LLM's own patterns" as a first principle; applies to SLMs equally.
- `../01-transformer-foundations/llm-training-pipeline.md` — the four-phase training pipeline (Base → FIM → SFT → RL/SWE-Bench) that makes SWE-Bench Verified meaningful for the "fix bugs in existing code" niche.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — why the graph + anchors matter for any model past 4k tokens, SLM or LLM.
- `../02-semantic-graph/graph-as-backbone.md` — the external scaffolding that a weight-limited model relies on heavily.
- `../03-contracts/contracts-overview.md` and `../03-contracts/contract-fields.md` — the contract fields Qwen 3.5-27B was observed to follow with "more discipline than LLMs".
- `../05-logging-ldd/log-to-code-correlation.md` — log-to-code correlation as model-agnostic anchor support.
- `../06-testing/tests-in-self-correction-loop.md` — the SWE-Bench-style RL regime that underwrites the "SLM for bug fixing" niche.
- `../12-economics-strategy/vendor-strategy.md` — model-selection economics and the niche-fitting framing.
- `./kirill-132k-loc.md` — contrast: a large codebase run on LLM-tier models (GPT-5.4(xh) orchestrator + GPT-5.4(m) workers).
- `./alexey-300k-loc.md` — contrast: the ceiling of current GRACE output, on LLM-tier infrastructure.
