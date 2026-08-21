| Field            | Value            |
| ---------------- | ---------------- |
| **Created**      | 2026-04-03       |
| **Last Updated** | 2026-08-21 v2.0  |
| **Version**      | 2.0              |
| **Status**       | Draft            |

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
|**2.0**|**2026-08-21**|**Full rewrite to match the shipped self-hosted pivot.** The product described in v1.0–1.9 (Google OAuth + Drive-as-database, Postgres app DB, tRPC, NextAuth, in-browser wa-sqlite/OPFS, Vercel/Upstash/Resend hosting, AI/Claude categorization, Israeli-specific financial-instrument cards, Stripe monetization, an invite-only waitlist beta) was never built. What shipped instead: a single-user, self-hosted Next.js 15 app where one server process owns exactly one SQLite file via `better-sqlite3` — no cloud dependency, no sync server, no OAuth. This version replaces nearly every section end-to-end, and documents three real, shipped subsystems the old design never mentioned: the household financial-planning "Plan" simulator, live bilingual i18n/RTL, and the Docker/CI-CD/public-demo deployment pipeline. Sections describing things that were never built (Postgres app DB, blob-queue ingest, AI categorization, Israeli instrument cards, monetization hooks) are deleted rather than marked historical — see the repo's git history for the pre-pivot design if it's ever needed for reference.|

---

## 0. Governing Philosophy

Goaldy is a **benevolent dictatorship**. There is one primary user, one opinionated design, and zero committees. Every architecture decision optimizes for: the builder's own daily use first, code clarity and maintainability second. There is no monetization headroom to optimize for — see below.

The app is **closed source, vibe-coded, and aggressively opinionated**. There is no design-by-committee, no abstract generalization for hypothetical future users, and no premature abstraction. Every layer is chosen because it is the best tool for this specific problem, not because it is popular or safe.

### The Architectural Thesis — Why This Design Exists

The earlier design (v1.0–1.9) chased a fourth model between full cloud SaaS, local-first-with-sync-server, and file-import-only: a "stateless encryption relay" server with the real data living in the user's own Google Drive, downloaded into an in-browser SQLite (wa-sqlite/OPFS) on every session. That thesis was never implemented.

**What actually shipped is simpler and more conventional:** a single Node process (the Next.js server) owns exactly one SQLite file (`goaldy.db`) on its own local disk, via `better-sqlite3`. There is no client-side database, no sync protocol, no encrypted relay, no external identity provider. You run the server (via Docker), the server owns the file, the browser talks to the server's REST API. If you want your data on your own infrastructure, you run the container on your own infrastructure — that is the entire self-hosting story.

```
Browser (React SPA, App Router)
        │  fetch() → REST API
        ▼
Next.js server (route handlers)
        │  better-sqlite3 (synchronous, in-process)
        ▼
goaldy.db  (one file, one Docker volume)
```

This trades away multi-device sync (there is exactly one place the data lives: wherever you run the container) for radical simplicity: no OAuth, no cloud storage quota, no token refresh logic, no "what happens when Drive is unreachable" edge cases. A user who wants their data elsewhere copies the file — `docker compose cp goaldy:/data/goaldy.db ./backup.db` — same guarantee the old Drive-based design was chasing, achieved by not needing a sync layer to chase it with.

**The public read-only demo** (see §11) exists to let people evaluate this without installing anything.

---

## 1. Technology Stack

|Layer|Technology|Version|Rationale|
|---|---|---|---|
|Framework|Next.js App Router|15.3.x|Route handlers double as the REST API; existing React UI preserved as-is across the pivot|
|UI|React|19|—|
|Language|TypeScript|~5.4, strict|—|
|Database|`better-sqlite3`|^12.8|Synchronous, in-process, native module — no network round-trip, no separate DB server to run|
|Validation|Zod|^3.23|Runtime validation everywhere; `@zod-to-openapi` generates the OpenAPI 3.1 spec directly from the same schemas|
|Client data fetching|`@tanstack/react-query`|^5|—|
|Charts|ECharts 6|—|Tree-shaken via `echarts/core` + SVG renderer; `components/charts/register.ts` and `echart.tsx` are the sole entry points|
|Styling|Tailwind CSS|3.4|—|
|Auth primitives|`bcryptjs`, Node `crypto`|—|Password hashing; sha256 for API-token and session-id hashing|
|i18n|`next-intl`|^4.9|Real, shipped — see §9|
|Test runner|Vitest|—|`environment: 'node'`, `**/*.test.ts` only — no jsdom/Testing Library, so testable logic lives in `.ts` files, not `.tsx` components|

**Removed from the old design entirely:** tRPC, Prisma, NextAuth (Auth.js), Supabase/Postgres, wa-sqlite, DuckDB-WASM, the Google Drive API, Vercel, Upstash (Redis/QStash), Resend, the Anthropic API. None of these are dependencies of the shipped app.

---

## 2. Repository Structure

```
goaldy/
├── apps/
│   └── web/                    # Next.js application — the entire product
│       ├── app/
│       │   ├── (auth)/login/   # Login page
│       │   ├── (app)/          # Authenticated app shell
│       │   │   ├── dashboard/  # Cashflow + Net Worth tabs (the real "reports" surface)
│       │   │   ├── transactions/
│       │   │   ├── tags/
│       │   │   ├── rules/
│       │   │   ├── plan/       # Household financial-planning simulator (§8)
│       │   │   ├── duplicates/
│       │   │   ├── budgets/    # Stub — "Coming soon" (data model exists, UI doesn't)
│       │   │   ├── reports/    # Stub — "Coming soon" (dashboard is the real reporting UI)
│       │   │   └── settings/
│       │   └── api/            # REST route handlers = the entire backend
│       ├── components/
│       ├── lib/
│       │   ├── server/         # db.ts, auth.ts, http.ts, ingest.ts, openapi.ts, queries/*, rules/*
│       │   ├── domain/         # pure logic: plan-engine.ts, accounting.ts, tag-flow.ts
│       │   └── queries/        # client-side fetch wrappers consumed by react-query hooks
│       └── i18n/                # next-intl config
│       └── messages/            # en.json, he.json — 744 keys each
├── packages/
│   └── schema/                 # @goaldy/schema — Zod schemas + the single-source-of-truth SQL
│       └── sql/goaldy.sql
├── scripts/                     # one-off CLI tools (tsx + better-sqlite3): migrate-buxfer.ts, generate-erd.ts, bump-version.ts
├── Dockerfile
├── docker-compose.yml
└── .github/workflows/{ci,release}.yml
```

`pnpm` workspaces + `turbo`. There is no `packages/db` (Prisma), no `packages/trpc`, no separate app-DB package — the schema package covers both the SQL DDL and its Zod mirror.

---

## 3. Data Architecture

> **One process owns one file. Nothing else touches it.**

`getDb()` (`apps/web/lib/server/db.ts`) opens `goaldy.db` (path from `GOALDY_LOCAL_DB_PATH` in dev, `DATABASE_URL=/data/goaldy.db` in Docker), enables WAL mode and foreign keys, and on first boot executes `packages/schema/sql/goaldy.sql` in full — every `CREATE TABLE`/`CREATE INDEX` is `IF NOT EXISTS`, so running it against an already-migrated database is a safe no-op. There is no separate migration runner tracking incremental versions beyond a single `meta.schema_version` stamp.

Every read and write is synchronous (`better-sqlite3` is a synchronous driver) and happens in-process — no network hop, no connection pool, no ORM layer translating between a wire protocol and objects. Async wrapper functions (`queryAll`/`queryOne`/`execSQL`/`execTransaction` in `db.ts`) exist only to keep the call-site shape consistent with the rest of the Next.js app; the underlying work is synchronous.

Writes hit disk immediately — there is no write-behind cache, no "flush" step, no background sync job. A `docker compose cp` of the volume at any moment is a consistent, complete backup.

---

## 4. Financial Database Schema

`packages/schema/sql/goaldy.sql` is the single source of truth for every table, index, and constraint — Zod types mirroring it live in `packages/schema/*.ts` (`account.ts`, `transaction.ts`, `tag.ts`, `rule.ts`, `plan.ts`, `settings.ts`, `notification.ts`, `duplicates.ts`, `accounting.ts`). `packages/schema/erd/goaldy-erd.md` is generated from the SQL file and must never be hand-edited — a test regenerates it and fails on drift.

### Auth / server state

```sql
settings (key TEXT PRIMARY KEY, value TEXT, updated_at)
sessions (id TEXT PRIMARY KEY, user_agent, ip_address, last_activity, expires_at)
```

`settings` is a flat key/value store holding `password_hash` (bcrypt), `api_token_hash` (sha256), `demo_seeded`, `password_reset_marker`, `base_currency`, `display_currency`, `language`, and a handful of feature-flag-shaped booleans (`process_rules_on_ingest`, `apply_rules_on_create`). There is no `users` table — a `goaldy.db` file has exactly one operator by construction.

### Accounts

```sql
CREATE TABLE accounts (
  id, name,
  type TEXT CHECK (type IN ('checking','savings','credit','investment','loan','real_estate','retirement','manual')),
  currency TEXT DEFAULT 'ILS',
  balance_source TEXT DEFAULT 'transactions' CHECK (balance_source IN ('transactions','reported_value')),
  reported_value REAL,          -- used when balance_source = 'reported_value'
  institution TEXT,             -- free text, deliberately unconstrained
  account_number TEXT UNIQUE,   -- external id, for integration matching (e.g. Moneyman)
  is_closed INTEGER DEFAULT 0,  -- archive flag
  sort_order, created_at, updated_at
);
```

**`liquidity_class` is not a column.** It is derived at the application layer from `type` via `getLiquidityClass()` (`packages/schema/account.ts`'s `TYPE_LIQUIDITY` map): `checking`/`savings` → liquid, `credit`/`loan` → liability, `investment` → semi_liquid, `real_estate`/`manual` → illiquid, `retirement` → locked. This is a deliberate simplification from the old design's stored `liquidity_class` column — one fewer thing that can drift out of sync with `type`.

### Transactions

```sql
CREATE TABLE transactions (
  id, account_id, amount, currency, date, description, merchant, notes,
  labels TEXT,                  -- JSON array; filter-only, no financial effect
  type TEXT DEFAULT 'expense' CHECK (type IN ('expense','income','transfer','investment')),
  investment_type TEXT CHECK (investment_type IN ('buy','sell','dividend')),  -- only when type='investment'
  transfer_source_account_id, transfer_dest_account_id,  -- only when type='transfer'
  import_hash BLOB UNIQUE,      -- SHA-256(account_id|date|amount|description|external_id) — idempotent re-import dedup
  external_id TEXT,             -- provider operation id; NOT unique — one op can span multiple rows
  status TEXT DEFAULT 'cleared' CHECK (status IN ('cleared','pending')),
  source TEXT CHECK (source IN ('moneyman','simplefin','csv','ofx','api','manual','buxfer')),
  category_source TEXT DEFAULT 'untagged' CHECK (category_source IN ('rule','ai','user','untagged')),
  ai_confidence REAL,           -- reserved schema seam — see note below
  tag_mode TEXT DEFAULT 'categories' CHECK (tag_mode IN ('categories','split')),
  created_at, updated_at
);
```

There is **no `tag_id` column** — every transaction's tags live in `transaction_tags` regardless of whether it's a single-tag or split transaction; `tag_mode` only governs how those rows are interpreted. `type` and tag are orthogonal by design: type controls financial treatment (does this count as income/expense/neither), tag controls categorization, and every type can carry a tag. Auto-assignment on import: negative amount → expense, positive → income, provider transfer metadata → transfer — always user-overridable afterward.

`category_source='ai'` and `ai_confidence` are a **reserved-but-unused schema seam**. No code path in the app ever sets either value today — see §6 for what categorization actually does.

`source='simplefin'`/`'csv'`/`'ofx'` are similarly anticipated-but-not-yet-wired values; only `moneyman` (webhook) and `buxfer` (developer CLI) have real ingestion code today (§7).

### Tags, splits, budgets

```sql
CREATE TABLE transaction_tags (
  transaction_id REFERENCES transactions(id) ON DELETE CASCADE,
  tag_id REFERENCES tags(id),
  position INTEGER,    -- ordering within a transaction
  weight REAL DEFAULT 0,  -- split allocation fraction; ignored when tag_mode='categories'
  PRIMARY KEY (transaction_id, tag_id)
);

CREATE TABLE tags (
  id, parent_id REFERENCES tags(id),  -- NULL = root; arbitrary-depth adjacency-list tree
  name, color, icon,
  budget_amount REAL,
  budget_type TEXT CHECK (budget_type IN ('expense','income')),
  budget_period TEXT CHECK (budget_period IN ('weekly','biweekly','monthly','quarterly','annually','custom')),
  budget_custom_days, budget_start_date, budget_rollover, budget_end_date,
  budget_account_id REFERENCES accounts(id),   -- optional account scope
  created_at
);
```

A tag doubles as a budget-line definition — every budget field is nullable, and absence means no budget target on that tag. Sibling-scoped name uniqueness only (case-insensitive) — the same name is legal under two different parents. **There is no `is_income` column anywhere in this schema** — income/expense classification comes from `budget_type` where set, or is inferred from the dominant cash-flow direction of the tag's transactions otherwise. (An earlier draft of this document's companion PRD asserted both that `is_income` was removed *and* that it drove a dashboard — that PRD contradiction is fixed as of this rewrite.)

Splits are enforced in `apps/web/lib/server/queries/transaction-tags.ts`: `categories` mode always writes a single row (position 0, weight 1); `split` mode requires weights to sum to 1 within a small tolerance and rejects negative weights (`TagWeightError`).

### Rules engine

```sql
rules (id, signature UNIQUE, priority, is_enabled, last_applied_at, created_at)
rule_conditions (id, rule_id, field, operator, value)
rule_actions (id, rule_id, action_type, value, position)
```

`signature` is a UNIQUE hash of a rule's condition set, deduplicating identical rules outright rather than merely warning about them.

### Plan feature (household financial-planning simulator)

See §8 for the full feature description; schema:

```sql
plan_settings (id='default' singleton, monthly_savings, inflation_rate DEFAULT 0.03,
                return_rate_override, horizon_years DEFAULT 37, start_year,
                emergency_pillar_active, emergency_target_months DEFAULT 6, retirement_age)
plan_members (id, plan_id, name, role CHECK IN ('self','partner','child','parent','other'), birth_year)
plan_items (id, plan_id, tier CHECK IN ('GOAL','EVENT'), name, icon, is_system,
            trigger_type CHECK IN ('YEAR','AGE','RECURRING'), trigger_year, trigger_age,
            member_id, recurrence_years, recurrence_end_year, amount, duration_years,
            saved_amount, status, is_active, tag_id, sort_order)
plan_account_layers (id, plan_id, account_id UNIQUE, layer CHECK IN ('investment','emergency','retirement'),
                      target_return_rate, monthly_contribution, linked_at)
```

### Notifications

```sql
notifications (id, kind, type, severity, category, title, payload TEXT, read_at, created_at)
```

An append-only event/condition log — not the old design's pluggable multi-channel (email/WhatsApp/Telegram) notification-provider architecture. 90-day TTL, a hard 5000-row cap, and a 24-hour condition-check sweep. See §10.

### Meta

```sql
meta (key TEXT PRIMARY KEY, value TEXT)  -- schema_version, and last_write_at (touched by nearly every mutation)
```

---

## 5. Auth Model

No OAuth, no external identity provider, no `users` table. Two equal-trust authentication paths (`apps/web/lib/server/auth.ts`):

- **Browser**: an httpOnly `session` cookie, backed by a row in `sessions` (30-day expiry, random 32-byte hex id).
- **Machine clients**: `Authorization: Bearer <token>`, compared via sha256 against `settings.api_token_hash`. Used by the Moneyman webhook integration and any script hitting the REST API directly.

**Bootstrap** (`ensureReady()`, memoized once per process): on first boot, generates an API token (from the `API_TOKEN` env var, or a random one logged once) and seeds the password hash from `ADMIN_PASSWORD` if set. If `ADMIN_PASSWORD` is unset, the **first successful password submitted at login becomes the password** — a bootstrap-via-first-login fallback. `GOALDY_RESET_PASSWORD` forces a password reset and clears all sessions, applied exactly once per value (tracked by a hash marker, so leaving the env var set doesn't repeatedly wipe sessions).

`DEMO_MODE=true` (used only by the public demo, §11) additionally seeds a curated fixture on first boot, and refuses to run if the database already has accounts it didn't create.

`withAuth()` (`apps/web/lib/server/http.ts`) wraps every API route: runs `ensureReady()`, requires one of the two auth paths above, and — when `DEMO_MODE=true` — blocks every non-GET/HEAD request except `/api/session` (login/logout), returning a 403 on any attempted write.

OpenAPI 3.1 is generated straight from the Zod schemas (`lib/server/openapi.ts`), served at `/api/openapi.json`, with Swagger UI at `/api/docs`.

---

## 6. Categorization — Rules Only, No AI

Categorization is **100% rule-based** today. `apps/web/lib/server/rules/` (`compile.ts`, `match.ts`, `apply.ts`, `actions.ts`, `regexp.ts`, `store.ts`) evaluates structured conditions (`description`/`merchant`/`notes`/`labels`/`amount`/`account_id`/`type`/`currency`/`source`/`status`/`investment_type`/`date`, with operators like `contains`/`equals`/`greater_than`/`matches_regex`) in priority order and applies ordered actions (`set_tag`, `add_label`, `set_type`, `set_transfer_source`, etc.). Rules run on ingest (gated by the `process_rules_on_ingest` setting) and optionally on manual transaction creation (`apply_rules_on_create`).

**There is no AI/LLM categorization anywhere in this codebase.** The schema's `category_source='ai'` value and `ai_confidence` column (§4) are a reserved seam for a feature that was designed but never built — a grep for "anthropic" across the entire app returns zero hits. Any future AI categorization work is a genuinely new feature, not a re-enablement of something dormant.

---

## 7. Data Ingestion

There is no encrypted-blob relay, no 48-hour TTL queue, no SimpleFIN integration. Real ingestion paths:

- **`ingestTransactions()`** (`apps/web/lib/server/ingest.ts`) — the single idempotent batch-insert primitive everything else calls. Dedups via the `import_hash` UNIQUE constraint (`ON CONFLICT DO NOTHING`, detected by `changes === 0`), runs the whole batch in one SQLite transaction, emits `ingest.completed`/`ingest.failed` notifications, and optionally fires the rules engine on the newly-created rows.
- **`POST /api/integrations/moneyman`** — adapts Moneyman's scraper output format (resolving each row's account field to a local account via `account_number`) into the shape `ingestTransactions()` expects. This is the one real webhook integration.
- **`scripts/migrate-buxfer.ts`** — a **developer-only CLI**, not reachable from the running app. Parses a Buxfer CSV/TSV export directly into a `goaldy.db` file via `better-sqlite3` (`pnpm buxfer:seed`). This is the only bulk-import path today; a self-hoster needs shell access to their own container to use it.
- Manual entry, directly through the UI.

`docs/features/2026-08-19-in-app-data-importer` documents a planned, generic in-app upload-and-map importer (multi-format, reusing the same `createAccount`/`createTag`/`ingestTransactions` primitives) intended to eventually replace the CLI script as the recommended path for real self-hosters — not yet built.

---

## 8. The Plan Feature — Household Financial-Planning Simulator

Undocumented in every prior version of this document. A long-horizon net-worth/solvency simulator, distinct from a household shared ledger — one `goaldy.db` still has exactly one operator; `plan_members` are labels used to trigger age-based projections, not separate logins.

- **Settings** (`plan_settings`, a singleton): a monthly savings target, an inflation-rate assumption (3% default), an optional blanket return-rate override, a projection horizon (37 years by default), a start year, and an optional "emergency fund" pillar with a target number of months of expenses.
- **Members** (`plan_members`): household members (self/partner/child/parent/other) with a birth year, used to compute displayed ages per projection year and to trigger AGE-type plan items (e.g. a child's tuition at a specific age).
- **Items** (`plan_items`): a unified GOAL/EVENT model. A **GOAL** is a tracked savings target — amount minus `saved_amount`, no inflation applied, typically short-horizon. An **EVENT** is a future one-off or recurring expense, projected forward with compound inflation from today. Three trigger shapes: `YEAR` (a fixed year, optionally recurring across `duration_years`), `AGE` (a specific member's age), `RECURRING` (an interval with an optional end year). Creating a plan item auto-creates a `Goals`/`Events` root tag plus a per-item leaf tag, so transactions tagged against it roll up into the plan automatically. System-created items (`is_system`) can't be toggled off.
- **Account layers** (`plan_account_layers`): assigns each account to one of three pillars — `investment` (carries a `target_return_rate` used in a weighted blended return), `emergency` (the liquidity buffer measured against the emergency target), or `retirement` (carries a `monthly_contribution`). One pillar per account.
- **Projection engine** (`apps/web/lib/domain/plan-engine.ts`, pure — no DB access): simulates year by year, `balance = balance × (1 + weightedReturn) + annualSavings − thatYear'sEventExpenses`, flagging `isDanger` (balance goes negative) and `isStress` (balance falls under 18 months of savings), and surfacing the first danger year, the lowest projected balance, and the required monthly savings to avoid a shortfall.
- **Emergency-fund summary**: compares the emergency-layer balance against trailing-12-month average expenses × the target month count, with a `CRITICAL`/`WARNING`/`OK` status.
- Multi-currency aware — account-layer balances are converted to `base_currency` (§9) before aggregation.

**Small Israeli-culture residue, distinct from anything in the old design's Israeli-instrument sections:** `packages/schema/plan-item-templates.ts` is a hardcoded, non-DB-stored array of ten life-event templates for the "add item from template" flow (Bar/Bat Mitzvah, university tuition, wedding contribution, etc.), each carrying both an English and Hebrew (`name_he`) name and an ILS amount hint. This is seed *content* for a generic feature, not an Israeli-specific *feature* the way the old design's קרן השתלמות/pension/mortgage-rate cards would have been — those account types and intelligence cards were never built.

---

## 9. Internationalization and Multi-Currency

Both real and shipped — differently from every prior design.

**i18n/RTL**: `next-intl`, configured via `apps/web/next.config.ts` and `apps/web/i18n/request.ts`. Full message catalogs at `apps/web/messages/en.json` and `he.json` (744 keys each). The root layout reads a `goaldy-locale` cookie server-side and sets `<html lang dir>` — `dir="rtl"` when the locale is Hebrew — real RTL at the document level, not just component-level flourishes. The `language` setting (`en`/`he`, nullable) is user-editable in Settings and writes the cookie directly; no server round-trip needed to take effect on next render.

**Multi-currency**: `apps/web/lib/server/fx.ts` fetches live rates from the **Frankfurter** API (`api.frankfurter.dev`), cached in-process for one hour per base currency with an 8-second timeout and a graceful fallback to a stale cache on upstream failure — no dependency on the old design's ECB/Bank-of-Israel-specific rate sources. `settings.base_currency` (default ILS) and an optional `settings.display_currency` override drive conversion; every account and every transaction carries its own currency independently. `convertToBase()` is used pervasively (net worth, dashboards, the Plan feature's account-layer aggregation). One deliberate guardrail: per-account balance history (`getBalanceHistory()`) does **not** convert currency — it assumes single-currency-per-account and throws loudly rather than silently mis-summing if that assumption is ever violated.

---

## 10. Notifications

`notifications` (§4) is an append-only event/condition log, not the old design's pluggable multi-provider (email/WhatsApp/Telegram) architecture with Postgres-backed per-user preferences. Rows are written on events like `ingest.completed`/`ingest.failed`/`api.error`, plus periodic condition checks run every 24 hours; the log is capped at 5000 rows with a 90-day TTL.

---

## 11. Deployment and CI/CD

Undocumented in every prior version of this document — built entirely after the pivot.

- **`Dockerfile`**: multi-stage build (deps → builder → runner), Node 22 Alpine, Next.js `output: 'standalone'`, `better-sqlite3` compiled against musl (python3/make/g++ in the build stage), one image, one `/data` volume, `DATABASE_URL=/data/goaldy.db` baked in. `APP_VERSION`/`GIT_SHA`/`BUILD_DATE` build args are surfaced in the running app (sidebar + Settings → About) and via `GET /api/version`.
- **`docker-compose.yml`**: a single `goaldy` service, one named volume, `ADMIN_PASSWORD`/`API_TOKEN`/`SESSION_SECRET` (reserved, currently unused) env vars. `docker compose up -d` is the entire deploy story.
- **`.github/workflows/ci.yml`**: on every PR into `main` — parallel typecheck/lint/test/build jobs, `actionlint` validating the workflow files themselves, and a required `semver:major|minor|patch` PR label check.
- **`.github/workflows/release.yml`**: a bot-maintained `release/next` PR accumulates the highest semver bump across merged, labeled PRs since the last tag; merging it tags `vX.Y.Z`, builds and pushes `ghcr.io/<owner>/goaldy:vX.Y.Z`/`:latest`/`:sha-<short>`, cuts a GitHub Release, verifies the image is actually pullable, and (if configured) pings a deploy hook to redeploy the public demo.
- **Public demo**: `DEMO_MODE=true` seeds a curated, realistic fixture (`apps/web/lib/server/demo-fixture.ts` — accounts across every liquidity class, a multi-level tag tree with budgets, plan settings/members/items/account-layers, and rules — all inserted through the app's own normal functions, never raw SQL) and makes the instance read-only (every mutating request 403s except login). Live today at the URL in the root README, password `demo`.

---

## 12. What We Are Not Building

Explicit non-goals, current as of this rewrite:

- **No AI/LLM categorization** — confirmed absent today (§6); the schema has a reserved seam, nothing more.
- **No OAuth / third-party identity provider** — the auth model (§5) is deliberately minimal for a single-operator instance.
- **No multi-tenancy, no waitlist, no invite-only beta** — this is self-hosted software, not a hosted service with a signup funnel.
- **No Postgres or any second database** — one SQLite file is the entire application database.
- **Israeli-specific financial-instrument account types or intelligence cards** (קרן השתלמות, pension, mortgage-rate comparison) — never built; the account model is generic.
- **No Stripe integration, no paid tiers** — removed in the pivot; there is nothing to gate.
- **No native mobile app.**
- **No bank-credential proxying, no Plaid/broker integrations** — the user brings their own data via CSV, the Buxfer CLI, or the Moneyman webhook.
- **No envelope budgeting or forced methodology** — the tag tree is whatever the user makes it.
- **No transaction execution** — read-only financial intelligence; no payments, no transfers initiated by the app itself.
- **No social/comparison features, no public third-party API** beyond the documented ingest webhook.

---

_Document created: 2026-04-03. Rewritten in full 2026-08-21 (v2.0) to match the shipped self-hosted architecture. Status: Living document — update in the same change as any architectural shift._
