# In-source Documentation — GRACE as Enterprise Variant

## What

GRACE-style markup is an **Enterprise-level variant** of the long-established IT practice known as **"In-source documentation"** or **"Documentation as Code"** [author-claim] (post 3878).

The author states the positioning directly:

> "По факту мой GRACE — это разновидность Enterprise-уровня стандартов IT-разработки, известных как 'In-source documentation' или 'Documentation as Code'." (post 3878)

Translation: "In fact, my GRACE is an Enterprise-level variant of the IT development standards known as 'In-source documentation' or 'Documentation as Code'." [author-claim] (post 3878)

The key implications:

1. GRACE is not a new invention at the **format** level — every mature IDE and language ecosystem ships with a built-in in-source documentation format (Python docstrings, Javadoc, XML Documentation for C#, etc.) [author-claim] (post 3878).
2. The author's own assessment: "Для этого любая приличная среда имеет встроенный формат, неприлично скорее этим не пользоваться." — "Any decent environment has a built-in format for this; it is rather improper not to use it." [author-claim] (post 3878).
3. What GRACE changes is **what goes inside the format**, not the format itself [author-claim] (post 3878).

In short: GRACE inherits the lineage of in-source documentation and selectively fills it with content that matches how an LLM actually reads code, rather than what a human documenter would write.

## Why

GRACE's choice to extend in-source documentation (rather than build an external documentation system) is rooted in the training distribution of modern coder LLMs and in the empirical behavior of agents on long contexts.

### 1. The training distribution forces it

The author repeatedly stresses that any practice that fights the model's training signal is a losing bet [author-claim] (post 3820):

> "Дело в том, что любая методика работы с LLM обречена, если она идёт вразрез с обучением ИИ." — "Any methodology for working with LLMs is doomed if it goes against how the AI was trained." [author-claim] (post 3820)

Across the four-phase coder training (base → FIM → SFT → RL), the model is rarely exposed to external documentation and almost never trained to query it as a first-class source [author-claim] (post 3820). Details:

- **FIM**: "полный и тотальный запрет на документацию к коду" — "complete and total ban on documentation for code" [author-claim] (post 3820).
- **SFT**: task-code pairs of roughly 2–3 paragraphs of task plus 200–300 lines of code; not classical documentation [author-claim] (post 3820).
- **RL** (SWE-Bench analog): "никакой документации на код" — "no documentation for code" [author-claim] (post 3820).

Consequence: documentation that lives **inside** the code file the agent is already reading bypasses this problem, while documentation that lives in a separate MCP server, graph DB, or website is likely to be ignored or hallucinated around [author-claim] (post 3820, post 3821).

### 2. Adjacency matters — empirically

The "in-source" part is not just ergonomics. Research cited by the author shows degradation when documentation is split from code by a file-read or MCP hop [research-backed] (post 3889):

> "Описание кода должно быть 'максимально прижато к коду' (как те же контракты в GRACE к функциям): если описание функции отделено от кода дистанцией токенов из-за чтения файла с документацией или MCP-сервера, агент деградирует." — "Code description must be 'pressed as close to code as possible' (like GRACE contracts to functions): if the function description is separated from code by a token-distance due to a doc file read or MCP server, the agent degrades." [research-backed] (post 3889)

Source: "In-source documentation" benchmarks at arxiv.org/abs/2601.16661v1, cited in post 3889.

Effect sizes reported from that work: **-95% degradation on bad structure, up to +435% improvement on good structure** [research-backed] (post 3889).

### 3. RL rewards goal achievement — so PURPOSE is the anchor, not exposition

The defining difference between classical in-source documentation and GRACE is the **content emphasis**. This is the sharpest claim in post 3878:

> "'По классике' во встроенную документацию идет 'описание функции'. Но для ИИ важнее большее конкретное как 'цели и намерения', т.е. на Reinforcement Learning он получал плюшки за 'достижение целей'. Поэтому PURPOSE — не болтовня, а alignment для ИИ при генерации контекста."

Translation: "'Classically', in-source documentation contains 'a description of the function'. But for an AI, what matters more is something more concrete: 'goals and intent', because during Reinforcement Learning it got rewards for 'achieving goals'. Therefore PURPOSE is not filler, it is **alignment** for the AI during context generation." [author-claim] (post 3878)

This reframes the purpose of documentation for AI:

- **Classical audience (human)**: "what does this function do?" → prose DESCRIPTION.
- **GRACE audience (LLM)**: "what is this function trying to achieve, and what patterns/modules does it connect to?" → **PURPOSE + correlation hooks (KEYWORDS, LINKS)**.

Post 3878 also cites Anthropic's **Purpose Breakdown Structure**: the author claims Anthropic showed that GPT always constructs a hierarchical decomposition of goals from global to local, and that verbalizing this decomposition is strictly better than leaving it implicit [author-claim, research-attributed-via-author] (post 3878).

### 4. External documentation systems fight the agent's primary task

The author and Sergei Muravyev discuss MCP-served documentation in post 3821. Key argument:

> "У агента в реальности навигационная задача на уровне 'найти модуль'. Такие задачи, как 'понять класс', будут решены автоматически после выполнения первой задачи, поскольку они окажутся в контексте."

Translation: "The agent's real task is navigational — 'find the module'. Tasks like 'understand the class' are solved automatically after the first one, because the module will already be in context." [author-claim] (post 3821)

An MCP that serves documentation substitutes for a task the agent does not actually have — understanding a class in isolation — instead of helping with the task it does have, which is greedy module discovery [author-claim] (post 3821). In-source documentation, by definition, rides along with the module the agent just loaded.

## How

### The inheritance

GRACE does not invent a new physical format. It reuses what the ecosystem provides [author-claim] (post 3878):

- In Python: comments above the declaration + optional `"""docstring"""` below it (post 3878).
- In C#: **XML Documentation** — "совместимость с семантическими разметками для ИИ агентов типа GRACE очень высокая" — "compatibility with semantic markups for AI agents like GRACE is very high" [author-claim] (post 3960). `region` directives provide paired START/END tags for sparse attention (post 3960).
- By implication, any language with a standard in-source doc format can carry GRACE content.

The author also notes (post 3848) that GRACE can be wrapped into these standards so that automatic documentation generators still work, but emphasizes that the reason to do so is aesthetic/compatibility — AI agents read documentation through the **code file**, not through MCP servers, in ≥50% of cases [author-claim] (post 3848).

### What changes: content, not format

Classical in-source documentation fields (from the author's description of "classical" practice in post 3878):

- DESCRIPTION / summary — prose explanation of what the function does.
- INPUTS — typed parameters and prose description.
- OUTPUTS — typed return value and prose description.

GRACE adjusts the priority:

| Field | Classical priority | GRACE priority | Why [cite] |
|---|---|---|---|
| **PURPOSE** | optional, often missing | **critical** — "alignment anchor" | RL rewards goal achievement; research shows agent can skip DESCRIPTION but not PURPOSE [author-claim, research-backed] (post 3878, post 3889) |
| **DESCRIPTION** | central | optional; often droppable | Research: "agent can perfectly get by without a function DESCRIPTION" [research-backed] (post 3889) |
| **INPUTS / OUTPUTS** | present | retained in standard form | "Стандартная практика описания входов/выходов во встроенной документации вполне хорошая для ИИ" — "the standard practice of describing inputs/outputs in in-source documentation is quite good for AI" [author-claim] (post 3878) |
| **KEYWORDS** | absent | **added** — wenyan-compressed tags for architectural patterns and task classification | RAG hooks; SFT datasets often carried task classification tags [author-claim] (post 3878); cross-link `wenyan-prompting.md` |
| **LINKS** | absent | **added** — cross-module references acting as Call Graph proxy | Enables vector search to find related modules without direct calls in code [author-claim] (post 3819) |
| **RATIONALE / AAG** (module-level) | usually absent | **added** at the module-contract level | Architectural intent raises bug-fix accuracy up to 3× [research-backed] (post 3890) |

### The canonical GRACE Python example

Reproduced verbatim from post 3878 (the author's own demonstration of the principle):

```python
# PURPOSE:Сохранение параметров в JSON файл.
# INPUTS:
# - dict - config: Словарь параметров
# - str - config_path: Путь к файлу (по умолчанию "config.json")
# OUTPUTS:
# - bool - Статус успеха операции
# KEYWORDS:[Saver, FileWrite]
def save_config(config: dict, config_path: str = "config.json") -> bool:
    """Сохраняет переданный словарь параметров в JSON файл."""
    ...
```

Two things worth noticing from post 3878:

1. **Placement above the declaration**: the author explicitly places PURPOSE/INPUTS/OUTPUTS/KEYWORDS **before** the `def`, deviating from the usual Python convention of a docstring-only description. Justification: "есть более крупные межязыковые соглашения о том как делать встроенную документацию и там описания идут до деклараций функций, что для каузального чтения ИИ на деле критическое значение имеет для формирования якорного вектора за токенами названия функции/метода" — "there are larger cross-language conventions for in-source documentation in which descriptions go **before** function declarations, which for the AI's causal reading is critically important for forming the anchor vector behind the function-name tokens" [author-claim] (post 3878).
2. **The docstring below the declaration is retained** as an SFT hint. Post 3892 notes that sparse attention typically covers roughly the most recent ~1000 tokens, so docstring-below-declaration still contributes at generation time, complementing the anchor comments above [research-backed] (post 3892).

### Compression rules that sit on top

GRACE content within the in-source format follows the compression discipline described elsewhere in the knowledge base (`markup-compression.md`, `wenyan-prompting.md`), cited here only to show how content discipline layers on top of the format decision:

- Shorter is better; long descriptions are neutral or harmful [research-backed] (post 3889).
- Keep comments in English; "sovereign AI" non-English comments degrade the agent [research-backed] (post 3889).
- ShortenDoc (dl.acm.org/doi/10.1145/3735636): 40% compression without quality loss, ~10% gain on HumanEval when only key concepts are kept [research-backed] (post 3892).

### What about auto-generated documentation systems?

Post 3848 answers the common developer question "can we wrap GRACE into standard auto-doc tooling (Doxygen, Sphinx, etc.) so we still get a website?" The author's position, paraphrased: yes, technically you can, but:

- The AI-developer agent reads documentation by reading the **code file**, not through the generated website, in 99% of cases [author-claim] (post 3848).
- The probability of an agent invoking an MCP (or similar mechanism) for documentation is below 50%, per a channel poll referenced by the author [author-claim] (post 3848).
- If you still need the auto-generated site, extend the doc format's supported fields to include what an AI needs: PURPOSE, architectural patterns, correlation links [author-claim] (post 3848).
- To raise the documentation-fetch probability, use a dedicated **context-gathering subagent** whose trajectory treats the doc tooling as a RAG source (post 3848).

## Evidence

Primary source for this file:

- **post 3878** — explicit statement that GRACE is an Enterprise variant of in-source documentation / Documentation as Code; every decent IDE has a built-in format; GRACE changes content, not format; PURPOSE is an alignment anchor, not exposition; Anthropic's Purpose Breakdown Structure attribution; Python example.

Supporting sources:

- **post 3889** — benchmark work at arxiv.org/abs/2601.16661v1: PURPOSE > DESCRIPTION; shorter > longer; English > other languages; adjacency matters. Effect range -95% to +435%.
- **post 3890** — researchgate.net/publication/400340807: architectural intent in comments lifts bug-fix accuracy up to 3×; attention analysis confirms model attends to intent tokens.
- **post 3892** — dl.acm.org/doi/10.1145/3735636 (ShortenDoc): 40% compression without quality loss, ~10% HumanEval gain; sparse attention covers ~1000 recent tokens so docstring-below-declaration still matters at generation time.
- **post 3820** — four-phase LLM coder training; external documentation fights the training distribution.
- **post 3821** — Sergei Muravyev discussion: agent's primary task is "find module", not "understand class"; MCP documentation substitutes the wrong task.
- **post 3848** — "Можно, а зачем?" on wrapping GRACE into standard auto-doc generators; <50% agent-invocation probability for documentation MCPs.
- **post 3819** — KEYWORDS and LINKS are "pure wenyan prompting"; 90% of Cursor semantic searches hit these enrichment points (author's trainee observation).
- **post 3960** — C# XML Documentation and `region` have high compatibility with GRACE.

Cross-references — 5 of the 6 source files reinforce the lineage:

- `most_important_output.md` §"Top-down логика и переход к контрактам" — base FIM training barely touched documentation, so AI heavily trusts code; contracts (living in code) cannot be ignored.
- `contracts_as_anchor_output.md` §"Почему AI-contracts не равны классическому DbC" — AI contracts descend from Python docstrings and Spec-Driven Development, not from Eiffel DbC; form is tuned to GPT. This reinforces the "in-source lineage, reshaped content" framing.
- `xml_mapping_anchors_output.md` §"Почему семантическая разметка логов важна" — without semantic markup, the agent is "blinded" in Cursor; the same principle that justifies in-source markup in code justifies it for logs.

Research works cited (see `../14-research-references/annotated-bibliography.md` for full entries):

- In-source documentation benchmarks — arxiv.org/abs/2601.16661v1 (post 3889).
- ShortenDoc — dl.acm.org/doi/10.1145/3735636 (post 3892).
- Impact of Code Comments on Automated Bug-Fixing — researchgate.net/publication/400340807 (post 3890).
- Anthropic's Purpose Breakdown Structure — attributed by the author; sources do not specify a direct publication URL (post 3878).

Empirical case studies that reinforce the choice:

- **Kirill's 132k LOC project** (post 3620): after applying GRACE markup, "агенты быстрее собирают нужный им контекст, лучше понимает, что от него требуется" — "agents gather the context they need faster, understand requirements better", zero rollbacks, less hallucination; two latent bugs surfaced during markup pass [empirical] (post 3620).
- **Vlad's CRM** (practical_applications.md §"Пример CRM"): GRACE in-source markup + specs made models feel interchangeable across Gemini Flash/Pro, Claude Opus 4.6, GLM-5, Kimi K2.5 [empirical] (practical_applications.md §"Пример CRM").
- **Qwen 3.5-27B SLM** (post 3902): the small model ran GRACE **more disciplined on in-source documentation than larger LLMs** [empirical] (post 3902).

## See also

- `../00-foundations/core-principles.md` — principle 2: "Documentation lives IN code, not beside it."
- `./xml-like-markup.md` — the format layer GRACE sits on top of.
- `./start-end-tags.md` — paired-tag mechanic that makes in-source markup Dyck-parseable in one pass.
- `./markup-compression.md` — empirical bounds on how far to compress the in-source content.
- `../03-contracts/contract-fields.md` — full field specification (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS, RATIONALE/AAG).
- `../03-contracts/wenyan-prompting.md` — compression technique behind KEYWORDS and LINKS.
- `../03-contracts/rationale-and-aag.md` — architectural-intent content carried in module-level in-source docs.
- `../03-contracts/language-specific/python.md` — Python-specific placement rules (description before `def`; docstring below retained).
- `../03-contracts/language-specific/csharp.md` — C# XML Documentation mapping.
- `../01-transformer-foundations/llm-training-pipeline.md` — why external documentation fights the training distribution.
- `../09-tooling/mcp-scepticism.md` — why MCP-served documentation rarely works.
- `../13-antipatterns/all-antipatterns.md` — "External documentation systems" antipattern; "Long summary-style descriptions" antipattern.
- `../14-research-references/annotated-bibliography.md` — full entries for the three research works cited above.
