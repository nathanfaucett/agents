---
name: goal
description: Guides agents to complete work against a measurable metric or testable outcome, repeatedly checking progress until the goal passes or a concrete blocker requires user input.
argument-hint: 'State the measurable goal, how to check it, and any limits.'
---

# Goal

## Use when
- The task has a measurable target, such as a passing test, build, lint check, benchmark, coverage threshold, or expected file output.
- An agent can run a command, inspect an artifact, or otherwise verify whether the target is met.
- The agent should keep working instead of stopping after a partial attempt.

## Don't use when
- Success is subjective and has no agreed verification method.
- The work requires an external decision, approval, credential, or unavailable environment.
- The user wants only advice, a plan, or an explanation rather than execution.

## Inputs
Capture or infer:
- **Goal:** the exact passing condition.
- **Check:** the command or repeatable inspection that proves the condition.
- **Scope:** files, component, or behavior that may change.
- **Limits:** time, cost, allowed changes, or maximum attempts, if provided.

If the goal or check is missing, ask for the minimum clarification needed. Do not invent a metric.

## Procedure
1. Restate the goal as a binary condition: pass or fail.
2. Run the check before editing when practical to establish the current state.
3. Diagnose the failed condition and make the smallest root-cause change within scope.
4. Run the same check again.
5. Repeat steps 3–4 until the check passes.
6. Stop and ask the user only when progress is blocked by missing information, access, an external dependency, or an explicit limit.

## Rules
- A change is not complete until the defined check passes.
- Do not treat a successful edit, command exit unrelated to the check, or a plausible explanation as success.
- Use the most specific check first; run broader validation only when useful or requested.
- Preserve unrelated user changes.
- If a check is flaky, rerun it enough to identify flakiness and report that the goal cannot be reliably verified.
- If the goal is already met, report that result and do not make unnecessary changes.

## Expected output
Report:
- Goal status: `met`, `not met`, or `blocked`.
- The exact check run and its final result.
- Changed files, if any.
- For `blocked`, the concrete blocker and the smallest user action needed.

## Examples

### Positive: fix until tests pass
Goal: `pytest tests/test_auth.py` exits with status 0.

Run the test, fix its root cause, rerun the same test, and continue until it exits 0.

### Positive: meet a coverage target
Goal: `npm run coverage` reports at least 80% line coverage.

Run coverage, add or adjust only relevant tests, and rerun coverage until the reported value is at least 80%.

### Negative: subjective improvement
Request: “Make this dashboard feel more polished.”

Do not use this skill unless the user provides acceptance criteria or a verifiable review method.

### Negative: inaccessible check
Goal: “Confirm production checkout works,” but production access is unavailable.

Do not claim success. Report `blocked` and request the access or a test environment.

## Gotchas
- A green targeted test does not prove a separate requested build or lint goal.
- Do not change the check or lower the threshold to make the goal pass unless the user explicitly changes the goal.
- Avoid unbounded retries for deterministic failures; investigate and change the cause between checks.
