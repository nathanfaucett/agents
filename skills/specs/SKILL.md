---
name: specs
description: |
  Runs a deterministic Spec-Driven Development (SDD) workflow using fixed files,
   hard phase gates and explicit state transitions so humans and agents can execute work
   with end-to-end traceability. Invoke when defining, planning, tasking, and
  implementing a feature from spec artifacts under specs/.
---

# Spec-Driven Development (SDD)

## When to use

Use this skill when work must follow explicit spec artifacts with strict ordering, hard gates, and machine-readable status.

## When not to use

Do not use this skill when work is ad hoc, low-risk, or too small to justify strict phase gates and orchestration artifacts.

- The task is a small bug fix or routine maintenance with obvious scope.
- Requirements are too volatile to commit to written phase artifacts yet.
- The team needs rapid exploratory iteration rather than deterministic gated execution.
- The repository does not use the required `specs/` artifact structure.

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
4. Do not use explicit redirect pointers; flow is derived from phase state, dependencies, and queue order.
5. Never transition a phase or task to complete/done without recorded completion evidence in a required artifact.

## Required Repository Layout

Required at repository root:

- specs/
- AGENTS.md

Each spec folder must be named as:

- specs/01_feature_name/
- specs/02_another_feature/

1. Prefix is two-digit zero-padded ordinal: 01, 02, 03, ...
2. Separator is underscore
3. Suffix is lowercase snake_case feature name


Each spec folder must contain:

- status.yaml
- specification.md
- technical-plan.md
- tasks/
- implementation.md

### Specs Orchestration Status

specs/status.yaml is the cross-spec orchestration index.

Rules:

1. File name is exactly `status.yaml` under `specs/`.
2. YAML only.
3. Must include all currently active specs.
4. Must include all currently active tasks.
5. `active_tasks` supports one or more entries for parallel execution.

Required keys:

- active_specs
- active_tasks
- updated_at

`specs/status.yaml` schema:

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

- index.yaml (machine-readable task queue and authoritative runtime task state)
- audit.log (append-only audit log of all transitions)

`tasks/index.yaml` is authoritative for task runtime state. If task file metadata differs, `tasks/index.yaml` wins.

index.yaml schema:

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


`audit.log` schema:

```yaml
- timestamp: 2026-04-06T00:00:00Z
  actor: agent-name
  task: 02_api-task.md
  from_state: pending
  to_state: in_progress
  artifact_changed: index.yaml
  note: "Started API task"
```

Per-phase questions files are optional and only created when unresolved questions exist:

- requirements.questions.md
- architecture.questions.md
- tasks.questions.md
- implementation.questions.md

## AGENTS.md Index Requirement

Do not create specs/README.md as the primary index.

AGENTS.md is navigation-only for SDD and must include:

1. Pointer to specs/
2. Pointer to specs/status.yaml
3. Optional current active spec hint (non-authoritative)


## Per-Spec status.yaml Is Authoritative

status.yaml is the authoritative phase state file.

Rules:

1. YAML only
2. Minimal, programmatic, concise
3. No narrative text
4. Must allow immediate next-step decision

Required keys:

- phase
- state
- next
- updated_at

Allowed enums:

- phase: specify | plan | task | implement
- state: in_progress | blocked | complete
- next: specify | plan | task | implement | done


Example status.yaml:

```yaml
phase: specify
state: blocked
next: specify
updated_at: 2026-04-06T00:00:00Z
```

## Hard Gate Rules

Progression from phase P to P+1 is allowed only when all of these are true:

1. The current phase artifact exists and is marked complete in status.yaml.
2. The current phase questions file does not exist.
3. Completion evidence for phase P exists in the phase artifact and in `audit.log` metadata.

Questions behavior:

1. If a phase questions file exists, the phase is unresolved by definition.
2. Resolve questions by updating the current phase artifact.
3. Delete the phase questions file after resolution.
4. Update status.yaml to complete for phase P, then advance.

If any gate condition fails, do not advance. Set:

- state: blocked
- next: current phase

## Evidence-First Completion Requirement

All completion transitions are proof-gated. "Proof somewhere" is not optional and must be recorded in machine-readable form.

Phase completion evidence:

1. Before setting a phase to `complete`, add a `completion_evidence` block in the active phase artifact.
2. `completion_evidence` must include at least one concrete verifier entry (artifact path, command output summary, or requirement coverage reference).
3. If `completion_evidence` is missing or empty, hard-refuse the phase transition and set phase state to `blocked`.

Task completion evidence:

1. Before transitioning a task from `in_progress` to `done`, write evidence under `tasks/index.yaml` for that task entry.
2. Required per-task key:

```yaml
tasks:
  02_api-task.md:
    status: done
    completion_evidence:
      - type: test|artifact|diff|check
        ref: path-or-command
        summary: short human-readable proof
    updated_at: 2026-04-06T00:00:00Z
```

3. `completion_evidence` must contain at least one non-empty entry.
4. The matching `audit.log` transition to `done` must include an `evidence_ref` field that points to the same proof source.
5. If evidence is absent, transition to `blocked` (or keep `in_progress`) and record `blocker_reason: missing completion evidence`.

Hard refusal example for missing evidence:

```
ERROR: Completion proof required before transition: in_progress -> done for 02_api-task.md. Missing tasks/index.yaml.tasks[02_api-task.md].completion_evidence.
```

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
status: pending|in_progress|done|blocked|failed
depends_on: [] # list of task_ids
assigned_agent: agent-name
next_agent: agent-name
acceptance_checks:
  - "API returns 200 for valid input"
max_attempts: 3
timeout_minutes: 30
```

Task files define intent and traceability. Runtime state is tracked in `tasks/index.yaml`.

trace_to entries must reference IDs defined in specification.md.

### Task State Machine

Legal transitions:

- pending -> in_progress
- in_progress -> done
- in_progress -> blocked
- blocked -> pending (after unblock)
- blocked -> failed (when max_attempts reached)
- in_progress -> failed (when max_attempts reached)

Illegal transitions must hard-refuse. A "hard refusal" means the agent must output a clear error message stating the illegal transition, the attempted action, and the required valid states. Example:

```
ERROR: Illegal task state transition attempted: pending -> done. Only allowed transitions are: pending -> in_progress.
```

Exactly one in_progress task is allowed per spec folder.
Cross-spec parallelism is allowed only through specs/status.yaml active_tasks.

### Multi-Agent Coordination Safety

When multiple agents run concurrently, updates to shared orchestration files must be serialized.

Rules:

1. Shared orchestration file is `specs/status.yaml`.
2. Before writing it, an agent must acquire a short lease in `specs/status.yaml` under:
   - lock.owner
   - lock.acquired_at
   - lock.expires_at
   - lock.fencing_token
   - Example lease duration: 10 minutes
3. Only the lock owner may write `specs/status.yaml` while the lease is valid.
4. If lease is expired, another agent may replace it and continue.
5. Per-spec files (`specs/NN_feature_name/**`) can be updated without the global lock unless they also modify `specs/status.yaml`.

## Deterministic Execution Procedure

For active spec folder specs/NN_feature_name/:

1. Read `specs/status.yaml` and ensure current spec is listed in `active_specs` when in progress.
2. Read status.yaml.
3. Evaluate phase gate for status.yaml phase.
4. If gate fails:
   Set `state: blocked` and `next: <current phase>`.
   Stop.
5. If gate passes:
   Mark current phase complete if needed, then advance to next phase and set `state: in_progress` with `next`.
   Continue immediately.
6. Update `status.yaml`, `specs/status.yaml`, and `updated_at` fields.

When next is done:

- phase: implement
- state: complete
- next: done

### Sub-Agent Task Run Loop

1. Read tasks/index.yaml and compute eligible pending tasks where all `depends_on` task_ids are `done`.
2. If multiple tasks are eligible, choose by queue_order. `depends_on` always overrides queue_order.
3. Mark the chosen task in_progress in tasks/index.yaml and append an audit.log entry.
4. Invoke the assigned_agent (sub-agent) with the task file and current spec context.
5. Sub-agent must:
   - Attempt the task (max_attempts enforced)
   - Update runtime status in tasks/index.yaml
   - Validate acceptance_checks
   - Collect and write completion_evidence before any done transition
   - Mark done, blocked, or failed (failed when attempts reach max_attempts)
   - If blocked or failed, set blocker_reason (including missing completion evidence)
   - Append audit.log entry for every transition
6. Within a single spec folder, only one task may be in_progress at a time; all others must be pending, done, blocked, or failed.
7. If all tasks are done and none are blocked or failed, Task phase is complete and Implement phase may start.
8. If any task is blocked or failed, Task phase is blocked.
9. If a task completes and another eligible pending task exists with no open question/blocker, immediately start the next eligible task.

## Validation Checks (Human Or Agent)

Run these checks before advancing phases:

1. Active spec folder matches specs/NN_name format.
2. Required fixed artifacts exist.
3. status.yaml exists and parses as YAML.
4. phase/state/next values are in allowed enums.
5. Current phase completion is recorded in status.yaml.
6. Current phase questions file does not exist.
7. If blocked, blocking reason is recorded in the relevant artifact.
8. tasks/ files are zero-padded, sequential, and gapless.
9. Every task file contains trace_to with at least one requirement/story ID.
10. Every trace_to ID resolves to an ID in specification.md.
11. tasks/index.yaml exists, parses as YAML, and matches task files.
12. For each spec folder, only one in_progress task at a time.
13. All task status values are legal (pending, in_progress, done, blocked, failed).
14. All assigned_agent and next_agent fields are valid agent names listed in AGENTS.md.
15. audit.log exists and records all transitions.
16. Blocked or failed task must include blocker_reason.
17. status.yaml and tasks/index.yaml must agree on phase-level state.
18. specs/status.yaml exists, parses as YAML, and includes active_specs and active_tasks.
19. Every active_tasks entry points to an existing spec folder and task file.
20. specs/status.yaml active_tasks entries must agree with per-spec tasks/index.yaml in_progress states.
21. Parallel execution is legal only when each active_tasks entry references a distinct task path and distinct spec folder.
22. Next runnable task selection always respects `depends_on` before queue_order.
23. A phase cannot transition to `complete` unless its artifact contains a non-empty `completion_evidence` block.
24. A task cannot transition to `done` unless `tasks/index.yaml` contains non-empty `completion_evidence` for that task.
25. Every `in_progress -> done` audit.log entry must include `evidence_ref` that resolves to a recorded completion evidence entry.
26. If completion evidence is missing, state must remain `in_progress` or move to `blocked` with `blocker_reason: missing completion evidence`.

If any check fails, do not advance phase; set blocked status deterministically.

## Hardening Recommendations

Apply these controls to improve safety and recovery in automated execution.

1. Use deterministic multi-file commit ordering.
   - For every transition, write per-spec state first, then global orchestration, then append audit.
   - If any write fails, stop and mark the current phase or task blocked with a blocker_reason.

2. Tighten lock lease semantics for `specs/status.yaml`.
   - Define lease duration, renewal cadence, and explicit stale-lock takeover conditions.
   - Include a fencing token on each successful lock acquisition to prevent split-brain writes.

3. Keep `tasks/index.yaml` schema strict.
   - Reject missing required keys for each task entry.
   - Treat unknown status values as invalid and block progression.

4. Enforce dependency graph validity before task execution.
   - Refuse execution if any `depends_on` references an unknown task_id.
   - Refuse execution if dependency cycles are detected.

5. Keep dependency-first scheduling deterministic.
   - Compute eligible tasks only from pending tasks whose dependencies are all done.
   - Use `queue_order` only as tie-breaker among eligible tasks.

6. Define terminal failure behavior.
   - On max_attempts exhaustion, transition to failed and require blocker_reason.
   - Do not auto-retry failed tasks; require explicit human or orchestrator reset.

7. Require idempotent task execution.
   - Tasks should be safe to re-run after partial failures.
   - Record side effects and recovery notes in implementation artifacts when relevant.

8. Strengthen timeout handling.
   - On timeout, record transition in audit.log with reason code timeout.
   - Increment attempts deterministically and apply normal blocked/failed rules.

9. Standardize recovery on restart.
   - Reconcile `status.yaml`, `tasks/index.yaml`, and `specs/status.yaml` before resuming work.
   - If reconciliation fails, block deterministically and require state repair before execution continues.

10. Add conformance fixtures for the state machine.
   - Maintain valid and invalid YAML examples for transitions, dependencies, and lock races.
   - Run these fixtures in CI to prevent regression in orchestration behavior.

## Output Expectations For Agents

When executing this skill, agents should produce:

1. Deterministic status updates in status.yaml
2. Phase artifact edits only in the active phase unless explicitly unblocked
3. Hard refusal to advance on gate failure (see above for example)
4. Traceable outputs linking tasks and implementation back to requirement IDs
5. All task state transitions and audit entries in audit.log
6. All queue and agent assignments in tasks/index.yaml (ensure both assigned_agent and next_agent are present)
7. AGENTS.md should remain navigation-only and point to specs/ and specs/status.yaml
8. Deterministic orchestration updates in specs/status.yaml for active_specs and active_tasks (including parallel tasks)
9. No phase/task completion transition without explicit completion_evidence and matching audit evidence_ref


