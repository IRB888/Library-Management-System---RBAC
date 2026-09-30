# CircuLib API

I built this Express API for CircuLib's admin, librarian, and student pages. It uses PostgreSQL queries and SQL routines for catalog, membership, circulation, fees, and reporting.

## Setup

Use Node.js 20+ and a TLS-enabled PostgreSQL database. Follow the [root setup guide](../README.md#database-setup) to create the database schema first.

```bash
cd backend
npm ci
cp .env.example .env
```

Set `DATABASE_URL` to your own PostgreSQL connection string and `JWT_SECRET` to your own long random secret. Don't use the example connection string or seeded passwords for a public deployment.

```dotenv
PORT=5000
NODE_ENV=development
DATABASE_URL=your_postgresql_connection_string
DB_SCHEMA=library_app
JWT_SECRET=your_own_long_random_secret
JWT_EXPIRY=24h
```

The [environment config](src/config/env.js) requires `DATABASE_URL`. It does not read the older `DB_HOST`, `DB_NAME`, or `DB_PASSWORD` settings. The [pool](src/config/db.js) uses TLS with certificate verification disabled; local PostgreSQL without TLS needs a deliberate pool configuration change.

```bash
npm run dev
```

The server listens on `http://localhost:5000` by default. `npm start` runs the same server without Nodemon. Optional pool and retry settings are listed in [.env.example](.env.example).

## Routes

| Group | Purpose |
| --- | --- |
| `/api/auth` | Login, student registration, current user, profile, password |
| `/api/admin` | Accounts, books, copies, membership types, admin overrides |
| `/api/librarian` | Student lookup, catalog, barcodes, alerts, reservations |
| `/api/student` | Personal loans, fees, payments, alerts, catalog |
| `/api/circulation` | Checkout, issue, return, loan and copy history |
| `/api/reports` | Library totals, overdue, circulation, inventory, balances |
| `/api/search` | Role-filtered search |
| `/api/features` | Reviews, wishlist, reservations, announcements |

[Route files](src/routes) show the exact methods and role checks. [Controllers](src/controllers) contain the queries. Roles and ownership rules are endpoint-specific; an authenticated token alone does not mean every action is allowed.

`GET /api/health` returns process status. It does not check the database connection.

## Authentication and responses

Protected requests use `Authorization: Bearer <token>`. Password hashes are created and checked with PostgreSQL `pgcrypto` functions, not a Node bcrypt dependency.

A successful login returns this envelope:

```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "<token>",
    "user": {
      "user_id": 1,
      "email": "student@example.com",
      "role": "student",
      "full_name": "Example Student",
      "is_demo": false
    }
  }
}
```

Errors use `success: false`, `message`, and `errors`. Database details stay out of the public error message.

## Database code

The data layer is [SQL schema and routines](../db), plus controller queries. There is no separate ORM model layer. The main routines include `sp_checkout_book`, `sp_return_book`, `sp_generate_overdue_alerts`, and `sp_apply_fee_payment`.

Use `db/schema/02_users_and_auth.sql` as the canonical credential-verification definition. The legacy standalone file only documents that choice. Reviews, wishlist, reservations, and announcements use `db/schema/03_modern_features.sql`.

Fees and payment history are accounting records, not an online payment gateway. Overdue generation is currently a librarian action. Use an isolated database for a public demo: demo profile/password checks are implemented, while wider write restrictions are a next step.

## Checks and next steps

```bash
npm run lint
```

Jest scripts and Playwright configuration are present, but the backend does not yet include its unit or end-to-end test files. My next testing work is database-backed coverage for auth, circulation, role boundaries, and failure paths. Frontend retry tests are documented in the [root README](../README.md#checks).

For production, use your own credentials, review the TLS configuration, and replace bootstrap account passwords. My next security work includes broader demo write restrictions and stricter auth configuration.
