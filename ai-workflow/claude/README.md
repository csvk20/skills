# Claude Code routed-agent workflow

Global Claude Code configuration for cost-aware, conditional multi-agent software development. It routes each task to the cheapest specialist that can do it well, escalates only on ambiguity or consequence, and never runs a full pipeline by default. It does not use Ultracode.

Set up on 2026-10-20 with Claude Code 2.1.304 (Linux). The files under [`config/`](./config) are copies of the live files in `~/.claude/`.

| Document | Contents |
|---|---|
| [README.md](./README.md) | Architecture, routing, skills, environment findings, limits (this file) |
| [new-workstation.md](./new-workstation.md) | Install on a new machine, and the prompts to give Claude Code |
| [setup-prompt.md](./setup-prompt.md) | Full specification prompt to rebuild the setup from scratch |
| [verification.md](./verification.md) | How to verify the setup, with the results observed on 2026-10-20 |

## Architecture

Three layers, each with one job:

| Layer | Live path | Copy here | Job |
|---|---|---|---|
| Routing policy | `~/.claude/CLAUDE.md` (section "Agent Routing") | [config/CLAUDE.md](./config/CLAUDE.md) | Always loaded. Six lines: use specialists for non-trivial work, route conditionally, agent files are authoritative. |
| Workflow skill | `~/.claude/skills/project-workflow/SKILL.md` | [config/skills/project-workflow/SKILL.md](./config/skills/project-workflow/SKILL.md) | Loaded on demand. Classification, routes, escalation, cost controls, handoffs, completion criteria. |
| Specialist agents | `~/.claude/agents/*.md` | [config/agents/](./config/agents) | Exact role, model, effort, tools, permissions, and skills for each specialist. |

`config/CLAUDE.md` also holds the user's pre-existing "Git Commits" rules. Only the "Agent Routing" section belongs to this setup.

## Routing table

| Agent | Model | Effort | Tools | Trigger |
|---|---|---|---|---|
| `explorer` | haiku | medium | Read, Grep, Glob | Code ownership, callers, or execution path unclear |
| `test-runner` | haiku | low | Read, Grep, Glob, Bash (no edit tools) | Run existing tests, linters, type checks, builds |
| `implementer` | sonnet | medium | Read, Edit, Write, Bash, Grep, Glob, Skill | Clear plan, supported diagnosis, or small task with acceptance criteria |
| `reviewer` | sonnet | high | Read, Grep, Glob, Skill | After deterministic checks, when semantic review adds value |
| `planner` | opus | high | Read, Grep, Glob, Skill | Ambiguous, cross-module, architectural, migration, or irreversible work |
| `diagnoser` | opus | high | Read, Grep, Glob, Bash, Skill (no edit tools) | Unclear cause, a failed evidence-based fix, or contradicting evidence |
| `critical-reviewer` | opus | high | Read, Grep, Glob, Skill | Only: security boundary, concurrency, data loss, destructive migration, public API, planner/reviewer disagreement, unverifiable invariant |
| `git-maintainer` | sonnet | medium | Read, Grep, Glob, Bash, Skill (no edit tools) | Commit, push, or PR, only on explicit user request |
| `project-workflow` (orchestrator skill) | sonnet | medium | session tools | `/project-workflow`, or phrases below |

Models are family aliases (`haiku`, `sonnet`, `opus`), so agents follow the newest version the account allows. On 2026-10-20 they resolved to `claude-haiku-5-5`, `claude-sonnet-5-5`, and `claude-opus-5-5`.

Workflow trigger phrases: "start the project workflow", "use routed agents", "run the structured development workflow", or typing `/project-workflow`.

## Workflow summary

The skill is a state machine. The orchestrator classifies the task (ambiguity, consequence, scope, familiarity, known cause, deterministic checks, parallelism) and takes the first route that fits:

| Step | Situation | Route |
|---|---|---|
| 2 | Small and clear | No subagents. Implement directly, run checks. |
| 3 | Location unclear | `explorer`, then reclassify. |
| 4 | Ambiguous or high impact | `explorer` if needed, `planner`, then `implementer` with the accepted plan. |
| 5 | Bug, cause supported | `implementer` with the diagnosis, checks, `reviewer` only if uncertain. |
| 6 | Bug, cause unclear | `test-runner` to reproduce, `diagnoser`, then `implementer` only on a supported cause. |
| 7 | Verification | Deterministic checks first, then `reviewer` if it adds value. |
| 8 | Critical escalation | `critical-reviewer` only for its explicit triggers. Never for size alone. |
| 9 | Commit and PR | `git-maintainer`, only when the user asks. |

Cost controls: at most 3 concurrent subagents, parallel only for independent read-only work, one automatic escalation (more needs new evidence), no max effort or Ultracode, no per-call model or effort overrides, stop when completion criteria hold.

## Skills assigned to agents

Preloaded skills are injected in full into every spawn of that agent, so only small, role-central skills are preloaded. Domain skills load on demand through the Skill tool when the repository or task matches.

| Agent | Preloaded | On demand |
|---|---|---|
| `implementer` | `ponytail` | `se3-package-cpp`, `se3-doxygen`, `tdd`, `arm-cpp-optimizer` |
| `reviewer` | `ponytail-review` | `se3-package-cpp`, `se3-doxygen`, `arm-cpp-optimizer` |
| `planner` | none | `se3-package-cpp`, `codebase-design`, `arm-cpp-optimizer` |
| `diagnoser` | `diagnosing-bugs`, `ponytail` | `arm-cpp-optimizer`, `se3-package-cpp` |
| `critical-reviewer` | none | `memory-model`, `se3-package-cpp` |
| `git-maintainer` | `se3-pr-description` | `git-commit` (read as a file, see below) |
| `explorer`, `test-runner` | none | none |

Role adaptations:

- `reviewer` writes two separate parts: a correctness review, then `ponytail-review` in its own format ending with `net: -N lines possible.`
- `diagnoser` runs phases 1 to 4 of `diagnosing-bugs` (feedback loop, minimise, ranked hypotheses, non-editing probes) and stops before phase 5 (fix). Harnesses go in a scratch directory. Instrumentation that needs code edits is returned as a requested probe. `ponytail` shapes only the fix direction.
- `implementer` follows the `ponytail` ladder, but the plan and acceptance criteria win on conflict.
- `git-commit` sets `disable-model-invocation: true`, which also blocks preloading into subagents. `git-maintainer` reads `~/.claude/skills/git-commit/SKILL.md` directly and acts only on an explicit request, which keeps the intent of that flag.

Skill sources on the original workstation:

| Skill | Source |
|---|---|
| `ponytail`, `ponytail-review`, `git-commit`, `memory-model` | `~/.agents/skills` (`csvk20/agent-skills`) |
| `diagnosing-bugs`, `codebase-design` | `~/.agents/skills/engineering` (from `mattpocock/skills`) |
| `se3-pr-description` | this repository, `skills/se3/se3-pr-description` |
| `se3-package-cpp`, `se3-doxygen`, `tdd`, `arm-cpp-optimizer` | `~/.local/share/se3ai/SE3-skills` (`se3-labs/SE3-skills`) |

A missing preloaded skill is skipped with a debug-log warning, so install the sources before relying on the agents.

## git-maintainer and pull requests

- Commits follow the `git-commit` skill: author `csvk20 <siva.krishnan@se3.ai>`, one-line caveman message, hooks run, no co-authors, no force-push.
- PRs never come from the default branch. In a fork, the agent pushes to and opens the PR on the user-owned repository and passes `--repo` to every `gh pr` command.
- Every PR is created with `gh pr create --assignee @me`, then checked with `gh pr view --json assignees`. If the user is missing, it runs `gh pr edit --add-assignee @me`. `@me` is the `gh` login (`csvk20`).
- PR descriptions use `se3-pr-description` in SE3 repositories and the same Change / Evidence / Limits shape elsewhere, with only verified facts.

## Read-only enforcement

The workstation runs with `permissions.defaultMode: auto`. In that mode a subagent's `permissionMode` is ignored, so read-only behavior comes from the `tools` list:

- On Linux and macOS, Grep and Glob are absent by default (search runs through Bash). A subagent that lists Grep or Glob and leaves out Bash gets them back. `explorer`, `planner`, `reviewer`, and `critical-reviewer` use this and have no write path at all.
- `reviewer` and `critical-reviewer` have no shell, so the orchestrator passes the diff inline or as a scratch file.
- `diagnoser`, `test-runner`, and `git-maintainer` need Bash. Edit, Write, and NotebookEdit are disallowed, but a Bash command can still write. Their limits are prompt rules, not enforcement.

## Environment findings (original workstation)

| Control | Finding | Effect |
|---|---|---|
| `CLAUDE_CODE_SUBAGENT_MODEL`, `CLAUDE_CODE_SUBAGENT_MODEL_FORCE`, `CLAUDE_CODE_EFFORT_LEVEL` | Not set | None |
| `"ultracode"` | Not set | None |
| Managed settings, org model or effort limits | None (`/etc/claude-code` absent, `policy-limits.json` has no model limits) | None |
| `"model": "opus"` in `settings.json` | User default | Main session runs Opus. The skill switches to Sonnet only for an explicit `/project-workflow` turn. |
| `modelSettings.claude-opus-5-5.effortLevel: medium` | Saved by `/effort` | Main Opus session at medium. Frontmatter effort overrides it. |
| Top-level `"effortLevel": "xhigh"` | Legacy, kept by user decision | Ignored for Opus 5.5. Default for Sonnet and Haiku sessions and for built-in agents without an effort. Custom agents and the skill override it. |
| `run-ledger@se3-mods` mod, `autoRoute` | Default on. Turned off with `/run-ledger auto off` | When on, it injects a per-call model from Jev's tier pick into every Agent call without a model. A per-call model outranks frontmatter, so it defeats routing. With it on, `planner`, `diagnoser`, and `implementer` ran on Haiku and `reviewer` on Opus. |

Resolution order (from the docs): subagent model is per-call parameter, then frontmatter, then `CLAUDE_CODE_SUBAGENT_MODEL`, then the main model. Subagent effort is `CLAUDE_CODE_EFFORT_LEVEL`, then per-call parameter, then frontmatter, then session level.

## Known limits

- **Implicit invocation keeps the session model.** When Claude invokes `project-workflow` from a trigger phrase, the turn stays on the session model (Opus here). Typing `/project-workflow` switches the turn to Sonnet. Reproduced twice on 2.1.304. The docs say the skill model applies in both cases. `context: fork` would avoid it but breaks multi-agent sequencing, so it is not used.
- **Skill model lasts one turn.** The session model resumes on the next prompt, as documented.
- **Bash agents are read-only by instruction only** (see above). A PreToolUse hook could enforce it, but no hook was added.
- **`run-ledger` must stay at `auto off`.** Re-enabling it silently overrides agent models.

## Maintenance

- Edit the live files in `~/.claude/`, then copy them back into `config/` and commit. The copies are documentation, not symlinks.
- After any change, run the checks in [verification.md](./verification.md).
- Keep agent descriptions short and specific. Claude routes by description, and all descriptions share one listing budget.
- No em-dashes in this folder (repository rule).
