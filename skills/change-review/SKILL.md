---
name: change-review
description: |
   Provides a structured code review of a proposed change, including feedback on correctness, style, maintainability, and potential impacts. Invoke when a team needs a second opinion on a code change, wants to ensure quality before merging, or needs to identify potential issues early in the development process.
---

## Summary
Use this skill for a structured review of a code change before merge.

Default stance is blocker-first and risk-first: identify merge blockers and required fixes before reporting non-blocking commentary.

Subagents should report all material findings they observe. The parent reviewer alone decides which findings are `must-change` before merge.

It uses a fan-out/fan-in model: the parent reviewer inspects the diff, discovers available agents, selects the best-fit agents for the change, launches them in parallel, then synthesizes their feedback into one review. Reviewer selection is dynamic: the skill scans the local `agents/` directory, reads each agent's `description` frontmatter field, and matches agents to required review lenses using keyword and intent matching. Use at least 2 reviewers for any non-trivial change: one code-and-QA reviewer plus specialists as needed. Use `Explore` for quick read-only repository context when the diff touches unfamiliar areas or needs dependency tracing.

For large projects or high-churn branches, run in staged mode: triage first, then review in chunks. Avoid loading entire diffs into one prompt. Prefer file lists, stats, and targeted hunks so the parent reviewer does not exceed context or tool limits.

If no local `agents/` directory exists or a lens has no matching agent, fall back to an inline prompt using the lens-specific prompt patterns defined in this skill.

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
   - Apply strict verdict rules:
     - `Block`: one or more confirmed merge blockers (must-change before merge)
     - `Needs changes`: no hard blocker, but at least one required fix remains
     - `Question`: missing context prevents confidence on merge readiness
     - `Approve`: no required fixes remain
2. Findings
   - Prioritized list of concrete issues
   - Each finding should include severity, merge impact (`must-change` or `non-blocking`), why it matters, and where it appears
3. Must-change list
   - Explicit list of issues that must be fixed before merge
   - Empty only when verdict is `Approve` or `Question`
4. Coverage summary
   - Which review lenses were applied: architecture, correctness, security, UX/accessibility, testing, operability
   - Which lenses were intentionally skipped and why
5. Open questions
   - Assumptions, missing context, or unclear intent that affects confidence
6. Residual risks
   - Risks not severe enough to block but worth noting before merge or release
7. Secondary summary
   - Brief change summary or positive notes only after findings are covered

## Blocker-first policy
- Prioritize actionable blockers and must-change findings over completeness of low-risk commentary.
- Treat the following as potential blockers by default when evidenced in code: correctness bugs, data loss/integrity risk, security boundary violations, broken deploy/runtime behavior, and critical test coverage gaps on changed behavior.
- Only include style or preference notes when they materially impact maintainability, readability, accessibility, or future defect risk.
- If blockers are found, present them first and avoid diluting the review with non-essential nits.

### Output Formatting
Use the provided templates to format the review response:
- **Subagent output**: Use `templates/subagent-output.md` to format each specialist review. Include the agent name, review lens, and findings in a consistent structure. Clearly identify the reviewer as an AI agent.
   - Subagents must not classify findings as `must-change` vs `non-blocking`; they should report complete findings and evidence.
- **Final review**: Use `templates/reviewer-output.md` to format the synthesized final review. This template provides consistent sections for verdict, findings, coverage, open questions, and residual risks. Always include:
  - A header identifying "Conducted By: AI Review Agent"
  - List of all reviewer agents used
  - Clear attribution of each finding to its source agent
  - A footer noting "Review conducted entirely by AI"

## VS Code parallel execution constraints
These rules apply whenever this skill runs inside VS Code Copilot's agent mode.

- **Pre-fetch all context before launching subagents.** Run every `run_in_terminal` call (git diff hunk generation, file stat collection, `rg` searches) in the parent turn and embed the results in each subagent prompt. Subagents must NOT issue `run_in_terminal` calls; VS Code exposes one shared persistent terminal and concurrent shell commands from separate subagents will interleave and corrupt output.
- **Issue all subagents in one batch per wave.** `runSubagent` calls only execute in parallel when issued in the same assistant tool-call batch (the same response turn). Sequential `runSubagent` calls run serially and negate the benefit of parallel review. Always dispatch the full wave in one turn.
- **Cap concurrent subagents at 2 in normal mode; 3 maximum per wave in safety mode.** Launching more agents simultaneously can exceed VS Code's internal resource limits and cause agents to silently hang or return incomplete results.
- **Do not share mutable workspace state between concurrent subagents.** Each subagent receives a self-contained prompt with all needed diff context inline. Do not have one subagent write files that another subagent will read during the same wave.
- **If a subagent returns no output or hangs, do not retry in parallel.** Re-run it alone in a follow-up turn with a simplified prompt before escalating.

## Required operating mode
- The parent reviewer agent must own the final review and synthesis.
- All output must be formatted using the provided templates to ensure consistency and clear AI attribution:
  - Use `templates/subagent-output.md` for each subagent's formatted findings
  - Use `templates/reviewer-output.md` for the final synthesized review
- Every output must clearly identify the reviewer(s) as AI and include the agent name(s) responsible
- Discover available agents from the `agents/` directory before selecting reviewers (see Agent discovery).
- Review the actual diff first before launching subagents. Collect all terminal output (stats, hunks, file lists) before issuing any `runSubagent` call.
- Start multiple subagents in parallel whenever the change is non-trivial. Issue all subagents for a wave in one tool-call batch (see VS Code parallel execution constraints).
- Use a minimum of 2 reviewers for any non-trivial review.
- Ensure one reviewer covers general code quality and correctness.
- Each subagent must receive a clear lens, relevant file set, and explicit instructions to return findings rather than generic commentary.
- Subagents should return all material findings in scope; classification of merge requirement is reserved for the parent reviewer.
- Prefer specialist independence: do not ask one subagent to incorporate another subagent's opinion.
- Deduplicate overlapping findings in the parent review and keep the strongest version.
- If a specialist area is not relevant, skip that subagent and state that in the coverage summary.
- The final review must be evidence-oriented, concise, and prioritized by user impact and regression risk.
- The final review must list must-change findings explicitly and place them before non-blocking observations.

## Large-project safety mode
Use this mode whenever the review target is large or likely to exceed context limits.

- Trigger safety mode if any of these are true:
   - More than 150 changed files.
   - More than 10,000 changed lines (`added + deleted`).
   - `git diff` output is too large to review in one pass.
- In safety mode, do a 2-pass review:
   - Pass 1 (triage): collect `--name-only`, `--stat`, `--numstat`, map risk hotspots, and select lenses.
   - Pass 2 (deep review): review prioritized chunks only, then optionally continue with remaining chunks.
- Chunking rules:
   - Use chunks of up to 25 files or about 2,500 changed lines, whichever comes first.
   - Group by concern (auth, infra, UI, data model, tests) so each subagent prompt stays focused.
   - Include only the chunk-specific hunks and minimal surrounding context.
- Parallelism limits:
   - Dispatch at most 2 subagents per wave in normal mode; at most 3 per wave in safety mode.
   - In VS Code, all agents in a wave must be issued in the same tool-call batch; do not split a wave across turns.
   - If more lenses/chunks are needed, process in waves and synthesize incrementally between turns.
- Reporting behavior:
   - If review is partial due to scale, state coverage explicitly and list unreviewed areas.
   - Prioritize merge-blocking findings first; defer lower-risk commentary until critical surfaces are covered.

## Parallel review workflow
1. Establish the review target
   - Identify the diff, commit range, PR, or changed files.
   - If the user does not provide one, determine the repository default branch and review the current branch against it.
   - Prefer the merge-base diff for branch review, for example `git diff <default-branch>...HEAD`, so the review covers only changes introduced by the current branch.
    - Helpful command sequence when no review target is provided:
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
      - Only run full `git diff` on selected high-risk chunks, not the entire branch, when the change is large.
   - If the default branch cannot be determined from repository metadata or git configuration, ask the user instead of guessing.
   - Confirm whether the user wants findings only or also suggested fixes.
2. Discover available agents
   - Scan the `agents/` directory (relative to the workspace root) for all `*.agent.md` files.
   - For each file, read the `name` and `description` frontmatter fields.
   - Build a candidate roster: a list of `{ name, description }` pairs representing agents available for selection.
   - If no `agents/` directory exists, proceed using inline lens prompts only (see Prompt patterns for specialist subagents).
3. Gather baseline context
   - Start from changed-file inventory and stats before reading file bodies.
   - Read only changed files and directly related surrounding code paths.
   - Check tests, build files, configs, and any touched documentation when relevant.
   - Use `Explore` first if the repository area is unfamiliar or dependency tracing is needed.
   - Prefer `rg`-based targeted lookups and symbol-level reads over full-file reads for large changes.
4. Choose specialist reviewers
   - Determine which review lenses are needed based on what the diff touches (see Branching logic).
   - For each required lens, select the best-matching agent from the candidate roster by comparing the lens intent against each agent's `description`. Use keyword and intent matching: an agent whose description mentions "security", "auth", or "threat" matches the security lens; one mentioning "UX", "accessibility", or "front-end" matches the UX lens; one mentioning "CI/CD", "deployment", or "infrastructure" matches the DevOps lens; one mentioning "correctness", "maintainability", "QA", or "test" matches the code-and-QA lens; one mentioning "architecture", "scalability", "system design", or "principal" matches the architecture lens; one mentioning "performance" or "optimization" matches the performance lens.
   - If multiple agents match a lens, prefer the one whose description is most specific to that lens.
   - If no agent matches a lens, use the inline prompt pattern for that lens with a generic subagent invocation.
   - Always start with a code-and-QA lens. Add additional lenses only when the diff justifies them.
   - Add a performance lens when hot paths, data processing loops, or profiling concerns are touched.
   - Add a DevOps lens when CI/CD, infra, runtime config, observability, rollout, or operational behavior is touched.
   - Add a security lens for auth, data handling, trust boundaries, secrets, dependency, or abuse-case risk.
   - Add a UX lens for user-facing behavior, accessibility, interaction design, responsive impact, or copy.
   - Omit lenses that are clearly irrelevant to the diff, but keep the minimum reviewer count of 2.
5. Dispatch subagents in parallel
   - **VS Code requirement:** Issue all subagents in a wave as a single tool-call batch in one response turn. Do not interleave agent calls with other tool calls in the same turn.
   - **VS Code requirement:** Embed all diff context (hunks, file stats, file content) inline in each subagent prompt. Do not ask subagents to run terminal commands or fetch context themselves.
   - Give each one the same change summary, but only chunk-specific diff context.
   - When a matched agent was found, invoke it by name using `runSubagent` with that agent name.
   - When falling back to an inline prompt, invoke a generic subagent and include the lens-specific prompt from the Prompt patterns section.
   - Tailor the ask to that specialist's lens.
   - Instruct them to return findings formatted using `templates/subagent-output.md` with their agent name clearly identified.
   - Instruct them to mark themselves as an AI agent in the output.
   - Request complete material findings, open questions, and notable testing gaps.
   - Do not ask subagents to decide merge gating (`must-change` vs `non-blocking`); the parent reviewer decides this during synthesis.
   - In safety mode, run subagents in waves (max 3 concurrent per wave) and merge findings after each wave before starting the next.
6. Aggregate results in the parent reviewer
   - Merge duplicate findings.
   - Normalize severity and merge impact labels (`must-change` vs `non-blocking`) so blockers are unambiguous.
   - Downgrade purely stylistic comments unless they create real maintenance or usability risk.
   - Resolve disagreements by favoring evidence from the code, tests, or documented behavior.
7. Produce the final review
   - Must-change findings first, then remaining findings ordered by severity.
   - Open questions second.
   - Brief change summary last, only if useful.

## Branching logic
In all cases, discover available agents first (see Agent discovery) and map lenses to agents by description before dispatching.

- If no review target is provided:
   - Resolve the default branch first.
   - Review the current branch diff against that default branch.
- If the diff is backend, infra, data, or architecture-heavy:
   - Always include the code-and-QA lens (select the best-matching agent or use the inline prompt).
   - Include the architecture lens when the change affects system boundaries, data models, distributed coordination, or operational design.
   - Include the security lens when secrets, permissions, network boundaries, data access, or third-party integrations are touched.
- If the diff changes CI, deployment, infrastructure-as-code, container setup, environment handling, runtime configuration, observability, or rollback behavior:
   - Include the DevOps/deployment lens.
- If the diff is UI-heavy:
   - Include the UX/UI lens.
   - Still include the code-and-QA lens for correctness and maintainability.
- If the diff changes critical behavior but test evidence is weak or absent:
   - Treat this as mandatory depth for the code-and-QA lens; escalate to a dedicated QA agent only if one is found in the roster.
- If the diff changes authentication, authorization, payments, file upload, cryptography, tenant isolation, or external callbacks:
   - Treat the security lens as mandatory.
- If the diff touches hot paths, data processing loops, or throughput-sensitive code:
   - Include the performance lens.
- If the diff is small but risky:
   - Run fewer subagents, but do not skip the lens covering the primary risk area.
   - Maintain at least 2 reviewers.
- If the diff is large and mixed:
   - Partition file groups or concerns before dispatching subagents so prompts stay focused.
   - Apply Large-project safety mode and review in waves.
- If the subagents return no findings:
   - State that explicitly.
   - Still report residual risks or testing gaps if confidence is limited.

## Review heuristics
Prioritize findings that indicate:

- Merge blockers that make the change unsafe to ship without remediation.
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
- Any feedback that is not tied to an actionable, user-impacting change.

## Quality bar
The skill is successful when it:

- Uses parallel subagent review to improve coverage, not just verbosity.
- Produces a final review that is consolidated, deduplicated, and prioritized.
- Makes merge blockers and must-change issues immediately obvious to the author.
- Clearly separates confirmed findings from open questions or assumptions.
- Explains why each issue matters in terms of behavior, risk, or user impact.
- Makes it obvious which specialist lenses were applied and which were skipped.

## Guardrails
- Do not present raw subagent output as the final review.
- Do not bury blockers under stylistic or optional suggestions.
- Do not inflate the review with low-signal style commentary.
- Do not claim a risk is present unless the diff or surrounding code supports it.
- Do not request or expose secrets, credentials, or sensitive data while reviewing.
- Do not block on hypothetical production-hardening concerns unrelated to the actual change.
- If confidence is low because context is missing, say so directly and convert the point into an open question instead of a finding.
- Do not paste massive raw diffs into a single prompt; use chunked, targeted hunks.
- Do not scan entire repositories when only a subset of changed paths is required.
- Do not launch unbounded parallel subagents on large reviews.
- Do not run `run_in_terminal` from within a subagent; collect all terminal output in the parent turn before dispatching any subagent.
- Do not issue subagent calls across multiple turns in the same wave; all agents in a wave must launch in one tool-call batch.
- Do not exceed 2 concurrent subagents in normal mode or 3 in safety mode per wave.

## Agent discovery
Before selecting reviewers, read each `agents/*.agent.md` file in the workspace to build the candidate roster:

1. List all `*.agent.md` files in the `agents/` directory (relative to workspace root).
2. For each file, parse the YAML frontmatter to extract `name` and `description`.
3. Store the roster as a list of `{ name, description }` pairs.
4. Match each required review lens against the roster by scanning `description` for keywords:
   - **Code / QA lens**: correctness, maintainability, QA, test, edge case, pre-merge, review
   - **Architecture lens**: architecture, system design, scalability, reliability, principal, distributed
   - **Security lens**: security, auth, threat, vulnerability, trust boundary, cryptography
   - **UX / UI lens**: UX, accessibility, front-end, interaction, design, a11y
   - **DevOps / deployment lens**: CI/CD, deployment, infrastructure, pipeline, observability, rollout, runtime
   - **Performance lens**: performance, optimization, bottleneck, profiling, latency
5. When multiple agents match a lens, prefer the one with the most focused description.
6. When no agent matches a lens, fall back to the inline prompt pattern for that lens.

## Parent reviewer checklist
- [ ] Diff and changed files inspected directly
- [ ] Scale assessed (files, changed lines, complexity)
- [ ] Large-project safety mode enabled when needed
- [ ] Relevant surrounding code read
- [ ] `agents/` directory scanned and candidate roster built
- [ ] Specialist agents matched to required lenses by description
- [ ] Subagents invoked by name when a match was found; inline prompts used as fallback
- [ ] All terminal commands (git diff, stats, file reads) completed before any subagent is launched
- [ ] Subagents launched in parallel: all agents in one wave issued in a single tool-call batch
- [ ] Wave size capped at 2 (normal mode) or 3 (safety mode) concurrent subagents
- [ ] Each subagent prompt is self-contained with inline diff context (no agent issues terminal commands)
- [ ] Chunking and wave execution used for large diffs
- [ ] Subagent outputs formatted using `templates/subagent-output.md`
- [ ] Subagent reviewer name and AI designation included in each formatted output
- [ ] Duplicate findings merged
- [ ] Findings ordered by severity and impact
- [ ] Open questions separated from confirmed issues
- [ ] Final review written by the parent reviewer using `templates/reviewer-output.md`
- [ ] Final review includes AI reviewer identification and list of all agent reviewers
- [ ] Not copied verbatim from subagents

## Prompt patterns for specialist subagents
- Code and QA reviewer
   - "Review this change for correctness, maintainability, edge cases, performance, test quality, and release confidence. Return all material findings with evidence, plus open questions and test gaps. Do not classify merge gating; the parent reviewer will decide required fixes."
- Architecture-focused reviewer
   - "Review this change for architectural fit, scalability, reliability, and system design risk. Return all material findings with evidence, plus open questions and operability concerns. Do not classify merge gating."
- Security reviewer
   - "Review this change for security issues, trust boundary mistakes, unsafe data handling, auth/authz problems, dependency risk, and missing abuse-case coverage. Return all material findings with evidence and likely exploit/impact context. Do not classify merge gating."
- UX/UI reviewer
   - "Review this user-facing change for UX regressions, accessibility issues, interaction inconsistencies, responsive concerns, and unclear copy. Return all material findings and testing gaps. Do not classify merge gating."
- Additional QA specialist (optional)
   - "Review this change for test adequacy, regression coverage, release confidence, and missing validation for changed behavior. Return all material coverage gaps and release-risk findings with evidence. Do not classify merge gating."
- DevOps or deployment reviewer
   - "Review this change for CI/CD risk, deployment safety, infrastructure correctness, runtime configuration issues, observability gaps, and rollback hazards. Return all material operational findings with evidence and mitigation ideas. Do not classify merge gating."
- `Explore`
  - "Read the relevant modules around this diff and summarize the architectural context, call paths, and likely regression surfaces so the specialist reviewers can stay focused."

## Example requests
- "Review this PR using a code-and-QA reviewer plus the relevant specialists in parallel, then synthesize a final findings list."
- "Do a security-heavy review of these authentication changes and only report merge-blocking issues."
- "Review this front-end diff with a UX/UI and code-and-QA lens, then give me a concise final review."
- "Run a parallel review on this branch and tell me whether there are any correctness, deployment, or rollout risks before merge."