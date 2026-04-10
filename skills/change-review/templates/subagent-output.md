---
name: subagent-output-template
description: Template for formatting subagent review findings from specialist reviewers
---

## Review by {{agent_name}} (AI Reviewer)
**Focus**: {{review_lens}}  
**Reviewer Type**: AI-generated specialist review  
**Reviewed By**: {{agent_name}}

> All file paths are relative to the project root. Line numbers must be included for every finding.
> Include a concrete fix or mitigation when known; otherwise say `No confident fix proposed`.
> Potential blockers are advisory only. The parent reviewer decides merge gating.
> Use Nitpicks only for trivial cosmetic items with zero correctness, accessibility, or maintenance consequence.
> Use Open Questions for missing context that blocks confident assessment. Use Testing Gaps for missing or insufficient verification of changed behavior.

### Potential blockers
{{#if blockers}}
{{#each blockers}}
- **{{issue_title}}**
  - **File**: `{{file_path}}`
  - **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
  - **Why potentially blocking**: {{reason}}
  - **Suggested fix**: {{#if suggested_fix}}{{suggested_fix}}{{else}}No confident fix proposed.{{/if}}
{{/each}}
{{else}}
None.
{{/if}}

### Bugs
{{#if bugs}}
{{#each bugs}}
- **{{issue_title}}**
  - **File**: `{{file_path}}`
  - **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
  - **Impact**: {{impact_description}}
  - **Why it matters**: {{explanation}}
  - **Suggested fix**: {{#if suggested_fix}}{{suggested_fix}}{{else}}No confident fix proposed.{{/if}}
{{/each}}
{{else}}
None.
{{/if}}

### Breaking Changes
{{#if breaking_changes}}
{{#each breaking_changes}}
- **{{issue_title}}**
  - **File**: `{{file_path}}`
  - **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
  - **What breaks**: {{what_breaks}}
  - **Affected consumers**: {{affected_consumers}}
  - **Migration path**: {{migration_path}}
{{#if suggested_fix}}
  - **Suggested fix**: {{suggested_fix}}
{{/if}}
{{/each}}
{{else}}
None.
{{/if}}

### Suggestions
{{#if suggestions}}
{{#each suggestions}}
- **{{issue_title}}**
  - **File**: `{{file_path}}`
  - **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
  - **Why it matters**: {{explanation}}
  - **Suggested fix**: {{#if suggested_fix}}{{suggested_fix}}{{else}}No confident fix proposed.{{/if}}
{{/each}}
{{else}}
None.
{{/if}}

### Nitpicks
{{#if nitpicks}}
{{#each nitpicks}}
- **{{issue_title}}**
  - **File**: `{{file_path}}`
  - **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
  - **Note**: {{explanation}}
{{/each}}
{{else}}
None.
{{/if}}

### Open Questions
{{#if open_questions}}
{{#each open_questions}}
- **{{question}}**
{{#if confidence_impact}}
  - **Affects confidence for**: {{confidence_impact}}
{{/if}}
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