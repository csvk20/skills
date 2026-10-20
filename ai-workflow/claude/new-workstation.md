# New workstation setup

Two ways to set up a machine:

- **Install** (normal case): copy the files from [`config/`](./config) and verify. Use Prompt A.
- **Rebuild** (Claude Code changed a lot, or you want to re-derive the design): give Claude the full specification in [setup-prompt.md](./setup-prompt.md). Use Prompt B.

## 1. Prerequisites

1. Install Claude Code and sign in. Check with `claude --version`.
2. Install and sign in to GitHub CLI: `gh auth login`. `gh api user --jq .login` must print your GitHub login, because `git-maintainer` assigns PRs to `@me`.
3. Set the git identity that the `git-commit` skill expects:
   ```bash
   git config --global user.name csvk20
   git config --global user.email siva.krishnan@se3.ai
   ```
4. Clone the skill sources and link them into `~/.claude/skills/` (see the source table in [README.md](./README.md#skills-assigned-to-agents)):
   - `git@github.com:csvk20/agent-skills.git` to `~/.agents/skills` (`ponytail`, `ponytail-review`, `git-commit`, `memory-model`, `diagnosing-bugs`, `codebase-design`)
   - `git@github.com:csvk20/skills.git` to `~/skills` (this repository, `se3-pr-description`)
   - `git@github.com:se3-labs/SE3-skills.git` to `~/.local/share/se3ai/SE3-skills` (`se3-package-cpp`, `se3-doxygen`, `tdd`, `arm-cpp-optimizer`)

   Each skill must resolve as `~/.claude/skills/<name>/SKILL.md`, usually through a symlink.
5. If you use the `run-ledger@se3-mods` mod, run `/run-ledger auto off` in a session. Its auto-routing overrides agent models.

## 2. Prompt A: install from this repository

Start `claude` in your home directory and paste:

```text
Install my Claude Code routed-agent workflow from ~/skills/ai-workflow/claude.
Read README.md and new-workstation.md there first.

1. Check the installed Claude Code version and read the current official docs for
   subagents, skills, and model/effort configuration. Report any frontmatter field
   in config/ that this version no longer supports.
2. Inspect everything under ~/.claude/. Preserve existing behavior and instructions.
   Merge, do not duplicate. Ask me only if a merge would remove intentional behavior.
3. Copy config/agents/*.md to ~/.claude/agents/ and
   config/skills/project-workflow/ to ~/.claude/skills/project-workflow/.
4. Add only the "Agent Routing" section of config/CLAUDE.md to ~/.claude/CLAUDE.md.
   Keep everything already there.
5. Check that every skill named in the agents' `skills:` lists and on-demand lists
   resolves under ~/.claude/skills/. Report missing ones; do not invent replacements.
6. Run the override audit from verification.md: subagent/effort env vars, ultracode,
   global effortLevel and modelSettings, managed settings, org limits, and any
   plugin or mod that rewrites Agent calls (for example run-ledger autoRoute).
   Report findings. Do not edit shell startup files or managed config.
7. Run every check in verification.md and report the observed results.
Do not enable ultracode, do not set a global xhigh or max effort, and do not force
one model on all subagents.
```

## 3. Prompt B: rebuild from the specification

Use this when the copies in `config/` no longer match what Claude Code supports. Paste the whole prompt from [setup-prompt.md](./setup-prompt.md). It has the original requirements and the later additions (skills per agent, `git-maintainer`, PR assignee). Afterwards, copy the new live files back into `config/` and update these documents.

## 4. After setup

1. Run the checks in [verification.md](./verification.md). Discovery, model resolution, and skill preloading must all pass.
2. Start a routed task with `/project-workflow <task>`. Typing the slash command, not a trigger phrase, makes the orchestrator run on Sonnet (see Known limits in the README).
3. Ask for commits and PRs explicitly ("commit this and open a PR to main"). The workflow never commits on its own.

## Daily use prompts

| Goal | Prompt |
|---|---|
| Routed task | `/project-workflow <task, constraints, acceptance criteria>` |
| Find code only | `Use the explorer agent to find <question>.` |
| Plan only | `Use the planner agent to plan <change>. Do not implement.` |
| Diagnose only | `Use the diagnoser agent on <failure>. Include the failing command.` |
| Review a branch | `Use the reviewer agent on the diff against main. Requirements: <...>.` |
| Commit and PR | `Commit this and open a PR to <base>.` |
