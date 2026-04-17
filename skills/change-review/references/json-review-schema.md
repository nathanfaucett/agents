# json_review Schema Reference

This document defines the `json_review` object produced by the change-review skill's
parent reviewer. This schema defines the internal structured artifact used to build the final
markdown review report and draft comments.

---

## Top-level object

```json
{
  "review_metadata": { ... },
  "summary": { ... },
  "reviewer_notes": "str",
  "blockers": [ ... ],
  "action_items": [ ... ],
  "recommendations": [ ... ],
  "jira_requirements": null,
  "compliance_evidence": null
}
```

---

## review_metadata (optional)

```json
{
  "repo_name": "group/project",
  "mr_number": "123"
}
```

Populate when the review targets a GitLab MR. Used to construct direct links in the rendered report.
Omit or set `null` for local diff or patch reviews.

---

## summary

```json
{
  "status": "approved",
  "explanation": "One-sentence rationale for the overall status.",
  "code_quality": { ... },
  "operational_integrity": { ... },
  "issues": {
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0
  }
}
```

### status values

| Value            | When to use |
|------------------|-------------|
| `"approved"`     | No blockers, no required action items remain |
| `"needs_attention"` | No hard blocker, but at least one required fix or open question remains |
| `"conditional"`  | Approvable if specific listed action items are addressed before merge |
| `"blocked"`      | One or more confirmed blockers prevent merge |
| `"degraded"`     | Review incomplete due to missing context or tool failure |

### issues counts

Count findings across all categories:
- `critical` — blockers with confirmed data-loss, security-boundary violation, or correctness break
- `high` — blockers and bugs without the above severity
- `medium` — action_items (non-blocker required fixes)
- `low` — recommendations and nitpicks

---

## code_quality (CQS — Code Quality Score)

```json
{
  "score": 3.8,
  "grade": "B",
  "status": "needs_attention",
  "pillars": {
    "security":        { "score": 4.0, "label": "Good",      "is_blocker": false, "is_conditional": false },
    "reliability":     { "score": 3.5, "label": "Adequate",  "is_blocker": false, "is_conditional": false },
    "performance":     { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false },
    "maintainability": { "score": 3.0, "label": "Fair",      "is_blocker": false, "is_conditional": true  },
    "readability":     { "score": 4.0, "label": "Good",      "is_blocker": false, "is_conditional": false }
  }
}
```

`score` = average of all scored CQS pillar scores. `grade` derived from score (see grade table below).
`status` mirrors the overall status unless this dimension alone justifies a different classification.

### CQS pillars

| Pillar | Scored by | What to assess |
|--------|-----------|----------------|
| `security` | Security specialist | Input validation, secrets, injection, auth boundaries, OWASP Top 10 |
| `reliability` | Code/QA specialist | Error handling, null safety, retry logic, test coverage of changed behavior |
| `performance` | Performance specialist | Hot-path complexity, unnecessary allocations, N+1 queries, caching correctness |
| `maintainability` | Architecture + Code/QA | Coupling, cohesion, naming clarity, dead code, duplication |
| `readability` | UX/Accessibility specialist | Cognitive load, comment quality, consistent style, API ergonomics |

---

## operational_integrity (OIS — Operational Integrity Score)

```json
{
  "score": 3.2,
  "grade": "C",
  "status": "needs_attention",
  "pillars": {
    "deployment_risk":        { "score": 3.0, "label": "Fair",    "is_blocker": false, "is_conditional": false },
    "scope_integrity":        { "score": 4.0, "label": "Good",    "is_blocker": false, "is_conditional": false },
    "requirement_alignment":  { "score": 3.5, "label": "Adequate","is_blocker": false, "is_conditional": false },
    "dependency_risk":        { "score": 2.5, "label": "Weak",    "is_blocker": false, "is_conditional": true  },
    "backward_compatibility": { "score": 4.0, "label": "Good",    "is_blocker": false, "is_conditional": false },
    "compliance_check":       { "score": 3.0, "label": "Fair",    "is_blocker": false, "is_conditional": false },
    "observability_coverage": { "score": 2.5, "label": "Weak",    "is_blocker": false, "is_conditional": true  },
    "data_integrity":         { "score": 3.5, "label": "Adequate","is_blocker": false, "is_conditional": false }
  }
}
```

`score` = average of all scored OIS pillar scores.

### OIS pillars

| Pillar | Scored by | What to assess |
|--------|-----------|----------------|
| `deployment_risk` | Performance specialist | Migration steps, feature flags, rollback safety, env config changes |
| `scope_integrity` | Architecture specialist | Change is scoped to stated ticket; no unrelated modifications |
| `requirement_alignment` | UX/Accessibility specialist | Implementation matches stated requirements and acceptance criteria |
| `dependency_risk` | Architecture specialist | New or changed deps: license, maintenance status, supply-chain risk |
| `backward_compatibility` | Architecture specialist | Public API / contract changes; migration path provided |
| `compliance_check` | Security specialist | Change management, audit trail, regulatory concerns |
| `observability_coverage` | Code/QA specialist | Logging, metrics, tracing on changed paths |
| `data_integrity` | Code/QA specialist | Data mutations, migrations, rollback safety for DB changes |

---

## Grade table

| Grade | Score range |
|-------|-------------|
| A     | 4.5 – 5.0   |
| B     | 3.5 – 4.4   |
| C     | 2.5 – 3.4   |
| D     | 1.5 – 2.4   |
| F     | 0.0 – 1.4   |

### Pillar score labels

| Score | Label     |
|-------|-----------|
| 5.0   | Excellent |
| 4.0   | Good      |
| 3.5   | Adequate  |
| 3.0   | Fair      |
| 2.5   | Weak      |
| 1.0   | Poor      |

Use the nearest label for intermediate scores.

### is_blocker / is_conditional

Set `is_blocker: true` when the pillar has a finding that blocks merge.
Set `is_conditional: true` when the pillar has a finding that is required but not an outright block.
Both default to `false`.

---

## blockers

Items here prevent merge. The renderer displays these as a distinct "MUST FIX" section.

```json
[
  {
    "type": "correctness_bug",
    "title": "Short one-line summary",
    "description": "Detailed explanation of the problem.",
    "why_blocks": "Explain the concrete risk or impact that prevents merge.",
    "resolution_required": "Exact steps or code change needed to resolve.",
    "note_id": null
  }
]
```

`note_id` is always `null` for skill-generated reviews (reserved for integrations that link findings to GitLab comment IDs).

### blocker type values

`correctness_bug` · `security_violation` · `data_loss_risk` · `broken_contract` ·
`missing_test` · `deployment_blocker` · `compliance_violation`

---

## action_items

Required changes that are not outright blockers. Rendered as a checklist when `status` is
`conditional`, `needs_attention`, or `blocked`.

```json
[
  {
    "type": "bug",
    "description": "What must be fixed — one actionable sentence.",
    "reason": "Why it is required before merge.",
    "note_id": null
  }
]
```

### action_item type values

`bug` · `breaking_change` · `test_gap` · `question` · `configuration` · `documentation`

Use `type: "question"` for open questions that must be answered before the reviewer can be
confident the change is safe to merge.

---

## recommendations

Non-blocking improvements. Rendered as an advisory section.

```json
[
  {
    "priority": "medium",
    "category": "Maintainability",
    "title": "Extract shared validation logic",
    "description": "The same null-check appears in three places.",
    "action_items": [
      "Create a shared validateInput() helper in src/utils/validate.ts",
      "Replace duplicated checks at src/auth/login.ts L42, src/api/users.ts L88, src/api/orders.ts L61"
    ]
  }
]
```

`priority`: `"high"` · `"medium"` · `"low"`

Use `category: "Risk"` with `priority: "high"` for residual risks that don't block merge but
deserve follow-up.

---

## jira_requirements / compliance_evidence

Both are always `null` in skill-generated reviews. They are populated by external integrations
when Jira and compliance data are available.

```json
"jira_requirements": null,
"compliance_evidence": null
```

---

## Subagent → pillar ownership

Each specialist agent scores only the pillars within its lens. The parent reviewer averages
all scored pillars to derive CQS and OIS totals.

| Specialist agent | CQS pillars                  | OIS pillars                                               |
|------------------|------------------------------|-----------------------------------------------------------|
| Security         | security                     | compliance_check                                          |
| Architecture     | maintainability              | scope_integrity, dependency_risk, backward_compatibility  |
| Performance      | performance                  | deployment_risk                                           |
| UX/Accessibility | readability                  | requirement_alignment                                     |
| Code/QA          | reliability, maintainability | observability_coverage, data_integrity                    |

When a specialist is not invoked, set that pillar's score to `null` and exclude it from the average.

---

## Minimal valid example

```json
{
  "review_metadata": null,
  "summary": {
    "status": "approved",
    "explanation": "No blockers or required changes found.",
    "code_quality": {
      "score": 4.5,
      "grade": "A",
      "status": "approved",
      "pillars": {
        "security":        { "score": 5.0, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "reliability":     { "score": 4.0, "label": "Good",      "is_blocker": false, "is_conditional": false },
        "performance":     { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "maintainability": { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "readability":     { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false }
      }
    },
    "operational_integrity": {
      "score": 4.5,
      "grade": "A",
      "status": "approved",
      "pillars": {
        "deployment_risk":        { "score": 5.0, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "scope_integrity":        { "score": 4.0, "label": "Good",      "is_blocker": false, "is_conditional": false },
        "requirement_alignment":  { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "dependency_risk":        { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "backward_compatibility": { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "compliance_check":       { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false },
        "observability_coverage": { "score": 4.0, "label": "Good",      "is_blocker": false, "is_conditional": false },
        "data_integrity":         { "score": 4.5, "label": "Excellent", "is_blocker": false, "is_conditional": false }
      }
    },
    "issues": { "critical": 0, "high": 0, "medium": 0, "low": 0 }
  },
  "reviewer_notes": "Straightforward bug fix with test coverage. No concerns.",
  "blockers": [],
  "action_items": [],
  "recommendations": [],
  "jira_requirements": null,
  "compliance_evidence": null
}
```
