---
name: simplify
user-invocable: true
description: "Simplify provides a focused workflow to simplify and unify code by improving composability, clarifying module boundaries, and producing concise, easy-to-use APIs. Use when: you want to reduce duplication, improve reusability, or make modules simpler to understand and compose."
---

# Simplify

Simplify is a that helps engineers reduce complexity and unify code across a codebase. It focuses on extracting composable pieces, defining clearer module boundaries, and producing small, well-documented APIs that are easy to use and understand.

## What it produces

- A short diagnosis describing complexity hotspots and recommended refactors.
- A prioritized list of small, refactor candidates (extracts, consolidations, interface clarifications).
- Changes that implement safe, minimal refactors.
- Rationale and tests or type-checks to run to validate behavioral safety.

## Workflow (step-by-step)

1. Scope: accept a target scope from the user file, directory, module name, or repo (default to whole repo).
2. Discover: run lightweight analysis to find duplication, large functions, cross-module imports, and public-surface size.
3. Propose: generate 1–3 prioritized refactor proposals with rationale and risk level.
4. Apply: produce atomic, well-documented patches and tests
5. Verify: run or suggest running tests/type-checks and list follow-ups.

## Decision points & branching logic

- If duplication appears in >2 places and is self-contained → prefer `extract shared helper`.
- If a module has >N exported functions serving distinct subdomains → recommend splitting responsibilities into smaller modules.
- If a proposed change is potentially breaking (public signature change) → mark as `high risk` and only propose, not apply.
- If tests or type checks are missing for the touched area → prefer proposal + test scaffolding rather than automatic apply.

## Quality criteria / completion checks

- All existing tests pass (if present) and static type checks succeed after changes.
- Cyclomatic complexity for modified functions decreases or is split into smaller helpers.
- Public API surface is smaller or better documented with examples.
- Duplication measured by similar code blocks is reduced for targeted hotspots.
- Run tests and type-checks where available before applying changes?

## Expected outputs

- Human-readable summary of findings and prioritized refactor list.
- Concrete patch(es) and short instructions for verifying changes
- Add tests, update docs, and add migration notes.

## Gotchas & edge cases

- Behavior changes are the primary risk; always prefer tests or type checks before applying non-trivial refactors.
- Large cross-cutting refactors may need coordination with downstream consumers; label those proposals as `coordination required`.
- Language or framework-specific idioms (macros, generated code, complex preprocessor steps) may require bespoke handling; call out these files and avoid blind edits.

## Questions the skill will ask the user when scope is ambiguous

- Should I preserve public APIs exactly, or are breaking changes allowed with migration notes?
