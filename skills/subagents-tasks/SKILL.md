---
name: subagents-tasks
description: |
   Converts input into an iterative plan file containing Pending/In progress/Done
   sections and next-subagent signals for subagents to consume and update. Invoke
   when preparing structured sub-agent plans from PRDs, issue lists, or conversations.
---

## Summary
Convert PRD/issue/todo/conversation into an Agents task breakdown markdown files with explicit sub-agent checkpoint workflow. The output file should be structured for iterative execution, with clear next steps and signals for sub-agents to pick up work in sequence.

## When to use
- You have a PRD, issue list, todo list, or raw discussion and need a structured, execution-ready plan.
- You want to make work decomposable into cycles.
- You need clear outputs for engineering, design, or PM handoff.

## Input options
- PRD text (goals, users, metrics, scope).
- GitHub issue list, Jira stories, or plain checklist.
- Todo list with tasks.
- Conversation notes (meeting, brainstorming, chat log).

## Output format
- Output file must be created as a filesystem artifact.
   - Default output directory: `/tmp/` unless the workspace contains a `.tasks` folder or the user explicitly requests a different output location. If a `.tasks` folder exists, write outputs into `.tasks/`. If the user specifies a path, use that instead.
   - File path pattern: `${descriptive-name}-${timestamp}.md` (agent should derive a concise descriptive name).
   - Use the agent filesystem action (e.g., `create_file`) to write the final markdown into the chosen directory using the pattern `directory/${descriptive-name}-${timestamp}.md`.
   - Ensure the target directory exists (create it if necessary) before writing.
   - Include the generated file path explicitly in the final summary.
- Separation: produce one plan folder named `${descriptive-name}` containing a `tasks.md` control file and individual `task{n}.md` files (required).
   - Output folder: create `${output_dir}/${descriptive-name}/`.
      - Default output directory: `/tmp/` unless the workspace contains a `.tasks` folder or the user explicitly requests a different output location. If a `.tasks` folder exists, prefer and use `.tasks/` inside the workspace.
   - Control file: `${descriptive-name}/tasks.md` — contains the master task list, overall idea, and each task's status (`Pending`, `In progress`, `Done`).
   - Task files: `${descriptive-name}/task1.md`, `${descriptive-name}/task2.md`, ... — each file contains full task context, acceptance criteria, dependencies, estimated effort, and an explicit `Next sub-agent` command.
   - The control `tasks.md` must reference each task file with workspace-relative paths and include `Sub-agent workflow` sections linking to the task files.
   - Use the agent filesystem action (`create_file`) to create the folder, `tasks.md`, and each `taskN.md`. Ensure directories exist before writing and include all created file paths in the final summary.
- The file must be **sub-agent checkpoint friendly**: include an ordered work queue and explicit `Next sub-agent` status markers.
- Example output skeleton:
   - `## Context`
   - `## Goal` + metrics
   - `## Hypothesis`
   - `## Experiments / increments`
   - `## Acceptance criteria`
   - `## Blockers/risks`
   - `## Next steps`
   - `## Sub-agent workflow` (`Pending`, `In progress`, `Done`, `Signals`)

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
