---
name: subagents-tasks
description: Turn input into an iterative plan file that includes Pending/In progress/Done sections + next-subagent signals; subagents can consume, execute, and update in sequence.
---

## Summary
Convert PRD/issue/todo/conversation into an Agents Loop markdown with explicit sub-agent checkpoint workflow. The output file should be structured for iterative execution, with clear next steps and signals for sub-agents to pick up work in sequence.

## When to use
- You have a PRD, issue list, todo list, or raw discussion and need a structured, execution-ready plan.
- You want to make work decomposable into experiment/loop cycles (Ralph Loop).
- You need clear outputs for engineering, design, or PM handoff.

## Input options
- PRD text (goals, users, metrics, scope).
- GitHub issue list, Jira stories, or plain checklist.
- Todo list with tasks.
- Conversation notes (meeting, brainstorming, chat log).

## Output format
- Markdown file only (required).
- File path pattern: `${descriptive-name}-${timestamp}.md` (agent should derive a concise descriptive name).
- Output file must be created as a filesystem artifact.
 - Default output directory: `/tmp/` unless the workspace contains a `.tasks` folder or the user explicitly requests a different output location. If a `.tasks` folder exists, write outputs into `.tasks/`. If the user specifies a path, use that instead.
 - Markdown file only (required).
 - File path pattern: `${descriptive-name}-${timestamp}.md` (agent should derive a concise descriptive name).
 - Output file must be created as a filesystem artifact.
   - Use the agent filesystem action (e.g., `create_file`) to write the final markdown into the chosen directory using the pattern `directory/${descriptive-name}-${timestamp}.md` (default `/tmp/`).
   - Ensure the target directory exists (create it if necessary) before writing.
   - Include the generated file path explicitly in the final summary.
- The file must be **sub-agent checkpoint friendly**: include an ordered work queue and explicit `Next sub-agent` status markers.
- Example output skeleton:
  - `## Context`
  - `## Goal` + metrics
  - `## Loop hypothesis`
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
4. Derive candidate Ralph Loop blocks
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
   - `## Loop hypothesis`
   - `## Experiments / increments`
   - `## Acceptance criteria`
   - `## Blockers/risks`
   - `## Next steps`
   - `## Sub-agent workflow`
     - `### Pending` (next work item)
     - `### In progress` (current work item, if any)
     - `### Done` (completed work items)
     - `### Signals` (ready for next sub-agent)
   - Add explicit candidate command text suitable for downstream sub-agents, e.g., "Execute item #2, then update this section." 
7. Validation checklist
   - [ ] Problem + outcomes are clear
   - [ ] Success criteria measurable
   - [ ] Scope is clearly surfaced
   - [ ] Tasks actionable and upper bound ready
   - [ ] No missing dependencies
   - [ ] Sub-agent state transitions are defined in the file

## Decision points / branching logic
- If input is PRD/issue/todo, prefer structured extraction and minimal clarifying questions.
- If input is conversation, ask clarifying questions to confirm:
  - objective, KPI, target audience, deliverables.
- If success criteria are missing, generate candidate metrics and ask for confirmation.

## Quality criteria
- Execution-ready with clear handoff artifacts.
- Includes at least one loop/hypothesis per feature area.
- Contains explicit acceptance criteria and test signals.
- Presents one-liner update for standups.

## Prompt examples
- "Convert this PRD into an iterative ready plan with hypothesis, experiments, and a next-step checklist."
- "I have this issue list: [...]; convert to a single tasks file."
- "From this meeting transcript, draft an iterative ready plan and identify missing info."
