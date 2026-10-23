# Building an Agent-Friendly Repository

[![vi](https://img.shields.io/badge/lang-vi-yellow.svg)](README.vi-VN.md)

A talk on small repository optimizations that make a coding agent work better: `AGENTS.md`, scripts, skills, hooks, MCP, isolation, and orchestration. Delivered at **Codegym Tech Talk** by Nguyen Van Duc.

Slides: [vanduc2514.github.io/presentation-repository-agent-ready](https://vanduc2514.github.io/presentation-repository-agent-ready/) · Vietnamese: [index.vi.html](https://vanduc2514.github.io/presentation-repository-agent-ready/index.vi.html)

## Background

A coding agent already works in a standard repository. It reads files, runs commands, edits code, and reports back; for a small local change it often succeeds without any special setup. The problem is not that it fails to run, it is how much of its effort is wasted rediscovering what the repository already knows.

Every unanswered question becomes an inference: which command runs the tests, which layer owns validation, which paths are off limits. Inferences cost tokens, time, and correctness. The talk frames these as friction points and treats repository configuration as a way to remove them, not as a way to make the model smarter.

Audience: students and early-career engineers who have used coding agents and watched them fail in ways that were hard to explain. Format: lecture with no live demo, so nothing needs to be installed to follow along.

## Presentation Outline

| # | Topic | Description |
|---|-------|-------------|
| 1 | Common "Friction Points" When Using Agents | Symptoms: plausible-looking code that breaks on deploy, edits to generated files, sessions that need constant steering, token spend out of proportion to the change |
| 2 | Why Does This Friction Happen? | The agent starts each session without a mental model of the repository; unanswered questions become guesses |
| 3 | Techniques to Optimize the Agent Environment | Overview of the seven areas and when each one applies |
| 4 | AGENTS.md | A short hand-written file for commands, conventions, boundaries, and definition of done |
| 5 | Scripts | One command per intent, non-interactive, stable exit codes, bounded output |
| 6 | Skills | A repeatable procedure packaged with staged loading so detail is read only when relevant |
| 7 | Hooks | Enforcement on harness events, independent of what the model decides to read |
| 8 | MCP | Bringing in data that lives outside the checkout, at the highest context cost of the seven |
| 9 | Isolation | Git worktrees per parallel task plus stacked PRs to keep reviews small |
| 10 | Orchestration | Steering several agents at once; worth it only when review, not generation, is the bottleneck |
| 11 | Common Anti-Patterns | Installing everything up front, auto-generating context files, over-long instructions, over-blocking hooks |
| 12 | DevEx: From Developers to Coding Agents | Every improvement that helps humans maps onto the environment an agent needs |
| 13 | Take Aways | What to adopt first, and how to know it helped |

## Render

```sh
npm install
npm run build      # renders both languages into output/
npm run preview    # opens output/index.html in the browser
```

Requires Node.js 20 (pinned via `mise.toml`). Run `mise install` first if you use mise.

| File | Language |
|---|---|
| `output/index.html` | English (default) |
| `output/index.vi.html` | Vietnamese |

Both files include a floating language switcher linking to the other version.

## Remote Control

A mobile remote control is included. Open `output/remote.html` on a phone to drive slide navigation and read speaker notes in real time through the ntfy.sh relay.

## Project Structure

```
slides/
  presentation.en.md  # English slide source
  presentation.vi.md  # Vietnamese slide source
  images/             # screenshots and QR codes referenced in slides
build.cjs             # build script and slide post-processing
public/
  presentation/       # presentation.css and presentation.js
  remote/             # mobile remote control
.github/workflows/
  deploy-pages.yml    # deploy to GitHub Pages on push to main
.agents/skills/       # agent skills for content, styling, and release notes
```

Theming lives in `public/presentation/presentation.css`; per-slide accents and entrance animations are CSS rules there. See [`.agents/skills/markpress-styling/SKILL.md`](.agents/skills/markpress-styling/SKILL.md).
