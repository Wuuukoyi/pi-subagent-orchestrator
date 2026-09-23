# pi-subagent-orchestrator

A [Pi](https://pi.dev) package that turns the main agent into a pure orchestrator: it keeps **only** a delegation tool and dispatches all real work to isolated subagents.

## What it does

- **Hard revocation**: launch with `pi --tools subagent` and the main agent has no `read` / `write` / `edit` / `bash` tools at all — not even their names or parameter schemas enter its context. It can only delegate.
- **Isolated subagents**: every subagent runs in a separate `pi` subprocess, with its own context window, its own system prompt, and its own tool allowlist. The main conversation never leaks in.
- **Autonomous dispatch**: the main agent decides by itself which subagent to call, whether to run tasks in parallel (up to 8, 4 concurrent), or to chain them sequentially with results fed forward.

## Install

```bash
pi install npm:pi-subagent-orchestrator
# or from git:
pi install git:github.com/<Wuuukoyi>/pi-subagent-orchestrator
```

Requires a configured model (e.g. via `pi` `/login`).

## Usage

Start with the delegation tool only:

```bash
pi --tools subagent
```

Then just talk to it:

```
Use scout to get an overview of this repository
Run 2 scouts in parallel: one to list models, one to list providers
First have scout find the config code, then have planner propose a refactor
```

Workflow prompt templates are also included: `/implement`, `/scout-and-plan`, `/implement-and-review`.

## Bundled subagents

Agents are bundled in the package (`agents/` directory) and load automatically — no manual copying needed. User-level agents in `~/.pi/agent/agents` and project-level agents in `.pi/agents` are still discovered, and a same-named agent overrides the bundled one.

| Agent      | Purpose                       | Tool allowlist        |
| ---------- | ----------------------------- | --------------------- |
| `scout`    | Fast recon, compressed report | read, grep, find, ls, bash |
| `planner`  | Implementation plans          | read, grep, find, ls  |
| `reviewer` | Code review                   | read, grep, find, ls, bash |
| `worker`   | General-purpose execution     | all default tools     |

All bundled agents inherit the current session's model. To pin a different model for an agent, add `model: <model-id>` to its frontmatter.

## Add your own subagent

Create a Markdown file in `~/.pi/agent/agents/` (or `.pi/agents/` in a project):

```markdown
---
name: calculator
description: Performs numerical calculations and data analysis
tools: bash
---

You are a calculation specialist. Execute computations via bash and report results precisely.
```

It becomes available on the next delegation — no restart needed.

## Notes

- Each subagent makes its own API calls and bills against your provider account; parallel dispatch multiplies this. Mind your rate limits (e.g. low-tier accounts).
- Project-local agents (`.pi/agents`) are only loaded with `agentScope: "both"` / `"project"` and require project trust.
- On Windows, Pi executes `bash` through Git Bash; install Git for Windows.

## License

MIT
