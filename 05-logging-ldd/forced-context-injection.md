# Forced Context Injection (Logs)

## What

**Forced context injection** is the practice of deliberately pushing specific log lines into the agent's context window instead of trusting the agent to fetch them itself via Tools. In Log Driven Development (LDD), some log content is important enough — or some agent states are fragile enough — that the designer hard-wires the lines into the prompt/context rather than waiting for the agent to request them. [author-claim] (post 3695)

This page covers the **logs-specific angle** of forced context: which log lines to inject, why, and how category markup in logs supports it. The broader autonomous-agent mechanism of forced context (as a stabilization technique for agents in general) is covered in `../07-autonomous-agents/forced-context.md`.

Key terms:

- **Forced context** / *принудительный контекст* — deliberate injection of information (here, specific log lines) into the agent's context, bypassing the "agent will call Tools when it needs info" assumption. [author-claim] (post 3695)
- **Category markup in logs** — paired/semantic markers inside log lines that make filtering and selective injection tractable. [author-claim] (post 3693)
- **Log-to-code correlation** — function ID and block ID tokens inside each log line, which allow GPT to bind a log line back to its originating code. Prerequisite for any meaningful forced injection. [author-claim] (post 3693) — see `./log-to-code-correlation.md`.

## Why

### 1. The "agent will call Tools if confused" assumption is naive

The author names this assumption directly and rejects it:

> "A calculation that the agent, especially if it has looped, will itself call Tools to read important information — is simply naive." [author-claim] (post 3695)

This matters most in two cases:

- **Autonomous / swarm agents on long trajectories.** If the agent is looping on a failing fix, failed attempts become *few-shots* for it, and the model becomes *more* confident about the same wrong approach. In this state, asking "why didn't it just fetch the relevant log?" misses how the model is behaving — it is not reasoning about what it lacks, it is doubling down on what it has. [author-claim] (post 3695)
- **Cursor and chunked-read RAG agents.** Cursor reads logs in 100–200-line chunks; only the first ~100 lines are guaranteed to be read. Past that, the agent has to rely on text/semantic search. Without semantic markers in the log, the AI is **blinded**: "AI in Cursor literally cannot read a log longer than 100 lines without semantic markup." [author-claim] (xml_mapping_anchors.md §"Почему семантическая разметка логов важна")

Both cases lead to the same remedy: do not hope that the agent fetches the critical lines — **put them in its context directly**.

### 2. Sparse attention is where log reading collapses

Past ~4k tokens, transformers fall back to sparse attention (100–200-token sliding windows stitched by graph anchors). Without the right anchors *in* the log, the windows don't stitch to code context, and the agent loses the thread. [research-backed] (`most_important.md` §"Почему без семантического графа…") — see `../01-transformer-foundations/sparse-attention-and-kv.md`.

Empirically, the author reports: "on logs even past 5000 lines [Gemini] already shows visible attention dropouts without such tags, and it simply cannot grasp the context of code and logs together correctly and misses errors." [empirical] (post 3693)

This is the mechanical reason forced injection is needed: the agent *would* read the line if attention reached it, but it doesn't. You pre-select what attention must see.

### 3. Tests without logs, and logs without forced injection, don't work for autonomous agents

Vlad's Contracts Project case (post 3693) had 282 contracts and 1049 autotests — a serious setup — but lacked GRACE-grade logging. The reflecting agent itself commented that "tests without logs are like vodka without beer." [empirical] (post 3693) Autonomous test loops cannot self-correct when the trace they need is present on disk but never reaches attention.

At swarm or long-autonomous scale, the risk is not a slow response — it is **burning a billion tokens in a loop**. [author-claim] (post 3695) Forced context injection is one of the two main protections (alongside attempt counters) that keep autonomous agents from running away. See `../07-autonomous-agents/anti-loop-protection.md`.

## How

### Which log lines to inject forcibly

The sources do not give a full enumerated list. They describe the *classes* of lines worth forcing:

- **Belief-state mismatches.** When the AI wrote the code, it verbalized a *belief state* (hypothesis) about how the code would behave; the log captures both belief and ground truth. Lines where these diverge are high-signal for self-correction — exactly the places the agent most needs to see. [author-claim] (post 3693) — see `./belief-state-logging.md`.
- **Lines relevant to a known prior failure.** When tests are integrated with a knowledge base of previous fixes, a regression should pull the matching KB-entry log lines into context *forcibly*. Otherwise the agent relearns the same lesson and may re-introduce the same bug. [author-claim] (post 3695) — see `../07-autonomous-agents/knowledge-base-in-tests.md`.
- **Entry/exit lines for the failing function or block**, carrying the function ID / block ID tokens that bind them to the code. Without log-to-code correlation tokens the line cannot be meaningfully injected anyway. [author-claim] (post 3693)

The author also explicitly mentions:

> "There are also forced injections of important log lines into AI's context, all kinds of subtleties of category markup." [author-claim] (post 3693)

…framing these together as related heuristics inside the broader LDD stack.

### Category markup in logs

Category markup is what makes forced injection tractable: without it, the framework that injects lines cannot pick the *right* lines cheaply.

- Markup is added inside the log line itself, alongside the function/block ID tokens. [author-claim] (post 3693)
- The general rule — **semantic markers in any long text** — applies to logs the same way it applies to code: without markers, the AI is "blinded" past ~100 lines in Cursor and past ~5000 lines even in Gemini's million-token window. [author-claim] (post 3693; xml_mapping_anchors.md §"Почему семантическая разметка логов важна")
- Markup lets a selector (framework, test harness, or injection script) filter log lines by category and pull only the high-signal ones into context, instead of shipping the whole tail and blowing the budget.
- Practical lineage: the author trains this via the LOFT log-analysis practices modified for AI — the sources do not go further than that attribution. [author-claim] (xml_mapping_anchors.md §"Почему семантическая разметка логов важна")

### Two pieces that must land together

Forced injection of a log line is only useful if the line itself can be parsed back to code:

1. **Log-to-code correlation.** Each injected line must carry the function ID + block ID tokens, so GPT can bind it back to the relevant code region. The author calls this "archimportant": without it, even perfect injection leaves GPT unable to use the line. [author-claim] (post 3693) — see `./log-to-code-correlation.md`.
2. **Category markup.** So the selector can choose *which* lines to inject without a human in the loop. [author-claim] (post 3693)

Together these two make an injected log line a single semantic atom for GPT: it reads "function X, block Y, category Z said this" and fuses the line with the code in its attention. Without these tokens, GPT "cannot grab the context of code and logs together correctly and misses errors." [author-claim] (post 3693)

### Scope inside GRACE

Where forced injection sits in the stack:

- **Not optional for autonomous agents.** "Regular autotests are in fact NOT compatible to a sufficient degree with autonomous AI." Forced context is part of what is needed to make them compatible. [author-claim] (post 3695)
- **Essential for swarm agents.** A swarm amplifies every weakness, including the tendency of looped agents to skip Tool calls; forced injection becomes non-negotiable at swarm scale. [author-claim] (post 3695) — see `../07-autonomous-agents/swarm-vs-single-agent.md`.
- **Designed in from the start.** "It is important at an early design stage of your tests to take into account features of their compatibility with autonomous agents." Retrofitting forced injection into a test framework that was not built for it is painful. [author-claim] (post 3695)

### What the sources do NOT specify

- A concrete injection API, template, or line format — the sources describe the principle, not an implementation.
- A precise threshold for "when does an autonomous agent need forced injection" vs. "when can it self-rescue via Tools." The author states the self-rescue assumption is naive but does not publish a cutoff.
- A standard taxonomy of log categories — "category markup in logs" is named as a practice but the categories themselves are not enumerated. *The sources do not specify.*

## Evidence

| Claim | Source |
|---|---|
| "Forced context" named as an important concept for autonomous agents; stabilizes them on complex tasks. | post 3695 |
| Expecting a looped agent to call Tools itself to fetch info is "simply naive." | post 3695 |
| Anti-loop stack = attempt counters + context injections. | post 3695 |
| Autonomous swarm or single agent can burn a billion tokens looping. | post 3695 |
| Forced injection must be designed into tests from the start; ordinary autotests are not compatible with autonomous agents. | post 3695 |
| Tests integrated with KB of prior fixes; forcibly inject relevant KB entries on regression. | post 3695 |
| "Forced injection of important log lines into AI's context" + "category markup" listed as core logging heuristics. | post 3693 |
| Log-to-code correlation (function ID + block ID tokens) is prerequisite — "archimportant." | post 3693 |
| Without correlation tokens, Gemini shows attention dropout past ~5000 log lines even with 1M context. | post 3693 |
| Belief-state logging: AI verbalizes its hypothesis, compares belief vs. ground truth. | post 3693 |
| Vlad's Contracts Project (282 contracts, 1049 tests) — agent's own reflection: "tests without logs are like vodka without beer." | post 3693 |
| Cursor reads logs in 100–200-line chunks; only first ~100 guaranteed; needs semantic markers. | xml_mapping_anchors.md §"Почему семантическая разметка логов важна" |
| Even Gemini's 1M window fails on markerless logs ("log without anchors = chaotic text entropy"). | xml_mapping_anchors.md §"Почему семантическая разметка логов важна" |
| Sparse attention past ~4k tokens; windows stitched by graph/semantic anchors. | `most_important.md` §"Почему без семантического графа…" |

## See also

- `./log-driven-development.md` — LDD overview; why trajectory analysis replaces strict asserts.
- `./log-to-code-correlation.md` — function/block ID tokens in every log line (prerequisite).
- `./belief-state-logging.md` — AI's verbalized hypotheses as log content worth injecting.
- `../07-autonomous-agents/forced-context.md` — the broader autonomous-agent mechanism of forced context injection (this file covers only the logs side).
- `../07-autonomous-agents/anti-loop-protection.md` — counters + context injection as the anti-loop stack.
- `../07-autonomous-agents/knowledge-base-in-tests.md` — KB-of-prior-fixes integration; forced injection on regression.
- `../07-autonomous-agents/swarm-vs-single-agent.md` — why forced injection becomes non-negotiable at swarm scale.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — the mechanical reason markerless logs collapse past 4k tokens.
- `../06-testing/tests-in-self-correction-loop.md` — where forcibly-injected log lines land in the modern test loop.
