---
name: lessons-learned
description: Generate a Project+ style lessons-learned register as .docx — one entry per lesson with category, what happened, root cause, impact, recommendation, and owner. Use whenever a user needs a lessons-learned document, retrospective write-up, post-mortem, project closeout learnings, or phase-gate retrospective. Triggers include "lessons learned", "retro", "retrospective", "post-mortem", "closeout learnings", "what went well", "what went wrong". Enforces never-invent discipline for root causes, never fabricates positive lessons to balance negatives, and preserves the user's exact phrasing.
---

# Lessons Learned

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Why this skill exists

Lessons-learned registers are the organizational memory of a project — they only work if they're **honest**. Unskilled baselines do two specific damaging things: (1) they manufacture root causes that sound plausible but weren't in the evidence ("vendor was slow" → "vendor capacity constraints due to Q4 demand cycle"), and (2) they invent "What went well" lessons to balance negatives the user described. Both of these poison the knowledge base. This skill refuses both moves structurally.

## Hard rules

1. **Never invent a root cause.** If the user described a symptom ("the Salesforce integration slipped 3 weeks"), the root cause field is only populated if the user stated it. Otherwise it's `[TBD — root cause not stated]`. Do NOT infer.
2. **Never balance negatives with fabricated positives.** If the user gave 4 things that went wrong and 0 things that went right, the register has 4 "Change" lessons and 0 "Keep" lessons. Do not manufacture "Keep doing: strong team morale" to round out the list.
3. **Preserve exact user wording.** The `what_happened` field is the user's phrasing. Don't editorialize "training was rushed" into "insufficient training investment due to compressed timeline."
4. **Category is one of 7.** `Process | Technical | People | Vendor | Scope | Schedule | Cost`. If the category isn't clear from the user's message, mark it `[TBD]`. Do not guess.
5. **Disposition: Keep or Change.** Every lesson is either a `Keep` (do this again on future projects) or a `Change` (do differently). Map from the user's tone: "this worked" → Keep; "this hurt us" → Change. If ambiguous, `[TBD]`.
6. **Recommendations must be concrete if present.** If the user said "we should improve vendor onboarding," that's the recommendation verbatim. If the user only described the problem without recommending an action, recommendation is `[TBD]`. Do NOT extrapolate.
7. **Owner is not the PM by default.** If the user didn't name who should own a recommendation, owner is `[TBD]`. Do not auto-assign to the PM.
8. **Reporting-line managers never appear.** If the user says "Marcus reports to Raj," Raj does not appear as a lesson owner, named cause, or recommendation target.
9. **No artificial severity scores.** Project+ lessons-learned do not require a severity or priority number. Unless the user explicitly gave one, the field is empty — do not score lessons 1–5 yourself.
10. **Fidelity pass.** Every lesson field must trace to a verbatim phrase in the user's message. If you can't cite it, leave the slot empty.

## Run

```bash
python3 scripts/create_lessons_learned.py --output "<dir>/<ProjectName>_LessonsLearned.docx" <<'JSON'
{ ...schema... }
JSON
```

Script produces a .docx with fixed section order: Header → Summary Counts (Keep vs Change, by category) → Keep Doing → Change Next Time → Appendix (Draft Gaps).

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "project_manager": "string or [TBD]",
    "sponsor": "string or [TBD]",
    "phase": "Closeout | Phase 1 gate | Phase 2 gate | [TBD]",
    "date_compiled": "YYYY-MM-DD",
    "contributors": ["string — names user provided"]
  },
  "lessons": [
    {
      "id": "L-01",
      "category": "Process | Technical | People | Vendor | Scope | Schedule | Cost | [TBD]",
      "disposition": "Keep | Change | [TBD]",
      "what_happened": "string — user's verbatim words",
      "root_cause": "string or [TBD]",
      "impact": "string or [TBD]",
      "recommendation": "string or [TBD]",
      "owner": "string or [TBD]"
    }
  ]
}
```

Notes:
- `lessons` is ordered by the user's mention order — don't reorder by category or severity.
- Empty arrays are valid. A register with 2 Change lessons and 0 Keep lessons is honest; a register with 2 Change and 2 fabricated Keep lessons is dishonest.
- `contributors` is who attended/contributed to the retro — not the cast list of the project.

## Worked example

User: *"Lessons learned for ServiceNow Ticket Migration. Ken PM, Priya sponsor. Closeout phase. Three things: (1) We should have scoped the Salesforce integration in Phase 1 instead of adding it mid-project — cost us 3 weeks and a change request. (2) Jen's ETL script pattern worked great, we should reuse it on future migrations. (3) Training timeline was too tight — 2 weeks wasn't enough for the volume. Marcus, Jen, Taylor attended the retro. Marcus reports to Raj."*

Correct JSON:
```json
{
  "header": {
    "project_name": "ServiceNow Ticket Migration",
    "project_manager": "Ken",
    "sponsor": "Priya",
    "phase": "Closeout",
    "date_compiled": "[TBD]",
    "contributors": ["Marcus", "Jen", "Taylor"]
  },
  "lessons": [
    {
      "id": "L-01",
      "category": "Scope",
      "disposition": "Change",
      "what_happened": "Salesforce integration was added mid-project instead of scoped in Phase 1",
      "root_cause": "[TBD — root cause not stated]",
      "impact": "Cost 3 weeks and a change request",
      "recommendation": "Scope integrations in Phase 1 rather than adding mid-project",
      "owner": "[TBD]"
    },
    {
      "id": "L-02",
      "category": "Technical",
      "disposition": "Keep",
      "what_happened": "Jen's ETL script pattern worked great",
      "root_cause": "[TBD]",
      "impact": "[TBD]",
      "recommendation": "Reuse the ETL script pattern on future migrations",
      "owner": "[TBD]"
    },
    {
      "id": "L-03",
      "category": "Schedule",
      "disposition": "Change",
      "what_happened": "Training timeline of 2 weeks was too tight for the volume",
      "root_cause": "[TBD]",
      "impact": "[TBD]",
      "recommendation": "[TBD]",
      "owner": "[TBD]"
    }
  ]
}
```

Note: 2 Change, 1 Keep — no fabricated balance. Raj does NOT appear. "Cost us 3 weeks and a change request" is preserved verbatim in impact. No inferred root causes.

## Grounding (Project+)

Lessons learned are a closeout-phase deliverable feeding the organizational knowledge base. Project+ treats them as factual records of what happened and what to do differently — not as sentiment reports. The value is in the signal-to-noise ratio of the register: future PMs searching "vendor issues on migration projects" should find real patterns, not fabricated filler. This skill enforces that honesty structurally.
