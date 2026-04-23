# LLM Coder Training Pipeline — Why AI Ignores External Docs

## What

Modern coding LLMs are built through a four-phase training pipeline. Understanding that pipeline explains a stubborn empirical fact GRACE is built around: **agents ignore external documentation systems** (separate doc MCPs, knowledge graphs about code, dedicated doc services), and when forced to use them, they hallucinate "plausibly" instead [author-claim] (post 3820).

The four phases (post 3820):

1. **Base coder pretraining** — ~80% code, ~20% natural language. The 20% exists so the model can *understand tasks*, not to expose it to code documentation.
2. **FIM (Fill-In-the-Middle)** — complete and total ban on documentation. The only question trained is "how correct is this fragment relative to the surrounding code?"
3. **SFT (Supervised Fine-Tuning)** — pairs of tasks and code. Task ≈ 2–3 paragraphs; code ≈ 200–300 lines. Strict training on Tools chains for popular agents and MCP servers, with an explicit ban on "kulibinshchina" (DIY tooling).
4. **RL (Reinforcement Learning, SWE-Bench-style)** — the model is given bug reports and patches, learns to run isolation tests, and learns to fix code from test signals. Again, no documentation.

GRACE takes this distribution seriously: **documentation is embedded in code** (contracts, in-source markup), not served from the side. The file at `../04-markup-system/in-source-documentation.md` explains the markup layer; this file explains *why* that placement is load-bearing.

## Why

Every training phase above is hostile to out-of-band documentation [research-backed, post 3820]:

- **Phase 1 (80/20 split)**: natural language is budgeted for task comprehension, not for describing code. The model's prior is "code is the source of truth" [author-claim] (post 3820).
- **Phase 2 (FIM)**: documentation is explicitly excluded. The completion target is *correctness relative to surrounding code*, not alignment with any specification text [author-claim] (post 3820). This is also why the author of GRACE claims that a module contract **embedded** in code cannot be ignored by the agent — the contract sits inside the span the model is natively trained to attend to (contracts_as_anchor.md §"Почему контрактное программирование").
- **Phase 3 (SFT)**: tasks are concise (2–3 paragraphs) and paired with concrete code (200–300 lines). The model never sees long, separate doc artifacts paired with code. It also sees only the top tier of agents/MCPs — DIY tooling is explicitly banned in training [research-backed, post 3820].
- **Phase 4 (RL / SWE-Bench)**: fix-from-tests. No documentation in the loop — only bug reports, code, and test signals [research-backed, post 3820].

The practical consequence: **any methodology that routes documentation through a separate service runs against the training distribution.** Agents respond in one of two ways [author-claim] (post 3820):

1. If they are *not forbidden* to read the code directly, they ignore the doc service and load code modules. This is what they were trained to do.
2. If they are *forbidden* from reading code and forced to rely on the external docs, they hallucinate — producing "plausible-looking" answers from the documentation alone.

A related claim the author flags explicitly: the intuition "code is harder for AI than documentation, just like for a human" is an **anthropocentric projection** [author-claim] (post 3820). For a modern coder LLM, Python or Rust is "as readable as English" — the model is not blocked by code illegibility. The real failure mode for bare code is **semantic incompleteness**: code that hides intent, preconditions, or architectural reasoning. GRACE's answer is to fix that incompleteness *inside the code itself*, via contracts, KEYWORDS, LINKS, and RATIONALE sections — not outside it.

See also:
- `../03-contracts/contracts-overview.md` — contracts as the in-code documentation layer that survives FIM training.
- `../03-contracts/ai-contracts-vs-dbc.md` — why AI contracts look more like SFT-shaped docstrings than Hoare-style assertions.
- `../03-contracts/rationale-and-aag.md` — how architectural-intent sections compensate for the semantic incompleteness of bare code.

## How

Practical rules that fall out of the training pipeline:

### 1. Don't build external doc stores for code

A graph database of code metadata, a dedicated "docs-MCP", a retrieval service that returns summaries — all of these misdefine the agent's task [author-claim] (post 3821). The agent's *actual* task, shaped by FIM training, is **"find the modules"** — a navigational task, not a "understand this class" task. Once the right modules are in context, "understanding" is automatic.

FIM trains the agent to build context the way it saw during pretraining: pull whole modules related to the task. Claude Code reinforces this with a "greedy reading" subagent prompt (post 3821). Competing with that — by rerouting the agent through a doc service — means competing with the ten-billion-dollar supercluster month that produced the FIM weights [author-claim] (post 3821). The custom doc service loses.

### 2. Treat MCPs the way SFT did

The SFT phase trained agents strictly against the *top-1000 MCP servers* (this figure is attributed to a Kimi paper in post 3834). Practical rule:

- **Market-leader MCPs work.** Asana and JIRA-tier tools are in the SFT distribution. The author notes that an Asana/JIRA MCP "will very likely work" because such tools sit inside that top-1000 [research-backed, post 3834].
- **DIY MCPs do not.** An agent faced with a custom MCP it was never trained on has a choice the framework often doesn't notice: it can simply *refuse to use the tool* [author-claim] (post 3834). This silent-refusal mode is specific to MCPs — for other tooling it may just hallucinate; for MCPs it can just not call them.

### 3. Prefer Bash over custom MCPs

`bash` is in the standard LLM toolset and was trained on heavily, so the probability that an agent uses it correctly *inside a long trajectory* is far higher than for custom MCPs [author-claim] (post 3834). But "in the standard toolset" is not a guarantee of everything wrapped around it:

> The author's empirical example (post 3834): in an earlier GRACE iteration there were two autotest CLIs — standard `pytest`, and a custom "kulibin" CLI for tests. In practice the agent either didn't call the custom CLI or hallucinated around it. `pytest` worked "like a rock". The custom CLI was removed and the setup moved strictly to industrial-standard tooling [empirical] (post 3834).

Practical implication: if you need to expose extra functionality to an agent, prefer a **CLI wrapper invoked via Bash** over a new MCP. The CLI inherits some of Bash's training support; a custom MCP inherits nothing.

### 4. Fix "semantic incompleteness" by injecting docs into code

This is the core GRACE move, and it follows directly from the training pipeline [author-claim] (post 3820):

- FIM trained the model to trust code → put the spec *in* the code (contracts before the declaration).
- SFT trained the model on 2–3-paragraph tasks with 200–300-line code → contracts are compact, task-shaped, close to the code they annotate (see `../03-contracts/contract-fields.md`).
- RL trained the model on fix-from-test-signals → logs (and structured, "flag"-style test output) are the right channel for correction signals (see `../05-logging-ldd/log-driven-development.md`).

All three map cleanly onto "documentation is IN the code, structured like what the model already saw during training". External doc services map onto nothing in the pipeline.

### 5. If you must serve docs through an MCP, serve *declarations and contracts*, not prose

The author flags one working exception to the "no doc MCP" rule [author-claim] (post 3622, cross-referenced from post 3821): Anton's contract-only MCP. Instead of serving documentation prose, it returns contracts plus function declarations plus implementation line-ranges. That pattern works *because* it still serves what the FIM pipeline trained the model to want — code-adjacent artifacts, not separate doc text. Details belong in `../09-tooling/mcp-scepticism.md`; here it only matters as a boundary case.

### 6. Small summary of the operational rules

| Pipeline phase | What training instilled | GRACE's operational response |
|---|---|---|
| Base coder 80/20 | Code is the primary signal; NL for task intent | Put intent in code (PURPOSE, RATIONALE) |
| FIM, no docs | Agent trusts code, not side-documents | Embed contracts *before* the declaration |
| SFT pairs | 2–3-paragraph task ↔ 200–300-line code | Compact contracts; see `../03-contracts/contract-fields.md` |
| SFT Tools | Strict top-1000 MCPs; ban on DIY | Use Bash + standard CLIs; avoid custom MCPs |
| RL / SWE-Bench | Fix from test signals | LDD + structured log signals; see `../05-logging-ldd/` |

## Evidence

**Post 3820 (grace_matches.md)** — primary source. Enumerates the four training phases with their exact properties:

- "Базовое обучение Coder-моделей. На 80% — только код; задача обучения — заставить ИИ выучить языки программирования. Обучение обычному языку 'кожаных' составляет 20%, но не в контексте 'документации', а чтобы ИИ понимал само задание." [research-backed]
- "FIM-обучение типа 'дополни код'. Полный и тотальный запрет на документацию к коду; только тренировка ИИ понимать, насколько верен этот фрагмент кода относительно остального." [research-backed]
- "SFT-обучение на парах 'задание–код'. Однако это не классическая документация, а задание примерно на 2–3 абзаца и около 200–300 строк кода." [research-backed]
- "На SFT GPT также учат строго и конкретно цепочкам Tools под самые популярные агенты и MCP, с полным и тотальным запретом на 'кулибинщину'." [research-backed]
- "RL-обучение агентов фиксам и мелким доработкам по аналогии со SWE Bench. Никакой документации на код; берутся задания на правки, агент учится проводить больше изоляционных тестов и по ним править код." [research-backed]

Same post on the consequence: agents either skip external doc services and go straight to code, or — if forced — hallucinate plausible-looking answers [author-claim]. Also the anti-anthropocentric point: "для ИИ тот же Python или Rust — такой же язык, как и английский" — code is not harder than documentation for the model; the actual problem with bare code is **semantic incompleteness**, which is what contracts embedded in code are designed to fix [author-claim] (post 3820).

**Post 3821 (grace_matches.md)** — Sergei Muravyev's doc-MCP discussion. Redefines the agent's task: the agent's "primary task is not 'understand the class' but 'find the module'" [author-claim]. FIM biases agents to load whole modules related to the task; Claude Code explicitly reinforces this with a "greedy reading" subagent prompt. The supercluster-month-of-FIM-training argument frames why custom doc MCPs lose — they compete with training directly [author-claim] (post 3821).

**Post 3834 (grace_matches.md)** — CLI-versus-MCP / kulibin-tooling case.

- "Поскольку bash как раз входит в стандартные инструменты из обучения LLM, то действительно вероятность того, что ИИ умеет пользоваться им внутри длинной траектории, намного выше." [author-claim]
- "Однако и это не гарантирует ничего. Для примера, я в GRACE у себя сделал автотесты сначала двух видов. Через командную строку вызывался стандартный pytest, а также мой 'кулибинский CLI' для тестов. В реале ИИ либо не вызывал мой CLI, либо галлюцинировал вокруг него." [empirical]
- "Asana (JIRA) — это не кулибинщина, а лидер рынка. Очень вероятно, что и MCP бы даже прокатил, т.к. топ-1000 основных MCP-серверов находятся в обучении GPT (это известно по публикации Kimi), и там JIRA точно есть." [research-backed, attributed to Kimi paper]
- "ИИ не любит кулибинщину в части Tools или MCP, т.к. банально ей не обучался. … в случае MCP у него ещё есть вариант просто отказаться от использования, что он и делает." [author-claim]

The "top-1000 MCPs are in training, per Kimi paper" and "`pytest` works like a rock" statements are the exact load-bearing claims; do not paraphrase them beyond what the post says. The URL of the Kimi paper itself is not provided in the sources and is deliberately not invented here.

**Research/benchmark values**: the 80/20 base-coder split, the "2–3 paragraphs, 200–300 lines" SFT ratio, and the "top-1000 MCPs" number are all cited from posts 3820 and 3834. No additional numbers are invented in this file.

## See also

- `../01-transformer-foundations/sparse-attention-and-kv.md` — why the model can't hold large code contexts without anchors, even though code "reads like English" to it.
- `../01-transformer-foundations/tc0-and-dyck-language.md` — the formal-logic basis for XML-like markup that makes in-code documentation parseable natively.
- `../01-transformer-foundations/xml-vs-json-vs-yaml.md` — format choice for the in-code doc layer this file justifies.
- `../03-contracts/contracts-overview.md` — contracts as the in-code documentation the FIM pipeline *cannot* ignore.
- `../03-contracts/ai-contracts-vs-dbc.md` — why AI contracts are shaped like SFT docstrings, not like Eiffel preconditions.
- `../03-contracts/contract-fields.md` — the concrete fields (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS) and their placement before the declaration.
- `../03-contracts/rationale-and-aag.md` — the architectural-intent layer that addresses "semantic incompleteness" of bare code.
- `../04-markup-system/in-source-documentation.md` — GRACE's in-source-doc lineage and how it differs from classical "Documentation-as-Code".
- `../05-logging-ldd/log-driven-development.md` — how the RL / SWE-Bench shape of the last training phase maps onto LDD.
- `../06-testing/cli-not-custom-mcp.md` — the pytest-versus-kulibin-CLI episode in full, including the Bash-in-standard-toolset argument.
- `../09-tooling/mcp-scepticism.md` — when MCPs work and when they don't, including Anton's contract-only MCP as a boundary case and the top-1000-per-Kimi rule.
- `../13-antipatterns/all-antipatterns.md` — "External documentation systems" and "Custom MCP tools (kulibin)" antipattern entries.
- `../14-research-references/annotated-bibliography.md` — Kimi paper entry (attribution only; URL not specified in sources).
