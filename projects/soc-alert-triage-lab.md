# Project 1: SOC Alert Triage Lab

## Objective
Build a lightweight SOC ticketing + triage system that simulates Tier 1 analyst workflow.

## Scope (MVP)
- Ingest alerts from CSV/JSON
- Create one ticket per alert
- Assign severity (Low/Medium/High/Critical)
- Track status lifecycle: New -> In Progress -> Escalated -> Closed
- Add analyst notes and escalation reason
- Export daily/weekly incident summary report

## Data Model (initial)
- ticket_id
- created_at
- alert_type
- source_ip
- username
- asset
- severity
- status
- analyst_notes
- escalation_reason
- disposition

## 10-Day Plan
1. Repo scaffolding + synthetic alert dataset
2. Ingestion and normalization
3. Severity scoring rules
4. Ticket creation workflow
5. Status transitions and validation
6. Analyst notes and escalation reason support
7. Reporting export (Markdown/CSV)
8. Basic dashboard or CLI summary
9. README polish + screenshots
10. Final QA + interview talking points

## Definition of Done
- [ ] 100+ sample alerts processed
- [ ] Ticket lifecycle implemented
- [ ] Escalation logic documented
- [ ] Report generated from real run
- [ ] Clear README with architecture and usage
