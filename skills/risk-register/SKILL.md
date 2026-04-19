---
name: risk-register
description: Generate a Project+ style risk register as .xlsx — risks with category, probability, impact, severity score, owner, mitigation, contingency, status, and trigger. Use whenever a user needs to capture project risks for review, tracking, or sponsor visibility — triggers include "risk register", "risk log", "risks and mitigations", "what could go wrong", "probability impact matrix", "risk review". Enforces never-invent discipline: no fabricated risks, probabilities, impacts, owners, or mitigations; unconfirmed risks stay unconfirmed; reporting-line managers are not owners.
---

# Risk Register

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent.** Any risk, probability, impact, owner, mitigation, or trigger the user did not state → `[TBD — reason]`. Made-up risks become real conversations with sponsors and waste everyone's time.
2. **Never invent a scoring formula.** Use the standard 3×4 matrix (prob Low/Medium/High × impact Low/Medium/High/Critical) baked into the script. If the user uses a different scale, mark severity `[TBD — custom scale needed]`; do not invent your own conversion.
3. **Unconfirmed risks stay unconfirmed.** If the user says "grumbling but no one has confirmed," set status to `Unconfirmed` and probability/impact to `[TBD — unconfirmed]`. Do not assign categorical values just because the slot exists.
4. **PM is not the default owner.** If the user does not name an owner, use `[TBD]`. Do not auto-assign the PM, sponsor, or whoever last spoke.
5. **Reporting-line managers are not risk owners.** If the user says "Aisha reports to Raj," Raj cannot own a risk. A manager named in the *mitigation text* ("escalate to Elena") is fine only if the user put them there explicitly — do not promote a mitigation mention into an owner slot.
6. **Exact user wording** for risk descriptions and mitigations. "data loss during ticket ETL" stays "Data loss during ticket ETL," not "Potential data integrity failure in extraction pipeline."
7. **Partial specification is valid.** A risk with `probability=[TBD]` and a real mitigation is still a real row. Don't drop it; let the Draft Gaps sheet surface the gap.
8. **Fidelity pass before running.** For every probability, impact, owner, category, or mitigation in your JSON, cite a verbatim phrase in the user's message. If you can't → `[TBD]`.

## Run

```bash
python3 scripts/create_risk_register.py --output "<dir>/<ProjectName>_RiskRegister_v1.xlsx" <<'JSON'
{ ...schema... }
JSON
```

Script handles severity scoring, TBD highlighting, heat-map coloring, Draft Gaps, Integrity Issues, and `open_questions.md` when gaps ≥ 5.

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
  "risks": [
    {
      "id": "R001",
      "risk": "string — short description in user's words",
      "category": "string or [TBD]",
      "probability": "Low | Medium | High | [TBD — reason]",
      "impact": "Low | Medium | High | Critical | [TBD — reason]",
      "owner": "string or [TBD]",
      "mitigation": "string or [TBD]",
      "contingency": "string or [TBD] or empty",
      "status": "Open | In Progress | Closed | Unconfirmed",
      "trigger": "string or [TBD] or empty"
    }
  ]
}
```

Notes:
- `probability` must be one of Low/Medium/High or a `[TBD]` string. Script enforces this.
- `impact` must be one of Low/Medium/High/Critical or a `[TBD]` string.
- Severity is computed by the script from the standard matrix: Low/Medium/High/Critical (not a 1–12 number). Do not compute it yourself.
- `status` defaults to `Open`; use `Unconfirmed` for risks the user flagged as unvalidated.

## Severity matrix (baked into the script)

```
              Impact
Probability   Low       Medium    High      Critical
High          Medium    High      Critical  Critical
Medium        Low       Medium    High      Critical
Low           Low       Low       Medium    High
```

Any cell with a `[TBD]` probability or impact gets `[TBD — severity awaits P/I]`.

## Worked example

User: *"Risks: (1) Training gap (Taylor, medium/high, mitigation: office hours). (2) Data loss during ETL (Marcus, low/critical, mitigation: dry-run + rollback). (3) Something about session tokens breaking a legacy integration — grumbling but unconfirmed."*

Correct risks:
```
R001  Training gap                     Taylor Kim    Medium  High       Office hours                Open
R002  Data loss during ETL             Marcus Chen   Low     Critical   Dry-run + rollback          Open
R003  Session tokens breaking legacy   [TBD]         [TBD]   [TBD]      [TBD]                       Unconfirmed
```

Do NOT: fill R003's probability/impact just because the slots exist. Do NOT invent a numeric score. Do NOT promote a manager mentioned in a mitigation to an owner.

## Grounding (Project+)

Risk register = living document tracking identified risks through the project lifecycle. Project+ scoring uses **probability × impact** matrices — 3×3 or 5×5 most common. Each risk has an owner accountable for monitoring and executing the response. Response strategies: **avoid, transfer, mitigate, accept**. This skill uses a 3×4 matrix (P: L/M/H, I: L/M/H/Critical) to avoid overstating precision.
