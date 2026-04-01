---
name: subagent-output-template
description: Template for formatting subagent review findings from specialist reviewers
---

## Review by {{agent_name}} (AI Reviewer)
**Focus**: {{review_lens}}  
**Reviewer Type**: Automated AI agent  
**Reviewed By**: {{agent_name}}

### Findings
{{#if findings}}
{{#each findings}}
- **[{{severity}}]** {{issue_title}}
  - **Location**: {{file_path}}:{{line_range}}
  - **Impact**: {{impact_description}}
  - **Why it matters**: {{explanation}}
  - **Suggestion**: {{suggested_fix}}
{{/each}}
{{else}}
No issues identified.
{{/if}}

### Open Questions
{{#if open_questions}}
{{#each open_questions}}
- {{question}}
{{/each}}
{{else}}
None.
{{/if}}

### Testing Gaps
{{#if testing_gaps}}
{{#each testing_gaps}}
- {{gap_description}}
{{/each}}
{{else}}
None identified.
{{/if}}

---