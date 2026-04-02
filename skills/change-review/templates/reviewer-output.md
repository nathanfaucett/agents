---
name: reviewer-output-template
description: Master template for the final synthesized code review from the parent reviewer
---

# Code Review: {{change_title}}
**Conducted By**: AI Review Agent (Multi-lens parallel review)  
**Review Date**: {{review_date}}  
**Reviewer**: Automated Code Review System  
**Reviewed Agents**: {{reviewer_list}}

---

## Verdict
**Status**: {{verdict}} (One of: Approve, Needs Changes, Questions, Block)

**Merge Readiness**: {{merge_readiness}} (Ready | Not Ready)

{{verdict_rationale}}

---

## Blockers
> Issues that must be resolved before this change can merge.

{{#if blockers}}
{{#each blockers}}
### {{issue_title}}
- **File**: `{{file_path}}`
- **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
- **Why blocking**: {{reason}}
- **Fix**: {{suggested_fix}}
- **Identified by**: {{agent_name}}
{{/each}}
{{else}}
None.
{{/if}}

---

## Bugs
> Defects causing incorrect or undefined behavior.

{{#if bugs}}
{{#each bugs}}
### {{issue_title}}
- **File**: `{{file_path}}`
- **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
- **Impact**: {{impact_description}}
- **Why it matters**: {{explanation}}
- **Fix**: {{suggested_fix}}
- **Identified by**: {{agent_name}}
{{/each}}
{{else}}
None.
{{/if}}

---

## Breaking Changes
> Changes that alter public contracts, APIs, or user-facing behavior in a backwards-incompatible way.

{{#if breaking_changes}}
{{#each breaking_changes}}
### {{issue_title}}
- **File**: `{{file_path}}`
- **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
- **What breaks**: {{what_breaks}}
- **Affected callers / consumers**: {{affected_consumers}}
- **Migration path**: {{migration_path}}
- **Identified by**: {{agent_name}}
{{/each}}
{{else}}
None.
{{/if}}

---

## Suggestions
> Non-blocking improvements: style, maintainability, performance, test coverage gaps, and minor UX polish.

{{#if suggestions}}
{{#each suggestions}}
- **{{issue_title}}** — `{{file_path}}` `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`  
  {{explanation}}  
  _Suggestion_: {{suggested_fix}}  
  _From_: {{agent_name}}
{{/each}}
{{else}}
None.
{{/if}}

---

## Coverage Summary
**Review Lenses Applied**:
{{#each applied_lenses}}
- ✓ {{lens_name}}: {{agent_name}}
{{/each}}

**Lenses Skipped**:
{{#each skipped_lenses}}
- — {{lens_name}}: {{reason}}
{{/each}}

---

## Open Questions
{{#if open_questions}}
{{#each open_questions}}
- **{{question}}**  
  _Affects confidence for_: {{confidence_impact}}
{{/each}}
{{else}}
None.
{{/if}}

---

## Residual Risks
{{#if residual_risks}}
{{#each residual_risks}}
- {{risk_description}}  
  _Severity_: {{risk_level}} | _Recommend_: {{mitigation}}
{{/each}}
{{else}}
None identified beyond findings above.
{{/if}}

---

## Change Summary
**Changed Files**: {{changed_file_count}} files  
**Lines Changed**: `{{lines_added}}` added, `{{lines_deleted}}` deleted  

{{change_summary_description}}

---

## Test Evidence
**Tests Run**: {{test_status}}  
**Coverage**: {{coverage_level}}  
{{test_notes}}

---

_Review conducted entirely by AI. For questions or disputes, escalate to a human reviewer._