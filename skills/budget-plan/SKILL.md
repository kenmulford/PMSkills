---
name: budget-plan
description: Generate a Project+ style project budget as .xlsx — line items by category with committed amounts, actuals, variance, approved total, contingency, and optional monthly burn. Use whenever a user needs to capture or review project costs — triggers include "budget", "cost plan", "cost baseline", "approved budget", "contingency reserve", "burn plan", "variance". Enforces never-invent discipline: no fabricated costs, categories, owners, contingency percentages, or reconciling "plug" lines; fuzzy amounts stay fuzzy.
---

# Project Budget

Draft-first: parse what the user gave, fill the schema, run the script. Only interview if the user explicitly asks.

## Hard rules

1. **Never invent costs.** Any line item, amount, vendor, or category the user did not state → `[TBD — reason]`. Fabricated budgets get approved and then overrun.
2. **No plug lines.** Do not create a "rounding" or "unallocated" row to make totals reconcile. If committed ≠ approved, the Variance cell tells the truth; don't paper over it.
3. **Preserve fuzzy amounts.** "~$20k/month" → `[TBD — user said ~$20k/mo]`, NOT `$20,000`. "Roughly $15k" → `[TBD — user said ~$15k]`. Fuzzy words upstream become anchored numbers downstream; stop the fabrication at the JSON.
4. **Never invent a contingency percentage.** If the user says "contingency TBD" or doesn't mention it, leave contingency as a TBD row. If the user says "10%", compute it; otherwise the script won't.
5. **No auto-assigned owners.** The PM is not the default owner of any line item. If the user didn't name an owner → `[TBD]`.
6. **Exact user wording** for line-item descriptions. "ServiceNow licenses" stays "ServiceNow licenses," not "SaaS Software Subscription (ServiceNow Platform)."
7. **Approved total is what the user stated.** If the user said "approved $150k," that's the approved cell. If they said nothing, approved is `[TBD]` and variance is `[TBD — no approved baseline]`.
8. **Preserve the user's labor math.** If the user gives an allocation (e.g., "50% for 2 months") AND a dollar figure, use the dollar figure. Do not recompute it from hourly rates. If the user gives only the allocation and no dollar figure → `[TBD]`.
9. **Fidelity pass before running.** For every amount, category, owner, or percentage in your JSON, cite a verbatim phrase in the user's message. If you can't → `[TBD]`.

## Run

```bash
python3 scripts/create_budget.py --output "<dir>/<ProjectName>_Budget_v1.xlsx" <<'JSON'
{ ...schema... }
JSON
```

Script handles total roll-up, variance vs approved, contingency math (only if percent is numeric), TBD highlighting, Draft Gaps, Integrity Issues, optional Monthly Burn sheet, and `open_questions.md` when gaps ≥ 5.

## JSON schema

```json
{
  "header": {
    "project_name": "string",
    "project_manager": "string or [TBD]",
    "sponsor": "string or [TBD]",
    "fiscal_year": "string or [TBD]",
    "approved_total": "number or [TBD — reason]",
    "contingency_percent": "number or [TBD — reason]",
    "date": "YYYY-MM-DD",
    "version": "v1"
  },
  "line_items": [
    {
      "id": "L001",
      "category": "Labor | Software | Vendor | Hardware | Travel | Contingency | Other",
      "description": "string — user's words",
      "amount": "number or [TBD — reason]",
      "vendor": "string or [TBD] or empty",
      "owner": "string or [TBD]",
      "status": "Committed | Planned | Contracted | Unfunded | TBD",
      "notes": "string or empty"
    }
  ],
  "monthly_burn": [
    {"month": "2026-06", "amount": "number or [TBD]"}
  ]
}
```

Notes:
- `amount` must be a number (dollars) or a `[TBD — reason]` string. Do not store strings like "$20k/mo"; use TBD with the user's verbatim phrase as the reason.
- `contingency_percent` — only a number if the user stated a percentage. Otherwise TBD. The script does NOT default to 10%.
- `monthly_burn` is optional; include only if the user gave a burn plan.
- `status = Unfunded` for items the user acknowledged but has no money against yet.

## Worked example

User: *"Budget: ServiceNow licenses $60k (vendor quote). Marcus 50%/2 months $18k. Slack integration TBD. Contingency 10% of $150k approved."*

Correct:
```
L001  Software  ServiceNow licenses   60000     Status: Contracted, vendor: ServiceNow
L002  Labor     Marcus 50%/2 months   18000     Owner: Marcus Chen, Status: Committed
L003  Vendor    Slack integration     [TBD]     Status: Unfunded
L004  Contingency  10% of approved   15000     (script computes from contingency_percent=10)
```
Approved total: 150000. Do NOT invent a reconciling line to force 150k.

User: *"Two backend engineers TBD at roughly $20k/month each for 3 months."*

Correct:
```
L005  Labor   Backend engineer 1 (~$20k/mo × 3mo)   [TBD — user said ~$20k/mo]
L006  Labor   Backend engineer 2 (~$20k/mo × 3mo)   [TBD — user said ~$20k/mo]
```
Do NOT encode as 60000 each. The "roughly" is load-bearing.

## Grounding (Project+)

Project budget = time-phased cost baseline approved by the sponsor. Components: **direct costs** (labor, materials, vendors), **indirect costs** (overhead), **contingency reserve** (for known risks), **management reserve** (for unknown-unknowns, typically not in the baseline). **Variance** = approved − committed (or actual). **Cost baseline** is frozen at approval; changes go through change control. This skill produces the *planning* artifact — the baseline that gets approved.
