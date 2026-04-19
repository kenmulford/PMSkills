---
name: change-request
description: Generate a Project+ style change request as .docx — request metadata, description, impact analysis across scope/schedule/cost/quality/resources/risk, options considered, recommendation, and CCB approval block. Use whenever a user needs a change request, CR, change control form, scope change document, RFC, or baseline change. Triggers include "change request", "CR", "change control", "scope change", "baseline change", "RFC". Enforces never-invent discipline for impact numbers, CCB members, and disposition — and preserves the user's fuzzy estimates exactly.
---

# Change Request

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Why this skill exists

Change requests are high-stakes governance artifacts. Baselines of scope/schedule/cost only move through a formal CR, so the artifact needs to be **auditable**: every impact number traceable to the requestor, every CCB member named by the user, and disposition left blank until the user says a decision was made. Unskilled baselines fabricate with enthusiasm here — they convert "maybe a week" to "5 business days," invent CCB rosters ("PM, Sponsor, Tech Lead"), and auto-stamp "Approved" on a request that was merely *submitted*. This skill refuses all of that.

## Hard rules

1. **Never invent impact numbers.** If the user said "maybe a week" or "a few thousand dollars," that stays as the verbatim phrase with a `[TBD — user said "maybe a week"]` marker in the structured slot. Do NOT convert to 5 days or $3,000.
2. **Never invent CCB members.** The Change Control Board roster is whoever the user named. If the user said "went to the CCB," the member list is `[TBD — CCB roster not stated]`. Do not default-fill "PM, Sponsor, Tech Lead."
3. **Disposition defaults to Submitted.** Valid dispositions: `Submitted | Under Review | Approved | Rejected | Deferred | Withdrawn`. Unless the user explicitly stated a decision, disposition is `Submitted` and the approval block is empty. Never stamp Approved based on tone.
4. **Requestor is not the PM by default.** If the user says "Marcus is asking for …," Marcus is the requestor. The PM field is only populated if the user names the PM separately.
5. **Baseline version only moves on approval.** If disposition is Approved, the script increments the baseline tag (e.g., v1.0 → v1.1). If disposition is anything else, the baseline version is unchanged and a note says "No baseline change — request [status]."
6. **Risk vs Issue discipline carries over.** If the user describes something currently biting the project as justification for the CR, it's an **issue** in the context section, not a risk introduced by the change.
7. **Preserve fuzzy quantities.** "2-3 weeks," "~$10k," "a couple engineers" stay as exact strings in a `user_estimate` field. The structured numeric fields (`schedule_impact_days`, `cost_impact_usd`) stay null/TBD until the user gives a single value.
8. **Reporting-line managers never appear.** If the user says "Marcus reports to Raj," Raj does not appear in CCB, approvals, or impact sections. Full stop.
9. **Options considered is optional but honest.** If the user only described the one change, `options_considered` is an empty array. Do not manufacture a fake "Option B: do nothing" unless the user said it.
10. **Fidelity pass.** Every field — description, impact bullet, CCB member, option — must trace to a verbatim phrase in the user's message. If you can't cite it, leave the slot empty.

## Run

```bash
python3 scripts/create_change_request.py --output "<dir>/<ProjectName>_CR_<id>.docx" <<'JSON'
{ ...schema... }
JSON
```

Script produces a .docx with fixed section order: Header → Request Summary → Description & Justification → Impact Analysis (scope/schedule/cost/resources/quality/risk) → Options Considered → Recommendation → CCB / Approvals → Disposition & Baseline Version → Appendix (Draft Gaps).

## JSON schema

```json
{
  "header": {
    "cr_id": "string (e.g., CR-0007)",
    "cr_title": "string",
    "project_name": "string",
    "requestor": "string",
    "project_manager": "string or [TBD]",
    "sponsor": "string or [TBD]",
    "date_submitted": "YYYY-MM-DD",
    "current_baseline_version": "string (e.g., v1.0)"
  },
  "category": "Scope | Schedule | Cost | Quality | Resource | [TBD]",
  "priority": "Low | Medium | High | Critical | [TBD]",
  "description": "string — what is being requested, in user's words",
  "justification": "string — why, in user's words",
  "impact": {
    "scope": "string or empty",
    "schedule_impact_days": "integer or null",
    "schedule_user_estimate": "string or empty (verbatim fuzzy phrase)",
    "cost_impact_usd": "number or null",
    "cost_user_estimate": "string or empty (verbatim fuzzy phrase)",
    "resources": "string or empty",
    "quality": "string or empty",
    "risk": "string or empty"
  },
  "options_considered": [
    {"option": "string", "pros": "string", "cons": "string"}
  ],
  "recommendation": "string or [TBD]",
  "ccb_members": ["string — names user provided"],
  "approvals": [
    {"name": "string", "role": "string", "decision": "Approve | Reject | Defer | Pending", "date": "YYYY-MM-DD or empty"}
  ],
  "disposition": "Submitted | Under Review | Approved | Rejected | Deferred | Withdrawn"
}
```

Notes:
- `schedule_impact_days` is an integer ONLY if the user gave a precise number of days. Otherwise leave null and put the verbatim phrase in `schedule_user_estimate`.
- `cost_impact_usd` same rule — null unless the user gave a specific dollar figure.
- `approvals` is empty unless the user said "X approved" or "Y rejected." Listing CCB *members* is separate from listing *decisions*.
- If `disposition == "Approved"` the script increments the baseline version and records the new version; otherwise the baseline stays put.

## Worked example

User: *"Need a change request for Project Aurora. Marcus is asking to add mobile push notifications to the MVP. He says it'll take maybe 2 weeks of Jen's time and probably a few thousand dollars in Firebase costs. We haven't gone to the CCB yet — just drafting. Ken PM, Mei Tanaka sponsor. Current baseline v1.2. Marcus reports to Raj."*

Correct JSON:
```json
{
  "header": {
    "cr_id": "[TBD — no CR ID stated]",
    "cr_title": "Add mobile push notifications to MVP",
    "project_name": "Project Aurora",
    "requestor": "Marcus",
    "project_manager": "Ken",
    "sponsor": "Mei Tanaka",
    "date_submitted": "[TBD]",
    "current_baseline_version": "v1.2"
  },
  "category": "Scope",
  "priority": "[TBD]",
  "description": "Add mobile push notifications to the MVP",
  "justification": "[TBD — requestor did not state justification]",
  "impact": {
    "scope": "Adds mobile push notification feature to MVP",
    "schedule_impact_days": null,
    "schedule_user_estimate": "maybe 2 weeks of Jen's time",
    "cost_impact_usd": null,
    "cost_user_estimate": "probably a few thousand dollars in Firebase costs",
    "resources": "Jen",
    "quality": "",
    "risk": ""
  },
  "options_considered": [],
  "recommendation": "[TBD]",
  "ccb_members": [],
  "approvals": [],
  "disposition": "Submitted"
}
```

Note: Raj is NOT in CCB or approvals. Jen is a resource (named by requestor). "2 weeks" stays as text — NOT converted to `schedule_impact_days: 10`. "A few thousand dollars" stays as text — NOT converted to `cost_impact_usd: 3000`. Disposition is `Submitted` because the user said "haven't gone to the CCB yet." Baseline version does NOT increment.

## Grounding (Project+)

Change control is the formal process for altering a baseline (scope, schedule, cost, or quality). The Change Control Board (CCB) is the governance body that dispositions each request. Project+ emphasizes that change requests preserve baseline integrity: nothing moves until a formal disposition is recorded. This skill enforces that structurally — the baseline version field is bound to the disposition field, so the artifact itself cannot claim a baseline shift that wasn't approved.
