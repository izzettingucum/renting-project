# Renting Project

A Laravel 8 REST API for listing offices, managing office availability, and handling reservation workflows between visitors and hosts.

The project is built as a backend case study for a marketplace-style office rental product. It includes authentication, role-based API permissions, office publishing flows, image management, reservation conflict checks, email notifications, and automated reservation reminders.

## Highlights

- Token-based API authentication with Laravel Sanctum
- Email verification, password reset, and optional two-factor login flow
- Role and permission model for protected API actions
- Office listing, filtering, creation, update, soft deletion, and approval status handling
- Office search by owner, visitor history, tags, and nearest location coordinates
- Polymorphic image upload and deletion rules for office media
- Reservation creation with overlap protection and minimum-stay validation
- Reservation pricing with monthly discount support
- Separate reservation views for visitors and office hosts
- Notification flow for new reservations, office review events, and reservation start dates
- Feature tests covering authentication, offices, images, reservations, and scheduler behavior

## Tech Stack

- PHP `^7.3|^8.0`
- Laravel `8.x`
- Laravel Sanctum
- Laravel UI
- MySQL or another Laravel-supported SQL database
- PHPUnit
- Laravel Mix

## Architecture

The codebase separates API concerns into focused layers:

- `app/Http/Controllers` handles HTTP request/response boundaries.
- `app/Http/Requests` centralizes validation rules.
- `app/Services` contains business workflows such as login, office updates, and reservations.
- `app/Repositories` isolates database access behind interfaces.
- `app/DTO` carries structured data between services and repositories.
- `app/Http/Resources` formats API responses.
- `app/Notifications` contains mail/notification messages for auth, offices, and reservations.
- `app/Policies` and custom middleware enforce ownership and token abilities.

This structure keeps controllers small and makes the core business rules easier to test.

## Core Domain

### Users and Auth

Users register through the API, receive an email verification notification, and get a Sanctum token with abilities derived from their role permissions. Login supports a normal password flow and an optional two-factor code sent by notification.

Supported auth flows include:

- Register
- Login
- Two-factor login confirmation
- Logout
- Reset password for authenticated users
- Forgot password recovery code
- Email verification resend and verification callback

### Offices

Hosts can create offices with location, price, discount, address, description, and tags. Office media is managed through dedicated image upload and delete endpoints. Offices are created as pending by default and can be hidden from public listing.

Public office listing supports:

- Pagination
- Filtering by host user
- Filtering by previous visitor
- Filtering by tags
- Sorting by nearest coordinates
- Active reservation count
- Included user, image, and tag resources

Sensitive updates such as location, address, or daily price automatically move the office back into pending review and notify admins.

### Reservations

Visitors can reserve approved and visible offices. The reservation service protects the booking flow with a cache lock, preventing concurrent requests from creating overlapping reservations for the same office.

Reservation rules include:

- Users cannot reserve their own office.
- Pending or hidden offices cannot be reserved.
- Reservations must start in the future.
- One-day reservations are rejected.
- Date ranges cannot overlap active reservations.
- Reservations can be cancelled only by the owner and only before they start.
- Monthly discount is applied for reservations of at least 28 days.
- A generated Wi-Fi password is stored encrypted on the reservation.

Hosts can list reservations for offices they own and filter them by office, visitor, status, and date range.

## API Overview

All routes are defined under `routes/api.php`.

### Auth

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/auth/register` | Register a new user |
| `POST` | `/api/auth/login` | Login with email and password |
| `POST` | `/api/auth/login/twoFactor` | Complete two-factor login |
| `POST` | `/api/auth/logout` | Revoke the current token |
| `POST` | `/api/auth/reset/password` | Reset password while authenticated |
| `POST` | `/api/auth/password/forgot` | Send forgot-password recovery code |
| `POST` | `/api/auth/password/forgot/check/code` | Validate recovery code |
| `POST` | `/api/auth/password/forgot/reset` | Reset password with recovery token |
| `POST` | `/api/email/verification-notification` | Resend email verification |

### Offices

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/offices` | List offices with filters |
| `GET` | `/api/offices/{id}` | Show office detail |
| `POST` | `/api/offices` | Create office |
| `PATCH` | `/api/offices/{id}` | Update office |
| `DELETE` | `/api/offices/{id}` | Delete office |
| `POST` | `/api/offices/{office}/images` | Upload office image |
| `DELETE` | `/api/offices/{office}/images/{image}` | Delete office image |

### Reservations

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/reservations` | List current user's reservations |
| `POST` | `/api/reservations` | Create reservation |
| `DELETE` | `/api/reservations/{id}` | Cancel reservation |
| `GET` | `/api/host/reservations` | List reservations for host-owned offices |

## Database Model

Main tables:

- `users`
- `roles`
- `permissions`
- `user_roles`
- `role_permissions`
- `offices`
- `office_infos`
- `images`
- `tags`
- `offices_tags`
- `reservations`
- `two_factor_codes`
- `user_verification_mails`
- `forgot_password_mails`

Notable implementation details:

- Offices use soft deletes.
- Office images use a polymorphic relation.
- Reservation date/status indexes are defined for common lookup paths.
- Reservation Wi-Fi passwords are encrypted at the model cast layer.

## Scheduled Jobs

The scheduler runs `send:due-reservations-notifications` daily at `00:00`.

That command finds active reservations starting on the current date and sends reminder notifications to both the visitor and the office host.

## Getting Started

### Requirements

- PHP 7.3 or newer
- Composer
- Node.js and npm
- MySQL or a compatible SQL database

### Installation

```bash
git clone https://github.com/izzettingucum/renting-project.git
cd renting-project
composer install
npm install
cp .env.example .env
php artisan key:generate
```

Configure the database and mail settings in `.env`, then run:

```bash
php artisan migrate
php artisan db:seed --class=RolePermissionSeeder
php artisan storage:link
```

Start the application:

```bash
php artisan serve
```

Compile frontend assets if needed:

```bash
npm run dev
```

## Running Tests

```bash
php artisan test
```

or:

```bash
vendor/bin/phpunit
```

The feature test suite covers:

- Register, login, logout, email verification, password reset, and forgot-password flows
- Two-factor login code generation, validation, and expiration
- Office listing, filtering, creation, update, review status, deletion, and image rules
- Reservation creation, conflict prevention, cancellation, pricing conditions, and notifications
- Host reservation filtering
- Scheduled reservation-start notifications

## Scheduler Setup

For production-like environments, add Laravel's scheduler to cron:

```bash
* * * * * cd /path/to/renting-project && php artisan schedule:run >> /dev/null 2>&1
```

## Design Notes

This repository intentionally emphasizes backend engineering practices:

- Business logic is kept out of controllers.
- Data access is abstracted through repository interfaces.
- Request classes validate API input before services run.
- Policies protect ownership-sensitive operations.
- Notifications and scheduled commands model asynchronous product events.
- Tests document the most important behavior and edge cases.

## License

This project is open-source software licensed under the MIT license.
