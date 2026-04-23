# Log-Driven Development

## What

**Log-Driven Development (LDD)** is the GRACE-aligned testing philosophy in which the AI **reads structured logs and makes a semantic judgment about whether an application's execution trajectory was correct**, instead of relying primarily on strict `assert` equality checks (`testing_and_autonomous_agents.md` §"Адаптация GRACE под автономных и роевых агентов"; §"Почему обычные автотесты"). Authority for correctness is moved from "did this value equal that value?" to "does the sequence of events the log describes match what the contract and the AI's belief state say should have happened?".

The author's [author-claim] one-line framing of the idea:

> "Besides the usual `assert`-checks, it is very important to apply a concept analogous to my **Log Driven Development (LDD)**: the AI evaluates the application's execution trajectory and makes a semantic judgment about correctness — this is much more reliable than strict equality checks."
> — `testing_and_autonomous_agents.md` §"Адаптация GRACE под автономных и роевых агентов"

LDD is not a separate product but a **design posture for GRACE test suites**: logs are engineered so a model can use them as primary evidence, and tests are engineered so they feed that log-reading loop instead of closing it with a binary pass/fail. It sits alongside automated asserts rather than replacing them outright, but it **inverts where the signal of truth lives**: the log becomes the judge, the assert becomes a cheap outer wrapper.

The phrase "Log-Driven Development" is the author's own term [author-claim], used in opposition to classical TDD (`testing_and_autonomous_agents.md` §"Адаптация GRACE…").

## Why

LDD exists because four compounding problems make classical assert-based testing **actively harmful** when the thing under test is read and written by an LLM, not a human.

### 1. LLMs overfit to assertions — strict equality collapses the solution space

[author-claim] If an AI is shown a contract and a body of strict tests in the same context, it **collapses its solution space and generates code that simply passes the tests**, not code that solves the problem (`testing_and_autonomous_agents.md` §"Еще один пост про 'тесты как шум'"). This is observable at training scale, too: [author-claim / research-backed] LLM vendors have **"burned test-like strict conditions out of training with napalm"** because they caused overfitting and the solutions did not generalise (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC"). The same pathology reappears at inference time whenever strict tests dominate the context.

LDD avoids this because **trajectory analysis doesn't hand the model a checksum to overfit to**. A semantic judgment over a log cannot be gamed by "just return this value" — the model has to produce an execution that makes sense as a whole.

### 2. Asserts are brittle — most real asserts are smoke tests anyway

[author-claim / empirical] Most automated tests the author has seen in the field are **smoke tests — assertions at the level of "did it crash?"** — which do not actually catch real bugs (`testing_and_autonomous_agents.md` §"Почему обычные автотесты"). On top of that, preparing the data for strict asserts **complicates the application for the AI** and is usually pointless because the asserts themselves are "as mindless as [the example above]" (same section).

Testing literature inside the LLM-training distribution is no help either: [author-claim] **LLMs were trained on "tests + code" pairs but without meta-reasoning about tests**, so they can generate plausible-looking test code but as a "parrot-self-taught" — without the semantic foundation for why a given test makes sense (`testing_and_autonomous_agents.md` §"Тесты как часть self-correction loop"). LLM-written assertions are therefore brittle by construction, not just by accident.

### 3. Tests are "just more code" — and unusually hostile code for sparse attention

[author-claim] From the model's point of view, an assert-heavy test suite is "just another body of code" — but a particularly **semantically noisy one**, because test code does heavy cross-module referencing and **"burns the scarce budget of sparse-attention global anchors on stupid test calls"** (`testing_and_autonomous_agents.md` §"Еще один пост про 'тесты как шум'"). In other words, the tests actively compete with the real code for the model's attention budget, and in a long context they win at the expense of comprehension.

LDD trades this expensive test-as-code surface for a **cheap, compact log-reading surface** that the model is strongly trained on (log/output reading is in FIM and SWE-Bench-style RL; see `../01-transformer-foundations/llm-training-pipeline.md`).

### 4. Classical TDD was designed for humans, not models

[author-claim] The GRACE author's blunt verdict: **"standard TDD is often an antipattern here, because TDD standards are made for humans, not for AI"** (`testing_and_autonomous_agents.md` §"Адаптация GRACE…"). Mindlessly translating human TDD practice into AI-driven development "can even be harmful" (`testing_and_autonomous_agents.md` §"Еще один пост про 'тесты как шум'"). The author [author-claim] reports direct cases from his training clients where their enthusiasm for autotests "wrecked the AI's brain with their tests, and instead of raising quality they got a collapse in quality" (`testing_and_autonomous_agents.md` §"Почему обычные автотесты").

LDD takes the cure seriously: the first recommended action is literally **"throw out asserts, or ask the AI to make them minimalist"** (`testing_and_autonomous_agents.md` §"Почему обычные автотесты"). The real test signal lives elsewhere — in the log the AI reads.

### 5. ERP-style fuzzy business logic is untestable with strict equality

[author-claim] There is a second, positive reason LDD matters: it **unlocks automated testing for whole problem domains that classical asserts cannot cover**. In ERP development (from 1C to SAP), "because of the huge complexity and simultaneous fuzziness of the tasks, automated testing is almost absent — millions of lines of code are often not covered by autotests at all" (`testing_and_autonomous_agents.md` §"Адаптация GRACE…").

The reason is that ERP business logic is too fuzzy for strict equality: "if it's hard to check business logic qualitatively with simple equalities, then **through semantic AI checks you absolutely can**" (same section). LDD therefore isn't merely a less-overfitting replacement for asserts — it is the mechanism that **brings automated testing into fuzzy-logic domains for the first time**, and with it the ability to "replace leather-skin programmers with bots" in ERP (same section). [author-claim]

### 6. Autonomous agents need log-reading, not assert-polling, to function

[author-claim] This is the structural argument: modern agent testing has moved from the analytical phase into the **self-correction loop during debug** — the new job of a test is not "help the model understand the task", but "help the model fix a specific bug" (`testing_and_autonomous_agents.md` §"Тесты как часть self-correction loop"). The self-correction loop is a **read-log → patch-code** loop; it is exactly what SWE-Bench-style RL trains for (same section). Without rich, readable logs, the loop has nothing to consume.

The author puts it more strongly for the colleague Vlad's contracts-project case [author-claim]: with 282 contracts and 1049 autotests but no GRACE-style logging, the agent itself reflected back that **"tests without logs are vodka without beer"** — because the agent has no other "sight" besides logs (post 3693). Assert suites in isolation, no matter how numerous, do not give the agent a trajectory to reason over. LDD is the counter-move.

## How

LDD is implemented as a **cluster of heuristics about how to write logs and how to let the AI consume them**, sitting on top of a skeletal assert layer. The full heuristics split across sibling files (`log-to-code-correlation.md`, `belief-state-logging.md`, `forced-context-injection.md`); this file is the overview, and the concrete mechanics below are the minimum set the author says a log needs before GPT can read it at all.

### Step 1 — demote asserts, don't eliminate them

[author-claim] The first concrete move is "throw out `assert`s, or ask the AI to make them minimalist" (`testing_and_autonomous_agents.md` §"Почему обычные автотесты"). Asserts remain as a cheap outer sanity-check (did the program crash? did it return something non-garbage?), but the **primary correctness signal shifts into the log**. If production genuinely needs an assert analogue, the author recommends replacing it with **"a call to an LLM to check the log against a prompt and return its judgment — works or not"** (same section) — i.e. an LLM-reviewed log, not a hand-coded equality.

### Step 2 — engineer the log so the AI can actually read it

[author-claim] Standard "old-style" logs do not work for AI agents (post 3693). Three heuristics in particular must be in place before a log is AI-readable at all; the author flags the first as "archimportant — without it the log does not work for AI at all":

1. **Log-to-code correlation**: inject function-ID and code-block-ID tokens into every log line, so the AI can bind each log line to the code that produced it (post 3693). Author's experiments showed Gemini's attention collapsed past ~5000 log lines without these tokens. With them, log and code "merge into a single semantic monolith for GPT." See `log-to-code-correlation.md`.
2. **Belief-state logging**: when the AI writes code, it **verbalises its hypothesis about how that code works** and logs the belief alongside observable state (post 3693). At analysis time, it compares "what I believed" to ground truth, and corrections are easy. "Not always clear to humans, but GPT understands the concept perfectly" (post 3693, author-claim). See `belief-state-logging.md`.
3. **Forced context injection**: some log lines are too important to trust the agent to fetch on its own — **inject them forcibly into the agent's context**, with category markup aiding filtering (post 3693; reinforced in post 3695 for autonomous agents). See `forced-context-injection.md`.

Kirill's case-study confirmation from post 3620 makes the same point in operational terms: alongside migrating to GRACE code markup, the project "updated the logs" because **"standard logging is a relic of the past, where you just print an error or some info — now you need logs so that the AI can see not just the errors but all the information about how the project works and all of its processes being traceable"** (post 3620). [empirical]

### Step 3 — use logs inside the self-correction loop, not in the analytical phase

[author-claim] Tests + logs are placed in the **debug-time self-correction loop** as feedback, not in the upstream design phase (`testing_and_autonomous_agents.md` §"Тесты как часть self-correction loop"). The logs feed the loop with **both binary and detailed signals at equal priority** — "a bare `TEST FAIL` even as a binary signal is bad practice"; modern frameworks spill "a large number of flag-style conclusions" that are easier for GPT to parse (same section). The AI reads those log fragments and emits a code patch — the same shape as SWE-Bench RL training (same section).

This also means LDD is the **foundation for the self-correction loop** as a whole. The loop reduces, mechanically, to: run → emit AI-readable logs → AI reads log → AI patches code → rerun. Anything that degrades the log's AI-readability degrades the loop. See `../06-testing/tests-in-self-correction-loop.md`.

### Step 4 — for complex / fuzzy logic, promote an in-app AI tester or an agent-based tester

[author-claim] For the ERP-style and complex-ETL end of the spectrum, where asserts were never going to work, LDD is usually paired with heavier mechanisms:

- An **AI agent with a console embedded inside the application**, with the API wired to Tools. "Such an agent can make tests much more complex than pytest scripts and produce proper reports" (`testing_and_autonomous_agents.md` §"Почему обычные автотесты"). The author notes this takes ~2 hours to build and "people are just lazy" (same section). See `../06-testing/in-app-ai-console.md`.
- An **AI-developer / AI-tester agent split**: for complex ETL rewrites, the developer-agent writes a Markdown guide for the tester-agent on how to test; on failure the tester-agent cuts the failing data to XML and sends a detailed report back; the developer-agent updates code, manuals, and tests (`testing_and_autonomous_agents.md` §"Agent-based testing при переписывании legacy"). See `../06-testing/agent-based-testing.md`.

Both of these rely on **logs as the inter-agent communication substrate** — the tester reads logs and writes log-shaped XML; the developer reads log-shaped reports. Without LDD-style logging, the inter-agent channel collapses.

### Step 5 — run under autonomous-agent protections

[author-claim] When LDD runs under an autonomous or swarm agent (not a human-in-the-loop), it must be wrapped in **anti-loop protection** — attempt counters plus context injections that return the agent "to sanity" — because failed fixes become few-shots and the agent will otherwise "burn even a billion tokens" looping (post 3695). It must also be integrated with a **knowledge base of prior fixes** that is forcibly injected on regression (post 3695). These are testing-framework concerns but they are LDD-adjacent because they also rely on the log being AI-readable. See `../07-autonomous-agents/anti-loop-protection.md` and `../07-autonomous-agents/forced-context.md`.

### Summary of the LDD stack

From the outside in:

```
outer wrapper:     minimalist smoke asserts (did it crash?)
primary truth:     AI semantic judgment over structured logs
log heuristics:    log-to-code correlation + belief-state + forced context
consumption:       self-correction loop (read log → patch code)
autonomous:        anti-loop + prior-fix KB
fuzzy logic case:  + in-app AI tester or dev-agent/test-agent pair
```

The stack inverts classical TDD: where TDD makes the assert the oracle, LDD makes the **AI + log pair** the oracle, and demotes the assert to a crash-detector.

## Evidence

### Primary source text — `testing_and_autonomous_agents.md`

- §"Адаптация GRACE под автономных и роевых агентов":
  - LDD defined: the AI "evaluates the application's execution trajectory and makes a semantic judgment about correctness"; "much more reliable than strict equality checks." [author-claim]
  - Classical TDD is an "antipattern" for AI because TDD standards are for humans. [author-claim]
  - Autonomous/swarm agents critically require full automated test suites **with rich, correct logs**. [author-claim]
  - LDD unlocks ERP (1C, SAP) automated testing: "if it's hard to check business logic qualitatively with simple equalities, through semantic AI checks you absolutely can." [author-claim]
  - Without automated tests "you can't replace leather-skin programmers with bots." [author-claim]

- §"Почему обычные автотесты":
  - Author has seen client cases where too much assert-testing "collapsed AI quality instead of raising it." [author-claim / empirical]
  - Tests are hostile code: they burn sparse-attention global-anchor budget on cross-module test calls. [author-claim]
  - First concrete LDD move: "throw out asserts, or ask the AI to make them minimalist." [author-claim]
  - Production assert analogue: an LLM call that reads the log and returns judgment. [author-claim]
  - Most real asserts are smoke tests — "did it crash?" — and miss real bugs. [author-claim / empirical]
  - In-app AI console with Tools is the most advanced option; ~2 hours to build; CLI/pytest is the pragmatic fallback. [author-claim]
  - Advanced testing = AI agents inside the application with flexible guides + LDD where tests exist to fill logs with context for the LLM. [author-claim]

- §"Еще один пост про 'тесты как шум'":
  - Tests as "just more code" with strong semantic noise due to heavy cross-module calls. [author-claim]
  - Including tests in the same context as contracts collapses the AI's solution space to "pass these tests". [author-claim]
  - Mindless translation of human coding practices to AI "can even be harmful". [author-claim]

- §"Тесты как часть self-correction loop":
  - Tests have moved from analytical phase to the debug-time self-correction loop; new job is "fix a bug", not "understand the task". [author-claim]
  - Equal importance of binary and detailed signals; "TEST FAIL" alone is bad practice. [author-claim]
  - Modern frameworks flood the LLM with flag-style conclusions in logs. [author-claim]
  - Tests are trained in SWE-Bench-style RL: read log fragments → emit code changes. [research-backed / author-claim]
  - LLM-written tests are typically low quality because training saw "tests + code" pairs but no meta-reasoning about tests. [author-claim]
  - Quote: "LLM is a self-taught parrot when it comes to tests." [author-claim]

- §"Agent-based testing при переписывании legacy":
  - For complex ETL rewrites, pytest autotests are ineffective and miss edge-case bugs. [author-claim]
  - Agent-based testing pattern: dev-agent writes Markdown test-guide for tester-agent; tester-agent cuts failing data to XML and reports back; dev-agent updates code/manuals/tests. [author-claim]
  - Moral: "in multi-agent development, be careful with old TDD practices — they can lead to formally-delivered but non-functional software, and customers will tear your head off." [author-claim]
  - LLMs are very prone to producing code that passes autotests but does not work in real scenarios; agent-based testing catches this. [author-claim]

### Supporting post — post 3693 (Vlad's contracts project)

[empirical / author-claim] Colleague Vlad applied GRACE contracts (282 contracts, 1049 autotests) but without GRACE logging. The agent itself reflected back to Vlad that **"tests without logs are vodka without beer"**; "the AI agent has essentially no other sight besides logs." This is the motivating case study for LDD's necessity alongside contracts (post 3693).

Post 3693 also gives the minimum log heuristics — log-to-code correlation, belief-state logging, forced-context injection — which define what an "AI-readable log" is (full detail in sibling files).

### Supporting post — post 3620 (Kirill's 132k-LOC case)

[empirical] Alongside adopting GRACE markup, the team **modernised its logging** and reported that "standard logging is a relic of the past — now you need logs so that AI can see not just errors but all the information about how the project works, with all processes traceable" (post 3620). Combined with the markup, this correlated with: fewer unnecessary file reads, no rolled-back changes during 20+ hours of continuous autonomous work, and noticeably reduced hallucination (post 3620). This is an operational confirmation that LDD-style logs + GRACE markup are symbiotic at production scale.

### Claim-type summary

The core LDD claims in this file are **[author-claim]** — the author's methodological position, grounded in his training clients' empirical experience. The related facts that **LLMs overfit to strict tests and that vendors removed strict test-like conditions from training** are presented by the author as **[research-backed]** — "see scientific works on this topic above in the channel" — but the specific papers are not named in the `contracts_as_anchor.md` excerpt, so they are cited here at face value. The SWE-Bench-style RL claim about read-log → patch-code is industry-known and presented by the author as reflecting how modern test pipelines are actually trained. The case-study confirmations (posts 3693 and 3620) are **[empirical]** — third-party engineers reporting observations on production-scale projects.

## See also

- `./log-to-code-correlation.md` — the "archimportant" heuristic of injecting function/block-ID tokens into every log line; post-3693 experiments on Gemini attention dropout past ~5000 lines.
- `./belief-state-logging.md` — how the AI verbalises hypotheses about how its own code works and logs them alongside observables.
- `./forced-context-injection.md` — forcing key log lines into agent context rather than hoping the agent fetches them via Tools.
- `../06-testing/classical-tests-antipattern.md` — the detailed antipattern argument against classical TDD with LLMs (test-code hostility, overfitting, self-taught-parrot).
- `../06-testing/tests-in-self-correction-loop.md` — where LDD plugs in: tests as feedback for the debug-time self-correction loop, not upstream oracle.
- `../06-testing/agent-based-testing.md` — dev-agent / tester-agent split for complex ETL, using LDD-style logs as inter-agent substrate.
- `../06-testing/in-app-ai-console.md` — embedded AI agent with Tools inside the running application; a richer assert analogue than pytest.
- `../06-testing/cli-not-custom-mcp.md` — why pytest + bash (standard training-distribution tools) are reliable and custom CLIs/MCPs are not.
- `../07-autonomous-agents/anti-loop-protection.md` — attempt counters + context injection to prevent the LDD loop from looping forever on a bad fix.
- `../07-autonomous-agents/forced-context.md` — the general principle of pushing context instead of trusting the agent to pull it.
- `../07-autonomous-agents/knowledge-base-in-tests.md` — prior-fix KB forced into context on regression; prevents repeat-failure loops inside the LDD cycle.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — why assert-heavy tests burn sparse-attention anchor budget (the mechanism behind "tests are hostile code for AI").
- `../01-transformer-foundations/llm-training-pipeline.md` — why log-reading is in-distribution (FIM, SWE-Bench-style RL) while elaborate custom test DSLs are not.
- `../11-case-studies/vlad-crm-and-contracts.md` — the motivating case ("tests without logs = vodka without beer").
- `../11-case-studies/kirill-132k-loc.md` — 132k-LOC production confirmation that GRACE markup + modernised logs cut rollbacks and hallucination.
- `../13-antipatterns/all-antipatterns.md` — "classical TDD with AI", "TEST FAIL binary signals", and "using strict assertions as primary truth" as consolidated antipatterns.
