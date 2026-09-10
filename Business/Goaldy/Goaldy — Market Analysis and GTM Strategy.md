| Field            | Value                                      |
| ---------------- | ------------------------------------------ |
| **Created**      | 2026-09-05 14:00 UTC                       |
| **Last Updated** | 2026-09-10 11:00 UTC v2.0                  |
| **Version**      | 2.0                                        |
| **Status**       | Decided — v2.0 is the committed plan       |
| **Author**       | Business development / market research session |
| **Related**      | PRD v2.6 (esp. §2, F18), TDD §13            |

### Change Log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-09-05 | Initial market analysis. Industry scan of personal-finance, self-hosted-finance, family-office and SMB-cashflow categories. Verdict on family-office and SMB fit (both rejected as primary segments, with a narrow qualified wedge defined for each). Feature gap analysis ranked by revenue proximity vs. build cost. GTM recommendation built around a free self-hosted tier, with a structural correction to PRD F18.4 (hosting is the primary revenue line, not an add-on to AI) and a challenge to the closed-source decision. |
| 1.1 | 2026-09-05 | **ICP/segmentation (§8) and a customer-anchored feature roadmap (§9) added** — the substance of this revision. Two corrections to v1.0 driven by new evidence. (a) **The beachhead is re-specified.** v1.0 named cross-border/multi-currency households as the beachhead on a "nobody serves this person" claim; that claim is wrong — a cohort of 2025–26 entrants (Borderless Budget, FlowFund, Tallyroot, Auritrack, Monavio) targets exactly this person, and Lunch Money owns the niche natively at a $60/yr pay-what-you-want minimum. Multi-currency is therefore re-classified as a **targeting filter and defensibility moat, not the value proposition** — it is the cheapest axis in the market. (b) **The value proposition is re-specified as planning, not budgeting**, on two findings: planning tools price 30% above full PFM suites (ProjectionLab $129/yr, $1,199 lifetime, $549/yr advisor; Boldin $144/yr — for a simulator with no ledger underneath), and the category's documented #1 churn cause is apps that "show data without producing behavior change" and fail to connect cashflow to a plan. Goaldy's shipped Plan simulator (PRD F8) is the only asset that addresses that, and no self-hosted competitor has one. Consequent changes: Goaldy Cloud repriced from $96/yr to $120–144/yr with a lifetime option added; the roadmap is re-ordered by **retention risk** rather than feature appeal (manual-entry apps churn at 3x the rate of auto-sync apps); Goaldy.AI moves from Phase 2 to the **last** horizon. |
| 1.2 | 2026-09-05 | **§10 added: licensing, hosting-vs-self-hosting economics, and the value-exchange model** — who revenue comes from versus who insight, defect discovery and credibility come from, treated as two separate transactions with different currencies. Licensing resolved with a recommendation rather than options: **AGPL-3.0 for the core with a separately-licensed proprietary Pro package (open core), DCO rather than CLA.** Reasoning turns on the fact that AGPL's one real weakness — it does not stop a competitor hosting your software — describes a risk that effectively does not exist for a trust-and-support-bound niche PFM, while its benefits (main awesome-selfhosted listing rather than `non-free.md`, OSI credibility with the distribution segment, and community-contributed bank/institution import adapters) attack the one problem money and coding agents both handle badly: long-tail institution coverage, which requires real accounts at real institutions to test. FSL-1.1 documented as the fallback if hosting rights must be protected outright. **H5 re-slotted** in light of the AI/MCP work already being delivered by a coding agent: collapsing build cost is the argument *against* monetizing that layer, not for it — a feature an agent produces in a week is reproducible by every competitor's agent and cannot hold a price. Chat and MCP therefore ship **free with BYO API key** as a free-tier differentiator, launch headline and discovery channel (MCP registries as distribution), with only *managed inference* metered. Also documents the per-tenant-SQLite hosting cost advantage (85–95% gross margin, and an exit promise that is literally a file copy) and the trust obligations that hosting creates. |
| **2.0** | **2026-09-10** | **Strategy reversed to self-hosted-first; §7, §9, §10 and §11 rewritten.** v1.0–v1.2 built toward a managed hosted service as the primary revenue line. That is now **deferred, not planned**, on three grounds: hosting makes you a controller of financial personal data with duties that `AS IS` cannot disclaim (Israel PPL Amendment 13 in force 14 Aug 2025, GDPR breach notification, DPIA, EU region choice); no one has yet paid for Goaldy at all, so building the infrastructure and legal wrapper for a subscription business inverts the risk; and adding services later is cheap while retreating from them is not. Four substantive reversals. (a) **Licensing: closed source, reversing v1.2's AGPL recommendation** — the AGPL case rested on needing contributors for long-tail bank adapters, but Obsidian demonstrates a plugin API delivers that ecosystem without source release, and `IMPORT_ADAPTERS` is already shaped as that boundary; trust comes from an open data format (the byte-identical backup/restore guarantee), not readable code, as Obsidian, Plex, Unraid and Blue Iris all demonstrate. (b) **Aggregation needs no server** — the app is already a server on the user's machine and can call SimpleFIN/GoCardless directly, so the single highest-impact anti-churn feature costs zero infrastructure, zero held credentials and zero support liability. (c) **REST API and MCP move to the free tier permanently** — gating them is unenforceable against a local SQLite file, they *are* portability, and MCP is both commoditized (Kubera, Era) and a discovery channel. (d) **"Goaldy Connect" retired as a concept** for bundling four services of very different risk profiles under one name; products are now named by what they cost you (Pro holds nothing, Sync holds ciphertext, Managed Accounts holds bank tokens). New in §7.5: a **telemetry and signal design** — update check on by default and disclosed, usage telemetry strictly opt-in with a published payload, bucketed counts and no financial values, plus the explicit warning that opt-in telemetry is selection-biased toward enthusiasts and its D30 figure is an upper bound rather than a measurement. §7.6 sets a dated 90-day decision point. §9 collapses five horizons into one, with the next chosen by observed demand. §10 reframes liability as the cost each deferred service would spend. §§1–6 and §8 (industry map, segment verdicts, ICP) are unchanged and still stand. |

---

## 0. Executive verdict (read this if you read nothing else)

**The plan, in one sentence: ship an excellent self-hosted product, closed source, free,
instrumented — sell nothing but a $25 supporter tier — and let demand decide what gets
built and hosted next.**

1. **Family offices: no.** No securities model, no cost basis, no performance calculation,
   no entity/ownership graph, no documents, no capital calls — those five things *are* the
   product that segment buys, and the buyer additionally requires SOC 2, insurance and an
   SLA a solo founder cannot supply. Kubera Black already sells entity nesting plus MCP at
   $2,499/yr. §3.
2. **SMB: no, as accounting.** AR/AP/VAT/payroll/statutory double-entry is a moat owned by
   QuickBooks and Xero, and the forecasting layer above them (Float, Pulse) wins by
   *reading* those ledgers. What's real is a persona, not a segment: the owner-operator who
   is also the household CFO — served by the same entity layer as the HNW household. §4.
3. **You sell the answer to a decision, not a budgeting app.** Planning tools price *above*
   full PFM suites (ProjectionLab $129/yr and $1,199 lifetime for a simulator with no
   ledger; Boldin $144/yr; Monarch $99.99 for a complete aggregated PFM). The documented #1
   cause of abandonment is apps that "show data without producing behavior change" and never
   connect cashflow to a plan. **Your Plan simulator answers the category's biggest churn
   problem and no self-hosted competitor has one.** §8.
4. **Multi-currency is the targeting filter and the moat — not the pitch.** The cross-border
   niche is not empty (Borderless Budget, FlowFund, Tallyroot, Auritrack, Monavio all
   launched into it) and Lunch Money owns it at a $60/yr minimum — the cheapest price point
   in the market. Use it to find and hold customers; sell them the plan. §8.
5. **Order by retention risk.** Manual-entry apps churn users at **3x** the rate of
   auto-sync apps; category D30 retention averages 38%. That settles the sequencing. §9.
6. **Aggregation requires no server of yours.** The app is already a server on the user's
   machine and can call SimpleFIN or GoCardless directly. The single highest-impact
   anti-churn feature costs zero infrastructure and holds zero credentials. §7.3.
7. **Hosting is deferred, and that is a liability decision as much as a commercial one.**
   Hosting makes you a controller of financial personal data — Israel's Amendment 13 (in
   force 14 Aug 2025, real fines, a DPO threshold), GDPR breach notification, DPIA, EU
   region choice — none of it disclaimable by an `AS IS` clause. Self-hosted software is a
   licence. §10.
8. **Closed source, open data format.** Reversing v1.2: a plugin API buys the ecosystem
   without a source release (Obsidian's model, and `IMPORT_ADAPTERS` is already that
   boundary). Trust comes from being able to leave with your data — your byte-identical
   export guarantee is a stronger claim than most open projects can make. Put it on the
   landing page. §7.4.
9. **The cheaper a feature is to build, the less it can be sold for.** Your agent shipping
   MCP and chat in a week is the reason that layer cannot be the paid tier. Free, BYO key,
   and let MCP registries recruit for you. §7.2.
10. **Never paywall portability, correctness, or security.** Gating the API also fails on
    its own terms — the ledger is a local SQLite file, so the paywall stops nobody and
    annoys the honest. §7.2.
11. **Instrument before you launch, and set the threshold before you're invested.** Update
    check on by default and disclosed; usage telemetry strictly opt-in with a published,
    inspectable payload and no financial values. Know that opt-in telemetry is
    selection-biased toward enthusiasts — the D30 figure it gives you is an upper bound.
    Money (Catalyst) is the only unfakeable signal. §7.5, §7.6.

---

## 1. What Goaldy actually is, stated without marketing

Strip the PRD and the schema says it plainly. Goaldy is a **single-entry cashflow ledger with a tag taxonomy, a per-tag budget engine, and a deterministic household planning simulator**, self-hosted as one SQLite file.

Tables that exist: `accounts`, `transactions`, `transaction_tags`, `tags`, `tag_budgets`, `rules*`, `plan_*`, `notifications`, `users`, `sessions`.

Tables that do **not** exist, and their absence defines the ceiling:

| Missing | What it blocks |
|---|---|
| `holdings` / `securities` / `prices` | Any investment product. Net worth for an `investment` account is a number a human types in. |
| `cost_basis`, `lots`, `corporate_actions` | Performance, unrealized P&L, tax lots, anything an investor calls "returns". |
| `entities` / `ownership` | Trusts, LLCs, companies, "whose money is this", consolidation. Blocks *both* family office and SMB owner. |
| `documents` / attachments | K-1s, statements, receipts, contracts. Table stakes in every professional-grade tool. |
| `audit_log` | Any buyer with a fiduciary duty or an accountant. |
| `commitments` / `capital_calls` | Alternative assets — the actual pain family offices pay to solve. |

This is not a criticism of the build. It's a precise statement of which markets are physically reachable and which are not.

---

## 2. The industry map (2026)

Five distinct categories, five distinct buyers. Confusing them is where most positioning dies.

### 2.1 Mass-market PFM (personal finance management)
- **Monarch** — $99.99/yr Core, $199/yr Plus. The post-Mint winner. Best-in-class household collaboration (unlimited members, shared + individual goals). US-centric.
- **Copilot Money** — $95/yr, iOS-only. Design leader, aggressively narrow platform bet.
- **YNAB** — $109/yr. Methodology religion (zero-based envelopes). Highest retention in the category *because* of the religion; also its ceiling.
- **Buxfer** — your explicit feature baseline. Deep, ugly, small.

**Read:** the category has settled at **$95–$110/yr** for a polished, aggregated, mobile-first, cloud product. That is your price anchor whether you like it or not. A self-hosted product cannot charge more than this without a categorically different value story.

### 2.2 Self-hosted / open-source finance — Goaldy's actual competitive set
- **Firefly III** — PHP, double-entry, powerful rules, multi-currency, huge Docker install base and growing on privacy + subscription fatigue. Weakness: forces users to learn accounting; UI is functional-ugly.
- **Actual Budget** — YNAB-style envelope budgeting, genuinely modern UI, local-first sync. Weakness: methodology lock-in; no ledger depth.
- **Ghostfolio** — AGPL, ~8,100 GitHub stars, the most popular OSS portfolio tracker; cloud Premium is a **~$48 one-time** payment. Investments only, no budgeting.
- **Beancount / Ledger / hledger** — plain-text double entry, git-versioned. Zero UI, infinite rigor, tiny addressable audience.
- **Maybe Finance** — open-sourced after failing commercially. A cautionary tale worth internalizing: great UX + OSS + no distribution ≠ business.

**Read:** the category has a **hole exactly Goaldy-shaped** — ledger depth *without* accounting religion, with Monarch-grade UI, plus budgets, plus a planning simulator. Nobody occupies it. That hole is real and it is your best asset. But note the monetization reality of this category: Ghostfolio, the *most popular* product in it, monetizes at $48 once. Willingness to pay among self-hosters is brutally low.

### 2.3 Net-worth / HNW dashboards (the segment adjacent to your family-office question)
- **Kubera** — $249/yr Essentials; **$2,499/yr "Black"** for HNWIs and family offices, adding **nested portfolios for trusts/LLCs/family ownership structures**, granular access control, concierge onboarding. Has shipped **MCP** and AI document/screenshot import; repositioned entirely around "Works for you. Works for your AI."
- **Vyzer** — HNWI + family office, multi-user, collaboration, built around alternative-investment reporting.
- **Era Context** — connects accounts, enriches, exposes to any agent via MCP.

**Read:** this is the closest reachable adjacency to Goaldy's stated HNW ambition — and the entry ticket is *nested entity structures + aggregation + AI/MCP*. Kubera charges 10x for exactly the entity layer Goaldy lacks. That is your pricing signal and your feature signal in one.

### 2.4 True family-office platforms
- **Addepar**, **Eton Solutions AtlasFive**, **Masttro**, **Asset Vantage**, **FundCount**, **Aleta**, **Asora**.
- Market: ~8,000+ family offices, $3.1–5.9T AUM. Pricing bifurcated: flat-fee consolidation tools from **~$900–$1,000/month**, enterprise reporting platforms at **six figures to $250k+/yr**, typically requiring $25M–$50M+ in assets.
- The pain they're paid to solve, per the vendor and analyst literature, is **not** budgeting: it is *private/alternative asset tracking, entity sprawl, multi-jurisdiction consolidation, and reconciliation of unstructured quarterly (and retroactively restated) data from GPs.*

**Read:** Goaldy addresses zero of the four. See §3.

### 2.5 SMB cash-flow forecasting
- **Float** (~$59/mo), **Pulse** (~$55/mo), **Fathom** (board reporting + 3-way modeling), Causal/OnPlan modules. Category price band **$29–$60/mo**.
- Critical structural fact: **every one of these is a layer on top of Xero/QuickBooks Online/FreeAgent.** None of them owns the ledger. They read it.

**Read:** Goaldy's Plan engine is architecturally the same *shape* as Float. But Float's entire viability rests on reading QBO/Xero, which Goaldy cannot do. See §4.

---

## 3. Family offices — the honest answer

### 3.1 Why this doesn't work
**Product gap.** The four things family-office software is bought for — consolidated multi-entity reporting, alternative/illiquid asset tracking, performance measurement, and document-driven reconciliation — map to four tables Goaldy doesn't have. Building them is not "features"; it's a portfolio-accounting engine, a corporate-actions pipeline, a market-data vendor relationship, and an entity/ownership graph. Realistically 12–18 months of focused work for a team, and you are one person.

**Buyer gap.** Family offices buy through relationships, consultant shortlists, and references from other family offices. Cycles are 3–9 months. Procurement will ask for SOC 2 Type II, penetration test results, cyber-liability insurance, a support SLA, and a business-continuity plan. A solo founder distributing a closed-source Docker image cannot answer a single one of those questions. This is not a marketing problem you can out-write.

**The self-hosting argument cuts both ways.** Yes, this audience is privacy-obsessed — the literature confirms genuine demand for client-controlled keys and private/Swiss-hosted infrastructure. But they do not self-host; their outsourced CFO or MSP does, and that person wants a *vendor* with insurance and a phone number, not a container. "You run it yourself" reads to them as "there is no one to sue."

**The differentiator is gone.** Kubera Black already sells entity nesting + access control + MCP at $2,499. You would be entering as the cheaper, less-featured, less-supported option in a market where nobody optimizes for cheap.

### 3.2 The narrow, qualified wedge — if you insist
There is one legitimate sub-segment: **the pre-family-office HNW household.** Net worth $2M–$20M, 2–6 legal entities (a trust, a holding company, a rental LLC, maybe a foreign account), no staff, currently running on a spreadsheet, and constitutionally unwilling to upload their complete balance sheet to a US SaaS.

For that person the ask is not Addepar. It is:
- accounts that belong to an **entity**, with a consolidate/filter toggle;
- a **holdings** table with cost basis and daily price fetch;
- **document attachment** per account/transaction;
- an **audit log**;
- **multi-user with roles** (spouse, accountant read-only, advisor read-only).

That is a **schema extension and five screens** — not a new product. It is genuinely reachable in a quarter. And it prices at **$25–$40/mo** as a "Household Pro" tier — 10x under Kubera Black, which is exactly where a self-hosted challenger should sit.

**But be clear-eyed about the size:** this is a low-thousands-of-buyers global market, hard to reach (they don't hang out anywhere), and long-sales-cycle even at $40/mo. Treat it as a **Phase 3 margin expander on an existing user base**, never as a beachhead.

**Do not** call it "family office software" in any public copy. You will be laughed at by the buyers and it will confuse the audience that actually converts.

---

## 4. SMBs — the honest answer

### 4.1 Why the obvious version doesn't work
Real SMB finance software requires: invoices and AR, bills and AP, VAT/GST/sales-tax reporting and filing, payroll, bank reconciliation, a statutory chart of accounts, true double-entry, an accountant-facing export, an audit trail, and multi-user roles with segregation of duties. Goaldy has single-entry with tags. Even Firefly III — which *does* have double entry and is free — has not meaningfully taken SMB share from QuickBooks/Xero, because the moat isn't the ledger, it's **the accountant, the tax authority, and the bank feed.**

The forecasting layer (Float, Pulse, Fathom) is where Goaldy's Plan engine would compete, and every player there wins by *integrating* with QBO/Xero, not replacing them. You would need to build QBO and Xero read integrations — which are cloud OAuth APIs, structurally at odds with your "no OAuth provider stands between you and your data" principle (PRD §1.2) and your "no third-party identity provider" non-goal (PRD §3).

### 4.2 The narrow, qualified wedge
Two sub-segments are real:

**(a) The owner-operator's consolidated personal + business view.** QuickBooks will not show your personal net worth. Monarch will not show your business runway. The founder/freelancer/landlord who is *both* has to keep two systems and a spreadsheet to bridge them. Goaldy — with the same `entities` layer §3.2 requires — solves this natively: one ledger, entity-tagged, consolidated or filtered. **The same feature serves both segments.** That's the efficient bet.

**(b) Micro-entities that don't need statutory accounting:** solo consultants, landlords with a few units, side businesses under a sole-trader structure. Real, but note this is a *low* willingness-to-pay segment already served free by Wave and Firefly III.

### 4.3 Verdict
**SMB is not a segment. "The owner-operator who is also the household CFO" is a persona inside your existing segment.** Serve it with the entity layer and stop there. Do not build AR/AP/VAT/payroll. Ever.

---

## 5. The AI/MCP plan (PRD F18) — stress test

**What survives:**
- BYO-API-key AI in the self-hosted app is smart and cheap. It costs you nothing in COGS, sidesteps most of the "you're shipping my bank data to a model vendor" objection, and creates the on-ramp to a managed tier.
- The insight-to-action and range-based-risk (Monte Carlo on the Plan engine) capabilities are genuinely differentiated. Nobody in self-hosted finance has a planning simulator at all, let alone a probabilistic one. **This, not categorization, is the defensible AI story.**
- Excluding tax optimization was correct. Don't revisit it.

**What does not survive:**
- **F18.1's core positioning claim is stale.** "The harmonized system of record that per-institution MCPs lack" was a good insight in August. By September, Kubera ships MCP over an aggregated, entity-nested portfolio, and Era ships MCP over aggregated accounts. You are not the first mover; you are the one **without aggregation**, which is the input the whole claim depends on. The PRD already flags ingestion as a prerequisite — that flag should be upgraded from "prerequisite" to "the actual product."
- **AI-tier-first monetization is a margin trap.** Inference COGS on a $10–15/mo add-on, sold to power users who will hammer it, has a real chance of negative gross margin per user. Any managed AI tier must be usage-capped with a hard ceiling and a BYO-key escape hatch.
- **F18.4 has hosting backwards.** See §7.

---

## 6. Feature gap analysis — ranked by (revenue proximity × cheapness)

Each item names the profitable product that proves the demand.

### Tier A — build before any public launch (weeks each, unblocks everything)

> **v1.1:** superseded as a *sequence* by the horizon plan in §9, which orders the same items by retention risk. This table remains the per-item justification.

| # | Feature | Proven by | Why now |
|---|---|---|---|
| A0 | **`asset_valuations` time series** (added v1.1) | Kubera, Vyzer — valuation history is the product | Not a gap, a **live defect**: `reported_value` is a scalar overwritten in place, so illiquid assets have no history, chart as flat lines, and feed today's number into every historical and projected figure. Fixes the asset class P0 cares most about. ~1 week. |
| A1 | **SimpleFIN Bridge ingestion** (user pays $15/yr direct, read-only, no BaaS contract, no credential proxying) | Actual Budget's entire growth story | The single highest-leverage feature in this document. Removes the #1 complaint in every review of every competitor. Preserves your "no bank-credential proxying" non-goal because the user holds the SimpleFIN relationship, not you. |
| A2 | **GoCardless Bank Account Data (PSD2)** for EU/UK; keep Moneyman for IL | Firefly III's importer ecosystem | Free tier available; makes the cross-border beachhead (§8) real rather than aspirational. |
| A3 | **OFX/QIF import** (already Planned, F2.1) | Every incumbent | Cheap. Also the migration path off Quicken/Mint exports. |
| A4 | **Recurring / subscription detection + upcoming-cashflow calendar** (F6.4) | Monarch, Copilot — the single most-cited "wow" feature in reviews | Pure logic over data you already have. Near-zero new infrastructure. Reuses the rules engine. |
| A5 | **Reports page** (currently a "Coming soon" stub) | — | Already flagged in the PRD as a launch blocker. It is. A visible stub on HN reads as abandonware. |
| A6 | **Onboarding + localized starter tag tree** (F15.2, F4.4) | YNAB's onboarding is a retention weapon | Empty-state is where self-hosted finance apps lose 80% of installs. |
| A7 | **One-click encrypted scheduled backup** (F13.3) | Every self-hosted product that survived | "Own your data" is a liability until it's "own your data, safely." Cheap trust. |
| A8 | **PWA installability** (F17.5) | — | Two files. Removes the "no mobile app" objection at near-zero cost. |

### Tier B — the revenue expanders (a quarter, opens §3.2 and §4.2)

| # | Feature | Proven by | Notes |
|---|---|---|---|
| B1 | **`entities` + ownership layer** — accounts belong to a person/trust/LLC/company; consolidate or filter | Kubera Black ($2,499/yr sells largely on this); Vyzer | **The highest-value single item in this document after A1.** One table, one FK, one selector. Opens the HNW-household *and* the owner-operator personas simultaneously. |
| B2 | **Holdings-lite**: `holdings` (symbol, qty, cost basis) + daily price fetch + unrealized P&L + allocation view | Ghostfolio (8.1k stars proves OSS feasibility); Kubera | Upgrades net worth from "a number you type" to a real balance sheet. Prerequisite for any investor conversation. |
| B3 | **Multi-user with roles** (owner / partner / accountant-read-only / advisor-read-only) — `users` table already exists | Monarch's collaboration is its top retention driver | Half-built already. Finish it. |
| B4 | **Document attachments** on transactions and accounts, with full-text search | Every family-office and SMB tool | SQLite + filesystem + FTS5. A weekend, honestly. |
| B5 | **Audit log** (immutable change history) | Required by any buyer with a fiduciary or accountant | Cheap in SQLite. Non-negotiable for B1's audience. |
| B6 | **Accountant export pack** (CSV/XLSX by entity by period) | — | Turns the accountant from a blocker into a distribution channel. |
| B7 | **Scheduled email/PDF report** | Fathom, Asora | Recurring re-engagement hook; also the killer feature for the advisor-read-only role. |

### Tier C — explicitly do not build
AR/AP/invoicing, VAT/tax filing, payroll, statutory double-entry accounting, tax optimization (correctly already excluded), bank-credential proxying, trade execution, partnership/GL accounting, capital-call workflows, a native mobile app.

---

## 7. Go-to-market — self-hosted first

> **Rewritten in v2.0.** v1.0–v1.2 built toward a managed hosted service as the primary
> revenue line. That is now **deferred, not planned**: it is unanswerable without users,
> and it carries a data-custody liability that is unjustifiable before demand is proven.

### 7.1 The strategy on one page

**Ship a genuinely excellent self-hosted product, closed source, free. Instrument it.
Sell nothing at first except a supporter tier. Let demand decide what gets built next.**

Every architectural and commercial question this document spent three revisions on —
Postgres vs SQLite, multi-tenancy, hosting liability, sync pricing, entity modelling —
depends on facts that do not exist yet. Users generate those facts. Nothing else does.

The three constraints that produced this:

1. **Liability.** Hosting makes you a data controller of financial personal data:
   breach notification, DPIA, Israel's Amendment 13 (in force 14 Aug 2025, with real
   administrative fines and a DPO threshold), EU region choice. None of it is
   disclaimable. Self-hosted software is a licence with an `AS IS` clause. §10.
2. **Proof.** Nobody has yet paid for Goaldy. Building a subscription business, its
   infrastructure and its legal wrapper before the first sale inverts the risk.
3. **Cost of reversal.** Launching self-hosted and adding services later is cheap.
   Launching a service and retreating from it is not.

### 7.2 The ladder — what exists, what's deferred

**Free — the complete application, forever, uncrippled**

Accounts, transactions, tags, budgets, rules, dashboard, **the Plan simulator**,
`asset_valuations` and net-worth history, manual + BYO-aggregator ingestion, full
export/backup/restore, **REST API and local MCP**, **AI chat with BYO API key**, the
import-adapter plugin API, Docker and (later) desktop app.

Three of those are gated by rule, not by tier:

- **Portability is never paywalled.** Export, backup, restore, and the API *are*
  portability. Gating the API also fails on its own terms: the ledger is a local SQLite
  file, so anyone technical enough to want the API can bypass it with `sqlite3`. An
  unenforceable paywall stops nobody and annoys the honest.
- **Correctness is never paywalled.** `asset_valuations` fixes a live defect.
- **MCP is distribution, not product.** It is already commoditized (Kubera, Era), and MCP
  registries are how the technical half of P0 finds tools. Give it away and let it recruit.

**Catalyst — $25 one-time.** Early builds, a badge, a private channel. Zero COGS, pure
margin, ships in a week. **The only pre-launch instrument that measures willingness to
pay rather than curiosity.** Live on day one, not later.

**Pro — $99–149 one-time, or $59/yr — built only when asked for.** Entities/ownership,
multi-user roles (partner, accountant read-only, advisor read-only), audit log, document
attachments, accountant export pack, scheduled reports. Local features, licence-gated.
Imperfectly enforceable and that is fine — Plex, Unraid and Blue Iris prove people pay.

**Deferred, with the trigger that would un-defer each:**

| Deferred | Build it when |
|---|---|
| Desktop app (Electron) | The landing page's "want a desktop app?" list crosses a few hundred |
| E2EE Sync ($4–6/mo) | Users have two devices and complain |
| Managed accounts (you hold bank tokens) | Demand is loud *and* revenue can fund the support load |
| Goaldy Cloud (full hosting) | Only if the numbers demand it — and with a lawyer first |
| Postgres migration, multi-tenancy | Only if hosting happens. Irrelevant otherwise. |

**"Goaldy Connect" is retired as a concept.** It bundled four services with very
different risk profiles under one name — licence, aggregation, sync, MCP relay — which
is precisely how a founder commits to the risky one by accident. Name products by what
they cost you: *Pro* (you hold nothing), *Sync* (ciphertext), *Managed Accounts* (tokens).

### 7.3 The aggregation insight that makes this work

The desktop app — and the Docker instance — **is already a server on the user's machine.
It can call SimpleFIN or GoCardless directly.** The user holds the aggregator
relationship (SimpleFIN is $15/yr paid to SimpleFIN); the app fetches on a schedule,
locally.

That delivers most of "connected accounts" with **zero servers, zero tokens held, zero
COGS, and zero broken-bank support tickets that are yours.** The only thing lost is the
setup chore, which is an onboarding problem, not a reason to take custody of credentials.

This is what makes the whole self-hosted-first plan viable rather than merely cheap:
the single feature that most reduces churn does not require you to run anything.

### 7.4 Licensing — resolved: closed source, open data format

v1.2 recommended AGPL. **That is reversed.** The argument turned on needing community
contributors for long-tail bank adapters — but Obsidian demonstrates that a **plugin API
delivers an ecosystem without source release**, and Goaldy's `IMPORT_ADAPTERS` registry
is already shaped as exactly that boundary (`parse()` is bytes → `ParsedImport`; adapters
never touch the DB).

Closed source is a proven model for precisely this audience: **Obsidian** (closed,
local-first, free core, paid services), **Plex**, **Unraid**, **Blue Iris**. What they
share is not open source — it is an **open data format**:

> **Trust comes from being able to leave with your data, not from reading the code.**

Goaldy's claim here is stronger than Obsidian's: a single SQLite file plus a
backup/restore pipeline with a **byte-identical, CI-gated roundtrip guarantee**. That has
been treated as an implementation detail; it is the central trust artifact and belongs on
the landing page.

**Accepted costs:** listing on awesome-selfhosted's `non-free.md` rather than the main
list; no GitHub Issues tab (so a feedback channel is a launch blocker, per PRD F13.2);
and the closed-source question on Show HN — for which the prepared answer is the export
guarantee above, delivered without defensiveness.

### 7.5 Signal — telemetry, and what it can and cannot tell you

**Opt-in telemetry is legitimate here, but for a privacy-positioned finance product the
design decides whether it produces signal or destroys the brand.** Two separate
mechanisms, deliberately, because they answer different questions and warrant different
defaults.

**(a) Update check — on by default, disclosed, disableable.** A plain "is there a newer
version?" request. This is expected behaviour for self-hosted software (Home Assistant,
Nextcloud), it has genuine user value, and it yields the workhorse metric —
**active-instance count, version distribution, and rough retention** — at near-100%
participation. Disclose it in onboarding and the README; provide `GOALDY_DISABLE_UPDATE_CHECK`.

**(b) Usage telemetry — opt-in, never opt-out.** For a product whose pitch is "your
financial data never leaves your machine," **default-on analytics is an existential
risk**: the day someone finds an undisclosed request in their firewall log, the brand is
finished, and "but it was anonymous" does not survive that thread.

Rules for the payload:

- **Ask once, in onboarding, in plain language, showing the actual JSON.**
- **Send:** a locally generated random instance UUID, app version, install date, coarse
  feature-usage flags, and **bucketed** counts ("10–50 accounts", never "23").
- **Never send:** amounts, currencies, balances, tag names, account names, descriptions,
  institution names, locale-identifying detail, or anything per-transaction.
- **Publish the exact payload** in the docs, and show the last payload sent in Settings.
  Inspectable beats promised.
- **Do not log IPs**, or truncate them. An instance UUID plus an IP is arguably personal
  data; truncate, document retention, and keep the surface trivial.

**The limitation to internalise before reading any of it:** opt-in telemetry is
**selection-biased toward enthusiasts.** The users who churn at day 20 are
disproportionately the ones who declined. **Your telemetry D30 number will be
optimistic** — treat it as an upper bound, not a measurement.

**Higher-signal instruments than telemetry, in ascending order of value:**

| Signal | Worth |
|---|---|
| GitHub stars, HN upvotes, Reddit comments | Vanity. Ignore. |
| Downloads, Docker pulls | Weak. Anyone installs anything. |
| **Instances active at D30** (from the update check) | **Strong — the core metric** |
| Detailed unsolicited bug reports | Strong. Nobody writes those about abandoned software. |
| **Catalyst purchases** | **Strongest. Money is the only unfakeable signal.** |

Add one cheap qualitative instrument: an **in-app prompt at day 14** asking what's
missing. Self-hosted software has no uninstall survey — people simply stop — so this is
the only chance to hear from someone before they go quiet.

**The selection bias that will mislead you most:** launching Docker-only to r/selfhosted
means you only ever hear from people who *can* run Docker. P0 — the anxious cross-border
household — bounces silently, and the absence reads as no demand. **Put a "Desktop app —
want this?" signup on the landing page** so non-Docker visitors self-identify. That
converts the bias into a measurable signal and tells you whether the Electron project is
worth 2–4 weeks.

### 7.6 The decision point — set before launch, while still honest

**90 days after launch:**

| Metric | Target |
|---|---|
| Instances active at D30 | **500+** |
| Catalyst supporters | **25+** |
| Desktop-app waitlist | 200+ |
| Detailed bug reports from real use | 20+ |

**Miss both of the first two and the problem is positioning, not features.** Re-open this
document rather than building more. Hit them and the next question — Pro, desktop, or
sync — gets answered by what users actually ask for, not by this analysis.


---

## 8. ICP — who the customer is, what hurts, what they'll pay, why us

### 8.1 The one-sentence positioning

> **Goaldy is the private household balance sheet that tells you whether the decision you're about to make actually works — across every currency, account and asset you own.**

Note what is *not* in that sentence: budgeting, categorization, self-hosting, AI. Those are how it works, not why anyone buys.

### 8.2 The evidence that sets the price

| Product | Price | What it sells |
|---|---|---|
| Ghostfolio (cloud Premium) | **$48 one-time** | Self-hosted portfolio tracking |
| Lunch Money | **$60/yr** min (PWYW) | Multi-currency budgeting, indie/bootstrapped |
| Copilot | $95/yr | Design-led PFM, iOS only |
| Monarch Core / Plus | $99.99 / $199 per yr | Full aggregated PFM + household collaboration |
| YNAB | $109/yr | A budgeting *method* |
| **ProjectionLab** | **$129/yr · $1,199 lifetime · $549/yr advisor** | **A planning simulator with no ledger under it** |
| **Boldin** | **$144/yr** | Retirement planning + Monte Carlo |
| Kubera / Kubera Black | $249 / $2,499 per yr | Net worth + entity nesting + MCP |
| Float / Pulse | $55–59/mo | SMB cashflow forecasting on top of QBO/Xero |
| Firefly III, Actual Budget | **$0** | Self-hosted budgeting |

**The three readings that matter:**
1. **Planning out-earns budgeting.** ProjectionLab charges 30% more than Monarch for strictly less product — no accounts, no transactions, no ledger. People pay for the answer, not the record-keeping.
2. **Multi-currency is the discount axis.** Its specialist charges the least in the table.
3. **Self-hosting has near-zero willingness to pay on its own.** The two most popular products in that row are free, and the most popular paid one monetizes at $48 *once*.

Goaldy's shipped assets sit on the *expensive* side of that table (a planning simulator, a real balance sheet) and its distribution sits on the *cheap* side (self-hosted, multi-currency). **Price and pitch from the expensive side; distribute from the cheap side.**

### 8.3 P0 — the primary ICP: "The Anxious Balance Sheet"

**Who.** 35–55. Household net worth $250k–$3M. 6–20 accounts spanning **two or more currencies and usually two countries**. Owns at least one asset that isn't a bank balance — an apartment, a pension or keren hishtalmut, RSUs, a foreign brokerage. Technically capable (can follow a Docker quickstart, or would rather just pay for hosting). Israeli, Israeli-expat, EU/US dual-country, or a tech worker paid in a currency they don't live in.

**The decision they're anxious about — this is the actual trigger.** Nobody adopts a finance app because they want a pie chart. They adopt one within weeks of a question they can't answer:
- *Can we afford to move countries / buy this apartment / put two kids through university?*
- *If I stop working in four years, does this hold?*
- *Am I actually richer than last year, or is that just the shekel?*

**The pain, specifically.**
1. **Their net worth is unknowable.** Assets in 2–4 currencies, an apartment whose value is a guess, a pension nobody can read. No single aggregator covers their institutions. The truth lives in a spreadsheet they update quarterly and don't trust.
2. **Every app they've tried is single-country.** "Most personal finance apps are built for one country. They connect to one country's banks, track one currency, and give advice relevant to one tax system." For them these apps are structurally useless, not merely inconvenient.
3. **The apps that do work show only the past.** The documented #1 churn cause across the category: apps that "show data without producing behavior change… do not tell them what to do next, do not connect cash flow to broader financial planning." That is exactly the gap between a ledger and a decision.
4. **They will not upload their complete balance sheet to a US startup.** For this cohort specifically — dual-jurisdiction, often with a real reason to be careful — this is a hard constraint, not a preference.

**What they'll pay.** **$120–144/yr for Cloud**, or **$449 lifetime**. They already pay for Wise, an accountant, sometimes a one-off financial plan at $1,500+. Goaldy at $12/mo is not competing with Monarch's $8/mo in their head — it is competing with the spreadsheet, and with the advisor they don't want to hire.

**Why us, honestly — three claims, all currently defensible:**
| Claim | Who else has it |
|---|---|
| A household planning simulator wired to a real ledger | ProjectionLab has the simulator, no ledger. Monarch has the ledger, only goal-tracking. **Nobody has both.** |
| True multi-currency + real RTL/Hebrew + an illiquid-asset valuation history | Lunch Money has multi-currency. Nobody has the combination, and nobody will have valuation history until you ship §Feature 1. |
| Runs entirely on infrastructure they control | Firefly and Actual — neither of which has planning or a usable UI |

**Where we are genuinely weaker, and must fix:** no automated ingestion (existential — see §9), no holdings, no valuation history, no reports, a "Coming soon" nav item.

### 8.4 S1 — "The Homelab Household": distribution, not revenue

Runs a NAS and 20 containers; has already tried Firefly III and bounced off its accounting model, or Actual Budget and rejected its envelope religion. Reachable in one place (r/selfhosted, 790k members, 98.3% containerized) at near-zero cost.

**Pain:** the self-hosted finance category is functional and ugly; the pretty products are cloud-only.
**Will pay:** ~$0. Budget them at **0.3–0.5% conversion** and treat everything above that as luck.
**Their real value:** launch velocity, GitHub stars, awesome-selfhosted inclusion, bug reports, and — critically — **credibility with P0**, who reads those forums before trusting anyone with their finances.
**The rule:** serve them completely and cripple nothing, but never let their feature requests set the roadmap. They will ask for double-entry, plain-text export and a CLI. P0 will not.

### 8.5 S2 / S3 — expansion segments (Phase 3, not now)

- **S2 "Pre-family-office household"** — $2M–$20M, 2–6 legal entities, no staff, a spreadsheet. Needs entities, holdings, documents, audit log, advisor read-only. **$25–40/mo.** Reachable *only* as an upsell out of P0; unreachable cold. (§3.2)
- **S3 "Owner-operator"** — founder/freelancer/landlord who is both the business's and the household's CFO. Served by the *same* entities feature as S2. **Never build AR/AP/VAT/payroll.** (§4.2)

### 8.6 The anti-ICP — say no out loud

**The US-only budget-app switcher.** The post-Mint refugee looking for a Monarch alternative. Do not chase them: they want one-click aggregation across 12,000 US institutions that you cannot match, they price-anchor at $8/mo, they churn at 62% by day 30, and they will never self-host. Every feature request from this segment pulls you toward being a worse Monarch.

---

## 9. The roadmap — one horizon, then listen

> **Rewritten in v2.0.** The five-horizon plan in v1.1 sequenced toward a hosted
> monetization event in H3. With hosting deferred, there is exactly one horizon worth
> planning, and the rest is decided by what users ask for.

The ordering principle is unchanged and still evidence-backed: **apps requiring manual
entry lose users at 3x the rate of auto-sync apps, and category D30 retention averages
38%.** Sequence by churn removed, not by interest.

### H1 — "Super useful, tells no lies" — the only planned horizon

Everything below is the definition of "super useful" derived from the two documented
causes of abandonment: manual entry, and apps that show data without connecting cashflow
to a decision.

| # | Item | Why it's in H1 |
|---|---|---|
| 1 | **`asset_valuations` time series** | Not a gap — a **live defect**. `reported_value` is a scalar overwritten in place, so illiquid assets chart flat and editing a property value retroactively rewrites net-worth history. It is currently wrong on the public demo, for P0's core asset class. Add purchase price/date for appreciation % nearly free. ~1 week. |
| 2 | **In-app BYO-aggregator ingestion** (SimpleFIN token pasted into the app; GoCardless for EU/UK) | Directly attacks the 3x churn multiplier, and per §7.3 requires **no server of yours**. |
| 3 | **OFX/QIF import** | Cheap; also the migration path off Quicken and Mint exports. |
| 4 | **Recurring / upcoming-cashflow detection** | The most-cited "wow" feature in Monarch and Copilot reviews. Pure logic over existing data, reusing the rules engine. |
| 5 | **The Plan promoted to a first-class surface** | The only asset no competitor in this category has, and the answer to the #1 documented churn cause. It is currently a nav item. It should be the dashboard, the onboarding, and the pitch. |
| 6 | **Reports** (retire the "Coming soon" stub) | A visible stub reads as abandonware to S1 on launch day. |
| 7 | **Onboarding + localized starter tag tree** | Empty state is where installs die before D1. |
| 8 | **Encrypted scheduled backup** | "Own your data" is a liability until it's "safely." |
| 9 | **Update check + opt-in telemetry** (§7.5) | Without this the launch produces anecdotes instead of data. |
| 10 | **Public feedback channel** | Closed source means no Issues tab; the bug reports are the entire point of the free tier. |
| 11 | **Catalyst supporter tier** | One Stripe link. The only instrument that measures willingness to pay. |
| 12 | **PWA installability** | Two files; removes the "no mobile" objection at near-zero cost. |

**Exit criterion:** a stranger can install, connect or import, and reach one true insight
about a real decision — without hitting a stub, a wrong number, or a manual chore.

### Then: listen

The next horizon is chosen by the §7.6 signals, not by this document. The candidates,
each already specified above, in the order they are most likely to be demanded:

**Desktop app** (if the waitlist fills) → **Pro features** (if people ask for entities,
roles or an accountant view) → **E2EE Sync** (if two-device complaints appear) →
**Managed accounts** (only if loud and fundable).

### Permanently out of scope

AR/AP/invoicing, VAT/tax filing, payroll, statutory double-entry, tax optimization,
bank-credential proxying, trade execution, partnership/GL accounting, capital calls, a
native mobile app, multi-tenancy, and anything requested primarily by S1 that P0 would
never use.


---

## 10. Liability — what self-hosting buys, and what each service would cost

*Not legal advice. An Israeli privacy/tech lawyer and an accountant are both required
before the first sale. This section maps the terrain so those conversations are short.*

### 10.1 The delta

| | Self-hosted (the plan) | If you hosted |
|---|---|---|
| Your legal role | Software vendor. You never touch their data. | **Controller and processor of financial personal data** |
| Breach duty | None — nothing to breach | GDPR 72h + Israeli PPA + data subjects |
| Data loss | Their problem, disclaimable | Yours, and your brand |
| Availability | No obligation | Contractual |
| Disclaimable by `AS IS`? | Largely yes | **No — statutory duties cannot be contracted away** |

That last row is why the sequencing in §7 is a liability decision as much as a
commercial one. An `AS IS` clause covers software defects; it does nothing about
data-protection obligations.

### 10.2 The regulatory facts that apply

- **Israel PPL Amendment 13, in force 14 August 2025** — administrative fines, power to
  suspend database operations, a real breach-notification regime; the PPA issued its first
  fine that same month. Mandatory DPO for large databases or sensitive-data processors;
  the grace period ended 31 October 2025. **Where Goaldy would fall on that threshold is a
  specific question for the lawyer** — and it only arises if you host.
- **EU adequacy for Israel** was reaffirmed January 2024 and is reviewed four-yearly, but
  is under sustained civil-society and parliamentary pressure to be reassessed. **If you
  ever host EU customers, pick an EU region** rather than depending on adequacy holding.
- **Aggregation is not a licence problem if you never touch credentials.** GoCardless is
  an authorised AISP across the EEA and UK; SimpleFIN's relationship is with the user.
  Consuming a licensed provider keeps you a software vendor.

**Write this into the PRD as a non-goal:**

> **Goaldy never handles, stores, or proxies bank credentials.** Aggregation is always via
> a licensed third party the user consents to directly.

### 10.3 Risks ranked, and their mitigations

1. **Personal liability from not incorporating** — near-certain exposure, catastrophic,
   cheapest fix on the list. **No revenue into a personal account. Incorporate first.**
2. **Advice liability** — grows with exactly the strategy §9 recommends (promoting the
   Plan, insight-to-action). Mitigation is framing: **arithmetic, not instruction**
   ("this goal is short by $400/mo"), never "you should," never a named security or
   product, every projection labelled illustrative. Keeps ~95% of the commercial value.
3. **VAT/tax on digital services** — boring, near-certain to be wrong if ignored.
   EU B2C digital services means OSS registration; plus Israeli VAT.
4. **Consumer/auto-renewal law** — low severity, easy; matters once Pro is annual.
5. **Data breach / loss** — **structurally near-zero while you host nothing.** This is
   the single largest benefit of the plan and the thing each deferred service spends.

**Before the first sale:** incorporate; ToS with liability capped at fees paid, warranty
disclaimer, explicit "not financial advice," and "verify figures against your bank";
privacy policy covering telemetry and the update check specifically; consider cyber /
tech E&O insurance (cheap relative to exposure, and the underwriting questionnaire forces
hygiene).

### 10.4 The value exchange — who pays in money, who pays in everything else

Two transactions, run simultaneously. **The free tier tests your software. The supporter
tier tests your business.** Neither substitutes for the other.

| Constituency | Gives you | Costs you | Never |
|---|---|---|---|
| **S1 self-hosters** | Defect discovery in configurations you'll never own; import adapters for institutions you don't bank with; translations; listings; **credibility with P0, who reads those forums before trusting you** | Support hours; feature-request gravity toward the wrong product | Cripple the free tier. Let them set the roadmap — they'll ask for double-entry, plain-text export and a CLI; P0 wants none of it |
| **P0 households** | The purchases and the retention that prove the thesis | Support; expectations | Paywall their data or their exit |
| **Catalyst supporters** | The only honest willingness-to-pay signal you have | Almost nothing | Over-promise what the badge buys |

**The asymmetry that justifies the free tier: P0 does not file bug reports — P0 churns
silently. S1 files a detailed issue with logs at 2am, for free.** Your self-hosted base
is an unpaid QA department. Gate a feature and you stop receiving defect reports about
it, then learn it was broken when someone leaves without telling you.

**The counter-rule:** S1's feedback is high-signal on *correctness* and near-worthless on
*priority*. Weight their bug reports at 100%, their feature requests at close to zero
unless P0 wants the same thing.

**Never sell, at any tier:** user data, advertising, advisor lead-generation on your
users, or anonymised aggregate financial data. Each converts a trust business into a data
business. This audience detects it, and there is no version of that trade that pays.

---

## 11. What to do Monday

1. **Incorporate.** Nothing else on this list should precede it, and Catalyst can't ship
   without it.
2. **Start H1 with `asset_valuations`** — one week, fixes a defect live on the public
   demo, and it's the prerequisite for holdings and entities if those ever come.
3. **Ship the Catalyst $25 tier and the feedback channel.** One Stripe link and one
   public repo with issues only. These are your two instruments; without them the launch
   produces anecdotes.
4. **Build the update check and opt-in telemetry to the §7.5 spec** — disclosed payload,
   inspectable in Settings, bucketed counts, no financial values, ever.
5. **Add the "Desktop app — want this?" signup to the landing page** before launch, so the
   people who can't run Docker self-identify instead of bouncing invisibly.
6. **Rewrite the landing-page pitch (PRD F14) around §8.1** — you sell the answer to a
   decision, and the trust artifact is the byte-identical export guarantee. Put both on
   the page.
7. **Write the §7.6 targets down and date them.** A gauge with no threshold runs forever.

---

_Document created 2026-09-05. Living document — revise as market facts or product state change, and log every revision above._
