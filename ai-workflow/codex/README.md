# Codex routed-agent workflow

Global Codex setup for cost-aware, conditional software-development workflows. The workflow skill is an explicit entry point; the global policy does not route every coding task through a fixed agent pipeline.

## Start the workflow

Use `$project-workflow: <task, constraints, acceptance criteria>` or say “start the project workflow”, “use routed agents”, or “run structured development workflow”. Small, clear tasks stay direct by default. Codex can discover the skill for these requests, but it does not automatically run the workflow for every software task.

## Architecture

| Layer | Live location | Snapshot here | Purpose |
|---|---|---|---|
| Global guidance | `~/.codex/AGENTS.md` | [config/AGENTS.md](./config/AGENTS.md) | Short pointer to conditional workflow and agent source of truth |
| Parent defaults | `[agents]` in `~/.codex/config.toml` | [config/agents-defaults.toml](./config/agents-defaults.toml) | Enables agents, sets medium parent defaults, caps concurrency at three |
| Specialist agents | `~/.codex/agents/*.toml` | [config/agents/](./config/agents) | Role instructions and explicit model/effort assignments |
| Workflow skill | `~/.agents/skills/project-workflow/SKILL.md` | [config/skills/project-workflow/SKILL.md](./config/skills/project-workflow/SKILL.md) | Task classification, conditional routes, escalation, and cost controls |

`~/.codex/config.toml` contains other workstation-specific settings such as projects and MCP servers. The snapshot contains only the workflow `[agents]` table. Merge it into an existing config; do not replace the whole file.

## Models and routing

| Agent | Model | Effort | Trigger |
|---|---|---|---|
| `explorer` | `gpt-6-luna` | high | Ownership, files, symbols, callers, or execution path unclear |
| `test-runner` | `gpt-6-luna` | low | Existing deterministic check needs to run or reproduce a failure |
| `implementer` | `gpt-6.1-sol` | medium | Bounded plan or supported diagnosis |
| `planner` | `gpt-6.1-sol` | high | Ambiguous design, cross-module change, migration, or irreversible work |
| `diagnoser` | `gpt-6.1-sol` | xhigh | Root cause unclear or evidence-based fix failed |
| `reviewer` | `gpt-6.1-sol` | high | Semantic uncertainty remains after deterministic checks |
| `critical-reviewer` | `gpt-6-astra` | xhigh | Explicit high-risk trigger or unresolved key invariant |
| `git-maintainer` | `gpt-6-luna` | low | Explicitly requested commit or PR operation |

Specialist role files set their own models and efforts, overriding the medium subagent defaults. No role is forced onto the same model or effort. Critical review falls back to `gpt-6.1-sol` at `max` only if Astra is unavailable; the workflow never selects `max` or `ultra` automatically otherwise.

## Skills assigned to roles

| Agent | Attached skills |
|---|---|
| `implementer` | `se3-package-cpp`, `se3-doxygen` |
| `reviewer` | `ponytail-review`, `se3-package-cpp`, `se3-doxygen` |
| `planner` | `se3-package-cpp` |
| `diagnoser` | `ponytail` |
| `git-maintainer` | `git-commit`, `se3-pr-description` |
| `explorer`, `test-runner`, `critical-reviewer` | None attached |

The installed skills live in `~/.agents/skills` or `~/.codex/skills`, often as symlinks to their source repositories. Agent TOML files point to skill manifests using absolute paths. A new workstation must install these sources and adapt each path to that workstation's home directory.

## Git maintainer behavior

The role acts only when the user requests a commit or PR operation. It follows `git-commit` for commits, does not add co-authors, and uses `se3-pr-description` for SE3 pull requests. Every created PR is assigned to the authenticated user with `gh pr create --assignee @me`, then verified through `gh pr view --json assignees`. It does not force-push or rewrite history.

## New workstation

Follow [new-workstation.md](./new-workstation.md) to install from these snapshots, verify discovery, and use the setup prompt.
