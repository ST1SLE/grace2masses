# Large, Rare Prompts and Billing Economics

## What

GRACE is an **architect-style workflow** whose request profile is **few, large, highly-structured prompts**, not many small ad-hoc ones. [author-claim] In Architect mode a typical design session is on the order of **~20 prompts that together generate roughly 300,000–400,000 tokens** of design context for the application. (practical_applications.md §"Большие, но редкие запросы")

Because of this shape, the economically rational billing model is a **flat-rate "coding assistant" subscription with a high per-day request quota**, not a per-token API meter. [author-claim] The specific recommendation made in the sources is **Google Gemini Code Assist** at **$20–$40 per month flat**, which on the referenced quota page provides on the order of **~2000 requests per day** — more than GRACE discipline is likely to consume. (practical_applications.md §"Большие, но редкие запросы")

Reference (vendor-doc, quotas page): `developers.google.com/gemini-code-assist/resources/quotas` (practical_applications.md §"Большие, но редкие запросы")

## Why

**The "many small prompts" anti-pattern is what flat-rate coding assistants are priced for, not what GRACE looks like.** [author-claim] The sources contrast "bardachnoe metanie" (chaotic darting) with GRACE's disciplined process: chaotic usage can actually hit high request counts, whereas GRACE, with a clear AI-driven development process, issues infrequent but heavy requests. (practical_applications.md §"Большие, но редкие запросы")

Three consequences follow from this load profile:

1. **Per-token API pricing is a bad fit.** [author-claim] GRACE deliberately inflates the *tokens per request* (Architect-mode prompts produce hundreds of thousands of tokens of structured design context), so paying by the token to a raw API endpoint means paying full price for exactly the thing GRACE makes heavier. (practical_applications.md §"Большие, но редкие запросы")

2. **Per-day request quotas are trivially under-consumed.** [author-claim] The source states explicitly that on a GRACE workflow it is hard to even reach 1,000 requests per day; a flat-rate plan offering ~2,000/day therefore leaves substantial headroom at a fixed cost. (practical_applications.md §"Большие, но редкие запросы")

3. **Operator cost becomes predictable.** [author-claim] The tradeoff "fewer big prompts" is what Gemini Code Assist's flat tier is economically tuned for, so the author concludes that for methodologies with a clear AI development process, **"Assist tariff is exactly right"**. (practical_applications.md §"Большие, но редкие запросы")

This billing shape is a downstream consequence of GRACE's upstream choices — **top-down pipeline** (T39) and **Architect mode** (T40) — so economics here are a symptom of methodology, not a tooling hack.

## How

### Request shape to expect

- **[author-claim]** Architect mode: **up to ~20 prompts per design session** (practical_applications.md §"Большие, но редкие запросы").
- **[author-claim]** Tokens produced per session: **~300,000–400,000** of design context for the application (practical_applications.md §"Большие, но редкие запросы").
- **[author-claim]** Daily request ceiling under GRACE discipline: even 1,000 requests/day is hard to reach (practical_applications.md §"Большие, но редкие запросы").

The sources do not specify request/token profiles for Code or Debug modes individually — the ~20 prompts / 300–400k tokens figures are given for **Architect-mode design sessions**. (practical_applications.md §"Большие, но редкие запросы")

### Billing recommendation

- **[author-claim]** Use **Google Gemini Code Assist**, not the Gemini API billed per token (practical_applications.md §"Большие, но редкие запросы").
- **[vendor-doc]** Price range cited: **$20–$40/month flat** (practical_applications.md §"Большие, но редкие запросы").
- **[vendor-doc]** Quota cited: on the order of **~2000 requests/day** (practical_applications.md §"Большие, но редкие запросы").
- **[vendor-doc]** Reference URL: `developers.google.com/gemini-code-assist/resources/quotas` (practical_applications.md §"Большие, но редкие запросы").

The sources do not specify whether other vendors (Anthropic, OpenAI) have equivalent flat-rate coding-assistant plans with comparable quotas — the concrete vendor example given is Gemini Code Assist. The sources do not specify exact Gemini Code Assist pricing tiers, feature differences between $20 and $40, or the precise break-down of the ~2000/day quota by model or request type; for current exact numbers the linked vendor quota page is the authority.

### Decision rule

If your working style already looks like GRACE — **prompt once, pay for lots of tokens per prompt, iterate rarely** — pick the flat-rate assistant tier. [author-claim] If your working style is **many small prompts throwing code at the wall** ("bardachnoe metanie" in the source), that is the case where per-token API pricing matches usage — but the source frames this as the weak case, not the recommended case. (practical_applications.md §"Большие, но редкие запросы")

### Relationship to Architect mode

This billing observation is not a standalone tip. It only makes sense because of GRACE's upstream phase model:

- **[author-claim]** Architect mode is where the "few large prompts" live — that is the phase the billing regime is sized for (practical_applications.md §"Большие, но редкие запросы"; see also T40 §Architect mode).
- **[author-claim]** Kilo Code makes Architect/Code/Debug explicit UI modes, so the per-session Architect burst is architecturally visible (practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"; see T40).
- **[author-claim]** If an operator sees their request counts climbing toward the 1000+/day range, that is more likely a signal that the pipeline isn't GRACE-shaped than that the assistant quota is inadequate (practical_applications.md §"Большие, но редкие запросы").

### What "few, large, structured" means in practice

- **Few** — **[author-claim]** ~20 prompts per Architect session (practical_applications.md §"Большие, но редкие запросы").
- **Large** — **[author-claim]** hundreds of thousands of tokens of generated design context per session (practical_applications.md §"Большие, но редкие запросы").
- **Structured** — the prompts carry GRACE's upstream artefacts (graph, module contracts, function contracts). The "structured" qualifier comes from the same source and is what lets the small number of prompts do the work of many small ones. (practical_applications.md §"Большие, но редкие запросы")

The sources do not specify exact prompt templates for the ~20 Architect-session prompts; that level of concrete content is the domain of the training materials referenced elsewhere in the corpus (`practical_applications.md` discusses Kilo Code prompt migration in §"GRACE + PCAM и интерфейс Kilo Code" but does not enumerate the Architect-session prompt set itself).

## Evidence

### Primary source

- **practical_applications.md §"Большие, но редкие запросы"** — the entire section. Key claims reproduced:
  - "На деле платить за Gemini лучше не по API по токенам, а именно через программу Google Assist." [author-claim]
  - "В этом случае у вас за $20-$40 в месяц порядка 2000 запросов в день." [vendor-doc, relayed via author]
  - "Если используется методика разработки как GRACE, где акцент сделан на крупные, но сравнительно редкие промпты, то выбрать даже 1000 запросов в день не просто." [author-claim]
  - "Это в бардачном метании нужного много запросов к ИИ, а в GRACE запросов в том же Architect режиме вряд ли будет больше 20 штук, хотя они сразу сгенерят порядка 300-400 тысяч токенов контекста для проектирования приложения." [author-claim]
  - "На мой взгляд, для методик с четким ИИ процессом разработки как раз Assist тарификация самое то, и.к. запросы нечастые, а контекста много." [author-claim]
  - Reference link: `developers.google.com/gemini-code-assist/resources/quotas`. [vendor-doc]

### Supporting cross-references

- **practical_applications.md §"GRACE + PCAM и интерфейс Kilo Code"** — establishes that Architect mode is an explicit UI mode in Kilo Code, making the "few large prompts" phase visible. [author-claim] Used here only as the upstream tie-in (why the billing shape exists); see T40 for the full Architect/Code/Debug treatment.

### Not claimed

The sources in this corpus do not provide:
- benchmark numbers for cost per LOC produced under Gemini Code Assist vs per-token API pricing;
- equivalent recommendations for Anthropic Claude or OpenAI GPT subscription tiers;
- a specific breakdown of what counts as "1 request" under Gemini Code Assist quotas (the vendor quota page is cited as the authority);
- token cost/price-per-million comparisons between the API and Code Assist tiers.

For the "why big labs won't build a full GRACE framework" economic argument (a related but separate strategic claim based on post 3708's $100M / $1B thresholds), see `../12-economics-strategy/vendor-strategy.md` (T56). That argument is about vendor product strategy, not per-operator billing.

## See also

- `../08-workflow-and-phases/top-down-pipeline.md` — the three-phase pipeline (Architectural graph → module contracts → function contracts + code) that produces the "few, large prompts" profile. [T39]
- `../08-workflow-and-phases/architect-code-debug-modes.md` — Kilo Code's explicit Architect/Code/Debug UI modes that surface the Architect-mode prompt burst this billing discussion is sized for. [T40]
- `../09-tooling/kilo-code-open-code.md` — Kilo Code / Open Code tooling that hosts the Architect-mode workflow in practice. [T42]
- `../12-economics-strategy/vendor-strategy.md` — strategic/vendor-scale economics (why Anthropic/Google/MS do not ship a full GRACE-tier framework). [T56]
- `../12-economics-strategy/project-size-trends.md` — projected 100k → 400–500k LOC per operator trend that sizes the long-run budget context. [T57]
- `../12-economics-strategy/division-of-labor.md` — framework designer vs framework user division of labor that underlies "a solo engineer running one Code Assist seat can still ship big projects". [T58]
