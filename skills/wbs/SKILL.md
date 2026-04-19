---
name: wbs
description: Generate a Project+ style Work Breakdown Structure (WBS) as a .xlsx, decomposing project deliverables into a hierarchy of work packages with codes, owners, effort estimates, and durations. Use whenever a user needs to break a project into plannable chunks — triggers include "WBS", "work breakdown structure", "work packages", "project decomposition", "break down the project", "deliverables hierarchy", "scope decomposition", or any context where a PM is turning a charter into plannable work. Enforces Project+ WBS conventions (100% rule, 8–80 hour work packages, hierarchical numbering) and prevents fabrication of work packages, owners, or estimates the user never stated.
---

# Work Breakdown Structure (WBS)

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent.** Any deliverable, work package, owner, effort, or duration the user did not state → `[TBD — reason]`. The WBS is load-bearing for schedule/budget; fabrication anchors bad decisions.
2. **100% rule.** Children fully describe the parent — nothing missing, nothing extra. Don't add a 4th child just because a typical project has one.
3. **Work packages = 8–80 hours, one owner.** Level 1 = project. Level 2 = phases or major deliverables (use whichever the user used — do not invent "Initiate/Plan/Execute/Close"). Level 3+ = work packages. If the user hasn't decomposed that far, stop at level 2 and mark level 3 `[TBD]`.
4. **Hierarchical codes.** Dotted: `1`, `1.1`, `1.1.1`. The script indents by code depth.
5. **Use the user's exact wording** for deliverables. "agent training" → "Agent training", not "Training Program Development".
6. **Reporting-line managers are not owners.** If the user named someone only as a boss ("Aisha reports to Raj"), Raj does not appear anywhere. Same rule as team-roster.
7. **Preserve numeric imprecision.** "About 2 weeks" → `[TBD — user said ~2 weeks]`, not `80`. "About a month" → `[TBD — user said ~1 month, exceeds 80-hr rule]`, not `160`.
8. **Fidelity pass before running.** For every hour count, day count, owner, or dep in your JSON, point to a verbatim phrase in the user's message. If you can't, replace with `[TBD]`.

## Run

```bash
python3 scripts/create_wbs.py --output "<dir>/<ProjectName>_WBS_v1.xlsx" <<'JSON'
{ ...schema... }
JSON
```

Script handles indentation, rollups, TBD highlighting, Draft Gaps sheet, Work Package Summary, Integrity Issues sheet, and `open_questions.md` when gaps ≥ 5.

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
  "elements": [
    {
      "code": "1",
      "level": 1,
      "name": "string",
      "type": "Project | Phase | Deliverable | Work Package",
      "description": "string or [TBD]",
      "owner": "string or [TBD]",
      "effort_hours": "number or [TBD]",
      "duration_days": "number or [TBD]",
      "dependencies": "comma-separated codes or [TBD] or empty"
    }
  ]
}
```

- Flat array; `code` encodes hierarchy. Script sorts by code and indents by level.
- Effort/duration only required on Work Packages (types Project/Phase/Deliverable roll up from children automatically).
- Work package `effort_hours` must be 8–80. Translate "a month" → `[TBD — exceeds 80-hr rule, needs decomposition]`, not 160.

## Worked example

User: *"ServiceNow migration has three pieces: ticket migration (Marcus's team, ~1 month), agent training (Taylor running workshops, ~2 weeks), and Salesforce+Slack integrations (owners TBD). I'm Ken, PM."*

Correct:
```
1     ServiceNow Migration                  Project        level 1
1.1   Ticket migration                      Deliverable    level 2, Marcus Chen, [TBD — ~1 month exceeds 80-hr rule]
1.2   Agent training                        Deliverable    level 2, Taylor Kim, [TBD — ~2 weeks]
1.3   Salesforce integration                Deliverable    level 2, [TBD]
1.4   Slack integration                     Deliverable    level 2, [TBD]
```

Do NOT invent level-3 packages ("Map ticket fields", "Write migration scripts") the user didn't name. Do NOT add phases the user didn't use. Do NOT convert "about a month" to 160h.

## Grounding (Project+)

WBS = deliverable-oriented decomposition; lowest level is **work packages** (estimable, manageable). **100% rule** = captures all deliverables, internal+external+interim. Work packages sized **8–80 hours**. Hierarchically coded. The script enforces columns, indentation, rollups, and integrity checks.
