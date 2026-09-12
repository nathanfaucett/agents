---
name: change-review
description: Reviews meaningful PRs and patches from relevant technical perspectives, then reports blocker-first findings with clear fixes. Use for pre-merge code review, especially broad or risky changes. Do not use for build checks, trivial style review, or implementation.
---

# Change Review

## Workflow

1. Inspect the diff and relevant code.
2. Select perspectives; default to code/QA, design, architecture, and security.
3. Run one parallel subagent per perspective; never invoke repository-defined agents.
4. Verify, deduplicate, and prioritize findings.
5. Report with `references/review-workflow.md`; use `references/subagent-prompts.md` for prompts.

Without a target, compare the branch with the default branch.

## Rules

- Report correctness, security, data, runtime, deployment, and critical-test risks first.
- Findings need a project-relative location, evidence, impact, and a clear fix when available.
- Design findings require a concrete simpler path; otherwise use Open Questions for missing intent.
- Omit unverified concerns, empty sections, and cosmetic comments.

## Use when

- Reviewing a PR, branch, commit range, or patch before merge.
- Reviewing risky security, architecture, UX, QA, performance, data, or deployment changes.

## Don't use when

- Fixing code or tests, checking builds, or reviewing only style.
