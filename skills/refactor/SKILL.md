---
name: refactor
description: |
   Improves code structure, readability, and maintainability through
   behavior-preserving refactoring. Invoke when reorganizing code, improving
   naming clarity, or reducing complexity without changing external behavior.
---

## Summary
Use this skill to perform behavior-preserving refactors that improve structure, readability, and maintainability while keeping external behavior and public APIs stable unless explicitly requested otherwise.

## When to use

- The user asks to "refactor", "clean up", "simplify", or "improve" code (file(s), module, or package).
- The codebase shows duplication, unclear names, large functions, or brittle tests and the user requests targeted improvements.
- During code review when a reviewer requests non-functional structural improvements.
- You want to extract helpers, rename for clarity, or consolidate duplicated logic without changing behavior.

## When not to use

- When the user asks for new features or behavior changes.
- When public API changes are intended (use a feature/migration workflow instead).
- For speculative, large-scale redesigns without explicit approval or incremental planning.
- When the repository lacks basic tests and the user demands a sweeping, unvetted refactor.

## Inputs

- `scope`: file(s), directory, or module to refactor. If omitted, ask for scope.
- `goals` / `constraints`: style guides, naming conventions, performance constraints, API stability requirements.
- Tests/CI commands to verify behavior preservation (e.g., `pytest`, `go test`, `npm test`).
- Preferred commit granularity (single commit vs multiple small commits) and desired commit message template.
- Optional: files or patterns to exclude, examples of desired naming/style.

## Outputs

- A clear summary of proposed changes and rationale.
- A minimal patch or series of small commits (diff/patch) implementing the refactor.
- A list of modified files and why each was changed.
- Test and linter results demonstrating behavior preservation when available.
- Suggested commit messages and a concise changelog entry.

## Procedure

1. Ask one concise question to clarify scope and constraints if missing.
2. Run static analysis and tests (if available) to gather a safety baseline.
3. Identify small, high-impact refactor opportunities (single-responsibility splits, renames, duplication removal).
4. Propose changes and request approval if the user expects a review before edits.
5. Apply changes as small, logical commits (one concept per commit).
6. Re-run tests and linters; revert or iterate if failures occur.
7. Provide a concise changelog entry, suggested commit messages, and rollback notes when relevant.

## Best practices and constraints

- Preserve behavior: do not change external behavior or public APIs unless explicitly requested.
- Keep commits small and atomic to ease review and reversion.
- Avoid unrelated dependency upgrades or formatting-only large diffs in the same change.
- When tests are absent, ask the user before making large edits; prefer conservative mechanical changes.
- Improve or maintain test coverage when extracting or moving logic that requires new tests.

## Gotchas and common risks

- Large refactors can hide subtle behavior changes; keep diffs small and focused.
- Mixing dependency upgrades with refactors increases review and rollback risk.
- Broad rename operations can miss transitive call sites; use symbol-aware rename tools when possible.
- Performance-focused refactors can regress latency or memory; verify before and after on critical paths.

## Examples

- "Refactor the `auth` module to remove duplication and improve testability; keep all tests passing."
- "Rename `process_item` -> `processRecord` across `src/` and update callers; run tests and lint."

## Commit & PR Templates

- Commit title format: `refactor(<scope>): Short description` (e.g., `refactor(utils): extract normalizeDate helper`).
- PR description should include motivation, summary of changes, files touched, test/lint results, and a note confirming behavior preservation.

## Acceptance Criteria

- Existing tests pass and linters report no new errors.
- Public API/behavior remains unchanged unless requested otherwise.
- Changes are documented in the PR/commit message with clear rationale.

## Notes for Agents

- Ask a single clarifying question if scope or constraints are not provided.
- Prefer mechanical, reversible edits and avoid speculative redesigns.
- When appropriate, split refactors into a chain of small PRs rather than one large change.

## References

- Follow repository style guides if present; otherwise use common community conventions for the language.
