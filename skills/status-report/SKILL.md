---
name: status-report
description: Generate a Project+ style weekly project status report as .docx — executive summary, RAG with trend vs prior week, accomplishments, upcoming, risks vs issues (distinguished), asks, metrics, and dependencies. Use whenever a user needs to produce a status report, weekly update, steering committee update, stakeholder update, or "where are we" document. Triggers include "status report", "weekly status", "exec update", "sponsor update", "RAG status", "this week's update". Enforces never-invent discipline and structural consistency across reporting weeks so stakeholders can scan the same sections in the same order every time.
---

# Status Report

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Why this skill exists

Status reports live or die by **consistency**. The value isn't in the single-week artifact — it's that every week has the same sections in the same order, the same RAG scale, and an explicit trend comparison vs the prior week. Unskilled baselines drift: they reorder sections, rename "Risks" to "Concerns," forget to compare RAG week-over-week, and silently convert the user's exact metrics into rounded percentages. This skill nails the format to the wall.

## Hard rules

1. **Never invent.** Accomplishments, upcoming work, risks, issues, metrics, asks, dependencies — if the user didn't say it, it's not in the report. No `[TBD]` filler either; an empty section is correct if nothing happened.
2. **Never assign a RAG color.** If the user did not say Green/Yellow/Red, the RAG cell is `[TBD — PM to assign]`. Do not infer.
3. **Trend only with a prior baseline.** Show `↑ improving`, `↓ declining`, `→ stable` ONLY if the user provided `previous_rag` or prior-week context. Otherwise show `— no prior baseline`. Do not guess.
4. **Risks vs Issues are different sections.** Per Project+: a **risk** is a potential future event; an **issue** is happening now. Don't collapse them. If the user says "X is slowing us down right now," that's an issue. If they say "X might slip later," that's a risk.
5. **Preserve exact user wording** for metrics and counts. "6 of 40 confirmed" stays "6 of 40 confirmed" — do NOT compute "15%". "Ahead of schedule" stays as text, not a fabricated "+3 days."
6. **Reporting-line managers never appear.** If the user says "Aisha reports to Raj," Raj is not in the report. Full stop.
7. **Section order is fixed.** Header → Executive Summary → RAG + Trend → Metrics (if any) → Accomplishments → Upcoming → Risks → Issues → Asks → Dependencies → Appendix (Draft Gaps if any). Same order every week. Do not editorialize the ordering.
8. **Asks have an owner and a needed-by.** If either is missing, mark that field TBD — but keep the ask in the report.
9. **Fidelity pass.** For every sentence in accomplishments/upcoming/risks/issues/asks/metrics, cite a verbatim phrase in the user's message. If you can't, leave the slot empty.

## Run

```bash
python3 scripts/create_status.py --output "<dir>/<ProjectName>_Status_<week>.docx" <<'JSON'
{ ...schema... }
JSON
```

Script produces a .docx with the fixed section order, a RAG badge, a trend arrow when possible, and an Appendix listing any TBD items. Emits `open_questions.md` when gaps ≥ 5.

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "project_manager": "string or [TBD]",
    "sponsor": "string or [TBD]",
    "week_ending": "YYYY-MM-DD",
    "version": "v1"
  },
  "rag": "Green | Yellow | Red | [TBD — reason]",
  "previous_rag": "Green | Yellow | Red or null",
  "executive_summary": "string (1-2 sentences) or [TBD]",
  "metrics": [
    {"name": "string", "value": "string — user's verbatim wording", "target": "string or empty"}
  ],
  "accomplishments": ["string — user's words"],
  "upcoming": ["string"],
  "risks": [
    {"risk": "string", "status": "Open | Mitigating | Closed", "mitigation": "string or [TBD]"}
  ],
  "issues": [
    {"issue": "string", "owner": "string or [TBD]", "impact": "string or [TBD]"}
  ],
  "asks": [
    {"ask": "string", "owner": "string or [TBD]", "needed_by": "string or [TBD]"}
  ],
  "dependencies": ["string"]
}
```

Notes:
- `metrics[i].value` is a STRING, not a number. "6 of 40 confirmed" goes in verbatim. Do not convert to percentage.
- `previous_rag` is `null` if the user didn't reference last week. The script handles the trend rendering.
- Empty arrays are valid and preferred over fabricated filler.

## Worked example

User: *"Weekly status for ServiceNow migration. Ken PM, reporting Priya. RAG yellow — last week was green. Field mapping complete ahead of schedule. Jen started ETL. Taylor drafted deck. Salesforce integration still has no owner, escalated to Priya, no reply yet. Ask: need owner confirmation by Friday. Upcoming: Jen continues ETL, Taylor runs workshop 1, mock cutover dry-run Tuesday."*

Correct JSON:
```json
{
  "rag": "Yellow",
  "previous_rag": "Green",
  "accomplishments": [
    "Field mapping complete ahead of schedule",
    "Jen started ETL",
    "Taylor drafted the training deck"
  ],
  "upcoming": ["Jen continues ETL", "Taylor runs workshop 1", "Mock cutover dry-run Tuesday"],
  "risks": [],
  "issues": [
    {"issue": "Salesforce integration has no owner", "owner": "[TBD]", "impact": "Escalated to Priya, no reply yet"}
  ],
  "asks": [
    {"ask": "Confirm Salesforce integration owner", "owner": "Priya Patel", "needed_by": "Friday"}
  ]
}
```

Note: Salesforce-no-owner is an **issue** (happening now), not a risk. RAG went Green → Yellow = `↓ declining`. Do NOT invent a percentage for "ahead of schedule." Do NOT add filler accomplishments.

## Grounding (Project+)

Status reports are the primary communication artifact to stakeholders. Project+ distinguishes **risks** (potential events that may occur) from **issues** (events that have occurred and are impacting the project). RAG (Red/Amber/Green) is the standard health indicator. Trend reporting (whether health is improving/declining vs the last period) is what makes status useful over time — a single Yellow is context-free; a Green→Yellow transition is a signal. This skill enforces that discipline structurally so PMs don't have to re-invent the format each week.
