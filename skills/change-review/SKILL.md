---
name: change-review
description: |
   Provides a structured code review of a proposed change, including feedback on correctness, style, maintainability, and potential impacts. Invoke when a team needs a second opinion on a code change, wants to ensure quality before merging, or needs to identify potential issues early in the development process.
---

## Summary
Use this skill for structured pre-merge review.

- Default stance: blocker-first and risk-first.
- Subagents report all material findings; only the parent reviewer decides `must-change`.
- Process: see "Parallel review workflow" below.
- Use at least 2 reviewers for any non-trivial change: one code-and-QA reviewer plus relevant specialists.
- Use read-only repository exploration (`Explore`) when the diff touches unfamiliar areas or needs dependency tracing.
- For large or high-churn changes, triage first and review in chunks using file lists, stats, and targeted hunks rather than a full raw diff.

## When to use
- A PR, patch, or local diff needs structured review before merge.
- A change spans multiple layers and benefits from independent specialist perspectives.
- The team wants a synthesized findings list rather than raw reviewer commentary.
- The repo or diff is large enough that parallel inspection is more efficient than one sequential pass.

## When not to use
- The user only wants a narrow answer such as whether something compiles or a short diff summary.
- The task is to implement or fix code rather than review it.
- The change is trivial enough that multi-agent review would add more noise than signal.
- The user wants a purely stylistic pass with no concern for correctness, risk, or regressions.

## Inputs
Provide as many of these as possible:

- Change under review: commit range, PR, branch diff, patch, or changed-file list.
- Scope: full review, security-only, architecture-only, UX-only, or release-blocking issues only.
- Purpose, linked issue, or expected behavior.
- Constraints: deadlines, exclusions, sensitive areas, non-goals.
- Existing evidence: tests run, screenshots, benchmarks, rollout notes.

If the review target is omitted, review the current branch diff against the repository default branch. If the target is ambiguous, resolve the minimum missing context first: what changed, intended behavior, and whether the user wants findings only or also suggested fixes.

## Outputs
Produce one synthesized review package with:

1. Verdict
   - One of `Approve`, `Needs changes`, `Question`, `Block`
   - Short rationale
   - Verdict rules:
     - `Block`: one or more confirmed merge blockers
     - `Needs changes`: no hard blocker, but at least one required fix remains
     - `Question`: missing context prevents merge confidence
     - `Approve`: no required fixes remain
2. Blockers — issues that must be resolved before merge
3. Bugs — defects causing incorrect or undefined behavior
4. Breaking Changes — backwards-incompatible changes to public contracts, APIs, or user-facing behavior
5. Suggestions — non-blocking improvements: style, maintainability, performance, test coverage, minor UX polish
6. Coverage summary
   - Applied lenses: architecture, correctness, security, UX/accessibility, testing, operability
   - Skipped lenses and why
7. Open questions
8. Residual risks
9. Change summary — brief description of what changed; positive notes only after findings

### Line-number and file-path requirements
Every finding in every section **must** include:
- `file`: path relative to the project root (never absolute, never shortened)
- `lines`: specific line number or range (e.g. `L42` or `L42-L56`) taken directly from the diff or file read

This ensures the final report can be used to generate targeted commits or patches. If a finding cannot be pinpointed to a line, it must be filed as an open question instead.

## Blocker-first policy
- Prioritize actionable blockers and required fixes over low-risk commentary.
- Treat the following as potential blockers when evidenced in code: correctness bugs, data loss or integrity risk, security boundary violations, broken deploy/runtime behavior, and critical test gaps on changed behavior.
- Include style or preference notes only when they materially affect maintainability, readability, accessibility, or future defect risk.
- If blockers exist, present them first and avoid diluting the review with non-essential nits.

## Output formatting
Use the provided templates.

- `templates/subagent-output.md` for specialist reviews.
  - Include agent name, review lens, and clear AI attribution.
  - Findings must be categorized into: Blockers, Bugs, Breaking Changes, Suggestions.
  - Every finding must include a file path relative to the project root and a line number or range.
  - Subagents must not classify merge gating; the parent reviewer decides what is a Blocker.
- `templates/reviewer-output.md` for the final synthesized review.
  - Always include:
    - Header: `Conducted By: AI Review Agent`
    - List of reviewer agents used
    - Attribution of each finding to its source agent
    - Footer: `Review conducted entirely by AI`
  - Sections must appear in order: Blockers → Bugs → Breaking Changes → Suggestions.
  - The report must be machine-parseable for commit generation: every entry must have `file` and `lines` fields in consistent format (`L<n>` or `L<n>-L<m>`).

## VS Code operating constraints
These rules apply in VS Code Copilot agent mode:

- Pre-fetch all context before launching subagents. Run every `run_in_terminal` call in the parent turn and embed results in each subagent prompt. Subagents must not call `run_in_terminal`.
- Launch every subagent in a wave in one tool-call batch. Sequential `runSubagent` calls run serially.
- Do not share mutable workspace state between concurrent subagents. Each prompt must be self-contained.
- If a subagent hangs or returns nothing, retry it alone in a later turn with a simpler prompt.

## Required operating mode
- The parent reviewer owns the final review and synthesis.
- Review the actual diff before launching subagents.
- Discover available agents from the local `agents/` directory before selecting reviewers.
- Ensure one reviewer covers general code quality and correctness.
- Give each subagent a clear lens, relevant file set, explicit instructions to return findings, and inline diff context.
- Keep specialists independent; do not ask one reviewer to incorporate another reviewer's opinion.
- Deduplicate overlapping findings in the parent review and keep the strongest version.
- If a lens is irrelevant, skip it and state that in coverage.
- Final output must be evidence-oriented, concise, and ordered by user impact and regression risk, with must-change findings before non-blocking observations.

## Large-project safety mode
Use this mode when the review target is large or likely to exceed context limits.

- Trigger if any are true:
  - More than 150 changed files
  - More than 10,000 changed lines (`added + deleted`)
  - `git diff` is too large to review in one pass
- Use a 2-pass review:
  - Pass 1: triage with `--name-only`, `--stat`, `--numstat`, hotspot mapping, and lens selection
  - Pass 2: deep review of prioritized chunks, then optional remaining chunks
- Chunking rules:
  - Up to 25 files or about 2,500 changed lines per chunk
  - Group by concern such as auth, infra, UI, data model, tests
  - Include only chunk-specific hunks plus minimal surrounding context
- Parallelism:
  - Follow VS Code operating constraints for launching subagent waves.
  - Process extra chunks or lenses in later waves and synthesize incrementally
- Reporting:
  - If review is partial, state coverage clearly and list unreviewed areas
  - Prioritize blockers before lower-risk commentary

## Parallel review workflow
1. Establish the review target.
   - Identify the diff, commit range, PR, or changed files.
   - If unspecified, resolve the default branch and review the current branch against it.
   - Prefer merge-base diff for branch review, for example `git diff <default-branch>...HEAD`.
   - Helpful command sequence:
     ```bash
     default_branch="$(git remote show origin | sed -n '/HEAD branch/s/.*: //p')"
     if [ -z "$default_branch" ]; then
        default_branch="$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')"
     fi
     [ -n "$default_branch" ] || { echo "Could not determine default branch"; exit 1; }

     git fetch origin "$default_branch" --quiet
     git diff --stat "origin/$default_branch...HEAD"
     git diff --name-only "origin/$default_branch...HEAD"
     git diff --numstat "origin/$default_branch...HEAD"
     ```
   - On large changes, run full `git diff` only for selected high-risk chunks.
   - If the default branch cannot be determined, ask the user instead of guessing.
   - Confirm whether the user wants findings only or also suggested fixes.
2. Discover available agents.
   - Scan `agents/*.agent.md`.
   - Read each file's frontmatter `name` and `description`.
   - Build a roster of `{ name, description }`.
   - If no local agents exist, use inline lens prompts only.
3. Gather baseline context.
   - Start with changed-file inventory and stats.
   - Read only changed files and directly related surrounding code.
   - Check tests, build files, configs, and touched docs when relevant.
  - Use read-only repository exploration (`Explore`) first when the area is unfamiliar.
   - Prefer targeted `grep -r` lookups and symbol-level reads over full-file reads on large changes.
4. Choose specialist reviewers.
   - Determine required lenses from the diff.
   - Match each lens to the best agent by scanning `description` for relevant intent.
   - If multiple agents match, choose the most specific.
   - If none match, use the inline lens prompt.
   - Always start with code-and-QA; add other lenses only when justified.
5. Dispatch subagents in parallel.
  - Follow VS Code operating constraints for launching subagent waves.
   - Embed the relevant hunks, stats, and file context directly in each prompt.
   - Use named agents when matched; otherwise use a generic subagent plus the lens prompt.
   - Ask for findings, open questions, and testing gaps.
   - Do not ask subagents to decide merge gating.
   - In safety mode, review in waves and synthesize after each wave.
6. Aggregate in the parent reviewer.
   - Merge duplicates.
   - Normalize severity and merge impact labels.
   - Downgrade purely stylistic comments unless they create real maintainability, usability, or defect risk.
   - Resolve disagreements by favoring code, tests, and documented behavior.
7. Produce the final review.
   - Must-change findings first, then remaining findings by severity, then open questions, then brief summary if useful.

## Branching logic
Discover available agents first, then map lenses to agents by description.

- If no review target is provided, resolve the default branch and review the current branch against it.
- If the diff is backend, infra, data, or architecture-heavy:
  - Always include code-and-QA.
  - Add architecture when system boundaries, data models, distributed coordination, or operational design are affected.
  - Add security when secrets, permissions, network boundaries, data access, or third-party integrations are touched.
- If the diff changes CI, deployment, infrastructure-as-code, container setup, environment handling, runtime config, observability, or rollback behavior, add DevOps/deployment.
- If the diff is UI-heavy, add UX/UI and still include code-and-QA.
- If critical behavior changed but test evidence is weak or absent, increase code-and-QA depth and add a dedicated QA agent only if one exists.
- If the diff changes authentication, authorization, payments, file upload, cryptography, tenant isolation, or external callbacks, security is mandatory.
- If the diff touches hot paths, data loops, or throughput-sensitive code, add performance.
- If the diff is large and mixed, partition by concern and apply safety mode.
- If subagents return no findings, say so explicitly and still report residual risks or testing gaps if confidence is limited.

## Review heuristics
Prioritize:

- Merge blockers that make the change unsafe to ship.
- Incorrect behavior or mismatch with stated intent.
- Regressions in edge cases, concurrency, state handling, or data integrity.
- Missing validation, unsafe defaults, privilege issues, or trust-boundary mistakes.
- Test gaps around changed behavior.
- User-facing confusion, accessibility regressions, or inconsistent interaction patterns.
- Maintainability hazards likely to cause near-term breakage.

Deprioritize or omit:

- Pure style nits with no correctness, readability, accessibility, or maintenance consequence.
- Redesign suggestions unless the current implementation is materially risky.
- Speculation unsupported by the diff or surrounding code.
- Feedback not tied to an actionable, user-impacting change.

## Quality bar
The skill succeeds when it:

- Uses parallel review to improve coverage, not just verbosity.
- Produces a consolidated, deduplicated, prioritized final review.
- Separates findings into the four required categories: Blockers, Bugs, Breaking Changes, Suggestions.
- Every finding includes a file path relative to the project root and a specific line number or range.
- Makes blockers immediately obvious at the top of the report.
- Clearly separates confirmed findings from open questions.
- Explains impact in terms of behavior, risk, or user effect.
- States which specialist lenses were applied or skipped.
- Report is structured so any finding can be turned into a targeted commit or patch without ambiguity.

## Guardrails
- Do not present raw subagent output as the final review.
- Do not bury blockers under optional suggestions.
- Do not inflate the review with low-signal style commentary.
- Do not claim a risk unless the diff or surrounding code supports it.
- Do not request or expose secrets or credentials.
- Do not block on hypothetical hardening concerns unrelated to the actual change.
- If confidence is low because context is missing, convert the point into an open question.
- Do not omit file paths or line numbers from any finding; if a line cannot be determined, file an open question instead.
- Do not use absolute file paths; all paths must be relative to the project root.
- Do not paste massive raw diffs into one prompt; use chunked, targeted hunks.
- Do not scan the entire repo when only changed paths matter.
- Do not launch unbounded parallel subagents.
- Do not split one wave of subagents across turns.

## Agent discovery
Before selecting reviewers, build a roster from `agents/*.agent.md`:

1. List all matching files in the local `agents/` directory.
2. Parse frontmatter `name` and `description` from each file.
3. Store `{ name, description }` pairs.
4. Match lenses by keywords in `description`:
   - Code / QA: correctness, maintainability, QA, test, edge case, pre-merge, review
   - Architecture: architecture, system design, scalability, reliability, principal, distributed
   - Security: security, auth, threat, vulnerability, trust boundary, cryptography
   - UX / UI: UX, accessibility, front-end, interaction, design, a11y
   - DevOps / deployment: CI/CD, deployment, infrastructure, pipeline, observability, rollout, runtime
   - Performance: performance, optimization, bottleneck, profiling, latency
5. If multiple agents match, prefer the most specific.
6. If no agent matches, use the inline prompt for that lens.

## Parent reviewer checklist
- [ ] Diff and changed files inspected directly
- [ ] Scale assessed: files, changed lines, complexity
- [ ] Safety mode enabled when needed
- [ ] Relevant surrounding code read
- [ ] `agents/` directory scanned and roster built
- [ ] Lenses matched to agents by description
- [ ] Named agents used when matched; inline prompts used otherwise
- [ ] All terminal commands completed before launching subagents
- [ ] Each wave launched in a single tool-call batch
- [ ] Each prompt is self-contained with inline diff context
- [ ] Chunking and waves used for large diffs
- [ ] Subagent outputs use `templates/subagent-output.md`
- [ ] Reviewer names and AI attribution included in subagent output
- [ ] Duplicate findings merged
- [ ] Findings categorized into: Blockers, Bugs, Breaking Changes, Suggestions (in that order)
- [ ] Every finding has a file path relative to project root and a line number or range
- [ ] Open questions used for any finding that cannot be pinpointed to a line
- [ ] Final review uses `templates/reviewer-output.md`
- [ ] Final review includes AI reviewer identification and reviewer list
- [ ] Final review is not copied verbatim from subagents

## Prompt patterns for specialist subagents
All subagents must:
- Categorize findings as: **Blockers**, **Bugs**, **Breaking Changes**, **Suggestions**.
- Include `file` (relative to project root) and `lines` (`L<n>` or `L<n>-L<m>`) for every finding.
- File an open question for any finding that cannot be pinpointed to a specific line.
- Do not classify merge gating; the parent reviewer decides what is a Blocker.

- Code and QA reviewer
  - "Review this change for correctness, maintainability, edge cases, performance, test quality, and release confidence. Categorize findings as Blockers, Bugs, Breaking Changes, or Suggestions. Include the file path (relative to project root) and line number(s) for every finding. Return open questions and test gaps. Do not classify merge gating."
- Architecture-focused reviewer
  - "Review this change for architectural fit, scalability, reliability, and system design risk. Categorize findings as Blockers, Bugs, Breaking Changes, or Suggestions. Include the file path (relative to project root) and line number(s) for every finding. Return open questions and operability concerns. Do not classify merge gating."
- Security reviewer
  - "Review this change for security issues, trust boundary mistakes, unsafe data handling, auth/authz problems, dependency risk, and missing abuse-case coverage. Categorize findings as Blockers, Bugs, Breaking Changes, or Suggestions. Include the file path (relative to project root) and line number(s) for every finding. Include likely exploit/impact context. Do not classify merge gating."
- UX/UI reviewer
  - "Review this user-facing change for UX regressions, accessibility issues, interaction inconsistencies, responsive concerns, and unclear copy. Categorize findings as Blockers, Bugs, Breaking Changes, or Suggestions. Include the file path (relative to project root) and line number(s) for every finding. Return testing gaps. Do not classify merge gating."
- Additional QA specialist (optional)
  - "Review this change for test adequacy, regression coverage, release confidence, and missing validation for changed behavior. Categorize findings as Blockers, Bugs, Breaking Changes, or Suggestions. Include the file path (relative to project root) and line number(s) for every finding. Do not classify merge gating."
- DevOps or deployment reviewer
  - "Review this change for CI/CD risk, deployment safety, infrastructure correctness, runtime configuration issues, observability gaps, and rollback hazards. Categorize findings as Blockers, Bugs, Breaking Changes, or Suggestions. Include the file path (relative to project root) and line number(s) for every finding. Include mitigation ideas. Do not classify merge gating."
- `Explore`
  - "Read the relevant modules around this diff and summarize the architectural context, call paths, and likely regression surfaces so the specialist reviewers can stay focused."

## Example requests
- "Review this PR using a code-and-QA reviewer plus the relevant specialists in parallel, then synthesize a final findings list."
- "Do a security-heavy review of these authentication changes and only report merge-blocking issues."
- "Review this front-end diff with a UX/UI and code-and-QA lens, then give me a concise final review."
- "Run a parallel review on this branch and tell me whether there are any correctness, deployment, or rollout risks before merge."