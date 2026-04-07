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

Use this skill when work must follow explicit spec artifacts with strict ordering, hard gates, and machine-readable status.

## Purpose

SDD has four ordered phases:

1. Specify: define requirements, stories, constraints, and success criteria.
2. Plan: map requirements to architecture and codebase.
3. Task: decompose into atomic executable tasks.
4. Implement: execute and validate against requirements.

## Core Operating Rules

1. Continue-forward by default: advance immediately when gates pass.
2. Stop only for an open phase questions file or a hard blocker.
3. Never pause for manual confirmation between successful deterministic steps.
4. On stop, set one exact `go_to` file path that unblocks progress.

## Required Repository Layout

Required at repository root:

- specs/
- status.yml

Each spec folder must be named as:

- specs/01_feature_name/
- specs/02_another_feature/

1. Prefix is two-digit zero-padded ordinal: 01, 02, 03, ...
2. Separator is underscore
3. Suffix is lowercase snake_case feature name

Each spec folder must contain:

- phase-status.yaml
- specification.md
- technical-plan.md
- tasks/
- implementation.md

### Root Orchestration Status

status.yml is the cross-spec orchestration index.

Rules:

1. File name is exactly `status.yml` at repo root.
2. YAML only.
3. Must include all currently active specs.
4. Must include all currently active tasks.
5. `active_tasks` supports one or more entries for parallel execution.

Required keys:

- active_specs
- active_tasks
- updated_at

Root `status.yml` schema:

```yaml
active_specs:
  - specs/01_feature_name
active_tasks:
  - spec: specs/01_feature_name
    task: tasks/02_api-task.md
    assigned_agent: code-qa-engineer
    state: in_progress
    mode: serial
  - spec: specs/02_another_feature
    task: tasks/04_validation-task.md
    assigned_agent: devops-engineer
    state: in_progress
    mode: parallel
updated_at: 2026-04-06T00:00:00Z
```

### Task Control and Audit Artifacts

Each tasks/ folder must contain:

- tasks-index.yaml (machine-readable task queue and state)
- tasks-audit-log.yaml (append-only audit log of all transitions)

tasks-index.yaml schema:

```yaml
current_task: 02_api-task.md
next_task: 03_validate-task.md
queue_order:
   - 01_setup-task.md
   - 02_api-task.md
   - 03_validate-task.md
tasks:
   01_setup-task.md:
      status: done
      assigned_agent: agent-name
      attempts: 1
      blocker_reason: null
      updated_at: 2026-04-06T00:00:00Z
   02_api-task.md:
      status: in_progress
      assigned_agent: agent-name
      attempts: 2
      blocker_reason: null
      updated_at: 2026-04-06T00:00:00Z
   03_validate-task.md:
      status: pending
      assigned_agent: agent-name
      attempts: 0
      blocker_reason: null
      updated_at: 2026-04-06T00:00:00Z
```

tasks-audit-log.yaml schema:

```yaml
- timestamp: 2026-04-06T00:00:00Z
   actor: agent-name
   task: 02_api-task.md
   from_state: pending
   to_state: in_progress
   artifact_changed: tasks-index.yaml
   note: "Started API task"
```

Per-phase questions files are optional and only created when unresolved questions exist:

- requirements.questions.md
- architecture.questions.md
- tasks.questions.md
- implementation.questions.md

## AGENTS.md Index Requirement

Do not create specs/README.md as the primary index.

AGENTS.md must provide the SDD project index with:

1. Pointer to specs/
2. Current active spec folder
3. Minimal navigation hints for agents

## phase-status.yaml Is Authoritative

phase-status.yaml is the authoritative phase state file.

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

Example phase-status.yaml:

```yaml
phase: specify
state: blocked
next: specify
go_to: requirements.questions.md
updated_at: 2026-04-06T00:00:00Z
```

## Hard Gate Rules

Progression from phase P to P+1 is allowed only when both are true:

1. The current phase artifact exists and is marked complete in phase-status.yaml.
2. The current phase questions file does not exist.

Questions behavior:

1. If a phase questions file exists, the phase is unresolved by definition.
2. Resolve questions by updating the current phase artifact.
3. Delete the phase questions file after resolution.
4. Update phase-status.yaml to complete for phase P, then advance.

If either gate condition fails, do not advance. Set:

- state: blocked
- next: current phase
- go_to: exact file path needed to unblock

Examples:

- go_to: requirements.questions.md
- go_to: specification.md
- go_to: phase-status.yaml

## Phase Artifacts And Completion Targets

Specify phase:

- Artifact: specification.md
- Questions file: requirements.questions.md
- Completion target: requirements, stories, constraints, success criteria defined with IDs

Plan phase:

- Artifact: technical-plan.md
- Questions file: architecture.questions.md
- Completion target: architecture, codebase mapping, and requirement-to-design trace defined

Task phase:

- Artifact: tasks/ (ordered task files)
- Questions file: tasks.questions.md
- Completion target: atomic executable tasks with requirement traceability

Implement phase:

- Artifact: implementation.md
- Questions file: implementation.questions.md
- Completion target: implementation evidence and validation against requirement IDs

## Task Decomposition Rules

Inside tasks/:

1. Files must be strictly ordered and zero-padded.
2. File pattern: NN_short_name-task.md where NN is 01, 02, 03, ...
3. No numbering gaps.

Examples:

- 01_setup-task.md
- 02_api-task.md

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

trace_to entries must reference IDs defined in specification.md.

### Task State Machine

Legal transitions:

- pending -> in_progress
- in_progress -> done
- in_progress -> blocked
- blocked -> pending (after unblock)

Illegal transitions must hard-refuse and set go_to to the exact file to fix.

Exactly one in_progress task is allowed per spec folder.
Cross-spec parallelism is allowed only through root status.yml active_tasks.

### Multi-Agent Coordination Safety

When multiple agents run concurrently, updates to shared orchestration files must be serialized.

Rules:

1. Shared files are `status.yml` at repo root and `AGENTS.md`.
2. Before writing a shared file, an agent must acquire a short lease in `status.yml` under:
   - lock.owner
   - lock.acquired_at
   - lock.expires_at
3. Only the lock owner may write shared files while the lease is valid.
4. If lease is expired, another agent may replace it and continue.
5. Per-spec files (`specs/NN_feature_name/**`) can be updated without the global lock unless they also modify shared files.

## Deterministic Execution Procedure

For active spec folder specs/NN_feature_name/:

1. Read root `status.yml` and ensure current spec is listed in `active_specs` when in progress.
2. Read phase-status.yaml.
3. Evaluate phase gate for phase-status.yaml phase.
4. If gate fails:
   Set `state: blocked`, `next: <current phase>`, and `go_to` to one exact unblock file.
   Stop.
5. If gate passes:
   Mark current phase complete if needed, advance to next phase, set `state: in_progress`, set `next`, and set `go_to` to the next artifact.
   Continue immediately.
6. Update `phase-status.yaml`, `status.yml`, and `updated_at` fields.

When next is done:

- phase: implement
- state: complete
- next: done
- go_to: implementation.md

### Sub-Agent Task Run Loop

1. Read tasks/tasks-index.yaml and select the next pending task by queue_order.
2. Mark the task in_progress in tasks-index.yaml and append a tasks-audit-log.yaml entry.
3. Invoke the assigned_agent (sub-agent) with the task file and current spec context.
4. Sub-agent must:
   - Attempt the task (max_attempts enforced)
   - Update status in both tasks-index.yaml and the task file
   - Validate acceptance_checks
   - Mark done or blocked
   - If blocked, set blocker_reason and go_to to the exact file to fix
   - Append tasks-audit-log.yaml entry for every transition
5. Within a single spec folder, only one task may be in_progress at a time; all others must be pending or done.
6. If all tasks are done and none blocked, Task phase is complete and Implement phase may start.
7. If any task is blocked, Task phase is blocked and go_to must point to the blocked task file.
8. If a task completes and another pending task exists with no open question/blocker, immediately start the next pending task.

## Validation Checks (Human Or Agent)

Run these checks before advancing phases:

1. Active spec folder matches specs/NN_name format.
2. Required fixed artifacts exist.
3. phase-status.yaml exists and parses as YAML.
4. phase/state/next values are in allowed enums.
5. Current phase completion is recorded in phase-status.yaml.
6. Current phase questions file does not exist.
7. If blocked, go_to points to exactly one existing file path needed to unblock.
8. tasks/ files are zero-padded, sequential, and gapless.
9. Every task file contains trace_to with at least one requirement/story ID.
10. Every trace_to ID resolves to an ID in specification.md.

11. tasks/tasks-index.yaml exists, parses as YAML, and matches task files.
12. For each spec folder, only one in_progress task at a time.
13. All task status values are legal (pending, in_progress, done, blocked).
14. All assigned_agent and next_agent fields are valid agent names listed in AGENTS.md.
15. tasks-audit-log.yaml exists and records all transitions.
16. Blocked task must include blocker_reason and go_to must target exact unblock file.
17. phase-status.yaml and tasks/tasks-index.yaml must agree on phase-level state.
18. Root status.yml exists, parses as YAML, and includes active_specs and active_tasks.
19. Every active_tasks entry points to an existing spec folder and task file.
20. Root status.yml active_tasks entries must agree with per-spec tasks/tasks-index.yaml in_progress states.
21. Parallel execution is legal only when each active_tasks entry references a distinct task path and distinct spec folder.

If any check fails, do not advance phase; set blocked status and deterministic go_to.

## Output Expectations For Agents

When executing this skill, agents should produce:

1. Deterministic status updates in phase-status.yaml
2. Phase artifact edits only in the active phase unless explicitly unblocked
3. Hard refusal to advance on gate failure with one go_to target
4. Traceable outputs linking tasks and implementation back to requirement IDs
5. All task state transitions and audit entries in tasks-audit-log.yaml
6. All queue and agent assignments in tasks-index.yaml
7. AGENTS.md must include:
   - active_spec: specs/NN_feature_name
   - active_phase
   - active_task
   - next_signal (e.g., run-task: specs/NN_feature_name/tasks/03_api-task.md)
   - This enables sub-agent orchestration and navigation
8. Deterministic root orchestration updates in status.yml for active_specs and active_tasks (including parallel tasks)


