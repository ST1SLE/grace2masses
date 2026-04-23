# Knowledge Base in Tests

## What

A **knowledge base in tests** is a GRACE pattern where the autonomous-agent test infrastructure is explicitly integrated with a persistent **knowledge base of previous fixes**. When a regression occurs — i.e. a test fails on territory that has already been fixed before — the system **forcibly injects the relevant KB entries into the agent's context** so that the agent is reminded of the prior failure and the fix that resolved it [author-claim] (post 3695).

This is not a generic "memory" layer for the agent. It is a narrow, load-bearing mechanism bolted onto the test framework with one job: prevent the agent from repeating a fix pattern that has already been tried and has already failed, or from regressing on logic that has already been stabilized [author-claim] (post 3695).

It is one of two explicit mechanisms the author lists under GRACE's **anti-loop protection** for autonomous / swarm agents; the other is attempt counters plus context injections to force sanity back into the agent [author-claim] (post 3695). Cross-link: `anti-loop-protection.md` and `forced-context.md`.

## Why

The motivating problem is specific to autonomous and swarm agents and does not exist in the same form for human-driven development [author-claim] (post 3695).

### The repeat-failure loop

The author's observation: when an autonomous agent retries a fix, each failed attempt effectively becomes a **few-shot example** for the model. The model does not treat the failure as a reason to change approach — it treats the failed trajectory as reinforcement. "Failures for it in reality become few-shot, and the AI with each erroneous cycle only becomes more confidently stuck in the loop." [author-claim] (post 3695).

This is why classical retry logic is not enough. The model is not making an error of arithmetic that a rerun will correct; it is drifting deeper into the same wrong pattern with growing confidence. Without external intervention, a swarm or even a single autonomous agent "can dumbly hang and burn a billion tokens" [author-claim] (post 3695).

### Why the agent will not self-rescue

A naive design assumes the agent will, on its own, call Tools to fetch relevant past-fix information when it notices it is stuck. The author flags this assumption as "naive" — "expecting that the agent, especially if it's looping, will itself call Tools to read important information is simply naive" [author-claim] (post 3695). The agent in a loop is **exactly the agent least able to break out of the loop through self-query**, because its context is already saturated with the wrong few-shots.

Therefore the knowledge of past fixes must be **pushed**, not pulled. The test framework does the pushing.

### Why the test framework is the right hook

In GRACE's operational layer, tests are relocated **from the analytical phase into the self-correction loop** — their job is "help the model fix a specific bug", not "help the model understand the task" [author-claim] (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"). Because the test system is already the trigger point for every regression event, it is also the natural place to hang an associative memory of "last time this test failed like this, here is what fixed it."

This also follows from another GRACE-wide premise: "classical autotests are **not compatible** with autonomous AI to the necessary degree" [author-claim] (post 3695). Simply porting a human-era test suite into an autonomous-agent pipeline is a failure mode. Tests must be **co-designed with the agent's context mechanics** from the start. The KB-in-tests integration is part of that co-design.

### What breaks without it

Without this mechanism (explicit claim from the author) (post 3695):

- Autonomous agents enter **repeat-failure loops** — the same bug is "fixed" by the same wrong approach, fails again, becomes a stronger few-shot, is tried again with more confidence.
- Token burn scales without bound; "a billion tokens" is the author's order-of-magnitude figure [author-claim] (post 3695).
- **Regular autotests are not compatible with autonomous agents** without these protections [author-claim] (post 3695).

The KB-in-tests integration is presented as preventive infrastructure: "it's not complex, but it's important to consider, at an early design stage of your tests, the specifics of their compatibility with autonomous agents." [author-claim] (post 3695).

## How

The author describes the mechanism at the level of principles and essential moving parts rather than a full algorithmic specification. The facts below are what the source material explicitly states; specifics of KB schema, eviction policy, or matching heuristics are not given — **the sources do not specify** (post 3695).

### The essential mechanic (post 3695)

Two things must be wired together:

1. **A knowledge base of previous fixes.** A persistent store of past-failure → past-fix entries, accumulated across the project's history of test runs [author-claim] (post 3695).
2. **Test-framework integration.** The KB must be cross-linked to the test suite so that a test failure can retrieve the relevant prior-fix entries [author-claim] (post 3695).

And one thing must happen at regression time:

- **On regression, force the relevant KB entries into the agent's context.** The author uses the word "принудительно" — "forcibly" — emphasizing that the agent must not be relied upon to pull the information itself [author-claim] (post 3695).

### What counts as a "regression" here

The author's framing: "if something fell again" — i.e. the same (or a functionally related) test has failed before and has been fixed before [author-claim] (post 3695). The exact matching criterion (test ID, error signature, code region) is not specified — **the sources do not specify** (post 3695).

### What "force into context" means

This is an instance of the broader **forced-context injection** pattern that GRACE uses throughout its autonomous-agent machinery [author-claim] (post 3695). In the same post, the author generalizes the principle:

> "Forced context is a critically important concept for the autonomous agent, because it allows stabilizing the agent in complex tasks." [author-claim] (post 3695).

Forced context appears in two adjacent GRACE mechanisms (cross-link):

- **Log-line injection** — important log lines are pushed into agent context so the agent is not left to decide whether to call a log-fetch Tool (post 3693; cross-link `../05-logging-ldd/forced-context-injection.md` and `forced-context.md`).
- **KB-of-fixes injection** — prior-fix KB entries are pushed into agent context on regression (post 3695; this file).

The underlying principle is the same: push when the cost of the agent not having the information is catastrophic (loops, silent failures), and when the agent in-loop is the worst possible querier of that information [author-claim] (post 3695).

### Placement in the anti-loop stack

Post 3695 groups the KB-in-tests integration under a two-part anti-loop design for autonomous-agent test suites [author-claim]:

1. **Anti-Loop Protection** — attempt counters + context injections to "automatically return the agent to its senses" [author-claim] (post 3695). Covered in `anti-loop-protection.md`.
2. **Knowledge-base accumulation in tests** — the mechanism this file documents [author-claim] (post 3695).

Both are required; both are part of "designing tests, from an early stage, with the specifics of their compatibility with autonomous agents in mind" [author-claim] (post 3695).

### Relationship to belief-state logs

A KB-in-tests entry ideally captures not just "the patch that fixed it" but the context of the prior fix. GRACE logs are written with **belief-state** semantics — when the AI writes code, it verbalizes its hypothesis about how the code works, so that post-fact analysis compares "what I believed" vs. ground truth [author-claim] (post 3693; cross-link `../05-logging-ldd/belief-state-logging.md`). A KB built on top of such logs therefore records not just mechanical patches but the reasoning context around them, which is what a future agent most needs to avoid the same loop.

Note: this connection between belief-state logs and the KB-of-fixes is **not explicitly stated in post 3695** — the post only names KB integration as a mechanism. Belief-state logging is documented in post 3693. The connection is natural given both are presented as part of the same autonomous-agent-ready test infrastructure in the same pair of posts (3693 and 3695), but the author does not explicitly wire the two together. Readers should not take the link as a direct author-claim.

### Implementation specifics not in the sources

The following are **not specified in the sources** (post 3695):

- Whether the KB is a file-based store, a vector DB, a graph DB, or a sqlite-vec hybrid.
- The schema of a KB entry beyond "prior fix".
- The retrieval method used to pick "relevant" entries for a given regression.
- Whether KB entries can be revised, deprecated, or merged.
- How the KB is scoped — per-project, per-module, per-test, per-agent.
- Any concrete code example.

Readers who need these details should treat them as design choices to be filled in per project, consistent with the broader GRACE principles (KEYWORDS/LINKS enrichment for retrieval, forced context for injection, test framework as the trigger).

## Evidence

### Primary source

- **Post 3695** (2026-03-21) — the only source in the supplied material that names "knowledge base in tests" as a GRACE mechanism [author-claim]. Direct quotes (translated):
  - Section header "Накопление базы знаний в тестах" — "Accumulation of knowledge base in tests" [author-claim] (post 3695).
  - "It is very important for agents that tests be integrated with a knowledge base of previous fixes, and if something fell again, forcibly inject this knowledge into the agent's context." [author-claim] (post 3695).
  - "Forced context is a critically important concept for the autonomous agent, because it allows stabilizing it in complex tasks." [author-claim] (post 3695).
  - "Simply expecting that the agent, especially if it's looping, will itself call Tools to read important information — is simply naive." [author-claim] (post 3695).
  - "Regular autotests in reality are NOT sufficiently compatible with autonomous AI." [author-claim] (post 3695).

### Supporting sources

- **Post 3693** (2026-03-21) — motivating context for autonomous-agent log/test infrastructure; documents forced context injection of log lines and belief-state logging as sibling mechanisms in the same design [author-claim] (post 3693).
- **testing_and_autonomous_agents.md §"Тесты как часть self-correction loop"** — establishes that modern GRACE-aligned test design puts tests in the debug-time feedback loop, not the analytical phase; makes the test framework the natural hook for context injection [author-claim] (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop").
- **testing_and_autonomous_agents.md §"Адаптация GRACE"** — overall framing that autonomous / swarm agents require complete test suites with rich, correct logs, and that classical TDD practices break under autonomous use [author-claim].

### Research backing in sources

Post 3695 does not cite an external research paper for the KB-in-tests mechanism specifically — **the sources do not specify** a paper for this mechanism. The author presents it as a design finding from his own work on "finishing GRACE for a swarm of automatic agents" [author-claim] (post 3695).

Adjacent research referenced in GRACE's autonomous-agent section (and cited elsewhere in this KB) provides the surrounding grounding:
- SWE-Bench-style reinforcement learning trains models to read log fragments and emit patches [author-claim] (testing_and_autonomous_agents.md §"Тесты как часть self-correction loop").

### Claim-type summary

All concrete claims about the KB-in-tests mechanism in this file are **[author-claim]** from post 3695, unless stated otherwise. The author presents them as distilled findings from his work adapting GRACE for autonomous and swarm agents. No external paper is cited for this specific mechanism in the sources.

## See also

- `anti-loop-protection.md` — the other half of GRACE's two-part anti-loop design for autonomous-agent test suites (attempt counters + context injections).
- `forced-context.md` — the general forced-context principle underlying KB-in-tests injection.
- `swarm-vs-single-agent.md` — scaling context in which KB-in-tests and anti-loop protection become non-negotiable.
- `../05-logging-ldd/forced-context-injection.md` — the log-line analog of the same forced-context principle.
- `../05-logging-ldd/belief-state-logging.md` — the hypothesis-recording logging pattern that makes KB entries semantically rich.
- `../05-logging-ldd/log-to-code-correlation.md` — the log-identifier heuristic that makes log-anchored KB entries retrievable at all.
- `../06-testing/tests-in-self-correction-loop.md` — why the test framework is the right hook for regression-time context injection.
- `../06-testing/classical-tests-antipattern.md` — why classical autotests without this infrastructure break under autonomous-agent use.
