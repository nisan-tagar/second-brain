| Field            | Value                  |
| ---------------- | ---------------------- |
| **Created**      | 2026-04-05             |
| **Last Updated** | 2026-04-06 v1.10       |
| **Version**      | 1.10                   |
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
|1.5|2026-04-05|F2.6 rewritten: transfer model corrected to user-initiated (not auto-detected on import); TYPE=TRANSFER with source/destination accounts triggers peer detection on save; CSV fallback is advisory only. F3.2: transfer banner scoped to CSV suggestions only. F3.3: Mark as Transfer added as explicit transaction action; Tag/Split/Transfer documented as mutually exclusive states.|
|1.6|2026-04-05|F3.3/F3.4 rewritten: full 6-type transaction model (EXPENSE/INCOME/TRANSFER/REFUND/INVESTMENT/IOU) matching Buxfer's proven taxonomy; TYPE and TAG explicitly documented as fully orthogonal — every type can carry a tag; financial treatment table added; REFUND defined as negative expense reducing tag budget; split availability per type clarified; F3.4→F3.5 renumbered; type filter added to F3.6 search.|
|1.7|2026-04-05|F3.4 simplified: type enum reduced to EXPENSE/INCOME/TRANSFER only; REFUND/INVESTMENT/IOU explicitly deferred with do-not-implement note.|
|1.8|2026-04-05|F3.4/F3.5: splits supported on all transaction types including TRANSFER; split-blocking removed from TRANSFER; financial treatment table updated; F3.5 split UI note corrected.|
|1.9|2026-04-05|F14 added: landing page. F15 added: auth + onboarding. F16 added: session management. F17 added: notification system. Non-goals and phased delivery updated.|
|1.10|2026-04-06|F2.5 rewritten: full ingestion surface documented — /api/ingest is machine-to-machine only; file imports are browser-only; two auto-detected formats (canonical Goaldy + Moneyman webPost); account mapping via moneyman_account_mappings; OpenAPI spec and Postman collection added as deliverables; Upstash deferred to Phase 2.|

---

## 1. Product Overview

### 1.1 Purpose

Goaldy is a personal financial intelligence platform for technically sophisticated users managing complex financial lives — multiple accounts, multiple currencies, investment portfolios, and real planning needs that current tools handle inadequately. The primary user is an Israeli professional with both Israeli and international financial accounts who currently duct-tapes together Moneyman, Buxfer, and manual spreadsheets to get an incomplete picture.

### 1.2 Design Principles

**No methodology religion.** Goaldy does not impose a budgeting system. It presents financial data clearly and lets the user draw their own conclusions. No envelopes, no "jobs for every dollar," no guilt mechanics.

**Tag-first architecture.** Everything is a tag. The tag tree is the universal taxonomy for accounts, transactions, budgets, goals, and reports. There is no separate "category" concept.

**The app hydrates from your data — it does not own it.** Financial data lives in the user's Google Drive. The app is a shell. This is a product principle, not just a technical decision — it should be communicated to users as a feature.

**Buxfer is the feature baseline, Monarch is the UX baseline.** Buxfer has earned its users through 15 years of financial tool discipline — the tag hierarchy, the budget table, the tag detail drill-down are all worth preserving. Monarch's visual language — clean, modern, professional — is the target aesthetic. Goaldy surpasses both.

**The primary user is User #1.** Every feature that ships must solve a real problem for the builder's own daily financial life. Hypothetical users come second.

### 1.3 Success Criteria

**Phase 0 success:** Builder stops using Buxfer and uses Goaldy daily for all financial tracking.

**Phase 2 success:** 10 Moneyman community users adopt Goaldy and report it superior to their previous tool.

**Phase 3 success:** Users report that Goaldy's goal planning feature replaced a spreadsheet or mental math they were doing manually.

---

## 2. User Personas

### Primary: The Dual-Market Professional

- Israeli tech professional, 30–50 years old
- Household income: combined ₪30,000–60,000/month
- Financial complexity: Israeli checking/savings + mortgage + pension + קרן השתלמות + US brokerage + credit cards across 2–3 banks
- Currently using: Moneyman (self-hosted Docker) + Buxfer or Google Sheets
- Pain points: no single view of Israeli + international accounts; Buxfer UI is dated; no forward planning tool; manual mental math for future expense planning
- Technical comfort: high — runs Docker, understands APIs, reads GitHub READMEs

### Secondary: The Technically Adjacent User

- Partner or friend of the primary persona, referred by word of mouth
- Less technically comfortable — won't self-host Moneyman but will use a well-designed web app
- Relies on CSV export from their bank
- Pain points: existing tools too complex or too US-centric; no Hebrew support

---

## 3. Feature Areas

---

### F1 — Accounts

#### F1.1 Account Overview

The left sidebar displays all accounts grouped by liquidity class, always visible while in the app. Net Worth total is pinned at the top of the account tree.

**Liquidity classes (groups):**

- Liquid Assets (checking, savings, cash)
- Investments (brokerage, stock plans, crypto)
- Locked Funds (pension, קרן השתלמות, long-term savings)
- Real Estate (property at estimated value)
- Loans (mortgage, car loan, credit lines)
- Credit Cards (treated as liabilities)
- Archived (closed accounts, hidden by default)

Each group is collapsible. State persists across sessions. Clicking any account navigates to its transaction view with account context header showing balance, institution, and currency.

**Net Worth total** clicking navigates to the Net Worth dashboard tab.

#### F1.2 Account Types

|Type|Sub-type|Balance source|Notes|
|---|---|---|---|
|Checking|—|Transaction ledger|Standard|
|Savings|—|Transaction ledger|Standard|
|Credit Card|—|Transaction ledger|Balance shown as negative|
|Investment|Brokerage, Stock Plan, Crypto|Manual snapshot|No individual holdings|
|Pension|Defined contribution|Manual snapshot|Locked|
|קרן השתלמות|—|Manual snapshot|Locked; vesting date tracked|
|Real Estate|Property|Manual snapshot|Illiquid|
|Loan|Mortgage, Car, Other|Transaction ledger|Shown as negative|
|Manual|Any|Manual snapshot|Catch-all for non-standard assets|

#### F1.3 Manual Snapshot Accounts

For accounts without a transaction stream, the user records point-in-time value updates. Each update stores: value, date, optional note. The system displays the most recent snapshot as the current balance and shows a value history chart on the account detail view.

Snapshot update is accessible via a prominent "Update Balance" action on any snapshot-based account.

#### F1.4 Multi-Currency

Each account has a base currency. All balances display in their native currency on the account line. The Net Worth total and all aggregations convert to the user's display currency (ILS by default) using daily exchange rates fetched from the Bank of Israel public API (for ILS pairs) and ECB (for EUR/USD pairs), cached in `goaldy.db`.

Exchange rates are shown transparently — hovering any converted amount shows the rate used and the date it was fetched.

#### F1.5 Account Management

- Create, edit, archive accounts
- Reorder accounts within groups via drag-and-drop
- Move accounts between groups (changes liquidity class)
- Link an account to a sync connection (SimpleFIN for US banks)
- View full transaction history per account
- Export account transactions as CSV

---

### F2 — Data Ingestion

#### F2.1 Ingest Sources

|Source|Who it serves|Setup|
|---|---|---|
|Moneyman webhook|Israeli bank users (technical)|API key + one Moneyman config line|
|SimpleFIN Bridge|US/Canadian bank users|Paste access token; $15/year to SimpleFIN|
|CSV file import|All users|Upload file; parser auto-detects format|
|OFX / QIF import|All users|Standard financial export format|
|Manual transaction entry|All users|Direct UI entry|

#### F2.2 File Import Parsers

Named parsers for Israeli institutions (each bank exports a different column schema): Hapoalim, Leumi, Discount, Mizrahi, Max/Leumi Card, Isracard, Visa Cal, Pepper, One Zero, Beinleumi, Otsar Hahayal.

Generic CSV parser with interactive column mapping UI for unrecognized formats. Column mapping is saved per institution — user maps once, never again.

Migration parsers: Buxfer full export (maps tags, accounts, rules), Actual Budget `.db` direct SQLite import, YNAB CSV.

#### F2.3 Import UX

1. User drops file or pastes API token
2. Parser detects format, previews first 10 rows with proposed field mapping
3. User confirms or adjusts mapping
4. System runs dedup check — shows count of new vs duplicate transactions
5. **System runs transfer detection pass** — flags likely transfer pairs for review (see F2.6)
6. Runs rules engine on non-transfer transactions — shows auto-tag preview
7. User confirms import
8. Unmatched transactions surface in triage queue; transfer candidates surface in Transfer Review queue

#### F2.4 Deduplication

Every transaction is assigned an `import_hash` computed from `SHA-256(date + amount + description + account_id)`. Re-importing the same file or overlapping date ranges produces no duplicates. The dedup preview shows exactly how many transactions were skipped and why.

#### F2.5 Ingest API

**`POST /api/ingest`** — the machine-to-machine data entry surface. Authenticated via per-user API keys (created in Settings → Integrations, shown once, stored as SHA-256 hash). Rate limiting stubbed for Phase 0; implemented at scale in Phase 2.

**This endpoint is exclusively for programmatic pushes.** File imports (CSV, OFX, Buxfer export) are processed entirely in the browser and written directly to OPFS — they never touch this endpoint.

**Supported formats (auto-detected — no format parameter needed):**

|Format|Detection signal|Who sends it|
|---|---|---|
|Canonical Goaldy|Array with `companyId` field|Custom scripts, migration tools|
|Moneyman `webPost`|Array with `"scraped at"` key (space in key name)|Moneyman automated scraper|

**Canonical Goaldy format** — the primary format, modeled on the israeli-bank-scrapers/Buxfer transaction schema:

```json
[{
  "type": "normal",
  "date": "2026-04-02T21:00:00.000Z",
  "originalAmount": -3065.14,
  "originalCurrency": "ILS",
  "chargedAmount": -3065.14,
  "description": "7578 - כרטיסי אשראי לי",
  "account": "59641",
  "companyId": "beinleumi",
  "hash": "2026-04-02T21:00:00.000Z_-3065.14_...",
  "uniqueId": "2026-04-03_beinleumi_59641_-3065.14_8547",
  "status": "completed"
}]
```

**Moneyman `webPost` format** — sent automatically when Moneyman is configured with `storage.webPost`. Adapted server-side to canonical format before encryption:

```json
[{
  "date": "02/04/2026",
  "amount": -3065.14,
  "description": "7578 - כרטיסי אשראי לי",
  "account": "59641",
  "scraped at": "2026-04-03",
  "scraped by": "beinleumi",
  "chargedCurrency": "ILS"
}]
```

Moneyman config (zero additional configuration required beyond URL and API key):

```json
{
  "storage": {
    "webPost": {
      "url": "https://app.goaldy.com/api/ingest",
      "authorizationToken": "Bearer <goaldy-api-key>"
    }
  }
}
```

**Account mapping** — both formats identify accounts by institution ID (`companyId`) and account key (`account` field), not by Goaldy UUID. The mapping from `{companyId, account}` → Goaldy account UUID is stored in Postgres (`moneyman_account_mappings`) and managed in Settings → Integrations → Moneyman Accounts. Transactions arriving with no matching mapping are queued as unmapped — never silently dropped — and surfaced in the triage banner for the user to resolve.

**Response:**

```json
{
  "queued": 12,
  "unmapped": 2,
  "format": "moneyman",
  "errors": []
}
```

**OpenAPI spec** is served at `/openapi.yaml`. **Postman collection** for testing is at `/goaldy-ingest.postman_collection.json`. Both are generated during repo bootstrap.

All server-side ingest (this endpoint, and later SimpleFIN cron) goes through the encrypted blob queue — the server stores AES-256-GCM encrypted payloads with 48h TTL, decrypted and ingested on next browser session open, then immediately deleted. No readable financial data ever persists on Goaldy's servers.

**Full ingestion surface:**

|Source|Path|Processing|
|---|---|---|
|Moneyman `webPost`|`POST /api/ingest`|Server: detect → adapt → encrypt → blob|
|Custom script|`POST /api/ingest`|Server: validate → encrypt → blob|
|CSV file upload|Browser only|Client: named parser → OPFS|
|OFX/QIF upload|Browser only|Client: standard parser → OPFS|
|Buxfer export|Browser only|Client: migration parser → OPFS|
|Manual entry|Browser only|Client: form → OPFS|
|SimpleFIN (Phase 2)|Internal cron|Server: fetch → encrypt → blob|

#### F2.6 Transfer Handling

Bank scrapers (Moneyman, israeli-bank-scrapers, SimpleFIN) import both sides of every internal transfer as separate transactions — one debit on the source account, one credit on the destination account. Both land in Goaldy's ledger. This is correct behavior. The user then marks one as a transfer, which triggers peer detection and deletion of the duplicate.

**The flow:**

```
Scraper imports both sides:
  059641-FIBI Joint    3 Apr    −₪2,507    כרטיסי אשראי ל - 8212
  8212-Nisan           3 Apr    +₪2,507    כרטיסי אשראי ל - 8212

User opens one transaction → sets TYPE = TRANSFER
  Source account:       059641-FIBI Joint
  Destination account:  8212-Nisan
  Tag (optional):       Credit Cards / Nisan

On save, detection fires:
  Searches 8212-Nisan for a transaction matching:
    amount = +₪2,507 (opposite sign)
    date within ±2 days
  Match found → "We found a matching transaction on 8212-Nisan — delete it?"

User confirms → peer hard-deleted → transfer record remains
```

**The transfer record** (the remaining transaction) is excluded from all expense/income calculations — budgets, dashboards, reports, cashflow — because its `type = 'transfer'`. The tag on a transfer transaction is informational only; it does not feed the budget engine or appear in tag reports. It remains visible in the account transaction list with arrow notation showing source → destination.

**Transfer type fields** — when TYPE is set to TRANSFER, the edit form shows:

- Source account (dropdown — the account the money came from)
- Destination account (dropdown — the account the money went to)
- Tag (optional — for user reference, no financial impact)

**Detection algorithm:**

- Searches destination account for opposite-sign transaction
- Amount match: exact or within 0.5% tolerance (handles rounding)
- Date tolerance: ±2 days (handles Israeli banking posting delays)
- If multiple candidates found: presents a list for user to select the correct peer
- If no match found: saves the transfer record as an unmatched transfer — the user is informed and can search manually

**Unmatched transfers** — occur when one account is not synced in Goaldy (e.g. an external wire to a US account not connected via SimpleFIN). The transfer record is saved with TYPE = TRANSFER, source set, destination left as "External account." Excluded from reports. No peer deletion attempted.

**Transfer Review banner** — removed from the triage flow entirely. There is no automatic detection on import. The banner in F3.2 is retained only to surface transactions that were previously auto-detected with high confidence (see below).

**CSV import fallback (best-effort only):** for CSV imports where no transfer metadata is provided by the source, the system runs a background heuristic pass after import — same matching criteria as above. High-confidence matches (same day, exact amount, known payment description pattern, different accounts) are surfaced as suggestions in the Transfer Review banner. This is advisory, never automatic. User opens the suggested pair, sets one as TYPE = TRANSFER manually, and confirms deletion. Medium and low confidence matches are silently skipped.

---

### F3 — Transaction Management

#### F3.1 Transaction List

The primary workspace for financial data. Accessible globally (all accounts) or scoped to a single account (from sidebar click) or a single tag (from tag detail view).

**Columns:** Date · Amount · Description · Merchant · Tag · Account · Status

Note: "Tag" is singular. A transaction has either one tag (covering 100% of the amount) or a split (multiple tags each covering a defined portion that sums to 100%). There is no multi-tag concept — see F3.4.

**Default sort:** Date descending. Sortable by any column.

**30-Day Forecast section** — pinned at the top of any transaction list, collapsible. Shows upcoming projected transactions based on:

- Budget repeat schedules (user-configured)
- Detected recurring patterns (system-detected from transaction history — e.g. same merchant, similar amount, regular interval, 3+ occurrences)
- Scheduled goals contributions

Forecast transactions are visually distinct (dashed border, lighter color, clock icon). Clicking a forecast row opens a detail panel showing the pattern that generated it and options to edit or dismiss.

#### F3.2 Triage and Transfer Banners

Two distinct banners appear above the transaction list when action is needed. They are visually separate and never merged into one.

**Triage banner** — untagged transactions:

```
14 transactions need tagging  ·  94% auto-tagged  ·  [Review now →]
```

The auto-tagged percentage is the primary health metric for the categorization system. A mature setup sustains >95%.

**Transfer Review banner** — CSV-imported transfer suggestions only (not shown for scraper-imported data where the user handles transfers manually via the Edit Transaction flow):

```
3 possible transfer pairs found  ·  [Review →]
```

Each suggestion shows the two candidate transactions side by side. The user opens one, sets TYPE = TRANSFER with source/destination accounts, and the peer is detected and deleted. "Not a transfer" dismisses the suggestion permanently.

#### F3.3 Transaction Actions

Available on every transaction row (expanded on click or hover):

|Action|Description|
|---|---|
|Tag|Assign or change the tag for this transaction (available on all types)|
|Split|Replace the single tag with a multi-row split covering 100% of the amount (expense, income, refund only)|
|Label|Add or remove labels — free-form dimensional markers with no financial impact|
|Set Type|Change the transaction type (see F3.4)|
|Edit|Change date, description, amount, notes|
|Copy|Duplicate with editable fields|
|Repeat|Create a scheduled recurrence from this transaction|
|Add Rule|Create a categorization rule from this transaction's fields|
|Add Memo|Attach a free-text note|
|Delete|Remove transaction (with confirmation)|

**TAG and TYPE are fully orthogonal.** Every transaction type can carry a tag. The type controls financial treatment — how the transaction is counted in calculations. The tag controls categorization — which budget line or report group it contributes to. They never interfere with each other.

#### F3.4 Transaction Types

Every transaction has a TYPE field. Auto-assigned on import based on amount sign and provider metadata. Always user-overridable.

**EXPENSE** (default for negative amounts) Counts in the expense dashboard. Tag drives which budget line is charged. Splits allowed.

**INCOME** (default for positive amounts) Counts in the income dashboard. Tag drives which income budget line is credited. Splits allowed.

**TRANSFER** Internal movement between two accounts in the user's ledger. Excluded from expense and income calculations entirely. Requires source account and destination account fields. Tag is for identification — does not affect any budget or report. Cannot be split. Triggers peer detection on save to identify and delete the duplicate counterpart transaction (see F2.6).

**Financial treatment:**

|Type|Expense dashboard|Income dashboard|Budget impact|Tag role|Splits|
|---|---|---|---|---|---|
|EXPENSE|✓|—|Charges tag budget|Full|✓|
|INCOME|—|✓|Credits tag budget|Full|✓|
|TRANSFER|—|—|None|Identification only|✓|

**Auto-assignment on import:**

- `amount < 0` → EXPENSE
- `amount > 0` → INCOME
- Provider marks as transfer → TRANSFER
- All auto-assignments are user-overridable

**TYPE and TAG are fully orthogonal.** Every type carries a tag independently. TYPE controls financial treatment. TAG controls categorization. They never interfere.

> **Deferred types:** REFUND, INVESTMENT, and IOU are explicitly out of scope for the current version. They will be considered if a concrete need arises from real usage. Do not implement them.

#### F3.5 Tagging Model — Single Tag or 100% Split

A transaction exists in one of three tag states at all times. Note: tag state is independent of transaction type — every type can be in any tag state.

**State A — Untagged** `tag_id: NULL`, no splits. Sits in the triage queue for EXPENSE and INCOME transactions. TRANSFER, INVESTMENT, and IOU types are excluded from the triage queue — they don't require a tag to function, though one can optionally be added.

**State B — Single tag** `tag_id` set to one tag. 100% of the transaction amount is attributed to that tag for the applicable budget and report calculations. This is the common case.

**State C — Split** `tag_id: NULL`. Two or more split rows exist, each with its own amount and tag. Splits must sum exactly to the parent transaction amount. **Splits are supported on all transaction types.**

**The constraint is absolute:** a transaction cannot carry both a `tag_id` and splits simultaneously.

**Split UI:**

- Triggered by the "Split" action on any transaction
- Modal showing parent transaction header (date, description, total amount, type)
- Two or more split rows: amount field + optional description override + tag selector
- Running balance bar — drains to zero; green when balanced, amber when partial, red if over
- Last row auto-fills to remaining balance when clicked
- All splits committed atomically on confirm

**Goal funding split — primary use case:**

```
₪5,000 deposit (INCOME) — FIBI Savings — 1 Apr
  ├── ₪1,500  Goals/Vacation 2026
  ├── ₪2,000  Goals/Car Replacement
  ├── ₪970    Goals/Roof Repair
  └── ₪530    Savings/Emergency Fund    ← auto-filled to balance
              ─────────────────────────
              ₪5,000 ✓  balanced
```

**The Goal Allocation Assistant** — when an INCOME transaction lands in any liquid or semi-liquid account and is untagged, the intelligence feed surfaces a prompt: "₪5,000 unlinked deposit — your active goals needed ₪4,970 this month. Allocate now?" Tapping it opens the split modal pre-populated with goal requirements in priority order.

#### F3.6 Transaction Search and Filter

Global search across all transactions: fuzzy match on description and merchant. Filters: date range, account, **type (one or more)**, tag (including subtree), amount range, status (cleared/pending), source (moneyman/simplefin/csv/manual), category source (rule/ai/user/untagged), labels (one or more).

Filter combinations are saveable as named views and accessible from a "Saved Views" dropdown in the transaction toolbar.

#### F3.7 Labels

Labels are dimensional markers on a transaction. They are completely separate from tags.

**The distinction:**

- **Tags** answer "where does this money go?" — financial classification, drives budgets, reports, goal attribution. One tag or splits summing to 100%.
- **Labels** answer "what else is true about this transaction?" — orthogonal facts that don't affect financial calculations. Multiple labels allowed, no financial impact whatsoever.

**What labels enable:**

```
Transaction: Business dinner  ₪800
  Tag:    Business/Meals        ← single tag, drives budget
  Labels: Tax/Deductible        ← marker only, filterable
          Reimbursable          ← marker only, filterable
```

Running "filter by Tax/Deductible" gives a correct total of all deductible transactions without any double-counting or budget impact. Exporting "Reimbursable" transactions produces an accurate expense claim list. The budget engine sees ₪800 against Business/Meals and nothing else — labels are invisible to it.

**Label properties:**

- Free-form string — no predefined list, no tree structure, no color, no icon
- Multiple labels per transaction — unlimited
- Created on the fly by typing — autocomplete suggests existing labels from the user's history
- Case-insensitive matching — "tax/deductible" and "Tax/Deductible" are the same label

**Where labels appear:**

- Transaction row (shown as small pills after the tag, visually distinct — outlined, not filled)
- Transaction detail panel (editable inline)
- Filter panel (select one or more labels to filter the transaction list)
- Export (labels included as a column in CSV exports)

**Where labels do not appear:**

- Budget table (no label column, no label budget)
- Dashboard charts (labels invisible to the report engine)
- Tag detail view (labels don't affect tag aggregations)
- Rules engine (rules assign tags, not labels — label assignment is always manual)
- Intelligence feed calculations (anomaly detection, cashflow projection — all tag-based)

**Common label vocabulary** — not enforced, just suggested on first use:

|Label|Use|
|---|---|
|`Tax/Deductible`|Business expense eligible for deduction|
|`Reimbursable`|Expense to be claimed back from employer|
|`Recurring`|Subscription or scheduled payment|
|`One-time`|Flag unusual non-recurring expenses|
|`Business`|Work-related on a personal account|
|`Joint`|Household shared expense on a personal account|
|`Disputed`|Transaction under review or dispute|
|`Cash`|Cash transaction manually entered|

The user's label vocabulary grows organically. Labels used once appear in autocomplete forever until the user explicitly deletes them from Settings → Labels.

---

### F4 — Tag System

#### F4.1 Tag Tree

Tags form an arbitrary-depth tree via parent-child relationships. There is no maximum depth. There is no separate "category" concept — everything is a tag.

**The tag is the source of truth for everything:** classification, reports, budgets, goals, and rules all reference the tag. There is no separate budget record — budget configuration is a property of the tag itself.

**Tag properties:**

- Name (required)
- Parent tag (optional — null means root tag)
- Color (hex — used in charts and the tag tree sidebar)
- Icon (emoji — optional, for display in lists)
- Sort order (user-controlled drag-and-drop ordering within parent)
- **Budget configuration** (optional — the full budget target, owned by the tag):
    - Budgeted amount
    - Budget type: EXPENSE or INCOME (determines which Budgets tab it appears on; inferred from dominant cash flow direction by default, overridable)
    - Repeat period: Weekly / Biweekly / Monthly / Quarterly / Annually / Custom interval
    - Start date
    - Enable rollover (boolean — unused budget carries to next period)
    - Repeat until (optional end date — for time-bounded budgets)
    - Account scope (optional — restrict budget to transactions from one account)

> **Note:** `is_income` has been removed. The income/expense classification for the Budgets screen is derived from the `budget_type` field on the tag's budget configuration, not from a flag on the tag itself. A tag with no budget configuration is classified by the system based on the dominant direction of its transactions (positive = income tendency, negative = expense tendency) when displaying in reports.

**Example structure:**

```
Food
  Groceries
  Restaurants & Cafés
  Snacks
Housing
  Rent (Or Mortgage)
  Cleaning
  HOA
  Municipal Tax
Kids
  Daycare
  Babysitter
  Activities
  Gifts
  Toys
Goals                    ← auto-generated root; see F8
  Goals/Vacation 2026
  Goals/Car Replacement
```

#### F4.2 Transaction Tagging Model

**A transaction has exactly one tag or a set of splits that cover 100% of its amount. Nothing else.**

This is the foundational constraint of the tagging system. It eliminates ambiguity in budget attribution, reports, and goal tracking. Every unit of money in the ledger is attributed to exactly one tag at any point in time.

- Single-tagged transactions: the full amount counts against that tag in the budget engine and reports
- Split transactions: each split's amount counts against its respective tag — splits are independent budget units
- Untagged transactions: excluded from all budget and report calculations until tagged

There is no primary/secondary tag concept. There are no secondary tags without amounts. The `transaction_tags` join table does not exist in the schema. Tag assignment is either a `tag_id` on the transaction row (single tag) or rows in `transaction_splits` (split case).

#### F4.3 Tag Management Screen

Accessible from the left sidebar "More → Tags" or by clicking any tag label.

Views:

- **Tree view:** hierarchical display with drag-and-drop reordering and reparenting; inline rename; color/icon picker
- **Table view:** flat list with columns for name, parent, transaction count, last used, monthly target; sortable; bulk actions

Actions: create, rename, recolor, merge into another tag (all transactions retag automatically), delete (with reassignment prompt for existing transactions), archive.

#### F4.4 OOTB Tag Starter Stack

On first launch after import, the system offers an opinionated default tag tree as a starting point. User can accept all, selectively accept, or skip.

**The starter stack is i18n — localized per the user's selected locale.** When the user chooses Hebrew on the locale selection screen, all tag names are pre-populated in Hebrew. When English, in English. The tree structure is identical across locales; only the names differ. This removes the friction of renaming a foreign-language tag tree on first use.

**Hebrew locale (עברית):**

```
הכנסות
  משכורת
  הכנסות נדל"ן
  תשואות השקעות
  פרילנס

דיור
  שכירות / משכנתא
  ניקיון
  ועד בית
  ארנונה
  תחזוקה ותיקונים

מזון
  סופרמרקט
  מסעדות ובתי קפה
  קפה

תחבורה
  דלק
  תחבורה ציבורית
  חנייה
  תחזוקת רכב

ילדים
  גן / צהרון
  חינוך
  חוגים
  ביגוד
  מתנות וצעצועים

בריאות
  רופאים
  בית מרקחת
  ביטוח בריאות

פיננסי
  מיסים
  עמלות בנק
  החזרי הלוואות
  ביטוח

אישי
  ביגוד
  בידור
  נסיעות
  מנויים
  מתנות

חסכונות
  קרן חירום
  חסכון לטווח קצר

יעדים                    ← ריק; מתמלא כאשר יוצרים יעדים
```

**English locale:**

```
Income
  Salary
  Real Estate Income
  Investment Returns
  Freelance

Housing
  Rent (Or Mortgage)
  Cleaning
  HOA
  Municipal Tax (Arnona)
  Maintenance & Repairs

Food
  Groceries
  Restaurants & Cafés
  Coffee

Transportation
  Fuel
  Public Transit
  Parking
  Car Maintenance

Kids
  Daycare / Gan
  Education
  Activities
  Clothing
  Gifts & Toys

Healthcare
  Doctors
  Pharmacy
  Health Insurance

Financial
  Taxes
  Bank Fees
  Loan Payments
  Insurance

Personal
  Clothing
  Entertainment
  Travel
  Subscriptions
  Gifts

Savings
  Emergency Fund
  Short-term Savings

Goals                    ← empty; populated as goals are created
```

Additional locales (US English, etc.) follow the same pattern with market-appropriate tag names (e.g. "HOA" vs "Arnona"). Adding a new locale requires only a new translation file — no structural changes.

#### F4.5 Tag Detail View

Clicking any tag anywhere in the app navigates to the tag detail screen — a first-class view, not a modal or sidebar panel.

**Header:** Tag path as clickable breadcrumb (e.g. "All / Housing / Rent (Or Mortgage)") — each level is clickable to navigate up the tree.

**KPI strip:**

- This period (current month / selected range)
- Average / period (last 12 periods, normalized to selected repeat period)
- Budgeted amount (if budget target set)
- Available to spend (budgeted minus actual; shown in green/red)

**Timeline chart:** Spending curve within the current period, compared to the same period last month. Line chart, two series.

**Trend chart:** 12-period bar chart of actual spending per period for this tag. Horizontal reference line at budget target if set. Period axis adjusts to the tag's configured repeat period (weekly tags show weeks, monthly tags show months).

**Transaction list:** All transactions tagged with this tag (primary or secondary), with the 30-Day Forecast section at the top. Full transaction actions available inline.

**Child tags section:** If the tag has children, a mini-table shows each child's contribution to the total for the current period, with a small bar proportional to contribution.

#### F4.6 Tag Tree Transformation

Two distinct tools for restructuring the tag taxonomy.

**Refactoring Tools (ongoing — accessible from More → Tags):**

_Merge:_ Combine two tags into one. All transactions tagged to the source tag are retagged to the target tag. Budget configuration from the source is discarded (user is prompted to review the target's budget). The source tag is deleted after merge. Merge preview shows a count of affected transactions before committing.

_Reparent:_ Move a tag to a different parent in the tree. All transactions retain their tag assignments — the tag simply moves in the hierarchy. Child tags move with it. Budget targets are unaffected.

_Split:_ Divide one tag into two or more new tags. Because splitting requires deciding which transactions go to which new tag, this opens a transaction review UI: the user assigns each transaction in the source tag to one of the new tags (with AI suggestion based on description). When complete, the source tag is deleted.

_Bulk Retag:_ Select multiple transactions (from any view) and change their primary tag in one action. Secondary tags are unaffected. Confirmation shows count before committing.

_Rename:_ Inline, always available. Affects display name only — all references update automatically.

**Migration Tool (one-time — accessible from Settings → Import → Tag Tree Migration):**

For users arriving from Buxfer, YNAB, or Actual Budget with an existing tag/category structure. The migration tool:

1. Parses the imported tag tree from the source export
2. Displays a side-by-side mapping UI: source tags on the left, Goaldy's current tag tree on the right
3. User maps each source tag to a Goaldy tag (create new, map to existing, or ignore)
4. System previews the transaction count affected by each mapping
5. User confirms — all historical transactions are retagged atomically
6. A migration log is saved showing what was changed (reversible within 24 hours via an undo snapshot)

The migration tool is also used when a user wants to wholesale restructure their existing Goaldy tag tree — e.g., consolidating tags after a year of organic growth into a cleaner taxonomy.

---

### F5 — Categorization Engine

#### F5.1 Rules Engine

Structured categorization rules evaluated on every imported transaction before AI categorization. Rules run in priority order.

**Rule structure:**

- Target tag (the tag to assign if matched)
- Match mode: ALL conditions must match, or ANY condition must match
- Conditions: one or more field-operator-value triples

**Supported fields:** description, merchant, amount, account

**Supported operators:**

- Text fields: contains, equals, starts_with, ends_with, matches_regex
- Amount fields: greater_than, less_than, between, equals

**Example rule:**

```
IF description contains "סופרסל" OR description contains "רמי לוי"
THEN tag as Food / Groceries
```

Rules are created from three entry points:

1. "Add Rule" action on any transaction — pre-fills fields from transaction data
2. Rules management screen (More → Rules) — create from scratch
3. AI suggestion confirmation — confirming an AI suggestion can auto-create a rule

#### F5.2 Self-Cleansing Rules

Rules track `match_count` and `last_matched_at`. The Rules screen surfaces staleness signals:

- **Never matched** (match_count = 0, created >30 days ago) → "This rule has never matched — delete it?"
- **Stale** (last_matched_at > 90 days ago) → "This rule hasn't matched in 3 months — still relevant?"
- **Single match** (match_count = 1) → "This rule matched only once — consolidate with another?"

These are surfaced as inline flags on the rule row, not blocking notifications. User dismisses or acts.

#### F5.3 AI Categorization

Transactions not matched by any rule are batched and sent to Claude for categorization. The prompt includes the user's tag tree and a few-shot set of their confirmed transactions as examples. Claude returns a tag assignment and confidence score per transaction.

Transactions with confidence ≥ 0.7 are auto-assigned and appear in the "AI-tagged" state. Transactions with confidence < 0.7 surface in the triage queue for user review.

**Category source tracking:** every transaction carries a `category_source` field — `rule`, `ai`, `user`, `inherited`, or `untagged`. Visible in the transaction detail panel. The triage health metric uses this to compute the auto-tagged percentage.

#### F5.4 Triage Queue

The triage queue is the untagged or low-confidence-AI-tagged transaction list, accessible via the triage banner.

**Triage view:** shows transactions one at a time (or in a compact list). For each transaction: description, merchant, amount, date, account, and the AI's top 3 tag suggestions with confidence scores.

User actions per transaction: accept top suggestion (→ creates rule prompt), choose different suggestion, type custom tag, mark as transfer (removes from categorization need), skip for later.

---

### F6 — Dashboards

The Dashboard screen has a horizontal tab selector at the top. Tabs are fixed (OOTB) and user-orderable. Custom dashboard creation is a Phase 2 feature.

#### F6.1 Expense Dashboard

**KPI strip:** Total expense this period · vs last period (delta + %) · Average / month (12m)

**Main chart:** Donut chart showing expense breakdown by top-level tags. Clicking any segment drills into that tag's detail view.

**Tag breakdown table:** Tag / Amount / % of total / vs last period — sorted by amount descending. Expandable rows for child tags.

**Top Movers panel:** Tags with the largest absolute change vs previous period. Shows tag name, this period, last period, delta. Red if up, green if down (for expenses).

**Timeline panel:** Spending curve within the current period (day-by-day running total vs same period last month).

**Trend chart:** 12-month bar chart of total expense per month.

#### F6.2 Income Dashboard

Same structure as Expense dashboard but for income transactions. Income tags are those marked `is_income = true`.

**Additional KPI:** Savings rate (income minus expenses, as % of income) — displayed prominently as a single large number with 12-month trend sparkline.

#### F6.3 Net Worth Dashboard

**KPI strip:** Total Net Worth · Liquid Net Worth (liquid + semi-liquid accounts only) · Change this month · Change YTD

**Liquidity decomposition pills (Goaldy-specific):**

```
Liquid ₪X  |  Semi-liquid ₪X  |  Illiquid ₪X  |  Locked ₪X  |  Debt −₪X
```

Each pill is clickable and filters the account list below to that liquidity class.

**Net Worth trend:** Line chart showing total net worth over time (month-end snapshots). Optional toggle to overlay individual account group contributions as stacked area.

**Account group breakdown:** Table showing each liquidity class, total value, change this month, change YTD. Expandable to show individual accounts.

#### F6.4 Cashflow Dashboard

**KPI strip:** Net cashflow this period · Income · Expenses · Projected remaining this month

**Cashflow bar chart:** Income vs Expense per month for the last 12 months. Side-by-side bars, net cashflow line overlaid.

**30-day cashflow timeline:** Day-by-day projection combining actual transactions (to date) with forecast transactions (remainder of month). Shows running balance trajectory. Tight months (projected net negative) are highlighted with an amber band.

**Largest upcoming items:** List of the top 5 projected expenses in the next 30 days by amount, sourced from the forecast engine.

#### F6.5 Budgets Dashboard Tab

Summary view linking to the full Budgets screen. Designed as a quick-check surface, not the primary budget management interface.

**Free Cashflow** — the headline number: budgeted income minus budgeted expenses. Displayed large at the top.

**Budget status summary:** Progress bars for the top 5 expense tag groups. Green if under target, amber if >80% of budget used, red if over. Each bar links to the full Budgets screen at that tag.

**Goals allocation summary:** How much of free cashflow is allocated to active goals. Links to Goals screen.

---

### F7 — Budgets

Budgets is a first-class nav item. It is the monthly financial planning surface — where you set spend targets per tag, review income, and see the resulting free cashflow that feeds the Goals system.

#### F7.1 Budget Screen Layout

**Period navigator:** Month selector with prev/next arrows. Defaults to current month.

**EXPENSE | INCOME tabs** — toggles the budget table between expense tags and income tags.

**KPI strip (Expense tab):**

- Budgeted: sum of all expense tag monthly targets
- Actual Expense: sum of actual transactions in period
- Available: Budgeted minus Actual (green if positive, red if negative)

**KPI strip (Income tab):**

- Budgeted: sum of all income tag monthly targets
- Actual Income: sum of actual income transactions
- Rollover: income carried from prior month (optional, if rollover enabled)
- Available: total available to allocate

**Free Cashflow pill** — always visible regardless of active tab: `Free Cashflow: ₪X,XXX` — income budget minus expense budget. Links to Goals screen.

#### F7.2 Budget Table

Columns: TAG / BUDGETED / ACTUAL / SCHEDULED / AVAILABLE

**Tag column:** Hierarchical display matching the tag tree. Parent rows show aggregate of children. Parent rows are expandable/collapsible. Indented child rows shown when expanded.

**Budgeted column:** The monthly target for this tag. Editable inline — click to open the edit panel (see F7.3).

**Actual column:** Sum of actual transactions in the period tagged to this tag. Clicking navigates to filtered transaction list for that tag in this period.

**Scheduled column:** Sum of projected recurring transactions for this tag in the period. Auto-populated from:

- User-configured repeat schedules
- System-detected recurring patterns (3+ months of consistent pattern) Shown in a muted style to distinguish from actuals. A small "i" icon opens a popover explaining which transactions contribute.

**Available column:** Budgeted minus Actual. Green if positive (room remaining), red if negative (over budget). Parent rows aggregate child availables.

**Row actions:**

- `+` on parent row → add child tag to budget
- Edit icon on any row → opens budget edit panel
- The row itself is clickable → navigates to tag detail view

#### F7.3 Budget Edit Panel

Slide-in panel (not a blocking modal) when editing a budget line. This panel edits the **budget configuration on the tag itself** — budget is a tag property, not a separate record.

**Fields:**

|Field|Type|Notes|
|---|---|---|
|Tag|Read-only display|Shows full path: Housing / Rent (Or Mortgage)|
|Budget type|EXPENSE or INCOME|Determines which tab it appears on. Defaults to the dominant cash flow direction of the tag's transactions. Overridable.|
|Budgeted amount|Number|The target per repeat period|
|Repeat period|Select|**Weekly / Biweekly / Monthly / Quarterly / Annually / Custom**|
|Custom interval|Number + unit|e.g. "every 6 weeks" — shown only when Custom selected|
|Start date|Date|When this budget configuration takes effect|
|Enable rollover|Checkbox|Unused budget from previous period carries forward|
|Repeat until|Date (optional)|For time-bounded budgets — leave blank for ongoing|
|Account scope|Account selector (optional)|Restrict this budget to transactions from one account only|

**Supported repeat periods in full:**

- Weekly — for weekly allowances, grocery runs
- Biweekly — paycheck-aligned budgets
- Monthly — the standard; most tags use this
- Quarterly — insurance premiums, quarterly taxes, HOA in some buildings
- Annually — Arnona (lump sum), car licensing, annual subscriptions
- Custom — any interval the user specifies

When a non-monthly period is selected, the Budget table adapts: the Budgeted column shows the per-period amount with the period label ("₪550 / quarter"), and the Available column prorates correctly based on how far into the current period the user is.

**Inline spending history** — always visible below the form fields:

- Average actual spend: 3 periods / 6 periods / 12 periods (normalized to the selected repeat period)
- Trend sparkline: 12 periods of actuals as a small bar chart
- "Your 12-month average is ₪X — set this as your budget?" one-tap suggestion

This is the primary onboarding mechanism for budget setup. Users see real spending history while entering a target — no separate wizard, no blank-form anxiety.

#### F7.4 Not Budgeted Section

Below the main budget table, a "Not Budgeted" section shows tags that have transactions in the period but no budget target set. Columns match the main table (actual, scheduled) but the budgeted column shows a "— Set target →" prompt.

This surface prompts gradual budget completion over time without forcing upfront setup. First month of use: many rows here. Six months in: this section is empty or near-empty.

#### F7.5 Empty State and First Use

On first access, the budget table is empty. The empty state shows:

1. A brief explanation of what budgets are for in Goaldy (spend targets, not methodology)
2. "Set up budgets from your history" button — analyzes last 3 months of tagged transactions and populates the Not Budgeted section with average actuals as suggested targets
3. User reviews each suggested target, clicks to accept or adjust, and the row moves to the main table

This is more effective than a wizard because the user can move at their own pace, see real data immediately, and skip tags they don't want to budget.

---

### F8 — Goals

#### F8.1 Goal Types

**Type A: Future Expense** Fund a known upcoming cost. Has a target amount and target date. The system computes the required monthly contribution and assesses feasibility against free cashflow.

Examples: vacation, home renovation, car replacement, school fees, wedding.

**Type B: Wealth Accumulation** Grow toward a long-term financial target. Has a target amount, target date, and an assumed annual yield rate (user-set planning assumption). The system models how much of the target will be covered by asset growth vs required contributions.

Examples: retirement portfolio target, investment account milestone, property purchase fund.

**Type C: Debt Payoff** Reduce a liability account to zero. Linked to a loan or credit card account. The system shows payoff date at current payment rate and models the impact of extra payments.

Examples: mortgage payoff, car loan, student loan.

#### F8.2 Goal Data Model

Each goal has:

- Name and type
- Target amount (Type A and B) or target account (Type C)
- Target date
- Linked tag — auto-generated as `Goals/[Goal Name]` on creation (see F8.5)
- Linked account (optional — for balance tracking alongside tag tracking)
- Assumed annual yield % (Type B only — planning assumption, not a prediction)
- Current accumulated amount (derived from tag-linked transactions)
- Status: active, paused, completed, archived

#### F8.3 Goals Planning Cockpit

The main Goals screen. Not a list of cards — a planning surface.

**Header:** Free Cashflow (from Budgets) · Total monthly goal allocation (sum of all active goal required contributions) · Surplus or shortfall

```
Free Cashflow ₪4,840/mo      Goals require ₪5,470/mo      Shortfall ₪630/mo ⚠
```

When a shortfall exists, the system generates resolution suggestions derived from actual spending data:

- "Reduce [tag] spending to its budget target: frees ₪X/mo"
- "Extend [goal] deadline by [N] months: reduces required contribution to ₪X/mo"
- "Pause [goal]: frees ₪X/mo"

These are tappable suggestions that open the relevant edit panel.

**Goal list sections:**

_Upcoming (next 24 months)_ — Type A goals sorted by target date ascending. Each row: goal name, target date, required/mo, progress bar, status indicator.

_Long-term_ — Type B goals. Each row: goal name, target, yield assumption, projected achievement date, required/mo contribution, status.

_Debt Payoff_ — Type C goals. Each row: account name, current balance, payoff date at current rate, monthly payment, extra payment modeling.

#### F8.4 Goal Detail View

Clicking any goal opens its detail view.

**Type A — Future Expense detail:**

- Progress bar: current accumulated vs target amount
- Timeline: months remaining / months elapsed
- Monthly required contribution
- Cashflow feasibility: "your free cashflow supports this goal" or gap warning
- Contribution history: monthly bar chart of tagged contributions
- "What if" adjustments: target amount slider, target date slider — both update the required contribution in real time
- Linked transactions: all transactions tagged `Goals/[Goal Name]`, with the ability to add splits tagging existing transactions to this goal

**Type B — Wealth Accumulation detail:**

- Current account value (from linked account snapshot)
- Projected value at target date at assumed yield (deterministic line, no Monte Carlo)
- Contribution needed to close gap between projected growth and target
- Yield assumption slider — adjusting yield rate updates projection in real time
- Disclaimer: "This is a planning assumption, not a prediction. Actual returns will vary."

**Type C — Debt Payoff detail:**

- Current balance (from linked liability account)
- Payoff date at current minimum payment
- Interest savings from extra payment — "adding ₪X/month saves ₪Y in interest and pays off N months earlier"
- Extra payment slider — updates payoff date and interest savings in real time
- Payment history: monthly bar chart of actual payments

#### F8.5 Auto-Generated Goal Tag Tree

When a goal is created, Goaldy automatically creates a tag subtree:

```
Goals/                          ← created on first goal if it doesn't exist
  Goals/Vacation 2026           ← created with the goal
  Goals/Car Replacement         ← created with the goal
```

The `Goals/` root tag is automatically marked with a goal icon and is excluded from the Expense dashboard's spending breakdown (it's an allocation, not an expense).

Goal tags work exactly like all other tags. The user can:

- Tag a transfer transaction `Goals/Vacation 2026` to register a contribution
- Split a single savings transfer across multiple goal tags (F3.4)
- View all goal contributions in the tag detail view for `Goals/Vacation 2026`

When a goal is completed or archived, its tag is preserved in the tree but marked inactive. Historical contributions remain tagged and visible.

#### F8.6 Goal Lifecycle

```
Create  → tag auto-generated, feasibility shown immediately vs free cashflow
Model   → adjust target / date / yield in detail view before committing
Track   → cockpit shows status; intelligence feed surfaces signals
Fund    → savings deposits tagged or split to goal tags; allocation assistant helps
Review  → period-end shortfall resolution (see F8.7) if contributions fell short
Adjust  → edit any goal parameter; system shows delta vs prior projection
Complete → goal reached; celebratory UI moment; freed cashflow flagged
Archive → moves to history; tag preserved; contributions remain in ledger
```

#### F8.7 Shortfall Resolution

When a goal period closes with actual contributions below the required amount, the system does not silently adjust projections. It surfaces an explicit resolution prompt requiring the user to make an active decision.

**Trigger:** period closes (or within 3 days of period end if real-time detection) with `actual_contributions < required_contribution` for any active goal.

**Batching:** if multiple goals have shortfalls in the same period, they are presented together in a single resolution flow — one modal, one "Apply all." Not separate interruptions per goal.

**Resolution modal:**

```
┌─────────────────────────────────────────────────────────────┐
│  Goal Review — April 2026                                   │
│                                                             │
│  2 goals were underfunded this month                        │
│                                                             │
│  Car Replacement                                            │
│  Required ₪2,000  ·  Funded ₪1,200  ·  Short ₪800         │
│                                                             │
│  ○  Extend deadline  Mar 2027 → Apr 2027                    │
│     Monthly contribution stays at ₪2,000                   │
│                                                             │
│  ○  Increase contributions                                  │
│     Keep Mar 2027 · Raise remaining months ₪2,000 → ₪2,089 │
│                                                             │
│  ○  One-time miss — no adjustment                           │
│     Record shortfall, keep original plan                    │
│     (Choose this if you'll fund the shortfall manually)     │
│  ─────────────────────────────────────────────────────────  │
│  Roof Repair                                                │
│  Required ₪970  ·  Funded ₪0  ·  Short ₪970               │
│  ○  Extend deadline  ○  Increase contributions  ○  Miss     │
│                                                             │
│  [Apply choices]                      [Remind me in 3 days] │
└─────────────────────────────────────────────────────────────┘
```

**The three options:**

_Extend deadline_ — target date pushed forward by the minimum number of periods needed to absorb the shortfall at the current contribution rate. Monthly requirement stays the same.

_Increase contributions_ — target date held fixed. The shortfall is distributed across remaining periods, increasing each period's required contribution by the prorated amount. Shows the new required amount before the user commits.

_One-time miss_ — no adjustment to timeline or contribution rate. The shortfall is recorded in goal history as a deliberate miss. Use when the user intends to catch up via a larger contribution in a future period. The intelligence feed monitors whether catch-up happens and resurfaces if it doesn't within 2 periods.

**Deferral handling:** "Remind me in 3 days" is the only deferral option. Maximum two deferrals. After the second deferral, the system automatically applies "Extend deadline" as the safest conservative default and notifies the user that a default resolution was applied. Goals do not remain in unresolved limbo indefinitely.

**Goal history:** regardless of which option is chosen, every period's contribution amount, required amount, shortfall, and resolution choice is recorded in the goal's history. The detail view can show this as a timeline of funding decisions — useful for understanding how a goal evolved over time.

---

### F9 — Intelligence Feed

A dedicated screen presenting a daily financial briefing. Not a notification list, not a chatbot. Each card is a self-contained insight with observation, context, and options.

#### F9.1 Card Types

**Anomaly alert (red):** Spending spike in a tag vs the user's own historical baseline.

```
Household Items up 280% vs your 6-month average
₪3,200 this month vs typical ₪850.
Three transactions: IKEA (₪1,800), Ace Hardware (₪1,100), unnamed (₪300).
[Tag as one-time]  [See transactions]  [Set new baseline]
```

**Cashflow forecast (amber):** Upcoming 30-day cash pressure with specific scheduled outflows.

```
Next 30 days: tighter than usual
Projected outflows ₪29,400 vs projected income ₪28,100. Net: -₪1,300.
Three large items: Mortgage (₪6,300 on 10th), HOA (₪550 on 15th),
Insurance (₪1,800 on 22nd).
Your savings buffer covers this comfortably.
[See cashflow timeline]
```

**Goal signal (green):** Goal on track, drifting, or ahead of schedule.

```
Vacation 2026 — drifting
You've contributed ₪2,800 this month vs required ₪1,500.
You're 3 months ahead. Consider reallocating ₪800/mo to Car Replacement.
[Rebalance goals]  [Keep current pace]
```

**Israeli-specific (blue):** קרן השתלמות, pension milestones, mortgage rate signals.

```
קרן השתלמות — 14 months to vesting
Projected value at vesting: ₪328,000 (tax-free withdrawal available).
Consider your options: withdraw and invest, leave to compound,
or use toward property purchase.
[Model scenarios]
```

**Budget nudge (amber/orange):** Tag approaching or over budget with days remaining.

```
Kids/Daycare — ₪100 over budget with 28 days remaining
₪5,100 vs ₪5,000 target. The month is only 2 days old.
[Adjust budget]  [See transactions]  [It's a one-time charge]
```

**Lead gen opportunity (gold) — Phase 4 only:** Contextually matched financial product offer. Always the last card in the feed. Visually distinct styling makes clear this is an offer, not a system insight. Pro users see no lead gen cards.

#### F9.2 Feed Management

Cards are generated daily. User can:

- Mark card as read (collapses)
- Dismiss card (removes from feed, won't regenerate for 30 days)
- Act on the card's primary action
- Scroll past (card remains until explicitly dismissed or read)

Feed is sorted by priority (system-assigned based on urgency). Anomalies and cashflow warnings appear before informational cards.

---

### F10 — Reports

Reports in Goaldy are primarily accessed via the **tag detail view** (F4.5) — clicking any tag from anywhere gives you its full report. The Reports nav item provides comparison and cross-tag analysis surfaces.

#### F10.1 Tag Comparison Report

Compare multiple tags over the same period. User selects 2–6 tags and a date range. Output: overlaid trend lines, side-by-side period breakdown, combined vs individual totals.

Use case: "How does my Groceries spending compare to Restaurants over the last year?"

#### F10.2 Income vs Expense Report

12-month bar chart (income bars, expense bars, net cashflow line). Period selectable. All accounts or filtered to selected accounts. Exportable as PNG.

#### F10.3 Net Worth Over Time Report

Line chart of total net worth by month, with optional breakdown by account group (liquid, investment, locked, real estate, debt) as stacked area. Date range selectable from account history start.

#### F10.4 Savings Rate Report

Monthly savings rate (net cashflow as % of income) over time. 12-month trend line, 3-month and 12-month rolling averages. Benchmark line optional (user-set target savings rate).

#### F10.5 Custom Date Ranges

All reports support custom date ranges including: this month, last month, last 3 / 6 / 12 months, this year, last year, custom range, all time.

---

### F11 — Internationalization and Multi-Currency

#### F11.1 Supported Locales at MVP

- English (LTR) — `en`
- Hebrew (RTL) — `he`

Locale is stored in user profile. No URL prefix. RTL layout switches automatically — all layout uses CSS logical properties (`padding-inline-start`, `margin-inline-end`) so direction switching requires no component changes.

#### F11.2 Currency Formatting

All amounts formatted via `Intl.NumberFormat` with explicit locale and currency. Never string-concatenated. The symbol position (left in USD, right in ILS) and number formatting (comma separators, decimal places) are locale-correct automatically.

#### F11.3 Multi-Currency Accounts

Each account has a base currency. Foreign-currency amounts are displayed in native currency on account lines and converted to display currency (ILS default) for aggregations. Conversion uses daily fetched rates with transparent source display.

#### F11.4 Hebrew-Specific Handling

Hebrew transaction descriptions from Israeli bank scrapers are preserved as-is regardless of UI locale. The rules engine matches Hebrew text. Claude categorizes Hebrew descriptions correctly without translation.

---

### F12 — Israeli Financial Instruments

#### F12.1 קרן השתלמות (Keren Hishtalmut)

Account type: Investment, Locked liquidity class.

Additional fields at account creation:

- Start date (when contributions began)
- Vesting date (6 years from start date — auto-calculated but editable)
- Employer + employee monthly contribution amounts

Intelligence feed card activates 24 months before vesting with countdown. At vesting, card presents three standard decision scenarios:

1. Withdraw (tax-free) — shows net amount after any applicable deductions
2. Leave to compound — shows projected value at user-defined future date
3. Roll into pension — shows projected pension impact

#### F12.2 Pension (פנסיה)

Account type: Investment, Locked liquidity class. Snapshot-based balance.

Additional fields: employer contribution %, employee contribution %, expected retirement age.

Intelligence feed card tracks projected value at retirement age using a user-set assumed annual yield rate (same model as Type B goals).

#### F12.3 Mortgage Refinancing Intelligence

For accounts of type Loan with subtype Mortgage, Goaldy fetches the current Bank of Israel prime rate (public API, daily) and compares against the user's mortgage rate (entered at account creation).

When the spread exceeds a configurable threshold (default: 0.5%), an intelligence feed card surfaces:

```
Mortgage refinancing opportunity
Current rate: 3.2%  |  Your rate: 4.1%  |  Spread: 0.9%
On your ₪850,000 balance, refinancing could save ₪637/month.
[Find refinancing options →]  ← Phase 4 lead gen
```

In Phase 3, the card shows the opportunity without the lead gen link. In Phase 4, the link activates.

#### F12.4 Arnona and Annual Israeli Expenses

The rules engine ships with OOTB detection patterns for major annual Israeli expenses: Arnona (municipal tax), insurance renewals, vehicle licensing. These are flagged as "annual" in the intelligence feed so they don't skew monthly averages.

---

### F13 — Settings and Integrations

#### F13.1 Profile Settings

Display currency, locale (en/he), name, email, timezone.

#### F13.2 Integration Settings

**API Keys:**

- Create named API key (Moneyman, custom script, etc.)
- Key shown once on creation
- List of active keys with name, created date, last used date
- Revoke individual keys

**SimpleFIN:**

- Paste access URL to connect US bank accounts
- Shows last sync date, sync status, error state if applicable
- Manual "Sync now" trigger

**Google Drive:**

- Drive connection status
- Last sync timestamp
- "Open goaldy.db in Drive" link
- "Export goaldy.db" — download a copy of the raw SQLite file
- "Disconnect Drive" — with clear warning about what this means

#### F13.3 Data Management

- Export all transactions as CSV (all accounts or filtered)
- Export goaldy.db (raw SQLite — the complete financial database)
- Import from Buxfer, Actual Budget, YNAB
- Delete account — removes session and Postgres profile; `goaldy.db` in Drive is the user's own property and is not deleted by Goaldy

---

### F14 — Landing Page

#### F14.1 Purpose and Access Model

The landing page is a **closed beta acquisition surface**. The primary CTA is "Request Early Access" — not immediate signup. Approved applicants receive an invitation email with a signup link. This controls rollout, ensures early users are genuinely motivated, and creates social proof through exclusivity.

The page is public (no auth required). All app screens behind `/app/*` require authentication. The landing page lives at the root `/`.

#### F14.2 Page Structure

Modeled after Monarch's structural approach — a single long-form page with a clear narrative arc: hook → problem → solution → features → social proof → CTA. Monarch's copy is strong because it names the emotional benefit ("money clarity") before the features. Goaldy does the same.

---

**Section 1 — Hero**

Full-width, above the fold. No app screenshot (nothing to show in beta).

**Headline:**

> Your finances. Your data. Finally in one place.

**Subheadline:**

> Goaldy connects your Israeli and international accounts, tracks your spending, plans your future — and stores everything in your own Google Drive. Not ours.

**CTA button:** `Request Early Access →` **Secondary link:** `Already have an invite? Sign in`

**Social proof line (below CTA):**

> Replacing Buxfer, Monarch, and spreadsheets — starting with Israel.

---

**Section 2 — The Problem (implicit comparison)**

Three-column cards. No competitor logos — just the pain points.

|Card 1|Card 2|Card 3|
|---|---|---|
|**Your data lives on their servers**|**Built for one market**|**Feature bloat you never use**|
|Most finance apps store your transaction history on their infrastructure. A breach is their problem — and yours.|Mint, YNAB, and Monarch are US-first. They don't know what a קרן השתלמות is.|Complex methodologies that require constant maintenance. You came here to understand your money, not do accounting homework.|

---

**Section 3 — The Goaldy Difference**

Three-column feature cards, icon + headline + short copy. On-brand gold accent on icons.

**Card 1 — Your data in your Drive**

> Every transaction, account, and balance lives in a single file in your Google Drive. Goaldy reads it. You own it. Export it, delete it, or keep it forever — we have no say.

**Card 2 — Israeli and international, unified**

> Hapoalim, Leumi, Max, Chase, Schwab, Fidelity — one dashboard. With native support for קרן השתלמות, pension, and mortgage intelligence that no international tool provides.

**Card 3 — Intelligence without the methodology**

> Goaldy tracks, categorises, and analyses your money without telling you how to manage it. No envelope budgeting, no rigid system. Just clear data and smart insights on your terms.

---

**Section 4 — Features (tabbed or stacked)**

Four feature highlights mirroring Monarch's "Track / Budget / Collaborate / Plan" structure but Goaldy-specific:

**Know your net worth — really** All accounts — liquid, locked, illiquid, liabilities — decomposed by accessibility. See how much is actually available vs tied up in property or pension.

**Budgets that flex to reality** Set spend targets per category, watch actuals track against them in real time, and let the system surface when something's drifting — without locking you into a methodology.

**Plan future expenses before they surprise you** Name upcoming costs, set timelines, and Goaldy shows you whether your free cashflow can absorb them — individually and in combination. The mental math you do every few months, done automatically.

**Categorise once, forget forever** A powerful rules engine learns how you categorise transactions. Claude AI handles the rest. After a few weeks, your inbox runs itself.

---

**Section 5 — Pricing / Beta**

Simple, one-line section. No pricing table.

> **Free during beta.** No credit card. No commitment. We're building this in the open and early users shape what it becomes.

---

**Section 6 — Final CTA**

Repeat of the hero CTA with a supporting line.

**Headline:** Ready to actually understand your finances? **CTA button:** `Request Early Access →` **Supporting copy:** Invite-only beta. No credit card required.

---

**Section 7 — Footer**

Minimal. Logo + "Goaldy © 2026" + links: Privacy Policy · Terms of Use · Contact

---

#### F14.3 Early Access Request Form

Clicking "Request Early Access" opens a modal (not a new page):

**Fields:**

- Email address (required)
- "How do you currently manage your finances?" — short text (optional, qualifies intent)
- "Do you have accounts in Israel, the US, or both?" — radio (Israel only / US only / Both)

**Submission:** records the applicant in a `waitlist` table in Postgres (email, answers, timestamp, source). Sends an automated acknowledgement email from `noreply@goaldy.ai`.

**Post-submission state:** modal updates to "You're on the list — we'll be in touch." No redirect.

**Invitation flow (separate, admin-triggered):** an invitation email contains a unique one-time signup link. The link expires in 7 days. Clicking it initiates the Google OAuth flow.

---

#### F14.4 Navigation Bar

Minimal. Fixed top.

**Left:** Goaldy logo (G mark + wordmark) **Right:** `Sign in` (text link) · `Request Access` (gold button)

No other nav items on the landing page. No marketing sub-pages in MVP.

---

#### F14.5 Design Language

The landing page uses the Goaldy brand palette and the same Plus Jakarta Sans typeface as the app. Dark background option for the hero section (Midnight Indigo `#283593` or near-black) with Amber Gold `#F9A825` accents on CTAs, icon highlights, and section dividers. Body sections on Cool Grey `#F4F7F6`. This creates a premium feel consistent with the product positioning while differentiating from the app's white-surface UI.

---

### F15 — Authentication and Onboarding

#### F15.1 Auth Model

**Google OAuth only.** No email/password. No magic links. No other SSO providers at MVP.

Rationale: the app requires Google Drive access (`drive.appdata` scope) as a core architectural dependency. Requiring Google auth aligns the account identity with the storage provider and eliminates an entire class of credential management complexity. Users who do not have a Google account cannot use Goaldy in its current form — this is an accepted constraint.

**Strict fallback policy:** Google account issues (suspension, loss of access) are the user's responsibility to resolve with Google. Goaldy provides no account recovery path. This is communicated clearly at signup.

#### F15.2 OAuth Scopes Requested at Signup

Both scopes are requested in a single combined consent flow at first sign-in. The user sees one Google consent screen, not two.

|Scope|Purpose|
|---|---|
|`openid`|Identity|
|`email`|User profile|
|`profile`|Display name|
|`https://www.googleapis.com/auth/drive.appdata`|Read/write `goaldy.db` in the user's Drive `appDataFolder`|

The `drive.appdata` scope must be justified in the Google OAuth app registration with a clear description: "Goaldy stores your financial database as a single file in your Google Drive's private app folder. This file is only accessible to Goaldy and is not visible in your regular Drive interface."

#### F15.3 First Login — New User Onboarding Flow

When a user completes OAuth for the first time (no existing `users` record in Postgres):

```
Step 1 — Account created
  Postgres user record created with email, name, locale default 'en'
  Drive connection record created with access + refresh tokens

Step 2 — Locale and currency selection
  Single screen: "Choose your language" (English / עברית)
  Display currency selector (ILS / USD / EUR — more can be added later)
  [Continue →]

Step 3 — Google Drive initialisation
  System creates goaldy.db in Drive appDataFolder
  Applies SQLite schema and inserts default meta row
  "Your private financial file has been created in Google Drive.
   Only you and Goaldy can access it."
  [Continue →]

Step 4 — Tag starter stack
  "Start with a suggested category structure?"
  Shows a preview of the localised OOTB tag tree (collapsed, first 5 tags visible)
  [Use suggested tags]  [Start empty]  [I'll import from Buxfer/YNAB →]

Step 5 — First import prompt
  "How would you like to bring in your financial data?"
  ○ Upload a CSV or OFX file
  ○ Connect via REST API (for Moneyman users)
  ○ I'll add data manually
  [Skip for now]
  
  → Routes to the appropriate import flow or drops into the empty app
```

Onboarding is linear and cannot be skipped past Step 3 (Drive initialisation is required). Steps 4 and 5 can be skipped. The onboarding state is tracked in Postgres (`users.onboarding_completed_at`).

#### F15.4 Returning User Login Flow

```
User clicks "Sign in" on landing page
  → Google OAuth (accounts.google.com)
  → Callback to /api/auth/callback/google
  → NextAuth creates/validates session
  → Drive token refreshed if within 7 days of expiry
  → Redirect to /app/dashboard
  → App loads: downloads goaldy.db from Drive → mounts in OPFS → renders
```

If the user has a valid session already (cookie present, not expired), `/app/*` renders directly without re-auth. The landing page `Sign in` button checks for an active session and redirects to `/app/dashboard` if found.

#### F15.5 Invitation Link Flow

```
Admin sends invitation email containing:
  https://app.goaldy.com/invite?token=<uuid>

User clicks link:
  → Token validated (exists in waitlist table, not expired, not used)
  → Redirected to Google OAuth with invite_token in state param
  → On OAuth callback: token marked used, user created, onboarding begins
  → Expired or already-used tokens show a clear error with contact link
```

Invitation tokens expire after 7 days. One invitation = one account. Tokens are single-use.

---

### F16 — Session Management

#### F16.1 Session Policy

**7-day hard expiry.** Sessions expire exactly 7 days after creation regardless of activity. No rolling/sliding window. The user is prompted to re-authenticate at the next app open after expiry.

Rationale: financial data is sensitive. A found or shared device should not provide indefinite access. 7 days balances security with daily-use convenience.

#### F16.2 Single Active Session

**One active session per user at all times.** A new login on any device invalidates all previous sessions.

Implementation: each session stores a `session_token` in Postgres. On every authenticated request, the token is validated against the current stored token. If a new login occurs, the stored token is replaced — all prior requests with the old token are rejected with a 401 and a clear message: "You've been signed in on another device. Please sign in again."

This is consistent with the single-session architecture constraint from the data model (no concurrent writes to `goaldy.db`).

#### F16.3 Session Storage

Sessions are stored as HTTP-only, Secure, SameSite=Strict cookies. No session data in localStorage. The session cookie contains the NextAuth JWT which is validated server-side on every tRPC request.

#### F16.4 Google OAuth Token Lifecycle

Google refresh tokens are stored encrypted in `drive_connections`. Access tokens (valid for 1 hour) are refreshed automatically when within 5 minutes of expiry on any Drive operation.

If the refresh token is revoked (user removed Goaldy from their Google account permissions), the next Drive operation fails with a Google auth error. The app detects this and:

1. Invalidates the current session
2. Shows a full-screen error: "Your Google Drive connection has been disconnected. Please sign in again to reconnect."
3. Re-initiating OAuth re-requests the `drive.appdata` scope

This is the **only forced re-auth trigger** beyond the 7-day expiry.

#### F16.5 Sign Out

Sign out invalidates the session cookie and deletes the session record from Postgres. The Drive connection record and `goaldy.db` in Drive are unaffected — the user's data remains intact. Redirect to the landing page after sign out.

---

### F17 — Notification System

#### F17.1 Architecture — NotificationService Abstraction

All notifications are sent through a `NotificationService` interface. The MVP implementation is `EmailProvider` (Resend). Future implementations (WhatsApp via Twilio, Telegram via Bot API) are drop-in additions.

```typescript
// packages/notifications/index.ts

interface Notification {
  userId: string;
  event: NotificationEvent;
  data: Record<string, unknown>;  // event-specific payload, never raw financial data
}

interface NotificationProvider {
  send(notification: Notification): Promise<void>;
}

class NotificationService {
  constructor(private providers: NotificationProvider[]) {}

  async notify(notification: Notification): Promise<void> {
    const prefs = await getUserNotificationPrefs(notification.userId);
    const enabledProviders = this.providers.filter(
      p => prefs.enabledChannels.includes(p.channel)
    );
    await Promise.allSettled(enabledProviders.map(p => p.send(notification)));
  }
}

// MVP wiring
const notificationService = new NotificationService([
  new EmailProvider({ from: 'noreply@goaldy.ai', client: resend }),
]);

// Future — add without changing existing code:
// new WhatsAppProvider({ ... })
// new TelegramProvider({ ... })
```

User notification preferences are stored in Postgres (`notification_preferences` table: user_id, channel, event_type, enabled). Users manage preferences in Settings → Notifications.

#### F17.2 Notification Events

**Real-time events** (sent immediately on trigger):

|Event|Trigger|Subject line|
|---|---|---|
|`sync.failed`|`/api/ingest` returns non-200 or SimpleFIN cron job errors|"Goaldy sync failed — action needed"|
|`drive.disconnected`|Google refresh token revoked|"Your Google Drive connection needs attention"|
|`session.new_device`|New login invalidates prior session|"New sign-in to your Goaldy account"|

**Weekly digest** (sent every Monday 09:00 user timezone):

|Content block|Included when|
|---|---|
|Untagged transactions count|triage queue has >0 items|
|Goal shortfalls awaiting resolution|any goal has unresolved shortfall|
|Budget summary|any tag is >90% of budget or over budget|
|Top insight of the week|highest-priority intelligence feed card|
|Savings rate vs prior week|always included|

The weekly digest is a single email combining all relevant blocks. If no blocks are relevant (everything is clean), no digest is sent that week — avoiding empty "nothing to report" emails.

**Notification preferences default state:**

|Event|Default|
|---|---|
|`sync.failed`|Enabled|
|`drive.disconnected`|Enabled|
|`session.new_device`|Enabled|
|Weekly digest|Enabled|

All can be disabled individually in Settings → Notifications.

#### F17.3 Email Design

Sender: `Goaldy <noreply@goaldy.ai>` Reply-to: none (no-reply is genuine — no monitored inbox)

Email design: plain, clean, on-brand. Midnight Indigo header with Goaldy logo. White body. Amber Gold CTA buttons. Plus Jakarta Sans via web-safe fallback (Arial). No tracking pixels. Unsubscribe link in footer for digest emails (required by law). Real-time alerts do not have an unsubscribe link — they are account security notifications.

**Email templates (Resend):**

|Template|Type|
|---|---|
|`sync-failed`|Real-time|
|`drive-disconnected`|Real-time|
|`session-new-device`|Real-time|
|`weekly-digest`|Weekly|
|`invite`|One-time|
|`waitlist-confirmation`|One-time|

#### F17.4 Privacy Constraint

**No financial data in notification content.** Email bodies may reference counts ("3 transactions need tagging") and status signals ("your sync failed") but never amounts, merchant names, account balances, or transaction descriptions. This is both a privacy principle and a practical security measure — email is unencrypted in transit.

#### F17.5 Future Channels (WhatsApp / Telegram)

When WhatsApp or Telegram providers are added:

- A new class implementing `NotificationProvider` is created
- User preference table gains a new channel option
- No changes to `NotificationService`, `NotificationEvent` types, or any existing email code
- Real-time events are well-suited for WhatsApp/Telegram; the weekly digest remains email-only

---

## 4. Non-Goals (Explicit Exclusions)

These will never be added without a deliberate PRD amendment:

|Feature|Reason excluded|
|---|---|
|Envelope budgeting / YNAB methodology|No opinion on budgeting methodology; conflicts with design principles|
|Bank credential management|Goaldy never holds or proxies bank credentials|
|Plaid / Finanda / broker-side integrations|User brings their own data pipeline|
|Transaction execution (payments, transfers)|Read-only financial intelligence only|
|Multi-user / household shared ledger|Single-user only; concurrent sessions unsupported by architecture|
|Individual investment holdings tracking|Portfolio value tracked at account level only|
|Mobile native app|Web-only until Phase 4; responsive design is not a mobile app|
|Social or comparison features|No sharing, no benchmarking against other users|
|Public API for third-party consumers|Ingest API is inbound only|
|Email/password authentication|Google OAuth only; no credential management|
|Multi-provider SSO|Google only at MVP; Drive dependency makes this a structural constraint|

---

## 5. Phased Delivery

### Phase 0 — Builder Replaces Buxfer

F2 (Buxfer import), F1 (accounts), F3 (transaction list, basic tagging), F4 (tag tree setup), F5 (rules engine), **F15 (auth + onboarding)**, **F16 (session management)**

**Gate:** Builder uses Goaldy daily and cancels Buxfer subscription.

### Phase 1 — Core Product

F5 (AI categorization + triage), F6 (Expense + Income + Net Worth dashboards), F7 (Budgets screen, basic), F11 (i18n, Hebrew + RTL), **F17 (notifications — real-time alerts only)**

### Phase 1.5 — Public Beta Launch

**F14 (landing page + waitlist)**, F17 (weekly digest), first external users onboarded

### Phase 2 — Intelligence Layer

F6 (Cashflow dashboard), F8 (Goals — Type A and C), F9 (Intelligence feed — anomaly + cashflow + goal cards), F12 (Israeli financial instruments — קרן השתלמות + pension), F13 (SimpleFIN integration, API keys)

### Phase 3 — Planning and Depth

F8 (Goals — Type B wealth accumulation), F9 (mortgage refinancing card), F10 (comparison reports, savings rate), F3 (split transactions, 30-day forecast), F7 (budget edit inline history, scheduled column)

### Phase 4 — Monetization

F9 (lead gen cards for free tier), Stripe integration, Pro tier feature flags, legal review of lead gen framing, mobile-responsive polish

---

_Document created: 2026-04-05. This is a living document — update before each phase begins and after any material product decision._