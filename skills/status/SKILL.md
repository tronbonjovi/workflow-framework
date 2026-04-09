---
name: status
description: Use when the user wants to check project status, see roadmap progress, or get an overview of milestones and tasks — reads roadmap files and prints a formatted terminal summary
---

# Status Dashboard

Read the project's roadmap files and print a formatted terminal overview of progress, milestones, tasks, blockers, and next actions.

## Step 1: Read Roadmap Files

Attempt to read these files from the current project:

1. `.claude/roadmap/MILESTONE.md` — milestone statuses, descriptions, task membership (primary source of truth)
2. `.claude/roadmap/TASK.md` — task statuses, phases, dependencies, complexity, parallel/batching notes
3. `.claude/roadmap/ROADMAP.md` — project name, vision (read for header info only)
4. `.claude/roadmap/ARCHIVE.md` *(optional)* — archived milestones and tasks. If missing, skip silently. Archived milestones are rendered separately in Step 3.
5. `.claude/roadmap/drafts/` *(optional)* — count any `.md` files present for the drafts notice.

### Handle Empty States

**No roadmap files exist** (`.claude/roadmap/` directory missing or no ROADMAP.md):
→ Print `No roadmap found. Run /setup-roadmap to get started.` — stop here.

**Roadmap exists but no milestones defined** (MILESTONE.md has no milestone sections):
→ Print `Roadmap initialized but no milestones yet. Run /build-roadmap to define milestones.` — stop here.

**All milestones complete** (every milestone has status `completed`):
→ Render the full dashboard (header + milestone table), then append `All milestones complete! Roadmap finished.`

## Step 2: Parse Data

From the files, extract:

- **Project name** — from ROADMAP.md heading or frontmatter
- **Vision** — from ROADMAP.md Vision section (first line/paragraph)
- **Milestones** — name, status, description, task list from MILESTONE.md
- **Tasks** — ID, title, status, dependencies, complexity, milestone membership from TASK.md

For each milestone, compute:
- Total task count, completed task count (status = `completed`)
- Progress percentage = completed / total (round to nearest integer)

## Step 3: Render Dashboard

Use plain text and Unicode only. No ANSI escape codes.

### Header

```
== <PROJECT NAME> ==
<one-line vision>
```

Use the project name in uppercase. Use the first sentence or line from the Vision section.

### Milestone Summary Table

```
Milestones
----------
Name                     Status        Progress
orchestrator-hardening   in_progress   [█████░░░░░] 50%  (2/4)
status-dashboard         in_progress   [██░░░░░░░░] 20%  (1/5)
```

Progress bar: 10 chars wide, `█` (U+2588) for filled (`round(pct/10)`), `░` (U+2591) for empty. Show percentage and fraction `(completed/total)` after the bar. Align columns with spaces.

### Active Milestone Detail

For each milestone that is NOT `completed`, NOT `cancelled`, and NOT `planned`, show its tasks. If a milestone is `planned`, show: `No tasks defined yet — run /build-roadmap to break it down.`

```
Milestone: <name> — <description>
  ID                                Title                          Status       Deps              Complexity
  orchestrator-hardening-task001    Pre-dispatch contract valid...  in_progress  none              standard
  orchestrator-hardening-task002    Session context injection       pending      task001           standard
```

Align columns, truncate titles at ~35 chars. For deps, abbreviate by dropping the milestone prefix when it matches (e.g., `task001` instead of `orchestrator-hardening-task001`).

### Blocked Items

If any tasks have status `blocked`, show a `Blocked` section with `- <task-id>: <reason or dependency>`. Include reason from task contract if available, otherwise "reason not specified in index." Omit if none blocked.

### Next Up

List `pending` tasks with all deps met (every dep `completed`) as `- <task-id>: <title> (<complexity>)` under a `Ready to Work` heading. Omit if none ready.

### Completed Milestones

If ARCHIVE.md contains archived milestones, render one line per milestone: name, task count, completion date. Parse name from `### <milestone-name>` headings, count task rows under `## Archived Tasks`, use `> Updated:` date. Omit section if no archived milestones.

```
Completed
---------
orchestrator-hardening    2 tasks    2026-04-07
```

## Step 4: Sync ROADMAP.md

After rendering the dashboard, silently sync ROADMAP.md to match current state. This is a side-effect — don't announce it unless something changed.

1. Read `.claude/roadmap/ROADMAP.md`
2. Update the **Active Milestones** table: populate from MILESTONE.md — all milestones with status `planned`, `pending`, `in_progress`, `blocked`, or `review`. Include Status and Progress columns.
3. Update the **Completed History** table: all milestones with status `completed` — one row per milestone with name, completion date, and description. Cancelled milestones also go here with a "(cancelled)" note.
4. Update the `updated:` date in ROADMAP.md frontmatter to today.
5. Ensure the `milestones:` list in frontmatter matches all milestone names from MILESTONE.md + ARCHIVE.md.

This is the **only** skill that writes to ROADMAP.md during normal operation. `build-roadmap` also writes ROADMAP.md during initial creation or when adding milestones, but ongoing sync is `/status`'s responsibility. The cascade (`update-task`) does not touch it.

## Step 5: Drafts Notice

If `.claude/roadmap/drafts/` contains any `.md` files, append to the dashboard output:

```
Drafts
------
N draft(s) waiting to be formalized. Run /build-roadmap to review.
```

If no drafts exist, omit this section.

## Output Rules

- Plain text only — no markdown formatting, no code fences in the actual output
- Unicode box-drawing and block characters are fine
- Keep it compact — should fit in a terminal without scrolling for typical projects (< 20 tasks)
- Present the output directly to the user as a formatted text block
