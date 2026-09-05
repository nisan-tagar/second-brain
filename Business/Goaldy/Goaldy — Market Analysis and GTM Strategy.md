| Field            | Value                                      |
| ---------------- | ------------------------------------------ |
| **Created**      | 2026-09-05 14:00 UTC                       |
| **Last Updated** | 2026-09-05 14:00 UTC v1.0                  |
| **Version**      | 1.0                                        |
| **Status**       | Draft — for CEO decision                   |
| **Author**       | Business development / market research session |
| **Related**      | PRD v2.6 (esp. §2, F18), TDD §13            |

### Change Log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-09-05 | Initial market analysis. Industry scan of personal-finance, self-hosted-finance, family-office and SMB-cashflow categories. Verdict on family-office and SMB fit (both rejected as primary segments, with a narrow qualified wedge defined for each). Feature gap analysis ranked by revenue proximity vs. build cost. GTM recommendation built around a free self-hosted tier, with a structural correction to PRD F18.4 (hosting is the primary revenue line, not an add-on to AI) and a challenge to the closed-source decision. |

---

## 0. Executive verdict (read this if you read nothing else)

1. **Family offices: no. Not a segment, a fantasy.** Goaldy has no securities/positions model, no cost basis, no performance calculation, no legal-entity or ownership layer, no capital-call/commitment tracking, no document vault, and no partnership accounting. Those five things *are* the family-office product. The gap is a second product, not a backlog. See §3.
2. **SMB: no, as "accounting". A narrow yes as "the owner's consolidated personal + business cashflow view."** Statutory accounting (AR/AP/VAT/payroll/double-entry/accountant export) is a moat you cannot cross, and QuickBooks/Xero own it. See §4.
3. **Your stated AI/MCP differentiator (PRD F18.1) has already been commoditized.** Kubera ships MCP today at $249/yr, Era ships MCP over aggregated accounts. "Harmonized system of record for agents" is now parity, and Goaldy has *worse* ingestion than both — no aggregator at all. See §5.
4. **The single highest-leverage missing feature is automated ingestion (SimpleFIN Bridge + GoCardless/PSD2), not AI.** Every competitor's #1 review complaint is manual CSV. This is a weeks-long build. See §6, Tier A.
5. **The self-host-only funnel cannot pay for itself, and the arithmetic is not close.** At an industry-standard 1% free→paid conversion, $10k MRR at $10/mo needs ~100,000 active self-hosters — larger than Firefly III's entire installed base after a decade. **Hosted must become the default CTA and the primary revenue line; self-hosting is your trust asset and top of funnel, not your product.** This inverts PRD F18.4. See §7.
6. **Closed source is a strategic error for the exact audience you're targeting.** You are asking r/selfhosted — the most paranoid software audience alive — to run an opaque binary over their complete bank history. Firefly III, Actual Budget and Ghostfolio are all open. See §7.4.
7. **Pick one beachhead: cross-border / multi-currency households (Israel + US/EU expats).** You already ship the two things nobody else ships for them (real RTL/Hebrew and true multi-currency), the audience has money and acute pain, and it is defensible against Monarch (US-only) and Firefly (unusable UX). See §8.

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

| # | Feature | Proven by | Why now |
|---|---|---|---|
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
| **Goaldy Cloud** | Managed, backed up, updated, PWA, no Docker | **$8–10/mo, $96/yr** | **The primary revenue line.** Prices at the category anchor (Monarch $99.99). |
| **Goaldy.AI** | Chat + MCP + Monte Carlo + proactive nudges. Usage-capped; BYO-key option. | **+$8–12/mo** | Attach-rate expansion on Cloud; license-key unlock for self-hosters (F18.5 architecture stands). |
| **Household Pro** (Phase 3) | Entities, roles, audit log, documents, advisor sharing, scheduled reports | **$25–40/mo** | §3.2 / §4.2 segments. 10x under Kubera Black. |

Self-hosters can buy Goaldy.AI and Household Pro via the license key. That keeps the promise ("nothing about where your data lives is ever paywalled") intact while giving self-hosters something to pay for.

### 7.3 Beachhead and sequencing
**Beachhead: cross-border / multi-currency households — Israeli, Israeli-expat, and dual-country EU/US families.**

Why this and not "privacy-conscious self-hosters" generally:
- You ship **real RTL/Hebrew** (744 keys/locale) and **true multi-currency with live FX**. Monarch is US-only. Copilot is US + iOS-only. Firefly has multi-currency but no usable UX and no Hebrew UI worth the name. **Nobody is serving this person.**
- The pain is acute and expensive: two to four currencies, two tax jurisdictions, accounts that no single aggregator covers, and a spreadsheet holding it together.
- It's reachable: Israeli tech/finance communities, expat forums, Hebrew-language personal-finance channels — concrete, addressable places, unlike "family offices."
- It's a natural on-ramp to Household Pro: these households disproportionately have entities.

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

## 8. What to do Monday

1. **Delete the family-office ambition from your own head.** Keep the entity layer (B1) — kill the label.
2. **Re-scope F18.1.** The MCP-as-system-of-record claim needs ingestion behind it or it's a press release. Ingestion *is* the roadmap.
3. **Decide the license.** This blocks the launch and nothing else can be sequenced around it.
4. **Start A1 (SimpleFIN).** Highest leverage item in this document.
5. **Stand up the Cloud beta waitlist** on the landing page before it goes live, so launch traffic has somewhere to convert.

---

_Document created 2026-09-05. Living document — revise as market facts or product state change, and log every revision above._
