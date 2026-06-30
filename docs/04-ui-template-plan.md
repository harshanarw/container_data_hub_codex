# Container Data Hub - Bootstrap UI Template Plan

## UI Direction

Use Laravel Blade with a Bootstrap 5 admin dashboard template.

Recommended options:

1. **AdminLTE 4** - Bootstrap 5 based, familiar admin panel structure, many dashboard components.
2. **CoreUI Laravel / CoreUI Bootstrap** - Strong enterprise-style components and Laravel admin template options.
3. **Tabler** - Clean Bootstrap 5 UI kit, good for modern dashboards, requires more manual Laravel integration.

## Recommended MVP Choice

Use **AdminLTE 4 or CoreUI Bootstrap** for the MVP.

For this project, the UI should prioritize:

- Fast data-entry screens
- Strong table/list views
- Search and filters
- Status badges
- Warning badges
- Timeline/history display
- Admin dashboards
- Responsive layout
- Clear sidebar navigation

## Main Layout

```text
resources/views/layouts/
  app.blade.php
  guest.blade.php
  partials/
    sidebar.blade.php
    navbar.blade.php
    footer.blade.php
    breadcrumbs.blade.php
```

## Suggested Sidebar Menu

```text
Dashboard
Parties
  - All Parties
  - Create Party
  - API Credentials
Access Control
  - Users
  - Roles
  - Permissions
Import Release
  - Container Records
  - Create / Upsert Record
  - Extensions
Yard Lookup
  - Lookup Container
  - Lookup Audit Logs
Monitoring
  - API Request Logs
  - Failed Requests
  - Expiry Monitoring
Reports
  - DO Expiry Report
  - Free Time Expiry Report
  - Reefer Electricity Expiry Report
Future
  - Movement Events
Settings
  - Code Lists
  - Warning Rules
```

## Dashboard Components

### Summary Cards

- Total active parties
- Active yards
- Today API submissions
- Failed API requests
- Containers released
- Containers on hold
- DO expiry within 3 days
- Reefer electricity expiry within 24 hours

### Charts

- Submissions by party type
- Failed requests by endpoint
- Release status distribution
- Lookup volume by yard

### Tables

- Latest failed API requests
- Latest container updates
- Expiring DO records
- Yard nomination mismatch records

## Common UI Components Required

### Tables

Use responsive Bootstrap tables with:

- Search
- Pagination
- Status badges
- Action dropdowns
- Export button placeholder

### Forms

Use Bootstrap forms with:

- Required field indicators
- Validation messages
- Date/time pickers
- Select dropdowns
- Textarea remarks fields
- Save / Cancel / Back actions

### Badges

Suggested status colors:

- ACTIVE - success
- INACTIVE - secondary
- SUSPENDED - danger
- RELEASED - success
- PENDING - warning
- HOLD - danger
- EXPIRED - danger
- VALID - success

### Warning Panel

Yard lookup should show warnings clearly:

```text
Warning code: DO_EXPIRED
Severity: HIGH
Message: Delivery Order expired on 2026-07-20.
```

### History Timeline

Container detail page should include a vertical timeline:

- Version number
- Change type
- Changed by party
- Submitted at
- Old values / new values summary

## MVP Screens

## 1. Dashboard

Route: `/dashboard`

Purpose:

- Operational overview
- API health
- Expiry monitoring
- Quick access to lookup

## 2. Party List

Route: `/parties`

Columns:

- Party Code
- Party Name
- Party Type
- Country
- API Enabled
- Portal Enabled
- Status
- Actions

## 3. Party Create / Edit

Route:

- `/parties/create`
- `/parties/{party}/edit`

Fields:

- Party code
- Party name
- Legal name
- Party type
- Country
- Address
- Email
- Phone
- API enabled
- Portal enabled
- Status

## 4. Import Container List

Route: `/imports`

Filters:

- Container number
- Shipping line
- Nominated yard
- Release status
- Hold status
- DO expiry date range

Columns:

- Container Number
- Line
- Vessel / Voyage
- DO Expiry
- Free Time Expiry
- Nominated Yard
- Release Status
- Hold Status
- Latest Version
- Actions

## 5. Import Container Detail

Route: `/imports/{id}`

Sections:

- Container overview
- Party ownership
- Vessel details
- DO and free time
- Extensions
- Reefer electricity
- Nominated yard
- Release / hold status
- Warning panel
- History timeline

## 6. Yard Lookup

Route: `/yard-lookup`

Fields:

- Container number
- Yard code

Result sections:

- Latest release information
- Warning flags
- Nominated yard result
- Expiry details
- Reefer electricity details

## 7. API Request Logs

Route: `/monitoring/api-requests`

Columns:

- Request ID
- Party
- Endpoint
- Method
- HTTP Status
- Processing Status
- Created At
- Actions

## UI Development Rules

1. Use reusable Blade components for cards, badges, forms, and tables.
2. Keep all admin screens consistent.
3. Use server-side validation and show errors in Bootstrap alert style.
4. Use badges for statuses and warnings.
5. Use confirmation modal for destructive actions.
6. Avoid putting business logic inside Blade views.
