# Context Efficiency Hardening — Design Spec

> Date: 2026-04-07
> Status: Draft
> Scope: claude-workflow plugin v0.3.0

## Problem

Every skill reads all three index files (ROADMAP.md, MILESTONE.md, TASK.md) in full on every invocation. As projects grow, these files grow linearly — a 40-task project forces skills to load completed work that's irrelevant to the current action. The Prior Work Brief in `work-task` also compounds by reading every completed sibling task contract.

At small scale (2-3 milestones, 5 tasks) this is invisible. At real scale (6-8 milestones, 30+ tasks) it wastes context on every skill call.

## Goal

Keep context costs proportional to *active work*, not total project history. No structural changes to the file layout. No new infrastructure. Changes are confined to skill instructions and one new archive mechanism.

---

## Change 1: Archive Completed Milestones

### What

When the `update-task` cascade logic marks a milestone `completed` or `cancelled`, it moves that milestone's entries out of the active index files into `ARCHIVE.md`.

### Mechanism

- Create `.claude/roadmap/ARCHIVE.md` (created on first archive, not at setup)
- When a milestone reaches terminal status (`completed` or `cancelled`):
  - Move its section from MILESTONE.md into ARCHIVE.md
  - Move its task rows from TASK.md into ARCHIVE.md
  - Update ROADMAP.md status column (the milestone row stays in ROADMAP.md as a one-line reference — it's already compact)
- The milestone directory and individual task files stay on disk untouched
- `ARCHIVE.md` is append-only — new completed milestones get appended

### What reads ARCHIVE.md

- `/status` — optionally, to show full project history (completed milestones as a summary line, not full detail)
- Nothing else. `work-task`, `build-roadmap`, `update-roadmap` never read it.

### Edge case

- If a cancelled milestone gets reactivated (`cancelled` → `pending`), its entries move back from ARCHIVE.md into the active indexes. This is handled by `update-roadmap` since reactivation is a structural change.

---

## Change 2: Scoped Reads Per Skill

### What

Each skill reads only what it needs instead of loading all three files in full.

### Skill-by-skill read scope

**`work-task`:**
- Read ROADMAP.md → identify the first `in_progress` milestone (or first `pending` if none are in-progress)
- Read only that milestone's section from MILESTONE.md
- Read only that milestone's task rows from TASK.md
- Skip completed/cancelled milestone sections entirely

**`status`:**
- Reads all three index files (this is the one skill that needs the full picture)
- Optionally reads ARCHIVE.md to show completed milestone summary lines

**`build-roadmap`:**
- Reads ROADMAP.md to check existing milestones
- Reads MILESTONE.md and TASK.md to understand current structure
- No change needed — this runs infrequently and needs the full picture to avoid conflicts

**`update-roadmap`:**
- No change — needs full picture for structural modifications

**`update-task`:**
- Reads only the specific task file and its parent milestone section
- Updates only the relevant rows in TASK.md and MILESTONE.md
- No change to cascade logic, just clarify that it doesn't need to read unrelated milestones

**`setup-roadmap`:**
- No change — doesn't read index files at all

---

## Change 3: Cap the Prior Work Brief

### What

`work-task` Step 5b generates a Prior Work Brief by reading completed sibling tasks. Currently it reads *all* completed siblings. At scale (milestone with 10+ tasks), this means loading 7-8 full task contracts just to generate a summary.

### Change

Read only the **last 3 completed sibling tasks** (by `updated` date in frontmatter). The subagent needs recent context and established patterns, not full history.

### In the skill instructions

Replace "Read each task file that has `status: completed` in its frontmatter" with "Read the 3 most recently completed task files in the milestone (by `updated` date). If fewer than 3 are completed, read all of them."

---

## Change 4: Remove Static Boilerplate from TASK.md

### What

TASK.md currently includes a 7-line status legend and a 4-line comment block explaining phases and parallel-safety. These are reference material that Claude doesn't need — it wrote the system.

### Change

- Remove the `## Status Legend` section from TASK.md
- Remove the comment block at the top of the TASK.md template (`<!-- Phase structure expresses... -->` etc.)
- Remove the comments from the MILESTONE.md template

### Impact

Saves ~11 lines per TASK.md read. Small per-call, but it's loaded on every skill invocation.

---

## Change 5: Trim Standard Task Contract Template

### What

The standard task contract template has 7 sections: Context, Objective, Instructions, Tests, Acceptance Criteria, Scope (in/out). For standard-complexity tasks, Context is redundant with the Prior Work Brief, and Scope rarely prevents real problems.

### Change

Standard template sections: **Objective, Instructions, Tests, Acceptance Criteria** (4 sections).

Complex template keeps all 7 sections: Context, Objective, Technical Approach, Instructions, Tests, Acceptance Criteria, Scope, Risk Notes.

### Impact

Standard contracts drop from ~45 content lines to ~30. Since most tasks are standard complexity, this saves ~15 lines per task contract loaded during dispatch.

---

## Error Handling (Included)

These guards get added as part of the scoped reads work:

**`work-task` Step 1:**
- If `.claude/roadmap/` doesn't exist → "No roadmap found. Run `/setup-roadmap` to get started."
- If index files exist but contain no milestones → "Roadmap initialized but empty. Run `/build-roadmap` to define milestones."
- If a task file referenced in TASK.md doesn't exist on disk → warn and skip it, don't crash

**`setup-roadmap` Pre-Check (already exists, minor tightening):**
- Current check is fine. No changes needed.

---

## What Does NOT Change

- File structure (`.claude/roadmap/` with milestone subdirectories)
- Workflow sequence (setup → build → work → status)
- Task contract format for complex tasks
- The cascade update logic in `update-task`
- How subagents are dispatched
- The session-start hook

---

## Testing Approach

Unit tests using mock roadmap files at different scales:

1. **Small project** (2 milestones, 3 tasks, 0 completed) — verify no archiving happens, skills work normally
2. **Medium project** (4 milestones, 12 tasks, 2 milestones completed) — verify archiving moves completed milestone entries, `work-task` only reads active milestones
3. **Large project** (8 milestones, 30 tasks, 6 milestones completed) — verify index files stay small, Prior Work Brief caps at 3 tasks
4. **Edge cases:**
   - All milestones completed (project done)
   - Reactivating a cancelled milestone (moves back from archive)
   - Task file missing from disk but referenced in TASK.md
   - Empty index files (no milestones defined)

Tests validate behavior through the skill instructions (what gets read, what gets skipped) rather than through code execution — these are markdown skill files, not code.

---

## Implementation Order

1. Archive mechanism in `update-task` + create ARCHIVE.md format
2. Scoped reads in `work-task`
3. Prior Work Brief cap in `work-task`
4. Remove boilerplate from TASK.md template and existing files
5. Trim standard task contract template
6. Error guards in `work-task`
7. Update `status` skill to show archived milestones as summary lines
