# Verification

Run these after installing or changing the setup. Run them from a scratch directory that is not a repository. The headless runs cost about $0.10 to $0.50 each.

## 1. Frontmatter parses and uses supported fields

```bash
cd ~/.claude && python3 - <<'EOF'
import yaml, glob, re
for f in sorted(glob.glob('agents/*.md')) + ['skills/project-workflow/SKILL.md']:
    d = yaml.safe_load(re.match(r'^---\n(.*?)\n---\n', open(f).read(), re.S).group(1))
    print(f, d.get('model'), d.get('effort'), d.get('tools', '(all)'), d.get('skills', []))
EOF
```

Compare the field names with the current docs (subagents and skills pages). Claude Code ignores unknown fields without an error, so a typo fails silently.

## 2. Discovery

```bash
claude -p "Reply OK." --model haiku --max-turns 1 --output-format stream-json --verbose \
  | head -1 | python3 -c "import json,sys; d=json.loads(sys.stdin.readline()); \
print(d['agents']); print('project-workflow' in d['skills'])"
```

Expected: the 8 custom agents in the list, and `True`.

## 3. Model, effort, and preloaded skills per agent

```bash
claude -p "Spawn these subagents in parallel, no model or effort override, each with the prompt \
'Use no tools. Reply with the names of skills preloaded in your context, or none.': explorer, \
test-runner, implementer, reviewer, planner, diagnoser, critical-reviewer, git-maintainer. \
Print each reply." --model sonnet --output-format json < /dev/null > res.json
```

Then read the subagent transcripts. Each one has a `.meta.json` (agent type and requested model) and a `.jsonl` (actual `message.model` and `effort`):

```bash
sid=$(python3 -c "import json;print(json.load(open('res.json'))['session_id'])")
D=$(ls -d ~/.claude/projects/*/"$sid"/subagents)
for m in "$D"/*.meta.json; do python3 - "$m" "${m%.meta.json}.jsonl" <<'EOF'
import json, sys
meta = json.load(open(sys.argv[1])); models, eff = set(), set()
for l in open(sys.argv[2]):
    d = json.loads(l); msg = d.get('message', {})
    if 'effort' in d: eff.add(d['effort'])
    if isinstance(msg, dict) and msg.get('model'): models.add(msg['model'])
print(meta['agentType'], 'requested', meta.get('model'), 'actual', sorted(models), 'effort', sorted(eff))
EOF
done
```

`requested` must be `None` for every agent. A value means something injected a per-call model; check `/run-ledger` auto-routing first.

## 4. Orchestrator model

Run `claude -p "/project-workflow <small task> DRY RUN: do not spawn agents, edit files, or run commands." --output-format stream-json --verbose` and check that the assistant messages report `claude-sonnet-*` with effort `medium` in the session transcript.

## 5. Routing dry runs

Append this suffix to each prompt: `DRY RUN for routing verification: do not spawn agents, edit files, or run commands. Reply with the workflow step and the agents you would invoke.`

| Prompt | Expected |
|---|---|
| `/project-workflow` rename a local variable in one named function, tests exist | Step 2, no agents |
| `/project-workflow` migrate SQLite to Postgres across 3 modules, rollback unsettled | Step 4, planner |
| `/project-workflow` intermittent CI timeout, a retry fix already failed, logs contradict | Step 6, diagnoser |
| `/project-workflow` change done, tests passed, commit and open a PR to main | Step 9, git-maintainer |

## 6. Override audit

```bash
env | grep -E '^CLAUDE_CODE_(SUBAGENT_MODEL|SUBAGENT_MODEL_FORCE|EFFORT_LEVEL)='
grep -nE '"(ultracode|effortLevel|modelSettings|model|availableModels|maxEffortLevel)"' \
  ~/.claude/settings.json ~/.claude/settings.local.json
ls /etc/claude-code/ 2>/dev/null
grep -rn "agent.spawn" ~/.claude/plugins/cache 2>/dev/null | grep -v node_modules
```

Any enabled plugin hook on `agent.spawn` that sets `model` overrides frontmatter.

## Results on 2026-10-20 (Claude Code 2.1.304)

| Check | Result |
|---|---|
| Frontmatter | All 8 agents and the skill parse. All fields are documented. |
| Discovery | 8 agents and `project-workflow` listed. |
| Models with run-ledger `autoRoute` on | Wrong: planner, diagnoser, implementer ran on `claude-haiku-5-5`, reviewer on `claude-opus-5-5`. |
| Models after `/run-ledger auto off` | Correct: explorer, test-runner haiku-5-5; implementer, reviewer, git-maintainer sonnet-5-5; planner, diagnoser, critical-reviewer opus-5-5. |
| Effort | Correct for all agents (low, medium, high as configured). |
| Preloaded skills | implementer: ponytail. reviewer: ponytail-review. diagnoser: diagnosing-bugs, ponytail. git-maintainer: se3-pr-description. planner: none. |
| On-demand skill | planner loaded `se3-package-cpp` through the Skill tool. |
| Orchestrator, `/project-workflow` | `claude-sonnet-5-5`, effort medium (3 runs). |
| Orchestrator, trigger phrase | Stays on session model `claude-opus-5-5` (2 runs). Known limit. |
| Routing dry runs | Step 2 none; step 4 planner; step 6 diagnoser; step 9 git-maintainer. |
| Override audit | No env overrides, no ultracode, no managed settings, no org model limits. Legacy top-level `effortLevel: xhigh` kept. |
| PR assignee | First real PR: this documentation, see the commit history of this folder. |
