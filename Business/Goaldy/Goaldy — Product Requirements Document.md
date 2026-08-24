| Field            | Value                  |
| ---------------- | ---------------------- |
| **Created**      | 2026-04-05             |
| **Last Updated** | 2026-08-24 v2.1        |
| **Version**      | 2.1                    |
| **Status**       | Draft                  |
| **Author**       | Product design session |

### Change Log

|Version|Date|Change|
|---|---|---|
|1.0|2026-04-05|Initial PRD — full feature set across 12 feature areas|
|1.1|2026-04-05|F4.1: removed `is_income` flag (redundant — inferred from cash flow); tag is source of truth for budget configuration with full period support; first-tag-is-primary rule made explicit. F4.2: primary tag rule clarified. F4.4: OOTB stack is i18n per locale. F4.6 added: tag tree transformation (refactor tools + migration). F7.3: budget repeat periods expanded to match Buxfer full support.|
|1.2|2026-04-05|F3.1/F3.3/F3.4: tagging model simplified to single-tag-or-100%-split; `transaction_tags` join table eliminated; Goal Allocation Assistant added to intelligence feed. F4.2: rewritten as single-or-split constraint documentation. F8.6: goal lifecycle updated with Fund and Review steps. F8.7 added: shortfall resolution flow — three options (extend deadline, increase contributions, one-time miss), deferral handling, automatic default after two deferrals, goal funding history.|
|1.3|2026-04-05|F3.5: label filter added. F3.6 added: Labels — dimensional markers separate from tags, multiple per transaction, no financial impact, filterable, common vocabulary suggested. Label action added to F3.3 transaction actions.|
|1.4|2026-04-05|F2.3: transfer detection pass added to import pipeline. F2.6 added: transfer detection — confidence scoring, known payment pattern heuristics, Transfer Review queue, confirm-and-delete flow, external transfer handling. F3.2: split into triage banner + transfer review banner as distinct UI elements.|
|1.5–1.9|2026-04-05|Various refinements to transaction types, budgets, goals, and intelligence feed — see git history for detail.|
|1.10|2026-04-06|F14–F17 added: landing page, authentication/onboarding, session management, notification system.|
|**2.0**|**2026-08-21**|**Full rewrite to match the shipped self-hosted pivot.** Versions 1.0–1.10 described a multi-tenant hosted product: Google OAuth + Drive-as-database, Postgres, an invite-only waitlist beta, AI/Claude categorization, Israeli-specific financial-instrument cards, and Phase-4 Stripe monetization. None of that was built. What shipped is single-user, self-hosted software — one operator, one `goaldy.db` file, `ADMIN_PASSWORD`-based login, no signup funnel — plus a household financial-planning simulator ("Plan") and bilingual (English/Hebrew, RTL) i18n that the old PRD never described at all. This version rewrites every feature section against the actual codebase, deletes sections describing things never built, and marks in-progress work ("Coming soon" stub pages) honestly rather than as shipped. It also fixes an internal contradiction present since v1.1: F6.2 (old) referenced an `is_income` tag flag that F4.1 (old) had already said was removed — confirmed via the schema that no such column has ever existed in the shipped app; this version does not reintroduce it.|
|2.1|2026-08-24|F2.1: the in-app data importer shipped (Buxfer full-migration adapter, Stage 0+1; a generic, single-account CSV adapter with a column-mapping wizard and Mint/YNAB presets, Stage 2) — corrected from "Not built — deferred" to Shipped. OFX/QIF remains not built and unscoped, split out as its own row.|

---

## 1. Product Overview

### 1.1 Purpose

Goaldy is personal financial intelligence software for someone who wants Buxfer's feature depth and Monarch's UX polish, without handing their transaction history to someone else's servers. It replaces a stack of Buxfer + spreadsheets + manual net-worth tracking with one self-hosted tool: run the Docker container, point it at your accounts, own the file it produces.

### 1.2 Design Principles

- **No methodology religion.** The tag tree is whatever the user makes it — no forced envelope budgeting, no rigid category taxonomy.
- **Tag-first architecture.** There is one universal taxonomy (tags), not a separate "category" concept layered on top.
- **The app owns exactly one file, and you own that file.** Financial data lives in a single SQLite file on infrastructure you control — a Docker volume on your own server, a NAS, a laptop. There is no cloud storage dependency, no OAuth provider standing between you and your data, and no service that can revoke your access to it.
- **Buxfer is the feature baseline. Monarch is the UX baseline.**

### 1.3 Current State

Goaldy is self-hosted software available today via `docker compose up -d` (see the repo README). A public, read-only demo (see F14) lets anyone try it before self-hosting. There is no beta, no waitlist, no rollout gate — the software is either running on your infrastructure or it isn't.

---

## 2. Who This Is For

Goaldy is built for one operator at a time — the person running the container. There is no persona funnel, no referral flow, no distinction between "primary" and "secondary" user types the product needs to design onboarding around. The design target is simply: someone comfortable running a Docker container who wants their financial data to live somewhere they control, with real bilingual (English/Hebrew) support and multi-currency accounts for people whose financial life spans more than one country or currency.

---

## F1 — Accounts

### F1.1 Account Organization

Sidebar account tree grouped by liquidity class — Liquid, Semi-Liquid, Illiquid, Locked, Liability, plus Archived — with a Net Worth total pinned at the top, clickable through to the Net Worth dashboard tab. `is_closed` is the archive flag; archived accounts are excluded from the tree and net worth by default and listed separately.

### F1.2 Account Types

|Type|Liquidity class|
|---|---|
|Checking|Liquid|
|Savings|Liquid|
|Credit|Liability|
|Investment|Semi-liquid|
|Loan|Liability|
|Real Estate|Illiquid|
|Retirement|Locked|
|Manual|Illiquid|

Liquidity class is **never stored** — it's derived from `type` at the application layer (`getLiquidityClass()`), so it can never drift out of sync with the type itself. There are no Israeli-specific account types (קרן השתלמות, pension as distinct types) — `investment` and `retirement` cover that ground generically.

### F1.3 Balance Modes

Each account picks how its balance is computed: `transactions` (balance = sum of its transactions; the transaction list shows a running balance column) or `reported_value` (balance = a manually or integration-set `reported_value` field; no running balance). Either mode works for any account type — a brokerage account synced via an integration typically uses `reported_value` because market moves aren't captured as discrete transactions.

### F1.4 Multi-Currency

Every account carries its own currency (3-letter code, default ILS). Aggregation (net worth, dashboards) converts to a base currency using **live daily rates from the Frankfurter API**, cached in-process for an hour with a graceful fallback to stale cache on failure. `settings.base_currency` sets the default; an optional `settings.display_currency` overrides it for display without changing what's stored. This replaces the earlier design's Bank-of-Israel/ECB-specific rate sourcing with one general-purpose live-rate provider.

### F1.5 Account Management

Create, edit, archive; link an `account_number` for integration matching (e.g. Moneyman's per-account identifier); export.

---

## F2 — Data Ingestion

### F2.1 Real Ingestion Paths Today

|Source|Status|
|---|---|
|Moneyman webhook (`POST /api/integrations/moneyman`)|Shipped|
|Manual entry|Shipped|
|Buxfer CSV/TSV migration (`scripts/migrate-buxfer.ts`)|Shipped, but developer-only CLI — requires shell access to the container, not reachable from the running app|
|In-app data importer — Buxfer full-migration adapter and a generic, single-account CSV adapter with a column-mapping wizard (presets for Mint/YNAB)|Shipped (`docs/features/2026-08-19-in-app-data-importer`) — an in-app wizard (Import Statement) replaces the old dev-only CLI path for both cases|
|OFX/QIF in-app upload|**Not built** — named as a future stage of the in-app importer, no design doc yet|
|SimpleFIN, bank-specific parsers|Not built|

There is no encrypted-blob relay, no server-side queue with a TTL, no in-browser SQLite receiving decrypted payloads. Everything lands directly in the one server-owned `goaldy.db` via the same idempotent insert path (`ingestTransactions()`).

### F2.2 Dedup

Idempotent re-import: `import_hash = SHA-256(account_id | date | amount | description | external_id?)`, enforced as a UNIQUE constraint with `ON CONFLICT DO NOTHING`. Re-posting an overlapping range creates zero duplicates.

### F2.3 Ingest API

`POST /api/transactions` accepts a JSON array of transactions, authenticated via `Authorization: Bearer <API token>`. Response: `{ created, skipped, errors }`. This is the integration surface for scripts, webhooks, and the Moneyman scraper.

### F2.4 Transfers

A transaction's `type` can be set to `transfer`, with `transfer_source_account_id`/`transfer_dest_account_id` populated. Transfers are excluded from all income/expense reporting but included in per-account balance history and (optionally) account-level in/out charts. Transfer detection today is a matter of the ingesting integration or the user setting `type='transfer'` directly — there is no separate automated peer-matching/confirm-and-delete review queue built yet.

---

## F3 — Transaction Management

### F3.1 Transaction List

Full ledger view with running balance (for `transactions`-mode accounts), inline tag assignment, and account-context filtering.

### F3.2 Transaction Actions

Tag (single or split), set type, edit, add label, delete. `type` and tag are orthogonal — every type can carry a tag; type controls financial treatment, tag controls categorization.

### F3.3 Tagging Model

Three states: untagged, single tag (`tag_mode='categories'`, one `transaction_tags` row at position 0, weight 1), or split (`tag_mode='split'`, multiple `transaction_tags` rows whose weights must sum to 1 within a small tolerance — enforced server-side, rejected otherwise).

### F3.4 Labels

Free-form JSON-array strings on a transaction (`labels` column) — no financial impact, filter-only, unlimited per transaction.

### F3.5 Duplicate Detection

A dedicated Duplicates view: read-only, exact-match grouping across a user-configurable dimension set (account, date, amount, currency, description, type). The dimension set is stored as JSON in `settings.duplicate_criteria`, resolved through a closed column allowlist so no user input ever becomes a raw SQL identifier.

### F3.6 Not Yet Built

A triage banner, saved views, and a 30-day forecast panel were speculative additions in earlier drafts of this document — none of them exist in the shipped UI today. If wanted, they're future feature work, not something already delivered.

---

## F4 — Tag System

### F4.1 Tag Tree

Arbitrary-depth tree via adjacency list (`parent_id`); a tag is the sole source of truth for categorization — there is no separate "category" concept. Every tag can optionally carry a full budget-line configuration (amount, expense/income type, repeat period, rollover, optional account scope) — budget fields are all nullable, and their absence means the tag has no budget target. **No `is_income` column exists or has ever existed in the shipped schema** — this document previously (v1.1) said it was removed and then (v1.2 predecessor, "old F6.2") contradicted that by describing a dashboard driven by it; that contradiction is resolved here in favor of the schema, which has no such column.

### F4.2 Tagging Constraint

Single tag XOR split — see F3.3. Tag name uniqueness is enforced only among siblings (case-insensitive); the same name is valid under two different parents.

### F4.3 Tag Management

Tree view, rename, recolor, delete (cascade-safe — descendant transactions are reassigned or untagged, and rules whose `set_tag` action pointed at the deleted tag are repointed or flagged stale).

### F4.4 No Default Tag Stack Yet

There is no out-of-the-box bilingual starter tag tree seeded on first boot — a new install starts with an empty tag tree (the public demo's tag tree, by contrast, is a hand-authored fixture for demo purposes only, not a default new-install experience). A localized OOTB stack remains a reasonable future feature, not something currently shipped.

---

## F5 — Categorization

### F5.1 Rules Engine

The entire categorization system today. Structured conditions (`description`/`merchant`/`notes`/`labels`/`amount`/`account_id`/`type`/`currency`/`source`/`status`/`investment_type`/`date`; operators including `contains`/`equals`/`starts_with`/`greater_than`/`matches_regex`) evaluated in priority order, with ordered actions (`set_tag`, `add_label`, `set_type`, `set_transfer_source`/`dest`, `set_merchant`, `set_notes`). A `signature` hash deduplicates identical rules outright. Rules run on ingest and, optionally, on manual transaction creation — both gated by user-toggleable settings.

### F5.2 No AI Categorization

Earlier drafts of this document specified Claude-based categorization for anything rules didn't match. **This was never built.** A verified grep of the entire codebase for any Anthropic/Claude dependency returns nothing. The schema retains `category_source='ai'` and `ai_confidence` as an unused, reserved seam — an implementation detail, not a shipped capability. Any future AI categorization work should be scoped as new product work, not assumed to already half-exist.

---

## F6 — Dashboard

The real reporting surface is the **Dashboard** (`/dashboard`), not a separate "Reports" screen (F6.3 explains why F10, below, is a stub).

### F6.1 Cashflow Tab

A Sankey flow diagram plus two tag-breakdown cards (donut + table, folded to a chosen depth from one shared API response), and an in/out stacked bar chart (income up, expense down from the zero line, net line unstacked) with an optional "include transfers" toggle.

### F6.2 Net Worth Tab

Net worth trend over time and a liquidity-class breakdown, reading from account balances and the same balance-history endpoint used per-account.

### F6.3 Correction from earlier drafts

The old F6.2 described income/expense classification as driven by an `is_income` tag flag. No such flag exists (F4.1) — income vs. expense classification for dashboard purposes comes from a tag's `budget_type` where set, or the transaction's own `type` (`income`/`expense`) otherwise.

---

## F7 — Budgets

**Data model exists; UI does not yet.** Tags carry full budget configuration (F4.1) and a `BudgetLineSchema` exists in the shared schema package, but the Budgets nav page currently renders "Coming soon." This is accurately a gap between the data model and the UI, not a fully speculative feature — the moment someone builds the screen, the data it needs is already there.

---

## F8 — Plan (Household Financial Planning)

This replaces the old document's "Goals" section (F8) entirely — what shipped is bigger and shaped differently than a simple savings-goal tracker.

### F8.1 What It Is

A long-horizon net-worth/solvency simulator. Not a shared household ledger — one `goaldy.db` still has exactly one operator; household members are labels used to trigger age-based projections, not separate accounts or logins.

### F8.2 Settings

A monthly savings target, an inflation-rate assumption (3% default), an optional blanket return-rate override, a projection horizon (37 years by default), a start year, and an optional emergency-fund pillar with a target number of months of expenses covered.

### F8.3 Members

Household members (self / partner / child / parent / other) with a birth year — used to compute displayed ages per projection year and to trigger age-based plan items (e.g. a child's university tuition at a specific age).

### F8.4 Plan Items — Goals and Events

A unified model with two tiers:

- **GOAL** — a tracked savings target: target amount minus amount saved so far, no inflation adjustment, typically a shorter horizon.
- **EVENT** — a future one-off or recurring expense, projected forward with compound inflation from today.

Three trigger shapes: a fixed **year** (optionally recurring across a duration), a member's **age**, or a **recurring** interval with an optional end year. Creating a plan item auto-creates a tag under a `Goals`/`Events` root so that transactions tagged against it roll up into the plan automatically.

### F8.5 Account Layers

Every account can be assigned to one of three planning pillars: **investment** (with its own target return-rate contributing to a weighted blended return), **emergency** (the liquidity buffer), or **retirement** (with a monthly contribution). One pillar per account.

### F8.6 Projection

Year-by-year simulation: `balance × (1 + weighted return) + annual savings − that year's event expenses`. Flags a **danger** year (balance goes negative) and a **stress** year (balance falls under 18 months of savings), and surfaces the first danger year, the lowest projected balance, and the monthly savings required to avoid a shortfall.

### F8.7 Emergency Fund Summary

Compares the emergency-layer balance against trailing-12-month average expenses × the target month count, with a Critical / Warning / OK status.

### F8.8 Item Templates

A hardcoded set of ten common life-event templates (Bar/Bat Mitzvah, university tuition, wedding contribution, home renovation, etc.), each with an English name, a Hebrew name, and an ILS amount hint, offered as starting points when adding a new plan item. This is content seeding for a generic feature — distinct from the old document's Israeli-specific *financial-instrument* concept (F12, below), which was never built.

---

## F9 — Notifications

Replaces the old "Intelligence Feed" (F9) and "Notification System" (F17) sections — what shipped is one simpler thing, not two.

An append-only event/condition log (not a pluggable multi-channel email/WhatsApp/Telegram architecture): entries like `ingest.completed`, `ingest.failed`, and `api.error` are written as things happen, plus periodic condition checks run every 24 hours. The log is capped at 5000 rows with a 90-day retention window. There is no email delivery, no per-channel preference table, no "lead-gen" or Israeli-instrument card types — none of that was built, and there's no monetization concept for a card type to gate behind.

---

## F10 — Reports

**Stub.** The Reports nav page currently renders "Coming soon." The Dashboard (F6) is the actual reporting surface today; a dedicated Reports section (tag comparison, exportable charts, custom date-range reports) remains a real gap, not a built feature.

---

## F11 — Internationalization and Multi-Currency

Both real and shipped, replacing the old design's plans wholesale.

### F11.1 Locales

English (LTR) and Hebrew (RTL), `next-intl`-driven, 744 translated message keys per locale. Locale is stored in a `goaldy-locale` cookie, editable in Settings, and takes effect via `<html lang dir>` at the document root — real RTL layout, not just mirrored icons.

### F11.2 Currency Formatting

`Intl.NumberFormat`, never manual string concatenation.

### F11.3 Multi-Currency Accounts

Covered in F1.4 — live Frankfurter rates, `base_currency`/`display_currency` settings, per-account and per-transaction currency.

### F11.4 No Hebrew-Specific AI Handling

The old document specified Claude correctly categorizing Hebrew-language transaction descriptions. Since there is no AI categorization at all (F5.2), this doesn't apply — the rules engine matches Hebrew text exactly the same way it matches any other text (plain string/regex operators, no language-aware behavior needed or present).

---

## F12 — Israeli Financial Instruments

**Not built.** The earlier document specified dedicated account types and intelligence cards for קרן השתלמות (Keren Hishtalmut), pension, and a Bank-of-Israel-rate mortgage-refinancing comparison. None of this exists — `investment` and `retirement` are the only relevant account types, and they're generic. The Plan feature's item templates (F8.8) carry some Hebrew/ILS-flavored *seed content*, but that's a different and much smaller thing than a first-class Israeli-instruments feature area, and this section is otherwise removed.

---

## F13 — Settings and Integrations

### F13.1 Profile Settings

Base currency, optional display-currency override, language (en/he).

### F13.2 Auth and Integration Settings

- **Password**: change it; `GOALDY_RESET_PASSWORD` (an environment variable, not a UI flow) forces a reset and clears sessions if the operator is locked out.
- **API token**: view/rotate via `POST /api/token/rotate`, used for `Authorization: Bearer` integrations (Moneyman, scripts).
- No Google Drive connection settings — there is nothing to connect; the database is a local file.

### F13.3 Data Management

- **Backup**: a single file copy (`docker compose cp goaldy:/data/goaldy.db ./backup.db`) — no export format to design, the real file *is* the export.
- **Clear data**: deleting/replacing `goaldy.db` wipes every table, including auth state, and logs everyone out; the next boot reseeds the password from `ADMIN_PASSWORD` as if freshly installed. A short list of UI-only preferences (locale, sidebar panel state) live outside the DB in cookies/`localStorage` and survive a clear — none of it is financial data.
- No Postgres-backed "delete account" flow exists or is needed — deleting the one file *is* deleting the account, by construction.

---

## F14 — Landing Page

Rewritten in full — the earlier version specified an invite-only-beta acquisition funnel ("Request Early Access," a waitlist, Google-OAuth-gated invitation links) that has no place in self-hosted software with a public read-only demo already live.

### F14.1 Purpose and Access Model

A single static marketing page (GitHub Pages), not a signup gate. There is nothing to request access to — the software is available today. The primary calls to action are **Try the live demo** and **Self-host it** (with a Docker quickstart), not a waitlist form.

### F14.2 Structure

Nav (logo, live-demo link, GitHub link) → hero (self-hosted pitch, primary CTA to the demo) → problem framing (your data on someone else's server) → feature highlights (net worth across liquidity classes, plan ahead of expenses, rules-based categorization) → "Try it" (demo link, read-only/resets-on-release note) → "Self-host it" (`docker compose up -d` snippet, link to the full README) → footer (GitHub repo, `/api/docs`).

No pricing section, no waitlist form, no sign-in on the landing page itself — none of that applies to self-hosted software.

### F14.3 Design Language

The landing page uses the Goaldy brand palette and Plus Jakarta Sans, the same typeface as the app: a dark hero in Midnight Indigo (`#283593`) with Amber Gold (`#F9A825`) accents on CTAs and dividers, body sections on Cool Grey (`#F4F7F6`) — a premium feel that differentiates the marketing page from the app's own white-surface UI. This design-language guidance from the earlier draft still applies; only the copy and access model above changed.

---

## F15 — Authentication

Replaces the old OAuth-based F15 entirely.

### F15.1 Auth Model

No third-party identity provider. Two equal-trust paths: a browser session cookie, or a bearer API token for machine clients. `ADMIN_PASSWORD` (an environment variable) seeds the login password on first boot; if unset, the first password successfully submitted at the login screen becomes the password going forward.

### F15.2 Onboarding

**Stub.** The onboarding route currently renders a placeholder ("5-step onboarding — coming soon"). A first-run experience — currency/locale selection, an initial import prompt — remains real future work, not something already delivered; do not assume any of the old OAuth-era onboarding steps (Drive initialization, invite-token validation) apply, since none of the systems they depended on exist.

### F15.3 Password Recovery

`GOALDY_RESET_PASSWORD`, set as an environment variable and the container restarted: force-resets the password and clears all sessions, applied exactly once per value so it's safe to leave set without repeatedly logging the operator out.

---

## F16 — Session Management

### F16.1 Session Storage

An httpOnly `session` cookie backed by a row in a `sessions` table, 30-day expiry. No JWT, no third-party token to refresh, no separate multi-device single-session-invalidation logic — a single-operator instance has no concept of "another user's device" to guard against.

### F16.2 API Tokens

A separate, longer-lived credential (sha256-hashed, compared on `Authorization: Bearer`) for integrations that shouldn't hold a browser session — this is the mechanism the Moneyman webhook and any script use.

---

## 3. Non-Goals (Explicit Exclusions)

Confirmed current, replacing the old table:

- No AI/LLM transaction categorization (rules only, today).
- No OAuth or third-party identity provider.
- No multi-tenancy, no waitlist, no invite-only beta, no signup funnel of any kind.
- No Postgres or any database beyond the one SQLite file.
- No Israeli-specific financial-instrument account types or cards.
- No Stripe integration, no paid tiers — there is nothing to monetize a self-hosted tool's own instance against.
- No native mobile app.
- No bank-credential proxying or broker integrations — bring your own data via CSV, the Buxfer CLI, or the Moneyman webhook.
- No envelope budgeting or forced methodology.
- No transaction execution (read-only financial intelligence; no payments initiated by the app).
- No social/comparison features, no public third-party API beyond the documented ingest endpoint.

---

## 4. Current State Summary

Rather than a forward-looking phased SaaS rollout (the old §5), here is what's actually true today:

**Shipped:** accounts (all liquidity classes, multi-currency), transactions (ingest, dedup, splits, labels, duplicate detection), tags (arbitrary tree, budget-config-capable), rules-based categorization, the Cashflow/Net-Worth dashboard, the Plan household simulator, notifications (event/condition log), bilingual i18n/RTL, self-host Docker deployment, versioned CI/CD releases, and a public read-only demo.

**Modeled but not surfaced (data exists, UI doesn't):** Budgets screen, Reports screen, onboarding flow.

**Explicitly deferred, documented elsewhere:** a generic in-app data importer (`docs/features/2026-08-19-in-app-data-importer`), replacing the developer-only Buxfer CLI script.

**Never built, and not on any current roadmap:** AI categorization, Israeli-specific financial-instrument cards, OAuth/multi-tenancy, monetization.

---

_Document created: 2026-04-05. Rewritten in full 2026-08-21 (v2.0) to match the shipped self-hosted architecture. Status: Living document — update in the same change as any feature that ships or changes shape._
