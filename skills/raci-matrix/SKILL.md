---
name: raci-matrix
description: Generate a Project+ style RACI (Responsibility Assignment Matrix) as a .xlsx, mapping project activities to team members with R/A/C/I assignments. Use whenever a user needs to clarify *who does what* — triggers include "RACI", "responsibility matrix", "RAM", "who owns what", "role clarity", "accountability matrix". Enforces Project+ RACI rules (exactly one A per row, ≥1 R) and prevents fabrication of people or activities.
---

# RACI Matrix

Generate a Project+ RACI .xlsx. Draft-first. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent people or activities.** Unknown cells → `[TBD]`. Use exact user wording for activity names.
2. **Reporting-line managers are NOT columns.** *"Marcus reports to Alicia"* → Alicia gets no column. Only people the user described as doing project work. Sponsor is the exception. Check every column before running the script.
3. **RACI integrity:** every row has **exactly one A** and **≥1 R**. R and A may be same person (use `R/A`). If the Accountable is unknown, set that cell to `[TBD]` — the script flags it on the Draft Gaps sheet.
4. **"No team yet"** → single column `[TBD — team unknown]` with all cells TBD. Do **not** invent role columns from activity keywords (e.g. "finance dashboard" ≠ "Finance Lead").
5. **Don't expand "the team"** into invented names. "The dev team deploys" → one column "Dev Team" with R.

## Verb → letter

- builds / does / runs / executes / writes → **R**
- approves / signs off / has final say → **A**
- reviews / consulted / advises → **C**
- informed / notified / FYI → **I**

The doer is R, the approver is A — they are usually different people. *"Ken drives scope, Mei approves scope"* → Ken=R, Mei=A (one row, one A). Do not put A on both.

**"I'm accountable overall" is PM framing, not a per-row A.** Do not propagate an A into every row just because the PM said they're accountable for schedule/scope. PMs use "accountable" loosely to mean "ultimately on the hook" — a RACI A is narrower: the single person who signs off *on this specific deliverable*. If the user said "I'm accountable for schedule and scope," the PM gets at most **two** As: one row for "Schedule management" and one for "Scope management" (if those appear as activities). They do NOT get an A on "Build the data export tool," "Run the migration," or "Validate ticket history" — those have their own approvers.

**Worked example (the trap):** User says *"Marcus owns the build. Priya approves cutover. I'm Ken, accountable for scope and schedule overall."*

- ❌ WRONG: Build=Marcus R + Ken A, Migration=Marcus R + Ken A, Cutover=Marcus R + Ken A + Priya A (2 As), ...
- ✓ RIGHT: Build=Marcus R/A, Migration=Marcus R/A, Cutover=Marcus R + Priya A, Scope management=Ken R/A, Schedule management=Ken R/A.

Before running the script, scan every row: if the same person holds A across more than 2–3 rows *and the user did not explicitly name them as approver on each*, you are propagating. Stop and re-read the verbs.

## Run it

```bash
python3 scripts/create_raci.py --output "<dir>/<ProjectName>_RACI_v1.xlsx" <<'JSON'
{ ...schema... }
JSON
```

Script validates RACI integrity, highlights TBDs, writes Draft Gaps + Integrity Issues sheets, and emits `open_questions.md` on many gaps.

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
  "people": ["Column header 1", "Column header 2"],
  "activities": [
    {
      "name": "activity / deliverable",
      "phase": "optional phase string or null",
      "assignments": {
        "Column header 1": "R | A | C | I | R/A | [TBD] | \"\""
      }
    }
  ]
}
```

`people` = column headers in order. Every assignment key must match a people entry exactly. Empty string = not involved. Keep activity names ≤ 60 chars.
