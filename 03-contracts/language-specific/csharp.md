# C# XML Documentation — Mapping to GRACE

## What

**C# XML Documentation** is the standard in-source documentation format for the C# ecosystem: structured comments (typically `///`-prefixed) with XML tags such as `<summary>`, `<param>`, `<returns>`, placed directly above type and member declarations and consumed by compilers, IDEs, and documentation generators.

This file describes how GRACE's contract model maps onto that standard and what a GRACE-flavored C# contract looks like.

The author's stance, in one line: **C# XML Documentation has very high compatibility with GRACE-style semantic markup for AI agents** [author-claim] (post 3960).

The adaptations are minor because the host format already gives GRACE most of what it wants (paired tags, IDE support, LLM familiarity). The main changes are **what goes inside** the tags — trading the classical free-prose `<summary>` for purpose-oriented, anchor-rich fields — not the tag layer itself.

## Why

Four reasons C# XML Documentation is a natural fit for GRACE [author-claim] (post 3960):

1. **Paired tags for sparse attention.** C#'s `#region` / `#endregion` directives work as paired START-END tags. GRACE relies on paired START-END tags for two overlapping reasons covered elsewhere in the KB: Dyck-Language compatibility (one-pass transformer parsing) and sparse-attention navigation past ~4k tokens (`../../04-markup-system/start-end-tags.md`, `../../01-transformer-foundations/tc0-and-dyck-language.md`). `#region` gives you those paired tags natively in C#, without a convention you have to invent.

2. **IDE integration and documentation generators work out of the box.** XML Doc is built into the C# toolchain — IDEs surface it inline, and automatic documentation generators already consume it. The author specifically notes this is **useful for search agents** (post 3960). An LLM-based agent reading the code indirectly benefits from the same metadata pipeline that powers IntelliSense and generated reference docs.

3. **High generation stability.** Because XML Doc is a long-standing standard, LLMs have seen a great deal of it during training — so **the format is familiar to the model** and generation is stable (post 3960). GRACE-adjacent content produced on top of that stable scaffold inherits the stability.

4. **GRACE's wider preference for XML-like markup already lives here.** OpenAI's GPT-4.1 Prompting Guide advises against JSON in long context and recommends XML-like delimiters (`../../01-transformer-foundations/xml-vs-json-vs-yaml.md`); the TC⁰/Dyck argument proves XML is natively parseable by a transformer in one pass (`../../01-transformer-foundations/tc0-and-dyck-language.md`). C# XML Doc is literally XML inside source — it sits inside the same family of formats GRACE already prefers globally.

What GRACE does *not* simply inherit is the **content model** of classical XML Doc — specifically the `<summary>` tag (see below).

## How

### The `<summary>` tag is sub-optimal for AI

Classical XML Doc — and most scientific work on in-source documentation — puts the load on a free-prose `<summary>` element that describes what the function does. The author's direct observation [author-claim] (post 3960):

> The usual `<summary>` as written in C# or as described in docstring research is not the most optimal for AI. If you shorten it to one line, nothing changes for AI. AI reads code fine.

Two points pack into that:

- **AI already reads code.** A long prose restatement of what the code does duplicates information the model can recover from the code itself. This is the same "Captain Obvious" effect documented from a separate paper in `../rationale-and-aag.md` and post 3890: restating what the code already says is useless and can be harmful.
- **Length does not buy anything.** A one-line `<summary>` is empirically as good as a long one for AI consumption (post 3960). This matches the separate ShortenDoc and in-source-documentation benchmarks summarised in `../wenyan-prompting.md` and `../../04-markup-system/markup-compression.md`: shorter is neutral or better, longer is neutral or harmful (posts 3889, 3892).

The implication for C#: **keep `<summary>` short — or replace its role with richer anchors below.**

### The richer anchors GRACE wants instead

Per post 3960, the AI agent gets more semantic value from:

- **PURPOSE** — goal/intent of the member, the alignment anchor for the model's Purpose Breakdown. Covered in depth in `../contract-fields.md` and posts 3878, 3889: PURPOSE is the *critical* field; agents can skip DESCRIPTION entirely but degrade without PURPOSE.
- **KEYWORDS** — wenyan-compressed architectural / task-classification tags that hook into RAG and attention correlations. Covered in `../wenyan-prompting.md` and posts 3819, 3892.
- **LINKS** — cross-module references that act as a Call Graph proxy for semantic discovery. Covered in `../contract-fields.md` and `../../02-semantic-graph/graph-rag-vs-vector-rag.md`.

Direct quote from the source, paraphrased [author-claim] (post 3960):

> For AI, purpose-setting and things like KEYWORDS and LINKS are more important — for catching correlations. They are fewer tokens than ordinary prose, but semantically much richer for AI.

"Fewer tokens, higher semantic value" is the operative trade. This is exactly the wenyan-prompting compression discipline (`../wenyan-prompting.md`) applied inside the XML Doc frame.

### `#region` as paired START-END tags

`#region NAME` / `#endregion` bracket a contiguous code block with an explicit opener and closer. From a Dyck-Language standpoint (`../../01-transformer-foundations/tc0-and-dyck-language.md`), that is exactly what the transformer needs to parse the block structure in one pass without drifting. For sparse attention past ~4k tokens (`../../01-transformer-foundations/sparse-attention-and-kv.md`), those named region boundaries also serve as navigation anchors.

Implication: **use `#region` actively in larger C# files** — not as cosmetic collapsing for human readers, but as the paired-tag infrastructure that lets the AI both parse and navigate. This mirrors GRACE's approach in Python, where the `#`-only comment syntax is not Dyck-friendly on its own and requires explicit paired START-END tags inside comments (post 3948, `./python.md`); C# already ships the primitive natively.

### IDE / doc-generator integration helps search agents

Post 3960 explicitly calls out that "the markup is immediately embedded into the IDE and is supported by automatic documentation generation tools, which is useful for search agents" [author-claim]. The point is operational rather than formal:

- An agent that uses IDE services, LSP, or Roslyn-backed tooling can query structured metadata directly, not just raw text.
- An agent that reads generated reference documentation sees the same fields already extracted and indexed.
- This reduces the number of places the agent has to guess or heuristically parse.

GRACE does not require this plumbing — but where it exists (as it does in C#), it shortens the path from "agent has a question" to "agent has the right anchor". Compare this to GRACE's concerns about custom MCPs and DIY tooling that the model was never trained on (`../../09-tooling/mcp-scepticism.md`, post 3834): XML Doc is the opposite — it is the mainstream, trained-on surface.

### Generation stability

The author notes "high stability of markup generation, because this is a familiar standard for AI" [author-claim] (post 3960). In practice this means:

- When you ask an LLM to produce a C# function with XML Doc comments, it will almost always emit well-formed `///` blocks with the standard tags (`<summary>`, `<param>`, `<returns>`, etc.).
- Layering GRACE content (PURPOSE / KEYWORDS / LINKS) on top of that stable scaffold is far easier than inventing a bespoke comment format, because the model is pattern-matching to training-distribution examples for the outer structure.

This is the same principle argued in `../ai-contracts-vs-dbc.md` (post "Почему AI-contracts не равны классическому DbC") — sit on the patterns the AI already knows rather than invent your own.

### Placement rule carries over

The general GRACE placement rule from post 3878 and `../contract-fields.md` is that the description goes **before** the declaration, because causal-read anchor vectors form around the name token and need preceding context. In C#, XML Doc is already placed above the declaration by standard convention — so the placement rule is satisfied by the native format. No deviation required, unlike Python where the standard docstring sits *inside* the function and GRACE deliberately moves the richer content to `#` comments *above* the `def` (see `./python.md`, post 3878).

### Minimal GRACE-flavored C# contract sketch

The sources do not give a verbatim C# example (unlike the Python example reproduced in post 3878 and quoted in `../contract-fields.md` / `./python.md`). The pattern below is a reconstruction from the rules stated in post 3960 only — `<summary>` kept short, richer anchors carried in additional tags, `#region` used as paired START-END:

```csharp
#region Saver

/// <summary>Saves config dict to JSON file.</summary>
/// <purpose>Persist parameters to JSON so the app can reload state on restart.</purpose>
/// <keywords>Saver, FileWrite</keywords>
/// <links>ConfigLoader.LoadConfig</links>
/// <param name="config">Dictionary of parameters.</param>
/// <param name="configPath">Target file path (default "config.json").</param>
/// <returns>True on success; false on IO failure.</returns>
public bool SaveConfig(IDictionary<string, object> config, string configPath = "config.json")
{
    // ...
}

#endregion
```

Notes on the sketch:

- `<summary>` is one line, per post 3960's guidance.
- `<purpose>`, `<keywords>`, `<links>` are **custom XML Doc tags**. XML Doc permits custom tags — the toolchain will pass them through even if it does not render them by default. This carries the PURPOSE / KEYWORDS / LINKS content defined in `../contract-fields.md` (post 3878) into the C# format.
- `#region Saver` / `#endregion` mirrors the KEYWORD and gives a paired, named boundary the agent can navigate to.
- `<param>` and `<returns>` are kept because they are the canonical mapping of INPUTS/OUTPUTS into C# XML Doc; the content inside them follows the "short and purpose-oriented" discipline from post 3960.
- **The sources do not specify** which exact tag names GRACE prescribes for C#; `<purpose>`, `<keywords>`, `<links>` are used here as reasonable transliterations of the Python field names defined in post 3878. If a project standard prefers a different spelling (e.g. `<grace-purpose>`), the reasoning from post 3960 is unchanged — the load-bearing part is the *content*, not the tag spelling.

### What to drop or minimise

Consistent with post 3960 and the broader in-source-documentation findings from posts 3889 and 3892 (`../../04-markup-system/markup-compression.md`):

- Long narrative `<summary>` blocks → shorten to one line or drop.
- "Captain Obvious" prose that restates what the code already says → drop (post 3890, `../rationale-and-aag.md`).
- Non-English `<summary>` text → prefer English, per the benchmarks in post 3889 that show degradation on non-English in-source docs.
- Free-text long descriptions that are not tied to an anchor role (PURPOSE / KEYWORDS / LINKS) → drop or compress.

What to keep and enrich:

- `#region` boundaries around logical code units.
- Short `<summary>` plus purpose/keywords/links anchors.
- Input/output documentation through `<param>` / `<returns>` with concise, purpose-oriented text.

## Evidence

### Primary source

- **post 3960** (2026-04-21) — the sole post in the source corpus on C#-specific mapping. All four top-level claims in this file trace directly to post 3960:
  1. C# XML Documentation has high GRACE compatibility.
  2. `#region` serves as paired START-END tags useful for sparse attention.
  3. IDE integration and automatic doc generation aid search agents; the format is familiar to LLMs so generation is stable.
  4. `<summary>` is sub-optimal for AI (one-line equivalent to long); PURPOSE / KEYWORDS / LINKS give fewer tokens with higher semantic value.

### Supporting facts imported from cross-cutting posts

These are not C#-specific but underpin the reasoning in this file:

- **post 3878** — canonical contract fields (PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS) and the placement-before-declaration rule; the fields GRACE wants C# XML Doc to carry. See `../contract-fields.md`.
- **post 3889** — in-source documentation benchmarks (arxiv.org/abs/2601.16661v1): PURPOSE critical, shorter better, English better, adjacency matters. See `../../04-markup-system/markup-compression.md`.
- **post 3890** — "Captain Obvious" comments are useless or harmful; architectural intent → 3× bug-fix accuracy (researchgate.net/publication/400340807). See `../rationale-and-aag.md`.
- **post 3892** — ShortenDoc (dl.acm.org/doi/10.1145/3735636): 40% docstring compression without quality loss, +~10% HumanEval on "mutilated" keyword-only text. See `../wenyan-prompting.md`.
- **post 3948** — TC⁰ / Dyck Language / arxiv.org/abs/2105.11115: formal substrate for why paired tags (including `#region`) are Dyck-compatible and parseable in one pass. See `../../01-transformer-foundations/tc0-and-dyck-language.md`.
- **post 3819** — wenyan prompting: KEYWORDS/LINKS as compressed semantic hooks. See `../wenyan-prompting.md`.

### External references

- The sources do not specify a URL for C# XML Documentation itself or for Roslyn / IDE integration; the claims about IDE integration and doc generation come from post 3960's general statement without a cited link.

### What the sources do not specify

- An exact verbatim GRACE-for-C# code template. Post 3960 describes the mapping in prose; it does not reproduce a full example. The sketch in the "How" section is a reconstruction grounded strictly in the rules given.
- Specific XML Doc tag names GRACE prescribes for purpose / keywords / links in C#.
- Whether GRACE mandates one `#region` per member, per logical block, or per contract — the post notes `#region` *works* as a paired START-END but does not prescribe granularity.
- Interaction with `/// <inheritdoc/>` and other advanced XML Doc tags.

## See also

- `../contracts-overview.md` — why contracts are central to GRACE.
- `../ai-contracts-vs-dbc.md` — AI-contracts are not classical Design by Contract; sit on patterns the AI already knows (applies directly to using XML Doc as the carrier).
- `../contract-fields.md` — canonical PURPOSE / INPUTS / OUTPUTS / KEYWORDS / LINKS fields; the content model carried into the C# XML Doc tags above.
- `../wenyan-prompting.md` — compression discipline for KEYWORDS and LINKS.
- `../rationale-and-aag.md` — architectural intent; the 3× bug-fix result and the "Captain Obvious" antipattern.
- `./python.md` — Python contract placement, for contrast: Python needs GRACE to move content above `def` and add START-END tags; C# gets the placement and paired-tag machinery for free.
- `../../01-transformer-foundations/tc0-and-dyck-language.md` — formal substrate; why paired `#region`/`#endregion` is Dyck-compatible.
- `../../01-transformer-foundations/sparse-attention-and-kv.md` — why paired anchors matter past ~4k tokens.
- `../../01-transformer-foundations/xml-vs-json-vs-yaml.md` — format-choice grounding; XML-family preference that C# XML Doc naturally inherits.
- `../../04-markup-system/xml-like-markup.md` — general GRACE XML-like markup rules.
- `../../04-markup-system/start-end-tags.md` — paired-tag discipline; `#region` is the C# native instance.
- `../../04-markup-system/in-source-documentation.md` — enterprise lineage of GRACE-style markup; C# XML Doc is a mature mainstream instance of in-source documentation.
- `../../04-markup-system/markup-compression.md` — empirical bounds on description length; why short `<summary>` is as good as long.
- `../../09-tooling/mcp-scepticism.md` — contrast: C# XML Doc is mainstream-trained, not a custom MCP; the model is on familiar ground.
- `../../13-antipatterns/all-antipatterns.md` — long summary-style descriptions, Captain-Obvious comments, non-English in-source text.
- `../../14-research-references/annotated-bibliography.md` — full entries for the research papers cited above.
