# MCP Scepticism — When MCPs Work and When They Don't

## What

MCP (Model Context Protocol) servers are tool interfaces that LLM agents call to fetch information or trigger side effects. Agents do not invoke MCPs reliably by default: whether a given MCP will be used, ignored, or hallucinated around depends almost entirely on whether the target model saw that MCP (or close analogs) during training. [empirical] (post 3834)

GRACE's position is sceptical by default: custom, home-grown MCPs ("kulibin" tools) are an antipattern unless they are narrow, strongly-typed around GRACE's own markup, and serve a navigation task the agent already wants to perform. [author-claim] (posts 3821, 3834)

## Why

The reason is rooted in how LLM coding models are trained. GRACE treats the four-phase pipeline (base coder → FIM → SFT → RL) as the ground truth of what an agent is able and willing to do; anything outside that distribution is a fight against a "$10 billion training supercluster that worked for a month". [author-claim] (post 3821)

Three specific mismatches explain the failure mode:

1. **Agents are not trained on custom MCPs.** On SFT the model is taught tool-chains for the most popular agents and MCPs, with "a total ban on DIY". Custom MCPs live outside that distribution. [empirical] (post 3820)
2. **Agents can silently refuse.** Unlike bash (which is in the standard toolset and "works like a rock"), an agent can just not invoke an MCP it was never trained on, or hallucinate around it. The author tried a "kulibin CLI" next to pytest in his own GRACE autotest harness; the LLM either ignored it or hallucinated around it, so he removed it. [empirical] (post 3834)
3. **The framing is wrong.** Doc-serving MCPs assume the agent's primary task is "understand this class". FIM training trained the agent toward a different primary task: "find modules" — load modules related to the task into context, just as training data did. A doc-MCP tries to solve a task the agent does not actually have. [author-claim] (post 3821)

A further structural reason: base FIM pretraining explicitly banned documentation in the training loop; the model was rewarded for evaluating code against surrounding code, not against prose description. An external doc-service system asks the agent to behave against its training distribution. [empirical] (post 3820)

## How

### The default rule

Do not ship DIY MCPs for general use. Prefer CLI wrappers over standard tools (bash, pytest, curl) and in-source documentation embedded in the code the agent already reads. [author-claim] (posts 3820, 3834)

Quote: "AI does not like kulibin work in Tools or MCP, because it simply wasn't trained on it." [empirical] (post 3834)

### Exception 1 — top-1000 MCP servers

The top ~1000 MCP servers are in LLM training distribution, per a Kimi-paper citation the author attributes. [research-backed] (post 3834)

Consequence: Asana / JIRA-tier MCPs likely work — they are market leaders, agents were trained on Tasks-in-context patterns, and the model can transfer by-analogy inside that family. A colleague ("Denis") reported solving MCP call instability by wrapping an MCP through a CLI call; for Asana this probably wasn't necessary either, because the MCP itself was in training. [empirical] (post 3834)

Rule of thumb: if the MCP corresponds to a market-leader SaaS that millions of developers write against, it is probably in training. If it is a home-grown wrapper for your private system, it is not. [author-claim] (post 3834)

### Exception 2 — Anton's contract-only MCP

Anton's contract-only MCP (post 3622) is a documented working exception. Its design choices matter:

- It does not serve "documentation" in the generic sense. It returns only **contracts + function declarations + the line ranges of implementations**, not prose. [empirical] (post 3622)
- It is strongly typed around GRACE START-END markup (Dyck-compatible paired tags), so a simple algorithm extracts contracts without AST parsing. [empirical] (posts 3622, 3948)
- Tools include `read_contracts` (returns contract IDs in a file, their text, and implementation line ranges) and `get_contracts_by_contract_path` (walks the PATH-header graph to return the subgraph of related contracts). [empirical] (post 3622)
- Intended use: **legacy code** where human-written code is often "face-palm" quality and seeding the LLM with it causes it to replicate bad practices. The MCP hides the implementation and exposes only the contract surface. [author-claim] (post 3622)

Crucially: for code natively written by the AI, the author recommends the opposite strategy — **greedy reading** of full modules (2-4 modules plus the graph, ~20k of context), because AI-written modules are usually under 1000 lines and serve as few-shot calibration to current framework versions. [author-claim] (post 3622)

### Exception 3 — context-gathering subagent

A context-gathering subagent is more likely to call a documentation tool than the main development agent, because it sits on a RAG trajectory rather than a code-writing trajectory. Survey data in the author's channel put the probability of the main dev-agent invoking a docs MCP at under 50%. The author plans to include such a context subagent in GRACE as a standard pattern. [empirical] (post 3848)

### Why agents drop doc MCPs (diagnosis)

Three interacting drivers make documentation low-priority for a code agent, regardless of the mechanism that serves the doc:

- The agent hypothesises "the code is simple, why do I need docs?" [author-claim] (post 3848)
- Aggressive FIM training biases the agent toward reading code directly. [empirical] (post 3848)
- LLMs have learned (from training) that external documentation has a low reliability rating as a class of context — it tends to lag behind the code, and models know this. [author-claim] (post 3848)

### Sergei Muravyev's doc-MCP discussion

Sergei Muravyev built a doc-MCP so an agent could fetch code documentation through it. The author's objection was structural, not implementation-level:

- **Substitution of the agent's task.** The MCP treats "understand how these classes are written" as primary. The agent's actual primary task is "find modules". Once modules are in context, the "understand the class" task resolves automatically. [author-claim] (post 3821)
- **Training mismatch.** Claude Code even has an explicit "greedy reading" prompt instructing subagents to pull everything tangentially related. The MCP asks the agent to work against that reinforcement. [empirical] (post 3821)

The author's 1C cursor-rules reference (github.com/comol/cursor_rules_1c) was cited in this same thread as his contrasting approach to tool guidance. [empirical] (post 3821)

### Embed, don't serve

GRACE's consistent answer to "AI needs documentation" is **embed docs IN code** (in-source documentation, PURPOSE/INPUTS/OUTPUTS/KEYWORDS/LINKS directly at the function site) rather than **serve docs NEXT TO code** through an MCP or graph DB. The former matches training distribution; the latter fights it. [author-claim] (post 3820)

Adjacency is not cosmetic: in-source benchmarks (arxiv 2601.16661v1) show that if description is separated from code by token-distance (e.g., via a file-read or MCP-hop), the agent degrades. [research-backed] (post 3889; cross-link T25)

## Evidence

### Direct citations

- **Post 3622** — Anton's contract-only MCP, `read_contracts` / `get_contracts_by_contract_path` design, legacy vs AI-native recommendation.
- **Post 3820** — four-phase LLM coding-training pipeline; FIM bans documentation; SFT bans "kulibin"; custom MCPs are off-distribution; asking an agent to use external docs provokes plausible hallucination.
- **Post 3821** — Sergei Muravyev's doc-MCP discussion; "find modules" is the real primary task, not "understand class"; Claude Code's "greedy reading" prompt; $10B-supercluster-month framing; reference to github.com/comol/cursor_rules_1c.
- **Post 3834** — Denis's Asana CLI workaround; bash's rock-solid reliability; "kulibin CLI" vs pytest experiment; Kimi-paper citation that top-1000 MCPs are in training; moral "AI does not like kulibin work in Tools or MCP".
- **Post 3848** — call-probability under 50% for docs MCPs; three reasons agents refuse (simple-code hypothesis, FIM pull toward code, low-reliability rating of docs); proposed context-gathering subagent.

### Research / vendor references

- **Kimi paper** — cited in post 3834 as source for "top-1000 MCPs in training". The sources do not specify a URL. [research-backed]
- **github.com/comol/cursor_rules_1c** — author's 1C cursor-rules repo, referenced in post 3821. [vendor-doc]

## See also

- `../01-transformer-foundations/llm-training-pipeline.md` — four-phase coder training (why DIY tools are off-distribution)
- `../04-markup-system/start-end-tags.md` — the markup that makes Anton's MCP possible without AST parsing
- `../06-testing/cli-not-custom-mcp.md` — bash/pytest reliability vs custom CLI tooling (same principle, test-side)
- `../03-contracts/legacy-contract-recovery.md` — legacy-code use-case for contract-only reads
- `../04-markup-system/markup-compression.md` — adjacency matters; token-distance between code and docs degrades agents
- `../09-tooling/hybrid-rag-infra.md` — why GRACE's KEYWORDS/LINKS enrichment replaces a separate graph-DB or doc-service layer
- `../13-antipatterns/all-antipatterns.md` — "Custom MCP tools" and "External documentation systems" entries
