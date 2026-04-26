# Reference `GraceLogCapture` Fixture

## What

A drop-in test fixture that captures GRACE LDD log lines during a single test run, exposes them as structured records, and supports ordered-trajectory assertions on top.

This chapter ships:

- A complete reference Python implementation (~150 LOC; tested with 8 cases).
- A pytest wiring example.
- Notes for porting to other test runners (vitest, jest, Go's `testing`).
- Adoption guidance — when to use the fixture, when not to.

The fixture is the bridge between the LDD log format pinned in `../05-logging-ldd/log-format-spec.md` and the tests-in-self-correction-loop pattern in `./tests-in-self-correction-loop.md`. Without it, every test that wants to assert on log trajectory has to reinvent the same handler-attach + parse + filter machinery.

## Why a reference implementation matters

Three forces converge:

1. **The log format is pinned, but the consumer is not.** A regex spec gives every team enough to write their own capture helper. Eight different teams will write eight slightly-incompatible helpers. The point of GRACE is that tooling generalizes.
2. **Test isolation is fiddly.** Attaching a `logging.Handler` for one test, removing it after, and not bleeding records into the next test is a common bug source. Pinning the lifecycle in one fixture removes a class of flake.
3. **Trajectory assertions need careful semantics.** "Did these markers appear in this order?" is not the same as "did all these markers appear?" — and the difference matters for state-machine verification (a transaction-commit must follow a state-transition, not precede it). Pinning `assert_trajectory` defines the ordering semantics once.

## Reference implementation — Python

The full module. Pure stdlib (`logging`, `re`, `dataclasses`); no test runner dependency at import time, so the same module is reusable in production scripts, CI tools, and notebooks — not only tests.

```python
"""GRACE LDD test capture helper.

Captures log records emitted via the GRACE logger into a structured
object that tests can grep, filter, and assert against.
"""
from __future__ import annotations

import logging
import re
from dataclasses import dataclass
from typing import Iterable, Optional


_BELIEF_RE = re.compile(
    r"\[(?P<module>[^\[\]]+)\]\[(?P<fn>[^\[\]]+)\]\[(?P<blk>[^\[\]]+)\]"
    r"\s+BELIEF:\s+(?P<belief>\S+)\s+ACTUAL:\s+(?P<actual>\S+)\s+STATUS:\s+(?P<status>\w+)"
)

_BLOCK_RE = re.compile(
    r"\[(?P<module>[^\[\]]+)\]\[(?P<fn>[^\[\]]+)\]\[(?P<blk>[^\[\]]+)\]"
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


class GraceLogCapture:
    def __init__(self) -> None:
        self._records: list[logging.LogRecord] = []
        self._handler: Optional[logging.Handler] = None
        self._target: Optional[logging.Logger] = None
        self._old_level: int = logging.NOTSET

    def install(self, target: Optional[logging.Logger] = None) -> "GraceLogCapture":
        self._target = target if target is not None else logging.getLogger()
        self._old_level = self._target.level
        records = self._records

        class _Handler(logging.Handler):
            def emit(self, record: logging.LogRecord) -> None:
                records.append(record)

        self._handler = _Handler(level=logging.INFO)
        self._target.addHandler(self._handler)
        if self._target.level == logging.NOTSET or self._target.level > logging.INFO:
            self._target.setLevel(logging.INFO)
        return self

    def uninstall(self) -> None:
        if self._handler is not None and self._target is not None:
            self._target.removeHandler(self._handler)
            if self._old_level != logging.NOTSET:
                self._target.setLevel(self._old_level)
        self._handler = None
        self._target = None

    def __enter__(self) -> "GraceLogCapture":
        return self.install()

    def __exit__(self, exc_type, exc_val, exc_tb) -> None:
        self.uninstall()

    @property
    def lines(self) -> list[str]:
        return [r.getMessage() for r in self._records]

    def blocks(
        self,
        fn: Optional[str] = None,
        blk: Optional[str] = None,
        module: Optional[str] = None,
    ) -> list[str]:
        out: list[str] = []
        for line in self.lines:
            m = _BLOCK_RE.search(line)
            if m is None:
                continue
            if module is not None and m.group("module") != module:
                continue
            if fn is not None and m.group("fn") != fn:
                continue
            if blk is not None and m.group("blk") != blk:
                continue
            out.append(line)
        return out

    def beliefs(
        self,
        fn: Optional[str] = None,
        blk: Optional[str] = None,
        module: Optional[str] = None,
        status: Optional[str] = None,
    ) -> list[BeliefLine]:
        out: list[BeliefLine] = []
        for line in self.lines:
            parsed = parse_belief(line)
            if parsed is None:
                continue
            if module is not None and parsed.module != module:
                continue
            if fn is not None and parsed.fn != fn:
                continue
            if blk is not None and parsed.blk != blk:
                continue
            if status is not None and parsed.status != status:
                continue
            out.append(parsed)
        return out

    def assert_trajectory(self, *expected: tuple[str, str]) -> None:
        idx = 0
        captured = self.lines
        for line in captured:
            if idx == len(expected):
                break
            fn, blk = expected[idx]
            pat = re.compile(rf"\[[^\[\]]+\]\[{re.escape(fn)}\]\[{re.escape(blk)}\]")
            if pat.search(line):
                idx += 1
        if idx < len(expected):
            missing = list(expected[idx:])
            joined = "\n  ".join(captured) if captured else "(no GRACE log lines captured)"
            raise AssertionError(
                "GRACE LDD trajectory missing markers (in order):\n  expected next: "
                f"{missing[0]}\n  remaining: {missing[1:]}\n"
                f"Captured lines:\n  {joined}"
            )
```

### Lifecycle

- `install(target=None)` — attach a handler to `target` (defaults to the root logger so any module-bound logger emits into it). Raises the target's level to `INFO` if it is currently higher, so info-level emissions are not silently dropped.
- `uninstall()` — detach the handler and restore the original level. Idempotent: safe to call twice.
- Context manager — `with GraceLogCapture() as cap: ...` installs on enter, uninstalls on exit. This is the recommended shape outside test-fixture wiring.

### Query API

- `cap.lines` — every captured message body, in order.
- `cap.blocks(fn=..., blk=..., module=...)` — list of full message strings whose bracket prefix matches the filters. Filters are AND-combined.
- `cap.beliefs(fn=..., blk=..., module=..., status=...)` — list of `BeliefLine` records that match the filters. Status filter is exact (`"MATCH"` or `"MISMATCH"`).
- `cap.assert_trajectory((fn, blk), (fn, blk), ...)` — assert that the given `(function, block)` pairs appear in the captured lines in this order. Allows other markers in between. Raises `AssertionError` with a useful diff if a pair is missing.

### Trajectory assertion semantics

`assert_trajectory` walks the captured lines once, advancing through the expected sequence. A line matches the next expected pair when its bracket prefix contains both `[<fn>]` and `[<blk>]`. The module bracket is intentionally NOT matched — most state-machine flows cross modules (a request enters `CoreApi`, a worker writes from `PaymentWorker`), and pinning the module would force the test to track which module emitted what.

This is **not** a strict-equality assertion. It is a subsequence assertion: missing markers fail; extra markers between expected ones do not. That semantic matches how state machines actually work — a transition fires, then arbitrary unrelated events may happen, then the next transition fires. Forcing strict equality would make the test brittle against unrelated code paths emitting their own markers in the same window.

## Pytest wiring

Drop this in `conftest.py` at any test root:

```python
from typing import Iterator
import pytest

from <your_pkg>.grace.testing import GraceLogCapture


@pytest.fixture
def grace_logs() -> Iterator[GraceLogCapture]:
    """Capture GRACE LDD log lines emitted during a test.

    Usage:
        def test_something(grace_logs):
            ... run code under test ...
            grace_logs.assert_trajectory(
                ("orders.create", "BLOCK_TX_BEGIN"),
                ("orders.create", "BLOCK_STATE_TRANSITION"),
                ("orders.create", "BLOCK_TX_COMMIT"),
            )
            assert grace_logs.beliefs(status="MISMATCH") == []
    """
    capture = GraceLogCapture()
    capture.install()
    try:
        yield capture
    finally:
        capture.uninstall()
```

Tests that opt into the fixture should carry a `# GRACE-LDD` header comment on the file. The header is a navigation hook for reviewers: a file marked `GRACE-LDD` is verified by log-trajectory assertions, not (only) by classical equality. Reviewers who see only the header know to read the fixture's filtered queries instead of looking for `assert ==` lines as the primary signal.

For a multi-package monorepo, add the fixture to each package's `conftest.py` (it is a local import; cross-package fixture wiring is fragile).

## Other test runners

Most test runners can host the same shape with a small wrapper.

### vitest / jest (TypeScript)

The same regex (with `(?<...>)` instead of `(?P<...>)`) applies. Build a class with `.lines`, `.beliefs(...)`, `.assert_trajectory(...)`, and use `beforeEach` / `afterEach` to install / uninstall a console transport (or a winston/pino transport) that pushes lines into the capture.

```typescript
let capture: GraceLogCapture;
beforeEach(() => {
  capture = new GraceLogCapture();
  capture.install();
});
afterEach(() => capture.uninstall());

it('emits the order-creation trajectory', () => {
  // ...act...
  capture.assertTrajectory(
    ['orders.create', 'BLOCK_TX_BEGIN'],
    ['orders.create', 'BLOCK_STATE_TRANSITION'],
    ['orders.create', 'BLOCK_TX_COMMIT'],
  );
});
```

### Go `testing`

Capture with a custom `slog.Handler` that appends to a slice. The trajectory check is a small loop; `regexp` from the stdlib has no named-capture limitation that matters for this format.

### Standalone CLI

The class is usable outside test runners. A CI step that scans the previous run's logs for unexpected `STATUS: MISMATCH` is one `parse_belief` loop on the file.

## Smoke-test suite

Eight tests cover the surface. Use them as both regression coverage and as worked examples for new contributors:

| Test | What it asserts |
|---|---|
| `test_block_marker_format` | `.block(fn, blk, msg, k=v)` emits the canonical prefix and includes the `key=value` field. |
| `test_belief_match` | `.belief(..., belief="X", actual="X")` produces a record with `status="MATCH"`. |
| `test_belief_mismatch` | `.belief(..., belief="X", actual="Y")` produces a record with `status="MISMATCH"`. |
| `test_assert_trajectory_in_order` | Three sequential blocks pass `assert_trajectory` in their original order. |
| `test_assert_trajectory_missing_marker` | `assert_trajectory` with a never-emitted marker raises `AssertionError` whose message names the missing marker. |
| `test_parse_belief_standalone` | `parse_belief` returns a populated `BeliefLine` for a real belief line, including trailing `key=value` metadata. |
| `test_parse_belief_returns_none_for_non_belief_lines` | `parse_belief` returns `None` for pure block lines and unrelated log text. |
| `test_module_label_from_logger_name` | `GraceLogger("Label", base=existing_logger)` does not override the underlying logger. |

Total runtime on stdlib `logging`: ~10 ms for the suite.

## When to use the fixture

**Use it when:**

- The test exercises a state-machine transition, and the value of the transition is the side effect (DB writes, external calls), not a return value.
- The flow crosses module boundaries (HTTP handler → service → worker), and a single returned value cannot capture the full path.
- The test is verifying invariants like atomicity (`INV-004`-style — full rollback on failure should mean no `BLOCK_TX_COMMIT` after a `BLOCK_STATE_TRANSITION` that triggered a failure).

**Do not use it when:**

- The test is a pure function check (input/output equality).
- The test is a snapshot/serialization round-trip.
- The system under test does not actually emit GRACE markers in the path being tested. Adding markers just to pass an LDD test is a fixture-driven antipattern; emit markers because the state-machine transition matters, not because the test wants something to grep.

## Common pitfalls

1. **Race on the root logger.** If your test runner is parallelized at the process level (pytest-xdist, vitest's worker pool), each worker has its own root logger and the fixture is safe per-worker. If parallelized at the thread level inside one process, the fixture's handler will catch records from sibling tests. Single-thread the test or scope the capture to a per-test logger via `install(target=...)`.
2. **Logger filters that drop INFO.** The fixture raises level to INFO, but a downstream filter on a parent logger can still drop the record. If `cap.lines` is empty when you expect content, check `logging.getLogger().handlers` for unexpected filters.
3. **Async code finishes after the fixture exits.** If a Celery task or a background coroutine emits markers after the test returns, those markers are lost. Await/join the work explicitly inside the test.
4. **Module-level imports that emit on first call.** A logger initialized at import time can emit before the fixture installs. Either lazy-init the logger or import the module at fixture scope.

## Worked example — aura_coffee

The Python/FastAPI + React project shipped this exact module at adoption. End state:

- `packages/shared/src/shared/grace/testing.py` — the module above, ~150 LOC.
- `packages/shared/tests/conftest.py` — exposes the `grace_logs` fixture for shared-package tests.
- `services/<service>/tests/conftest.py` (per service) — appends an identical `grace_logs` fixture.
- `packages/shared/tests/test_grace_logging.py` — the eight smoke tests, file tagged with `# GRACE-LDD`.

Total install: 3 files plus the fixture-wiring snippets. 11/11 tests pass in ~10 ms (8 LDD tests + 3 pre-existing enum tests in the same package).

## See also

- `../05-logging-ldd/log-format-spec.md` — the regex pinned in this fixture, in standalone form.
- `../05-logging-ldd/log-driven-development.md` — the philosophical case for trajectory-over-equality.
- `./agent-based-testing.md` — how this fixture interacts with the dev-agent / test-agent pattern.
- `./tests-in-self-correction-loop.md` — where log-trajectory assertions plug into the read-log → patch-code loop.
- `./classical-tests-antipattern.md` — what the fixture is trying NOT to be.
- `./in-app-ai-console.md` — heavier sibling pattern when fixture-style tests are insufficient.
- `../04-markup-system/canonical-example-python.md` — the in-source `START_BLOCK_*` markup whose tokens this fixture greps for.
- `../04-markup-system/substrate-xml-schemas.md` — `<required-log-markers>` is the substrate-side counterpart to this fixture's runtime check.
