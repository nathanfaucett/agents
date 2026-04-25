---
name: simplify
user-invocable: true
description: "Simplify provides a focused workflow to simplify and unify code by improving composability, clarifying module boundaries, and producing concise, easy-to-use APIs. Use when: you want to reduce duplication, improve reusability, or make modules simpler to understand and compose."
---

# Simplify

Simplify is a multi-step workflow skill that helps engineers reduce complexity and unify code across a codebase. It focuses on extracting composable pieces, defining clearer module boundaries, and producing small, well-documented APIs that are easy to use and understand.

## Use when

- Duplicate or near-duplicate logic appears across files or modules.
- Modules expose large, awkward public surfaces that are hard to compose.
- Teams want to convert imperative blocks into small composable functions or classes.
- Onboarding is slowed by unclear module responsibilities or scattered helper code.
- You want a non-destructive plan to make code easier to understand and reuse.

## Don't use when

- The goal is to micro-optimize hot, performance-critical inner loops.
- You must preserve exact public APIs for backward-compatibility without planned migration.
- The code is intentionally experimental or a prototype where churn is expected.

## What it produces

- A short diagnosis describing complexity hotspots and recommended refactors.
- A prioritized list of small, reversible refactor candidates (extracts, consolidations, interface clarifications).
- Optional patch(es) (git-style diff) that implement safe, minimal refactors.
- Rationale and tests or type-checks to run to validate behavioral safety.

## Workflow (step-by-step)

1. Scope: accept a target scope from the user (file, directory, module name, or repo).
2. Discover: run lightweight analysis to find duplication, large functions, cross-module imports, and public-surface size.
3. Propose: generate 1–3 prioritized refactor proposals with rationale and risk level.
4. Decide: ask whether to `propose` only or `apply` patches.
5. Apply (optional): produce atomic, well-documented patches with small commits and suggested test runs.
6. Verify: run or suggest running tests/type-checks and list follow-ups.

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

## How to run

- Chat invocation: start a conversation and invoke the skill with a prompt that describes scope and constraints. Example: "Use the `simplify` skill on `src/components` to extract reusable hooks and reduce duplication; propose changes only."
- Parameters you should provide: `scope` (file/dir/module), `mode` (`propose` or `apply`), `constraints` (e.g., `non-breaking`, `keep-public-api`), `tests` (`run` or `skip`).
- Example quick commands (chat):
  - "/simplify scope=src/lib mode=propose constraints=non-breaking"
  - "/simplify scope=src/foo.js mode=apply constraints=non-breaking run-tests=true"

## Example prompts to try

- "Simplify the `auth` module: extract shared validation into a helper, reduce public surface, propose patches."
- "Unify duplicated parsing logic across `parser/*.py`; propose non-breaking refactors and test hints."
- "Review `ui/button` and make its API composable for consumers; include usage examples after refactor."

## Expected outputs

- Human-readable summary of findings and prioritized refactor list.
- For `propose` mode: one or more suggested diffs with clear commit messages and rationale.
- For `apply` mode: concrete patch(es) and short instructions for verifying changes (commands to run tests/type-checks).
- Suggested follow-up tasks (add tests, update docs, add migration notes).

## Gotchas & edge cases

- Behavior changes are the primary risk; always prefer tests or type checks before applying non-trivial refactors.
- Large cross-cutting refactors may need coordination with downstream consumers; label those proposals as `coordination required`.
- Language- or framework-specific idioms (macros, generated code, complex preprocessor steps) may require bespoke handling; call out these files and avoid blind edits.

## Suggested acceptance criteria for automated apply

- Mode = `apply` only when: tests exist and pass locally, or scope is small (single file) and changes are syntactic restructures (extract/rename) that preserve behavior.

## Related customizations / next steps

- Add a small prompt-level helper (`prompts/simplify.prompt.md`) to collect `scope`, `mode`, and `constraints` interactively.
- Pair with the existing `refactor` skill for broader refactors that intentionally break compatibility.
- Consider adding a CI job that runs `simplify --check` on PRs to surface complexity hotspots early.

## Questions the skill will ask the user when scope is ambiguous

- Do you want proposals only, or do you want me to apply patches?
- Should I preserve public APIs exactly, or are breaking changes allowed with migration notes?
- Should I run tests and type-checks where available before applying changes?

---

Generated by the workspace `simplify` skill template.  
Keep this SKILL.md concise; iterate as you use it.
