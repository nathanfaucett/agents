# Subagent Prompts

Use these prompt patterns when dispatching specialist reviewers.

## Shared rules for every specialist

- Categorize findings as Blockers, Bugs, Breaking Changes, Suggestions, or Nitpicks.
- Reserve Nitpicks for trivial cosmetic items with no correctness, accessibility, or maintenance consequence.
- Include `file` and `lines` for every finding.
- Include a concrete `suggested_fix` when clear; otherwise say `No confident fix proposed`.
- Convert anything that cannot be pinned to a specific line into an open question.
- Do not decide merge gating. The parent reviewer decides that.

## Prompt patterns

### Code and QA reviewer

`Review this change for correctness, maintainability, edge cases, performance, test quality, and release confidence. Categorize findings as Blockers, Bugs, Breaking Changes, Suggestions, or Nitpicks. Include the file path relative to the project root and line numbers for every finding. Return open questions and test gaps. Follow the fix guidance in the parent prompt.`

### Architecture reviewer

`Review this change for architectural fit, scalability, reliability, and system design risk. Categorize findings as Blockers, Bugs, Breaking Changes, Suggestions, or Nitpicks. Include the file path relative to the project root and line numbers for every finding. Return open questions and operability concerns. Follow the fix guidance in the parent prompt.`

### Security reviewer

`Review this change for security issues, trust-boundary mistakes, unsafe data handling, auth or authz problems, dependency risk, and abuse-case coverage gaps. Categorize findings as Blockers, Bugs, Breaking Changes, Suggestions, or Nitpicks. Include the file path relative to the project root and line numbers for every finding. Include likely exploit or impact context. Follow the fix guidance in the parent prompt.`

### UX or UI reviewer

`Review this user-facing change for UX regressions, accessibility issues, interaction inconsistencies, responsive concerns, and unclear copy. Categorize findings as Blockers, Bugs, Breaking Changes, Suggestions, or Nitpicks. Include the file path relative to the project root and line numbers for every finding. Return testing gaps. Follow the fix guidance in the parent prompt.`

### Additional QA specialist

`Review this change for test adequacy, regression coverage, release confidence, and missing validation for changed behavior. Categorize findings as Blockers, Bugs, Breaking Changes, Suggestions, or Nitpicks. Include the file path relative to the project root and line numbers for every finding. Follow the fix guidance in the parent prompt.`

### DevOps or deployment reviewer

`Review this change for CI or CD risk, deployment safety, infrastructure correctness, runtime configuration issues, observability gaps, and rollback hazards. Categorize findings as Blockers, Bugs, Breaking Changes, Suggestions, or Nitpicks. Include the file path relative to the project root and line numbers for every finding. Include mitigation ideas. Follow the fix guidance in the parent prompt.`

### Explore helper

`Read the relevant modules around this diff and summarize the architectural context, call paths, and likely regression surfaces so the specialist reviewers can stay focused.`