# TC⁰ and Dyck Language — Formal Basis for XML Preference

## What

Two formal concepts from complexity theory and formal language theory together explain why GRACE — and modern LLM practice in general — strongly prefers **paired-bracket data formats (XML, HTML, JSON, bracket-based programming languages)** over non-bracketed formats like **YAML**:

- **TC⁰** (Threshold Circuit class 0): the complexity class of discrete logic accessible to a transformer without chain-of-thought (CoT) reasoning [research-backed] (post 3948).
- **Dyck Language** (named after German mathematician **Walter von Dyck**): the family of formal languages built from matched pairs of brackets. A Dyck-3 string uses three kinds of brackets and requires every opening bracket to have a corresponding closing bracket (post 3948).

Walter von Dyck showed that algorithms without loops — the same shape of computation that TC⁰ captures — can interpret these "paired-bracket languages" [research-backed] (post 3948). Shunyu Yao et al. later proved the specific result for transformer networks: transformers natively support Dyck Language parsing, published at **arxiv.org/abs/2105.11115** [research-backed] (post 3948).

Put together: a transformer doing one forward pass (no CoT) can natively parse any Dyck-compatible format. It cannot natively parse formats that are not Dyck-compatible — such as YAML, which relies on indentation rather than explicit paired delimiters.

## Why

GRACE treats this as the **formal substrate** under every format-selection decision made in the methodology:

1. **Native = reliable without tricks.** When data is Dyck-compatible, a single transformer pass can determine the structure. When it is not, the model has to rely on heuristics that are fragile in long contexts, producing hallucinations, drift, and silent corruption [author-claim] (post 3948).
2. **Justifies the XML preference empirically observed elsewhere.** Other nodes in the GRACE KB (`../04-markup-system/xml-like-markup.md`, `./xml-vs-json-vs-yaml.md`) document OpenAI's and Google's public preference for XML-like delimiters over JSON in long context — `./xml-vs-json-vs-yaml.md` covers the vendor-doc side; this file gives the **formal "why"** underneath it: the format either lives in the Dyck family or it does not [research-backed] (post 3948).
3. **Makes YAML's problems a matter of form, not taste.** YAML is not a paired-bracket format, so it sits outside what TC⁰ / native transformer decoding can cleanly parse. Google responded to this by **banning YAML libraries inside Gemini's Code Execution sandbox** — specifically disabling Python YAML libraries so the LLM would not degrade while trying to use this common format [vendor-doc] (post 3948). A vendor disabling a library is a strong signal about the underlying constraint.
4. **Explains the GRACE-specific Python pattern.** Python's comment syntax (`#`) is line-based and not inherently Dyck-friendly. GRACE adds **paired START-END tags inside comments** so the comment block becomes Dyck-compatible and GPT can parse it in a single pass without degradation [author-claim] (post 3948). See `../04-markup-system/start-end-tags.md` and `../03-contracts/language-specific/python.md`.

Without this foundation, "use XML instead of JSON" and "do not use YAML" are just vendor folklore. With it, they are consequences of a proof about what a no-CoT transformer can and cannot decide in one pass.

## How

### The formal chain

1. **TC⁰ is the circuit-complexity class of a transformer without chain-of-thought.** The author summarizes this as "GPT without CoT has access only to discrete logic of class TC⁰" [research-backed] (post 3948). The post references this as mathematically established in the transformer-theory literature; the sources do not specify the primary citation for the TC⁰ result beyond Yao et al. on Dyck support.
2. **TC⁰ supports Dyck Languages.** Mathematically proved: TC⁰ can interpret "paired-bracket languages" in the sense of von Dyck [research-backed] (post 3948).
3. **For transformers specifically:** Shunyu Yao et al., **arxiv.org/abs/2105.11115**, proved native transformer support for Dyck Language [research-backed] (post 3948).
4. **Consequence:** formats that are Dyck-compatible are parseable by a transformer in a single forward pass; formats that are not are not (post 3948).

### The Dyck-3 canonical example

The canonical illustration of Dyck-3 — three kinds of brackets (round, square, curly) each with explicit opening and closing delimiters — reproduced exactly as given in post 3948:

```
( [ [ ] { } ] ( ) { ( ) } ) [ ]
```

Every opener has a matching closer; the structure is fully balanced. This is the kind of structure a transformer can parse natively. A Dyck language strictly requires the closing paired construct — without it, the string falls outside what TC⁰ can compute [research-backed] (post 3948).

### Format classification

From post 3948 [research-backed] + [vendor-doc]:

| Format | Dyck-compatible? | GRACE stance |
|---|---|---|
| **XML** | Yes (paired `<tag>…</tag>`) | Preferred for long contexts; cross-links to `./xml-vs-json-vs-yaml.md` and `../04-markup-system/xml-like-markup.md`. |
| **HTML** | Yes | Native to GPT (post 3948). |
| **JSON** | Yes (paired `{}` and `[]`) | Dyck-compatible in form, but OpenAI still advises against JSON in long context for separate reasons (brace-counting over long spans, false correlations from `{}`) — see `./xml-vs-json-vs-yaml.md`. Short-context JSON remains fine. |
| **Bracket-based programming languages** | Yes | Native (post 3948). |
| **YAML** | **No** | Not Dyck-compatible; Google **banned YAML libraries inside Gemini's Code Execution sandbox** to prevent degradation (post 3948). Recommended: convert stored YAML to a Dyck-compatible format. |
| **Python comments on their own** | **No** | Line-based `#`, not paired. Solution: add paired **START-END tags inside the comments** — GRACE's specific pattern. The result is Dyck-compatible and can be parsed in one pass without degradation (post 3948). |

### Important qualification

Post 3948 explicitly separates **two effects**:

- **The Dyck/TC⁰ effect** — whether a single forward pass can decide structure **even on a small context**. This is what this file is about.
- **The sparse-attention effect** — what happens past ~4k tokens when attention becomes sparse and needs navigation anchors. Covered in `./sparse-attention-and-kv.md` and `../04-markup-system/start-end-tags.md`.

Quote of the structural point from post 3948: "This does not consider the sparse-attention question, where XML comes out the winner, but even on a small context only Dyck-Language-compatible structures can be easily parsed by GPT" (paraphrased from the Russian original) [research-backed].

So even if sparse attention were not a factor, YAML would still be worse than XML for a single-pass decision — and it gets worse again once sparse attention kicks in. The two effects compound.

### Why "YAML is not Dyck-compatible" is not hand-waving

A Dyck language's defining feature is that every opening construct must have an explicit closing construct [research-backed] (post 3948). YAML's structure is encoded primarily by **indentation** and **line-start markers** (`-`, `key:`), with no explicit "close this block" delimiter. A transformer doing discrete logic in TC⁰ therefore cannot resolve YAML's block structure via the same pairing mechanism it uses for `<tag>…</tag>`, `{…}`, `[…]`, or `(…)`. Google's sandbox ban is a direct consequence [vendor-doc] (post 3948).

### Why Python needs START-END tags to survive in GRACE

In Python, a block of in-source documentation might span many `#`-prefixed lines before `def`. Each `#` line is independent; there is no inherent pairing that tells the model "this whole block is one semantic unit". Without pairing, a transformer treating this as Dyck structure fails — no closers to match [author-claim] (post 3948).

GRACE's fix is simple: wrap the contract block in paired **START** and **END** comment markers. The moment there is an explicit opener and closer, the block becomes a Dyck construct and TC⁰ can parse it in one pass [author-claim] (post 3948). This is the formal justification for the START-END tag convention documented in `../04-markup-system/start-end-tags.md`.

### Practical conversion rule

From post 3948 [author-claim]:

> If you have something stored in YAML, I recommend converting it to Dyck-Language-compatible formats. YAML is often inserted even into LLM agents by engineers hired for second-string roles at AI vendors, who plug it in out of incompetence.

The second sentence is the author's editorial, but the conversion recommendation is grounded in the TC⁰/Dyck argument above.

## Evidence

### Primary source

- **post 3948** (2026-04-19) — "TC⁰ and Dyck Language — why XML and JSON are more native for GPT than YAML". The full formal argument, the Dyck-3 example, the Yao et al. reference, and the Gemini YAML ban all come from this post. All claims in this file trace back to post 3948 unless noted.

### External research cited in the source

- **Shunyu Yao et al.**, **arxiv.org/abs/2105.11115** — native transformer Dyck-Language support. Referenced directly at the bottom of post 3948. [research-backed]
- The TC⁰ result for transformers without CoT is stated as mathematically established; **the sources do not specify** a separate arxiv ID or author for the TC⁰-without-CoT result beyond the Yao et al. paper, so this file does not attach one.

### Vendor data point

- Google's Gemini **Code Execution sandbox** disables Python YAML libraries to prevent LLM degradation on YAML [vendor-doc] (post 3948). Post 3948 does not specify a Google documentation URL for this ban.

### Cross-file confirmations

- `xml_mapping_anchors_output.md` §"Почему автор предпочитает XML-like разметку вместо JSON" — OpenAI's GPT-4.1 Prompting Guide advising against JSON in long context, referencing the Lee paper *"Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?"* This is the vendor-doc layer that complements the Dyck-language formal layer in post 3948.
- `most_important_output.md` §"Почему без семантического графа большие контексты разваливаются" — the sparse-attention context (~4k tokens, sliding windows) that post 3948 explicitly says is a **separate** concern from the Dyck argument. Both effects compound in long contexts.

## See also

- `./sparse-attention-and-kv.md` — the post-4k sparse-attention effect; the second axis that favors XML on long contexts.
- `./xml-vs-json-vs-yaml.md` — empirical and vendor-doc layer on format choice; cites OpenAI's GPT-4.1 Prompting Guide and Google's XML endorsement. This file's Dyck argument is the **formal** companion.
- `./llm-training-pipeline.md` — why AI ignores external docs; relevant to why DIY/YAML-heavy config systems underperform.
- `../04-markup-system/xml-like-markup.md` — concrete rules for GRACE XML-like markup.
- `../04-markup-system/start-end-tags.md` — paired tag convention; directly implements the Dyck-compatibility requirement for Python and other comment-based languages.
- `../03-contracts/language-specific/python.md` — Python-specific placement of START-END tags around contract blocks.
- `../03-contracts/language-specific/csharp.md` — C# XML Doc and `region` directives as paired START-END tags.
- `../13-antipatterns/all-antipatterns.md` — YAML configuration and JSON-in-long-context antipattern entries.
- `../14-research-references/annotated-bibliography.md` — full annotation for arxiv.org/abs/2105.11115 (Yao et al.).
