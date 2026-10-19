---
name: se3-pr-description
description: Write or rewrite pull request descriptions for SE3 repositories. Use when creating a PR, editing a PR body, or when the user asks for a "PR description" in an SE3 repo. Use this instead of the generic `pr` skill in SE3 repos. Output is 5 to 10 technical points in ASD-STE100 Simplified Technical English, with measured numbers.
---

# SE3 PR description

A reviewer reads the description to learn three things: what changed, the evidence that it works, and what is not proven. Give only those. Do not narrate the work.

## Collect the facts first

Do not write until you have these. If a fact is missing, write the limit, not a guess.

- The diff and the base branch: `gh pr diff <n>`, `gh pr view <n> --json baseRefName,commits`.
- The test commands that ran and their counts (`188 passed`). Not "tests pass".
- Hardware results: the board serial, the OS commit or signing job, the container image tag, and the median of the repeated runs with units.
- Related PRs by number and repo (`<repo> #<n>`).

## Format

```markdown
<One or two STE sentences: what the change does.>

## Change
- <file or function>: <what it does now>.

## Evidence
- <command>: <count> passed.
- <board>, <N> runs, medians:

  | <config> | <metric> s | <metric> s |
  |---|---|---|

## Limits
- <what is not tested or not proven, with the reason>.
```

- 5 to 10 bullet points in total, across all sections. A measurement table is one point.
- Each point names a concrete thing: a file, a function, a flag, a unit, a commit SHA, an image tag, a test count.
- A comparison needs a baseline row. Measure the base commit, not a different branch.
- Put the before and after numbers in the same table, with units, and say what each column measures.
- Put a speedup that is inside the run-to-run range in **Limits**, with the range.
- Leave out: process history ("I traced..."), review-tool output, checkbox lists, plans, and repeated facts.
- Title: use the prefix that the target branch uses (for example `[NXP]`), then an imperative verb. Keep titles under 72 characters.
- End with the attribution line that the harness requires, if any.

## ASD-STE100 Simplified Technical English

- One statement per sentence. A procedure sentence has 20 words or fewer. A descriptive sentence has 25 words or fewer.
- Use the active voice and the present tense: "The coordinator deletes `ready.env`", not "`ready.env` is deleted".
- Use one term for one thing. If you write "sensor prep" one time, do not change to "sensor setup" later.
- Do not use "-ing" words as nouns or adjectives when a verb or a noun can replace them.
- Use the articles ("the", "a"). Do not drop them to make telegraph text.
- Do not use filler or hedges: "just", "simply", "basically", "very", "should", "might".
- Write numbers as digits with units: `2.64 s`, `15 Hz`, `507 MB`.
- Write "one time", not "once"; "not tested", not "untested" when the sentence is clearer.

## Example

```markdown
Prepare the camera and the IMU in parallel. Initialize the container environment one time per `podman exec`.

## Change
- New `prepare-sensors` (`ExecStartPre` of `workload.service`): it gets the sensor lock, runs the calibration preflight, then prepares the camera and the IMU in parallel.
- On a failure or a stop, `prepare-sensors` stops both helpers and deletes the readiness files.
- `container`: `exec`, `shell` and `stop` use one environment initializer. No login shell.

## Evidence
- `pytest tests/test_sensor_preparation.py tests/test_runtime_environment.py`: 188 passed.
- <board serial>, 5 warm start/stop cycles, medians:

  | Host scripts | Sensor prep | Start to ready | Stop |
  |---|---|---|---|
  | base `<sha>` | 3.04 s | 29.11 s | 10.53 s |
  | this PR `<sha>` | 2.64 s | 28.08 s | 9.92 s |
- Board checks passed: camera only, IMU only, held sensor lock, failed preflight, stop during preparation (0 helper processes, lock free).

## Limits
- The start-to-ready change (1.0 s) is in the run-to-run range (28.0 s to 29.6 s).
- No signed image build. The board test used bind mounts.
```

## Check before you publish

- [ ] 5 to 10 points in total.
- [ ] Each number has a unit and a source (command, board, image).
- [ ] Every claim in Evidence happened in this work. Nothing is predicted.
- [ ] Limits says what was not built, not run on hardware, or not measured.
- [ ] No sentence breaks the STE rules above.
