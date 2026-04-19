---
name: stakeholder-engagement-plan
description: Generate a Project+ style Stakeholder Engagement Plan as a .docx, with a stakeholder register, power/interest grid, current-vs-desired engagement levels, and a communication plan per stakeholder. Use whenever a user needs to plan how to manage, communicate with, or influence project stakeholders — triggers include "stakeholder engagement plan", "stakeholder register", "stakeholder analysis", "power/interest grid", "communication plan", "stakeholder management", "SEP". Enforces Project+ engagement-level vocabulary and prevents fabrication of stakeholders, positions, or concerns.
---

# Stakeholder Engagement Plan

Generate a Project+ SEP .docx. Draft-first. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent stakeholders, positions, or concerns.** Unknown → `[TBD — reason]`. Do not paraphrase *"skeptical about the budget"* into *"concerns about budget overrun"* — those are different claims. If the user said "skeptical," write "skeptical." Fabricated concerns become fabricated comms pitched at problems the stakeholder never raised.
2. **Reporting-line managers are NOT stakeholders.** *"Noor reports to Elena"* → Elena does NOT get a row unless the user separately described her as someone the project needs to engage. The sponsor IS a stakeholder by definition. Check every row before running the script.
3. **Do NOT invent stakeholders from project description keywords.** *"Finance dashboard project"* does NOT create a "Finance Team" stakeholder. *"Auth service"* does NOT create a "Security Team". Only add rows for people/groups the user explicitly named as needing to be informed, influenced, or consulted.
4. **Engagement levels are a controlled vocabulary.** Use only `Unaware | Resistant | Neutral | Supportive | Leading`. No *"Skeptical"* or *"Enthusiastic"*. Each stakeholder gets a `current` and `desired` level.
5. **Group "the X team"** as one row rather than inventing individuals. *"Security team approves"* → one row "Security Team", not 3 invented engineers.
6. **Quadrants:** `Manage Closely` (hi power/hi interest), `Keep Satisfied` (hi/lo), `Keep Informed` (lo/hi), `Monitor` (lo/lo). Must match stated power + interest — don't contradict.

## Run it

```bash
python3 scripts/create_sep.py --output "<dir>/<ProjectName>_Stakeholder_Engagement_Plan_v1.docx" <<'JSON'
{ ...schema... }
JSON
```

Script handles formatting, Draft Gaps callout, power/interest grid, engagement matrix, comms plan, and `open_questions.md` when TBDs ≥ 5.

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
  "stakeholders": [
    {
      "name": "string or [TBD]",
      "role_or_title": "string or [TBD]",
      "organization": "string or [TBD]",
      "interest": "what they care about (1–2 sentences) or [TBD]",
      "power": "High | Medium | Low | [TBD]",
      "interest_level": "High | Medium | Low | [TBD]",
      "quadrant": "Manage Closely | Keep Satisfied | Keep Informed | Monitor | [TBD]",
      "current_engagement": "Unaware | Resistant | Neutral | Supportive | Leading | [TBD]",
      "desired_engagement": "Unaware | Resistant | Neutral | Supportive | Leading | [TBD]",
      "concerns_or_risks": "string or [TBD]",
      "communication": {
        "frequency": "string or [TBD]",
        "channel": "string or [TBD]",
        "message_type": "string or [TBD]",
        "owner": "string or [TBD]"
      }
    }
  ]
}
```

Stakeholders may be empty. Any string may be `[TBD — reason]`. Keep prose fields ≤ 2 sentences.

Before running the script, for every specific rating, concern, or comms detail ask: *can I point to a verbatim phrase in the user's message?* If not, replace with `[TBD]`.
