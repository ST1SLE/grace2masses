# AI-Contracts vs Classical Design by Contract

## What

GRACE's "AI-contracts" — the in-code specification blocks placed near functions and modules (PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS, RATIONALE/AAG) — look superficially like classical Design by Contract (DbC), but they are **not** the same thing. Classical DbC (Eiffel, JML, Hoare logic) is a formal-verification apparatus aimed at proving program correctness; AI-contracts are short, LLM-aligned "micro-specs" whose goal is to steer code generation and modification, not to verify it. [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

The two share **only** two things:

1. **Location** — both live in or directly next to the code they describe. [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")
2. **Intent** — both express requirements on the algorithm being described. [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

The **form** — what goes inside the contract block, and how that text is consumed — is fundamentally different, because AI-contracts are tuned to how GPT-class models actually read code, not to how a theorem-prover reads formal logic. [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

## Why

Classical DbC assumes a symbolic verifier on the other end of the contract. LLMs are not symbolic verifiers — they are statistical next-token predictors — and their training pipeline was actively scrubbed of the style of strict test-like conditions DbC relies on. That makes the classical form an actively bad fit, not merely an awkward one. The sections below unpack what classical DbC looks like, why vendors burned test-shaped artefacts out of training, and what AI-contracts inherit instead.

### Classical DbC — Hoare logic, Eiffel, JML

Classical Design by Contract was built on **Hoare logic**, the formalism `{P}C{Q}` — precondition P, command C, postcondition Q. The idea was to formalize requirements on an algorithm via pre- and post-conditions (and invariants) and then **verify** correctness of the implementation against those conditions. [research-backed] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

This formalism was embodied in languages and tools:

- **Eiffel** — the language that introduced Design by Contract into mainstream practice. Classical contracts "were based on Hoare logic `{P}C{Q}` — the idea was to formalize requirements on the algorithm through pre- and post-conditions and then verify correct operation." [research-backed] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")
- **JML (Java Modeling Language)** — "a formal specification language designed for annotating Java code in the Design by Contract paradigm." [research-backed] (`contracts_as_anchor.md` §"Научная опора")

Both assume a downstream consumer that can check predicates symbolically. That downstream consumer is not the LLM.

### Why vendors "burned" strict test-like conditions from training

This is the single load-bearing claim that separates AI-contracts from DbC. Putting strict, test-like conditions next to code during training produced **overfitting**: the model would start fitting solutions to those specific conditions, and those solutions failed to generalize. [empirical] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

> "In the case of LLMs this, on the contrary, turned out to be bad practice. If during training you show the AI strict test conditions, it starts fitting solutions to them specifically — and they then fail to generalize. So LLM vendors effectively burned test examples 'with napalm' out of training." [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

The same pattern is visible throughout the GRACE training-pipeline account: the FIM (Fill-In-the-Middle) phase carries a "total and complete ban on code documentation," because its only goal was "how correct is this fragment relative to the rest." [research-backed] (post 3820) The SFT phase trains on 2–3-paragraph tasks paired with 200–300 lines of code — natural-language intent, **not** formal pre/post-conditions. [research-backed] (post 3820) And the RL phase (SWE-Bench-style) works on fix/enhance patches driven by isolation tests, again without documentation. [research-backed] (post 3820)

In short: strict Hoare-style clauses are either absent from, or actively purged from, the distribution the model was shaped by. Asking an LLM to generate or obey them puts it off-distribution.

The echo of this decision shows up elsewhere in GRACE: when AI-contracts and tests are both placed into the generation context together, the model "simply narrows its solution space and immediately generates code that passes your tests" — overfitting at inference time that mirrors the overfitting vendors removed at training time. [empirical] (`testing_and_autonomous_agents.md` §"Еще один пост про 'тесты как шум'") This is why GRACE treats tests as a **self-correction-loop signal** rather than a contract clause (see [testing antipatterns](../06-testing/classical-tests-antipattern.md) and [tests in self-correction](../06-testing/tests-in-self-correction-loop.md)).

### AI-contracts' actual lineage: Python docstrings + Spec-Driven Development

If AI-contracts are not DbC, what are they? Two closer relatives:

1. **Python docstrings** — "from the LLM's point of view, its 'contracts' are much closer to the Python practice of docstrings, which is orders of magnitude more widespread." OpenAI is cited as training ChatGPT specifically on docstrings for code generation. [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC"; post 3892)
2. **Spec-Driven Development (SDD)** — "in spirit, [closer] to the philosophy of Spec-Driven Development (SDD). Here a contract is not a validator but a 'micro-spec' (micro-TZ)." [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

"Micro-spec, not validator" is the phrase to remember. An AI-contract exists to tell the model *what this piece of code is for and how it fits in*, so that generation and later edits stay aligned. It does not exist to let a verifier prove properties of the implementation.

### Do not invent your own format — extract the model's patterns from SFT

This is the operational rule that falls out of the above. Because the model's relationship to contracts was shaped by SFT ("huge number of task → code pairs"), the model **already knows** what a good contract for a given function looks like: it will reproduce task descriptions close in meaning to ones it saw during supervised fine-tuning. [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

The GRACE rule is blunt:

> "A very important point: DO NOT FOLLOW the standards of classical contract programming. They have all flown into the trash. You need to first extract the target AI's own contract-programming patterns, then follow them. I did exactly this. Want predictability from the AI? Sit on the AI's own patterns." [author-claim] (`contracts_as_anchor.md` §"Почему контрактное программирование")

The positive consequence is that "you don't really need to explain to the AI in detail how to write semantic contracts on functions, because it … already knows it from its SFT." [author-claim] (`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC")

## How

### The distinction, operationally

| Axis                  | Classical DbC (Eiffel / JML / Hoare)                        | AI-contracts (GRACE)                                                         |
| --------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Formalism             | Hoare logic `{P}C{Q}` — pre/post/invariants                 | Natural-language micro-spec aligned to SFT task descriptions                 |
| Consumer              | Symbolic verifier / compiler / tool                         | LLM attention, in-context                                                    |
| Training relationship | Not in LLM training; strict test-like conditions purged     | Neighbourhood of docstring/SFT distribution the model was trained on         |
| Purpose               | Verification of correctness                                 | Alignment of generation and modification with original intent                |
| Form                  | Predicate logic, quantifiers, assertions                    | PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS, RATIONALE — short, plain language |
| What is shared        | *Location* (near/in code) and *intent* (requirements on algorithm) — nothing else |

Citations for the table rows are consolidated in the Evidence section below; the "what is shared" row is from `contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC".

### The practical rule, stated three ways

1. **Do not port Eiffel/JML predicates into comments.** The vendor removed this style from training on purpose; you are fighting the model. [author-claim] (`contracts_as_anchor.md`)
2. **Do not invent a novel formalism.** "Home-grown" formats that the LLM never saw during training are "kulibinshchina" and will be ignored or hallucinated around, exactly like the DIY CLI and custom MCPs the author had to rip out of his own testing harness. [empirical] (post 3834)
3. **Do use the fields the model already expects** — PURPOSE, INPUTS, OUTPUTS, KEYWORDS, LINKS — and let it fill them in its own idiom from SFT. Full field spec is in [contract-fields](contract-fields.md); the concrete Python example (from post 3878) shows the form. [author-claim] (`contracts_as_anchor.md`; post 3878)

### What this means for legacy code

AI-contracts being "softer" than DbC does not mean formal contract recovery has no place. Google's **CodeT5+**, fine-tuned on JML (classical DbC), recovered contracts from legacy Java at **90–97% quality** depending on test class, focusing on critical input/output conditions and invariants. [research-backed] (`contracts_as_anchor.md` §"Научная опора"; dl.acm.org/doi/10.1145/3689484.3690738)

The takeaway is **not** "therefore use JML-style contracts for AI" — it is the opposite: a formal-spec framework can *extract* critical conditions from old code at high fidelity, and those extracted invariants can then seed the softer AI-contract fields (PURPOSE, INPUTS, OUTPUTS). Author's own legacy recovery was evaluated via an LLM-reviewer rather than formal tests, and reported positive. [empirical] (`contracts_as_anchor.md` §"Научная опора") See [legacy-contract-recovery](legacy-contract-recovery.md).

### Why the form difference matters in practice

Two concrete mechanisms explain why AI-contracts have to look different, beyond the training-distribution argument:

- **Causal read / anchor vectors.** Descriptions placed **before** a function declaration form the anchor vector behind the function-name token during causal reading — critical for the LLM's later attention patterns. [author-claim] (post 3878) A Hoare clause attached *after* a function (as postconditions) or in a separate file (as JML annotations) does not contribute to this anchor.
- **Sparse attention and compression.** Past ~4k tokens, attention collapses to sliding windows of ~100–200 tokens and needs explicit anchors to stitch them together. [research-backed] (`most_important.md` §"Без семантического графа") Keyword/link-style compression (wenyan prompting) gives ~1:15 token compression vs full prose; [author-claim] (post 3819) ShortenDoc demonstrated 40% compression without quality loss and ~10% HumanEval gain by keeping only key concepts. [research-backed] (post 3892, dl.acm.org/doi/10.1145/3735636) Classical DbC predicates do not compress in this way — they are load-bearing in a symbolic sense, not a statistical one.

So the form difference is not a stylistic preference. AI-contracts are shaped by how transformers read code; classical DbC is shaped by how verifiers read code. The two optimisations pull in different directions.

## Evidence

### Primary source — the DbC-contrast post

- **`contracts_as_anchor.md` §"Почему AI-contracts не равны классическому DbC"** — the single most important passage for this topic. States explicitly: classical DbC was based on Hoare logic `{P}C{Q}`; LLMs overfit to strict test-like conditions, so vendors burned them from training; AI-contracts are closer to Python docstrings and the SDD philosophy; the common ground with classical DbC is only *location in code* and *intent* (requirements on the algorithm); the form is fundamentally different and tuned for GPT; do not invent your own format, extract the model's patterns from SFT.

### Supporting sources on the training pipeline

- **Post 3820** — four-phase training (base / FIM / SFT / RL). Establishes why external-documentation-style and formal-spec-style artefacts are off-distribution: FIM has total ban on documentation; SFT uses 2–3-paragraph natural-language tasks plus code; RL uses isolation tests without documentation. [research-backed]
- **Post 3834** — reliability of standard tools vs "kulibinshchina" (DIY). Author's own DIY CLI next to pytest was ignored or hallucinated around because it was not in training; pytest is "reliable like a rock" because it was. Same principle applies to contract formats: stay on what the vendor trained the model on. [empirical]

### Supporting sources on contract form

- **Post 3878** — the canonical GRACE Python contract block (PURPOSE / INPUTS / OUTPUTS / KEYWORDS with docstring below the def), and the Purpose Breakdown Structure / Anthropic result about hierarchical goal decomposition. Also makes the "descriptions go before the declaration for causal-read anchor vectors" argument. [author-claim]
- **Post 3889** — in-source documentation benchmarks (arxiv.org/abs/2601.16661v1): PURPOSE critical; shorter is better; English outperforms other languages; adjacency matters; effects range from −95% to +435%. [research-backed] Reinforces that the useful content of an AI-contract is different from the useful content of a DbC assertion.
- **Post 3890** — architectural-intent comments raise bug-fix accuracy up to 3× (researchgate pub 400340807); attention analysis showed agents attend to intent tokens when reconstructing broken logic. [research-backed] This is why AI-contracts carry RATIONALE/AAG — something classical DbC does not contain.
- **Post 3892** — ShortenDoc (dl.acm.org/doi/10.1145/3735636): 40% compression without quality loss, ~10% HumanEval gain by dropping prepositional phrases in favour of key concepts; verified via perplexity. [research-backed] AI-contract fields tolerate and benefit from this compression; Hoare predicates do not.
- **Post 3819** — wenyan prompting; KEYWORDS/LINKS as the mechanism; ~1:15 token compression vs full prose; Grok recovered hidden meaning from the contract alone; Cursor semantic search lands on these enrichment points ~90% of the time. [empirical]

### Supporting source on legacy contract recovery

- **`contracts_as_anchor.md` §"Научная опора"** — Google CodeT5+ fine-tuned for contract recovery on JML, 90–97% quality on critical input/output conditions and invariants (dl.acm.org/doi/10.1145/3689484.3690738). Important boundary case: a formal DbC pipeline is useful for *extracting* invariants from legacy, which then seed AI-contract fields. It is not an endorsement of Hoare-style authoring for new AI-native code. [research-backed]

### Indirect support (training-context overfitting)

- **`testing_and_autonomous_agents.md` §"Еще один пост про 'тесты как шум'"** — "if you directly include tests in the same AI contract conditions, the AI simply truncates its solution space and immediately generates code that passes your tests." [empirical] This is the inference-time version of the training-time overfitting vendors worked to eliminate, and it is the operational reason GRACE keeps tests out of the contract and in the self-correction loop.

## See also

- [../03-contracts/contracts-overview.md](contracts-overview.md) — why contracts are central to GRACE (semantic shield; written by the AI-author; displaces classical documentation).
- [../03-contracts/contract-fields.md](contract-fields.md) — full field specification (PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS / RATIONALE).
- [../03-contracts/wenyan-prompting.md](wenyan-prompting.md) — compression technique used in KEYWORDS/LINKS.
- [../03-contracts/rationale-and-aag.md](rationale-and-aag.md) — architectural intent, the field that makes AI-contracts carry information classical DbC does not.
- [../03-contracts/legacy-contract-recovery.md](legacy-contract-recovery.md) — where JML-style formal recovery fits in AI-native workflow.
- [../03-contracts/language-specific/python.md](language-specific/python.md) — placement of AI-contracts relative to `def` in Python.
- [../01-transformer-foundations/llm-training-pipeline.md](../01-transformer-foundations/llm-training-pipeline.md) — four-phase training; why strict test-like conditions were purged.
- [../06-testing/classical-tests-antipattern.md](../06-testing/classical-tests-antipattern.md) — why putting tests next to contracts overfits, echoing the DbC overfit problem.
- [../06-testing/tests-in-self-correction-loop.md](../06-testing/tests-in-self-correction-loop.md) — where tests live in GRACE instead of in contracts.
- [../GLOSSARY.md](../GLOSSARY.md) — DbC, Hoare logic, JML, SDD, wenyan prompting, SFT.
