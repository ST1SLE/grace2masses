# Project-Size Trends in 100%-AI-Built Applications

## What

A snapshot and a short-horizon projection, delivered by the GRACE author
in post 3837, of **how large 100%-AI-generated applications currently
are** and **how large they are likely to become over the next 1–2
years** as practitioners improve their agent-orchestration techniques.

Three headline facts from the source:

1. The **current practical size** of 100%-AI-made applications is
   **~100,000 LOC**, based on the author's training-cohort observations
   across "dozens of projects per month". [empirical] (post 3837)
2. The **user profile** is shifting: more and more of the author's
   trainees are engineers who **do not themselves code** in the target
   framework, and **do not want to** — they are engineers of *agent
   management* across design, coding, and testing phases.
   [empirical] (post 3837)
3. The **1–2-year projection** is **400,000–500,000 LOC per operator
   managing a swarm of bots** — conditional on today's development pace
   simply not decreasing. [author-claim] (post 3837)

The post also includes a taxonomy remark about *what kind of apps* these
are (vertical / niche industrial solutions, not pet projects, not
horizontal-market SaaS), which explains why these codebases are largely
invisible from the outside. [empirical] (post 3837)

## Why

This page exists because the economics of GRACE rest on a concrete
empirical observation that the author keeps returning to: the *size
class* of what a single AI-orchestrating engineer can build today — and
can be expected to build soon — has already moved well past the regime
for which classical dev tooling and TDD were designed. Several upstream
pages in this KB derive their weight from exactly this scaling
trajectory:

- `../00-foundations/why-grace-exists.md` argues that **most AI-code
  efforts collapse after 1–2 modules**, with a critical-mass failure
  window in the **5,000–10,000 LOC of interconnected logic** range
  (post 3654). Post 3837's 100k-LOC baseline is what happens when the
  upstream collapse is avoided — the roof is 10–20× higher than the
  naive collapse band. [empirical, cross-ref posts 3654, 3837]
- `../11-case-studies/alexey-300k-loc.md` reports a single GRACE
  project at **more than 300,000 LOC** (post 3838, one hour after
  post 3837 on the same day). That data point sits **above the
  "current 100k-LOC" baseline** and roughly at **the low end of the
  400–500k-LOC projection** — i.e., it is plausibly the public-facing
  evidence that the projection is not a fantasy. [cross-ref posts 3837,
  3838]
- `../11-case-studies/kirill-132k-loc.md` is a detail-rich case of
  ~132,666 prod LOC + ~50,026 test LOC (post 3620). That is
  consistent with post 3837's "around 100,000 LOC" baseline (prod-only;
  add tests and it overshoots), showing the baseline is not a single-
  data-point claim. [empirical, cross-ref posts 3620, 3837]
- `./division-of-labor.md` (T58) explains *who builds methodologies
  like GRACE* versus *who adapts and applies them*. The users of this
  page's projection are the **framework-users** in Sergey's split,
  not the framework-designers. [cross-ref post 3804]
- `./vendor-strategy.md` (T56) explains why big vendors will **not**
  build the full orchestration pipeline implied by "one operator
  managing a bot swarm of 400–500k-LOC apps". They target $1B/year
  products; $100M/year is "seeds" to them. The productivity gain lands
  in partners and individual operators, not in vendor SKUs.
  [cross-ref post 3708]

The reason this matters operationally: if you are an engineer or a CTO
evaluating GRACE, post 3837 is the author's stated *target size class*.
A methodology whose only justification is "it helps scale to 500 LOC"
is not interesting; a methodology whose stated target is a 100k-LOC
*today* and a 400–500k-LOC *in 1–2 years* requires a different
tool-chain, a different testing philosophy (see `../06-testing/`),
a different logging philosophy (see `../05-logging-ldd/`), and a
different billing model (see `./large-rare-prompts-billing.md` — fewer,
larger, more structured prompts, post 3837's operators are not the
ones sending 10,000 small prompts per day). [author-claim]

## How

The post is short (five paragraphs plus a comment prompt). This section
is a faithful reproduction of everything it asserts mechanically,
annotated with claim-type labels and with an explicit "not specified"
section for each load-bearing number that the post does not back with
external evidence.

### The current baseline: ~100,000 LOC

- **Quantity**: "around 100,000 lines of code" (`около 100 000 строк
  кода`). [empirical] (post 3837)
- **Scope**: 100%-AI-made applications. [empirical] (post 3837)
- **Evidence base** the author gives: "my experience from training
  (dozens of projects per month)" (`по моему опыту c обучения (десятки
  проектов в месяц)`). [empirical] (post 3837)
- **Notable secondary remark**: "Usually [these projects] reached this
  size even without my GRACE, but the engineers simply are not going to
  stop at what's been achieved and keep improving their agent-management
  techniques." (`Обычно такого размера достигли и без моего GRACE, но
  инженеры просто не собираются останавливаться на достигнутом и
  улучшают техники управления агентами`.) [empirical / author-claim]
  (post 3837)
  - **Implication inside the source**: the 100k-LOC baseline is not
    claimed as a GRACE-only achievement; GRACE is positioned as the
    next step these same engineers are taking to push *past* the
    baseline. This is consistent with GRACE's self-positioning in
    `../00-foundations/what-is-grace.md` ("target: 100%-AI-generated
    apps at 800–1000+ LOC scale; overkill for small scripts") — GRACE
    does not claim 100k LOC is its invention; it claims GRACE is what
    you need to keep going past it without drift.

### The user profile

The source makes a specific sociological claim about *who* these
100k-LOC builders are. All three points below are empirical
observations from the author's training cohort — not vendor data and
not benchmark figures. [empirical] (post 3837)

- The engineers in question **do not themselves code** in the target
  frameworks (`инженеры, которые сами не умеют программировать в
  целевых фреймворках`). [empirical] (post 3837)
- They **do not want to learn** to code in those frameworks (`и даже
  не хотят это делать`). [empirical] (post 3837)
- They *are* engineers — but engineers **of agent management**, across
  three phases: **design, coding, testing** (`скорее инженеры по
  управлению агентами проектирования, кодирования и тестирования`).
  [empirical] (post 3837)

This profile is congruent with the IDE 2.0 vision in
`../02-semantic-graph/hierarchy-and-anchors.md`, which notes that a
non-programmer can work at the contract level if they know *class*,
*function*, and *variable* concepts. Post 3837 is the real-world
prequel to that UI claim: the user already exists today.

### Application taxonomy: "vertical" not "horizontal", not pet-projects

- The applications are **NOT pet-projects** (`Нет, это не
  «пет-проекты»`). [empirical] (post 3837)
- They are **niche / vertical solutions** for industry cases
  (`нишевые («вертикальные») решения под отраслевые кейсы`).
  [empirical] (post 3837)
- **Because of that narrow specialization**, they are **invisible to
  anyone looking at horizontal-market solutions** (`Из-за узкой
  специализации их и не видно тем, кто ищет что-то из «горизонтальных
  решений»`). [empirical / author-claim] (post 3837)
- They **are rarely "vibe-coded"** (`Они как раз редко
  «вайбкодятся»`). [empirical / author-claim] (post 3837)
  - Cross-link: `../13-antipatterns/all-antipatterns.md`
    "Vibe-coding-at-scale" — post 3654's 5–10k LOC collapse band is
    why vibe-coding does not stretch to these vertical projects;
    post 3837 is the observation that the successful engineers in this
    space already know this. [cross-ref posts 3654, 3837]

### The 1–2-year projection: 400,000–500,000 LOC per operator

- **Quantity**: "on the order of 400–500 thousand lines of code"
  (`порядка 400–500 тысяч строк кода`). [author-claim] (post 3837)
- **Unit**: per **one professional managing a swarm of bots**
  (`при условии, что роем ботов управляет один профессионал`).
  [author-claim] (post 3837)
- **Horizon**: somewhere around 1–2 years from the post date of
  2026-04-07 — so roughly 2027–2028. [author-claim] (post 3837)
- **Stated precondition**: "if the current development pace of
  [my] colleagues simply does not decrease" (`если просто текущий темп
  разработки у коллег не снизится`). [author-claim] (post 3837)
  - In other words, the author does **not** model acceleration; he
    models continuation. The projection is a straight-line
    extrapolation of observed throughput. [author-claim]
- **Operational implication**: the projection *assumes* the human is
  an **operator** of a **bot swarm**, not a solo coder. This ties
  directly to:
  - `../07-autonomous-agents/swarm-vs-single-agent.md` — swarm vs
    single-agent scaling modes (post 3793, post 3859).
  - `../07-autonomous-agents/anti-loop-protection.md` and
    `../07-autonomous-agents/forced-context.md` — the protection
    mechanisms that are **non-negotiable at swarm scale** (post 3695).
  - `../07-autonomous-agents/knowledge-base-in-tests.md` — regression
    memory, without which swarms loop and burn billions of tokens
    (post 3695).
  - If the projection is correct, the cost of NOT having anti-loop +
    forced-context + KB-backed regression tests scales linearly with
    LOC: a 500k-LOC codebase orchestrated by a bot swarm without
    these mechanisms is, per post 3695, a token-incineration engine.
    [cross-ref post 3695, 3837]

### What the source does NOT specify (do not invent)

- The **mix of frameworks** (Next.js / Python / Rust / Go / etc.)
  behind the 100k-LOC baseline. The sources do not specify.
- The **test-LOC-to-prod-LOC ratio** inside the baseline. The sources
  do not specify. (Contrast: post 3620 on Kirill's project gives
  132,666 prod + 50,026 test — that is ONE data point, not a
  population ratio.)
- The **time-to-build** distribution across "dozens of projects per
  month". The sources do not specify.
- The **team structure** behind these apps (solo operator vs. pair
  vs. small team). The author speaks of "engineers" (plural, across
  projects) and of "one professional" (for the 1–2y projection); the
  sources do not specify the team structure of the *current* 100k-LOC
  builds beyond that.
- The **revenue, users, or commercial outcomes** of these vertical
  apps. The sources do not specify.
- The **error/bug rate** or **maintenance cost** of these 100k-LOC
  applications. The sources do not specify. (GRACE's claims in
  `../03-contracts/contracts-overview.md` about contracts as a
  "semantic shield" for modification are relevant but are made in
  other posts, not in 3837.)
- The **geographic distribution** of the trainee cohort. The sources
  do not specify.
- Whether the **400–500k-LOC projection includes tests** or is
  prod-only. The sources do not specify. (Kirill's 132k prod /
  50k test ratio would suggest that 400–500k prod-only implies a
  ~650–700k combined codebase, but post 3837 does not itself draw
  this arithmetic — do not attribute it to the post.)
- The **token-economics** of operating such a swarm (cost per LOC,
  cost per day). The sources do not specify in post 3837.
  `./large-rare-prompts-billing.md` covers the GRACE billing pattern
  for Architect-style prompts (post 3837's author-post companion
  post 3867 in `practical_applications.md`), but the "1 operator /
  500k LOC" economics specifically are not itemized.

## Evidence

**Primary source**: `grace_matches.md`, post 3837 (2026-04-07 19:25
UTC). The post in full (Russian, with English rendering):

> Возможно, кому-то интересная информация. Сейчас на обучении у меня
> довольно часто инженеры, которые сами не умеют программировать в
> целевых фреймворках и даже не хотят это делать. По факту они скорее
> инженеры по управлению агентами проектирования, кодирования и
> тестирования.
>
> Интересный момент — какого размера у них приложения и что это такое.
> Нет, это не «пет-проекты», а скорее нишевые («вертикальные») решения
> под отраслевые кейсы. Из-за узкой специализации их и не видно тем,
> кто ищет что-то из «горизонтальных решений». Они как раз редко
> «вайбкодятся».
>
> Какой размер? Сейчас современный размер 100% сделанных с ИИ
> приложений — около 100 000 строк кода по моему опыту c обучения
> (десятки проектов в месяц). Обычно такого размера достигли и без
> моего GRACE, но инженеры просто не собираются останавливаться на
> достигнутом и улучшают техники управления агентами.
>
> По моей оценке, где-то за 1–2 года такие приложения будут порядка
> 400–500 тысяч строк кода при условии, что роем ботов управляет один
> профессионал. Такой размер получится, если просто текущий темп
> разработки у коллег не снизится.
>
> Как у вас? Пишите в комментариях.

English rendering (faithful, preserving terms):

> Possibly useful information for someone. Right now, very often in my
> training I see engineers who themselves cannot program in the target
> frameworks and do not even want to. In effect they are rather
> engineers of agent management for design, coding, and testing.
>
> An interesting point — what size their applications are and what
> they are. No, these are not pet projects; rather, they are niche
> ("vertical") solutions for industry cases. Because of the narrow
> specialization, they are not visible to those searching for
> horizontal solutions. They are rarely "vibe-coded".
>
> What size? Right now the current size of 100%-AI-made applications
> is around 100,000 lines of code, from my training experience
> (dozens of projects per month). Usually this size has been reached
> even without my GRACE, but the engineers simply are not going to
> stop at what has been achieved and are improving their
> agent-management techniques.
>
> By my estimate, in about 1–2 years such applications will be on the
> order of 400–500 thousand lines of code, provided that one
> professional manages a swarm of bots. That size follows if the
> current development pace of [my] colleagues simply does not
> decrease.
>
> How is it with you? Write in the comments.

**Claim-type summary for the facts on this page**:

- "Engineers don't code in target frameworks, they manage design /
  coding / testing agents." — [empirical], author's training-cohort
  observation. (post 3837)
- "Vertical, niche, industry-case applications; rarely vibe-coded;
  invisible from horizontal-market view." — [empirical / author-claim],
  observation plus characterization. (post 3837)
- "Current 100% AI-made apps: ~100,000 LOC." — [empirical], grounded
  in "dozens of projects per month" exposure. (post 3837)
- "Reached without GRACE; GRACE is the next-step improvement." —
  [empirical / author-claim]. (post 3837)
- "1–2y projection: 400–500K LOC per operator managing a bot swarm." —
  [author-claim], explicitly framed as the author's estimate, with the
  stated precondition "if the pace does not decrease". (post 3837)

**Supporting cross-references inside the KB** (not new evidence; links
to pages that cite the same post or contextually adjacent posts):

- `../11-case-studies/alexey-300k-loc.md` — post 3838's 300k+ LOC
  GRACE project. Posted one hour after 3837; the two posts read
  naturally as a pair (3837 states the baseline and projection, 3838
  gives the concrete above-baseline data point). [cross-ref posts
  3837, 3838]
- `../11-case-studies/kirill-132k-loc.md` — post 3620's 132,666 prod
  + 50,026 test LOC case, consistent with 3837's 100k-LOC baseline
  when prod-only slice is compared. [cross-ref posts 3620, 3837]
- `../00-foundations/why-grace-exists.md` — post 3654's 5–10k LOC
  noodle-code collapse band, which 3837's engineers have already
  pushed past 10–20×. [cross-ref posts 3654, 3837]
- `../00-foundations/what-is-grace.md` — GRACE's stated entry
  threshold of 800–1000+ LOC; post 3837's baseline is ~100× that
  threshold. [cross-ref post 3837]
- `../02-semantic-graph/hierarchy-and-anchors.md` — IDE 2.0 vision
  where non-programmers work at the contract level; post 3837 is the
  sociological ground-truth for that vision. [cross-ref]
- `../07-autonomous-agents/swarm-vs-single-agent.md` — swarm scaling
  modes, the substrate of "one operator managing a bot swarm".
  [cross-ref posts 3793, 3859, 3837]
- `../07-autonomous-agents/anti-loop-protection.md`,
  `../07-autonomous-agents/forced-context.md`,
  `../07-autonomous-agents/knowledge-base-in-tests.md` — the
  protection mechanisms whose absence at swarm scale wastes tokens
  proportionally to LOC. [cross-ref post 3695, 3837]

## See also

- `./vendor-strategy.md` (T56) — why big vendors will not build the
  full orchestration pipeline implied by the projection; they chase
  $1B+/year SKUs, and sub-$100M/year tooling is "seeds" to them
  (post 3708).
- `./division-of-labor.md` (T58) — framework designers vs
  framework users; post 3837's trainees are users who need others to
  design the methodology (post 3804).
- `./large-rare-prompts-billing.md` — the billing-shape that fits
  "few, large, structured prompts" characteristic of Architect-mode
  work (GRACE's ~20 prompts generating 300–400k tokens of design
  context; `practical_applications.md` §"Большие, но редкие запросы").
- `../11-case-studies/alexey-300k-loc.md` — the concrete above-baseline
  data point (post 3838).
- `../11-case-studies/kirill-132k-loc.md` — the detail-rich
  at-baseline data point (post 3620).
- `../00-foundations/why-grace-exists.md` — the collapse band GRACE
  is designed to clear (post 3654).
- `../00-foundations/what-is-grace.md` — GRACE's entry-threshold
  positioning (`most_important.md` §"Формальное введение в GRACE").
- `../07-autonomous-agents/swarm-vs-single-agent.md` — the operator /
  bot-swarm substrate of the 1–2y projection.
- `../13-antipatterns/all-antipatterns.md` — "Vibe-coding-at-scale"
  entry; post 3837 explicitly notes these vertical apps are rarely
  vibe-coded.
