---
name: work-task
description: Use when the user wants to work on tasks, check project status, or continue development — reads the task graph, presents current state, recommends next work, and dispatches subagents on user approval
---

# Work Task

## When to Use

- User says "what's next?", "work the next task", "let's keep going", "status?", "continue"
- Beginning of a session where a roadmap exists with pending tasks
- After completing a task, to present the next options

## Step 1: Assess State

Scope reads to the active milestone only — don't load the entire roadmap into context. **Do not read ROADMAP.md** — it is a planning/history document, not needed for execution.

### Guards

1. If `.claude/roadmap/` directory doesn't exist → stop and say: **"No roadmap found. Run `/setup-roadmap` to get started."**
2. Read `.claude/roadmap/MILESTONE.md`.
   - If no milestone sections exist → stop and say: **"Roadmap initialized but empty. Run `/build-roadmap` to define milestones."**

### Identify Active Milestone

3. From MILESTONE.md, find the **first milestone section** with status `in_progress`. If none are in-progress, use the **first milestone section** with status `pending`. Skip milestones with status `planned` (no tasks yet), `completed`, or `cancelled`. This is the **active milestone**.
4. Note the active milestone's name.

### Scoped Reads

5. From MILESTONE.md, read **only the active milestone's section**. Skip all other milestone sections.
6. Read `.claude/roadmap/TASK.md` — read **only task rows belonging to the active milestone**. Skip rows for other milestones.

### Determine

From the scoped data:
- Active milestone and its progress
- Tasks that are `completed`, `in_progress`, `blocked`
- Tasks that are `pending` with all dependencies met (ready to work)
- Phase ordering — what phase are we in?
- Parallel opportunities — which ready tasks can run simultaneously?
- Batching opportunities — which sequential simple tasks share context?

## Step 2: Present Status

Concise summary. Don't overwhelm — focus on what matters right now.

> "**Milestone: <name>** — <N>/<M> tasks done. Phase <X>.
> 
> Next up: `<task-id>` — <title> (<complexity>).
> [If parallel opportunity]: `<task-id>` can also run in parallel — different files, no dependency.
> [If batching opportunity]: `<task-id>` and `<task-id>` are sequential and simple — could batch to one subagent.
> [If blocked tasks]: `<task-id>` is blocked: <reason>.
>
> Want me to spin up a subagent for <task-id>? [Or: Want me to dispatch both in parallel?]"

If no tasks are ready (all blocked or all done):
- All done → "Milestone **<name>** is complete. <next milestone or roadmap status>"
- All blocked → "No unblocked tasks. Here's what's blocking: <list>. What do you want to do?"

## Step 3: Wait for User Approval

The user decides:
- Which task(s) to work on
- Whether to run in parallel or sequential
- Whether to batch
- Or to skip, reorder, discuss, or do something else entirely

**Never dispatch without explicit approval.**

## Step 4: Validate Contract

Before dispatching, verify each approved task contract is complete enough for a subagent to execute.

For each approved task file:

1. **Check file exists:** Verify the task contract file exists at `.claude/roadmap/<milestone>/<task-id>.md`.
   - If the file is missing: warn the user and skip the task:
     > "`<task-id>` contract file not found at expected path. Skipping — you may need to recreate it with `/update-roadmap`."
   - Continue with the next approved task if any remain. If no approved tasks remain, return to Step 2.
2. Read the task contract file
3. Verify the task contract meets the requirements defined in `/build-roadmap` (Instructions, Tests, AC, filesTouch — all substantive, no placeholders).

4. **If all fields are present and substantive:** proceed to Step 5 (Dispatch).

5. **If any field is missing or placeholder:** tell the user which fields are missing and offer: proceed anyway, fill gaps now, or skip the task. Wait for their choice.

## Step 5: Dispatch Subagents

For each approved task:

### 5a. Pre-dispatch
- Mark task `in_progress` (cascade update to TASK.md, MILESTONE.md)
- Read the full task contract file

### 5b. Dispatch
- **Generate Prior Work Brief:** Read up to 3 most-recently-completed task files in the same milestone (by `updated` date). For each, extract title, objective, `filesTouch`, and what was implemented (from contract — do NOT re-read source files). Format as:
     ```
     ## Prior Work Brief
     - **<task-id>: <title>** — <objective>. Files: <list>. Patterns: <if apparent>
     ```
  If no completed tasks exist, note "First task in milestone — no prior work."

- Spin up a subagent (using the Agent tool) with:
  - The full task contract content as the primary prompt
  - The **Prior Work Brief** (generated above) so the subagent knows what sibling tasks already built
  - Project context: what the project is, relevant architecture
  - Working directory
  - Instruction to follow TDD (tests first, then implementation)
  - Instruction to commit work when done
  - Instruction to report: what was implemented, what was tested, files changed, any concerns

### 5c. Parallel dispatch
- If multiple tasks approved for parallel execution:
  - Verify `parallelSafe: true` on all tasks
  - Verify no overlap in `filesTouch` arrays
  - If overlap detected: warn user, suggest sequential instead
  - If clear: dispatch all subagents in a single message (they run concurrently)

### 5d. Batch dispatch
- If sequential simple tasks approved for batching:
  - Combine all task contracts into one subagent prompt
  - Instruct: "Complete these tasks in order. Commit after each."
  - One subagent handles the batch

## Step 6: Handle Results

When a subagent completes:

### 6a. Mark for review
- Update task status to `review` (cascade update)

### 6b. Dispatch reviewer
- Spin up a reviewer subagent to check:
  - Did the implementation match the contract? (compliance)
  - Do tests pass?
  - Was anything built that wasn't requested? (scope creep)
  - Code quality check

### 6c. Review outcome
- **Pass:** Mark task `completed` (cascade update). Report to user.
- **Fail (cycle 1):** Dispatch fix subagent with reviewer's feedback. Re-review after fix.
- **Fail (cycle 2):** Mark task `blocked` with explanation. Inform user:
  > "`<task-id>` failed review twice. Issue: <summary>. What do you want to do?"

### 6d. Max 2 review cycles
After 2 failed reviews, the task is blocked and escalated to the user. Do not loop further.

## Step 7: Continue

After task(s) complete, return to Step 1 — assess the updated state and present next options. The user drives the pace.

**Sequential is the default.** Parallel only when the task graph supports it. Escalate to the user if a subagent is blocked, reports concerns, or creates merge conflicts.
