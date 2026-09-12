---
name: goal
description: Guides agents to complete work against a measurable metric or testable outcome, repeatedly checking progress until the goal passes or a concrete blocker requires user input; uses a verified checklist when no goal is given.
argument-hint: "State the goal and check, or describe the work for a tracked checklist."
---

# Goal

## Use when

- The task has a measurable target, such as a passing test, build, lint check, benchmark, coverage threshold, or expected file output.
- The agent can run a command, inspect an artifact, or otherwise verify whether the target is met.
- The task lacks an explicit goal but needs persistent execution; use a checklist to define and track completion.
- The agent should keep working instead of stopping after a partial attempt.

## Don't use when

- The user wants only advice, a plan, or an explanation rather than execution.
- The work cannot proceed without an external decision, approval, credential, or unavailable environment.

## Inputs

Capture or infer:

- **Goal:** the exact passing condition, when specified.
- **Check:** the command or repeatable inspection that proves the goal, when available.
- **Scope:** files, component, or behavior that may change.
- **Limits:** time, cost, allowed changes, or maximum attempts, if provided.

If the user specifies a goal, do not invent or weaken its metric. If no goal is specified, create a short, concrete checklist from the requested work. Each item must be observable and completable; use existing tests, build commands, or artifact inspection where available.

## How to run

Invoke this skill with a measurable goal and its check. If the task has no goal, invoke it with the requested work; the skill creates a tracked checklist before implementation.

## Procedure

1. If a goal is specified, restate it as a binary condition: pass or fail. Otherwise, create and show a concise checklist of the requested work.
2. Run the defined check before editing when practical. For a checklist, mark items as pending and identify the most specific verification for each item.
3. Diagnose the failed condition or complete the next pending checklist item with the smallest root-cause change within scope.
4. Run the same check again, or verify and mark the completed checklist item.
5. Repeat steps 3–4 until the goal passes or every checklist item is complete.
6. Stop and ask the user only when progress is blocked by missing information, access, an external dependency, or an explicit limit.

## Rules

- A goal-driven change is not complete until its defined check passes; a checklist-driven task is not complete until every item is verified.
- Do not treat a successful edit, command exit unrelated to the check, or a plausible explanation as success.
- Do not create a checklist to evade an explicit measurable goal.
- Use the most specific check first; run broader validation only when useful or requested.
- Preserve unrelated user changes.
- If a check is flaky, rerun it enough to identify flakiness and report that the goal cannot be reliably verified.
- If the goal is already met, report that result and do not make unnecessary changes.

## Expected output

Report:

- Status: `met`, `not met`, or `blocked`.
- The exact goal and final check result, or the completed checklist with verification for each item.
- Changed files, if any.
- For `blocked`, the concrete blocker and the smallest user action needed.

## Verification checks

- For a goal, run its stated check and confirm the required result.
- For a checklist, verify and mark every item complete.
- If verification cannot run, report `blocked`; do not report `met`.

## Examples

### Positive: fix until tests pass

Goal: `pytest tests/test_auth.py` exits with status 0.

Run the test, fix its root cause, rerun the same test, and continue until it exits 0.

### Positive: meet a coverage target

Goal: `npm run coverage` reports at least 80% line coverage.

Run coverage, add or adjust only relevant tests, and rerun coverage until the reported value is at least 80%.

### Checklist fallback: no explicit goal

Request: “Add a user settings page.”

Create a short checklist, for example: `[ ] page is reachable`, `[ ] settings form saves`, `[ ] relevant tests pass`. Complete and verify each item before reporting success.

### Negative: inaccessible check

Goal: “Confirm production checkout works,” but production access is unavailable.

Do not claim success. Report `blocked` and request the access or a test environment.

## Gotchas

- A green targeted test does not prove a separate requested build or lint goal.
- Do not change the check or lower the threshold to make the goal pass unless the user explicitly changes the goal.
- Avoid unbounded retries for deterministic failures; investigate and change the cause between checks.
