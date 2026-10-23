# DevEx for Coding Agents — Research

Research date: 2026-10-22

Purpose: evidence base for the talk **DevEx for Coding Agents: What Transfers, What Breaks**. This document covers developer experience (DevEx), its application to coding agents, prior art, and measurement. It stays deliberately focused on DevEx and its agent-side analogues.

---

## 1. Scope and definitions

"Developer experience" is used in two different ways. This document uses the second one.

| Meaning | Definition | Focus |
|---|---|---|
| **DX as product experience** | The user experience of a developer using a product: API, SDK, docs, CLI, errors | Developer-facing products |
| **DevEx as conditions of work** | The sociotechnical conditions that shape developer performance: feedback loops, cognitive load, flow, team and organizational factors | How work gets done |

**DevEx for Coding Agents** means applying the second meaning to the conditions in which coding agents work: instructions, tools, feedback, context, guardrails, and orchestration.

It does not claim that agents have feelings, satisfaction, or well-being. It uses DevEx as a design and measurement discipline for the environment agents operate in.

**Agent-side analogue** means the agent equivalent of a specific DevEx principle. It is the right-hand column of the transfer map, not a separate field.

---

## 2. What DevEx research says

### 2.1 The three core dimensions

The canonical paper, "DevEx: What Actually Drives Productivity" (Noda, Storey, Forsgren, and Greiler, ACM Queue 2023), defines DevEx through three dimensions ([summary](https://develocity.io/a-summary-devex-what-actually-drives-productivity-by-noda-et-al-2023)):

- **Feedback loops**: the speed and quality of responses to a developer's actions.
- **Cognitive load**: the mental processing required to perform a task.
- **Flow state**: a state of full immersion and focus.

These dimensions organize 25 sociotechnical factors identified in earlier research. The framework is explicitly multidimensional: improving one dimension can degrade another.

### 2.2 Measurement philosophy

The paper is as much about measurement as about definitions:

- Capture **developer perceptions** through surveys.
- Capture **system data** through telemetry and workflow instrumentation.
- Break results down by team and persona; compare against benchmarks.
- Follow up on findings, or participation collapses.

The core lesson for agent work: **triangulate self-report with system data, and never rely on a single number.**

### 2.3 Other frameworks

| Framework | Dimensions | What it is good for |
|---|---|---|
| **SPACE** (Forsgren et al., 2021) | Satisfaction and well-being; Performance; Activity; Communication and collaboration; Efficiency and flow | Avoiding single-metric productivity; balancing perceptions and system data |
| **DORA** (Google Cloud) | Deployment frequency, lead time for changes, change failure rate, time to restore | Delivery outcomes at the team and organization level |
| **DX Core 4** (DX) | Speed, effectiveness, quality, business impact | A unified scorecard that encapsulates DORA, SPACE, and DevEx |

DX describes the Core 4 as "four counterbalanced dimensions" because optimizing one can hurt another ([DX Core 4](https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4)).

### 2.4 DevEx is not just an input to productivity

DX draws a careful distinction:

> "Developer productivity measures business outcomes (features shipped, bugs fixed, delivery speed). Developer experience measures how developers feel about their work environment and processes. Good developer experience typically leads to higher productivity, but focusing on developer experience first creates sustainable improvements because it addresses the root causes of productivity issues." ([DX guide](https://getdx.com/blog/developer-experience))

Two implications:

1. DevEx is partly an **outcome** (satisfaction, well-being), not only a means to productivity.
2. The causal claim "good DevEx leads to higher productivity" is typical, not guaranteed. The research is more careful than common marketing.

---

## 3. The transfer map: what carries over

This is the core intellectual contribution of the talk.

| DevEx principle | Agent-side analogue | Strength | What to measure |
|---|---|---|---|
| The environment shapes performance more than the individual | The environment shapes agent performance more than the model | Strong | Same task across repositories or configurations |
| Reduce friction in the working environment | Remove ambiguous instructions, slow setup, missing commands | Strong | Time to first useful action; failed setup attempts |
| Shorten feedback loops | Faster tests, deterministic scripts, clear failures | Strong | Iterations per task; time from action to result |
| Reduce cognitive load | Reduce context size, tool count, and irrelevant instructions | Strong as a design principle; context rot is the agent equivalent | Context tokens per turn; tools in scope |
| Triangulate perceptions with system data | Agent self-report plus task success, tokens, and time | Moderate; agent self-report is less validated | Agreement between self-report and outcome |
| Use multidimensional, counterbalanced metrics | Track success, cost, time, and rework together | Strong; prevents gaming a single metric | Four-metric dashboard |
| Avoid one-number productivity metrics | Avoid "tokens saved" or "PRs per agent" as sole success measures | Strong | Cost per successful task, not tokens per run |

### Where the analogy breaks

| DevEx assumption | Agent reality | Consequence |
|---|---|---|
| Developers have a lived experience: satisfaction, well-being, motivation | Agents have none of these | Do not import satisfaction metrics; measure task outcomes |
| Humans accumulate tacit knowledge over months | Agents start fresh unless knowledge is externalized | Documentation becomes runtime input, not onboarding |
| Human cognitive load is working memory and context switching | Agent cognitive load is the context window and attention budget | More instructions can make things worse |
| Human performance includes team culture and psychological safety | Agent performance includes task specification, permissions, and environment determinism | The scope is narrower and more technical |
| DevEx has decades of validation | DevEx for coding agents has a handful of studies | Present it as emerging practice, not settled science |

The rule for the audience:

> Transfer the principles. Re-derive the metrics. Measure before you standardize.

---

## 4. Evidence on agent-side conditions

### 4.1 Cost and quality are not fixed by the model

| Finding | Source | What it shows |
|---|---|---|
| Same model and thinking effort, two harnesses: more than 2x cost difference, same quality; one harness sent about 3x less context per turn | [Databricks benchmark](https://www.databricks.com/blog/benchmarking-coding-agents-databricks-multi-million-line-codebase) | Harness and context management can matter as much as model choice |
| Most expensive task consumed roughly 7 million more tokens than the cheapest; the most expensive run of the same task cost about 2x the cheapest run | [How Do AI Agents Spend Your Money?](https://arxiv.org/html/2604.22750v3) | Cost variance is structural, especially on complex tasks |
| Explore plus Fix phases account for about 64% of agent rounds; cache reads dominate token volume | [How Do AI Agents Spend Your Money?](https://arxiv.org/html/2604.22750v3) | Most spend goes to navigation and iteration, which repository structure can influence |
| Experienced open-source developers took 19% longer with AI tools while believing they were faster | [METR RCT](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study) | Human perception is unreliable; measurement is mandatory |

### 4.2 Context files

Two 2026 studies are the core evidence, and they disagree in useful ways.

**Study A: efficiency gains** — [On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents](https://arxiv.org/html/2601.20404v2) (JAWS 2026)

- Design: 26 repositories with AGENTS.md; 10 randomly sampled; up to 15 merged PRs each; PRs limited to at most 100 changed lines and 5 files; paired runs with and without the file.
- Results: mean output tokens fell **20.08%**; mean wall-clock time fell **20.27%**; median time fell **28.64%**.
- Limitation: measures efficiency, not correctness. A manual sanity check of 50 outputs found non-trivial changes, but there was no full quality evaluation.

**Study B: context files can hurt** — [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988) (ETH Zurich and LogicStar.ai)

- Design: four coding agents on SWE-bench Lite plus AGENTbench.
- LLM-generated context files: success rate **-0.5%** on SWE-bench Lite and **-2%** on AGENTbench; steps **+2.45** and **+3.92**; cost **+20%** and **+23%**.
- Developer-written context files: marginal performance gain, but still more steps in every setting.
- Behavioral finding: agents follow the instructions, which produces more testing and broader exploration, but the files do not work as effective repository overviews.
- Conclusion: context files are "likely only desirable when manually written."

**Reconciliation**

- A concise, accurate file reduces search effort; a long or auto-generated file adds context and steps without improving correctness.
- Practical rule: **write it by hand, keep it short, include only what the agent cannot discover quickly, and measure both success and cost.**

### 4.3 Capability: scripts, MCP, skills

- Databricks found that harness context management, not model choice, drove more than 2x cost differences. Scripts that replace exploratory tool calls with one deterministic command reduce turns and context.
- The token study shows Explore (30.37%) and Fix (33.53%) dominate rounds. A script that shortens setup or reproduces a bug attacks the two largest cost centers.
- MCP is an open standard for connecting AI applications to external systems ([MCP](https://modelcontextprotocol.io/docs/getting-started/intro)). It pays off when it provides capability the agent cannot get locally, not when it duplicates repository files.
- Agent skills package instructions, scripts, and resources in a `SKILL.md` folder and load them progressively: name and description first, full body only when relevant. Keep always-on context small.
- Project-specific skills need project-specific evaluation. A skill that sounds useful can add context without improving outcomes.

### 4.4 Guardrails and orchestration

- Hooks run shell commands at lifecycle points in an agent session, such as before or after a tool invocation ([VS Code hooks](https://code.visualstudio.com/docs/copilot/customization/hooks)). They are the only lever here that can enforce behavior deterministically.
- Use hooks for behavior that must not depend on model compliance: protected paths, destructive commands, formatting, tests before completion. Log their executions; they add latency and can fail silently.
- `git worktree` creates additional working directories attached to the same repository, each on its own branch, sharing the object database. This enables parallel agents without branch conflicts in one directory.
- Parallelism moves cost downstream: merge conflicts, duplicated exploration, and review load. Measure cost per merged change, not agent throughput.
- VS Code custom agents define a specialized agent with its own instructions, tools, model, and handoffs ([Custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)). The value is role separation and smaller contexts; the risk is orchestration overhead.

### 4.5 Delivery-level caution

- DORA 2024: a 25% increase in AI adoption was associated with **+7.5% documentation quality, +3.4% code quality, +3.1% review speed**, but **-1.5% delivery throughput and -7.2% delivery stability** ([Google Cloud](https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report)).
- METR: experienced developers were 19% slower with early-2025 AI tools while believing they were faster.
- A 2026 study found 84% perceived productivity gains while the share reporting worse flow rose from 7% to 20% and worse cognitive load from 6% to 9% ([Annie Vella](https://annievella.com/posts/the-productivity-experience-paradox)).

Implication: individual or agent-level gains do not automatically become delivery gains. The talk should promise stable agent output, lower cost per successful task, and fewer interventions, not guaranteed development-time reductions.

---

## 5. Prior art: who has already talked about DevEx for coding agents

The principle is public. The talk cannot claim novelty for the insight. This section documents the closest prior work and what remains open.

### 5.1 Timeline

| Date | Source | Contribution |
|---|---|---|
| Nov 19, 2025 | [DevEx.md](https://devex.md) | "Developer Experience for People and Agents": two contributors, one setup; agent-loop metrics |
| Nov 20, 2025 | [Modal](https://modal.com/blog/agents-devex) | "Agents need good developer experience too": a framework for DevEx in the age of coding agents |
| Dec 19, 2025 | [incident.io](https://incident.io/blog/ai-developer-tools) | "DevEx matters for coding agents, too": feedback loops for humans and agents |
| Dec 23, 2025 | [Max Kanat-Alexander](https://www.youtube.com/watch?v=rT2Del5pwg4) | "Developer Experience in the Age of AI Coding Agents": fundamentals for humans are fundamentals for agents |
| Mar 2026 | [Quarkus DevStar](https://github.com/quarkusio/quarkus/discussions/53093) | "Quarkus Developer Experience for AI Coding Agents": MCP, skills, docs, Dev UI |
| Apr 2, 2026 | [arXiv 2604.02544](https://arxiv.org/abs/2604.02544) | Empirical study of how coding agents access documentation portals |
| Jun 3, 2026 | [Spotify](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint) | "Developer experience is for agents, too": consistency, Backstage, MCP, guardrails |

### 5.2 The closest sources

**DevEx.md (November 19, 2025)**

- "A repository now has two kinds of contributor, and both of them experience your setup."
- People need access, context, a working local environment, and someone to ask.
- Agents need the same things expressed differently: an instruction file they can work from, exact commands, credentialed tool access, and a fast way to prove a change.
- Defines four agent-loop metrics: time to verified, first-pass acceptance rate, human edit distance, review findings per change.
- It is a template and checklist, not an evidence review.

**Modal (November 20, 2025)**

- Meta description: "Turns out, good devex for agents looks a lot like good devex for humans."
- Principles: tight feedback loops, helpful errors, runnable examples, unified logic and infrastructure, readable code.
- Programmatic access (CLI, SDK, API) shortens loops; agents need text-based, structured interfaces rather than graphical ones.
- Asserts the transfer; does not test where it breaks.

**incident.io (December 19, 2025)**

- Uses Claude Code, Cursor, and Codex daily; over 50 billion tokens per month on development.
- Cut the typical lint plus compile feedback loop from over 90 seconds to under 10 seconds.
- "It's the same for coding agents": fast verification matters for one agent or many with worktrees.
- A single-company tooling story, not a general framework.

**Max Kanat-Alexander (December 23, 2025)**

- AI coding agents depend on foundational DevEx: standardized environments, comprehensive validation, testable architecture, clear documentation, high-quality code review.
- Summary line: "What's good for humans is good for AI."
- Argues the principle rather than mapping which principles transfer or how to measure them.

**Quarkus DevStar (March 2026)**

- Working group to improve Quarkus developer experience for AI coding agents.
- Goals: external MCP server for lifecycle and docs, skills investigation, agent access to Dev UI, end-to-end demos, documentation.
- Framework-level implementation, not a general framework.

**arXiv 2604.02544 (April 2, 2026)**

- Studies HTTP fingerprints from nine coding agents and six assistant services accessing a developer documentation endpoint.
- Finds that agent access compresses multi-page navigation into one or two requests, making session depth, time-on-page, click path, and bounce rate unreliable.
- Recommends AGENTS.md, llms.txt, skill.md, agent-permissions.json, MCP feedback channels, and AI-referral analytics.

**Spotify (June 3, 2026)**

- More than 99% of engineers use AI coding tools weekly; 94% report productivity gains; PR frequency is up 76%.
- Fleet Management has merged more than 2.5 million automated maintenance PRs.
- Section titled "Developer experience is for agents, too."
- Consistency improves agent performance; fragmented codebases are measurably worse.
- Backstage acts as context for humans and agents; capabilities are exposed as MCPs and CLI tools.
- Soundcheck and golden state provide standards and guardrails; linting gives immediate feedback.
- Strongest industry validation, but a platform story, not a small-repo measurement story.

**Laura Tacho / DX (2026)**

- "When we look at what makes agentic workflows work really well, it's clearly defined services, really robust testing, the ability to detect anomalies in the system and fix them quickly, production debugging, observability. All of these things are core components to developer experience." ([Dynatrace interview](https://www.dynatrace.com/news/blog/ais-biggest-impact-isnt-cranking-out-more-code-why-devex-matters))
- AI's biggest impact is not writing more code; typing speed was never the bottleneck.

**SSRN (2025)**

- [Developer Experience (DevEx) Measurement in the Age of AI](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5316738), Adeeb Valiulla.
- Proposes frameworks and methodologies for measuring DevEx in AI-augmented environments.

### 5.3 What is settled, contested, and open

**Settled**

- Good DevEx is widely believed to help coding agents.
- Feedback loops, consistency, programmatic access, documentation, tests, and guardrails recur in every source.
- The principle is already public; claiming it as new would be inaccurate.

**Contested**

- Whether agent-specific practices are genuinely new or just good DevEx applied to a new user.
- Whether context files such as AGENTS.md help. The evidence is conditional and mixed.
- Whether MCP, skills, and orchestration pay off in practice. Results vary by repository.
- Whether individual productivity gains become delivery gains. DORA and METR say not automatically.

**Open, and therefore where the talk can contribute**

1. **A principle-by-principle transfer map with evidence strength.** Most sources assert the principle; few show which parts transfer, which weaken, and which break.
2. **Boundary conditions.** Agents have no lived experience, no tacit memory, and a hard context budget. Industry content rarely explores the limits.
3. **A reproducible measurement protocol for one repository.** Paired trials, multiple runs, cost per successful task, interventions, and rework.
4. **A staged rollout with counter-evidence.** Baseline, one variable at a time, a failed intervention, and a keep-or-revert rule.
5. **A small or medium repository case study.** Spotify is a hyperscale platform; Modal and incident.io are vendor stories.

---

## 6. Measurement protocol

### 6.1 Two signals

**Agent self-report**

Ask the agent, at the end of a session, to describe:

- Whether the task and acceptance criteria were clear.
- Whether it could find the files, commands, and context it needed.
- Where it lost time or had to guess.
- What would have made the task easier.

This is fast and surfaces friction that telemetry misses. It is a proxy, not ground truth.

**System and outcome data**

- Task success against acceptance checks.
- Human interventions per task.
- Steps and tool calls.
- Input, output, and cached tokens.
- Cost per successful task.
- Wall-clock time.
- Rework after the agent declares done.

### 6.2 The metric set

| Question | Metric |
|---|---|
| Did it work? | Pass rate on tests and acceptance checks |
| Did it need help? | Interventions per task |
| Was it efficient? | Steps, tokens, wall-clock time |
| Was it worth it? | Cost per successful task |
| Was it clean? | Rework, review comments, change failure rate |
| Was it safe? | Blocked unsafe actions; permission violations |

### 6.3 Protocol

1. Freeze the task set and the harness.
2. Run paired trials: same task with and without the change.
3. Use at least 3-5 runs per condition.
4. Report median and spread.
5. Pre-declare the keep-or-revert rule.
6. Sanity-check that agents are not passing by weakening tests.

### 6.4 Cautions

- **Efficiency is not correctness.** The AGENTS.md efficiency study did not evaluate output quality.
- **Self-report is not ground truth.** Agents can report a good session while producing a failing change.
- **Variance is large.** A single run can mislead in either direction.
- **Benchmarks can be contaminated.** Prefer recent, private, or repository-specific tasks.
- **Delivery metrics can diverge.** DORA and METR show individual gains coexisting with team-level slowdowns.

---

## 7. Claims and confidence

| Claim | Key number | Confidence | Caveat to state |
|---|---|---|---|
| Environment changes outcomes as much as the model | >2x cost for the same model and task across harnesses | High | One company's internal benchmark |
| Run-to-run cost varies widely | Most expensive run about 2x the cheapest for the same task | High | Single study; demonstrates variance, not an average |
| A repository-maintained AGENTS.md can reduce effort | -20% output tokens, -20% wall-clock time | Moderate | Small tasks, 10 repos, no correctness evaluation |
| A generated AGENTS.md can hurt | -0.5% to -2% success, +20% to +23% cost | Moderate | Applies to LLM-generated files |
| Most spend is exploration and fixing | About 64% of rounds | High | One agent family; phase labels approximate |
| Coding-agent adoption is broad | 22%-29% of 128,018 projects | High | Trace-based estimate; likely a lower bound |
| Experienced developers can be slowed down | 19% longer | High | Early-2025 tools; perception differed |
| Individual AI gains can hurt delivery | -1.5% throughput, -7.2% stability at 25% more adoption | Moderate | Observational, not causal proof |
| Agent self-report is useful but unvalidated | Qualitative session feedback | Moderate | Treat as a signal, not ground truth |

---

## 8. What not to claim

- Do not claim the idea that DevEx matters for agents is new. It is already public.
- Do not claim DevEx always improves productivity.
- Do not claim agent work produces guaranteed delivery-level productivity gains.
- Do not claim AGENTS.md always helps. The evidence is conditional.
- Do not claim more context, more tools, or more agents is better.
- Do not claim MCP or skills are universal wins. Test them in the target repository.
- Do not present a single run as evidence.
- Do not use "agent happiness" or "agent satisfaction" as metrics.

---

## 9. Open questions to resolve in the demo

- Which DevEx principle produces the largest gain in this repository: friction, feedback loops, or cognitive load?
- Does a short hand-written AGENTS.md help this stack, or are conventions already discoverable?
- Which scripts save the most turns: setup, test selection, reproduction, or scaffolding?
- Does an MCP server add capability the repository lacks, or duplicate local files?
- Do hooks reduce rework, or only move failures earlier?
- Does worktree parallelism reduce time to merge, or increase review load?
- How much of the gain survives when tasks move from small fixes to multi-module features?

---

## 10. Sources

### DevEx definitions and frameworks

- Noda, Storey, Forsgren, Greiler, [DevEx: What Actually Drives Productivity](https://queue.acm.org/detail.cfm?id=3595878) (ACM Queue, 2023)
- [Summary of the DevEx paper](https://develocity.io/a-summary-devex-what-actually-drives-productivity-by-noda-et-al-2023)
- Forsgren et al., [The SPACE of Developer Productivity](https://www.devstats.com/blog/space-framework-developer-productivity) (2021)
- DX, [Measuring developer productivity with the DX Core 4](https://getdx.com/research/measuring-developer-productivity-with-the-dx-core-4)
- DX, [What is developer experience?](https://getdx.com/blog/developer-experience)

### Evidence on agents and delivery

- [On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents](https://arxiv.org/html/2601.20404v2)
- [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988)
- [Agentic Much? Adoption of Coding Agents on GitHub](https://arxiv.org/abs/2601.18341)
- [How Do AI Agents Spend Your Money?](https://arxiv.org/html/2604.22750v3)
- METR, [Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study)
- Databricks, [Benchmarking Coding Agents on a Multi-Million Line Codebase](https://www.databricks.com/blog/benchmarking-coding-agents-databricks-multi-million-line-codebase)
- Google Cloud, [Announcing the 2024 DORA report](https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report)
- Annie Vella, [The Productivity-Experience Paradox](https://annievella.com/posts/the-productivity-experience-paradox)

### Prior art on DevEx for coding agents

- [DevEx.md](https://devex.md), November 19, 2025
- [Modal, Agents need good developer experience too](https://modal.com/blog/agents-devex), November 20, 2025
- [incident.io, DevEx matters for coding agents, too](https://incident.io/blog/ai-developer-tools), December 19, 2025
- [Max Kanat-Alexander, Developer Experience in the Age of AI Coding Agents](https://www.youtube.com/watch?v=rT2Del5pwg4), December 23, 2025
- [Quarkus DevStar discussion](https://github.com/quarkusio/quarkus/discussions/53093), March 2026
- [Developer Experience with AI Coding Agents: HTTP Behavioral Signatures in Documentation Portals](https://arxiv.org/abs/2604.02544), April 2, 2026
- [Spotify, Coding Is No Longer the Constraint](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint), June 3, 2026
- [Dynatrace interview with Laura Tacho](https://www.dynatrace.com/news/blog/ais-biggest-impact-isnt-cranking-out-more-code-why-devex-matters)
- [Adeeb Valiulla, Developer Experience (DevEx) Measurement in the Age of AI](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5316738), 2025

### Tooling references

- [Agent-Friendly Repository Technology Guide](./agent-friendly-repository-tech.md)
- [AGENTS.md standard](https://agents.md/)
- [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro)
- Anthropic, [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- VS Code, [Custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- VS Code, [Agent hooks](https://code.visualstudio.com/docs/copilot/customization/hooks)
