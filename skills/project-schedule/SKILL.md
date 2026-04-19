---
name: project-schedule
description: Generate a Project+ style project schedule as .xlsx — task list with owners, durations, predecessors, computed start/finish dates, a Gantt-style bar column, and a critical path when dates are real. Use whenever a user needs to sequence tasks on a timeline — triggers include "project schedule", "schedule", "Gantt chart", "timeline", "task plan", "sequence the work", "when does X finish", "critical path". Enforces never-invent discipline: no fabricated tasks, durations, dates, dependencies, or owners; preserves fuzzy durations as TBD; excludes reporting-line managers.
---

# Project Schedule

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent.** Any task, duration, owner, predecessor, or date the user did not state → `[TBD — reason]`. Fabricated schedules become committed deadlines.
2. **No anchor date → no dates.** If the user did not give a kickoff/start date, every computed start/finish is `[TBD — no anchor date]`. Do not pick one.
3. **Preserve fuzzy durations.** "~2 weeks" → `[TBD — user said ~2 weeks]`, NOT `10`. "~1 week" → `[TBD — user said ~1 week]`, NOT `5`. The script propagates TBD to downstream task dates.
4. **Dependencies: only what the user said.** Default type is FS (finish-to-start). Do not invent SS/FF/SF relationships or lag/lead unless user stated them. "A after B" = FS, zero lag.
5. **One owner per task.** "Marcus's team" with no named lead → `[TBD — team owner not named]`. Multi-owner → split into subtasks or `[TBD — assign a single owner]`.
6. **Reporting-line managers never appear.** No "Reports To" column. If the user says "Aisha reports to Raj," Raj is not in the schedule. Period.
7. **Critical path only if all dates are real.** If any task has TBD duration, the Critical Path sheet says "unavailable — N tasks missing duration."
8. **Weekends skipped by default.** Holidays not modeled. Document these assumptions on the Notes sheet.
9. **Exact user wording** for task names — do not rename "mock cutover" to "Pre-production Validation."
10. **Fidelity pass before running.** For every duration, date, owner, or predecessor in your JSON, cite a verbatim phrase in the user's message. If you can't → `[TBD]`.

## Run

```bash
python3 scripts/create_schedule.py --output "<dir>/<ProjectName>_Schedule_v1.xlsx" <<'JSON'
{ ...schema... }
JSON
```

Script handles topological ordering, weekend-skip date math, Gantt bars, Draft Gaps, Critical Path (when computable), Integrity Issues, Notes, and `open_questions.md` when gaps ≥ 5.

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "project_manager": "string or [TBD]",
    "sponsor": "string or [TBD]",
    "anchor_start_date": "YYYY-MM-DD or [TBD]",
    "date": "YYYY-MM-DD",
    "version": "v1"
  },
  "tasks": [
    {
      "id": "1",
      "name": "string",
      "owner": "string or [TBD]",
      "duration_days": "number or [TBD — reason]",
      "predecessors": "comma-separated ids or empty",
      "dependency_type": "FS | SS | FF | SF (default FS)",
      "lag_days": "number (default 0)"
    }
  ]
}
```

Notes:
- `tasks` is flat; `id` is any string (e.g. "1", "1.1", "T01"). Predecessors reference these ids.
- Only include `dependency_type` or `lag_days` if the user explicitly stated them. Omit otherwise — script defaults FS/0.
- `duration_days` must be a positive number or TBD. Never convert fuzzy words to numbers.
- `anchor_start_date` missing → all computed dates are TBD; the schedule is still useful for sequencing review.

## Worked example

User: *"Schedule for ServiceNow migration. Field mapping 3 days Marcus, ETL 1 week Jen after field mapping, mock cutover 2 days Marcus after ETL. Kickoff June 1, 2026."*

Correct tasks:
```
1  Field mapping       Marcus Chen   3 days       preds=       FS
2  ETL                 Jen Wu        5 days       preds=1      FS
3  Mock cutover        Marcus Chen   2 days       preds=2      FS
```
Anchor start = 2026-06-01. Script computes dates.

User: *"...Aisha works on the core auth service, about 2 weeks. Aisha reports to Raj Patel."*

Correct task:
```
4  Core auth service   Aisha Khan    [TBD — user said ~2 weeks]   ...
```
Raj Patel does not appear anywhere. Do NOT encode "2 weeks" as `10`.

## Grounding (Project+)

Project schedule = time-phased task list with dependencies. Project+ uses **PDM (precedence diagramming method)** with four dependency types: FS (most common), SS, FF, SF, plus lag/lead. **Critical path** = longest chain of dependent tasks; slack = difference between late and early dates. Baselines are frozen at approval and compared to actuals to compute **schedule variance**. The bundled script handles sequencing, date math, Gantt rendering, and critical-path computation when inputs are complete.
