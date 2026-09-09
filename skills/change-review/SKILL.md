---
name: change-review
description: Reviews meaningful PRs and patches from relevant technical perspectives, then reports blocker-first findings with clear fixes. Use for pre-merge code review, especially broad or risky changes. Do not use for build checks, trivial style review, or implementation.
---

# Change Review

## Workflow

1. Inspect the diff and relevant surrounding code.
2. Choose perspectives based on the changed domains. Default to code/QA, architecture, and security.
3. Ask parallel subagents to think about the code from one perspective each. Do not discover or invoke repository-defined agents.
4. Verify, deduplicate, and prioritize their findings.
5. Return concise markdown using `references/review-workflow.md`.

Use `references/subagent-prompts.md` for perspective prompts.

If no target is given, compare the current branch with the repository default branch.

## Rules

- Report correctness, security, data, runtime, deployment, and critical test risks first.
- Include a project-relative file and line range for each finding.
- Include a fix only when supported by the code.
- Put unverified concerns under Open Questions.
- Omit empty sections and cosmetic comments.

## Use when

- Reviewing a PR, branch, commit range, or patch before merge.
- Reviewing risky changes from security, architecture, UX, QA, performance, or deployment perspectives.

## Don't use when

- Fixing code or failing tests.
- Checking whether code builds.
- Reviewing only formatting or style.