## What it does

`ponytail-debt` harvests every `ponytail:` comment in the codebase into a single debt ledger, so the deliberate shortcuts [ponytail](https://aihero.dev/skills-ponytail) leaves behind get tracked instead of rotting into "later means never."

The defining constraint is that it reports, it never writes. A `ponytail:` comment names a ceiling (the limit the shortcut is good for) and an upgrade path (the trigger to revisit it); this skill greps them all up, flags any comment missing a trigger as `no-trigger` (the rot risk), and only persists the ledger to a file if you ask it to.

## When to reach for it

Type `/ponytail-debt`, or the agent reaches for it when you ask "what did ponytail defer" or "list the shortcuts."

| Your situation | Where to go |
| --- | --- |
| Checking what shortcuts are still outstanding and why | `ponytail-debt` |
| Finding dead or cuttable code that was never marked as a deliberate shortcut | [ponytail-audit](https://aihero.dev/skills-ponytail-audit) |

## It's working if

- Every `ponytail:` comment in the repo shows up as one ledger row, grouped by file.
- A comment with no named upgrade trigger gets flagged `no-trigger`, not silently listed as if it were fine.

## Where it fits

A reach-for-it-anytime standalone, the read side of the shortcuts [ponytail](https://aihero.dev/skills-ponytail) writes. When you're unsure which skill or flow fits a task, [ask-matt](https://aihero.dev/skills-ask-matt) routes you over the whole set.
