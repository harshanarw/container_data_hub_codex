# Container Data Hub

Centralized Import Container Release and Yard Movement Exchange Platform.

This repository is for building a PHP Laravel + MySQL application for managing import container release data exchange between Shipping Lines, NVOCCs, Freight Forwarders, Container Yards, Depots, and Platform Admin users.

## Product Goal

Container Data Hub acts as a trusted central source of truth for import release information such as:

- Import container details
- Shipping line / NVOCC / freight forwarder references
- Discharged vessel and voyage details
- Vessel ETA and berthing information
- Delivery Order issue and expiry details
- Free days and free time expiry
- FCL, detention, and demurrage extensions
- Reefer electricity extension and electricity expiry
- Nominated yard
- Release status and hold status
- Full change history and audit trail

## Recommended Technical Stack

| Layer | Technology |
|---|---|
| Backend | Laravel 13.x or latest stable Laravel |
| Language | PHP 8.3+ |
| Database | MySQL 8.x |
| UI | Blade + Bootstrap 5 admin template |
| Admin Template | AdminLTE 4 or CoreUI Bootstrap template |
| Auth | Laravel session auth for portal, Sanctum/token auth for APIs |
| ORM | Eloquent |
| Testing | PHPUnit / Pest |
| Build | Vite + NPM |

## MVP Modules

1. Party Management
2. User, Role, and Permission Management
3. API Client / Credential Management
4. Import Container Current State
5. Import Container History
6. Container Extensions
7. Yard Lookup / Inquiry
8. Warning Engine
9. API Request Logs
10. Admin Dashboard and Monitoring

## Future Modules

1. Yard Movement Exchange
2. Notifications and Alerts
3. Bulk File Upload
4. Analytics and SLA Dashboards
5. Partner Self-Service Portal
6. EDI / Integration Adapter Layer

## Initial Laravel Setup

Run these commands locally after cloning this repository:

```bash
composer create-project laravel/laravel .
cp .env.example .env
php artisan key:generate
```

Configure MySQL in `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=container_data_hub
DB_USERNAME=root
DB_PASSWORD=
```

Then run:

```bash
php artisan migrate
npm install
npm run build
php artisan serve
```

## Development Approach

This project should be built module by module using a latest-state-plus-history design.

- The `import_container_current_states` table stores the latest effective state for fast yard lookup.
- The `import_container_histories` table stores every versioned update for auditability.
- Each API request must include a `request_id` or `source_reference` so duplicate submissions can be handled safely.
- Yard lookup should return both data and warning flags.

## Documentation

See the `/docs` folder for implementation roadmap, data model, API scope, and UI template plan.
