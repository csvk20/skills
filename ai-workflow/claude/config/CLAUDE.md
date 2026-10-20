# Global Rules

## Git Commits
- Never add Claude as co-author in any commit.
- Keep commit messages under 2 sentences unless absolutely necessary.

## Agent Routing
- For non-trivial work, use the specialist agents in `~/.claude/agents/`; route conditionally, not through a mandatory full pipeline.
- Invoke the `project-workflow` skill when I ask for routed agents or the structured development workflow.
- Use deterministic tools (tests, linters, type checks, builds) for mechanical verification.
- Escalate by ambiguity and consequence of failure, not by size.
- Skip delegation when handoff overhead exceeds its value.
- Agent definitions are authoritative for model, effort, permissions, and tools; don't override them at invocation.
