# Substrate XML Schemas

## What

GRACE's substrate lives in **six XML files in `docs/`** of the project being managed. Each file has a fixed root element, a fixed set of children, and a fixed naming convention for IDs. This chapter pins those tag-by-tag, so authors and tooling have a single reference instead of inferring the shape from one example project.

The six files:

| File | Root element | Purpose |
|---|---|---|
| `requirements.xml` | `<RequirementsAnalysis>` | Index into the canonical product spec; actors, use cases, invariants, risks. |
| `technology.xml` | `<TechnologyStack>` | Runtime, language, dependencies, tooling, observability, autonomy policy, delivery shape. |
| `development-plan.xml` | `<DevelopmentPlan>` | Modules with contracts and observability hooks; data flows; implementation phases. |
| `verification-plan.xml` | `<VerificationPlan>` | Critical-flow scenarios, per-module verification, phase gates, failure-packet template. |
| `knowledge-graph.xml` | `<KnowledgeGraph>` | Module nodes with annotations and `<CrossLink>` edges. |
| `operational-packets.xml` | `<OperationalPackets>` | Templates: ExecutionPacket, GraphDelta, VerificationDelta, FailurePacket, CheckpointReport. |

Sister chapter: `./xml-like-markup.md` covers *why* XML-like markup is the format. This chapter covers *what tags* go inside.

## Why a locked schema

A pattern-only spec (e.g., "use XML-like tags naming semantic roles") is enough to start, but it leaves three gaps that bite at scale:

1. **Tooling cannot generalize.** A contract-extractor or graph builder needs to know which element holds module IDs and which holds dependency lists. If those tags drift between projects, every project needs a custom parser.
2. **CrossLink validation breaks.** `<CrossLink from="M-A" to="M-B">` only validates if both projects use the same `M-*` ID convention.
3. **`grace-refresh` and equivalent sync tools cannot diff substrate against code.** The diff requires that "what counts as a module declaration" has a fixed shape.

Pinning the schema once removes all three failure modes. Teams may extend with project-specific child elements; the locked vocabulary below is the **floor**, not the ceiling.

## Naming conventions

These conventions cut across every file. Following them is the cheapest way to keep tools portable.

| ID prefix | Meaning | Example |
|---|---|---|
| `M-<NAME>` | Module identifier | `M-CORE-API`, `M-PAYMENT-WORKER` |
| `V-M-<NAME>` | Module verification block | `V-M-CORE-API` |
| `VF-<NNN>` | Critical verification flow (numbered) | `VF-001`, `VF-002` |
| `UC-<NAME>` | Use case | `UC-Order-Place` |
| `DF-<NAME>` | Data flow | `DF-Order-Place`, `DF-Payment` |
| `INV-<NNN>` | Invariant constraint | `INV-002`, `INV-013` |
| `Phase-<NAME>` | Implementation phase | `Phase-Migration`, `Phase-PostMigration` |
| `Gate-<PhaseName>` | Phase gate inside `<PhaseGates>` | `Gate-Phase-Migration` |
| `BLOCK_<UPPER_SNAKE>` | LDD log marker | `BLOCK_TX_BEGIN`, `BLOCK_STATE_TRANSITION` |

Casing rules:

- Root elements and major sections — `PascalCase` (`<DevelopmentPlan>`, `<CriticalFlows>`).
- ID-bearing elements — the ID itself becomes the tag (`<M-CORE-API>`, `<VF-001>`). This is GRACE's convention because it lets a tag-walker recognize an entity without parsing attributes.
- Plain field elements inside an entity — `kebab-case` or one-word (`<purpose>`, `<verification-ref>`, `<log-prefix>`).
- Attributes — `UPPER_SNAKE` for fixed-vocabulary fields (`STATUS`, `TYPE`, `PRIORITY`), `camelCase` or lowercase for free-form (`from`, `to`, `path`, `commit`).

Common attributes that recur across files:

| Attribute | Allowed values | Where it appears |
|---|---|---|
| `VERSION="X.Y.Z"` | semver string | Root element of every file |
| `STATUS` | `planned` \| `active` \| `done` \| `implemented` | Modules, phase steps |
| `PRIORITY` | `high` \| `medium` \| `low` | Use cases, verification flows |
| `TYPE` | `ENTRY_POINT` \| `INTEGRATION` \| `DATA_LAYER` \| `UI_COMPONENT` \| `UTILITY` | Modules |
| `LAYER` | `0`..`N` (integer) | Modules (lower = closer to foundation) |

---

## File 1 — `requirements.xml`

Root: `<RequirementsAnalysis VERSION="...">`.

Required children (in this order):

```xml
<RequirementsAnalysis VERSION="0.2.0">
  <Project>
    <name>...</name>
    <annotation>...</annotation>
    <keywords>comma, separated, list</keywords>
  </Project>

  <CanonicalReferences>
    <doc-PDD path="docs/PRODUCT_DESIGN_DOCUMENT.md" role="authoritative">
      ...one paragraph stating this XML is an INDEX, not a duplicate...
    </doc-PDD>
    <doc-AGENTS path="AGENTS.md" role="navigation">...</doc-AGENTS>
  </CanonicalReferences>

  <Actors>
    <actor-<Name>>...one-line description...</actor-<Name>>
    <!-- one element per actor; tag is `actor-` + ActorName -->
  </Actors>

  <UseCases>
    <UC-<Name> ref="<spec-anchor>">
      <Actor>comma-separated</Actor>
      <Action>...</Action>
      <Goal>...</Goal>
      <Preconditions>...</Preconditions>
      <AcceptanceCriteria>...</AcceptanceCriteria>
      <Priority>high|medium|low</Priority>
      <RelatedFlows>DF-..., DF-...</RelatedFlows>
    </UC-<Name>>
  </UseCases>

  <NonGoals>
    <item-1>...</item-1>
    <item-2>...</item-2>
  </NonGoals>

  <Constraints>
    <inv-INV-<NNN>>...one-line statement... (<spec-anchor>)</inv-INV-<NNN>>
  </Constraints>

  <Risks>
    <risk-1>...</risk-1>
  </Risks>

  <OpenQuestions>
    <question-1>...</question-1>
  </OpenQuestions>
</RequirementsAnalysis>
```

Rules:

- **Do not duplicate the canonical product spec.** If you have a Product Design Document or equivalent, this file is an index — it points back at it via `ref="<spec-anchor>"` on each `<UC-...>` and via `<CanonicalReferences>`.
- `<Constraints>` lists invariants. Each `<inv-INV-NNN>` should fit on one line and end with the spec anchor in parentheses.
- `<NonGoals>` is non-optional even when empty — agents read absence as a signal, and an empty section is more informative than a missing one.

---

## File 2 — `technology.xml`

Root: `<TechnologyStack VERSION="...">`.

Required children (order is conventional, not enforced):

```xml
<TechnologyStack VERSION="0.2.0">
  <Runtime>...one line: language runtimes + datastores...</Runtime>
  <Language>...primary language(s)...</Language>
  <Framework>...primary framework(s)...</Framework>

  <Dependencies>
    <dep name="..." version="..." purpose="..." />
  </Dependencies>

  <VersionConstraints>
    <constraint lib="..." min="..." max="..." reason="..." />
  </VersionConstraints>

  <Tooling>
    <tool name="<role>" value="<concrete>" version="..." />
  </Tooling>

  <PreferredAgentStack>
    <preferred-runtime-library>...</preferred-runtime-library>
    <preferred-test-library>...</preferred-test-library>
    <preferred-observability-library>...</preferred-observability-library>
    <discouraged-surface reason="...">...</discouraged-surface>
  </PreferredAgentStack>

  <Testing>
    <module-level><command>...</command><focus>...</focus></module-level>
    <wave-level><command>...</command><focus>...</focus></wave-level>
    <phase-level><command>...</command><focus>...</focus></phase-level>
    <mocking-policy>...</mocking-policy>
  </Testing>

  <Observability>
    <logger>...</logger>
    <log-format>[ModuleName][functionName][BLOCK_NAME] message</log-format>
    <required-fields>correlationId, actorId, eventType, status</required-fields>
    <belief-state-fields>belief, actual, status (MATCH | MISMATCH)</belief-state-fields>
    <redaction>...</redaction>
  </Observability>

  <AutonomyPolicy>
    <default-execution-profile>conservative|balanced|aggressive</default-execution-profile>
    <max-fix-attempts-per-step>2</max-fix-attempts-per-step>
    <replan-trigger>...</replan-trigger>
    <checkpoint-requirement>...</checkpoint-requirement>
  </AutonomyPolicy>

  <DeliveryShape>
    <shape>...</shape>
    <local-entry-point>...</local-entry-point>
  </DeliveryShape>
</TechnologyStack>
```

Rules:

- `<dep>` elements use **attributes**, not children, for `name`/`version`/`purpose`. This is the only place in the substrate where attributes carry semantic content; everywhere else they are metadata.
- `<discouraged-surface>` is the negative space — surfaces the project deliberately avoids. Agents read this to filter their solution space. **Always include at least one entry**, even if it is "no current discouraged surfaces."
- `<log-format>` and `<belief-state-fields>` lock the LDD format for the project. See `../05-logging-ldd/log-driven-development.md` for the canonical pin.

---

## File 3 — `development-plan.xml`

Root: `<DevelopmentPlan VERSION="...">`. The largest of the six files in any non-trivial project.

```xml
<DevelopmentPlan VERSION="0.2.0">
  <ArchitectureNotes>
    <objective>...</objective>
    <non-goal>...</non-goal>
    <risk-N>...</risk-N>
  </ArchitectureNotes>

  <Modules>
    <M-<NAME> NAME="..." TYPE="..." LAYER="N" ORDER="N" STATUS="...">
      <contract>
        <purpose>...</purpose>
        <inputs><param name="..." type="..." /></inputs>
        <outputs><param name="..." type="..." /></outputs>
        <errors><error code="..." /></errors>
      </contract>
      <interface>
        <export-<name> PURPOSE="..." />
      </interface>
      <depends>M-OTHER-A, M-OTHER-B  <!-- comma-separated module IDs, or "none" --></depends>
      <target>
        <source>relative/path/to/source/</source>
        <tests>relative/path/to/tests/</tests>
      </target>
      <observability>
        <log-prefix>[ModuleLabel]</log-prefix>
        <critical-block>BLOCK_NAME_1, BLOCK_NAME_2  <!-- optional; lists the LDD markers this module owns --></critical-block>
      </observability>
      <verification-ref>V-M-<NAME></verification-ref>
      <notes>
        <note-N>...</note-N>
      </notes>
    </M-<NAME>>
  </Modules>

  <DataFlow>
    <DF-<NAME> NAME="..." TRIGGER="...">
      <step-1>...</step-1>
      <step-2>...</step-2>
      <evidence>[Module][fn][BLOCK_NAME]; [Module][fn][BLOCK_NAME]  <!-- log markers expected on the path --></evidence>
    </DF-<NAME>>
  </DataFlow>

  <ImplementationOrder>
    <Phase-<NAME> name="..." status="planned|active|done">
      <goal>...</goal>
      <step-<ID> module="M-X,M-Y" status="..." verification="V-M-X" commit="<sha>">...</step-<ID>>
    </Phase-<NAME>>
  </ImplementationOrder>

  <ExecutionPolicy>
    <default-profile>conservative|balanced|aggressive</default-profile>
    <controller-owns>...comma-separated paths the controller agent may write...</controller-owns>
    <worker-owns>...what worker agents may write...</worker-owns>
    <max-fix-attempts-per-step>2</max-fix-attempts-per-step>
    <replan-trigger>...</replan-trigger>
    <checkpoint-requirement>...</checkpoint-requirement>
  </ExecutionPolicy>
</DevelopmentPlan>
```

Rules:

- Every `<M-...>` MUST have `<contract>`, `<interface>`, `<depends>`, `<target>`, `<observability>`, `<verification-ref>`. Optional: `<notes>`.
- `<depends>` is **comma-separated** inside the element body, not a list of `<dep>` children. This makes simple text walkers cheap.
- `<critical-block>` inside `<observability>` declares the BLOCK names this module emits. Verification-plan markers MUST be a subset of these.
- `<step-<ID>>` inside a phase: ID is `CP1`, `CP2`, `step-1`, etc. Including `commit="<sha>"` after a step lands turns the file into a project-history index agents can grep.
- `<controller-owns>` and `<worker-owns>` define the write-scope contract for autonomous runs. See `../07-autonomous-agents/`.

---

## File 4 — `verification-plan.xml`

Root: `<VerificationPlan VERSION="...">`.

```xml
<VerificationPlan VERSION="0.2.0">
  <GlobalPolicy>
    <log-format>[ModuleName][functionName][BLOCK_NAME] message</log-format>
    <belief-state>BELIEF: hypothesis ACTUAL: observed STATUS: MATCH | MISMATCH</belief-state>
    <deterministic-first>true</deterministic-first>
    <module-level-focus>...</module-level-focus>
    <wave-level-focus>...</wave-level-focus>
    <phase-level-focus>...</phase-level-focus>
    <autonomy-gate>...</autonomy-gate>
    <redaction>...</redaction>
  </GlobalPolicy>

  <CriticalFlows>
    <VF-<NNN> NAME="..." USE_CASES="UC-A,UC-B" DATA_FLOW="DF-A,DF-B" PRIORITY="...">
      <scenario>...</scenario>
      <expected-outcome>...</expected-outcome>
      <required-signals>
        <log-marker>[Module][fn][BLOCK_NAME]</log-marker>
        <trace-sequence>BLOCK_A -> BLOCK_B -> BLOCK_C</trace-sequence>
      </required-signals>
    </VF-<NNN>>
  </CriticalFlows>

  <ModuleVerification>
    <V-M-<NAME> MODULE="M-<NAME>" PRIORITY="...">
      <test-files>
        <file>path/to/test_file.py</file>
      </test-files>
      <module-checks>
        <check-N>...command...</check-N>
      </module-checks>
      <scenarios>
        <scenario-N kind="success|failure">...</scenario-N>
      </scenarios>
      <required-log-markers>
        <marker-N>[Module][*][BLOCK_NAME]</marker-N>
      </required-log-markers>
      <required-trace-assertions>
        <assertion-N>...</assertion-N>
      </required-trace-assertions>
      <wave-follow-up>...</wave-follow-up>
      <phase-follow-up>...</phase-follow-up>
    </V-M-<NAME>>
  </ModuleVerification>

  <PhaseGates>
    <Gate-<PhaseName> PHASE="<PhaseName>">
      <goal>...</goal>
      <commands>
        <command-N>...</command-N>
      </commands>
      <required-evidence>
        <evidence-N>...</evidence-N>
      </required-evidence>
    </Gate-<PhaseName>>
  </PhaseGates>

  <FailurePackets>
    <packet-template>
      <expected-fields>scenario, contract-ref, expected-evidence, observed-evidence, first-divergent-block, suggested-next-action</expected-fields>
    </packet-template>
  </FailurePackets>
</VerificationPlan>
```

Rules:

- `<VF-NNN>` IDs are zero-padded three digits (`VF-001`, `VF-002`). They survive growth past 100 flows without ID collisions.
- The `*` wildcard inside a marker (`[Module][*][BLOCK_NAME]`) means "any function in that module." Used in `<required-log-markers>` to require the BLOCK without naming a specific function.
- `<trace-sequence>` uses `->` (ASCII arrow) between BLOCK names, no spaces around the arrow optional but conventional. Not `→` (Unicode).
- Every BLOCK name in this file MUST appear in some module's `<critical-block>` in `development-plan.xml`. Cross-file consistency is the only way `grace-refresh` (or equivalent sync tooling) can flag drift.
- `<scenarios>` items declare `kind="success"` (the path must work) or `kind="failure"` (the path must fail in this specific way). This distinction matters for how a verifier reads the scenario.

---

## File 5 — `knowledge-graph.xml`

Root: `<KnowledgeGraph>` — note: NO `VERSION` attribute on the root; version goes on the inner `<Project>`. This is the only file with this asymmetry.

```xml
<KnowledgeGraph>
  <Project NAME="..." VERSION="0.2.0">
    <keywords>comma, separated</keywords>
    <annotation>...one-paragraph project summary...</annotation>

    <M-<NAME> NAME="..." TYPE="..." STATUS="...">
      <purpose>...</purpose>
      <path>relative/path/to/module</path>
      <depends>M-OTHER, M-OTHER</depends>
      <verification-ref>V-M-<NAME></verification-ref>
      <annotations>
        <export-<name> PURPOSE="..." />
      </annotations>
    </M-<NAME>>

    <CrossLink from="M-A" to="M-B" relation="..." />
  </Project>
</KnowledgeGraph>
```

Rules:

- The graph file is **denormalized** by design. Every module appears with its purpose, path, and exports — duplicating data also held in `development-plan.xml`. The denormalization lets a graph walker stay inside one file.
- `<CrossLink>` is the only edge element. It uses attributes (not children) because graph edges are dense — a 7-module project has 10–15 cross-links and an attribute form keeps each on one line.
- `relation="..."` is free-form prose. Common patterns: `"imports domain enums"`, `"HTTP client (auto-generated from OpenAPI)"`, `"reads and writes order tables"`. Keep it short and concrete.
- Cross-links between same modules in different directions ARE distinct (`<CrossLink from="A" to="B">` ≠ `<CrossLink from="B" to="A">`). Direction matters.
- See `../02-semantic-graph/` for graph-design principles and `./multi-module-cross-links.md` (forthcoming) for monorepo guidance.

---

## File 6 — `operational-packets.xml`

Root: `<OperationalPackets VERSION="...">`. Pure templates — agents copy these shapes when emitting deltas at runtime.

```xml
<OperationalPackets VERSION="0.1.0">
  <ExecutionPacketTemplate>
    <ExecutionPacket>
      <module-id>M-<NAME></module-id>
      <module-name>...</module-name>
      <purpose>...</purpose>
      <write-scope>
        <file-N>relative/path</file-N>
      </write-scope>
      <contract-excerpt><![CDATA[
...module contract excerpt cited from development-plan.xml...
      ]]></contract-excerpt>
      <graph-entry-excerpt><![CDATA[
...graph node excerpt cited from knowledge-graph.xml...
      ]]></graph-entry-excerpt>
      <dependency-contract-summaries>
        <dependency-N module="M-<NAME>">...one-line dependency summary...</dependency-N>
      </dependency-contract-summaries>
      <verification-excerpt><![CDATA[
...V-M-<NAME> excerpt from verification-plan.xml...
      ]]></verification-excerpt>
      <assumptions>
        <assumption-N>...</assumption-N>
      </assumptions>
      <stop-conditions>
        <condition-N>...</condition-N>
      </stop-conditions>
      <retry-budget>2</retry-budget>
      <checkpoint-fields>
        <field-N>assumptions-kept|commands-run|evidence-captured|next-action</field-N>
      </checkpoint-fields>
      <expected-graph-delta-fields>...</expected-graph-delta-fields>
      <expected-verification-delta-fields>...</expected-verification-delta-fields>
      <notes>
        <note-N>...</note-N>
      </notes>
    </ExecutionPacket>
  </ExecutionPacketTemplate>

  <GraphDeltaTemplate>
    <GraphDelta module="M-<NAME>">
      <imports-added><import-N>...</import-N></imports-added>
      <imports-removed />
      <exports-added><export-N>...</export-N></exports-added>
      <exports-removed />
      <annotations-added><annotation-N>...</annotation-N></annotations-added>
      <annotations-removed />
      <cross-links-added><CrossLink from="..." to="..." relation="..." /></cross-links-added>
      <cross-links-removed />
      <notes><note-N>...</note-N></notes>
    </GraphDelta>
  </GraphDeltaTemplate>

  <VerificationDeltaTemplate>
    <VerificationDelta module="M-<NAME>">
      <test-files-added><file-N>...</file-N></test-files-added>
      <test-files-removed />
      <module-checks><command-N>...</command-N></module-checks>
      <required-log-markers><marker-N>[Module][fn][BLOCK_NAME]</marker-N></required-log-markers>
      <required-trace-assertions><assertion-N>...</assertion-N></required-trace-assertions>
      <wave-follow-up><item-N>...</item-N></wave-follow-up>
      <phase-follow-up><item-N>...</item-N></phase-follow-up>
    </VerificationDelta>
  </VerificationDeltaTemplate>

  <FailurePacketTemplate>
    <FailurePacket>
      <scenario>...</scenario>
      <contract-ref>M-<NAME> / V-M-<NAME> / scenario-N</contract-ref>
      <expected-evidence>
        <marker-N>[Module][fn][BLOCK_NAME]</marker-N>
        <assertion-N>...</assertion-N>
      </expected-evidence>
      <observed-evidence>
        <log-N>...</log-N>
        <trace-N>BLOCK_X -> threw <ERROR></trace-N>
      </observed-evidence>
      <first-divergent-block>BLOCK_NAME</first-divergent-block>
      <suggested-next-action>...</suggested-next-action>
    </FailurePacket>
  </FailurePacketTemplate>

  <CheckpointReportTemplate>
    <CheckpointReport>
      <scope>M-<NAME> / step-<ID></scope>
      <assumptions-kept><item-N>...</item-N></assumptions-kept>
      <commands-run><command-N>...</command-N></commands-run>
      <evidence-captured><item-N>...</item-N></evidence-captured>
      <retry-budget-used>0</retry-budget-used>
      <next-action>...</next-action>
    </CheckpointReport>
  </CheckpointReportTemplate>
</OperationalPackets>
```

Rules:

- Templates contain `<![CDATA[...]]>` blocks where prose excerpts go; the CDATA is required because excerpts may contain unescaped `<` or `&`.
- `<expected-graph-delta-fields>` and `<expected-verification-delta-fields>` are NOT prose — they are the fixed list of children the agent will emit when producing the actual delta. Pinning them in the template prevents agents from inventing new field names.
- These templates are read by agents at runtime via `grace-execute` and equivalent skills. Keep them small (each template fits in roughly one screen) so the agent can hold the whole template in attention while filling it in.

---

## Cross-file integrity rules

These are the constraints `grace-refresh` (or any equivalent sync tool) checks. They are the value the substrate provides over a pile of separate docs.

1. **Every `<M-...>` ID in `development-plan.xml` MUST have a matching `<M-...>` in `knowledge-graph.xml` and a matching `<V-M-...>` in `verification-plan.xml`.** Three-way consistency is the floor.
2. **Every BLOCK name referenced in `verification-plan.xml` `<required-log-markers>` MUST appear in some module's `<critical-block>` in `development-plan.xml`.** Otherwise the verification asks for evidence the codebase never emits.
3. **Every `<CrossLink>` MUST connect modules that exist in `<Modules>`.** Dangling refs break graph walks.
4. **Every `INV-NNN` cited inside an `<inv-...>` body or `<note-...>` MUST be defined exactly once in `requirements.xml` `<Constraints>`.**
5. **Every `<step-<ID> commit="<sha>">` MUST point at a real commit on the migration branch.** A 7-character abbreviated SHA is enough; CI can enforce.
6. **Every `<UC-...>` MUST list at least one `<RelatedFlows>` value if the use case has any runtime side effect.** Otherwise data-flow correlation breaks.
7. **`<discouraged-surface>` in `technology.xml` MUST appear at least once.** The negative space is load-bearing for agent solution-space pruning.

A simple ~150-LOC validator can check rules 1–4 with no XML library beyond stdlib. Rule 5 needs a `git` call. Rules 6–7 are stylistic and OK to enforce as warnings.

## Worked example — aura_coffee

A Python/FastAPI + React/TypeScript project populated all six files at GRACE adoption time:

- `requirements.xml` — 1 project, 7 actors, 5 use cases (`UC-Order-Place`, `UC-Payment`, `UC-Delivery`, `UC-OTP-Login`, `UC-Admin-Manage`), 6 invariants (`INV-002` / `004` / `013` / `014` / `015` / `016`), 5 risks, 2 open questions.
- `technology.xml` — Python 3.12 / Node 22 / PostgreSQL 16 / Redis 7 stack, 14 deps, 3 version constraints, 7 tools, 2 discouraged surfaces.
- `development-plan.xml` — 7 modules (`M-SHARED`, `M-DATABASE`, `M-CORE-API`, `M-PAYMENT-WORKER`, `M-SMS-WORKER`, `M-WEB-CUSTOMER`, `M-WEB-ADMIN`), 4 data flows, 2 phases (`Phase-Migration` with 7 CP steps + `Phase-PostMigration`).
- `verification-plan.xml` — 3 critical flows (`VF-001`, `VF-002`, `VF-003`), 7 module verifications, 1 phase gate, failure-packet template.
- `knowledge-graph.xml` — 7 module nodes + 10 `<CrossLink>` edges.
- `operational-packets.xml` — 5 templates (Execution / GraphDelta / VerificationDelta / FailurePacket / CheckpointReport).

Total: 838 lines across six files. All lock above.

Numbers will differ for your project; the **shape** of the schema is what carries forward.

## See also

- `./xml-like-markup.md` — why XML-like markup is the format.
- `./start-end-tags.md` — paired-tag mechanism inside source files (related but for in-source markup, not for these XML artifacts).
- `./markup-validators.md` — algorithmic validators on top of paired markup (extends naturally to XML schemas).
- `../02-semantic-graph/graph-as-backbone.md` — why the graph file exists.
- `../02-semantic-graph/hierarchy-and-anchors.md` — module hierarchy that the graph encodes.
- `../03-contracts/contracts-overview.md` — function/module contracts inside source files; the substrate XMLs cite excerpts from these.
- `../05-logging-ldd/log-driven-development.md` — pinning BLOCK names that the verification-plan markers reference.
- `../07-autonomous-agents/` — how `<controller-owns>` / `<worker-owns>` and operational-packet templates are consumed at runtime.
- `../08-workflow-and-phases/brownfield-adoption.md` — how a CP1 substrate bootstrap populates these six files from existing project docs.
