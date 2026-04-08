---
name: subagents-tasks
description: |
   Converts input into an iterative plan file containing Pending/In progress/Done
   sections and next-subagent signals for subagents to consume and update. Invoke
   when preparing structured sub-agent plans from PRDs, issue lists, or conversations.
---

## Summary
Convert PRD/issue/todo/conversation input into a sub-agent task breakdown with explicit checkpoint workflow. The output is a plan folder structured for iterative execution, with clear next steps and machine-friendly signals for downstream sub-agents.

## When to use
- You have a PRD, issue list, todo list, or raw discussion and need a structured, execution-ready plan.
- You want to make work decomposable into cycles.
- You need clear outputs for engineering, design, or PM handoff.

## When not to use
- You only need a quick ad hoc checklist with no sub-agent workflow.
- The input is too incomplete to infer goals, scope, or success criteria.
- You are trying to execute tasks immediately rather than prepare a plan (use `subagents-tasks-run`).

## Input options
- PRD text (goals, users, metrics, scope).
- GitHub issue list, Jira stories, or plain checklist.
- Todo list with tasks.
- Conversation notes (meeting, brainstorming, chat log).

## Output format
Use a folder-based structure as the canonical output model.

- Output directory decision:
   1. Use a user-provided path if specified.
   2. Otherwise, use `.tasks/` if it exists in the workspace.
   3. Otherwise, use `/tmp/`.
- Output folder: `${output_dir}/${descriptive-name}/`
- Required files:
   - `${descriptive-name}/tasks.md` control file
   - `${descriptive-name}/task1.md`, `${descriptive-name}/task2.md`, ...
- `tasks.md` must include:
   - Ordered work queue
   - `Sub-agent workflow` sections: `Pending`, `In progress`, `Done`, `Signals`
   - Workspace-relative links to each `taskN.md`
- Each `taskN.md` must include:
   - Full context and acceptance criteria
   - Dependencies and estimated effort
   - Explicit `Next sub-agent` command
- Use filesystem actions to create directories/files as needed and include all created file paths in the final summary.

## Step-by-step process
1. Identify source type
   - Ask: "Is this a PRD, issue list, todo list, or conversation?"
   - If ambiguous: request the most structured form available.
   - If user submits unstructured raw text: ask for the most concise, structured reframing (e.g., PRD/issue/todo format).
2. Extract top-level problem and success criteria
   - User problem, business objective, and measurable KPIs.
   - Constraints (timeline, budget, tech limits, compliance).
3. Normalize scope
   - Must-have vs nice-to-have.
   - Key user scenarios and personas.
4. Derive candidate blocks
   - Problem statement
   - Hypothesis (if unknown, propose)
   - Experiment(s) / increment(s)
   - Data/metrics for validation
   - Dependencies, blockers, risk mitigations
5. Split into action items
   - Concrete tasks (design/dev/test/launch)
   - Estimated effort (relative: small/medium/large)
   - If estimate is large, consider breaking down further or ask the user for more details to refine.
   - Each task is handled by a sub-agent (task-specific processing)
6. Build the `Sub Agent Iterative Ready` file
   - `## Context`
   - `## Goal` + metrics
   - `## Hypothesis`
   - `## Experiments / increments`
   - `## Acceptance criteria`
   - `## Blockers/risks`
   - `## Next steps`
    - Build artifacts inside `${descriptive-name}/`:
       1. `tasks.md` (control): ordered master work queue, global metadata, and the `Sub-agent workflow` with `Pending`, `In progress`, `Done`, and explicit links to each `taskN.md`.
       2. `taskN.md` files: each contains full context, dependencies, acceptance criteria, estimated effort, and an explicit `Next sub-agent` command instructing the downstream sub-agent how to execute and how to update both the task file and `tasks.md`.
    - `## Sub-agent workflow` (in `tasks.md`)
       - `### Pending` (list of `taskN.md` links)
       - `### In progress` (single current task entry with assignee/sub-agent and link to the `taskN.md`)
       - `### Done` (completed task entries with links to `taskN.md` and short status notes)
       - `### Signals` (machine-friendly flags or short commands for sub-agents; e.g., `run-task: ${descriptive-name}/task2.md`)
    - Each `taskN.md` must include an explicit candidate command text suitable for downstream sub-agents, e.g., "Execute step 2, run tests, then update `task2.md` and set `tasks.md` status to Done." 
7. Validation checklist
   - [ ] Problem + outcomes are clear
   - [ ] Success criteria measurable
   - [ ] Scope is clearly surfaced
   - [ ] Tasks actionable and upper bound ready
   - [ ] No missing dependencies
   - [ ] Sub-agent state transitions are defined in the file
    - [ ] Control file created and references all task files
    - [ ] Each task has a standalone markdown file with context and `Next sub-agent` command
    - [ ] All file paths included in final summary
    - [ ] `tasks.md` created inside `${descriptive-name}/` and references all `taskN.md` files
    - [ ] Each `taskN.md` is created inside `${descriptive-name}/` with context and `Next sub-agent` command
    - [ ] All created file paths included in final summary

## Decision points / branching logic
- If input is PRD/issue/todo, prefer structured extraction and minimal clarifying questions.
- If input is conversation, ask clarifying questions to confirm:
  - objective, KPI, target audience, deliverables.
- If success criteria are missing, generate candidate metrics and ask for confirmation.

## Quality criteria
- Execution-ready with clear handoff artifacts.
- Includes at least one hypothesis per feature area.
- Contains explicit acceptance criteria and test signals.
- Presents one-liner update for standups.

## Prompt examples
- "Convert this PRD into an iterative ready plan with hypothesis, experiments, and a next-step checklist."
- "I have this issue list: [...]; convert to a single tasks file."
- "From this meeting transcript, draft an iterative ready plan and identify missing info."
