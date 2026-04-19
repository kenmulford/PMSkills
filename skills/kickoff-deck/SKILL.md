---
name: kickoff-deck
description: Generate a Project+ style project kickoff deck as a .pptx — title, vision, objectives, scope, team, milestones, risks, ways of working, and asks. Use whenever a user needs to run or prepare for a project kickoff meeting — triggers include "kickoff deck", "kickoff slides", "kickoff presentation", "kickoff meeting", "project kickoff", "launch deck", or any context where a PM is formally introducing a new project to stakeholders and team. Enforces Project+ kickoff structure and prevents fabrication of metrics, dates, names, or commitments the user never stated.
---

# Kickoff Deck

Generate a Project+ kickoff deck .pptx. Draft-first: parse what the user gave, fill the schema with facts + TBDs, run the script. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent.** No made-up metrics, dates, names, risks, or commitments. Unknown → `[TBD — reason]`. A kickoff deck shown to stakeholders with invented content damages PM credibility on day one.
2. **Preserve exact time/date/quantity wording — by table.** Never convert imprecision into precision to make a milestone or SLA look concrete.

   | User said | NEVER write | Write |
   |---|---|---|
   | October | October 31, end of October, 10/31 | October |
   | Q2 | June 30, Q2 2026-06-30, end of June | Q2 |
   | end of April | April 30, late April 2026 | end of April |
   | $150k | $150,000, $150K USD | $150k |
   | ~6 months | 180 days, 6.0 months | ~6 months |
   | zero downtime | 99.9% uptime, 99.99% SLA | zero downtime |

3. **Verbatim-grep check (mandatory pre-render step).** Before emitting JSON, for every number, percentage, date, dollar figure, or SLA in your draft, search the user's original message for that exact token. If the token is not present **literally**, replace it with `[TBD]`. This converts "don't invent" from a principle into a mechanical check. `"99.9% uptime"` in the draft when Ken said only `"zero downtime"` fails this check — strip it.
4. **Reporting-line managers are NOT team slide entries.** *"Marcus reports to Raj"* → Raj does NOT appear on the team slide. Build a ban list from any name appearing only inside `"X reports to Y"`, `"under Y"`, `"Y's report"`, `"rolls up to Y"`, or `"Y is X's manager"` phrases. That ban list cannot appear on the team slide. The sponsor is the exception (and only if named separately from the reporting-line phrase).
5. **Don't invent stakeholders from project keywords.** *"Finance dashboard"* does NOT create a "Finance Team" bullet. Only list people/groups the user explicitly named.
6. **Keep slides tight.** Max 5 bullets per slide, ≤12 words per bullet. Kickoff decks are read aloud — cramped slides fail.

## Run it

```bash
python3 scripts/create_kickoff.py --output "<dir>/<ProjectName>_Kickoff_v1.pptx" --source /tmp/source.txt <<'JSON'
{ ...schema... }
JSON
```

Pass `--source <path>` with the user's raw message text. The script will lint every `\d+%`, `$\d+[kKmM]?`, and calendar date in the draft against the source and add any unverified figures to the Draft Gaps slide with a `⚠ unverified` marker. If you omit `--source`, linting is skipped (acceptable for testing, not acceptable for user deliverables).

Script handles all layout, TBD highlighting, Draft Gaps summary slide, figure linting, and `open_questions.md` when TBDs ≥ 5.

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
  "vision": "1–2 sentences on why this project exists (from user's words)",
  "objectives": [
    "SMART-ish objective bullet or [TBD]"
  ],
  "scope": {
    "in_scope": ["bullet"],
    "out_of_scope": ["bullet"]
  },
  "team": [
    {"name": "string or [TBD]", "role": "string or [TBD]"}
  ],
  "milestones": [
    {"name": "string", "target": "date or phase wording or [TBD]"}
  ],
  "risks": [
    {"risk": "string", "mitigation": "string or [TBD]"}
  ],
  "ways_of_working": {
    "cadence": "e.g. weekly standup / biweekly steering / [TBD]",
    "tools": "e.g. Slack #channel, Jira, email / [TBD]",
    "decision_forum": "where decisions get made / [TBD]"
  },
  "asks": [
    "what the PM needs from stakeholders today or [TBD]"
  ]
}
```

Arrays may be empty. Any string may be `[TBD — reason]`. Keep each bullet ≤12 words.

Before running the script, do a fidelity pass: for every name, date, metric, or risk, point to a verbatim phrase in the user's message. If you can't, replace with `[TBD]`.
