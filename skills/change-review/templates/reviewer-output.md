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

## Must-Change Before Merge
{{#if must_change_items}}
{{#each must_change_items}}
- **{{issue_title}}** {{file_path}}:{{line_range}}
  - **Why required**: {{required_reason}}
  - **Owner hint**: {{owner_hint}}
{{/each}}
{{else}}
None.
{{/if}}

---

## Critical Findings
{{#if critical_findings}}
{{#each critical_findings}}
### {{severity}}: {{issue_title}}
- **Location**: {{file_path}}:{{line_range}}
- **Impact**: {{impact_level}} — {{impact_description}}
- **Why it matters**: {{explanation}}
- **Suggested fix**: {{suggested_fix}}
- **Identified by**: {{agent_name}}
{{/each}}
{{else}}
None.
{{/if}}

---

## All Findings (by severity)
{{#each findings_by_severity}}
### {{severity_level}} ({{count}} finding{{#if_not_one}}s{{/if_not_one}})
{{#each findings}}
- **{{issue_title}}** {{file_path}}:{{line_range}}  
  {{explanation}}
{{/each}}
{{/each}}

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
**Lines Changed**: {{lines_added}} added, {{lines_deleted}} deleted  

{{change_summary_description}}

---

## Test Evidence
**Tests Run**: {{test_status}}  
**Coverage**: {{coverage_level}}  
{{test_notes}}

---

_Review conducted entirely by AI. For questions or disputes, escalate to a human reviewer._