---
name: subagent-output-template
description: Template for specialist reviewer subagents — output a JSON partial consumed by the parent reviewer
---

<!--
SPECIALIST REVIEWER INSTRUCTIONS

You are one specialist lens in a parallel code review. Produce a single JSON object
following this template exactly. The parent reviewer merges all subagent partials.

Rules:
- Score ONLY the pillars listed under your lens in references/json-review-schema.md.
  Leave every other pillar as null/null/false/false.
- Every finding in blockers, bugs, breaking_changes, suggestions, and nitpicks MUST
  include a file path (project-root-relative) and a line number or range.
- suggested_fix: provide when you have a high-confidence recommendation.
  Include a fenced code block if the fix involves a source change.
  Use "No confident fix proposed" if uncertain — never invent a fix.
- is_blocker / is_conditional on pillar scores: advisory only.
  The parent reviewer decides merge gating.
- nitpicks: trivial cosmetic items only — zero correctness, accessibility, or
  maintenance consequence.
- open_questions: missing context that blocks confident assessment of a pillar.
-->

```json
{
  "agent_name": "<agent name, e.g. Security, Architecture, Performance, UX, Code/QA>",
  "review_lens": "<short description of the specialist focus>",

  "pillar_scores": {
    "security":               { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "reliability":            { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "performance":            { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "maintainability":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "readability":            { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "deployment_risk":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "scope_integrity":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "requirement_alignment":  { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "dependency_risk":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "backward_compatibility": { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "compliance_check":       { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "observability_coverage": { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
    "data_integrity":         { "score": null, "label": null, "is_blocker": false, "is_conditional": false }
  },

  "findings": {

    "blockers": [
      {
        "type": "<correctness_bug|security_violation|data_loss_risk|broken_contract|missing_test|deployment_blocker|compliance_violation>",
        "title": "<one-line summary>",
        "description": "<detailed explanation>",
        "why_blocks": "<concrete risk or impact that prevents merge>",
        "resolution_required": "<exact steps or code change to resolve>",
        "file": "<path/relative/to/project/root>",
        "lines": "<L42 or L42-L56>",
        "suggested_fix": "<fix description or code block, or 'No confident fix proposed'>"
      }
    ],

    "bugs": [
      {
        "type": "bug",
        "title": "<one-line summary>",
        "description": "<detailed explanation>",
        "reason": "<why this must be fixed>",
        "file": "<path/relative/to/project/root>",
        "lines": "<L42 or L42-L56>",
        "suggested_fix": "<fix description or code block, or 'No confident fix proposed'>"
      }
    ],

    "breaking_changes": [
      {
        "type": "breaking_change",
        "title": "<one-line summary>",
        "description": "<what breaks and for whom>",
        "reason": "<why this must be addressed before merge>",
        "file": "<path/relative/to/project/root>",
        "lines": "<L42 or L42-L56>",
        "migration_path": "<steps callers or consumers must take>",
        "suggested_fix": "<compatibility shim or 'No confident fix proposed'>"
      }
    ],

    "suggestions": [
      {
        "priority": "<high|medium|low>",
        "category": "<Maintainability|Performance|Security|Testing|Readability|Risk|Other>",
        "title": "<one-line summary>",
        "description": "<explanation and concrete improvement>",
        "file": "<path/relative/to/project/root>",
        "lines": "<L42 or L42-L56>",
        "action_items": [ "<step 1>", "<step 2>" ]
      }
    ],

    "nitpicks": [
      {
        "title": "<one-line cosmetic note>",
        "file": "<path/relative/to/project/root>",
        "lines": "<L42 or L42-L56>",
        "note": "<explanation>"
      }
    ],

    "open_questions": [
      {
        "question": "<what you need to know>",
        "confidence_impact": "<which pillar or finding this blocks>"
      }
    ]

  }
}
```