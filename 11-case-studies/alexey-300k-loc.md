# Alexey — 300,000+ LOC GRACE Project

## What

A GRACE-built project reported by a practitioner named Alexey, weighing in
at **more than 300,000 lines of code**. The author frames this as the
largest reported GRACE project at the time of writing, and as a concrete
signal of the efficiency ceiling reachable today with an AI-native
workflow. [empirical] (post 3838)

The post is very short (three sentences plus the link). The summary below
is a faithful reproduction of what the source actually states; everything
beyond that is explicitly flagged as "not specified in the sources".

## Why

This case study matters for three reasons:

1. **Scale proof-point.** [empirical] It gives a concrete data point for
   how far a single practitioner can push a 100%-AI-generated codebase
   when GRACE discipline is applied. The project sits well above the
   "current 100%-AI-made apps: ~100,000 LOC" baseline the author reports
   one post earlier (post 3837) [empirical]; it also sits roughly at the
   low end of the 1–2-year projection of "400–500K LOC per operator
   managing a bot swarm" (post 3837) [author-claim].
2. **Cost framing.** [author-claim] The author converts the 300k+ LOC
   figure into a dollar anchor: "at US prices such a project costs more
   than $7 million if you hire local developers" (post 3838). This is
   used rhetorically to signal the cost-efficiency delta of AI-native
   development versus a traditional US dev team; the author does not
   show the arithmetic behind the $7M figure. [author-claim]
3. **Closes the "vibe-coding doesn't scale" loop.** GRACE's foundational
   claim (post 3654, `most_important.md` §"Почему без семантического
   графа…") is that most AI-code efforts stall after 1–2 modules because
   sparse attention collapses past ~4k tokens and RAG agents read in
   narrow chunks. A 300k-LOC GRACE deliverable is the empirical
   counter-example: with the semantic graph, contracts, and in-source
   documentation in place, the ceiling moves. [empirical, cross-ref
   posts 3654, 3838]

## How

The source post does not describe Alexey's project in any mechanical
detail. **What the source DOES specify:**

- The project "was done on GRACE" (`делался на GRACE`). [empirical]
  (post 3838)
- The size is "more than 300,000 lines of code"
  (`более 300.000 строк кода`). [empirical] (post 3838)
- At US developer rates the equivalent traditional build would cost
  "more than $7 million" (`более $7 миллионов долларов`, "if you hire
  local developers"). [author-claim] (post 3838)
- The author's closing sentence: "Such is the efficiency of AI
  development already" (`Вот такая эффективность ИИ разработки уже`).
  [author-claim] (post 3838)

**What the sources DO NOT specify:**

- The domain of Alexey's project (web, ERP, CRM, bank, construction, etc.). The sources do not specify.
- The language(s) / framework(s) used. The sources do not specify.
- The number of files, modules, contracts, or tests. The sources do not specify.
  (Contrast with the Kirill case, where the author does give 1494 files,
  132,666 prod LOC, 50,026 test LOC — post 3620, see `kirill-132k-loc.md`.)
- The development duration or headcount (sources do not specify whether
  Alexey worked alone, with a team, or with an agent swarm).
- Whether test coverage followed GRACE's LDD pattern (see T26) — the sources
  do not specify.
- The arithmetic and assumptions behind the $7M figure
  (LOC-to-dollar conversion rate, seniority mix, geography inside the
  US, overhead). The sources do not specify; the number is stated flat.
- Alexey's surname, affiliation, or public identity. The sources do
  not specify. (A separate post 3848 refers to an "Alexey Chendemerov"
  who built the Claude Code GRACE plugin — see `kb/09-tooling/claude-code-grace-plugin.md` — but the
  sources do not identify that as the same person; treat them as distinct
  until/unless a later source says otherwise.)

Because the post is this terse, the case study's weight comes from
cross-referencing rather than from the post's own detail.

## Evidence

- **Primary source**: `grace_matches.md`, post 3838 (2026-04-07 20:36
  UTC), full text preserved here:

  > Алексей поделился проектом, который делался на GRACE, он более
  > 300.000 строк кода.
  >
  > По ценам в США такой проект стоит более $7 миллионов долларов, если
  > там нанять местных девелоперов.
  >
  > Вот такая эффективность ИИ разработки уже.

  English rendering: "Alexey shared a project that was built on GRACE,
  more than 300,000 lines of code. By US prices such a project costs
  more than $7M if you hire local developers there. Such is the
  efficiency of AI development already."

- **Supporting context** (post 3837, one hour earlier on the same day,
  2026-04-07 19:25 UTC): the author's broader observation that
  "current size of 100%-AI-made applications is around 100,000 LOC"
  from his training experience (dozens of projects per month), with a
  1–2 year projection of 400–500K LOC per operator managing a bot
  swarm. [empirical / author-claim] See `kb/12-economics-strategy/project-size-trends.md`.

- **Claim type summary**:
  - 300,000+ LOC figure — [empirical], reported by the author on
    Alexey's behalf.
  - $7M+ US-equivalent cost — [author-claim], no methodology given.
  - "Efficiency ceiling of AI-native development today" framing —
    [author-claim].

## See also

- `../12-economics-strategy/project-size-trends.md` — post 3837's
  100k-LOC baseline and 400–500k-LOC 1–2-year projection; Alexey's
  project is the concrete upper-end data point for the baseline.
- `./kirill-132k-loc.md` — the other large-codebase case (132k prod
  LOC + 50k test LOC) where the sources give mechanical detail
  (file counts, annotation migration, 20+ hour autonomous runs). Read
  alongside Alexey as the "detail-rich companion" to post 3838.
- `./vlad-crm-and-contracts.md` — CRM and contracts-project cases
  showing GRACE's effect on model-interchangeability and on the
  tests-without-logs antipattern.
- `../00-foundations/why-grace-exists.md` — the 5k–10k LOC
  noodle-code collapse threshold (post 3654) that a 300k-LOC GRACE
  project empirically refutes.
- `../00-foundations/what-is-grace.md` — GRACE's stated target of
  "100%-AI-generated apps at 800–1000+ LOC scale"; Alexey's project
  is ~300× that entry threshold.
