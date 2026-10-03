## What it does

`ponytail` is a lazy/YAGNI-first coding discipline: before writing anything, it climbs a ladder of seven rungs (does this need to exist at all, is it already in this codebase, does the standard library do it, does a native platform feature cover it, does an already-installed dependency solve it, can it be one line, only then the minimum code) and stops at the first rung that holds.

The defining constraint is that the ladder runs *after* comprehension, never instead of it. It traces the real flow the change touches before climbing, so the shortest diff it ships is the shortest diff in the right place, not just the shortest diff. A bug fix always goes to the root cause (every caller of the touched function, fixed once where they all route through), never the symptom at the one call site the ticket named.

## When to reach for it

Type `/ponytail`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically on any coding task: writing, adding, refactoring, fixing, reviewing, or designing code, and choosing libraries or dependencies.

| Your situation | Where to go |
| --- | --- |
| Writing or changing code, right now | `ponytail`, active every response once invoked |
| Reviewing a diff someone else wrote, looking only for over-engineering | [ponytail-review](https://aihero.dev/skills-ponytail-review) |
| Auditing a whole repo for cuttable complexity | [ponytail-audit](https://aihero.dev/skills-ponytail-audit) |
| Checking what shortcuts got deferred and never revisited | [ponytail-debt](https://aihero.dev/skills-ponytail-debt) |

## Intensity levels

Three levels trade off how hard the ladder is enforced, switched with `/ponytail lite|full|ultra`:

- **lite**: builds what's asked, names the lazier alternative in one line, you pick.
- **full** (default): the ladder enforced, stdlib and native first, shortest diff and shortest explanation.
- **ultra**: YAGNI extremist, deletion before addition, ships the one-liner and challenges the rest of the requirement in the same breath.

## The debt it leaves on purpose

A deliberate simplification that cuts a real corner (a global lock, an O(n²) scan, a naive heuristic) gets marked with a `ponytail:` comment naming the ceiling and the upgrade trigger, rather than silently shipped as if it were the final answer. [ponytail-debt](https://aihero.dev/skills-ponytail-debt) is the ledger that collects those comments back up so a deferral can't quietly become permanent.

## Common questions

**Won't this skip validation or error handling to stay short?**
No. Input validation at trust boundaries, error handling that prevents data loss, security measures, and accessibility basics are named exceptions the ladder never climbs past, along with anything you explicitly requested.

**Does it skip writing tests to keep the diff small?**
No. Non-trivial logic (a branch, a loop, a parser, a money or security path) always leaves one runnable check behind: an assert-based self-check or one small test, not a full suite unless asked.

## It's working if

- Each diff is the shortest one that is also in the right place, not just the shortest one.
- A deliberate shortcut carries a `ponytail:` comment naming its ceiling, not a silent omission.
- The explanation after the code is three lines or fewer, and gets shorter as the fit gets better.

## Where it fits

A reach-for-it-anytime standalone that runs underneath other skills rather than beside them: it governs *how* code gets built inside [tdd](https://aihero.dev/skills-tdd), [implement](https://aihero.dev/skills-implement), or any ad hoc coding, not a step of its own in the chain. Its companions are [ponytail-review](https://aihero.dev/skills-ponytail-review) and [ponytail-audit](https://aihero.dev/skills-ponytail-audit) (the same complexity hunt, applied after the fact instead of during writing) and [ponytail-debt](https://aihero.dev/skills-ponytail-debt) (the ledger of what it deferred). When you're unsure which skill or flow fits a task, [ask-matt](https://aihero.dev/skills-ask-matt) routes you over the whole set.
