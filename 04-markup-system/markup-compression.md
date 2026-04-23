# Markup Compression

## What

**Markup compression** in GRACE is the practice of deliberately shortening the in-source documentation that GRACE places inside code — contracts, docstrings, and their fields — so that each anchor delivers maximum semantic value per token (post 3892; post 3819). It sits at the intersection of two lines: the GRACE-specific contract format (PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS) and the wider empirical finding that **shorter in-source descriptions are either neutral or outright better for LLM agents working on that code** (post 3889; post 3892).

Compression here is not a cost-saving afterthought. It is a first-class design concern: the length, language, position, and form of the descriptions directly determine whether the agent improves or collapses when it reads the code [research-backed] (post 3889).

## Why

The author's rationale for aggressive compression rests on **empirical benchmarks of in-source documentation on code-LLM behavior**, summarized from two papers (post 3889; post 3892) and the author's own logit-level experiments (post 3892).

### Benchmarks on in-source documentation (post 3889; arxiv.org/abs/2601.16661v1)

The paper linked in post 3889 measures the effect of various in-source documentation styles on code-agent performance across many programming languages and multiple vendor LLMs [research-backed] (post 3889). Four findings are load-bearing for GRACE:

1. **PURPOSE is critical; DESCRIPTION is optional.** An agent can get along perfectly well without a DESCRIPTION of what a function does — the AI reads code fine — but **without the "why" (PURPOSE), the agent provably degrades** [research-backed] (post 3889). This is why GRACE elevates PURPOSE to the first contract field and treats it as non-negotiable, while treating free-form description as at-best an SFT hint (post 3878; post 3889).

2. **Shorter > longer.** The shorter the comment, the more effective it is [research-backed] (post 3889). Long descriptions are either neutral or cause degradation of the agent. This overturns a classical documentation instinct ("more context is better") and justifies the compressed, wenyan-style KEYWORDS/LINKS approach in GRACE (cross-link `wenyan-prompting.md`, T16; post 3819; post 3889).

3. **English > other languages.** The paper explicitly warns about "sovereign-AI" dogmatism — inserting non-English comments into code so the local community feels at home. Benchmarks show that **English is the best language for comments; other languages cause degradation** [research-backed] (post 3889). The author frames this as a warning specifically to "LLM-nationalists" (post 3889).

4. **Adjacency-to-code matters.** The description must be "maximally pressed against the code" — as GRACE contracts are pressed against the functions they describe [research-backed] (post 3889). If the description is separated from the code by a token-distance — for example, pulled from a separate file via `read_file`, or fetched through an MCP hop — the agent degrades (post 3889). This is one of the author's core arguments for placing contracts **inside the source file, immediately adjacent to the declaration** rather than in a separate documentation system (cross-link `in-source-documentation.md`, T23; `../09-tooling/mcp-scepticism.md`, T45; post 3821; post 3878).

### The effect sizes are not small

The benchmarks yield very wide swings: **from −95% degradation on bad description structure, up to +435% improvement when the in-source description is well-structured** [research-backed] (post 3889). These numbers are from tests "on a large set of programming languages and LLMs from different vendors" (post 3889), which the author explicitly flags as an important strength of the paper — the effects are not a single-vendor artifact.

The takeaway GRACE draws from these numbers: **markup decisions are not cosmetic.** The same fields, placed and compressed differently, can flip agent behavior by almost an order of magnitude in either direction [research-backed] (post 3889).

### ShortenDoc (post 3892; dl.acm.org/doi/10.1145/3735636)

Post 3892 discusses ShortenDoc, a small neural network trained to compress Python docstrings while preserving code-generation quality. Its results tighten the argument above:

- Full prepositional phrases and filler wording in docstrings **cause LLM degradation** during code generation [research-backed] (post 3892).
- "Mutilating" the text — throwing away words until only the key concepts remain — **improves HumanEval (simple-function generation) pass rate by roughly 10%** [research-backed] (post 3892).
- More importantly, **docstrings can be compressed by 40% by dropping words, either with no loss of code-generation quality or with a small gain** [research-backed] (post 3892).
- The authors verify this rigorously through **perplexity** (token-level closeness of the compressed description to the model's expectation), proving the LLM actually understands the compressed form rather than merely tolerating it [research-backed] (post 3892).

The author's phrase for what this reveals: *"GPT has in fact already invented a secret transformer language, which it understands and you do not"* [author-claim] (post 3892).

### The "why" behind both findings

Why does less work better? Two mechanisms appear across the sources:

- **FIM training dominance.** In the base coder training regime, LLMs see code overwhelmingly, not natural-language documentation; docs are actively banned in the FIM phase (cross-link `../01-transformer-foundations/llm-training-pipeline.md`, T08; post 3820). So the model's cheapest path to understanding is always "read the code"; heavy prose around the code acts as noise that competes with the high-signal code tokens [author-claim] (post 3820).
- **Sparse-attention budget.** Past ~4k tokens, attention runs on sliding windows of ~100–200 tokens stitched via the semantic graph (cross-link `../01-transformer-foundations/sparse-attention-and-kv.md`, T06; `most_important.md` §"Без семантического графа"). Every unnecessary token of prose in the contract consumes a slot that a real anchor could have taken. Shorter markers mean more real anchors per window [author-claim] (post 3819; post 3892).

## How

### The compression heuristic

GRACE applies a deliberately aggressive compression rule to in-source documentation:

1. **Start with the smallest possible form** that still captures PURPOSE, the typed INPUTS/OUTPUTS line, and compressed KEYWORDS/LINKS. The canonical example (reproduced verbatim from post 3878) is only seven lines of comments for a `save_config` function:

    ```python
    # PURPOSE:Сохранение параметров в JSON файл.
    # INPUTS:
    # - dict - config: Словарь параметров
    # - str - config_path: Путь к файлу (по умолчанию "config.json")
    # OUTPUTS:
    # - bool - Статус успеха операции
    # KEYWORDS:[Saver, FileWrite]
    def save_config(config: dict, config_path: str = "config.json") -> bool:
        """Сохраняет переданный словарь параметров в JSON файл."""
    ```

    (Note: this file quotes the example as-is; the author's own code here is in Russian. In a production GRACE project, per the benchmarks in post 3889, the comments would be in English.)

2. **Drop what you do not need.** DESCRIPTION is optional — the agent can do without it (post 3889). Long prose in the docstring is optional and sometimes harmful (post 3892).

3. **Never drop PURPOSE.** It is the field with the strongest proven effect on agent behavior (post 3889; post 3890) and is the alignment anchor for the RL-trained "goal-achievement" behavior in modern LLMs (post 3878).

4. **Keep English.** Even if the team is non-English-speaking, write the in-source documentation in English for the benchmark-verified quality edge (post 3889).

5. **Keep it adjacent.** Do not move the description into a separate documentation file, Confluence page, MCP-served docstore, or graph DB — every hop costs agent performance (post 3889; cross-link `../09-tooling/mcp-scepticism.md`, T45).

### The author's two-part acceptance test for a compression

Before accepting a compressed phrase into a GRACE contract, the author runs **two validations** (post 3892):

1. **Logit prediction check.** Feed the compressed phrase to the model and inspect whether it correctly predicts the follow-up tokens (the same trick the ShortenDoc authors used via perplexity). If the LLM smoothly continues the compressed form, it has internalized the compression [author-claim] (post 3892).

2. **Blind LLM-gloss test.** Show GPT only the compressed phrase — no code, no instructions — and ask it to describe what the phrase means on "the secret transformer language". If GPT correctly unpacks the compression without code context, the compression is semantically safe [author-claim] (post 3892).

Only a compression that passes **both** checks is accepted into the GRACE KEYWORDS/LINKS vocabulary [author-claim] (post 3892). This is the discipline the author advises over blind abbreviation.

### Cross-link to ShortenDoc as practical lower bound

The ShortenDoc result establishes a concrete, empirically-supported floor: **~40% compression of in-source descriptions is safe by default** (post 3892). This gives practitioners a defensible starting point when negotiating contract-length review: compress docs down by roughly 40% without needing per-project validation, because the effect has been measured across the HumanEval distribution (cross-link `wenyan-prompting.md`, T16).

Beyond 40% the author recommends falling back to the two-part test above, because the empirical guarantee no longer applies [author-claim] (post 3892).

### What the compression heuristic does NOT say

- It does **not** say "remove contracts." GRACE is unambiguous that contracts are central (cross-link `../03-contracts/contracts-overview.md`, T13). The argument is to keep the contract and compress its prose — not to drop the contract.
- It does **not** say "never have a docstring." Docstrings can still live below a function declaration as an SFT hint; sparse attention's ~1000-token recent window typically covers them (post 3892; cross-link `../03-contracts/contract-fields.md`, T15).
- It does **not** say "single-line summary is enough for AI." It says that a one-line summary is no worse than a multi-line one for the AI (post 3960) — the AI reads code fine — while PURPOSE / KEYWORDS / LINKS carry more semantic value per token than a summary of any length [author-claim] (post 3960).

### The practical bundle

Rolling the findings together, the operational rule in GRACE is:

> **Aggressively minimize the prose inside in-source documentation, then verify each compressed phrase with a logit prediction check and a blind LLM-gloss test. Keep PURPOSE, keep adjacency-to-code, keep English.**

This is the rule the author actually applies before adding a compressed phrase to a GRACE template (post 3892).

## Evidence

### Primary source posts

- **Post 3889** (2026-04-12) — arxiv.org/abs/2601.16661v1. The decisive benchmark paper for this file. Establishes: PURPOSE critical, shorter > longer, English > other languages, adjacency-to-code matters. Effect sizes: −95% to +435% swings across many programming languages and vendor LLMs [research-backed].
- **Post 3892** (2026-04-13) — dl.acm.org/doi/10.1145/3735636 (ShortenDoc). Establishes: 40% compression of docstrings without quality loss, ~10% HumanEval gain when filler is cut, perplexity used as verification. Author's two-part check (logit + blind LLM-gloss) is described here [research-backed + author-claim].
- **Post 3890** (2026-04-12) — researchgate.net/publication/400340807. Complementary: "Captain Obvious" comments are useless or harmful; architectural-intent comments raise bug-fix accuracy up to 3× [research-backed]. Supports the "don't repeat what the code says, do state the why" direction of compression (cross-link `../03-contracts/rationale-and-aag.md`, T17).
- **Post 3819** (2026-04-05) — wenyan-prompting as the compression technique used in KEYWORDS/LINKS; ~1:15 token compression vs full prose; 90% of Cursor's semantic searches land on compressed enrichment points [author-claim] (cross-link `wenyan-prompting.md`, T16).
- **Post 3878** (2026-04-12) — the canonical Python contract example reproduced here; establishes PURPOSE placement before the declaration and the "in-source documentation / documentation-as-code" lineage.
- **Post 3960** (2026-04-21) — C# observation: single-line summary equivalent to long summary for AI; KEYWORDS/LINKS carry more semantic value than summary of any length (cross-link `../03-contracts/language-specific/csharp.md`, T20).

### External references cited by the sources

- **arxiv.org/abs/2601.16661v1** — in-source documentation benchmarks (post 3889). Paper title and authors are not specified in the source text.
- **dl.acm.org/doi/10.1145/3735636** — ShortenDoc (post 3892). Paper title and authors are not specified in the source text beyond "ShortenDoc".
- **researchgate.net/publication/400340807** — *On the Impact of Code Comments for Automated Bug-Fixing: An Empirical Study* (post 3890). Authors are not specified in the source text.

The sources do not specify additional bibliographic details for these papers beyond the URLs above.

## See also

- `wenyan-prompting.md` (T16) — the compression technique itself: KEYWORDS/LINKS, ~1:15 ratio, the author's validation loop.
- `../03-contracts/contract-fields.md` (T15) — full contract-field spec and the PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS canonical example.
- `../03-contracts/rationale-and-aag.md` (T17) — why architectural-intent comments help while "Captain Obvious" comments hurt; up-to-3× bug-fix accuracy finding.
- `in-source-documentation.md` (T23) — the lineage: "In-source documentation" / "Documentation as Code" as Enterprise standard; GRACE differs in *what* to put inside, not the format.
- `../01-transformer-foundations/llm-training-pipeline.md` (T08) — why the FIM training regime biases the model toward reading code rather than prose docs.
- `../01-transformer-foundations/sparse-attention-and-kv.md` (T06) — why token budget inside the ~100–200 sliding window is scarce, motivating compression.
- `../03-contracts/language-specific/python.md` (T19) — Python-specific placement: PURPOSE/INPUTS/OUTPUTS/KEYWORDS before the `def`, docstring below.
- `../03-contracts/language-specific/csharp.md` (T20) — C# note that the classical `summary` tag is sub-optimal vs PURPOSE + KEYWORDS + LINKS.
- `../09-tooling/mcp-scepticism.md` (T45) — why moving docs out of the source file (into an MCP or graph DB) violates the adjacency finding.
- `../13-antipatterns/all-antipatterns.md` (T59) — catalog entries for "long summary-style descriptions", "sovereign-AI non-English comments", and "Captain Obvious comments".
- `../14-research-references/annotated-bibliography.md` (T60) — full annotated entries for arxiv 2601.16661v1, ShortenDoc, and the bug-fixing comments paper.
