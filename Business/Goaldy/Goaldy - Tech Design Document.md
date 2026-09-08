# Goaldy Tech Design — Self-Hosted Personal Finance

| Field            | Value                |
| ---------------- | -------------------- |
| **Created**      | 2026-06-07           |
| **Last Updated** | 2026-07-02           |
| **Version**      | 4.1 (implementation reconciliation) |
| **Status**       | Active               |

> **Reset note:** Reset to a clean, forward-looking baseline on 2026-06-07. All prior architecture models (cloud encryption relay, wa-sqlite/OPFS, Google Drive sync, Postgres metadata, Free/Pro tiers) are **superseded and dropped**. This document describes exactly one architecture: a single-user, self-hosted app where a Node backend owns one SQLite file containing financial data, user settings, and preferences. The changelog starts here.

> **Implementation note (v4.1 — read before the sections below).** The production code implements this exact architecture on the existing **Next.js 15 App Router** app, not a separate Express server + Vite SPA. The mapping is 1:1:
> - Express routers → **Next.js route handlers** under `apps/web/app/api/*` (full verb coverage, same envelopes).
> - Express server + middleware → `apps/web/lib/server/*` — `db.ts` (better-sqlite3 owning `goaldy.db`), `auth.ts` (session-cookie + bearer, hashes in `settings`), `http.ts` (`withAuth` guard + envelopes), `openapi.ts`, `ingest.ts`.
> - Vite React SPA → the **Next.js App Router pages + existing React components**, kept visually unchanged. Data flows through `lib/queries/*` fetch wrappers → the REST API.
>
> Everything else in this document holds verbatim: one `goaldy.db` owned by the server, REST for every resource, one auth path, OpenAPI 3.1 + Swagger at `/api/docs`, single Docker image with a `/data` volume, and generated frontend types (`pnpm gen:api-types` → `apps/web/types/api.d.ts`). Where the text below says "Express" or "Vite", read it as the reference shape that these Next.js pieces fulfill. Repo layout is a `pnpm`/`turbo` monorepo: `apps/web` (UI + API + server layer) and `packages/financial-db` (Zod schemas, plan engine, `sql/financial.sql`).

---

## Change Log

| Version | Date       | Change                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **4.1** | 2026-07-02 | Implementation reconciliation. Shipped on the existing **Next.js 15 App Router** app rather than Express + Vite: route handlers (`app/api/*`) are the backend, the App Router + existing React UI are the frontend (preserved unchanged). Architecture otherwise as specified — one `better-sqlite3`-owned `goaldy.db`, REST for every resource, session-cookie + bearer auth, OpenAPI 3.1 at `/api/docs`, single Docker image. Deviations from the reference: PATCH-only updates (no PUT); frontend types generated from the spec; the financial schema kept from the prior app rather than §3's proposed tables (to preserve the working UI + money math). See the v4.1 implementation note at the top. |
| **4.0** | 2026-06-07 | Clean baseline. Stack: Node.js + Express, React (Vite), one SQLite file (financial data + settings + preferences), Docker. REST API for every resource with full verb coverage (GET/POST/PATCH/PUT/DELETE). Bearer-token + session auth share one path. `POST /api/transactions` is the batch, idempotent ingest primitive. OpenAPI 3.1 spec + Swagger UI required deliverable, served at `/api/docs`. All prior history collapsed and dropped. |

---

## 0. Architectural Philosophy

**No cleverness. No premature optimization. No microservices.**

This is a **single-user financial ledger that one person runs on their own server.** Every architecture decision favors:

1. Simplicity (can you explain it in one sentence?)
2. Speed (all queries < 100ms)
3. Reliability (no external dependencies)
4. Deployability (docker-compose up, done)

**Trade-offs we're making:**

- ✅ **One SQLite file for everything** → financial data, settings, and preferences in a single file the backend owns. No second store, no config service, no env-driven state that isn't reproducible from the file.
- ✅ Single-user only → eliminates user table, permissions, concurrency
- ✅ SQLite only → eliminates Postgres, migrations, connection pooling
- ✅ Bearer + session auth (one path) → eliminates OAuth, JWTs, token refresh complexity
- ✅ REST API only → eliminates GraphQL, tRPC, API-level complexity
- ✅ React SPA → eliminates SSR, Next.js routing complexity
- ✅ Docker first → eliminates deployment decisions
- ❌ No multi-device sync (user must access from same server)
- ❌ No offline support (requires server connection)
- ❌ No scalability (designed for one user, not thousands)

**These aren't gaps. They're scope boundaries.**

**The one-file rule:** everything durable lives in `goaldy.db`. That includes the password hash, the API token hash, display currency, language, date format, column-mapping memory, and every financial row. The only things _not_ in the file are process-level secrets passed at boot (`SESSION_SECRET`, optional seed `API_TOKEN`). Back up the file → back up the app. This is a hard architectural rule, not a convenience.

---

## 1. Technology Stack

| Layer          | Technology     | Version    | Why                                              |
| -------------- | -------------- | ---------- | ------------------------------------------------ |
| **Backend**    | Node.js        | 20.x LTS   | TS, async, single-threaded is fine for one user  |
| **Framework**  | Express *(impl: Next.js route handlers)* | — | Reference shape; the app uses Next.js `app/api/*` handlers + `lib/server/*` |
| **Database**   | SQLite         | 3.x        | Single file, ACID, sub-100ms for one user        |
| **Frontend**   | React          | 18.x       | Component-driven, familiar, no SSR needed        |
| **Build (FE)** | Vite *(impl: Next.js App Router)* | — | Reference shape; the app builds with Next.js (`output: standalone`) |
| **Styling**    | Tailwind CSS   | 3.x        | Utility-first, RTL support, no custom CSS        |
| **Language**   | TypeScript     | 5.x strict | Type safety everywhere, strict mode              |
| **API Client** | TanStack Query | 5.x        | Caching, refetching, error handling              |
| **Deployment** | Docker         | latest     | Single image, runs everywhere                    |

**Key dependencies:**

```json
{
  "express": "^4.x",
  "better-sqlite3": "^9.x",
  "typescript": "^5.x",
  "zod": "^3.x",
  "@asteasolutions/zod-to-openapi": "^7.x",
  "swagger-ui-express": "^5.x",
  "bcrypt": "^5.x",
  "cookie-parser": "^1.x",
  "date-fns": "^3.x",
  "react": "^18.x",
  "react-router": "^6.x",
  "@tanstack/react-query": "^5.x",
  "tailwindcss": "^3.x"
}
```

> Frontend dev dependency: `openapi-typescript` (generates `src/types/api.d.ts` from the live spec). `sqlite3` is intentionally omitted — `better-sqlite3` (synchronous) is the only driver.

**Deliberately excluded:**

- ❌ Prisma (SQL is simple enough)
- ⚠️ Next.js — the reference stack excluded it (no SSR needed), but the implementation **kept the existing Next.js App Router** to preserve the already-built UI. It is used only as a Node server (route handlers) + client-rendered React; there is no financial SSR. This is the one intentional divergence from the reference stack (see v4.1 note).
- ❌ GraphQL (REST is sufficient)
- ❌ Docker Compose networking (single container)
- ❌ Redis / caching layer (SQLite is fast enough)
- ❌ Load balancing (one user)
- ❌ Kubernetes (overkill)

---

## 2. Repository Structure

> **As-built (v4.1):** the repo is a `pnpm`/`turbo` monorepo, not the `backend/` + `frontend/` split below. The reference tree maps as: `backend/src/api/*` → `apps/web/app/api/*` route handlers; `backend/src/{db,middleware,services}` → `apps/web/lib/server/*` (`db.ts`, `auth.ts`, `http.ts`, `ingest.ts`, `openapi.ts`, `queries/*`); `frontend/src/*` → `apps/web/app` + `apps/web/components` + `apps/web/lib/{queries,hooks,api}`; shared Zod/schema/plan-engine → `packages/financial-db`. The tree below remains the conceptual reference.

```
goaldy/
├── backend/
│   ├── src/
│   │   ├── server.ts                   # Express app entry
│   │   ├── db/
│   │   │   ├── schema.ts               # SQLite schema + init
│   │   │   ├── queries.ts              # Database operations
│   │   │   └── types.ts                # TS types for DB
│   │   ├── api/
│   │   │   ├── accounts.ts             # GET/POST/PATCH/DELETE /accounts + snapshots
│   │   │   ├── transactions.ts         # CRUD + batch create + transfer detect
│   │   │   ├── tags.ts                 # Tags hierarchy + budget config
│   │   │   ├── budgets.ts              # Budget calculations (derived, read-only)
│   │   │   ├── rules.ts                # Rules engine + matching + test
│   │   │   ├── goals.ts                # Goals CRUD
│   │   │   ├── dashboards.ts           # Dashboard data (expense, income, NW)
│   │   │   ├── settings.ts             # GET/PATCH settings + preferences
│   │   │   ├── session.ts              # Login/logout + token show/rotate
│   │   │   └── openapi.ts              # OpenAPI 3.1 doc builder (from Zod)
│   │   ├── middleware/
│   │   │   ├── auth.ts                 # Session cookie OR bearer token
│   │   │   ├── error.ts                # Error handling
│   │   │   └── cors.ts                 # CORS if needed
│   │   ├── services/
│   │   │   ├── rules-engine.ts         # Apply rules to transactions
│   │   │   ├── transfer-detector.ts    # Find transfer peers
│   │   │   ├── import-parser.ts        # CSV/OFX parsing
│   │   │   ├── exchange-rates.ts       # Currency conversion
│   │   │   └── validators.ts           # Zod schemas
│   │   └── utils/
│   │       ├── logger.ts
│   │       ├── error.ts
│   │       └── helpers.ts
│   ├── migrations/                     # (SQLite doesn't need these, but doc schemas)
│   ├── Dockerfile
│   ├── tsconfig.json
│   ├── package.json
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── main.tsx                    # React entry
│   │   ├── App.tsx                     # Router setup
│   │   ├── pages/
│   │   │   ├── Login.tsx
│   │   │   ├── Dashboard.tsx           # Main expense/income/NW
│   │   │   ├── Transactions.tsx        # List + detail
│   │   │   ├── Accounts.tsx            # Account tree + detail
│   │   │   ├── Tags.tsx                # Tag hierarchy + detail
│   │   │   ├── Budgets.tsx             # Budget table + detail
│   │   │   ├── Import.tsx              # File upload + preview
│   │   │   ├── Reports.tsx             # Trends, merchants, etc.
│   │   │   └── Settings.tsx            # Rules, export, etc.
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   ├── Header.tsx
│   │   │   │   └── MainLayout.tsx
│   │   │   ├── common/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Modal.tsx
│   │   │   │   ├── Table.tsx
│   │   │   │   └── Charts.tsx
│   │   │   ├── accounts/
│   │   │   ├── transactions/
│   │   │   ├── tags/
│   │   │   ├── budgets/
│   │   │   └── dashboards/
│   │   ├── hooks/
│   │   │   ├── useApi.ts               # Wrapper around fetch + TanStack Query
│   │   │   ├── useAccounts.ts
│   │   │   ├── useTransactions.ts
│   │   │   ├── useTags.ts
│   │   │   └── ...
│   │   ├── api/
│   │   │   └── client.ts               # Axios/fetch client config
│   │   ├── utils/
│   │   │   ├── format.ts
│   │   │   ├── calc.ts
│   │   │   └── helpers.ts
│   │   ├── types/
│   │   │   └── index.ts                # Shared types (import from backend)
│   │   └── App.css                     # Tailwind imports
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   ├── package.json
│   └── .env.example
│
├── Dockerfile                          # Multi-stage build: backend + frontend
├── docker-compose.yml
├── .gitignore
├── README.md
└── pnpm-workspace.yaml                 # Monorepo (optional)
```

---

## 3. SQLite Database Schema

**Single file: `goaldy.db`. ACID compliance. All queries run locally.**

```sql
-- Accounts
CREATE TABLE accounts (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL UNIQUE,
  type TEXT NOT NULL CHECK (type IN ('checking', 'savings', 'credit_card', 'investment', 'pension', 'real_estate', 'loan', 'manual')),
  institution TEXT,
  account_number TEXT,
  currency TEXT DEFAULT 'ILS',
  balance REAL DEFAULT 0,
  color TEXT DEFAULT '#000000',
  icon TEXT,
  archived_at TIMESTAMP,
  sort_order INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Transactions
CREATE TABLE transactions (
  id TEXT PRIMARY KEY,
  account_id TEXT NOT NULL REFERENCES accounts(id),
  date DATE NOT NULL,
  type TEXT NOT NULL CHECK (type IN ('expense', 'income', 'transfer')),
  amount REAL NOT NULL,
  currency TEXT NOT NULL DEFAULT 'ILS',
  description TEXT,
  merchant TEXT,
  tag_id TEXT REFERENCES tags(id),
  status TEXT DEFAULT 'cleared' CHECK (status IN ('cleared', 'pending')),
  transfer_source_id TEXT REFERENCES accounts(id),
  transfer_dest_id TEXT REFERENCES accounts(id),
  labels JSON DEFAULT '[]',
  notes TEXT,
  import_hash TEXT UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  CHECK ((tag_id IS NOT NULL AND transfer_source_id IS NULL) OR (type = 'transfer'))
);

-- Splits (multiple tags for one transaction)
CREATE TABLE splits (
  id TEXT PRIMARY KEY,
  transaction_id TEXT NOT NULL UNIQUE REFERENCES transactions(id),
  split_data JSON NOT NULL,  -- [{ tag_id: "...", amount: 100.50 }, ...]
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tags (hierarchical)
CREATE TABLE tags (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  parent_id TEXT REFERENCES tags(id),
  color TEXT DEFAULT '#000000',
  icon TEXT,
  sort_order INTEGER DEFAULT 0,
  -- Budget config
  budget_amount REAL,
  budget_type TEXT CHECK (budget_type IN ('expense', 'income')),
  budget_period TEXT CHECK (budget_period IN ('weekly', 'biweekly', 'monthly', 'quarterly', 'custom')),
  budget_start_date DATE,
  budget_end_date DATE,
  budget_rollover BOOLEAN DEFAULT false,
  budget_account_scope TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Account snapshots (for investments, property, pension)
CREATE TABLE snapshots (
  id TEXT PRIMARY KEY,
  account_id TEXT NOT NULL REFERENCES accounts(id),
  value REAL NOT NULL,
  date DATE NOT NULL,
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Rules (merchant pattern → tag)
CREATE TABLE rules (
  id TEXT PRIMARY KEY,
  name TEXT,
  pattern_type TEXT CHECK (pattern_type IN ('contains', 'regex', 'amount_gte', 'amount_lte')),
  pattern_value TEXT NOT NULL,
  tag_id TEXT NOT NULL REFERENCES tags(id),
  enabled BOOLEAN DEFAULT true,
  sort_order INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Exchange rates (cached)
CREATE TABLE exchange_rates (
  id TEXT PRIMARY KEY,
  from_currency TEXT NOT NULL,
  to_currency TEXT NOT NULL,
  rate REAL NOT NULL,
  date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(from_currency, to_currency, date)
);

-- Sessions (one row per browser login)
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  user_agent TEXT,
  ip_address TEXT,
  last_activity TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP NOT NULL
);

-- Goals (basic Type A savings goal for Phase 0)
CREATE TABLE goals (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  target_amount REAL NOT NULL,
  target_date DATE,
  tag_id TEXT REFERENCES tags(id),
  current_amount REAL DEFAULT 0,
  status TEXT DEFAULT 'active' CHECK (status IN ('active', 'completed', 'abandoned')),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Settings (single-row key/value store: password hash, API token hash, prefs)
CREATE TABLE settings (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- Reserved keys: 'password_hash', 'api_token_hash', 'display_currency', 'language', 'date_format'
```

**Key design decisions:**

- Single user → no user table
- `import_hash` prevents duplicates — SHA-256 of `account_id | date | amount | description | externalId?`, stored as a **BLOB** with a UNIQUE index (see §4.4b for the three dedup modes and the null-hash rule)
- `type = 'transfer'` is atomic, no splits
- `tag_id` is nullable for transfers
- Budget config lives on tags (not separate table)
- Exchange rates cached to avoid API calls on every conversion
- `settings` holds the bcrypt password hash and the hashed API bearer token — no user table needed
- The API token is the integration credential; it and the session cookie share one auth path

---

## 4. Backend Architecture

### 4.1 Server Setup

```typescript
// src/server.ts
import express from "express";
import { initDb } from "./db/schema";
import { authMiddleware } from "./middleware/auth";
import accountsRouter from "./api/accounts";
import transactionsRouter from "./api/transactions";
import tagsRouter from "./api/tags";
import goalsRouter from "./api/goals";
// ... more routes

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());
app.use(express.static("frontend/dist")); // Serve React build
app.use(authMiddleware); // Protect all /api routes

// Routes
app.post("/session", sessionController.login);
app.delete("/session", sessionController.logout);
app.use("/api/accounts", accountsRouter);
app.use("/api/transactions", transactionsRouter);
app.use("/api/tags", tagsRouter);
app.use("/api/budgets", budgetsRouter);
app.use("/api/rules", rulesRouter);
app.use("/api/goals", goalsRouter);
app.use("/api/dashboards", dashboardsRouter);
// No /api/ingest — integrations post to /api/transactions

// Error handler
app.use(errorHandler);

// Start
initDb().then(() => {
  app.listen(PORT, () => console.log(`Server on ${PORT}`));
});
```

### 4.2 Database Layer

```typescript
// src/db/queries.ts
import Database from "better-sqlite3";

const db = new Database("data/goaldy.db");

export const queries = {
  // Accounts
  getAccounts: () =>
    db.prepare("SELECT * FROM accounts ORDER BY sort_order").all(),
  getAccount: (id: string) =>
    db.prepare("SELECT * FROM accounts WHERE id = ?").get(id),
  createAccount: (data: Account) =>
    db.prepare("INSERT INTO accounts (...) VALUES (...)").run(data),
  updateAccount: (id: string, data: Partial<Account>) =>
    db.prepare("UPDATE accounts SET ... WHERE id = ?").run(data, id),

  // Transactions
  getTransactions: (filters: { accountId?; tagId?; dateFrom?; dateTo? }) => {
    // Build dynamic query with filters
    return db
      .prepare(buildQuery("SELECT * FROM transactions WHERE ...", filters))
      .all();
  },
  createTransaction: (data: Transaction) => {
    /* ... */
  },
  updateTransaction: (id: string, data: Partial<Transaction>) => {
    /* ... */
  },
  deleteTransaction: (id: string) => {
    /* ... */
  },

  // Tags
  getTags: () =>
    db.prepare("SELECT * FROM tags ORDER BY parent_id, sort_order").all(),

  // ... all CRUD operations
};
```

**Use `better-sqlite3` (synchronous):** Simpler than sqlite3 (callback-based), single-threaded is fine for one user, better performance.

### 4.3 API Routes (REST)

```typescript
// src/api/accounts.ts
import express from "express";
import { queries } from "../db/queries";
import { validateAccount } from "../services/validators";

const router = express.Router();

// GET /api/accounts
router.get("/", (req, res) => {
  const accounts = queries.getAccounts();
  res.json(accounts);
});

// POST /api/accounts
router.post("/", (req, res) => {
  const { data, error } = validateAccount(req.body);
  if (error) return res.status(400).json({ error });
  const account = queries.createAccount(data);
  res.status(201).json(account);
});

// PATCH /api/accounts/:id  — partial update (only send changed fields)
router.patch("/:id", (req, res) => {
  const { data, error } = validateAccount(req.body, { partial: true });
  if (error) return res.status(400).json({ error });
  const account = queries.updateAccount(req.params.id, data);
  if (!account) return res.status(404).json({ error: "Not found" });
  res.json(account);
});

// DELETE /api/accounts/:id
router.delete("/:id", (req, res) => {
  const existed = queries.deleteAccount(req.params.id);
  if (!existed) return res.status(404).json({ error: "Not found" });
  res.status(204).send();
});

export default router;
```

**Pattern:** Every resource (accounts, transactions, tags, goals, rules, snapshots) gets the full verb set:

- `GET /api/entity` → list (paginated, filterable)
- `POST /api/entity` → create (transactions also accept an array)
- `GET /api/entity/:id` → detail
- `PATCH /api/entity/:id` → partial update (send only changed fields)
- `DELETE /api/entity/:id` → delete
- Plus resource-specific actions (e.g., `POST /api/transactions/:id/transfers/find`, `POST /api/token/rotate`)

**Verb convention:** `PATCH` is the update verb — partial, idempotent, only the fields you send. We do **not** use `PUT` (no full-resource replacement semantics needed for a single-user app; PATCH covers every real edit). `POST` creates, `GET` reads, `DELETE` removes. This convention is enforced identically across every resource so the OpenAPI spec (§4.6) is mechanical to generate and the frontend client is mechanical to consume.

### 4.4 Batch + Idempotent Transaction Create (the universal ingest primitive)

`POST /api/transactions` accepts a single object or an array. It dedups on `import_hash` and runs the rules engine. This one endpoint serves the UI, Moneyman, scripts, and webhooks identically.

```typescript
// src/api/transactions.ts
import { createHash } from "crypto";

function importHash(tx: TxInput): string {
  return createHash("sha256")
    .update(`${tx.date}|${tx.amount}|${tx.description}|${tx.account_id}`)
    .digest("hex");
}

// POST /api/transactions  — single object OR array
router.post("/", (req, res) => {
  const input = Array.isArray(req.body) ? req.body : [req.body];

  let created = 0,
    skipped = 0;
  const errors: { index: number; message: string }[] = [];

  // Wrap the whole batch in one SQLite transaction for speed + atomicity
  const runBatch = db.transaction((items: TxInput[]) => {
    items.forEach((raw, index) => {
      try {
        const { data, error } = validateTransaction(raw); // Zod
        if (error) {
          errors.push({ index, message: error });
          return;
        }

        const hash = importHash(data);
        if (queries.getTransactionByHash(hash)) {
          skipped++;
          return;
        } // idempotent

        const tag = data.tag_id ? { id: data.tag_id } : rulesEngine.apply(data); // auto-tag

        queries.createTransaction({
          ...data,
          tag_id: tag?.id,
          import_hash: hash,
        });
        created++;
      } catch (e) {
        errors.push({ index, message: (e as Error).message });
      }
    });
  });

  runBatch(input);

  res.status(created > 0 ? 201 : 200).json({ created, skipped, errors });
});
```

**Notes:**

- `account_id` must already be a valid Goaldy account. Callers resolve their own source-account identifiers to Goaldy accounts before posting (see PRD §5.3). The server does not embed scraper-specific mapping.
- Dedup is idempotent: re-posting an overlapping date range creates zero duplicates.
- The batch runs inside one `db.transaction()` — 200 rows insert in a few ms and either all validation-passing rows commit or the process errors cleanly.
- Auth is the standard bearer token or session (see §4.5), not a bespoke API key check.

### 4.4a File Import — the four-stage pipeline

`POST /api/transactions` (§4.4) is the ingest primitive for structured data that is already
in Goaldy's shape. **File import is a layer above it**, because a bank's CSV, an Excel
workbook or a QIF file has to be turned into that shape first — and the user has to see what
will happen before it does.

**Formats and entities are independent axes**, which is the whole architecture. The old
design had one adapter per *vendor*, so every combination of source and destination needed
its own code, and a vendor changing its export silently broke detection. Now:

| Stage | Module | Input → output | Touches the DB? |
|---|---|---|---|
| Read | `readers/` (csv, xlsx, json, qif) | bytes + `ParseOptions` → `SourceTable` | No |
| Map | `mappers/` (transactions, accounts, tags, budgets) | `SourceTable` + `MapConfig` → `Normalized*[]` | No |
| Resolve | `refs/` | natural keys → ids, per binding | Reads (`plan`) or writes (`commit`) |
| Plan / commit | `run.ts` | resolved rows → `ImportPlan` or `ImportResult` | Reads only / writes |

A reader knows nothing about entities; a mapper knows nothing about formats. **Adding OFX is
one new reader** — no wizard change, because the UI renders from the format and entity
registries rather than from hardcoded per-vendor knowledge.

**`planRun` and `commitRun` share one `prepare()`**, differing only in the `ResolveMode`
passed down. That is the structural guarantee that the preview predicts the commit: they
cannot drift into two resolution policies, because there is only one.

**One file is one entity per run.** That decision buys reference resolution against the live
database — no in-flight symbol table, no dependency graph inside a commit — and it is what
makes order independence achievable: every entity import is an **upsert on its natural key**
(accounts by name, then account number as a rename fallback; tags by full path; transactions
by `import_hash`), never a blind insert.

**The one exception to order independence** is an account's opening balance.
`adjustBalance({ method: 'opening' })` recalibrates an account's whole total, so applying it
to an account that already carries imported transactions would erase that history from the
reported balance. It is therefore applied only by the run that *created* the account, and
skipped with a per-account warning otherwise. The entity data converges in any order; this
one derived figure cannot, and `buxfer-acceptance.test.ts` pins both halves against a real
5,734-row export.

**Cross-references are one mechanism.** A reference field's value is resolved against
existing rows per a `ReferenceBinding` (`matchBy`, `onMissing`, `onAmbiguous`). Resolution
answers "what happened to this *value*"; a separate pure step (`apply.ts`) answers "what
happens to this *row*", which is the only place `onAmbiguous` and `onMissing: skipRow |
leaveNull` are consumed. **Ambiguity never creates** — minting a duplicate to resolve an
ambiguity is how an import corrupts a ledger irreversibly.

**The required-field gate has one implementation and two callers.** `resolveState` /
`isSatisfied` (`mappers/resolve.ts`) is read by the wizard's disabled Continue *and* by
`ImportConfigSchema`'s `superRefine`, so the affordance and the guarantee cannot disagree. A
required field is satisfied by a column **or** a constant; the API returns 400 naming every
unresolved field, with machine field keys alongside the prose so the ledger can highlight
rows without parsing translated text.

**Requirements a per-field boolean cannot express are declared as data.** A transaction's
amount is satisfied by `amount` *or* `amountOut`/`amountIn`; a tag by `path` *or* `name`.
Both live on the mapper as `alternations`, read by the gate and the ledger alike, so a new
entity cannot forget one.

**Presets are data.** `presets/` holds built-ins (Mint, YNAB, Buxfer × 4) and the user's
saved mappings in one shape, applied by one code path. Columns are addressed by **header
name**, never index, so a vendor reordering or inserting a column cannot silently remap
every field after it. Detection is a strict superset check over header hints, never a fuzzy
score: a confidently wrong preset is worse than none.

**Format detection** sniffs content before extension — a bank that names its Excel export
`.csv` is common enough that trusting the extension misroutes the file. Detection returns
`null` rather than guessing when the shape is ambiguous, and the wizard asks.

**A commit is one transaction**, via `withTransaction()` (`db.ts`) rather than
`db.transaction()`, which takes a synchronous callback and would commit before an async
pipeline finished. The rule it enforces: **structural failures roll back, per-row rejections
still degrade** — a rejected budget row, a malformed tag path and an inapplicable opening
balance all warn and continue, because turning those into total import failures would be
strictly worse than the partial write atomicity prevents.

### 4.4b Dedup modes

`import_hash` is `sha256(account_id | date | amount | description | externalId?)`, stored as a
**BLOB** with a UNIQUE index. The insert carries `ON CONFLICT(import_hash) DO NOTHING`, and a
dedup hit is detected via `info.changes === 0`. The conflict is **targeted** deliberately:
untargeted, it swallowed every unique violation on the table, so a duplicate primary key
silently dropped a distinct transaction and was miscounted as a dedup skip.

| Mode | Behaviour |
|---|---|
| `skip` (default) | A colliding row is dropped. Every pre-existing caller gets this. |
| `all` | Every row is inserted regardless of collision. |
| `review` | A colliding row is re-inserted **only if its hash appears in the request's `keepHashes` allow-list**. |

`review` exists because the preview shows a sample, not the whole file. Resolving it as "any
tick means `all`" would bypass the hash check for duplicates the user never saw — on a large
re-import, silently double-counting history. The allow-list scopes the bypass to exactly the
rows the user ticked.

A deliberately re-admitted row is stored with a **null** hash, because the column is UNIQUE
and the same hash cannot be written twice. The original row keeps its hash, so future imports
continue to dedup normally.

### 4.4c Export — two profiles, one serializer registry

`lib/server/exporters/` mirrors the importer registry's shape. Each entity module exports
`columns` (natural-key names, never an internal id) and `rows(filters)`, calling the same
`lib/server/queries/*` the UI uses — **no exporter writes SQL of its own**, for the same
reason no adapter does: money math lives in one place.

`GET /api/export?entity&format&profile` serves both profiles.

- **`portable`** — natural keys, lossy by construction. An account is referenced by name, a
  tag by its slash path. This is what a human reads, diffs, or hands to another tool.
- **`backup`** — `buildBackupDocument()`, every financial table verbatim via `SELECT *`. This
  is the one place `SELECT *` is correct: a backup must carry whatever columns the table has,
  and pinning a list would silently drop any column added later.

`backup` is valid only as `entity=all&format=json` (a partial backup is a restore that cannot
succeed), and `entity=all` is JSON-only. The route enforces both with a 400, and the modal
cannot construct either combination.

BLOBs are hex-encoded, detected via `PRAGMA table_info` rather than a hardcoded column name.
`settings`, `sessions` and `meta` are excluded from `BACKUP_TABLES` — they hold the password
hash, API token and live session of the exporting machine. **A backup is a financial artifact,
not a credential dump.**

`LEDGER_TABLES` is `BACKUP_TABLES` minus `plan_settings`, and exists solely for the "is this
ledger empty?" question. `plan_settings` is a lazily-created singleton that `delete-all`
preserves, so counting it there would report "not empty" forever and make restore permanently
unreachable.

### 4.4d Restore — deliberately not an import adapter

Restore is a separate pipeline, not an adapter. An adapter mints new ids and matches by name;
byte-exactness forbids both. `preflightRestore` decides, `restoreBackup` executes.

**Preflight refuses rather than half-applies:**

| Reason | Status |
|---|---|
| `malformed` — not an object, no `tables`, a table name outside `BACKUP_TABLES`, or a row that is not an object | 400 |
| `schema-mismatch` — the document's `schemaVersion` differs from `meta.schema_version` | 400 |
| `ledger-not-empty` — counted over `LEDGER_TABLES` | 409, carrying per-table destroy counts so the UI's wipe-and-restore prompt states its blast radius rather than asserting it |

**The wipe and the restore are ONE transaction.** They used to be two — `deleteAll
FinancialData()` opened its own, then `restoreBackup()` opened another — so a document that
cleared preflight and then failed the restore's own `foreign_key_check` left the user with an
empty ledger and nothing restored, strictly worse than either outcome alone. The gap is
reachable because preflight validates shape and schema version while the FK check runs inside
the restore. Both halves now expose transaction-participating forms so the route owns a single
transaction over the pair; the unknown-table validation lives in the *inner* function, so a
caller reaching for that form cannot skip an injection guard.

**Restore runs as one transaction** with `PRAGMA defer_foreign_keys = ON` (not
`foreign_keys = OFF`, which is an outside-transaction control and would let a `DROP` cascade
silently). `tags.parent_id` is self-referential, so a child may legitimately appear before its
parent. `PRAGMA foreign_key_check` runs before commit; any violation throws and rolls the
whole thing back.

**Both table AND column names from the document are whitelisted** against the live schema.
SQLite cannot bind an identifier, so a row key reaches the SQL string verbatim — an
unvalidated one can close the column list early and append arbitrary SQL. That was a real
vulnerability during development: a crafted document read the install's password hash out of
`settings` and wrote it into a restored ledger row, crossing the exact boundary the excluded-
tables rule exists to hold. Authentication is not the control here; a restore endpoint exists
to accept a file the user obtained elsewhere.

`plan_settings` restores with `INSERT OR REPLACE` because the wipe preserves it; every other
table uses a plain `INSERT`, where a collision is a real error that should roll back.

### 4.4e The round-trip benchmark

`lib/server/restore/roundtrip.test.ts` is the acceptance gate for the guarantee, and CI-gating
since CI already runs `pnpm test`. Seed an adversarial ledger → export → delete-all → restore →
**table-by-table full-row diff**. Never a spot-check: a spot-check is how a dropped split
weight or a mangled hash slips through.

It also asserts, by reading `goaldy.sql` directly, that every `CREATE TABLE` is either in
`BACKUP_TABLES` or one of the three deliberately excluded tables. Without that, the diff
polices its own list — a table dropped from `BACKUP_TABLES` disappears from both sides of the
comparison and the gate stays green.

### 4.5 Auth Middleware (session cookie OR bearer token)

```typescript
// src/middleware/auth.ts
export function authMiddleware(req, res, next) {
  if (req.path === "/health" || req.path === "/api/session") return next();

  // Machine clients: Authorization: Bearer <token>
  const bearer = req.headers.authorization?.split(" ")[1];
  if (bearer && bearer === getApiToken()) return next();

  // Browser clients: signed session cookie
  if (req.cookies?.session && isValidSession(req.cookies.session))
    return next();

  return res.status(401).json({ error: "Unauthorized" });
}
```

Both paths grant identical full access — single user, single scope. The API token is generated at first run, stored hashed in the `settings` table, surfaced in Settings → Integrations, and rotatable via `POST /api/token/rotate`.

### 4.6 OpenAPI Spec + Swagger UI (required deliverable)

The backend is the single source of API truth. It ships a complete **OpenAPI 3.1** document describing every resource, every verb, every request/response schema, and the two auth schemes. Swagger UI renders it in-app.

**Endpoints:**

```
GET /api/openapi.json     # The raw OpenAPI 3.1 document (machine-readable)
GET /api/docs             # Swagger UI (human-readable, "try it out" enabled)
```

**How the spec is produced — schema-first, zero drift:**

Zod schemas already validate every request body (§4.3). We reuse them as the single definition and generate OpenAPI from them with `@asteasolutions/zod-to-openapi`. This means the validation logic and the documentation come from the _same_ source — the docs cannot drift from what the server actually accepts, because they're generated from the code path that enforces it.

```typescript
// src/api/openapi.ts
import {
  OpenAPIRegistry,
  OpenApiGeneratorV31,
} from "@asteasolutions/zod-to-openapi";
import {
  AccountSchema,
  TransactionSchema,
  TagSchema,
  GoalSchema,
  RuleSchema,
  SnapshotSchema,
} from "../services/validators";

const registry = new OpenAPIRegistry();

// Register the bearer + cookie auth schemes
registry.registerComponent("securitySchemes", "bearerAuth", {
  type: "http",
  scheme: "bearer",
  description: "Integration token (Settings → Integrations)",
});
registry.registerComponent("securitySchemes", "cookieAuth", {
  type: "apiKey",
  in: "cookie",
  name: "session",
});

// Register each resource schema once; reuse across paths
const Account = registry.register("Account", AccountSchema);
const Transaction = registry.register("Transaction", TransactionSchema);
// ... Tag, Goal, Rule, Snapshot

// Register paths (one helper call per resource generates all 5 verbs)
registerCrud(registry, {
  path: "/api/accounts",
  tag: "Accounts",
  schema: Account,
  listFilters: ["archived", "type"],
});
registerCrud(registry, {
  path: "/api/transactions",
  tag: "Transactions",
  schema: Transaction,
  listFilters: [
    "account_id",
    "tag_id",
    "type",
    "date_from",
    "date_to",
    "limit",
    "offset",
  ],
  batchCreate: true, // documents the array-or-object body + {created,skipped,errors} response
});
// ... tags, goals, rules

export function buildOpenApiDocument() {
  const generator = new OpenApiGeneratorV31(registry.definitions);
  return generator.generateDocument({
    openapi: "3.1.0",
    info: {
      title: "Goaldy API",
      version: "1.0.0",
      description:
        "Self-hosted personal finance. Single user. One SQLite file. Every resource, every verb.",
    },
    servers: [{ url: "/" }],
    security: [{ bearerAuth: [] }, { cookieAuth: [] }],
  });
}
```

```typescript
// wire into server.ts
import swaggerUi from "swagger-ui-express";
import { buildOpenApiDocument } from "./api/openapi";

const openApiDoc = buildOpenApiDocument();
app.get("/api/openapi.json", (_req, res) => res.json(openApiDoc));
app.use("/api/docs", swaggerUi.serve, swaggerUi.setup(openApiDoc));
```

**Coverage requirement (non-negotiable):** the spec MUST document, for every resource below, all of `GET (list)`, `POST (create)`, `GET /:id`, `PATCH /:id`, `DELETE /:id`, plus resource-specific actions:

| Resource     | Base path           | Resource-specific actions                                                             |
| ------------ | ------------------- | ------------------------------------------------------------------------------------- |
| Accounts     | `/api/accounts`     | `GET/POST /:id/snapshots`                                                             |
| Transactions | `/api/transactions` | `POST` accepts array (batch); `POST /:id/transfers/find`                              |
| Tags         | `/api/tags`         | `GET /:id/transactions`, `GET /:id/budget-status`                                     |
| Budgets      | `/api/budgets`      | `GET /summary`, `GET /:tag_id` (read-only; derived from tags)                         |
| Rules        | `/api/rules`        | `POST /test`                                                                          |
| Goals        | `/api/goals`        | —                                                                                     |
| Dashboards   | `/api/dashboards`   | `GET /expense`, `/income`, `/net-worth`, `/merchants` (read-only)                     |
| Settings     | `/api/settings`     | `GET`, `PATCH` (currency, language, date format)                                      |
| System       | —                   | `GET /health`, `POST/DELETE /api/session`, `GET /api/token`, `POST /api/token/rotate` |

**Response envelope conventions (documented in the spec):**

- List endpoints: `{ "data": [...], "total": N, "limit": L, "offset": O }`
- Errors: `{ "error": "message", "details"?: [...] }` with correct HTTP status (400 validation, 401 auth, 404 missing, 409 conflict)
- Batch create: `{ "created": N, "skipped": M, "errors": [{ "index": i, "message": "..." }] }`

**Client generation (frontend consumes, never hand-writes types):** the frontend's API types are generated from `/api/openapi.json` via `openapi-typescript`, so `frontend/src/types/api.d.ts` is a build artifact, not hand-maintained. Backend schema change → regenerate → TypeScript compile errors surface every consumer that needs updating. This is the mechanism that keeps the "backend exposes it all, frontend consumes it" contract honest.

```
# frontend build step
npx openapi-typescript http://localhost:3000/api/openapi.json -o src/types/api.d.ts
```

---

## 5. Frontend Architecture

### 5.1 React Structure

```typescript
// src/App.tsx
import { BrowserRouter, Routes, Route } from 'react-router';
import { QueryClientProvider } from '@tanstack/react-query';
import MainLayout from './components/layout/MainLayout';
import Login from './pages/Login';
import Dashboard from './pages/Dashboard';
import Transactions from './pages/Transactions';
import Accounts from './pages/Accounts';
// ... other pages

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <Routes>
          <Route path="/login" element={<Login />} />
          <Route element={<MainLayout />}>
            <Route path="/" element={<Dashboard />} />
            <Route path="/transactions" element={<Transactions />} />
            <Route path="/accounts" element={<Accounts />} />
            {/* ... more routes */}
          </Route>
        </Routes>
      </BrowserRouter>
    </QueryClientProvider>
  );
}
```

### 5.2 Data Fetching with TanStack Query

```typescript
// src/hooks/useApi.ts
import { useQuery, useMutation } from '@tanstack/react-query';

export function useAccounts() {
  return useQuery({
    queryKey: ['accounts'],
    queryFn: () => fetch('/api/accounts').then(r => r.json()),
  });
}

export function useCreateAccount() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data) => fetch('/api/accounts', { method: 'POST', body: JSON.stringify(data) }).then(r => r.json()),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['accounts'] }),
  });
}

// Usage in component
function Accounts() {
  const { data: accounts, isLoading } = useAccounts();
  const createAccount = useCreateAccount();

  if (isLoading) return <p>Loading...</p>;

  return (
    <div>
      {accounts.map(acc => <AccountItem key={acc.id} account={acc} />)}
    </div>
  );
}
```

**Benefits:**

- Automatic caching
- Refetch on focus
- Mutations auto-invalidate related queries
- Loading/error states
- No manual state management

### 5.3 Components

**Sidebar (account tree):**

```typescript
// src/components/layout/Sidebar.tsx
export default function Sidebar() {
  const { data: accounts } = useAccounts();

  const grouped = groupByLiquidity(accounts);

  return (
    <aside className="sidebar">
      <div className="net-worth">${netWorth}</div>
      {Object.entries(grouped).map(([group, accs]) => (
        <details key={group} open>
          <summary>{group}</summary>
          {accs.map(acc => (
            <Link to={`/accounts/${acc.id}`} key={acc.id}>
              {acc.name} <span>{acc.balance}</span>
            </Link>
          ))}
        </details>
      ))}
    </aside>
  );
}
```

**Dashboard:**

```typescript
// src/pages/Dashboard.tsx
export default function Dashboard() {
  const { data: expenses } = useQuery({
    queryKey: ['dashboard', 'expense'],
    queryFn: () => fetch('/api/dashboards/expense').then(r => r.json()),
  });

  return (
    <div className="dashboard">
      <PieChart data={expenses} />
      <BudgetTable />
      <Trends />
    </div>
  );
}
```

### 5.4 Styling

**Tailwind + custom tokens:**

```css
/* src/App.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .card {
    @apply bg-white border border-gray-200 rounded-lg p-4;
  }

  .button-primary {
    @apply px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700;
  }
}

/* RTL support */
[dir="rtl"] .sidebar {
  @apply right-0;
}
```

---

## 6. Docker Deployment

### 6.1 Dockerfile (Multi-stage)

```dockerfile
# Stage 1: Build backend
FROM node:20-alpine AS backend-build
WORKDIR /app/backend
COPY backend/package*.json ./
RUN npm ci
COPY backend .
RUN npm run build

# Stage 2: Build frontend
FROM node:20-alpine AS frontend-build
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm ci
COPY frontend .
RUN npm run build

# Stage 3: Runtime
FROM node:20-alpine
WORKDIR /app
RUN mkdir -p /data

# Copy backend
COPY --from=backend-build /app/backend/dist ./backend
COPY --from=backend-build /app/backend/package*.json ./

# Copy frontend to backend's public directory
COPY --from=frontend-build /app/frontend/dist ./backend/public

# Install production dependencies only
RUN npm ci --production

EXPOSE 3000
ENV NODE_ENV=production \
    DATABASE_URL=/data/goaldy.db \
    PORT=3000

VOLUME /data

CMD ["node", "backend/index.js"]
```

### 6.2 docker-compose.yml

```yaml
version: "3.9"
services:
  goaldy:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      SESSION_SECRET: ${SESSION_SECRET:-change-me-in-production}
      # Optional: seed the API bearer token. If unset, one is generated at first
      # run and shown in Settings → Integrations. Used by ALL integrations
      # (Moneyman, scripts, webhooks) against POST /api/transactions.
      API_TOKEN: ${API_TOKEN:-}
    volumes:
      - goaldy-data:/data
    restart: unless-stopped

volumes:
  goaldy-data:
    driver: local
```

**Run:**

```bash
docker-compose up -d
# Goaldy available at http://localhost:3000
# Data persists to /data/goaldy.db (survives restart)
```

---

## 7. Development Workflow

### Setup

```bash
git clone <repo>
cd goaldy

# Backend
cd backend
npm install
npm run dev  # Port 3000

# Frontend (new terminal)
cd frontend
npm install
npm run dev  # Port 5173 (Vite)

# Frontend will proxy /api/* to http://localhost:3000
```

### Build & Deploy

```bash
# Docker build (automatic on push if using CI)
docker build -t goaldy:latest .

# Run locally
docker-compose up -d

# Access at http://localhost:3000
```

### Testing

```bash
# Backend: Unit tests for queries, rules engine, validators
npm run test

# Frontend: Component tests, integration tests
npm run test

# E2E (Playwright): Login, import, create transaction, etc.
npm run test:e2e
```

---

## 8. Security & Performance

### Security

**Session auth (browser):**

- User logs in with password (bcrypt-hashed, stored in `settings`)
- Server creates a session row; token delivered as a **httpOnly, SameSite=Lax, Secure** cookie
- Session expires after 30 days (configurable); expiry returns 401 → frontend redirects to login
- CSRF: because auth is a SameSite cookie, cross-site form posts are blocked; state-changing routes also require the `application/json` content type (a second cheap guard)

**API bearer token (machine clients):**

- Single token, generated at first run or seeded via `API_TOKEN` env var
- Stored **hashed** in `settings` (`api_token_hash`); compared on each request
- Sent as `Authorization: Bearer <token>`; used by Moneyman adapters, scripts, webhooks
- Rotatable via `POST /api/token/rotate` (invalidates the old token)
- No JWT, no refresh flow — one long-lived token for a single-user self-hosted box

**Both auth paths grant identical access** (single user, single scope). Every `/api/*` route except `/health` and `/api/session` accepts either a valid session cookie or a valid bearer token.

**Database:**

- SQLite (single process, no network exposure)
- Stored in `/data` volume (user's server only)
- No sensitive data outside database

### Performance

**Queries:**

- All indexed on common filters (account_id, tag_id, date)
- Pagination on list endpoints (limit 50)
- SQLite handles 10k+ transactions fine

**Caching (frontend):**

- TanStack Query caches API responses
- Refetch on window focus
- Manual invalidation on mutations

**Static files:**

- React build (CSS, JS, HTML) served from Express
- Gzip compression via middleware
- Browser caching via Cache-Control headers

---

## 9. Limitations & Future Considerations

| Limitation     | Why                                        | Phase?              |
| -------------- | ------------------------------------------ | ------------------- |
| Single user    | Scope gate — no user table, no permissions | Always              |
| No offline     | Requires server connection                 | Phase 2 (if needed) |
| No sync        | Data lives on one server                   | Phase 2 (if needed) |
| SQLite only    | Not designed for concurrent writes         | OK for 1 user       |
| Single process | No clustering                              | OK for 1 user       |

**If demand emerges for multi-user:**

- Add `users` table
- Add `user_id` foreign key to all data tables
- Implement row-level security
- Switch to Postgres
- Add real auth (OAuth, SAML)
- Deploy with process manager (PM2)

**For now:** None of this is needed.

---

## 10. Deployment Targets

**Tested on:**

- Docker Desktop (local)
- Synology NAS (Docker support)
- Home server (Ubuntu + Docker)
- VPS (Linode, Hetzner, DigitalOcean)

**Not tested:**

- Kubernetes (overkill)
- ARM (Raspberry Pi) — probably works but untested
- Windows (use WSL2)

---

_Created: 2026-06-07. Classic backend/frontend. No magic. No scale. Just a financial ledger._
