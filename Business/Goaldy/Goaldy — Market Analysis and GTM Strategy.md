| Field            | Value                                      |
| ---------------- | ------------------------------------------ |
| **Created**      | 2026-09-05 14:00 UTC                       |
| **Last Updated** | 2026-09-05 21:15 UTC v1.2                  |
| **Version**      | 1.2                                        |
| **Status**       | Draft — for CEO decision                   |
| **Author**       | Business development / market research session |
| **Related**      | PRD v2.6 (esp. §2, F18), TDD §13            |

### Change Log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-09-05 | Initial market analysis. Industry scan of personal-finance, self-hosted-finance, family-office and SMB-cashflow categories. Verdict on family-office and SMB fit (both rejected as primary segments, with a narrow qualified wedge defined for each). Feature gap analysis ranked by revenue proximity vs. build cost. GTM recommendation built around a free self-hosted tier, with a structural correction to PRD F18.4 (hosting is the primary revenue line, not an add-on to AI) and a challenge to the closed-source decision. |
| 1.1 | 2026-09-05 | **ICP/segmentation (§8) and a customer-anchored feature roadmap (§9) added** — the substance of this revision. Two corrections to v1.0 driven by new evidence. (a) **The beachhead is re-specified.** v1.0 named cross-border/multi-currency households as the beachhead on a "nobody serves this person" claim; that claim is wrong — a cohort of 2025–26 entrants (Borderless Budget, FlowFund, Tallyroot, Auritrack, Monavio) targets exactly this person, and Lunch Money owns the niche natively at a $60/yr pay-what-you-want minimum. Multi-currency is therefore re-classified as a **targeting filter and defensibility moat, not the value proposition** — it is the cheapest axis in the market. (b) **The value proposition is re-specified as planning, not budgeting**, on two findings: planning tools price 30% above full PFM suites (ProjectionLab $129/yr, $1,199 lifetime, $549/yr advisor; Boldin $144/yr — for a simulator with no ledger underneath), and the category's documented #1 churn cause is apps that "show data without producing behavior change" and fail to connect cashflow to a plan. Goaldy's shipped Plan simulator (PRD F8) is the only asset that addresses that, and no self-hosted competitor has one. Consequent changes: Goaldy Cloud repriced from $96/yr to $120–144/yr with a lifetime option added; the roadmap is re-ordered by **retention risk** rather than feature appeal (manual-entry apps churn at 3x the rate of auto-sync apps); Goaldy.AI moves from Phase 2 to the **last** horizon. |
| 1.2 | 2026-09-05 | **§10 added: licensing, hosting-vs-self-hosting economics, and the value-exchange model** — who revenue comes from versus who insight, defect discovery and credibility come from, treated as two separate transactions with different currencies. Licensing resolved with a recommendation rather than options: **AGPL-3.0 for the core with a separately-licensed proprietary Pro package (open core), DCO rather than CLA.** Reasoning turns on the fact that AGPL's one real weakness — it does not stop a competitor hosting your software — describes a risk that effectively does not exist for a trust-and-support-bound niche PFM, while its benefits (main awesome-selfhosted listing rather than `non-free.md`, OSI credibility with the distribution segment, and community-contributed bank/institution import adapters) attack the one problem money and coding agents both handle badly: long-tail institution coverage, which requires real accounts at real institutions to test. FSL-1.1 documented as the fallback if hosting rights must be protected outright. **H5 re-slotted** in light of the AI/MCP work already being delivered by a coding agent: collapsing build cost is the argument *against* monetizing that layer, not for it — a feature an agent produces in a week is reproducible by every competitor's agent and cannot hold a price. Chat and MCP therefore ship **free with BYO API key** as a free-tier differentiator, launch headline and discovery channel (MCP registries as distribution), with only *managed inference* metered. Also documents the per-tenant-SQLite hosting cost advantage (85–95% gross margin, and an exit promise that is literally a file copy) and the trust obligations that hosting creates. |

---

## 0. Executive verdict (read this if you read nothing else)

1. **Family offices: no. Not a segment, a fantasy.** Goaldy has no securities/positions model, no cost basis, no performance calculation, no legal-entity or ownership layer, no capital-call/commitment tracking, no document vault, and no partnership accounting. Those five things *are* the family-office product. The gap is a second product, not a backlog. See §3.
2. **SMB: no, as "accounting". A narrow yes as "the owner's consolidated personal + business cashflow view."** Statutory accounting (AR/AP/VAT/payroll/double-entry/accountant export) is a moat you cannot cross, and QuickBooks/Xero own it. See §4.
3. **Your stated AI/MCP differentiator (PRD F18.1) has already been commoditized.** Kubera ships MCP today at $249/yr, Era ships MCP over aggregated accounts. "Harmonized system of record for agents" is now parity, and Goaldy has *worse* ingestion than both — no aggregator at all. See §5.
4. **The single highest-leverage missing feature is automated ingestion (SimpleFIN Bridge + GoCardless/PSD2), not AI.** Every competitor's #1 review complaint is manual CSV. This is a weeks-long build. See §6, Tier A.
5. **The self-host-only funnel cannot pay for itself, and the arithmetic is not close.** At an industry-standard 1% free→paid conversion, $10k MRR at $10/mo needs ~100,000 active self-hosters — larger than Firefly III's entire installed base after a decade. **Hosted must become the default CTA and the primary revenue line; self-hosting is your trust asset and top of funnel, not your product.** This inverts PRD F18.4. See §7.
6. **Closed source is a strategic error for the exact audience you're targeting.** You are asking r/selfhosted — the most paranoid software audience alive — to run an opaque binary over their complete bank history. Firefly III, Actual Budget and Ghostfolio are all open. See §7.4.
7. **You are not selling a budgeting app. You are selling an answer to a question the household is anxious about.** Planning tools price *above* full PFM suites — ProjectionLab is $129/yr and $1,199 lifetime for a simulator with no ledger under it; Monarch is $99.99 for a complete aggregated PFM. Meanwhile the documented #1 reason people abandon budgeting apps is that they "show data without producing behavior change" and never connect cashflow to a plan. **Your Plan simulator is the answer to the category's biggest churn problem, and no self-hosted competitor has one.** Lead with it. See §8.
8. **Multi-currency is your targeting filter and your moat — not your pitch.** Correcting v1.0: the cross-border niche is *not* empty. Borderless Budget, FlowFund, Tallyroot, Auritrack and Monavio all launched into it in 2025–26, and Lunch Money serves it natively at a $60/yr minimum — the cheapest price point in the market. Compete on multi-currency and you compete on the discount axis. Use it to *find and hold* customers; sell them the plan. See §8.
9. **Order the roadmap by retention risk, not feature appeal.** Apps requiring manual entry churn users at **3x** the rate of auto-sync apps; category D30 retention averages 38%. That single number settles the sequencing debate. See §9.
10. **The cheaper a feature is to build, the less it can be sold for.** Your coding agent delivering MCP and in-app chat in a week is the reason that layer cannot be the paid tier — every competitor's agent can do the same, and Kubera and Era already did. Ship it **free with BYO API key**; meter only managed inference. What stays defensible is what agents can't cheaply produce: long-tail institution coverage, accumulated user data and its switching cost, trust, and the unglamorous grind of running the hosting. See §10.
11. **Licence: AGPL-3.0 core + proprietary Pro package, DCO not CLA.** AGPL's one real weakness is a risk that doesn't exist here; its benefits attack the problem money can't solve. See §10.2.

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

## 7. Go-to-market

### 7.1 The arithmetic that should drive every decision
Open-source / self-hosted free→paid conversion benchmarks: **0.3–1%** for mass-market developer tools, **1–3%** for enterprise-leaning, 3%+ exceptional.

Run it at a *generous* 1%, targeting $10k MRR at $10/mo:
- 1,000 paying users → **100,000 active self-hosters.**
- That exceeds Firefly III's installed base after ~a decade of being the category default, free, open, and multilingual.

**Conclusion: a self-host-only funnel does not produce a business at any conversion rate you can realistically achieve.** Not "it's slow" — the numbers do not close.

### 7.2 The correction: invert PRD F18.4
Today the PRD treats a hosted instance as an "optional add-on" bundled with an AI subscription. That is backwards, and every comparable company demonstrates why: **Ghost, Plausible, Cal.com, n8n, GitLab, Discourse — all monetize managed hosting first, and everything else second.** Confluent reported 35% of $100k+ customers originated in the free tier; the free tier is a *funnel*, and the funnel has to terminate in something with a subscription attached.

Proposed structure:

| Tier | What | Price | Role |
|---|---|---|---|
| **Free — Goaldy Self-Hosted** | Everything in F1–F17. Unlimited. Forever. BYO AI key. | $0 | Trust asset + top of funnel + distribution. Never crippled. |
| **Goaldy Cloud** | Managed, backed up, updated, PWA, no Docker | **$12/mo, $120–144/yr** + **$449 lifetime** | **The primary revenue line.** Repriced upward in v1.1: you sell planning (ProjectionLab $129, Boldin $144), not budgeting (Monarch $99.99, Lunch Money $60). The lifetime option is copied from ProjectionLab deliberately — privacy-minded buyers self-select into it, and it solves a bootstrapper's cash-timing problem. |
| **Goaldy.AI** | Chat + MCP + Monte Carlo + proactive nudges. Usage-capped; BYO-key option. | **+$8–12/mo** | Attach-rate expansion on Cloud; license-key unlock for self-hosters (F18.5 architecture stands). |
| **Household Pro** (Phase 3) | Entities, roles, audit log, documents, advisor sharing, scheduled reports | **$25–40/mo** | §3.2 / §4.2 segments. 10x under Kubera Black. |

Self-hosters can buy Goaldy.AI and Household Pro via the license key. That keeps the promise ("nothing about where your data lives is ever paywalled") intact while giving self-hosters something to pay for.

### 7.3 Beachhead and sequencing
**Beachhead: cross-border / multi-currency households with a decision pending.** Full ICP definition in §8 — the short version is that multi-currency is how you *find and hold* them, and the Plan simulator is what they *pay for*.

Correcting v1.0, which claimed nobody serves this person: several 2025–26 entrants do (Borderless Budget, FlowFund, Tallyroot, Auritrack, Monavio), and Lunch Money serves it natively at 160+ currencies with historical rates for a $60/yr minimum. That cohort is simultaneously **evidence the pain is real** (five founders independently built for it) and **evidence it is commoditizing** (low barriers, thin products, race to the bottom on price). The defensible position is not "we do multi-currency too" — it is that none of them are self-hosted, none have a household planning simulator, none have real RTL/Hebrew, and none will ever hold an illiquid asset's valuation history.

**Sequence:**
1. **Close launch blockers** — A5 Reports, A6 onboarding, A7 backup, A3 OFX/QIF, A8 PWA, plus the public feedback channel (F13.2). ~4–6 weeks.
2. **Ship ingestion** — A1 SimpleFIN + A2 GoCardless + A4 recurring detection. This is what makes the product *demoable* rather than *explainable*. ~4–6 weeks.
3. **Stand up Goaldy Cloud in beta** before the public launch, with a waitlist on the landing page. Launching to HN with no hosted option wastes 90%+ of the traffic — those visitors will not install Docker, and you will never see them again.
4. **Ignition:** Show HN (Tuesday ~09:00 ET) → r/selfhosted (790k members; 98.3% run containers) → awesome-selfhosted PR → selfh.st newsletter → Product Hunt. README as a product page: one hero image, 5–7 functional GIFs. Avoid US holiday months. **Every CTA points at the hosted beta or the demo; the Docker command is the third link, not the first.**
5. **Beachhead campaign in parallel** — Hebrew landing page, Israeli community seeding, multi-currency as the headline claim, not a footnote.
6. **Then** Goaldy.AI (Q+1) and Household Pro (Q+2), sold into an existing base rather than cold.

### 7.4 The closed-source problem — resolve this before launch
You are asking the single most adversarial software audience in existence to run an **opaque binary over their complete financial history**, when the three incumbents in the category (Firefly III, Actual Budget, Ghostfolio) are all open source. This will be the top comment on your Show HN. It is not a hypothetical.

You also inherit the cost without the benefit: closed source means no GitHub Issues tab (the PRD already notes you need to build a feedback channel to compensate), no contributors, no trust signal, no awesome-selfhosted credibility — while the "protection" it buys is negligible, because nobody is going to out-execute you by forking a personal finance app.

**Recommendation: AGPL-3.0 the core, license-key the paid layers (Goaldy.AI, Household Pro), own the hosted service.** AGPL is precisely the license that permits self-hosting while deterring a competing hosted service — the same structural bet Ghostfolio and n8n (Sustainable Use License) make. If you're unwilling to go full OSS, go **source-available (BSL/FSL, converting to Apache after 3–4 years)**, which gets you most of the audit-ability trust at none of the competitive risk. What you should *not* do is ship a closed binary and hope the audience doesn't notice.

### 7.5 Metrics and kill criteria
Instrument these from day one (privacy-respecting, opt-in telemetry — an anonymous version-check ping, nothing about financial data):

| Metric | 90-day target post-launch | Kill / pivot signal |
|---|---|---|
| Active self-hosted instances | 2,000 | <500 → positioning is wrong, not the marketing |
| Hosted beta signups | 500 | <100 → the free tier isn't generating demand for convenience |
| Hosted → paid conversion | ≥8% of trials | <3% → the hosted value prop is too thin |
| Self-host → paid (any tier) | ≥0.5% | <0.2% at 12 months → self-host funnel is marketing spend, not a revenue channel; budget it as such |
| D30 retention of connected accounts (post-A1) | ≥40% | <20% → ingestion isn't sticking; the ledger is going stale and everything downstream is dead |
| Beachhead share of signups | ≥25% from IL/multi-currency | <10% → the beachhead thesis is wrong; re-pick before spending more |

**The one-year kill criterion for the paid business:** if 12 months post-launch you have not reached 100 paying subscribers across all tiers, the problem is not features or pricing — it is that self-hosted personal finance is a hobbyist category with structurally low willingness to pay, and the correct response is to go hosted-first and treat self-hosting purely as a marketing artifact.

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

## 9. The feature roadmap, ordered by retention risk

The ordering principle, stated once: **apps requiring manual entry lose users at 3x the rate of auto-sync apps, and category D30 retention averages 38%.** Features are therefore sequenced by how much churn they remove, not by how interesting they are to build.

### H1 — "Tell no lies" (next ~6 weeks)
*Theme: everything the product currently claims should be true. This is a credibility gate on the public launch, not a growth phase.*

| Item | Why it's here |
|---|---|
| **`asset_valuations` time series** | Fixes a live correctness defect: `reported_value` is a scalar overwritten in place, so illiquid assets chart as a flat line and editing a property value retroactively rewrites net-worth history. P0's core asset class is exactly the one currently modelled wrong. Add purchase price/date for appreciation % nearly free. |
| **Reports page** (retire the "Coming soon" stub) | A visible stub is read as abandonware by S1 on launch day |
| **Onboarding + localized starter tag tree** | Empty state is where the category loses people before D1 |
| **Encrypted scheduled backup** | "Own your data" is a liability until it's "safely" |
| **PWA installability** | Two files; removes the "no mobile app" objection |
| **Public feedback channel** | A closed distribution with no way to report a bug reads as dead |

**Exit criterion:** a stranger can install, import, and reach one true insight without hitting a stub or a wrong number.

### H2 — "Stop the bleeding" (~Q1 2027)
*Theme: remove the 3x churn multiplier. This is the single highest-ROI horizon in the plan.*

- **SimpleFIN Bridge** ingestion (user holds the $15/yr relationship — preserves the no-credential-proxying non-goal)
- **GoCardless Bank Account Data (PSD2)** for EU/UK; Moneyman continues for IL
- **OFX/QIF import** — also the migration path off Quicken/Mint exports
- **Recurring / subscription detection + upcoming-cashflow calendar** — the most-cited "wow" feature in Monarch and Copilot reviews, and pure logic over data you already hold
- **Automated transfer peer-matching** — cross-currency transfers currently read as phantom income/expense, which is P0's most visible daily annoyance

**Exit criterion:** D30 retention of connected accounts ≥40%. If this horizon doesn't move that number, nothing downstream matters.

### H3 — "Answer the question" (~Q2 2027) — *the monetization horizon*
*Theme: ship what P0 actually pays for, and open Cloud.*

- **Plan promoted to a first-class surface**, not a nav item — the answer to the anxious question belongs on the dashboard, in the onboarding, and in the marketing
- **Monte Carlo / range-based projection** — a probability band, not one deterministic line. This is the direct ProjectionLab comparable and justifies the $129–144 price point
- **Insight-to-action**: the Plan engine already computes required monthly savings and the first danger year — surface it as a move, not arithmetic
- **Holdings-lite**: symbol/qty/cost basis + daily price → allocation, unrealized P&L, and an auto-written valuation row. Explicitly punt tax lots, corporate actions, TWR/IRR; label it a tracker, not a book of record
- **Scheduled email/PDF report** — recurring re-engagement, and the hook for advisor sharing later
- **Goaldy Cloud GA** at $12/mo · $120–144/yr · $449 lifetime

**Exit criterion:** 100 paying subscribers. This is the go/no-go for the whole commercial thesis (§7.5).

### H4 — "Expand the household" (~Q3 2027)
*Theme: raise ARPU on an existing base; open S2/S3 without a new GTM motion.*

- **`entities` / ownership layer** — the single highest-value unbuilt feature after ingestion; opens S2 and S3 with one table
- **Multi-user roles** (partner / accountant read-only / advisor read-only) — `users` table already exists; Monarch's collaboration is its top retention driver
- **Document attachments** + full-text search (SQLite FTS5)
- **Audit log** — non-negotiable for anyone with an accountant
- **Accountant export pack** by entity by period — turns the accountant from a blocker into a channel
- → ships as **Household Pro, $25–40/mo**

### H5 — "Agents" — re-slotted in v1.2
*Theme: already being delivered by a coding agent. The question is no longer when to build it, but which side of the paywall it lands on. Answer: the free side.*

- **MCP server** over the existing OpenAPI surface; read-only by default, write scopes opt-in — **free, all tiers**
- **Chat / ask-your-ledger** and **LLM-assisted categorization** — **free with BYO API key**, self-hosted and Cloud alike
- **Managed inference** (no key required, capped) — the only metered part, sold as convenience, not capability
- **Proactive goal-drift push** — free

**Why free, when the PRD has this as the first paid layer.** Build cost has collapsed, and that cuts against you, not for you: a capability your agent produces in a week is one every competitor's agent produces in a week. Kubera and Era already shipped MCP. A commodity cannot hold a price. Meanwhile the same feature *given away* buys three things a paid tier would not:
1. **A launch headline** — "the only self-hosted personal finance app with a built-in agent and an MCP server" is a true, checkable, category-first claim to S1 on launch day.
2. **A discovery channel** — MCP registries and directories are where P0's more technical half now looks for tools. That is distribution, and it is free.
3. **A defensible line to stand on** — *you pay for compute we run, never for a capability we gate.* That single rule survives contact with an audience that will scrutinise every paywall.

Keep the F18.5 licensing-service architecture; it now gates **Household Pro** and managed inference rather than the AI surface itself.

### Permanently out of scope
AR/AP/invoicing, VAT/tax filing, payroll, statutory double-entry, tax optimization, bank-credential proxying, trade execution, partnership/GL accounting, capital calls, a native mobile app, and anything requested primarily by S1 that P0 would not use.

---

## 10. Licensing, hosting, and the value exchange

### 10.1 The principle that decides everything below

**You cannot charge for what is cheap to build. You can charge for what is expensive to run, and for what is expensive to leave.**

An agent now writes a chat interface, an MCP server, a categorization model call, a chart, a report in days. So can every competitor's agent. Anything in that class is a *feature*, and features have gone to zero. What has not gone to zero:

| Durable | Why an agent can't collapse it |
|---|---|
| **Long-tail institution coverage** | Requires real accounts at real banks to test. No amount of code generation produces a CSV export from an obscure Israeli credit-card issuer that you don't hold a card with. |
| **Accumulated user data** | Three years of valuation history, a tuned tag tree, a household Plan. Switching cost measured in years, and it belongs to the user — which is exactly why it's honest to rely on. |
| **Operational burden** | Uptime, backups, restores, migrations, support at 11pm. Nobody wants this, which is why they pay for it. |
| **Trust** | Takes years, dies in one incident, cannot be generated. |
| **Distribution** | The list, the forum, the newsletter, the community that vouches for you. |

**Monetize the boring, undifferentiated heavy lifting and the data gravity. Give away the clever parts.** This is the opposite of the instinct that built PRD F18, and it is the single most important correction in this document.

### 10.2 Licensing — the recommendation

**AGPL-3.0 for the core. A separately-licensed proprietary package for Pro (entities, roles, audit log, advisor share). DCO on contributions, not a CLA.**

**The options, honestly weighed:**

| Option | Buys you | Costs you |
|---|---|---|
| **Closed (today)** | Nothing you actually need | Top comment on your Show HN; no contributors; no awesome-selfhosted; forces you to build a feedback channel to replace the Issues tab you gave up |
| **FSL-1.1 / BSL** (source-available, converts to Apache in 2 yrs) | Absolute protection of hosting rights | Lands you on awesome-selfhosted's `non-free.md` rather than the main list; forfeits the OSI-credibility signal with S1; measurably dampens contribution |
| **AGPL-3.0 + proprietary Pro** ← **recommended** | Main list, OSI credibility, contributions, Ghostfolio precedent in this exact category, and a hosted fork is still obliged to publish its changes | Does not, in theory, stop someone hosting Goaldy commercially |

**Why that theoretical cost is acceptable:** the AGPL free-riding scenario is a hyperscaler or a rival SaaS operating your code at scale. That is a real threat to a database or an observability platform. It is not a threat to a niche personal-finance app whose moat is trust, support and long-tail bank coverage — the code is the *cheapest* part of what you'd be handing over. AWS is not launching Managed Goaldy. A competitor who forks you still has to earn the trust of people handing over their bank history, and AGPL forces them to publish every improvement back.

**Why AGPL beats FSL *specifically for Goaldy*:** your hardest problem is H2 — long-tail import adapters for institutions you don't bank with. That problem is unsolvable by money at your scale and unsolvable by coding agents at any scale. It is solvable by **contributors who hold accounts at those institutions**, and contributors need a real open-source licence and a public repo. Open-sourcing converts your most agent-resistant problem into community labour. Nothing else on the table does that.

**DCO, not CLA.** A CLA lets you relicense or sell later; it also visibly suppresses drive-by contributions, which is the entire benefit you're buying. Take the DCO. Revisit only if an exit becomes a live plan — and understand that retro-fitting a CLA later means re-contacting every contributor.

**The paywall rule, stated once so it can be enforced:**
> Never paywall data ownership, portability, correctness, or security. Paywall convenience, collaboration, scale, and compute we pay for.

By that rule: export, backup, encryption, every bug fix, the AI surface, and the MCP server are free forever. Hosting, managed inference, multi-user roles, entities, audit log and advisor sharing are paid. That line is defensible in public; "AI costs extra because AI is valuable" is not.

### 10.3 Hosting vs self-hosting — the economics you actually have

**Your architecture is an unusually good hosting business and you may not have noticed.** One SQLite file per tenant means: no multi-tenancy rewrite, trivial per-customer isolation, hundreds of tenants per small VPS, backup and restore that are a file copy, and — the part that matters commercially — **an exit promise nobody else in this market can make honestly: "here is your entire database, take it and go."**

| | Self-hosted | Goaldy Cloud |
|---|---|---|
| Revenue / user | ~$0 (0.3–0.5% buy a Pro key) | $120–144/yr, or $449 lifetime |
| COGS / user | $0 | ~$0.50–2/mo at density |
| Gross margin | n/a | **85–95%** |
| CAC | ~$0 (forum, list, search) | Low — converted from free tier and content |
| Support cost | Moderate, and **it is the product you're buying from them** | High per user, and rising with tenure |
| What you actually get | Defect discovery, adapters, credibility, translations | Money, retention data, WTP signal, referrals |

**The obligation hosting creates, which you must engineer rather than assert.** The moment you host, you become the custodian your own positioning warns people about. Non-negotiables before Cloud GA: per-tenant encryption at rest, a published architecture and security page stating exactly what you can and cannot see, region choice (EU / IL at minimum for P0), a documented incident plan, and one-click full export. And honest copy — not "your data is safe," but: *"Self-host and we can never see it. Or let us host it, and here is precisely what we can see, and here is the button that hands you the file and ends the relationship."* That sentence sells better than any privacy claim, because it's falsifiable.

### 10.4 The value exchange — who pays in money, who pays in everything else

This is two separate transactions and they must be run simultaneously. The free tier tests your *software*. The paid tier tests your *business*. Neither substitutes for the other.

| Constituency | What they give you | What they cost you | What you owe them | Never do |
|---|---|---|---|---|
| **S1 self-hosters** | Defect discovery in configurations you'll never own; import adapters for institutions you don't bank with; translations; GitHub stars; awesome-selfhosted and newsletter listings; **credibility with P0, who reads those forums before trusting you** | Support hours; feature-request gravity pulling toward the wrong product; occasional entitlement | A genuinely complete free product, fast issue triage, a public roadmap, credit for contributions | Cripple the free tier. Let them set the roadmap — they'll ask for double-entry, plain-text export and a CLI; P0 wants none of it |
| **P0 Cloud payers** | Money; retention data; willingness-to-pay signal; referrals inside tight expat/IL communities | Support; an uptime obligation; data-custody risk | Reliability, portability, privacy, and a working plan | Paywall their data or their exit |
| **S2 / S3 Pro** | ARPU expansion; concrete requirements for entities and audit | Long, consultative support cycles | Roles, audit, export, and patience | Build them AR/AP/VAT/payroll |
| **Advisors (H4+)** | Distribution leverage — one advisor brings N households | White-label and integration pressure | Read-only client views, scheduled PDF reports | Become an advisor CRM |

**The asymmetry that justifies the whole free tier: P0 does not file bug reports. P0 churns silently.** S1 files a detailed issue with logs at 2am for free. That makes your self-hosted base a QA department and an early-warning system you are not paying for — and it is why a *crippled* free tier is self-defeating. Gate a feature and you stop receiving defect reports about it; you'll learn it was broken when a paying customer leaves without telling you why.

**The counter-rule, so this doesn't become an excuse:** S1's feedback is high-signal about *correctness* and near-worthless about *priority*. Weight their bug reports at 100%. Weight their feature requests at close to zero unless P0 wants the same thing.

**What you must never sell, at any tier:** user data, advertising, advisor lead-generation on your users, or anonymised aggregate financial data. Each converts a trust-based business into a data business, and this audience will detect it and leave. There is no version of that trade that pays.

---

## 11. What to do Monday

1. **Adopt the positioning sentence in §8.1** and rewrite the landing-page pitch (PRD F14) against it. You sell the answer to a decision, not a budgeting app.
2. **Decide the licence: AGPL-3.0 core + proprietary Pro, DCO.** (§10.2) This unblocks the public repo, the feedback channel, and the launch — everything else queues behind it.
3. **Re-slot the in-flight AI/MCP work to the free tier** with BYO API key. (§10.1, H5) It becomes the launch headline instead of a paywall argument you'd lose.
4. **Commit to P0 (§8.3) and to the anti-ICP (§8.6) in writing.** Half this document's value is the features you now get to refuse.
5. **Start H1 with `asset_valuations`** — one week, fixes a correctness defect live on the public demo, prerequisite for holdings and entities both.
6. **Write the paywall rule (§10.2) into the PRD as a governing principle**, before any tier work starts. It is much cheaper to hold a line than to retreat from one publicly.
7. **Stand up the Cloud waitlist** at $120–144/yr with a $449 lifetime option, and **instrument D30 retention of connected accounts** — the number that decides whether H3 is worth building.

---

_Document created 2026-09-05. Living document — revise as market facts or product state change, and log every revision above._
