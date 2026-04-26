# LDD Log Format Specification

## What

This chapter pins the **exact wire format** of a GRACE LDD log line, the **regex** that parses it, and the **edge cases** the regex must handle. Once the format is pinned, every downstream tool — log-trajectory test fixtures, log-to-context injectors, in-app AI consoles, dashboard parsers — speaks the same language.

The format is short. The whole spec fits in two lines:

```
[Module][function][BLOCK_NAME] message [optional key=value ...]
[Module][function][BLOCK_NAME] BELIEF: hypothesis ACTUAL: observed STATUS: MATCH|MISMATCH [optional key=value ...]
```

Everything else in this file is the consequences of those two lines.

## Why a locked format

Sister chapters in `05-logging-ldd/` argue *why* logs need function and block tokens at all (`log-to-code-correlation.md`), *why* the model must verbalise its hypothesis next to the observed state (`belief-state-logging.md`), and *why* asserts step aside in favor of trajectory analysis (`log-driven-development.md`). They establish that something like this format is required.

This chapter answers the next question: *which exact spelling?* Without a locked spelling, three tools that should agree do not:

1. The log-trajectory test fixture (`assert_trajectory(...)`) — has to match logs by regex.
2. The forced-context injector — has to filter important lines from a flood.
3. The cross-checkpoint diff (`grace-refresh` and equivalents) — has to know which lines pin a state-machine transition.

Three tools, one regex. Pin once, reuse.

## The format — line shapes

Every GRACE LDD log line has the same three-bracket prefix:

```
[Module][function][BLOCK_NAME]
```

After the prefix, one of two body shapes:

### Shape A — block marker

A non-belief line at a notable point in execution (transaction begin, external-call boundary, validation pass, etc.):

```
[Module][function][BLOCK_NAME] message text [extra_field=value other_field=value ...]
```

- `message text` is free-form, short (one phrase, no newlines).
- `extra_field=value` pairs are optional and append-only. Tooling that does not understand a field MUST ignore it.
- All fields after the prefix are space-separated. Spaces inside a value are not allowed; if you need a multi-word value, use `_` or `-`.

### Shape B — belief line

A line at a state-machine transition point that compares a hypothesis to the observed state:

```
[Module][function][BLOCK_NAME] BELIEF: hypothesis ACTUAL: observed STATUS: MATCH|MISMATCH [extra_field=value ...]
```

- The literal tokens `BELIEF:`, `ACTUAL:`, `STATUS:` (with the colon, in this order) carry semantic weight. They are the parser's match anchors.
- `hypothesis` and `observed` are single space-free tokens. Stringify enums (`OrderStatus.PAID` or `PAID` — pick one project-wide and stick with it). Do not use multi-word phrases here.
- `STATUS` MUST be exactly `MATCH` or `MISMATCH`. No `OK`, no `FAIL`, no project variants — those break log-grep aggregation across projects.

## The bracket prefix in detail

The three brackets carry one fixed contract:

| Bracket | Identifier | Constraint |
|---|---|---|
| 1 | Module label | One word, `PascalCase` or `kebab-case`. Must match the `<log-prefix>` declared in `development-plan.xml` for that module. |
| 2 | Function or step label | Free; common forms are `dotted.path` (`orders.create`), `Class.method` (`OTPService.verify`), or a step name (`webhook.dispatch`). |
| 3 | BLOCK_NAME | `UPPER_SNAKE`. Must appear in the module's `<critical-block>` declaration in `development-plan.xml` (see `../04-markup-system/substrate-xml-schemas.md`). |

The Cartesian product of (module, function, block) is the addressable space of the LDD substrate. Verification-plan markers, in-source `START_BLOCK_*` tags, and runtime log lines all index into the same space.

**Brackets cannot contain literal `[` or `]`.** This is not a Python escaping issue; it is a parser-simplicity choice. The parser regex below relies on that constraint and gets ~30% smaller because of it.

## The reference regex

Two regexes — one for each line shape:

```python
import re

_BLOCK_RE = re.compile(
    r"\[(?P<module>[^\[\]]+)\]\[(?P<fn>[^\[\]]+)\]\[(?P<blk>[^\[\]]+)\]"
)

_BELIEF_RE = re.compile(
    r"\[(?P<module>[^\[\]]+)\]\[(?P<fn>[^\[\]]+)\]\[(?P<blk>[^\[\]]+)\]"
    r"\s+BELIEF:\s+(?P<belief>\S+)\s+ACTUAL:\s+(?P<actual>\S+)\s+STATUS:\s+(?P<status>\w+)"
)
```

In one line each, in PCRE / RE2 / JavaScript-compatible form:

```
BLOCK:  \[(?P<module>[^\[\]]+)\]\[(?P<fn>[^\[\]]+)\]\[(?P<blk>[^\[\]]+)\]
BELIEF: \[(?P<module>[^\[\]]+)\]\[(?P<fn>[^\[\]]+)\]\[(?P<blk>[^\[\]]+)\]\s+BELIEF:\s+(?P<belief>\S+)\s+ACTUAL:\s+(?P<actual>\S+)\s+STATUS:\s+(?P<status>\w+)
```

Properties of this regex set, by design:

- **No backtracking pathology.** Negated character classes `[^\[\]]+` and `\S+` are linear under PCRE2 / RE2 / Go regexp / JS RegExp. Safe to apply line-by-line to ten-million-line logs.
- **No language-specific syntax.** Named-capture groups `(?P<...>)` work in Python and PCRE2; JavaScript uses `(?<...>)` (drop the `P`). Other than that one substitution, the regex is portable verbatim.
- **`_BLOCK_RE` is a prefix of `_BELIEF_RE`.** A belief line ALSO matches `_BLOCK_RE`. This is intentional: the same `.search()` over a belief line returns the bracket prefix, and only the more specific `_BELIEF_RE` parses the body. Tools can do a cheap prefix scan first and a structured parse on hits only.
- **Trailing `key=value` fields are not captured by the reference regex.** They are append-only metadata; if your project standardizes a field (e.g., `request_id`), add a project-local regex that runs after the reference one. Do not change the reference regex.

## Reference parser — Python

Drop-in helper that converts a belief line into a structured record:

```python
from dataclasses import dataclass
from typing import Optional
import re

_BELIEF_RE = re.compile(
    r"\[(?P<module>[^\[\]]+)\]\[(?P<fn>[^\[\]]+)\]\[(?P<blk>[^\[\]]+)\]"
    r"\s+BELIEF:\s+(?P<belief>\S+)\s+ACTUAL:\s+(?P<actual>\S+)\s+STATUS:\s+(?P<status>\w+)"
)


@dataclass
class BeliefLine:
    module: str
    fn: str
    blk: str
    belief: str
    actual: str
    status: str  # "MATCH" | "MISMATCH"


def parse_belief(line: str) -> Optional[BeliefLine]:
    m = _BELIEF_RE.search(line)
    if m is None:
        return None
    return BeliefLine(**m.groupdict())
```

`parse_belief` returns `None` for any line that is not a belief line, including pure block-marker lines. Tests that need only the prefix should run a separate `_BLOCK_RE.search(line)`.

## Reference parser — TypeScript

```typescript
const BELIEF_RE =
  /\[(?<module>[^\[\]]+)\]\[(?<fn>[^\[\]]+)\]\[(?<blk>[^\[\]]+)\]\s+BELIEF:\s+(?<belief>\S+)\s+ACTUAL:\s+(?<actual>\S+)\s+STATUS:\s+(?<status>\w+)/;

export interface BeliefLine {
  module: string;
  fn: string;
  blk: string;
  belief: string;
  actual: string;
  status: 'MATCH' | 'MISMATCH';
}

export function parseBelief(line: string): BeliefLine | null {
  const m = line.match(BELIEF_RE);
  if (!m || !m.groups) return null;
  return m.groups as unknown as BeliefLine;
}
```

The only delta from the Python version is the named-capture syntax (`(?<...>)` instead of `(?P<...>)`) and the case style for the status field.

## Edge cases the format must handle

These come up enough in practice to call out:

1. **Multi-line messages.** The prefix is one line. If a tool emits multi-line content (a stack trace, a SQL statement), it MUST appear on lines that DO NOT start with the bracket prefix. Parsers MUST NOT join continuation lines into the previous record automatically. If you need a multi-line block, emit one bracketed line per logical event.
2. **Trailing punctuation.** Do not put a period or exclamation mark after `STATUS: MATCH`. A trailing punctuation character changes `\w+` capture into `MATCH.` which breaks aggregation. The status value is the last load-bearing token on the line; metadata `key=value` pairs go AFTER it.
3. **Numeric BELIEF/ACTUAL.** Numbers are valid. `BELIEF: 100 ACTUAL: 100 STATUS: MATCH` parses. So does `BELIEF: 100.50 ACTUAL: 99.99 STATUS: MISMATCH`. The `\S+` token captures up to the next whitespace.
4. **Enum values with dots.** `BELIEF: OrderStatus.PAID ACTUAL: OrderStatus.PAID STATUS: MATCH` parses. Pick a project convention (with-prefix vs. without-prefix) and use it everywhere — mixing degrades string equality on the comparison side.
5. **Empty body.** A bare prefix line `[Module][function][BLOCK_NAME]` (no message at all) is valid. `_BLOCK_RE` matches it. Some block markers have no useful body — a transaction-begin marker is mostly the prefix.
6. **PII redaction (`INV-013` and equivalents).** Phone numbers, OTP codes, JWT contents, full PANs, raw webhook bodies, and personally-identifying free-text MUST NEVER appear after the prefix. The format does not enforce this — the contract for the emitter does. See `../05-logging-ldd/log-driven-development.md` and the `<redaction>` element in `technology.xml` (see `../04-markup-system/substrate-xml-schemas.md`).
7. **Process / thread / correlation IDs.** Add them as trailing `key=value` pairs (`request_id=abc-123 actor_id=u-7`). DO NOT shove them into the bracket prefix. A fourth bracket would break every parser written against this spec.
8. **Color codes / ANSI sequences.** Strip them at the handler boundary. The reference regex does not anticipate ANSI, and stripping at emit time is universally cheaper than parsing-time normalization.

## How the format relates to in-source markup

The runtime line shape mirrors the in-source markup shape, deliberately:

| In source | At runtime | Same string? |
|---|---|---|
| `# START_BLOCK_TX_BEGIN` | `[Module][function][BLOCK_TX_BEGIN]` | The `BLOCK_TX_BEGIN` token is byte-identical |
| `# END_BLOCK_TX_BEGIN` | (no runtime emission) | Pure source-side closure marker |
| `<critical-block>BLOCK_TX_BEGIN</critical-block>` (development-plan.xml) | `[Module][function][BLOCK_TX_BEGIN]` | Cross-checked by `grace-refresh` |

Result: a single grep across `(*.py | *.ts | *.xml | logs/)` for `BLOCK_TX_BEGIN` returns the source lines that emit the marker, the substrate declaration that allows it, and the runtime evidence that it fired. This is the value of pinning one spelling.

## Round-trip integrity rules

These four rules turn the format into a contract between code and runtime:

1. **Every BLOCK name in source `START_BLOCK_*` markup MUST appear in the module's `<critical-block>` list in `development-plan.xml`.**
2. **Every BLOCK name in `verification-plan.xml` `<required-log-markers>` MUST be emitted somewhere in source under that module.**
3. **Every belief line MUST sit at a transition that the canonical product spec lists.** `STATUS: MISMATCH` on a transition not in the spec is a bug in the spec OR the code; pick one and fix it (see `INV-016` in the worked example below).
4. **No `[` or `]` literal in any of the three bracket fields.** Validators MUST flag this on emit, not on parse.

A ~50-LOC validator can check rules 1, 2, and 4 with no XML library beyond stdlib. Rule 3 is a runtime aggregation across captured logs; it lives in the verification harness, not the validator.

## Worked example — aura_coffee

A Python/FastAPI + React project pinned this exact format at GRACE adoption. The reference parser regex shipped above is verbatim from that project's `packages/shared/src/shared/grace/testing.py`. Eight test cases covered:

- Block-marker emission shape.
- Belief MATCH and MISMATCH paths.
- `assert_trajectory(*expected)` happy path: ordered match against captured lines.
- `assert_trajectory(*expected)` failure path: missing-marker assertion message.
- Standalone `parse_belief` for both belief lines and non-belief lines.
- Custom-base-logger constructor.

The eight tests run in 0.01s on stdlib `logging` with no test runner dependency at import time. The same regex is used by:

- The pytest fixture (`grace_logs`) that ships with the project.
- The trajectory-assertion helper (`GraceLogCapture.assert_trajectory`).
- An in-app log filter that pulls "interesting" markers into operator dashboards.
- A CI step that scans the previous run's logs for unexpected `STATUS: MISMATCH`.

Three consumers, one regex.

## See also

- `./log-driven-development.md` — why LDD exists at all; the philosophical case against assert-only testing.
- `./log-to-code-correlation.md` — why the function and block tokens in the prefix are non-optional.
- `./belief-state-logging.md` — why every state-machine transition emits the belief shape, not just the block marker.
- `./forced-context-injection.md` — how matched markers get pulled into agent context.
- `../04-markup-system/canonical-example-python.md` — the `START_BLOCK_*` markup whose tokens this format mirrors at runtime.
- `../04-markup-system/substrate-xml-schemas.md` — `<critical-block>` and `<required-log-markers>` declarations that tie this format to the substrate.
- `../06-testing/agent-based-testing.md` — `GraceLogCapture` reference fixture (forthcoming sibling chapter).
- `../09-tooling/cli-not-custom-mcp.md` — why the regex stays portable enough to use from a bash one-liner instead of a custom MCP.
