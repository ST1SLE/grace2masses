# Tests in the Self-Correction Loop

## What

In modern AI-native engineering, **tests have relocated**. They are no
longer primarily analytical artifacts that help a model *understand* a
task before writing code. Instead, they have been pushed into the
**self-correction loop** during debug, where their job is to help the
model *fix a specific bug* it has just produced
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [empirical / author-claim]

Concretely, a test in this new role is:

- a **feedback signal** consumed by an agent's loop, not a spec read
  before coding;
- a source of **structured, flag-style log evidence** about a specific
  failure mode;
- a training target for **SWE-Bench-style reinforcement learning**,
  which teaches a model to read log fragments and emit patches
  (`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
  loop"). [research-backed]

The shift is often summarised as: the goal of modern test work with AI
is **not** "help the model understand the task" but **"fix the bug"**
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [author-claim]

## Why

### Why tests were moved out of the analytical phase

Placing tests in the analytical phase (classical TDD style) collapses
the solution space. An LLM given a contract plus a full set of tests in
context will overfit: it generates whatever passes the tests, not code
that actually works in production
(`testing_and_autonomous_agents.md` §"Еще один пост про 'тесты как
шум'"). [empirical]

LLM vendors reinforced this by **burning strict test-like conditions
out of training**, because they caused overfitting that did not
generalise (`contracts_as_anchor.md` §"Почему AI-contracts не равны
классическому DbC"). [author-claim] The model therefore cannot be
expected to treat analytical-phase tests as spec; by construction, it
treats them as constraints to satisfy.

Tests are also "just more code" from the sparse-attention point of
view, and unusually hostile code: they reference many modules across
the codebase and therefore burn scarce global-anchor budget
(`testing_and_autonomous_agents.md` §"Почему обычные автотесты",
§"Еще один пост про 'тесты как шум'"). [author-claim] Leaving them in
the analytical phase compounds this cost.

### Why the self-correction loop is the right home

The self-correction loop is where the **concrete, local question**
lives: "this code just produced this behaviour — is it correct, and
how do I fix it?" At that moment the model genuinely benefits from
execution evidence. LDD makes this feedback semantically rich instead
of brittle (cross-link `../05-logging-ldd/log-driven-development.md`;
`testing_and_autonomous_agents.md` §"Адаптация GRACE"). [author-claim]

SWE-Bench-style RL training aligns with this setup. Models are trained
on the specific trajectory **"read log fragment → emit patch"**
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [research-backed] A test that lives in the self-correction
loop is therefore on the training distribution; a test consumed only
during analysis is not.

### Why `TEST FAIL` alone is bad practice

Modern test methodology treats **binary and detailed signals as
equally important**. Emitting only a bare `TEST FAIL` flag is
explicitly called out as bad practice — it strips away exactly the
log texture the model was trained to read
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [author-claim] Pass/fail tells the agent *that* something
broke; detailed flag-style log conclusions tell it *what* and *where*,
which is what the post-SWE-Bench patch head expects.

## How

### The new contract between tests and the agent

| Old role (analytical TDD) | New role (self-correction loop) |
|---|---|
| Spec the model reads before coding | Feedback the model reads after coding |
| Assert equality as ground truth | Structured logs plus semantic LDD judgment |
| Binary pass/fail is the output | Binary flag **and** detailed per-step flags |
| Few/no logs beyond assertions | Flood of flag-style conclusions per run |
| Coverage metric mindset | "Fix this bug" mindset |

(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop", §"Почему обычные автотесты") [author-claim]

### Signals modern test frameworks should emit

1. **A clear binary flag** for pass/fail, so a loop can decide whether
   to continue iterating.
2. **Flag-style detailed conclusions** embedded in the log — short,
   parseable, structured — covering intermediate steps, preconditions,
   observed values, and divergence from expectations
   (`testing_and_autonomous_agents.md` §"Тесты как часть
   self-correction loop"). [author-claim]

Modern AI test frameworks deliberately **flood the model with many
"flag"-style conclusions** through logging, because this shape is
exactly what GPT finds easy to parse
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [author-claim] The self-correction agent then consumes the
flag stream to localise the fault.

This aligns with GRACE's broader logging discipline:
log-to-code correlation, belief-state logs, and forced context
injection are what make these flag-style logs actually usable
(cross-links `../05-logging-ldd/log-to-code-correlation.md`,
`../05-logging-ldd/belief-state-logging.md`,
`../05-logging-ldd/forced-context-injection.md`).

### How SWE-Bench-style RL shaped this

SWE-Bench-style reinforcement learning trains the model directly on
the loop: **read log fragments, emit code patches**
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [research-backed] Two consequences follow:

1. **Logs are first-class training input**, so log shape matters —
   coarse pass/fail wastes the trained behaviour, while flag-rich logs
   activate it.
2. **The model is trained to patch, not to plan from tests**. Tests
   viewed as planning documents miss the trajectory the model was
   actually trained on.

A separate note from the same source addresses a common objection:
*if GPT is good at few-shot learning, why does it generalise badly
from tests?* The answer is **which few-shots were present during
training**. High-level analytical reasoning about tests was
deliberately excluded from LLM training. The model saw pairs of
"tests + code" but **no accompanying meta-reasoning about tests**,
neither in pre-training nor in RL
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [research-backed / author-claim] So the model can produce
plausible-looking tests by pattern-matching, but it does so as a
**"self-taught parrot"** — a phrase the author uses explicitly
(`testing_and_autonomous_agents.md` §"Тесты как часть self-correction
loop"). [author-claim]

The practical consequence: do not rely on the model to *design* the
test strategy, and do not treat LLM-authored tests as a source of
truth. Use tests where the model *was* trained — inside the
debug-fix loop.

### Practical checklist for moving tests into the loop

- **Position**: run tests during debug/self-correction, not during
  initial specification intake. Contracts (cross-link
  `../03-contracts/contracts-overview.md`) carry the spec role;
  tests carry the feedback role. [author-claim]
- **Emit binary and detailed signals together**. Never emit bare
  `TEST FAIL` (`testing_and_autonomous_agents.md` §"Тесты как часть
  self-correction loop"). [author-claim]
- **Make logs flag-rich**. Add per-step, short, parseable flags so
  the model can read the fragment and act
  (`testing_and_autonomous_agents.md` §"Тесты как часть
  self-correction loop"). [author-claim]
- **Pair with LDD**. Let the AI make a semantic judgment from the
  execution trajectory, not only from assertions (cross-link
  `../05-logging-ldd/log-driven-development.md`;
  `testing_and_autonomous_agents.md` §"Адаптация GRACE"). [author-claim]
- **Keep contract-in-context away from test-in-context during
  analysis**. That combination collapses the solution space via
  overfitting (`testing_and_autonomous_agents.md` §"Еще один пост
  про 'тесты как шум'"). [empirical]
- **Pair with anti-loop protection at autonomous-agent scale**. A
  self-correction loop without attempt counters and forced context
  can burn billions of tokens on the same failed patch (cross-link
  `../07-autonomous-agents/anti-loop-protection.md`; post 3695).
  [author-claim]

### What this does *not* change

- Tests still exist. The move is about **where** they live in the
  workflow, not an abolition of testing
  (`testing_and_autonomous_agents.md` §"Тесты как часть
  self-correction loop"). [author-claim]
- Classical analytical-TDD is still an antipattern for AI-native
  workflows (cross-link `./classical-tests-antipattern.md`).
  [author-claim]
- Agent-based testing (cross-link `./agent-based-testing.md`) remains
  the recommended way to cover complex business flows that strict
  asserts cannot express. [author-claim]
- CLI-backed standard runners (`pytest`, bash) stay preferred over
  DIY "kulibin" tooling because models were trained on them
  (cross-link `./cli-not-custom-mcp.md`; post 3834). [author-claim]

## Evidence

- `testing_and_autonomous_agents.md` §"Тесты как часть
  self-correction loop" — primary source: tests have moved out of the
  analytical phase into the self-correction loop; goal is "fix the
  bug" not "understand the task"; both binary and detailed signals
  matter; `TEST FAIL` alone is bad practice; modern frameworks emit
  flag-rich logs; SWE-Bench-style RL trains models to read log
  fragments and emit patches; LLM is a "self-taught parrot" for
  test-authoring because meta-reasoning about tests was not in
  training.
- `testing_and_autonomous_agents.md` §"Почему обычные автотесты" —
  the sparse-attention cost of test code; LDD replaces brittle
  asserts; log-rich testing is "advanced testing".
- `testing_and_autonomous_agents.md` §"Еще один пост про 'тесты как
  шум'" — contract + tests in context together collapses the solution
  space (overfitting).
- `testing_and_autonomous_agents.md` §"Адаптация GRACE" — LDD as the
  semantic-judgment replacement for strict equality; foundation for
  the self-correction loop.
- `contracts_as_anchor.md` §"Почему AI-contracts не равны
  классическому DbC" — LLM vendors burned strict test-like conditions
  out of training because they caused overfitting.
- Post 3695 — anti-loop protection is mandatory when tests run inside
  an autonomous self-correction loop; without it agents burn tokens
  on the same failed fix.
- No arxiv / paper URL is given in the sources for SWE-Bench itself;
  the sources do not specify a SWE-Bench paper link. The reference is
  to "SWE-Bench-style RL" as a training regime
  (`testing_and_autonomous_agents.md` §"Тесты как часть
  self-correction loop").

## See also

- `./classical-tests-antipattern.md` — why classical TDD is harmful
  to LLMs; the negative framing of this file's positive claim.
- `./agent-based-testing.md` — agent-pair testing as the practical
  alternative for complex business logic.
- `./in-app-ai-console.md` — embedding an AI test agent with Tools
  inside the application.
- `./cli-not-custom-mcp.md` — why standard runners (`pytest`, bash)
  outperform DIY test CLIs in long agent trajectories.
- `../05-logging-ldd/log-driven-development.md` — LDD is the
  semantic-judgment layer that makes the self-correction loop
  generalise.
- `../05-logging-ldd/log-to-code-correlation.md` — log-line tokens
  that bind flag-style logs back to code.
- `../05-logging-ldd/belief-state-logging.md` — what the AI "believed"
  vs ground truth inside the loop.
- `../05-logging-ldd/forced-context-injection.md` — pushing critical
  log lines into the agent's context instead of hoping it fetches
  them.
- `../07-autonomous-agents/anti-loop-protection.md` — mandatory
  safeguard for any self-correction loop running autonomously.
- `../07-autonomous-agents/knowledge-base-in-tests.md` — regression
  memory that feeds the self-correction loop.
- `../03-contracts/contracts-overview.md` — contracts carry the spec
  role that tests vacated.
