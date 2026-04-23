# Log-to-Code Correlation

## What

**Log-to-code correlation** is the GRACE heuristic of **injecting a function identifier token and a block identifier token into every log line** the application emits. Each emitted log record carries the name (or ID) of the function that produced it and an identifier for the specific code block inside that function. [author-claim] (post 3693)

With these tokens present in the trace, the AI agent can bind any log line directly back to the exact piece of code that produced it, and — crucially — the log and the code start behaving as **one semantic monolith** under the transformer's sparse-attention mechanism. The author calls this heuristic "archimportant" (архиважно) and treats it as non-optional for AI-readable logging. [author-claim] (post 3693)

Main source: **post 3693** (2026-03-21).

## Why

This heuristic exists because the transformer's attention mechanism collapses on long logs — even in very large context windows — unless the log text and the code text share common anchor tokens.

### The concrete empirical failure it addresses

From **post 3693** directly:

> "I did experiments with Gemini — on logs even past 5000 lines his attention shows drop-outs without such tags, and he simply cannot correctly grasp the context of code and logs together and misses errors."

Three points from that single experiment are load-bearing:

1. **The failure happens even with Gemini's 1M-token context.** Having the log fit inside the context window is not sufficient. The attention mechanism — not the context-window size — is what fails. [empirical] (post 3693)
2. **The threshold is around ~5000 log lines.** This is the order of magnitude at which attention drop-outs become visible in the author's experiments. Sources do not specify the exact line count beyond "even after 5000 lines." [empirical] (post 3693)
3. **Without function-ID / block-ID tokens in the log, the model "cannot correctly grasp the context of code and logs together and misses errors."** The failure is not "it reads more slowly" — it is "it misses errors." [empirical] (post 3693)

### The transformer-internals reason

The deeper cause is the same sparse-attention problem that motivates all of GRACE. Past the dense-attention ceiling (~4k tokens), the model operates through ~100–200-token sliding windows that have to be stitched together by shared anchor tokens; without those anchors it falls back to random attention or caches a wrong implicit structure in the KV cache. (Cross-link: `../01-transformer-foundations/sparse-attention-and-kv.md`.) [research-backed — author paraphrasing transformer internals] (`most_important_output.md` §"Почему без семантического графа"; post 3693)

Long logs are an especially bad case for sparse attention:

- **They are long.** Production traces cross the dense-attention boundary almost immediately.
- **They are repetitive.** Without distinguishing tokens, windows from different parts of the log look semantically identical.
- **They are detached from the code.** A raw log line has no textual pointer to the source code that emitted it — so the sliding windows over the log share no anchors with the sliding windows over the code.

Log-to-code correlation fixes the third problem and, through it, the first two: shared function-ID / block-ID tokens give sparse-attention windows on the log side something to hook into sparse-attention windows on the code side. [author-claim, paraphrasing post 3693]

### Why the author calls it non-optional

From post 3693, after listing belief-state logging and other heuristics:

> "These are not all the heuristics, there are also various forced injections of important log lines into the AI's context, various subtleties of category markup. However, at the **minimum** there should be what I wrote — **otherwise the log may not be read at all, even by a GPT with full understanding**."

The author explicitly frames log-to-code correlation (together with belief-state logging) as the minimum floor below which logging is broken for AI, not merely suboptimal. Other log-enrichment techniques (forced injection, category markup) sit above this floor; they do not substitute for it. [author-claim] (post 3693)

### What breaks without it

Empirical observations reported in the sources:

- **Attention drop-outs past ~5000 log lines** on Gemini. The model stops correctly binding log evidence to code during error analysis. [empirical] (post 3693)
- **Missed errors.** The model reads the log but cannot attach what it reads to the right code location, so bugs visible in the trace are not diagnosed. [empirical] (post 3693)
- **Tests without logs become useless.** In Vlad's contracts-heavy project (282 contracts, 1049 autotests) referenced in post 3693, the author notes: *"tests without logs — that's like vodka without beer"* (тесты без логов — это как водка без пива). The AI agent itself reflected to Vlad that it had no other "vision" into the running system besides logs. Missing log-to-code correlation is one of the reasons adding logs alone is not enough. [empirical, author-claim] (post 3693)

## How

### The rule

For every log line the application emits, **inject at least these two tokens**:

1. A **function identifier** — a stable token that names the function producing the line.
2. A **block identifier** — a stable token that names the specific block inside that function (for example, the particular branch, loop iteration, or phase).

Source: post 3693: *"Прежде всего это log-to-code correlation: я вставляю в лог идентификатор функции и блока кода."* (Translation: "First of all — log-to-code correlation: I insert into the log the identifier of the function and of the code block.") [author-claim]

The sources do not specify:

- the exact token format (prefix, delimiter, casing),
- a canonical naming scheme for function IDs,
- whether block IDs should be line-based, label-based, or generated,
- whether additional IDs (module, class, request, trace) should be added.

Do not invent these. The source text pins down only the two token categories — function ID and block ID — and the fact that they must be present on **every** log line. [author-claim, inference from the one-sentence source]

### What the two tokens accomplish together

From post 3693:

> "AI not only immediately easily understands which part of the code a log line relates to, but the log correlates with the code through these tokens, and **for GPT the log and the code merge into a single semantic monolith**."

Two effects, both framed by the author:

1. **Direct lookup.** Given any log line, the function-ID token identifies which function emitted it; the block-ID token narrows it down within that function. The model does not have to guess or search — the binding is explicit in the tokens. [author-claim]
2. **Semantic monolith.** Because the same function-ID and block-ID tokens appear both in the code (in contracts, comments, and markup around those functions/blocks) and in the log (on every line from those functions/blocks), sparse-attention windows over the log hit the same anchor tokens as sparse-attention windows over the code. The effect — as the author frames it — is that for the transformer, log and code cease to be two texts and become one semantic object. [author-claim]

### Why this specifically beats long-context alone

Sources explicitly distinguish two things that programmers often conflate:

- **A long context window** (e.g., Gemini's 1M tokens) — the amount of text the model will accept.
- **Attention quality across that window** — how well the model actually correlates distant parts of the input.

Gemini's 1M-token window does not keep attention intact across a 5000-line log on its own (post 3693). Log-to-code correlation tokens are a **sparse-attention navigation** solution, not a context-size solution. Loading the log into a bigger window does not fix what the heuristic fixes. [empirical + author-claim] (post 3693)

Cross-link: `../01-transformer-foundations/sparse-attention-and-kv.md` for the mechanism; `../09-tooling/cursor-limitations.md` for the related "copy the log into Gemini" workflow, which still requires the markers (from `xml_mapping_anchors_output.md` §"Почему семантическая разметка логов важна": *"Gemini also will not help you without semantic markup of the logs"*).

### Relationship to the other logging heuristics

Post 3693 lists three logging heuristics, in order. Log-to-code correlation is the first and the minimum. The other two sit on top of it:

1. **Log-to-code correlation** (this file) — function-ID + block-ID in every line.
2. **Belief-state logging** — logs capture the AI's hypothesis about how its code works, so later the agent can compare "what I believed" vs "ground truth." Cross-link: `./belief-state-logging.md`. [author-claim] (post 3693)
3. **Forced context injection** — important log lines are pushed into the agent's context rather than relying on the agent to call Tools to fetch them. Category markup in logs makes selection easier. Cross-link: `./forced-context-injection.md`. [author-claim] (post 3693, 3695)

The author's framing: heuristic 1 is the floor (non-optional); heuristics 2 and 3 are powerful additions on top.

### How this connects to standard AI-friendly logging markup

The logging heuristic is the operational half of a more general GRACE position that logs, like code, need **semantic markup** to be AI-readable. From `xml_mapping_anchors_output.md` §"Почему семантическая разметка логов важна":

- Cursor reads logs in ~100–200-line chunks via `read_file`; only the first ~100 lines are guaranteed to be read. [vendor-doc / author-claim]
- Without semantic markers, the AI is "blinded" in Cursor when trying to find relevant parts of a long log. [author-claim]
- For very long logs, the recommended workaround is to copy them into Gemini's 1M-token window — but this **still** requires semantic markup: a log without anchors is "chaotic text entropy" for Gemini. [author-claim]

Log-to-code correlation is the specific set of anchor tokens that does the job in a GRACE pipeline. Category markup (mentioned in post 3693 as a further refinement) layers on top of it. [author-claim] (`xml_mapping_anchors_output.md`, post 3693)

### What it enables downstream

Once the log and the code share anchor tokens:

- **LDD (Log-Driven Development) becomes possible.** The agent reads the trajectory from the log and makes semantic judgments about correctness, because it can bind each step back to the specific code that produced it. Cross-link: `./log-driven-development.md`. [author-claim] (`testing_and_autonomous_agents_output.md` §"Адаптация GRACE")
- **Belief-state comparison becomes possible.** To compare "what the AI believed" vs "what actually happened," both the belief record and the run-time observation need to be bound to the same code location. Shared function-ID / block-ID tokens provide that binding. Cross-link: `./belief-state-logging.md`. [author-claim] (post 3693)
- **Forced context injection becomes selective.** Once log lines are addressable by function/block, the surrounding tooling can pull exactly the lines that matter into the agent's context rather than dumping the whole trace. Cross-link: `./forced-context-injection.md`. [author-claim] (posts 3693, 3695)
- **Autonomous-agent anti-loop protection has material to work with.** Attempt counters and context injections (post 3695) need to point the agent at specific failing sites; function-ID / block-ID tokens in the trace are how the agent finds those sites. Cross-link: `../07-autonomous-agents/anti-loop-protection.md`. [author-claim] (posts 3693, 3695)

### Practical consequence for legacy logging

From the case-study evidence (post 3620, Kirill's 132k-LOC project):

> "After several posts it became clear that standard logging — is a relic of the past, where just an error or some info is printed; now you need to do it so that the AI can not only see errors in the logs, but all the information about how the project works, and all its processes were traceable."

The team migrating to GRACE updated their logs alongside their code markup. The author's framing throughout is that "traditional logging is obsolete" for AI-era development and that log-to-code correlation is one of the structural changes that makes logs traceable by the agent. [empirical, reported by Kirill] (post 3620)

### What the sources do NOT specify

- **No exact tag format.** The sources give the concept ("inject function ID + block ID") and the effect ("log and code merge into a semantic monolith"), but no canonical syntax. [author-claim — sources do not specify]
- **No universal block-ID scheme.** Whether blocks are numbered, labeled by the author, or extracted mechanically — the sources do not specify.
- **No specific threshold beyond "~5000 lines."** The author gives that number as an observation of when Gemini's attention drop-outs become visible; sources do not specify the exact line at which collapse begins or a formula relating lines to model.
- **No quantitative "after/before" improvement number.** The claim is categorical — "otherwise the log may not be read at all, even by a GPT with full understanding" — not a percentage. Do not invent one.

## Evidence

### Primary source — post 3693 (canonical)

Post 3693 is the single canonical source for this heuristic. The substantive passage:

> "Above all this is **log-to-code correlation**: I insert into the log the identifier of the function and of the code block. This is simply **archimportant** for AI with its sparse attention. AI not only immediately easily understands which part of the code a log line relates to, but the log correlates with the code through these tokens, and **for GPT the log and the code merge into a single semantic monolith**. I did experiments with Gemini — on logs even past 5000 lines his attention shows drop-outs without such tags, and he simply cannot correctly grasp the context of code and logs together and misses errors."

Claim types:

- "Archimportant" and "not optional" framing — **[author-claim]**.
- "AI immediately understands which part of the code a log line relates to" — **[author-claim]**.
- "For GPT the log and the code merge into a single semantic monolith" — **[author-claim]**, framed as a property of sparse attention on shared anchor tokens.
- "Experiments with Gemini on logs past 5000 lines show attention drop-outs" — **[empirical, author-conducted experiment]**.
- "Without such tags, the model misses errors" — **[empirical, author-conducted experiment]**.

### Supporting — post 3693's framing of the minimum set

Post 3693 also establishes that log-to-code correlation is the first of three heuristics and the minimum floor below which the log is unreadable even for strong models:

> "At the minimum there should be what I wrote — otherwise the log may not be read at all, even by a GPT with full understanding."

[author-claim] (post 3693)

### Supporting — post 3693 case context (Vlad's contracts project)

Post 3693 was written in response to a colleague (Vlad) applying GRACE contracts to a large project (282 contracts, 1049 autotests) without GRACE logging. The author notes that the AI agent itself reflected to Vlad that logs are the only "vision" it has, and that "tests without logs — that's like vodka without beer." The case motivates log-to-code correlation as the missing piece. [empirical, reported case] (post 3693; cross-referenced in case study at `../11-case-studies/vlad-crm-and-contracts.md`)

### Supporting — `xml_mapping_anchors_output.md` on log markup

The broader context: Cursor reads logs in 100–200-line chunks via `read_file`; without semantic markers the agent is blinded on long logs; copying logs to Gemini's 1M-token window helps only if the log has anchors. This is the general "logs need markup" position; log-to-code correlation is its concrete realisation inside GRACE. [author-claim] (`xml_mapping_anchors_output.md` §"Почему семантическая разметка логов важна")

### Supporting — post 3620 (Kirill's project)

After migrating to GRACE markup, Kirill's team updated logs too: "standard logging — is a relic of the past… now you need to do it so that the AI can not only see errors in the logs, but all the information about how the project works, and all its processes were traceable." Empirical corroboration that teams adopting GRACE overhaul logging as part of the migration. [empirical, reported by Kirill] (post 3620)

### Supporting — sparse-attention mechanics (background)

The transformer reasons the heuristic works are in `most_important_output.md` §"Почему без семантического графа большие контексты разваливаются": sparse attention with 100–200-token sliding windows past ~4k tokens, random-attention fallback otherwise, and the "table of contents" role of the semantic graph. The log-to-code correlation tokens play the same role for logs that the graph plays for code. [research-backed — author paraphrasing transformer internals; author-claim on the log-side extension] (`most_important_output.md`; post 3693)

### What is NOT in the sources (do not invent)

- The exact tag syntax used by the author (e.g., `[FN=foo][BLK=3]` vs `fn:foo#b3` vs anything else). Sources do not specify.
- The specific model version of Gemini used in the ~5000-line experiment. Sources do not specify.
- A formal paper URL for "log-to-code correlation." The heuristic is the author's own terminology; sources do not cite an external paper for it.
- Any percentage bug-recall or error-miss-rate numbers. Sources state the effect categorically, not quantitatively.
- Whether the tokens must be literally human-readable strings or can be hashes/IDs. Sources do not specify.

## See also

- `./log-driven-development.md` — the broader LDD methodology this heuristic is the foundation of.
- `./belief-state-logging.md` — the second heuristic from post 3693; logs record the AI's hypothesis about its code. Shares this file's function-ID / block-ID anchors.
- `./forced-context-injection.md` — the third heuristic from post 3693 / 3695; important log lines are pushed into the agent's context instead of being fetched on demand.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — the mechanism behind why shared anchor tokens are what makes long logs readable at all.
- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal basis for XML-like paired markup that also applies to log-line framing.
- `../02-semantic-graph/graph-as-backbone.md` — the general "anchor tokens across large text" principle, specialized to logs here.
- `../03-contracts/contract-fields.md` — function-side anchor tokens (PURPOSE / KEYWORDS / LINKS) that the log-side tokens will bind to.
- `../06-testing/tests-in-self-correction-loop.md` — once log and code merge, the self-correction loop can use log evidence to diagnose and patch.
- `../07-autonomous-agents/anti-loop-protection.md` — anti-loop injections need specific failing sites to point at; function-ID / block-ID tokens are how the agent finds them.
- `../07-autonomous-agents/forced-context.md` — forced context as a stabilization mechanism; selective injection depends on log lines being addressable.
- `../09-tooling/cursor-limitations.md` — why Cursor specifically needs these anchors on long logs (100–200-line chunked reads).
- `../11-case-studies/vlad-crm-and-contracts.md` — the motivating case for post 3693 (contracts without GRACE logging).
- `../11-case-studies/kirill-132k-loc.md` — empirical corroboration of "standard logging is obsolete" after GRACE migration.
