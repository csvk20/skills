# Codex workflow on a new workstation

This guide installs the snapshots in `config/` into the global Codex locations. It preserves other Codex settings and adapts absolute skill paths for the new home directory.

## Prerequisites

1. Install Codex and sign in. Confirm `codex --version`.
2. Clone this repository to `~/skills` or update the path in the setup prompt.
3. Install the skill source repositories:
   - `git@github.com:csvk20/agent-skills.git` for `ponytail`, `ponytail-review`, and `git-commit`.
   - `git@github.com:se3-labs/SE3-skills.git` for `se3-package-cpp` and `se3-doxygen`.
   - `se3-pr-description` is in this repository under `skills/se3/se3-pr-description`.
4. Authenticate GitHub CLI with `gh auth login` if you want `git-maintainer` to create PRs. Confirm `gh api user --jq .login` prints the intended account. The role assigns PRs with `@me`.

## Install prompt

Start Codex in your home directory and paste:

```text
Install my global Codex routed-agent workflow from ~/skills/ai-workflow/codex.
Read README.md and new-workstation.md first.

1. Check the installed Codex version and current official docs for global
   AGENTS.md discovery, skill discovery and front matter, custom agent TOML,
   model and reasoning effort fields, and agent defaults in config.toml.
2. Inspect ~/.codex/config.toml, ~/.codex/AGENTS.override.md,
   ~/.codex/AGENTS.md, ~/.codex/agents/, ~/.agents/skills/, and
   ~/.codex/skills/. Preserve existing user settings and instructions. Merge
   definitions rather than duplicate them. Do not modify project repositories.
3. Install any missing skill sources listed in README.md. Do not proceed with
   enabled references to missing skills; report any source that cannot be found.
4. Install config/AGENTS.md as ~/.codex/AGENTS.md only if no active
   ~/.codex/AGENTS.override.md supersedes it. Keep existing user instructions.
   Copy config/agents/*.toml to ~/.codex/agents/ and
   config/skills/project-workflow/SKILL.md to
   ~/.agents/skills/project-workflow/SKILL.md. Adapt absolute skill paths in
   agent TOML files to this workstation.
5. Merge config/agents-defaults.toml into ~/.codex/config.toml without
   replacing unrelated settings. Keep the primary model and reasoning effort
   at the user's existing values; parent reasoning must stay medium. Keep no
   more than three concurrent subagents. Do not set a global xhigh, max, or
   ultra effort, and do not force all subagents to one model or effort.
6. Audit root model and effort, [models.new_thread], [agents], explicit
   per-agent settings, model availability, and any managed limits. Do not edit
   shell startup files or managed policy.
7. Validate all TOML and skill front matter. Use Codex prompt discovery to
   verify the global AGENTS instructions and project-workflow skill appear.
   Confirm each role file and attached skill path exists, model and effort
   fields are valid, and the medium parent defaults do not erase specialist
   routing. Report anything that cannot be verified without launching a model.
8. Report changed files, preserved settings, routing table, prompt invocation,
   commands and results. Do not claim a runtime role selection was tested unless
   it was exercised in a session.
```

The prompt lets the installed Codex version confirm current schemas before it writes configuration. Review the resulting diff before using the workflow.

## How to invoke it

Use the skill explicitly:

```text
$project-workflow: Fix <bug>. Constraints: <constraints>. Acceptance criteria: <criteria>.
```

Or request it in natural language:

```text
Start the project workflow for <task>. Use these constraints: <constraints>. Done means <acceptance criteria>.
```

The skill is not a mandatory pipeline. Small, clear tasks normally stay with the main agent. For tasks needing routing, the skill selects only the stages justified by ambiguity, consequence, ownership, cause, deterministic verification, and independent parallelism.

Specialist examples:

| Need | Prompt |
|---|---|
| Locate behavior | `Use explorer to map the callers and execution path for <behavior>.` |
| Plan a risky change | `Use planner to make a bounded plan for <change>; do not implement.` |
| Diagnose a failure | `Use test-runner to reproduce <failure>, then diagnoser to establish the cause.` |
| Review implementation | `Use reviewer to check the diff for semantic risks after the checks.` |
| Commit and open PR | `Use git-maintainer to commit this and open a PR to <base>. Assign me as the PR assignee.` |
