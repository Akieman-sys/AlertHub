# Incident Model v1 — AlertHub

## 1. Objective
The v1 incident model defines the core actionable object created and managed by the system after grouping one or more raw events that represent the same underlying problem.

An incident is not a raw signal. It is a tracked operational problem with state, ownership, timestamps, and evidence.

## 2. Design principles
- incident-centric, not raw-event-centric
- optimized for action, tracking, and proof
- simple lifecycle for v1
- compatible with deduplication and SLA logic
- readable by humans and usable by machines

## 3. Standard JSON v1

```json
{
  "incident_id": "inc_01JABCXYZ001",
  "org_id": "org_agency_001",
  "client_id": "client_store_007",
  "fingerprint": "fp_e3b0c44298fc",
  "source_tool": "n8n",
  "workflow_id": "wf_orders_sync",
  "workflow_name": "Orders Sync",
  "node_name": "HTTP Request",
  "event_type": "technical_error",
  "severity": "P1",
  "status": "open",
  "title": "ERP sync failed",
  "message": "ERP API returned status 502",
  "first_seen": "2026-03-06T09:40:00Z",
  "last_seen": "2026-03-06T09:44:00Z",
  "occurrences": 4,
  "owner_user_id": null,
  "sla_policy_id": "sla_default_p1",
  "notification_channels": ["slack", "email"],
  "links": {
    "latest_run_url": "https://n8n.example.com/executions/run_001"
  },
  "created_at": "2026-03-06T09:40:03Z",
  "updated_at": "2026-03-06T09:44:01Z",
  "resolved_at": null
}
```

## 4. Required fields

### `incident_id`
Unique identifier of the incident.

### `org_id`
Identifier of the agency owning the incident context.

### `client_id`
Identifier of the end client concerned by the incident.

### `fingerprint`
Deduplication identity representing the underlying grouped problem.

### `source_tool`
Source tool associated with the incident.
Expected v1 value: `n8n`.

### `workflow_id`
Stable technical identifier of the workflow.

### `workflow_name`
Human-readable name of the workflow.

### `event_type`
Primary type of event represented by the incident.

### `severity`
Current severity of the incident:
- `P0`
- `P1`
- `P2`

### `status`
Current incident status:
- `open`
- `acknowledged`
- `resolved`

### `title`
Short human-readable summary of the incident.

### `message`
Primary human-readable incident message.

### `first_seen`
Timestamp of the earliest event grouped into the incident.

### `last_seen`
Timestamp of the latest event grouped into the incident.

### `occurrences`
Number of grouped event occurrences.

### `created_at`
Timestamp when the incident record was created in the system.

### `updated_at`
Timestamp when the incident record was last updated.

## 5. Optional fields

### `node_name`
Name of the node or component associated with the incident.

### `owner_user_id`
Assigned user currently responsible for the incident.

### `sla_policy_id`
Identifier of the SLA policy applied to the incident.

### `notification_channels`
Channels used or configured for notification delivery.
Examples:
- `slack`
- `email`

### `links`
Useful links associated with the incident.
Examples:
- latest run URL
- workflow URL
- documentation URL

### `resolved_at`
Timestamp set when the incident is resolved.

## 6. Business rules v1
- one incident represents one grouped operational problem
- an incident may contain one or many events
- all grouped events within one incident must share the same fingerprint
- `occurrences` must be incremented when matching events are grouped into the incident
- `first_seen` must reflect the earliest grouped event timestamp
- `last_seen` must reflect the latest grouped event timestamp
- `status` must move only through the allowed lifecycle: `open` → `acknowledged` → `resolved`
- `resolved_at` must be null unless status is `resolved`
- an incident may exist without an assigned owner in v1

## 7. Lifecycle v1

### `open`
Incident exists and has not yet been formally acknowledged.

### `acknowledged`
A user has seen the incident and taken responsibility for handling it.

### `resolved`
The incident is considered handled and closed.

## 8. Examples

### Example A — open incident
```json
{
  "incident_id": "inc_001",
  "org_id": "org_agency_001",
  "client_id": "client_store_007",
  "fingerprint": "fp_orders_http_502",
  "source_tool": "n8n",
  "workflow_id": "wf_orders_sync",
  "workflow_name": "Orders Sync",
  "node_name": "HTTP Request",
  "event_type": "technical_error",
  "severity": "P1",
  "status": "open",
  "title": "ERP sync failed",
  "message": "ERP API returned status 502",
  "first_seen": "2026-03-06T09:40:00Z",
  "last_seen": "2026-03-06T09:44:00Z",
  "occurrences": 4,
  "created_at": "2026-03-06T09:40:03Z",
  "updated_at": "2026-03-06T09:44:01Z"
}
```

### Example B — resolved incident
```json
{
  "incident_id": "inc_002",
  "org_id": "org_agency_001",
  "client_id": "client_school_002",
  "fingerprint": "fp_missing_phone_field",
  "source_tool": "n8n",
  "workflow_id": "wf_lead_capture",
  "workflow_name": "Lead Capture",
  "node_name": "Validate Input",
  "event_type": "business_error",
  "severity": "P0",
  "status": "resolved",
  "title": "Missing required field",
  "message": "Field phone_number is missing from incoming lead",
  "first_seen": "2026-03-06T09:45:00Z",
  "last_seen": "2026-03-06T09:45:00Z",
  "occurrences": 1,
  "owner_user_id": "user_ops_001",
  "created_at": "2026-03-06T09:45:02Z",
  "updated_at": "2026-03-06T10:02:10Z",
  "resolved_at": "2026-03-06T10:02:10Z"
}
```

## 9. Out of scope for v1
The v1 incident model does not yet handle:
- mitigated state
- parent/child incidents
- root cause classification
- postmortem objects
- linked ticketing systems
- automatic runbook recommendations