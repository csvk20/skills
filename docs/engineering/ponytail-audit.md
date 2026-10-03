## What it does

`ponytail-audit` runs the same over-engineering hunt as [ponytail-review](https://aihero.dev/skills-ponytail-review), scanned across the whole codebase instead of a diff. It produces a ranked list of what to delete, simplify, or replace with a stdlib or native equivalent, biggest cut first, and applies nothing.

The defining constraint is the grep-first rule before any `delete:` finding: before claiming a symbol is dead, it greps the whole tree for it, including tests, fixtures, and string or dynamic references, so a finding that looks unused but is reached dynamically never gets reported as deletable.

## When to reach for it

Type `/ponytail-audit`, or the agent reaches for it when you ask to "audit this codebase for over-engineering" or "find bloat" across the whole repo.

| Your situation | Where to go |
| --- | --- |
| A periodic sweep of the whole repo for cuttable complexity | `ponytail-audit` |
| Reviewing one diff instead of the whole tree | [ponytail-review](https://aihero.dev/skills-ponytail-review) |
| Checking what was deliberately deferred, not what's dead | [ponytail-debt](https://aihero.dev/skills-ponytail-debt) |

## It's working if

- Each finding names a tag, what to cut, its replacement, and a file path.
- The closing `net:` line gives a real line-count and dependency-count estimate, not a vague "looks cleaner."

## Where it fits

Periodic maintenance, run whenever you have a spare moment to keep the codebase lean, alongside [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) (which looks for deepening opportunities rather than deletions). When you're unsure which skill or flow fits a task, [ask-matt](https://aihero.dev/skills-ask-matt) routes you over the whole set.
