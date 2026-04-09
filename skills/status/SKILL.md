---
name: status
description: Use when the user wants to check project status, see roadmap progress, or get an overview of milestones and tasks — reads roadmap files and prints a formatted terminal summary
---

# Status Dashboard

Read the project's roadmap files and print a formatted terminal overview of progress, milestones, tasks, blockers, and next actions.

## When to Use

- User asks "status?", "how's the project?", "what's the progress?", "show me the dashboard"
- Beginning of a session to get oriented
- After completing work to see updated progress

**Note:** This is a read-only overview. If the user wants to act on what they see (start a task, dispatch work), use `/work-task` instead.

## Step 1: Read Roadmap Files

Attempt to read these files from the current project:

1. `.claude/roadmap/MILESTONE.md` — milestone statuses, descriptions, task membership (primary source of truth)
2. `.claude/roadmap/TASK.md` — task statuses, phases, dependencies, complexity, parallel/batching notes
3. `.claude/roadmap/ROADMAP.md` — project name, vision (read for header info only)
4. `.claude/roadmap/ARCHIVE.md` *(optional)* — archived milestones and tasks. If this file does not exist, skip it — no error, no warning. Archived milestones are rendered separately in Step 3.
5. `.claude/roadmap/drafts/` *(optional)* — count any `.md` files present for the drafts notice.

### Handle Empty States

**No roadmap files exist** (`.claude/roadmap/` directory missing or no ROADMAP.md):

```
No roadmap found. Run /setup-roadmap to get started.
```

Stop here — do not render anything else.

**Roadmap exists but no milestones defined** (MILESTONE.md has no milestone sections):

```
Roadmap initialized but no milestones yet. Run /build-roadmap to define milestones.
```

Stop here — do not render anything else.

**All milestones complete** (every milestone has status `completed`):

Render the full dashboard (header + milestone table), then append:

```
All milestones complete! Roadmap finished.
```

## Step 2: Parse Data

From the files, extract:

- **Project name** — from ROADMAP.md heading or frontmatter
- **Vision** — from ROADMAP.md Vision section (first line/paragraph)
- **Milestones** — name, status, description, task list from MILESTONE.md
- **Tasks** — ID, title, status, dependencies, complexity, milestone membership from TASK.md

For each milestone, compute:
- Total task count
- Completed task count (status = `completed`)
- Progress percentage = completed / total (round to nearest integer)

## Step 3: Render Dashboard

Use plain text and Unicode only. No ANSI color codes, no external dependencies.

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

Progress bar rules:
- 10 characters wide
- Use `█` (U+2588) for filled portions
- Use `░` (U+2591) for empty portions
- Number of filled blocks = round(percentage / 10)
- Show percentage after the bar
- Show fraction in parentheses: `(completed/total)`

Column alignment: pad columns with spaces so they line up visually. Milestone names should be left-aligned.

### Active Milestone Detail

For each milestone that is NOT `completed`, NOT `cancelled`, and NOT `planned`, show its tasks. If a milestone is `planned` (no tasks defined), show a single line: `No tasks defined yet — run /build-roadmap to break it down.`

```
Milestone: <name> — <description>
  ID                                Title                          Status       Deps              Complexity
  orchestrator-hardening-task001    Pre-dispatch contract valid...  in_progress  none              standard
  orchestrator-hardening-task002    Session context injection       pending      task001           standard
```

Rules:
- Show all tasks belonging to the milestone
- Truncate long titles with `...` if needed to keep the table readable (max ~35 chars for title)
- For dependencies, show task IDs (can abbreviate by dropping the milestone prefix if it matches, e.g., `task001` instead of `orchestrator-hardening-task001`)
- Keep it scannable — align columns with spaces

### Blocked Items

If any tasks across all milestones have status `blocked`, show a dedicated section:

```
Blocked
-------
- <task-id>: <blocking reason or dependency>
```

If the task contract file exists and contains blocking details, include the reason. If only the status is known, show the task ID and note "reason not specified in index."

If no tasks are blocked, omit this section entirely.

### Next Up

List tasks that are `pending` with all dependencies met — these are available to work on right now.

```
Ready to Work
-------------
- <task-id>: <title> (<complexity>)
- <task-id>: <title> (<complexity>)
```

A task's dependencies are "met" when every task ID listed in its Dependencies column has status `completed`.

If no tasks are ready (all are blocked, in-progress, or completed), omit this section.

### Completed Milestones

If ARCHIVE.md was read and contains archived milestones, show a compact summary section after all other content. Parse each milestone under `## Archived Milestones` to extract:

- **Name** — from the `### <milestone-name>` heading
- **Task count** — count the task rows under the matching milestone heading in `## Archived Tasks`
- **Completion date** — use the `> Updated:` date from ARCHIVE.md (this is the archive modification date)

Render as:

```
Completed
---------
orchestrator-hardening    2 tasks    2026-04-07
status-dashboard          1 task     2026-04-07
```

Rules:
- One line per archived milestone — name, task count, date
- Use "task" (singular) when count is 1, "tasks" (plural) otherwise
- No task-level detail — milestones only
- Align columns with spaces for readability
- If no archived milestones exist (ARCHIVE.md missing or empty), omit this section entirely

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
- No ANSI escape codes or color codes
- Keep it compact — this should fit in a terminal without scrolling for typical projects (< 20 tasks)
- Present the output directly to the user as a formatted text block
- Do not offer to take actions — this is a read-only status view. If the user wants to work on something, they can invoke `/work-task`

## Complete Example

For a project with 2 active milestones, 3 tasks total, 1 completed task, and 1 archived milestone:

```
== WORKFLOW-FRAMEWORK ==
Enhance the plugin with smarter orchestration and better visibility.

Milestones
----------
Name                     Status        Progress
orchestrator-hardening   in_progress   [█████░░░░░] 50%  (1/2)
status-dashboard         in_progress   [░░░░░░░░░░]  0%  (0/1)

Milestone: orchestrator-hardening — Enhance work-task with contract validation and context injection
  ID                                Title                            Status       Deps     Complexity
  orchestrator-hardening-task001    Pre-dispatch contract validation  completed    none     standard
  orchestrator-hardening-task002    Session context injection         pending      task001  standard

Milestone: status-dashboard — New skill for formatted terminal progress summary
  ID                                Title                            Status       Deps     Complexity
  status-dashboard-task001          Status dashboard skill            in_progress  none     standard

Ready to Work
-------------
- orchestrator-hardening-task002: Session context injection (standard)

Completed
---------
initial-setup             3 tasks    2026-04-01
```
