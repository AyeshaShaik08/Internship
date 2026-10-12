# Ashoka Women’s Engineering College Library

JNTU affiliated · Created by Ayesha Shaik

A local-first library operations dashboard built with React, Express, Prisma, and SQLite. The database is a file on the laptop; no hosted database or backend service is used.

## Prerequisites

- Node.js 20 or later
- npm 10 or later

## Install and run

From the repository root:

```powershell
npm install
npm install --prefix server
npm install --prefix client
Copy-Item server/.env.example server/.env
npm run db:setup
npm run dev
```

The dashboard runs at `http://localhost:5173`. The REST API runs at `http://localhost:4000`.

`server/.env` is local and ignored by Git. The default `DATABASE_URL="file:./library.db"` resolves to `server/prisma/library.db` alongside the Prisma schema. To change the file location, update `DATABASE_URL` in `server/.env` using a SQLite `file:` URL.

## Database commands

```powershell
npm run db:migrate
npm run db:seed
npm --prefix server run db:generate
npm run db:setup
npm run db:populate-academic
```

`db:setup` is for a fresh demo database: it applies migrations and seeds 38 academic B.Tech textbooks, 8 common-collection titles, 12 members, and 16 issue/return records. Academic books include branch, B.Tech year, semesters 1–8, subject, shelf, and volume/part; Common books are marked All Years, All Branches, and Not Applicable for semester and volume. Re-running `db:seed` resets sample records.

For an existing library database, use this non-destructive sequence instead:

```powershell
npm run db:migrate
npm --prefix server run db:generate
npm run db:populate-academic
```

`db:populate-academic` inserts the 38 B.Tech records by ISBN, skips matching records already present, and refuses conflicting ISBNs. It does not modify members, transactions, or other books. It is safe to run repeatedly. Do not run `db:seed` or `db:setup` against a database whose records must be retained.

Set `DAILY_FINE` in `server/.env` to configure the fine per overdue day. The default is `₹0.50`. Other settings are `PORT` and `CLIENT_ORIGIN`.

## Features

- Dashboard with a quick desk guide, title/copy counts, active members, circulation rate, subject availability, recent activity, and overdue follow-up.
- Book catalogue with Academic and Common collections, B.Tech year/branch/semester/subject/volume filters and columns, year-prefilled dashboard shortcuts, searchable paginated results, details, edit and safe delete, and a cover fallback.
- Member directory with search, type/status filters, profile and borrowing history; edit, safe deactivate, and delete.
- Issue workflow for active members and available titles, custom due dates, transaction history, returns, overdue status promotion, and late fine calculation.
- Library identity and contact settings.
- Responsive navigation and tables, shared forms, validation feedback, confirmation dialogs, and toast notifications.

Inventory changes and issue/return records are committed in Prisma transactions. Available-copy counts are conditionally decremented, and records with active issues cannot be removed or deactivated. Historical transaction relations are retained, so a member or book with borrowing history cannot be hard-deleted; members can be safely deactivated after open loans are returned.

## API overview

All endpoints are prefixed with `/api`; successful responses use `{ "data": ... }`, with pagination metadata under `meta`.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/health` | API health |
| GET | `/dashboard` | Collection, circulation, and overdue summary |
| GET, POST | `/books` | Search/filter/page or create books |
| GET, PATCH, DELETE | `/books/:id` | Read, update, or safely delete a book |
| GET, POST | `/members` | Search/filter/page or create members |
| GET, PATCH, DELETE | `/members/:id` | Read profile/history, update, deactivate, or delete |
| GET | `/transactions` | Filter circulation history |
| POST | `/transactions/issue` | Issue an available title to an active member |
| GET | `/transactions/:id` | Read a transaction |
| POST | `/transactions/:id/return` | Return a title and calculate any fine |
| GET, PUT | `/settings` | Read or update library details and read the configured daily fine |

Book and member collection endpoints accept `search`, `page`, `pageSize`, `sort`, and `direction`; books also accept `collectionType`, `academicYear`, `branch`, `semester`, `volume`, `category`, and `availability`, while members accept `status` and `type`. Transactions accept `status`, `memberId`, `bookId`, `from`, `to`, `overdue`, and `search`.

## Verification

```powershell
npm --prefix server test
npm run build
```

The API integration test exercises current B.Tech catalog presence, safe Common defaults, Academic/Common creation, year/branch/semester/volume filtering, dashboard year counts, input validation, unique constraints, book/member CRUD, issue/return inventory updates, duplicate and inactive-member protection, overdue calculation, fines, dashboard totals, API error handling, and database persistence after restarting the Express process.
