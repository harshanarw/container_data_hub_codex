# Container Data Hub - Database Design

## Design Principle

The platform uses a latest-state-plus-history model.

- `import_container_current_states` stores the latest effective state used for fast yard lookup.
- `import_container_histories` stores every versioned change.
- `container_extensions` stores structured extension records separately because extensions can happen multiple times.
- `api_request_logs` stores request/response traces for integration monitoring.

## Core Tables

## 1. parties

Stores registered organizations.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| party_code | varchar(30) | Unique business code, e.g. SL001, NV015, FF032, YD008 |
| party_name | varchar(150) | Display name |
| legal_name | varchar(200) | Registered legal name |
| party_type | enum/string | SHIPPING_LINE, NVOCC, FREIGHT_FORWARDER, YARD, DEPOT, ADMIN |
| country | varchar(80) | Country |
| address | text nullable | Business address |
| email | varchar(150) nullable | Main email |
| phone | varchar(50) nullable | Main phone |
| status | enum/string | ACTIVE, INACTIVE, SUSPENDED |
| api_enabled | boolean | Whether API access is enabled |
| portal_enabled | boolean | Whether portal access is enabled |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |

Indexes:

- unique `party_code`
- index `party_type`
- index `status`

## 2. party_credentials

Stores API client access for parties.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| party_id | foreignId | FK to parties |
| client_id | varchar(100) | Unique client id |
| client_secret_hash | varchar(255) | Hashed secret only |
| auth_mode | enum/string | CLIENT_CREDENTIALS, API_TOKEN, SIGNED_KEY |
| status | enum/string | ACTIVE, INACTIVE, REVOKED |
| allowed_ips_json | json nullable | Optional IP restrictions |
| last_used_at | timestamp nullable | Last API usage |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |

Indexes:

- unique `client_id`
- index `party_id`
- index `status`

## 3. import_container_current_states

Stores the latest effective import release state.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| container_number | varchar(20) | Standardized primary operational key |
| shipping_line_party_id | foreignId nullable | FK to parties |
| nvocc_party_id | foreignId nullable | FK to parties |
| freight_forwarder_party_id | foreignId nullable | FK to parties |
| container_owner | varchar(150) nullable | Container owner/customer |
| customer_name | varchar(150) nullable | Customer |
| consignee_name | varchar(200) nullable | Consignee |
| container_iso_type | varchar(20) nullable | ISO type |
| bl_reference | varchar(100) nullable | BL/HBL reference |
| vessel_name | varchar(150) nullable | Discharged vessel |
| voyage_no | varchar(50) nullable | Voyage |
| eta | datetime nullable | ETA |
| berthing_datetime | datetime nullable | Actual berthing date/time |
| port_of_discharge | varchar(100) nullable | Port |
| do_number | varchar(100) nullable | Delivery Order number |
| do_issued_date | date nullable | DO issued date |
| do_expiry_date | date nullable | DO expiry date |
| free_days | integer nullable | Free days |
| free_time_start_date | date nullable | Free time start |
| free_time_end_date | date nullable | Free time expiry |
| fcl_extended_days | integer nullable | Latest FCL extension days |
| fcl_expiry_date | date nullable | Latest FCL expiry |
| detention_extended_days | integer nullable | Latest detention extension days |
| detention_expiry_date | date nullable | Latest detention expiry |
| demurrage_extended_days | integer nullable | Latest demurrage extension days |
| demurrage_expiry_date | date nullable | Latest demurrage expiry |
| is_reefer | boolean | Reefer flag |
| reefer_electricity_extended_value | decimal(10,2) nullable | Days or hours depending on unit |
| reefer_electricity_extended_unit | varchar(20) nullable | DAYS or HOURS |
| reefer_electricity_expiry_date | datetime nullable | Electricity expiry |
| nominated_yard_party_id | foreignId nullable | FK to parties |
| release_status | varchar(50) nullable | PENDING, RELEASED, HOLD, etc. |
| hold_status | varchar(50) nullable | NONE, ACTIVE, CUSTOMS_HOLD, LINE_HOLD, etc. |
| remarks | text nullable | Instructions |
| source_reference | varchar(100) nullable | Source system reference |
| submitted_by_party_id | foreignId nullable | FK to parties |
| latest_version_no | integer | Current version number |
| last_request_id | varchar(100) nullable | Latest accepted request id |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |

Indexes:

- unique/index strategy should be finalized based on ownership rules.
- index `container_number`
- index `do_expiry_date`
- index `free_time_end_date`
- index `nominated_yard_party_id`
- index `release_status`
- index `hold_status`

## 4. import_container_histories

Stores each versioned state update.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| import_container_current_state_id | foreignId | FK to current state |
| version_no | integer | Incremented per container current state |
| change_type | varchar(80) | INITIAL_IMPORT, DO_UPDATE, EXTENSION_UPDATE, YARD_NOMINATION_UPDATE, STATUS_UPDATE |
| old_values_json | json nullable | Previous values |
| new_values_json | json | Submitted/effective new values |
| submitted_by_party_id | foreignId | FK to parties |
| request_reference | varchar(100) nullable | External request/source reference |
| request_id | varchar(100) nullable | Idempotency key |
| submitted_at | timestamp | Submission timestamp |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |

Indexes:

- index `import_container_current_state_id`
- index `version_no`
- index `change_type`
- index `request_id`

## 5. container_extensions

Stores structured extension records.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| import_container_current_state_id | foreignId | FK to current state |
| extension_type | varchar(50) | FCL, DETENTION, DEMURRAGE, REEFER_ELECTRICITY |
| granted_value | decimal(10,2) | Days/hours |
| granted_unit | varchar(20) | DAYS or HOURS |
| effective_date | date nullable | Effective date |
| previous_expiry | datetime/date nullable | Previous expiry |
| new_expiry | datetime/date nullable | New expiry |
| remarks | text nullable | Remarks |
| submitted_by_party_id | foreignId | FK to parties |
| request_reference | varchar(100) nullable | Source reference |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |

Indexes:

- index `import_container_current_state_id`
- index `extension_type`
- index `new_expiry`

## 6. lookup_audit_logs

Stores yard lookup access tracking.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| container_number | varchar(20) | Lookup key |
| yard_party_id | foreignId nullable | Calling yard |
| requester_party_id | foreignId nullable | Authenticated party |
| result_status | varchar(50) | FOUND, NOT_FOUND, UNAUTHORIZED, ERROR |
| warnings_json | json nullable | Warnings returned |
| ip_address | varchar(60) nullable | Request IP |
| user_agent | varchar(500) nullable | User agent |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |

## 7. api_request_logs

Stores integration request/response trace.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| request_id | varchar(100) nullable | External idempotency key |
| party_id | foreignId nullable | Calling party |
| endpoint | varchar(255) | Endpoint |
| http_method | varchar(10) | GET, POST, PATCH, etc. |
| request_body_json | json nullable | Request body |
| response_body_json | json nullable | Response body |
| http_status | integer | HTTP status |
| processing_status | varchar(50) | SUCCESS, FAILED, DUPLICATE, VALIDATION_ERROR |
| error_code | varchar(100) nullable | Error code |
| error_message | text nullable | Error summary |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |

Indexes:

- index `request_id`
- index `party_id`
- index `endpoint`
- index `processing_status`
- index `created_at`

## 8. warning_rules / warning_results

Warning rules can start as service logic. Store warning results only if operational reporting requires persistence.

Suggested warning codes:

- `DO_EXPIRED`
- `FREE_TIME_EXPIRED`
- `FCL_EXPIRED`
- `DETENTION_EXPIRED`
- `DEMURRAGE_EXPIRED`
- `REEFER_ELECTRICITY_EXPIRED`
- `YARD_NOMINATION_MISMATCH`
- `HOLD_ACTIVE`
- `RELEASE_PENDING`
- `MISSING_DO`

## 9. movement_events - Future Phase

Stores yard movement events separately from import release current state.

| Column | Type | Notes |
|---|---|---|
| id | bigint unsigned | PK |
| container_number | varchar(20) | Container number |
| yard_party_id | foreignId | Source yard |
| event_type | varchar(80) | GATE_IN, GATE_OUT, SURVEY, REPAIR_IN, REPAIR_OUT, WASH, STORAGE, HOLD, RELEASE |
| event_datetime | datetime | Event date/time |
| reference_no | varchar(100) nullable | Yard reference |
| remarks | text nullable | Remarks |
| submitted_by_party_id | foreignId nullable | FK to parties |
| source_reference | varchar(100) nullable | Source system ref |
| created_at | timestamp | Laravel timestamp |
| updated_at | timestamp | Laravel timestamp |
