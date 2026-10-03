## What it does

`asd-ste100` rewrites dense or ambiguous English into [Simplified Technical English](https://www.asd-ste100.org/) (ASD-STE100): the controlled-language standard the aerospace and defense industry built so a maintenance technician, with no author to call, cannot misread an instruction. The skill borrows that same discipline for a different reader: an AI [agent](https://www.aihero.dev/ai-coding-dictionary/agent) or downstream system parsing a tool description, an error message, an inter-agent instruction, or a system prompt, with no human in the loop to resolve ambiguity.

The defining constraint is that it cannot fully apply the standard it is named for. ASD-STE100's lexical rules are defined entirely by an official ~900-word dictionary that is free to read but not free to redistribute, so this skill never reproduces it. It applies the standard's structural rules (active voice, one instruction per sentence, length caps, no semicolons, no phrasal verbs) with full confidence, and treats the lexical rules (one word, one meaning) as a direction of travel it names rather than a dictionary compliance it can verify.

## When to reach for it

Type `/asd-ste100`, or the agent reaches for it automatically when a piece of text reads as dense, hedged, or easy to misparse, and misreading it would cost something.

| Text | Use `asd-ste100`? |
|---|---|
| A tool description, error message, inter-agent instruction, or system prompt a model must parse unsupervised | Yes, in Strict mode |
| A README, PR description, changelog, or other explanatory prose for a mixed human/agent audience | Yes, in STE-flavored mode |
| Creative writing or marketing copy, where voice and persuasion are the point | No. The skill says so and declines rather than flattening the voice |
| The structure of a document an agent reads (a skill, `AGENTS.md`, a spec) | Use [writing-for-agents](https://aihero.dev/skills-writing-for-agents) instead; this skill fixes sentences, not document shape |

## Two modes

The skill picks, or asks you to pick, between two enforcement levels before it rewrites anything:

- **Strict**: procedures, error messages, tool and function descriptions, inter-agent instructions, safety text. Every rule applies, including the lexical lockdown (one word, one meaning, one part of speech).
- **STE-flavored**: READMEs, PR descriptions, changelogs, explanatory prose. The sentence-shape rules still apply in full; the lexical lockdown is dropped, because prose needs some range and a strict rewrite of prose reads as a personality transplant.

## Modality is the guardrail

The skill's single most important rule is one STE itself never needed: **never promote a hedge to a fact.** "The request may have failed" and "the request failed" are different claims, and an aircraft manual never has to choose between them because it only ever states what is true. Agent-facing text constantly reports what a system merely suspects, so collapsing "may have" into a flat assertion is not a simplification, it is a different, false claim. Every rewrite keeps the source's hedges at their original strength and adds no cause, frequency, or mechanism the source did not state.

## Output format

Default output is the rewritten text alone, with no preamble, mode announcement, or violation count. The one addition it allows itself is a trailing `Kept as-is:` line when it deliberately kept a longer phrasing to avoid losing precision. Ask to "show the diff" or "which rules did it break" and it switches to a before/after table naming each violated rule instead.

A deterministic linter, `scripts/ste-lint.py`, backs the structural pass: it flags semicolons, phrasal verbs, nominalization, marketing adjectives, synonym rotation, passive voice, and compound tenses, and by design never flags a hedge.

## Common questions

**Does this replace a real ASD-STE100 compliance check?**
No. The skill never reproduces ASD's licensed ~900-word dictionary, so it cannot verify word-by-word compliance the way the real standard does. For aircraft maintenance documentation or any other certified use, get the standard from its [official downloads page](https://www.asd-ste100.org/STE_downloads.html) and check against it directly.

**Will it make my writing sound robotic?**
In Strict mode, yes, deliberately: that flatness is what makes a sentence parse only one way. STE-flavored mode keeps the explanatory rhythm of normal prose and drops the fixed-vocabulary lockdown, so a README or PR description reads like writing, not a maintenance manual.

**Can it fix a paragraph that has nothing to say?**
No, and it says so rather than polishing around the gap. STE fixes a text's *form*; a hollow paragraph rewritten under these rules comes out short, clean, and still empty.

## It's working if

- The rewritten text reads as one clear claim per sentence, with no hedge lost or invented.
- A violation you ask about comes back naming the specific rule and the specific words that broke it, not a vague "this was unclear."
- Creative or marketing copy you hand it gets declined or left alone, not flattened.

## Where it fits

This is a reach-for-it-anytime standalone, with no state and no chain position: run it on any passage, any time. Its closest neighbour is [writing-for-agents](https://aihero.dev/skills-writing-for-agents), which governs a document's *structure* (steps, reference, pointers) rather than its sentences; use that one when the problem is what the document discloses and where, and this one when the problem is that a sentence itself can be misread. When you're unsure which skill or flow fits a task, [ask-matt](https://aihero.dev/skills-ask-matt) routes you over the whole set.
