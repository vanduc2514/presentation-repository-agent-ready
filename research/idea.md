
**Setting Up a Coding-Agent-Friendly Repository**

**1. Central Thesis**

Same model, same task: a "bare" repo gives erratic results; a well-configured repo gives stable, cheaper results with less need for human intervention.

**2. Why — Why is this needed?**

* The model is non-deterministic and can hallucinate.
* The Agent has no memory, tends to repeat its line of reasoning, and keeps "fumbling around" over and over.
* Without constraints, the Agent easily goes in the wrong direction, output is unstable, and a fair amount of manual fixing is required.
* The Agent still needs human supervision, so there must be tools for humans to supervise and review effectively.

**3. How — The Solution**

Before the AI era, a well-set-up repo was a repo with good developer experience (DevEx) (Notes: revisit the standards that define this). Good DevEx leads to improved developer performance and higher project productivity.

After the AI era, besides good DevEx, it must be further improved so that Agent Experience (AgentEx) is also good. Good AgentEx and DevEx lead to good performance from both the Agent and the Developer (who is now the one directing, monitoring, and checking the Agent), project development productivity leaps, and development time drops noticeably.

To improve AgentEx there are the following ways:

[**AGENTS.MD**](http://AGENTS.MD)

**Helpful Scripts**

**MCP**

**SKILLS**

**VSCode Agents (advanced — optional — the ideal interface for an Agent Orchestrator)**

**Hooks (advanced — optional — Guard Rails for the Agent)**

**Worktree (advanced — optional — allows multiple agents to run at the same time)**

How to roll out these technologies: roll out gradually from a baseline.

To deploy well and effectively, treat this setup like software performance optimization — improve and measure continuously, that is:

Run the Agent on the original repo -> Monitor and check input and output -> Build [AGENTS.md](http://AGENTS.md) to avoid the phenomena above -> Run the Agent on the repo that now has AGENTS.md -> Continue measuring and checking input and output -> Create scripts (or use tools) or add MCP to extend the capabilities of the Coding Agents -> Run the Agent with [AGENTS.md](http://AGENTS.md) + scripts + mcp in place -> Continue measuring and checking input and output -> Apply existing SKILLs or create SKILLs for the Coding Agent -> Run the Coding Agent -> Complete

Key points:

* Use existing technologies to improve the quality of an agent-friendly repo
* A well-set-up repo helps the Agent run stably, gives highly deterministic results, speeds up Agent runs, and reduces Agent running costs
* Do not apply the above technologies blindly without real verification and measurement.
* The above technologies are set up and installed based on a specific repo (tailored).
* General-purpose SKILLS (for example docx, pptx — Anthropic skills) can be applied directly, but project-specific SKILLS must be tuned and installed according to each project.

**4. Storytelling Approach**

* Use a single sample repo with a medium-size microservice structure of 3 to 5 full-stack modules including IAC, DB, Front end, Back end, with bugs and missing features
* Bare repo + default agent = baseline
* Record a demo video of running the Agent on the baseline to point out the Agent's limitations in the Why section
* Each How section has a demo video of the setup and the Agent running on it.
* A live demo with VSCode Agents is possible if time allows
* End with before/after and lessons learned: the skill of designing the working environment matters more than waiting for a better model.

**5. The Future of Software Engineers (Optional if there is not enough time)**

* In the next few years, what will Software Engineers do? (If AI can already implement features, analyze, and fix bugs, what should the engineer's job be in the future?)
* Software Engineers now shift to building the runtime environment / building a Software Factory for Coding Agents (or a Factory for AI Agents in general). This includes building, deploying, operating, monitoring, and improving the factory
* Software Engineers will shift to AI Engineer and ultimately to FDE (Forward Deployed Engineer)

**6. Strengths Compared with the Feedback from the Previous Session**

Feedback shows that students want: hands-on practice, case studies, AI code quality control, token/cost optimization, context engineering, agent evaluation, and safety when the Agent accesses the machine. This idea hits almost all of it, and upgrades from "understand the agent" to "master the environment for the agent".

7. Presentation Template

[https\://github.com/vanduc2514/agent-presentation-markpress](https://github.com/vanduc2514/agent-presentation-markpress)
