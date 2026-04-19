---
name: decision-log
description: Generate a project decision log as .xlsx — one row per binding decision with ID, title, decision maker, date, context, options considered, decision, rationale, consequences, and review date. Use whenever a user needs a decision log, decision record, ADR, architectural decision record, choice history, or governance trail. Triggers include "decision log", "decision record", "ADR", "decision history", "document this decision", "what did we decide". Enforces never-invent discipline for decision makers, rationale, and options considered.
---

# Decision Log

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Why this skill exists

A decision log is the audit trail of *why* a project looks the way it does. It answers questions like "why did we pick Postgres over DynamoDB" three years after the people who made the call have moved on. Its value collapses if entries are fabricated: inventing rationale changes the record of what was actually considered, and misattributing a decision to the wrong person creates false accountability. Unskilled baselines do exactly these things — they invent "options considered" lists to look thorough, fabricate rationale ("based on performance testing") the user didn't provide, and assign decision makers based on role rather than on who actually made the call. This skill refuses all of that.

## Hard rules

1. **Decision maker is whoever actually decided — not whoever was in the room.** If the user said "Priya approved Option A," the decision maker is Priya. If the user said "we discussed it," the decision maker is `[TBD]`. Do NOT default to PM or sponsor.
2. **Rationale is only what the user stated.** If the user said "we picked Mulesoft because it's what the vendor recommended," rationale is "vendor recommendation." Do NOT elaborate to "vendor recommendation based on enterprise-grade SLA, existing integrations, and total cost of ownership."
3. **Options considered must be real.** If the user only described the chosen option, `options_considered` has ONE entry (the chosen one) and a note "other options not recorded." Do NOT manufacture "Option B: build in-house" or "Option C: do nothing."
4. **Consequences are only what the user stated.** If the user said "this means we need to hire a Mulesoft expert," that's the consequence. Do NOT add "may introduce vendor lock-in" unless the user said it.
5. **Status: Proposed | Accepted | Rejected | Superseded | Deferred.** Default is `Accepted` only if the user said something like "we decided," "approved," "went with." Otherwise `Proposed`.
6. **Review dates are only if stated.** Do not default to "6 months from now."
7. **Never convert informal preferences to recorded decisions.** If the user said "Marcus leans toward Option A," that's NOT a decision — it's a note in the context field. The decision field stays `[TBD]` until the user says a call was made.
8. **Superseded decisions keep their original entry.** If the user says "we changed our mind from Option A to Option B," that creates a new entry that references the old one in `supersedes_id`. The old entry stays with status `Superseded`.
9. **Reporting-line managers never appear — build a ban list.** Scan the user's message for any phrase of the form `"X reports to Y"`, `"Y is X's manager"`, `"under Y"`, `"rolls up to Y"`, or `"Y's report"`. Every name appearing ONLY inside such a phrase goes on a ban list. Names on the ban list are **forbidden** from `decision_maker`, `options_considered`, `decision`, and `consequences` fields. They may appear in `context` only if the reporting relationship itself is load-bearing for the decision narrative. Before rendering, scan every `decision_maker` slot against the ban list.
10. **Fidelity pass.** Every cell in the log traces to a verbatim phrase in the user's message. If you can't cite it, leave the slot empty.

## The reporting-line-to-authority leak (before/after)

User: *"We're still debating whether to do the data archive migration in Phase 1 or Phase 2 — Marcus leans toward Phase 2. No call made yet. Marcus reports to Raj Patel."*

❌ **WRONG:**
```json
{
  "id": "D-003",
  "decision_maker": "Raj Patel",
  "status": "Accepted",
  "decision": "Phase 2",
  "rationale": "Decision authority rests with Raj since Marcus reports to him"
}
```
Every field is wrong. Raj's name appears only in a `"reports to"` phrase → ban-listed. Marcus's lean is not a call. No call was made → status is Proposed, decision is `[TBD]`, rationale is `[TBD]`.

✓ **RIGHT:**
```json
{
  "id": "D-003",
  "decision_maker": "[TBD]",
  "status": "Proposed",
  "decision": "[TBD — no call made yet]",
  "context": "Debating Phase 1 vs Phase 2 for data archive migration. Marcus leans toward Phase 2.",
  "options_considered": [
    {"option": "Phase 1", "chosen": false},
    {"option": "Phase 2", "chosen": false}
  ],
  "rationale": "[TBD]"
}
```

## Run

```bash
python3 scripts/create_decision_log.py --output "<dir>/<ProjectName>_DecisionLog.xlsx" --source /tmp/source.txt <<'JSON'
{ ...schema... }
JSON
```

Pass `--source <path>` with the user's raw message text. The script builds a reporting-line ban list from `"X reports to Y"`-style phrases and flags any decision field that contains a ban-listed name on the Integrity sheet. Omitting `--source` disables the check (acceptable for testing, not acceptable for user deliverables).

Output: .xlsx with one sheet per decision log + an Integrity sheet that surfaces TBD mismatches and reporting-line leaks. TBD cells flagged in orange.

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "project_manager": "string or [TBD]",
    "compiled_date": "YYYY-MM-DD"
  },
  "decisions": [
    {
      "id": "D-001",
      "title": "string — short name for the decision",
      "date": "YYYY-MM-DD or [TBD]",
      "decision_maker": "string or [TBD]",
      "status": "Proposed | Accepted | Rejected | Superseded | Deferred",
      "context": "string — what was the situation, in user's words",
      "options_considered": [
        {"option": "string", "chosen": true}
      ],
      "decision": "string — what was decided",
      "rationale": "string or [TBD]",
      "consequences": "string or [TBD]",
      "supersedes_id": "string or empty",
      "review_date": "YYYY-MM-DD or empty"
    }
  ]
}
```

Notes:
- `options_considered` has at least one entry with `chosen: true` if a decision was made. Additional entries are only those the user explicitly listed.
- `decision_maker` is `[TBD]` if the user didn't name them — never default to PM.
- `status` is `Proposed` unless the user explicitly indicated acceptance.

## Worked example

User: *"Log these decisions from last week on ServiceNow migration. Ken PM. (1) We're using Mulesoft for the Salesforce integration. Priya signed off on this Tuesday. We picked Mulesoft because the vendor recommended it — we didn't really look at alternatives. Consequence: we need to budget $25k for the Mulesoft contract. (2) Training will be 2 weeks instead of 3. Marcus and Taylor agreed to this in our Monday standup. Reason: to hold the cutover date. (3) We're still debating whether to do the data archive migration in Phase 1 or Phase 2 — Marcus leans toward Phase 2. No call made yet. Marcus reports to Raj."*

Correct JSON sketch:
```json
{
  "decisions": [
    {
      "id": "D-001",
      "title": "Use Mulesoft for Salesforce integration",
      "date": "[TBD — user said Tuesday]",
      "decision_maker": "Priya",
      "status": "Accepted",
      "context": "Salesforce integration technology selection",
      "options_considered": [{"option": "Mulesoft", "chosen": true}],
      "decision": "Use Mulesoft for the Salesforce integration",
      "rationale": "Vendor recommendation",
      "consequences": "Need to budget $25k for Mulesoft contract",
      "supersedes_id": "",
      "review_date": ""
    },
    {
      "id": "D-002",
      "title": "Training duration: 2 weeks instead of 3",
      "date": "[TBD — user said Monday standup]",
      "decision_maker": "Marcus and Taylor",
      "status": "Accepted",
      "context": "Training timeline tradeoff against cutover date",
      "options_considered": [{"option": "2 weeks", "chosen": true}],
      "decision": "Training will be 2 weeks instead of 3",
      "rationale": "To hold the cutover date",
      "consequences": "[TBD]",
      "supersedes_id": "",
      "review_date": ""
    },
    {
      "id": "D-003",
      "title": "Data archive migration phase",
      "date": "[TBD]",
      "decision_maker": "[TBD]",
      "status": "Proposed",
      "context": "Debating whether to do data archive migration in Phase 1 or Phase 2. Marcus leans toward Phase 2.",
      "options_considered": [{"option": "Phase 1", "chosen": false}, {"option": "Phase 2", "chosen": false}],
      "decision": "[TBD — no call made yet]",
      "rationale": "[TBD]",
      "consequences": "[TBD]",
      "supersedes_id": "",
      "review_date": ""
    }
  ]
}
```

Note: D-001 rationale is "Vendor recommendation" — NOT elaborated. Options list has only Mulesoft (user said "didn't really look at alternatives"). D-003 is status `Proposed` with decision `[TBD]` because no call was made. Raj is NOT the decision maker anywhere. Marcus's lean is in context, not decision.

## Grounding (Project+)

Decision logs (also called decision records or ADRs) are governance artifacts that preserve the reasoning behind past choices. Project+ treats them as part of the project's historical record, useful both for audits and for onboarding future team members. The skill enforces that recorded decisions reflect what was actually decided, by whom, and why — nothing more.
