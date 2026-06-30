# Container Data Hub - Implementation Roadmap

## Source Documents

This roadmap is based on the Container Data Hub SRS and Scope documents.

## Architecture Direction

Use a modular Laravel monolith first. This keeps the application simple to build, test, and deploy while still allowing future module extraction if required.

Recommended Laravel module boundaries:

```text
app/
  Domain/
    Parties/
    AccessControl/
    ImportRelease/
    YardLookup/
    Warnings/
    Monitoring/
    Movements/
  Http/
    Controllers/
      Web/
      Api/
  Services/
  Actions/
  Policies/
  DTOs/
  Enums/
```

## Phase 1 - Foundation

### Goals

- Set up Laravel project
- Configure MySQL
- Configure Bootstrap admin UI
- Create authentication foundation
- Create base layout, sidebar, dashboard, and login pages

### Deliverables

- Laravel application scaffold
- `.env.example` configured for MySQL
- Bootstrap admin layout
- Admin dashboard screen
- Basic auth
- Base route groups for portal and API

## Phase 2 - Party and Access Management

### Goals

- Register all platform parties
- Manage party codes and party types
- Enable or disable API and portal access
- Prepare role-based access control

### Tables

- `parties`
- `party_credentials`
- `users`
- `roles`
- `permissions`
- `role_user`

### Screens

- Party list
- Party create/edit/view
- Party credential management
- User management
- Role management

## Phase 3 - Import Release Current State and History

### Goals

- Create latest effective import container record
- Create full history/version record for every update
- Support repeated submissions for the same container

### Tables

- `import_container_current_states`
- `import_container_histories`
- `container_extensions`
- `api_request_logs`

### Screens

- Import container list
- Import container detail
- Import container history timeline
- Extension detail section

## Phase 4 - API Submission Layer

### Goals

- Build secure API endpoints for party integrations
- Support create/update workflows
- Log every API request and response
- Add request idempotency checks

### API Groups

- Auth / token
- Party profile
- Import create/upsert
- Delivery Order update
- Free time update
- Extension update
- Yard nomination update
- Release / hold status update

## Phase 5 - Yard Lookup

### Goals

- Allow yard systems to retrieve latest container release data
- Return validation and warning flags
- Validate yard access using party code and nominated yard

### API Endpoints

- `GET /api/imports/{containerNumber}`
- `GET /api/imports/{containerNumber}/history`
- `GET /api/imports/{containerNumber}/warnings?yardCode=...`

### Portal Screens

- Yard lookup screen
- Container result view
- Warning flag panel

## Phase 6 - Monitoring and Admin Support

### Goals

- Provide operational monitoring for support users
- Review failed requests
- View request/response logs
- Search expired DO / free time / electricity expiry records

### Screens

- API request log list
- Failed requests
- Dashboard summary cards
- Expiry monitoring reports

## Phase 7 - Future Yard Movement Exchange

### Goals

- Receive yard movement events
- Store event history separately from release data
- Expose movement history to authorized principals

### Tables

- `movement_events`

### Events

- Gate In
- Gate Out
- Survey
- Repair In
- Repair Out
- Wash
- Storage
- Hold
- Release

## Implementation Rules

1. Never overwrite history.
2. Each state-changing update must create a history record.
3. Each API call must be logged.
4. Container number must be standardized before lookup.
5. Party code must be unique.
6. Inactive or suspended parties cannot submit or retrieve data.
7. Yard lookup returns latest state by default.
8. Warning calculation should be handled in a dedicated service.
9. Future movement events must not overwrite import release current state.
