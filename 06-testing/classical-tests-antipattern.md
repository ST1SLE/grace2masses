# Classical Tests as Antipattern for AI

## What

Classical Test-Driven Development (TDD) — where exhaustive assert-based autotests are written alongside or before production code — is an imported anthropomorphism when applied to AI-generated code. GRACE treats the naive transposition of human TDD practices onto LLM workflows as a concrete antipattern: tests are "just more code" from the AI's perspective, they burn sparse-attention budget through cross-module references, and they narrow the model's solution space to "whatever passes the given assertions" [author-claim] (testing_and_autonomous_agents.md §"Почему обычные автотесты"; §"Еще один пост про 'тесты как шум'").

This file describes **why** classical TDD hurts AI; see `./tests-in-self-correction-loop.md` for where tests actually belong (the debug feedback layer) and `./agent-based-testing.md` for the agent-testing replacement pattern.

## Why

### 1. Tests are "just more code", and unusually hostile code at that

From the LLM's viewpoint, a test file is not a separate artifact — it is one more code module added to the project. It must be read, parsed, and held in context just like production code [author-claim] (testing_and_autonomous_agents.md §"Почему обычные автотесты").

The hostile part: test code is **structurally crossref-heavy**. Tests, by design, import and call functions from many modules, instantiate fixtures, reach into DB/HTTP/filesystem mocks, and assert against expected outputs that reference real types. This means tests burn through the model's scarce **sparse-attention budget** for cross-module anchors — the author's exact phrase is that tests "burn the stove with banknotes" (in Russian: "топите печку ассигнациями"), spending the limited global anchor budget of sparse attention on "silly test calls" [author-claim] (testing_and_autonomous_agents.md §"Почему обычные автотесты"; §"Еще один пост про 'тесты как шум'").

Why this matters formally: past the ~4k dense-attention window, the transformer falls back to sparse attention with ~100–200-token sliding windows connected by a few global anchor tokens; every cross-module reference in a test file consumes one of those anchor slots. See `../01-transformer-foundations/sparse-attention-and-kv.md` for the mechanism.

### 2. In-context tests collapse the solution space (overfitting in-the-small)

If you put tests into the AI's context alongside the contract — i.e., give the model "here is the contract **and** here are the assertions the code must satisfy" — the LLM truncates its own solution space to "whatever passes the assertions" [author-claim] (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'"):

> If you directly include tests in the same conditions as the contracts with AI, then AI simply truncates its solution space and generates code that passes your tests right away. (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'") [author-claim]

The AI stops reasoning about the algorithm and reasons about the scoreboard. The result is code that passes the given tests but does not generalize to inputs the tests do not enumerate.

### 3. Vendor-level overfitting already burned out strict test-shaped conditions

The overfitting effect is not only an in-context problem — it was so pronounced during training that LLM vendors **actively removed strict test-style conditions from training**, because models exposed to them would overfit and fail to generalize [author-claim] (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"):

> If you train the AI on strict test conditions, it starts fitting solutions specifically to them — and they then do not generalize. Therefore LLM vendors effectively "burned with napalm" test examples out of training. (contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC") [author-claim]

This is the same underlying mechanism as Hoare-style `{P}C{Q}` contracts being unsuitable as in-context conditions for LLMs. See `../03-contracts/ai-contracts-vs-dbc.md`.

### 4. Classical TDD is designed for humans, not models

TDD as a discipline predates LLMs by decades. Its value propositions — red/green/refactor rhythm, test-as-specification, regression-via-assert — are engineered around **human** developer psychology and workflow [author-claim] (testing_and_autonomous_agents.md §"Адаптация GRACE"):

> Classical Test Driven Development (TDD) often carries antipatterns here, because TDD standards are made for people, not for AI. (testing_and_autonomous_agents.md §"Адаптация GRACE") [author-claim]

Transposing this discipline onto an agent is **imported anthropomorphism** — the same category of error as pushing external documentation systems or custom MCPs onto a model whose training distribution never rewarded using them (see `../13-antipatterns/all-antipatterns.md` and `../09-tooling/mcp-scepticism.md`).

### 5. Most autotests are smoke tests — they do not catch real bugs anyway

The author's field observation across hundreds of trainees: the overwhelming majority of autotests in real codebases are **smoke tests** whose effective assertion is "did it crash?" [author-claim] (testing_and_autonomous_agents.md §"Почему обычные автотесты"):

> Most automated tests that I have seen in real life are smoke tests, where assert is at the level of "did it fall or not?" Such tests as a rule do not catch real bugs, although formally they exist. (testing_and_autonomous_agents.md §"Почему обычные автотесты") [author-claim]

So the trade-off with AI is not "lose correctness guarantees vs keep them" — it is "burn sparse-attention budget on smoke-tests that catch nothing, vs redirect that budget to trajectory-level semantic checks that actually catch bugs." The second option wins.

### 6. LLM-written tests are low quality — "self-taught parrot"

When an LLM writes tests, it writes them as a **parrot**, not as an analyst. This follows directly from the training regime:

- Pre-training (base coder, ~80% code / 20% NL) had no "test analysis" signal [author-claim] (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'").
- FIM (Fill-In-the-Middle) explicitly bans documentation-style reasoning about code; it optimizes "how correct is this fragment against the rest" [research-backed] (see `../01-transformer-foundations/llm-training-pipeline.md` and most_important.md §"Top-down логика"; the training-pipeline framing is most_important.md; the ban-on-documentation claim is author-claim repeated in `grace_matches.md` post 3820).
- SFT trained on task-code pairs (2–3 paragraph task + 200–300 LoC code) — so pair-shaped "tests + code" data existed, but **without any meta-reasoning about why the tests exist or what they cover** [author-claim] (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'").
- RL (SWE-Bench-style) rewards "fix the code so the isolation test passes" — it rewards patch-and-pass, not test design.

The author's exact diagnosis (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'"):

> LLM could see in training simply "tests + code" but WITHOUT any "analytics in words". Therefore LLM can write plausibly looking tests for code, but like a PARROT, because it had no training for the meaning of such actions either in pre-training or in reinforcement learning. Hence the indication noted in some works that tests written by AI usually have comparatively low quality.
>
> No wonder — AI was not taught to write tests, it is a "**self-taught parrot**" here. [author-claim]

This lands hard on teams that ask AI agents to "generate the test suite" as part of feature work: the tests will look right, the grammar will be right, the `assert` calls will be there, but the **coverage of failure modes** will be shallow and the assertions will often tautologically mirror the implementation the model just wrote.

### 7. Few-shot prompting doesn't save classical tests either

A natural counter-argument: "GPT handles few-shot well — can't we just feed it good test examples as few-shots?" The author's reply (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'"): the problem isn't whether few-shots work in general, it is **what few-shots were available for tests** during training. High-level test-design analytics was removed from training data. So in-context few-shots about test design have to land as "deep analogies from general human conversational logic" or as "very specific code-level pattern matching" — neither is reliable [author-claim] (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'").

## How

### Heuristics GRACE applies

1. **Do not put tests in the same context as the contract during code generation.** If the model sees `PURPOSE + INPUTS + OUTPUTS + KEYWORDS` together with `assert result == [1,2,3]`, it will collapse to "whatever satisfies the assert" and stop reasoning about `PURPOSE`. Keep tests out of the generation phase and introduce them only at the debug/self-correction phase. See `./tests-in-self-correction-loop.md`. [author-claim] (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'")

2. **Throw out `assert`, or ask the AI to keep them minimal.** The author's first-rule-of-thumb when auditing someone's AI-testing pipeline (testing_and_autonomous_agents.md §"Почему обычные автотесты"):

   > The first thing you should do — throw out assert or ask AI to make them minimalistic. [author-claim]

   The replacement is Log-Driven Development (LDD): have the AI **read the structured execution log** and make a semantic judgment about whether the run was correct. See `../05-logging-ldd/log-driven-development.md` and post 3693 for the log-to-code correlation heuristic.

3. **If you need an assert-analog in production, call an LLM on the log.** Rather than adding brittle equality checks, wire an API call to an LLM that reads the structured log segment and returns "working / not working" [author-claim] (testing_and_autonomous_agents.md §"Почему обычные автотесты"). That LLM can also be embedded directly in your app — see `./in-app-ai-console.md`.

4. **Prefer pytest over a DIY CLI, prefer bash over a custom MCP.** Classical tools that are in the standard LLM toolset (bash, pytest) are reliable "like a rock"; "kulibin" (DIY) wrappers around them get ignored or hallucinated around. See `./cli-not-custom-mcp.md` and post 3834.

5. **For autonomous agents, classical autotests are not sufficient even with the above fixes.** They need anti-loop protection (attempt counters + forced-context injections) designed INTO the test framework, or a single failing test becomes a few-shot the agent gets more confident about on every retry. See `../07-autonomous-agents/anti-loop-protection.md` and post 3695.

### Concrete symptoms that the antipattern is active

If you see any of these in an AI-coding pipeline, the classical-TDD antipattern is in play:

- The agent confidently generates code that passes the visible tests and fails in production.
- The agent regresses a fixed bug the moment it is asked to touch the same file.
- The agent burns huge token budgets reading test files alongside production code, then still misses obvious module relationships.
- Test failures come back as bare `AssertionError: expected 5, got 4` with no trajectory context for the model to reason about.
- The team has "good test coverage" as a number but debugs bugs in production anyway — i.e., the tests are smoke-level.
- When two humans disagree with an AI, it is almost always the person who insists "just write more tests" whose pipeline is collapsing.

### What to replace classical TDD with

GRACE relocates testing along two axes:

- **Phase axis**: tests move from the analytical/generation phase to the **self-correction loop during debug**. Their job is "help the model fix this bug", not "help the model understand the task". Task-understanding is done by contracts + graph. See `./tests-in-self-correction-loop.md`.
- **Signal axis**: tests move from binary assert signals to **rich structured log streams** that the model can read semantically (flags, trajectory, belief state). See `../05-logging-ldd/log-driven-development.md` and `../05-logging-ldd/belief-state-logging.md`.

For complex cases (especially ETL-style logic where classical autotests barely launch at all), GRACE replaces code-level autotests with **agent-based testing**: a developer-agent writes a Markdown guide, a tester-agent runs it and reports XML-clipped failures back to the developer-agent. See `./agent-based-testing.md` and testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy".

For in-app validation, install an **AI agent with a console inside the application**, with its API wired to Tools — this can run tests "much more sophisticated than pytest scripts" [author-claim] (testing_and_autonomous_agents.md §"Почему обычные автотесты"). See `./in-app-ai-console.md`.

## Evidence

Primary sources for this file:

- **testing_and_autonomous_agents.md §"Почему обычные автотесты считаются антипаттерном"** — the main exposition of why classical autotests hurt AI: sparse-attention budget burn, smoke-test uselessness, assert-minimization advice, in-app AI console as the pragmatic endgame, and pytest-via-CLI as the fallback. All claims [author-claim] unless otherwise noted.
- **testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум для ИИ'"** — reinforcement of the same line: cross-module call noise, AI can mentally simulate algorithms so 90% of "silly tests" are unnecessary, in-context tests collapse the solution space, the "self-taught parrot" diagnosis of LLM-written tests, the "transposing human practices to AI can even be harmful" conclusion. All [author-claim].
- **testing_and_autonomous_agents.md §"Адаптация GRACE под автономных и роевых агентов"** — the "TDD is made for humans, not AI" framing, the LDD-as-alternative proposal, the ERP unlock via semantic trajectory checks. [author-claim].
- **testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"** — the training-pipeline explanation of why LLM-written tests are shallow (no meta-analytics in pre-training, only "tests + code" pairs in SFT without analytic context, RL trains patch-to-pass not test design). [author-claim] with [research-backed] background on RL training regime.
- **contracts_as_anchor.md §"Почему AI-contracts не равны классическому DbC"** — the background claim that vendors actively removed strict test-shaped conditions from training because they caused overfitting; this is why the overfit-to-assertions behavior is deeply baked in, not just an in-context quirk. [author-claim].

Supporting references:

- **post 3695** — anti-loop protection: failed fixes become few-shots, making the model more confident about the same failed approach; regular autotests are **not compatible** with autonomous agents without attempt-counters and forced-context injections. [author-claim] (grace_matches.md post 3695).
- **post 3693** — the colleague Vlad case: 282 contracts + 1049 autotests applied GRACE contracts well but lacked GRACE logging; the agent itself reflected "tests without logs are like vodka without beer". Concrete case study of the same antipattern. [empirical] (grace_matches.md post 3693).
- **post 3834** — the kulibin-CLI evidence: the author ran a custom test CLI alongside pytest, and the LLM either ignored it or hallucinated around it; pytest worked "like a rock". Confirms why classical-TDD's tooling layer must stick to standard instruments even when the methodology moves. [author-claim] (grace_matches.md post 3834).
- **post 3820** — the four-phase LLM training pipeline that grounds the "no analytics about tests" argument: base coder (80% code), FIM (ban on documentation), SFT (task-code pairs, no meta-analytics), RL (SWE-Bench, patch-to-pass). [author-claim] (grace_matches.md post 3820); see also `../01-transformer-foundations/llm-training-pipeline.md`.

Research-backed background on the sparse-attention mechanism that makes cross-module test references expensive:

- `most_important.md §"Почему без семантического графа большие контексты разваливаются"` — the formal description of sparse attention past 4k tokens: 100–200 token sliding windows stitched by graph anchors, with fallback to random attention when no anchor structure exists. [author-claim] linked to transformer architecture facts. See `../01-transformer-foundations/sparse-attention-and-kv.md`.

## See also

- `./tests-in-self-correction-loop.md` — where tests actually belong: the debug feedback layer, not the analytical/generation phase.
- `./agent-based-testing.md` — developer-agent / tester-agent split for complex cases where classical autotests don't launch.
- `./in-app-ai-console.md` — embedded AI tester with Tools-wired API.
- `./cli-not-custom-mcp.md` — pytest/bash reliability vs DIY tooling.
- `../05-logging-ldd/log-driven-development.md` — LDD as the replacement for strict-assert-driven testing.
- `../05-logging-ldd/log-to-code-correlation.md` — function-ID/block-ID tokens in log lines, the critical heuristic that makes LDD work.
- `../05-logging-ldd/belief-state-logging.md` — AI-verbalized belief state in logs as the hypothesis-vs-ground-truth substrate.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — the mechanism that makes test cross-module references expensive.
- `../01-transformer-foundations/llm-training-pipeline.md` — the four training phases that explain why LLM-written tests are parrots.
- `../03-contracts/ai-contracts-vs-dbc.md` — the parallel story for contracts: strict Hoare-style conditions were burned out of training for the same overfitting reason.
- `../07-autonomous-agents/anti-loop-protection.md` — why regular autotests are incompatible with autonomous agents without anti-loop design.
- `../13-antipatterns/all-antipatterns.md` — the consolidated antipattern catalog; this file is the deep-dive for the "Classical TDD with AI" subsection.
