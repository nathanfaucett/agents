---
name: subagent-output-template
description: Template for formatting subagent review findings from specialist reviewers
---

## Review by {{agent_name}} (AI Reviewer)
**Focus**: {{review_lens}}  
**Reviewer Type**: Automated AI agent  
**Reviewed By**: {{agent_name}}

> All file paths are relative to the project root. Line numbers must be included for every finding.

### Blockers
{{#if blockers}}
{{#each blockers}}
- **{{issue_title}}**
  - **File**: `{{file_path}}`
  - **Lines**: `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`
  - **Why blocking**: {{reason}}
  - **Suggestion**: {{suggested_fix}}
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
  - **Suggestion**: {{suggested_fix}}
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
{{/each}}
{{else}}
None.
{{/if}}

### Suggestions
{{#if suggestions}}
{{#each suggestions}}
- **{{issue_title}}** — `{{file_path}}` `L{{line_start}}{{#if line_end}}-L{{line_end}}{{/if}}`  
  {{explanation}}  
  _Suggestion_: {{suggested_fix}}
{{/each}}
{{else}}
None.
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