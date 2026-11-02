# Output Contract

Choose one format and follow it strictly.

## A) Workflow Authoring Or Refactor
1. Goal summary (1-2 lines).
2. Updated YAML snippet or full workflow.
3. Rationale mapped to Guidance Priorities.
4. Verification checklist with pass/fail criteria.

## B) Workflow Review
1. Findings ordered by severity.
2. Each finding includes:
   - Location (`file`, `job`, `step`, or expression context).
   - Risk.
   - Exact fix.
3. Residual risks and assumptions.
4. Verification checklist.

## C) Troubleshooting
1. Symptom summary.
2. Most likely causes (ranked).
3. Minimal fix sequence.
4. Validation steps and expected signals.

## D) Migration (Actions Importer)
1. Source-to-target mapping assumptions.
2. Migration plan with phases.
3. Required transformers or supplemental args.
4. Validation and rollback checks.
