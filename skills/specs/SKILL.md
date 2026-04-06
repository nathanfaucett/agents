---
name: specs
description: |
  Runs a deterministic Spec-Driven Development (SDD) workflow using fixed files,
  hard phase gates, and explicit redirection so humans and agents can execute work
  with end-to-end traceability. Use when defining, planning, tasking, and
  implementing a feature from spec artifacts under specs/.
---

# Spec-Driven Development (SDD)

## When To Use

Use this skill when work must be executed from explicit specification artifacts with deterministic ordering, strict gate checks, and machine-friendly status tracking.

## Purpose

This skill defines a strict four-phase SDD workflow:

1. Specify
   Define requirements, user stories, constraints, and success criteria.
2. Plan
   Map requirements to technical architecture and existing codebase.
3. Task
   Decompose plan into atomic, actionable tasks for agents or developers.
4. Implement
   Generate code from specs and validate against requirements.

The workflow is gate-driven. Advancing to the next phase is blocked unless all gate conditions are satisfied.

## Required Repository Layout

Root directory:

- specs/

Each spec uses a strictly ordered, zero-padded folder name:

- specs/01_feature_name/
- specs/02_another_feature/

Folder naming is mandatory:

1. Prefix is two-digit zero-padded ordinal: 01, 02, 03, ...
2. Separator is underscore
3. Suffix is lowercase snake_case feature name

Each spec folder must contain these fixed artifacts:

- status.yaml
- spec.md
- plan.md
- tasks/
- implement.md

### Task Control and Audit Artifacts

Each tasks/ folder must contain:

- index.yaml (machine-readable task queue and state)
- audit.yaml (append-only audit log of all transitions)

index.yaml schema:

```yaml
current_task: 02_api.md
next_task: 03_validate.md
queue_order:
   - 01_setup.md
   - 02_api.md
   - 03_validate.md
tasks:
   01_setup.md:
      status: done
      assigned_agent: agent-name
      attempts: 1
      blocker_reason: null
      updated_at: 2026-04-06T00:00:00Z
   02_api.md:
      status: in_progress
      assigned_agent: agent-name
      attempts: 2
      blocker_reason: null
      updated_at: 2026-04-06T00:00:00Z
   03_validate.md:
      status: pending
      assigned_agent: agent-name
      attempts: 0
      blocker_reason: null
      updated_at: 2026-04-06T00:00:00Z
```

audit.yaml schema:

```yaml
- timestamp: 2026-04-06T00:00:00Z
   actor: agent-name
   task: 02_api.md
   from_state: pending
   to_state: in_progress
   artifact_changed: index.yaml
   note: "Started API task"
```

Per-phase question files are fixed and optional. Create only when unresolved questions exist:

- specify.questions.md
- plan.questions.md
- task.questions.md
- implement.questions.md

## AGENTS.md Index Requirement

Do not create specs/README.md as the primary index.

AGENTS.md must provide the SDD project index with:

1. Pointer to specs/
2. Current active spec folder
3. Minimal navigation hints for agents

## status.yaml Is Authoritative

status.yaml is the authoritative quick-reference state file used for phase decisions.

Rules:

1. YAML only
2. Minimal, programmatic, concise
3. No narrative text
4. Must allow immediate next-step decision

Required keys:

- phase
- state
- next
- go_to
- updated_at

Allowed enums:

- phase: specify | plan | task | implement
- state: in_progress | blocked | complete
- next: specify | plan | task | implement | done

go_to must be a deterministic, exact relative file path from the spec folder.

Example status.yaml:

```yaml
phase: specify
state: blocked
next: specify
go_to: specify.questions.md
updated_at: 2026-04-06T00:00:00Z
```

## Hard Gate Rules

Progression from current phase P to next phase is allowed only when BOTH are true:

1. The current phase artifact exists and is marked complete in status.yaml.
2. The current phase questions file does not exist.

Questions behavior is strict:

1. If a phase questions file exists, the phase is unresolved by definition.
2. Resolve questions by updating the current phase artifact.
3. Delete the phase questions file after resolution.
4. Update status.yaml to complete for phase P.
5. Only then may phase P+1 start.

If either gate condition fails, hard-refuse advancement and set deterministic redirect:

- go_to: <exact-file-path>

Examples:

- go_to: specify.questions.md
- go_to: spec.md
- go_to: status.yaml

## Phase Artifacts And Completion Targets

Specify phase:

- Artifact: spec.md
- Questions file: specify.questions.md
- Completion target: requirements, stories, constraints, success criteria defined with IDs

Plan phase:

- Artifact: plan.md
- Questions file: plan.questions.md
- Completion target: architecture, codebase mapping, and requirement-to-design trace defined

Task phase:

- Artifact: tasks/ (ordered task files)
- Questions file: task.questions.md
- Completion target: atomic executable tasks with requirement traceability

Implement phase:

- Artifact: implement.md
- Questions file: implement.questions.md
- Completion target: implementation evidence and validation against requirement IDs

## Task Decomposition Rules

Inside tasks/:

1. Files must be strictly ordered and zero-padded.
2. File pattern: NN_short_name.md where NN is 01, 02, 03, ...
3. No numbering gaps.

Examples:

- 01_setup.md
- 02_api.md

Each task file must include traceability to requirements/user stories via trace_to.

Required task header block:

```yaml
task_id: T-01
title: Setup feature scaffold
trace_to:
  - R-001
  - US-001
status: pending|in_progress|done|blocked
depends_on: [] # list of task_ids
assigned_agent: agent-name
next_agent: agent-name
acceptance_checks:
   - "API returns 200 for valid input"
max_attempts: 3
timeout_minutes: 30
```

trace_to entries must reference IDs defined in spec.md.

### Task State Machine

Legal transitions:

- pending -> in_progress
- in_progress -> done
- in_progress -> blocked
- blocked -> pending (after unblock)

Illegal transitions must hard-refuse and set go_to to the exact file to fix.

Exactly one in_progress task allowed at a time.

## Deterministic Execution Procedure

For active spec folder specs/NN_feature_name/:

1. Read status.yaml.
2. Evaluate phase gate for status.yaml phase.
3. If gate fails:
   Set state: blocked.
   Set next to current phase.
   Set go_to to one exact file that unblocks progress.
   Stop phase advancement.
4. If gate passes:
   Mark current phase complete (if not already).
   Advance phase to next.
   Set state: in_progress.
   Set next accordingly.
   Set go_to to next phase artifact.
5. Update updated_at.

When next is done:

- phase: implement
- state: complete
- next: done
- go_to: implement.md

### Sub-Agent Task Run Loop

1. Read tasks/index.yaml and select the next pending task by queue_order.
2. Mark the task in_progress in index.yaml and append an audit.yaml entry.
3. Invoke the assigned_agent (sub-agent) with the task file and current spec context.
4. Sub-agent must:
   - Attempt the task (max_attempts enforced)
   - Update status in both index.yaml and the task file
   - Validate acceptance_checks
   - Mark done or blocked
   - If blocked, set blocker_reason and go_to to the exact file to fix
   - Append audit.yaml entry for every transition
5. Only one task may be in_progress at a time; all others must be pending or done.
6. If all tasks are done and none blocked, Task phase is complete and Implement phase may start.
7. If any task is blocked, Task phase is blocked and go_to must point to the blocked task file.

## Validation Checks (Human Or Agent)

Run these checks before advancing phases:

1. Active spec folder matches specs/NN_name format.
2. Required fixed artifacts exist.
3. status.yaml exists and parses as YAML.
4. phase/state/next values are in allowed enums.
5. Current phase completion is recorded in status.yaml.
6. Current phase questions file does not exist.
7. If blocked, go_to points to exactly one existing file path needed to unblock.
8. tasks/ files are zero-padded, sequential, and gapless.
9. Every task file contains trace_to with at least one requirement/story ID.
10. Every trace_to ID resolves to an ID in spec.md.

11. tasks/index.yaml exists, parses as YAML, and matches task files.
12. Only one in_progress task at a time.
13. All task status values are legal (pending, in_progress, done, blocked).
14. All assigned_agent and next_agent fields are valid agent names.
15. audit.yaml exists and records all transitions.
16. Blocked task must include blocker_reason and go_to must target exact unblock file.
17. status.yaml and tasks/index.yaml must agree on phase-level state.

If any check fails, do not advance phase; set blocked status and deterministic go_to.

## Output Expectations For Agents

When executing this skill, agents should produce:

1. Deterministic status updates in status.yaml
2. Phase artifact edits only in the active phase unless explicitly unblocked
3. Hard refusal to advance on gate failure with one go_to target
4. Traceable outputs linking tasks and implementation back to requirement IDs
5. All task state transitions and audit entries in tasks/audit.yaml
6. All queue and agent assignments in tasks/index.yaml
7. AGENTS.md must include:
   - active_spec: specs/NN_feature_name
   - active_phase
   - active_task
   - next_signal (e.g., run-task: specs/NN_feature_name/tasks/03_api.md)
   - This enables sub-agent orchestration and navigation


