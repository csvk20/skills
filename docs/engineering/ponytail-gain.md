## What it does

`ponytail-gain` displays [ponytail](https://aihero.dev/skills-ponytail)'s published benchmark impact, lines of code, cost, and speed, as a compact one-shot ASCII scoreboard.

The defining constraint is that the figures are fixed benchmark medians from upstream (5 everyday tasks across three models), never a number computed from your current repo. It will not print a per-repo savings claim, because the unbuilt version of your code was never written, so there is no real baseline to subtract from.

## When to reach for it

Type `/ponytail-gain`, or ask "what does ponytail save" or "show ponytail impact."

| Your situation | Where to go |
| --- | --- |
| Want the published benchmark numbers for ponytail itself | `ponytail-gain` |
| Want a real, per-repo number for what was actually deferred | [ponytail-debt](https://aihero.dev/skills-ponytail-debt), a counted ledger, not an estimate |

## It's working if

- The scoreboard never claims a number specific to your repo.
- It points you at `/ponytail-debt` and `/ponytail-audit` for the real, repo-specific figures.

## Where it fits

A one-shot display, not a step in any flow. When you're unsure which skill or flow fits a task, [ask-matt](https://aihero.dev/skills-ask-matt) routes you over the whole set.
