---
name: change-review
description: |
   Provides a structured code review of a proposed change, including feedback on correctness, style, maintainability, and potential impacts. Invoke when a team needs a second opinion on a code change, wants to ensure quality before merging, or needs to identify potential issues early in the development process.
---

## Summary
Use this skill for a structured review of a code change before merge.

It uses a fan-out/fan-in model: the parent reviewer inspects the diff, launches multiple specialist subagents in parallel, then synthesizes their feedback into one review. Reviewer selection is role-based, not tied to fixed agent files. Use at least 2 reviewers for any non-trivial change: one code-and-QA reviewer plus specialists as needed, such as UX/UI, security, and DevOps or deployment when infra, release, or runtime configuration is involved. Use `Explore` for quick read-only repository context when the diff touches unfamiliar areas or needs dependency tracing.

This repository ships a default reviewer set for code and test quality, architecture, security, UX/UI, and DevOps review, but the workflow should still work with any user-provided agents that cover the same lenses.

## When to use
- A pull request, patch, or local diff needs a structured review before merge.
- A team wants broader coverage across architecture, security, and UX/accessibility.
- A change spans multiple layers and would benefit from independent specialist perspectives.
- A reviewer wants a synthesized findings list rather than raw commentary from multiple reviewers.
- The repository or change is large enough that parallel inspection is faster than a single sequential review.

## When not to use
- The user only wants a narrow answer such as "does this compile" or "summarize this diff".
- The task is to implement or fix the code rather than review it.
- The change is trivial and a heavyweight multi-agent review would add noise instead of signal.
- The user asks for a purely stylistic pass with no concern for correctness, risk, or regressions.

## Inputs
Provide as many of these as possible:

- The change under review: commit range, pull request, branch diff, patch, or list of changed files.
- If the user does not specify the change under review, use the current branch diff against the repository default branch as the review target.
- Review scope: full review, security-only, architecture-only, UX-only, or release-blocking issues only.
- Context on the purpose of the change, linked issue, or expected behavior.
- Constraints such as deadlines, non-goals, sensitive areas, or files that should be excluded.
- Evidence already available, such as tests run, screenshots, benchmark results, or rollout notes.

If the review target is ambiguous, resolve the minimum missing context first: what changed, what the change is meant to do, and whether the user wants only findings or also suggested fixes.

## Outputs
Produce a single synthesized review package containing:

1. Verdict
   - One of: `Approve`, `Needs changes`, `Question`, or `Block`
   - Short rationale
2. Findings
   - Prioritized list of concrete issues
   - Each finding should include severity, impact, why it matters, and where it appears
3. Coverage summary
   - Which review lenses were applied: architecture, correctness, security, UX/accessibility, testing, operability
   - Which lenses were intentionally skipped and why
4. Open questions
   - Assumptions, missing context, or unclear intent that affects confidence
5. Residual risks
   - Risks not severe enough to block but worth noting before merge or release
6. Secondary summary
   - Brief change summary or positive notes only after findings are covered

## Required operating mode
- The parent reviewer agent must own the final review and synthesis.
- Review the actual diff first before launching subagents.
- Start multiple subagents in parallel whenever the change is non-trivial.
- Use a minimum of 2 reviewers for any non-trivial review.
- Ensure one reviewer covers general code quality and correctness.
- Each subagent must receive a clear lens, relevant file set, and explicit instructions to return findings rather than generic commentary.
- Prefer specialist independence: do not ask one subagent to incorporate another subagent's opinion.
- Deduplicate overlapping findings in the parent review and keep the strongest version.
- If a specialist area is not relevant, skip that subagent and state that in the coverage summary.
- The final review must be evidence-oriented, concise, and prioritized by user impact and regression risk.

## Parallel review workflow
1. Establish the review target
   - Identify the diff, commit range, PR, or changed files.
   - If the user does not provide one, determine the repository default branch and review the current branch against it.
   - Prefer the merge-base diff for branch review, for example `git diff <default-branch>...HEAD`, so the review covers only changes introduced by the current branch.
   - If the default branch cannot be determined from repository metadata or git configuration, ask the user instead of guessing.
   - Confirm whether the user wants findings only or also suggested fixes.
2. Gather baseline context
   - Read the changed files and surrounding code paths.
   - Check tests, build files, configs, and any touched documentation when relevant.
   - Use `Explore` first if the repository area is unfamiliar or dependency tracing is needed.
3. Choose specialist reviewers
   - Start with a code and QA reviewer for correctness, maintainability, edge cases, test quality, and release confidence.
   - Use an architecture-focused reviewer separately when the diff raises broader system design, scalability, or cross-cutting design concerns.
   - Add a security reviewer for auth, data handling, trust boundaries, secrets, dependency, or abuse-case risk.
   - Add a UX/UI reviewer for user-facing behavior, accessibility, interaction design, responsive impact, or copy.
   - Add a dedicated QA specialist only if one is available and the change has unusually high release or regression risk.
   - Add a DevOps or deployment reviewer when CI/CD, infra, runtime config, observability, rollout, or operational behavior is touched.
   - Add more specialists only when the change justifies it.
   - Omit lenses that are clearly irrelevant to the diff, but keep the minimum reviewer count.
4. Dispatch subagents in parallel
   - Give each one the same change summary and diff context.
   - Tailor the ask to that specialist's lens.
   - Instruct them to return only actionable findings, open questions, and notable testing gaps.
5. Aggregate results in the parent reviewer
   - Merge duplicate findings.
   - Downgrade purely stylistic comments unless they create real maintenance or usability risk.
   - Resolve disagreements by favoring evidence from the code, tests, or documented behavior.
6. Produce the final review
   - Findings first, ordered by severity.
   - Open questions second.
   - Brief change summary last, only if useful.

## Branching logic
- If no review target is provided:
   - Resolve the default branch first.
   - Review the current branch diff against that default branch.
- If the diff is backend, infra, data, or architecture-heavy:
   - Always include a code-and-QA reviewer.
   - Include an architecture-focused reviewer when the change affects system boundaries, data models, distributed coordination, or operational design.
  - Include a security reviewer when secrets, permissions, network boundaries, data access, or third-party integrations are touched.
- If the diff changes CI, deployment, infrastructure-as-code, container setup, environment handling, runtime configuration, observability, or rollback behavior:
  - Include a DevOps or deployment reviewer.
- If the diff is UI-heavy:
   - Include a UX/UI reviewer.
   - Still include a code-and-QA reviewer for correctness and maintainability.
- If the diff changes critical behavior but test evidence is weak or absent:
   - Treat this as mandatory depth for the code and QA reviewer, and escalate with an extra QA specialist only when available.
- If the diff changes authentication, authorization, payments, file upload, cryptography, tenant isolation, or external callbacks:
  - Treat the security reviewer as mandatory.
- If the diff is small but risky:
  - Run fewer subagents, but do not skip the relevant specialist for the risk area.
  - Maintain at least 2 reviewers.
- If the diff is large and mixed:
  - Partition file groups or concerns before dispatching subagents so prompts stay focused.
- If the subagents return no findings:
  - State that explicitly.
  - Still report residual risks or testing gaps if confidence is limited.

## Review heuristics
Prioritize findings that indicate:

- Incorrect behavior or mismatch with stated intent.
- Regressions in edge cases, concurrency, state handling, or data integrity.
- Missing validation, unsafe defaults, privilege issues, or trust boundary mistakes.
- Test gaps around newly introduced behavior or bug fixes.
- User-facing confusion, accessibility regressions, or inconsistent interaction patterns.
- Maintainability hazards that are likely to cause near-term breakage.

Deprioritize or omit:

- Pure style nits with no correctness, readability, accessibility, or maintenance consequence.
- Suggestions that require redesign unless the current implementation is meaningfully risky.
- Speculation that is unsupported by the diff or surrounding code.

## Quality bar
The skill is successful when it:

- Uses parallel subagent review to improve coverage, not just verbosity.
- Produces a final review that is consolidated, deduplicated, and prioritized.
- Clearly separates confirmed findings from open questions or assumptions.
- Explains why each issue matters in terms of behavior, risk, or user impact.
- Makes it obvious which specialist lenses were applied and which were skipped.

## Guardrails
- Do not present raw subagent output as the final review.
- Do not inflate the review with low-signal style commentary.
- Do not claim a risk is present unless the diff or surrounding code supports it.
- Do not request or expose secrets, credentials, or sensitive data while reviewing.
- Do not block on hypothetical production-hardening concerns unrelated to the actual change.
- If confidence is low because context is missing, say so directly and convert the point into an open question instead of a finding.

## Parent reviewer checklist
- [ ] Diff and changed files inspected directly
- [ ] Relevant surrounding code read
- [ ] Specialist subagents chosen based on actual change risk
- [ ] Subagents launched in parallel where useful
- [ ] Duplicate findings merged
- [ ] Findings ordered by severity and impact
- [ ] Open questions separated from confirmed issues
- [ ] Final review written by the parent reviewer, not copied verbatim from subagents

## Prompt patterns for specialist subagents
- Code and QA reviewer
   - "Review this change for correctness, maintainability, edge cases, performance, test quality, and release confidence. Return only concrete findings, open questions, and important test gaps."
- Architecture-focused reviewer
   - "Review this change for architectural fit, scalability, reliability, and system design risk. Return only concrete findings, open questions, and notable migration or operability concerns."
- Security reviewer
   - "Review this change for security issues, trust boundary mistakes, unsafe data handling, auth/authz problems, dependency risk, and missing abuse-case coverage. Return prioritized findings only."
- UX/UI reviewer
   - "Review this user-facing change for UX regressions, accessibility issues, interaction inconsistencies, responsive concerns, and unclear copy. Return only actionable findings and testing gaps."
- Additional QA specialist (optional)
   - "Review this change for test adequacy, regression coverage, release confidence, and missing validation for changed behavior. Return concrete coverage gaps, open questions, and release-risk findings only."
- DevOps or deployment reviewer
   - "Review this change for CI/CD risk, deployment safety, infrastructure correctness, runtime configuration issues, observability gaps, and rollback hazards. Return prioritized operational findings only."
- `Explore`
  - "Read the relevant modules around this diff and summarize the architectural context, call paths, and likely regression surfaces so the specialist reviewers can stay focused."

## Example requests
- "Review this PR using a code-and-QA reviewer plus the relevant specialists in parallel, then synthesize a final findings list."
- "Do a security-heavy review of these authentication changes and only report merge-blocking issues."
- "Review this front-end diff with a UX/UI and code-and-QA lens, then give me a concise final review."
- "Run a parallel review on this branch and tell me whether there are any correctness, deployment, or rollout risks before merge."