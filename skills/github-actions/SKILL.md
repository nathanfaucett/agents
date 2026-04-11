---
name: github-actions
description: |
  This skill provides best practices and guidance for GitHub Actions workflow design, review, and troubleshooting. Invoke when a user needs correct, secure, and maintainable CI/CD workflow recommendations.
---

# GitHub Actions

## Summary
This skill provides practical guidance for designing, reviewing, and troubleshooting GitHub Actions workflows using the local reference corpus in `references/`.

## Objective
Produce correct, secure, and maintainable GitHub Actions guidance with explicit references, concrete fixes, and verifiable output.

## Use When
- You need to author or review workflow YAML.
- You need to choose correct events, expressions, and contexts.
- You need to configure caching, environments, runners, or secrets.
- You need OIDC and secure-use recommendations for CI/CD hardening.
- You need to understand platform limits or Actions Importer behavior.

## Do Not Use When
- The task is unrelated to GitHub Actions workflow design, review, or troubleshooting.
- The user asks for runtime infrastructure changes that cannot be inferred from workflow artifacts.
- The request requires executing external systems rather than producing workflow guidance.

## Required Inputs
Capture or infer these before answering:
- Outcome: what the workflow or review must accomplish.
- Scope: single workflow file, reusable workflow, composite action, or migration set.
- Output contract: patched YAML, review findings, migration plan, or troubleshooting steps.
- Constraints: security, runner type, cost limits, deployment policies, and required checks.

If key details are missing, ask the minimum needed for a usable v1 answer.

## Reference-Driven Workflow
Use this sequence on every run:
1. Identify the task type: authoring, review, debugging, migration, or hardening.
2. Start from the category index files to find the canonical reference page.
3. Read the focused page for the exact topic.
4. Produce output using the output contract template in this skill.
5. Run verification checks before finalizing.

## Reference Indexes
Start from these files:
- `references/index.md`
- `references/workflows-and-actions/index.md`
- `references/runners/index.md`
- `references/security/index.md`
- `references/github-actions-importer/index.md`

## Reference Outline

### Workflows And Actions
- Author or review workflow YAML.
- Choose correct events, expressions, and contexts.
- Configure caching, environments, runners, and secrets.
- Apply OIDC and secure-use best practices.
- Understand platform limits and Actions Importer behavior.
- Workflow syntax: `references/workflows-and-actions/workflow-syntax.md`
- Trigger events: `references/workflows-and-actions/events-that-trigger-workflows.md`
- Workflow commands: `references/workflows-and-actions/workflow-commands.md`
- Variables: `references/workflows-and-actions/variables.md`
- Expressions: `references/workflows-and-actions/expressions.md`
- Contexts: `references/workflows-and-actions/contexts.md`
- Deployments and environments: `references/workflows-and-actions/deployments-and-environments.md`
- Dependency caching: `references/workflows-and-actions/dependency-caching.md`
- Reusable workflow configs: `references/workflows-and-actions/reusing-workflow-configurations.md`
- Action metadata syntax: `references/workflows-and-actions/metadata-syntax.md`
- Workflow cancellation behavior: `references/workflows-and-actions/workflow-cancellation.md`
- Dockerfile support: `references/workflows-and-actions/dockerfile-support.md`

### Runners
- GitHub-hosted runners: `references/runners/github-hosted-runners.md`
- Larger runners: `references/runners/larger-runners.md`
- Self-hosted runners: `references/runners/self-hosted-runners.md`

### Security
- Secure use reference: `references/security/secure-use.md`
- Secrets reference: `references/security/secrets.md`
- OIDC reference: `references/security/oidc.md`

### Limits
- Actions limits: `references/limits.md`

### GitHub Actions Importer
- Importer reference: `references/github-actions-importer/index.md`
- Supplemental arguments and settings: `references/github-actions-importer/supplemental-arguments-and-settings.md`
- Custom transformers: `references/github-actions-importer/custom-transformers.md`

## Guidance Priorities
When producing output, prioritize this order:
1. Correctness against workflow syntax and event behavior.
2. Security posture (secrets handling, token permissions, OIDC, untrusted inputs).
3. Runner compatibility and cost/performance tradeoffs.
4. Reliability (caching strategy, cancellation controls, retries, concurrency).
5. Maintainability (reusable workflows, clear naming, minimal duplication).

## Output Contract
Choose one format and follow it strictly.

### A) Workflow Authoring Or Refactor
1. Goal summary (1-2 lines).
2. Updated YAML snippet or full workflow.
3. Rationale mapped to Guidance Priorities.
4. Verification checklist with pass/fail criteria.

### B) Workflow Review
1. Findings ordered by severity.
2. Each finding includes:
- Location (`file`, `job`, `step`, or expression context).
- Risk.
- Exact fix.
3. Residual risks and assumptions.
4. Verification checklist.

### C) Troubleshooting
1. Symptom summary.
2. Most likely causes (ranked).
3. Minimal fix sequence.
4. Validation steps and expected signals.

### D) Migration (Actions Importer)
1. Source-to-target mapping assumptions.
2. Migration plan with phases.
3. Required transformers or supplemental args.
4. Validation and rollback checks.

## Routing Examples
Positive examples:
- "Review this workflow for security and cancellation behavior."
- "Write a reusable workflow for Node CI with caching and matrix builds."
- "Why is this workflow not triggering on pull_request_target?"

Negative examples:
- "Design our full Kubernetes cluster topology."
- "Debug a production outage unrelated to CI pipelines."
- "Write Terraform for VPC networking with no Actions context."

## Verification Checklist
- Confirm event triggers match the requested behavior.
- Confirm permission scope is least-privilege for `GITHUB_TOKEN`.
- Confirm secret handling and untrusted input boundaries are safe.
- Confirm runner selection is compatible with required tooling.
- Confirm caching keys and restore logic are deterministic.
- Confirm concurrency/cancellation behavior matches intent.
- Confirm environment and deployment protection rules are respected.
- Confirm all recommendations cite at least one relevant reference file.

## Prompt Formatting Checklist
- [ ] Objective is explicit and testable.
- [ ] Required rules appear before optional preferences.
- [ ] Output contract format is selected and followed exactly.
- [ ] At least one positive and one negative routing example are included.
- [ ] Guidance priorities are applied in the required order.
- [ ] Validation steps are included before final output.

## Worked Examples
### Example 1: Security Review
Input:
- "Review this workflow for token and secret risks."

Expected output shape:
1. Findings sorted by severity.
2. Exact YAML-level fixes.
3. Verification checks for token permissions and secret usage.

### Example 2: Reusable Workflow Authoring
Input:
- "Create a reusable workflow for Python tests across 3.10-3.12 with caching."

Expected output shape:
1. Reusable workflow YAML.
2. Calling workflow usage snippet.
3. Verification checks for cache hits and matrix execution.

### Example 3: Migration Planning Edge Case
Input:
- "Migrate a Jenkins pipeline using custom shell wrappers and org secrets."

Expected output shape:
1. Assumptions and gaps requiring user confirmation.
2. Importer strategy with custom transformer notes.
3. Rollback and verification plan.

## Gotchas
- `pull_request_target` can expose elevated permissions with untrusted code; apply strict guards.
- OIDC trust policies fail silently when audience/subject claims are mismatched.
- Cache misses often come from unstable keys or path mismatches.
- Self-hosted runners can drift from expected tooling versions.

## Guardrails
- Do not fabricate workflow behavior that conflicts with documented syntax.
- Do not recommend broad token permissions when narrower scopes are sufficient.
- Do not assume trusted inputs for fork-based events.
- Do not output partial fixes without validation steps.
