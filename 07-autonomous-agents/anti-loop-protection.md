# Anti-Loop Protection

## What

**Anti-Loop Protection** is a pair of defensive mechanisms — **attempt counters** plus **forced context injections** — built directly into a test framework so that an autonomous or swarm-based AI agent cannot enter, and stay stuck in, a cycle of failed fixes. [author-claim] (post 3695)

The protection has two operational parts:

1. **Attempt counters** — explicit bookkeeping of how many times the agent has tried to fix the same failure, used as a hard boundary on further attempts. [author-claim] (post 3695)
2. **Context injections** — deliberate pushes of information (prior fix knowledge, key log lines, reminders) into the agent's context when it is looping or stuck, to "automatically return it to its senses" rather than waiting for the agent to self-rescue. [author-claim] (post 3695)

These mechanisms must be **designed into the test framework from the start**; they are not a post-hoc patch. [author-claim] (post 3695)

> "Regular autotests are in fact NOT sufficiently compatible with autonomous AI." [author-claim] (post 3695)

## Why

### Failed fixes become few-shots — and reinforce the wrong hypothesis

The core pathology is specific to how GPT behaves on a long trajectory of failures:

- When an autonomous agent makes a failed fix and then sees its own prior attempts in-context, those failed attempts operate on the model as **few-shot examples**. [author-claim] (post 3695)
- Each new loop makes the model **more confident** about the same failed approach, not less — it reads its own prior attempts as "this is how the problem is being solved here." [author-claim] (post 3695)
- "'Failures' for [GPT] in reality become few-shot, and the AI with every erroneous cycle only becomes more and more confidently locked in the loop." [author-claim] (post 3695)

This inverts the human intuition that "it will figure out the right answer eventually by trial and error." Without external guardrails, the failure trajectory is self-reinforcing.

### The cost of an unbounded loop is catastrophic

Because failed attempts feed the next attempt, a loop does not decay:

- A swarm or even a single autonomous agent can "dumbly hang and burn up to a billion tokens." [author-claim] (post 3695)
- Wider context: GRACE-style work typically concentrates on **few, large, structured prompts** (Architect-mode sessions of ~20 prompts generating 300-400k tokens — see `../08-workflow-and-phases/large-rare-prompts-billing.md`). A looping autonomous agent breaks that economics completely by producing runaway cost with no forward progress. (contextual cross-reference, scope of cost order of magnitude from post 3695)

### Regular autotests are not compatible with autonomous agents

The author makes this claim explicitly and categorically:

- "Regular autotests are in fact NOT sufficiently compatible with autonomous AI." [author-claim] (post 3695)
- The incompatibility is not a matter of tuning — it is structural: classical autotests were designed with the assumption that the thing being debugged is a human or a non-looping program, not an autoregressive model whose prior failures are taken as examples to imitate.
- See `../06-testing/classical-tests-antipattern.md` for the broader argument that classical TDD imports human-oriented assumptions that hurt AI agents. (cross-reference)

### Self-rescue via Tools is naive

A common but wrong assumption is that the agent will simply call a Tool to disambiguate or gather info when it gets stuck:

- "Simply counting on the agent, especially if it has gone into a loop, to itself call Tools to read important information — is just naive." [author-claim] (post 3695)
- This is why the second half of anti-loop protection is **push**, not **pull**: important context is injected into the agent, not left for the agent to fetch.
- Cross-link: `forced-context.md` (T36) generalizes this push-over-pull discipline for autonomous agents; `../05-logging-ldd/forced-context-injection.md` (T29) covers the log-line variant of the same principle. (cross-reference; posts 3693, 3695)

## How

### Mechanism 1 — Attempt counters

The sources describe attempt counters only at principle level ("usually counters of attempts" — post 3695); they do not specify a numeric threshold, a storage schema, or an exact abort behavior. The sources do not specify these details.

What the sources do specify:

- Counters are a property of the **test framework**, not of the agent. [author-claim] (post 3695)
- They are one of the two required anti-loop ingredients, alongside context injections. [author-claim] (post 3695)
- Their job is to put a bound on how many times the agent may retry the same failure before the framework intervenes. [author-claim] (post 3695)

### Mechanism 2 — Forced context injections

Injections are the active intervention that makes a counter useful — hitting a counter without pushing corrective context back into the agent would just stall the loop without redirecting it.

What the sources specify:

- "Various injections into the agent's context, to automatically return it to its senses." [author-claim] (post 3695)
- The injected content should include knowledge from a **knowledge base of previous fixes** so that when a regression repeats, the agent is reminded of how the issue was handled before. [author-claim] (post 3695) — This is the dedicated topic of `knowledge-base-in-tests.md` (T37). (cross-reference)
- Injections must be **forced**, not optional: the agent cannot be trusted to request the needed context itself, especially when looping. [author-claim] (post 3695)
- For log-line-level injections, see `../05-logging-ldd/forced-context-injection.md` (T29). (cross-reference; post 3693, 3695)

### Mechanism 3 — Integration with a knowledge base of prior fixes

The author explicitly ties anti-loop protection to test-level regression memory:

- "It is very important for agents that tests are integrated with a knowledge base of previous fixes, and if something breaks again, the relevant knowledge is forcibly injected into the agent's context." [author-claim] (post 3695)
- This turns anti-loop protection from a local "retry cap" into a **cross-session memory** of known failure modes.
- Fuller treatment in `knowledge-base-in-tests.md` (T37). (cross-reference)

### Where this protection belongs

- In the **test framework itself** — counters and injections are part of how tests are written, not a layer added after tests already exist. [author-claim] (post 3695)
- This has to be accepted at the **early design stage** of the test suite: "All this is not hard, but it is important at the early stage of designing your tests to take into account the specifics of their compatibility with autonomous agents." [author-claim] (post 3695)

### The broader design implication for test frameworks

The author frames anti-loop protection as one of several structural adjustments needed before an autonomous agent can use a test suite reliably:

- Rich, AI-friendly logs (cross-link `../05-logging-ldd/log-driven-development.md` and `../05-logging-ldd/log-to-code-correlation.md`). (cross-reference)
- Integration with a knowledge base of prior fixes (T37). (cross-reference)
- Anti-loop protection proper (this file). (post 3695)

All three have to be in place together; anti-loop protection without rich logs still leaves the agent blind about what actually went wrong, and rich logs without anti-loop protection still let the agent burn through a billion tokens convinced of the wrong fix. [empirical / author-claim] (post 3695)

### Why this is not a "tuning" issue

It is worth separating anti-loop protection from ordinary retry policy:

- Ordinary retry policy assumes the retry might succeed because the failure was transient (flaky network, nondeterminism, etc.).
- Anti-loop protection assumes the retry might make the agent **worse**, because it confirms a wrong hypothesis. [author-claim] (post 3695)
- Therefore the counter's role is not "give it another chance" — it is "do not let it convince itself further." [author-claim] (post 3695)

## Evidence

All content in this file is sourced from one post:

- **Post 3695** (2026-03-21) — The canonical anti-loop protection post. Introduces:
  - The few-shot reinforcement mechanism by which failed fixes compound agent confidence.
  - The "billion tokens burned by a hung swarm" cost model.
  - Attempt counters + context injections as the two required ingredients.
  - The KB-of-previous-fixes integration.
  - The blunt claim that regular autotests are not compatible with autonomous agents.
  - The naive-self-rescue-via-Tools antipattern.

Supporting / adjacent posts (used only for cross-referencing, not for primary claims in this file):

- **Post 3693** — log-to-code correlation and belief-state logging; the "forced injection of important log lines" idea that anti-loop protection builds on.
- Broader testing-autonomous-agents discussion: `testing_and_autonomous_agents_output.md` §"Адаптация GRACE" — frames why autonomous and swarm agents need full auto-test kits with rich logs as a precondition; does not add new anti-loop specifics beyond post 3695.

Classifications:

- [author-claim] for all specific mechanics — the sources present anti-loop protection as the author's own framework adaptation to autonomous agents; no external paper is cited as evidence within this scope.
- [empirical] where the author reports direct observation ("I have already collected plenty of rakes here, though solvable ones" — post 3695).

The sources do not specify:

- Exact numeric attempt-counter thresholds.
- Concrete data-structure schemas for the knowledge base of previous fixes.
- Benchmark numbers on the cost savings from adding anti-loop protection.
- A named reference paper specifically on anti-loop protection. (The "billion tokens" figure is the author's characterization of worst-case cost, not a cited measurement.)

## See also

- `forced-context.md` — generalization of the "push, don't rely on pull" principle for autonomous agents.
- `knowledge-base-in-tests.md` — the regression-memory component that anti-loop injections draw from.
- `swarm-vs-single-agent.md` — why anti-loop protection becomes non-negotiable at swarm scale.
- `../05-logging-ldd/forced-context-injection.md` — log-line-level variant of forced context.
- `../05-logging-ldd/log-driven-development.md` — LDD as the reading surface the agent uses during self-correction.
- `../05-logging-ldd/log-to-code-correlation.md` — function/block ID injection that makes logs legible to the agent in the first place.
- `../06-testing/classical-tests-antipattern.md` — why classical TDD is hostile to AI agents (and therefore why anti-loop protection cannot be bolted onto classical tests after the fact).
- `../06-testing/tests-in-self-correction-loop.md` — tests repositioned into debug feedback, the environment anti-loop protection operates in.
- `../13-antipatterns/all-antipatterns.md` — entries on "Assuming the agent will self-rescue via Tools" and on classical TDD with AI.
