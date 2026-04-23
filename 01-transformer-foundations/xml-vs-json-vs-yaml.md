# XML vs JSON vs YAML — Format Choice for LLM Contexts

## What

GRACE deliberately prefers **XML-like markup** over JSON for structuring code, logs, contracts, and any payload that will be read by an LLM in a long context. YAML is explicitly avoided and flagged for conversion.

The choice is not stylistic. It is a layered decision grounded in:

1. A **vendor recommendation** from OpenAI's GPT-4.1 Prompting Guide, which explicitly tells prompt engineers not to use JSON in long contexts and to prefer XML-like delimiters instead [vendor-doc].
2. An **empirical observation** by the author — via logit analysis — that JSON induces semantic noise and slower convergence across models from multiple vendors, not just one [author-claim].
3. A **formal complexity-theoretic argument** that XML/HTML/JSON fit the Dyck Language class natively parsable by a transformer's TC⁰ logic, while YAML does not [research-backed].

Practical rule: use XML-like markup by default; convert YAML configuration to a Dyck-compatible format; for JSON, confine it to short, local fragments and never use it as the primary long-context container.

## Why

### The vendor evidence: OpenAI's GPT-4.1 Prompting Guide

OpenAI updated its GPT-4.1 Prompting Guide several months ago and included an explicit instruction: **do not use JSON in long context**. The guide recommends XML or XML-like markup of the kind Google's prompt engineers and GRACE already use. This is not author inference — it is the vendor's published guidance at `cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters` (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [vendor-doc].

The OpenAI guide references a paper by Lee et al. — **"Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?"** — as the underlying research supporting the recommendation. The sources do not specify the arxiv ID or venue beyond the title given (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [vendor-doc].

Before publishing this guidance, OpenAI had an earlier article with a more extensive empirical test on the same topic. The author's position: for most practitioners, it is simpler to follow the vendor's explicit recommendation than to relitigate it (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [author-claim].

### Google's alignment

Google's prompt engineers also use XML-like markup, and OpenAI's GPT-4.1 Prompting Guide has now officially joined that position (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [vendor-doc]. GRACE's formatting conventions were chosen to sit inside this already-established consensus, not against it [author-claim].

More narrowly for YAML: Google went a step further and **banned YAML libraries from Gemini's Code Execution sandbox**, disabling the relevant Python libraries so the LLM would not degrade while trying to apply YAML as a common format (post 3948) [vendor-doc].

### The empirical mechanics: why JSON degrades in long contexts

The author, using logit analysis on model outputs, observes concrete degradation modes when JSON appears in long contexts (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [author-claim]:

- **Brace counting across long spans.** The model effectively starts "counting braces" to maintain structural tracking. In a long context that budget is eaten by unrelated tokens, and the model loses the count.
- **False correlations from `{}`.** Curly braces "pull in" unrelated correlations — the same bracket tokens appear everywhere in the training distribution (code, configs, embedded expressions), so they are weak, noisy anchors for a specific structural role.
- **Slower convergence.** Compared to XML and XML-like markup, the model converges more slowly on semantics inside JSON payloads — more tokens are consumed resolving the structure before the content becomes usable.

The effect is **universal across vendors**, not a quirk of a single product. The author names Qwen as exhibiting the same pattern as OpenAI models, supporting the claim that this is a property of how transformers handle Dyck-3-style delimiters in long contexts rather than a vendor-specific training artifact (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [author-claim].

### The formal layer: Dyck Language and TC⁰

The empirical picture has a matching formal explanation (detailed in `tc0-and-dyck-language.md`, T07). Without chain-of-thought, transformers are limited to **TC⁰**, a discrete-logic class that **natively supports Dyck Languages** — paired-bracket languages formalized by Walter von Dyck. Shunyu Yao et al. proved native transformer support for Dyck Language (post 3948; arxiv.org/abs/2105.11115) [research-backed].

Implications for format choice (post 3948) [research-backed]:

- **Dyck-compatible formats:** XML, HTML, JSON, and bracket-based programming languages. These are the formats a transformer can parse in a single pass without chain-of-thought.
- **NOT Dyck-compatible:** YAML. Its indentation-based, unpaired structure falls outside what TC⁰ can natively disambiguate.

So why, if JSON is Dyck-compatible, does the vendor guidance still steer away from it in long context? The Dyck result establishes the **lower bound** — JSON is parseable in principle. The empirical degradation (brace counting, curly-brace noise, slower convergence) is a separate, practical layer that operates on top of formal parseability. XML-like markup wins at both layers: Dyck-compatible **and** empirically cleaner at scale. YAML fails the formal test outright.

### Why YAML is actively worse (not just suboptimal)

YAML is the one format that fails the Dyck test entirely. In a transformer's TC⁰ regime, YAML structures have no cleanly paired closing construct — indentation tracking is not a natively supported operation for a circuit-depth-bounded classifier (post 3948) [research-backed]. Google's decision to disable YAML libraries in Gemini's Code Execution sandbox is the operational consequence of that theoretical gap (post 3948) [vendor-doc].

The author's blunt observation on how YAML gets into AI pipelines: it is often inserted by "second-role" engineers hired by vendors who wire it into LLM-adjacent tools out of incompetence rather than because the format fits the task (post 3948) [author-claim]. The correct response when you find YAML in an LLM pipeline is to **convert it to a Dyck-compatible format**, not to try to make the LLM cope.

## How

### Default format by use case

| Use case | Preferred format | Why |
|---|---|---|
| Long-context structured payloads for LLM | XML-like markup | OpenAI + Google consensus; Dyck-compatible; empirically cleaner (xml_mapping_anchors.md; post 3948) |
| In-source code documentation (contracts) | XML-like or paired START-END tags in comments | Single-pass parseability (post 3948) |
| Short, local JSON fragments | JSON is acceptable | Degradation is a long-context phenomenon (xml_mapping_anchors.md) |
| Configuration files touched by an LLM | Convert YAML to XML, TOML-via-brackets, or JSON | YAML is not Dyck-compatible (post 3948) |
| Code in languages whose comment syntax isn't paired (e.g. Python `#`) | Add paired START-END tags inside comments | Makes comments Dyck-parseable in one pass (post 3948) |

### Concrete rules derived from the sources

1. **Do not use JSON as the primary container for long-context prompts.** Follow the vendor's explicit guidance (cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters) [vendor-doc].
2. **When you see JSON in a long-context RAG pipeline, question it.** The author frames this as a litmus test: "if someone in RAG is running around with JSON, it's time to ask whether that was a good choice" (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [author-claim].
3. **Convert existing YAML to a Dyck-compatible format.** YAML degrades the model; the cost of conversion is paid once, the cost of running YAML-on-LLM is paid forever (post 3948) [author-claim].
4. **Use XML-like markup rather than full XML.** GRACE prefers a lightweight XML-like syntax over verbose XML (see T21 `xml-like-markup.md`); the Dyck property is about paired delimiters, not angle-bracket specifics.
5. **In Python and other languages without natively paired comment syntax, add START-END tags inside comments** (see T22 `start-end-tags.md`). This is the concrete mechanism that lets GPT parse code + embedded documentation in a single Dyck-compatible pass (post 3948) [research-backed].

### What JSON-in-long-context looks like when it fails

The author's logit-analysis observations describe the failure mode qualitatively rather than with a published benchmark (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [author-claim]:

- The model starts "counting braces." Past a certain context length, the count drifts — closing braces get associated with the wrong openings, or the model loses track of which object it is inside.
- The ubiquity of `{}` in training data means curly braces pull attention toward weakly-related contexts (shell expansions, format strings, dictionary literals, set comprehensions), creating **false correlations** that compete with the structural signal the JSON is supposed to carry.
- Convergence on the underlying semantics is measurably slower relative to XML/XML-like markup — a direct observation the author attributes to noisier anchoring from the `{}` tokens.

These three effects — **brace counting**, **semantic noise from `{}`**, **slower convergence** — are the mechanistic story under the OpenAI recommendation, and they are what tie the empirical and formal layers together.

### What YAML looks like when it fails

YAML's failure mode is more categorical: the model cannot reliably parse it at all inside TC⁰ constraints because the format does not present paired closing constructs (post 3948) [research-backed]. Operationally this is visible as: the LLM produces structurally wrong YAML when asked to emit it, or silently misinterprets nested structure when reading it. Google's sandbox-level block is a cleaner solution than trying to harden a prompt to make YAML work (post 3948) [vendor-doc].

### Validating that a chosen format actually helps

The author's preferred empirical check for format choices is **logit analysis**: inspect the model's token probabilities on the structured payload to see whether the tokens that should be anchors (tag names, role markers) are in fact anchoring the right downstream predictions (xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку") [author-claim]. This is the same technique GRACE uses to validate compressed contract fields (see T16 `wenyan-prompting.md`, T25 `markup-compression.md`).

## Evidence

### Primary citations

- **xml_mapping_anchors.md §"Почему автор предпочитает XML-like разметку"** — central source. States the OpenAI GPT-4.1 Prompting Guide recommendation against JSON in long context; references the Lee paper *"Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?"* cited inside that guide; documents the author's logit-analysis observations of JSON degradation (brace counting, curly-brace noise, slower convergence); notes that the effect is universal across vendors (Qwen as named example); records the OpenAI + Google alignment on XML-like markup.
- **Post 3948** — formal basis. TC⁰ restriction without chain-of-thought; native Dyck Language support (Shunyu Yao et al., arxiv.org/abs/2105.11115); Dyck-compatible formats (XML, HTML, JSON, bracket-based languages); YAML is not Dyck-compatible; Google's ban of YAML libraries in Gemini's Code Execution sandbox; Python's need for paired START-END tags in comments; author's note about YAML being inserted "by incompetence" by second-role engineers at vendors.

### Vendor and research URLs (as given in sources)

- **OpenAI GPT-4.1 Prompting Guide (delimiters section)** — `cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters` (xml_mapping_anchors.md). Explicitly recommends XML-like over JSON in long context [vendor-doc].
- **Lee et al., "Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?"** — referenced from the OpenAI guide (xml_mapping_anchors.md). The sources do not specify the arxiv ID or venue beyond the title. [research-backed, indirect via vendor-doc]
- **Shunyu Yao et al. on Dyck Language in transformers** — `arxiv.org/abs/2105.11115` (post 3948). Native transformer Dyck Language support [research-backed].

### Claim-type summary

- OpenAI's recommendation against JSON in long context: [vendor-doc].
- Google's use of XML-like markup and ban of YAML in Gemini sandbox: [vendor-doc].
- Dyck Language class native to TC⁰ / transformers (Yao et al.): [research-backed].
- JSON degradation mechanics (brace counting, curly-brace noise, slower convergence): [author-claim] based on logit analysis.
- Universal cross-vendor effect (Qwen named alongside OpenAI-family models): [author-claim].
- YAML often inserted by "second-role" vendor engineers out of incompetence: [author-claim].

### What the sources do not specify

- Exact numeric thresholds for when JSON degrades (no context-length cutoff, no benchmark numbers tied to the author's logit analysis).
- The Lee paper's arxiv ID, authors beyond "Lee," or venue — only the title as quoted from the OpenAI guide.
- Specific Qwen model versions tested — the claim is that "Qwen has the same effects," not a specific SKU.
- A published side-by-side benchmark of XML vs JSON vs YAML under controlled conditions from the author — the evidence is logit-level inspection, not a public score.

Where these gaps exist, the sources do not specify.

## See also

- `../01-transformer-foundations/tc0-and-dyck-language.md` — T07, formal basis: TC⁰, Dyck Language, Yao et al., and the full list of Dyck-compatible vs non-compatible formats. The formal argument for XML preference lives there; this file focuses on the format-choice decision.
- `../01-transformer-foundations/sparse-attention-and-kv.md` — T06, why long contexts collapse without good anchoring. The brace-counting failure is specifically a long-context phenomenon, so the sparse-attention foundation is the relevant substrate.
- `../04-markup-system/xml-like-markup.md` — T21, rules for GRACE's lightweight XML-like markup.
- `../04-markup-system/start-end-tags.md` — T22, paired START/END tags as the Dyck-compatible mechanism inside languages (e.g. Python) whose comment syntax isn't natively paired.
- `../03-contracts/language-specific/python.md` — T19, concrete Python example of using START-END tags in comments to achieve Dyck compatibility (post 3948).
- `../13-antipatterns/all-antipatterns.md` — T59, catalog entries for "JSON in long context" and "YAML configuration" as antipatterns.
- `../14-research-references/annotated-bibliography.md` — T60, bibliographic entries for the OpenAI GPT-4.1 Prompting Guide, the Lee paper, and the Yao et al. Dyck Language paper.
