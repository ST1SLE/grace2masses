# Belief-State Logging

## What

Belief-state logging is the GRACE heuristic that, when the AI writes code, it should
**verbalize its own hypothesis** about how that code is supposed to work — its
"state of belief" (Russian: *состояние веры*) — and then have the running code
emit that hypothesis into the log alongside the observable runtime state. The
log line records, side-by-side:

1. What the AI-author **believed** would happen at this point in the code.
2. What the program **actually** produced at this point (the observable state).

During later analysis, the AI-agent simply compares the two columns — "what I
believed" vs. "ground truth" — and issues a correction. [author-claim]
[empirical]

The author notes explicitly that this concept "is not always clear to humans,
but GPT understands it perfectly" (post 3693). [author-claim]

## Why

LLMs overfit to strict assertion-style tests and are poor at the
meta-reasoning required to design equality checks for fuzzy business logic —
which is why GRACE leans on Log-Driven Development (LDD) rather than strict
`assert` equality as the primary truth signal (see `log-driven-development.md`).
[empirical] But for LDD to work, the log must contain enough *semantic* signal
for the AI to make a judgment. Two things bootstrap that signal:

- **Log-to-code correlation tokens** — function-ID and block-ID tokens wired
  into every line, so the log and the code merge into one semantic monolith
  for GPT (post 3693; see `log-to-code-correlation.md`). [author-claim]
- **Belief-state entries** — the code's own verbalized hypothesis about what
  it expected, co-located in the log with the observed value (post 3693).
  [author-claim]

Without the belief column, the AI reading the log must infer the intended
behavior from the code on every pass — the trace is *just* observations.
With belief entries, the contrast between hypothesis and ground truth is a
direct token-level signal: the disagreement itself tells the analysis agent
what is broken, and corrections become "easy" (post 3693). [author-claim]

This is the operational counterpart of the contract system: a function's
contract records architectural intent statically (see
`../03-contracts/contracts-overview.md`), while belief-state log lines record
execution-time intent dynamically. Together they pin "what the AI meant"
at both ends.

## How

### The core mechanic

When the AI generates a function or block, it is prompted — as part of GRACE's
logging discipline — to emit log lines that pair belief with observation
(post 3693). [author-claim] The author gives a minimum-viable example in
post 3693: log entries that record the hypothesis ("I expect X to hold here
because of Y") alongside the actual runtime value that was produced. The
sources do not specify a formal field schema for belief entries beyond this.

### Why "belief" and not "assertion"

A belief entry is **not** an `assert`. If it were an assertion, the
divergence would trigger a crash or a `TEST FAIL` and the classical-TDD
failure modes would return — overfitting to the assertion text, rigid
equality requirements, semantic noise (see
`../06-testing/classical-tests-antipattern.md`). [author-claim] Belief-state
lines are observational artifacts: the code keeps running, the log
accumulates the divergence, and the AI-agent reads the whole trajectory
during analysis.

The author states: "logs with AI are best done from the point of view of
belief state — when the AI writes code, it verbalizes its state of belief,
its hypothesis about how the code works. This concept is not always clear to
humans, but GPT understands it perfectly" (post 3693). [author-claim]

### The analysis step

At analysis time, the AI-agent (or a debugging subagent) reads the log with
its embedded log-to-code correlation tokens (post 3693). [author-claim]
Because each line carries both the function/block identifier *and* the
belief/observation pair, the agent can:

1. Locate the exact source block the line came from (via the correlation
   tokens — see `log-to-code-correlation.md`).
2. See the author-AI's prior belief at that block.
3. See the runtime ground truth at that block.
4. Compute the diff and emit a code patch.

The author reports that, given these two layers (log-to-code correlation +
belief state), "corrections are easy" (post 3693). [author-claim]

### What the sources show, minimally

The concrete technical specification for belief-state logs in the source
material is terse. Post 3693 is the primary source and states:

- Belief-state logging is one of several heuristics the author uses.
- It pairs the AI's hypothesis with the runtime state in the log line.
- GPT understands the concept "perfectly"; humans sometimes do not.

Further fields, exact formatting, line templates, or code samples of
belief-state lines are **not specified in the sources**.

### Relation to the Vlad contracts case (post 3693)

Post 3693 is the case study that motivated the author's explicit write-up
of belief-state logging. Colleague Vlad had built a project with **282
contracts and 1049 autotests** using GRACE's contract discipline, but
without GRACE-style logging (post 3693). [empirical] His agent reflected back
to him that "tests without logs is like vodka without beer" — the AI has no
other "vision" into running behavior than its logs, so even a strong contract
layer cannot compensate for blind traces. [empirical] The author uses the
case to enumerate the two minimum-viable log heuristics — **log-to-code
correlation** and **belief-state** — and states that "without them the log
may not be readable even by GPT with full understanding" (post 3693).
[author-claim]

### Placement inside the GRACE stack

Belief-state logging composes with, and depends on, other GRACE machinery:

- **Log-to-code correlation tokens** (post 3693) are prerequisite — belief
  lines that cannot be attributed back to a code block lose most of their
  analytical value. See `log-to-code-correlation.md`. [author-claim]
- **Forced context injection** (posts 3693, 3695) is the delivery mechanism
  for the most critical belief/ground-truth divergences: they get pushed
  into the agent's context rather than left for the agent to fetch. See
  `forced-context-injection.md`. [author-claim]
- **Self-correction loop** in testing (section 06): belief-state divergences
  are one of the signal types the loop consumes. See
  `../06-testing/tests-in-self-correction-loop.md`. [author-claim]
- **Anti-loop protection** (post 3695): once a failed fix is observed,
  belief-state traces from earlier iterations should be among the context
  items force-injected to prevent repeated wrong-direction fixes. See
  `../07-autonomous-agents/anti-loop-protection.md`. [author-claim]

### The human-intelligibility footnote

The author notes that belief-state logging "is not always clear to humans,
but GPT understands the concept perfectly" (post 3693). [author-claim] The
practical implication: do not evaluate the readability of belief-state log
lines by human log-reading conventions. The lines exist for the downstream
AI-agent reader, not the operator. A belief-state line that looks redundant
or strange to a human engineer can still be the exact token pattern the
analysis agent needs.

This is consistent with the broader GRACE stance that format choices should
follow the LLM's training patterns, not human conventions (see
`../00-foundations/core-principles.md`, principle 3).

## Evidence

- **Primary source**: post 3693 — introduces belief-state logging as one of
  two minimum-viable log heuristics (the other being log-to-code correlation),
  and states that GPT understands the concept while humans often do not.
  [author-claim]
- **Supporting context**: post 3693 also frames the heuristic inside the
  Vlad contracts-project case (282 contracts, 1049 autotests, no
  GRACE-style logs; agent self-reflection: "tests without logs is like
  vodka without beer"). [empirical]
- **Adjacent mechanic**: post 3693 documents the Gemini-attention
  observation that motivates paired heuristics — even Gemini's 1M-token
  window shows attention drop-outs past ~5000 lines of log without the
  correlation tokens. [author-claim]
- **Operational complement**: posts 3693, 3695 establish forced-context
  injection as the delivery mechanism for the most important log lines,
  including belief/ground-truth divergences. [author-claim]

The sources do not contain a paper citation for belief-state logging itself;
this heuristic is presented as the author's own GRACE-specific addition to
Log-Driven Development (post 3693). [author-claim] Do not attribute it to a
research paper.

## See also

- `../05-logging-ldd/log-driven-development.md` — the broader LDD concept
  that belief-state logging operationalizes.
- `../05-logging-ldd/log-to-code-correlation.md` — the sibling heuristic
  (function-ID and block-ID tokens in every log line); belief-state entries
  without correlation tokens are much less useful.
- `../05-logging-ldd/forced-context-injection.md` — how critical
  belief/ground-truth divergences are delivered into the agent's context.
- `../06-testing/tests-in-self-correction-loop.md` — belief-state
  divergences as one of the signal types the self-correction loop consumes.
- `../06-testing/classical-tests-antipattern.md` — why strict `assert`
  equality is avoided; belief-state logging is the informational alternative.
- `../07-autonomous-agents/anti-loop-protection.md` — belief-state traces
  from failed iterations as context the anti-loop system pushes back.
- `../03-contracts/contracts-overview.md` — contracts hold architectural
  intent statically; belief-state log lines hold execution-time intent
  dynamically.
- `../11-case-studies/vlad-crm-and-contracts.md` — the Vlad case (post 3693)
  that motivated the author's explicit write-up of this heuristic.
