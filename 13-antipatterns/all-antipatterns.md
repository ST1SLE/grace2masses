# GRACE Antipatterns — Consolidated Catalog

This file collects the antipatterns GRACE explicitly warns against. Each entry follows the same shape: **What it is / Why wrong / What to do instead / Source refs**. These antipatterns are not style preferences — most of them trace directly to how LLMs were trained (FIM, SFT, RL) or to transformer internals (sparse attention, KV cache, TC⁰/Dyck-Language). Ignoring them typically causes silent degradation rather than a loud failure, which is why they're worth naming. Cross-links point to the deeper KB files that develop each topic in full.

---

## 01 — Vibe-coding-at-scale

**What it is.** Treating small-pet-project coding habits (improvise a prompt, skim the result, iterate fast, no markup, no contracts, no graph) as if they scale to production-sized codebases. [empirical]

**Why wrong.** The author observes a "critical mass" threshold around **5,000–10,000 LOC of interconnected logic** where the AI loses the ability to understand the code (post 3654). This is NOT the whole-app size — it can be a "noodle-code" pocket inside a 100,000-LOC app. [empirical] Survivorship bias on small demos is misleading: "don't assume you can build a large application with a vibe-coding approach suitable for a small program" (post 3654). [author-claim] Past the threshold, the LLM "starts to blow up the code during bug fixes or minor tweaks" (post 3654). [empirical]

**Root causes.** Two transformer-level reasons: (1) sparse-attention collapse past ~4k tokens without semantic anchors (`most_important.md` §"Без семантического графа"); (2) RAG agents such as Cursor read code in 100–200-line chunks and go blind without semantic markers (`xml_mapping_anchors.md` §"Почему семантическая разметка"). [research-backed]

**What to do instead.** Apply GRACE discipline proportional to size: graph first, contracts second, code third (see `../00-foundations/core-principles.md`). For anything past the 5-10k threshold or anything with dense cross-module logic, inject documentation **into** code via START-END markup and contracts (post 3620 shows this in production at 132k LOC).

**Source refs.** post 3654; post 3620; `most_important.md` §"Без семантического графа"; `../00-foundations/why-grace-exists.md`.

---

## 02 — Classical TDD with AI

**What it is.** Transplanting human-centric Test-Driven-Development (write tests first, assert equality, treat the test suite as the spec) directly onto AI-agent workflows. [empirical]

**Why wrong.** Four reinforcing problems, all stated by the author. [author-claim, empirical]

1. **Tests are "just more code"** to the LLM — and unusually hostile code, because tests inherently cross-reference many modules, "burning the precious limited budget of sparse-attention global anchors on silly test invocations" (`testing_and_autonomous_agents.md` §"Почему обычные автотесты").
2. **LLMs overfit to tests**: if tests are placed alongside the contract in-context, the model "truncates its solution space and generates code that just passes your tests" (`testing_and_autonomous_agents.md` §"Еще один пост"). LLM vendors explicitly "burned strict test-like conditions out of training" because they caused overfitting (`contracts_as_anchor.md` §"Почему AI-contracts").
3. **LLM-written tests are low quality**: training saw pairs of tests+code but no meta-reasoning about tests — "LLM is a self-taught parrot when it comes to tests" (`testing_and_autonomous_agents.md` §"Тесты как часть self-correction loop").
4. **Most autotests are smoke tests** (assert on "did it crash?") and don't catch real bugs anyway (`testing_and_autonomous_agents.md` §"Почему обычные автотесты").

**What to do instead.** Move tests into the self-correction loop during debug, not the analytical phase (see `../06-testing/tests-in-self-correction-loop.md`). Prefer Log Driven Development and agent-based testing over assertion-dense unit tests (see `../05-logging-ldd/log-driven-development.md` and `../06-testing/agent-based-testing.md`).

**Source refs.** `testing_and_autonomous_agents.md` §"Почему обычные автотесты", §"Еще один пост", §"Тесты как часть self-correction loop"; `contracts_as_anchor.md` §"Почему AI-contracts"; `../06-testing/classical-tests-antipattern.md`.

---

## 03 — External documentation systems

**What it is.** Building graph databases, separate doc MCPs, or dedicated documentation services that sit **outside** the code file itself and expect the agent to query them. [empirical]

**Why wrong.** The four-phase training pipeline never exposed the model to such sources (post 3820). [research-backed] Base-coder pretraining is ~80% code + ~20% natural language used **for task understanding, not documentation**. FIM pretraining imposed "a complete and total ban on documentation". SFT used task-code pairs (~2-3 paragraphs of task → ~200-300 lines of code) and "strict training on Tool chains for popular agents and MCPs, with a complete and total ban on DIY". RL (SWE-Bench analog) used no documentation at all (post 3820). Consequence: "if the agent is not forbidden from reading code, it will go straight to code — that's what it was trained for. If you forbid it, it will just hallucinate plausibly-looking answers from the descriptions" (post 3820). [author-claim, research-backed]

Sergei Muravyev's doc-MCP case (post 3821) sharpens the point: the agent's **primary task is "find the modules"**, not "understand this class". An MCP that serves documentation misdefines the task. FIM pushes agents to load whole modules into context (Claude Code's subagents even have an explicit "greedy reading" prompt).

**What to do instead.** Embed documentation **into** the code itself via GRACE markup and contracts (see `../04-markup-system/in-source-documentation.md`). If you really need a docs-MCP, scope it to legacy code (Anton's contract-only MCP pattern, post 3622) or use it through a context-collecting subagent whose trajectory aligns with RAG-style retrieval rather than development (post 3848).

**Source refs.** posts 3820, 3821, 3848; `../01-transformer-foundations/llm-training-pipeline.md`; `../09-tooling/mcp-scepticism.md`.

---

## 04 — Custom MCP tools ("kulibin")

**What it is.** Building homegrown / DIY Tools or MCP servers and expecting the LLM to use them as reliably as standard tools. "Kulibin" (кулибин) is a Russian colloquialism for garage-style DIY engineering; the author uses it pejoratively for non-industry-standard tooling. [author-claim]

**Why wrong.** In SFT, models were trained "strictly and specifically on Tool chains for the most popular agents and MCPs, with a complete and total ban on kulibin-style" (post 3820). [research-backed] The author built his own "kulibin CLI" for tests alongside pytest; the LLM "either didn't call my CLI or hallucinated around it. I removed my kulibin and returned strictly to industrial standards the AI was trained on" (post 3834). [empirical] Unlike missing training where the model can adapt by analogy, with MCPs "the model still has the option of simply refusing to use them, which it does" (post 3834). [author-claim]

**Exceptions.** Per the Kimi paper cited in post 3834, the **top-1000 MCP servers** are in training distribution, so Asana/JIRA-tier MCPs likely work. Bash is also in the standard toolset — CLI wrappers on standard frameworks (pytest, etc.) are reliable.

**What to do instead.** Wrap your logic behind bash + a battle-tested framework (pytest is "reliable like a rock", post 3834). If you must build custom, prefer CLI over MCP because of the refusal-risk asymmetry. When a contract-only reading helper is genuinely useful (legacy code), implement it the way Anton did (post 3622): simple algorithmic parsing of START-END tags, returning contract IDs + text + line ranges, without AST.

**Source refs.** posts 3820, 3834, 3622; `../06-testing/cli-not-custom-mcp.md`; `../09-tooling/mcp-scepticism.md`.

---

## 05 — YAML configuration

**What it is.** Using YAML as a format for agent configuration, prompt scaffolding, or any structured data that the LLM has to parse inside its context. [empirical]

**Why wrong.** YAML is **not Dyck-compatible**. Without chain-of-thought, transformers are limited to TC⁰ discrete logic, which natively supports Dyck languages (paired-bracket languages named after Walter von Dyck); Shunyu Yao et al. proved native Dyck support for transformers (arxiv 2105.11115, post 3948). [research-backed] Dyck-compatible formats include XML, HTML, JSON, and bracket-based programming languages. YAML's indentation-based grouping falls outside this class — Google **banned YAML libraries in Gemini's Code Execution sandbox** specifically to prevent LLM degradation when the model tried to use YAML as a common format (post 3948). [vendor-doc]

**What to do instead.** "Rewrite it in a Dyck-Language-compatible format" (post 3948). Prefer XML-like markup; use JSON only in short contexts (see antipattern 06). For Python-in-comment configuration, add paired START-END tags so the comment payload itself is Dyck-compatible in a single pass (post 3948). The author notes YAML is often inserted into agents "by second-role engineers hired by vendors who connect it out of incompetence" (post 3948) — treat it as a red flag.

**Source refs.** post 3948; `../01-transformer-foundations/tc0-and-dyck-language.md`; `../04-markup-system/start-end-tags.md`.

---

## 06 — JSON in long context

**What it is.** Using JSON as the primary delimiter format for long-context prompts, large documents, or long RAG-retrieved payloads. [empirical]

**Why wrong.** The **OpenAI GPT-4.1 Prompting Guide explicitly advises against JSON in long context** and recommends XML or XML-like markup instead (`xml_mapping_anchors.md` §"Почему автор предпочитает XML"; cookbook.openai.com/examples/gpt4-1_prompting_guide#delimiters). [vendor-doc] That guide references the Lee paper *"Can Long-Context Language Models Subsume Retrieval, RAG, SQL, and More?"* The author's own logit-analysis observation: JSON degradation comes from "counting braces" across long spans, "false correlations from `{}`", and slower convergence — universal across vendors including Qwen (`xml_mapping_anchors.md`). [author-claim, research-backed] JSON is still Dyck-compatible (post 3948), so short-context use is fine; the failure mode is specifically long-context.

**What to do instead.** Use XML or XML-like lightweight markup in long contexts (endorsed by both OpenAI and Google, `xml_mapping_anchors.md`). For structured data, keep JSON for small payloads and API boundaries; switch to XML for documents, code contracts, prompts, and log markup.

**Source refs.** `xml_mapping_anchors.md` §"Почему автор предпочитает XML"; post 3948; `../01-transformer-foundations/xml-vs-json-vs-yaml.md`.

---

## 07 — "Sovereign AI" non-English comments

**What it is.** Writing in-source comments / documentation / contracts in languages other than English, often on ideological "sovereign AI" grounds. [empirical]

**Why wrong.** Benchmarks on in-source documentation across multiple languages and multiple vendor LLMs show **English is the best language for comments; other languages cause agent degradation** (arxiv 2601.16661v1, post 3889). [research-backed] The effect-size envelope on this body of work runs from **-95% degradation on bad structure to +435% improvement on good structure** (post 3889) — meaning language choice is not a cosmetic issue. [research-backed] The author calls this out specifically against "LLM nationalists pushing 'sovereign AI' ideas directly into code" (post 3889).

**What to do instead.** Write all in-source contracts, PURPOSE, KEYWORDS, LINKS, and RATIONALE sections in English, even if the project is otherwise non-English. Variable and function names in other languages are a separate issue and not covered by this finding.

**Source refs.** post 3889; arxiv 2601.16661v1; `../04-markup-system/markup-compression.md`.

---

## 08 — "Captain Obvious" comments

**What it is.** Writing in-source comments that merely restate what the code already says (e.g., `# increment counter` above `counter += 1`). [empirical]

**Why wrong.** "The LLM doesn't need 'Captain Obvious' documentation — it reads code perfectly well" (post 3890). [author-claim] The empirical work backs this: agents attending to architectural-intent tokens recover bug-fix accuracy **up to 3× higher** than runs without intent in comments (ResearchGate publication 400340807, post 3890). [research-backed] Captain-Obvious comments take up tokens that should carry **architectural intent** instead. Attention analysis shows the AI literally attends to intent tokens when reconstructing broken logic (post 3890). Tokens that restate code add noise, not signal.

**What to do instead.** Put architectural intent in comments: what pattern, what the module's reason-for-existence is, what constraint this section preserves. GRACE puts this in RATIONALE / AAG sections within module contracts and links to architectural patterns (post 3890, see `../03-contracts/rationale-and-aag.md`).

**Source refs.** post 3890; researchgate.net/publication/400340807; `../03-contracts/rationale-and-aag.md`.

---

## 09 — Long summary-style descriptions

**What it is.** Writing long prose descriptions ("This function takes the input parameters and processes them according to the specified algorithm, returning the computed result…") in docstrings, XML summary tags, or contract descriptions. [empirical]

**Why wrong.** In-source documentation benchmarks show "the shorter the comments, the more effective they are; long descriptions are either neutral or cause agent degradation" (post 3889). [research-backed] ShortenDoc (dl.acm.org/doi/10.1145/3735636, post 3892) confirms this with stronger evidence: "full prepositional phrases cause degradation"; mutilating text by keeping only key concepts yields ~10% HumanEval improvement; you can compress docstring text by **40% without quality loss, sometimes with slight quality gain** (post 3892). [research-backed] C# Summary tags and Python long docstrings behave the same way: "if you shorten Summary to one line, for the AI nothing changes — it reads code fine" (post 3960). [author-claim] What actually carries semantic weight for the model: PURPOSE (goal/intent), KEYWORDS (architectural patterns + task classifications), LINKS (cross-module references) — "fewer tokens than prose but semantically much richer" (post 3960).

**What to do instead.** Aggressively compress. Use wenyan-style KEYWORDS and LINKS (see `../03-contracts/wenyan-prompting.md`). Keep PURPOSE short and declarative. Verify compression two ways: logit-prediction check + blind LLM-gloss test (ask GPT to describe the compressed phrase without seeing the code; if it matches intent, the compression is safe) (post 3892).

**Source refs.** post 3889, post 3892, post 3960; arxiv 2601.16661v1; dl.acm.org/doi/10.1145/3735636; `../04-markup-system/markup-compression.md`; `../03-contracts/wenyan-prompting.md`.

---

## 10 — TEST FAIL binary signals

**What it is.** Reporting test outcomes back to an AI agent as a single binary flag — most commonly a plain `TEST FAIL` string with no structured context. [empirical]

**Why wrong.** "Modern approaches to AI testing give equal importance to binary and detailed signals — a plain 'TEST FAIL' even as a binary signal is bad practice" (`testing_and_autonomous_agents.md` §"Тесты как часть self-correction loop"). [author-claim] Modern AI-testing frameworks "dump a large number of 'flag-type' conclusions at the AI via logging, which is easier for GPT to read". The SWE-Bench-style RL training regime specifically "teaches the AI to read log fragments and, in response, make changes to the code" — so the AI expects detail-rich flag-oriented logs during the self-correction loop (§"Тесты как часть self-correction loop"). [research-backed] Asserts alone are generally brittle; trajectory analysis from rich logs generalizes better (§"Адаптация GRACE"). [author-claim]

**What to do instead.** Emit structured, flag-rich, categorized log output from tests. Apply Log Driven Development conventions — log-to-code correlation tokens (`../05-logging-ldd/log-to-code-correlation.md`), belief-state logging (`../05-logging-ldd/belief-state-logging.md`), category markers. On test failure, the test framework should hand the agent enough signal to diagnose without re-running.

**Source refs.** `testing_and_autonomous_agents.md` §"Тесты как часть self-correction loop", §"Адаптация GRACE"; `../06-testing/tests-in-self-correction-loop.md`; `../05-logging-ldd/log-driven-development.md`.

---

## 11 — Assuming the agent self-rescues via Tools when looping

**What it is.** Designing autonomous/multi-agent systems on the assumption that "if the agent gets confused, it will call the right Tool to gather the context it needs". [empirical]

**Why wrong.** "Simply counting on the agent — especially if it's looping — to call Tools on its own to read the important information is naive" (post 3695). [author-claim] The looping behavior itself makes the risk acute: "failed fix attempts become few-shots for the model → it just becomes more confident in the same failed approach with each iteration" (post 3695). A swarm or a single autonomous agent can "stupidly hang and burn a billion tokens" in this state. [author-claim] Tools-on-demand does not solve this because the model's confidence trajectory points in exactly the wrong direction — it stops reaching for new context as it "learns" from its own failures.

**What to do instead.** Build **forced context injection** into the agent infrastructure: deliberately push key log lines, prior-fix knowledge, and critical constraints into context rather than waiting for the agent to ask (posts 3693, 3695). Add **anti-loop protection**: attempt counters + automatic context injections to reset the agent's posture (post 3695). Integrate tests with a **knowledge base of prior fixes**, and on regression inject the relevant KB entries forcibly into context (post 3695). All three must be designed into the test framework from the start.

**Source refs.** posts 3693, 3695; `../07-autonomous-agents/anti-loop-protection.md`; `../07-autonomous-agents/forced-context.md`; `../07-autonomous-agents/knowledge-base-in-tests.md`; `../05-logging-ldd/forced-context-injection.md`.

---

## 12 — Skills without forced loading

**What it is.** Using dynamic skills (Open Code / new Kilo Code style) by dropping them into the system and trusting the LLM to load them when relevant. [empirical]

**Why wrong.** "Scientific work documents that skills are an extremely dangerous form of prompting because the LLM may decide it 'already knows' how to do this and simply ignore loading the skill" (post 3809). [research-backed] The author calls this "a truly serious risk of agent collapse" and says "only dilettantes drop skills in without thinking" (post 3809). [author-claim] The failure mode is silent: the skill exists, but the agent never loaded it, and you get answers from the model's training prior instead of the curated skill.

**What to do instead.** Two mitigations per post 3809:

1. **Trigger words**: use **MANDATORY MODE** or **MANDATORY PROTOCOL** phrasing to flag critical rules (not optional KB material). [author-claim]
2. **Forced tracing** ("principal mode of reliable skill loading"): start with an explicit directive telling the AI which skill to read first; each loaded skill explicitly names the next skill to load (by name). This chains loading deterministically. [author-claim]

In Open Code specifically, you can further strengthen this by requiring an explicit `skill` tool call. With trace-based loading, dynamic skills become "as reliable as dynamic prompts on tracing" (post 3809). [author-claim]

**Source refs.** post 3809; `../09-tooling/skills-system.md`.

---

## 13 — Using strict assertions as primary truth

**What it is.** Making `assert expected == actual`-style equality checks the primary evaluator of correctness for AI-generated code, especially in business-logic-heavy domains (ERP, 1C, SAP, complex ETL). [empirical]

**Why wrong.** Four overlapping problems. [author-claim, empirical]

1. **Overfitting**. "If you directly include tests in the same conditions as contracts with the AI, the AI just truncates its solution space and immediately generates code that passes your tests" (`testing_and_autonomous_agents.md` §"Еще один пост"). LLM vendors explicitly burned strict test-like conditions out of training for this reason (`contracts_as_anchor.md` §"Почему AI-contracts").
2. **Assert preparation is expensive and shallow**. "Most autotests I've seen are smoke tests — assert at the level of 'did it crash or not?' — and don't catch real bugs anyway" (`testing_and_autonomous_agents.md` §"Почему обычные автотесты").
3. **Strict equality is wrong shape for fuzzy business logic**. "For ERP developers… business logic is hard to check with simple equalities; but through semantic checks by the AI, it can be checked more than adequately" (`testing_and_autonomous_agents.md` §"Адаптация GRACE"). Log-Driven Development unlocks autotesting for ERP-style logic that was previously "almost not covered by autotests at all". [author-claim]
4. **Pass-green-but-broken**. "LLMs are very inclined to produce code that passes autotests but doesn't work in real tasks" (`testing_and_autonomous_agents.md` §"Agent-based testing"). Agent-based testing catches this where assert suites don't.

**What to do instead.** Use **Log Driven Development**: the AI reads structured logs and makes a semantic judgment about correctness from the execution trajectory (`../05-logging-ldd/log-driven-development.md`). Minimize or drop asserts ("the first thing you should do — throw out asserts, or ask the AI to make them minimalistic", §"Почему обычные автотесты"). For complex ETL and ERP-style logic, prefer **agent-based testing** where a tester-agent follows a Markdown guide from the developer-agent (`../06-testing/agent-based-testing.md`). Even if you need an assert-like production check, "it's better to add the AI as an LLM call for log-checks-by-prompt, not as a chatbot" (§"Почему обычные автотесты").

**Source refs.** `testing_and_autonomous_agents.md` §"Адаптация GRACE", §"Почему обычные автотесты", §"Еще один пост", §"Agent-based testing"; `contracts_as_anchor.md` §"Почему AI-contracts"; `../06-testing/classical-tests-antipattern.md`; `../05-logging-ldd/log-driven-development.md`.

---

## See also

- `../00-foundations/core-principles.md` — the positive statements of which these antipatterns are the inverses.
- `../01-transformer-foundations/` — the training-pipeline, Dyck-Language, and sparse-attention reasons underlying many of these.
- `../06-testing/` — classical-tests antipattern expanded; LDD, agent-based testing, CLI-not-custom-MCP.
- `../07-autonomous-agents/` — forced context, anti-loop protection, skills-forced-loading.
- `../14-research-references/annotated-bibliography.md` — the cited papers (arxiv 2105.11115, arxiv 2601.16661v1, dl.acm 3735636, dl.acm 3689484.3690738, researchgate 400340807, OpenAI Prompting Guide, Kimi paper).
