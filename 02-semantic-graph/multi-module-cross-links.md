# Multi-Module CrossLinks — Monorepo and Multi-Service Graphs

## What

`<CrossLink>` is the edge element of `knowledge-graph.xml` (see `../04-markup-system/substrate-xml-schemas.md`). One project's graph file might have ten cross-links; another's might have a hundred. The element itself is simple — `from`, `to`, `relation` — but choosing **which edges to draw** in a multi-module project is where most graph rot starts.

This chapter covers:

- The five edge-typologies that recur in monorepos and multi-service projects.
- Layering rules that prevent cycle-shaped graphs.
- The "hub module" pattern and how to keep its incoming edges tractable.
- Edges to draw and edges to NOT draw (external systems, codegen artifacts, transitive dependencies).
- A worked example with a 7-module / 10-edge graph.

Sister chapters: `./graph-as-backbone.md` argues *why* the graph exists at all; `./hierarchy-and-anchors.md` argues *why hierarchical*. This chapter is the practical edge-drawing guide.

## Why drawing edges deliberately matters

A graph file with one missing edge looks complete; the agent reading it does not know what it does not know. Three concrete failure modes when edges are wrong:

1. **Phantom dependencies the agent invents.** If `M-A` actually calls `M-B` but no `<CrossLink from="M-A" to="M-B">` exists, the agent reading `M-A` does not load `M-B`'s contract. It then *guesses* the contract from `M-A`'s call sites — and guesses wrong on field names, error shapes, and return types.
2. **False dependencies the agent respects.** If a stale `<CrossLink>` points at a removed module, the agent dutifully tries to load context that does not exist, fails, and may hallucinate a new module to fill the gap.
3. **Hub modules pretending to be peripheral.** A module that is the entry point for five other modules but lists only three incoming edges presents the wrong shape to the planner. The planner under-allocates context budget for the hub and over-allocates for actually-peripheral modules.

The fix is the same in all three cases: edges must be drawn from the *actual* runtime dependency surface, not from how the architecture was originally sketched.

## Five edge typologies

Almost every CrossLink in a real monorepo falls into one of these five buckets. Pick the right one before writing the `relation` text.

### 1. Code-import edges

The most common. `M-A` imports `M-B`'s public API at compile time.

```xml
<CrossLink from="M-CORE-API" to="M-SHARED" relation="imports domain enums and pydantic helpers" />
```

- One edge per imported module, regardless of how many symbols. A single edge is enough for the graph to know the dependency.
- `relation` names the *category* of imports (enums, helpers, types), not every imported symbol. The function-level detail lives in module contracts; the graph stays compressed.
- If `M-A` imports both runtime symbols AND types from `M-B`, one edge covers both. Splitting is over-fitting.

### 2. HTTP/RPC client edges

Frontend → backend, service → service over the network.

```xml
<CrossLink from="M-WEB-CUSTOMER" to="M-CORE-API" relation="HTTP client (auto-generated from OpenAPI)" />
```

- Treat as a real edge even when the client is auto-generated. The dependency is real; the agent needs to know it.
- The `relation` should mention the protocol (HTTP, gRPC, GraphQL) and whether it is hand-written or codegen.
- Do NOT also draw an edge for the codegen artifact itself ("imports openapi-generated types"). The HTTP edge subsumes it.

### 3. Async dispatch edges

A service enqueues a task; a worker eventually processes it. The dependency is real even though no Python `import` statement exists at the dispatch site.

```xml
<CrossLink from="M-PAYMENT-WORKER" to="M-CORE-API" relation="receives webhook events forwarded by core-api" />
<CrossLink from="M-SMS-WORKER" to="M-CORE-API" relation="receives task dispatches via Celery from core-api" />
```

- Direction matters. The edge points from the *receiver* to the *dispatcher* IF the receiver depends on the dispatcher's payload schema. If the dispatcher just publishes to a topic and walks away, the edge points the other way (dispatcher → receiver) because the dispatcher needs the receiver's contract to know how to shape the payload.
- For two-way dependencies (dispatcher knows the receiver's input schema; receiver knows where the work came from), draw two edges. They are distinct facts.

### 4. Read/write edges into a shared store

Multiple modules touching the same database, the same cache, the same shared filesystem.

```xml
<CrossLink from="M-PAYMENT-WORKER" to="M-DATABASE" relation="reads and writes order and payment tables" />
```

- Draw one per writer-module → store-module pair. If three services write to the same DB, three edges.
- The `relation` text should name *which tables / namespaces*, not just "the DB." Tables are the next level of granularity the agent will look for.
- For caches, name *which keys or key-prefixes*. "Reads session state from Redis" is more useful than "uses redis."

### 5. Build / runtime / config edges

The squishy category. Examples: a seed script reads schema definitions from another module; an Alembic migrations directory depends on shared enum names being stable.

```xml
<CrossLink from="M-DATABASE" to="M-SHARED" relation="seed scripts use shared enums for valid initial values" />
```

- Use sparingly. Most build-time dependencies are not load-bearing for the agent.
- Draw the edge if a change in `to` would silently break `from`. Don't draw it if the dependency is purely informational.

## Layering rules

Once you have edges, ask: do they form a DAG?

GRACE projects layer their modules. Lower-numbered layers depend on nothing or on layers below themselves; higher layers depend on lower. The `<M-...> LAYER="N">` attribute in `development-plan.xml` (see `../04-markup-system/substrate-xml-schemas.md`) declares the layer.

A typical layer stack:

| Layer | Role | Examples |
|---|---|---|
| 0 | Pure utilities, no domain knowledge | `M-SHARED` (enums, logging, primitives) |
| 1 | Data layer | `M-DATABASE` (migrations, schema) |
| 2 | Services / domain logic | `M-CORE-API`, `M-PAYMENT-WORKER`, `M-SMS-WORKER` |
| 3 | UI / external interfaces | `M-WEB-CUSTOMER`, `M-WEB-ADMIN` |

**The rule:** edges go from higher layers to lower (or equal) layers. **No edge goes downward to a higher layer.** A `M-SHARED` (LAYER 0) cross-link pointing at `M-CORE-API` (LAYER 2) is almost always wrong — it means a utility module depends on a service, which inverts the dependency direction the rest of the graph relies on.

What if the project genuinely has a cycle? Two patterns to consider before encoding it:

1. **Extract the shared concern.** If `M-A` and `M-B` both depend on each other's types, extract a third module `M-A-B-CONTRACTS` at a lower layer and have both depend on it.
2. **Move the dependency to runtime, not import.** If `M-A` calls `M-B` at runtime via a registered handler, the edge is an async dispatch edge, not a code-import edge. The graph may already model this correctly with two opposite-direction edges.

If after both checks the cycle is real, document it. A cyclic graph is allowed; it is just a signal that the planner should treat the cycle as a single super-module for context-loading purposes.

## Hub modules

Some modules become hubs — `M-SHARED` and `M-CORE-API` in a typical mid-size project. A hub has many incoming edges and few outgoing ones.

Hub-aware practices:

1. **Tighten the hub's `<critical-block>` list in `development-plan.xml`.** Every BLOCK name listed there becomes part of the contract every other module depends on. Loose lists make hub changes ripple everywhere.
2. **Keep the hub's `<interface>` exports narrow.** A hub with 30 exports forces every consumer to scan all of them when loading context. A hub with 6 exports keeps consumer context tight.
3. **Do not let one hub depend on another hub.** `M-CORE-API` → `M-SHARED` is fine (service depends on utility). `M-SHARED` → `M-CORE-API` would be catastrophic — every consumer of `M-SHARED` would transitively depend on the API.
4. **Document the hub pattern in the hub's contract.** A `MODULE_CONTRACT` `RATIONALE` line ("this module is a hub; consumers across all services import from it; changes here have cross-service impact") helps agents reason about blast radius before editing.

## Edges to NOT draw

Three categories tempt the eye but degrade the graph.

### External systems (YuKassa, Stripe, Twilio, SMS.ru, Slack)

External systems are not modules in your graph. They are:

- **Actors** in `requirements.xml` (`<actor-PaymentProvider>...</actor-PaymentProvider>`).
- **Risks** in `requirements.xml` if their failure modes affect the system.
- **Discouraged surfaces** in `technology.xml` if the project deliberately avoids deeper integration.

Drawing a `<CrossLink from="M-PAYMENT-WORKER" to="YUKASSA">` pulls the agent toward "how do I read YUKASSA's contract?" — but YUKASSA has no contract in the graph. It is third-party prose. Keep external systems out of the edge list.

### Codegen artifacts

Auto-generated OpenAPI clients, gRPC stubs, GraphQL operation files. These are dependencies, but they are dependencies of the *codegen step*, not the source graph.

- The HTTP edge (typology 2) already carries the dependency.
- The codegen step itself can be documented in `<technology.xml>` `<Tooling>` (see `../04-markup-system/substrate-xml-schemas.md`).
- Drawing an edge for the codegen output makes the graph look fragmented and forces the agent to load files that change with every codegen run.

### Transitive dependencies

If `M-A` imports `M-B` and `M-B` imports `M-C`, you do NOT need an `M-A` → `M-C` edge. The graph walker handles transitivity. Drawing the edge clutters the file and degrades the compression that makes the graph cheap.

The exception: if `M-A` imports a *re-exported* symbol from `M-B` whose true source is `M-C`, AND if changing `M-C`'s symbol breaks `M-A` even when `M-B` re-exports correctly, then `M-A` truly depends on `M-C`. This is rare; when in doubt, do not draw the edge.

## A worked example — 7-module monorepo

A Python/FastAPI + React/TypeScript project with two backend SPAs, three Python services, one shared package, and one data layer. The `<CrossLink>` set is:

```xml
<!-- LAYER 2 services depend on LAYER 0 utilities -->
<CrossLink from="M-CORE-API"        to="M-SHARED"   relation="imports domain enums and pydantic helpers" />
<CrossLink from="M-PAYMENT-WORKER"  to="M-SHARED"   relation="imports domain enums" />
<CrossLink from="M-SMS-WORKER"      to="M-SHARED"   relation="imports domain enums" />

<!-- LAYER 2 services depend on LAYER 1 data -->
<CrossLink from="M-CORE-API"        to="M-DATABASE" relation="depends on schema and migrations applied at runtime" />
<CrossLink from="M-PAYMENT-WORKER"  to="M-DATABASE" relation="reads and writes order and payment tables" />

<!-- async dispatch edges between LAYER 2 services -->
<CrossLink from="M-PAYMENT-WORKER"  to="M-CORE-API" relation="receives webhook events forwarded by core-api" />
<CrossLink from="M-SMS-WORKER"      to="M-CORE-API" relation="receives task dispatches via Celery from core-api" />

<!-- LAYER 3 UIs depend on LAYER 2 service -->
<CrossLink from="M-WEB-CUSTOMER"    to="M-CORE-API" relation="HTTP client (auto-generated from OpenAPI)" />
<CrossLink from="M-WEB-ADMIN"       to="M-CORE-API" relation="HTTP client (auto-generated from OpenAPI)" />

<!-- LAYER 1 build-time dependency on LAYER 0 -->
<CrossLink from="M-DATABASE"        to="M-SHARED"   relation="seed scripts use shared enums for valid initial values" />
```

10 edges across 7 modules. Properties:

- `M-SHARED` (LAYER 0) is the deepest hub: 4 incoming edges. Its `<interface>` exports are tight (enums, GRACE logger).
- `M-CORE-API` (LAYER 2) is the runtime hub: 4 incoming edges from the two workers and the two SPAs.
- Direction discipline: every edge goes from a higher LAYER to a lower or equal one. No cycles.
- One edge per pair: `M-WEB-CUSTOMER` → `M-CORE-API` does not also draw `M-WEB-CUSTOMER` → `M-SHARED` (transitive).
- External systems (a payment provider, an SMS provider) are absent from the edge list. They live in `requirements.xml` `<actor-...>` and in `<Risks>`.
- The build-time edge from `M-DATABASE` to `M-SHARED` is drawn because seed scripts genuinely break if shared enum values change. Without that edge, the agent reading `M-DATABASE` misses a real dependency.

A graph walker can answer typical agent questions in 1–2 hops:

- *"What does the customer SPA depend on?"* → `M-WEB-CUSTOMER` → `M-CORE-API` → (`M-SHARED`, `M-DATABASE`).
- *"What ripples if I change `OrderStatus`?"* → modules with edges to `M-SHARED`: 4 modules at LAYER ≥ 1.
- *"Who consumes core-api?"* → in-edges of `M-CORE-API`: payment-worker, sms-worker, web-customer, web-admin.

That is the value the graph provides. None of those answers require the agent to scan source files.

## Maintenance — keeping edges true over time

The graph rots as the code moves. Two practices keep it true:

1. **`grace-refresh` (or equivalent sync) at every checkpoint.** It reports orphan edges (referencing modules that no longer exist), missing edges (modules that import each other in code but have no `<CrossLink>`), and stale `relation` text (edges whose described dependency no longer exists in code).
2. **A CI check that walks every `<CrossLink from="A" to="B">` and confirms B is in the `<Modules>` block.** Cheap, ~30 LOC, catches dangling edges before they confuse an agent.

For more on the substrate-vs-code consistency story, see `../08-workflow-and-phases/brownfield-adoption.md` (the CP7 docs-sync section names `grace-refresh` as the discrepancy detector).

## Anti-patterns

1. **Drawing edges for every imported symbol.** One edge per module pair is enough; the contract layer carries the symbol-level detail.
2. **Drawing edges to and from a "shared types" pseudo-module that has no source.** If types live somewhere, that somewhere is a real module. Make it explicit.
3. **Edges that name behaviors the modules do not actually have.** `relation="provides authentication for"` when the destination has no authentication code is fiction. Use the runtime fact, not the design intent.
4. **Bidirectional edges drawn as one element.** `<CrossLink>` is directed. If both directions exist, draw two elements. Conflating them hides which direction is actually load-bearing.
5. **Edges that span layers in the wrong direction.** A LAYER-0 utility module never has outgoing edges to a LAYER-2 service.

## See also

- `./graph-as-backbone.md` — why the graph is non-optional.
- `./hierarchy-and-anchors.md` — the spec → contract → code hierarchy that the cross-links sit inside.
- `./graph-rag-vs-vector-rag.md` — the retrieval-side counterpart to deliberately-drawn edges.
- `../04-markup-system/substrate-xml-schemas.md` — `<CrossLink>` schema (attributes, allowed values).
- `../03-contracts/contract-fields.md` — `LINKS` field at the contract level; pairs naturally with `<CrossLink>` at the graph level.
- `../03-contracts/wenyan-prompting.md` — keep the `relation` field compressed.
- `../08-workflow-and-phases/brownfield-adoption.md` — when these edges are populated during a migration (CP1).
- `../09-tooling/cli-not-custom-mcp.md` — why a 30-LOC graph-walking script beats a custom MCP for cross-link queries.
