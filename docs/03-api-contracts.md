# Container Data Hub - API Contract Plan

## API Design Principles

1. API-first design.
2. Every machine-to-machine call must be authenticated.
3. Every submission must include `party_code`, `request_id`, and `source_reference` where applicable.
4. Every state-changing request must create a history entry.
5. Duplicate `request_id` values should be idempotently handled.
6. Responses must include structured status, code, message, and request reference.
7. Yard lookup must return both latest data and warning flags.

## Common Request Headers

```http
Authorization: Bearer <token>
Content-Type: application/json
X-Request-Id: <unique-request-id>
X-Party-Code: <party-code>
```

## Common Response Shape

```json
{
  "success": true,
  "code": "SUCCESS",
  "message": "Request processed successfully.",
  "request_id": "REQ-2026-0001",
  "data": {}
}
```

## Error Response Shape

```json
{
  "success": false,
  "code": "VALIDATION_ERROR",
  "message": "Invalid container number.",
  "request_id": "REQ-2026-0001",
  "errors": {
    "container_number": ["Container number format is invalid."]
  }
}
```

## Auth APIs

### POST /api/auth/token

Issues token for API client.

Request:

```json
{
  "client_id": "SL001_CLIENT",
  "client_secret": "secret"
}
```

Response:

```json
{
  "success": true,
  "code": "TOKEN_ISSUED",
  "message": "Token issued successfully.",
  "data": {
    "access_token": "token",
    "token_type": "Bearer",
    "expires_in": 3600,
    "party_code": "SL001"
  }
}
```

## Party APIs

### POST /api/admin/parties

Admin-only party creation.

Request:

```json
{
  "party_code": "SL001",
  "party_name": "ABC Shipping Line",
  "legal_name": "ABC Shipping Line Ltd",
  "party_type": "SHIPPING_LINE",
  "country": "Sri Lanka",
  "email": "ops@example.com",
  "phone": "+94110000000",
  "api_enabled": true,
  "portal_enabled": true,
  "status": "ACTIVE"
}
```

### GET /api/admin/parties/{partyCode}

Returns party profile.

### PATCH /api/admin/parties/{partyCode}

Updates party profile.

### POST /api/admin/parties/{partyCode}/status

Activates, deactivates, or suspends a party.

## Import Release APIs

## POST /api/imports

Creates or upserts an initial import record.

Request:

```json
{
  "request_id": "REQ-IMPORT-0001",
  "party_code": "SL001",
  "source_reference": "LINE-SYS-12345",
  "container_number": "MSCU1234567",
  "container_iso_type": "45G1",
  "shipping_line_party_code": "SL001",
  "nvocc_party_code": null,
  "freight_forwarder_party_code": "FF001",
  "container_owner": "ABC Line",
  "customer_name": "Customer A",
  "consignee_name": "Consignee A",
  "bl_reference": "BL123456",
  "vessel_name": "MV Example",
  "voyage_no": "V001",
  "eta": "2026-07-10 08:00:00",
  "berthing_datetime": null,
  "port_of_discharge": "CMB",
  "nominated_yard_party_code": "YD001",
  "release_status": "PENDING",
  "hold_status": "NONE",
  "remarks": "Initial import record"
}
```

Response:

```json
{
  "success": true,
  "code": "IMPORT_UPSERTED",
  "message": "Import container record saved successfully.",
  "request_id": "REQ-IMPORT-0001",
  "data": {
    "container_number": "MSCU1234567",
    "version_no": 1
  }
}
```

## PATCH /api/imports/{containerNumber}/delivery-order

Updates Delivery Order information.

Request:

```json
{
  "request_id": "REQ-DO-0001",
  "party_code": "SL001",
  "source_reference": "LINE-DO-555",
  "do_number": "DO123456",
  "do_issued_date": "2026-07-11",
  "do_expiry_date": "2026-07-20"
}
```

## PATCH /api/imports/{containerNumber}/free-time

Updates free time information.

Request:

```json
{
  "request_id": "REQ-FREE-0001",
  "party_code": "SL001",
  "free_days": 7,
  "free_time_start_date": "2026-07-11",
  "free_time_end_date": "2026-07-18"
}
```

## PATCH /api/imports/{containerNumber}/extensions

Submits extension data.

Request:

```json
{
  "request_id": "REQ-EXT-0001",
  "party_code": "SL001",
  "extension_type": "DETENTION",
  "granted_value": 3,
  "granted_unit": "DAYS",
  "effective_date": "2026-07-18",
  "previous_expiry": "2026-07-18",
  "new_expiry": "2026-07-21",
  "remarks": "Detention extension granted"
}
```

Allowed extension types:

- `FCL`
- `DETENTION`
- `DEMURRAGE`
- `REEFER_ELECTRICITY`

## PATCH /api/imports/{containerNumber}/yard-nomination

Updates nominated yard.

Request:

```json
{
  "request_id": "REQ-YARD-0001",
  "party_code": "SL001",
  "nominated_yard_party_code": "YD002",
  "remarks": "Yard changed by line"
}
```

## PATCH /api/imports/{containerNumber}/status

Updates release and hold status.

Request:

```json
{
  "request_id": "REQ-STATUS-0001",
  "party_code": "SL001",
  "release_status": "RELEASED",
  "hold_status": "NONE",
  "remarks": "Released for yard acceptance"
}
```

## Lookup APIs

## GET /api/imports/{containerNumber}?yardCode=YD001

Returns latest effective import release data.

Response:

```json
{
  "success": true,
  "code": "IMPORT_FOUND",
  "message": "Latest import container release information returned.",
  "data": {
    "container_number": "MSCU1234567",
    "shipping_line": "ABC Shipping Line",
    "consignee_name": "Consignee A",
    "vessel_name": "MV Example",
    "voyage_no": "V001",
    "do_number": "DO123456",
    "do_expiry_date": "2026-07-20",
    "free_time_end_date": "2026-07-18",
    "detention_expiry_date": "2026-07-21",
    "demurrage_expiry_date": null,
    "reefer_electricity_expiry_date": null,
    "nominated_yard_party_code": "YD001",
    "release_status": "RELEASED",
    "hold_status": "NONE",
    "latest_version_no": 4,
    "warnings": []
  }
}
```

## GET /api/imports/{containerNumber}/history

Returns version history.

## GET /api/imports/{containerNumber}/warnings?yardCode=YD001

Returns warning flags only.

## Monitoring APIs

### GET /api/admin/logs/requests

Query request logs.

### GET /api/admin/logs/errors

Query failed submissions.

### GET /api/admin/dashboard/summary

Dashboard summary values.

## Future Movement APIs

### POST /api/movements

Submits yard movement event.

Request:

```json
{
  "request_id": "REQ-MOVE-0001",
  "party_code": "YD001",
  "container_number": "MSCU1234567",
  "event_type": "GATE_IN",
  "event_datetime": "2026-07-21 09:15:00",
  "reference_no": "YARD-GI-10001",
  "remarks": "Gate in completed"
}
```

### GET /api/movements/{containerNumber}

Returns movement history.

### GET /api/movements/{containerNumber}/latest

Returns latest operational event.
