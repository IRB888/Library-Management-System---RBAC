# CircuLib

I built CircuLib to handle library catalog and borrowing workflows across three roles: admin, librarian, and student. It has a React frontend, an Express API, and a PostgreSQL database. The borrowing rules, loans, fees, and reports use database records and SQL routines.

[Open the demo](https://library-management-system-rbac.vercel.app/)

## What it does

- **Admin:** manage books, physical copies, librarian accounts, and membership types; view library totals and reports.
- **Librarian:** look up students and barcodes, issue and return books, review overdue loans and alerts, and handle reservations.
- **Student:** register, browse books, see personal loans, fees, payment history, and alerts; save books to a wishlist, leave reviews, and request reservations.
- **Shared:** JWT login, role-based routes, profile and password updates, announcements, and light/dark themes.

Membership types set borrowing limits, loan periods, and daily late fees. Checkout and return logic lives in both API transactions and SQL routines. The reports cover inventory, circulation, overdue loans, member activity, and outstanding balances.

Fee payments are database accounting records, not an online payment gateway. Overdue alert generation is a librarian action. Reservations have a basic request, fulfillment, and cancellation flow; automated queue handling is a next step.

## Stack

- React 18, Vite, Material UI, React Router, Zustand, Axios, and Recharts
- Node.js, Express, JWT, Helmet, CORS, and `pg`
- PostgreSQL tables, constraints, functions, procedures, and views; `pgcrypto` for password hashes

## Code layout

```text
frontend/src/
  api/           API client and role-specific requests
  pages/         Admin, librarian, student, and auth screens
  components/    Layout, route protection, charts, and shared UI
  store/         Auth, theme, and UI state
backend/src/
  routes/        API routes and role checks
  controllers/   Queries and request handling
  config/        Environment and database connection
  middleware/   Authentication and authorization
  services/      Global search
  utils/         Error and response helpers
db/
  schema/        Tables, constraints, auth, and optional features
  procedures/    Checkout, returns, overdue alerts, and fees
  views/         Loan and reporting views
  admin/         Admin functions and views
  reports/       SQL reports
  seeds/         Sample library data
```

## Run it locally

Use Node.js 20+ and PostgreSQL with permission to create extensions. The current database client uses TLS, so use a TLS-enabled PostgreSQL connection. For a local server without TLS, adjust [the pool configuration](backend/src/config/db.js) before starting the API.

```bash
git clone https://github.com/miteshanshu/Library-Management-System---RBAC.git
cd Library-Management-System---RBAC
cd backend
npm ci
cp .env.example .env
```

Set these values in `backend/.env`:

```dotenv
PORT=5000
NODE_ENV=development
DATABASE_URL=your_postgresql_connection_string
DB_SCHEMA=library_app
JWT_SECRET=your_own_long_random_secret
JWT_EXPIRY=24h
```

### Database setup

Start with a fresh database. The base tables and indexes are bootstrap scripts, not a migration system, so don't rerun the full sequence against an existing database.

Apply the files in this order with `psql` or your database SQL editor:

1. [Core tables](db/schema/00_init_schema.sql)
2. [Constraints and indexes](db/schema/01_constraints_indexes.sql)
3. Enable `pgcrypto` in `library_app`: `CREATE EXTENSION IF NOT EXISTS pgcrypto WITH SCHEMA library_app;`. If it already exists in another schema, make sure that schema is in the search path when running the auth script.
4. [Users and authentication](db/schema/02_users_and_auth.sql)
5. [Reviews, wishlist, reservations, and announcements](db/schema/03_modern_features.sql)
6. [Checkout and returns](db/procedures/checkout_and_return.sql)
7. [Overdue alerts and fees](db/procedures/overdue_and_fees.sql)
8. [Analytics views](db/views/analytics_views.sql) and [overdue loans view](db/views/vw_overdue_loans.sql)
9. [Admin views](db/admin/admin_views.sql) and [admin functions](db/admin/admin_functions.sql)
10. [Inventory and member reports](db/reports/inventory_and_member_reports.sql)
11. For fuzzy search, apply [search indexes](db/schema/07_fuzzy_search_indexes.sql). This script enables `pg_trgm` and uses the application and public search paths.
12. Optionally load [sample library data](db/seeds/sample_data.sql).

Use the credential-verification function in the auth schema script above. The standalone `db/functions/fn_verify_user_credentials.sql` is now a pointer to that canonical definition, not a second function to install.

The auth bootstrap includes demo and admin accounts. Replace the seeded passwords before exposing your own deployment. Demo restrictions currently cover profile and password changes, so use an isolated database for a public demo.

### Start the API and frontend

In the backend directory:

```bash
npm run dev
```

In a second terminal, from the repository root:

```bash
cd frontend
npm ci
```

Create `frontend/.env.local`:

```dotenv
VITE_API_URL=http://localhost:5000/api
```

Then run:

```bash
npm run dev
```

The API runs on `http://localhost:5000` and Vite on `http://localhost:5173`. Set `VITE_API_URL` explicitly for local development; otherwise the client uses the hosted Render API.

## API and SQL

The API groups are `/api/auth`, `/api/admin`, `/api/librarian`, `/api/student`, `/api/circulation`, `/api/reports`, `/api/search`, and `/api/features`. [Route files](backend/src/routes) show the endpoints and role checks; [controllers](backend/src/controllers) show how they use the database.

`GET /api/health` checks that the API process responds. It is not a database readiness check.

## Checks

```bash
cd frontend
npm run build
node --test tests/retry.test.js
```

The retry tests cover transient errors, retry limits, authentication errors, cancellation, and avoiding automatic retries on other writes. The backend also has Jest and Playwright configuration; database-backed integration coverage is a next step.

## Next steps

My next priorities are tighter public-demo write restrictions, repeatable database migrations, tests for circulation and role boundaries, automated reservation expiry and queue handling, and consistent search and loan-close behavior.
