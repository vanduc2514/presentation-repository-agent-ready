# Agent-Friendly Repository Technology Guide

Research date: 2026-10-22

Purpose: practical setup guidance for the six areas in [idea.md](./idea.md): AGENTS.md, helpful scripts, MCP, skills, hooks, and orchestration (custom agents and worktrees). The focus is a repository where coding agents work more effectively, not a survey of agent products.

---

## 1. Levels of setup

These levels describe coverage, not a required sequence. Each level is useful on its own, and later levels assume the earlier ones are stable.

| Level | What it adds | Why |
|---|---|---|
| **0. Ad hoc** | The repository as it is, nothing standardized | Results depend on the prompt and the agent's mood |
| **1. Hygiene and instructions** | Reproducible setup, one-command verify, consistent structure, short AGENTS.md | Removes the largest sources of friction |
| **2. Procedures** | Scripts for setup, test selection, reproduction, scaffolding; project skills | Turns repeated instructions into deterministic actions |
| **3. Guardrails** | Hooks, permissions, sandboxing | Enforces behavior that must not depend on model compliance |
| **4. External capability** | MCP servers, only where the repository lacks something | Adds tools without duplicating local files |
| **5. Orchestration** | Agents window, custom agents, subagents, worktrees | Runs many sessions and keeps them reviewable |

A team can stop at level 2 and still get most of the benefit. Levels 4 and 5 add capability and cost; they are not defaults.

Rule of thumb:

- **Facts** go in AGENTS.md.
- **Procedures** go in skills or scripts.
- **External capability** goes in MCP.
- **Enforcement** goes in hooks.
- **Roles** go in custom agents.
- **Supervision** goes in the Agents window.
- **Parallelism** goes in worktrees.

---

## 2. Decision matrix

| Technology | Problem it solves | Context cost | Risk | Evidence | Use when |
|---|---|---|---|---|---|
| **Hygiene** | Non-reproducible setup, slow feedback | None | Low | Strong | Always |
| **AGENTS.md** | Agent does not know commands, conventions, boundaries | Low if short | Low | Mixed | Always, kept short |
| **Scripts** | Repeated exploration and manual commands | Low | Low | Moderate | A procedure repeats |
| **Skills** | Reusable procedures and domain knowledge | Low, progressive | Low-medium | Moderate | A procedure repeats and needs detail |
| **Hooks** | Behavior that must be deterministic | None | Medium | Strong mechanism | Safety or formatting must be enforced |
| **MCP** | Access to external systems or live data | High | Medium-high | Weak-moderate | The repo cannot provide the capability |
| **Agents window** | Supervising many sessions across workspaces | None | Medium | Moderate | More than one task runs at a time |
| **Custom agents** | Role separation and smaller contexts | Reduces main context | Medium | Moderate | Roles and handoffs repeat |
| **Worktrees** | Parallel tasks without file conflicts | None | Medium | Strong mechanism | Two or more independent tasks run at once |

---

## 3. Hygiene

Do this before adding agent-specific technology. Hygiene is a prerequisite, not one of the agent-specific areas.

### 3.1 Reproducible environment

- Pin the runtime: `.nvmrc`, `.tool-versions`, `mise.toml`, or a dev container.
- Commit the lockfile. Use `npm ci`, `pnpm install --frozen-lockfile`, or the equivalent.
- Make setup non-interactive and idempotent.
- Document the one command that installs everything.

### 3.2 One-command verification

Every repository should have a single command that runs what CI runs:

```json
{
  "scripts": {
    "setup": "npm ci",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "verify": "npm run lint && npm run typecheck && npm run test"
  }
}
```

The agent should not have to discover five commands. Give it one command for "is this change good?" and one for "run the smallest relevant test." Ensure the test script is non-watch; an agent that runs `npm test` should not hang in a watcher.

### 3.3 Consistent structure

- Small modules with clear interfaces.
- Predictable file names and directory layout.
- Tests near the code they cover.
- One obvious place for API routes, data access, and shared types.
- No duplicate file names such as ten `index.ts` files.

Consistency helps agents find code and helps humans review it. It is the highest-leverage change in the repository and costs no context.

### 3.4 Fast feedback

Watch and budget the loops:

| Loop | Target |
|---|---|
| Hot reload | under 1s |
| Incremental type check | under 5s |
| Unit test suite | under 15s |
| `verify` | under 60s |
| Full CI | under 8 min |
| Clean install and first boot | under 5 min |

These are examples, not universal targets. The point is to make the loop fast enough that an agent can guess, check, and correct rather than trying to be right the first time.

---

## 4. AGENTS.md

### What it is

A cross-agent Markdown file at the repository root. Copilot and OpenAI Codex support it; other harnesses recognize it or have equivalents. VS Code also supports `.github/copilot-instructions.md` and `CLAUDE.md`.

### Setup

- Put the primary file at the repository root: `AGENTS.md`.
- For monorepos, add nested `AGENTS.md` files only where conventions differ.
- In VS Code, `chat.useAgentsMdFile` is enabled by default; nested discovery requires `chat.useNestedAgentsMdFiles` and is experimental.
- Copilot discovers instructions at the repository root, the session's working directory, and intermediate directories. Avoid conflicting guidance; there is no general precedence order across instruction files.

### What to include

Keep it short and specific:

```markdown
# AGENTS.md

## Project
TypeScript monorepo. Backend in `services/api`, frontend in `apps/web`.
Infrastructure in `infra` (Terraform). Database migrations in `db/migrations`.

## Commands
- Setup: `./scripts/setup.sh`
- Verify: `./scripts/verify.sh`
- Test one file: `./scripts/test-one.sh <path>`
- Reproduce a bug: `./scripts/reproduce.sh <issue>`

## Conventions
- API handlers go in `services/api/src/routes`; business logic in `services/api/src/domain`.
- Use the repository logger, not `console.log`.
- Every behavior change needs a test.

## Boundaries
- Do not edit `infra/prod/**`, `.env*`, or generated files in `gen/**`.
- Ask before adding a dependency.
- Never run destructive database commands against a shared environment.

## Done
- `./scripts/verify.sh` passes.
- The change is small enough to review in one sitting.
```

### What not to include

- A copy of the README.
- Long architecture essays.
- Generic advice such as "write clean code."
- Secrets, tokens, or internal URLs.
- Auto-generated summaries of the codebase.

### Evidence

- One study finds about **20% lower output tokens** and **20% faster completion** with a repository-maintained AGENTS.md, on small tasks.
- Another finds LLM-generated context files slightly reduce success while raising cost **20-23%**; developer-written files help only marginally.
- Practical rule: write it by hand, keep it short, and measure.

### What to watch

- Task success.
- Output and cached input tokens.
- Wall-clock time.
- Number of failed commands before the first useful action.

---

## 5. Helpful scripts

### What they are

Small, deterministic commands that replace exploratory work: setup, test selection, reproduction, scaffolding, formatting, and verification.

### Design rules

Follow the Command Line Interface Guidelines ([clig.dev](https://clig.dev/)):

- **Non-interactive by default.** Provide `--yes`, `--no-input`, and `CI=1`.
- **Machine-readable output.** Provide `--json`; keep human diagnostics on stderr.
- **Stable exit codes.** `0` success, non-zero failure; do not swallow errors.
- **No color when not a TTY.** Respect `NO_COLOR` and `--no-color`.
- **Idempotent.** Running setup twice should be safe.
- **Bounded output.** Print a summary, not thousands of log lines. Cap or redirect raw logs.
- **Fast failure.** Stop at the first error unless the user asks for a full report.
- **Timeout.** Long commands should fail with a clear message rather than hang.
- **Discoverable.** `--help` should show examples, not only flags.

### Repository layout

```text
scripts/
├── setup.sh          # install dependencies, create local env, seed data
├── verify.sh         # lint + typecheck + tests + build
├── test-one.sh       # run the smallest relevant test for a path
├── reproduce.sh      # reproduce a bug with a failing test
├── scaffold.sh       # create a module with the standard layout
└── db-reset.sh       # reset local database only
```

Example `verify.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

npm run lint
npm run typecheck
npm test
npm run build
```

Example `test-one.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

if [[ $# -ne 1 ]]; then
  echo "usage: $0 <path>" >&2
  exit 2
fi

# Adjust the path filter to your test runner (Vitest, Jest, etc.)
npm test -- "$1"
```

### Evidence

- Explore plus Fix phases account for about **64%** of agent rounds; most spend goes to navigation and iteration.
- Databricks found that harness context management drove more than **2x** cost differences, with one harness sending about **3x** less context per turn.
- A script that replaces ten exploratory tool calls with one deterministic command attacks both problems.

### What to watch

- Steps per task.
- Failed commands per task.
- Time from task start to first passing test.
- Context tokens per turn.

---

## 6. MCP

### What it is

The Model Context Protocol is an open standard for connecting AI applications to external systems. A server exposes tools, resources, and prompts.

### Use when

- The agent needs live external data or actions the repository cannot provide.
- A stable, maintained server already exists.
- The capability is worth the context and security overhead.

### Avoid when

- A local script can do the same thing.
- The server duplicates repository files.
- You have not reviewed the publisher, permissions, and failure behavior.
- The tool is installed "just in case."

### Setup

- Shared team configuration: workspace-root `.mcp.json` (portable format, top-level `mcpServers`).
- VS Code configuration: workspace `.vscode/mcp.json` or user profile (VS Code format, top-level `servers`).
- Global Copilot configuration: `$COPILOT_HOME/mcp-config.json` or `~/.copilot/mcp-config.json`.
- Do not copy a configuration between portable and VS Code formats without checking the schema.

Illustrative shape:

```json
{
  "servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

Check the current configuration reference before committing; the format has changed and differs by harness.

### Verify it actually works

A configuration is not proof that the agent can use the server.

1. Ask the agent to use the server by name for a low-impact action.
2. Confirm a tool from that server was called.
3. Confirm the output came from the tool.
4. Confirm the answer reflects the output.

If the tool never loads or is never called, remove the server.

### Security

- Local MCP servers can run code on your machine.
- Tools can access external services with your permissions.
- Review the publisher, command or endpoint, and requested access before installing.
- Enable sandboxing and require approval for writes and network access.
- Treat tool output as untrusted; it can contain prompt injection.
- Keep credentials out of configuration files. Use the supported authentication and secret mechanisms.
- Limit the number of tools. Unused tools add context and attack surface.

### What to watch

- Tool call rate per task.
- Load failures and timeouts.
- Steps and cost on sessions where the server is used.
- Whether the agent used the server or a local alternative.

---

## 7. Skills

### What they are

A skill is a directory with a `SKILL.md` file. It packages instructions, scripts, and resources and loads them progressively: name and description first, body only when relevant, resources only when referenced.

### Setup

Project skill locations in VS Code:

- `.github/skills/`
- `.claude/skills/`
- `.agents/skills/`

Personal locations:

- `~/.copilot/skills/`
- `~/.claude/skills/`
- `~/.agents/skills/`

The directory name and the `name` field should match. For extension-contributed skills, they must match; a mismatch prevents loading. Matching is recommended for project skills as well.

### Format

```markdown
---
name: reproduce-bug
description: Reproduce a bug with the smallest failing test before changing code. Use when asked to fix a bug or investigate a regression.
---

# Reproduce a bug

## When to use
Use this when a bug report arrives without a failing test.

## Steps
1. Read the issue and identify the smallest observable behavior.
2. Find the closest existing test.
3. Write a failing test that reproduces the behavior.
4. Run `./scripts/test-one.sh <path>` and confirm the failure.
5. Only then change production code.
6. Run `./scripts/verify.sh`.

## Output
Return the failing test path, the observed failure, and the root cause.
```

### Design rules

- The description is the trigger. State what the skill does and when to use it.
- Keep the body procedural. Link to scripts and references instead of pasting them.
- Put large templates, fixtures, and examples in separate files.
- Prefer one skill per repeatable procedure.
- Test that the skill is discovered and invoked for realistic prompts.
- For long-running or noisy procedures, consider a forked context so only the result returns to the main conversation.
- Review shared skills before use; they can include scripts.

### Control

- `user-invocable: false` for background knowledge the model should load when relevant.
- `disable-model-invocation: true` for on-demand skills only.
- Both disable the skill.

### What to watch

- Invocation rate on relevant tasks.
- Task success on sessions where the skill is invoked.
- Steps and context tokens.
- Whether the agent follows the procedure or improvises.

---

## 8. Orchestration: the Agents window and custom agents

Orchestration is a workflow change, not a repository change. Nothing in the checkout changes; what changes is how a person runs, sees and reviews the agents.

It has two parts:

- **A surface** to run, monitor, and review many sessions: the VS Code **Agents window**.
- **Roles** that define how each agent behaves: custom agents, subagents, and handoffs.

Most teams stop before this layer. Treat it as an experiment until the single-agent path is stable.

### 8.1 The Agents window as the orchestrator UI

#### What it is

The Agents window is a **dedicated VS Code window**, separate from the editor, built for assigning higher-level tasks, coordinating work across projects, monitoring multiple sessions, and reviewing outcomes in one place. It is in Preview.

Two interfaces, two jobs:

| Interface | Best for |
|---|---|
| **Chat view** | Collaborating over code in the current workspace, with the editor and terminal beside you |
| **Agents window** | Orchestrating tasks across workspaces and projects, with many sessions running at once |

This is the piece most "agent setup" guides miss. The repository can be perfectly configured and still produce chaos if a human cannot see which session is doing what. The Agents window is where that supervision happens.

#### Layout

| Area | What it shows |
|---|---|
| **Sessions list** | Sessions across all workspaces, with status, harness, and file-change stats |
| **Customizations panel** | Agents, skills, instructions, hooks, and Automations when enabled |
| **Chat area** | The active session's conversation |
| **Changes panel** | Branch changes, uncommitted changes, all changes, or the last agent turn |
| **Files panel** | The active session's workspace folder or worktree |

Only the **active session** drives these panels. Selecting a session changes what Chat, Files, Changes, Terminal, and Browser show.

#### Open and drive it

| Action | How |
|---|---|
| Open the window | **Open in Agents** in the title bar, **Chat: Open Agents window** from the Command Palette, or `code --agents` |
| Open from a browser | `insiders.vscode.dev/agents` (requires GitHub sign-in) |
| New session | ⌘N / Ctrl+N, or **+** on a workspace in the sessions list |
| Start in the background | Alt+Enter instead of Enter |
| Start from a PR | **New Session from Pull Request** on a workspace |
| Quick, workspace-less chat | **New Quick Chat** |
| Ask without polluting the chat | `/btw <question>` side chat, or select response text and **Ask Question** |
| Split the chat area | Drag a chat tab to an edge, or **Sessions: Split Chat Group Right/Down** |
| Triage | Filter by state, group by workspace or timeframe, create custom groups, pin, archive |

For the demo, enable the features you plan to show ahead of time and rehearse with the same repository. Preview features move between releases.

#### Start a session

1. Select **New**, or **+** on a workspace in the sessions list.
2. Choose **Folder** or **Repository** as the execution target. Use **Remote Setup** for SSH or a dev tunnel.
3. Attach context: additional projects, GitHub issues, or pull requests. A pasted issue or PR URL becomes a context attachment.
4. Choose the harness, model, permission level, and isolation mode.
5. Enter the prompt. Press Alt+Enter to start it in the background without leaving the current session.

Sessions can also start from a pull request. That creates an isolated worktree tracking the PR branch, with the PR title, changes, and comments as context. **Sync Changes** pushes commits back to the PR branch.

#### Choose where the agent works

| Choice | Use it for | Trade-off |
|---|---|---|
| **New Worktree** | Parallel tasks that must not touch your active workspace | Starts from committed Git state; you integrate the result |
| **Folder** | Small interactive tasks that need your uncommitted changes | Edits appear immediately in your workspace |

Security note for the talk: **worktree sessions use Allow all and do not offer manual approval prompts.** Code isolation is not a security boundary. Use sandboxing and hooks for that, and say so on stage.

Related details:

- Worktrees start from committed state. Uncommitted, untracked, and git-ignored files such as `.env` and `node_modules` are absent unless you configure include or symlink settings.
- Dev Container sessions run in the container workspace and cannot be combined with New Worktree.
- An idle container stops after five minutes by default. Approval prompts count as activity and keep it alive.

#### Review and finish

- **Files** and **Changes** panels for diffs; leave range-based feedback on a changed file.
- **Integrated browser** and **Tasks** to validate the change in the same window.
- **Commit Changes** with a generated message.
- **Create PR** with draft, merge method, and optional agent merge (experimental) to address reviews, fix failed checks, resolve conflicts, and merge when ready.

#### Monitoring many sessions

- The sessions list spans workspaces, so you can supervise unrelated projects from one place.
- Filter by state, group by workspace or timeframe, or create custom groups.
- Pin and archive sessions to keep the list usable.
- The Changes pill on a session header combines edits from all chats in that session.

#### Why this matters for the talk

The orchestrator is a DevEx surface in its own right. It makes the repository's weaknesses visible at scale:

- A repository without a verify command produces sessions that end "done" with failing tests.
- A repository without boundaries produces sessions that edit generated files and production config.
- A repository without isolation produces worktree sessions that silently inherit none of the local setup.
- A repository without fast tests produces five sessions all waiting on the same slow loop.

The demo can use this: run three sessions in parallel on the bare repository, then again after the setup work. The difference in review time is the argument.

#### Demo script: three sessions, before and after

Run this twice, once on the bare repository and once after the setup work.

1. Open the Agents window on the repository.
2. Start three sessions in **New Worktree**, one prompt each:
   - "Add validation to the create-user endpoint."
   - "Fix the failing pagination test."
   - "Add a health check route with a test."
3. Press Alt+Enter for each so they run in the background.
4. Start a timer. Do not intervene.
5. When each session reports done, record:
   - Did `verify` pass without you running anything?
   - How many files did you have to check to trust the result?
   - How many follow-up prompts did you send?
   - How long did review take?
6. Repeat on the configured repository and compare.

What the audience sees on the bare repository:

- Three sessions, three different ideas of how to run tests.
- Sessions that mark themselves done with failing checks.
- Review time dominated by figuring out what the agent actually did.

What they see after the setup work:

- The same three prompts, all ending in the same `verify` command.
- Fewer changed files outside the task.
- Review starts from a diff that passes checks, not from a repair job.

#### Caveats

- Agents window features are in Preview and change between releases.
- Worktree sessions skip manual approvals; rely on sandboxing, hooks, and review.
- More sessions is not automatically better. Parallelism raises review load and merge cost.
- The window does not fix a bad repository. It only makes the consequences easier to see.

### 8.2 Custom agents: defining the roles

A custom agent is a Markdown file with YAML frontmatter that defines instructions, tools, and optionally a model. It appears in the agent picker and can run as a subagent or receive handoffs.

#### Setup

- Workspace: `.github/agents/*.agent.md` (or `.md`).
- Claude format: `.claude/agents/*.md`.
- User: `~/.copilot/agents` or `~/.claude/agents`.

Example planner:

```markdown
---
name: Planner
description: Generate an implementation plan without changing code
tools: ['read', 'search']
---

Research the requested feature or refactoring task.
Do not edit files. Return a Markdown plan with:
- Overview
- Requirements
- Implementation steps
- Tests to add or update
- Risks
```

Handoff example:

```yaml
handoffs:
  - label: Implement Plan
    agent: agent
    prompt: Implement the plan outlined above.
    send: false
```

`send: false` lets a human review the prefilled prompt before implementation.

### 8.3 Subagents and handoffs

- A subagent is a **context-isolated** agent. The main agent delegates, and only the final result returns. Intermediate exploration stays out of the main conversation.
- Multiple subagents can run in parallel. The UI shows which task is running, which agent handles it, and the full prompt and result.
- Combine subagents with custom agents to give each one a role and a tool set: a read-only research agent, an implementation agent with edit access, a security agent that only scans.
- Handoffs chain agents into a workflow: plan, then implement, then review. Use `send: false` to keep a human checkpoint between stages.

### 8.4 Patterns

| Agent | Tools | Purpose |
|---|---|---|
| **Planner** | Read-only | Produce a plan without changing code |
| **Implementer** | Edit and terminal | Apply the plan |
| **Reviewer** | Read-only | Check correctness, tests, and conventions |
| **Researcher** | Read-only, run as subagent | Explore the codebase without polluting the main context |

### 8.5 Rules

- Give each agent the smallest tool set it needs.
- Use read-only tools for review and research agents.
- Prefer subagents when the work should not consume the main conversation's context.
- Use handoffs for staged workflows with human review. Handoffs are a Local-harness feature; other harnesses have their own orchestration.
- In the Copilot harness, agent instructions take precedence over project instructions when they conflict. Review agent files before sharing them.
- Do not create an agent for a one-off task. Create one when the role, tools, or model recur.
- In the Agents window, verify the isolation mode and permission level before starting a session. Worktree sessions do not prompt for approval.
- Do not scale to many parallel sessions until one session consistently passes `verify`.

### 8.6 What to watch

Session and orchestration signals:

- Sessions that end with `verify` passing, as a share of all sessions.
- Human review minutes per session.
- Rework after a session is marked done.
- Share of sessions merged without a follow-up session.
- Main-conversation context tokens with subagents versus without.
- Cost per merged change, not cost per session.
- Parallel sessions versus sequential: wall-clock time, conflicts, review load.

---

## 9. Hooks

### What they are

Hooks run commands at lifecycle points in an agent session. They are the only lever here that can enforce behavior deterministically. VS Code agent hooks are currently a Preview feature.

### Setup

Repository hooks live in `.github/hooks/*.json`.

```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "command": "node .github/hooks/log-tool-use.cjs",
        "timeoutSec": 10
      }
    ]
  }
}
```

A command hook receives event data as JSON on stdin and returns one JSON object on stdout. `{}` means no decision. A denial looks like:

```json
{
  "permissionDecision": "deny",
  "permissionDecisionReason": "This operation requires manual review."
}
```

### Events

| Goal | Event |
|---|---|
| Add context when a session starts | `sessionStart` |
| Observe a prompt | `userPromptSubmitted` |
| Inspect or decide about a tool call | `preToolUse` |
| Validate a result or add context | `postToolUse` |
| Save information before compaction | `preCompact` |
| Observe subagents | `subagentStart`, `subagentStop` |
| Check whether the agent should finish | `agentStop` |
| Clean up | `sessionEnd` |

### Guardrail patterns

- **Block destructive commands** in `preToolUse`: `rm -rf`, `git push --force`, `DROP TABLE`, writes to protected paths.
- **Format and lint changed files** in `postToolUse`.
- **Run a targeted test** after edits to a specific directory.
- **Append an audit log** of tool names and outcomes.
- **Inject environment context** at `sessionStart`: branch, runtime version, task ID.
- **Clean up** temporary files at `sessionEnd`.

### Rules

- Hooks are executable code. Review them before use.
- Validate event input. Do not interpolate untrusted tool arguments into a shell command.
- Keep credentials out of configuration, scripts, output, and logs.
- Set timeouts and test failure behavior. A timed-out `preToolUse` may fall through to normal permissions.
- Hooks do not replace permissions, sandboxing, or enterprise controls.
- Start with an audit hook that records tool names, not arguments or results.

### What to watch

- Unsafe actions blocked.
- Rework after the agent declares done.
- Hook latency.
- Silent failures and timeouts.

---

## 10. Worktrees

### What they are

A Git worktree is a linked working directory that shares the repository's object database and refs but has its own HEAD, index, and files. It lets two agents work on different branches without file conflicts or `.git/index.lock` contention.

### Setup

```bash
# Create a worktree with a new branch based on main
git worktree add -b feat/auth ../repo-auth main

# List worktrees
git worktree list

# Remove when merged or abandoned
git worktree remove ../repo-auth

# Clean up stale references
git worktree prune
```

Open each worktree as a separate VS Code window. Run one agent per worktree.

### What worktrees do not solve

- **Port collisions.** Every dev server defaults to 3000, 5432, or 8080. Assign a per-worktree port range or use isolated containers.
- **Missing dependencies.** `node_modules` and virtual environments are not shared. Run setup in each worktree.
- **Missing environment files.** `.env` is gitignored. Create or symlink the local environment per worktree.
- **Shared databases and caches.** Use a schema or container per worktree, or serialize migrations.
- **Merge and review cost.** Parallel output creates more branches, conflicts, and review load.
- **Permission defaults.** In VS Code, worktree sessions can use broader approval defaults. Verify the session's permission level before running an agent.

### Rules

- One task per worktree.
- Name branches and directories predictably: `feat/auth`, `fix/api-timeout`.
- Remove worktrees when merged; prune stale entries.
- Do not run two agents in the same worktree.
- Watch cost per merged change, not agent throughput.

### What to watch

- Wall-clock time to merge.
- Merge conflicts per change.
- Review comments and rework.
- Cost per merged change.

---

## 11. Supporting infrastructure

### Dev containers

A `.devcontainer/devcontainer.json` makes the environment reproducible for humans and agents:

```json
{
  "name": "app",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:22",
  "postCreateCommand": "npm ci",
  "forwardPorts": [3000],
  "customizations": {
    "vscode": { "extensions": ["dbaeumer.vscode-eslint"] }
  }
}
```

Use it when the toolchain is complex or onboarding is slow. It adds Docker overhead and startup time; a pinned runtime plus lockfile may be enough for a small repository.

### Task runner

Use `just`, `make`, `mise`, or package scripts to expose one command per intent:

```text
setup
dev
lint
typecheck
test
test-one
verify
format
db-reset
```

The names should be consistent across repositories. Consistency is worth more than the perfect name in one repository.

### CI parity

The local `verify` command should run the same checks as CI. If CI runs a different command, the agent will pass locally and fail remotely.

### Observability

- Keep agent session logs or transcripts for review.
- Record token usage and cost per task.
- Record tool calls and failures.
- Keep a small evaluation task set with acceptance checks.

---

## 12. Reference stack for the demo repository

```text
.
├── AGENTS.md
├── .github/
│   ├── instructions/
│   │   └── api.instructions.md
│   ├── skills/
│   │   └── reproduce-bug/
│   │       ├── SKILL.md
│   │       └── failing-test-template.ts
│   ├── agents/
│   │   ├── planner.agent.md
│   │   └── reviewer.agent.md
│   └── hooks/
│       └── audit.json
├── .devcontainer/
│   └── devcontainer.json
├── scripts/
│   ├── setup.sh
│   ├── verify.sh
│   ├── test-one.sh
│   └── reproduce.sh
├── .mcp.json                 # only if an external capability is needed
└── docs/
    └── architecture.md
```

Start with the first four items. Add MCP only after a demonstrated gap. Add custom agents and worktrees only after the single-agent path is stable.

---

## 13. Steps and checkpoints

| Step | Change | Checkpoint |
|---|---|---|
| 0 | Hygiene: runtime pin, lockfile, `verify`, fast tests | `verify` passes from a clean clone |
| 1 | AGENTS.md | Same task: fewer failed commands, lower tokens, no success drop |
| 2 | Scripts | Fewer steps per task; time to first passing test drops |
| 3 | Skills | Skill is invoked on relevant prompts; success improves or stays flat |
| 4 | Hooks | Unsafe actions blocked; rework does not increase |
| 5 | MCP | Tool is actually called; cost and success justify it |
| 6 | Agents window | Sessions are visible, reviewable, and finish with `verify` passing |
| 7 | Custom agents | Main context shrinks; review findings and cost improve |
| 8 | Worktrees | Time to merge drops; conflicts and review load stay manageable |

There is no required order; the list is a way to check coverage. At every step: change one thing, watch the same signals for a few sessions, keep what helps, and remove what does not.

---

## 14. Anti-patterns

- Auto-generating AGENTS.md and never reviewing it.
- Long instruction files that duplicate documentation.
- Installing MCP servers "just in case."
- Adding many tools and skills without checking whether they are invoked.
- Hooks that fail silently or block everything.
- Running parallel agents without port, dependency, and database isolation.
- Orchestrating agents before a single agent runs reliably.
- Treating the Agents window as a dashboard to admire instead of a review queue.
- Starting worktree sessions and forgetting that they skip manual approvals.
- Measuring tokens saved instead of cost per successful task.
- Auto-approving destructive commands.
- Treating a configuration file as proof that a capability works.

---

## 15. Monitoring signals per technology

| Technology | Signals to watch |
|---|---|
| Hygiene | Whether a clean clone sets up; time to the first passing test |
| AGENTS.md | Failed commands; output tokens; time to completion; whether changes pass `verify` |
| Scripts | Failed commands per task; time to first passing test; context tokens |
| Skills | Whether the skill is invoked; whether changes pass `verify`; steps |
| Hooks | Unsafe actions blocked; rework after done; hook latency; silent failures |
| MCP | Whether the tool is called; load failures; steps; cost |
| Agents window | Sessions ending with `verify` passing; review minutes; rework; merged without follow-up |
| Custom agents | Main-context tokens; handoffs; review findings; cost |
| Worktrees | Time to merge; conflicts; review load; cost per merged change |

---

## 16. Sources

### Official documentation

- VS Code, [Use the Agents window](https://code.visualstudio.com/docs/agents/run/agents-window)
- VS Code, [Manage agent sessions in VS Code](https://code.visualstudio.com/docs/agents/run/sessions/manage-sessions)
- VS Code, [Build with AI in VS Code](https://code.visualstudio.com/docs/agents/overview)
- VS Code, [Your Home for Multi-Agent Development](https://code.visualstudio.com/blogs/2026/02/05/multi-agent-development)
- VS Code, [Agents feature page](https://code.visualstudio.com/features/agents)
- VS Code, [Use custom instructions in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- VS Code, [Use Agent Skills in VS Code](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- VS Code, [Custom agents in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- VS Code, [Configure agent hooks in Visual Studio Code](https://code.visualstudio.com/docs/copilot/customization/hooks)
- VS Code, [Add MCP servers in VS Code](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)
- VS Code, [Manage approvals and permissions](https://code.visualstudio.com/docs/agents/run/approvals)
- VS Code, [Secure AI-assisted development](https://code.visualstudio.com/docs/agents/run/security)
- [AGENTS.md standard](https://agents.md/)
- [Agent Skills specification](https://agentskills.io/specification)
- [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro)
- [Model Context Protocol security best practices](https://modelcontextprotocol.io/specification/2025-06-18/basic/security_best_practices)
- [Git worktree documentation](https://git-scm.com/docs/git-worktree)
- Upsun, [Git worktrees for parallel AI coding agents](https://developer.upsun.com/posts/ai/git-worktrees-for-parallel-ai-coding-agents)
- Augment Code, [Git worktrees for parallel AI agent execution](https://www.augmentcode.com/guides/git-worktrees-parallel-ai-agent-execution)
- [Dev Container metadata reference](https://containers.dev/implementors/json_reference/)
- [Command Line Interface Guidelines](https://clig.dev/)

### Evidence

- [On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents](https://arxiv.org/html/2601.20404v2)
- [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988)
- [How Do AI Agents Spend Your Money?](https://arxiv.org/html/2604.22750v3)
- [Databricks coding agent benchmark](https://www.databricks.com/blog/benchmarking-coding-agents-databricks-multi-million-line-codebase)
- [DevEx for Coding Agents research](./devex-for-coding-agents.md)
