---
name: reviewer-output-template
description: Parent reviewer instructions — merge subagent partials into a complete json_review JSON object matching the review report schema
---

<!--
PARENT REVIEWER INSTRUCTIONS

You have received JSON partial objects from each specialist subagent. Your job is to merge
them into one valid json_review JSON object matching the review report schema.
Schema contract: references/json-review-schema.md

The assembled `json_review` is an internal working artifact. Do not print the raw JSON in the
final response. Use it to render the markdown report and draft review comments in Stage 4.

## Merge procedure

1. COLLECT pillar scores
   For each subagent partial, read pillar_scores. Each pillar with a non-null score is owned
   by that agent. If two agents scored the same pillar (e.g. maintainability by Architecture
   and Code/QA), average their scores.

2. COMPUTE CQS score and grade
   CQS pillars: security, reliability, performance, maintainability, readability.
   Score = average of all non-null CQS pillar scores.
   Grade from score: A ≥ 4.5 | B ≥ 3.5 | C ≥ 2.5 | D ≥ 1.5 | F < 1.5

3. COMPUTE OIS score and grade
   OIS pillars: deployment_risk, scope_integrity, requirement_alignment, dependency_risk,
                backward_compatibility, compliance_check, observability_coverage, data_integrity.
   Score = average of all non-null OIS pillar scores. Same grade scale.

4. AGGREGATE findings
   - findings.blockers from all subagents → json_review.blockers
     Map: type, title, description, why_blocks, resolution_required, file, lines, suggested_fix → directly.
   - findings.bugs + findings.breaking_changes from all subagents → json_review.action_items
     Map: type, title, description, reason, file, lines, suggested_fix directly.
     Preserve migration_path for breaking_changes.
   - findings.open_questions → action_items with type="question"
   - findings.suggestions from all subagents → json_review.recommendations
     Map: priority, category, title, description, file, lines, action_items directly.
   - findings.nitpicks → json_review.recommendations with priority="low", category="Style"
     Map: title directly; description = note; preserve file and lines; set action_items = [].
   - Preserve structured location metadata through synthesis. Do not rely on description prose to recover a target later.

5. SET status
   - "blocked"         → any blocker exists
   - "conditional"     → no blockers, but action_items exist and all can be resolved pre-merge
   - "needs_attention" → no blockers, action_items exist, or open questions exist
   - "approved"        → blockers=[], action_items=[], no open questions

6. COUNT issues
   - critical: blockers with type in [security_violation, data_loss_risk, correctness_bug]
   - high:     remaining blockers + bugs in action_items
   - medium:   breaking_changes and questions in action_items
   - low:      recommendations count

7. WRITE reviewer_notes
   2–4 sentences. Summarise what was reviewed, what specialist lenses were applied,
   any coverage gaps (skipped lenses), and the single most important concern or positive signal.

8. SET jira_requirements and compliance_evidence to null (populated by external integrations when available).

## Output format

First assemble the complete JSON object below as an internal artifact.
Then render only the Stage 4 markdown output for the user.
-->

```json
{
  "review_metadata": {
    "repo_name": "<group/project or null>",
    "mr_number": "<MR number string or null>"
  },

  "summary": {
    "status": "<approved|needs_attention|conditional|blocked|degraded>",
    "explanation": "<one sentence rationale for the overall status>",

    "code_quality": {
      "score": 0.0,
      "grade": "<A|B|C|D|F>",
      "status": "<approved|needs_attention|conditional|blocked>",
      "pillars": {
        "security":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "reliability":     { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "performance":     { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "maintainability": { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "readability":     { "score": null, "label": null, "is_blocker": false, "is_conditional": false }
      }
    },

    "operational_integrity": {
      "score": 0.0,
      "grade": "<A|B|C|D|F>",
      "status": "<approved|needs_attention|conditional|blocked>",
      "pillars": {
        "deployment_risk":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "scope_integrity":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "requirement_alignment":  { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "dependency_risk":        { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "backward_compatibility": { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "compliance_check":       { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "observability_coverage": { "score": null, "label": null, "is_blocker": false, "is_conditional": false },
        "data_integrity":         { "score": null, "label": null, "is_blocker": false, "is_conditional": false }
      }
    },

    "issues": {
      "critical": 0,
      "high": 0,
      "medium": 0,
      "low": 0
    }
  },

  "reviewer_notes": "<2-4 sentences: lenses applied, coverage gaps, key concern or positive signal>",

  "blockers": [
    {
      "type": "<correctness_bug|security_violation|data_loss_risk|broken_contract|missing_test|deployment_blocker|compliance_violation>",
      "title": "<one-line summary>",
      "description": "<detailed explanation>",
      "why_blocks": "<concrete risk or impact that prevents merge>",
      "resolution_required": "<exact steps or code change to resolve>",
      "file": "<path/relative/to/project/root>",
      "lines": "<L42 or L42-L56>",
      "suggested_fix": "<fix description or code block, or null>",
      "note_id": null
    }
  ],

  "action_items": [
    {
      "type": "<bug|breaking_change|test_gap|question|configuration|documentation>",
      "title": "<one-line summary or null>",
      "description": "<what must be done — one actionable sentence>",
      "reason": "<why this is required before or shortly after merge>",
      "file": "<path/relative/to/project/root or null>",
      "lines": "<L42 or L42-L56, or null>",
      "suggested_fix": "<fix description or code block, or null>",
      "migration_path": "<required for breaking_change, else null>",
      "note_id": null
    }
  ],

  "recommendations": [
    {
      "priority": "<high|medium|low>",
      "category": "<Maintainability|Performance|Security|Testing|Readability|Risk|Style|Other>",
      "title": "<one-line summary>",
      "description": "<explanation and concrete improvement>",
      "file": "<path/relative/to/project/root>",
      "lines": "<L42 or L42-L56>",
      "action_items": [ "<step 1>", "<step 2>" ]
    }
  ],

  "jira_requirements": null,
  "compliance_evidence": null
}
```

<!--
## STAGE 4 — Human-readable output

After assembling the JSON above internally, render the following markdown output.
This is what the user reads. Use values from the assembled json_review object.

### Formatting rules

- Final output must be markdown only. Do not print the raw `json_review` object.
- Use the internal `json_review` object to populate both the review report and the draft review comments.

- Lead with the overall status using a clear badge label:
    - BLOCKED  →  🚫 BLOCKED
    - conditional →  ⚠️ CONDITIONAL
    - needs_attention →  ⚠️ NEEDS ATTENTION
    - approved →  ✅ APPROVED
    - degraded →  🔶 DEGRADED
- Omit any section that has no entries (e.g. skip "Blockers" when blockers=[]).
- For each blocker include: title, file, lines, why_blocks, resolution_required, and suggested_fix when it materially clarifies the fix.
- For each action item include: type label, title when present, file, lines, description, reason, and migration_path when present.
- For each recommendation include: priority badge, category, title, file, lines, description, and action_items list.
- Pillar scores: show score and label side-by-side; mark is_blocker pillars with 🚫, is_conditional with ⚠️.
- After the review report, include a `Draft Review Comments` section.
- Draft one comment for each blocker and required action item. Include high-priority recommendations when they are concrete enough to be posted as review feedback.
- Prefer inline comments whenever structured `file` and `lines` metadata exists and the finding can be mapped to a changed diff line or hunk.
- Use a general MR comment only when the finding spans multiple files, is intentionally summary-level, or cannot be positioned reliably on the diff.
- Never recover target metadata by reparsing prose in `description` or `message`.
- Omit trivial or duplicate draft comments; combine closely related findings when a single comment is clearer.
- End with reviewer_notes verbatim.
- Keep the report skimmable — use bold for titles and inline code for file paths and line refs.

### Template

---

## Code Review — [repo_name if set, else "Local diff"]

**Status:** [badge from above]  
**Reviewers:** [list of specialist agents that ran]

---

### Scores

| Dimension | Score | Grade |
|-----------|------:|-------|
| Code Quality (CQS) | [code_quality.score] | [code_quality.grade] |
| Operational Integrity (OIS) | [operational_integrity.score] | [operational_integrity.grade] |

**CQS pillars**

| Pillar | Score | Label | Flags |
|--------|------:|-------|-------|
[one row per non-null CQS pillar: security, reliability, performance, maintainability, readability]
| [pillar] | [score] | [label] | [🚫 if is_blocker] [⚠️ if is_conditional] |

**OIS pillars**

| Pillar | Score | Label | Flags |
|--------|------:|-------|-------|
[one row per non-null OIS pillar]
| [pillar] | [score] | [label] | [🚫 if is_blocker] [⚠️ if is_conditional] |

**Issue counts:** [critical] critical · [high] high · [medium] medium · [low] low

---

### 🚫 Blockers — must fix before merge
[if blockers is empty, omit this section]

**[n]. [title]** `[type]`
> [description]

**Location:** `[file] [lines]`

**Why this blocks:** [why_blocks]  
**Resolution:** [resolution_required]
[optional: **Suggested fix:** [suggested_fix]]

---

### ⚠️ Required Changes
[if action_items is empty, omit this section]

- **[[type]] [title if present]** `[file lines if present]`  
  [description]  
  _[reason]_
[optional: Migration path: [migration_path]]

---

### 💡 Recommendations
[if recommendations is empty, omit this section]

**[title]** · [category] · [priority] · `[file] [lines]`  
[description]  
[action_items as a sub-bullet list if present]

---

### Reviewer Notes

[reviewer_notes verbatim]

---

### Draft Review Comments
[omit this section if there are no blockers, action_items, or comment-worthy high-priority recommendations]

**Draft 1 — [Blocker title or action item type]**
- **Severity:** [critical|high|medium|low]
- **Target:** [Inline comment at file/lines when structured location maps to the diff, otherwise General MR comment]
- **Comment:**
  [2-5 sentence draft comment that states the issue, why it matters, and the requested change]

[repeat for each drafted comment]

---
-->
