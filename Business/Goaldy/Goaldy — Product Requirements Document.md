| Field            | Value                  |
| ---------------- | ---------------------- |
| **Created**      | 2026-04-05             |
| **Last Updated** | 2026-08-29 v2.6        |
| **Version**      | 2.6                    |
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
|2.2|2026-08-26|F7 (Budgets) shipped — corrected from "data model exists, UI does not" to Shipped: per-leaf-tag budget generations, in-place editing of the open generation, closed-form rollover, automatic budget-move offers when a budgeted tag gains a child or is reparented under one, trailing-average hint, and the `/budgets` page (list/grid, summary strip, per-tag history, not-budgeted section). F4.1: removed the stale description of budget fields as columns on the tag row — budgeting now lives in its own per-tag generation history, not tag columns. F6.3: corrected the dashboard income/expense classification description — it was never actually driven by any tag-level budget field; fixed to state it comes solely from the transaction's own `type`. F14.2: added budgets to the feature-highlights list (the landing page itself is not yet built; this is the source-of-truth update for whenever it is, per this repo's CLAUDE.md convention).|
|2.3|2026-08-26|F7.7 added: direct edit/delete of any generation (open or closed) from one merged per-tag history view — a deliberate revision away from "closed history is immutable," matching Buxfer's direct budget-history management; deleting the open generation reopens the previous closed one where one exists. F7.5: the `/budgets` page gained a page-level "+ Set a budget" action and the app's standard date-horizon selector (replacing a bare date input), matching the navigation chrome convention other views (e.g. the Dashboard) already use; the global "Add" menu's long-standing disabled "Budget" placeholder is now live, opening the same per-tag view via a leaf-tag picker. F7.4: the trailing-average hint no longer has a dedicated UI slot (data still computed server-side) now that every generation is independently editable rather than only the open one — noted as an open question for a future revisit, not a removed capability.|
|2.4|2026-08-27|**F17 (Mobile Experience) added — corrected from no-mobile-support to Shipped.** A responsive mobile web shell below 768px: hamburger-drawer navigation with Settings in a top-bar overflow menu, a universal floating add-menu, every modal (and the drawer itself) rendering as a full-screen sheet, a mobile-specific compact time-horizon picker, and a new Accounts list screen grouped by liquidity class. Explicitly not covered yet: per-view mobile layouts beyond the shell, RTL support on mobile, and PWA installability (all named as deferred follow-on work in F17.5). §4 Current State Summary: "Budgets screen" removed from the modeled-but-not-surfaced list (stale since F7 shipped in v2.2 — corrected in passing) and the mobile shell added to Shipped.|
|2.5|2026-08-29|**F18 (Goaldy.AI) added — the first documented monetization plan for the product.** Product/marketing strategy work concluded that Goaldy stays free and self-hosted forever, with a separate opt-in paid layer (Goaldy.AI) sold as a subscription: an in-app assistant scoped to one instance's own data (categorization assist, insight-to-action recommendations against the Plan simulator, range-based risk simulation, proactive goal-drift nudges) plus a higher, MCP-enabled tier that exposes an instance's harmonized accounts/tags/budgets/Plan to external agents (Claude Desktop, Claude Code, etc.) — positioned as a system-of-record layer for users who already run per-institution agents/MCPs but lack any cross-account, goal-aware memory between sessions. Tax optimization was evaluated and explicitly excluded as a separate, higher-liability product. §3 Non-Goals: the blanket "no Stripe, no paid tiers" line is corrected — monetization is now planned, gated through a small externally-run licensing service (Stripe-backed, signed offline-verifiable entitlement tokens) kept structurally separate from the single-tenant self-hosted app, never touching a user's financial data. §4 Current State Summary: "monetization" removed from the never-built/no-roadmap bucket.|
|2.6|2026-08-29|**Free-tier roadmap items added across F2, F4, F6, F10, F13, F15, F17**, carrying the same product/marketing strategy session's conclusions for what ships ahead of (and independent of) Goaldy.AI: F2.1/F2.4 — OFX/QIF import, a SimpleFIN bridge, and automated transfer peer-matching all marked Planned (the ingestion items are also named prerequisites for F18.1's "harmonizes every account" MCP pitch to hold up). F4.4 — a localized starter tag tree and a config-only rule/tag-tree template exchange (cross-referenced to F18.6) planned against the empty-tag-tree problem. F6.4 (new) — a recurring/upcoming-transaction preview, planned as an extension of the existing rules engine. F10 and F15.2 — both stubs explicitly flagged as launch blockers to close before any public GTM push, not just open gaps. F13.2 — a planned feedback/issue-reporting channel, needed because the app is closed-source and therefore carries no GitHub Issues tab by default. F13.3 — a planned one-click encrypted scheduled backup, replacing the manual `docker compose cp` step. F17 — PWA installability reclassified from "not currently planned" to Planned.|

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
|OFX/QIF in-app upload|**Planned** — the next stage of the in-app importer, no design doc yet. Widens the importable-bank surface beyond CSV/Buxfer, and is treated as a prerequisite for the Goaldy.AI MCP tier's "harmonizes every account" claim (F18.1), not just an adoption nice-to-have.|
|SimpleFIN bridge|**Planned** — an optional bridge to an existing bank aggregator (SimpleFIN), not Goaldy holding bank credentials itself (§3 Non-Goals still rules out Goaldy proxying broker/bank credentials directly). Lowers the switching-cost objection for someone currently on Monarch/YNAB.|
|Other bank-specific parsers|Not built, not currently planned|

There is no encrypted-blob relay, no server-side queue with a TTL, no in-browser SQLite receiving decrypted payloads. Everything lands directly in the one server-owned `goaldy.db` via the same idempotent insert path (`ingestTransactions()`).

### F2.2 Dedup

Idempotent re-import: `import_hash = SHA-256(account_id | date | amount | description | external_id?)`, enforced as a UNIQUE constraint with `ON CONFLICT DO NOTHING`. Re-posting an overlapping range creates zero duplicates.

### F2.3 Ingest API

`POST /api/transactions` accepts a JSON array of transactions, authenticated via `Authorization: Bearer <API token>`. Response: `{ created, skipped, errors }`. This is the integration surface for scripts, webhooks, and the Moneyman scraper.

### F2.4 Transfers

A transaction's `type` can be set to `transfer`, with `transfer_source_account_id`/`transfer_dest_account_id` populated. Transfers are excluded from all income/expense reporting but included in per-account balance history and (optionally) account-level in/out charts. Transfer detection today is a matter of the ingesting integration or the user setting `type='transfer'` directly — there is no separate automated peer-matching/confirm-and-delete review queue built yet.

**Planned:** automated transfer detection — a peer-matching pass (candidate pairs by amount/date proximity across accounts) surfaced as a confirm-and-delete review queue, matching the automatic behavior every listed competitor (Buxfer, Monarch, YNAB) already ships. Today's manual step is real daily friction for anyone with 4+ accounts.

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

Arbitrary-depth tree via adjacency list (`parent_id`); a tag is the sole source of truth for categorization — there is no separate "category" concept. Budgeting (F7) is no longer stored as columns on the tag row — as of the Tag Budgets feature, a leaf tag's budget history lives in its own table, and deleting a tag cascades to delete that history (F4.3). **No `is_income` column exists or has ever existed in the shipped schema** — this document previously (v1.1) said it was removed and then (v1.2 predecessor, "old F6.2") contradicted that by describing a dashboard driven by it; that contradiction is resolved here in favor of the schema, which has no such column.

### F4.2 Tagging Constraint

Single tag XOR split — see F3.3. Tag name uniqueness is enforced only among siblings (case-insensitive); the same name is valid under two different parents.

### F4.3 Tag Management

Tree view, rename, recolor, delete (cascade-safe — descendant transactions are reassigned or untagged, and rules whose `set_tag` action pointed at the deleted tag are repointed or flagged stale).

### F4.4 No Default Tag Stack Yet

There is no out-of-the-box bilingual starter tag tree seeded on first boot — a new install starts with an empty tag tree (the public demo's tag tree, by contrast, is a hand-authored fixture for demo purposes only, not a default new-install experience).

**Planned:** a localized (EN/HE) starter tag tree, fully editable, seeded on first boot — an empty canvas is a bigger day-1 drop-off risk than a bad default. Also planned, on the same "don't start from nothing" theme: a shareable rule-set/tag-tree template exchange (config only, never transaction data — see F18.6) so a new install can adopt a community starter pack instead of only the built-in default.

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

The old F6.2 described income/expense classification as driven by an `is_income` tag flag. No such flag exists (F4.1) — income vs. expense classification for dashboard purposes comes solely from the transaction's own `type` (`income`/`expense`); no tag-level field is involved. (A budget's own expense/income type, F7, is a separate per-budget setting scoped to the Budgets page — it does not feed the dashboard's classification.)

### F6.4 Recurring/Upcoming Preview (Planned)

**Not built.** A forecasted view of upcoming recurring charges/income (matching the "upcoming" surfacing Monarch and YNAB both ship), built as a natural extension of the existing rules engine (F5.1) rather than new ingestion infrastructure — a recurrence pattern is detected from a rule/description match already flowing through the pipeline, not a new data source.

---

## F7 — Budgets

**Shipped.** Any leaf tag (one with no children) can carry a budget: an amount, a currency (snapshotted per budget, independent of the base/display currency setting), a period (weekly/biweekly/monthly/quarterly/annually/custom-N-days), an expense/income type, and an optional rollover toggle. A parent tag can never carry a budget directly — the moment a budgeted tag gains a child, its budget closes automatically rather than silently becoming a double-counted rollup over its own children's spending.

### F7.1 Generations

A tag's budget is not one row that gets overwritten on edit — it is a sequence of **generations**, each a budget configuration valid for a date range. Editing the amount effective immediately updates the current (open) generation in place; scheduling a change for a future date closes the current generation and opens a new one starting then. See F7.7 for what "editing a generation" means now that any generation — open or closed — can be revisited, not only the open one.

### F7.2 Rollover

When enabled, an occurrence's unspent (or over-spent) amount carries forward within the same budget's ongoing generation — computed as a closed form across all of that generation's occurrences to date, not a step-by-step carry that could drift. Rollover never crosses a generation boundary: closing a generation (by editing the amount, or by the tag gaining a child) also closes its rollover math: the next generation starts fresh.

### F7.3 Moving a Budget

Two situations move a budget automatically instead of just deleting it:

- **A budgeted tag gains a child.** Its budget generation closes (F7.1), and the person is offered the option to copy that closed budget's configuration onto the newly added child as its own fresh budget — since the child is now where the actual spending/income the budget was tracking will land.
- **A tag is reparented under an already-budgeted tag.** Same offer, in the other direction: the moved tag can adopt the new parent's (now-closed) budget.

If the destination tag already has its own budget, or has children of its own, there's no single unambiguous place to move the old budget to — the person sees a plain notice instead of an offer, and nothing is moved automatically.

### F7.4 Trailing Average

The backend computes an average and a trend over the budget's own trailing occurrences (e.g. 12 trailing months for a monthly budget, 3 trailing years for an annual one) — not a fixed calendar window, since a fixed window means something different for every period length. The data exists at the API level; as of the direct-edit rework (F7.7) it no longer has a dedicated slot in the UI, since it was tied to editing the open generation specifically and doesn't have a clean per-row home now that every generation is independently editable — a future revisit can decide where it resurfaces.

### F7.5 The `/budgets` Page

Expense and Income tabs, a summary strip (Budgeted / Actual / Available across the tab), a choice of two layouts — a grouped list or a card grid with a circular progress indicator — toggled client-side, no new setting, and a "Not budgeted" section listing leaf tags with activity but no budget. A page-level "+ Set a budget" action next to the page title, and a reference-date selector reusing the app's standard date-horizon control (same component the Dashboard uses) — matching this repo's convention of consistent per-view navigation chrome — replace the page's original bare date input.

### F7.6 What Deleting a Budgeted Tag Does

Deleting a tag that currently has an active budget warns explicitly before proceeding; deletion itself is unchanged (F4.3's existing cascade/reassign behavior) — the budget's history is removed along with the tag, it is not silently orphaned.

### F7.7 Direct Editing and Deletion of Any Generation

Clicking a budgeted (or not-yet-budgeted) tag's budget action opens one view listing every generation for that tag, newest-first — the single place to manage a tag's whole budget history, reachable from a table row's action, the page-level "+ Set a budget" button, or the app's global "Add" menu (which can also open it for a not-yet-chosen tag, via a leaf-tag picker).

Within that view, **every generation — open or closed — can be edited or deleted**, a deliberate revision of the original "closed history is immutable" design: amount, currency, type, period, and rollover can be changed on any entry (start and end dates are never hand-edited — they stay fixed by the generation sequence; only creating a brand-new entry sets a start date). Deleting the currently open generation reopens the tag's next most recent closed generation, if one exists, so a deletion behaves like undoing the last change rather than leaving the tag abruptly unbudgeted; deleting any other (already-closed) generation simply removes it, leaving that date range with no budget coverage — the same as if it had never existed. This trades away the original point-in-time-correctness guarantee (a past period's figures can now change if its generation is edited or removed after the fact) in favor of Buxfer-style direct management of budget history.

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

**Stub, now a launch blocker for the planned public GTM push.** The Reports nav page currently renders "Coming soon." The Dashboard (F6) is the actual reporting surface today; a dedicated Reports section (tag comparison, exportable charts, custom date-range reports) remains a real gap, not a built feature. Closing this is prioritized ahead of any public launch — a visible "Coming soon" nav item on a self-hosted-software forum (Hacker News, r/selfhosted) reads as unfinished software to exactly the launch-day audience Goaldy is targeting.

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
- **Planned:** a "Send Feedback" / "Report an Issue" link. The application code is closed-source (no public repo to carry a GitHub Issues tab), so a code-free public repo (issues/discussions only, no source) is planned as the feedback channel, linked here, from the Docker image description, and from the landing page footer — otherwise a publicly distributed Docker image with no visible way to report a bug reads as abandoned rather than private.

### F13.3 Data Management

- **Backup**: a single file copy (`docker compose cp goaldy:/data/goaldy.db ./backup.db`) — no export format to design, the real file *is* the export.
- **Clear data**: deleting/replacing `goaldy.db` wipes every table, including auth state, and logs everyone out; the next boot reseeds the password from `ADMIN_PASSWORD` as if freshly installed. A short list of UI-only preferences (locale, sidebar panel state) live outside the DB in cookies/`localStorage` and survive a clear — none of it is financial data.
- No Postgres-backed "delete account" flow exists or is needed — deleting the one file *is* deleting the account, by construction.
- **Planned:** a one-click scheduled backup feature — encrypting `goaldy.db` and copying it to a second location on a schedule, in-app, rather than requiring the operator to remember a manual `docker compose cp`. Losing the one file that *is* their financial history is a self-hoster's core fear; this turns "own your data" into "own your data, safely."

---

## F14 — Landing Page

Rewritten in full — the earlier version specified an invite-only-beta acquisition funnel ("Request Early Access," a waitlist, Google-OAuth-gated invitation links) that has no place in self-hosted software with a public read-only demo already live.

### F14.1 Purpose and Access Model

A single static marketing page (GitHub Pages), not a signup gate. There is nothing to request access to — the software is available today. The primary calls to action are **Try the live demo** and **Self-host it** (with a Docker quickstart), not a waitlist form.

### F14.2 Structure

Nav (logo, live-demo link, GitHub link) → hero (self-hosted pitch, primary CTA to the demo) → problem framing (your data on someone else's server) → feature highlights (net worth across liquidity classes, per-tag budgets with rollover, plan ahead of expenses, rules-based categorization) → "Try it" (demo link, read-only/resets-on-release note) → "Self-host it" (`docker compose up -d` snippet, link to the full README) → footer (GitHub repo, `/api/docs`).

As of this writing the landing page has no built artifact yet — the highlights list above is this document's source of truth for what a built landing page should claim, per this repo's CLAUDE.md convention that landing-page copy traces back to this section.

No pricing section, no waitlist form, no sign-in on the landing page itself — none of that applies to self-hosted software.

### F14.3 Design Language

The landing page uses the Goaldy brand palette and Plus Jakarta Sans, the same typeface as the app: a dark hero in Midnight Indigo (`#283593`) with Amber Gold (`#F9A825`) accents on CTAs and dividers, body sections on Cool Grey (`#F4F7F6`) — a premium feel that differentiates the marketing page from the app's own white-surface UI. This design-language guidance from the earlier draft still applies; only the copy and access model above changed.

---

## F15 — Authentication

Replaces the old OAuth-based F15 entirely.

### F15.1 Auth Model

No third-party identity provider. Two equal-trust paths: a browser session cookie, or a bearer API token for machine clients. `ADMIN_PASSWORD` (an environment variable) seeds the login password on first boot; if unset, the first password successfully submitted at the login screen becomes the password going forward.

### F15.2 Onboarding

**Stub, now a launch blocker for the planned public GTM push.** The onboarding route currently renders a placeholder ("5-step onboarding — coming soon"). A first-run experience — currency/locale selection, the planned starter tag tree (F4.4) as an option, an initial import prompt — remains real future work, not something already delivered; do not assume any of the old OAuth-era onboarding steps (Drive initialization, invite-token validation) apply, since none of the systems they depended on exist. A cold self-hoster's first five minutes is the entire first impression a public launch gets — this ships before any Show HN / r/selfhosted post, alongside F10.

### F15.3 Password Recovery

`GOALDY_RESET_PASSWORD`, set as an environment variable and the container restarted: force-resets the password and clears all sessions, applied exactly once per value so it's safe to leave set without repeatedly logging the operator out.

---

## F16 — Session Management

### F16.1 Session Storage

An httpOnly `session` cookie backed by a row in a `sessions` table, 30-day expiry. No JWT, no third-party token to refresh, no separate multi-device single-session-invalidation logic — a single-operator instance has no concept of "another user's device" to guard against.

### F16.2 API Tokens

A separate, longer-lived credential (sha256-hashed, compared on `Authorization: Bearer`) for integrations that shouldn't hold a browser session — this is the mechanism the Moneyman webhook and any script use.

---

## F17 — Mobile Experience

Goaldy is a **responsive web app, not a native app or a wrapped hybrid** (see §3
Non-Goals) — the self-hosted, single-Docker-container architecture has no app-store
distribution model to build against, so "works well in any mobile browser" is the
target. Full workflows are expected to genuinely work on a phone — tagging
transactions, editing rules, managing budgets — not merely be legible; this is
parity-of-capability, not graceful degradation.

### F17.1 Navigation

Below a 768px viewport width, the desktop sidebar is replaced by a hamburger drawer
listing every section in one ranked list, with no bottom tab bar and no "primary few
vs. demoted rest" split — every section gets equal footing (Dashboard, Transactions,
Tags, Budgets, Rules, Plan, Reconciliation, Reports, Accounts). Settings is
deliberately excluded from the drawer; it lives only in the mobile top bar's "⋮"
overflow menu, since it's reached far less often than any drawer item. A universal
floating action button (the same global Add-menu the desktop topbar already offers,
covering every entity type — Account, Transaction, Goal, Rule, Tag, Budget, Import
Statement) sits bottom-right on every mobile screen for quick-add from anywhere.

### F17.2 Modals and Popovers

Every modal in the app (all of them go through one shared component) renders as a
full-screen bottom sheet below 768px instead of a small centered dialog, matching how
mobile apps typically present forms and detail views. This same shared shell also
backs the navigation drawer, so both inherit the same Escape-to-close, focus-trap, and
scroll-lock behavior rather than a second, independently-built full-screen surface.

### F17.3 Time-Horizon Selection

The dashboard/transactions/budgets/accounts time-range picker gets a mobile-specific
compact rendering below 768px (a one-line `‹ 📅 THIS MONTH ▾ ›` bar with a full-screen
range-list sheet on tap) rather than the desktop's sidebar-width dropdown — the
underlying set of selectable ranges is unchanged, no new ranges were added for mobile.

### F17.4 Accounts List

A new, mobile-reachable Accounts screen (not present on desktop, which already has its
own sidebar account-tree panel serving the same purpose there) groups every account by
liquidity class, each group showing a colored subtotal, with a persistent search field
above the list.

### F17.5 What This Does Not Yet Cover

This foundation does not rebuild any individual view's own mobile layout in detail
(Dashboard's charts, Transactions' filter bar, Rules' condition-builder form, Plan's
simulator UI, Reconciliation's two-leg comparison cards) — that is deliberate,
sequenced follow-on work per view, so each has a system to build against rather than
inventing its own mobile treatment. Migrating the app's other anchored popovers
(`TagPicker`, the account-picker dropdown, various table-cell dropdowns) onto the
same full-screen-sheet pattern F17.2 establishes is likewise deferred to each
component's own future work. Full mobile support for Hebrew/RTL layout is also
deferred — the mobile shell currently renders left-to-right unconditionally, matching
the rest of the app's current LTR-only assumption, rather than partially supporting
RTL on some screens and not others.

**Not built, and not currently planned:** a native mobile app or app-store
distribution (see §3).

**Planned:** PWA installability (a home-screen icon, offline support) — no
`manifest.json` or service worker exists yet, and nothing in the mobile UI precludes
adding them; a cheap, high-visibility signal that the mobile experience (F17) isn't a
half-finished afterthought.

---

## F18 — Goaldy.AI (Roadmap, not yet built)

**Not built.** This section documents the product direction agreed for the first paid
layer on top of Goaldy, arrived at through product/marketing strategy work — it is a
roadmap commitment, not shipped software. The core app (F1–F17) stays free and
self-hosted forever; Goaldy.AI is a separate, opt-in subscription. Nothing about
*where a user's data lives* is ever paywalled — only optional AI compute and the MCP
surface (F18.3) sit behind the subscription.

### F18.1 Positioning

Goaldy.AI is not "AI categorization bolted onto a finance app." The target user already
runs agents against several per-institution MCP servers (brokerage, bank) — each
accurate but stateless and siloed to one account, with no persisted goal context
between sessions. Goaldy.AI's differentiated claim is being the **harmonized system of
record** those per-institution tools lack: one normalized net worth (across liquidity
classes, per F1/F6), one tag tree, one budget history, one household Plan (F8) — the
aggregation Goaldy already does for its own UI, made queryable by any external agent.
This is pitched as a complement to per-institution MCPs, not a replacement for them: an
agent still calls a brokerage's own MCP for a live quote or a trade; it calls Goaldy.AI
to know whether that trade fits the household's emergency-fund pillar or its Plan
horizon. This positioning is only as credible as ingestion breadth — the OFX/QIF and
SimpleFIN-bridge ingestion paths (F2.1) are treated as prerequisites for marketing this
tier hard, not independent nice-to-haves.

### F18.2 Chat Tier

An in-app assistant scoped strictly to one instance's own data. Planned capabilities:

- LLM-assisted categorization for anything the rules engine (F5.1) doesn't match — the
  schema's reserved `category_source='ai'`/`ai_confidence` fields (F5.2) are the seam
  this would finally use.
- Natural-language "ask your ledger" queries against that instance's own data.
- **Insight-to-action recommendations**: the Plan simulator (F8.6) already computes what
  a shortfall needs (required monthly savings, the first danger year) — surface it as a
  concrete move ("shift $400/mo to your emergency-fund pillar") instead of leaving the
  arithmetic to the user.
- **Range-based risk simulation**: today's Plan projects one deterministic line; a
  Monte Carlo–style band around it (sequence-of-returns risk, a probability of hitting
  a danger year rather than a single point estimate) computed locally against the
  existing projection engine, no new data source.
- **Proactive goal-drift nudges**: F9's event/condition log already runs a 24h check but
  is currently pull-only (read on demand); push a notification the moment a goal or
  budget visibly drifts, rather than waiting for the dashboard to be opened.
- AI-narrated Plan scenarios ("what if I front-load university savings by 2 years?").

**Explicitly excluded from scope:** tax optimization (loss-harvesting, contribution-limit
tracking, RSU-vesting timing). A real request from this audience, but judged a distinct,
higher-liability product — not on this roadmap.

### F18.3 Chat + MCP Tier

Everything in F18.2, plus an MCP server exposing that instance's harmonized
accounts/tags/budgets/Plan to external agents (Claude Desktop, Claude Code, and
similar). Reuses the existing OpenAPI 3.1 surface (`lib/server/openapi.ts`) as the
source the MCP tool/resource definitions are generated from, rather than a new
hand-built API. Write-capable MCP scopes (tag, categorize, move a budget) are opt-in
and separate from a default read-only scope — an external agent should never get
unscoped write access to a ledger by default.

### F18.4 Hosted Instance (optional add-on)

A managed, backed-up hosted instance for people who want the self-hosted ownership
story without running Docker themselves, bundled with a Goaldy.AI subscription.

### F18.5 Licensing and Entitlement Gating

The self-hosted app is deliberately single-tenant with no OAuth (§3) — licensing state
cannot live inside `goaldy.db`. Planned architecture:

1. **Stripe** owns billing and identity for the paid layer; no separate accounts system.
2. A small, separately-operated licensing service holds one table (license key →
   entitlements) and issues a signed, short-lived, offline-verifiable token per
   activation. It never receives a user's financial data — only entitlement state.
3. A self-hosted instance activates once via a license key entered in Settings, then
   verifies that signed token locally on every gated request (public key baked into the
   Docker image), with a grace window if the licensing service is briefly unreachable —
   no constant phone-home, which would undercut the self-hosted trust model.
4. MCP credentials themselves continue to be minted by the existing scoped bearer-token
   mechanism (F13.2/F16.2); the licensing token only gates whether that capability is
   unlocked, rather than introducing a second auth system.

### F18.6 Community/Config Sharing (low-effort, privacy-safe)

A shareable rule-set and tag-tree template exchange — config only, never transaction
data — addressing the empty-tag-tree problem (F4.4) and giving the product a community
surface without ever centralizing a user's ledger.

---

## 3. Non-Goals (Explicit Exclusions)

Confirmed current, replacing the old table:

- No AI/LLM transaction categorization (rules only, today).
- No OAuth or third-party identity provider.
- No multi-tenancy, no waitlist, no invite-only beta, no signup funnel of any kind.
- No Postgres or any database beyond the one SQLite file.
- No Israeli-specific financial-instrument account types or cards.
- No paid tier, Stripe integration, or AI feature exists **in the self-hosted app today** — see F18 for the planned Goaldy.AI subscription layer, which is roadmap, not shipped, and is architected as a separate service rather than something built into `goaldy.db`.
- No native mobile app.
- No bank-credential proxying or broker integrations — bring your own data via CSV, the Buxfer CLI, or the Moneyman webhook.
- No envelope budgeting or forced methodology.
- No transaction execution (read-only financial intelligence; no payments initiated by the app).
- No social/comparison features, no public third-party API beyond the documented ingest endpoint.

---

## 4. Current State Summary

Rather than a forward-looking phased SaaS rollout (the old §5), here is what's actually true today:

**Shipped:** accounts (all liquidity classes, multi-currency), transactions (ingest, dedup, splits, labels, duplicate detection), tags (arbitrary tree), per-tag budgets with generations/rollover/trailing-average (F7), rules-based categorization, the Cashflow/Net-Worth dashboard, the Plan household simulator, notifications (event/condition log), bilingual i18n/RTL, self-host Docker deployment, versioned CI/CD releases, a public read-only demo, and a responsive mobile web shell (F17).

**Modeled but not surfaced (data exists, UI doesn't):** Reports screen (F10), onboarding flow (F15.2) — both now flagged as launch blockers for the planned public GTM push, not just open gaps.

**Explicitly deferred, documented elsewhere:** a generic in-app data importer (`docs/features/2026-08-19-in-app-data-importer`), replacing the developer-only Buxfer CLI script.

**Never built, and not on any current roadmap:** Israeli-specific financial-instrument cards, OAuth/multi-tenancy.

**Planned, not yet built (free tier):** OFX/QIF import and a SimpleFIN bridge (F2.1), automated transfer detection (F2.4), a localized starter tag tree and config-only template exchange (F4.4), a recurring/upcoming-transaction preview (F6.4), a public feedback/issue-reporting channel (F13.2), a one-click encrypted scheduled backup (F13.3), and PWA installability (F17.5).

**Planned, not yet built (paid):** Goaldy.AI (F18) — an opt-in paid layer (in-app AI assistant, an MCP server for external agents, an optional hosted instance) gated by a separately-run licensing service. AI categorization (F5.2) is part of this plan rather than a dead schema seam.

---

_Document created: 2026-04-05. Rewritten in full 2026-08-21 (v2.0) to match the shipped self-hosted architecture. Status: Living document — update in the same change as any feature that ships or changes shape._
