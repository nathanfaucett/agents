# Worked Examples

## Example 1: Security Review
Input:
- "Review this workflow for token and secret risks."

Expected output shape:
1. Findings sorted by severity.
2. Exact YAML-level fixes.
3. Verification checks for token permissions and secret usage.

## Example 2: Reusable Workflow Authoring
Input:
- "Create a reusable workflow for Python tests across 3.10-3.12 with caching."

Expected output shape:
1. Reusable workflow YAML.
2. Calling workflow usage snippet.
3. Verification checks for cache hits and matrix execution.

## Example 3: Migration Planning Edge Case
Input:
- "Migrate a Jenkins pipeline using custom shell wrappers and org secrets."

Expected output shape:
1. Assumptions and gaps requiring user confirmation.
2. Importer strategy with custom transformer notes.
3. Rollback and verification plan.
