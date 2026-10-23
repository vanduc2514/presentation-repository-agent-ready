# Building an Agent-Friendly Repository

Research notes for a 120 minute talk.

The talk is about a narrow idea: a coding agent already works in a plain repository. What the repository setup changes is how well it works. The techniques in this talk are small optimizations to the environment the model works in, not requirements and not a new capability. After the setup, the work continues as monitoring: watch the agent, notice where it wastes effort, and make the next small fix.

Supporting material:

- [devex-for-coding-agents.md](./devex-for-coding-agents.md): literature and prior art
- [agent-friendly-repository-tech.md](./agent-friendly-repository-tech.md): setup detail for each technology

## 1. Talk at a glance

**Title:** Building an Agent-Friendly Repository

**Subtitle:** Small optimizations that make a coding agent work better in your repository

**Duration:** 120 minutes

**Audience:** computer science students and early-career engineers who have used coding agents and seen them fail in ways that were hard to explain.

**Harness:** the tool that runs the model and its tools. The examples use Copilot in VS Code, so hook names and skill frontmatter follow Copilot; other harnesses have equivalents under different names.

**Prerequisites:** none. The talk is a lecture; nothing needs to be installed to follow along. The setup is shown so the audience can repeat it later. The Agents window is in Preview, which matters only to those who try it afterwards.

**Format:** no live demo, no exercise and no break. The first slide of each area carries one still image, one per area and listed in section 4, captured beforehand from a synthetic demo repository named on the slide. Use PNG at 1440 px wide under 250 KB, or SVG when the content is text. The stills live in `slides/images/`; markpress embeds them into the HTML, and the build also copies the folder as a fallback. The talk uses stills rather than video. The areas segment runs about four minutes per slide; the still gives the slide its concrete half, and the explanation carries the rest.

**Out of scope:** model choice and prompt technique. The model is held fixed for the whole talk; improving it is a different subject.

### Segments

| Segment | Slides | Minutes | Content |
|---|---|---|---|
| Open | 1-6 | 20 | The agent already works, what "not effectively" looks like, why, the reframe, what the talk is not |
| Areas | 7-17 | 45 | The six questions, the six areas, orchestration, adding improvements |
| Monitor | 18-21 | 20 | Watching the agent after setup, the signals, the fix loop |
| Close | 22-27 | 25 | DevEx conditions, the mapping, the transfer map, anti-patterns, lessons, one line |
| Q&A | | 10 | Questions and answers |

Minutes add up to 120. There is no break. The DevEx frame closes the talk because it labels work the audience has already seen; it summarizes the six areas rather than introducing them.

## 2. The idea

### 2.1 The claim

With the model and the task held fixed, changing the repository changes how the agent works: fewer mistakes, less spend, shorter sessions.

That is the argument of the talk, not a result it measures. The talk gives the mechanism, which is that unanswered questions become inferences and inferences cost tokens, time and correctness, and it gives the signals that would show the effect in your own sessions. If those signals do not move after an improvement, the claim did not hold for that repository.

The agent is not blocked without this work. It runs, produces something, and often produces something usable. The difference is in the friction around that work: the wrong turns, the repeated searches, the confident summary over a failing build, the review time at the end.

### 2.2 The agent works either way

An audience that has watched agents fail may doubt this. A plain repository is not a broken one.

- The agent reads files, runs commands, edits code and reports back.
- For a small, local change it often succeeds.
- Nothing in this talk is required for the agent to start.

So the question is not "will it work". It is "how much of the work goes into finding out what the repository already knows".

### 2.3 What "not effectively" looks like

The failure is rarely a crash. It looks like this:

- A change that looks right and does not pass the build.
- Edits in generated files, environment files or code the change had no business touching.
- A repeated approach that already failed earlier in the same session.
- A long session that is mostly searching: listing directories, reading files, guessing commands.
- Token spend out of proportion to the size of the change.
- Review and repair that take back the time the agent saved.

Two published studies are relevant. A repository-maintained AGENTS.md is associated with lower token use and shorter runs (Lulla et al., 2026). Generated context files did not improve success and raised cost (Gloaguen et al., 2026). Neither measures the failure modes above; they are context from other repositories, not a reproduction.

### 2.4 Why it happens

A coding agent starts each session without a working model of this repository. It has instructions, conversation history and tools, but not the knowledge a teammate accumulates. Everything it needs has to come from the files, the conversation or the tools.

- The build command is not stated, so it guesses, and guesses again when the first fails.
- The feedback loop is slow, so it iterates without knowing whether it is getting closer.
- The conventions are not stated, so it copies the nearest example, which may be the wrong one.
- The boundaries are not stated, so nothing stops it from editing a generated or production file.
- The work depends on something outside the checkout, and it cannot see it.
- The person supervising has no view of what ran, so they review blind.

Each of these is an inference the model has to make. Inferences cost tokens, time and correctness. That is the whole problem the rest of the talk addresses, and section 3 turns the six bullets into the six questions the repository should answer.

### 2.5 The reframe

The techniques in this talk are best understood as optimizations to the model's working environment:

- They do not make the model smarter. They remove work the model would otherwise repeat.
- They are small, local and reversible: a file, a script, a hook, a server.
- Each one can be tried alone and removed again if it does not help.
- They are optimizations, not a gate. A repository without them still works.

The model is the same before and after. What changes is how much of its budget goes into figuring out the repository instead of doing the task.

### 2.6 What this talk is not

- Not about choosing a better model or writing better prompts. Both matter; neither is the subject.
- Not a claim that repository setup makes an agent correct.
- Not a measurement. The claim is argued from mechanism and checked by monitoring, not proved here.
- Not a checklist to install everything. Several of the areas are conditional, and one is optional.

## 3. What "agent-friendly" means here

A repository is agent-friendly when it, or the setup around it, answers the questions an agent and its supervisor would otherwise have to ask:

1. How do I build and run this?
2. How do I check that a change is correct?
3. Which conventions apply to this code?
4. What must I not touch?
5. What does the work depend on outside the checkout, and how do I reach it?
6. How does a person control and review the work?

Questions 1 to 4 are about the checkout. Question 5 is about what the checkout points to. Question 6 is about the surface a person uses to start, supervise and review runs.

The title says repository, and four areas are files and directories in the checkout. MCP is a server plus client configuration, and orchestration is the tool a person uses to run the agents. The talk says this plainly when it gets there.

The point is not that the repository must answer them perfectly. It is that every unanswered question becomes an inference, and inferences are where the mistakes, the tokens and the long sessions come from.

## 4. The six areas

Six areas, in no required order. Four are files and directories in the checkout; MCP is a server plus client configuration; orchestration is the tool a person uses to run the agents. Each one is described by what it is, what it removes, its context cost, when it pays, and the artifact that shows it.

There is no required order for adding them, but they do layer logically: AGENTS.md states the answer, scripts automate it, skills package the procedure, hooks enforce it. Layering explains how they fit; it does not say what to do first.

Context cost is how much of the agent's working window the area takes in a normal session.

| Area | Question it answers | What it removes | Add it when |
|---|---|---|---|
| AGENTS.md | 1 to 4 | Wrong turns from inferring commands, conventions and boundaries | The agent is guessing commands or conventions |
| Scripts | 1, 2 | Rounds spent discovering how to run things | A procedure repeats |
| Skills | 3 | Inconsistent execution of a repeated procedure | The same procedure repeats with detail |
| Hooks | 4 | Failures that instructions alone cannot prevent | The agent can reach something it must not change |
| MCP | 5 | Guesses about things outside the checkout | The answer lives outside the checkout |
| Orchestration | 6 | Blind review when several sessions run at once | Review, not generation, is the bottleneck |

Orchestration is the exception in this table: it is the tool a person uses to run the agents, not a change to the repository. It is listed because it answers the last question, and it is optional until several sessions run at once.

This table is the single map from questions to areas, with the trigger for each. Section 5 is only about choosing between them.

### 4.1 AGENTS.md

What it is: one short file, written by hand, with commands, a short layout, conventions, boundaries and what "done" means.

```markdown
# AGENTS.md

## Commands
- Setup: `./scripts/setup.sh`
- Verify: `./scripts/verify.sh`
- Test one file: `./scripts/test-one.sh <path>`

## Layout
- `services/api` routes and handlers; `src/domain` logic; `infra` IaC; `web` frontend; `db` schema and migrations.

## Conventions
- API handlers in `services/api/src/routes`; logic in `src/domain`.
- Use the repository logger, not `console.log`.
- Every behavior change needs a test.

## Boundaries
- Do not edit `infra/prod/**`, `.env*`, or `gen/**`.
- Ask before adding a dependency.

## Done
- `./scripts/verify.sh` passes.
- The change is reviewable in one sitting.
```

What it removes: the wrong turns that come from inferring commands, conventions and boundaries.

Context cost: one file, a small amount in every session. Keep it short or the context cost outweighs the benefit.

When it pays: when the agent's guesses about commands, conventions or boundaries cost more than a short file costs to keep current.

Write it by hand. The published case where context files raised cost and did not improve success used generated files, which is the case to avoid. One short layout block is not an architecture essay; keep it to paths and one line each. Leave out README copies, long architecture essays, generic advice, secrets and auto-generated summaries.

Artifact: a screenshot of the file beside the session that used it.

### 4.2 Scripts

What it is: one command per intent, with a stable interface: `setup`, `verify`, `test-one <path>`, `reproduce <issue>`, `scaffold`.

What it removes: the rounds the agent spends discovering how to run anything, and the long sessions that are mostly failed commands.

Context cost: low. The agent has to know the command's name, which means AGENTS.md or an instruction has to say it, and the script's output enters the session.

Design rules: non-interactive, `--json` where useful, stable exit codes, no color when not a TTY, bounded output, fast failure.

When it pays: as soon as a procedure repeats.

Artifact: one split still, `verify` failing on the left and passing on the right.

### 4.3 Skills

What it is: a repeatable procedure packaged as a directory with a `SKILL.md`, loaded in stages.

What it removes: inconsistent execution of a procedure that already works when someone remembers all the steps.

Context cost: low. The name and description load first; the body loads only when relevant.

Keep templates and fixtures in separate files, mark background knowledge as not user-invocable, and test each skill on a task the agent has not seen.

When it pays: the same procedure repeats across tasks and needs detail. Do not package a one-off.

Artifact: a screenshot of a skill being selected, with the name and description visible before the body.

### 4.4 Hooks

What it is: commands that run on harness events rather than on the model's decisions. In Copilot these are `preToolUse`, `postToolUse`, `sessionStart` and `sessionEnd`; other harnesses use different names for the same idea.

What it removes: the failures that a written instruction cannot prevent, such as editing a protected path or leaving a file unformatted.

Context cost: low. A hook runs outside the model, but a blocked action is reported back to it.

Start with an audit hook that records tool names only, then add enforcement for the paths and commands that actually caused trouble.

When it pays: the agent can reach files, credentials or infrastructure it must not change. Hooks are the only repository change that does not depend on the model reading something.

Artifact: a still of a blocked edit and the message the agent receives.

### 4.5 MCP

What it is: a server that gives the agent access to something the checkout does not contain, such as issues, CI logs or a database.

What it removes: guesses about work that lives outside the repository.

Context cost: high. Tool definitions and their output both enter the session, which is more than any other area here adds. Prefer a local script when one can do the job, and confirm the tool is actually called before keeping the server.

When it pays: the answer lives outside the checkout. If it does not, an MCP server adds cost for nothing.

Artifact: a still of the tool call and its result in the session.

### 4.6 Orchestration

What it is: the tool a person uses to run and control several agents: start sessions, see what each one changed, review the work, and intervene.

What it removes: blind review when several sessions run at once, and the time lost moving between them.

This is not a repository change and it is not an optimization to the agent's environment. Nothing in the checkout changes; what changes is how the person supervises the work.

Context cost: none in the agent's window. The cost is the person's attention, not the model's context.

It amplifies whatever the repository already does. Do not scale to several sessions until one session reliably passes `verify`.

Artifact: a screenshot of two sessions side by side with their file changes.

When it pays: review, not generation, is the bottleneck. The worked example is the Agents window in VS Code, which lists sessions with status and file changes, isolates each in a folder or worktree, and brings changes, tasks, terminal and browser into one view. Other harnesses offer the same shape under different names.

Custom agents and subagents define roles: planner, implementer, reviewer, researcher. Review and research agents get read-only tools. Worktrees give each parallel task its own checkout, with per-worktree ports, dependencies and database. Two caveats: worktree sessions usually run with broad approvals, so sandboxing and hooks still apply; and several sessions reporting "done" with failing tests point to a repository problem, not a model problem.

## 5. Improvements, not a sequence

Each area is independent, and each is worth adding when its trigger appears; the table in section 4 is the trigger list. What matters is how many of the repository's questions you have answered, not whether you completed a list or followed a sequence.

Two examples of how the choice goes:

- A repository where the agent guesses the test command needs a script, not a skill.
- A repository where the agent edits generated files starts with a boundary line in AGENTS.md. If the edits continue, a hook is the next change.

Add one at a time. If something gets worse, one change is easy to undo.

Stop when the friction is gone. Friction is gone when none of the four signals in section 6.2 stands out in normal sessions: changes pass `verify` without repair, the same correction stops repeating, session length is comparable to similar tasks, and spend no longer stands out.

Stopping is a decision about what to do next, not a verdict on the setup. A signal that stops standing out means there is no next fix to make now, not that the setup is proven.

## 6. Monitoring

### 6.1 Why monitor

The talk is about everyday work, and everyday work holds nothing still: tasks differ, the harness changes, and nobody has time to run the same task twenty times.

What normal work does provide is a stream of observations. Monitoring means keeping an eye on that stream after the setup: is the agent running correctly, how many tokens is it spending, how long do sessions take, and where is the next thing worth fixing.

The purpose is not to prove a number. It is to notice the next optimization.

It is also how you would notice that the claim did not hold. If the same corrections and the same long sessions continue after you have answered a question, that change did not address the cause. Notice is the right verb: this is a pattern across sessions, not a verdict on the setup, and the difference is the subject of 6.5.

### 6.2 What to watch

Four signals, all of them visible in the normal course of work:

| Signal | What it looks like | What it usually means |
|---|---|---|
| Correctness | A change that passes review but fails `verify`; edits to protected files; tests weakened or deleted | A missing answer or a missing guardrail |
| Tokens | Spend out of proportion to the size of the change; repeated reads of the same files | The agent is searching for something the repository should state |
| Time | Sessions that run long, or much longer than similar tasks | Stuck, exploring, or waiting on a slow check |
| Human effort | Repeated corrections of the same kind; repair after "done" | The repository is not answering a question the person keeps answering |

The signals are read together. Fast and wrong is not an improvement, and cheap and wrong is not a saving.

### 6.3 From signal to fix

Monitoring becomes useful when it ends in a small change. The loop:

1. Notice the signal in a normal working session.
2. Find the question the agent had to infer.
3. Apply the smallest fix that answers it.
4. Keep watching the same signal.

Some examples:

- The agent keeps re-reading the same directories, so sessions get long. Add the short layout block to AGENTS.md. If the reading continues, the conventions are a second change.
- The agent runs the whole suite for a one-line change. Add a `test-one` script first. Once it works, naming it in AGENTS.md is a second change.
- The agent edits generated files. Add the path to the boundaries first. If the edits continue, the `preToolUse` hook is a second change.
- A slow check makes every iteration expensive. Split the check, or make the fast one the default.
- The same review comment appears in three pull requests. Move it to AGENTS.md, or package it as a skill.
- The agent never calls the MCP server you added. Remove the server.

The last one is the point of the loop: monitoring should also remove things. An optimization that does not earn its place is a cost.

A conditional second step is the next iteration, not a second change made at the same time. Try the first one, look at the next session, then decide.

For slide 21, use one of your own sessions and fill in five fields: the task, what happened, the signal that stood out, the smallest fix, and what the next similar session did. An illustrative shape, not a measured result:

| Field | Example |
|---|---|
| Task | A one-file change that should take one session |
| What happened | Three sessions; most of the time went to re-reading the same directories, and the test command was guessed |
| Signal | Time and tokens, both above the range for that kind of change |
| Fix | A short layout section in AGENTS.md. One change, not two |
| Next session | One session, no repeated reading. The test command was still guessed, which is the next improvement |

### 6.4 Where the data already is

No new tooling is needed:

- The session list shows status, harness and the files each session changed.
- The harness reports token usage per session or per task.
- CI shows whether the change actually passed.
- Review comments show the corrections a person keeps making.
- A short note after a bad session is enough to spot a pattern.

### 6.5 What monitoring is not

- Not a verdict. You are looking for patterns across normal sessions, not a score for the setup. That is why 6.1 says notice, not prove.
- Not a dashboard to admire. A number that does not lead to a change is noise.
- Not a reason to chase token counts. Read spend next to correctness.
- Not a one-time check. The repository, the harness and the tasks keep changing.

## 7. DevEx for coding agents

Developer experience has two halves. One is the experience of the person: satisfaction, well-being, flow. The other is the conditions of work: instructions, tools, feedback, context, guardrails and how the work is supervised. The second half is the one that transfers to an agent.

> DevEx for Coding Agents: applying the conditions of work to the environment an agent operates in.

This section adds no new technique. It names what the previous slides did and says how much confidence to place in each part.

The six conditions, and where the talk met each one:

| Condition | In this talk | Where it lands |
|---|---|---|
| Instructions | AGENTS.md, skills | Questions 1 to 4 |
| Tools | Scripts, MCP | Questions 1, 5 |
| Feedback | `verify`, the fast check | Question 2 |
| Context | A short AGENTS.md, skills that load in stages | The budget for all six, not a question |
| Guardrails | Hooks, permissions | Question 4 |
| Orchestration | The Agents window and the roles | Question 6 |

What transfers, one line each:

- Instructions: nothing is absorbed, so the answers have to be written down.
- Tools: the environment decides what work is possible.
- Feedback: the loop repeats many times per task, so a slow one is paid repeatedly.
- Context: the budget is fixed, so more input can hurt.
- Guardrails: enforcement cannot depend on the model complying.
- Orchestration: supervision is work, and it needs a surface.

Tacit knowledge is the partial case. It does not transfer as tacit knowledge, because nothing is absorbed. It transfers once it is externalized: documentation becomes runtime input, not onboarding. That is what AGENTS.md is.

What does not: the experience half. Satisfaction, well-being and team culture belong to the person. The talk does not claim agents have them, and nothing here depends on that claim.

The name is borrowed. Spotify, Modal and incident.io have published it. Use it to label what the audience just watched, not as a new idea.

One line for the slide: a person learns the setup once, an agent pays for it every session.

### 7.1 The transfer map

The evidence file calls this the core of the frame: each DevEx principle, its agent analogue, and how much confidence to place in it. Slide 24 shows this, with the strength column.

The six conditions are what the talk did; the seven principles are why, and how much confidence to place in each. Rows 1 to 4 justify the environment work; rows 5 to 7 justify the monitoring.

| DevEx principle | Agent-side analogue | Strength |
|---|---|---|
| The environment shapes performance more than the individual | The environment shapes agent performance more than the model | Strong |
| Reduce friction in the working environment | Remove ambiguous instructions, slow setup and missing commands | Strong |
| Shorten feedback loops | Faster tests, deterministic scripts, clear failures | Strong |
| Reduce cognitive load | Smaller context, fewer tools, less irrelevant instruction | Strong as a design principle; context rot is the agent equivalent |
| Triangulate perceptions with system data | Agent self-report plus what the session actually produced | Moderate |
| Read several dimensions together | Success, cost, time and rework side by side | Strong |
| Avoid one-number productivity metrics | Do not make any single dimension the target | Strong |

The talk uses the strong rows as the frame and says out loud that the self-report row is the thin one. It does not claim the map is new; it claims the mapping is useful.

## 8. What the audience leaves with

- An idea they can use: the agent already works; the repository decides how effectively.
- Six questions a repository should answer, five for the agent and one for the person running it.
- Six areas, each with the friction it removes and the trigger that makes it worth adding.
- A rule: add one improvement at a time, and stop when the friction is gone.
- A habit: watch correctness, tokens, time and human effort, then make the next small fix.
- A name for the work you saw: DevEx for Coding Agents, with the six conditions it maps to.

## Appendix A. Slide outline

1. Title.
2. The agent already works. This talk is about how well.
3. What "not effectively" looks like.
4. Why: every unanswered question becomes an inference. The six questions arrive at slide 7.
5. The reframe: small optimizations to the working environment.
6. What this talk is not: no model choice, no prompting, no checklist, no measurement.
7. Six questions the repository should answer.
8. Six areas, in no required order, with the trigger for each.
9. AGENTS.md: commands, layout, conventions, boundaries, done.
10. A bad AGENTS.md against a good one.
11. Scripts: one command per intent.
12. Skills: procedures on demand.
13. Hooks: rules the model cannot skip.
14. MCP: only for a real gap.
15. Orchestration: the tool a person uses to control agents, not a repository change.
16. Custom agents, subagents and worktrees.
17. Add improvements one at a time, and stop when the friction is gone.
18. Monitoring: the work after setup.
19. Four signals to watch: correctness, tokens, time, human effort.
20. From signal to fix: the loop.
21. One example: a long session and what fixed it, filled from your own log.
22. DevEx for coding agents: the six conditions of work.
23. The conditions mapped to the areas and questions, and the half that does not transfer.
24. The transfer map: seven principles, and why they sit behind the six conditions.
25. Anti-patterns: the five that matter, with the rest on a handout.
26. Lessons.
27. Close: the model is the same; the environment is what you can change.

Artifact source: one synthetic repository, named on slide 9, with four modules (API, web, infrastructure, database) and seeded bugs. Every still comes from it. It is constructed to show the pattern, not to be representative; say this out loud when the first still appears.

## Appendix B. Anti-patterns to name on stage

For the stage, name five: installing everything before noticing the friction, auto-generating AGENTS.md, long instruction files, hooks that block everything, and watching tokens without checking correctness. The rest go on a handout.

Handout: one page with all eleven bullets below and a one-line explanation each, shipped with the deck.

- Installing everything before noticing which friction you have.
- Auto-generating AGENTS.md and never reading it.
- Long instruction files that duplicate documentation.
- Installing MCP servers on the chance they will be useful.
- Adding tools without checking whether they are invoked.
- Hooks that block everything or fail silently.
- Parallel agents without port and dependency isolation.
- Orchestrating before the single-agent case is stable.
- Watching token counts without checking whether the change was correct.
- Building a dashboard without a habit of acting on it.
- Treating a configuration file as proof that a capability works.

## Appendix C. Claims to avoid

- That the repository is the only factor. The model matters too, and it is held fixed here.
- That the agent cannot work without this setup. It can; the setup changes how well.
- That published numbers were produced on repositories like the audience's.
- That the AGENTS.md gain is pure cause and effect.
- That every part of DevEx transfers. The experience half does not, and tacit knowledge transfers only once it is written down.
- That AGENTS.md always helps. A generated context file can cost more than it saves.
- That orchestration is a repository change. It is the tool a person uses to control agents.
- That the framing is new. Spotify, Modal, incident.io and others have made the same case.
- That agents have feelings or satisfaction. Talk about outcomes.

## Appendix D. Sources and further reading

- Lulla et al., [On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents](https://arxiv.org/html/2601.20404v2)
- Gloaguen et al., [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988)
- [How Do AI Agents Spend Your Money?](https://arxiv.org/html/2604.22750v3)
- [METR: Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study)
- [DORA 2024 report](https://cloud.google.com/blog/products/devops-sre/announcing-the-2024-dora-report)
- [Spotify: Coding Is No Longer the Constraint](https://engineering.atspotify.com/2026/6/code-with-claude-coding-is-no-longer-the-constraint)
- [Modal: Agents need good developer experience too](https://modal.com/blog/agents-devex)
- [incident.io: DevEx matters for coding agents, too](https://incident.io/blog/ai-developer-tools)
