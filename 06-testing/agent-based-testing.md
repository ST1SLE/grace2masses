# Agent-Based Testing

## What

Agent-based testing is a GRACE testing pattern used in multi-agent scenarios — most notably when **rewriting legacy code from scratch with AI** or handling complex transformations such as ETL pipelines. [author-claim] Instead of relying on classical `pytest`-style algorithmic autotests, the methodology splits testing across two cooperating AI agents:

- A **developer-agent** that writes production code and ALSO authors a Markdown guide describing how to test the new algorithms.
- A **tester-agent** (framed as an "AI for operation-testing") that follows that Markdown guide, runs the tests itself, and reports findings back. (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

The pattern replaces the idea of "test code as the executable specification" with "natural-language test procedures as the executable specification, run by an agent". [author-claim] Source: (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy").

## Why

### Classical autotests fail on complex ETL

[empirical] The author's direct observation: "Simple automated `pytest` tests for complex ETL transformations are inefficient, especially with AI. They miss very many bugs in various edge cases." (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

[author-claim] Stronger: "Complex ETL on classical autotests with AI — you either will not launch it at all, or you will launch it with an enormous over-expenditure of your time." (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

### LLMs ship code that passes tests but fails in reality

[empirical] "LLMs are extremely prone to releasing code that passes automated tests but does not work on real tasks. So it is very important here to shift the emphasis with AI development in mind." (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

This compounds the classical overfitting problem documented in the sibling file on [classical-tests-antipattern](./classical-tests-antipattern.md): if contracts and tests sit in the same context, the model collapses its solution space to "whatever passes the tests" rather than "whatever satisfies the intent". (testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум'")

### Markdown is a compact representation for agents

[author-claim] "The testing procedures are in their Markdown — if you were to program them, they would be huge modules of code. Instead, the agent simply passes the tests itself." (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

In other words: a testing discipline rich enough to catch ETL edge cases would require a large body of imperative test code, which in turn burns sparse-attention budget via cross-module references (see [classical-tests-antipattern](./classical-tests-antipattern.md)). The same discipline expressed as natural-language guidance for an agent stays compact. [author-claim]

### Moral for multi-agent teams

[author-claim] "In multi-agent development you should be more careful with old practices like TDD — I have written many times that they can simply lead to formal hand-off of non-working software, and customers will tear your head off." (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

[author-claim] In AI contexts, "agentic tests", Log-Driven Development, and "extensive communication on context between the testing agent and the development agent" are often the effective alternatives. (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

## How

### Division of responsibilities

[author-claim] The two-agent arrangement the author applied to a legacy-rewrite project:

1. **AI developer-agent** — responsible for:
   - Producing the production code.
   - Writing a Markdown **guide** to the tester-agent covering:
     - how to use the new algorithms exposed via Tools;
     - how to test them.
2. **AI tester-agent** — responsible for:
   - Reading the Markdown guide.
   - Running the described tests against the Tools.
   - On failure, **extracting the problematic data into XML** and sending a **detailed report** back to the developer-agent describing exactly what failed.

On receiving a failure report, the developer-agent **updates the code, the manuals, and the tests** — all three, not just the code. (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

### Why XML for the failure payload

[author-claim] The tester-agent cuts failing data into XML (not JSON, not YAML) before sending it to the developer-agent. The sources do not state this rationale explicitly in this section, but the choice is consistent with the broader GRACE pattern that XML-like formats are Dyck-compatible and stable in long contexts, unlike JSON brace-counting or non-Dyck YAML. (See [../01-transformer-foundations/xml-vs-json-vs-yaml.md](../01-transformer-foundations/xml-vs-json-vs-yaml.md), [../04-markup-system/xml-like-markup.md](../04-markup-system/xml-like-markup.md).)

### Shape of the loop

The reporting/fix cycle resembles the broader self-correction loop (see [tests-in-self-correction-loop](./tests-in-self-correction-loop.md)), but with a human-language signal path between two agents rather than a test-framework feedback path inside one:

```
developer-agent → writes code + Markdown test guide
                    ↓
tester-agent    → runs tests per guide
                    ↓ (on failure)
tester-agent    → cuts failing data to XML
                → writes detailed report
                    ↓
developer-agent ← updates code, manuals, tests
                    ↻ loop
```

[author-claim] Source sketch: (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy").

### Role of classical autotests — not zero, but narrow

[author-claim] Classical algorithmic autotests still have a place, but a reduced one: "Completely automatic tests on classical algorithms are suitable with AI more for surface-level evaluation of application integrity." (testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy")

They function as a smoke layer, while agent-based tests carry the weight of actual correctness judgment.

### Relationship to Log-Driven Development

[author-claim] Agent-based testing composes with LDD: the tester-agent's "semantic judgment on the trajectory" style is LDD applied at the testing level. (testing_and_autonomous_agents.md §"Адаптация GRACE"; testing_and_autonomous_agents.md §"Почему обычные автотесты")

See:
- [../05-logging-ldd/log-driven-development.md](../05-logging-ldd/log-driven-development.md) — LDD overview.
- [../05-logging-ldd/log-to-code-correlation.md](../05-logging-ldd/log-to-code-correlation.md) — function-ID token injection so the agent can bind logs back to code.

### Where agent-based testing fits among other GRACE testing patterns

- [classical-tests-antipattern](./classical-tests-antipattern.md): explains WHY classical TDD hurts AI — the prerequisite for accepting agent-based testing.
- [tests-in-self-correction-loop](./tests-in-self-correction-loop.md): explains that tests have moved into the debug feedback layer; agent-based testing is the Markdown-driven variant of that same move for complex cases.
- [in-app-ai-console](./in-app-ai-console.md): the "AI agent with a console inside the application" — a more deeply embedded relative of the tester-agent idea.
- [cli-not-custom-mcp](./cli-not-custom-mcp.md): explains why the tester-agent is reliable when it drives `pytest` via bash (both in training distribution) instead of custom DIY tooling.

## Evidence

### Primary source

- **testing_and_autonomous_agents.md §"Agent-based testing при переписывании legacy"** — single, concentrated source for this pattern. All non-bracketed claim-type labels above draw from this section unless otherwise noted. Key claims, verbatim-translated from the Russian:
  - "Simple automated `pytest` tests for complex ETL transformation are ineffective, especially with AI. They miss very many bugs in different edge cases." [empirical]
  - "I placed a larger bet on agentic testing → AI-developer wrote the AI-agent for operation-testing a guide: how to use the new algorithms in Tools, and how to test them first." [author-claim]
  - "Testing procedures are in their Markdown — if you were to program, these would be huge modules of code. Instead, the agent simply passes the tests itself." [author-claim]
  - "If something fell, he cut the problem data into XML for the AI-developer and wrote him a detailed report on what exactly fell. The AI-developer updated the code, manuals, and tests." [author-claim]
  - "In the topic of multi-agent development, one must be more careful with old practices like TDD — I have written many times that they can simply lead to the formal handover of non-working software, and customers will tear your head off." [author-claim]
  - "In the case of AI, 'agentic tests', Log-Driven Development, and also extensive communication on context between the testing agent and the development agent are very often effective." [author-claim]
  - "Completely automatic tests on algorithms by the classics are suitable with AI more for surface-level evaluation of application integrity." [author-claim]
  - "LLMs are extremely prone to releasing code that passes automated tests but does not work on real tasks." [empirical]
  - "Complex ETL on classical autotests with AI — you either will not launch at all, or will launch with huge over-expenditure of your time." [author-claim]

### Supporting sources in the same file

- **testing_and_autonomous_agents.md §"Почему обычные автотесты считаются антипаттерном"**: frames the broader reason why `assert`-centric tests are semantic noise to LLMs, motivating the move to agent-based procedures. [author-claim]
- **testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"**: places agent-based testing within the modern shift of tests from the analytical phase to the feedback layer. [author-claim]
- **testing_and_autonomous_agents.md §"Адаптация GRACE"**: "For autonomous/swarm agents, complete sets of automated tests with rich and correct logs are critically important." [author-claim] — pairs with agent-based testing because the Markdown guide in the two-agent pattern is what makes "rich and correct" feasible for fuzzy ERP-class logic.

### Cross-referenced claim types

- The author's empirical observation that LLMs release code passing tests but failing reality is corroborated by the overfitting argument in testing_and_autonomous_agents.md §"Еще один пост про 'тесты как шум для ИИ'": "If you directly include tests in the same conditions of contracts with AI, then AI simply truncates its solution space and generates at once code that passes your tests." [author-claim]
- The author's "testing procedures in Markdown" claim is consistent with broader GRACE positioning that natural-language-for-agents is a first-class runtime artifact, not just documentation (see [../04-markup-system/in-source-documentation.md](../04-markup-system/in-source-documentation.md)). That broader positioning is documented in post 3878 — the agent-based testing section itself does not cite research; it is presented as author's practice. [author-claim]

### What the sources do NOT specify

- The sources do not specify the **concrete structure** of the Markdown test guide (sections, headers, what goes where). Only the general idea — "how to use the new algorithms in Tools, and how to test them first" — is given. [gap]
- The sources do not specify the **XML schema** used when the tester-agent cuts failing data. [gap]
- The sources do not specify the **exact prompts** of either agent. [gap]
- The sources do not specify the **project details** (name, domain, tech stack, scale) of the legacy-rewrite case. The only framing is "if you are rewriting legacy code from scratch anew with AI" and the domain example "complex ETL transformation". [gap]
- The sources do not specify any **research paper** backing this specific pattern. The other GRACE testing patterns have research backing (SWE-Bench-style RL for the self-correction loop, arxiv 2601.16661v1 for in-source docs, etc.), but agent-based testing is presented as author's practice. [gap]

## See also

- [classical-tests-antipattern](./classical-tests-antipattern.md) — Why classical TDD hurts AI; the prerequisite for adopting agent-based testing.
- [tests-in-self-correction-loop](./tests-in-self-correction-loop.md) — The broader shift of tests to the debug feedback layer; agent-based testing is its Markdown-driven variant for complex cases.
- [in-app-ai-console](./in-app-ai-console.md) — A closely related pattern: embed an AI agent with a console inside the application itself so it can run deeper tests than scripted `pytest`.
- [cli-not-custom-mcp](./cli-not-custom-mcp.md) — Why the tester-agent should drive standard tools (`pytest`, bash) rather than DIY "kulibin" MCPs that LLMs silently ignore.
- [../05-logging-ldd/log-driven-development.md](../05-logging-ldd/log-driven-development.md) — LDD; the semantic-judgment style of evaluation that underpins agent-based test reasoning.
- [../05-logging-ldd/log-to-code-correlation.md](../05-logging-ldd/log-to-code-correlation.md) — Function-ID token injection into log lines; enables the tester-agent to bind execution traces back to source.
- [../05-logging-ldd/forced-context-injection.md](../05-logging-ldd/forced-context-injection.md) — Pushing critical log lines into an agent's context; relevant when the tester-agent's report must reach the developer-agent reliably.
- [../07-autonomous-agents/anti-loop-protection.md](../07-autonomous-agents/anti-loop-protection.md) — Attempt counters and sanity-injection that agent-based testing presumes for autonomous-mode operation.
- [../07-autonomous-agents/forced-context.md](../07-autonomous-agents/forced-context.md) — The stabilization mechanism that keeps the failure report visible to the developer-agent.
- [../07-autonomous-agents/knowledge-base-in-tests.md](../07-autonomous-agents/knowledge-base-in-tests.md) — Regression memory for tester-agents, complementing single-shot reports with accumulated fix history.
- [../04-markup-system/xml-like-markup.md](../04-markup-system/xml-like-markup.md) — Rationale for XML in the tester-agent's failure payload (format stability in long contexts).
- [../01-transformer-foundations/xml-vs-json-vs-yaml.md](../01-transformer-foundations/xml-vs-json-vs-yaml.md) — Formal and empirical case for XML over JSON/YAML; informs the "cut to XML" step.
