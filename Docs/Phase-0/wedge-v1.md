# Wedge v1 — AlertHub

## 1. Chosen wedge
The v1 wedge is:

**n8n Error Workflow + Business Alerts + Multi-tenant Agency Console**

## 2. What this means
The first version of the product is designed for automation agencies that use n8n for multiple clients.

The system receives events sent by n8n workflows:
- technical errors
- business errors
- simple warnings
- useful health/execution information

Then it:
- normalizes events
- deduplicates similar errors
- creates or updates an incident
- allows ack / assign / resolve
- sends Slack and email notifications
- displays incidents in a multi-client console

## 3. v1 input
Main input:
- n8n events sent to an `/ingest` endpoint

Targeted event types:
- technical_error
- business_error
- warning
- info

## 4. v1 output
The product must produce:
- deduplicated incidents
- incident timeline
- open / acknowledged / resolved status
- simple assignment
- Slack / email notifications
- global multi-client view
- basic CSV export
- simple proof of handling

## 5. Why this wedge
- clear initial segment
- simpler initial integration
- real business pain
- strong differentiation compared with “Slack on error”
- solid foundation for future expansion

## 6. What the wedge does not include
Not included in v1:
- Make
- Zapier
- Pipedream
- advanced on-call rotation
- PagerDuty / Opsgenie
- complex heartbeats
- AI auto-triage
- automatic runbooks
- enterprise SSO
- sophisticated PDF reporting
- multiple integrations

## 7. v1 positioning
Automation Incident OS starts as an n8n-first system for agencies, focused on turning technical and business events into deduplicated, actionable, and provable incidents inside a multi-client console.