---
name: review
description: |
   Provides structured analysis and feedback for documents, code, and data by
   identifying duplication, clarity gaps, accuracy issues, and completeness
   risks. Invoke when a user requests a quality review, second opinion, or
   actionable improvement recommendations for static content.
---

## Summary

Use this skill for static content quality reviews that produce clear findings and practical improvements.

## When to use
- Reviewing documents for clarity, correctness, and completeness.
- Reviewing code for duplication, maintainability, and readability risks.
- Reviewing datasets for obvious consistency or redundancy issues.
- Providing a second opinion before publishing or merging content.

## When not to use
- Runtime debugging or execution failures that require running systems or logs.
- Security audits or compliance checks that need specialized security workflows.
- Tasks requiring direct implementation rather than analysis.
- Reviews that depend on unavailable external business context.

## Inputs
- Content to review (document, code, or data).
- Requested focus (for example: clarity, duplication, completeness, consistency).
- Optional context: style guide, acceptance criteria, constraints, or target audience.

## Outputs
- Structured review report with:
  1. Key findings (prioritized)
  2. Duplications
  3. Clarity/completeness gaps
  4. Accuracy/consistency issues
  5. Actionable improvements
  6. Open questions (if context is missing)

## Procedure
1. Clarify scope only if review focus is ambiguous.
2. Inspect content for duplication, ambiguity, inconsistency, and omission.
3. Apply provided standards or default community conventions.
4. Produce a concise, prioritized report with actionable fixes.
5. Separate confirmed findings from context-dependent questions.

## Best practices and constraints
- Keep findings specific and evidence-based.
- Distinguish objective issues from style preferences.
- Prefer actionable recommendations over generic commentary.
- Do not claim runtime behavior; this skill is static analysis only.

## Gotchas and edge cases
- Missing context can make valid patterns appear inconsistent; surface as open questions.
- Very small snippets can hide dependencies; avoid overconfident conclusions.
- Domain-specific correctness may require expert validation beyond this review.

## Examples
- "Review this requirements doc for missing acceptance criteria and ambiguity."
- "Review this module for duplication and maintainability risks."
- "Review this dataset description for consistency and clarity."

## Acceptance criteria
- Report is structured, prioritized, and actionable.
- Findings clearly separate confirmed issues from open questions.
- Recommendations are tied to observed content, not speculation.

## Notes for agents
- Ask at most one clarifying question when scope is unclear.
- Cite exact sections or lines when available.
- Keep output concise and decision-oriented.

## Limitations
- Does not execute code or validate runtime behavior.
- May miss highly domain-specific issues without additional context.
- Feedback quality depends on the completeness of provided inputs.
