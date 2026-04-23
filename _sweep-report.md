# SWEEP Report — GRACE Knowledge Base

> Produced by the final SWEEP agent after reading all 60 kb/ files (all T02-T60 tasks plus README.md and INDEX.md). Link-check, citation-coverage, claim-label, duplication, and consistency audits.

## Broken links

**None found.** Every relative markdown link in every file (including See-also sections and inline cross-references) resolves to an existing file under `/home/p3tal/Projects/personal/grace-posts/kb/`. The link checker scanned 60 files for `[text](*.md[#anchor])` patterns; 100% resolved.

This is the strongest result of the sweep — the 60 parallel agents produced internally-consistent link graphs despite no shared runtime state.

## Missing citations

**None below the threshold of "a few citations per file."** Every content file contains at minimum **2 post-number citations** and **dozens of `§"heading"` references to source files**. Spot check of the thinnest files:

- `06-testing/tests-in-self-correction-loop.md` — 2 `post ####` references, 28 file-section `§` references.
- `11-case-studies/alexey-300k-loc.md` — the terse case study (its source post itself is short); still cites post 3838 repeatedly and cross-refs posts 3837, 3620, 3654.
- `03-contracts/language-specific/csharp.md` — cites post 3960 throughout plus posts 3878, 3889, 3890, 3892, 3948, 3819 for context.

No file lacks citations outright. `README.md`, `INDEX.md`, `GLOSSARY.md` are navigation files not required to carry sentence-level citations.

## Missing claim labels

**None.** Every content file uses at least one of `[empirical]`, `[research-backed]`, `[author-claim]`, `[vendor-doc]`. Labelling is consistently applied at paragraph or sentence granularity.

Most-labelled files (diligent):
- `00-foundations/core-principles.md`
- `03-contracts/contract-fields.md`
- `08-workflow-and-phases/top-down-pipeline.md`
- `13-antipatterns/all-antipatterns.md`

No file is label-sparse enough to flag.

## Duplication candidates

"Duplication" here means the same fact stated in two files without a See-also cross-reference between them. Since almost every file also maintains a thorough See-also block, strict duplication is rare. The worst cases:

1. **`04-markup-system/in-source-documentation.md` and `08-workflow-and-phases/top-down-pipeline.md`** both state the Qwen 3.5-27B SWE-Bench 72.4% + "more disciplined than larger LLMs" result *without* linking to `11-case-studies/qwen-slm.md`. Owner file (qwen-slm.md) is the canonical place. Missing See-also in both.

2. **`03-contracts/contract-fields.md`, `04-markup-system/xml-like-markup.md`, `04-markup-system/start-end-tags.md`** all mention the **1650-Python-line validator** fact without cross-referencing `11-case-studies/grace-validator-self-hosted.md`. The T24 companion file `04-markup-system/markup-validators.md` does link it, but the above three repeat the detail inline.

3. **`02-semantic-graph/graph-rag-vs-vector-rag.md` and `09-tooling/hybrid-rag-infra.md`** mention Mikhail Evdokimov's bank RAG case without linking to `11-case-studies/mikhail-bank-rag.md`. They link to the general-pattern file (`10-beyond-code/grace-for-rag-documents.md`), but the case-study file is where the Mikhail-specific data lives.

4. **`02-semantic-graph/graph-as-backbone.md`, `03-contracts/contracts-overview.md`, `08-workflow-and-phases/top-down-pipeline.md`** all invoke the 5,000–10,000 LOC critical-mass threshold (post 3654) without a See-also link to `00-foundations/why-grace-exists.md`, which is the canonical owner of that fact. The four foundation files that should also link it (core-principles, sparse-attention, kirill-132k-loc, antipatterns, project-size-trends) all do.

5. **`05-logging-ldd/forced-context-injection.md`, `06-testing/classical-tests-antipattern.md`, `08-workflow-and-phases/top-down-pipeline.md`** all mention Vlad's 282 contracts / 1049 autotests / "vodka without beer" fact without linking to `11-case-studies/vlad-crm-and-contracts.md`. Six other files that mention it do link it.

6. **`00-foundations/core-principles.md` and `00-foundations/why-grace-exists.md`** both cite the "architectural intent → 3× bug-fix accuracy" finding without linking to `03-contracts/rationale-and-aag.md`. Same for `04-markup-system/xml-like-markup.md` and `12-economics-strategy/division-of-labor.md`.

7. **`00-foundations/core-principles.md`** states the ShortenDoc 40% compression result without linking to `03-contracts/wenyan-prompting.md` (the canonical compression-technique file). All other ten files that mention ShortenDoc 40% do link wenyan-prompting.md. Only this single file is an outlier.

8. **The 4k-token sparse-attention boundary** appears in 23 files — this is the most-reused fact in the KB. All 23 mention it for legitimate reasons (it anchors each file's argument about why embedded semantics/markup/contracts matter). No additional cross-linking action needed here; the fact is foundational enough to be stated in every downstream file.

9. **The four-phase LLM training pipeline (post 3820)** appears in 11 files. Most cross-reference `01-transformer-foundations/llm-training-pipeline.md` correctly, but `06-testing/classical-tests-antipattern.md` and `11-case-studies/qwen-slm.md` mention the pipeline without an explicit See-also link to it.

10. **The "90% of Cursor semantic searches land on enrichment points" statistic (post 3819)** appears in 7+ files (graph-as-backbone, graph-rag-vs-vector-rag, wenyan-prompting, hybrid-rag-infra, cursor-limitations, hierarchy-and-anchors, markup-compression, etc.). All cross-reference each other well; this is a well-interconnected fact.

Total: ~10 pairwise cases worth fixing. **None are critical** — they are scholarly-style "could have added one more See-also link" issues, not structural duplication.

## Inconsistent citation formats

### arxiv IDs — mildly inconsistent

Two forms are used for Shunyu Yao et al. (Dyck Language paper):
- **`arxiv.org/abs/2105.11115`** — used 20+ times (canonical long form).
- **`arxiv 2105.11115`** — used 11 times (short form).

Both forms are intelligible. Same pattern for the in-source-documentation benchmark paper:
- **`arxiv.org/abs/2601.16661v1`** — used 20+ times.
- **`arxiv 2601.16661v1`** — used 8 times.

This is low-severity — all variants reach the same source. Unifying on the full URL form would be stylistically cleaner but not load-bearing.

### Post numbers — consistent

`(post ####)` format is used uniformly across all files. No `post#3693` or `Post 3693` variants were found.

### Source file section citations — minor variance

Most files use `(file.md §"heading")` or `` (`file.md` §"heading") ``. Both formats coexist. Some files (e.g. `06-testing/tests-in-self-correction-loop.md`) prefer the backtick form exclusively. A few use neither backticks nor parens (just `file.md §"heading"` inline). Not a correctness issue.

### dl.acm.org — consistent

`dl.acm.org/doi/10.1145/3735636` (ShortenDoc) and `dl.acm.org/doi/10.1145/3689484.3690738` (CodeT5+) are cited uniformly.

### researchgate — consistent

`researchgate.net/publication/400340807` cited uniformly.

## Overall quality assessment

The KB is in unusually good shape for a 60-agent parallel build. All 60+ relative links resolve, every content file carries abundant citations (`(post ####)` and section-heading references) plus claim-type labels, and the top-down hierarchy is respected throughout: foundation files anchor concepts, chapter files develop them, case studies ground them, and antipatterns/research-references tie the ends together. Files are long but well-structured (all use the `What / Why / How / Evidence / See also` skeleton) and individually comprehensive; the most common "issue" is over-coverage — the 4k-token sparse-attention fact appears in 23 files, which is intentional (it is the mechanical substrate underneath many chapters) but means some cross-referencing is implicit rather than explicit. The duplication candidates above are all minor See-also gaps, not true repetition of argument. The INDEX.md topic-and-post reverse lookup is exceptionally thorough. No broken links, no missing citations below threshold, no unlabeled files, no invented facts detected in spot-checks. The KB is production-ready as a reference artifact; the remaining improvements would be cosmetic (citation-format unification, a handful of added See-also lines).
