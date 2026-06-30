# Codex / AI Agent Implementation Instructions

## Project

Container Data Hub - Laravel + MySQL application.

## Before Editing Code

1. Read these documents first:
   - `README.md`
   - `docs/01-implementation-roadmap.md`
   - `docs/02-database-design.md`
   - `docs/03-api-contracts.md`
   - `docs/04-ui-template-plan.md`
2. Create a short implementation plan.
3. Identify files to be created or changed.
4. Confirm the plan before making large or risky changes.

## Technical Direction

Use Laravel as a modular monolith.

Preferred stack:

- Laravel 13.x or latest stable Laravel
- PHP 8.3+
- MySQL 8.x
- Blade templates
- Bootstrap 5 admin template
- Laravel Sanctum or token-based auth for API clients
- Laravel session auth for portal users
- Eloquent ORM
- Form Request validation
- Service classes for business logic
- Policies / Gates for authorization
- PHPUnit or Pest tests

## Naming Rules

Use clear business names.

Examples:

- `Party`
- `PartyCredential`
- `ImportContainerCurrentState`
- `ImportContainerHistory`
- `ContainerExtension`
- `LookupAuditLog`
- `ApiRequestLog`
- `MovementEvent`

## Laravel Structure

Recommended structure:

```text
app/
  Actions/
    ImportRelease/
    Parties/
    YardLookup/
  Domain/
    Parties/
    ImportRelease/
    YardLookup/
    Monitoring/
  DTOs/
  Enums/
  Http/
    Controllers/
      Api/
      Web/
    Requests/
    Resources/
  Models/
  Policies/
  Services/
```

## Important Business Rules

1. Party code must be unique.
2. Inactive or suspended parties cannot submit or retrieve data.
3. Container number must be normalized before save or lookup.
4. Every state-changing submission must create a history record.
5. Latest state must be updated without deleting history.
6. Every API call must be logged.
7. Duplicate `request_id` should be handled idempotently.
8. Yard lookup should return latest effective state and warnings.
9. Nominated yard mismatch must be returned as a warning.
10. Reefer electricity extension and expiry must be supported.
11. Future movement events must be separate from import release data.

## Suggested Implementation Sequence

### Step 1 - Laravel Scaffold

Create Laravel project files in this repository.

### Step 2 - Environment

Configure:

- `.env.example`
- MySQL connection
- App name
- Timezone

Suggested settings:

```env
APP_NAME="Container Data Hub"
APP_TIMEZONE=Asia/Colombo
DB_CONNECTION=mysql
DB_DATABASE=container_data_hub
```

### Step 3 - Database Migrations

Create migrations for:

- `parties`
- `party_credentials`
- `import_container_current_states`
- `import_container_histories`
- `container_extensions`
- `lookup_audit_logs`
- `api_request_logs`
- `movement_events`

### Step 4 - Models and Relationships

Create Eloquent models and relationships.

Important relationships:

- Party has many credentials
- Import current state belongs to shipping line party
- Import current state belongs to NVOCC party
- Import current state belongs to freight forwarder party
- Import current state belongs to nominated yard party
- Import current state has many histories
- Import current state has many extensions

### Step 5 - Web UI Foundation

Create Bootstrap admin layout:

- Sidebar
- Navbar
- Dashboard
- Shared components
- Auth pages

### Step 6 - Party Management

Create CRUD:

- Party list
- Party create
- Party edit
- Party view
- Credential management

### Step 7 - Import Release Management

Create:

- Import container list
- Detail screen
- Create/upsert screen
- History timeline
- Extension panel

### Step 8 - API Endpoints

Create API controllers and request validation.

Use service classes for update logic:

- `ImportRecordService`
- `DeliveryOrderService`
- `ExtensionService`
- `YardNominationService`
- `ReleaseStatusService`
- `WarningService`
- `ApiRequestLogService`

### Step 9 - Yard Lookup

Create lookup endpoint and portal screen.

Return:

- Latest current state
- Warning flags
- Yard nomination validation
- Expiry warnings

### Step 10 - Tests

Add tests for:

- Party creation
- Unique party code
- Import record creation
- Repeated update creates history
- Duplicate request id handling
- Yard lookup found / not found
- Nominated yard mismatch warning
- Expired DO warning

## Code Quality Rules

1. Keep controllers thin.
2. Put business logic in services/actions.
3. Use Form Requests for validation.
4. Use API Resources for API responses.
5. Do not store raw client secrets.
6. Do not commit `.env`.
7. Add indexes for lookup fields.
8. Add tests for main workflows.
9. Use database transactions for state-changing imports.
10. Use enums or config files for statuses and code lists.

## First Coding Task

After Laravel is scaffolded, implement these first:

1. `parties` migration
2. `Party` model
3. `PartyType` enum
4. `PartyStatus` enum
5. Party factory and seeder
6. Party CRUD routes and controller
7. Party list/create/edit Blade screens
