# GitHub Actions Skill - Overview

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
