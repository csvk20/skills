---
name: project-workflow
description: Use when I explicitly request $project-workflow, “start the project workflow,” “use routed agents,” or “run structured development workflow” for software changes. Routes development work conditionally by ambiguity, risk, ownership, cause, verification, and parallelism.
---

# Project workflow

Use this workflow for explicit routed/structured software-development requests. Do not force it onto unrelated tasks. Follow the specialist definitions in `~/.codex/agents/*.toml` for role boundaries and model settings. Keep the parent at medium effort and never automatically select `max` or `ultra`.

## Classify first

Assess ambiguity, consequence, scope, ownership clarity, known versus unknown cause, deterministic verifiability, and useful independent parallelism. Choose only the stages needed. Keep handoffs concise: task, constraints, relevant evidence, expected output. Allow one automatic escalation step; further escalation requires new evidence. Stop spawning agents once completion criteria are met.

## Routing

1. **Small and clear:** avoid subagents by default. Implement directly, or use `implementer` only when isolation helps. Run relevant deterministic checks.
2. **Ownership or path unclear:** use `explorer`, then reclassify from its file and execution-path evidence.
3. **Ambiguous design, cross-module work, migration, or irreversible change:** use read-only `planner`; use `explorer` first only if ownership/path is unclear. Require a bounded plan, acceptance criteria, and verification strategy. Give only relevant evidence to `implementer`.
4. **Bug with supported cause:** give the evidence and bounded scope to `implementer` for a minimal fix, then run deterministic checks.
5. **Bug with unclear cause:** use `test-runner` to reproduce, then `diagnoser` to establish an evidence-backed causal chain. Do not implement until diagnosis is supported. Do not run `planner` and `diagnoser` together unless both design and cause uncertainty independently exist.
6. **After implementation:** run deterministic checks first. Use `reviewer` only when semantic uncertainty remains beyond those checks.
7. **Critical review:** use `critical-reviewer` only for security, concurrency/races, data integrity/loss, destructive migration, public API compatibility, planner/reviewer disagreement, a failed evidence-based fix, or a key invariant not established by deterministic checks. Use Astra at xhigh when available. If Astra is unavailable, use Sol at max only for this critical reviewer and report the fallback explicitly.

## Cost and coordination

- Never invoke `max` or `ultra` automatically except the stated critical-reviewer fallback to Sol at `max` when Astra is unavailable.
- Use no more than three concurrent subagents. Parallelize only independent read-only investigations.
- Never allow concurrent agents to edit overlapping files. Prefer one implementation owner.
- Do not run every stage unconditionally; avoid planner and diagnoser together unless both kinds of uncertainty are present.
- Prefer deterministic checks for mechanical verification. Delegate only when the expected quality, focus, or time gain exceeds context and coordination cost.
