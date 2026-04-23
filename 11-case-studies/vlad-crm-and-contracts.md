# Vlad: CRM with Model-Swap and the Contracts-Without-Logs Project

Two case studies involving the same practitioner (a colleague the author calls "Vlad", an ex-SAP engineer) on two distinct projects. Together they frame two of GRACE's operational claims: (1) strong semantic structure makes vendor/model choice nearly irrelevant, and (2) contracts without GRACE-style logging are only half the methodology.

## What

**Case A — CRM for a security-company group.** A custom CRM, built by Vlad using GRACE, serving a group of five security companies with 1000+ personnel per shift. It tracks enterprise resources (including weapons), employee documents, shift schedules, and renders personnel on a map, with document-recognition flows. Built over "a few days with AI agents" on a **Next.js + shadcn + Supabase** stack. Vlad frequently switched models mid-development: **Gemini Flash, Gemini Pro, Claude Opus 4.6, GLM-5, Kimi K2.5**. (practical_applications.md §"Пример CRM") [empirical]

**Case B — Contracts project (post 3693).** A separate, evidently larger project by the same colleague, attempting to apply GRACE from the articles alone. Scale: **282 contracts** (author notes these are probably classes) and **1049 autotests**. The contracts layer was applied "more or less correctly", and the autotests-as-foundation-for-autonomous-agents concept was grasped — but GRACE-style logging was missing. (post 3693) [empirical]

## Why

Each case illustrates a different half of the methodology:

- **Case A** is the author's canonical public example of **vendor-agnosticism via semantic scaffolding** — an operational consequence of GRACE that pipeline strength, not model identity, dominates output quality. (practical_applications.md §"Пример CRM") [author-claim]
- **Case B** is the motivating case for **log-to-code correlation** and **belief-state logging** — the author uses it to argue that contracts alone are a half-methodology: "tests without logs is like vodka without beer." (post 3693) [author-claim]

Both cases are cited by the author to push back against two common reactions from engineers: "but which model is best?" (Case A) and "I applied the markup, why is my agent still blind?" (Case B).

## How

### Case A — multi-model swap

Setup per the source post (practical_applications.md §"Пример CRM"):

- **Domain**: CRM for a 5-company security group; 1000+ personnel per shift.
- **Scope covered**: weapons/resource tracking, employee document storage, shift accounting, map display of personnel, document recognition.
- **Stack**: Next.js, shadcn, Supabase. [empirical]
- **Models used, swapped freely during development**:
  - Gemini Flash
  - Gemini Pro
  - Claude Opus 4.6
  - GLM-5
  - Kimi K2.5
- **Reason for swapping** (as Vlad reported): tariff and token limits — switching was pragmatic, not experimental. (practical_applications.md §"Пример CRM") [empirical]

Vlad's direct observation — quoted and endorsed by the author: with GRACE semantic markup and specs in place, switching models "doesn't let you feel a particular difference between them — all generate equally well." In practice the developer's time went into **task formulation**, not into compensating for model quirks. (practical_applications.md §"Пример CRM") [empirical]

The author's generalization from this observation:

> If you notice you have high LLM-sensitivity, that almost always means problems in your development pipeline. With proper agent methodology, switching between LLMs happens the way it does for Vlad — usually without issues — because model idiosyncrasies stop mattering as much. Advanced prompt frameworks box the LLM into a defined channel, so vendor-specific differences surface less. Strong prompt frameworks also smooth over LLM weaknesses.

(practical_applications.md §"Пример CRM") [author-claim]

Operational takeaway the author hangs on this case: **if swapping Claude Opus 4.6 for GLM-5 causes a sharp quality drop, do not debate which model is better — suspect the pipeline**. (practical_applications.md §"Пример CRM") [author-claim]

See `../00-foundations/core-principles.md` for this as a core principle (vendor-agnosticism through semantic scaffolding).

### Case B — contracts without logs

Setup per post 3693:

- Same colleague, different project (not the CRM).
- Vlad attempted to implement GRACE from the public articles alone. [empirical]
- **Scale observed by the author** from a screenshot Vlad shared:
  - **282 contracts** (the author assumes these are class-level contracts).
  - **1049 autotests**.
- The author's reading of Vlad's setup: contracts were applied "more or less correctly"; the core concept that **autotests are the foundation for autonomous-agent work** was also correctly grasped. (post 3693) [empirical]

What was missing: **GRACE-style logging**. The agent itself reflected this back to Vlad in a remark the author quotes — paraphrased in the source as **"tests without logs is like vodka without beer"**. (post 3693) [empirical]

The author's expansion on why this gap matters (post 3693) [author-claim, partly empirical]:

- An AI agent has "no other eyesight besides logs" — logs are the agent's view into execution reality.
- Logging for AI agents is not plug-and-play: **"old-style" logging does not work.** (post 3693)
- Two minimum-viable heuristics the author calls out as critical, both motivated by this case:
  1. **Log-to-code correlation.** Insert the function identifier and the code-block identifier into every log line. This is not optional — the author calls it "archimportant" for GPT's sparse attention. Without these tokens, the agent cannot bind log lines back to code; log + code must fuse into a single semantic monolith for the model to reason over both. The author's Gemini experiments: past ~5000 log lines, attention visibly drops without these tags, even with a 1M-token context. The agent then cannot correlate log fragments with code and misses errors. (post 3693) [empirical, author-claim]
  2. **Belief-state logging.** When the AI writes code, it verbalizes its **belief state** — its hypothesis about how the code works — into the log. During analysis, the AI compares "what I believed" against ground truth and produces corrections from the delta. The author notes: "this concept is not always clear to humans, but GPT understands it perfectly." (post 3693) [empirical, author-claim]
- Beyond these minimums the author mentions (but does not detail in this post): **forced injection of important log lines into the agent's context**, plus **category-based markup of log entries** for filtering. (post 3693) [author-claim]

The author's overall verdict on Case B: contracts did their job (the semantic shield was in place), but without correlation-tagged, belief-state logs the autonomous-agent loop could not close. This is the concrete case the author points to when arguing that GRACE's logging discipline is a load-bearing peer to the contracts discipline, not an optional extra. (post 3693) [author-claim]

### How the two cases relate

They are complementary, not duplicative:

- Case A says: *with* full GRACE structure (markup + specs), the model is nearly a commodity.
- Case B says: *without* logs, even a big correct contracts layer (282 contracts!) and a big test suite (1049 tests) still leaves the agent effectively blind for autonomous debug.

Together they frame the author's view of a "complete" GRACE setup as **graph + contracts + markup + logging**, with any missing leg compromising the others. (Cross-references in the Evidence section below.)

## Evidence

### Case A — CRM

- Source: `practical_applications.md` §"Пример CRM, собранной через GRACE". [empirical reporting from Vlad, endorsed by the author]
- Direct observations cited:
  - Project scale: 5 security companies, 1000+ personnel per shift, enterprise-resource tracking (incl. weapons), employee documents, shift accounting, map display, document recognition.
  - Stack: Next.js + shadcn + Supabase.
  - Model pool: Gemini Flash, Gemini Pro, Claude Opus 4.6, GLM-5, Kimi K2.5.
  - Switching driver: tariff/token limits.
  - Vlad's reported observation: GRACE markup + specs make models feel interchangeable in generation quality.
  - Author's generalization: high LLM-sensitivity is a pipeline smell, not a model-selection problem.

### Case B — contracts project

- Source: `grace_matches.md` (post 3693). [empirical reporting from Vlad's screenshot + author commentary]
- Direct observations cited:
  - 282 contracts + 1049 autotests visible in Vlad's screenshot.
  - Contracts applied "more or less correctly" per the author's review.
  - Missing: GRACE-style logging.
  - Agent's own remark (paraphrased in source): tests without logs is like vodka without beer.
  - Two minimum logging heuristics the author spells out in the same post:
    - log-to-code correlation (function + block ID tokens in every line),
    - belief-state logging (AI writes its hypothesis alongside observable state).
  - Additional heuristics mentioned but not detailed in post 3693: forced context injection of important log lines; category markup in logs.
  - Gemini 1M-token experiment: attention collapse past ~5000 log lines without correlation tags. [empirical]

### Claim-type summary

- **[empirical]**: project scale numbers (Case A staff count; Case B 282/1049 contract/test counts), stack, model list, Vlad's reported interchangeability experience, author's Gemini log experiments.
- **[author-claim]**: the generalization "high LLM-sensitivity = weak pipeline, not a model problem"; the centrality of log-to-code correlation and belief-state logging; the framing of logging as a load-bearing peer of contracts.
- **[research-backed]** / **[vendor-doc]**: none directly — the two cases are empirical field reports; the underlying transformer-internals arguments (sparse attention, Dyck compatibility, PURPOSE/keywords) are research-backed and covered in the cross-linked files.

The sources do not specify Vlad's full name, the exact calendar dates of Case A's build, or the domain industry of Case B beyond "contracts project".

## See also

- `../00-foundations/core-principles.md` — Principle 6 (vendor-agnosticism through semantic scaffolding) is the generalization of Case A.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — explains why without log-to-code correlation attention collapses past ~4k–5k tokens (Case B mechanism).
- `../03-contracts/contracts-overview.md` — the contracts Vlad applied in Case B; semantic-shield framing.
- `../03-contracts/contract-fields.md` — what a "correct" contract layer includes.
- `../05-logging-ldd/log-driven-development.md` — the broader LDD concept Case B motivates.
- `../05-logging-ldd/log-to-code-correlation.md` — heuristic #1 from the Case B analysis.
- `../05-logging-ldd/belief-state-logging.md` — heuristic #2 from the Case B analysis.
- `../05-logging-ldd/forced-context-injection.md` — the follow-on mechanism mentioned in post 3693.
- `../07-autonomous-agents/anti-loop-protection.md` — why autotests-without-logs is especially dangerous for autonomous agents.
- `../11-case-studies/kirill-132k-loc.md` — another case where adopting markup + modernized logging jointly improved agent behavior.
- `../11-case-studies/alexey-300k-loc.md` — the largest reported GRACE project, for scale context.
- `../13-antipatterns/all-antipatterns.md` — "Assuming the agent will self-rescue via Tools" and "Using strict assertions as primary truth" both trace back to the Case B lesson.
