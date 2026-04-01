---
name: change-review
description: >
  Structured, blocker-first AI change review that runs specialist reviewers in parallel and synthesizes a single, evidence-oriented final review.
---

## Summary
Structured AI-driven review that dispatches 2+ specialist subagents, prioritizes merge blockers, and returns a single synthesized verdict and prioritized findings. Long examples and scripts are moved to an appendix.

## When to use
Use for PRs/patches needing multi-lens coverage (correctness, security, architecture, UX, operability) or when parallel specialist review improves confidence before merge.

## When not to use
Avoid for trivial edits, single-line checks (compile-only), or when the task is to implement fixes rather than review them.

## Inputs
- Change target: commit range / PR / branch diff or list of changed files.
- Review scope: full / security-only / architecture-only / release-blocking.
- Context: purpose, linked issue, constraints, and any evidence (tests, screenshots, benchmarks).
- If missing, parent reviewer must gather minimal context before launching subagents.

## Outputs
A single synthesized review package containing:
- Verdict: `Approve` | `Needs changes` | `Question` | `Block` with short rationale.
- Prioritized findings with severity and merge impact (`must-change` or `non-blocking`).
- Explicit `must-change` list.
- Coverage summary (lenses applied / skipped).
- Open questions and residual risks.
- Secondary summary or positive notes.

## Blocker-first policy
Present confirmed merge blockers first (correctness, data-integrity, security boundary violations, broken runtime behavior, or critical test gaps). Downgrade pure style notes unless they materially affect maintenance or risk.

## Output formatting
- Subagents: use `templates/subagent-output.md`. They return findings and evidence but must not classify merge gating.
- Final review: use `templates/reviewer-output.md`. The parent reviewer assigns `must-change` labels.

## Execution constraints (VS Code / parallelism)
- Parent reviewer must pre-fetch and inline all diff context for subagents.
- Dispatch all subagents in one tool-call batch per wave.
- Cap concurrent subagents at 2 (normal) or 3 (safety).
- Subagents must not run terminal commands (`run_in_terminal`) or share mutable workspace state.

## Large-project safety mode
Trigger when >150 changed files or >10,000 changed lines. Triage with `--name-only`/`--stat`/`--numstat` to map hotspots, then deep-review in chunks (≤25 files or ≈2,500 lines) in waves. Report partial coverage explicitly.

## Parallel review workflow (high level)
1. Set review target (prefer merge-base diff).
2. Discover agents and pick lenses (always include code-and-QA).
3. Collect and inline chunked context.
4. Dispatch subagents in a single wave with embedded hunks.
5. Synthesize, dedupe, mark `must-change`, and produce final review.

## Agent discovery & lens mapping
Scan `agents/*.agent.md` for `name` and `description`. Map lenses by keywords: code/QA, security, devops, UX, performance, architecture. Prefer the most focused agent; fall back to inline prompts when needed.

## Prompt template
"Review this change for <lens> and return material findings with evidence and open questions; do not decide merge gating." Example lenses: security, UX, devops, performance, architecture.

## Parent reviewer checklist (essential)
- Inspect diff and assess scale.
- Prefetch context and choose lenses.
- Dispatch subagents in one batch.
- Synthesize findings; list `must-change` first.
- Format final output with the reviewer template.

## Guardrails & quality bar
- Do not expose secrets or credentials.
- Do not paste massive raw diffs; chunk instead.
- Do not run terminal commands inside subagents.
- Favor evidence from code/tests; downgrade pure style nits.
- If confidence is low, convert issues into open questions.

## Examples & appendix
Move long examples, extended checklists, and scripts to `EXAMPLES.md` or `SKILL.md.bak`.


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