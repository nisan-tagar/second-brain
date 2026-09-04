| Field            | Value            |
| ---------------- | ---------------- |
| **Created**      | 2026-04-03       |
| **Last Updated** | 2026-09-04 v3.1  |
| **Version**      | 3.1              |
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
|2.1|2026-08-26|**Budgets (`tag_budgets`) shipped — replaces the `tags.budget_*` columns entirely.** New §4a documents the generation model (one row per budget-configuration-over-a-date-range, a partial unique index enforcing at most one open generation per tag), the leaf-only invariant enforced server-side (a budgeted tag closes its own generation the moment it gains a child), the three-way edit/schedule branch in `setBudget()`, the closed-form rollover computation, the weighted/cash-flow-filtered `Actual` sum shared with every other tag-scoped money query in the app, the trailing-average hint, and the migration's leaf-only-aware backfill of the old columns. §4's `tags` table definition had the 7 `budget_*` columns (and `budget_account_id`, an 8th, dropped without replacement per the PRD's non-goal on forecasting) removed. §4's stale claim that `budget_type` fed dashboard income/expense classification is corrected — it never did; that classification has only ever come from a transaction's own `type`.|
|2.2|2026-08-26|§4a: documents `updateGeneration`/`deleteGeneration` — direct edit/delete of any generation (open or closed) by id, superseding the original "closed history is immutable" design. `updateGeneration` touches only amount/currency/type/period/custom_days/rollover, never `start_date`/`end_date`, so the partial-unique-index and CHECK-constraint invariants remain untouched by this change. `deleteGeneration` reopens the tag's previous closed generation when the deleted one was open (if one exists), and leaves a plain coverage gap when the deleted one was already closed — both in one transaction. New `PATCH`/`DELETE /api/tags/:id/budget/:generationId` routes.|
|2.3|2026-08-27|**Mobile-responsive foundation shipped — new §11a.** CSS-only dual-shell switch (both desktop and mobile shells always mounted server-side, toggled by `hidden md:flex`/`flex md:hidden` — no JS viewport-detection hook, since `(app)/layout.tsx` is a server component); hamburger-drawer navigation with Settings relocated to a top-bar overflow menu; `Modal` gained a `variant: 'sheet' \| 'drawer'` prop so the drawer reuses the same shared modal shell (Escape/focus-trap/scroll-lock/dialog-role) instead of a second hand-rolled overlay; new `/accounts` list screen grouped by `liquidityClass`; `AccountSelector`'s left-edge dropdown-clamp bug fixed as an unrelated side effect.|
|2.4|2026-08-29|**New §13 documents the planned Goaldy.AI architecture** (roadmap, not built) — the technical counterpart to the PRD's F18. Key decision: licensing/entitlement state cannot live in `goaldy.db` (§0/§5's single-tenant, no-`users`-table model holds), so a small, separately-run licensing service (Stripe-backed, one table, issues short-lived signed entitlement tokens) is architected as a second, distinct system rather than a new subsystem of the Next.js app — the self-hosted instance verifies that token's signature locally/offline (public key baked into the Docker image) rather than phoning home per-request. The planned MCP server is designed to be generated from the same OpenAPI 3.1 surface `lib/server/openapi.ts` already produces, and mints its own credentials through the *existing* scoped bearer-token mechanism (§5) — the licensing token only gates whether that capability is unlocked. §12: the blanket "no Stripe, no paid tiers" non-goal is corrected to point at §13 instead of asserting nothing will ever be gated.|
|**2.5**|**2026-09-04**|**§5 rewritten: security hardening + household multi-login (design, not yet built).** Prompted by a security assessment ahead of public self-host distribution (see `docs/superpowers/specs/2026-09-04-security-hardening-household-auth.md` in the app repo). Two changes, deliberately scoped together: (1) **login hardening** — per-identity rate limiting/lockout on failed logins, session-ID rotation on login, an idle timeout in addition to the 30-day absolute TTL, timing-safe bearer-token comparison (`crypto.timingSafeEqual` replacing `===` on the sha256 digest), and mandatory security response headers (CSP, `X-Frame-Options`, `Strict-Transport-Security`, `X-Content-Type-Options`) — none of this existed before. (2) **household multi-login** — a new `users` table (`id`, `email` UNIQUE, `name`, `password_hash`, `disabled_at`) replaces the single `settings.password_hash` key; `sessions` gains a `user_id` FK for per-person "log out everywhere" and audit attribution ("who did this" in logs and a "logged in as" UI surface). **Explicitly not multi-tenancy**: `goaldy.db` remains single-ledger — every household member sees the same accounts/transactions, so no query in `lib/server/queries/*` needs `user_id` scoping and none is added. `email` is captured as a **login identifier only** in this iteration, not a verified channel — no SMTP dependency, no password-reset-by-email yet; `GOALDY_RESET_PASSWORD` remains the recovery path. The shared `Authorization: Bearer` API token stays a single deployment-wide credential (a machine credential for household infra, not a person) — not made per-user. `plan_members` (§8) is a distinct, pre-existing concept (age-labels for plan projections) and is not merged with `users`.|
|2.6|2026-09-04|**New §5.3a and §5.5.** §5.3a: the default-credential bootstrap (`ADMIN_PASSWORD`/first-login-wins) is now explicitly transient — the bootstrap identity is created with `must_reset = true` and is server-side confined to a mandatory setup screen (real name/email, server-validated password strength against a local common-password list) until completed; this closes the "shipped with a known default password" pattern a public distribution can't risk. §5.1's `users` table gains `must_reset` and `removed_at` (soft-delete) columns. §5.5 is the full UI/UX spec for the Settings "Users" section — list/add/edit/disable/remove/reset-password, a self-lockout guardrail (can't disable/remove yourself as the last active identity), and the "logged in as" attribution surface — promoted from a one-line bullet to a real spec, matching the corresponding detail in `docs/superpowers/specs/2026-09-04-security-hardening-household-auth.md` v1.1.|
|**2.7**|**2026-09-04**|**§5.3a replaced, §5.4 updated: `ADMIN_PASSWORD` and "first password wins" removed entirely, not hardened.** `users` now boots empty; a one-time setup token (printed to container logs, same pattern as `API_TOKEN`) gates a first-boot setup wizard that collects a `settings.workspace_name` label plus the first real user in one step, then the wizard is permanently unreachable. This closes an unauthenticated-setup race that an empty-table design would otherwise open (whichever request reaches an unconfigured, network-reachable instance first would otherwise win it). `must_reset` is retained on `users` solely as a general admin-forced-reset mechanism, no longer tied to bootstrap. Matches `docs/superpowers/specs/2026-09-04-security-hardening-household-auth.md` v1.2.|
|2.8|2026-09-04|**§5.3a gains an explicit "public demo" exception; §11's demo bullet updated to match.** Removing `ADMIN_PASSWORD` in v2.7 would silently break the public Render demo, which logs in with a fixed published password (`demo`) seeded via that exact env var. `DEMO_MODE=true` is now specified to bypass the setup-token gate and auto-provision a fixed demo identity on every boot — justified only by the demo's read-only + no-persistent-disk properties, and explicitly not a precedent for the real self-host boot path, which keeps no fixed-credential mechanism at all. Matches `docs/superpowers/specs/2026-09-04-security-hardening-household-auth.md` v1.3.|
|**2.9**|**2026-09-04**|**New §5.6 (MFA) and §5.7 (password recovery ladder).** §5.6: TOTP chosen over WebAuthn as the first MFA method specifically because it has no secure-context/stable-origin precondition, unlike WebAuthn — fits self-hosted reality; opt-in per user, hashed one-time recovery codes, WebAuthn documented as a later addition once TLS is the assumed default. §5.7 replaces the single-tier `GOALDY_RESET_PASSWORD` mention with a three-tier ladder: another logged-in household member (zero infra), optional self-configured SMTP (off by default, opt-in), and `GOALDY_RESET_PASSWORD` extended to target a user by email and clear their MFA in the same operation. Explicitly excludes any Goaldy-operated recovery service. Matches `docs/superpowers/specs/2026-09-04-security-hardening-household-auth.md` v1.4.|
|**3.0**|**2026-09-04**|**§5.1, §5.3, §5.6, §5.7 flipped from "design, not yet built" to shipped** — Phases 1–7 of the implementation plan (`docs/superpowers/plans/2026-09-04-security-hardening-household-auth.md`) all landed: the `users`/`mfa_recovery_codes` schema, the full auth rewrite (no default credential, per-identity rate limiting, session rotation/idle timeout, timing-safe bearer compare), household user management in Settings, TOTP MFA (encrypted at rest, opt-in), and all three password-recovery tiers including the new SMTP-based Tier 2. §5.3's body corrected to state plainly which pieces of "login hardening" actually shipped (rate limiting, session hardening) versus what's still open (security response headers, the TLS/reverse-proxy deploy-docs requirement — tracked under Phase 3/8). No design changes in this entry, purely a shipped/not-shipped status correction against the real code.|
|3.1|2026-09-04|**§5.3's remaining open items closed.** Security response headers now shipped (`apps/web/lib/server/security-headers.ts`, wired via `next.config.ts`'s `headers()` hook — verified against a real running production server via curl, not just config review): `X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`, `Referrer-Policy`, and a same-origin-scoped CSP (deliberately keeps `'unsafe-inline'`/`'unsafe-eval'` for scripts, since Next.js's own hydration needs it and a nonce-based CSP is separate follow-up work — but blocks every cross-origin fetch/image/font/script/style/WebSocket, closing the exfiltration half of an XSS payload). README gained a "Putting it on the internet" section stating a reverse-proxy-terminated-TLS deployment is required for anything reachable outside the operator's own network, with a Caddy example; `docker-compose.yml` gained a matching inline comment pointing operators to it. This closes every item from the original pre-launch security assessment.|

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
│       │   │   ├── budgets/    # Shipped — list/grid views, edit + history modals (§4a)
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
  created_at
);
```

Sibling-scoped name uniqueness only (case-insensitive) — the same name is legal under two different parents. **There is no `is_income` column anywhere in this schema, and no `budget_*` columns either** — a tag no longer doubles as a budget-line definition (that was the pre-2026-08-25 shape; see §4a for what replaced it). Income/expense classification for the dashboard comes solely from a transaction's own `type`, never from any tag-level field. (An earlier draft of this document's companion PRD asserted both that `is_income` was removed *and* that it drove a dashboard — that PRD contradiction is fixed as of the v2.0 rewrite; a later draft of the same PRD also claimed a tag's `budget_type` fed dashboard classification, which was never actually true and is corrected in PRD v2.2.)

Splits are enforced in `apps/web/lib/server/queries/transaction-tags.ts`: `categories` mode always writes a single row (position 0, weight 1); `split` mode requires weights to sum to 1 within a small tolerance and rejects negative weights (`TagWeightError`).

### 4a. Budgets (`tag_budgets`)

Budgeting moved off the `tags` row entirely into its own table, one row per budget **generation**:

```sql
CREATE TABLE tag_budgets (
  id, tag_id REFERENCES tags(id) ON DELETE CASCADE,
  amount REAL NOT NULL,
  currency TEXT NOT NULL,        -- snapshotted at generation creation (§ below) — independent of base/display currency
  type TEXT CHECK (type IN ('expense','income')),
  period TEXT CHECK (period IN ('weekly','biweekly','monthly','quarterly','annually','custom')),
  custom_days INTEGER,           -- only when period='custom'; clamped to >=1 everywhere it's consumed
  rollover INTEGER DEFAULT 0,
  start_date TEXT NOT NULL,
  end_date TEXT,                 -- NULL = this generation is open (current)
  created_at,
  CHECK (end_date IS NULL OR end_date >= start_date)
);
CREATE UNIQUE INDEX idx_tag_budgets_open ON tag_budgets(tag_id) WHERE end_date IS NULL;
```

**One row is not a budget — a sequence of rows is.** A tag accumulates a new generation every time its amount/period/rollover changes on a future date, or every time it's closed by the leaf-only rule below. `idx_tag_budgets_open` is a *partial* unique index: it enforces "at most one open (`end_date IS NULL`) generation per tag" at the database layer, not just in application code.

**Any generation can be edited or deleted (2026-08-26 revision — supersedes the original "closed history is immutable" design).** `updateGeneration(generationId, fields)` updates amount/currency/type/period/custom_days/rollover on a generation by id with no open/closed guard at all — `start_date`/`end_date` are never part of that update, so the partial-unique-index and CHECK-constraint invariants above are untouched. `deleteGeneration(generationId)` removes a generation by id: if it was the tag's OPEN generation, the tag's next most-recent CLOSED generation (by `end_date`, if one exists) is REOPENED in the same transaction (`end_date` set back to `NULL`) so a deletion reads as "undo the last change," not "abruptly unbudget the tag"; deleting any other (already-closed) generation just removes it, leaving that date range with no covering generation — the same as if it had never existed, with no gap-filling. This trades the original point-in-time-correctness guarantee (a past period's reported figures could not previously change after the fact) for direct, Buxfer-style management of a tag's whole budget history from one view.

**Leaf-only (enforced server-side, not just in the UI).** `setBudget()` (`lib/server/queries/tag-budgets.ts`) rejects opening a generation on any tag with ≥1 child. The moment a budgeted tag gains a child — reparenting an existing tag under it, or creating a new one — `closeOpenGenerationForNewParent()` fires unconditionally (exploiting the fact that a leaf-only invariant means "gained a child" and "must close now" are the same event) and closes that tag's open generation, clamping `end_date` to `max(day-before-the-change, the generation's own start_date)` so a same-day open-then-close can never violate the CHECK constraint above. The API layer (`POST`/`PATCH /api/tags`) surfaces the closed generation back to the client as `closedParentBudget`, which is what drives the "move this budget to the new child?" prompt (D6 below) — a UI convenience, not a second source of truth.

**Editing vs. scheduling.** `setBudget(tagId, input)` is a three-way branch on `input.startDate` against the tag's current open generation: an earlier date is rejected outright; the *same* start date updates the open row in place (amount/currency/type/period/custom_days/rollover — this is what "editing your budget" means from the UI); a *later* date closes the current generation (`end_date = day before`) and inserts a new open one. A new generation's `start_date` is also checked against `MAX(end_date)` across *all* of the tag's history (not just the open row) so a back-dated start can never land inside an already-closed generation's range — `getGenerationCovering()` has no tiebreaker, so two generations covering the same day would make point-in-time lookups arbitrary.

**Moving a budget (D6).** Two situations produce a `closedParentBudget` the client can offer to copy forward: a budgeted tag gaining a child (above), and reparenting a tag under an already-budgeted tag (the same close fires on the *parent*, for the same leaf-only reason, before the reparent completes). If the destination for that offer already has an open generation of its own, or has children of its own, the roll endpoint (`POST /api/tags/:id/budget/roll`) rejects it (409) and the UI shows a plain notice instead of an accept/decline choice — there is no ambiguous partial-move.

**Rollover — a closed form, not a running total.** When `rollover=1`, an occurrence's `Available` folds in every prior occurrence of the *same generation*: `Available_n = n·amount − Σ(Actual_1..n)`, computed fresh from two aggregate queries every time (current occurrence's actual, and the cumulative actual since the generation's `start_date`) — never a step-by-step carry stored anywhere, so there is nothing to drift or replay. Rollover never crosses a generation boundary by construction: closing a generation and opening a new one resets the sum to zero, because the new generation's own `start_date` is where the aggregate window begins.

**`Actual` is always a weighted, cash-flow-filtered sum.** `computeBudgetStatus()`'s `sumTaggedActualInRange()`, the "not budgeted" aggregator's sum, and the `budget.exceeded` notification check all join `transaction_tags` and compute `SUM(t.amount * COALESCE(tt.weight, 1))` under the same `cashflowExcludeSQL()` predicate every other tag-scoped money query in the app uses (`lib/server/queries/tag-flow.ts` is the reference implementation) — a weight-0 secondary tag (the default for a transaction's non-primary tag in `categories` mode) contributes nothing, a split contributes its proportional share, and a transfer between the user's own accounts is excluded, exactly as it is on the dashboard. `Actual`/`Budgeted`/`Available` are always reported as positive-oriented "how much" figures (via a `resolvedType: 'expense'|'income'` resolved once per status computation and applied consistently), never as raw signed sums a caller would have to re-interpret.

**Trailing average.** `computeTrailingAverage()` reuses the same occurrence-window primitive (`lib/domain/budget-period.ts`'s `walk()`/`stepEnd()`) that resolves "what window does this reference date fall in" — applied N times backward instead of once, so index and window can never disagree with the rest of the feature. Occurrence count is chosen per period type to keep the total look-back at roughly a year or two regardless of granularity (12 trailing weeks/months, 8 quarters, 3 years, 12×N days for custom).

**Migration note.** `migrate.ts`'s one-time backfill of the old `tags.budget_*` columns into `tag_budgets` checks each budgeted tag's child count: a leaf tag's old budget becomes an open generation as before, but a **non-leaf** tag's old budget (legal under the pre-2026-08-25 schema, since parent-tag budgets were allowed then) is inserted **already closed**, on the migration day — preserving it as history rather than landing a real upgrade with a generation that immediately violates the new leaf-only invariant and can never be edited.

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

No OAuth, no external identity provider. **Still single-tenant, single-ledger** — `goaldy.db` holds one household's financial data, full stop. What changed in v2.5 is *who can authenticate as a member of that household*, not what they can see once in.

### 5.1 Identities — `users` table (shipped)

Replaces the single `settings.password_hash` key with a `users` table: `id`, `email` (UNIQUE, the login identifier), `name` (display name — "logged in as" UI, audit/log attribution), `password_hash` (bcrypt), `disabled_at` (nullable — revoke one member without touching anyone else's credential), `created_at`.

**Deliberately not multi-tenancy.** Every household member sees the exact same accounts, transactions, budgets, and plan — nothing in `lib/server/queries/*` gains a `user_id` filter, and none should ever be added under this design; that would be a different, much larger feature with its own IDOR/row-isolation risk this design explicitly avoids. The only things scoped per-user are **identity, credentials, sessions, and attribution** — not data.

`email` is captured as a **login identifier only** in this iteration, not a verified channel: no SMTP dependency, no verification loop, no password-reset-by-email. `GOALDY_RESET_PASSWORD` (below) remains the sole recovery path until a mail-sending capability is deliberately added as separate future work (it would also enable notifications/digests — the reason to capture the field now rather than retrofit it later).

`plan_members` (§8) is a **distinct, pre-existing concept** — birth-year labels used to trigger AGE-based plan projections — and is not merged with `users`; a plan member is not necessarily someone who logs in, and vice versa.

`users` also carries `must_reset` (bool) and `removed_at` (soft-delete, nullable) — see §5.3a and §5.5.

The shared `Authorization: Bearer` token (below) stays a **single deployment-wide credential**, not per-user — it authenticates household infra/integrations (Moneyman, scripts), not a person, and every logged-in person already sees the same data regardless of which credential reaches it.

### 5.2 Authentication paths (`apps/web/lib/server/auth.ts`)

- **Browser**: an httpOnly `session` cookie, backed by a row in `sessions` — now carrying a `user_id` FK (was anonymous). 30-day absolute expiry **plus a shorter idle timeout** (new — sessions with no activity for the idle window expire even inside the 30-day window). Session ID rotates on every successful login (new — mitigates session fixation).
- **Machine clients**: `Authorization: Bearer <token>`, compared against `settings.api_token_hash`. The digest comparison moves from `===` to `crypto.timingSafeEqual` (new — the prior string-equality compare on a public sha256 digest was a low-severity timing side-channel, cheap to close).

### 5.3 Login hardening (shipped)

Prompted by a pre-public-launch security assessment (`docs/superpowers/specs/2026-09-04-security-hardening-household-auth.md` in the app repo) — the single biggest gap found was **zero brute-force protection**: unlimited password attempts against `/api/session`, no lockout, no delay. v2.5 adds, per-identity (not per-deployment, so one member's mistyped password doesn't lock out the household):

- Rate limiting + progressive lockout on failed logins against a given email (`lib/server/login-rate-limit.ts`) — shipped.
- Session-ID rotation on every login, a 7-day idle timeout on top of the 30-day absolute TTL, and timing-safe bearer-token comparison (`lib/server/session.ts`, `lib/server/auth.ts`) — shipped.
- Security response headers app-wide — shipped (`apps/web/lib/server/security-headers.ts`, wired via `next.config.ts`'s `headers()` hook): `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Strict-Transport-Security`, `Referrer-Policy`, and a same-origin-scoped Content-Security-Policy. The CSP is deliberately not maximally strict — it keeps `'unsafe-inline'`/`'unsafe-eval'` for scripts since Next.js's own hydration relies on inline script execution and a nonce-based CSP is real, separate follow-up work — but it does block every fetch/XHR/image/font/script/style load and every WebSocket connection to any origin but this one, which closes the exfiltration half of an XSS payload even without fully hardening the injection half.
- **Still open:** documentation that a reverse proxy terminating TLS is **required**, not optional, for any non-localhost deployment — `docker-compose.yml` binding `3000:3000` directly with no TLS story is a real gap the deploy docs must still close. Tracked under Phase 8 of the implementation plan.

### 5.3a No default credential — empty-table bootstrap + setup wizard (revised)

**Superseded approach, kept for context:** an earlier revision of this section kept `ADMIN_PASSWORD`/"first password wins" as a bootstrap credential and *forced* a graduation off it via a `must_reset` flag. That closes the window but doesn't remove the hole — a public self-host distribution means a scanner knows Goaldy's bootstrap behavior as well as any legitimate operator. **This revision removes the default credential from existing at all.**

`users` boots **empty**. There is no password to guess and no "first submission wins" race against the network. The app instead runs a **one-time setup wizard** the first time `users` is empty:

1. First boot generates a **one-time setup token**, printed to container logs once (identical reveal-once pattern to today's `API_TOKEN`).
2. The setup wizard requires that token plus, in one step: a **workspace label** (free-text display name, e.g. "Tagar Family" — stored as `settings.workspace_name`, shown on the login screen and topbar) and the **first user's** name, email, and a password passing strength validation (below).
3. Completing the wizard creates the first `users` row, sets `workspace_name`, single-use-consumes the setup token, and logs the new user in directly.
4. **Once `users` is non-empty, the setup wizard is permanently unreachable** — no lingering unauthenticated surface remains to probe. Every subsequent household member is added only via the Settings "Users" screen (§5.5) by an already-authenticated member, never through a second setup flow.

**Why the token, not just an empty table:** an empty-`users`-table design *without* a gate is itself an unauthenticated setup race — whichever HTTP request reaches an unconfigured, network-reachable instance first becomes its sole account holder. A public VPS is reachable by definition, so this isn't hypothetical. The setup token requires container-log access (i.e., the actual operator) to complete setup, closing the race entirely regardless of how long the instance sits exposed before the operator finishes clicking through it.

**Exception, scoped and explicit: the public demo (§11).** The public demo deployment logs in with a fixed, published password (`demo`) today via `ADMIN_PASSWORD`, and removing that env var without a replacement would break it. `DEMO_MODE=true` therefore bypasses the setup-token flow and auto-provisions a fixed demo identity (`demo@goaldy.app` / `demo`, workspace label "Goaldy Demo") on every boot, alongside the existing fixture seeding. This is safe *only* because the demo is simultaneously read-only (every mutating request 403s except login), backed by no persistent disk (reseeded from empty on every release, so nothing a visitor does survives), and gated by `ensureReady()`'s existing refusal to seed onto a DB with pre-existing accounts it didn't create — none of which hold for a real self-hosted deployment. This exception must never be read as license to reintroduce any fixed-credential concept into the real boot path described above.

**Password strength** is validated server-side (a client-only check is not a control, since this app explicitly supports scripting directly against its REST API): minimum length plus a composition or strength-score rule, checked against a small local common/breached-password list — no live external API call, preserving the "no cloud dependency" property of §0.

`must_reset` (on `users`, §5.1) survives purely as a general **admin-forced-reset** mechanism (e.g. "force this member to change their password next login," from Settings) — it is no longer load-bearing for bootstrap, since there's no bootstrap credential left to graduate off of. `ADMIN_PASSWORD` and any `ADMIN_EMAIL` notion are dropped entirely, not deprecated — the setup wizard replaces their function.

### 5.4 Bootstrap and recovery

`ensureReady()` (`apps/web/lib/server/auth.ts`, memoized once per process): on first boot, generates an API token (from the `API_TOKEN` env var, or a random one logged once) and, separately, the one-time setup token described in §5.3a if `users` is still empty. There is no password-related env var seeding — the setup wizard is the only way a first user comes into existence. `GOALDY_RESET_PASSWORD` remains a valid emergency recovery path once real users exist (targets a named user by email via `GOALDY_RESET_PASSWORD_FOR`, or the sole user if only one exists) — see §5.7 for the full recovery ladder this sits at the top of — and clears all sessions, applied exactly once per value (tracked by a hash marker, so leaving the env var set doesn't repeatedly wipe sessions).

`DEMO_MODE=true` (used only by the public demo, §11) additionally seeds a curated fixture on first boot, and refuses to run if the database already has accounts it didn't create.

`withAuth()` (`apps/web/lib/server/http.ts`) wraps every API route: runs `ensureReady()`, requires one of the two auth paths above, and — when `DEMO_MODE=true` — blocks every non-GET/HEAD request except `/api/session` (login/logout), returning a 403 on any attempted write.

OpenAPI 3.1 is generated straight from the Zod schemas (`lib/server/openapi.ts`), served at `/api/openapi.json`, with Swagger UI at `/api/docs`.

### 5.5 User management (Settings) — UI/UX

A new "Users"/"Household" section inside the existing Settings page (not a standalone route), visible identically to every logged-in identity — this design has **no roles/permissions tier**; every household member has equal standing to manage every other member's identity, matching the equal data access already established in §5.1. Requirements:

- **List**: name, email, status (Active/Disabled), last-active timestamp (from `sessions.last_activity`), row actions (Edit, Disable/Enable, Reset password, Remove).
- **Add**: name + email (validated, unique) + an inviter-set temporary password, shared out-of-band (no SMTP invite flow yet) — created with `must_reset = true` so the invitee is forced through real password setup on first login rather than the temporary password persisting.
- **Edit**: name/email only; password changes are a separate, explicitly logged "Reset password" action.
- **Disable**: sets `disabled_at`, immediately invalidates that identity's active sessions; reversible.
- **Remove**: soft-delete (`removed_at`) rather than a hard delete, so `user_id` attribution elsewhere (e.g. import-batch history) doesn't dangle; gated behind a plain-language confirmation dialog (`Modal`/`ConfirmDialog` per the existing convention).
- **Guardrail**: the UI blocks a user from disabling or removing themselves while they are the only remaining active identity — "lock the household out" is a mistake class the UI prevents, not just documents.
- **Attribution surface**: a "logged in as {name}" indicator in the topbar (desktop) and the mobile shell's equivalent, with Logout. New activity going forward (not backfilled onto pre-`users` history) records `user_id` wherever the underlying table has a natural place for it.

Full detail: `docs/superpowers/specs/2026-09-04-security-hardening-household-auth.md` in the app repo.

### 5.6 Multi-Factor Authentication — TOTP (shipped)

**TOTP (RFC 6238), not WebAuthn, first — a deployment-shape decision, not a simplicity shortcut.** WebAuthn/passkeys need a secure context bound to a stable origin; a self-hosted instance is routinely reached at a bare IP or mid-setup on a self-signed cert, which WebAuthn can't operate under. TOTP has no such precondition — a shared secret plus a clock works identically over plain HTTP on a LAN or HTTPS on the internet.

`users` gains `mfa_secret` (encrypted at rest — a live symmetric secret, more sensitive than a one-way password hash) and `mfa_enabled`. A new `mfa_recovery_codes` table holds 10 hashed one-time codes per enrollment (same `api_token_hash`-style hashing posture), issued once at setup and never re-shown. Setup (Settings, per user, **opt-in, not mandatory**) generates the secret, renders a QR code from a local library (no hosted QR-generation API — keeps §0's no-cloud-dependency property intact), and requires one valid code to confirm before `mfa_enabled` flips. Login gains a second step when `mfa_enabled`: a 6-digit code or an unused recovery code, checked before a session issues; a consumed recovery code is marked used and never accepted again.

Opt-in rather than mandatory because a solo self-hoster's risk profile differs from the "stranger on a public VPS" case this whole design otherwise targets — forcing MFA everywhere adds friction disproportionate to that case; revisit as mandatory if/when any multi-household or paid-tier context changes that calculus (§13). WebAuthn/passkeys are the deliberate next step once a stable domain + valid TLS is the assumed default deployment shape rather than aspirational README guidance — not attempted this pass.

### 5.7 Password (and MFA) recovery — a three-tier ladder (shipped)

Self-hosting means no central party can verify identity and hand back access, so "email you a reset link" can't be assumed to work — it needs mail infrastructure a self-hoster may not have. Recovery is a ladder, cheapest/most-available first, rather than one mechanism assumed to cover everyone:

- **Tier 1 — another logged-in household member resets it** (§5.5's "Reset password" action, zero new infrastructure). The primary answer for the common case, and the direct payoff of household multi-login over one shared password.
- **Tier 2 — optional self-configured SMTP, off by default.** A self-hoster who wants real password-reset-by-email enters their own SMTP credentials into Settings; only then does "Forgot password?" render on the login screen. Unconfigured, the link doesn't appear — no dependency forced on anyone who doesn't opt in. This is the opt-in realization of the "channel, not yet a channel" note on `users.email` from §5.1.
- **Tier 3 — `GOALDY_RESET_PASSWORD`, extended.** The host-level escape hatch for total lockout now needs `GOALDY_RESET_PASSWORD_FOR=<email>` to target a specific user (rather than assuming one password for the whole app) and must also clear that user's `mfa_enabled`/`mfa_secret` in the same operation — otherwise a password-lockout fix immediately surfaces an MFA lockout behind it, a worse outcome than the original problem. Requires container/host access, proportionate to the same bar as reading the API token or setup token from logs.

**Deliberately excluded:** any Goaldy-operated password-reset or support channel — that would reintroduce the cloud dependency and support liability the self-hosted pivot exists to avoid, and contradicts the free/no-liability framing this security effort was scoped under.

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
- **Public demo**: `DEMO_MODE=true` seeds a curated, realistic fixture (`apps/web/lib/server/demo-fixture.ts` — accounts across every liquidity class, a multi-level tag tree with budgets, plan settings/members/items/account-layers, and rules — all inserted through the app's own normal functions, never raw SQL) and makes the instance read-only (every mutating request 403s except login). Live today at the URL in the root README, password `demo`. **Auth mechanism (§5.3a):** once household multi-login/no-default-credential (v2.5/v2.7) ships, the real self-host boot path drops `ADMIN_PASSWORD` entirely — `DEMO_MODE` keeps a fixed, published demo identity as a scoped, explicitly-carved-out exception (bypasses the setup-token gate, safe only because the demo is simultaneously read-only and non-persistent), not a surviving general mechanism.

---

## 11a. Mobile Shell Architecture

Shipped 2026-08-26. Goaldy has no responsive design before this — the sidebar only ever
collapsed to an icon-only "mini" mode, never a true mobile shell. This section documents
the mechanism; the durable project rules it establishes also live in the app repo's root
`CLAUDE.md` under "Mobile and Other Devices," since they bind all future work on the app,
not just this feature.

**The shell switch is CSS-only, not a JS mode flag.** `apps/web/app/(app)/layout.tsx` is
a **server component** — it reads the session cookie and redirects unauthenticated users,
so it genuinely renders on the server before any client-side JS runs. A `useIsMobile()`
hook picking between two shells would have to guess on first render (the server has no
viewport width) and correct itself after hydration — a visible flash of the wrong shell
on a phone. Instead: both the existing desktop `<Sidebar>`/`<Topbar>` tree and a new
`<MobileShell>` (drawer + mobile top bar + floating action button) render unconditionally
from that server layout, wrapped in Tailwind's `hidden md:flex` / `flex md:hidden`
respectively — real `display: none` (never a soft hide like `opacity-0`), so the hidden
shell's controls are non-focusable and invisible to assistive tech, not just visually
gone. The breakpoint is Tailwind's default `md` (768px); the app has no custom breakpoint
config. Both shells receive the same `children`, so the CSS toggle is instant on resize
with no re-fetch. `MobileShell` pins `direction: 'ltr'` on its own root, mirroring the
desktop wrapper's existing pin — the whole app currently assumes LTR layout (many
physical-direction Tailwind classes depend on it), so this preserves today's status quo
on a surface (mobile) that never existed before, rather than introducing untested RTL
behavior; a dedicated mobile-RTL pass is separate, larger future work.

**Navigation is a hamburger drawer, not a bottom tab bar** — one ranked list covering
every section (Dashboard, Transactions, Tags, Budgets, Rules, Plan, Reconciliation,
Reports, Accounts), no primary-vs-demoted split. Settings is deliberately excluded from
the drawer; it lives only in the mobile top bar's "⋮" overflow menu, one tap further than
the drawer's contents, since it's reached far less often. A universal floating action
button (the same global Add-menu component the desktop topbar already used, given a
`variant="fab"` rendering) sits bottom-right on every mobile screen for quick-add from
anywhere.

**Shared full-screen-sheet primitive.** `Modal`/`ConfirmDialog` (the app's only modal
primitive; every modal in the codebase goes through one of them) already becomes
viewport-height-aware for the on-screen-keyboard case. This shipped alongside it: below
768px, both render as a full-screen bottom sheet (flush to the screen edges, top corners
only, `items-end` overlay) instead of a small centered dialog. `Modal` also gained a
`variant?: 'sheet' | 'drawer'` prop (default `'sheet'`, zero behavior change for every
pre-existing caller) so the same shared shell — Escape handling, focus trap, body-scroll
lock, dialog `role`/`aria-modal`, and the existing modal-stack coordination for
nested-modal cases — could back the hamburger drawer too, rather than a second,
hand-rolled full-screen overlay. (A hand-rolled drawer overlay was the original design;
it was rejected mid-implementation specifically because this repo enforces, via a
regression test, that `Modal`/`ConfirmDialog` are the *only* place a full-screen overlay
may be constructed — the same discipline that already prevented eleven independently
hand-rolled modal shells from drifting on Escape/focus/z-index years earlier.) Migrating
the app's other anchored popovers (`TagPicker`, `AccountSelector`, several table-cell
dropdowns) onto this same primitive is deferred to each component's own future per-view
work — this shipped scope only guarantees the primitive exists and what its contract is.

**New `/accounts` list screen.** Previously, browsing accounts required the desktop
sidebar's own account-tree panel; there was no equivalent on a screen with no sidebar.
The new screen groups accounts by `liquidityClass` (`liquid` → `semi_liquid` →
`illiquid` → `locked` → `liability`, matching `packages/schema/account.ts`'s
`LiquidityClassEnum` exactly — a grouping distinct from the desktop sidebar's own
account-tree panel, which groups by account *type* instead), each group a header with a
colored subtotal (green/red, following the app's existing money-direction color rule);
individual account balances are shown in the *base* currency alongside their group's
subtotal, not each account's own native currency, since the page's entire structure is
cross-account subtotaling — showing native currencies per row while subtotaling in base
would read as inconsistent, not more precise. Reachable only from the mobile drawer, not
the desktop sidebar, which already serves the same purpose there via its own tree panel.

**Fixed alongside this work, unrelated to the mobile breakpoint**: `AccountSelector`'s
dropdown clamped its horizontal position against the right edge only
(`Math.min(rect.left, windowWidth - dropWidth - margin)`), with no lower bound — on a
narrow viewport, or simply a trigger near the left edge with a wide dropdown, the
expression could go negative and the panel would render partially off-screen to the
left. Extracted into a small, unit-tested pure function
(`components/ui/dropdown-position.ts`) since this reproduces on desktop too.

---

## 13. Goaldy.AI Architecture (Roadmap, Not Built)

Technical counterpart to PRD F18. Nothing here is implemented — this documents the
agreed design so implementation can start from settled decisions rather than
re-litigating them.

### 13.1 Why This Can't Live in `goaldy.db`

§0/§5 establish the whole app around one hard invariant: one process, one file, no
`users` table, no OAuth, no multi-tenancy. Licensing/entitlement state — "does this
installation currently have an active Goaldy.AI subscription" — is inherently
multi-tenant (it spans every self-hosted instance that ever subscribes), so it cannot be
added as a table in `goaldy.db` without breaking that invariant. It has to be a second,
separately-run system, not a new subsystem of the Next.js app.

### 13.2 Licensing Service

A small service, run centrally, entirely separate from anything a user self-hosts:

- **Stripe** owns billing and identity — Checkout + Billing + webhooks
  (`customer.subscription.updated`/`deleted`). No accounts system of its own.
- One table: `license_key`, `stripe_customer_id`, `stripe_subscription_id`, `status`,
  `entitlements` (JSON: `{chat: bool, mcp: bool}`).
- Two endpoints: `POST /activate {license_key}` → validates against Stripe status,
  returns a signed (Ed25519), short-lived token asserting entitlements + expiry; a
  Stripe webhook handler flips `status` on cancellation/payment failure.
- **This service never receives a user's financial data** — only a license key in,
  an entitlement token out. That boundary is load-bearing for the self-hosted trust
  story and should be stated in any future public architecture docs, not just here.

### 13.3 Entitlement Verification in the Self-Hosted Instance

The licensing service's public key is baked into the Docker image at build time. A new
Settings field ("Goaldy.AI license key") calls `/activate` once; the resulting signed
token is cached in the existing `settings` key/value store (§4 — same pattern as
`password_hash`/`api_token_hash`, no new storage concept). Every AI/MCP-gated request
checks that cached token's signature and expiry **locally**, with a background daily
re-sync and a 7–14 day grace window if the licensing service is briefly unreachable.
Constant phone-home per request was explicitly rejected: it would contradict the
self-hosted "own your data" pitch even though the licensing service itself never touches
that data.

### 13.4 Chat Tier

An in-app assistant scoped to that instance's own data, finally exercising the
`category_source='ai'`/`ai_confidence` reserved seam (§4, §6) for LLM-assisted
categorization, plus natural-language query, insight-to-action recommendations layered
on the existing Plan projection engine (§8) output, a Monte Carlo–style range around that
same projection, and a push surface built on the existing notification log (§10) rather
than a new delivery mechanism.

### 13.5 MCP Tier

An MCP server exposing one instance's harmonized accounts/tags/budgets/Plan state to
external agents. Planned to be **generated from the existing OpenAPI 3.1 surface**
(`lib/server/openapi.ts`) rather than hand-built as a second API — the same Zod schemas
already drive both. MCP credentials are minted through the *existing* scoped
bearer-token mechanism (§5), not a new auth system; the licensing token from §13.3 only
gates whether the capability is unlocked at all. Default scope is read-only; a
write-capable scope (tag, categorize, move a budget) requires an explicit, separate
grant — an external agent should never get unscoped write access to a ledger by default.

### 13.6 Explicitly Out of Scope

Tax optimization (loss-harvesting, contribution-limit tracking, RSU-vesting timing) —
evaluated and rejected as a distinct, higher-liability product, not part of this
architecture.

---

## 12. What We Are Not Building

Explicit non-goals, current as of this rewrite:

- **No AI/LLM categorization** — confirmed absent today (§6); the schema has a reserved seam, nothing more.
- **No OAuth / third-party identity provider** — the auth model (§5) is deliberately minimal for a single-operator instance.
- **No multi-tenancy, no waitlist, no invite-only beta** — this is self-hosted software, not a hosted service with a signup funnel.
- **No Postgres or any second database** — one SQLite file is the entire application database.
- **Israeli-specific financial-instrument account types or intelligence cards** (קרן השתלמות, pension, mortgage-rate comparison) — never built; the account model is generic.
- **No paid tier or Stripe integration exists in the self-hosted app today.** A paid Goaldy.AI layer, and the licensing service that gates it, are planned — see §13 — and are deliberately architected as a system separate from `goaldy.db`, not a violation of this app's single-tenant model.
- **No native mobile app.**
- **No bank-credential proxying, no Plaid/broker integrations** — the user brings their own data via CSV, the Buxfer CLI, or the Moneyman webhook.
- **No envelope budgeting or forced methodology** — the tag tree is whatever the user makes it.
- **No transaction execution** — read-only financial intelligence; no payments, no transfers initiated by the app itself.
- **No social/comparison features, no public third-party API** beyond the documented ingest webhook.

---

_Document created: 2026-04-03. Rewritten in full 2026-08-21 (v2.0) to match the shipped self-hosted architecture. Status: Living document — update in the same change as any architectural shift._
