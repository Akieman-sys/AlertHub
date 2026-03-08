# Event Model v1 — AlertHub

## 1. Objective
The v1 standard event model defines the minimum format accepted by the system to represent a signal coming from a monitored workflow.

An event is a raw fact. It can then be transformed by the incident engine into a deduplicated and actionable incident.

## 2. Design principles
- simple and stable format
- n8n-first but not n8n-locked
- sufficient for deduplication, routing, audit, and debugging
- readable by humans and usable by machines
- extensible without breaking v1

## 3. Standard JSON v1

```json
{
  "event_id": "evt_01HXYZABC123",
  "org_id": "org_agency_001",
  "client_id": "client_abc_001",
  "source_tool": "n8n",
  "workflow_id": "wf_12345",
  "workflow_name": "Lead Intake Pipeline",
  "run_id": "run_98765",
  "node_name": "HTTP Request",
  "event_type": "technical_error",
  "severity": "P1",
  "title": "HTTP request failed",
  "message": "Request to CRM API returned status 500",
  "raw_payload": {
    "status_code": 500,
    "provider": "hubspot",
    "error": "Internal Server Error"
  },
  "ts": "2026-03-06T10:15:42Z"
}
```

## 4. Required fields

### `event_id`

Unique identifier of the event.
Mandatory in v1.
Used for idempotency and traceability.

### `org_id`

Identifier of the main organization using the SaaS.
In v1, this corresponds to the agency.

### `client_id`

Identifier of the end client managed by the agency.

### `source_tool`

Source tool that produced the event.
Expected v1 value: `n8n`.

### `workflow_id`

Stable technical identifier of the workflow.

### `workflow_name`

Human-readable name of the workflow.

### `run_id`

Identifier of the workflow execution concerned by the event.

### `event_type`

Accepted event types in v1:

* `technical_error`
* `business_error`
* `warning`
* `info`

### `severity`

Declared severity of the event:

* `P0`
* `P1`
* `P2`

### `title`

Short and readable summary.

### `message`

Main human-readable message describing the issue or information.

### `ts`

ISO 8601 timestamp of the event.

## 5. Optional fields

### `node_name`

Name of the n8n node or component that produced the event.

### `raw_payload`

Raw payload or useful fragment of technical or business context.
Optional to avoid very heavy or sensitive payloads.

## 6. Business rules v1

* every event must have a unique `event_id`
* every event must belong to an organization and a client
* every event must reference a workflow
* every event must have a type and a severity
* every event must carry a timestamp
* `message` must be useful to a human, not only to a machine
* `raw_payload` must not contain secrets in plain text
* retries sending the same event must reuse the same `event_id`

## 7. Event types v1

### `technical_error`

Technical failure, blocking or non-blocking.
Examples:

* timeout
* authentication failure
* HTTP 500
* DNS or network failure

### `business_error`

Business or data quality issue.
Examples:

* invalid file
* missing field
* no expected data found
* violated business rule

### `warning`

Low-level signal or non-critical degradation.

### `info`

Informational or simple health event.

## 8. Examples

### Example A — technical error

```json
{
  "event_id": "evt_tech_001",
  "org_id": "org_agency_001",
  "client_id": "client_store_007",
  "source_tool": "n8n",
  "workflow_id": "wf_orders_sync",
  "workflow_name": "Orders Sync",
  "run_id": "run_001",
  "node_name": "HTTP Request",
  "event_type": "technical_error",
  "severity": "P1",
  "title": "ERP sync failed",
  "message": "ERP API returned status 502",
  "ts": "2026-03-06T09:40:00Z"
}
```

### Example B — business error

```json
{
  "event_id": "evt_biz_001",
  "org_id": "org_agency_001",
  "client_id": "client_school_002",
  "source_tool": "n8n",
  "workflow_id": "wf_lead_capture",
  "workflow_name": "Lead Capture",
  "run_id": "run_002",
  "node_name": "Validate Input",
  "event_type": "business_error",
  "severity": "P0",
  "title": "Missing required field",
  "message": "Field phone_number is missing from incoming lead",
  "ts": "2026-03-06T09:45:00Z"
}
```

## 9. Out of scope for v1

The v1 event model does not yet handle:

* Make / Zapier / Pipedream-specific schemas
* binary attachments
* distributed tracing
* advanced throughput metrics
* AI-based automatic classification
