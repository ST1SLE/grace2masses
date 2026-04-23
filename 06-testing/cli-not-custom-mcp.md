# CLI over Custom MCP — Why Standard Tools Beat Kulibin Tooling

## What

When exposing functionality to an AI coding agent, **prefer standard tools invoked via Bash over custom MCPs or bespoke CLIs** [author-claim] (post 3834). The rule is a direct consequence of the LLM coder training pipeline: `bash` sits inside the standard LLM toolset, and widely-used frameworks such as `pytest` are in the training distribution "like a rock", whereas home-grown ("kulibin") tooling is not in training and the agent is free to ignore or hallucinate around it [author-claim] (post 3834).

The same post records the author's empirical test of this rule: an earlier GRACE iteration offered **two** autotest entry points — the standard `pytest` runner and a custom "kulibin CLI" for tests. In practice the agent either did not call the custom CLI or hallucinated its behaviour. The author removed the custom CLI and moved strictly to industrial-standard tooling [empirical] (post 3834).

The concept extends to MCP servers. The top-1000 MCP servers are in LLM training (attributed to a Kimi paper in post 3834), so a market-leader MCP such as Asana or JIRA is very likely to work. A custom MCP outside that distribution is not — and an agent facing an MCP it was never trained on can **silently refuse to call it**, a failure mode specific to MCPs [author-claim] (post 3834).

The Russian term **"kulibin"** (кулибин / кулибинщина) is used by the author throughout: a colloquial Russian label for garage-style DIY engineering. In GRACE context it is pejorative — "kulibin tooling" means tools the target LLM was never trained on.

## Why

The rule follows from how LLM coders are trained [research-backed] (post 3820, cross-link `../01-transformer-foundations/llm-training-pipeline.md`):

1. **SFT pairs tasks with strict Tools-chain training on popular agents/MCPs, with an explicit ban on kulibin tooling** [research-backed] (post 3820). The model never saw your custom tool at SFT time.
2. **The top-1000 MCP servers are in training** (per Kimi paper, cited in post 3834). Outside that cutoff, the model has no prior on your tool.
3. **`bash` is in the standard toolset** that SFT drilled into the agent. Invoking `bash` inside a long trajectory has a much higher success rate than invoking a custom MCP [author-claim] (post 3834).
4. **For MCPs specifically, the agent has a special out: silent refusal.** It can simply not call the tool — a failure mode the framework often does not notice [author-claim] (post 3834). With a custom CLI the equivalent failure is hallucination; with a custom MCP it can be total absence.

The broader principle: **the methodology cannot fight the training distribution** [author-claim] (post 3820). A custom MCP competes directly with the work of a ten-billion-dollar supercluster month that produced the agent's FIM and SFT weights, and it loses [author-claim] (post 3821). A custom CLI at least inherits some of Bash's training support.

There is a secondary reason specific to test tooling. `pytest` is not just "widely used" — it is reliable "like a rock" inside long agent trajectories because the agent has seen it thousands of times during training [author-claim] (post 3834). Autonomous agents running 20+ hours (e.g. post 3620) or swarm configurations depend on tool calls not quietly failing mid-run. Substituting a home-grown test runner is a trajectory-stability regression before it is a convenience regression.

Third, the recommendation is **not** that Bash is a guarantee. Even wrapped in Bash, a kulibin tool can fail:

> Однако и это не гарантирует ничего. Для примера, я в GRACE у себя сделал автотесты сначала двух видов. Через командную строку вызывался стандартный pytest, а также мой «кулибинский CLI» для тестов. В реале ИИ либо не вызывал мой CLI, либо галлюцинировал вокруг него. (post 3834)

Translation: "But even that guarantees nothing. For example, in my GRACE I initially set up autotests of two kinds. Standard `pytest` was called via the command line, along with my 'kulibin CLI' for tests. In reality the AI either did not call my CLI or hallucinated around it." [empirical] (post 3834)

The lesson is that the Bash channel pulls the **tool itself** into the training distribution by convention — `pytest`, `grep`, `git`, `jq`, `curl`, and similar. Home-grown tools wrapped in a Bash call still fall outside that distribution and still fail.

## How

### Rule 1 — Prefer Bash + standard tools over custom MCPs

When you need the agent to run tests, read files, hit an API, manage version control, or perform any other routine operation, **reach for a tool the training distribution already knows** [author-claim] (post 3834). Typical shortlist:

- Tests: `pytest` (Python), equivalent standard frameworks elsewhere.
- File operations: `grep`, `find`, `cat`, `ls` via Bash.
- Version control: `git`.
- HTTP: `curl`.
- Text processing: `jq`, `sed`, `awk`.

These are the tools the model has been drilled on during SFT. A Bash invocation inherits that training signal [author-claim] (post 3834).

### Rule 2 — If you need a custom wrapper, wrap a standard tool via Bash — do not ship an MCP

If the task really requires extra logic that no standard tool handles, build a **CLI wrapper** and invoke it via Bash rather than creating a new MCP [author-claim] (post 3834). The CLI wrapper inherits some of the training signal Bash has. The MCP inherits none.

But the author's own episode (post 3834) is the caveat: even a CLI wrapper is not magic. The agent may still ignore or hallucinate around a kulibin CLI. If the wrapper is optional — if a standard tool already covers the case — remove the wrapper. Keep only what the agent actually uses [empirical] (post 3834).

### Rule 3 — For MCPs, stay inside the top-1000

The top-1000 MCP servers are in the SFT distribution per the Kimi paper (post 3834). Practical implication [author-claim] (post 3834):

- **Market-leader MCPs likely work.** Asana, JIRA and similar tools appear inside that cutoff. The author records a chat contributor ("Denis") working around MCP-via-Skill instability by using a CLI wrapper — but notes that Asana/JIRA MCP itself would very likely have worked because it is a market leader, not kulibin (post 3834).
- **Custom / internal MCPs likely do not work.** If you wrote it, the agent did not see it. An MCP-ignoring agent does not fail loudly; it simply does not call the tool [author-claim] (post 3834).

The single documented GRACE-compatible exception (post 3622, cross-link `../09-tooling/mcp-scepticism.md`) is Anton's contract-only MCP, which serves contracts and declarations — *code-adjacent artifacts the FIM pipeline trained the agent to want*, not standalone documentation prose.

### Rule 4 — For tests specifically, default to `pytest` (or the language equivalent)

The author's concrete recommendation for test tooling in AI pipelines is simple [author-claim] (post 3834):

- Use `pytest` as the default test runner. It is reliable "like a rock" inside long trajectories because it is heavily represented in training.
- Do not add a parallel "better" test runner for the agent to choose from. The agent will either ignore it or hallucinate.
- If the GUI is the thing under test, keep a **CLI fallback** on top of `pytest` — GUI-driven AI testing is slow and expensive (cross-link `../06-testing/in-app-ai-console.md`), but the CLI path must stay in the standard-tool lane [author-claim] (testing_and_autonomous_agents.md §"Почему обычные автотесты").

### Rule 5 — Recognise silent refusal as a real mode

With custom CLIs, the usual failure is **hallucination** — the agent generates a plausible-looking invocation or confident output without actually exercising the tool [empirical] (post 3834).

With custom MCPs, a second failure mode opens: **silent refusal**. The agent simply does not call the MCP and proceeds as if it were not there [author-claim] (post 3834). Because the agent never declares "I am refusing this tool", test frameworks and harnesses that only check for tool errors will miss the regression entirely.

Monitoring guidance that follows from this [inferred; the sources do not specify explicit monitoring steps]:

- Count tool invocations per trajectory and compare against expected baselines.
- Treat "MCP invoked zero times" as a failure signal, not a neutral event.

(The sources do not specify monitoring tooling; only the failure modes themselves.)

### Rule 6 — Distinguish this from "agents should not read docs via MCP"

This file is about **action-tooling MCPs**: things the agent calls to *do* something (run tests, manage tasks, hit an API). The closely related rule about **doc-serving MCPs** — that agents routed through doc services will ignore them or hallucinate — is covered in `../01-transformer-foundations/llm-training-pipeline.md` and `../09-tooling/mcp-scepticism.md`. Both rules share a common root (training distribution), but the doc-MCP case has a Sergei-Muravyev-specific failure mechanism ("the agent's real task is to *find modules*, not *understand a class*") that belongs in that file (post 3821), not here.

## Evidence

**Post 3834 (grace_matches.md)** — primary source for this file. Key load-bearing statements:

- "Поскольку bash как раз входит в стандартные инструменты из обучения LLM, то действительно вероятность того, что ИИ умеет пользоваться им внутри длинной траектории, намного выше." [author-claim]
  Translation: "Because `bash` is in the standard tools from LLM training, the probability that an AI knows how to use it inside a long trajectory is indeed far higher."
- "Однако и это не гарантирует ничего. Для примера, я в GRACE у себя сделал автотесты сначала двух видов. Через командную строку вызывался стандартный pytest, а также мой «кулибинский CLI» для тестов. В реале ИИ либо не вызывал мой CLI, либо галлюцинировал вокруг него. Просто это не было в обучении GPT, а вот стандартный фреймворк pytest работает как скала по надёжности. Поэтому я убрал свою кулибинщину и вернулся строго к промышленным стандартам разработки ПО, которым ИИ и обучался." [empirical]
  Translation: "But even that guarantees nothing. For example, in my GRACE I initially set up autotests of two kinds. Standard `pytest` was called via the command line, along with my 'kulibin CLI' for tests. In reality the AI either did not call my CLI or hallucinated around it. It simply was not in GPT's training, whereas standard `pytest` works like a rock in reliability. So I removed my kulibin work and went back strictly to industrial-standard software-development tooling that the AI was trained on."
- "Asana (JIRA) — это не кулибинщина, а лидер рынка. Очень вероятно, что и MCP бы даже прокатил, т.к. топ-1000 основных MCP-серверов находятся в обучении GPT (это известно по публикации Kimi), и там JIRA точно есть." [research-backed, attributed to Kimi paper]
  Translation: "Asana (JIRA) is not kulibin but a market leader. It is very likely that even its MCP would have gone through, because the top-1000 main MCP servers are in GPT training (this is known from the Kimi publication) and JIRA is definitely there."
- "Мораль тут очень простая: ИИ не любит кулибинщину в части Tools или MCP, т.к. банально ей не обучался. ИИ на деле много чему не обучался прямо, но в случае MCP у него ещё есть вариант просто отказаться от использования, что он и делает." [author-claim]
  Translation: "The moral is very simple: AI does not like kulibin tooling in Tools or MCP because it was plainly not trained on it. AI has not been trained on many things directly, but in the case of MCP it has an additional option — simply to refuse to use it, which is what it does."

**Post 3820 (grace_matches.md)** — the supporting training-pipeline argument:

- SFT-phase detail: "На SFT GPT также учат строго и конкретно цепочкам Tools под самые популярные агенты и MCP, с полным и тотальным запретом на 'кулибинщину'." [research-backed]
  Translation: "At SFT, GPT is also trained strictly and concretely on Tools chains for the most popular agents and MCPs, with a complete and total ban on 'kulibin tooling'."

**Post 3621 / 3622 (grace_matches.md)** — documented exception for contract-only MCPs. The relevant rule here is only that it is an *exception*: Anton's MCP returns contracts plus declarations plus implementation line-ranges, which are still code-adjacent artifacts (post 3622). Full detail lives in `../09-tooling/mcp-scepticism.md`.

**Source not cited here, intentionally**: the URL or venue of the Kimi paper. The sources attribute the "top-1000 MCPs in training" claim to a Kimi paper but do not specify the title or URL; nothing is invented (post 3834).

**Research/benchmark values**: the "top-1000 MCP servers in training" figure is from post 3834, attributed to a Kimi paper. No additional numbers are introduced in this file.

## See also

- `../01-transformer-foundations/llm-training-pipeline.md` — the four-phase training pipeline that makes this rule follow mechanically from SFT Tools training and the top-1000 MCP distribution.
- `../06-testing/classical-tests-antipattern.md` — why `pytest`-as-reliable still does not excuse classical TDD habits with AI agents.
- `../06-testing/tests-in-self-correction-loop.md` — where `pytest`'s signals actually belong in a modern AI workflow (debug-phase self-correction, not analytical phase).
- `../06-testing/in-app-ai-console.md` — the in-application AI agent with console + Tools is an alternative to running external CLIs; this file explains why the external-tool path still defaults to `pytest`-backed CLI when used.
- `../09-tooling/mcp-scepticism.md` — the broader "when MCPs work and when they don't" discussion, including Anton's contract-only MCP as a boundary case and the Sergei Muravyev doc-MCP failure mechanism.
- `../07-autonomous-agents/anti-loop-protection.md` — long autonomous trajectories make silent-refusal and tool-instability failures more costly; this rule is partly a trajectory-stability rule.
- `../13-antipatterns/all-antipatterns.md` — "Custom MCP tools (kulibin)" antipattern entry cross-references this file.
- `../14-research-references/annotated-bibliography.md` — Kimi paper entry, attribution only; URL not specified in sources.
