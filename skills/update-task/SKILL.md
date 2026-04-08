---
name: update-task
description: Internal plumbing for cascade status updates — not user-invoked. Called by the orchestrator (work-task) and other skills to keep all index files in sync when task status changes.
---

# Update Task (Internal)

**This is internal plumbing, not a user-facing skill.** It is called by the `work-task` orchestrator and `update-roadmap` skill to maintain cross-file consistency when task statuses change. If the user asks Claude to change a task's status directly, Claude uses this logic but the user doesn't invoke it as a command.

## Cascade Update Logic

When a task status changes, update ALL of these files in order:

### 1. Individual task file
- Update `status` in frontmatter
- Update `updated` date to today
- If marking `blocked`: set `blockedBy` to the reason provided
- If moving out of `blocked`: clear `blockedBy` to null

### 2. TASK.md (battle plan)
- Find the task row in the phase table
- Update the Status column

### 3. MILESTONE.md
- Find the milestone section for this task
- Recalculate derived status using these rules:
  - All tasks `pending` → milestone `pending`
  - Any task `in_progress` → milestone `in_progress`
  - Any task `blocked` AND no tasks `in_progress` → milestone `blocked`
  - All tasks `completed` or `cancelled` → milestone `review`
  - If `status_override` is set (not null): keep the override, but note the derived status changed
- Update the milestone's status field

### 4. ROADMAP.md
- Find the milestone row in the overview table
- Update Status column to match milestone's current status
- Update Progress column (count completed+cancelled / total tasks)

## Status Transitions

Valid transitions:

| From | To |
|------|-----|
| `pending` | `in_progress`, `cancelled` |
| `in_progress` | `blocked`, `review`, `cancelled` |
| `blocked` | `in_progress`, `cancelled` |
| `review` | `completed`, `in_progress` (if review failed) |
| `completed` | (terminal — no transitions out) |
| `cancelled` | `pending` (if reactivated) |

If a requested transition is invalid, report the error to the calling skill.

## Task Location

Task ID format is `<milestone>-task<NNN>`. File lives at `.claude/roadmap/<milestone>/<task-id>.md`.

## Adding Notes

If adding a note (not changing status):
1. Read the task file
2. Append the note to the end of the file with today's date
3. Update `updated` date in frontmatter
4. No cascade needed — notes don't affect status
