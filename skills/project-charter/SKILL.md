---
name: project-charter
description: Generate a sponsor-ready project charter and preliminary scope statement as a .docx. Use whenever a user is starting a new project and needs to formally frame it — triggers include "project charter", "charter doc", "scope statement", "preliminary scope", "initiating a project", "PID", "authorize the project", or any context where a PM needs to capture vision, scope, deliverables, success criteria, and sponsor approval in one document. Enforces Project+ conventions and prevents fabrication of numbers, dates, or constraints the user never stated.
---

# Project Charter

Generate a Project+ charter + preliminary scope statement .docx. Draft-first: parse what the user gave, fill schema with facts + TBDs, run the script. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent.** No made-up numbers, dates, names, metrics, constraints, or stakeholders. Anything unknown → `[TBD — <reason>]`. Fabricated charter content becomes fabricated requirements downstream.
2. **Preserve exact time/date wording.** *"October"* stays *"October"* — never *"October 31"* or *"end of October"*. *"Q2"* stays *"Q2"* — never *"June 30"*. *"$150k"* stays *"$150k"* — never *"$150,000"*. If you find yourself converting an imprecise phrase into a specific date to complete a SMART objective, stop and mark that date portion `[TBD]`.
3. **Out-of-scope must not be empty.** If the user didn't state exclusions, use `[TBD — ask sponsor what adjacencies should be excluded]` as a single bullet. Empty exclusions = charter bug.
4. **≤3 sentences per field.** Project+ convention.

## Run it

Pipe JSON directly to the script (one call, no temp file):

```bash
python3 scripts/create_charter.py --output "<dir>/<ProjectName>_Charter_v1.docx" <<'JSON'
{ ...schema... }
JSON
```

Script handles all formatting, TBD highlighting, Draft Gaps box at top, and `open_questions.md` when TBDs ≥ 5.

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
  "vision": "1–2 sentence problem statement and why this project exists",
  "objectives": [
    {
      "statement": "SMART statement",
      "leading_indicator": "early predictive signal",
      "lagging_indicator": "realized outcome metric"
    }
  ],
  "scope": {
    "summary": "2–3 sentence scope description",
    "in_scope": ["bullet"],
    "out_of_scope": ["bullet"]
  },
  "preliminary_scope": {
    "deliverables": ["bullet"],
    "acceptance_criteria": ["bullet"],
    "assumptions": ["bullet"],
    "constraints": {
      "budget": "string or [TBD]",
      "timeline": "string or [TBD]",
      "resources": "string or [TBD]",
      "technical_or_regulatory": "string or [TBD]"
    }
  },
  "stakeholders": [{"name_or_group": "string", "role": "string"}],
  "implementation": {
    "milestones": [{"name": "string", "target_date": "string or [TBD]"}],
    "risks": [{"risk": "string", "impact": "H/M/L", "likelihood": "H/M/L", "response": "string"}],
    "dependencies": ["bullet"]
  }
}
```

Arrays may be empty. Any string may be `[TBD — reason]`.
