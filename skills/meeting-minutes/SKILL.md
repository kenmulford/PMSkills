---
name: meeting-minutes
description: Generate project meeting minutes as .docx — attendee list (present/absent), agenda items discussed, decisions made, action items with owners and due dates, parking lot, and next meeting. Use whenever a user needs meeting minutes, meeting notes, action items writeup, steering committee minutes, standup recap, or retro notes. Triggers include "meeting minutes", "meeting notes", "action items", "steering committee minutes", "standup notes", "MoM". Enforces never-invent discipline for attendees, decisions, action items, and quoted statements.
---

# Meeting Minutes

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Why this skill exists

Meeting minutes are legally and organizationally load-bearing: they're the record of who was there, what was decided, and who is on the hook. Unskilled baselines invent with abandon here — they add attendees "who probably were there," turn discussion into "decisions," assign action items to people who weren't named as owners, and convert "we should look into that" into "ACTION: John to research by Friday." Every one of those fabrications creates downstream accountability failures. This skill refuses all of it.

## Hard rules

1. **Attendees are who the user named.** If the user listed 4 people as attending, the minutes have 4 present. Do NOT add the PM, sponsor, or "standard attendees" by default. Absent attendees are only those the user explicitly said were absent.
2. **Decisions are only what the user said was decided.** A discussion is not a decision. "We talked about pricing" is not "Decision: set price at $X." If the user described discussion without a resolution, the decisions list is empty for that topic.
3. **Action items require an owner and a due date.** If either is missing, the action item still gets recorded but the missing field is `[TBD]`. Do NOT guess an owner ("probably Marcus since it's technical") or a due date ("next Friday seems likely").
4. **Never fabricate quotes.** If the user said "Priya pushed back on timeline," the minutes say that. Do NOT render it as "Priya stated: 'The timeline is unrealistic.'"
5. **Parking lot is separate from action items.** Things raised but not resolved go to the parking lot — they are not action items with fake owners. Parking lot entries have no owner, no due date.
6. **Risks surfaced ≠ new risks accepted.** If the user said "Taylor raised a concern about training time," that's a parking lot item or a risk-surfaced note, not a commitment to mitigate.
7. **Reporting-line managers never appear.** If the user says "Marcus reports to Raj," Raj is not listed as attending unless the user explicitly said Raj attended.
8. **Next meeting date only if stated.** Do not default to "next week same time."
9. **Agenda items are the user's list.** Do not interpolate "Review of last meeting's action items" at the top or "AOB" at the bottom unless the user stated them.
10. **Fidelity pass.** Every attendee, decision, action item, and quote must trace to a verbatim phrase in the user's message. If you can't cite it, leave the slot empty.

## Run

```bash
python3 scripts/create_minutes.py --output "<dir>/<ProjectName>_Minutes_<date>.docx" <<'JSON'
{ ...schema... }
JSON
```

Section order: Header → Attendees (Present / Absent) → Agenda → Decisions → Action Items → Parking Lot → Next Meeting → Appendix (Draft Gaps).

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "meeting_title": "string",
    "meeting_date": "YYYY-MM-DD",
    "meeting_time": "string or empty",
    "location": "string or empty",
    "chair": "string or [TBD]",
    "scribe": "string or [TBD]"
  },
  "attendees": {
    "present": ["string — name, optionally with role"],
    "absent": ["string"]
  },
  "agenda_items": [
    {"topic": "string", "discussion": "string — user's summary", "presenter": "string or empty"}
  ],
  "decisions": [
    {"decision": "string", "decided_by": "string or [TBD]", "rationale": "string or empty"}
  ],
  "action_items": [
    {"action": "string", "owner": "string or [TBD]", "due_date": "YYYY-MM-DD or [TBD]", "status": "Open"}
  ],
  "parking_lot": ["string"],
  "next_meeting": {
    "date": "YYYY-MM-DD or [TBD]",
    "topic": "string or empty"
  }
}
```

Notes:
- `attendees.present` is exactly what the user said — no inference.
- `decisions` is empty if the user only described discussion.
- `action_items` always includes the action, even if owner or due date is `[TBD]`. Owner and due date are never guessed.
- `parking_lot` is where discussion-without-resolution goes.

## Worked example

User: *"Minutes for ServiceNow weekly sync, April 15 2026, 10am, Zoom. Present: Ken (PM, chair), Marcus Chen, Jen Wu, Taylor Kim. Priya was out. Agenda: (1) Salesforce integration owner — discussed candidates, no decision yet. (2) Workshop 2 prep — Taylor walked through slides, team approved them. (3) Data mapping bug — Marcus reported 2-day fix estimate, agreed to run full cutover test Thursday. Action: Marcus completes bug fix by EOD Wednesday. Taylor to send workshop 2 materials to attendees Friday. Parking lot: training expansion to 3 weeks. Marcus reports to Raj."*

Correct JSON sketch:
```json
{
  "attendees": {
    "present": ["Ken (PM, chair)", "Marcus Chen", "Jen Wu", "Taylor Kim"],
    "absent": ["Priya"]
  },
  "agenda_items": [
    {"topic": "Salesforce integration owner", "discussion": "Discussed candidates, no decision yet", "presenter": ""},
    {"topic": "Workshop 2 prep", "discussion": "Taylor walked through slides, team approved them", "presenter": "Taylor"},
    {"topic": "Data mapping bug", "discussion": "Marcus reported 2-day fix estimate", "presenter": "Marcus"}
  ],
  "decisions": [
    {"decision": "Approved Workshop 2 slides", "decided_by": "team", "rationale": ""},
    {"decision": "Run full cutover test Thursday", "decided_by": "[TBD]", "rationale": ""}
  ],
  "action_items": [
    {"action": "Complete data mapping bug fix", "owner": "Marcus", "due_date": "[TBD — user said EOD Wednesday]", "status": "Open"},
    {"action": "Send workshop 2 materials to attendees", "owner": "Taylor", "due_date": "[TBD — user said Friday]", "status": "Open"}
  ],
  "parking_lot": ["Training expansion to 3 weeks"]
}
```

Note: Raj is NOT listed as attending. "No decision yet" on Salesforce owner means that topic has NO entry in the decisions list. "EOD Wednesday" is preserved as text in the due_date gap rather than converted to an ISO date without knowing what week's Wednesday. Parking lot item has no owner.

## Grounding (Project+)

Meeting minutes are a communication artifact and an audit record. Project+ treats them as the canonical record of attendance, decisions, and commitments. This skill enforces that the minutes cannot overreach the meeting itself — every recorded decision was actually decided, every action item was actually assigned, every attendee was actually present.
