# PM Skills

15 Project+ skills for technical project managers. Each skill generates a production-ready artifact (.docx, .xlsx, or .pptx) from natural language input, enforcing never-invent discipline throughout: no fabricated names, dates, metrics, stakeholders, or rationale.

## Skills

| Skill | Artifact | Format |
|---|---|---|
| project-charter | Project charter + preliminary scope statement | .docx |
| kickoff-deck | Project kickoff presentation | .pptx |
| team-roster | Project team roster | .xlsx |
| raci-matrix | Responsibility assignment matrix | .xlsx |
| stakeholder-engagement-plan | Stakeholder register, power/interest grid, comms plan | .docx |
| risk-register | Risk register with RAG ratings | .xlsx |
| status-report | Weekly/biweekly status report | .docx |
| project-schedule | Project schedule with milestones | .xlsx |
| budget-plan | Project budget breakdown | .xlsx |
| wbs | Work breakdown structure | .xlsx |
| meeting-minutes | Structured meeting minutes | .docx |
| decision-log | Decision record / ADR log | .xlsx |
| change-request | Change request with CCB governance | .docx |
| lessons-learned | Lessons learned register | .docx |
| closeout-report | Project closeout report | .docx |

## How it works

Each skill contains a SKILL.md with hard rules that prevent common LLM failure modes in PM artifacts, plus a Python script that renders the structured output. You describe a project situation in plain language; the skill parses your input into a JSON schema, applies fidelity checks, and generates the formatted document.

## Key protections

These skills enforce discipline that stock LLMs consistently violate:

- **Never-invent rule**: unknown fields get `[TBD]` placeholders, not fabricated content
- **Reporting-line manager trap**: "Marcus reports to Raj" does not make Raj a decision maker, team member, or stakeholder
- **Time-wording preservation**: "October" stays "October" — never narrowed to "October 31"
- **Controlled vocabulary**: engagement levels, risk statuses, and dispositions use Project+ standard terms only
- **No fabricated balance**: if the user gave 4 negatives and 0 positives, the output has 4 negatives and 0 positives
- **Figure linting**: scripts flag any number, date, or SLA in the draft that doesn't appear in the user's original message

## Requirements

- Python 3.9+
- `python-docx` (for .docx skills)
- `openpyxl` (for .xlsx skills)
- `python-pptx` (for .pptx skills)

Install: `pip install python-docx openpyxl python-pptx`

## Author

Ken Mulford — [ken@kenmulford.com](mailto:ken@kenmulford.com)
