---
name: change-review
description: |
   Provides a structured code review of a proposed change, including feedback on correctness, style, maintainability, and potential impacts. Invoke when a team needs a second opinion on a code change, wants to ensure quality before merging, or needs to identify potential issues early in the development process.
---

## Summary
Use this skill for structured pre-merge review. The default stance is blocker-first and risk-first, with concrete fixes included when they are clear and high confidence.

## When to use
- A PR, patch, or local diff needs structured review before merge.
- A change spans multiple layers and benefits from independent specialist perspectives.
- The team wants a synthesized findings list rather than raw reviewer commentary.
- The diff is large enough that parallel inspection is more efficient than one sequential pass.

## When not to use
- The user only wants a compile check or a short diff summary.
- The task is to implement or fix code rather than review it.
- The change is trivial enough that multi-agent review would add more noise than signal.
- The user wants a purely stylistic pass with no concern for correctness, risk, or regressions.

## Inputs

- Change under review: commit range, PR, branch diff, patch, or changed-file list.
- Scope such as full review, security-only, architecture-only, UX-only, or release-blocking issues only.
- Purpose, linked issue, or expected behavior.
- Constraints, exclusions, or sensitive areas.
- Existing evidence such as tests run, screenshots, benchmarks, or rollout notes.

If the review target is omitted, review the current branch diff against the repository default branch.

## How to run

1. Inspect the actual diff before launching any reviewers.
2. Choose the default review lenses: code-and-QA, architecture, and security.
3. Follow the detailed workflow in `references/review-workflow.md` for reviewer selection, safety mode, and synthesis.
4. Use `references/subagent-prompts.md` when dispatching specialist reviewers.
5. Render the final review from the existing templates and schema references.

## Expected outputs

The final user-visible deliverable is markdown only. Internally, the parent reviewer may assemble `json_review` data, but it must not be printed raw.

The final review should:

- Put must-change findings first.
- Separate findings into Blockers, Bugs, Breaking Changes, Suggestions, and Nitpicks.
- Include a project-relative file path and specific line or range for every finding.
- Preserve clear, high-confidence remediation ideas when available.
- Distinguish confirmed findings from open questions.

## Blocker-first policy

- Prioritize correctness, data integrity, security boundary issues, broken deploy or runtime behavior, and critical test gaps.
- Downgrade style-only comments unless they materially affect maintainability, accessibility, or future defect risk.
- Route pure cosmetic commentary to Nitpicks.

## Case references

| Case | Reference |
|------|-----------|
| Full review workflow, safety mode, agent discovery, and synthesis rules | `references/review-workflow.md` |
| Reusable specialist subagent prompts | `references/subagent-prompts.md` |
| Internal review schema and scoring rules | `references/json-review-schema.md` |
| Parent reviewer rendering contract | `templates/reviewer-output.md` |
| Specialist subagent output contract | `templates/subagent-output.md` |

## VS Code operating constraints

- Pre-fetch terminal context in the parent turn before launching subagents.
- Launch each subagent wave in a single batch.
- Keep prompts self-contained.
- Retry only the failed reviewer if a subagent hangs or returns nothing.

## Gotchas

- Do not present raw subagent output as the final review.
- Do not invent fixes unsupported by the diff or surrounding code.
- Do not drop file and line metadata during synthesis.
- Do not paste massive raw diffs into one prompt; chunk large changes.

## Routing examples

- `Review this PR using code-and-QA plus the relevant specialists in parallel.`
- `Do a security-heavy review of these auth changes and only report merge blockers.`
- `Review this front-end diff with UX and code-and-QA lenses.`
- `Run a parallel review on this branch and tell me if there are rollout risks before merge.`

Negative:

- `Fix the failing test in this branch.`
- `Tell me whether this builds.`