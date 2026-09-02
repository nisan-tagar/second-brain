# Goaldy PRD — Self-Hosted Personal Finance App

| Field            | Value                |
| ---------------- | -------------------- |
| **Created**      | 2026-06-07           |
| **Last Updated** | 2026-08-08           |
| **Version**      | 3.1                  |
| **Status**       | Active               |

> **Reset note:** This document was reset to a clean, forward-looking baseline on 2026-06-07. All prior architecture explorations (cloud relay, browser-side SQLite, Google Drive sync, Postgres, tiered Free/Pro model) are **superseded and intentionally not preserved**. The product is now a single-user, self-hosted app with one SQLite file managed by the backend. The changelog below starts here and tracks changes going forward.

---

## Change Log

| Version | Date       | Change                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **3.0** | 2026-06-07 | Clean baseline. Single-user self-hosted app. Node + Express backend, React frontend, one SQLite file holding financial data + user settings + preferences. Generic REST API (all resources, all verbs) with bearer-token auth; `POST /api/transactions` is the batch, idempotent ingest primitive. Full OpenAPI/Swagger spec is a required deliverable. All prior session history collapsed and dropped. |
| **3.1** | 2026-08-08 | Added a Visual Design Language subsection under §1.4 (Design Principles): colour roles and the green/red money-direction-only rule, the six-step type scale (with the four smallest steps' grandfathering explained), three radii, three elevation levels, and five named interaction states — codifying the existing identity as rules a reader can apply, not a token dump. Companion divergence list: `docs/design/2026-08-07-divergences.md` in the app repo. |

---

## 1. Overview

### 1.1 What Is Goaldy?

Goaldy is a **self-hosted personal finance ledger** you run on your own server (or NAS). It's a single-user web app that replaces Buxfer with a cleaner, faster interface and a complete REST API for integration.

**The pitch:** Download Docker image. Run on your server. Log in. Import your Buxfer export. Never use Buxfer again.

**One file, one source of truth.** Financial data, user settings, and preferences all live in a single SQLite file managed entirely by the backend. There is no second datastore, no external config service, no browser-side database. Back up one file, you've backed up everything. Move one file, you've migrated the whole app.

### 1.2 Why It Exists

Buxfer is 15+ years old. Its UI is dated. Its feature set is bloated with things you don't need. You manage Israeli + international accounts, run Moneyman locally, and want a **single dashboard that doesn't make you angry every time you open it.**

Goaldy solves that. It's:

- **Fast** — all queries sub-100ms (local SQLite)
- **Clean UI** — Monarch-grade visual design, not Buxfer's 2010s aesthetic
- **Yours** — runs on your hardware, data never leaves
- **Open** — full REST API (you can build integrations)
- **Simple** — one user, no collaboration, no complexity

### 1.3 Success Criteria

**Phase 0 ship gate:** You delete your Buxfer account and use Goaldy daily.

That's it. No secondary goals. One user. Done.

### 1.4 Design Principles

**No methodology religion.** Present data clearly. Don't impose a budgeting philosophy. YNAB has "jobs for every dollar." Goaldy has tags. Users pick how to use them.

**Tag-first.** Tags are the universal taxonomy. Everything maps to tags: accounts, transactions, budgets, reports. No separate "category" concept.

**Buxfer feature baseline.** If Buxfer does it and it works, keep it:

- Tag hierarchy (parent/child)
- Budget tracking by tag + period
- Split transactions (tag + amount pairs)
- Transfer detection and deletion
- CSV import
- Multi-currency + exchange rates
- Account snapshots (for investments, property)

**Monarch UX baseline.** Clean. Modern. Professional. No cutesy icons or artificial gamification.

**Single-user only.** No household mode. No shared ledger. One login, one data set. This is not a feature gap—it's a design choice that keeps code simple.

**Metrics are true to their data.** The horizon selects *which data is in scope*. It does not define the metric. Any figure normalised over time — an average, a rate, a per-month
figure — divides by the **real elapsed span of the data actually present**, never by the nominal length of the selected window.

A tag whose first transaction is three months old reads the same monthly average on an ALL horizon as on a "last 12 months" one, because in both cases there are three months of data to average. Dividing by twelve would understate it fourfold and, worse, would make *widening* the horizon *shrink* the average — the opposite of what the label promises.

This applies to every KPI or metric card in the app, so the same number never means two things in two places:

| Metric | Divisor |
|---|---|
| Tag view — Average Income / Average Expense | months from first activity in the horizon to the horizon end |
| Plan — average monthly income / expenses | months of real data in the trailing window, not a fixed 12 |
| Plan — emergency fund `avgMonthlyExpenses` | months of real data, not a fixed 12 |

Two rules follow from it:

- **Name the divisor.** A card that averages says what it divided by ("per month · over 3 months"). A denominator the user cannot see is a denominator they will assume.
- **Refuse rather than guess.** When the span cannot be measured, show no average at all. A blank is honest; a number derived from a fallback divisor is not, because nothing on screen distinguishes it from a real one.

The second rule is written from experience: an unbounded horizon once produced an infinite span, the divisor silently fell back to `1`, and a tag with three years of history at ₪2,000/month rendered "Average Expense ₪72,000 per month" beside two entirely correct totals. Every failure in this area has been a plausible-looking wrong number rather than a crash, which is why suppression is the required behaviour and why these divisors carry unit tests that assert the *number*, not merely that a card appeared.

**Visual design language.** Goaldy already reads as a dense instrument for someone who reads their own numbers closely: cool near-white ground rather than a warm cream, a single chromatic anchor, colour reserved for meaning, small type with tabular numerals, tables built for reading many rows at once. That identity is worth keeping — the rules below make it explicit and checkable, not a new direction. Write UI against these rules; do not invent a colour, size, radius or elevation outside them.

*Colour has five roles. Money direction owns green and red — nothing else may.*

| Role | Token(s) | Value(s) | Rule |
|---|---|---|---|
| Ground | `bg` / `surface` | `#F4F7F6` / `#FFFFFF` | Cool. Never warm — no cream, no off-white with a yellow cast. |
| Interactive | `indigo` | `#283593` | The one chromatic anchor. Everything that is clickable, selected, or focused and needs to read as "part of the app acting" uses it; nothing else does. |
| Money direction | `green` / `red` | `#2E7D32` / `#C62828` | **Reserved for the direction money moved — income vs. expense — and nothing else.** Never a status colour (no green "success" toast, no red "error" badge), never decoration. The one sanctioned exception: a destructive confirm button (`ConfirmDialog`'s `destructive` prop, e.g. "Delete account") is red, because "this destroys something" is the other thing red means in this app. |
| Text | `t1` / `t2` / `t3` | `#212121` / `#4A5568` / `#9AA09F` | Hierarchy — primary, secondary, muted/label. Pick by importance, not by taste. |
| Structure | `border` / `border2` | `#E8ECEF` / `#D0D5DD` | Dividers and outlines. `border2` only for a stronger internal separation (e.g. a heavier rule inside a table); `border` everywhere else. |

If you are about to reach for `green` or `red` for anything other than "this is money and this is which way it moved," stop — that is the single rule this section exists to protect, and the destructive-confirm exception above is not a precedent for a second one.

*Type is six named steps, chosen by role, not by an eyeballed pixel value.*

| Step | Size | Role | Where it's used today |
|---|---|---|---|
| `micro` | 10px | Table meta, badges, chart labels | 60 sites |
| `label` | 11px | Column headers, controls, toggles | 109 sites |
| `body` | 12px | Table data — the app's default reading size | 140 sites |
| `bodyLg` | 13px | Card titles, primary reading text | 121 sites |
| `title` | 15px | Section headings | 12 sites |
| `display` | 18px | Page titles | 3 sites |

A named step sets font-size *and* line-height together. Pick the step that matches what the text is for; never introduce a pixel size outside these six.

One deliberate, permanent exception: the four smallest steps (`micro`/`label`/`body`/`bodyLg`, 10–13px) still coexist with their un-migrated Tailwind arbitrary-value form (`text-[12px]` etc.) at roughly 430 call sites, because the arbitrary form sets font-size only — it never carried a line-height, so swapping it for the named step is a real visual change (a reflow) at every one of those sites, and nothing in this codebase can verify a visual change. Those four sizes are frozen, not open: no *new* arbitrary size may appear beside them, and the seven sizes that used to exist outside the six-step scale (8/9/14/16/17/20/22px) are gone and may not return. Collapsing the remaining 430 sites onto the named steps is its own future project, gated on deciding the line-height per site — it is not a cleanup to do opportunistically, and a reader should not "finish the job" without that per-site review.

*Three radii.* `md` (8px) is the default, used almost everywhere. `lg` (12px) is for cards. `full` is for pills — tags, badges, anything pill- or circle-shaped. Nothing else; a fourth radius is a design decision that needs the same rigour as a new colour.

*Three elevation levels, chosen by what the element is, not how prominent you want it to look.*

| Level | Looks like | Used for |
|---|---|---|
| Flat | A border, no shadow | The app's default — tables, panels, most surfaces |
| Raised | A border plus a soft shadow | Cards |
| Overlay | A border plus a strong shadow | Modals and popovers — anything that floats above the page |

*Named interaction states — decided once, not per component.* Every interactive element implements five states consistently: **focus** (a visible indigo ring), **hover** (a subtle shift in the same hue family as the element), **disabled** (reduced opacity, no hover or focus response), **error** (a red border/text on a form field — this is validation, a separate, scoped use of red, not the money-direction rule above), and **loading** (a spinner or skeleton, never a control that silently stops responding). A control that looks interactive but implements none of these five is a bug.

---

## 2. User Persona

### The Builder (You)

- Israeli tech professional
- Runs Moneyman in Docker locally
- Multiple accounts: Israeli banks + international brokerage + credit cards
- Currently uses Buxfer (frustrated) + Google Sheets (manual)
- Technical: deploys Docker, writes bash scripts, reads REST API docs
- Pain: "Why is Buxfer's UI so slow? Why is there no Hebrew support? Why do I have to manually track my pension?"

**What you want:**

- One dashboard: Israeli + international accounts
- Import from Moneyman (CSV export or API)
- Tag everything, slice reports by tag
- Budget tracking (monthly, quarterly, custom periods)
- See net worth, expense trends, where money went
- Hebrew UI + RTL support
- Run it on your NAS, keep all data local
- Never depend on a third-party SaaS again

---

## 3. Core Features (Phase 0)

### F1 — Accounts

**Liquidity-based grouping (left sidebar):**

- Liquid Assets (checking, savings)
- Investments (brokerage, stocks)
- Locked (pension, קרן השתלמות)
- Real Estate (property, estimated value)
- Loans (mortgage, car, credit)
- Credit Cards (liabilities, shown negative)
- Archived (hidden by default)

**Account types:**

- Checking / Savings / Credit Card / Loan → transaction ledger balance
- Investment / Pension / Real Estate / Manual → point-in-time snapshots
  - User enters: date, value, optional note
  - System shows balance + value history chart

**Multi-currency:**

- Each account has base currency (ILS, USD, EUR, etc.)
- Net Worth converts all to display currency (ILS default) using daily rates
- Exchange rates cached in SQLite, refreshed daily

**Account management:**

- Create, edit, archive
- Reorder (drag-and-drop)
- Rename, pick institution, pick icon

### F2 — Transactions

**Core fields:**

- Date
- Account
- Type (expense | income | transfer)
- Amount
- Description
- Merchant (optional, for rules matching)
- Status (cleared | pending)
- Tag (single) OR Splits (multiple tag + amount pairs)
- Labels (optional, multiple, non-financial)
- Notes

**TYPE vs TAG are orthogonal:**

- TYPE controls financial treatment (expense reduces budget, income increases cashflow, transfer skipped in reports)
- TAG is the user's categorization (Food, Rent, Healthcare, etc.)
- Every type can carry any tag
- Example: Transfer between Checking → Savings tagged as "Emergency Fund" (tells you what the transfer was for)

**Transaction actions:**

- Edit
- Mark as transfer (triggers peer detection)
- Add/edit splits
- Add labels
- Delete (with confirmation)

**Splits:**

- Single-tag transactions: one tag, 100% of amount
- Split transactions: multiple tag+amount pairs, must sum to 100%
- Cannot split a transfer transaction (TYPE = transfer is atomic)
- UI: one-click "Split this" opens split editor

**Transfer detection:**
User marks one leg as TYPE = TRANSFER → system searches opposite account for peer:

- Same amount (±0.5% tolerance)
- Within ±2 days
- If found: shows match, user confirms delete
- If not found: saves as unmatched transfer (no peer to delete)

**Labels:**

- Multiple per transaction, non-financial (do not affect budgets/reports)
- Example: #lunch, #reimbursable, #work
- Useful for filtering/slicing but don't change number meaning
- Suggested vocabulary: work, reimbursable, lunch-and-learns, recurring, etc.

### F3 — Tagging

**Tag hierarchy:**

- Parent → children (unlimited depth)
- Example: Food → Coffee, Groceries, Restaurants
- Siblings are ordered alphabetically. There is no manual ordering — dropped as unnecessary, and `tags.sort_order` was removed with it.

**Budget configuration (on tags):**

- Budget amount (optional)
- Budget type (expense or income)
- Budget period (weekly, biweekly, monthly, quarterly, custom)
- Period dates (start/end)
- Rollover (carry unused budget to next period?)
- Account scope (restrict to one account, or all?)

**Tag detail view:**

- Show all transactions tagged (with splits showing partial amount)
- Show budget progress (amount spent / budget)
- Show trend (spending over last 3, 6, 12 months)
- Show burn rate (if trending, when will budget run out?)

#### Time horizon on tag views

The selected horizon is app-wide and persists across views, with one
exception: **navigation resets, drill-down preserves.**

- Arriving at a tag from the **tags table**, or from the **sidebar** when not
  already on a tag detail page, pins the horizon to the **last 12 months**.
  The app-wide default (this month) usually renders an empty tag, which reads
  as "this tag has no data" rather than "no data this month".
- Arriving via a **drill-down** — a dashboard chart row or a transaction's tag
  chip — or a **lateral move** — a breadcrumb ancestor, or the sidebar while
  already on a tag — preserves the current horizon, so the context the user
  came from carries into the detail view.
- Choosing a horizon while on a tag view updates the app-wide value, so the
  choice survives subsequent tag-to-tag navigation.

**Tag management:**

- Create, rename, reparent, delete
- Create tags on the fly from a typed path (`Food / Groceries / Tea` mints the missing segments)
- Deleting a tag with transactions or children forces a choice: untag them, or move them to another tag
- An unused tag (no transactions, no children) deletes with no confirmation
- Color + icon per tag (for UI)

### F4 — Budgets

**Display:**

- Dashboard: all tags with budgets, progress bars, over/under status
- Table view: sort by status (over budget, on track, way under)
- Detailed view per tag: spending by week/month, trend, remaining days in period

**Budget math:**

- For monthly budget: (amount spent / budget) × 100
- If period is custom (e.g., "next 3 months starting Jul 1"), calculate prorated
- Show projected end-of-period (if spending continues, will you overspend?)

**Non-budgeted tags:**

- Transactions without budgets still tracked
- Dashboard shows them in "Unbudgeted" section
- Can convert to budgeted by editing tag

### F5 — Rules & Auto-Tagging

**Rules (Phase 0 basic):**

- Pattern: merchant name contains "X" → assign tag Y
- Regex support for advanced users
- Amount range (if amount > 500, tag as "big purchase")
- Account filter (only apply to one account)
- Rule priority (execute in order, first match wins)
- Enable/disable per rule

**UI:**

- Settings → Rules
- Add rule: pick pattern type, enter pattern, pick tag, set priority
- Test rule: preview against last 10 transactions
- Delete rule

**Fire on:** import, manual entry, editing

### F6 — Dashboards

**Expense Dashboard:**

- Pie chart: top tags by amount (last 30 days)
- Sparkline per tag: spending trend (last 90 days)
- Total monthly expense (current + last 3 months for trend)
- Top merchants (by frequency, by amount)

**Income Dashboard:**

- Total income (current month + trend)
- Income by source (tag breakdown)
- Income vs expense (waterfall or simple comparison)

**Net Worth Dashboard:**

- Current net worth (assets - liabilities)
- Trend (last 12 months, calculated from account snapshots)
- Breakdown by account type (liquid, investments, real estate, loans)
- Drill down to account detail

**All dashboards:**

- Date range picker (last 30 days, 90 days, YTD, custom)
- Filterable by tag, account, label
- Export as CSV or PNG

### F7 — Import

**Supported formats (all shipped):**

| Format | Notes |
|---|---|
| CSV | Israeli banks and generic, via interactive column mapping |
| XLSX | First *visible* sheet, flattened through the same column mapping as CSV |
| QIF | Tagged-line format; bracketed `L[Account]` transfers synthesise the second leg |
| JSON | An array of objects; a column may name its own `sourceKey` when the document's field names differ from the roles (`txn_date`, `amt`, …) |
| Buxfer | Three-file export (accounts + tags + transactions), plus an optional `budgets.csv` |

**Not built:** OFX. It was listed here through Phase 0 planning but no adapter exists;
adding one is a self-contained follow-up (a new module under `lib/server/importers/`
registered in `IMPORT_ADAPTERS` — the wizard needs no change).

**One wizard, two entry modes.** The same modal serves both, differing only in whether it
opens on the platform picker:

- **Statement mode** — the target account is locked; no platform picker. **Built but not yet
  reachable from the UI:** the entry points that open it (an account view, and the transaction
  table's upload action) are Stage 4 work and are not wired yet. Until they are, every user
  path lands in migration mode.
- **Migration mode** — from Settings → Data → "Import Financial Data", or the Add menu.
  Starts on the format picker.

**Import flow:**

1. Pick file → auto-detect format from content, not extension (a bank that names its Excel
   export `.csv` is common enough that trusting the extension misroutes it). Detection
   returns "unsure" rather than guessing, and the user picks.
2. If the format needs mapping: interactive column mapping.
3. If a `budgets.csv` is attached: budget settings step (period, currency, start date).
4. Preview up to 20 resolved rows, with duplicates marked, plus counts of accounts and tags
   to be created and budgets to be set.
5. User confirms → written to SQLite.

**Dedup — three modes:**

- **Skip duplicates** (default) — a row matching an existing transaction is left out.
- **Import everything** — insert every row regardless. Useful for re-importing a corrected
  file.
- **Let me review** — duplicates shown in the preview get a checkbox; **only the rows the
  user ticks are re-admitted**, identified by hash. Because the preview shows a sample
  rather than the whole file, it states how many duplicates lie beyond it, so ticking a few
  boxes never implies having reviewed them all.

**Dedup key:** `sha256(account_id | date | amount | description | externalId?)`, stored on
the row as `import_hash`. Re-importing the same file is a no-op. A row deliberately
re-admitted is stored with a null hash, since the column is UNIQUE — the original still
dedups against future imports.

**Budget import.** A Buxfer `budgets.csv` sets budgets by tag path. Budgets are leaf-only:
a parent-tag row (which real Buxfer exports contain) is warned and skipped rather than
failing the import.

### F8 — Settings

**General:**

- App name / logo (for white-label if self-hosting multiple instances)
- Display currency (ILS default, affects all conversions + reporting)
- Date format (DD/MM/YYYY vs MM/DD/YYYY)
- Language (English, Hebrew)

**Data:**

- Import financial data (opens the import wizard in migration mode — see F7)
- Export database (download SQLite file for backup)
- Export transactions (CSV of all transactions)
- **Delete all financial data** — wipes the 12 financial tables (transactions, accounts,
  tags, budgets, rules, plan items, notifications and their join tables) while preserving
  `settings`, `sessions`, `plan_settings` and `meta`, so the install stays logged in and
  configured. Requires an explicit typed confirmation.

**Rules:**

- Add, edit, delete, reorder, test

**Integrations:**

- API bearer token (show current token, rotate button, copy button)
- Short explainer: "Any tool can post transactions to `POST /api/transactions` with this token. See API docs."
- API documentation link (self-hosted Swagger UI / OpenAPI)
- Link to example adapter scripts in `/scripts` (e.g. Moneyman → Goaldy)

---

## 4. Non-Goals (Explicit Exclusions)

| Feature                       | Why Not                                 |
| ----------------------------- | --------------------------------------- |
| Multi-user / household mode   | Single-user only. Zero complexity.      |
| Plaid / bank integrations     | User brings data via Moneyman or CSV    |
| Transaction execution         | Read-only financial intelligence        |
| Investment portfolio tracking | Account-level balance only, no holdings |
| Mobile app                    | Web responsive only                     |
| Comparison / benchmarking     | No social features                      |
| Email notifications           | Phase 1+ if needed                      |
| Multi-language beyond Hebrew  | English + Hebrew only                   |
| Advanced forecasting          | Phase 2+                                |

---

## 5. Integrations (Generic REST — No Special Cases)

**There is no Moneyman-specific endpoint, and no special ingest surface.** Every integration — Moneyman, a bash script, a Zapier webhook, a spreadsheet macro, your own future tools — uses the exact same REST API the frontend uses, authenticated with a bearer token. One door, one contract, one set of docs.

**Why this is the design:** A dedicated ingest endpoint would be a second code path duplicating what `POST /api/transactions` already does, with no added capability. The generic endpoint is the integration primitive. If it's good enough for the UI, it's good enough for Moneyman.

### 5.1 Auth

All API calls (except `POST /session`) require a bearer token:

```
Authorization: Bearer <API-TOKEN>
```

The token is generated at first run (or set via env var) and shown in Settings → Integrations. It grants the same full access the logged-in user has — this is a single-user app, so there is one identity and one token scope. Rotating the token is a Settings action.

### 5.2 The two properties that make generic ingest work

Because scrapers post batches and re-post overlapping date ranges on every run, the transaction endpoint must guarantee:

1. **Batch create.** `POST /api/transactions` accepts either a single object or an array. A scraper posts one array per run, not N requests.
2. **Idempotent dedup.** The server computes `import_hash = SHA-256(date + amount + description + account_id)` for each incoming transaction and silently skips any hash that already exists. Re-posting the same range creates zero duplicates. The response reports how many were created vs skipped.

These two properties live on the generic endpoint, so no integration ever needs anything bespoke.

### 5.3 Account mapping is the caller's job

Incoming data identifies accounts however the source does (e.g. Moneyman uses `companyId` + `account`). Mapping those to a Goaldy `account_id` is a **caller-side convention**, not server code. Two supported patterns:

- The caller looks up the Goaldy `account_id` (via `GET /api/accounts`) and posts transactions already carrying the correct `account_id`.
- A thin adapter script (maintained by the user, outside Goaldy) translates source fields to the Goaldy transaction shape before posting.

Goaldy stays clean: it accepts well-formed transactions and does not embed knowledge of any specific scraper's payload.

### 5.4 Example: Moneyman via a tiny adapter

Moneyman's `webPost` payload isn't the Goaldy shape, so the user points Moneyman at a ~20-line adapter (their own, or a community recipe) that maps fields and forwards to `POST /api/transactions` with the bearer token. Goaldy itself needs no Moneyman code. This adapter pattern is documented in the README, not baked into the app.

> Design note: if a specific adapter becomes universally wanted, it ships as an optional example script in `/scripts`, never as a core endpoint.

---

## 6. REST API Contract

**Every resource exposed, every verb, no locked features.** The backend serves a complete OpenAPI 3.1 spec and Swagger UI; the frontend consumes generated types from it. Verb convention across all resources: `GET` (read), `POST` (create), `PATCH` (partial update), `DELETE` (remove). No `PUT`.

### Interactive docs

```
GET /api/docs             # Swagger UI (browsable, "try it out")
GET /api/openapi.json     # OpenAPI 3.1 document (machine-readable, source of client types)
```

### Accounts

```
GET    /api/accounts                 # List (filters: archived, type)
POST   /api/accounts                 # Create
GET    /api/accounts/:id             # Get one
PATCH  /api/accounts/:id             # Partial update
DELETE /api/accounts/:id             # Archive
GET    /api/accounts/:id/snapshots   # List snapshots
POST   /api/accounts/:id/snapshots   # Add snapshot
```

### Transactions

```
GET    /api/transactions                      # List (filters: tag, account, date_from/to, type; paginated)
POST   /api/transactions                      # Create — ONE object OR an array (batch, idempotent)
GET    /api/transactions/:id                  # Get one
PATCH  /api/transactions/:id                  # Partial update
DELETE /api/transactions/:id                  # Delete
POST   /api/transactions/:id/transfers/find   # Find transfer peer
```

**`POST /api/transactions` is the universal ingest primitive:**

- Accepts a single object or an array.
- Computes `import_hash` per transaction; silently skips existing hashes (idempotent).
- Runs the rules engine on created transactions (auto-tag).
- Response: `{ "created": N, "skipped": M, "errors": [ { index, message } ] }`
- Moneyman, scripts, and webhooks all post here. There is no separate ingest endpoint.

### Tags

```
GET    /api/tags                     # Tree structure
POST   /api/tags                     # Create
GET    /api/tags/:id                 # Get one
PATCH  /api/tags/:id                 # Partial update
DELETE /api/tags/:id                 # Delete (blocked if transactions exist; reassign first)
GET    /api/tags/:id/transactions    # Transactions with this tag
GET    /api/tags/:id/budget-status   # Budget progress
```

### Budgets (derived, read-only)

```
GET    /api/budgets              # All tags with budgets + progress
GET    /api/budgets/summary      # Month-to-date summary
GET    /api/budgets/:tag_id      # Budget + transactions for one tag
```

### Rules

```
GET    /api/rules                # List
POST   /api/rules                # Create
GET    /api/rules/:id            # Get one
PATCH  /api/rules/:id            # Partial update
DELETE /api/rules/:id            # Delete
POST   /api/rules/test           # Test rule against recent transactions
```

### Goals

```
GET    /api/goals                # List
POST   /api/goals                # Create
GET    /api/goals/:id            # Get one
PATCH  /api/goals/:id            # Partial update
DELETE /api/goals/:id            # Delete
```

### Dashboards (derived, read-only)

```
GET    /api/dashboards/expense          # Expense breakdown (pie + trend)
GET    /api/dashboards/income           # Income breakdown
GET    /api/dashboards/net-worth        # Net worth trend
GET    /api/dashboards/merchants        # Top merchants
```

### Settings & Preferences

```
GET    /api/settings             # All settings + preferences (currency, language, date format)
PATCH  /api/settings             # Update one or more settings/preferences
```

> Settings and preferences are rows in the same SQLite file as financial data — read and written through this resource, not a config file or env var.

### System & Auth

```
GET    /health                   # Server status (no auth)
POST   /api/session              # Login (password) — issues httpOnly session cookie
DELETE /api/session              # Logout
GET    /api/token                # Show current API bearer token (Settings → Integrations)
POST   /api/token/rotate         # Rotate the API bearer token
```

**Auth model:** Frontend uses an httpOnly session cookie (`POST /api/session`). Machine integrations use a bearer token (`Authorization: Bearer <token>`). Both grant identical full access (single-user app). Every `/api/*` route except `/health`, `/api/session`, `/api/docs`, and `/api/openapi.json` accepts either.

**Conventions:** All responses JSON. List endpoints paginated (`limit`, `offset`) and return `{ data, total, limit, offset }`. Errors return `{ error, details? }` with correct status (400/401/404/409). There is no `/api/ingest`.

---

## 7. Docker Deployment

**docker-compose.yml:**

```yaml
version: "3.9"
services:
  app:
    image: goaldy:latest
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: /data/goaldy.db
      SESSION_SECRET: ${SESSION_SECRET}
      API_KEY: ${API_KEY}
    volumes:
      - goaldy-data:/data
    restart: unless-stopped

volumes:
  goaldy-data:
    driver: local
```

**User runs:**

```bash
docker-compose up -d
# App available at http://localhost:3000
# Database persisted to Docker volume (survives restart)
```

**Backup:**

```bash
docker cp goaldy:/data/goaldy.db ./backup.db
```

---

## 8. Phase 0 Scope & Timeline

**What ships:**

- ✅ Accounts (all types)
- ✅ Transactions (create, edit, delete, transfer detection)
- ✅ Tags (hierarchy, budget config)
- ✅ Import (CSV, XLSX, QIF, JSON, Buxfer — **not** OFX; see F7)
- ✅ Rules (merchant pattern → tag)
- ✅ Dashboards (expense, income, net worth)
- ✅ REST API (all resources, all verbs) with bearer-token auth
- ✅ Batch + idempotent `POST /api/transactions` (the universal ingest primitive)
- ✅ OpenAPI 3.1 spec + Swagger UI at `/api/docs`; frontend types generated from it
- ✅ Settings + preferences stored in the same SQLite file (via `/api/settings`)
- ✅ Session auth (password login) + API bearer token
- ✅ Hebrew + RTL
- ✅ Docker deployment

**What doesn't:**

- ❌ Splits (defer if timeline tight, but low-hanging fruit)
- ❌ Labels (defer, non-critical)
- ❌ Advanced rules (regex, amount range, account scope — basic substring only)
- ❌ Notifications (Phase 1)
- ❌ Import history (Phase 1)
- ❌ API key management UI (Phase 1, hardcode API key for now)

**Timeline:**

- Week 1: Backend setup, database schema, accounts + transactions CRUD, auth
- Week 2: Import pipeline, Moneyman ingest, rules engine
- Week 3: Frontend (React, dashboards), CSS/RTL
- Week 4: Polish, deploy Docker, testing
- **Total: 4 weeks to Phase 0 ship**

---

## 9. Success Metrics (You)

1. **Can log in** → Yes
2. **Can import Buxfer export** → All data appears
3. **Can see accounts + balances** → Correct totals
4. **Can tag transactions** → Tags apply, budgets calculate
5. **Can see dashboards** → Numbers match manual math
6. **Can delete Buxfer account** → Never look back

---

_Created: 2026-06-07. Self-hosted, single-user, stupid-simple model. No complexity. No excuses._
