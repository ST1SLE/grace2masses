# Skills System — Making Skills Actually Load

## What

Skills are a dynamic-prompting mechanism (most prominent in Open Code / the new Kilo Code core) where the agent is given a Knowledge-Base-like catalog of discrete "skill" prompts that it is meant to load on demand during a trajectory. GRACE was migrated entirely onto dynamic skills on top of the Open Code core `[empirical]` (post 3809).

The deceptively-simple appearance of skills hides a **specific, scientifically-noted failure mode**: the LLM can decide that it "already knows" how to do what the skill describes and simply **skip the load** `[research-backed]` (post 3809). Author's framing: "skills are a crucially dangerous form of prompting; only dilettantes drop them in without thinking" `[author-claim]` (post 3809).

This file covers **how to make a skill actually load**, which is the non-negotiable engineering problem that sits under any skills-based methodology.

## Why

The risk is structural, not cosmetic:

- **Self-substitution of the skill.** The LLM forms an internal hypothesis that the topic is familiar SFT territory and produces a plausible-looking answer **without ever invoking the skill-load tool** `[research-backed]` (post 3809). The output looks fine but bypasses the methodology-specific rules the skill was supposed to enforce.
- **Silent degradation at scale.** On autonomous/swarm trajectories this means the agent drifts from GRACE discipline to generic vibe-coding — exactly the regime GRACE exists to prevent (cross-link `../00-foundations/why-grace-exists.md`).
- **Alignment with broader GRACE principles.** This is the same family of problem as "agents ignore custom MCPs" (post 3834) and "agents silently skip external docs" (post 3820): the model prefers what its training biased it toward, and needs **forced context** to override that bias (cross-link `../07-autonomous-agents/forced-context.md`).

Because GRACE runs on dynamic skills end-to-end in the current Open Code deployment `[empirical]` (post 3809), every skill that the agent silently skips is a direct hole in the methodology at runtime.

## How

The author's three-part mitigation stack for making skills reliable `[author-claim]` (post 3809):

### 1. MANDATORY MODE / MANDATORY PROTOCOL trigger words

Inject explicit trigger phrases such as `MANDATORY MODE` or `MANDATORY PROTOCOL` in the skill description / invocation surface. These signal to the LLM that the content is **critical rules, not an optional Knowledge Base** `[author-claim]` (post 3809). Without such trigger words the LLM treats skills as suggestive reference material — easy to skip on the "I already know" hypothesis.

### 2. Forced tracing (skill → names next skill → by exact name)

This is the **main method** of reliable skill loading per the author `[author-claim]` (post 3809). Mechanics:

- Start with an explicit instruction telling the AI **which skills to read first**.
- Each loaded skill **names the next skill(s) to load — by literal name** — so the chain is deterministic and traceable.
- No "load something relevant" — only "now load `<skill-exact-name>`".

This converts dynamic skill loading from a model-discretion problem into a trace-driven load sequence. The author's claim: with this pattern **dynamic skills are no less reliable than trace-based dynamic prompts** `[author-claim]` (post 3809).

### 3. In Open Code: require the `skill` tool call explicitly

Open Code exposes a `skill` tool for loading skills. Strengthen loading by **explicitly requiring the `skill` tool call** in the skill-chain instructions `[empirical]` (post 3809, screenshot referenced but not reproducible here). This removes the ambiguity of "did the model internally consult the skill" — there is now a verifiable tool invocation the orchestrator can audit.

### Engineering heuristic

Combine all three:

- MANDATORY trigger words mark importance.
- Trace chain (by-name next-skill instruction) eliminates discretion on *which* skill.
- Explicit `skill` tool requirement eliminates discretion on *whether* a load occurred.

Anything weaker — e.g., "load the relevant skill when appropriate" — is in "only dilettantes" territory per the author `[author-claim]` (post 3809).

### What this does NOT do

The sources do not specify:

- quantitative skip-rate numbers for skills (the scientific notice is qualitative in the source);
- which specific paper the author refers to when saying the danger is "scientifically noted" — post 3809 references "the scientific article" without title or URL;
- how the trigger words are recognized inside the model (pattern / fine-tune / prompt artifact — the sources do not specify).

## Evidence

- **Primary source**: post 3809 — "Перевёл я весь GRACE на динамические скилы ядра Open Code… скилы — крайне опасный вид промптинга, поскольку LLM может решить, что «и так знает», как это делается, и просто проигнорирует загрузку скила." `[empirical]` `[author-claim]`.
- **Trigger-word mitigation**: post 3809 — "повысить вероятность вызова скила специальными словами-триггерами, такими как MANDATORY MODE или MANDATORY PROTOCOL" `[author-claim]`.
- **Forced tracing as main method**: post 3809 — "главный метод надёжной загрузки скилов — в другом: «принудительный трейсинг»… считанные скилы будут прямо (по имени) указывать, какие другие скилы считывать" `[author-claim]`.
- **Open Code `skill` tool reinforcement**: post 3809 — "В Open Code можно усилить требование на чтение скила, прямо указав на необходимость вызова инструмента skill" `[empirical]`.
- **Reliability equivalence to trace-based dynamic prompts**: post 3809 — "в таком виде это не менее надёжно, чем динамические промпты на трассировке" `[author-claim]`.

Related but distinct evidence supporting the general "agents skip content that doesn't match training distribution" thesis:

- Agents silently skip / hallucinate around custom MCPs: post 3834 `[empirical]` (cross-link `./mcp-scepticism.md`).
- Agents deprioritize external docs they weren't trained on: post 3820, post 3821 `[research-backed]` (cross-link `../01-transformer-foundations/llm-training-pipeline.md`).
- Forced context as a stabilization mechanism: posts 3695, 3693 `[empirical]` (cross-link `../07-autonomous-agents/forced-context.md`).

The author's research-basis citation for the skill-skipping phenomenon itself is noted as "scientific article" in post 3809 — the sources do not specify the paper's title or URL.

## See also

- `./kilo-code-open-code.md` — Open Code core and its `skill` tool, on which GRACE's current dynamic-skills runtime depends.
- `./mcp-scepticism.md` — parallel failure mode (agents ignoring custom MCPs for training-distribution reasons).
- `../07-autonomous-agents/forced-context.md` — forced context is the sister discipline; the skill-reliability problem is a special case of "don't trust the agent to self-fetch".
- `../07-autonomous-agents/anti-loop-protection.md` — skill-skipping compounds with loop behavior in autonomous agents; mitigations co-occur in the GRACE stack.
- `../01-transformer-foundations/llm-training-pipeline.md` — training distribution is the root cause of "I already know" self-substitution.
- `../13-antipatterns/all-antipatterns.md` — "Skills without forced loading" is catalogued as a GRACE antipattern (post 3809).
