---
name: closeout-report
description: Generate a Project+ style project closeout report as .docx — baseline-vs-actual for scope/schedule/cost, deliverables acceptance log, open items and known defects, contract/vendor closure, and formal sign-off block. Use whenever a user needs a closeout report, project closure document, final report, phase-end report, or handover document. Triggers include "closeout report", "project closure", "final report", "handover", "close the project", "wrap-up report". Enforces never-invent discipline for variance numbers, acceptance status, and sign-offs.
---

# Closeout Report

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Why this skill exists

Closeout reports are the final governance artifact — they become the historical record of what a project delivered, for how much, by when, and with whose sign-off. They have the highest fabrication surface of any PM artifact because they invite *computed* fields: "planned $150k vs actual $147k = $3k under budget (2%)." If the user didn't give actuals, unskilled baselines happily make up actuals. If the user didn't say a deliverable was accepted, baselines stamp "Accepted." If the user didn't name sign-off, baselines draw a signature line for the sponsor anyway. This skill refuses all of that structurally.

## Hard rules

1. **Never compute variance from missing data.** If the user gave planned $150k but no actual, variance is `[TBD — actuals not stated]`. Do NOT fill in "assumed $150k" or "on budget."
2. **Never stamp acceptance.** Each deliverable has an acceptance status: `Accepted | Conditionally Accepted | Rejected | Pending | [TBD]`. Default is `[TBD]` unless the user explicitly stated the status. Do NOT default to Accepted because the project is "closing."
3. **Never fabricate sign-offs.** The sign-off block names only people the user specifically identified as sign-offs. Every signatory has a `decision` field: `Signed | Pending | [TBD]`. Drawing an empty signature line for "PM / Sponsor / Customer" by default is a fabrication.
4. **Preserve exact figures.** If the user said "came in at $147,200," the actual cost is $147,200 — not rounded to $147k.
5. **Schedule variance is computed only if both dates given.** Planned end 2026-06-30 vs actual end 2026-07-14 → variance +14 days. If either is missing, variance is `[TBD]`. Do not use "today's date" as a substitute for actual end.
6. **Known defects and open items are separate sections.** Open items are things not finished; known defects are things finished-but-broken. Don't collapse them.
7. **Risk vs Issue discipline carries over.** Closeout reports do not introduce new risks — any risk language refers to residual risks the user explicitly noted for the handover phase.
8. **Reporting-line managers never appear.** If the user says "Marcus reports to Raj," Raj does not appear as a sign-off, deliverable owner, or contact.
9. **Scope delivered is the user's list, not the WBS.** Do not backfill deliverables from an imagined WBS. If the user said "we delivered the dashboard and the training," the scope section has those two items — not five.
10. **Fidelity pass.** Every field traces to a verbatim phrase in the user's message. If you can't cite it, leave the slot empty.

## Run

```bash
python3 scripts/create_closeout_report.py --output "<dir>/<ProjectName>_Closeout.docx" <<'JSON'
{ ...schema... }
JSON
```

Script produces a .docx with fixed section order: Header → Executive Summary → Scope Delivered → Schedule (planned vs actual) → Cost (planned vs actual + variance) → Deliverables & Acceptance → Open Items → Known Defects → Contract/Vendor Closure → Sign-off Block → Appendix (Draft Gaps).

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "project_manager": "string or [TBD]",
    "sponsor": "string or [TBD]",
    "start_date": "YYYY-MM-DD or [TBD]",
    "closeout_date": "YYYY-MM-DD or [TBD]",
    "customer": "string or [TBD]"
  },
  "executive_summary": "string (1-2 sentences) or [TBD]",
  "scope_delivered": ["string — user's verbatim list"],
  "scope_deferred": ["string"],
  "schedule": {
    "planned_start": "YYYY-MM-DD or null",
    "actual_start": "YYYY-MM-DD or null",
    "planned_end": "YYYY-MM-DD or null",
    "actual_end": "YYYY-MM-DD or null"
  },
  "cost": {
    "planned_usd": "number or null",
    "actual_usd": "number or null",
    "currency": "USD"
  },
  "deliverables": [
    {"name": "string", "acceptance_status": "Accepted | Conditionally Accepted | Rejected | Pending | [TBD]", "accepted_by": "string or [TBD]", "acceptance_date": "YYYY-MM-DD or empty"}
  ],
  "open_items": ["string"],
  "known_defects": [
    {"defect": "string", "severity": "Low | Medium | High | [TBD]", "workaround": "string or [TBD]"}
  ],
  "vendor_closure": [
    {"vendor": "string", "contract_status": "Closed | Pending Final Payment | Open | [TBD]", "notes": "string"}
  ],
  "signoffs": [
    {"name": "string", "role": "string", "decision": "Signed | Pending | [TBD]", "date": "YYYY-MM-DD or empty"}
  ]
}
```

Notes:
- `cost.actual_usd` is null unless the user gave an explicit actual. Script computes variance only when both planned AND actual are present.
- `schedule` dates are null unless user stated them.
- `deliverables[].acceptance_status` defaults to `[TBD]`, NEVER `Accepted`.
- `signoffs` is `[]` if the user didn't name any. Do not default-fill PM/Sponsor/Customer.

## Worked example

User: *"Closeout report for ServiceNow Ticket Migration. Ken PM, Priya sponsor, customer is internal IT ops. Start Jan 6 2026, actual end April 10 2026, planned end was April 3. Budget was $150k, actuals came in at $147,200. Delivered: ticket migration for all 8 tracks, training program, and the ETL runbook. Salesforce bidirectional sync deferred to Phase 2. Priya signed off on Monday (April 13). Known defect: workflow rule SNW-42 misfires on tickets over 90 days old, workaround is manual tagging. Mulesoft contract closed, final payment sent."*

Correct JSON sketch:
```json
{
  "scope_delivered": ["Ticket migration for all 8 tracks", "Training program", "ETL runbook"],
  "scope_deferred": ["Salesforce bidirectional sync (deferred to Phase 2)"],
  "schedule": {"planned_start": null, "actual_start": "2026-01-06", "planned_end": "2026-04-03", "actual_end": "2026-04-10"},
  "cost": {"planned_usd": 150000, "actual_usd": 147200, "currency": "USD"},
  "deliverables": [
    {"name": "Ticket migration (8 tracks)", "acceptance_status": "[TBD]", "accepted_by": "[TBD]", "acceptance_date": ""},
    {"name": "Training program", "acceptance_status": "[TBD]", "accepted_by": "[TBD]", "acceptance_date": ""},
    {"name": "ETL runbook", "acceptance_status": "[TBD]", "accepted_by": "[TBD]", "acceptance_date": ""}
  ],
  "known_defects": [
    {"defect": "Workflow rule SNW-42 misfires on tickets over 90 days old", "severity": "[TBD]", "workaround": "Manual tagging"}
  ],
  "vendor_closure": [{"vendor": "Mulesoft", "contract_status": "Closed", "notes": "Final payment sent"}],
  "signoffs": [{"name": "Priya Patel", "role": "Sponsor", "decision": "Signed", "date": "2026-04-13"}]
}
```

Note: Schedule variance is +7 days (computed from given dates). Cost variance is −$2,800 (computed from given figures). Deliverables acceptance is `[TBD]` even though the project is closing — user said Priya signed off, but didn't say who accepted each deliverable. Only Priya is in the sign-off block; no fabricated PM/Customer signatories.

## Grounding (Project+)

Closeout reports formally release project resources and hand deliverables to operations. Project+ emphasizes the baseline-vs-actual discipline (scope/schedule/cost) and formal customer acceptance as the trigger for project closure. This skill enforces that the artifact cannot claim closure conditions that didn't actually occur — the sign-off block is empty until a real signatory exists, acceptance is TBD until explicitly granted, and variance calculations refuse to run on fabricated data.
