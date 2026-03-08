# Fingerprint v1 — AlertHub

## 1. Objective
The v1 fingerprint defines how the system decides whether multiple incoming events belong to the same underlying operational problem and should therefore be grouped into a single incident.

The fingerprint is the deduplication identity of the problem, not the identity of an individual event.

## 2. Design principles
- stable across retries and repeated executions
- resistant to noisy variable data
- strict enough to avoid merging unrelated problems
- simple enough for v1 implementation
- compatible with future normalization improvements

## 3. Fingerprint formula v1

```text
fingerprint_input =
  org_id
  + client_id
  + source_tool
  + workflow_id
  + node_name
  + event_type
  + normalized_error_code
  + normalized_message
```

The final fingerprint is produced by hashing the fingerprint input.

```text
fingerprint = hash(fingerprint_input)
```

## 4. Included fields

### `org_id`
Prevents grouping problems across different agencies.

### `client_id`
Prevents grouping problems across different end clients.

### `source_tool`
Separates problems by source tool family.

### `workflow_id`
Keeps grouping scoped to a specific workflow.

### `node_name`
Helps distinguish failures coming from different workflow components.

### `event_type`
Separates technical errors, business errors, warnings, and informational signals.

### `normalized_error_code`
Represents a stable normalized error category when available.
Examples:
- `HTTP_5XX`
- `HTTP_4XX`
- `AUTH_ERROR`
- `TIMEOUT`
- `RATE_LIMIT`
- `MISSING_FIELD`
- `INVALID_FILE_TYPE`

### `normalized_message`
Normalized human-readable message with noisy variable fragments removed.

## 5. Excluded fields

### `event_id`
Must not be part of the fingerprint because it identifies a single event, not the grouped problem.

### `run_id`
Must not be part of the fingerprint because each execution may differ while still representing the same recurring problem.

### `ts`
Must not be part of the fingerprint because timestamps vary across repeated occurrences of the same issue.

### `raw_payload`
Must not be used directly as fingerprint input in v1 because it is too noisy, too large, and often unstable.

## 6. Normalization rules v1

### Rule 1 — remove timestamps
Timestamps must be removed from normalized message content.

### Rule 2 — remove random or execution-specific IDs
Execution IDs, request IDs, UUIDs, trace IDs, and similar variable identifiers must be removed.

### Rule 3 — normalize HTTP errors into buckets
Examples:
- `500`, `502`, `503` may become `HTTP_5XX`
- `400`, `401`, `403`, `404` may become `HTTP_4XX`

### Rule 4 — lowercase normalized text
Normalized message content must be lowercased.

### Rule 5 — trim spaces
Excess whitespace must be removed.

### Rule 6 — truncate excessive message length
Very long messages must be truncated to preserve stable and compact grouping input.

### Rule 7 — remove known volatile fragments
Signed URLs, retry counters, generated IDs, or similar unstable fragments must be removed when recognized.

## 7. Business rules v1
- events with the same fingerprint may be grouped into the same incident
- grouped events must belong to the same organization and client context
- the fingerprint must be stable across retries of the same underlying issue
- fingerprint quality is expected to improve over time as normalization rules evolve
- v1 favors operational clarity over perfect classification

## 8. Examples

### Example A — same problem, same fingerprint
Raw messages:
- `Request to CRM API failed at 2026-03-08T10:01:02Z for run run_001 with status 502`
- `Request to CRM API failed at 2026-03-08T10:03:44Z for run run_002 with status 500`

Normalized result:
- normalized_error_code: `HTTP_5XX`
- normalized_message: `request to crm api failed with status http_5xx`

Result:
Both events should produce the same fingerprint and be grouped into one incident.

### Example B — different node, different fingerprint
Event 1:
- workflow_id: `wf_orders_sync`
- node_name: `HTTP Request`
- normalized_error_code: `TIMEOUT`

Event 2:
- workflow_id: `wf_orders_sync`
- node_name: `Validate Input`
- normalized_error_code: `MISSING_FIELD`

Result:
These should not produce the same fingerprint.

### Example C — different client, different fingerprint
Two events may share the same workflow structure and message pattern, but if `client_id` differs, they must not be grouped into the same incident.

## 9. Out of scope for v1
The v1 fingerprint does not yet handle:
- semantic similarity scoring
- machine learning clustering
- historical adaptive grouping
- cross-workflow correlation
- parent/child incident grouping