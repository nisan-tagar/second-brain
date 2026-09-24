| Field            | Value                                      |
| ---------------- | ------------------------------------------ |
| **Created**      | 2026-09-05 14:00 UTC                       |
| **Last Updated** | 2026-09-24 v3.0                            |
| **Version**      | 3.0                                        |
| **Status**       | Decided — v3.0 is the committed plan       |
| **Author**       | Business development / market research session |
| **Related**      | PRD v2.23 and TDD — both now in the app repo at `docs/project/`. Code state: `origin/main` @ v6.1.0 (`79250cd`, 2026-09-21) |

### Change Log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-09-05 | Initial market analysis. Industry scan of personal-finance, self-hosted-finance, family-office and SMB-cashflow categories. Verdict on family-office and SMB fit (both rejected as primary segments, with a narrow qualified wedge defined for each). Feature gap analysis ranked by revenue proximity vs. build cost. GTM recommendation built around a free self-hosted tier, with a structural correction to PRD F18.4 (hosting is the primary revenue line, not an add-on to AI) and a challenge to the closed-source decision. |
| 1.1 | 2026-09-05 | **ICP/segmentation (§8) and a customer-anchored feature roadmap (§9) added** — the substance of this revision. Two corrections to v1.0 driven by new evidence. (a) **The beachhead is re-specified.** v1.0 named cross-border/multi-currency households as the beachhead on a "nobody serves this person" claim; that claim is wrong — a cohort of 2025–26 entrants (Borderless Budget, FlowFund, Tallyroot, Auritrack, Monavio) targets exactly this person, and Lunch Money owns the niche natively at a $60/yr pay-what-you-want minimum. Multi-currency is therefore re-classified as a **targeting filter and defensibility moat, not the value proposition** — it is the cheapest axis in the market. (b) **The value proposition is re-specified as planning, not budgeting**, on two findings: planning tools price 30% above full PFM suites (ProjectionLab $129/yr, $1,199 lifetime, $549/yr advisor; Boldin $144/yr — for a simulator with no ledger underneath), and the category's documented #1 churn cause is apps that "show data without producing behavior change" and fail to connect cashflow to a plan. Goaldy's shipped Plan simulator (PRD F8) is the only asset that addresses that, and no self-hosted competitor has one. Consequent changes: Goaldy Cloud repriced from $96/yr to $120–144/yr with a lifetime option added; the roadmap is re-ordered by **retention risk** rather than feature appeal (manual-entry apps churn at 3x the rate of auto-sync apps); Goaldy.AI moves from Phase 2 to the **last** horizon. |
| 1.2 | 2026-09-05 | **§10 added: licensing, hosting-vs-self-hosting economics, and the value-exchange model** — who revenue comes from versus who insight, defect discovery and credibility come from, treated as two separate transactions with different currencies. Licensing resolved with a recommendation rather than options: **AGPL-3.0 for the core with a separately-licensed proprietary Pro package (open core), DCO rather than CLA.** Reasoning turns on the fact that AGPL's one real weakness — it does not stop a competitor hosting your software — describes a risk that effectively does not exist for a trust-and-support-bound niche PFM, while its benefits (main awesome-selfhosted listing rather than `non-free.md`, OSI credibility with the distribution segment, and community-contributed bank/institution import adapters) attack the one problem money and coding agents both handle badly: long-tail institution coverage, which requires real accounts at real institutions to test. FSL-1.1 documented as the fallback if hosting rights must be protected outright. **H5 re-slotted** in light of the AI/MCP work already being delivered by a coding agent: collapsing build cost is the argument *against* monetizing that layer, not for it — a feature an agent produces in a week is reproducible by every competitor's agent and cannot hold a price. Chat and MCP therefore ship **free with BYO API key** as a free-tier differentiator, launch headline and discovery channel (MCP registries as distribution), with only *managed inference* metered. Also documents the per-tenant-SQLite hosting cost advantage (85–95% gross margin, and an exit promise that is literally a file copy) and the trust obligations that hosting creates. |
| **2.0** | **2026-09-10** | **Strategy reversed to self-hosted-first; §7, §9, §10 and §11 rewritten.** v1.0–v1.2 built toward a managed hosted service as the primary revenue line. That is now **deferred, not planned**, on three grounds: hosting makes you a controller of financial personal data with duties that `AS IS` cannot disclaim (Israel PPL Amendment 13 in force 14 Aug 2025, GDPR breach notification, DPIA, EU region choice); no one has yet paid for Goaldy at all, so building the infrastructure and legal wrapper for a subscription business inverts the risk; and adding services later is cheap while retreating from them is not. Four substantive reversals. (a) **Licensing: closed source, reversing v1.2's AGPL recommendation** — the AGPL case rested on needing contributors for long-tail bank adapters, but Obsidian demonstrates a plugin API delivers that ecosystem without source release, and `IMPORT_ADAPTERS` is already shaped as that boundary; trust comes from an open data format (the byte-identical backup/restore guarantee), not readable code, as Obsidian, Plex, Unraid and Blue Iris all demonstrate. (b) **Aggregation needs no server** — the app is already a server on the user's machine and can call SimpleFIN/GoCardless directly, so the single highest-impact anti-churn feature costs zero infrastructure, zero held credentials and zero support liability. (c) **REST API and MCP move to the free tier permanently** — gating them is unenforceable against a local SQLite file, they *are* portability, and MCP is both commoditized (Kubera, Era) and a discovery channel. (d) **"Goaldy Connect" retired as a concept** for bundling four services of very different risk profiles under one name; products are now named by what they cost you (Pro holds nothing, Sync holds ciphertext, Managed Accounts holds bank tokens). New in §7.5: a **telemetry and signal design** — update check on by default and disclosed, usage telemetry strictly opt-in with a published payload, bucketed counts and no financial values, plus the explicit warning that opt-in telemetry is selection-biased toward enthusiasts and its D30 figure is an upper bound rather than a measurement. §7.6 sets a dated 90-day decision point. §9 collapses five horizons into one, with the next chosen by observed demand. §10 reframes liability as the cost each deferred service would spend. §§1–6 and §8 (industry map, segment verdicts, ICP) are unchanged and still stand. |
| **3.0** | **2026-09-24** | **Re-cut against the shipped product at v6.1.0; §9 rewritten, §12 and §13 added, §0, §8.2 and §11 updated.** v2.0 was written when the product was mid-build and assumed product quality was the constraint. It is not, and has not been since roughly 2026-09-17. Six major versions shipped in six weeks: `asset_valuations` with dated history (and `reported_value`/`balance_source` **dropped from the schema entirely**), the monetary-representation fix to integer minor units, Net Worth as a real screen with a change waterfall closing at a zero residual, the metrics engine with null-with-a-reason semantics, Trends, value accounts, loans with principal/interest decomposition, household multi-user auth with TOTP MFA, and **AI chat with BYO key plus an MCP server exposing ~18 agent tools** — the whole of what PRD F18 once proposed to sell, shipped free, as v2.0 recommended. **Seven of v2.0's twelve H1 items are done; every one of the five that remain is a distribution artifact, not a feature** — no `LICENSE` file after six major versions, no telemetry of any kind, a landing page untouched since 2026-08-22 that advertises none of the above, and an onboarding route that renders the literal words *"coming soon"* to every new installer. §9 is therefore restructured from one feature horizon into **three sequential gates** (G0 Launchable · G1 Stays installed · G2 Worth paying for), each with a measured exit criterion; the D30 target is revised **down** from 500 instances to 250, because v2.0's figure assumed a desktop app and a wider funnel than a Docker-only launch has. **§12 (new) is the financial model**: Pro repriced from §7.2's "$99–149 one-time, or $59/yr" to **$119/yr or $399 lifetime** on the ProjectionLab evidence ($129/yr for a simulator with no ledger under it); realistic blended payment fees of ~5% rather than 3.5% for an Israeli entity selling globally; a **break-even of 31–53 paying users**; and three scenarios whose honest conclusion is that the base case is break-even rather than a salary, and that **$10k MRR is structurally unreachable on self-hosted licences** (~1,062 paying users ≈ 15,000 active instances ≈ Firefly III's scale after a decade). That names the real argument for hosting correctly: it is **TAM, not margin** — self-hosted wins on margin — and it is a decision about reach, not about Postgres. **§13 (new) prices the business model rather than the category**, with comparables inside personal finance (ProjectionLab, Kubera, Lunch Money, Ghostfolio, and Firefly III/Actual Budget as the cautionary tale) and — more usefully — outside it (Home Assistant/Nabu Casa as the model to copy, Plex's lifetime pass rising $119→$249→$749.99, Unraid, Obsidian, Sublime Text, Bitwarden, Tailscale). Also records the rejection of an externally-proposed $39–49/yr subscription model: it described a per-workspace multi-tenant architecture Goaldy does not have, targeted envelope budgeting (a PRD §3 non-goal) and non-technical users (who cannot run Docker), relied on SimpleFIN pass-through economics that do not cover Israeli banks, and omitted data-controller liability entirely. §§1–7, §8.1, §8.3–8.6 and §10 are unchanged and still stand. |


---

## 0. Executive verdict (read this if you read nothing else)

**The plan, in one sentence: the product is built — now ship a licence file, a landing
page, telemetry and an onboarding screen, and let 250 measured installs decide what gets
built and hosted next.**

> **v3.0 update.** v2.0 assumed the constraint was product quality. At v6.1.0 it is not.
> The product is differentiated on an axis no self-hosted competitor occupies — §1.2a's
> refusal to report a figure it cannot substantiate, a net-worth waterfall closing at a
> zero residual, and a Plan simulator. **Every remaining blocker is a distribution
> artifact and none of them is engineering.** See §9.0 for the evidence and §12 for what
> the arithmetic says it is worth.

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

> **v3.0 note.** This table prices the *category*. §13 prices the *business model*, which is
> the question that actually applies to Goaldy — and its best analogues are outside personal
> finance. Prices below re-verified 2026-09-24.

| Product | Price | What it sells |
|---|---|---|
| Ghostfolio (cloud Premium) | **$48 one-time** | Self-hosted portfolio tracking |
| Lunch Money | **$60/yr** min (PWYW) | Multi-currency budgeting, indie/bootstrapped |
| Copilot | $95/yr | Design-led PFM, iOS only |
| Monarch Core / Plus | $99.99 / $199 per yr | Full aggregated PFM + household collaboration |
| YNAB | $109/yr | A budgeting *method* |
| **ProjectionLab** | **$129/yr · $1,199 lifetime · $549/yr advisor** | **A planning simulator with no ledger under it** |
| **Boldin** | **$144/yr** | Retirement planning + Monte Carlo |
| Kubera / Kubera Black | $249 / $2,499 per yr | Net worth + entity nesting + MCP (verified 2026-09) |
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

## 9. The roadmap — where the product actually is, and the three gates ahead

> **Rewritten in v3.0 against `origin/main` at v6.1.0 (commit `79250cd`, 2026-09-21).**
> v2.0's H1 was a twelve-item list written when the product was mid-build. Seven of those
> items are now shipped, several things nobody planned shipped alongside them, and the
> bottleneck has moved. This section is re-cut against the code, not against the previous plan.

### 9.0 State of play — what v2.0's H1 asked for, and what happened

| v2.0 H1 item | Status on `main` @ v6.1.0 | Evidence |
|---|---|---|
| 1. `asset_valuations` time series | ✅ **Shipped, and then some** | `asset_valuations` table with `UNIQUE(account_id, as_of_date)`; `balance_source`/`reported_value` **dropped from the schema entirely** (`a1c95ce`) — one as-of clause, no scalar left to overwrite |
| 2. BYO-aggregator ingestion (SimpleFIN/GoCardless) | ❌ **Not built** | Only `integrations/moneyman` exists |
| 3. OFX/QIF import | 🟡 **QIF yes, OFX no** | `FORMAT_READERS` covers csv/xlsx/qif/json; presets for Mint, YNAB, Buxfer |
| 4. Recurring / upcoming detection | ❌ **Not built** | PRD F6.4 still Planned |
| 5. Plan promoted to first-class | 🟡 **Fixed, not promoted** | v2.21/v2.22 fixed Add Goal (broken for every realistic amount) and per-item currency — still a nav item, not the pitch |
| 6. Reports stub retired | ✅ **Shipped as Trends** | F6.5; `/reports` redirects, nav entries deleted |
| 7. Onboarding + starter tag tree | ❌ **Still a live stub** | `app/(app)/onboarding/page.tsx` renders the literal words *"5-step onboarding — coming soon (F15.3)"* |
| 8. Encrypted scheduled backup | ❌ **Not built** | PRD F13.3 Planned |
| 9. Update check + opt-in telemetry | ❌ **Not built** | Zero instrumentation in the repo |
| 10. Public feedback channel | ❌ **Not built** | PRD F13.2 Planned |
| 11. Catalyst supporter tier | ❌ **Not built** | No `LICENSE` file either |
| 12. PWA installability | ❌ **Not built** | No manifest, no service worker |

**Shipped that no roadmap asked for** — and this is the more important half:

- **Net Worth as a real screen** — composition, provenance/freshness marks, and a change
  waterfall closing at a **residual of exactly zero** on the reference household.
- **A metrics engine** (F19) with null-with-a-reason semantics and sample-sufficiency levels.
- **Trends** — period-over-period and year-over-year, with delta and trend as separate columns.
- **Value accounts and loans** — basis derived from type, principal/interest decomposition
  from the lender balance, payoff estimates that refuse rather than guess, property↔mortgage
  equity links, cross-currency transfer pairing.
- **§1.2a — the substantiation-refusal rule.** The single most valuable thing in the product.
- **Household multi-user auth** — `users`, bcrypt, TOTP MFA, recovery codes, invite flow.
- **AI chat with BYO API key and an MCP server exposing ~18 agent tools** — the entirety of
  what v1.2's PRD F18 proposed to *sell*, shipped free, which v2.0 called correctly.
- **Monetary representation fixed** — every money column is now `INTEGER` minor units with a
  `typeof(...) = 'integer'` check; `transaction_tags.amount_minor` makes splits sum exactly.

**The finding.** v2.0 said the constraint was product quality. It isn't any more. Six
major versions shipped in six weeks and the product is now differentiated on an axis no
self-hosted competitor occupies. **Every remaining blocker is a distribution artifact, and
none of them is engineering.** No `LICENSE`. No telemetry. A landing page last touched
2026-08-22 that advertises none of the above. An onboarding route that says "coming soon"
to the first stranger who installs it.

### 9.1 The shape of the roadmap now

Three gates, strictly sequential. **A gate is not a phase — you do not start the next one
until the previous one's exit criterion is measured, not felt.**

---

### G0 — "Launchable" · 2–3 weeks · the only thing that matters

Nothing here is a feature. Everything here is the difference between software that exists
and software that can be found, installed, measured and paid for.

| # | Item | Size | Why it blocks everything |
|---|---|---|---|
| 1 | **`LICENSE` file** | 1 day | Six major versions shipped with no terms at all. Closed source is not a decision you have made; it is a decision you have not written down. Reversible toward permissive, irreversible the other way — so write the restrictive one now. |
| 2 | **Kill the onboarding stub** | 3–5 days | `onboarding/page.tsx` says *"coming soon"* to every new installer. A visible stub reads as abandonware on the one screen where you get one chance. Even redirecting to a tag-tree picker beats it. |
| 3 | **Localized starter tag tree** | 2 days | Empty state is where installs die before D1. En + He. |
| 4 | **Update check** (default-on, disclosed, disableable) | 2 days | Without it the launch produces anecdotes. Version + random install id + a timestamp. Nothing else. |
| 5 | **Opt-in usage telemetry** to the §7.5 spec | 3 days | Bucketed counts, published payload, inspectable in Settings, no monetary values ever. Selection-biased upward — treat its D30 as a **ceiling**, not a measurement. |
| 6 | **Public feedback channel** | 1 day | Closed source means no Issues tab. The defect reports *are* the free tier's price. |
| 7 | **Landing page rewritten against §8.1** | 3 days | Currently sells "self-hosted personal finance" and mentions none of Net Worth, Trends, valuations, loans, metrics or the refusal rule. Lead with the answer-to-a-decision, and put the byte-identical export guarantee on the page as the trust artifact. |
| 8 | **Demo that doesn't cold-start** | 1 day | A Render free instance takes 30–50s to wake. That is your first impression, and it is currently a spinner. ~$7/mo fixes it. Cheapest CAC reduction available. |
| 9 | **Catalyst $25 one-time** | 1 day | One Stripe link. The only pre-revenue instrument that measures **willingness to pay** rather than curiosity. Needs incorporation first. |

**Exit criterion (measured, dated):** 90 days after G0 ships — **250 instances reporting at
D30** and **20 Catalyst supporters**. Revised down from v2.0's 500/25, because v2.0's number
was set against an assumed desktop app and a wider funnel than a Docker-only launch has.

**Miss both and the problem is positioning, not features. Re-open this document instead of
building.**

---

### G1 — "Stays installed" · 4–6 weeks · only after G0's numbers land

The ordering principle is unchanged and still the best-evidenced thing in this document:
**manual-entry apps churn at ~3x the rate of auto-sync apps, and category D30 retention
averages 38%.** Sequence by churn removed.

| # | Item | Size | Churn mechanism it attacks |
|---|---|---|---|
| 1 | **In-app BYO-aggregator ingestion** — SimpleFIN token pasted into the instance; GoCardless for EEA/UK | 2 weeks | The 3x multiplier, directly. Per §7.3 it needs **no server of yours**: the instance is already a server on the user's machine and calls the aggregator itself. Zero tokens held, zero COGS, zero broken-bank tickets that are yours. |
| 2 | **An Israeli path that isn't Moneyman-only** | 1 week | SimpleFIN does not cover Israeli banks. Your beachhead is the market your flagship aggregation story doesn't reach — this is the honest gap in the plan and it needs naming, not hiding. |
| 3 | **Recurring / upcoming-cashflow detection** | 1 week | The most-cited "wow" in Monarch and Copilot reviews. Pure logic over existing data, reusing the rules engine. |
| 4 | **Plan promoted to the pitch** | 1 week | The only asset no self-hosted competitor has, and the answer to the #1 documented churn cause. It is a nav item. It should be the dashboard, the onboarding destination and the landing-page hero. |
| 5 | **Encrypted scheduled backup** | 1 week | "Own your data" is a liability until it's "safely." |
| 6 | **PWA installability** | 2 days | Two files. Removes the "no mobile" objection at near-zero cost. |
| 7 | **OFX import** | 3 days | The migration path off Quicken. |

**Exit criterion:** **D30 instance retention ≥ 50%** (category average is 38% — beating it
is the whole thesis) and ≥ 20 substantive defect reports from people who are not you.

---

### G2 — "Worth paying for" · sized when reached · shape decided by demand, not here

Only enter G2 with G1's retention number in hand. The candidates are already specified in
§7.2 and are listed here **in the order the evidence says they'll be demanded**, not in the
order they're interesting to build:

1. **Pro, local and licence-gated** — entities/ownership, roles (partner, accountant
   read-only, advisor read-only), audit log, document attachments, accountant export pack.
   Note the multi-user auth work already shipped: **roles are now the cheapest paid feature
   on the list**, because the hard part (`users`, MFA, invites) is done.
2. **Desktop app** — if the waitlist fills. Already architecturally ready:
   `output: 'standalone'` and `serverExternalPackages: ['better-sqlite3']`.
3. **E2EE Sync** — if two-device complaints appear. You store ciphertext you cannot decrypt.
4. **Managed accounts / hosting** — only if loud *and* fundable *and* after a lawyer.

**Permanently out of scope** (unchanged): AR/AP/invoicing, VAT/tax filing, payroll,
statutory double-entry, tax optimization, bank-credential proxying, trade execution,
capital calls, a native mobile app, multi-tenancy, and anything requested primarily by S1
that P0 would never use.


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

> **Rewritten in v3.0.** Items 2 and 3 of the v2.0 list are done (`asset_valuations`
> shipped and went further — the scalar is gone from the schema). The list below is G0
> (§9.1) in the order that unblocks the most.

1. **Write the `LICENSE` file.** One day. Six major versions have shipped with no terms.
   Closed source is not a decision you have made — it is one you have not written down.
   Reversible toward permissive, irreversible the other way, so write the restrictive one.
2. **Incorporate.** Catalyst cannot take a shekel without it, and it gates items 8 and 9.
3. **Delete the onboarding stub.** `app/(app)/onboarding/page.tsx` currently renders
   *"5-step onboarding — coming soon (F15.3)"* to every stranger who installs Goaldy. Even a
   redirect into the starter tag-tree picker beats a visible "coming soon" on day one.
4. **Build the update check**, default-on and disclosed: version, a random install id, a
   timestamp. Nothing else. Without it the launch produces anecdotes instead of data.
5. **Then opt-in telemetry to the §7.5 spec** — published payload, inspectable in Settings,
   bucketed counts, never a monetary value. Treat its D30 as a **ceiling**, not a measurement.
6. **Rewrite the landing page (PRD F14) around §8.1.** It has not been touched since
   2026-08-22 and advertises none of Net Worth, Trends, valuations, loans, metrics or the
   §1.2a refusal rule — the six things that make Goaldy differentiated rather than another
   budgeting app. Put the byte-identical export guarantee on the page as the trust artifact.
7. **Pay for the demo instance.** ~$7/mo removes a 30–50 second cold start from your first
   impression. It is the cheapest CAC reduction on this list by an order of magnitude.
8. **Open the feedback channel.** Closed source means no Issues tab, and the defect reports
   are the entire price the free tier charges.
9. **Ship Catalyst at $25.** One Stripe link. The only instrument that measures willingness
   to pay rather than curiosity.
10. **Write §9.1's G0 exit criterion down and date it**: 250 instances at D30 and 20
    Catalyst supporters, 90 days out. A gauge with no threshold runs forever.

**Not on this list, deliberately:** Postgres, open-sourcing, multi-tenancy, hosting, a
pricing page for Pro, and any further feature work. Per §12.6, revenue moves ~8× on install
count and only ~2.5× on conversion rate — **distribution is the variable**, and every hour
spent on pricing before item 6 ships is an hour spent on the wrong term of the equation.

---

## 12. The financial model — built on what is actually shipped

> **New in v3.0.** Supersedes every pricing figure elsewhere in this document, including
> §7.2's "$99–149 one-time, or $59/yr" for Pro, which was written before the product had
> Net Worth, the metrics engine, loans, valuations or MFA and is now too low.
>
> *Not accounting advice. Israeli incorporation form, VAT (including EU VAT OSS on B2C
> digital sales) and the deductibility of everything below require an accountant before the
> first sale.*

### 12.1 The pricing decision, resolved

| Tier | Price | What it is | Enforcement |
|---|---|---|---|
| **Free** | $0 forever | The complete application, uncrippled: accounts, transactions, tags, budgets, rules, Cashflow/Net Worth/Trends, the Plan simulator, valuations, loans, metrics, full export/backup/restore, **REST API, MCP, AI chat with BYO key** | n/a |
| **Catalyst** | **$25 one-time** | A badge, early builds, a private channel. Zero COGS. | Honour |
| **Pro** | **$119/yr, or $399 lifetime** | Entities/ownership, roles (partner · accountant read-only · advisor read-only), audit log, document attachments, accountant export pack, scheduled reports | Signed offline-verifiable licence token |

**Why $119 and not $49.** $49 prices Goaldy as a budgeting app, which is the one thing it
is not. [ProjectionLab](https://projectionlab.com/) charges **$129/yr and $1,199 lifetime**
for a planning simulator *with no ledger underneath it* — no transactions, no tags, no
budgets, no import, no multi-currency accounts. Goaldy has all of that plus a Plan
simulator plus a net-worth waterfall that closes at zero residual. Pricing below
[Lunch Money](https://lunchmoney.app/)'s $60 floor — which has real bank aggregation you
do not — means undercutting on price, and undercutting is the strategy that demands the
most marketing spend per dollar of revenue. That is the one thing a bootstrapper cannot buy.

**Why lifetime at 3.35× annual.** The self-hosted audience specifically resents
subscriptions — that is what made [Unraid](https://unraid.net/blog/new-pricing)'s 2024
shift controversial. Lifetime is a cash-flow accelerator when you have no runway, and a
trust signal to exactly this buyer. [Plex](https://www.plex.tv/blog/new-lifetime-plex-pass-pricing/)
proves the downside is survivable: it raised its lifetime pass from $119 → $249 → **$749.99
in July 2026**, a 6× increase against an installed base that had already paid.

**Why free stays genuinely free.** Portability and correctness are never paywalled (PRD §3
non-goals). Gating the REST API fails on its own terms — the ledger is a local SQLite file,
so anyone technical enough to want the API can open it with `sqlite3`.

### 12.2 Unit economics — and why self-hosting beats hosting on margin

Gemini's model projected **88–92% gross margin** for a hosted product at $0.10–0.30/user/month
of compute. That model argued *for* the architecture with the **worse** margin. Self-hosted
licence revenue has no per-user compute at all:

| Line | Hosted (Gemini's model) | **Self-hosted licence (the plan)** |
|---|---|---|
| Compute + DB per user | $0.10–0.30/mo → $1.20–3.60/yr | **$0** — it runs on their hardware |
| Aggregation | Passed through | **Passed through** (user pays SimpleFIN directly) |
| Payment processing | ~3.5% claimed | **~4.5–5.5% realistic** — see below |
| Support | Yours, unbounded | Yours, but no infra incidents |
| **Gross margin** | 88–92% | **≈ 94–95%** |

**On the payment-fee figure.** Gemini's "13% on micro-monthly" is inflated: Stripe is
2.9% + $0.30, which is 8.9% on $5/mo and 10.4% on $4/mo — you'd need a ~$2.10 charge to
reach 13%. Its annual figure (~3.5% on $49) is right *domestically*. For an Israeli entity
selling globally, add ~1.5% for international cards and ~1% for currency conversion, so
model **~5%**, not 3.5%. The conclusion — **bill annually** — is correct anyway, and for
better reasons than fees: cash up front and a churn decision deferred twelve months.

**Net per Pro user:** $119 × ~0.95 = **≈ $113/yr**.

### 12.3 Fixed costs — the denominator that actually matters

| Line | Annual | Note |
|---|---|---|
| Incorporation + accountant | $2,000–4,500 | Dominant line. Required before Catalyst can take money. |
| Legal — ToS, privacy policy, licence terms | $1,500–4,000 one-time | One-time; amortize over 3 years ≈ $500–1,300/yr |
| Demo instance (no cold start) | $100–150 | ~$7–12/mo. **The cheapest CAC reduction available** — a 30–50s spinner is your first impression today. |
| Domain + landing page | $50 | |
| Licensing service (Stripe + entitlement tokens) | $60–120 | Tiny; holds no financial data by design |
| Code signing (only if desktop ships) | $400–500 | Apple $99 + Windows EV cert |
| **Total, year 1, no desktop** | **≈ $3,500–6,000** | |

### 12.4 Break-even — the only number worth memorising

> **$3,500–6,000 ÷ $113 = 31–53 paying Pro users.**

That is the entire bar. It is reachable. Every scenario below is measured against it.

### 12.5 Three scenarios, 12 months after G0 ships

Conversion assumptions are anchored on the OSS free→paid band (0.3–1% mass market, 1–3%
enterprise, 3%+ exceptional), adjusted **upward** because Goaldy's ICP is narrow and
qualified rather than mass market, and the paid tier targets a real pain (roles, entities,
an accountant view) rather than convenience.

| | **Bear** | **Base** | **Bull** |
|---|---|---|---|
| Installs, year 1 | 600 | 2,500 | 8,000 |
| D30 retention | 30% | 40% | 45% |
| Active instances | 180 | 1,000 | 3,600 |
| Pro conversion | 2% | 5% | 8% |
| **Paying users** | **4** | **50** | **288** |
| Gross revenue | $476 | $5,950 | $34,272 |
| Net after fees | $452 | $5,652 | $32,558 |
| **vs. break-even** | ❌ **−$4,500** | ✅ **≈ break-even** | ✅ **+$27,000** |

Year 2, with compounding installs and a mature Pro tier:

| | **Bear** | **Base** | **Bull** |
|---|---|---|---|
| Active instances | 350 | 6,000 | 15,000 |
| Pro conversion | 3% | 7% | 10% |
| **Paying users** | **11** | **420** | **1,500** |
| Net revenue | $1,244 | **$47,481** | **$169,575** |

### 12.6 What this model says that nobody wants to hear

**1. The base case is break-even, not a salary.** Year 1 base is ~$5,652 against ~$5,000 of
cost. This is a profitable side product, not a company, and it does not become one until
year 2 even in the bull case. Decide *now* whether that is acceptable, because the strategy
is correct for a side product and wrong for a venture.

**2. $10k MRR is structurally unreachable on self-hosted licences.** $120k/yr ÷ $113 =
**1,062 paying users**. At a 7% conversion that needs **~15,000 active instances** — Firefly
III's scale after a decade, with a decade's head start and open-source distribution. Selling
licences to self-hosters cannot get there on any plausible timeline.

**3. Which makes hosting the only path to a salary — and names its real cost correctly.**
The argument for hosting is **not margin** (self-hosted wins on margin, §12.2). It is **TAM**:
a hosted tier at ~$180/yr reaches the enormous non-technical majority who will never run
`docker compose up -d`. That audience is 10–100× the self-hosted one. The price of reaching
it is becoming a **data controller** of financial personal data under GDPR and Israel's PPL
Amendment 13 — breach notification, DPIA, administrative fines, database-suspension powers,
none of it disclaimable by an `AS IS` licence (§10).

**That is the actual trade, and it is a trade about reach, not about databases.** Postgres
versus SQLite is an implementation detail downstream of it. Do not make it before G1's
retention number exists.

**4. The sensitivity that dominates everything.** Revenue moves ~8× between bear and base on
**install count**, and only ~2.5× on conversion rate. **Distribution is the variable.** Every
hour spent on pricing strategy before the landing page is rewritten is an hour spent on the
wrong term of the equation.


---

## 13. Comparables — what to copy, and from whom

> **New in v3.0.** §8.2 prices the *category*. This section prices the *business model*,
> which is a different question and the one that actually applies to Goaldy — because
> Goaldy's shape (closed-source, self-hosted, single-tenant, sensitive data, solo founder)
> has better analogues outside personal finance than inside it.

### 13.1 Inside personal finance

| Product | Model | Price | What it proves for Goaldy |
|---|---|---|---|
| **[ProjectionLab](https://projectionlab.com/)** | Freemium SaaS, small team | **$129/yr · $1,199 lifetime · $549/yr advisor**; free Basic tier | **The single most important datapoint in this document.** It charges more than Monarch for a simulator with *no ledger, no transactions, no import, no multi-currency accounts*. Goaldy has all of that **plus** a Plan simulator. If PL sustains $129, Goaldy's $119 is conservative. |
| **[Kubera](https://www.kubera.com/)** | Annual-only, no free tier | **$249/yr**, Black **$2,499/yr** | The ceiling for net-worth tracking, and proof the HNW/family-office tier is already occupied (§3). Also proves no free tier is survivable when the value is obvious. |
| **Lunch Money** | Indie, bootstrapped, PWYW | **$60/yr** minimum | The floor. Multi-currency native, real aggregation. Pricing under it means competing on price against a product with a feature you lack. |
| **Monarch / YNAB / Copilot** | Subscription PFM | $99.99–$199 / $109 / $95 per yr | The mass market. All spend heavily on marketing and aggregation fees. Not Goaldy's game — see §13.3, lesson 6. |
| **Ghostfolio** | **OSS self-hosted + cheap cloud** | $0 self-host · **$48 one-time** cloud | The closest *structural* analogue inside finance. Also a warning: $48 one-time for hosted means it funds a side project, not a founder. |
| **Firefly III · Actual Budget · Beancount** | OSS, no paid tier at all | **$0** | **The cautionary tale, and Goaldy's default outcome if G0 doesn't ship.** A decade of work, the largest installed bases in self-hosted finance, and essentially no revenue. Building excellent free software is not a business model — it is a hobby with users. |

### 13.2 Outside personal finance — the more useful half

| Product | Model | Price | Lesson |
|---|---|---|---|
| **[Home Assistant / Nabu Casa](https://www.nabucasa.com/)** | OSS self-hosted core, paid cloud for what self-hosting is *bad at* | **$6.50/mo · $65/yr** | **The single best model to copy.** It doesn't sell the software or gate features — it sells remote access and voice, the two things that are genuinely painful to self-host. Revenue funds full-time staff on the free core. **Goaldy's equivalents: E2EE sync, scheduled encrypted backup, and a relay for mobile access.** Sell the thing self-hosting is bad at, never the software. |
| **[Plex](https://www.plex.tv/plans/)** | Closed source, self-hosted server, lifetime pass | **$749.99 lifetime** (was $119 → $249 → $749.99 in July 2026) | Closed-source self-hosted is a durable business, and **prices go up**, not down. A 6× lifetime increase against a locked-in installed base. Price Goaldy's lifetime tier knowing you can raise it later. |
| **[Unraid](https://unraid.net/blog/new-pricing)** | Closed source, one-time licence + paid updates | **$49 / $109 / $249** one-time, **$36/yr** update extension | The self-hosted audience **will** pay a licence and **resents** pure subscription — the 2024 shift to annual extensions was contentious. Validates Goaldy's lifetime option and the licence-token approach. |
| **[Obsidian](https://obsidian.md/pricing)** | Closed source, free personal, paid sync | Sync from **$4/mo**; commercial licence **$50/yr — made optional in Feb 2026** | Two lessons. (a) **A plugin API delivers an ecosystem without releasing source** — Goaldy's `IMPORT_ADAPTERS` registry is already that boundary. (b) Obsidian *dropped* its commercial licence requirement, which says gating by user *type* is weak; gating by *capability* is what holds. |
| **Sublime Text** | Closed, honour-system licence | **$99 / 3 years**, unlimited free evaluation | **Imperfect enforcement is fine.** It ran profitably for years on a licence anybody could ignore. Directly answers the "Pro is bypassable" objection — Plex, Unraid and Blue Iris say the same. |
| **Bitwarden** | OSS, self-hostable, freemium | **$10/yr** personal · $40/yr family | Sensitive data + self-host + freemium works — **but only at enormous volume.** The $10 price is a *warning*, not a target: it requires millions of users to matter, and Goaldy will never have millions. Narrow and expensive, not broad and cheap. |
| **Tailscale** | Generous free tier, paid teams | Free 3 users/100 devices · **$6/user/mo** | The free tier converts on **collaboration**, not on limits. Goaldy's analogue is roles (partner, accountant, advisor) — which is why roles are the cheapest paid feature on the list now that `users` + MFA have shipped. |
| **Paperless-ngx · Immich** | OSS, donation-funded | **$0** | Same cautionary tale as Firefly III, in a different category. Enormous installed bases, no revenue, maintainer burnout as the recurring ending. |

### 13.3 The six lessons, stated plainly

1. **Sell what self-hosting is bad at, not the software itself.** Home Assistant's model. Sync, backup and mobile relay are Goaldy's version. Feature-gating a local app is the weaker play.
2. **Closed-source self-hosted is a real business and prices rise.** Plex, Unraid, Blue Iris, Sublime. The licence-file decision is not a risk; not making it is.
3. **Imperfect enforcement is not a reason to avoid charging.** Sublime Text ran profitably on the honour system for a decade.
4. **Free-with-no-paid-tier is the default failure mode of this exact category.** Firefly III, Actual Budget, Paperless-ngx, Immich. All beloved. All broke. **This is what happens to Goaldy if G0 never ships.**
5. **Narrow and expensive beats broad and cheap for a solo founder.** Bitwarden's $10/yr needs millions of users; ProjectionLab's $129/yr does not. Goaldy is structurally a ProjectionLab, not a Bitwarden.
6. **Never compete on price against a product with a feature you lack.** Lunch Money at $60 has bank aggregation Goaldy doesn't. Pricing at $49 picks that fight and loses it — which is precisely what the $39–49 proposal did.


---

_Document created 2026-09-05 14:00 UTC. Last updated 2026-09-24 (v3.0), re-cut against `origin/main` @ v6.1.0. Living document — revise as market facts or product state change, and log every revision above._
