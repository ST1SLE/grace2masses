# Sparse Attention and the KV Cache

## What

Transformer attention is not uniform across context length. At a particular context size, the attention mechanism switches from the familiar **dense** mode (every token attends to every other token) to a **sparse** mode (each token sees only a small neighbor window). At the same time, the **KV cache** stores the key/value states computed for earlier tokens so they are not recomputed — which also means whatever the model decided early about the text is effectively frozen.

These two mechanics together explain why long code contexts "fall apart" for an LLM past a certain size, and why GRACE insists on an explicit semantic graph rather than trusting the model to hold a large codebase in mind. [author-claim]

Source: `most_important_output.md` §"Почему без семантического графа большие контексты разваливаются" and §"Граф как основа навигации для LLM".

## Why

Two root causes of collapse on long code:

1. **Dense attention is O(n²) in tokens and the matrix has to fit on a GPU.** The attention matrix grows as n² because every token is compared to every other token. Past a hardware-driven ceiling, the GPU cannot hold the full matrix, and the model switches to sparse attention. [research-backed — author paraphrasing transformer internals]

2. **Without anchors, sparse attention has nothing to stitch together.** A model operating in sparse mode sees only tiny neighbor windows. Without an explicit navigation structure (a semantic graph), it tries to fall back to **random attention** — described by the author as inefficient and little-used in modern GPT. [author-claim]

On top of that, the KV cache makes the problem stickier: whatever wrong turn the model took early in the context gets cached and becomes hard to override. The author's phrasing is that GPT "becomes stubborn as a donkey" when a wrong early branch is frozen. [author-claim] (`most_important_output.md` §"Граф как основа навигации").

GRACE uses this as a core justification for the methodology: if you do not provide an explicit graph of anchors, the model will build one implicitly, often wrongly, and then KV-cache-freeze the wrong version.

## How

### The numbers the sources actually give

The following are the specific numbers stated in the sources — nothing more is invented:

- **Dense attention matrix size**: grows as n² (every token compared to every token). [research-backed]
- **Dense capacity on Nvidia GPU**: "around 4k tokens fits; about 16 million values." Quote from `most_important_output.md` §"Почему без семантического графа": *"В процессор Nvidia влезает максимум 4к токенов такая матрица (16 млн. значений)."* [author-claim / research-backed — author paraphrasing hardware limit; sources do not specify which Nvidia model, which attention-head layout, or exactly how 4096² ≈ 16.8M is counted]
- **Past 4k tokens, sparse attention kicks in**, using small **sliding windows of ~100–200 neighbor tokens** around each current token. [research-backed — author paraphrasing transformer internals] (`most_important_output.md`)
- **Fallback when no explicit graph exists**: **random attention** between blocks. Author states this is inefficient and used less in modern GPT. [author-claim] (`most_important_output.md`)
- **Empirical log-attention collapse**: in experiments with Gemini on unmarked logs, the author reports visible attention drop-outs past about **5000 lines** even in a 1M-token context window. Without log-to-code correlation tokens the model "just cannot correctly grasp the context of code and logs together and misses errors." [empirical] (post 3693)

### What the sources do NOT specify

- Exact GPU model, generation, or VRAM number behind the "4k tokens ≈ 16M values" figure. Sources do not specify.
- Whether "4k" is a per-head or per-attention-layer boundary. Sources do not specify.
- The precise algorithm used to select neighbors in the 100–200-token sliding window. Sources do not specify.
- Exact mechanics of KV-cache mutation or invalidation. Sources do not specify.

Do not invent these.

### The "table of contents" analogy

The author's operational framing: the top of a semantic graph acts as a **table of contents** ("оглавление книги") that sparse attention uses to navigate. The 100–200-token windows, stitched through the anchor nodes of that graph, let the model move around a large text without needing to hold all of it in dense attention at once. (`most_important_output.md` §"Почему без семантического графа"). [author-claim]

If the anchors are not there, the model either:

- Falls back to random attention (inefficient; rare in modern models). [author-claim]
- Builds its own anchor graph implicitly — usually wrong, because of causal-read truncation. [author-claim] (`most_important_output.md` §"Граф как основа навигации")
- KV-caches that wrong implicit graph and becomes stuck on it. [author-claim]

### KV cache as stubbornness

From `most_important_output.md` §"Граф как основа навигации":

> "If you do not make a graph, GPT still makes one anyway, but due to causal reading [not reading to the end of your content] it easily grows a 'crooked branch' of the graph and… freezes it in KV cache, then becomes stubborn as a donkey about some nonsense."

The practical consequence for GRACE:

- **Build the graph explicitly, early, at the top of the context**, so the model's implicit graph-building converges on the right structure before KV-cache locks it in. [author-claim]
- **Anchor every downstream reference back to that graph** (keywords, links, contract fields), so sparse-attention windows all contain tokens that point back to the shared structure. [author-claim] Cross-link: `../03-contracts/contract-fields.md`.

### Log-attention collapse past ~5000 lines

From post 3693:

> "I did experiments with Gemini on logs — even after 5000 lines his attention shows drop-outs without such tags, and he simply cannot correctly grasp the context of code and logs together and misses errors."

This is the direct empirical motivation for **log-to-code correlation** (injecting function-ID and block-ID tokens into every log line) and for **forced-context injection** of important log lines — covered in the logging section. [empirical] Cross-link: `../05-logging-ldd/log-to-code-correlation.md` and `../05-logging-ldd/forced-context-injection.md`.

### Implications for GRACE workflow

All of the following are author-claims derived from the two mechanics above:

- **Keep the top of the context dense with the graph.** In a GRACE session, the first tokens the model sees should include the semantic graph or its projection into the current file — because those are the tokens that will still be in "dense-attention reach" when everything else slides into sparse mode. [author-claim]
- **Rely on LINKS/KEYWORDS as sparse-attention hooks.** `~100–200`-token sliding windows need something semantically loaded *inside* them; LINKS and KEYWORDS tokens are designed to be hit by those windows and pull back the graph structure. Cross-link: `../03-contracts/wenyan-prompting.md`.
- **Never assume the agent will rebuild the structure on-demand via Tools.** The agent's sparse-attention state is already shaped by whatever ran first; forcing a Tool call later does not undo KV-cache anchoring on an early wrong branch. Cross-link: `../07-autonomous-agents/forced-context.md`. [author-claim]

### Scale at which this starts to matter

The author gives an empirical "critical mass" threshold (post 3654):

- **~5,000–10,000 lines of interconnected logic** is where AI loses understanding. This is NOT necessarily the whole application — can be a "noodle-code" pocket inside a much larger (100k-LOC) codebase.
- Below that, small pet-project habits work by accident because the whole thing fits inside dense attention. Above it, sparse attention + missing anchors kick in and drift begins. [empirical] (post 3654)

Cross-link: `../00-foundations/why-grace-exists.md` develops this further.

## Evidence

- **`most_important_output.md` §"Почему без семантического графа большие контексты разваливаются"** — the canonical source for:
  - n² attention matrix scaling,
  - "Nvidia GPU fits at most 4k tokens of this matrix, 16 million values,"
  - sparse attention past 4k with 100–200-token sliding windows,
  - semantic graph as "table of contents" stitching the windows,
  - random attention as inefficient fallback.
  [research-backed where the transformer mechanics are concerned; author-claim for the specific framing of "table of contents" and efficiency of random attention]

- **`most_important_output.md` §"Граф как основа навигации для LLM"** — the canonical source for:
  - graphs as emergent property of attention heads (correlations = edges),
  - "if you do not build the graph, GPT builds one anyway, but crooked, and freezes it in KV cache — stubborn as a donkey,"
  - top of the graph = anchor system for sparse-attention navigation past 4k tokens.
  [author-claim]

- **post 3693** — empirical evidence for attention collapse on long **logs**:
  - without log-to-code correlation tokens, Gemini's attention visibly drops out past ~5000 log lines even at 1M-token context,
  - motivation for injecting function-ID and block-ID tokens into every log line,
  - belief-state logging as a related mechanism.
  [empirical, author-conducted experiment]

- **post 3654** — empirical "critical mass" of **~5,000–10,000 LOC of interconnected logic** at which AI understanding collapses without explicit semantics. [empirical]

### What is NOT in the sources (do not invent)

- No specific GPU model or VRAM figure beyond "Nvidia" and "4k tokens / 16M values."
- No paper URL for the sparse-attention / sliding-window claim; the author treats it as well-known transformer mechanics.
- No per-vendor breakdown of where the 4k boundary sits on current production models.
- No formal measurement of "random attention is inefficient in modern GPT" — stated as author observation.

## See also

- `../00-foundations/why-grace-exists.md` — the 5–10k LOC critical-mass threshold and why vibe-coding stalls.
- `../01-transformer-foundations/tc0-and-dyck-language.md` — formal reason XML-like, paired-tag structure is natively parsed by the same model inside sparse attention.
- `../01-transformer-foundations/llm-training-pipeline.md` — why the model will not bail itself out via an external-docs MCP past the sparse-attention boundary.
- `../01-transformer-foundations/xml-vs-json-vs-yaml.md` — format-level consequences of sparse attention + Dyck compatibility.
- `../02-semantic-graph/graph-as-backbone.md` — how the explicit graph solves the navigation problem sparse attention creates.
- `../02-semantic-graph/hierarchy-and-anchors.md` — TZ → contracts → code hierarchy as a ready-made anchor tree.
- `../03-contracts/contract-fields.md` — PURPOSE / KEYWORDS / LINKS as the per-function hooks that keep sparse-attention windows pointing back to the graph.
- `../03-contracts/wenyan-prompting.md` — why compressed keywords survive sparse attention better than long prose.
- `../05-logging-ldd/log-to-code-correlation.md` — the archimportant heuristic for binding log lines back to code under sparse attention (post 3693).
- `../05-logging-ldd/forced-context-injection.md` — why waiting for the agent to self-query is unreliable once sparse attention has set in.
- `../07-autonomous-agents/forced-context.md` — forced context as a stabilization mechanism against wrong KV-cache branches.
