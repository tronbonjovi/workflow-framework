---
name: update-roadmap
description: Use when adding, removing, or modifying milestones and tasks in an existing roadmap — handles structural changes like new milestones, new tasks, reordering phases, or cancelling work after direction changes
---

# Update Roadmap

Modify the roadmap structure after brainstorming revisions, scope changes, new features, or course corrections. Unlike the internal `update-task` logic (which handles individual status changes), this skill handles structural changes to the roadmap itself.

## When to Use

- Adding a new milestone
- Adding new tasks to an existing milestone
- Cancelling a milestone or set of tasks
- Reordering phases or changing parallelism groups
- After a brainstorming session that changes project direction
- Adjusting milestone scope (splitting, combining, renaming)

## Pre-Check

1. Verify `.claude/roadmap/` exists with ROADMAP.md, MILESTONE.md, and TASK.md. If not: "No roadmap found. Run `/setup-roadmap` first."
2. Read current state: ROADMAP.md, MILESTONE.md, TASK.md to understand existing structure.
3. If new drafts exist in `.claude/roadmap/drafts/` that haven't been referenced in ROADMAP.md `sources:`, mention them: "I see new drafts that aren't part of the current roadmap: <list>. Should I incorporate them?"

## Operations

Parse the user's request and determine which operation(s) are needed:

### Add Milestone

1. Confirm with the user:
   - Milestone name (lowercase, hyphenated)
   - Description
   - What it achieves
2. Create the milestone directory: `.claude/roadmap/<milestone-name>/`
3. Add milestone section to MILESTONE.md (status: pending, tasks: empty)
4. Add milestone to ROADMAP.md:
   - Append to `milestones:` list in frontmatter
   - Add row to milestone overview table
5. Ask: "Want to define tasks for this milestone now?" If yes, proceed to Add Tasks.

### Add Tasks

1. Confirm milestone the tasks belong to.
2. For each new task, write a full AI-executable contract:
   - Title, complexity (standard or complex)
   - Context, objective, instructions, tests, acceptance criteria, scope
   - Technical approach and risk notes (if complex)
   - Files to touch, parallel safety assessment
   - Dependencies on existing tasks
   - Phase placement
3. Create individual task files using the appropriate template from the plugin's root `templates/` directory.
4. **Naming convention:** `<milestone-name>-task<NNN>.md` — continue numbering from the highest existing task number in that milestone.
5. Update MILESTONE.md: add task IDs to the milestone's task list.
6. Update TASK.md: add task rows to the appropriate phase table with all columns (including Complexity and Parallel-Safe). If a new phase is needed, create it.
7. Update ROADMAP.md: update progress column for the affected milestone.

### Cancel Milestone

1. Confirm with the user: "Cancel milestone **<name>** and all its tasks? This sets status to `cancelled` — files are preserved."
2. Set all task files in the milestone directory to `status: cancelled` (using update-task cascade logic).
3. Update MILESTONE.md: set milestone status to `cancelled`.
4. Update TASK.md: set all task rows for this milestone to `cancelled`.
5. Update ROADMAP.md: update milestone status and progress.

### Cancel Tasks

1. Confirm which tasks to cancel.
2. For each task: use the update-task cascade logic, setting status to `cancelled`.

### Reorder Phases

1. Present current phase structure from TASK.md.
2. Discuss desired changes with the user.
3. Update TASK.md phase sections, moving task rows as needed.
4. Update `phase` field in affected individual task files.

### Override Milestone Status

1. Confirm the milestone and desired status override.
2. Set `status_override` in MILESTONE.md to the requested status.
3. Update ROADMAP.md milestone overview to reflect the override.
4. Inform user: "Milestone **<name>** status overridden to **<status>**. Derived status would be **<derived>**."

## Consistency Guarantee

After every operation, verify:
- Every task file in a milestone directory is listed in MILESTONE.md's task list
- Every task listed in MILESTONE.md appears in TASK.md
- ROADMAP.md milestone list matches MILESTONE.md sections
- Progress counts in ROADMAP.md match actual task statuses
- Task files follow naming convention: `<milestone-name>-task<NNN>.md`

If any inconsistency is found, fix it and inform the user.
