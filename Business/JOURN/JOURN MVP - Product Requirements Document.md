# JOURN — Product Requirements Document

**Created:** 2026-03-15 **Updated:** 2026-03-29T00:00 **Version:** 2.7 **Status:** Active — aligned with Tech Design v2.5

> **Brand note (v2.7):** Product renamed from Journiq → **JOURN** (derived from _journée_, Old French for "a day's work"; connects to journeyman trades credential). Domain: **tryjourn.app** (confirmed available; .app TLD requires HTTPS). Historical name "Journiq" preserved only in this note.

> **This document is a complete rewrite (v2.0).** All previous versions contained accumulated inconsistencies. This version is written from ground truth: Tech Design v2.5, the onboarding mock (`journiq-onboarding.jsx`), the dashboard mock (`journiq-unified.jsx`), and decisions made in the product sessions documented in this project.

---

## Table of Contents

1. [Product Definition](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#1-product-definition)
2. [Problem Statement](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#2-problem-statement)
3. [Target Users & Beachhead ICP](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#3-target-users--beachhead-icp)
4. [Goals & Non-Goals](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#4-goals--non-goals)
5. [MVP Scope](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#5-mvp-scope)
6. [System Architecture Overview](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#6-system-architecture-overview)
7. [Onboarding Flow (7 Steps)](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#7-onboarding-flow-7-steps)
8. [IC Dashboard](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#8-ic-dashboard)
9. [Business Storefront & Customer Booking](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#9-business-storefront--customer-booking)
10. [Explore Page](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#10-explore-page)
11. [Appointment Lifecycle](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#11-appointment-lifecycle)
12. [Notifications](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#12-notifications)
13. [Settings Page](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#13-settings-page)
14. [Subscription Tiers](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#14-subscription-tiers)
15. [Guardrails & Constraints](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#15-guardrails--constraints)
16. [Success Metrics](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#16-success-metrics)
17. [Competitive Positioning](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#17-competitive-positioning)
18. [Product Roadmap](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#18-product-roadmap)
19. [Go-To-Market & Discovery Plan](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#19-go-to-market--discovery-plan)
20. [Glossary](https://claude.ai/chat/89f68114-7862-4b3c-ae44-7da8e4a6f0a5#20-glossary)

---

## 1. Product Definition

**JOURN** is a route-aware booking platform for field service professionals (ICs) in Israel. It enables customers to self-book appointments in under 2 minutes without disrupting the IC's daily route, requiring a dispatcher, or playing WhatsApp phone tag.

**Core value proposition:**

|For the IC|For the Customer|
|---|---|
|Never miss a booking — customers self-book 24/7|Book any field service in under 2 minutes|
|Route harm prevention — bad routes can't form|Know upfront if you're in the service area|
|Go live in 15 minutes — no dispatcher, no training|Clear pricing before committing|
|WhatsApp notifications — stay informed without checking a dashboard|Magic link to manage your appointment|

**What "route-aware" means at MVP:** JOURN prevents customers from booking slots that would create geographically infeasible days. A customer in Tel Aviv cannot book a 10am slot if the IC already has a 9am job in Netanya and an 11am in Ra'anana — that slot simply does not appear. The IC retains full control over their daily job sequence. This is **route harm prevention**, not fleet optimization. For a solo IC managing their calendar on WhatsApp today, it is a transformative improvement.

---

## 2. Problem Statement

Field service ICs in Israel manage bookings primarily through WhatsApp and phone calls. This creates four compounding problems:

**Missed bookings** — calls and messages go unanswered while the IC is on a job. Those customers go to a competitor.

**Scheduling chaos** — manual coordination leads to double-bookings, forgotten requests, and hours spent on back-and-forth that could be spent on jobs.

**Wasteful routes** — jobs booked without geographic awareness create days where the IC zigzags across the city, burning fuel and losing billable hours.

**No self-service** — every inquiry requires the IC's personal attention. This creates a hard ceiling on how many jobs the IC can take.

Existing tools don't solve this:

|Category|Examples|Problem|
|---|---|---|
|Simple booking tools|Calendly, Cal.com|Time-based only. No geographic awareness. Dangerous for field work.|
|FSM suites|Salesforce FSM, ServiceTitan, Housecall Pro, Jobber|Powerful but built for dispatched teams. Days to implement. Wrong scale for solo ICs.|

**The gap:** No lightweight, customer-facing, route-aware booking tool exists that a solo IC can set up in 15 minutes and start using today.

---

## 3. Target Users & Beachhead ICP

### Primary User — IC / Micro-Business Owner

A solo or small-team (1–5 person) mobile service professional who travels to customer locations.

**Current state:** Manages bookings via WhatsApp. No scheduling software. Route planning is intuitive and often poor. Tech-comfortable but not tech-enthusiastic — needs tools that just work.

**Beachhead ICP (validated through discovery):**

Solar/electrical technicians and AC technicians in the Sharon region (Netanya, Ra'anana, Herzliya, Kfar Saba).

|Why this ICP|Evidence|
|---|---|
|Year-round demand, no seasonality|AC and solar/electrical work does not spike|
|High booking frequency|3–10 requests per week per IC|
|High route sensitivity|Multiple home visits daily — bad routing costs real money|
|Accessible|126K+ reviews on Midrag in solar/electrical category alone|
|Geographic concentration|Sharon region is compact — referrals travel within the community|
|Tech-comfortable|Already use WhatsApp Business, many have Google Business profiles|

**Secondary beachhead:** Plumbers — same booking frequency, same route sensitivity, accessible via Midrag and Yad2.

**Note:** The product is horizontal. The GTM is not. Learnings from the beachhead inform expansion to adjacent trades.

### Secondary User — End Customer

Israeli homeowner or tenant needing a field service professional.

- Reaches the IC's booking page via a WhatsApp link, social media bio, or Google Business profile
- Expects to know upfront whether they're in the service area and what the job will cost
- Prefers a fast self-service flow over a phone call
- No login required — guest booking throughout

---

## 4. Goals & Non-Goals

### Goals (MVP)

- IC can go live and accept their first customer booking within 24 hours of signup
- Customers can complete a booking in under 2 minutes
- Geographically infeasible slots never reach the customer
- Route protection is visible to the IC — not a black box
- Onboarding completes in under 15 minutes without any help from JOURN

### Non-Goals (MVP)

- Route optimization or VRP solving — MVP is harm prevention only
- Payments or deposits
- Real-time technician tracking
- Multi-technician route optimization
- Resource dashboard login (owner-only access in MVP)
- Reschedule flows (post-MVP)
- AI transactional booking — AI Chat is informational only in MVP
- Inventory or invoicing

---

## 5. MVP Scope

|Feature|Status|Notes|
|---|---|---|
|7-step IC onboarding|✅|Target: <15 minutes|
|Service management|✅|CRUD, urgency level, pricing model|
|Resource management|✅|Up to 5 resources, working hours, absences with geocoded locations|
|Service area (Google Places regions)|✅|Business-level. Resource override is post-MVP.|
|Route-aware slot generation|✅|Greedy gap-filling with location propagation|
|Customer self-booking|✅|Guest flow, no login|
|Auto / manual confirmation|✅|IC-configurable per business|
|Schedule View (Gantt + Day Detail)|✅|Week overview + day timeline + route map panel|
|Day Route Map|✅|IC sees geographic corridor for the day|
|IC-created appointments|✅|Bypass slot engine, always auto-confirmed|
|Magic link / appointment page|✅|Customer self-service, cancellation|
|WhatsApp notifications|✅|Primary channel via Twilio BSP|
|Email notifications|✅|Secondary channel via SendGrid|
|AI Chat (informational)|✅|Professional + Team tiers only. Answers questions, does not book.|
|Subscription tiers + tier gates|✅|Solo / Professional / Team|
|Explore page|✅|Supply-dependent — see caveat in Section 10|
|**Marketing landing page**|✅|Hebrew, dark theme, RTL. Waitlist + early access form. Deployed at tryjourn.app. Copy reviewed by native specialist.|
|**Waitlist backend**|🔲|Supabase `waitlist` table + tRPC mutation. Pending wiring.|
|**WhatsApp notification mocks**|🔲|Screenshot assets for landing page social proof section. Pending.|
|Reschedule flows|❌|Post-MVP|
|Payments / deposits|❌|Post-MVP|
|Resource dashboard login|❌|Post-MVP|
|Optimized Scheduling / Schedule Optimization|❌|V2 — Professional+|
|Resource-level service area|❌|Post-MVP — Professional+|

---

## 6. System Architecture Overview

Three surfaces, one backend:

**IC Dashboard** (`/dashboard/*`) — authenticated, owner-only in MVP. Manages services, resources, schedule, settings, appointments.

**Customer Storefront** (`/b/{slug}`) — public, no auth. The IC's booking page. Contains AI chat, service list, and entry to the booking flow.

**Customer Appointment Page** (`/appointment/{token}`) — magic link, no auth. Customer views their booking, can cancel.

**Core services:**

- **Slot Generator** — greedy gap-filling algorithm. Inputs: resource schedules, existing appointments, customer location. Outputs: feasible time slots.
- **GeoService** — Google Places region matching. Validates customer address against business service area.
- **Routing Service** — OpenRouteService API. Calculates travel time between locations. Provider-agnostic interface for future migration.
- **Notification Service** — WhatsApp (primary) + Email (secondary).
- **AI Service** — Claude. Informational only. Answers questions about services, pricing, area. Directs to booking flow.

**Stack:** Next.js 14, tRPC v11, PostgreSQL (Supabase), Prisma, Tailwind, TypeScript. Deployed on AWS.

---

## 7. Onboarding Flow (7 Steps)

Target completion: **under 15 minutes**. IC is live and can accept bookings at the end of Step 7.

Steps (from onboarding mock `journiq-onboarding.jsx`):

|Step|Name|Time|What happens|
|---|---|---|---|
|1|Account|1 min|Email, password, phone|
|2|Business|2 min|Business name, category, language, fallback contact|
|3|Services|5 min|Define 1–3 services to start|
|4|Resources|3 min|Add resources — home base required per resource|
|5|Service Area|1 min|Business-level service area (cities/districts)|
|6|Booking Settings|1 min|Confirmation mode, reminders|
|7|Customize & Launch|2 min|Slug, social links, go live|

> **AI Setup removed from onboarding.** AI grounding is available post-onboarding in Settings → AI Assistant. Blocking a solo plumber's 15-minute go-live on AI FAQ configuration is the wrong trade-off. ICs on Solo tier don't have AI access anyway.

---

### Step 1 — Account

- Email + password (no SSO in MVP)
- Phone number — used for WhatsApp notifications and as fallback contact

---

### Step 2 — Business Profile

|Field|Required|Notes|
|---|---|---|
|Business name|✅|Shown on storefront and in notifications|
|Business category|✅|Select from managed Category table (see Section 10)|
|Default language|✅|Hebrew (default) / English — selects WhatsApp template variant|
|Tagline|Optional|One-liner shown on storefront|
|Logo|Optional|Upload|
|Fallback contact method|✅|WhatsApp / Phone / Email|
|Fallback contact value|✅|Phone number or email — shown when booking is unavailable|

---

### Step 3 — Services

IC defines 1–3 services to start (more can be added post-onboarding from the dashboard).

Per service:

|Field|Required|Notes|
|---|---|---|
|Service name|✅|e.g. "Leak Repair"|
|Description|✅|Short description shown to customers|
|Visit fee|✅|Fixed fee for IC to arrive|
|Visit fee policy|✅|`ALWAYS_CHARGED` or `WAIVED_IF_ACCEPTED`|
|Job fee min|✅|Lower bound of job cost estimate|
|Job fee max|✅|Upper bound. If min = max → shown as fixed price.|
|Duration (minutes)|✅|Drives slot generation|
|Urgency level|✅|`REGULAR` (default) or `EMERGENCY`|
|Scheduling mode|✅|`SELECT_SLOT` (default). `OPTIMIZE` is V2, Professional+ only.|
|Confirmation note|Optional|Per-service message shown to customer after booking (e.g. "Bring access code")|
|Required inputs|Optional|Multi-select: Photos, Access notes, etc.|

**Urgency level — customer-facing impact:**

|Level|Storefront Badge|Booking Flow|
|---|---|---|
|REGULAR|None|Determined by scheduling mode|
|EMERGENCY|⚡ Fast Response|Always Instant Booking|

"Emergency" never appears on the storefront — only the "Fast Response" badge. The badge is static — no live availability check. IC guidance: _Emergency services typically command a higher visit fee._

---

### Step 4 — Resources

This is the most important step for route protection. IC configures their resources (people who perform jobs). For a solo IC this is just themselves.

**Per resource:**

|Field|Required|Notes|
|---|---|---|
|Name|✅|Display name|
|Role|Optional|e.g. "Senior Plumber", "Apprentice"|
|Phone|Optional|Internal use and IC notifications|
|Home base|✅ **Required**|Routing origin. Google Places address search. No business-level fallback — must be set.|
|Travel mode|Optional|Falls back to business default if not set. Car / Motorcycle / E-Bike / Bicycle / Walking|
|Parking buffer|Optional|Falls back to business default. Minutes added on arrival.|
|Job buffer|Optional|Falls back to business default. Gap between job end and next start.|
|Working days|✅|Days this resource works. Default: Sun–Thu.|
|Working hours|✅|Single shift in onboarding. Multiple shifts added from dashboard post-onboarding.|

**Constraints:** Max 1 resource on Solo, 3 on Professional, 5 on Team. Home base cannot be skipped. Deletion blocked if resource has future SCHEDULED or PENDING_APPROVAL appointments.

---

### Step 5 — Service Area

Business-level. All resources inherit the same service area in MVP. Resource-level override is post-MVP (Professional+).

Uses the `ServiceAreaPicker` component:

- Search by city or district via Google Places region autocomplete (`geo.regionAutocomplete`)
- Selected regions appear as dismissible chips
- Minimum 1 region required to advance
- Leaflet map preview showing selected regions (same component as Settings → Service Area)

Hint: _"Where do you travel to for jobs? Add all the cities or districts you cover."_

### Step 6 — Booking Settings

Business-level settings applied to all services and resources.

|Setting|Default|Notes|
|---|---|---|
|Confirmation mode|Auto-confirm|Auto-confirm: booking → SCHEDULED immediately. Manual approval: booking → PENDING_APPROVAL, IC reviews before confirming.|
|Cancellation window|24 hours|Free cancellation period. Late cancellations flagged (fees post-MVP).|
|Customer reminders|Enabled|WhatsApp 24h + 2h before appointment|
|Allow reschedule|Enabled|Post-MVP feature — toggle reserved for future activation|
|Working day pattern|Sun–Thu|Used to calculate reschedule window in business days|

---

### Step 7 — Customize & Launch

|Element|Notes|
|---|---|
|Storefront URL slug|Auto-generated from business name, editable. `tryjourn.app/b/{slug}`|
|Social links|Instagram, Facebook, TikTok, YouTube, LinkedIn, X, Website|
|Photo gallery|Up to 6 images or video links|
|Availability indicator|Toggle: show "● Available until 6PM" on storefront|

At completion: storefront is live, booking link is shareable. IC is shown a launch screen with the link and a WhatsApp share button.

---

## 8. IC Dashboard

Accessible at `/dashboard/*`. Owner-only in MVP. Five sections accessible via sidebar nav:

### 8.1 Schedule View (`/dashboard/schedule`)

The primary daily view. Two modes:

**Week Overview** — Gantt grid. Rows = resources. Columns = days. Each cell shows appointment blocks (green = confirmed, amber = pending) and absence blocks (grey/striped). Click any cell → Day Detail.

**Day Detail** — Horizontal timeline for a selected date. One row per resource. X-axis = time.

Each timeline row shows:

- Working hours window (light background)
- Appointment blocks (green/amber) — click to open detail panel
- Absence/break blocks (grey striped) — click to edit
- **Travel connectors** between consecutive events — see spec below

**Travel connector visual spec:**

A narrow muted strip between consecutive Gantt blocks. Sourced from `travelToMinutes` on the destination event — no live routing calls on render.

```
┌──────────────────┐  ╌╌14 min╌╌▶  ┌──────────────────┐
│  Appointment A   │                │  Appointment B   │
│  09:00–10:00     │                │  10:14–11:14     │
└──────────────────┘                └──────────────────┘
```

|Case|Connector shown|
|---|---|
|Located → located|✅ `travelToMinutes` of destination event|
|Located → non-located absence|❌ null — no connector, blocks are adjacent|
|Non-located absence → located|✅ `travelToMinutes` of the located event|
|Last event → home base|✅ `travelFromMinutes` of last event|
|`travelToMinutes` is null|❌ no connector|

- Width proportional to travel duration on the timeline X-axis
- Label: travel minutes centred on the connector strip
- Hover tooltip: _"14 min from previous job"_ or _"8 min from home base"_

**Manual travel refresh:** A "Recalculate" button (refresh icon) in the Schedule View toolbar. Triggers `schedule.updateTravels` for the current view's horizon. Week view → full visible 7-day window for all resources. Day view → selected date only. Shows brief loading state; updates connectors on completion without full page reload.

**Day Route Map panel** — Displayed alongside the Day Detail timeline. This is the primary mechanism by which JOURN's route intelligence becomes visible and trusted.

What the map shows:

- IC home base pin (start and end of day anchor)
- All SCHEDULED appointments as numbered pins in chronological order
- Located absences as distinct pins (e.g. lunch at home, supplier visit)
- A shaded geographic corridor connecting all stops in sequence

What it communicates to the IC: their day clusters in one area rather than zigzagging. If the day looks geographically incoherent they can see why — and take action (cancel, contact customer, add the appointment to a different day).

Implementation: reuses the existing map component from the Gantt/schedule view. No new mapping infrastructure required.

MVP scope: read-only. The IC cannot drag-and-drop to reorder from this view. Reordering is V2.

**Appointment detail side panel** — opened by clicking an appointment block:

- Customer name, phone, address
- Service, duration, scheduled time
- Pricing snapshot
- Customer notes / access instructions
- Approve / Decline (if PENDING_APPROVAL)
- Cancel (if SCHEDULED)

**Inline break management** — click empty timeline slot:

- Add recurring break (creates RECURRING `ResourceAbsence`)
- Block one-off time (creates ONE_TIME `ResourceAbsence`)
- Form: start time, end time, reason (optional), break location (optional)

**IC-created appointments** — `+ Create` button in Schedule View opens a modal for IC to manually add an appointment. Bypasses slot engine and service area validation (IC takes responsibility). Always auto-confirmed. Sets `createdByIc: true`.

---

### 8.2 Appointments (`/dashboard/appointments`)

List of all appointments — filterable by status, date range, resource. Shows customer name, service, time, status. Click to open detail panel.

---

### 8.3 Services (`/dashboard/services`)

CRUD for services. Add, edit, deactivate. Fields as defined in Step 3. Urgency level and scheduling mode configurable per service.

---

### 8.4 Resources (`/dashboard/resources`)

List of resources. Each row expands into a panel with three tabs:

**Profile tab** — name, role, phone, avatar, home base, travel mode, parking buffer, job buffer. All routing override fields show inherited value when not set.

**Schedule tab** — working hours per day. Add/remove shifts. Multiple shifts per day supported. Apply preset templates. Copy from another resource.

**Absences tab** — recurring and one-time absences.

Creating/editing an absence:

- Type: RECURRING (weekly) or ONE_TIME (specific date)
- Time range: start + end
- Reason (optional)
- **Break location (optional)** — Google Places address search. When provided, geocoded and stored as `absenceLat`/`absenceLng`. The slot algorithm uses this location as a routing anchor instead of home base — producing more accurate slot generation for adjacent gaps (e.g. lunch at home, doctor visit, supplier pickup).

---

### 8.5 Settings (`/dashboard/settings`)

Seven sections, each with its own Save button. See Section 13 for full spec.

---

## 9. Business Storefront & Customer Booking

### 9.1 Storefront (`/b/{slug}`)

Public page, no auth required.

|Component|Notes|
|---|---|
|Business header|Logo, name, tagline|
|Category badge|Trade category with icon|
|Availability indicator|"● Available until 6PM" / "○ Back Sunday 8AM" — toggled by IC|
|Service area badge|"📍 Serving Tel Aviv & Central District"|
|Social links|Configured in onboarding / settings|
|Photo gallery|Up to 6 items|
|AI Chat|Embedded assistant (Professional + Team tiers only)|
|Service list|Each service shows name, description, pricing, urgency badge|
|Book Now CTA|Per service. Passes `?service={id}` into booking flow.|

**Fast Response badge** — shown on EMERGENCY urgency services. Static — no availability check.

**AI Chat (informational):**

- Answers questions about services, pricing, area coverage, policies
- Cannot confirm bookings — directs to "Book Now"
- Grounded on IC's service config + FAQ text
- Out-of-area customers are told clearly and given the fallback contact

---

### 9.2 Customer Booking Flow (6 Steps)

Triggered by clicking "Book Now" on a service. Service is pre-selected.

|Step|Content|
|---|---|
|1 — Location|Customer enters address via Google Places search → `geo.validateServiceArea` → if out of area: out-of-area screen with fallback contact. If in area: proceed.|
|2 — Pricing & Terms|Shows selected service pricing, visit fee policy, cancellation terms|
|3 — Time Slots|Shows available slots from slot generator. Customer selects date + time.|
|4 — Job Details|Description, photos (if required), access notes, urgency|
|5 — Contact Info|Name, phone (required), email (optional), reminder opt-in toggle|
|6 — Confirmation|Auto-confirm: appointment created as SCHEDULED. Manual: created as PENDING_APPROVAL with "awaiting confirmation" message.|

**Slot generation pre-condition:** Service area is validated in Step 1 **before** `getAvailableSlots` is called. The slot generator does not re-validate service area.

**On confirmation:** Customer receives WhatsApp confirmation with magic link to their appointment page.

---

### 9.3 Out-of-Area Screen

When customer's address fails service area validation:

- Shows the business's service area regions
- Customer's entered location
- Fallback contact button (WhatsApp / phone / email as configured by IC)
- Back button

---

### 9.4 Customer Appointment Page (`/appointment/{token}`)

Accessed via magic link in WhatsApp confirmation. No login required. Token expires 7 days after scheduled appointment.

Shows:

- Appointment details (service, time, address)
- IC contact info
- Pricing snapshot
- Cancellation button (within free cancellation window)
- Calendar add options (Google Calendar deep link, ICS download)
- Service confirmation note (per-service IC message)

---

## 10. Explore Page

Customers can browse businesses by category at `journiq.ai/explore`.

> **Supply caveat:** The Explore page is built in MVP but its value is supply-dependent. With fewer than 20–30 active ICs, a customer searching for a plumber in Tel Aviv may find 1–2 results or none. The page should not be promoted as a discovery channel until supply reaches critical mass in at least 2–3 categories in the target region. During beta and Stage 1, ICs should share their direct storefront link.

**Layout:** Category filter chips at top. Business cards below. Each card shows: business name, category, up to 3 service names, service area, availability indicator.

**Filtering:** By category (from Category table), by location (customer's location or entered address), optionally by "Available now".

**Category management:** Managed by JOURN — not IC-editable. ~11 categories seeded. New categories added via DB migration when 3+ businesses select "Other" with similar descriptions. Admin panel is post-MVP.

---

## 11. Appointment Lifecycle

### 11.1 Status Definitions

|Status|Meaning|
|---|---|
|`PENDING_APPROVAL`|Instant Booking, manual confirmation mode — awaiting IC action|
|`SCHEDULED`|Exact time confirmed and on the Gantt. The only "real" confirmed state.|
|`CANCELLED`|Cancelled by any party. Reason stored in `cancellationReason`.|
|`COMPLETED`|Service delivered.|
|`PENDING_OPTIMIZATION`|Optimized Scheduling only (V2) — in optimizer queue, no time assigned yet.|

`EXPIRED` and `NO_SHOW` are `cancellationReason` values on `CANCELLED` appointments — not separate statuses.

### 11.2 State Transitions

```
Customer books (auto-confirm)     →  SCHEDULED
Customer books (manual confirm)   →  PENDING_APPROVAL
  IC approves                     →  SCHEDULED
  IC declines                     →  CANCELLED (cancellationReason: DECLINED)
  Approval deadline expires       →  CANCELLED (cancellationReason: EXPIRED)
IC creates appointment            →  SCHEDULED (always, bypasses confirmation mode)
SCHEDULED                         →  COMPLETED
Any state except COMPLETED        →  CANCELLED
```

### 11.3 Approval Deadline

Manual confirmation mode: IC must respond by end of the same business day the request arrived. If submitted within the last hour of the business day, deadline extends to end of the following business day. Enforced by pg_cron job (`expire-approvals` every 30 minutes).

### 11.4 Cancellation Policy

|Actor|Timing|V1 Behaviour|
|---|---|---|
|Customer|≥24 hours before|Free cancellation. Slot freed.|
|Customer|<24 hours before|Allowed. Late cancellation flagged (fee enforcement post-MVP).|
|IC|Any time|Allowed. Customer notified. No fee to customer.|

Customer cancellation via magic link on appointment page. IC cancellation from dashboard.

### 11.5 Reschedule Flow

Post-MVP. Toggle reserved in booking settings for future activation.

### 11.6 Late Cancellation Flagging

Late cancellations are soft-flagged via `CustomerFlag` model — scoped per business, keyed by phone number. IC-initiated reschedules are **exempt** from flagging. Flag data is informational in MVP — fee enforcement requires payments integration (post-MVP).

---

## 12. Notifications

**Primary channel: WhatsApp** (Twilio BSP, Meta-approved templates) **Secondary channel: Email** (Twilio SendGrid)

> WhatsApp templates require Meta pre-approval (1–7 business days). Submit all templates before launch. 13 event types × 2 languages (Hebrew/English) = 26 template registrations.

Language selection: `Business.defaultLanguage` — `he` (default) or `en`.

### 12.1 Notification Events

|Event|Customer|IC|
|---|---|---|
|Booking confirmed (auto)|✅ Confirmation + magic link + calendar options|✅ New booking alert|
|Booking pending (manual)|✅ "Awaiting confirmation"|✅ Approval request with deadline|
|Booking approved|✅ Confirmation + magic link|—|
|Booking declined|✅ "Select another time" + fallback contact|—|
|Approval expired|✅ Apology + fallback contact|✅ Missed approval alert|
|Customer cancels|✅ Cancellation confirmed|✅ Booking cancelled|
|IC cancels|✅ Apology + fallback contact|✅ Confirmation|
|Reminder 24h before|✅ Tomorrow's appointment|✅ Daily agenda (morning)|
|Reminder 2h before|✅ Appointment soon|—|

### 12.2 Scheduled Jobs (pg_cron)

All time-based automation runs via Supabase pg_cron. Three jobs:

|Job|Schedule|Action|
|---|---|---|
|expire-approvals|Every 30 minutes|Query PENDING_APPROVAL where deadline ≤ now → set CANCELLED + send notifications|
|reminders-24h|Daily 06:00 UTC (08:00 Israel)|Query SCHEDULED appointments in next 24h where reminder not sent → send + stamp|
|reminders-2h|Every hour|Query SCHEDULED appointments in next 2h where reminder not sent → send + stamp|

All jobs authenticate via `CRON_SECRET` Bearer token on Next.js API routes.

---

## 13. Settings Page

(`/dashboard/settings`) — Seven sections. Each section except Subscription and Business Data has its own independent Save button. Sections are rendered in the order listed below.

> **Tier simulation note for development:** The tier preview toggle in the header renders the locked/unlocked states for tier-gated sections (Optimized Scheduling, AI Booking Assistant). The seed test user is always TEAM tier.

---

### 13.1 Business Profile

Fields:

- Business Name
- Storefront URL slug — editable, with live preview: `tryjourn.app/b/{slug}`. Uniqueness validated on save via `business.updateSlug`.
- Business Category — IC can change post-onboarding. Dropdown from Category table.
- Tagline — shown on storefront, 200 char max
- Contact Phone + Contact Email (grid)
- **Default Language** — Hebrew (default) / English. Selects WhatsApp template variant for all notifications sent to this business's customers and IC.

**Business model fields written:** `name`, `slug`, `categoryId`, `tagline`, `fallbackContactValue` (phone), `defaultLanguage`

---

### 13.2 Service Area

`ServiceAreaPicker` component — same as onboarding Step 5.

- Active regions displayed as dismissible chips
- Add via Google Places region search: `serviceArea.add`
- Remove via chip × button: `serviceArea.remove`
- Minimum 1 region enforced — remove button disabled on last chip
- Hint: _"Search for a city or district — e.g. 'Netanya', 'Ra'anana', 'Tel Aviv-Yafo'."_

---

### 13.3 Scheduling Logic

Combines booking behaviour and optimized scheduling settings in one section with internal dividers.

**Confirmation Mode** — radio cards:

- Auto-confirm: _"Bookings confirmed instantly — no action required from you."_
- Manual approval: _"You review and approve each booking request before it's confirmed."_

**Booking Parameters** — three-column grid:

- Slot Interval (select: 15/30/60 min) — granularity of offered time slots
- Booking Horizon (select: 7/14/30/60 days) — how far ahead customers can book
- Cancellation Window (number + `h`) — free cancellation threshold in hours

Additional:

- Show live availability on storefront (toggle) — _"If off, customers see a contact form instead of live slots."_

**Fallback Contact** — two-column grid:

- Method (WhatsApp / Phone / Email)
- Value — _"Shown when no slots are available or customer is out of area."_

**Default Routing** — three-column grid:

- Travel Mode (Car / Motorcycle / E-Bike / Bicycle / Walking)
- Parking Buffer (number + `min`) — added on arrival
- Job Buffer (number + `min`) — gap between job end and next start

**Optimized Scheduling subsection** _(Professional + Team tiers only)_

Tier-gated. Solo tier sees a locked banner with upgrade prompt instead of the fields.

- Arrival Window Size (select: 60/90/120/180/240 min) — divides working hours into customer-selectable arrival windows
- Schedule Settling Time (number + `h`) — _"Shown to customers as: 'We'll confirm your exact arrival time within X hours'."_
- Helper note: _"Optimization horizon is selected at run time — not stored here."_

**Business model fields written:** `confirmationMode`, `slotIntervalMinutes`, `bookingHorizonDays`, `cancellationWindowHours`, `allowReschedule`, `showAvailability`, `fallbackContactMethod`, `fallbackContactValue`, `defaultTravelMode`, `defaultParkingBuffer`, `defaultBufferMinutes`, `windowSizeMinutes` (Professional+), `scheduleSettlingHours` (Professional+)

---

### 13.4 Notifications

Info banner: _"WhatsApp templates are managed by JOURN and sent via Twilio."_

**Customer Notifications** (toggles):

- Booking confirmation — _"Sent immediately on confirmation."_
- Reminder — 24 hours before
- Reminder — 2 hours before

**IC / Owner Notifications** (toggles):

- New booking alert — _"Sent for every new booking, auto or manual."_
- Daily agenda (7 AM) — _"Morning summary of today's confirmed appointments."_

**Business model fields written:** `remindersEnabled`, `notifyIcNewBooking`, `notifyIcDailyAgenda`

---

### 13.5 AI Booking Assistant

**Professional + Team tiers only.** Solo tier sees a locked banner with upgrade prompt.

- Enable AI assistant (toggle) — _"If off, customers see a standard booking form instead of the AI chat."_
- Opening message (textarea) — _"The first message the AI sends to customers on the storefront."_
- FAQ / Context (textarea, 5 lines) — _"Service area, hours, pricing notes, anything the AI should know. More detail = better performance."_
- Special Instructions (textarea, 3 lines) — _"Operational rules — e.g. always collect apartment number, confirm boiler brand for boiler service."_

Warning banner: _"The AI cannot confirm bookings or invent availability. It collects customer details and hands off to the booking engine."_

**Business model fields written:** `aiEnabled`, `aiGreeting`, `faqText`, `specialInstructions`

---

### 13.6 Subscription

No Save button. Read-only display.

- Plan card — three states (ACTIVE / TRIAL / LOCKED):

**ACTIVE:**

```
Professional Plan                      ₪199/month
Up to 3 resources · AI included · Renews 9 Apr 2026
                                              [Manage]
```

**TRIAL:**

```
Free Trial — Professional Features
⏳ 8 days remaining · No credit card required
                                          [Choose Plan]
```

**LOCKED:**

```
⚠️ Account Locked
Trial ended. Choose a plan to continue accepting bookings.
Your data is safe.
                                          [Choose Plan]
```

- Usage stats grid: Resources (used / max), Services (used / max or ∞), AI Sessions (count / month)

MVP billing note: Payment processing is post-MVP. During beta: _"Billing is managed by your JOURN account manager during the beta period."_

---

### 13.7 Business Data

No Save button. Contains the Danger Zone.

**Danger Zone** — red-bordered card with three operations. Business profile and settings are **always preserved** regardless of operation.

|Operation|What is deleted|Confirmation required|
|---|---|---|
|**Clear all appointments**|All `BookedAppointment` records only. Resources, services, service areas intact.|Standard confirmation dialog|
|**Clear all operational data**|All `BookedAppointment` + all `Resource` + `WorkingHours` + `ResourceAbsence` + all `Service` + all `BusinessServiceArea`|Typed: must type `DELETE`|
|**Start over with setup wizard**|Same as above, then redirects to onboarding Step 3 (Services)|Typed: must type `DELETE`|

**Clear all appointments** is the low-risk option — primarily for ICs who want to wipe test bookings before going live. Standard confirm dialog (two-click, no typed confirmation).

**Typed confirmation pattern:** A modal with an input field. The confirm button is disabled until the IC types the exact word. On submit the mutation fires and a success banner is shown inline.

**Start over with setup wizard:** After clearing, the client is redirected to the onboarding wizard. Because the business profile already exists, the wizard resumes from Step 3 (Services) — not Step 1 (Account).

## 14. Subscription Tiers

### 14.1 Tier Definitions

|Feature|Solo ₪99/mo|Professional ₪199/mo|Team ₪349/mo|
|---|---|---|---|
|Resources|1|3|5|
|Services|5|Unlimited|Unlimited|
|Instant Booking|✅|✅|✅|
|Optimized Scheduling (V2)|❌|✅|✅|
|AI Chat Assistant|❌|✅|✅|
|Manual confirmation mode|✅|✅|✅|
|Resource home base override|✅|✅|✅|
|Resource service area override|❌|✅|✅|
|Cross-resource optimization (V2)|❌|❌|✅|
|Annual discount|2 months free|2 months free|2 months free|

### 14.2 Trial

All new businesses start on a **14-day free trial** of the Professional tier. No credit card required. At trial end: choose a plan or account locks (data retained, no new bookings).

### 14.3 Trial During Development

The development seed user (`test@journiq.dev`) is always provisioned at TEAM tier with `trialEndsAt: 2099-01-01`. All tier gates pass in development without any configuration.

### 14.4 Tier Gate Implementation

Gates are enforced server-side in tRPC procedures. Client-side hiding is UX only — never a security boundary. See Tech Design Section 7.3 for full `TIER_LIMITS` constants and `assertTierFeature` / `assertTierLimit` helper implementations.

LOCKED accounts: all mutating procedures throw FORBIDDEN. Read-only procedures (dashboard, schedule view) remain accessible.

---

## 15. Guardrails & Constraints

- **Route harm prevention, not optimization.** JOURN prevents geographically infeasible slots from being offered. It does not guarantee an optimal sequence — the IC retains full control over daily job ordering. This is a deliberate MVP scope boundary.
- **Fail safe always.** If the routing API is unavailable, no slots are shown. Customer is directed to IC's fallback contact. Never show uncertain availability.
- **Service area validated before slots.** `geo.validateServiceArea` must pass before `getAvailableSlots` is called. The slot generator does not re-validate.
- **AI is subordinate to the booking engine.** AI cannot override availability rules, invent services, or confirm bookings.
- **IC configuration takes precedence over AI inference.**
- **Edge cases reject rather than auto-book.** When uncertain, show fewer slots.
- **Schedule conflicts: zero tolerance.** Double-bookings are prevented via serializable transaction with row-level locking on the time slot.
- **IC always sees route intelligence.** The Day Route Map is always visible. The algorithm is never a black box.

---

## 16. Success Metrics

### Stage 0 — Discovery (Now → Month 2)

|Signal|What It Tests|
|---|---|
|ICs mention missed bookings unprompted|Is the pain real?|
|ICs describe bad route days as a financial cost|Does route protection resonate?|
|ICs pay for any software today|Is there willingness to pay?|
|At least 8 of 15 conversations describe the same pain|Is the problem homogeneous enough for a product?|
|ICs ask "when can I try it?"|Is there pull vs push demand?|

### Stage 0.5 — Beta (Month 2–4)

|Metric|Target|
|---|---|
|Beta ICs completing onboarding without help|≥80%|
|Beta ICs with ≥1 real booking via JOURN in 14 days|≥3 of 5|
|Customer booking completion rate|>60%|
|Routing failures during beta|0|
|Beta ICs who say unprompted they want to keep using it|≥3 of 5|

### Stage 1 — Beachhead (Month 4–8, 30 paying ICs)

|Metric|Target|
|---|---|
|Onboarding completion rate|>80%|
|First booking within 14 days of signup|>70%|
|IC churn (monthly)|<10%|
|ICs acquired via referral|≥5 (17%)|
|Bookings per active IC per week|≥3|
|MRR|~₪4,300|
|NPS at 30 days|>40|

### Stage 2+ — Expansion (Month 8+, 150+ ICs)

|Metric|Target|
|---|---|
|Self-serve onboarding (no founder involvement)|>90%|
|IC churn (monthly)|<7%|
|Referral-sourced ICs|>25% of new signups|
|MRR|~₪25,000|

### Permanent Quality Floor

|Metric|Target|
|---|---|
|Schedule conflicts created|0 — non-negotiable|
|Routing API failure rate|<1%|
|Out-of-area bookings created|0|

---

## 17. Competitive Positioning

**JOURN is not FSM software. It's booking infrastructure.**

|Competitor|What They Are|Time to Value|Built for Solo ICs|
|---|---|---|---|
|Housecall Pro|Full FSM suite|Days–weeks|❌|
|Jobber|Full FSM suite|Days–weeks|⚠️|
|ServiceM8|Full FSM suite|Days–weeks|❌|
|Calendly / Cal.com|Time-based booking|Minutes|⚠️ (location-agnostic)|
|**JOURN**|Route-aware booking|<15 minutes|✅|

**Three Pillars:**

**1. Never Miss a Booking Again** Customers self-book 24/7 without calling or messaging the IC. Every inquiry becomes a confirmed appointment.

**2. Route Protection Built In** Customers can only book slots that fit the IC's existing geographic corridor. IC sees their day on a map. Route intelligence is visible, not hidden.

**3. Live in 15 Minutes, Not 15 Days** No training. No implementation consultant. IC configures services, shares a link, starts accepting bookings the same day.

**Messaging:**

> _"Stop missing bookings. Keep your route sane. Live in 15 minutes."_

> _"Your customers book themselves. You just show up."_

> _"You don't need dispatching software. You need customers in your calendar."_

> _"Calendly doesn't know where your jobs are. We do."_

**Honest positioning note:** JOURN does not claim to solve the travelling salesman problem. It claims to stop the WhatsApp chaos and prevent obviously bad route days. That is the promise we make and the promise we keep.

---

## 18. Product Roadmap

### 18.1 MVP — Core Booking Infrastructure

Current build target. IC goes live in 15 minutes. Customers self-book. Routes are protected from obviously bad days. See Section 5 for full scope.

**Appointment lifecycle:**

```
Instant Booking + Auto-confirm  →  SCHEDULED
Instant Booking + Manual        →  PENDING_APPROVAL → SCHEDULED or CANCELLED
SCHEDULED                       →  COMPLETED
Any state except COMPLETED      →  CANCELLED
```

---

### 18.2 V2 — Optimized Scheduling (Professional+)

**Goal:** ICs with predictable booking volume offer flexible arrival windows. Optimizer builds their day. Eliminates manual confirmation for window-based services. Introduces automatic dispatching foundation.

**Key additions:**

**Optimized Scheduling mode (per-service):** Customer selects preferred days + arrival windows instead of exact times. Appointment created as `PENDING_OPTIMIZATION`. Not placed on Gantt until optimizer runs.

**Arrival Windows:** Derived at query time from union of active resource `WorkingHours` divided by `windowSizeMinutes` (Business-level setting). Not stored as a separate entity. Window = guaranteed start time, not completion time.

**Schedule Settling Time:** IC-configured SLA — _"I'll confirm within X hours."_ Stored as `scheduleSettlingHours`. Drives V2 dashboard nudge; drives V3 auto-trigger.

**Manual Optimizer:** IC triggers "Optimize my schedule" from Schedule View. Selects horizon (default 3 days, max 14 — per-run parameter, not stored). OR-Tools solver assigns exact times within committed windows. Post-run report shows scheduled vs still-pending counts.

**Full appointment lifecycle (V2):**

```
Optimized Scheduling         →  PENDING_OPTIMIZATION → SCHEDULED or CANCELLED
Instant Booking        →  (unchanged from MVP)
```

**Cross-resource optimization (Team only):** Optimizer assigns appointments across all resources to minimize total fleet travel.

---

**Travel Time Display — implemented in MVP:**

Travel chain is persisted on `BookedAppointment` and `ResourceAbsence` via `TravelChainService`. Gantt travel connectors are an MVP feature. See Tech Design Section 5.3 for full specification.

**Confirmation mode deprecation:** Manual confirmation deprecated for Optimized Scheduling services in V2 (pre-approved by window selection). Fully deprecated in V3.

---

### 18.3 V3 — Automatic Dispatching

Same infrastructure as V2. Only the trigger changes.

pg_cron checks hourly for `PENDING_OPTIMIZATION` appointments approaching `settlingDeadline`. If found → optimizer runs automatically → IC sees result on Gantt → IC receives WhatsApp summary.

This is safe to automate because the IC pre-approved every window by configuring them. The settling time is the IC's own SLA commitment. Automation honors it reliably.

---

### 18.4 Post-MVP Backlog

|Feature|Tier / Notes|
|---|---|
|Resource dashboard login|All tiers|
|Resource self-service absences|Depends on resource login|
|Resource-level service area override|Professional + Team|
|Customer table (persistent records)|All tiers|
|Payments / deposits|When booking volume justifies|
|Embeddable widget|All tiers|
|QR code generator|All tiers|
|IC job reordering from Day Route Map|All tiers|
|Review integrations (Google, Facebook)|V2|
|Service templates per category|V2|
|Business tags ("24/7", "Licensed & Insured")|V2|
|Google Reserve|Partnership track|
|Midrag / Pro.co.il integration|Partnership track|
|AI transactional booking|Market demand signal|
|AI voice calls|Market demand signal|
|CRM integrations|Enterprise demand|
|Real-time tracking|Out of scope|
|Inventory / invoicing|Out of scope|

---

## 19. Go-To-Market & Discovery Plan

### 19.1 Why This Section Exists

A product reaches market fit when the right customers find it and stay. The GTM motion is as important as the product itself. This section captures the decisions made during our business development sessions.

### 19.2 Beachhead Strategy

**Chosen beachhead:** Solar/electrical technicians + AC technicians, Central District (Merkaz / Sharon region). **Secondary:** Plumbers, same region.

The beachhead was chosen based on four criteria: booking frequency, route sensitivity, accessibility (Midrag data), and geographic concentration. See Section 3 for full rationale.

**Geographic copy rule (established during landing page review):** All example cities in copy and UI must stay within the "Golden Ring" of the center: Netanya, Kfar Saba, Ra'anana, Herzliya, Hod HaSharon, Petah Tikva, Rishon LeZion. No examples from Haifa, Beer Sheva, or other distant cities — undermines credibility with the ICP.

**Discovery sources:**

- Midrag (`midrag.co.il`) — 126K+ solar/electrical reviews. Solo operators are identifiable. Direct phone numbers available.
- Google Maps — search "טכנאי מזגנים נתניה", call small operators
- Personal network warm introductions
- Facebook professional groups for Israeli tradespeople

### 19.3 Discovery Script (15 ICs in 3 Weeks)

**Opening:** _"היי, קוראים לי נסאן, אני עושה מחקר על איך טכנאים עצמאיים מנהלים הזמנות. אין לי מה למכור לך — רק רוצה להבין את העבודה שלך. 15 דקות בטלפון?"_

**Questions:**

1. How do customers reach you today? Walk me through what happens after a new inquiry.
2. Tell me about the last time you missed a booking request — what happened?
3. What makes a day feel like a waste of time and fuel?
4. Have you ever paid for any business software?
5. If a tool managed bookings automatically and kept your jobs in the same area — what would it need to do for you to use it?

**Pivot triggers:**

- ICs shrug at missed bookings → wrong vertical or pain framing
- ICs care about bookings but not routing → lead with self-service, not route protection
- ICs describe a different primary pain → pivot the product hook

### 19.4 Beta Plan (Month 2–4)

5–10 ICs, free for 60 days, explicit feedback contract.

**The deal:** _"אני נותן לך את הפלטפורמה בחינם ל-60 יום. בתמורה — שיחה של 20 דקות כל שבועיים. תגיד לי מה עובד, מה לא, ומה חסר."_

**Good beta IC profile:** Tech-comfortable, 3–10 booking requests/week, opinionated and vocal, respected in trade community.

**Beta success criteria:**

- ≥3 of 5 beta ICs received at least 1 real booking via JOURN
- ≥3 of 5 say unprompted they want to keep using it
- Onboarding completes without founder involvement
- Zero routing failures or double-bookings

### 19.5 Deployment Architecture

**Domain:** `tryjourn.app` (confirmed available; .app TLD mandatory HTTPS via Cloudflare/Namecheap).

**Separation of concerns:**

- `tryjourn.app` (root `/`) — marketing landing page only. Public. No auth.
- `tryjourn.app/dashboard` (or `app.tryjourn.app`) — MVP app, behind auth gate.

**Access control during beta:**

- Middleware blocks all app routes for unauthenticated users → redirect to `/`
- Registration route (`/auth/signup`) blocked at middleware level
- tRPC `register` mutation rejects calls not accompanied by an approved waitlist token
- Waitlist `approved` boolean set manually in Supabase Studio per IC

**Waitlist → beta IC flow:**

1. IC fills form on landing page → row created in `waitlist` table
2. Founder sets `approved = true` in Supabase Studio
3. Founder sends personal WhatsApp message with direct signup link
4. IC completes onboarding — no self-serve registration possible otherwise

**Pending before go-live:**

- [ ] Supabase `waitlist` table creation
- [ ] tRPC `waitlist.submit` mutation
- [ ] Middleware file (`middleware.ts`) wired to auth session
- [ ] WhatsApp notification mock screenshots (3 messages: new booking, cancellation, daily summary)
- [ ] Domain purchase and Vercel DNS configuration

### 19.6 Growth Stages & Revenue

|Stage|Timeline|ICs|MRR (₪)|ARR ($)|
|---|---|---|---|---|
|0 — Discovery|Now → M2|0|0|0|
|0.5 — Beta|M2 → M4|5–10 free|0|0|
|1 — Beachhead|M4 → M8|30 paying|~₪4,300|~$14K|
|2 — Expansion|M8 → M16|150 paying|~₪25,000|~$84K|
|3 — Scale|M16 → M26|500 paying|~₪90,000|~$300K|

**Current stage:** Stage 0 — Discovery. Landing page built and copy-reviewed. Domain pending purchase (`tryjourn.app`). MVP app complete but gated — not publicly accessible.

**Pricing:**

- Founders Rate (first 20 ICs): ₪79 Solo / ₪159 Professional — locked for life
- Standard: ₪99 / ₪199 / ₪349
- Annual: 2 months free

**Key activation insight:** ICs who receive their first customer booking via JOURN within 14 days of signup retain at very high rates. ICs who don't churn within 60 days. Every touchpoint from onboarding through early support should be optimized toward engineering this first-booking moment as fast as possible.

### 19.6 Partnership Pipeline

**Midrag and Pro.co.il** have a structural leakage problem: customers find ICs on their platform, then bookings happen off-platform via WhatsApp. JOURN solves this leakage.

|Stage|Action|
|---|---|
|Stage 1 (M4–M8)|ICs add JOURN link to their Midrag profile manually|
|Stage 2 (M8–M16)|Pilot conversation with Midrag — "Book via JOURN" button on IC profiles|
|Stage 3 (M16+)|Formal partnership with revenue share or referral arrangement|

Discovery question for beta ICs: _"איך הגעת למידרג? האם זה מביא לך עבודה?"_ — validates both customer research and partnership hypothesis simultaneously.

---

## 20. Glossary

|Term|Definition|
|---|---|
|IC|Independent Contractor — solo field service professional|
|FSM|Field Service Management — enterprise software category|
|Route harm prevention|JOURN's MVP routing promise: customers can only book slots that are geographically feasible given the IC's existing schedule. Not full route optimization.|
|Instant Booking|Scheduling mode where customer picks exact date + time slot. Available all tiers.|
|Optimized Scheduling|Scheduling mode (`OPTIMIZE`) where customer selects arrival window constraints and the optimizer assigns the exact time (V2, Professional+ only).|
|Arrival Window|A named time range the IC commits to start within. Guaranteed start time, not completion time. Derived from resource WorkingHours — not a stored entity.|
|Urgency Level|Service-level classification: REGULAR (default) or EMERGENCY. Drives booking flow and storefront badge.|
|Fast Response|Storefront badge on EMERGENCY services. Static — no live availability check.|
|SCHEDULED|Appointment has an exact confirmed time on the Gantt. The only real confirmed state.|
|PENDING_APPROVAL|Instant Booking awaiting IC manual approval.|
|PENDING_OPTIMIZATION|Optimized Scheduling in optimizer queue — no time assigned yet (V2).|
|Schedule Settling Time|IC-configured SLA: hours to confirm a Optimized Scheduling. Drives V2 nudge and V3 auto-trigger.|
|Optimization Horizon|Days ahead the optimizer looks. Per-run parameter — never stored.|
|Plan Tier|Subscription level: Solo (₪99), Professional (₪199), Team (₪349). Gates features and resource/service limits.|
|Storefront|Public-facing booking page at `tryjourn.app/b/{slug}`|
|Magic Link|Token-based URL sent to customer via WhatsApp after booking. Powers self-service appointment page.|
|Visit fee|Fixed fee for IC to arrive and assess.|
|Job fee|Variable cost of the actual work. Quoted on-site.|
|Buffer|Time between job end and next job start, for travel and prep.|
|ServiceAreaRegion|Cached Google Places region. Keyed by `googlePlaceId`. Used for customer address matching.|
|Beachhead|The single vertical + geographic cluster used for initial market entry.|

---

_Document Version: 2.7_ _Created: 2026-03-15_ _Last Updated: 2026-03-29_ _Status: Active — canonical product reference_ _Aligned with: Tech Design v2.5, onboarding mock (journiq-onboarding.jsx), dashboard mock (journiq-unified.jsx)_

**Changelog:**

- v2.7 (2026-03-29): Brand renamed Journiq → JOURN. Domain locked to tryjourn.app. Landing page complete — Hebrew, dark theme, copy specialist reviewed. Deployment architecture documented (Section 19.5). WhatsApp mock screenshots pending. MVP scope table updated with landing page and pending items.
- v2.5 (2026-03-16): Previous version.