## What it does

`ponytail-review` reviews a diff for over-engineering only: what to delete, and what stdlib or native equivalent replaces it. It never touches correctness, security, or performance, those stay in scope for a normal review pass.

The defining constraint is its one-line format. Every finding is `L<line>: <tag> <what>. <replacement>.`, tagged `delete:`, `stdlib:`, `native:`, `reuse:`, `yagni:`, or `shrink:`, never a paragraph hedging about whether something "might be more complex than necessary." It ends on one metric, `net: -<N> lines possible`, or `Lean already. Ship.` when there is nothing to cut.

## When to reach for it

Type `/ponytail-review`, or the agent reaches for it when you ask "what can we delete," "is this over-engineered," or for a "simplify review" of a diff.

| Your situation | Where to go |
| --- | --- |
| Reviewing a diff for cuttable complexity | `ponytail-review` |
| Scanning a whole repo instead of a diff | [ponytail-audit](https://aihero.dev/skills-ponytail-audit) |
| Reviewing the same diff for correctness bugs | [code-review](https://aihero.dev/skills-code-review), a separate pass |
| Writing the code in the first place, lazily | [ponytail](https://aihero.dev/skills-ponytail) |

## Common questions

**Does it flag the self-check ponytail itself asks for?**
No. A single smoke test or assert-based self-check is the ponytail minimum, not bloat, and this skill is told never to flag it for deletion.

**Does it fix what it finds?**
No. It lists findings only; applying them is a separate step you (or another skill) take afterward.

## It's working if

- Every finding names a location, what to cut, and what replaces it, in one line.
- The diff's own complexity goes down after you apply the findings, measured in the `net:` line.

## Where it fits

A reach-for-it-anytime standalone, paired with [ponytail](https://aihero.dev/skills-ponytail) (which prevents the complexity before it's written) and [ponytail-audit](https://aihero.dev/skills-ponytail-audit) (the same hunt, repo-wide). When you're unsure which skill or flow fits a task, [ask-matt](https://aihero.dev/skills-ask-matt) routes you over the whole set.
