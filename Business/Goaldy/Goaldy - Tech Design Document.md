| Field            | Value           |
| ---------------- | --------------- |
| **Created**      | 2026-04-03      |
| **Last Updated** | 2026-04-05 v1.9 |
| **Version**      | 1.9             |
| **Status**       | Draft           |

### Change Log

|Version|Date|Change|
|---|---|---|
|1.0|2026-04-03|Initial tech design — all 11 pillars + 8 additions from design review|
|1.1|2026-04-03|Replaced `pending_sync` table with encrypted blob queue pattern (Section 4 + Ingest Surface A/B); generalised pattern to all async ingest sources|
|1.2|2026-04-03|Elevated architectural thesis to Section 0 — Model 4 differentiation table, one-sentence pitch, full flow diagram|
|1.3|2026-04-05|SQLite schema updated: `transaction_tags` join table removed; `tag_id` moved to `transactions` row; `transaction_splits` replaces join table for split case; `labels` JSON column added to `transactions`; `is_income` removed from `tags`; full budget configuration columns added to `tags` table|
|1.4|2026-04-05|Transfer detection added: `transfer_review` table added to SQLite schema; transfer detection pass added to ingest pipeline with TypeScript pseudocode; no `type` field on transactions — transfers are a UI state resolved by deletion, not a data type|
|1.5|2026-04-05|Transfer model corrected: `type TEXT` field added to transactions ('normal' or 'transfer'); `transfer_source_account_id` and `transfer_dest_account_id` added; ingest pipeline updated — transfer detection is user-initiated on save, not automatic; `findTransferPeer` function replaces batch detection; CSV advisory-only background pass documented|
|1.6|2026-04-05|`type` field expanded to full 6-value enum: 'expense' \| 'income' \| 'transfer' \| 'refund' \| 'investment' \| 'iou'; schema comments document TYPE/TAG orthogonality and financial treatment per type; split constraint updated — splits blocked on transfer/investment/iou, allowed on expense/income/refund; auto-assignment rules documented in schema comments|
|1.7|2026-04-05|`type` enum reduced to 3 values: 'expense' \| 'income' \| 'transfer'; REFUND/INVESTMENT/IOU removed and explicitly deferred; CHECK constraint simplified; transfer split-block constraint simplified|
|1.8|2026-04-05|Split-blocking CHECK constraint on TRANSFER removed — splits supported on all transaction types; only constraint remaining is single-tag/split mutual exclusivity|
|1.9|2026-04-05|Postgres schema additions: `waitlist` table (early access + invitation token lifecycle), `notification_preferences` table (per-user per-channel per-event preferences with future channel support); subscriptions table expanded with onboarding_completed_at field reference|

---

## 0. Governing Philosophy

Goaldy is a **benevolent dictatorship**. There is one primary user, one opinionated design, and zero committees. Every architecture decision optimizes for: the builder's own daily use first, code clarity and maintainability second, and future monetization headroom third. In that order. When these conflict, that order is the tiebreaker.

The app is **closed source, vibe-coded, and aggressively opinionated**. There is no design-by-committee, no abstract generalization for hypothetical future users, and no premature abstraction. Every layer is chosen because it is the best tool for this specific problem, not because it is popular or safe.

> **Reference implementation:** Actual Budget (`actualbudget/actual`) — MIT licensed, TypeScript, local-first SQLite, well-architected. We study it, borrow liberally where sensible, and deviate deliberately where our model differs.

---

### The Architectural Thesis — Why This Design Exists

There are three existing models in the personal finance space. Goaldy is none of them — it is a fourth model that takes the best property of each without their core liability.

**Model 1 — Full Cloud SaaS** (Monarch, Buxfer, YNAB) The aggregator connects to your bank, writes transactions to their database, your browser reads from their API. Convenient automation. They own your financial history. You are a tenant.

**Model 2 — Local-First with Sync Server** (Actual Budget) Data lives in a local SQLite file. A sync server handles automated bank feeds and multi-device access. You own the data — but you either self-host the sync server (real technical burden) or pay for Actual Cloud (back to tenancy). The sync server is load-bearing infrastructure that must be run by someone.

**Model 3 — File Import Only** (early Actual, YNAB offline) Data is local, no server needed, but automation is manual. You export a CSV, you import it. Acceptable for discipline; unusable for daily convenience.

**Model 4 — Goaldy**

```
External pipeline       Goaldy server           User's own storage     Browser
(Moneyman, SimpleFIN)   (stateless function)    (Google Drive)
        │                       │                       │                 │
        ▼                       ▼                       ▼                 ▼
  fetch/receive tx  →  encrypt blob (AES-256) →  goaldy.db (SQLite) → OPFS
                        write to ingest_blobs    (Drive appDataFolder)  wa-sqlite
                        (48h TTL, opaque)          ↑                    DuckDB-WASM
                              │               downloaded on
                              └──────────────→ session open
                         claimed + deleted
                         on browser load
```

The server is a **stateless encryption relay**. It never stores readable financial data. It holds an encrypted blob for at most 48 hours. The financial system of record is the user's own Google Drive file. All reads and queries run in the browser against a local OPFS copy.

This architecture achieves something no existing PFM tool has managed simultaneously:

|Capability|Monarch / Buxfer|Actual Budget|Goaldy|
|---|---|---|---|
|Automated bank sync|✅ (they own infra)|✅ (sync server required)|✅ (blob queue, no server)|
|User owns financial data|❌|✅|✅|
|No sync server to maintain|✅ (it's theirs)|❌|✅|
|REST API for external push|❌|❌|✅|
|Works fully offline|Partial|✅|✅ (OPFS cache)|
|Zero readable financial data on dev servers|❌|✅|✅ (encrypted blobs, 48h max)|
|Minimal GDPR / regulatory surface|❌|✅|✅|

The REST ingest API is the property that most clearly separates Goaldy from Actual. Actual has no external push API — automated sync requires their sync server or self-hosting. Goaldy's `/api/ingest` endpoint makes it a first-class target for Moneyman, custom scripts, and any future integration the user builds, with no server for the user to run and no persistent financial data on Goaldy's infrastructure.

**The one-sentence pitch to a technical Moneyman user:**

> Goaldy gives you automated sync and a full REST API — with your data in your own Google Drive and nothing financial sitting in our database.

Every technical decision in this document flows from that statement.

---

## 1. Technology Stack

### The T3 Core

|Layer|Technology|Version Target|Rationale|
|---|---|---|---|
|Framework|Next.js App Router|15.x|Server components for auth + metadata shell; client components for all financial UI|
|API|tRPC|v11|End-to-end type safety without OpenAPI ceremony; monorepo-native|
|ORM|Prisma|5.x|Self-join tag tree support; clean migrations; strong TS types|
|Auth|NextAuth.js (Auth.js v5)|5.x|Google OAuth as primary (same account as Drive); email magic link fallback|
|App DB|Supabase (Postgres)|—|Non-financial metadata only — see Section 4|
|Styling|Tailwind CSS|3.x|RTL support via `rtl:` variants; design system tokens|
|Language|TypeScript|5.x strict|Strict mode on. No `any`. No exceptions.|

### In-Browser Financial Engine

|Layer|Technology|Rationale|
|---|---|---|
|Financial DB|wa-sqlite (WASM)|SQLite in browser via OPFS; sub-ms queries; no server round-trip|
|Storage target|Origin Private File System (OPFS)|Browser-native private storage; persists across sessions; invisible to user|
|Analytical queries|DuckDB-WASM|Heavy aggregations — net worth trends, cashflow projections, tag rollups|
|Drive sync|Google Drive API v3|`appDataFolder` scope; `goaldy.db` blob; upload on write, download on load|

### Infrastructure

|Layer|Technology|Rationale|
|---|---|---|
|Hosting|Vercel|Zero-config Next.js; edge functions for ingest API; preview deploys|
|Queue|Upstash (Redis + QStash)|Background AI categorization jobs; webhook ingest queue|
|Email|Resend|Magic link auth; sync failure notifications|
|AI|Anthropic API (Claude claude-sonnet-4-6)|Transaction categorization; intelligence feed generation|
|Repo|Private GitHub|Single monorepo; no public access|
|CI|GitHub Actions|Type check → lint → test → deploy on merge to main|

### Key Dependencies (Non-Negotiable)

```
wa-sqlite          — SQLite WASM runtime
@sqlite.org/sqlite-wasm — OPFS VFS layer
@duckdb/duckdb-wasm — analytical query engine  
@anthropic-ai/sdk  — Claude API client
googleapis         — Drive API v3
zod                — runtime validation everywhere (tRPC + ingest API)
date-fns           — date manipulation (no moment, no dayjs)
next-intl          — i18n + RTL (see Section 11)
```

---

## 2. Repository Structure

```
goaldy/
├── apps/
│   └── web/                    # Next.js application
│       ├── app/                # App Router pages
│       │   ├── (auth)/         # Login, OAuth callback
│       │   ├── (app)/          # Protected app shell
│       │   │   ├── dashboard/
│       │   │   ├── transactions/
│       │   │   ├── reports/
│       │   │   ├── goals/
│       │   │   ├── budgets/
│       │   │   └── settings/
│       │   └── api/
│       │       ├── trpc/       # tRPC handler
│       │       ├── ingest/     # REST ingest endpoint (Moneyman, webhooks)
│       │       └── auth/       # NextAuth handler
│       ├── components/
│       │   ├── ui/             # Primitive components (shadcn/ui base)
│       │   ├── layout/         # Sidebar, nav, shell
│       │   └── features/       # Dashboard, transactions, reports etc
│       └── lib/
│           ├── db/             # wa-sqlite + OPFS layer
│           ├── drive/          # Google Drive sync
│           ├── engine/         # Rules engine, dedup, categorization
│           ├── importers/      # CSV, OFX, Buxfer, Actual parsers
│           ├── ai/             # Claude categorization + intelligence feed
│           └── analytics/      # Client-side event tracking
├── packages/
│   ├── db/                     # Prisma schema + client (app DB)
│   ├── trpc/                   # tRPC router definitions
│   ├── schema/                 # Zod schemas shared across layers
│   └── types/                  # Shared TypeScript types
├── tooling/
│   ├── eslint/
│   └── typescript/
└── turbo.json                  # Turborepo config
```

---

## 3. Financial Data Architecture — The Core Thesis

> **The app does not own the user's financial data. It hydrates from it.**

### The Model

```
Google Drive (appDataFolder scope)
  └── goaldy.db                 ← the user's entire financial life
       └── downloaded to OPFS on app load
            └── queried in-browser via wa-sqlite
                 └── uploaded back to Drive on every write
```

The app is a **stateless shell** relative to financial data. At launch it downloads `goaldy.db` from the user's Drive, mounts it in OPFS, and every subsequent read is local (sub-ms). Every write goes to OPFS first (instant), then queues a Drive upload (background). The UI never waits for Drive.

### Drive Scope

The app requests only `https://www.googleapis.com/auth/drive.appdata` — the `appDataFolder` scope. This creates a hidden folder visible only to the Goaldy app. Users never see it in their Drive UI. The permission dialog reads as narrow and non-threatening, which materially improves signup conversion.

### Offline Behavior

If Drive is unreachable at launch:

1. Check OPFS for an existing `goaldy.db` — if found, load it and display a "working offline" banner
2. All reads and writes continue against the local OPFS copy
3. When Drive reconnects, sync the diff (last-write-wins — acceptable for single-user personal finance where simultaneous writes are impossible by design)
4. If OPFS is empty and Drive is unreachable, display a graceful error — the app cannot function without data on first load

### Single-Session Constraint

**Goaldy does not support concurrent sessions.** This is an explicit, permanent design decision — not a limitation to be engineered away later. One user, one device active at a time. The last-write-wins sync model is only safe under this constraint. No locking primitives, no conflict resolution, no CRDT complexity. The user acknowledges this at onboarding.

Multiple device access is supported sequentially — open on phone, Drive syncs, open on laptop, downloads the latest file. This is the model. It works for personal finance. It does not work for shared household use (multi-user is a Phase 4 problem explicitly deferred).

---

## 4. Application Database (Postgres via Supabase)

This database holds **zero financial data**. It exists to manage the application layer.

### Schema

```sql
-- Core identity
users
  id            uuid PK
  email         text UNIQUE
  name          text
  locale        text DEFAULT 'en'        -- 'en' | 'he'
  currency      text DEFAULT 'ILS'
  created_at    timestamptz
  last_seen_at  timestamptz

-- Drive integration
drive_connections
  user_id       uuid FK → users
  access_token  text  (encrypted at rest)
  refresh_token text  (encrypted at rest)
  token_expiry  timestamptz
  drive_file_id text  -- the goaldy.db file ID in Drive

-- Ingest API
api_keys
  id            uuid PK
  user_id       uuid FK → users
  key_hash      text  -- SHA-256 of the key; raw key shown once on creation
  name          text  -- user-defined label ("Moneyman webhook")
  last_used_at  timestamptz
  created_at    timestamptz
  revoked_at    timestamptz

-- Third-party sync tokens (SimpleFIN etc.)
sync_connections
  id            uuid PK
  user_id       uuid FK → users
  provider      text  -- 'simplefin' | 'future_provider'
  access_url    text  (encrypted at rest)
  last_synced_at timestamptz
  last_error    text

-- Monetization hooks (ready from day 1, unused until Phase 4)
subscriptions
  user_id             uuid FK → users
  tier                text DEFAULT 'free'   -- 'free' | 'pro'
  status              text DEFAULT 'active' -- 'active' | 'cancelled' | 'past_due'
  stripe_customer_id  text
  stripe_sub_id       text
  current_period_end  timestamptz

-- Analytics events (privacy-safe, no financial data)
analytics_events
  id            uuid PK
  user_id       uuid FK → users
  event         text  -- 'import_completed', 'ai_categorization_run', etc.
  properties    jsonb -- { count: 47, source: 'moneyman' } — never amounts
  created_at    timestamptz

-- Early access waitlist
waitlist
  id            uuid PK
  email         text UNIQUE NOT NULL
  answer_usage  text        -- "how do you manage finances today?"
  answer_market text        -- 'il' | 'us' | 'both'
  invited_at    timestamptz -- NULL until invitation sent
  invite_token  text UNIQUE -- one-time signup token; NULL after use
  invite_expires_at timestamptz
  invite_used_at    timestamptz
  created_at    timestamptz NOT NULL

-- User notification preferences
notification_preferences
  id            uuid PK
  user_id       uuid FK → users
  channel       text NOT NULL  -- 'email' | 'whatsapp' | 'telegram'
  event_type    text NOT NULL  -- 'sync.failed' | 'drive.disconnected' |
                               -- 'session.new_device' | 'weekly.digest'
  enabled       boolean NOT NULL DEFAULT true
  UNIQUE (user_id, channel, event_type)

-- Async ingest queue (encrypted blobs only — see Section 6)
ingest_blobs
  id            uuid PK
  user_id       uuid FK → users
  source        text NOT NULL   -- 'simplefin' | 'webhook' | 'api'
  iv            bytea NOT NULL  -- AES-256-GCM initialisation vector (12 bytes)
  ciphertext    bytea NOT NULL  -- AES-256-GCM encrypted transaction payload
  created_at    timestamptz NOT NULL
  expires_at    timestamptz NOT NULL  -- TTL: created_at + 48h (hard delete by cron)
  ingested_at   timestamptz          -- set on successful browser ingest; row deleted immediately after
```

**What is never stored here in plaintext:** transaction amounts, account balances, merchant names, tag assignments, or any readable financial data. The `ingest_blobs` table holds only opaque encrypted ciphertext — Goaldy's Postgres cannot be queried for financial content. All other financial data lives exclusively in `goaldy.db` in Drive.

---

## 5. Financial Database Schema (SQLite in OPFS)

This is the schema for `goaldy.db`. Modeled after Actual Budget's `loot-core` schema conventions with deliberate deviations for Goaldy's tag-tree model and snapshot accounts.

```sql
-- Accounts
CREATE TABLE accounts (
  id              TEXT PRIMARY KEY,         -- uuid
  name            TEXT NOT NULL,
  type            TEXT NOT NULL,            -- 'checking' | 'savings' | 'credit' |
                                            -- 'investment' | 'loan' | 'manual'
  liquidity_class TEXT NOT NULL,            -- 'liquid' | 'semi_liquid' | 'illiquid'
                                            -- | 'locked' | 'liability'
  currency        TEXT NOT NULL DEFAULT 'ILS',
  is_manual       INTEGER NOT NULL DEFAULT 0,  -- 1 = snapshot-based, no tx stream
  institution     TEXT,                     -- 'hapoalim' | 'leumi' | 'chase' | etc
  is_closed       INTEGER NOT NULL DEFAULT 0,
  sort_order      INTEGER NOT NULL DEFAULT 0,
  created_at      TEXT NOT NULL,
  updated_at      TEXT NOT NULL
);

-- Account value snapshots (manual accounts only)
CREATE TABLE account_snapshots (
  id          TEXT PRIMARY KEY,
  account_id  TEXT NOT NULL REFERENCES accounts(id),
  value       REAL NOT NULL,               -- signed; liabilities are negative
  note        TEXT,
  snapshot_at TEXT NOT NULL               -- ISO date string
);

-- Transactions
CREATE TABLE transactions (
  id              TEXT PRIMARY KEY,         -- uuid
  account_id      TEXT NOT NULL REFERENCES accounts(id),
  amount          REAL NOT NULL,            -- signed; expenses/transfers-out negative,
                                            -- income/refunds/transfers-in positive
  currency        TEXT NOT NULL DEFAULT 'ILS',
  date            TEXT NOT NULL,            -- ISO date string YYYY-MM-DD
  description     TEXT NOT NULL,            -- raw from source
  merchant        TEXT,                     -- normalized merchant name
  notes           TEXT,
  labels          TEXT,                     -- JSON array of strings e.g. '["Tax/Deductible","Reimbursable"]'
                                            -- no financial impact; filterable only
  type            TEXT NOT NULL DEFAULT 'expense',
                  -- 'expense'   → counts in expense reports; tag drives budget line
                  -- 'income'    → counts in income reports; tag drives budget line
                  -- 'transfer'  → excluded from all expense/income reports; tag is identification only
                  --
                  -- REFUND, INVESTMENT, IOU explicitly deferred — not in scope for current version.
                  --
                  -- TYPE and TAG are fully orthogonal. Every type can carry a tag.
                  -- TYPE controls financial treatment. TAG controls categorization.
                  -- Auto-assigned on import: amount < 0 → 'expense', amount > 0 → 'income',
                  -- provider transfer metadata → 'transfer'. Always user-overridable.
  tag_id          TEXT REFERENCES tags(id), -- NULL if untagged or split
                                            -- meaningful on all transaction types
                                            -- mutually exclusive with transaction_splits rows
  -- Transfer fields (populated only when type = 'transfer')
  transfer_source_account_id TEXT REFERENCES accounts(id),
  transfer_dest_account_id   TEXT REFERENCES accounts(id),
  import_hash     TEXT UNIQUE,             -- SHA-256(date|amount|description|account_id)
  status          TEXT NOT NULL DEFAULT 'cleared', -- 'cleared' | 'pending'
  source          TEXT NOT NULL,           -- 'moneyman' | 'simplefin' | 'csv' |
                                           -- 'ofx' | 'api' | 'manual'
  category_source TEXT NOT NULL DEFAULT 'untagged', -- 'rule' | 'ai' | 'user' | 'untagged'
  ai_confidence   REAL,                    -- 0.0-1.0; set by Claude on AI categorization
  created_at      TEXT NOT NULL,
  updated_at      TEXT NOT NULL,
  -- Constraints
  CHECK (type IN ('expense', 'income', 'transfer')),
  CHECK (
    -- single-tag and split are mutually exclusive
    tag_id IS NULL OR NOT EXISTS (
      SELECT 1 FROM transaction_splits WHERE transaction_id = id
    )
  )
);

-- Transaction splits (used when tag_id IS NULL and amount must be fully allocated)
-- Replaces the old transaction_tags join table entirely
-- Each split is an independent budget unit with its own tag and amount
CREATE TABLE transaction_splits (
  id              TEXT PRIMARY KEY,
  transaction_id  TEXT NOT NULL REFERENCES transactions(id),
  amount          REAL NOT NULL,            -- signed; must sum to parent transaction amount
  description     TEXT,                    -- optional label for this split line
  tag_id          TEXT NOT NULL REFERENCES tags(id),
  sort_order      INTEGER NOT NULL DEFAULT 0
  -- CHECK: sum of splits = parent amount is enforced at application layer
  -- SQLite cannot reference other rows in a CHECK constraint
);

-- Tag tree (arbitrary depth via adjacency list)
-- Tag is the source of truth for classification AND budget configuration
CREATE TABLE tags (
  id               TEXT PRIMARY KEY,
  parent_id        TEXT REFERENCES tags(id),  -- NULL = root tag
  name             TEXT NOT NULL,
  color            TEXT,                       -- hex, for UI display
  icon             TEXT,                       -- emoji or icon key
  sort_order       INTEGER NOT NULL DEFAULT 0,
  -- Budget configuration (all nullable — absence means no budget target)
  budget_amount    REAL,                       -- target per repeat period
  budget_type      TEXT,                       -- 'expense' | 'income'
                                              -- inferred from cash flow if null
  budget_period    TEXT,                       -- 'weekly' | 'biweekly' | 'monthly' |
                                              -- 'quarterly' | 'annually' | 'custom'
  budget_custom_days INTEGER,                  -- used when budget_period = 'custom'
  budget_start_date TEXT,                      -- ISO date
  budget_rollover  INTEGER NOT NULL DEFAULT 0, -- 1 = unused carries to next period
  budget_end_date  TEXT,                       -- optional; NULL = ongoing
  budget_account_id TEXT REFERENCES accounts(id), -- optional account scope
  created_at       TEXT NOT NULL
);

-- note: is_income removed from tags. Income/expense classification for the
-- Budgets screen is determined by budget_type on the tag's budget configuration,
-- or inferred from dominant transaction direction when budget_type is null.

-- Categorization rules
CREATE TABLE rules (
  id            TEXT PRIMARY KEY,
  target_tag_id TEXT NOT NULL REFERENCES tags(id),
  priority      INTEGER NOT NULL DEFAULT 0,   -- lower number = higher priority
  match_mode    TEXT NOT NULL DEFAULT 'all',  -- 'all' | 'any'
  match_count   INTEGER NOT NULL DEFAULT 0,   -- incremented on every match
  last_matched_at TEXT,                        -- ISO datetime; NULL = never matched
  created_at    TEXT NOT NULL
);

-- Rule conditions (structured, queryable)
CREATE TABLE rule_conditions (
  id        TEXT PRIMARY KEY,
  rule_id   TEXT NOT NULL REFERENCES rules(id),
  field     TEXT NOT NULL,    -- 'description' | 'merchant' | 'amount' | 'account_id'
  operator  TEXT NOT NULL,    -- 'contains' | 'equals' | 'starts_with' |
                              -- 'greater_than' | 'less_than' | 'matches_regex'
  value     TEXT NOT NULL
);

-- Goals
CREATE TABLE goals (
  id            TEXT PRIMARY KEY,
  name          TEXT NOT NULL,
  type          TEXT NOT NULL,          -- 'savings' | 'debt_payoff' | 'purchase' | 'retirement'
  target_amount REAL,
  target_date   TEXT,                   -- ISO date
  linked_account_id TEXT REFERENCES accounts(id),
  linked_tag_id TEXT REFERENCES tags(id),
  notes         TEXT,
  created_at    TEXT NOT NULL,
  updated_at    TEXT NOT NULL
);

-- Intelligence feed cards (persisted state)
CREATE TABLE insight_cards (
  id          TEXT PRIMARY KEY,
  type        TEXT NOT NULL,   -- 'anomaly' | 'cashflow' | 'goal' | 'israeli' | 'offer'
  title       TEXT NOT NULL,
  body        TEXT NOT NULL,
  priority    INTEGER NOT NULL DEFAULT 0,
  is_read     INTEGER NOT NULL DEFAULT 0,
  is_dismissed INTEGER NOT NULL DEFAULT 0,
  valid_until TEXT,
  created_at  TEXT NOT NULL
);

-- Transfer detection candidates (transient — resolved by user action, then deleted)
CREATE TABLE transfer_review (
  id            TEXT PRIMARY KEY,
  tx_a_id       TEXT NOT NULL REFERENCES transactions(id),
  tx_b_id       TEXT NOT NULL REFERENCES transactions(id),
  confidence    TEXT NOT NULL,   -- 'high' | 'medium'
  created_at    TEXT NOT NULL
);

-- Schema version (for migrations)
CREATE TABLE meta (
  key   TEXT PRIMARY KEY,
  value TEXT NOT NULL
);
INSERT INTO meta VALUES ('schema_version', '1');

-- Indexes
CREATE INDEX idx_transactions_account ON transactions(account_id);
CREATE INDEX idx_transactions_date ON transactions(date);
CREATE INDEX idx_transactions_hash ON transactions(import_hash);
CREATE INDEX idx_transactions_tag ON transactions(tag_id);
CREATE INDEX idx_transaction_splits_tx ON transaction_splits(transaction_id);
CREATE INDEX idx_tags_parent ON tags(parent_id);
CREATE INDEX idx_rule_conditions_rule ON rule_conditions(rule_id);
CREATE INDEX idx_transfer_review_a ON transfer_review(tx_a_id);
CREATE INDEX idx_transfer_review_b ON transfer_review(tx_b_id);
```

---

## 6. Data Ingestion Architecture

Goaldy does not provision financial integrations. It exposes surfaces for the user's own pipelines to push data into. All financial data written through any ingest path lands directly in `goaldy.db` in OPFS — it never touches Goaldy's servers in transit.

### Ingest Pipeline (Common to All Sources)

```
Raw input (any format)
  → Parser (source-specific normalizer)
  → Zod schema validation
  → Dedup check (import_hash lookup against SQLite)
  → CSV transfer hint pass (best-effort, background, CSV imports only — see below)
  → Rules engine (structured condition matching — skips type='transfer')
  → Unmatched queue (normal transactions with no rule match)
  → AI categorization batch (Claude, background, Upstash queue)
  → Write to SQLite (wa-sqlite, OPFS)
  → Background Drive upload (non-blocking)
```

**Transfer handling — user-initiated, not automatic:**

Transfers are resolved by the user explicitly marking a transaction as TYPE = TRANSFER via the Edit Transaction panel. This sets `transfer_source_account_id` and `transfer_dest_account_id` on the transaction, then triggers peer detection on save:

```typescript
// packages/engine/transfer-detection.ts
async function findTransferPeer(
  tx: Transaction,
  destAccountId: string,
  db: SqliteDb
): Promise<Transaction | TransferCandidate[]> {

  const candidates = await db.all(`
    SELECT * FROM transactions
    WHERE account_id = ?
      AND ABS(amount) BETWEEN ? AND ?
      AND amount * ? < 0          -- opposite sign
      AND date BETWEEN ? AND ?
      AND type != 'transfer'      -- don't match already-resolved transfers
    ORDER BY ABS(julianday(date) - julianday(?))  -- closest date first
  `,
    destAccountId,
    Math.abs(tx.amount) * 0.995,   // 0.5% tolerance lower bound
    Math.abs(tx.amount) * 1.005,   // 0.5% tolerance upper bound
    tx.amount,
    addDays(tx.date, -2),
    addDays(tx.date, +2),
    tx.date
  );

  if (candidates.length === 1) return candidates[0];  // unambiguous match
  if (candidates.length > 1)  return candidates;      // user selects from list
  return null;                                         // unmatched — save as external transfer
}
```

On peer found: UI presents "Delete matching transaction on [account]? This cannot be undone." User confirms → peer is hard-deleted → transfer record `type = 'transfer'` remains in the ledger. Transfer transactions are excluded from all expense/income/budget calculations at the query layer (`WHERE type != 'transfer'`).

**CSV import best-effort hint (background, advisory only):**

For CSV imports where provider metadata is absent, a background pass runs after import using the same matching criteria. High-confidence matches (same day, exact amount, known payment description pattern in either Hebrew or English) are surfaced as suggestions in the Transfer Review banner. No automatic action. User handles each suggestion via the normal Edit Transaction → TYPE = TRANSFER flow.

### Async Ingest Queue — Encrypted Blob Pattern

Any ingest source that runs server-side (without a live browser session) uses this pattern. The server never stores readable financial data — only an encrypted opaque blob the server itself cannot query.

**Encryption scheme:** AES-256-GCM with a 12-byte random IV per blob. Key source: `INGEST_ENCRYPTION_KEY` environment secret (256-bit, Vercel environment variable, never in source). This is the MVP key model — the server holds the key but the data is short-lived and immediately deleted on ingest.

**Full lifecycle:**

```
Server-side job runs (SimpleFIN cron, webhook receipt):
  1. Fetch/receive raw transaction payload
  2. Normalise to ingest schema (Zod-validated)
  3. Generate random 12-byte IV
  4. Encrypt full JSON payload with AES-256-GCM
  5. INSERT into ingest_blobs: { user_id, source, iv, ciphertext, expires_at: now+48h }
  6. Job complete. Server holds only ciphertext. Cannot query amounts or descriptions.

Next browser session open:
  1. tRPC call: checkPendingBlobs() → returns count only (not contents)
  2. If count > 0: tRPC call: claimIngestBlob(id)
     → server decrypts in-memory, returns plaintext payload over HTTPS (TLS in transit)
     → row marked ingested_at = now
  3. Browser runs standard ingest pipeline on received payload:
     dedup → rules engine → AI queue → OPFS write → Drive upload
  4. tRPC call: confirmBlobIngested(id)
     → server hard-deletes the row immediately
  5. Blob lifetime on server in decrypted form: 0ms (never persisted decrypted)
  6. Blob lifetime on server total: minutes to 48h max

Background cleanup:
  Upstash cron (hourly): DELETE FROM ingest_blobs WHERE expires_at < now
  Belt-and-suspenders TTL — rows are normally deleted in step 4 but this catches failures
```

**No cap on payload size** — a single blob can hold an entire month of SimpleFIN transactions as one encrypted JSON array. The 500-transaction-per-call limit in the REST API does not apply to blobs (they're assembled server-side under our control).

---

### Ingest Surface A — Webhook / REST API

A POST endpoint at `/api/ingest` accepts transaction payloads from external systems (Moneyman, custom scripts, anything the user builds).

**Auth:** API key in `Authorization: Bearer <key>` header. Keys are created in Settings → Integrations, displayed once, stored as SHA-256 hash in Postgres. Per-user, named, revocable.

**Payload (Zod-validated):**

```typescript
const IngestPayload = z.object({
  account_id: z.string().uuid(),
  transactions: z.array(z.object({
    date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
    amount: z.number(),                       // signed
    currency: z.string().length(3),
    description: z.string(),
    merchant: z.string().optional(),
    status: z.enum(['cleared', 'pending']).default('cleared'),
  })).min(1).max(500),
});
```

**Two delivery paths depending on whether a browser session is active:**

- **Session active:** payload routes directly to the tRPC ingest handler → browser pipeline → OPFS. Never hits Postgres.
- **No active session:** payload is encrypted and written to `ingest_blobs`. Picked up on next browser open.

The webhook endpoint itself has no way to know if a session is active — it always writes to `ingest_blobs`. The browser checks for pending blobs on every load and drains the queue before rendering the UI. This is simpler, more reliable, and eliminates any race condition between the webhook and an open session.

**Response:** `{ queued: number, errors: string[] }` — always synchronous acknowledgement to the caller; actual OPFS write is always async.

This is the Moneyman integration target. One config line in Moneyman points at `https://app.goaldy.com/api/ingest` with the user's API key. Done.

### Ingest Surface B — SimpleFIN (US Banks)

The user pastes their SimpleFIN access URL into Settings → Integrations. Goaldy:

1. Stores the encrypted access URL in `sync_connections` in Postgres
2. Upstash QStash triggers a daily sync job per user who has a SimpleFIN connection
3. The job calls SimpleFIN's `/accounts` endpoint, normalises to the ingest schema
4. Encrypts the full payload and writes to `ingest_blobs`
5. On next browser session: blob is claimed, decrypted in transit, ingested to OPFS, row deleted

SimpleFIN pulls up to 90 days of history. On first connection, the full history blob may be large — this is fine, the blob pattern handles arbitrary payload size. Subsequent daily syncs are small (1-10 transactions typically).

### Ingest Surface C — File Import (Browser-Side)

All file parsing happens **in the browser**. Files are never uploaded to Goaldy's servers.

|Format|Parser|Notes|
|---|---|---|
|CSV (generic)|Client-side column auto-detection|Learns mapping from first import|
|Hapoalim CSV|Named parser|Specific column schema|
|Leumi CSV|Named parser||
|Max/Leumi Card CSV|Named parser||
|Discount CSV|Named parser||
|Isracard CSV|Named parser||
|Mizrahi CSV|Named parser||
|Visa Cal CSV|Named parser||
|OFX / QIF|Standard parser|Works for US banks, EU banks|
|Buxfer export|Named parser|Migration path; handles tag mapping|
|Actual Budget `.db`|SQLite reader|Direct schema mapping|
|YNAB CSV|Named parser||

### Ingest Surface D — Manual Entry

Transactions entered directly in the UI. No pipeline complexity — writes directly to SQLite.

---

## 7. Categorization Engine

### Rules Engine (Runs First, Free, No API Call)

Rules are evaluated in `priority` order (ascending). Each rule has one or more conditions combined by `match_mode` ('all' = AND, 'any' = OR).

```typescript
// Pseudocode — actual impl in packages/engine/rules.ts
function applyRules(tx: Transaction, rules: Rule[]): TagAssignment | null {
  for (const rule of rules.sortedByPriority) {
    const matched = rule.matchMode === 'all'
      ? rule.conditions.every(c => evaluateCondition(c, tx))
      : rule.conditions.some(c => evaluateCondition(c, tx));
    
    if (matched) {
      incrementMatchCount(rule.id);  // for staleness tracking
      return { tagId: rule.targetTagId, source: 'rule' };
    }
  }
  return null;  // → goes to AI queue
}
```

**Self-cleansing signals** — surfaced in UI under Tags → Rules:

- Rules with `match_count = 0` and `created_at > 30 days ago` → "Never matched — delete?"
- Rules with `last_matched_at < 90 days ago` → "Stale — still relevant?"
- Rules with `match_count = 1` ever → "Only matched once — consolidate?"

### AI Categorization (Claude, Runs Second, Background)

Unmatched transactions are batched and sent to Claude claude-sonnet-4-6 via a few-shot prompt:

```
System: You are a financial transaction categorizer. 
The user's tag tree is: [serialized tag tree]
Here are examples of confirmed categorizations: [10 most recent user-confirmed tx]
Respond with ONLY a JSON array: [{ "id": "...", "tag_id": "...", "confidence": 0.0-1.0 }]

User: Categorize these transactions: [batch of unmatched tx — descriptions only, no amounts]
```

**Privacy constraint:** Amounts are not sent to Claude. Description and merchant only. Tag assignments do not require amount data.

The response is applied to SQLite with `category_source = 'ai'` and `ai_confidence` set. Transactions with `ai_confidence < 0.7` are surfaced in the triage queue for user review.

### Triage Health Metric

Displayed prominently in the UI: `"X% auto-categorized · Y transactions need review"`. Goal is to minimize manual review over time as rules accumulate from confirmed AI suggestions. A mature setup should sustain >95% auto-categorization.

---

## 8. Tag System Design

**There are no categories.** Only tags. This is non-negotiable and drives the entire data model.

- Tags form an **arbitrary-depth tree** via adjacency list (`parent_id` self-reference)
- A transaction can have **multiple tags** (e.g. "Groceries" + "Family" + "Recurring")
- Exactly **one tag is `is_primary = 1`** per transaction — used for display in lists and default grouping
- Tags can be marked `is_income = 1` to classify income streams
- The tag tree drives: simulation rollups, report grouping, cashflow forecasting, intelligence feed card generation

**No budget envelopes. No category groups. No forced methodology.** The tree is whatever the user makes it. The app is not in love with any budgeting system.

---

## 9. UI Architecture

### Layout

Monarch-style left sidebar navigation, wide content area, no persistent right panel. Clean, modern, professional. Not Buxfer's three-column density.

**Sidebar top section:** Dashboard · Transactions · Reports · Goals · Budgets **Sidebar collapsible "More":** Tags · Rules · Merchants **Sidebar bottom:** Account tree (collapsible groups by liquidity class; Net Worth total pinned at top; clicking an account navigates to filtered transaction view)

### Key Screens

|Screen|Primary purpose|
|---|---|
|Dashboard|Net worth total + liquidity decomposition pills; horizontal tab selector (Expense / Income / Cash Flow / Net Worth dashboards); intelligence feed cards|
|Transactions|Full ledger with triage banner; inline tag assignment; account context header when filtered; swipe-to-tag on mobile|
|Reports|Spending by tag (donut + ranked list); income vs expense bar; net worth over time; cashflow timeline|
|Goals|Target tracking; simulation scenarios; Monte Carlo projection|
|Budgets|Tag-based budget assignment; period comparison|
|Import|File upload + parser selection; column mapping UI; dedup preview|
|Settings → Integrations|API key management; SimpleFIN token; sync connection status|

### Component Library

shadcn/ui as the primitive base — unstyled, composable, accessible. Tailwind for everything above primitives. No component library lock-in beyond shadcn. The design system is owned in `apps/web/lib/design-tokens.ts`.

### Brand

|Token|Value|
|---|---|
|`color-gold`|`#F9A825`|
|`color-indigo`|`#283593`|
|`color-bg`|`#F4F7F6`|
|`color-surface`|`#FFFFFF`|
|Font|Plus Jakarta Sans (400, 500, 700)|

---

## 10. Internationalization (i18n) — First-Class from Day One

**This is non-negotiable infrastructure, not a feature.** Retrofitting RTL and i18n post-launch is one of the most expensive mistakes in web development.

### Implementation

**Library:** `next-intl` — App Router native, type-safe, supports RTL direction switching.

**Locale storage:** `users.locale` in Postgres. Loaded at session level. No URL locale prefix (this is an authenticated app, not a public site).

**Supported locales at MVP:** `en` (LTR), `he` (RTL)

**RTL architecture:**

```css
/* All layout uses logical properties — never left/right */
padding-inline-start: 1rem;  /* not padding-left */
margin-inline-end: 0.5rem;   /* not margin-right */

/* Tailwind RTL variants on directional elements */
<div class="ml-4 rtl:mr-4 rtl:ml-0">
```

**Currency formatting — always `Intl.NumberFormat`, never string concatenation:**

```typescript
// ✅ correct
new Intl.NumberFormat('he-IL', { style: 'currency', currency: 'ILS' }).format(1234.56)
// → ‏1,234.56 ₪  (RTL mark + symbol on right in Hebrew)

// ✅ correct
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(1234.56)
// → $1,234.56

// ❌ never
`₪${amount}`
```

**Hebrew bank transaction descriptions** come in Hebrew regardless of UI locale. The parser layer preserves them as-is. The rules engine matches against Hebrew text. Claude categorizes Hebrew descriptions correctly.

---

## 11. Israeli-Specific Financial Instruments

These are first-class features, not afterthoughts. No competitor implements them.

### קרן השתלמות (Keren Hishtalmut)

- Account type in the schema: `type = 'investment'`, `liquidity_class = 'locked'`
- Intelligence card tracks days to vesting (6-year lock), projects value at vesting date
- Post-vesting: card presents the three standard options (withdraw tax-free, leave to compound, roll to pension) with simple scenario calculations
- Vesting date is user-input at account creation

### Pension (פנסיה)

- `type = 'investment'`, `liquidity_class = 'locked'`
- Snapshot-based account (estimated value updated periodically)
- Intelligence card tracks projected value at retirement age (user-input target age)

### Mortgage (משכנתא)

- `type = 'loan'`, `liquidity_class = 'liability'`
- User enters current rate and balance
- Intelligence card compares current rate vs current market rate (fetched from Bank of Israel public API)
- If spread > configurable threshold, surfaces refinancing insight card (lead gen hook, Phase 4)

### Multi-Currency

The schema supports per-transaction currency. Net worth aggregation converts via ECB/Bank of Israel rates (fetched daily, cached in `goaldy.db` as a rates table). ILS is the base display currency; USD, EUR supported from day one.

---

## 12. Security Model

### What Goaldy Never Stores

- Bank credentials (never touched)
- Transaction amounts in transit (SimpleFIN sync stores descriptions only temporarily)
- Full financial history server-side (lives in user's Drive only)

### Auth

- Google OAuth via NextAuth — access token and refresh token stored encrypted in `drive_connections`
- Magic link email fallback for non-Google auth
- Sessions: JWT, 30-day expiry, refresh on activity
- All Postgres sensitive fields (tokens, access URLs) encrypted at rest via Supabase column encryption

### Ingest API Security

- API keys: 32-byte random, shown once, stored as SHA-256
- Rate limited per key: 10 requests/minute, 1,000 transactions/day
- Keys are per-user, named, revocable individually from Settings

### Drive Security

- `appDataFolder` scope only — narrow, non-threatening to users
- `goaldy.db` is readable only by the Goaldy app via OAuth
- Users can delete the file from Drive at any time — complete data deletion with no Goaldy involvement
- Drive-level encryption at rest provided by Google

---

## 13. Buxfer Migration (Day One Priority)

The builder is User #1. Goaldy is not production-ready until it replaces Buxfer for daily use.

### Migration Script

A one-time CLI tool (not in the UI — runs locally):

```
1. Input: Buxfer full export (CSV) + scraped rules JSON
2. Map Buxfer accounts → Goaldy accounts (with liquidity_class assignment prompt)
3. Map Buxfer tags → Goaldy tag tree (hierarchical structure preserved)
4. Map Buxfer transactions → Goaldy transactions (with import_hash generation)
5. Convert Buxfer rules (string patterns) → Goaldy rule_conditions (structured)
6. Output: populated goaldy.db ready to upload to Drive
```

This is Phase 0 work. Nothing else ships until this works.

---

## 14. Monetization Architecture Hooks (Phase 4 Ready, Unused Now)

The subscription and analytics tables exist in Postgres from day one. Feature flags are evaluated at the tRPC layer:

```typescript
// packages/trpc/middleware/subscription.ts
export const requirePro = t.middleware(async ({ ctx, next }) => {
  if (ctx.user.subscription.tier !== 'pro') {
    throw new TRPCError({ code: 'FORBIDDEN', message: 'Pro feature' });
  }
  return next();
});
```

Free tier limits enforced at tRPC layer — not at the UI layer. The UI can show the feature but the procedure throws if the user is not Pro.

Analytics events fire on key actions with no financial data:

```typescript
// Example: import completed
trackEvent(userId, 'import_completed', { 
  source: 'moneyman', 
  count: 47,
  auto_tagged_pct: 89 
  // never: amounts, descriptions, account names
});
```

Stripe integration wired at Phase 4. The schema is ready. The code is not written until there is something worth paying for.

---

## 15. Development Phases

### Phase 0 — Foundation (Before Anything Else)

- [ ] T3 monorepo scaffold with all tooling
- [ ] Google OAuth + NextAuth
- [ ] Supabase Postgres with full schema migrated
- [ ] wa-sqlite + OPFS layer working in browser
- [ ] Google Drive `appDataFolder` read/write working
- [ ] `goaldy.db` schema initialized on first launch
- [ ] Buxfer migration CLI — import full history and rules
- [ ] Builder stops using Buxfer

### Phase 1 — Core Ledger

- [ ] Account CRUD with liquidity class
- [ ] Transaction list view
- [ ] CSV file import with Israeli bank parsers
- [ ] Manual tag assignment in UI
- [ ] Tag tree management
- [ ] Net worth calculation from accounts + snapshots
- [ ] Basic dashboard (balances, recent transactions)

### Phase 2 — Intelligence Layer

- [ ] Rules engine with structured conditions
- [ ] Moneyman webhook ingest endpoint + API key management
- [ ] SimpleFIN integration (US banks)
- [ ] Claude AI categorization pipeline
- [ ] Triage health metric in UI
- [ ] Intelligence feed cards (anomaly, cashflow)
- [ ] Self-cleansing rules signals in UI

### Phase 3 — Israeli Features + Reports

- [ ] קרן השתלמות countdown card
- [ ] Mortgage rate comparison card (Bank of Israel API)
- [ ] Pension projection card
- [ ] Full reports screen (spending, NW trend, cashflow)
- [ ] Goal tracking and simulation
- [ ] DuckDB-WASM for heavy analytical queries
- [ ] Hebrew UI (RTL, `next-intl` wired)

### Phase 4 — Monetization (Only If Warranted)

- [ ] Legal review of lead gen card framing
- [ ] Stripe integration
- [ ] Pro tier feature flags active
- [ ] Lead gen cards in intelligence feed
- [ ] Partner referral links

---

## 16. What We Are Not Building

Explicit non-goals. These will never be added without a deliberate design decision documented here:

- **Bank credential management** — Goaldy never holds or proxies bank credentials
- **Plaid / Finanda / any broker integration** — user brings their own data pipeline
- **Multi-user / household shared ledger** — single user only; concurrent sessions explicitly unsupported
- **Envelope budgeting** — the app has no opinion on budgeting methodology
- **Transaction execution** — read-only financial intelligence; no payments, no transfers
- **Mobile app** — web only until Phase 4; responsive design is not a mobile app
- **Public API** — the ingest API is inbound only; no outbound data API for third parties
- **Social features** — no sharing, no comparison, no community features

---

_Document created: 2026-04-03. Status: Living document — update before each phase begins._