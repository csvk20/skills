# Full setup prompt

Paste everything inside the block into Claude Code, started in your home directory. It is the original specification (2026-10-20) merged with the later requests: skills per agent, the `git-maintainer` role, PR assignment, and fork handling. It produces the files in [`config/`](./config).

````text
Configure my global Claude Code environment for cost-aware, conditional multi-agent
software-development workflows. Work directly on the configuration. Complete the setup
and verify it rather than merely proposing files.

SCOPE
Modify only my global Claude Code configuration:
- ~/.claude/CLAUDE.md
- ~/.claude/skills/project-workflow/
- ~/.claude/agents/
- ~/.claude/settings.json only when necessary
Do not modify project repositories.

TARGET ARCHITECTURE
1. ~/.claude/CLAUDE.md: short, always-loaded routing policy.
2. ~/.claude/skills/project-workflow/SKILL.md: conditional workflow, orchestration,
   escalation, and completion criteria.
3. ~/.claude/agents/*.md: exact specialist roles, model families, effort levels,
   permissions, tools, and skills.

BEFORE EDITING
1. Check the installed Claude Code version.
2. Consult current official Claude Code docs for: custom subagents (including the
   `skills` field), Skills and their frontmatter, model and effort configuration,
   CLAUDE.md hierarchy, configuration precedence, and the tools reference (Grep/Glob
   availability on this OS).
3. Inspect all existing files under ~/.claude/.
4. Preserve existing behavior and user instructions. Merge with existing agents or
   skills rather than creating duplicates. Make the smallest coherent change.
5. Ask only if resolving a conflict would remove or materially change intentional
   existing behavior. Otherwise proceed and report the resolution.

OVERRIDE AUDIT
Check settings and the environment for controls that could defeat per-agent routing:
CLAUDE_CODE_SUBAGENT_MODEL, CLAUDE_CODE_SUBAGENT_MODEL_FORCE, CLAUDE_CODE_EFFORT_LEVEL,
global modelSettings, global effortLevel, "ultracode": true, organization model or
effort restrictions, and any enabled plugin or mod hook that rewrites Agent calls
(for example run-ledger autoRoute). Report findings. Do not edit shell startup files
or organization-managed configuration.

DO NOT ENABLE ULTRACODE
- Do not enable "ultracode": true. Do not globally select xhigh or max.
- Do not force all subagents onto one model or effort. Never invoke max automatically.
- Keep orchestration at Sonnet Medium. Specialist agent files control their own model
  and effort.

MODEL POLICY
Use family aliases so agents follow the newest permitted version:
- haiku: inexpensive exploration and mechanical work.
- sonnet: implementation and normal semantic reasoning.
- opus: ambiguous or expensive-to-get-wrong reasoning.
If an alias is unavailable or blocked, report the resolved model or fallback. Do not
silently substitute a weaker role configuration.

READ-ONLY ENFORCEMENT
Enforce read-only roles through the `tools` list, because permissionMode is ignored
when the main session runs in auto, acceptEdits, or bypassPermissions mode. Where the
OS omits Grep/Glob by default, a subagent that lists Grep/Glob without Bash gets them
back; use that for agents that need no shell.

AGENTS (concise descriptions with precise triggers; model and effort are authoritative)

explorer (~/.claude/agents/explorer.md): haiku, medium, read-only search tools, no
editing. Use when code ownership, relevant files, callers, or execution paths are
unclear. Return relevant files and symbols, callers and execution path, file:line
evidence, constraints, unresolved questions. Do not diagnose, design, or propose fixes.

test-runner: haiku, low, Read/Grep/Glob/Bash, no file or test editing. Run existing
deterministic checks (targeted tests, linters, type checks, builds). Report exact
commands, exit status, relevant output. Do not diagnose, weaken checks, modify tests,
or fix.

implementer: sonnet, medium, may edit. Only for a clear bounded plan, a supported
diagnosis, or a small task with explicit acceptance criteria. Minimum necessary change,
no unrelated cleanup or speculative refactoring, verify affected behavior. Never commits.
Skills: preload ponytail (plan and acceptance criteria win on conflict). On demand via
the Skill tool: se3-package-cpp (se3_package/se3_common/se3_ros C++), se3-doxygen, tdd
(test-first requests), arm-cpp-optimizer (ARM performance).

reviewer: sonnet, high, read-only. After implementation when semantic review adds value
beyond deterministic checks. Review requirements and diff for correctness, preserved
invariants, regressions, edge cases, missing meaningful tests. Evidence-backed findings
with file locations only. Skills: preload ponytail-review and output it as a separate
second part in its own format. On demand: se3-package-cpp, se3-doxygen, arm-cpp-optimizer.

planner: opus, high, plan mode or equivalent read-only. For ambiguous, cross-module,
architectural, migration, or irreversible work. Inspect real code first. Return verified
current behavior, assumptions and unknowns, bounded implementation sequence, affected
files and interfaces, acceptance criteria, verification strategy, material risks.
On demand: se3-package-cpp, codebase-design, arm-cpp-optimizer.

diagnoser: opus, high, read-only (Bash for reproduction only). When the cause is
unclear, one evidence-based fix failed, or evidence contradicts the hypothesis.
Reproduce where practical; separate verified facts, hypotheses, unknowns, causal chain.
No fix. Skills: preload diagnosing-bugs (phases 1 to 4 only, harnesses in a scratch
directory, no edits to tracked files, return needed instrumentation as requested
probes, stop before the fix phase) and ponytail (only to shape the fix direction:
smallest root-cause fix at the shared function). On demand: arm-cpp-optimizer,
se3-package-cpp.

critical-reviewer: opus, high, read-only. Only for security boundaries, concurrency or
races, data loss or corruption, destructive migrations, public API compatibility,
planner/reviewer disagreement, or key invariants that deterministic checks cannot
establish. Report concrete failure modes and required corrections. On demand:
memory-model (races), se3-package-cpp.

git-maintainer: sonnet, medium, Read/Grep/Glob/Bash/Skill, no edit tools. Commits,
pushes, and opens PRs only on explicit user request. The git-commit skill sets
disable-model-invocation, which blocks preloading, so the agent reads
~/.claude/skills/git-commit/SKILL.md and follows it exactly; where it says to ask the
user, stop and return the question. Preload se3-pr-description for PR bodies. Never PR
from the default branch, never force-push, never merge or close unless asked. In a fork,
push to and open the PR on the user-owned repository and pass --repo to every gh pr
command. Create every PR with `gh pr create --assignee @me`, verify with
`gh pr view --json assignees`, and fix with `gh pr edit --add-assignee @me` if needed.

WORKFLOW SKILL (~/.claude/skills/project-workflow/SKILL.md)
Frontmatter:
  name: project-workflow
  description: Route non-trivial software tasks through appropriate exploration,
    planning, implementation, diagnosis, and verification agents. Use when the user asks
    to start the project workflow, use routed agents, or run the structured development
    workflow.
  when_to_use: Use for non-trivial coding tasks that benefit from staged specialist
    work, independent verification, or evidence-based escalation.
  model: sonnet
  effort: medium
  disable-model-invocation: false
Do not set context: fork; the workflow must stay in the parent context to sequence
specialists. Support /project-workflow and the phrases "start the project workflow",
"use routed agents", "run the structured development workflow".

State machine:
1. Classify by ambiguity, consequence, scope, familiarity, known cause, deterministic
   verification, parallelism.
2. Small and clear: no subagents by default; implementer only when isolation helps;
   run checks; no full pipeline.
3. Ownership unclear: explorer, then reclassify. Never ask explorer to plan or diagnose.
4. Ambiguous or high impact: explorer if still unclear, planner, require bounded plan,
   acceptance criteria, verification strategy; give implementer only the accepted plan,
   constraints, evidence.
5. Bug with supported cause: implementer with the diagnosis, deterministic checks,
   reviewer only if semantic uncertainty remains.
6. Bug with unclear cause: test-runner to reproduce, diagnoser, require facts,
   hypotheses, unknowns, causal chain; no implementation until the cause is supported.
7. Verification: deterministic checks first, reviewer only when it adds value, keep
   semantic verification separate from implementation, resolve or report findings.
   Pass diffs to shell-less reviewers inline or as a scratch file.
8. Critical escalation: critical-reviewer only for its explicit triggers, never for size.
9. Commit and PR: git-maintainer only on explicit user request, after step 7.

Cost and concurrency: max 3 concurrent subagents; parallelize only independent
read-only work; no concurrent edits to overlapping files; do not run every stage;
planner and diagnoser together only for design uncertainty plus an unexplained
failure; deterministic before semantic; one automatic escalation, more needs new
evidence; never max effort or Ultracode automatically; no per-call model or effort
overrides; stop when completion criteria hold; skip delegation when overhead exceeds
value.

Handoffs: exact task, relevant evidence, constraints, expected output, acceptance
criteria. No full conversation copies. Structured summaries with file references,
commands, results, open questions.

Completion: behavior implemented or analysis complete; deterministic checks run;
semantic findings resolved or reported; exact commands and results recorded;
uncertainty stated precisely; no unnecessary subagent active.

GLOBAL CLAUDE.md
Add only a short routing section: use the specialist agents for non-trivial work;
invoke project-workflow for routed or structured development; conditional routing,
not a mandatory pipeline; deterministic tools for mechanical verification; escalate by
ambiguity and consequence; avoid delegation when overhead exceeds value; agent
definitions are authoritative for model, effort, permissions, tools. Do not duplicate
the workflow or model matrix.

AVOID UNNECESSARY CONFIGURATION
No hooks unless a requirement needs deterministic lifecycle enforcement. No new
dependencies. No scripts that wrap built-in behavior. No unrelated settings changes.
No permissions broader than those already present. Preload only small role-central
skills; load domain skills on demand.

VERIFICATION
1. Validate every YAML frontmatter block and confirm every field is supported.
2. Confirm the skill and all agents are discoverable (headless init message).
3. Confirm the orchestrator resolves to Sonnet Medium.
4. Confirm each specialist resolves to its model family and effort (subagent
   transcripts), and that preloaded skills are injected.
5. Confirm no setting, environment variable, or plugin hook overrides routing.
6. Dry-run routing tests without spawning agents or editing files: a small task does
   not launch the workflow; an ambiguous migration selects planner; an unclear
   intermittent failure selects diagnoser; a commit/PR request selects git-maintainer.
7. Do not run expensive real tasks to test routing.

FINAL REPORT
Files created and modified, preserved content, conflicts and resolutions, file tree,
routing table (agent, model, effort, permissions, trigger, skills), trigger phrases,
verification with observed results, overriding environment or settings, anything not
verified. Do not claim completion unless the configuration parses and discovery is
verified.
````
