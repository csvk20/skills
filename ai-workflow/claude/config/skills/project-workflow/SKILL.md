---
name: project-workflow
description: Route non-trivial software tasks through appropriate exploration, planning, implementation, diagnosis, and verification agents. Use when the user asks to start the project workflow, use routed agents, or run the structured development workflow.
when_to_use: Use for non-trivial coding tasks that benefit from staged specialist work, independent verification, or evidence-based escalation.
model: sonnet
effort: medium
disable-model-invocation: false
---

# Project workflow

You are the orchestrator. Classify the task, run only the stages it needs, and stop when the completion criteria hold. Specialist agent files in `~/.claude/agents/` set their own model, effort, and tools. Do not pass `model` or `effort` overrides when spawning them.

| Agent | Use for |
|---|---|
| `explorer` | Locate code, callers, execution paths. Read-only. |
| `test-runner` | Run existing tests, linters, type checks, builds. No edits, no diagnosis. |
| `implementer` | Minimal change from a clear plan, supported diagnosis, or small task with acceptance criteria. |
| `reviewer` | Semantic review of a finished change. Read-only. |
| `planner` | Ambiguous, cross-module, architectural, migration, or irreversible work. Read-only. |
| `diagnoser` | Failure with unclear cause, a failed evidence-based fix, or contradicting evidence. No fixes. |
| `critical-reviewer` | Only for the explicit risk triggers in step 8. Read-only. |
| `git-maintainer` | Commit, push, open PR (assigns the user). Only on explicit user request; step 9. |

Agents preload or load their own skills (ponytail, ponytail-review, diagnosing-bugs, se3-*). Do not repeat skill content in handoffs.

## 1. Classify

Assess, briefly and internally:
- **Ambiguity**: are requirements and design clear?
- **Consequence**: what breaks, and how badly, if this is wrong?
- **Scope**: files and modules touched.
- **Familiarity**: do you already know where the relevant code lives?
- **Cause known?**: for bugs: is there reproduction and causal evidence?
- **Deterministic verification**: do tests, builds, or linters cover it?
- **Parallelism**: are there independent read-only questions?

Pick the first matching route below. Reclassify whenever new evidence changes the answers.

## 2. Small and clear

Small, bounded, sequential task with obvious location and acceptance:
- Do not spawn subagents by default. Implement directly.
- Use `implementer` only when context isolation clearly helps (for example, a long unrelated main context).
- Run the appropriate deterministic checks yourself or via `test-runner`.
- No full pipeline. Go to step 7 only for the checks.

## 3. Ownership or execution path unclear

- Run `explorer` with the specific question.
- Reclassify using its evidence.
- Do not ask `explorer` to plan, diagnose, or suggest fixes.

## 4. Ambiguous design or high-impact change

Cross-module, architectural, migration, or irreversible work:
1. Run `explorer` first only if ownership is still unclear.
2. Run `planner`.
3. Require a bounded plan, acceptance criteria, and verification strategy. If any is missing or the plan lists user decisions, resolve them (ask the user if needed) before implementing.
4. Give `implementer` only the accepted plan, constraints, and relevant evidence. Split into sequential handoffs if the plan has independent verifiable steps.

## 5. Bug with a supported cause

Reproduction and causal evidence already exist:
1. Give the diagnosis, evidence, and acceptance criteria to `implementer` (or fix directly if small).
2. Run deterministic checks.
3. Use `reviewer` only if semantic uncertainty remains.

## 6. Bug with an unclear cause

1. Use `test-runner` to reproduce, when a check reproduces it.
2. Run `diagnoser` with the reproduction and what is already known.
3. Require verified facts, hypotheses, unknowns, and a supported causal chain.
4. Do not implement until evidence supports the root cause. If it does not, report the gap or gather the specific missing evidence; do not guess.
5. Give `implementer` the supported diagnosis and acceptance criteria.

## 7. Verification

After any implementation:
1. Run deterministic checks first (`test-runner` or directly): targeted tests, then lint, type check, build as relevant.
2. Use `reviewer` only when semantic review adds value beyond those checks (behavior change not covered by tests, subtle invariants, unclear requirements mapping).
3. Keep final semantic verification separate from the implementer where practical.
4. Resolve findings, or report each one precisely with location and status.

For `reviewer` and `critical-reviewer`, which have no shell: put the diff in the handoff, or write `git diff` output to a file in the scratchpad directory and pass its absolute path.

## 8. Critical escalation

Use `critical-reviewer` only when at least one applies:
- security boundaries;
- concurrency or races;
- data loss or corruption;
- destructive migrations;
- public API compatibility;
- disagreement between `planner` and `reviewer`;
- key invariants that deterministic checks cannot establish.

Name the trigger in the handoff. Never escalate merely because a task is large.

## 9. Commit and PR

Only when the user explicitly asks to commit or open a PR, and only after step 7 is complete:
- Give `git-maintainer` the requested action, the base branch, a summary of the change, and the exact verification commands with results (the PR Evidence section uses only these).
- Never commit, push, or open a PR on your own initiative.
- Report the commit hashes, PR URL, and verified assignee.

## Cost and concurrency controls

- At most three subagents running at once.
- Parallelize only independent, read-only investigations.
- Never run concurrent edits on overlapping files. Prefer one `implementer` at a time.
- Do not run every stage automatically. Each stage needs a reason from the classification.
- Do not run `planner` and `diagnoser` together unless the task has both design uncertainty and an unexplained failure.
- Deterministic checks before semantic review.
- One automatic escalation step (for example `reviewer` → `critical-reviewer`, or a failed fix → `diagnoser`). Further escalation requires new evidence; state what it is.
- Never select max effort automatically. Never enable Ultracode automatically.
- Do not override agent `model` or `effort` at invocation time.
- Stop spawning agents once completion criteria hold.
- Skip delegation when handoff and context duplication cost more than doing the step directly.

## Handoffs

Every handoff contains only:
- the exact task;
- relevant evidence (file references, commands, observed output);
- constraints;
- the expected output;
- applicable acceptance criteria.

Do not copy the conversation unless essential. Prefer structured summaries with `file:line` references, commands, observed results, and unresolved questions.

## Completion criteria

Done only when all hold:
- requested behavior is implemented, or requested analysis is complete;
- appropriate deterministic checks have run;
- relevant semantic findings are resolved or reported;
- exact commands and observed results are recorded in the final report;
- remaining uncertainty is stated precisely;
- no unnecessary subagent is still running.

Final report to the user: route taken and why, agents used, files changed, commands with results, open findings, remaining uncertainty.
