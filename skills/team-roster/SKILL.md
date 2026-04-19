---
name: team-roster
description: Generate a Project+ style team roster as a .xlsx listing every known project team member with role, responsibilities, availability, manager, contact, and core/extended status. Use whenever a user needs to capture "who is on the project" in a structured artifact — triggers include "team roster", "project team", "staffing plan", "team structure", "who's on the project", "team list", "resource roster", or any context where a PM is standing up a team. Enforces Project+ conventions and prevents fabrication of people, titles, or details the user never stated.
---

# Team Roster

Generate a Project+ team roster .xlsx. Draft-first: parse what the user gave, fill schema with facts + TBDs, run the script. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent people, names, titles, availability, managers, or contact methods.** Anything not in the user's message → `[TBD — <reason>]`. Preserve exact user phrasing ("half time" stays "half time").
2. **Reporting-line managers are NOT team members.** If the user says *"Marcus reports to Alicia"* or *"6 engineers led by Raj"*, Alicia/Raj go in the `manager` field of the people they manage — they do **NOT** get their own row. Exception: the sponsor. Before running the script, check every row: *"Did the user describe this person as doing project work, or only as someone's boss?"* If only boss → delete the row.
3. **Core vs. Extended:** only Core if the user explicitly said so. Else `[TBD]`.
4. **"N engineers" with no names** → N rows with `name: "[TBD — only count given]"`. Do not invent "Engineer 1".

## Run it

Pipe JSON directly to the script (one call, no temp file):

```bash
python3 scripts/create_roster.py --output "<dir>/<ProjectName>_Team_Roster_v1.xlsx" <<'JSON'
{ ...schema... }
JSON
```

The script handles formatting, TBD highlighting, Draft Gaps sheet, and `open_questions.md` when TBDs ≥ 5.

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "project_manager": "string or [TBD]",
    "sponsor": "string or [TBD]",
    "date": "YYYY-MM-DD",
    "version": "v1"
  },
  "members": [
    {
      "name": "string or [TBD]",
      "job_position": "string or [TBD]",
      "core_or_extended": "Core | Extended | [TBD]",
      "subteam": "string or [TBD]",
      "responsibilities": "string or [TBD]",
      "availability": "e.g. 100%, 2 days/wk, or [TBD]",
      "manager": "string or [TBD]",
      "organization": "string or [TBD]",
      "contact_method": "string or [TBD]"
    }
  ]
}
```

Keep responsibilities ≤ 2 sentences per member. Any field may be `[TBD — reason]`.
