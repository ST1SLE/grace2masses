# What is GRACE

## What

**GRACE** stands for **Graph-RAG Anchored Code Engineering** — a methodology for automatic generation of production code by LLM agents at scale. The name is load-bearing: the "RAG" component is not decorative. GRACE is built around the combination of an explicit semantic **graph** over code plus a **retrieval-augmented** workflow for agents operating inside that graph. [author-claim] (most_important.md §"Формальное введение в GRACE")

Positioning: the author describes GRACE as **"something like RUP for the AI era"** — a full-pipeline methodology for AI-generated code, comparable in ambition to the Rational Unified Process in its time. [author-claim] (most_important.md §"Формальное введение в GRACE")

GRACE is a **general methodology for complex text**, not only code. The templates ship standardized for code, but the approach — graph-first, semantic scaffolding, anchored retrieval — extends to any domain where LLM agents must navigate large, interconnected artifacts (banking documents, construction-project methodology, etc.). [author-claim] (grace_matches.md post 3839; most_important.md §"Методология как граф")

GRACE is **compatible with the author's companion methodology PCAM** (the agent-building methodology), but PCAM is **not a prerequisite**. GRACE is specifically a technology of large-code generation plus the AI's ability to keep the project under control without piling up the kind of "technical debt" new AI-coders typically accumulate. [author-claim] (most_important.md §"Формальное введение в GRACE")

## Why

GRACE exists because most ad-hoc AI coding efforts collapse past a single working module — a failure the author observed across more than 200 practitioners in his training programs. [empirical] (most_important.md §"Формальное введение в GRACE") The methodology is the distilled, empirically-validated residue: **"refined over several months on 200+ specialists; only what actually works made it into the final version."** [empirical] (most_important.md §"Формальное введение в GRACE")

GRACE targets a specific operating range:

- **Lower bound — 800–1000 LOC.** Below this, GRACE is "rather overkill"; simple isolated tasks do not benefit much from contracts, and the AI handles them fine without extra scaffolding. [author-claim] (most_important.md §"Формальное введение в GRACE"; grace_matches.md post 3654)
- **Upper bound — the current ceiling of 100%-AI-generated apps.** The author's current observation across "dozens of projects a month" in his training programs: ~**100,000 LOC** is the present-day size of fully-AI-built applications, rising to a projected **400–500k LOC** per operator-managing-a-swarm within 1–2 years. [empirical] (grace_matches.md post 3837) The largest reported project built on GRACE to date is Alexey's **300,000+ LOC** codebase, valued at **$7M+** at US developer rates. [empirical] (grace_matches.md post 3838)
- **Scope of the problem being solved.** GRACE is "more for medium and large Enterprise applications." [author-claim] (most_important.md §"Формальное введение в GRACE")

The methodology is grounded in **specific transformer internals**: the behavior of **sparse attention** past ~4k tokens in GPT models, and the operational profile of **RAG agents** working over code (Cursor-style chunk readers). [research-backed] (most_important.md §"Формальное введение в GRACE"; §"Почему без семантического графа") For a fuller treatment of these transformer foundations see `../01-transformer-foundations/sparse-attention-and-kv.md`.

A secondary "why" is disclosure. GRACE publicly exposes a technique the author calls **"semantic scaffold of code"** ("семантический каркас кода"), which he claims is **currently locked behind heavy NDAs at the largest IT corporations** for competitive-advantage reasons. [author-claim] (most_important.md §"Формальное введение в GRACE")

## How

GRACE's basic shape — expanded in detail throughout the rest of this knowledge base:

1. **Graph first.** The AI models the application as a graph of main classes/modules, and simultaneously maps business-process modeling onto that graph. The graph is the persistent scaffolding from which everything else descends. [author-claim] (most_important.md §"Top-down логика"; §"Граф как основа навигации") See `../02-semantic-graph/graph-as-backbone.md`.
2. **Contracts second.** From the graph, the AI produces a **module contract** per module, then function-level contracts — each one a local refinement of the overall application intent at that site. Contracts are a **semantic shield** the AI cross-checks against when modifying code. [author-claim] (contracts_as_anchor.md §"Почему контрактное программирование") See `../03-contracts/contracts-overview.md`.
3. **Code third.** Only after graph and contracts exist does the AI emit classes and methods. The AI needs to be **actively assisted through this top-down sequence** — left to itself it jumps straight to code. [author-claim] (most_important.md §"Top-down логика")

Two practical consequences of this shape at the GRACE scale:

- **Zero-doc.** Graph + module contract + function contracts **fully replace traditional external documentation**: architecture and business model are embedded directly in the code in a form the AI understands. [author-claim] (most_important.md §"Top-down логика")
- **Vendor-agnostic stability.** Strong semantic scaffolding compresses vendor-to-vendor differences: in Vlad's CRM case, switching between Gemini Flash, Gemini Pro, Claude Opus 4.6, GLM-5, and Kimi K2.5 produced no appreciable quality delta — "all generate equally well." [empirical] (practical_applications.md §"Пример CRM") If you *do* feel high LLM sensitivity, the author's position is that your pipeline is weak, not the model. [author-claim] (practical_applications.md §"Пример CRM")

What this file does **not** attempt: describe the full field set of a contract, the markup rules, the logging heuristics, etc. Those are owned by the downstream files in this KB.

## Evidence

**Origin and positioning**

- Formal introduction of GRACE by the author: see `most_important.md §"Формальное введение в GRACE"` — this is the canonical announcement post declaring GRACE as "Graph-RAG Anchored Code Engineering" and framing it as "something like RUP for the AI era." [author-claim]
- Empirical refinement: **200+ specialists on the author's training programs; only empirically-working parts retained.** [empirical] (most_important.md §"Формальное введение в GRACE")

**Scale evidence**

- Kirill's pre-stage product: **1494 files, 132,666 production LOC + 50,026 test LOC, 11,481 documentation lines**. After adopting GRACE markup and modernized logging: agents read fewer unnecessary files, zero rollbacks during the observation period, less hallucination, per-file tokens up but total tokens down. Two latent bugs surfaced during markup annotation. [empirical] (grace_matches.md post 3620)
- Vlad's CRM (security-company CRM for 5 firms, 1000+ personnel per shift, Next.js + shadcn + Supabase): cross-vendor model-swapping with flat quality. [empirical] (practical_applications.md §"Пример CRM")
- Vlad's contracts project: **282 contracts, 1049 autotests** — a non-toy project, though lacking the log-driven part of GRACE, which motivated the log-to-code-correlation posts. [empirical] (grace_matches.md post 3693)
- Alexey's project: **300,000+ LOC** built on GRACE; author estimates **$7M+** equivalent at US dev rates. [author-claim] (grace_matches.md post 3838)

**Generalization beyond code**

- Mikhail Evdokimov (CPO ALGA Group International; 1st Deputy Chair of O!Bank) built a RAG system on GRACE principles for documents, using a graph-based main map plus per-document "semantic squeeze" analogous to code contracts. The author notes **at least three private bank cases** with similar approaches. [empirical] (grace_matches.md post 3839; reference: habr.com/ru/articles/1020548/)
- The author has applied the same graph-first methodology to **construction-work planning** (MS Project methodology graph), reinforcing GRACE as a general methodology rather than a code-specific trick. [author-claim] (most_important.md §"Методология как граф")

**NDA claim about "semantic scaffold of code"**

- Author's assertion that this technique is **currently under NDA at the largest IT corporations** for competitive reasons, and GRACE is its first public exposure. [author-claim] (most_important.md §"Формальное введение в GRACE") The sources do not specify which corporations or present any external corroboration.

**Transformer grounding (pointer, not detailed here)**

- Sparse-attention collapse past ~4k tokens and the role of semantic graphs as "table of contents" for sliding-window attention: see `../01-transformer-foundations/sparse-attention-and-kv.md`. [research-backed] (most_important.md §"Почему без семантического графа")

## See also

- `./why-grace-exists.md` — the problem GRACE solves (critical-mass threshold, sparse-attention collapse, RAG blindness).
- `./core-principles.md` — the first principles in condensed form.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — why large code contexts collapse without anchors.
- `../02-semantic-graph/graph-as-backbone.md` — graph as persistent scaffolding.
- `../03-contracts/contracts-overview.md` — contracts as semantic shield.
- `../08-workflow-and-phases/top-down-pipeline.md` — the three-phase pipeline.
- `../10-beyond-code/grace-for-rag-documents.md` — GRACE applied to bank RAG.
- `../10-beyond-code/general-complex-text.md` — methodology generalization (construction, etc.).
- `../11-case-studies/kirill-132k-loc.md` — large codebase adoption case.
- `../11-case-studies/vlad-crm-and-contracts.md` — Vlad CRM + contracts cases.
- `../11-case-studies/alexey-300k-loc.md` — largest reported project.
- `../11-case-studies/mikhail-bank-rag.md` — bank RAG case.
- `../GLOSSARY.md` — term definitions (GRACE, PCAM, RAG, etc.).
