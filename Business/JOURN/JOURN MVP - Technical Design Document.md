**Created:** 2025-01-01 **Updated:** 2026-03-28T00:00 **Version:** 2.5 **Status:** Active

---

## Changelog

|Version|Date|Changes|
|---|---|---|
|2.5|2026-03-28|Product renamed from _Journiq_ to _JOURN_ throughout. All references updated. No schema, API, or logic changes. See PRD Section 1.1 for name rationale.|
|2.4|2026-03-16|Business-level home base removed. `defaultHomeBaseLat`/`defaultHomeBaseLng` removed from Business model. Resource `homeBaseLat`/`homeBaseLng` made non-nullable — required at resource creation. `business.create` and `business.update` input schemas updated. `getEffectiveSettings` no longer falls back to business default. Map center references updated to use first active resource.|
|2.3|2026-03-16|Business model: `defaultLanguage`, `aiEnabled`, `aiGreeting`, `notifyIcNewBooking`, `notifyIcDailyAgenda` fields added. `business.update` input extended with all new fields plus `windowSizeMinutes`, `scheduleSettlingHours`, `slotIntervalMinutes`, `bookingHorizonDays`, `cancellationWindowHours`. PRD Section 13 settings page fully specified (7 sections, Scheduling Logic merged, Danger Zone three-operation spec).|
|2.2|2026-03-16|Section 4.11.1 added: Slot Conflict Prevention. Two-layer solution: `pg_advisory_xact_lock(hashtext(resourceId), hashtext(date))` serialises concurrent booking attempts for same resource+date; overlap check queries existing SCHEDULED/PENDING_APPROVAL appointments for interval conflict. SLOT_TAKEN error triggers client re-fetch of available slots. No new index required — existing `[resourceId, scheduledStart]` index covers the overlap query.|
|2.1|2026-03-15|`MatrixSnapshot` type added. Embedded in `ResourceAvailability`. `appointment.create` extended with optional `matrixSnapshot`. Step 9 two-path logic: fresh snapshot → `applyMatrixSnapshotToChain` (0 ORS calls); expired/absent → `TravelChainService.rebuild` fallback (1 ORS call). No silent skips — chain always updated on create. Cancel paths unchanged.|
|1.9|2026-03-15|`SchedulingMode` enum: `INSTANT` → `SELECT_SLOT`, `WINDOW` → `OPTIMIZE`. Feature flag `windowBooking` → `optimizedScheduling` in `TIER_LIMITS` and all `assertTierFeature` calls. Prose: "Window Booking" → "Optimized Scheduling" throughout.|
|1.8|2026-03-14|Subscription tier model added: `PlanTier` enum, `subscriptionStatus`, `trialEndsAt`, `planTier` on Business model. `TIER_LIMITS` gate constants and `assertTierFeature`/`assertTierLimit` helpers specified. Seed test user provisioned at TEAM tier. `AppointmentStatus` enum overhauled: PENDING→PENDING_APPROVAL, CONFIRMED→SCHEDULED, PENDING_OPTIMIZATION added, EXPIRED/NO_SHOW folded into CANCELLED+cancellationReason. `Service` model: `urgencyLevel` (REGULAR/EMERGENCY) and `schedulingMode` (SELECT_SLOT/OPTIMIZE) fields added. `Business` model: `planTier`, `subscriptionStatus`, `trialEndsAt`, `windowSizeMinutes`, `scheduleSettlingHours` fields added. `optimizationHorizonDays` deliberately excluded — per-run parameter, not a persistent setting. `BookedAppointment`: `preferredWindows` Json field and `settlingDeadline` DateTime field added.|
|1.7|2026-03-13|Three changes: (1) `ResourceAbsence` schema — added optional `absenceLat`, `absenceLng`, `absenceAddress` fields for geocoded break locations; (2) `buildAnchorsForShift` updated — location propagation pass added: each anchor inherits the resolved location of the previous anchor in sequence unless it has an explicit geocoded location. `day_start` = home base always. Unlocated absences assume they occur at the previous anchor's location (e.g. break after appointment A is assumed at A's location). `day_end` resolves to last known location — not forced back to home base. Located absences (doctor, restaurant) override and propagate forward. (3) `Anchor` type extended with `locationType` field (`home_base`|
|1.6|2026-03-09|Three design decisions locked: (1) Service area validation moved to separate prior step — `geo.validateServiceArea` added, `getAvailableSlots` pre-condition documented, sequence diagram updated; (2) Supabase pg_cron chosen for all scheduled jobs — full spec added (6.3.5): approval expiry, 24h/2h reminders, API route pattern, env vars, DB settings; (3) TravelMode extended with MOTORCYCLE + WALKING, ORS profile mapping updated. BookedAppointment schema: reminder24hSentAt + reminder2hSentAt fields added with supporting indexes.|
|1.5|2026-03-09|Service area architecture overhaul: removed TravelTimeCache, Nominatim, polygon approach; PostGIS deferred. ServiceAreaRegion + BusinessServiceArea models added. Routing interface updated with departureTime. ServiceAreaPicker spec added (6.4.5).|
|1.4|2026-03-09|Added Google Places Autocomplete session token implementation spec (6.4.4)|
|1.3|2026-03-09|WhatsApp integration spec, geocoding layer, manual approval expiry, IC-create flow, magic link / appointment page, CustomerFlag and AppointmentToken models, API usage metering, confirmationMode scoped to business, service area UX aligned with onboarding mock, onboarding corrected to 7 steps|
|1.2|2026-03-05|Added Category model, updated Business model with categoryId FK, added category/explore routers, added seed data|
|1.1|2026-03-02|Previous updates|
|1.0|2025-01-01|Initial release|

---

## Table of Contents

1. [Executive Summary](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#1-executive-summary)
2. [System Architecture](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#2-system-architecture)
3. [Data Model](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#3-data-model)
4. [API Design](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#4-api-design)
5. [Service Layer Specifications](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#5-service-layer-specifications)
6. [Integration Specifications](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#6-integration-specifications)
7. [Security Considerations](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#7-security-considerations)
8. [Implementation Phases](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#8-implementation-phases)
9. [Project Structure](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#9-project-structure)
10. [Environment Configuration](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#10-environment-configuration)
11. [Deployment Guide](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#11-deployment-guide)
12. [Project Setup Guide](https://claude.ai/chat/04222eee-b6fa-4261-ba8b-168ec8e5946c#12-project-setup-guide)

---

## 1. Executive Summary

### 1.1 Product Overview

JOURN is a route-aware booking platform for field service professionals (independent contractors) in Israel. The platform enables customers to self-book services while automatically protecting the IC's daily route efficiency.

### 1.2 Core Value Proposition

- **For ICs**: Stop missing bookings, eliminate WhatsApp scheduling chaos, and prevent obviously bad route days — all without a dispatcher or complex software
- **For Customers**: Book services in under 2 minutes with guaranteed geographic availability

> **What "route-aware" means at MVP scope:** JOURN prevents customers from booking slots that would create geographically infeasible days. A customer in Tel Aviv cannot book a 10am slot if the IC has a 9am job in Netanya and an 11am in Ra'anana — that slot simply does not appear. The IC retains full control over their daily job sequence. This is route harm prevention, not fleet optimization — and for a solo IC currently managing their calendar on WhatsApp, it is a transformative improvement.

### 1.3 MVP Scope

|Feature|Included|Notes|
|---|---|---|
|IC Onboarding|✅|7-step wizard|
|Service Management|✅|CRUD operations|
|Resource Management (post-onboarding)|✅|Owner-only: add/edit/deactivate/delete resources, manage working hours and absences|
|Schedule View|✅|Week overview (Gantt) + Day detail with inline break management|
|Route-Aware Slot Generation|✅|Full bidirectional travel validation|
|Customer Booking|✅|Guest booking (no auth required)|
|Auto/Manual Confirmation|✅|IC-configurable|
|Email Notifications|✅|Via Twilio SendGrid (secondary to WhatsApp)|
|AI Chat (Informational)|✅|Claude-powered, no transactional booking|
|Resource Dashboard Login|❌|Post-MVP|
|Resource-Level Service Area Override|❌|Post-MVP|
|WhatsApp Notifications|✅|MVP — primary notification channel (Twilio BSP)|
|Reschedule Flows|❌|Post-MVP|
|Payment Integration|❌|Post-MVP|

### 1.4 Technical Stack

|Layer|Technology|
|---|---|
|**Framework**|Next.js 14 (App Router)|
|**API**|tRPC v11|
|**Database**|PostgreSQL via Supabase|
|**ORM**|Prisma|
|**Auth**|Custom (email/password) with JWT|
|**File Storage**|Supabase Storage|
|**Routing API**|OpenRouteService (OSRM-ready)|
|**AI**|Claude claude-sonnet-4-20250514|
|**Email**|Twilio SendGrid|
|**Styling**|Tailwind CSS|
|**Language**|TypeScript|
|**Deployment**|AWS (ECS/Fargate or Amplify)|

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENTS                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐          │
│  │   IC Dashboard   │  │  Customer        │  │   Landing Page   │          │
│  │   (Authenticated)│  │  Booking Page    │  │   (Marketing)    │          │
│  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘          │
└───────────┼─────────────────────┼─────────────────────┼─────────────────────┘
            └──────────────────┬──┴─────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           NEXT.JS APPLICATION                               │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         tRPC API Layer                               │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │   │
│  │  │  auth   │ │business │ │ service │ │ booking │ │   chat  │       │   │
│  │  │ router  │ │ router  │ │ router  │ │ router  │ │ router  │       │   │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘       │   │
│  └───────┼──────────┼──────────┼──────────┼──────────┼────────────────┘   │
│  ┌───────┴──────────┴──────────┴──────────┴──────────┴────────────────┐   │
│  │                       Service Layer                                 │   │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐       │   │
│  │  │  Routing   │ │   Slot     │ │Notification│ │    AI      │       │   │
│  │  │  Service   │ │ Generator  │ │  Service   │ │  Service   │       │   │
│  │  └────────────┘ └────────────┘ └────────────┘ └────────────┘       │   │
│  └────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         EXTERNAL SERVICES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   Supabase   │  │ OpenRoute    │  │   Twilio     │  │  Anthropic   │   │
│  │  (Postgres)  │  │  Service     │  │  SendGrid    │  │   Claude     │   │
│  │  (Storage)   │  │  (Routing)   │  │  (Email)     │  │   (AI)       │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Request Flow - Get Available Slots

```
┌──────────┐          ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│ Customer │          │   Next.js    │          │   Service    │          │   External   │
│          │          │   (tRPC)     │          │    Layer     │          │   Services   │
└────┬─────┘          └──────┬───────┘          └──────┬───────┘          └──────┬───────┘
     │                       │                         │                         │
     │ 1. geo.getPlaceDetails│                         │                         │
     │   (address selection) │                         │                         │
     │──────────────────────>│                         │                         │
     │                       │─────────────────────────────────────────────────>│
     │                       │  Google Places Details (lat/lng + addressComponents)
     │                       │<─────────────────────────────────────────────────│
     │<──────────────────────│                         │                         │
     │                       │                         │                         │
     │ 2. geo.validateService│                         │                         │
     │    Area (businessId,  │                         │                         │
     │    addressComponents) │                         │                         │
     │──────────────────────>│                         │                         │
     │                       │ GeoService.isInService  │                         │
     │                       │ Area() — DB lookup +    │                         │
     │                       │ in-memory component     │                         │
     │                       │ match (no external call)│                         │
     │                       │────────────────────────>│                         │
     │                       │<────────────────────────│                         │
     │  { inArea: false } ───│                         │                         │
     │  → show out-of-area   │                         │                         │
     │    screen             │                         │                         │
     │  { inArea: true } ────│                         │                         │
     │  → proceed to slots   │                         │                         │
     │                       │                         │                         │
     │ 3. getAvailableSlots  │                         │                         │
     │   (service, location) │                         │                         │
     │   PRE-CONDITION:      │                         │                         │
     │   area already valid  │                         │                         │
     │──────────────────────>│                         │                         │
     │                       │                         │                         │
     │                       │ 4. slotGenerator.       │                         │
     │                       │    generate()           │                         │
     │                       │────────────────────────>│                         │
     │                       │                         │                         │
     │                       │                         │ 5. Load resources,      │
     │                       │                         │    appointments,        │
     │                       │                         │    absences (Supabase)  │
     │                       │                         │────────────────────────>│
     │                       │                         │<────────────────────────│
     │                       │                         │                         │
     │                       │                         │ 6. Get travel times     │
     │                       │                         │    (OpenRouteService)   │
     │                       │                         │────────────────────────>│
     │                       │                         │<────────────────────────│
     │                       │                         │                         │
     │                       │                         │ 7. Build anchors,       │
     │                       │                         │    find gaps,           │
     │                       │                         │    generate slots       │
     │                       │                         │         (in memory)     │
     │                       │                         │                         │
     │                       │ 8. Return slots         │                         │
     │                       │    grouped by resource  │                         │
     │                       │<────────────────────────│                         │
     │                       │                         │                         │
     │ 9. Display available  │                         │                         │
     │    time slots         │                         │                         │
     │<──────────────────────│                         │                         │
     │                       │                         │                         │
```

### 2.3 Request Flow - Create Booking

```
┌──────────┐          ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│ Customer │          │   Next.js    │          │   Service    │          │   External   │
│          │          │   (tRPC)     │          │    Layer     │          │   Services   │
└────┬─────┘          └──────┬───────┘          └──────┬───────┘          └──────┬───────┘
     │                       │                         │                         │
     │ 1. appointment.create │                         │                         │
     │   (resourceId, slot,  │                         │                         │
     │    customer details)  │                         │                         │
     │──────────────────────>│                         │                         │
     │                       │                         │                         │
     │                       │ 2. Start DB transaction │                         │
     │                       │────────────────────────>│                         │
     │                       │                         │                         │
     │                       │                         │ 3. SELECT FOR UPDATE    │
     │                       │                         │    (lock time slot)     │
     │                       │                         │────────────────────────>│
     │                       │                         │<────────────────────────│
     │                       │                         │                         │
     │                       │                         │ 4. Re-validate slot     │
     │                       │                         │    still available      │
     │                       │                         │    for this resource    │
     │                       │                         │                         │
     │                       │                         │ 5. INSERT appointment   │
     │                       │                         │    (Supabase)           │
     │                       │                         │────────────────────────>│
     │                       │                         │<────────────────────────│
     │                       │                         │                         │
     │                       │ 6. COMMIT transaction   │                         │
     │                       │<────────────────────────│                         │
     │                       │                         │                         │
     │                       │                         │ 7. Send confirmation    │
     │                       │                         │    emails (SendGrid)    │
     │                       │                         │    - to customer        │
     │                       │                         │    - to resource/owner  │
     │                       │                         │────────────────────────>│
     │                       │                         │                         │
     │ 8. Return booking     │                         │                         │
     │    confirmation       │                         │                         │
     │<──────────────────────│                         │                         │
     │                       │                         │                         │
```

---

## 3. Data Model

### 3.1 Domain Terminology

|Term|Definition|
|---|---|
|**Business**|A service company (e.g., "David's Plumbing")|
|**Resource**|A service provider/technician working for a business (max 5 per business)|
|**Service**|A type of work offered (e.g., "Leak Repair")|
|**ResourceAbsence**|Time when a resource is unavailable (lunch, vacation, etc.)|
|**BookedAppointment**|A confirmed or pending customer appointment|

### 3.2 Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────────┐
│      User       │       │    Business     │       │      Resource       │
├─────────────────┤       ├─────────────────┤       ├─────────────────────┤
│ id              │──────<│ ownerId         │       │ id                  │
│ email           │       │ id              │>──────│ businessId          │
│ passwordHash    │       │ name            │       │ userId              │>──┐
│ role            │       │ slug            │       │ name                │   │
│ createdAt       │       │ serviceArea*    │       │ homeBaseLat/Lng     │   │
└────────┬────────┘       │ timezone        │       │ isActive            │   │
         │                │ ...             │       └──────────┬──────────┘   │
         │                └────────┬────────┘                  │              │
         │                         │                           │              │
         └─────────────────────────┼───────────────────────────┼──────────────┘
                                   │                           │
              ┌────────────────────┤                           │
              │                    │              ┌────────────┴────────────┐
              ▼                    │              │                         │
┌─────────────────┐                │              ▼                         ▼
│    Service      │                │  ┌─────────────────────┐   ┌─────────────────────┐
├─────────────────┤                │  │    WorkingHours     │   │   ResourceAbsence   │
│ id              │                │  ├─────────────────────┤   ├─────────────────────┤
│ businessId      │<───────────────┤  │ id                  │   │ id                  │
│ name            │                │  │ resourceId          │   │ resourceId          │
│ description     │                │  │ dayOfWeek (0-6)     │   │ type                │
│ durationMinutes │                │  │ startTime (DateTime)│   │ dayOfWeek / date    │
│ visitFee        │                │  │ endTime (DateTime)  │   │ startTime (DateTime)│
│ ...             │                │  └─────────────────────┘   │ endTime (DateTime)  │
└─────────────────┘                │   * Multiple rows per day  │ reason              │
                                   │     for split shifts       └─────────────────────┘
                                   ▼
┌─────────────────────────┐      ┌──────────────────────────┐
│   BookedAppointment     │      │   BusinessServiceArea    │
├─────────────────────────┤      ├──────────────────────────┤
│ id                      │      │ id                       │
│ businessId              │      │ businessId               │
│ resourceId              │      │ regionId                 │
│ serviceId               │      └──────────────────────────┘
│ scheduledStart (DT)     │               │ N:1
│ scheduledEnd (DT)       │               ▼
│ customerName            │      ┌──────────────────────────┐
│ status                  │      │   ServiceAreaRegion      │
│ ...                     │      ├──────────────────────────┤
└─────────────────────────┘      │ googlePlaceId (unique)   │
                                 │ label / labelHe          │
                                 │ componentType            │
                                 │ componentValue           │
                                 └──────────────────────────┘

* serviceArea defined at Business level via BusinessServiceArea join table
* Resource-level service area override is post-MVP (Professional/Team tiers)
* All times are in business timezone (MVP); multi-timezone support is post-MVP
```

### Key Relationships:

- **Business → Resources**: One-to-many (max 5 resources per business in MVP)
- **Resource → WorkingHours**: One-to-many (multiple shifts per day allowed)
- **Resource → ResourceAbsence**: One-to-many (recurring or one-time blocks)
- **Resource → BookedAppointment**: One-to-many (all appointments assigned to a resource)
- **Business → Services**: One-to-many (services available for the business)
- **User → Resource**: One-to-one optional (resource can have login access)

### WorkingHours Design:

The WorkingHours model supports **multiple shifts per day** for maximum flexibility:

```
Example: Resource works split shifts on Monday
┌────────────┬───────────┬───────────┬───────────┐
│ resourceId │ dayOfWeek │ startTime │ endTime   │
├────────────┼───────────┼───────────┼───────────┤
│ res_abc    │ 1 (Mon)   │ 08:00     │ 12:00     │  ← Morning shift
│ res_abc    │ 1 (Mon)   │ 14:00     │ 18:00     │  ← Afternoon shift
│ res_abc    │ 2 (Tue)   │ 08:00     │ 17:00     │  ← Full day
│ res_abc    │ 3 (Wed)   │ 08:00     │ 17:00     │
└────────────┴───────────┴───────────┴───────────┘
```

**Timezone Note (MVP)**: All times stored in business timezone. The Business model has a `timezone` field (e.g., "Asia/Jerusalem"). Multi-timezone support (resources in different timezones) is post-MVP.

### Settings Inheritance:

Resources inherit settings from their Business but can optionally override:

- **Home base location** (where the resource starts/ends their day)
- **Travel mode** (driving, cycling electric, cycling regular)
- **Parking buffer** (minutes added for parking at customer location)
- **Buffer minutes** (gap between appointments)
- **Service area** (POST-MVP: where the resource provides service)

**MVP Note**: For MVP, service area is only configurable at the business level. All resources use the business service area. Resource-level service area overrides are planned for post-MVP.

### 3.3 Prisma Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

// ============================================================================
// USER & AUTHENTICATION
// ============================================================================

model User {
  id           String    @id @default(cuid())
  email        String    @unique
  passwordHash String
  role         UserRole  @default(RESOURCE)
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt

  // A user who owns a business
  ownedBusiness Business? @relation("BusinessOwner")
  
  // A user who is a resource in a business
  resource      Resource?
  
  sessions      Session[]

  @@map("users")
}

enum UserRole {
  OWNER     // Business owner - full access
  RESOURCE  // Team member - limited access
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique
  expiresAt DateTime
  createdAt DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([token])
  @@index([userId])
  @@map("sessions")
}

// ============================================================================
// CATEGORY (managed by JOURN - not user-editable)
// ============================================================================

model Category {
  id         String    @id @default(cuid())
  slug       String    @unique  // "plumbing", "electrical", etc.
  icon       String              // Emoji: "🔧"
  names      Json                // { "en": "Plumbing", "he": "אינסטלציה" }
  sortOrder  Int       @default(0)
  isActive   Boolean   @default(true)
  
  createdAt  DateTime  @default(now())
  updatedAt  DateTime  @updatedAt

  businesses Business[]

  @@index([slug])
  @@index([isActive, sortOrder])
  @@map("categories")
}

// ============================================================================
// BUSINESS
// ============================================================================

model Business {
  id       String @id @default(cuid())
  ownerId  String @unique  // The user who owns this business
  
  // Profile
  name     String
  slug     String @unique
  
  // Category (REQUIRED - select from Category table)
  categoryId    String
  category      Category @relation(fields: [categoryId], references: [id])
  categoryOther String?  // If "Other" selected, capture what they wanted
  
  tagline  String?
  logoUrl  String?
  
  // Storefront customization
  socialLinks Json?     // { instagram: "...", facebook: "...", ... }
  gallery     Json?     // [{ type: "image", url: "..." }, ...]
  showAvailability Boolean @default(true)
  
  // Location — business-level home base removed.
  // Each Resource must have homeBaseLat/homeBaseLng set (non-nullable).
  // There is no business-level fallback — resource creation requires home base.
  timezone           String  // e.g., "Asia/Jerusalem"
  
  // Contact fallback
  fallbackContactMethod ContactMethod
  fallbackContactValue  String
  
  // Subscription tier
  planTier              PlanTier   @default(SOLO)
  subscriptionStatus    SubscriptionStatus @default(TRIAL)
  trialEndsAt           DateTime?  // Null after trial converts to paid

  // Settings (business-wide defaults)
  currency              String  @default("ILS")
  defaultTravelMode     TravelMode @default(DRIVING_CAR)
  defaultParkingBuffer  Int     @default(5)      // minutes
  defaultBufferMinutes  Int     @default(15)     // between jobs
  slotIntervalMinutes   Int     @default(30)
  bookingHorizonDays    Int     @default(14)
  confirmationMode      ConfirmationMode @default(AUTO)
  approvalTimeoutHours  Int     @default(2)
  cancellationWindowHours Int   @default(24)
  allowReschedule       Boolean @default(true)
  remindersEnabled      Boolean @default(true)
  rescheduleWindowDays  Int     @default(1)

  // Schedule Optimization settings (Professional+ tier)
  windowSizeMinutes        Int  @default(120)  // Arrival window duration in minutes — divides working hours into equal segments
  scheduleSettlingHours    Int  @default(24)   // SLA shown to customer: "We'll confirm within X hours"
  // NOTE: optimizationHorizonDays is NOT stored here.
  // It is a per-run parameter passed when the IC triggers optimization.
  // The UI suggests a default of 3 days but the IC chooses freely each time.
  
  // Service Area — defined via BusinessServiceArea join table
  // MVP: business-level only. Post-MVP: resource-level override (Professional/Team tiers)
  
  // Language & localisation
  defaultLanguage       String  @default("he")  // "he" | "en" — selects WhatsApp template variant

  // AI Grounding
  aiEnabled             Boolean @default(true)
  aiGreeting            String?
  faqText               String?
  specialInstructions   String?

  // IC notification preferences
  notifyIcNewBooking    Boolean @default(true)  // WhatsApp alert on every new booking
  notifyIcDailyAgenda   Boolean @default(true)  // Morning summary at 7 AM
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  owner               User                  @relation("BusinessOwner", fields: [ownerId], references: [id], onDelete: Cascade)
  resources           Resource[]
  services            Service[]
  bookedAppointments  BookedAppointment[]
  serviceAreas        BusinessServiceArea[]

  @@index([slug])
  @@index([categoryId])
  @@map("businesses")
}

enum ContactMethod {
  PHONE
  WHATSAPP
  EMAIL
}

enum TravelMode {
  DRIVING_CAR
  MOTORCYCLE
  CYCLING_ELECTRIC
  CYCLING_REGULAR
  WALKING
}

enum ConfirmationMode {
  AUTO
  MANUAL
}

enum PlanTier {
  SOLO          // 1 resource, 5 services, no AI, no window booking
  PROFESSIONAL  // 3 resources, unlimited services, AI chat, window booking
  TEAM          // 5 resources, unlimited services, AI chat, window booking, cross-resource optimization
}

enum SubscriptionStatus {
  TRIAL    // Within 14-day free trial (provisioned at Professional features)
  ACTIVE   // Paid subscription
  LOCKED   // Trial expired or payment failed — data retained, no new bookings
}

// ============================================================================
// SERVICE AREA (region lookup cache + business membership)
// ============================================================================

// Lazily-populated cache of Google Places regions resolved to address_component values.
// A region is inserted once (on first selection by any IC) and reused forever.
// No polygons — matching uses Google's own address_component strings, eliminating
// any cross-source naming mismatch.
//
// POST-MVP: PostGIS geometry column may be added here for spatial queries at global scale.

model ServiceAreaRegion {
  id             String   @id @default(cuid())
  googlePlaceId  String   @unique          // Google Place ID — canonical dedup key
  label          String                    // Display name, e.g. "Tel Aviv-Yafo"
  labelHe        String?                   // Hebrew label, e.g. "תל אביב-יפו"
  componentType  String                    // Google type: "locality" | "administrative_area_level_1" etc.
  componentValue String                    // Exact Google address_component long_name value
  countryCode    String   @default("IL")
  createdAt      DateTime @default(now())

  businessAreas  BusinessServiceArea[]

  @@index([countryCode])
  @@index([googlePlaceId])
  @@map("service_area_regions")
}

// Join table: many businesses ↔ many regions
model BusinessServiceArea {
  id         String @id @default(cuid())
  businessId String
  regionId   String

  business   Business          @relation(fields: [businessId], references: [id], onDelete: Cascade)
  region     ServiceAreaRegion @relation(fields: [regionId], references: [id])

  @@unique([businessId, regionId])
  @@index([businessId])
  @@map("business_service_areas")
}

// ============================================================================
// RESOURCE (service provider / technician)
// ============================================================================

model Resource {
  id         String  @id @default(cuid())
  businessId String
  userId     String? @unique  // Optional: linked user account for login
  
  // Profile
  name       String           // Display name
  email      String?          // Contact email (may differ from user email)
  phone      String?          // Contact phone
  role       String?          // e.g., "Senior Plumber", "Apprentice"
  avatarUrl  String?
  
  // Location (resource's home base - can differ from business HQ)
  homeBaseLat Float           // Required — no business-level fallback
  homeBaseLng Float
  
  // Settings (resource-specific overrides)
  travelMode        TravelMode?  // If null, uses business default
  parkingBuffer     Int?         // If null, uses business default
  bufferMinutes     Int?         // If null, uses business default
  
  // Service Area override (POST-MVP — Professional/Team tiers only)
  // At MVP, resources inherit the business service area.
  // Post-MVP: resource selects their own regions via BusinessServiceArea-equivalent join.
  // No fields needed here at MVP — override lookup handled in slot generator.
  
  // Status
  isActive   Boolean @default(true)
  sortOrder  Int     @default(0)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  business            Business              @relation(fields: [businessId], references: [id], onDelete: Cascade)
  user                User?                 @relation(fields: [userId], references: [id], onDelete: SetNull)
  workingHours        WorkingHours[]        // Multiple shifts per day supported
  absences            ResourceAbsence[]
  bookedAppointments  BookedAppointment[]

  @@index([businessId])
  @@map("resources")
}

// ============================================================================
// WORKING HOURS (per Resource - supports multiple shifts per day)
// ============================================================================

model WorkingHours {
  id         String   @id @default(cuid())
  resourceId String
  
  dayOfWeek  Int      // 0 = Sunday, 6 = Saturday
  startTime  DateTime // Time portion only (date part ignored)
  endTime    DateTime // Time portion only (date part ignored)
  
  // Note: All times are in business timezone (MVP)
  // POST-MVP: Add timezone support for multi-timezone businesses
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  resource Resource @relation(fields: [resourceId], references: [id], onDelete: Cascade)

  // No unique constraint on [resourceId, dayOfWeek] - allows multiple shifts per day
  // Example: Morning shift 08:00-12:00, Afternoon shift 14:00-18:00
  @@index([resourceId, dayOfWeek])
  @@index([resourceId])
  @@map("working_hours")
}

// ============================================================================
// SERVICE
// ============================================================================

model Service {
  id              String  @id @default(cuid())
  businessId      String
  
  name            String
  description     String
  durationMinutes Int
  
  // Pricing
  visitFee        Decimal @db.Decimal(10, 2)
  visitFeePolicy  VisitFeePolicy
  jobFeeMin       Decimal @db.Decimal(10, 2)
  jobFeeMax       Decimal @db.Decimal(10, 2)
  
  // Requirements
  requiredInputs  String[] @default([])
  
  // Confirmation message shown to customer after booking
  confirmationNote    String?  @db.VarChar(300)  // Optional per-service message (e.g. "Bring access code")

  // Urgency and scheduling mode
  urgencyLevel    UrgencyLevel    @default(REGULAR)   // Drives booking flow and storefront badge
  schedulingMode  SchedulingMode  @default(SELECT_SLOT)   // OPTIMIZE only available on Professional+ tier; ignored for EMERGENCY services

  // Status
  isActive        Boolean @default(true)
  sortOrder       Int     @default(0)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  business            Business              @relation(fields: [businessId], references: [id], onDelete: Cascade)
  bookedAppointments  BookedAppointment[]

  @@index([businessId])
  @@map("services")
}

enum VisitFeePolicy {
  ALWAYS_CHARGED
  WAIVED_IF_ACCEPTED
}

enum UrgencyLevel {
  REGULAR    // Default — IC chooses scheduling mode; no storefront badge
  EMERGENCY  // Always Instant Booking; shows "Fast Response" badge on storefront
}

enum SchedulingMode {
  SELECT_SLOT // Customer picks exact date + time slot
  WINDOW   // Customer picks preferred days + arrival windows (Professional+ only)
}

// ============================================================================
// RESOURCE ABSENCE (lunch breaks, vacation, appointments)
// ============================================================================

model ResourceAbsence {
  id         String @id @default(cuid())
  resourceId String
  
  type       AbsenceType
  
  // For recurring (e.g., daily lunch break)
  dayOfWeek  Int?    // 0-6, null if one-time
  
  // For one-time (e.g., doctor appointment, vacation day)
  date       DateTime? @db.Date
  
  // Time range (DateTime used for time portion only)
  // For recurring: date portion ignored, only time matters
  // For one-time: combined with date field for full datetime
  startTime  DateTime  // e.g., "2000-01-01T12:00:00Z" - only time portion used
  endTime    DateTime  // e.g., "2000-01-01T13:00:00Z" - only time portion used
  
  reason     String? // "Lunch", "Doctor appointment", "Vacation"

  // Optional geocoded location for this break.
  // When provided, the slot algorithm uses this as the routing anchor
  // for gaps adjacent to this absence instead of the resource home base.
  // Example use cases: lunch at home, supplier visit, school pickup.
  // Uses the same Google Places address search as the rest of the product.
  absenceAddress String?  // Human-readable address string
  absenceLat     Float?   // Geocoded latitude
  absenceLng     Float?   // Geocoded longitude

  // Travel chain fields — populated by TravelChainService after any scheduling operation.
  // See TravelChainService for full chain rule documentation.
  // travelToMinutes:   travel from previous chain event to this absence (null if non-located)
  // travelFromMinutes: travel from this absence to home base — populated on LAST event only
  travelToMinutes   Int?
  travelFromMinutes Int?
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  resource Resource @relation(fields: [resourceId], references: [id], onDelete: Cascade)

  @@index([resourceId])
  @@index([resourceId, date])
  @@map("resource_absences")
}

enum AbsenceType {
  RECURRING   // Repeats weekly (e.g., lunch break every day)
  ONE_TIME    // Single occurrence (e.g., vacation, appointment)
}

// ============================================================================
// BOOKED APPOINTMENT
// ============================================================================

model BookedAppointment {
  id         String @id @default(cuid())
  businessId String
  resourceId String  // The resource performing this service
  serviceId  String
  
  // Schedule
  scheduledStart DateTime
  scheduledEnd   DateTime
  
  // Customer info
  customerName    String
  customerPhone   String
  customerEmail   String?
  customerAddress String
  customerLat     Float
  customerLng     Float
  customerNotes   String?
  
  // Pricing snapshot (at time of booking)
  visitFee        Decimal @db.Decimal(10, 2)
  visitFeePolicy  VisitFeePolicy
  jobFeeMin       Decimal @db.Decimal(10, 2)
  jobFeeMax       Decimal @db.Decimal(10, 2)
  currency        String
  
  // IC-created appointment fields
  icNotes         String?   // Internal notes added by IC at booking time (not shown to customer)
  createdByIc     Boolean   @default(false)  // true if IC created via dashboard (always auto-confirmed)
  
  // Status
  status              AppointmentStatus @default(PENDING_APPROVAL)
  confirmationMode    ConfirmationMode
  approvalDeadline    DateTime?         // End of same or following business day (manual mode)

  // Window Booking fields (PENDING_OPTIMIZATION appointments only)
  // Null for Instant Booking appointments
  preferredWindows  Json?     // WindowPreference[] — { dayOfWeek, startTime, endTime }[]
                              // Always drawn from business-derived window boundaries
  settlingDeadline  DateTime? // createdAt + business.scheduleSettlingHours
                              // V2: dashboard nudge trigger; V3: auto-optimization trigger
  
  // Rescheduling
  rescheduledFromId   String?           // Previous appointment ID if this is a reschedule

  // Travel chain fields — populated by TravelChainService after any scheduling operation.
  // See TravelChainService for full chain rule documentation.
  // travelToMinutes:   travel from previous chain event to this appointment
  // travelFromMinutes: travel from this appointment to next event — populated on LAST event only
  travelToMinutes   Int?
  travelFromMinutes Int?

  // Reminder tracking (prevents duplicate sends)
  reminder24hSentAt   DateTime?
  reminder2hSentAt    DateTime?

  // Cancellation tracking
  cancelledAt         DateTime?
  cancelledBy         CancellationActor?
  cancellationReason  String?
  isLateCancellation  Boolean @default(false)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  business          Business           @relation(fields: [businessId], references: [id], onDelete: Cascade)
  resource          Resource           @relation(fields: [resourceId], references: [id])
  service           Service            @relation(fields: [serviceId], references: [id])
  appointmentToken  AppointmentToken?
  customerFlag      CustomerFlag?

  @@index([businessId])
  @@index([resourceId])
  @@index([resourceId, scheduledStart])
  @@index([businessId, status])
  @@index([status, approvalDeadline])              // PENDING_APPROVAL expiry job
  @@index([status, settlingDeadline])              // PENDING_OPTIMIZATION settling nudge/auto-trigger
  @@index([status, scheduledStart, reminder24hSentAt]) // reminder job query
  @@index([status, scheduledStart, reminder2hSentAt])  // reminder job query
  @@map("booked_appointments")
}

enum AppointmentStatus {
  PENDING_APPROVAL      // Instant Booking + manual confirmation mode — awaiting IC action
  PENDING_OPTIMIZATION  // Window Booking — in optimizer queue, no exact time assigned yet
  SCHEDULED             // Exact time assigned and on the Gantt — the confirmed real state
  CANCELLED             // Cancelled by any party at any stage; reason in cancellationReason field
  COMPLETED             // Service delivered
  // NOTE: EXPIRED and NO_SHOW are not separate statuses.
  // They are stored as cancellationReason values on CANCELLED appointments.
}

enum CancellationActor {
  CUSTOMER
  BUSINESS  // Owner or resource
  SYSTEM    // Auto-expired
}

// ============================================================================
// APPOINTMENT TOKEN (magic link for customer appointment page)
// ============================================================================

model AppointmentToken {
  id            String   @id @default(cuid())
  appointmentId String   @unique
  token         String   @unique  // HMAC-signed, URL-safe
  expiresAt     DateTime           // 7 days after appointment scheduledStart
  createdAt     DateTime @default(now())

  appointment BookedAppointment @relation(fields: [appointmentId], references: [id], onDelete: Cascade)

  @@index([token])
  @@map("appointment_tokens")
}

// ============================================================================
// CUSTOMER FLAG (soft flag scoped per business — late cancellations etc.)
// ============================================================================

model CustomerFlag {
  id            String   @id @default(cuid())
  phone         String               // Customer phone — lookup key
  businessId    String
  appointmentId String   @unique     // Which appointment triggered the flag
  reason        String               // e.g., "late_cancellation", "no_show"
  createdAt     DateTime @default(now())

  business    Business          @relation(fields: [businessId], references: [id], onDelete: Cascade)
  appointment BookedAppointment @relation(fields: [appointmentId], references: [id], onDelete: Cascade)

  @@index([phone, businessId])
  @@index([businessId])
  @@map("customer_flags")
}

// ============================================================================
// API USAGE METERING (Google Places sessions per business — billing deferred to post-MVP)
// ============================================================================

model ApiUsageEvent {
  id         String   @id @default(cuid())
  businessId String
  provider   String   // "google_places"
  operation  String   // "autocomplete_session" | "region_lookup"
  callCount  Int      @default(1)
  createdAt  DateTime @default(now())

  business Business @relation(fields: [businessId], references: [id], onDelete: Cascade)

  @@index([businessId, createdAt])
  @@index([createdAt])
  @@map("api_usage_events")
}
```

### 3.4 Type Definitions

```typescript
// src/types/index.ts

// Coordinate type used throughout the application
export interface Coordinate {
  lat: number;
  lng: number;
}

// ============================================================================
// Category Types
// ============================================================================

export interface Category {
  id: string;
  slug: string;
  icon: string;
  names: {
    en: string;
    he: string;
    [key: string]: string;
  };
  sortOrder: number;
  isActive: boolean;
}

export interface CategoryDisplay {
  id: string;
  slug: string;
  icon: string;
  name: string;  // Localized name
}

export interface CategoryWithCount extends CategoryDisplay {
  count: number;
}

// Helper function
export function getCategoryName(
  category: Category, 
  locale: string = 'en'
): string {
  return category.names[locale] || category.names['en'];
}

// ============================================================================
// Service Area Types
// ============================================================================

// A resolved region from Google Places — stored in ServiceAreaRegion table.
// componentValue is the exact Google address_component long_name used for matching.
export interface ServiceAreaRegion {
  id: string;
  googlePlaceId: string;
  label: string;           // Display name: "Tel Aviv-Yafo"
  labelHe?: string;        // Hebrew: "תל אביב-יפו"
  componentType: string;   // "locality" | "administrative_area_level_1"
  componentValue: string;  // Exact Google long_name — used for point-in-area check
  countryCode: string;
}

// Address component returned by Google Places Details
export interface GoogleAddressComponent {
  long_name: string;
  short_name: string;
  types: string[];
}

// Service area check result
export interface ServiceAreaCheckResult {
  inArea: boolean;
  matchedRegion?: ServiceAreaRegion; // Which region matched (for display)
}

// ============================================================================
// Working Hours (separate table - supports multiple shifts per day)
// ============================================================================

// Single working hours entry (one shift)
export interface WorkingHoursEntry {
  id: string;
  resourceId: string;
  dayOfWeek: number;      // 0 = Sunday, 6 = Saturday
  startTime: Date;        // Time portion only (date ignored)
  endTime: Date;          // Time portion only (date ignored)
}

// Working hours grouped by day (for UI display)
export interface DayShifts {
  dayOfWeek: number;
  dayName: string;        // "Sunday", "Monday", etc.
  shifts: Array<{
    id: string;
    startTime: Date;
    endTime: Date;
  }>;
  isWorking: boolean;     // true if any shifts exist for this day
}

// Full week schedule (for UI)
export type WeekSchedule = DayShifts[];  // Always 7 elements, one per day

// ============================================================================
// Resource Types
// ============================================================================

// Resource representation (for API responses)
export interface ResourceSummary {
  id: string;
  name: string;
  role: string | null;
  avatarUrl: string | null;
}

// Full resource with working hours
export interface Resource {
  id: string;
  businessId: string;
  name: string;
  email: string | null;
  phone: string | null;
  role: string | null;
  avatarUrl: string | null;
  homeBaseLat: number | null;
  homeBaseLng: number | null;
  workingHours: WorkingHours;
  travelMode: TravelMode | null;
  parkingBuffer: number | null;
  bufferMinutes: number | null;
  isActive: boolean;
  sortOrder: number;
}

// ============================================================================
// Slot Generation Types
// ============================================================================

/**
 * Serialisable snapshot of the ORS matrix computed during getAvailableSlots.
 * Passed client-side and returned in appointment.create so the create path
 * can rebuild the full travel chain without a second ORS call.
 *
 * TTL: 10 minutes. appointment.create checks (Date.now() - timestamp) before
 * using. If expired, chain update is skipped — appointment still creates
 * cleanly, travels remain stale until next TravelChainService.rebuild.
 *
 * Payload size: ~200 bytes for a typical 5×5 matrix — negligible.
 *
 * Future traffic-aware path: when ORS is replaced with a traffic-aware
 * provider (Google Routes), these durations will be traffic-adjusted at
 * slot generation time. The create path is identical — no structural change.
 */
export interface MatrixSnapshot {
  resourceId: string;
  date: string;           // ISO date — guards against stale matrix used for wrong day
  timestamp: number;      // Date.now() at matrix computation time — used for TTL check
  locations: Array<{
    lat: number;
    lng: number;
    type: 'home_base' | 'appointment' | 'located_absence';
    refId?: string;       // appointmentId or absenceId — used when persisting chain updates
  }>;
  durations: number[][];  // [fromIndex][toIndex] = minutes. Same index space as locations[].
}

// Slot representation
export interface Slot {
  date: Date;              // The date of the slot
  startTime: Date;         // Full datetime of slot start
  endTime: Date;           // Full datetime of slot end
  travelFromPrevious: {
    durationMinutes: number;
    fromType: 'home_base' | 'previous_appointment';
  };
  travelToNext: {
    durationMinutes: number;
    toType: 'home_base' | 'next_appointment';
  };
  isFirstOfDay: boolean;
  isLastOfDay: boolean;
}

// Day availability (per resource)
export interface DayAvailability {
  date: Date;
  dayOfWeek: number;       // 0-6
  status: 'available' | 'not_working' | 'fully_booked';
  workingHours?: {
    start: Date;           // First shift start
    end: Date;             // Last shift end
    shifts: Array<{        // Individual shift windows
      start: Date;
      end: Date;
    }>;
  };
  slots: Slot[];
}

// Resource availability (slots grouped by resource)
export interface ResourceAvailability {
  resource: ResourceSummary;
  days: DayAvailability[];
  matrixSnapshot: MatrixSnapshot; // Passed through to appointment.create for travel chain update
}

// Full slots response
export interface AvailableSlotsResponse {
  business: {
    id: string;
    name: string;
    serviceAreaLabels: string[]; // e.g. ["Tel Aviv-Yafo", "Herzliya"] — for UI display
  };
  service: {
    id: string;
    name: string;
    durationMinutes: number;
    visitFee: number;
    visitFeePolicy: 'ALWAYS_CHARGED' | 'WAIVED_IF_ACCEPTED';
    jobFeeMin: number;
    jobFeeMax: number;
  };
  resources: ResourceAvailability[];
}

// ============================================================================
// Internal Service Types
// ============================================================================

// Travel time matrix
export interface TravelTimeMatrix {
  getDuration(from: Coordinate, to: Coordinate): number; // minutes
  getDistance(from: Coordinate, to: Coordinate): number; // meters
}

// Effective settings for a resource (resolved from resource overrides + business defaults)
export interface EffectiveResourceSettings {
  homeBase: Coordinate;  // Always from resource — required at creation, no business fallback
  travelMode: TravelMode;
  parkingBuffer: number;     // minutes
  bufferMinutes: number;     // minutes between appointments
}

// Absence types
export interface ResourceAbsence {
  id: string;
  resourceId: string;
  type: 'RECURRING' | 'ONE_TIME';
  dayOfWeek: number | null;  // For recurring
  date: Date | null;         // For one-time
  startTime: Date;           // Time portion
  endTime: Date;             // Time portion
  reason: string | null;
  // Optional geocoded location — when present, used as routing anchor instead of home base
  absenceAddress: string | null;
  absenceLat: number | null;
  absenceLng: number | null;
}

// Anchor for slot generation algorithm
export interface Anchor {
  type: 'day_start' | 'appointment' | 'absence' | 'day_end';
  startTime: Date;
  endTime: Date;
  location: Coordinate;
  // Origin of this anchor's resolved location.
  // 'inherit' is a transient value used only during buildAnchorsForShift construction —
  // after the propagation pass all anchors have a fully resolved locationType.
  locationType: 'home_base' | 'appointment' | 'located_absence' | 'inherit';
}
```

## 4. API Design

### 4.1 tRPC Router Structure

```
src/server/api/
├── root.ts                 # Root router combining all routers
├── trpc.ts                 # tRPC initialization and context
└── routers/
    ├── auth.ts             # Authentication
    ├── business.ts         # Business profile management
    ├── category.ts         # Category listing (managed taxonomy)   ← NEW
    ├── explore.ts          # Explore page queries                  ← NEW
    ├── resource.ts         # Resource (team member) management
    ├── service.ts          # Service CRUD
    ├── availability.ts     # Working hours management
    ├── absence.ts          # Resource absence management
    ├── schedule.ts         # Schedule view queries
    ├── appointment.ts      # Appointment operations
    └── chat.ts             # AI chat
```

### 4.2 Auth Router

```typescript
// src/server/api/routers/auth.ts

import { z } from 'zod';
import { createTRPCRouter, publicProcedure, protectedProcedure } from '../trpc';

export const authRouter = createTRPCRouter({
  
  // Register new business owner account
  register: publicProcedure
    .input(z.object({
      email: z.string().email(),
      password: z.string().min(8),
    }))
    .mutation(async ({ ctx, input }) => {
      // Returns: { userId, sessionToken }
    }),

  // Login
  login: publicProcedure
    .input(z.object({
      email: z.string().email(),
      password: z.string(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Returns: { userId, sessionToken, business? }
    }),

  // Logout
  logout: protectedProcedure
    .mutation(async ({ ctx }) => {
      // Invalidates current session
      // Returns: { success: true }
    }),

  // Get current session
  getSession: publicProcedure
    .query(async ({ ctx }) => {
      // Returns: { user, business } | null
    }),
});
```

### 4.3 Business Router

```typescript
// src/server/api/routers/business.ts

import { z } from 'zod';
import { createTRPCRouter, protectedProcedure, publicProcedure } from '../trpc';

// Service area regions are managed separately via the serviceArea router (add/remove).
// business.create no longer accepts serviceArea inline — regions are linked post-creation
// via serviceArea.add mutations during onboarding Step 5.

export const businessRouter = createTRPCRouter({

  // Create business profile (onboarding)
  create: protectedProcedure
    .input(z.object({
      name: z.string().min(2).max(100),
      category: z.string(),
      tagline: z.string().max(200).optional(),
      timezone: z.string(),
      fallbackContactMethod: z.enum(['PHONE', 'WHATSAPP', 'EMAIL']),
      fallbackContactValue: z.string(),
      currency: z.string().default('ILS'),
      serviceArea: serviceAreaSchema,
    }))
    .mutation(async ({ ctx, input }) => {
      // Creates business with auto-generated slug
      // Returns: { business }
    }),

  // Update business profile
  update: protectedProcedure
    .input(z.object({
      name: z.string().min(2).max(100).optional(),
      tagline: z.string().max(200).optional(),
      logoUrl: z.string().url().optional(),
      fallbackContactMethod: z.enum(['PHONE', 'WHATSAPP', 'EMAIL']).optional(),
      fallbackContactValue: z.string().optional(),
      travelMode: z.enum(['DRIVING_CAR', 'CYCLING_ELECTRIC', 'CYCLING_REGULAR']).optional(),
      parkingBufferMinutes: z.number().min(0).max(30).optional(),
      bufferMinutes: z.number().min(0).max(60).optional(),
      slotIntervalMinutes: z.number().refine(v => [15, 30, 60].includes(v)).optional(),
      bookingHorizonDays: z.number().min(1).max(90).optional(),
      confirmationMode: z.enum(['AUTO', 'MANUAL']).optional(),
      approvalTimeoutHours: z.number().min(1).max(24).optional(),
      cancellationWindowHours: z.number().min(0).max(72).optional(),
      allowReschedule: z.boolean().optional(),
      remindersEnabled: z.boolean().optional(),
      defaultLanguage: z.enum(['he', 'en']).optional(),
      aiEnabled: z.boolean().optional(),
      aiGreeting: z.string().max(500).optional(),
      notifyIcNewBooking: z.boolean().optional(),
      notifyIcDailyAgenda: z.boolean().optional(),
      windowSizeMinutes: z.number().refine(v => [60,90,120,180,240].includes(v)).optional(),
      scheduleSettlingHours: z.number().min(1).max(72).optional(),
      slotIntervalMinutes: z.number().refine(v => [15,30,60].includes(v)).optional(),
      bookingHorizonDays: z.number().min(1).max(90).optional(),
      serviceArea: serviceAreaSchema.optional(),
      faqText: z.string().max(5000).optional(),
      specialInstructions: z.string().max(2000).optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Updates authenticated user's business
      // Returns: { business }
    }),

  // Get business by ID (authenticated - for dashboard)
  getById: protectedProcedure
    .query(async ({ ctx }) => {
      // Returns authenticated user's business with all details
    }),

  // Get business by slug (public - for booking page)
  getBySlug: publicProcedure
    .input(z.object({ slug: z.string() }))
    .query(async ({ ctx, input }) => {
      // Returns public business info for customer booking page
    }),

  // Update slug
  updateSlug: protectedProcedure
    .input(z.object({
      slug: z.string().min(3).max(50).regex(/^[a-z0-9-]+$/),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates uniqueness and updates slug
    }),
});
```

### 4.4 Category Router

```typescript
// src/server/api/routers/category.ts

import { z } from 'zod';
import { createTRPCRouter, publicProcedure } from '../trpc';

export const categoryRouter = createTRPCRouter({
  
  // List all active categories (for onboarding dropdown)
  list: publicProcedure
    .input(z.object({
      locale: z.enum(['en', 'he']).default('en')
    }).optional())
    .query(async ({ ctx, input }) => {
      const locale = input?.locale || 'en';
      
      const categories = await ctx.prisma.category.findMany({
        where: { isActive: true },
        orderBy: { sortOrder: 'asc' }
      });
      
      return categories.map(cat => ({
        id: cat.id,
        slug: cat.slug,
        icon: cat.icon,
        name: (cat.names as Record<string, string>)[locale] 
              || (cat.names as Record<string, string>)['en'],
        sortOrder: cat.sortOrder
      }));
    }),
  
  // Get category by slug (for Explore page)
  getBySlug: publicProcedure
    .input(z.object({
      slug: z.string(),
      locale: z.enum(['en', 'he']).default('en')
    }))
    .query(async ({ ctx, input }) => {
      const category = await ctx.prisma.category.findUnique({
        where: { slug: input.slug }
      });
      
      if (!category) return null;
      
      return {
        id: category.id,
        slug: category.slug,
        icon: category.icon,
        name: (category.names as Record<string, string>)[input.locale] 
              || (category.names as Record<string, string>)['en']
      };
    })
});
```

### 4.5 Explore Router

```typescript
// src/server/api/routers/explore.ts

import { z } from 'zod';
import { TRPCError } from '@trpc/server';
import { createTRPCRouter, publicProcedure } from '../trpc';

export const exploreRouter = createTRPCRouter({
  
  // Get businesses by category
  byCategory: publicProcedure
    .input(z.object({
      categorySlug: z.string(),
      location: z.object({
        lat: z.number(),
        lng: z.number()
      }).optional(),
      limit: z.number().min(1).max(50).default(20),
      cursor: z.string().optional()
    }))
    .query(async ({ ctx, input }) => {
      const category = await ctx.prisma.category.findUnique({
        where: { slug: input.categorySlug }
      });
      
      if (!category) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Category not found'
        });
      }
      
      const businesses = await ctx.prisma.business.findMany({
        where: {
          categoryId: category.id,
          // TODO: Add service area filtering when location provided
        },
        include: {
          services: {
            where: { isActive: true },
            take: 3,
            orderBy: { sortOrder: 'asc' }
          },
          category: true
        },
        take: input.limit + 1,
        cursor: input.cursor ? { id: input.cursor } : undefined,
        orderBy: { createdAt: 'desc' }
      });
      
      let nextCursor: string | undefined;
      if (businesses.length > input.limit) {
        const nextItem = businesses.pop();
        nextCursor = nextItem?.id;
      }
      
      return {
        businesses,
        nextCursor
      };
    }),
  
  // Get category counts (for Explore page chips)
  categoryCounts: publicProcedure
    .input(z.object({
      location: z.object({
        lat: z.number(),
        lng: z.number()
      }).optional()
    }).optional())
    .query(async ({ ctx }) => {
      const counts = await ctx.prisma.category.findMany({
        where: { isActive: true },
        include: {
          _count: {
            select: { businesses: true }
          }
        },
        orderBy: { sortOrder: 'asc' }
      });
      
      return counts.map(cat => ({
        id: cat.id,
        slug: cat.slug,
        icon: cat.icon,
        names: cat.names,
        count: cat._count.businesses
      }));
    })
});
```

### 4.6 Resource Router

```typescript
// src/server/api/routers/resource.ts

import { z } from 'zod';
import { createTRPCRouter, protectedProcedure, publicProcedure } from '../trpc';

export const resourceRouter = createTRPCRouter({

  // Create a new resource for the business
  create: protectedProcedure
    .input(z.object({
      name: z.string().min(2).max(100),
      email: z.string().email().optional(),
      phone: z.string().optional(),
      role: z.string().max(100).optional(),
      homeBase: z.object({ 
        lat: z.number(), 
        lng: z.number() 
      }).optional(),
      travelMode: z.enum(['DRIVING_CAR', 'CYCLING_ELECTRIC', 'CYCLING_REGULAR']).optional(),
      parkingBuffer: z.number().min(0).max(30).optional(),
      bufferMinutes: z.number().min(0).max(60).optional(),
      // POST-MVP: serviceArea override
      // serviceArea: serviceAreaSchema.optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates max 5 resources per business
      // Creates resource with optional settings overrides
      // MVP: Resources inherit business service area
    }),

  // Update resource
  update: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      name: z.string().min(2).max(100).optional(),
      email: z.string().email().optional(),
      phone: z.string().optional(),
      role: z.string().max(100).optional(),
      homeBase: z.object({ 
        lat: z.number(), 
        lng: z.number() 
      }).nullable().optional(),
      travelMode: z.enum(['DRIVING_CAR', 'CYCLING_ELECTRIC', 'CYCLING_REGULAR']).nullable().optional(),
      parkingBuffer: z.number().min(0).max(30).nullable().optional(),
      bufferMinutes: z.number().min(0).max(60).nullable().optional(),
      isActive: z.boolean().optional(),
      sortOrder: z.number().optional(),
      // POST-MVP: serviceArea override
      // serviceArea: serviceAreaSchema.nullable().optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates resource belongs to authenticated user's business
    }),

  // Deactivate resource (soft disable)
  deactivate: protectedProcedure
    .input(z.object({ resourceId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Checks for future appointments before deactivation
      // Sets isActive = false
    }),

  // Reactivate resource
  reactivate: protectedProcedure
    .input(z.object({ resourceId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Sets isActive = true
    }),

  // Delete resource (hard delete)
  delete: protectedProcedure
    .input(z.object({ resourceId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Only if no appointments exist
    }),

  // List resources for business (authenticated)
  list: protectedProcedure.query(async ({ ctx }) => {
    // Returns all resources with working hours summary
  }),

  // List active resources for business (public - booking page)
  listPublic: publicProcedure
    .input(z.object({ businessId: z.string() }))
    .query(async ({ ctx, input }) => {
      // Returns only active resources (id, name, role, avatarUrl)
    }),

  // Get resource by ID
  getById: protectedProcedure
    .input(z.object({ resourceId: z.string() }))
    .query(async ({ ctx, input }) => {}),

  // Invite resource to create user account (sends email)
  invite: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      email: z.string().email(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Generates invite token, sends email with signup link
    }),
});
```

### 4.7 Service Router

```typescript
// src/server/api/routers/service.ts

import { z } from 'zod';
import { createTRPCRouter, protectedProcedure, publicProcedure } from '../trpc';

export const serviceRouter = createTRPCRouter({

  // Create service
  create: protectedProcedure
    .input(z.object({
      name: z.string().min(2).max(100),
      description: z.string().max(500),
      durationMinutes: z.number().min(15).max(480),
      visitFee: z.number().min(0),
      visitFeePolicy: z.enum(['ALWAYS_CHARGED', 'WAIVED_IF_ACCEPTED']),
      jobFeeMin: z.number().min(0),
      jobFeeMax: z.number().min(0),
      requiredInputs: z.array(z.string()).default([]),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates jobFeeMax >= jobFeeMin
    }),

  // Update service
  update: protectedProcedure
    .input(z.object({
      serviceId: z.string(),
      name: z.string().min(2).max(100).optional(),
      description: z.string().max(500).optional(),
      durationMinutes: z.number().min(15).max(480).optional(),
      visitFee: z.number().min(0).optional(),
      visitFeePolicy: z.enum(['ALWAYS_CHARGED', 'WAIVED_IF_ACCEPTED']).optional(),
      jobFeeMin: z.number().min(0).optional(),
      jobFeeMax: z.number().min(0).optional(),
      requiredInputs: z.array(z.string()).optional(),
      isActive: z.boolean().optional(),
      sortOrder: z.number().optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates service belongs to authenticated user's business
    }),

  // Delete service (hard delete)
  delete: protectedProcedure
    .input(z.object({ serviceId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Checks for active appointments before deletion
    }),

  // List services for business (authenticated)
  list: protectedProcedure.query(async ({ ctx }) => {
    // Returns all services for authenticated user's business
  }),

  // List active services for business (public - booking page)
  listPublic: publicProcedure
    .input(z.object({ businessId: z.string() }))
    .query(async ({ ctx, input }) => {
      // Returns only active services
    }),

  // Get service by ID
  getById: publicProcedure
    .input(z.object({ serviceId: z.string() }))
    .query(async ({ ctx, input }) => {}),
});
```

### 4.8 Availability Router (Working Hours)

```typescript
// src/server/api/routers/availability.ts

import { z } from 'zod';
import { createTRPCRouter, protectedProcedure } from '../trpc';

// Schema for a single shift
const shiftSchema = z.object({
  startTime: z.date(),  // Time portion only
  endTime: z.date(),    // Time portion only
});

export const availabilityRouter = createTRPCRouter({

  // ============================================================================
  // SHIFT MANAGEMENT (supports multiple shifts per day)
  // ============================================================================

  // Add a shift to a specific day
  addShift: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      dayOfWeek: z.number().min(0).max(6),
      startTime: z.date(),
      endTime: z.date(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates resource belongs to business
      // Validates endTime > startTime
      // Validates no overlap with existing shifts on same day
      // Creates new WorkingHours record
    }),

  // Update an existing shift
  updateShift: protectedProcedure
    .input(z.object({
      shiftId: z.string(),
      startTime: z.date().optional(),
      endTime: z.date().optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates shift belongs to resource owned by business
      // Validates no overlap if times changed
      // Updates WorkingHours record
    }),

  // Delete a shift
  deleteShift: protectedProcedure
    .input(z.object({ shiftId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Validates shift belongs to resource owned by business
      // Deletes WorkingHours record
    }),

  // ============================================================================
  // BULK OPERATIONS
  // ============================================================================

  // Set all shifts for a specific day (replaces existing)
  setDayShifts: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      dayOfWeek: z.number().min(0).max(6),
      shifts: z.array(shiftSchema),  // Empty array = day off
    }))
    .mutation(async ({ ctx, input }) => {
      // Deletes all existing shifts for this day
      // Creates new shifts (if any)
      // Validates no overlaps within the new shifts
    }),

  // Set entire week schedule (replaces all)
  setWeekSchedule: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      schedule: z.array(z.object({
        dayOfWeek: z.number().min(0).max(6),
        shifts: z.array(shiftSchema),
      })),
    }))
    .mutation(async ({ ctx, input }) => {
      // Deletes all existing WorkingHours for resource
      // Creates new shifts for each day
      // Used for initial setup or full reset
    }),

  // ============================================================================
  // QUERIES
  // ============================================================================

  // Get working hours for a resource (grouped by day)
  getSchedule: protectedProcedure
    .input(z.object({ resourceId: z.string() }))
    .query(async ({ ctx, input }) => {
      // Returns WeekSchedule (7 days, each with array of shifts)
      // Days with no shifts are marked isWorking: false
    }),

  // ============================================================================
  // TEMPLATES & COPYING
  // ============================================================================

  // Copy working hours from one resource to another
  copySchedule: protectedProcedure
    .input(z.object({
      fromResourceId: z.string(),
      toResourceId: z.string(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Deletes target's existing schedule
      // Copies all shifts from source to target
    }),

  // Apply a preset template
  applyTemplate: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      template: z.enum([
        'STANDARD_FULL',     // Sun-Thu 08:00-17:00 (Israel standard)
        'STANDARD_SPLIT',    // Sun-Thu 08:00-12:00, 14:00-18:00
        'PART_TIME_MORNING', // Sun-Thu 08:00-13:00
        'PART_TIME_AFTERNOON', // Sun-Thu 13:00-18:00
        'WEEKEND_ONLY',      // Fri-Sat 09:00-17:00
      ]),
    }))
    .mutation(async ({ ctx, input }) => {
      // Deletes existing schedule
      // Creates shifts based on template
    }),
});
```

### 4.9 Absence Router

```typescript
// src/server/api/routers/absence.ts

import { z } from 'zod';
import { createTRPCRouter, protectedProcedure } from '../trpc';

export const absenceRouter = createTRPCRouter({

  // Create absence for a resource
  create: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      type: z.enum(['RECURRING', 'ONE_TIME']),
      dayOfWeek: z.number().min(0).max(6).optional(), // Required for RECURRING
      date: z.date().optional(),      // Required for ONE_TIME
      startTime: z.date(),            // Time portion only
      endTime: z.date(),              // Time portion only
      reason: z.string().max(200).optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates resource belongs to business
      // Validates required fields based on type
      // For ONE_TIME full day: startTime=00:00, endTime=23:59
    }),

  // Update absence
  update: protectedProcedure
    .input(z.object({
      absenceId: z.string(),
      startTime: z.date().optional(),
      endTime: z.date().optional(),
      reason: z.string().max(200).optional(),
    }))
    .mutation(async ({ ctx, input }) => {}),

  // Delete absence
  delete: protectedProcedure
    .input(z.object({ absenceId: z.string() }))
    .mutation(async ({ ctx, input }) => {}),

  // List absences for a resource
  list: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      dateRange: z.object({
        startDate: z.date(),
        endDate: z.date(),
      }).optional(),
      type: z.enum(['RECURRING', 'ONE_TIME']).optional(),
    }))
    .query(async ({ ctx, input }) => {
      // Returns recurring + one-time absences within range
    }),
});
```

### 4.10 Schedule Router

```typescript
// src/server/api/routers/schedule.ts

import { z } from 'zod';
import { createTRPCRouter, protectedProcedure } from '../trpc';

export const scheduleRouter = createTRPCRouter({

  // ============================================================================
  // CORE SCHEDULE QUERY
  // Used by both Week Overview and Day Detail views in the Schedule View section.
  // Returns all data needed to render the schedule in a single efficient query.
  // ============================================================================

  getForDateRange: protectedProcedure
    .input(z.object({
      startDate: z.string(), // ISO date string, e.g. "2025-03-01"
      endDate: z.string(),   // ISO date string, e.g. "2025-03-07" (for week) or same as startDate (for day)
      resourceIds: z.array(z.string()).optional(), // Filter to specific resources; if omitted, returns all active
    }))
    .query(async ({ ctx, input }) => {
      /**
       * 1. Load all active resources for business (filtered by resourceIds if provided)
       * 2. For each resource:
       *    a. Load WorkingHours for each relevant dayOfWeek in the date range
       *    b. Load BookedAppointments where scheduledStart falls within the date range
       *    c. Load ResourceAbsences:
       *       - ONE_TIME: where date falls within the date range
       *       - RECURRING: where dayOfWeek matches any day in the date range
       * 3. Return structured data grouped by resourceId and date
       *
       * Returns: ScheduleViewResponse
       */
    }),

  // ============================================================================
  // CONVENIENCE QUERIES
  // ============================================================================

  // Get schedule for a single resource on a single date (used by Day Detail)
  getDayForResource: protectedProcedure
    .input(z.object({
      resourceId: z.string(),
      date: z.string(), // ISO date string
    }))
    .query(async ({ ctx, input }) => {
      // Convenience wrapper around getForDateRange for a single resource × day
      // Returns: ResourceDaySchedule
    }),
});
```

#### Schedule View Response Types

```typescript
// src/types/schedule.ts

// A single appointment as displayed in the schedule view
export interface ScheduleAppointment {
  id: string;
  scheduledStart: Date;
  scheduledEnd: Date;
  status: 'PENDING' | 'CONFIRMED' | 'CANCELLED' | 'COMPLETED' | 'EXPIRED' | 'NO_SHOW';
  customerName: string;
  customerAddress: string;
  serviceName: string;
  durationMinutes: number;
}

// A single absence/break block as displayed in the schedule view
export interface ScheduleAbsence {
  id: string;
  type: 'RECURRING' | 'ONE_TIME';
  startTime: Date;   // Full datetime for the specific date being rendered
  endTime: Date;     // Full datetime for the specific date being rendered
  reason: string | null;
  // For recurring: dayOfWeek is preserved so UI can confirm edits affect all future occurrences
  dayOfWeek: number | null;
}

// Schedule data for one resource on one specific date
export interface ResourceDaySchedule {
  date: string;            // ISO date
  dayOfWeek: number;       // 0–6
  isWorkingDay: boolean;   // False if no WorkingHours for this dayOfWeek
  workingShifts: Array<{
    startTime: Date;
    endTime: Date;
  }>;
  appointments: ScheduleAppointment[];
  absences: ScheduleAbsence[];
}

// Full response structure for the schedule view
export interface ScheduleViewResponse {
  resources: Array<{
    id: string;
    name: string;
    role: string | null;
    avatarUrl: string | null;
    isActive: boolean;
    days: ResourceDaySchedule[];
  }>;
  dateRange: {
    startDate: string;
    endDate: string;
  };
}
```

---

### 4.11 Appointment Router

```typescript
// src/server/api/routers/appointment.ts

import { z } from 'zod';
import { createTRPCRouter, protectedProcedure, publicProcedure } from '../trpc';

export const appointmentRouter = createTRPCRouter({

  // ============================================================================
  // GET AVAILABLE SLOTS (Core Algorithm)
  // ============================================================================
  
  getAvailableSlots: publicProcedure
    .input(z.object({
      businessId: z.string(),
      serviceId: z.string(),
      resourceId: z.string().optional(), // Filter to specific resource
      customerLocation: z.object({
        address: z.string(),
        lat: z.number(),
        lng: z.number(),
        // Note: addressComponents are NOT passed here.
        // Service area validation is a separate prior step (geo.validateServiceArea).
        // By the time this endpoint is called, the customer location is confirmed in-area.
      }),
      dateRange: z.object({
        startDate: z.string(), // ISO date
        endDate: z.string(),
      }),
    }))
    .query(async ({ ctx, input }) => {
      /**
       * PRE-CONDITION: Customer location has already been validated against business
       * service area via geo.validateServiceArea — this endpoint does NOT re-validate.
       *
       * If resourceId provided: generate slots for that resource only
       * If resourceId not provided: generate slots for all active resources
       * Returns slots grouped by resource
       *
       * Full implementation in SlotGeneratorService
       * See Section 5.2 for detailed algorithm
       */
    }),

  // ============================================================================
  // CREATE APPOINTMENT
  // ============================================================================
  
  create: publicProcedure
    .input(z.object({
      businessId: z.string(),
      serviceId: z.string(),
      resourceId: z.string(), // Required: which resource performs the service
      slot: z.object({
        date: z.string(),
        startTime: z.string(),
        endTime: z.string(),
      }),
      customer: z.object({
        name: z.string().min(2).max(100),
        phone: z.string().min(9).max(20),
        email: z.string().email().optional(),
        location: z.object({
          address: z.string(),
          coordinates: z.object({
            lat: z.number(),
            lng: z.number(),
          }),
        }),
      }),
      notes: z.string().max(500).optional(),
      // Matrix snapshot from getAvailableSlots — used to update the full travel
      // chain for this resource on this date without a second ORS call.
      // Optional: if absent or expired (> 10 min), chain update is skipped.
      matrixSnapshot: z.object({
        resourceId: z.string(),
        date: z.string(),
        timestamp: z.number(),
        locations: z.array(z.object({
          lat: z.number(),
          lng: z.number(),
          type: z.enum(['home_base', 'appointment', 'located_absence']),
          refId: z.string().optional(),
        })),
        durations: z.array(z.array(z.number())),
      }).optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      /**
       * 1. Validate business, service, resource exist and are active
       * 2. Validate customer location in service area
       * 3. Start transaction
       * 4. Lock and verify slot still available for THIS RESOURCE
       * 5. Create appointment with appropriate status
       * 6. Commit transaction
       * 7. Send notifications (outside transaction)
       * 8. Schedule expiry job if manual approval
       * 9. Travel chain update (outside transaction, after notifications):
       *
       *    Fast path — matrix is fresh (≤10 min old and correct day):
       *      → applyMatrixSnapshotToChain: rebuild full chain from snapshot, 0 ORS calls
       *
       *    Fallback path — matrix expired, absent, or wrong day:
       *      → TravelChainService.rebuild: 1 fresh ORS matrix call for (resourceId, date)
       *      → Same result as cancellation path — chain always accurate
       *
       *    There is no silent skip. One of the two paths always runs.
       *    The snapshot is an optimisation; TravelChainService.rebuild is the canonical fallback.
       *
       *    Note: slot staleness — see Section 4.11.1 for full conflict prevention spec.
       */
    }),

  // ============================================================================
  // APPOINTMENT MANAGEMENT (Business Owner/Resource)
  // ============================================================================

  approve: protectedProcedure
    .input(z.object({ appointmentId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Updates status to CONFIRMED, notifies customer
    }),

  decline: protectedProcedure
    .input(z.object({
      appointmentId: z.string(),
      reason: z.string().max(500).optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Updates status to CANCELLED, notifies customer
    }),

  cancelByBusiness: protectedProcedure
    .input(z.object({
      appointmentId: z.string(),
      reason: z.string().max(500).optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Updates status to CANCELLED, notifies customer
    }),

  // List appointments for business (all resources or filtered)
  listForBusiness: protectedProcedure
    .input(z.object({
      resourceId: z.string().optional(), // Filter by specific resource
      dateRange: z.object({
        startDate: z.string(),
        endDate: z.string(),
      }).optional(),
      status: z.array(z.enum([
        'PENDING_APPROVAL', 'PENDING_OPTIMIZATION', 'SCHEDULED', 'CANCELLED', 'COMPLETED'
      ])).optional(),
      limit: z.number().min(1).max(100).default(50),
      cursor: z.string().optional(),
    }))
    .query(async ({ ctx, input }) => {
      // Returns paginated appointments with resource info
    }),

  getById: protectedProcedure
    .input(z.object({ appointmentId: z.string() }))
    .query(async ({ ctx, input }) => {}),

  markCompleted: protectedProcedure
    .input(z.object({ appointmentId: z.string() }))
    .mutation(async ({ ctx, input }) => {}),

  markNoShow: protectedProcedure
    .input(z.object({ appointmentId: z.string() }))
    .mutation(async ({ ctx, input }) => {}),

  // ============================================================================
  // APPOINTMENT MANAGEMENT (Customer)
  // ============================================================================

  cancelByCustomer: publicProcedure
    .input(z.object({
      appointmentId: z.string(),
      customerPhone: z.string(), // For verification
    }))
    .mutation(async ({ ctx, input }) => {
      // Validates phone, checks cancellation window
    }),

  getStatus: publicProcedure
    .input(z.object({
      appointmentId: z.string(),
      customerPhone: z.string(),
    }))
    .query(async ({ ctx, input }) => {
      // Returns appointment details if phone matches
    }),
});
```

---

#### 4.11.1 Slot Conflict Prevention

**The problem:**

Slots are derived time windows — not rows in a table. Two customers can both receive the same slot from `getAvailableSlots` and submit `appointment.create` concurrently. Without a conflict check, both pass and a double-booking is created.

**Solution: two layers inside the same transaction.**

**Layer 1 — Advisory lock:** Serialises all booking attempts for the same resource on the same date. The conflict check and insert become atomic — no concurrent request can sneak between them.

**Layer 2 — Overlap check:** Queries for any existing `SCHEDULED` or `PENDING_APPROVAL` appointment that overlaps the requested window. If found → slot is taken → throw error.

Both layers run inside the transaction started in step 3. The advisory lock is transaction-scoped — Postgres releases it automatically on commit or rollback. No cleanup code needed.

---

**Layer 1 — Advisory lock:**

`pg_advisory_xact_lock(key1 int4, key2 int4)` acquires an exclusive lock identified by two integers. Any other transaction that calls the same function with the same key pair blocks until the first transaction finishes.

```typescript
// Step 4a — acquire lock before conflict check
// Two-argument form: hashtext() converts any string to a stable int4.
// Two separate hashes combined = effectively 64-bit key space.
// Collision probability negligible in practice.
//
// Same resourceId + same date → same key pair → second request blocks.
// Different resource or different date → different key pair → no blocking.

await tx.$executeRaw`
  SELECT pg_advisory_xact_lock(
    hashtext(${resourceId}),
    hashtext(${appointmentDate})  -- ISO date string e.g. "2026-10-15"
  )
`
```

**Why two arguments, not one `hashtext(resourceId || date)`:**

`hashtext` returns `int4` (32-bit). The single-argument `pg_advisory_xact_lock` takes a `bigint` (64-bit). Postgres auto-casts, but concatenating two 32-bit hashes into one 64-bit integer slightly increases collision risk across all resource+date combinations on the platform. The two-argument form passes both `int4` values directly and Postgres combines them internally into a proper 64-bit key — safer, no cast involved.

---

**Layer 2 — Overlap check:**

After acquiring the lock, check for existing appointments that overlap the requested window:

```typescript
// Step 4b — conflict check (runs after advisory lock acquired)
//
// Standard interval overlap condition:
// Intervals [A_start, A_end) and [B_start, B_end) overlap when:
//   A_start < B_end  AND  A_end > B_start

const conflict = await tx.bookedAppointment.findFirst({
  where: {
    resourceId,
    status: { in: ['SCHEDULED', 'PENDING_APPROVAL'] },
    scheduledStart: { lt: requestedEnd },   // existing starts before requested ends
    scheduledEnd:   { gt: requestedStart }, // existing ends after requested starts
  },
  select: { id: true },
})

if (conflict) {
  throw new TRPCError({
    code: 'CONFLICT',
    message: 'SLOT_TAKEN',
  })
}
// If no conflict → proceed to INSERT in step 5
```

`PENDING_APPROVAL` is included because that appointment already holds the time window — it is awaiting IC confirmation but the slot is reserved.

---

**Full flow for two concurrent requests:**

```
Request A (transaction starts)
  ├─ pg_advisory_xact_lock(hash(resourceId), hash(date))  ← A acquires lock
  ├─ Overlap check → no conflict found
  ├─ INSERT booked_appointment
  └─ COMMIT → lock released

Request B (arrives while A holds lock)
  ├─ pg_advisory_xact_lock(hash(resourceId), hash(date))  ← B blocks here
  │   ... waits for A to commit ...
  ├─ Overlap check → finds A's new appointment → SLOT_TAKEN thrown
  └─ ROLLBACK → lock released
```

---

**Client error handling — `SLOT_TAKEN`:**

When `appointment.create` returns a `CONFLICT / SLOT_TAKEN` error:

1. Show message: _"That time slot was just taken by another customer."_
2. Automatically re-fetch `getAvailableSlots` for the same service and date
3. Return the customer to the slot selection step with fresh availability — no full restart of the booking flow

---

**Performance note:**

Advisory lock acquisition + overlap check add ~1–2ms to the transaction. Imperceptible to customers. At MVP scale (tens of concurrent bookings at most) lock contention is vanishingly rare — the advisory lock cost is paid on every create, the wait cost is paid only when two users literally collide on the same resource and date within the same millisecond.

**Index coverage:** The overlap query uses the existing `@@index([resourceId, scheduledStart])` on `BookedAppointment`. No additional index needed.

---

### 4.12 Chat Router

```typescript
// src/server/api/routers/chat.ts

import { z } from 'zod';
import { createTRPCRouter, publicProcedure } from '../trpc';

export const chatRouter = createTRPCRouter({

  sendMessage: publicProcedure
    .input(z.object({
      businessId: z.string(),
      message: z.string().max(1000),
    }))
    .mutation(async ({ ctx, input }) => {
      /**
       * 1. Load business config for grounding
       * 2. Build system prompt with services, pricing, FAQ
       * 3. Call Claude API
       * 4. Return response (no persistence)
       * 
       * Returns: { response: string }
       */
    }),
});
```

### 4.13 Response Types

```typescript
// src/server/api/types.ts

// Get Available Slots Response - Success
export interface GetAvailableSlotsResponse {
  success: true;
  data: {
    business: {
      id: string;
      name: string;
      currency: string;
      timezone: string;
    };
    service: {
      id: string;
      name: string;
      durationMinutes: number;
      visitFee: number;
      visitFeePolicy: 'ALWAYS_CHARGED' | 'WAIVED_IF_ACCEPTED';
      jobFeeMin: number;
      jobFeeMax: number;
    };
    customerLocation: {
      address: string;
      coordinates: { lat: number; lng: number };
    };
    requestedRange: {
      startDate: string;
      endDate: string;
    };
    effectiveRange: {
      startDate: string;
      endDate: string;
      clampedByHorizon: boolean;
    };
    days: DayAvailability[];
  };
}

// Get Available Slots Response - Error
export interface GetAvailableSlotsError {
  success: false;
  error: {
    code: 'OUTSIDE_SERVICE_AREA' | 'ROUTING_UNAVAILABLE' | 'BUSINESS_NOT_FOUND' | 'SERVICE_NOT_FOUND' | 'SERVICE_INACTIVE';
    message: string;
    data?: {
      serviceAreaName?: string;
      fallbackContact?: {
        method: 'PHONE' | 'WHATSAPP' | 'EMAIL';
        value: string;
      };
      retryAfterSeconds?: number;
    };
  };
}

// Create Booking Response - Success
export interface CreateBookingResponse {
  success: true;
  data: {
    bookingId: string;
    status: 'CONFIRMED' | 'PENDING';
    booking: {
      id: string;
      businessId: string;
      businessName: string;
      serviceId: string;
      serviceName: string;
      scheduledDate: string;
      scheduledStart: string;
      scheduledEnd: string;
      customerName: string;
      customerAddress: string;
      visitFee: number;
      jobFeeEstimate: string;
      currency: string;
      status: 'CONFIRMED' | 'PENDING';
      approvalDeadline?: string;
    };
    message: string;
  };
}

// Create Booking Response - Error
export interface CreateBookingError {
  success: false;
  error: {
    code: 'SLOT_NO_LONGER_AVAILABLE' | 'VALIDATION_ERROR' | 'OUTSIDE_SERVICE_AREA';
    message: string;
    alternativeSlots?: Slot[];
    fields?: Record<string, string>;
  };
}
```

## 5. Service Layer Specifications

### 5.1 Overview

The service layer contains the core business logic, separated from the API layer for testability and reusability.

```
src/server/services/
├── routing/
│   ├── interface.ts        # Provider interface (departureTime-ready)
│   ├── openroute.ts        # OpenRouteService implementation
│   └── index.ts            # Provider export (swap here to change provider)
├── slots/
│   └── generator.ts        # Slot generation algorithm
├── geo/
│   └── service.ts          # GeoService: isInServiceArea, resolveOrCreateRegion
├── notifications/
│   └── service.ts          # Email notifications
└── ai/
    └── chat.ts             # AI chat service
```

---

### 5.2 Tier Gates

Feature gates enforce subscription tier limits server-side in tRPC procedures. Client-side hiding is for UX only — never a security boundary. Every gated feature must call the appropriate gate helper before executing.

```typescript
// src/server/lib/tierGates.ts

import { TRPCError } from '@trpc/server';
import { PlanTier } from '@prisma/client';

export const TIER_LIMITS = {
  SOLO: {
    maxResources: 1,
    maxServices: 5,
    aiChat: false,
    optimizedScheduling: false,
    resourceServiceAreaOverride: false,
    crossResourceOptimization: false,
  },
  PROFESSIONAL: {
    maxResources: 3,
    maxServices: Infinity,
    aiChat: true,
    optimizedScheduling: true,
    resourceServiceAreaOverride: true,
    crossResourceOptimization: false,
  },
  TEAM: {
    maxResources: 5,
    maxServices: Infinity,
    aiChat: true,
    optimizedScheduling: true,
    resourceServiceAreaOverride: true,
    crossResourceOptimization: true,
  },
} as const satisfies Record<PlanTier, object>;

type TierFeatureFlag = keyof typeof TIER_LIMITS.SOLO;
type TierCountLimit = 'maxResources' | 'maxServices';

/**
 * Assert a boolean feature is available on the given tier.
 * Throws FORBIDDEN if not available.
 */
export function assertTierFeature(
  tier: PlanTier,
  feature: TierFeatureFlag,
  label: string
): void {
  if (!TIER_LIMITS[tier][feature]) {
    throw new TRPCError({
      code: 'FORBIDDEN',
      message: `${label} is not available on your current plan. Please upgrade to access this feature.`,
    });
  }
}

/**
 * Assert a count limit has not been reached on the given tier.
 * Throws FORBIDDEN if at or over limit.
 */
export function assertTierLimit(
  tier: PlanTier,
  limit: TierCountLimit,
  currentCount: number,
  label: string
): void {
  if (currentCount >= TIER_LIMITS[tier][limit]) {
    throw new TRPCError({
      code: 'FORBIDDEN',
      message: `${label} limit reached for your current plan. Please upgrade to add more.`,
    });
  }
}

/**
 * Returns whether a feature is available on the given tier without throwing.
 * Use for conditional UI data (e.g. returning feature flags to client).
 */
export function hasTierFeature(
  tier: PlanTier,
  feature: TierFeatureFlag
): boolean {
  return Boolean(TIER_LIMITS[tier][feature]);
}
```

**Gate usage in tRPC procedures:**

```typescript
// resource.create — enforce max resources
assertTierLimit(ctx.business.planTier, 'maxResources', currentResourceCount, 'Resource');

// service.create — enforce max services
assertTierLimit(ctx.business.planTier, 'maxServices', currentServiceCount, 'Service');

// chat router — enforce AI chat access
assertTierFeature(ctx.business.planTier, 'aiChat', 'AI Chat');

// appointment.getAvailableSlots — enforce window booking tier
// (only called when service.schedulingMode === 'OPTIMIZE')
assertTierFeature(ctx.business.planTier, 'optimizedScheduling', 'Optimized Scheduling');
```

**Subscription status gate:**

LOCKED businesses (trial expired or payment failed) must be blocked from accepting new bookings. Add this check to the `publicProcedure` context or the `getAvailableSlots` procedure:

```typescript
if (business.subscriptionStatus === 'LOCKED') {
  throw new TRPCError({
    code: 'FORBIDDEN',
    message: 'This business is not currently accepting bookings.',
  });
}
```

---

### 5.3 Slot Generation Algorithm (Complete)

This is the core differentiating logic of JOURN.

#### 5.3.0 Conceptual Overview: How Availability is Determined

A resource's availability is the intersection of what they CAN work minus what's already committed:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Available Time = Working Hours (Shifts) − Absences − Existing Appointments │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Visual Example: Monday Schedule for Resource "David"**

```
06:00   08:00   10:00   12:00   14:00   16:00   18:00   20:00
  │       │       │       │       │       │       │       │
  │       ╔═══════════════╗       ╔═══════════════════════╗
  │       ║  Morning Shift ║       ║   Afternoon Shift    ║  ← WorkingHours
  │       ║  08:00-12:00   ║       ║   14:00-18:00        ║     (defines WHEN
  │       ╚═══════════════╝       ╚═══════════════════════╝      they CAN work)
  │               │                       │
  │               ▼                       ▼
  │       ┌───────────────┐       ┌───────────────────────┐
  │       │ ████░░░░░░░░░ │       │ ░░░░░░████░░░░░░░░░░░ │  ← After subtracting
  │       │ Appt  Avail   │       │ Avail Absence  Avail  │     Absences & Appts
  │       │ 8-9   9-12    │       │ 14-15  15-16  16-18   │
  │       └───────────────┘       └───────────────────────┘
  │
  │       ████ = Blocked (existing appointment or absence)
  │       ░░░░ = Available gaps (where new slots can be generated)
```

**The Three Data Sources:**

|Source|Defines|Example|
|---|---|---|
|**WorkingHours**|When they CAN work|Mon 08:00-12:00, Mon 14:00-18:00 (split shift)|
|**ResourceAbsence**|When they CAN'T work|Recurring lunch 12:00-13:00, One-time doctor 15:00-16:00|
|**BookedAppointment**|What's already booked|Appointment with customer @ 08:00-09:00|

**Algorithm Steps:**

1. **For each shift window** → Identify the working period
2. **Build anchors** → day_start, appointments, absences, day_end
3. **Find gaps** → Time between anchors where nothing is scheduled
4. **Generate slots** → Within each gap, considering:
    - Travel time FROM previous location
    - Travel time TO next location
    - Service duration
    - Buffer time between appointments
    - Parking buffer

The result is a set of bookable time slots that respect both the resource's schedule AND route efficiency.

#### 5.3.1 Algorithm Overview

```
INPUT:
- businessId, serviceId
- resourceId (optional - filter to specific resource)
- customerLocation (coordinates)
- dateRange (start, end)

OUTPUT:
- Array of ResourceAvailability objects, each containing:
  - resource: { id, name, role }
  - days: Array of DayAvailability objects

ALGORITHM:
1. Load and validate business, service
2. Validate customer location in service area
3. Determine resources to process:
   - If resourceId provided: single resource
   - Otherwise: all active resources
4. Calculate effective date range (clamp to booking horizon)
5. For each resource:
   a. Load working hours, absences, existing appointments
   b. Build unique location set for travel time matrix
   c. Fetch travel times (from cache or API)
   d. For each day in range:
      i. Check if working day for this resource
      ii. Build anchors (day_start, appointments, absences, day_end)
      iii. Find gaps between anchors
      iv. Generate slots within valid gaps
6. Return response grouped by resource
```

#### 5.3.2 Anchor-Based Gap Finding

**Anchor Definition:** An anchor is a fixed point in a resource's schedule that constrains slot placement.

```typescript
interface Anchor {
  type: 'day_start' | 'day_end' | 'appointment' | 'absence';
  startTime: Date;    // When this period starts
  endTime: Date;      // When this period ends (resource becomes free)
  location: Coordinate; // Where resource is at endTime — resolved via propagation (see buildAnchorsForShift)
  locationType: 'home_base' | 'appointment' | 'located_absence' | 'inherit';
  label?: string;     // For debugging
}
```

**Location propagation rule:** Every anchor's location is determined by the first explicit geocoded location at or before it in the sorted sequence. `day_start` is always home base. Unlocated absences and `day_end` inherit the previous anchor's resolved location. This means:

- A break after appointment A is assumed to be at A's location
- A break with an explicit address (doctor, restaurant) uses that address — and subsequent unlocated anchors propagate from it
- `day_end` resolves to wherever the IC last was, so the final gap is calculated correctly without forcing a return-to-home assumption

```

**Anchor Examples for a Day (Single Resource):**

```

08:00 ────────────────────────────────────────────────────────────── 18:00 │ │ ├─ day_start (resource at home, free from 08:00) │ │ │ │ 09:00-10:00 [Appointment A @ Ramat Gan] │ │ │ │ 12:00-13:00 [Absence: Lunch @ home] │ │ │ │ 14:00-15:30 [Appointment B @ Herzliya] │ │ │ └─ day_end (resource must be available until 18:00) ──────────────────┘

Anchors (sorted by startTime):

1. day_start @ 08:00, location: home
2. Appointment A @ 09:00-10:00, location: Ramat Gan
3. Absence @ 12:00-13:00, location: home
4. Appointment B @ 14:00-15:30, location: Herzliya
5. day_end @ 18:00, location: home

```

#### 5.3.3 Slot Generation Within a Gap

For each gap between consecutive anchors:

```

Gap: prevAnchor ──────────────────────────── nextAnchor

GIVEN:

- prevAnchor.endTime = when IC is free
- prevAnchor.location = where IC is
- nextAnchor.startTime = when IC must be at next commitment
- nextAnchor.location = where next commitment is
- customerLocation = where customer is
- serviceDuration = how long the service takes
- buffer = time between jobs (travel prep)
- parkingBuffer = parking time (driving only)

CALCULATE:

1. travelFromPrev = travel(prevAnchor.location → customerLocation)
    
2. travelToNext = travel(customerLocation → nextAnchor.location)
    
3. earliestArrival = prevAnchor.endTime + buffer + travelFromPrev + parkingBuffer
    
4. latestDeparture = nextAnchor.startTime - travelToNext - parkingBuffer - buffer
    
5. latestStart = latestDeparture - serviceDuration
    
6. IF earliestArrival > latestStart → No valid slots in this gap
    
7. earliestStart = roundUp(earliestArrival, slotInterval)
    
8. FOR t = earliestStart; t <= latestStart; t += slotInterval: ADD slot { start: t, end: t + serviceDuration }
    

```

#### 5.3.4 Worked Example

**Setup:**
- IC working hours: 08:00-18:00
- buffer: 15 min
- parkingBuffer: 5 min
- slotInterval: 30 min
- serviceDuration: 60 min

**Existing Schedule:**
- Booking A: 09:00-10:00 @ Ramat Gan
- Booking B: 14:00-15:30 @ Herzliya

**Customer Location:** Givatayim

**Travel Times (from matrix):**
- Home → Givatayim: 15 min
- Ramat Gan → Givatayim: 10 min
- Givatayim → Home: 15 min
- Givatayim → Herzliya: 25 min
- Herzliya → Givatayim: 25 min

**Gap 1: day_start → Booking A**
```

prevAnchor: day_start @ 08:00, home nextAnchor: Booking A @ 09:00, Ramat Gan

travelFromPrev = 15 min (home → customer) travelToNext = ? (customer → Ramat Gan) - but we need customer → Ramat Gan

Wait - Ramat Gan is the BOOKING location, not where IC needs to be. IC needs to travel: customer → Ramat Gan (to get to Booking A on time)

Let's assume Givatayim → Ramat Gan = 8 min

earliestArrival = 08:00 + 15 + 15 + 5 = 08:35 latestDeparture = 09:00 - 8 - 5 - 15 = 08:32

latestStart = 08:32 - 60 = 07:32

earliestStart (08:35) > latestStart (07:32) → NO SLOTS in this gap

```

**Gap 2: Booking A → Blocked (Lunch)**
```

prevAnchor: Booking A ends @ 10:00, Ramat Gan nextAnchor: Blocked @ 12:00, home

travelFromPrev = 10 min (Ramat Gan → Givatayim) travelToNext = 15 min (Givatayim → home)

earliestArrival = 10:00 + 15 + 10 + 5 = 10:30 latestDeparture = 12:00 - 15 - 5 - 15 = 11:25 latestStart = 11:25 - 60 = 10:25

earliestStart = roundUp(10:30, 30) = 10:30 10:30 <= 10:25? NO → NO SLOTS in this gap

```

**Gap 3: Blocked (Lunch) → Booking B**
```

prevAnchor: Blocked ends @ 13:00, home nextAnchor: Booking B @ 14:00, Herzliya

travelFromPrev = 15 min (home → Givatayim) travelToNext = 25 min (Givatayim → Herzliya)

earliestArrival = 13:00 + 15 + 15 + 5 = 13:35 latestDeparture = 14:00 - 25 - 5 - 15 = 13:15 latestStart = 13:15 - 60 = 12:15

earliestStart (13:35) > latestStart (12:15) → NO SLOTS in this gap

```

**Gap 4: Booking B → day_end**
```

prevAnchor: Booking B ends @ 15:30, Herzliya nextAnchor: day_end @ 18:00, home

travelFromPrev = 25 min (Herzliya → Givatayim) travelToNext = 15 min (Givatayim → home)

earliestArrival = 15:30 + 15 + 25 + 5 = 16:15 latestDeparture = 18:00 - 15 - 5 - 15 = 17:25

BUT for day_end, we don't require IC to return home before end of day. So: latestDeparture = 18:00 (just need to finish by end of working hours)

latestStart = 18:00 - 60 = 17:00 earliestStart = roundUp(16:15, 30) = 16:30

SLOTS:

- 16:30 - 17:30 ✓
- 17:00 - 18:00 ✓

```

**Result for this day:**
```

{ date: "2025-01-27", dayOfWeek: "monday", status: "available", workingHours: { start: "08:00", end: "18:00" }, slots: [ { date: "2025-01-27", startTime: "16:30", endTime: "17:30", travelFromPrevious: { durationMinutes: 25, fromType: "previous_booking" }, travelToNext: { durationMinutes: 15, toType: "home_base" }, isFirstOfDay: false, isLastOfDay: false }, { date: "2025-01-27", startTime: "17:00", endTime: "18:00", travelFromPrevious: { durationMinutes: 25, fromType: "previous_booking" }, travelToNext: { durationMinutes: 15, toType: "home_base" }, isFirstOfDay: false, isLastOfDay: true } ] }

```

#### 5.3.5 Edge Cases

**Empty Day (No Existing Bookings):**
```

Anchors: [day_start, day_end] Gap: 08:00 → 18:00

If customer travel from home is 20 min: earliestArrival = 08:00 + 15 + 20 + 5 = 08:40 latestStart = 18:00 - 60 = 17:00

Slots: 09:00, 09:30, 10:00, ... 17:00 (many slots)

````

**Day-End Treatment:**
For the last gap of the day (to day_end), we ignore return-to-home travel time. The IC just needs to finish the service before working hours end.

**Routing API Failure:**
Return error response with fallback contact info. Never show slots without validated travel times.

**Customer Outside Service Area:**
Return OUTSIDE_SERVICE_AREA error immediately (before any travel time calculations).

#### 5.2.6 Performance Optimizations

**Travel Time Caching:**
- Cache in Supabase with 7-day TTL
- Key: (originLat, originLng, destLat, destLng, travelMode) - all rounded to 4 decimals
- Check cache before API call
- Store matrix results after successful API call

**Matrix API Efficiency:**
- OpenRouteService free tier: 5 locations max per call
- For typical day (3-5 bookings): single matrix call
- For busy day (6+ bookings): multiple calls or point-to-point

**Performance Targets:**
- Empty day (3 locations): ~200ms
- 3 bookings (5 locations): ~300ms
- 6 bookings (8 locations): ~500ms
- 10 bookings (12 locations): ~700ms

#### 5.2.7 Implementation Code Structure

```typescript
// src/server/services/slots/generator.ts

export class SlotGeneratorService {
  constructor(
    private routingProvider: RoutingProvider,
    private geoService: GeoService
  ) {}

  async generateSlots(config: SlotGeneratorConfig): Promise<SlotResponse> {
    // Step 1: Load and validate business + service
    const { business, service } = await this.loadAndValidate(config);
    
    // Step 2: Determine which resources to process
    const resources = config.resourceId
      ? [await this.loadResource(config.resourceId)]
      : await this.loadActiveResources(business.id);
    
    // Step 3: Calculate effective date range
    const effectiveRange = this.calculateEffectiveDateRange(config.dateRange, business);
    
    // Step 4: Generate slots for each resource
    const resourceResults: ResourceAvailability[] = [];
    
    for (const resource of resources) {
      // Get effective settings (resource override or business default)
      const effectiveSettings = this.getEffectiveSettings(resource, business);
      
      // Check service area for this resource
      // MVP: Uses business service area (resource override is post-MVP, Professional/Team tiers)
      const inArea = await this.geoService.isInServiceArea(
        business.id,
        config.customerAddressComponents  // Google address_components from booking flow
      );

      if (!inArea) {
        // Skip this resource — customer outside service area
        // (In multi-resource scenario, other resources might still serve this area)
        continue;
      }
      
      // Load schedule for this resource
      const schedule = await this.loadResourceSchedule(resource.id, effectiveRange);
      
      // Get travel times using resource's home base and travel mode
      const travelMatrix = await this.getTravelTimeMatrix(
        effectiveSettings.homeBase,
        config.customerLocation,
        schedule.appointments,
        effectiveSettings.travelMode
      );
      
      // Generate slots for each day
      const days = this.generateDaysForResource(
        effectiveRange,
        resource,
        effectiveSettings,
        service,
        schedule,
        config.customerLocation,
        travelMatrix
      );
      
      resourceResults.push({
        resource: {
          id: resource.id,
          name: resource.name,
          role: resource.role,
          avatarUrl: resource.avatarUrl,
        },
        days
      });
    }
    
    // If NO resources can serve this location, return outside service area error
    if (resourceResults.length === 0) {
      return this.outsideServiceAreaError(business);
    }
    
    return { 
      success: true, 
      data: { 
        business: { id: business.id, name: business.name, serviceAreaLabels: regions.map(r => r.region.label) },
        service,
        resources: resourceResults 
      } 
    };
  }

  /**
   * Service area validation — Google address_component matching
   * Fetches business regions from DB, checks if any component matches customer's address.
   * Pure in-memory comparison — no spatial index, no external call.
   * POST-MVP: extend to check resource-level override regions (Professional/Team tiers).
   */
  // (Implemented on GeoService — see Section 6.4.5)

  /**
   * Get effective settings for a resource (with business defaults as fallback)
   */
  private getEffectiveSettings(resource: Resource, business: Business): EffectiveResourceSettings {
    return {
      homeBase: {
        lat: resource.homeBaseLat,  // Required — always set at resource creation
        lng: resource.homeBaseLng,
      },
      travelMode: resource.travelMode ?? business.defaultTravelMode,
      parkingBuffer: resource.parkingBuffer ?? business.defaultParkingBuffer,
      bufferMinutes: resource.bufferMinutes ?? business.defaultBufferMinutes,
    };
  }

  private generateDaysForResource(
    effectiveRange: DateRange,
    resource: Resource,
    settings: EffectiveResourceSettings,
    service: Service,
    schedule: ResourceSchedule,
    customerLocation: Coordinate,
    travelMatrix: TravelTimeMatrix
  ): DayAvailability[] {
    const days: DayAvailability[] = [];
    
    for (const date of this.iterateDates(effectiveRange)) {
      const dayOfWeek = date.getDay();
      
      // Get all shifts for this day (may be multiple or none)
      const shiftsForDay = schedule.workingHours.filter(wh => wh.dayOfWeek === dayOfWeek);
      
      if (shiftsForDay.length === 0) {
        days.push({ date, dayOfWeek, status: 'not_working', slots: [] });
        continue;
      }
      
      // Sort shifts by start time
      shiftsForDay.sort((a, b) => a.startTime.getTime() - b.startTime.getTime());
      
      // Generate slots for each shift window
      const allSlots: Slot[] = [];
      const scheduleForDay = this.getScheduleForDay(schedule, date);
      
      for (const shift of shiftsForDay) {
        const anchors = this.buildAnchorsForShift(
          date, 
          shift, 
          scheduleForDay, 
          settings.homeBase
        );
        
        const slotsInShift = this.generateSlotsFromAnchors(
          anchors,
          service.durationMinutes,
          settings,
          customerLocation,
          travelMatrix
        );
        
        allSlots.push(...slotsInShift);
      }
      
      // Calculate overall working hours for display (first shift start to last shift end)
      const workingHours = {
        start: shiftsForDay[0].startTime,
        end: shiftsForDay[shiftsForDay.length - 1].endTime,
        shifts: shiftsForDay.map(s => ({ start: s.startTime, end: s.endTime }))
      };
      
      days.push({
        date,
        dayOfWeek,
        status: allSlots.length > 0 ? 'available' : 'fully_booked',
        workingHours,
        slots: allSlots
      });
    }
    
    return days;
  }

  /**
   * Build anchors for a single shift window
   * Each shift is processed independently with its own day_start and day_end
   */
  private buildAnchorsForShift(
    date: Date,
    shift: WorkingHoursEntry,
    scheduleForDay: ResourceDaySchedule,
    homeBase: Coordinate
  ): Anchor[] {
    // Combine date with time portion from shift
    const shiftStart = this.combineDateAndTime(date, shift.startTime);
    const shiftEnd = this.combineDateAndTime(date, shift.endTime);
    
    const anchors: Anchor[] = [
      { type: 'day_start', startTime: shiftStart, endTime: shiftStart, location: homeBase, locationType: 'home_base' },
    ];
    
    // Add existing appointments WITHIN this shift window
    for (const appointment of scheduleForDay.appointments) {
      if (appointment.scheduledStart < shiftEnd && appointment.scheduledEnd > shiftStart) {
        anchors.push({
          type: 'appointment',
          startTime: appointment.scheduledStart,
          endTime: appointment.scheduledEnd,
          location: { lat: appointment.customerLat, lng: appointment.customerLng },
          locationType: 'appointment'
        });
      }
    }
    
    // Add absences WITHIN this shift window
    for (const absence of scheduleForDay.absences) {
      const absenceStart = this.combineDateAndTime(date, absence.startTime);
      const absenceEnd = this.combineDateAndTime(date, absence.endTime);
      
      if (absenceStart < shiftEnd && absenceEnd > shiftStart) {
        anchors.push({
          type: 'absence',
          startTime: absenceStart,
          endTime: absenceEnd,
          // Location resolved after sorting — placeholder for now
          location: absence.absenceLat != null && absence.absenceLng != null
            ? { lat: absence.absenceLat!, lng: absence.absenceLng! }
            : null,  // null = inherit from previous anchor
          locationType: absence.absenceLat != null ? 'located_absence' : 'inherit'
        });
      }
    }
    
    // Sort all anchors chronologically before resolving inherited locations
    anchors.sort((a, b) => a.startTime.getTime() - b.startTime.getTime());
    
    // day_end placeholder — location resolved by propagation below
    anchors.push({
      type: 'day_end',
      startTime: shiftEnd,
      endTime: shiftEnd,
      location: null,  // resolved by propagation
      locationType: 'inherit'
    });

    // -------------------------------------------------------------------------
    // Location propagation pass
    //
    // Rule: if an anchor has no explicit geocoded location (locationType === 'inherit'),
    // it inherits the resolved location of the previous anchor in the sequence.
    //
    // This means:
    //   - A break after an appointment is assumed to be at that appointment's location
    //   - A break after another break inherits that break's resolved location
    //   - day_end resolves to wherever the IC last was — not forced back to home base
    //   - day_start is always home base (nothing before it)
    //
    // If an absence has an explicit geocoded location (e.g. doctor, restaurant),
    // that location wins — no inheritance — and subsequent unlocated anchors
    // then propagate from that explicit location forward.
    // -------------------------------------------------------------------------
    for (let i = 1; i < anchors.length; i++) {
      if (anchors[i].location === null) {
        anchors[i].location = anchors[i - 1].location;
        anchors[i].locationType = anchors[i - 1].locationType;
      }
    }

    return anchors as Anchor[];  // all locations now resolved
    
    return anchors.sort((a, b) => a.startTime.getTime() - b.startTime.getTime());
  }
  
  /**
   * Helper to combine a date with time from another Date object
   */
  private combineDateAndTime(date: Date, time: Date): Date {
    const result = new Date(date);
    result.setHours(time.getHours(), time.getMinutes(), time.getSeconds(), 0);
    return result;
  }

  private generateSlotsFromAnchors(
    anchors: Anchor[],
    customerLocation: Coordinate,
    serviceDuration: number,
    buffer: number,
    parkingBuffer: number,
    slotInterval: number,
    travelMatrix: TravelTimeMatrix
  ): Slot[] {
    const slots: Slot[] = [];
    
    for (let i = 0; i < anchors.length - 1; i++) {
      const prev = anchors[i];
      const next = anchors[i + 1];
      
      const gapSlots = this.generateSlotsInGap(
        prev, next, customerLocation, serviceDuration,
        buffer, parkingBuffer, slotInterval, travelMatrix
      );
      
      slots.push(...gapSlots);
    }
    
    return slots;
  }

  private generateSlotsInGap(
    prev: Anchor,
    next: Anchor,
    customerLocation: Coordinate,
    serviceDuration: number,
    buffer: number,
    parkingBuffer: number,
    slotInterval: number,
    travelMatrix: TravelTimeMatrix
  ): Slot[] {
    const travelFromPrev = travelMatrix.getDuration(prev.location, customerLocation);
    const travelToNext = travelMatrix.getDuration(customerLocation, next.location);
    
    const earliestArrival = addMinutes(prev.endTime, buffer + travelFromPrev + parkingBuffer);
    
    // For day_end, ignore return travel
    const latestDeparture = next.type === 'day_end'
      ? next.startTime
      : addMinutes(next.startTime, -(travelToNext + parkingBuffer + buffer));
    
    const latestStart = addMinutes(latestDeparture, -serviceDuration);
    const earliestStart = this.roundUpToInterval(earliestArrival, slotInterval);
    
    if (isAfter(earliestStart, latestStart)) {
      return [];
    }
    
    const slots: Slot[] = [];
    let slotStart = earliestStart;
    
    while (!isAfter(slotStart, latestStart)) {
      slots.push({
        date: format(slotStart, 'yyyy-MM-dd'),
        startTime: format(slotStart, 'HH:mm'),
        endTime: format(addMinutes(slotStart, serviceDuration), 'HH:mm'),
        travelFromPrevious: {
          durationMinutes: travelFromPrev,
          fromType: prev.type === 'day_start' ? 'home_base' : 'previous_booking'
        },
        travelToNext: {
          durationMinutes: travelToNext,
          toType: next.type === 'day_end' ? 'home_base' : 'next_booking'
        },
        isFirstOfDay: prev.type === 'day_start',
        isLastOfDay: next.type === 'day_end'
      });
      
      slotStart = addMinutes(slotStart, slotInterval);
    }
    
    return slots;
  }
}
````

---

### 5.3 TravelChainService

`src/server/services/travel/travelChain.ts`

Responsible for building and persisting the travel chain for a resource's schedule across a date range. Called after any scheduling operation that changes the Gantt. The V2 optimizer handles travel chain calculation internally as part of its scheduling output — it does not call this service.

#### 5.3.1 Chain Rule

For each date in the range, load all chain events for the resource ordered chronologically:

- **Chain events** = all `BookedAppointment` records with status `SCHEDULED` + all `ResourceAbsence` records (both RECURRING applicable to that day of week and ONE_TIME for that date)

Apply the following rules to produce `travelToMinutes` and `travelFromMinutes` for each event:

|Event type|Position in day|`travelToMinutes`|`travelFromMinutes`|
|---|---|---|---|
|Appointment (always located)|Not last|`routing(prev_location → customerLat/Lng)`|`null`|
|Appointment|Last|`routing(prev_location → customerLat/Lng)`|`routing(customerLat/Lng → homeBase)`|
|Located absence (`absenceLat` present)|Not last|`routing(prev_location → absenceLat/Lng)`|`null`|
|Located absence|Last|`routing(prev_location → absenceLat/Lng)`|`routing(absenceLat/Lng → homeBase)`|
|Non-located absence|Not last|`null`|`null`|
|Non-located absence|Last|`null`|`routing(prev_located_anchor → homeBase)`|

**`prev_location`** — the location of the previous located event in the chain. For the first event of the day, `prev_location` = resource home base.

**`prev_located_anchor`** — for a non-located last event, the travel-from distance is calculated from the last known located position before it (walking back up the chain to find the nearest located event).

**`homeBase`** — effective resource home base (resource override if set, otherwise business default).

#### 5.3.2 Interface

```typescript
// src/server/services/travel/travelChain.ts

export interface TravelChainInput {
  businessId: string
  resourceIds?: string[]  // If omitted — all active resources for the business
  startDate: Date
  endDate: Date
}

export class TravelChainService {
  constructor(private routingProvider: RoutingProvider) {}

  /**
   * Rebuild the full travel chain for all specified resources across the date range.
   * Persists travelToMinutes and travelFromMinutes on BookedAppointment and ResourceAbsence.
   *
   * Called after any scheduling operation that modifies the Gantt:
   *   - appointment.create (customer self-booking)
   *   - appointment.cancelByBusiness
   *   - appointmentToken.cancelByToken (customer magic link cancellation)
   *   - schedule.updateTravels (manual IC trigger from Schedule View)
   *
   * NOT called after:
   *   - Status transitions that don't affect schedule position (PENDING_APPROVAL → SCHEDULED)
   *   - Approval expiry (appointment was never SCHEDULED, no chain change)
   *   - Reminder sends
   *
   * The V2 optimizer calls this internally as part of its scheduling output —
   * the optimizer already has all travel times from its OR-Tools solve.
   *
   * Uses a single ORS matrix call per resource per day — same approach as SlotGeneratorService.
   * Typical day: 6 locations (homeBase + 5 events) = 1 matrix call.
   * ORS free tier: 5 locations max per call. Days with 6+ located events use 2 calls.
   * 7-day horizon × 3 resources = 7–14 matrix calls total — well within free tier.
   */
  async rebuild(input: TravelChainInput): Promise<void> {
    const resources = input.resourceIds
      ? await this.loadResources(input.resourceIds)
      : await this.loadActiveResources(input.businessId)

    for (const resource of resources) {
      const effectiveHomeBase = this.getEffectiveHomeBase(resource)

      for (const date of this.iterateDates(input.startDate, input.endDate)) {
        const events = await this.loadChainEvents(resource.id, date)

        if (events.length === 0) continue

        const updates = await this.buildChain(events, effectiveHomeBase)
        await this.persistChain(updates)
      }
    }
  }

  /**
   * Load all chain events for a resource on a given date, ordered chronologically.
   * Includes: SCHEDULED appointments + applicable absences (RECURRING for day-of-week,
   * ONE_TIME for exact date).
   */
  private async loadChainEvents(resourceId: string, date: Date): Promise<ChainEvent[]> {
    // Load appointments (SCHEDULED only — PENDING_APPROVAL not yet on Gantt)
    // Load absences (RECURRING where dayOfWeek matches + ONE_TIME where date matches)
    // Merge and sort by start time
    // Return as ChainEvent[] with type tag
  }

  /**
   * Build travel values for a chain of events using a single matrix API call.
   *
   * Collects all unique located positions in the chain (home base + located events),
   * requests a single ORS matrix covering all of them, then reads durations from
   * the matrix result. This mirrors the SlotGeneratorService approach exactly.
   *
   * ORS free tier: 5 locations per matrix call.
   * For a typical day of 5 located events: 6 locations (home + 5) = 1 matrix call.
   * For a busy day exceeding 5 locations: multiple sequential matrix calls,
   * each covering up to 5 locations — same batching logic as SlotGeneratorService.
   *
   * This is significantly more efficient than per-pair routing calls.
   */
  private async buildChain(
    events: ChainEvent[],
    homeBase: Coordinate,
    travelMode: TravelMode
  ): Promise<ChainUpdate[]> {
    // Step 1: Collect all unique located positions in the chain
    const locatedPositions: Coordinate[] = [homeBase]
    for (const event of events) {
      if (this.isLocated(event)) {
        locatedPositions.push(this.getLocation(event))
      }
    }

    // Step 2: Build travel time matrix covering all located positions in one call
    // (or multiple calls if > 5 locations due to ORS free tier limit)
    const matrix = await this.routingProvider.getTravelTimeMatrix(
      locatedPositions,
      travelMode
    )

    // Step 3: Walk the chain and read durations from matrix
    const updates: ChainUpdate[] = []
    let prevLocatedIndex = 0  // index into locatedPositions — starts at homeBase
    let locatedEventCount = 0

    for (let i = 0; i < events.length; i++) {
      const event = events[i]
      const isLast = i === events.length - 1
      const isLocated = this.isLocated(event)

      let travelTo: number | null = null
      let travelFrom: number | null = null

      if (isLocated) {
        const thisIndex = ++locatedEventCount  // homeBase is index 0
        travelTo = matrix.getDuration(
          locatedPositions[prevLocatedIndex],
          locatedPositions[thisIndex]
        )
        if (isLast) {
          travelFrom = matrix.getDuration(
            locatedPositions[thisIndex],
            homeBase  // index 0
          )
        }
        prevLocatedIndex = thisIndex
      } else {
        // Non-located: travelTo = null, prevLocatedIndex unchanged
        if (isLast) {
          // Return from last known located position to home base
          travelFrom = matrix.getDuration(
            locatedPositions[prevLocatedIndex],
            homeBase
          )
        }
      }

      updates.push({ event, travelToMinutes: travelTo, travelFromMinutes: travelFrom })
    }

    return updates
  }

  private isLocated(event: ChainEvent): boolean {
    if (event.type === 'appointment') return true  // appointments always have customerLat/Lng
    return event.absence.absenceLat != null
  }

  private getLocation(event: ChainEvent): Coordinate {
    if (event.type === 'appointment') {
      return { lat: event.appointment.customerLat, lng: event.appointment.customerLng }
    }
    return { lat: event.absence.absenceLat!, lng: event.absence.absenceLng! }
  }

  /**
   * Persist travelToMinutes and travelFromMinutes on all updated records
   * in a single transaction.
   */
  private async persistChain(updates: ChainUpdate[]): Promise<void> {
    await db.$transaction(
      updates.map(({ event, travelToMinutes, travelFromMinutes }) => {
        if (event.type === 'appointment') {
          return db.bookedAppointment.update({
            where: { id: event.appointment.id },
            data: { travelToMinutes, travelFromMinutes },
          })
        } else {
          return db.resourceAbsence.update({
            where: { id: event.absence.id },
            data: { travelToMinutes, travelFromMinutes },
          })
        }
      })
    )
  }
}

interface ChainEvent {
  type: 'appointment' | 'absence'
  startTime: Date
  endTime: Date
  travelMode: TravelMode
  appointment?: BookedAppointment
  absence?: ResourceAbsence
}

interface ChainUpdate {
  event: ChainEvent
  travelToMinutes: number | null
  travelFromMinutes: number | null
}
```

#### 5.3.3 Call Sites

`TravelChainService.rebuild` is called **outside the booking transaction** on cancellation and manual refresh only. It is **not called on `appointment.create`** — the create path uses the `matrixSnapshot` from `getAvailableSlots` instead (see below).

```typescript
// appointment.create — uses matrixSnapshot directly, NOT TravelChainService.rebuild
// See appointment router step 9 for full TTL check + chain rebuild logic.
// TravelChainService is bypassed entirely on this path.

// appointment.cancelByBusiness — after status update + notifications
await travelChainService.rebuild({
  businessId: appointment.businessId,
  resourceIds: [appointment.resourceId],
  startDate: appointmentDate,
  endDate: appointmentDate,
})

// appointmentToken.cancelByToken — after customer magic link cancellation
await travelChainService.rebuild({
  businessId: appointment.businessId,
  resourceIds: [appointment.resourceId],
  startDate: appointmentDate,
  endDate: appointmentDate,
})
```

**Why `appointment.create` skips `TravelChainService`:**

The `matrixSnapshot` returned by `getAvailableSlots` already contains all durations needed to rebuild the full chain. Using it on create means:

- 0 additional ORS calls
- All anchors for the day updated with the freshest available durations (traffic-aware when the routing provider supports it)
- TTL of 10 minutes guards against stale data reaching the chain

**Travel chain update logic inside `appointment.create` step 9:**

One of the two paths below always runs — there is no silent skip.

```typescript
const MATRIX_TTL_MS = 10 * 60 * 1000 // 10 minutes

const snapshot = input.matrixSnapshot
const snapshotFresh =
  snapshot &&
  Date.now() - snapshot.timestamp <= MATRIX_TTL_MS &&
  snapshot.date === formatISO(appointmentDate, { representation: 'date' })

if (snapshotFresh) {
  // Fast path: 0 ORS calls — matrix from getAvailableSlots is still valid
  await applyMatrixSnapshotToChain(snapshot!, appointmentDate, resourceId)
} else {
  // Fallback path: matrix expired (customer idled >10 min), absent, or wrong day
  // Make a fresh ORS call — same cost as a cancellation, same result
  await travelChainService.rebuild({
    businessId: appointment.businessId,
    resourceIds: [appointment.resourceId],
    startDate: appointmentDate,
    endDate: appointmentDate,
  })
}
```

**`applyMatrixSnapshotToChain` — fast path implementation:**

````typescript
async function applyMatrixSnapshotToChain(
  snapshot: MatrixSnapshot,
  appointmentDate: Date,
  resourceId: string
): Promise<void> {
  // Load all chain events for (resourceId, appointmentDate) — same query as TravelChainService
  const events = await loadChainEvents(resourceId, appointmentDate)
  if (events.length === 0) return

  // Construct a TravelTimeMatrix adapter over the snapshot's duration table
  // getDuration is an O(1) index lookup — no ORS call
  const matrix: TravelTimeMatrix = {
    getDuration: (from, to) => {
      const fi = snapshot.locations.findIndex(l => l.lat === from.lat && l.lng === from.lng)
      const ti = snapshot.locations.findIndex(l => l.lat === to.lat && l.lng === to.lng)
      if (fi === -1 || ti === -1) return 0  // location not in snapshot — treat as 0
      return snapshot.durations[fi]![ti]!
    },
    getDistance: () => 0,  // not needed for chain persistence
  }

  const homeBase = await getEffectiveHomeBase(resourceId)
  const updates = await buildChainFromMatrix(events, homeBase, matrix)
  await persistChain(updates)
}

#### 5.3.4 Manual Trigger — schedule.updateTravels

A dedicated tRPC procedure for IC-initiated travel chain refresh from the Schedule View toolbar.

```typescript
// Added to scheduleRouter

updateTravels: protectedProcedure
  .input(z.object({
    resourceIds: z.array(z.string()).optional(), // omit = all active resources
    startDate: z.string(), // ISO date
    endDate: z.string(),   // ISO date — max 14 days from startDate
  }))
  .mutation(async ({ ctx, input }) => {
    // Validate business ownership
    // Validate date range (max 14 days)
    // Call travelChainService.rebuild({
    //   businessId: ctx.business.id,
    //   resourceIds: input.resourceIds,
    //   startDate: parseISO(input.startDate),
    //   endDate: parseISO(input.endDate),
    // })
    // Returns: { updatedCount: number } for UI feedback
  }),
````

**UI placement:** A "Recalculate travels" button in the Schedule View toolbar (refresh icon, subtle). In Week view — passes the visible 7-day horizon. In Day view — passes just the selected date. Non-blocking — shows a brief loading state, updates Gantt connectors on completion.

#### 5.3.5 V2 Optimizer Integration

The V2 optimizer (OR-Tools) calculates travel times as part of its solve. When it writes scheduled appointments to the database, it sets `travelToMinutes` and `travelFromMinutes` directly — it does **not** call `TravelChainService.rebuild`. This avoids redundant routing API calls since the optimizer already has all travel times.

The `TravelChainService` is MVP infrastructure only. In V2, the optimizer owns the travel chain for any day it schedules.

## 6. Integration Specifications

### 6.1 OpenRouteService Integration

**Base URL:** `https://api.openrouteservice.org`

**Endpoints Used:**

- `POST /v2/matrix/{profile}` - Travel time matrix
- `POST /v2/directions/{profile}` - Single route (fallback)

**Profiles:**

- `driving-car` — Standard vehicle and motorcycle (ORS has no dedicated motorcycle profile)
- `cycling-electric` — E-bike
- `cycling-regular` — Regular bicycle
- `foot-walking` — Walking

**Rate Limits (Free Tier):**

- 2,000 requests/day
- 40 requests/minute
- 5 locations per matrix request

> **No caching layer.** Travel time calls go directly to ORS. At MVP scale (~200 ICs, ~100 bookings/day) expected daily usage is ~500 requests — well within free tier headroom. When usage approaches limits, the correct solution is upgrading to a traffic-aware provider (Google Routes API, HERE), not building a cache on top of static routing. The interface below is already traffic-ready via the `departureTime` parameter.

**Routing Provider Interface:**

```typescript
// src/server/services/routing/interface.ts

export interface RoutingProvider {
  getTravelTime(
    from: Coordinate,
    to: Coordinate,
    mode: TravelMode,
    departureTime?: Date  // Unused in MVP (ORS free tier is static).
                          // Passed through when upgrading to traffic-aware provider.
  ): Promise<number>;     // minutes
}
```

**Implementation:**

```typescript
// src/server/services/routing/openroute.ts

export class OpenRouteServiceProvider implements RoutingProvider {
  private baseUrl = 'https://api.openrouteservice.org';
  private apiKey = process.env.OPENROUTE_API_KEY!;

  async getTravelTime(
    from: Coordinate,
    to: Coordinate,
    mode: TravelMode,
    departureTime?: Date  // Ignored — ORS free tier does not support real-time traffic
  ): Promise<number> {
    const profile = this.modeToProfile(mode);

    const response = await fetch(`${this.baseUrl}/v2/matrix/${profile}`, {
      method: 'POST',
      headers: {
        'Authorization': this.apiKey,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        locations: [[from.lng, from.lat], [to.lng, to.lat]], // ORS uses [lng, lat]
        metrics: ['duration'],
      }),
    });

    if (!response.ok) {
      if (response.status === 429) throw new RateLimitError('ORS rate limit exceeded');
      throw new RoutingError(`ORS error: ${response.status}`);
    }

    const data = await response.json();
    // Matrix[0][1] = duration in seconds from location 0 to location 1
    return data.durations[0][1] / 60;
  }

  private modeToProfile(mode: TravelMode): string {
    const profiles: Record<TravelMode, string> = {
      DRIVING_CAR:      'driving-car',
      MOTORCYCLE:       'driving-car',  // ORS has no motorcycle profile — driving-car is the correct fallback
      CYCLING_ELECTRIC: 'cycling-electric',
      CYCLING_REGULAR:  'cycling-regular',
      WALKING:          'foot-walking',
    };
    return profiles[mode];
  }
}
```

**Future upgrade path (no interface changes required):**

```typescript
// When upgrading to Google Routes API for traffic-aware routing:
// src/server/services/routing/google-routes.ts

export class GoogleRoutesProvider implements RoutingProvider {
  async getTravelTime(from, to, mode, departureTime?: Date): Promise<number> {
    // departureTime now used — Google Returns traffic-adjusted duration
    // Slot generator passes actual slot start time here automatically
  }
}

// src/server/services/routing/index.ts — swap here only
export const routingProvider: RoutingProvider = new OpenRouteServiceProvider();
// → new GoogleRoutesProvider() when upgrading
```

### 6.2 Twilio SendGrid Integration

**Setup:**

```typescript
import sgMail from '@sendgrid/mail';
sgMail.setApiKey(process.env.SENDGRID_API_KEY!);
```

**Email Templates:**

|Template|Recipient|Trigger|
|---|---|---|
|Booking Confirmed|Customer|Auto-confirm booking|
|Booking Pending|Customer|Manual approval booking|
|Booking Approved|Customer|IC approves|
|Booking Declined|Customer|IC declines|
|Booking Cancelled|Customer|Cancellation|
|Reminder 24h|Customer|24h before appointment|
|Reminder 2h|Customer|2h before appointment|
|New Booking Alert|IC|Any new booking|
|Approval Request|IC|Manual approval needed|
|Booking Cancelled|IC|Customer cancels|
|Daily Agenda|IC|Morning summary|

### 6.3 WhatsApp Notifications (Twilio BSP)

WhatsApp is the **primary notification channel** for MVP. All customer-facing notifications are sent via WhatsApp using Meta-approved message templates through Twilio as the Business Service Provider (BSP).

> **Action required before launch:** Meta template pre-approval takes 1–7 business days. Submit templates as early as possible in the development cycle.

#### 6.3.1 Twilio WhatsApp Setup

```typescript
// src/server/services/notifications/whatsapp.ts
import twilio from 'twilio';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID!,
  process.env.TWILIO_AUTH_TOKEN!
);

async function sendWhatsAppTemplate(
  to: string,          // +972501234567
  templateSid: string, // Pre-approved template SID
  variables: Record<string, string>
): Promise<void> {
  await client.messages.create({
    from: `whatsapp:${process.env.TWILIO_WHATSAPP_NUMBER}`,
    to: `whatsapp:${to}`,
    contentSid: templateSid,
    contentVariables: JSON.stringify(variables),
  });
}
```

#### 6.3.2 Required WhatsApp Templates

All templates must be submitted to Meta for approval in **both Hebrew and English**. Template names use the `journiq_` prefix.

|Template Name|Recipient|Trigger|Variables|
|---|---|---|---|
|`journiq_booking_confirmed`|Customer|Auto-confirm|business_name, service, date, time, address|
|`journiq_booking_pending`|Customer|Manual approval — awaiting IC|business_name, service, date, time|
|`journiq_booking_approved`|Customer|IC approves|business_name, service, date, time, address|
|`journiq_booking_declined`|Customer|IC declines|business_name, fallback_contact|
|`journiq_booking_expired`|Customer|Approval window expired (SYSTEM)|business_name, fallback_contact|
|`journiq_booking_cancelled_by_ic`|Customer|IC cancels|business_name, service, date, fallback_contact|
|`journiq_reminder_24h`|Customer|24h before appointment|business_name, service, time, address|
|`journiq_reminder_2h`|Customer|2h before appointment|business_name, service, time, address|
|`journiq_new_booking_alert`|IC (owner)|Any new booking|customer_name, service, date, time, customer_address|
|`journiq_approval_request`|IC (owner)|Manual booking request|customer_name, service, date, time, approve_url|
|`journiq_booking_cancelled_by_customer`|IC (owner)|Customer cancels|customer_name, service, date|
|`journiq_daily_agenda`|IC (owner)|Morning summary (7 AM)|date, appointment_count, first_appointment_time|

#### 6.3.3 Bilingual Template Strategy

Each template has two language variants registered separately with Meta:

- `journiq_booking_confirmed_he` — Hebrew (default for Israeli market)
- `journiq_booking_confirmed_en` — English (fallback)

Language selection: Use `Business.defaultLanguage` to select template variant. Defaults to Hebrew.

#### 6.3.4 Manual Approval Expiry Automation

The approval window is enforced by a scheduled job (cron or Supabase pg_cron):

```typescript
// src/server/jobs/expire-approvals.ts
// Runs every 30 minutes

export async function expireStaleApprovals(): Promise<void> {
  const now = new Date();
  
  const expired = await db.bookedAppointment.findMany({
    where: {
      status: 'PENDING',
      approvalDeadline: { lte: now },
    },
    include: { business: true, service: true },
  });

  for (const appt of expired) {
    await db.$transaction(async (tx) => {
      await tx.bookedAppointment.update({
        where: { id: appt.id },
        data: {
          status: 'EXPIRED',
          cancelledBy: 'SYSTEM',
          cancelledAt: now,
          cancellationReason: 'approval_window_expired',
        },
      });
    });

    // Notify customer via WhatsApp (outside transaction)
    await sendWhatsAppTemplate(
      appt.customerPhone,
      getTemplateSid('journiq_booking_expired', appt.business.defaultLanguage),
      {
        business_name: appt.business.name,
        fallback_contact: appt.business.fallbackContactValue,
      }
    );
  }
}
```

**`approvalDeadline` calculation (set at booking creation):**

```typescript
function calculateApprovalDeadline(
  bookingCreatedAt: Date,
  business: Business
): Date {
  // Business day = Sun–Thu in Israel (MVP default, not configurable)
  const BUSINESS_DAY_END_HOUR = 18; // 6 PM
  
  const createdAtLocal = toBusinessTimezone(bookingCreatedAt, business.timezone);
  const endOfDay = setHours(createdAtLocal, BUSINESS_DAY_END_HOUR);
  
  // If booking arrived within 1 hour of business day end, extend to next business day
  const minutesUntilEndOfDay = differenceInMinutes(endOfDay, createdAtLocal);
  if (minutesUntilEndOfDay < 60) {
    return endOfNextBusinessDay(createdAtLocal, business.timezone);
  }
  
  return endOfDay;
}
```

#### 6.3.5 Scheduled Jobs — Supabase pg_cron

All time-based automation (approval expiry, appointment reminders) runs via **Supabase pg_cron**. This is already available in Supabase without additional infrastructure — enable the extension once, register jobs as SQL, done.

**Enable pg_cron (one-time setup via Supabase SQL editor):**

```sql
-- Run once in Supabase SQL editor
CREATE EXTENSION IF NOT EXISTS pg_cron;
GRANT USAGE ON SCHEMA cron TO postgres;
```

**Job registrations (run in Supabase SQL editor after deploy):**

```sql
-- Expire stale manual approvals — every 30 minutes
SELECT cron.schedule(
  'expire-stale-approvals',
  '*/30 * * * *',
  $$
    SELECT net.http_post(
      url := current_setting('app.base_url') || '/api/jobs/expire-approvals',
      headers := jsonb_build_object(
        'Content-Type', 'application/json',
        'Authorization', 'Bearer ' || current_setting('app.cron_secret')
      ),
      body := '{}'::jsonb
    );
  $$
);

-- Send 24h appointment reminders — daily at 8 AM Israel time (UTC+2/3)
SELECT cron.schedule(
  'reminders-24h',
  '0 6 * * *',   -- 06:00 UTC = 08:00 Israel Standard Time
  $$
    SELECT net.http_post(
      url := current_setting('app.base_url') || '/api/jobs/reminders-24h',
      headers := jsonb_build_object(
        'Content-Type', 'application/json',
        'Authorization', 'Bearer ' || current_setting('app.cron_secret')
      ),
      body := '{}'::jsonb
    );
  $$
);

-- Send 2h appointment reminders — every hour
SELECT cron.schedule(
  'reminders-2h',
  '0 * * * *',
  $$
    SELECT net.http_post(
      url := current_setting('app.base_url') || '/api/jobs/reminders-2h',
      headers := jsonb_build_object(
        'Content-Type', 'application/json',
        'Authorization', 'Bearer ' || current_setting('app.cron_secret')
      ),
      body := '{}'::jsonb
    );
  $$
);
```

**How it works:** pg_cron calls your Next.js API routes on a schedule. The routes are protected by a `CRON_SECRET` env var — pg_cron passes it as a Bearer token, your route verifies it before executing. Supabase requires the `pg_net` extension (enabled by default) for HTTP calls from pg_cron.

**Job API routes (Next.js):**

```typescript
// src/app/api/jobs/expire-approvals/route.ts
// src/app/api/jobs/reminders-24h/route.ts
// src/app/api/jobs/reminders-2h/route.ts

// All follow this pattern:
export async function POST(req: Request) {
  const auth = req.headers.get('Authorization');
  if (auth !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }
  await runJob(); // expireStaleApprovals() | sendReminders24h() | sendReminders2h()
  return Response.json({ ok: true });
}
```

**Required env vars (add to Section 10):**

```bash
CRON_SECRET="generate-with-openssl-rand-base64-32"
```

**Supabase database settings (set once):**

```sql
ALTER DATABASE postgres SET app.base_url = 'https://your-app-domain.com';
ALTER DATABASE postgres SET app.cron_secret = 'your-cron-secret-here';
```

**Reminder job logic:**

```typescript
// src/server/jobs/reminders-24h.ts
export async function sendReminders24h(): Promise<void> {
  const now = new Date();
  const windowStart = addHours(now, 23);  // appointments starting in 23–25h
  const windowEnd = addHours(now, 25);

  const appointments = await db.bookedAppointment.findMany({
    where: {
      status: 'CONFIRMED',
      scheduledStart: { gte: windowStart, lte: windowEnd },
      reminder24hSentAt: null,  // not yet sent
    },
    include: { business: true, service: true },
  });

  for (const appt of appointments) {
    await sendWhatsAppTemplate(appt.customerPhone, getTemplateSid('journiq_reminder_24h', appt.business.defaultLanguage), {
      business_name: appt.business.name,
      service: appt.service.name,
      date: formatDate(appt.scheduledStart, appt.business.timezone),
      time: formatTime(appt.scheduledStart, appt.business.timezone),
    });
    await db.bookedAppointment.update({
      where: { id: appt.id },
      data: { reminder24hSentAt: now },
    });
  }
}

// src/server/jobs/reminders-2h.ts — identical pattern, 1h45m–2h15m window
```

> **Schema addition required:** Add `reminder24hSentAt DateTime?` and `reminder2hSentAt DateTime?` to `BookedAppointment` to prevent duplicate reminder sends.

---

All address → coordinate resolution is routed through a **provider-agnostic geocoding service**. This decouples the application from any specific geocoding provider and enables migration to self-hosted Photon + Nominatim when Google costs exceed threshold.

> **Note on Nominatim:** Nominatim was previously considered for polygon fetching. This approach was dropped — Google and OSM use different naming conventions and administrative hierarchies for Israeli regions, making cross-source matching unreliable. All service area resolution uses Google exclusively via address_component matching (see 6.4.5).

#### 6.4.1 Provider Interface

```typescript
// src/server/services/geo/geocoding-interface.ts

export interface GeocodeResult {
  lat: number;
  lng: number;
  formattedAddress: string;
  placeId?: string;
  addressComponents: GoogleAddressComponent[];  // Included — used for service area matching
}

export interface GeocodingProvider {
  geocode(address: string): Promise<GeocodeResult | null>;
  reverseGeocode(lat: number, lng: number): Promise<GeocodeResult | null>;
  autocomplete(input: string, sessionToken: string): Promise<AutocompleteResult[]>;
  getPlaceDetails(placeId: string, sessionToken: string): Promise<GeocodeResult | null>;
}
```

#### 6.4.2 Google Places Implementation (MVP)

```typescript
// src/server/services/geo/google-places.ts

export class GooglePlacesProvider implements GeocodingProvider {
  private apiKey = process.env.GOOGLE_PLACES_API_KEY!;

  async geocode(address: string): Promise<GeocodeResult | null> {
    const url = `https://maps.googleapis.com/maps/api/geocode/json?address=${encodeURIComponent(address)}&key=${this.apiKey}&region=il`;
    const res = await fetch(url);
    const data = await res.json();
    if (data.status !== 'OK') return null;
    const { lat, lng } = data.results[0].geometry.location;
    return {
      lat, lng,
      formattedAddress: data.results[0].formatted_address,
      placeId: data.results[0].place_id,
    };
  }

  async autocomplete(input: string): Promise<AutocompleteResult[]> {
    // Uses Places Autocomplete API with Israel bias
    const url = `https://maps.googleapis.com/maps/api/place/autocomplete/json?input=${encodeURIComponent(input)}&key=${this.apiKey}&components=country:il`;
    // ...
  }
}
```

**All geocoding calls log to `ApiUsageEvent`:**

```typescript
await db.apiUsageEvent.create({
  data: { businessId, provider: 'google_places', operation: 'geocode', callCount: 1 }
});
```

#### 6.4.3 Migration Path to Self-Hosted (Future)

When Google API costs exceed threshold, swap the provider implementation:

```typescript
// src/server/services/geo/photon.ts
export class PhotonProvider implements GeocodingProvider { ... }

// src/server/services/geo/index.ts — swap here only
export const geocodingProvider: GeocodingProvider = new GooglePlacesProvider();
// → new PhotonProvider() when migrating
```

No other application code changes required.

#### 6.4.4 Google Places Autocomplete — Session Token Implementation

**Why this matters:** Without session tokens, every keystroke in an address field is a separately billed API request. With session tokens, all keystrokes for one address lookup are bundled into a single billable event. At JOURN's scale this keeps usage comfortably within the free tier (10,000 events/month).

**How sessions work:**

1. User focuses an address input → generate a UUID session token
2. Every autocomplete request while the user types sends the same token
3. User selects a suggestion → one final Place Details request closes the session with the same token
4. Generate a fresh token for the next address interaction

**Rule:** One token per address field focus. Never reuse a token across sessions — Google bills each request individually if a token is reused.

**Implementation:**

```typescript
// src/components/booking/AddressInput.tsx
import { useRef, useState, useCallback } from 'react';
import { v4 as uuidv4 } from 'uuid';

interface Prediction {
  placeId: string;
  description: string;
}

interface AddressResult {
  address: string;
  lat: number;
  lng: number;
  placeId: string;
}

interface AddressInputProps {
  onSelect: (result: AddressResult) => void;
  placeholder?: string;
  bias?: { lat: number; lng: number }; // Bias results toward a location
}

export function AddressInput({ onSelect, placeholder, bias }: AddressInputProps) {
  const [query, setQuery] = useState('');
  const [predictions, setPredictions] = useState<Prediction[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const sessionTokenRef = useRef<string>(uuidv4()); // New token on mount
  const debounceRef = useRef<ReturnType<typeof setTimeout>>();

  // Rotate token when input is focused (new address session)
  const handleFocus = useCallback(() => {
    sessionTokenRef.current = uuidv4();
  }, []);

  const handleChange = useCallback((value: string) => {
    setQuery(value);

    // Don't fire until user has typed 3+ characters
    if (value.length < 3) {
      setPredictions([]);
      return;
    }

    // Debounce: wait 300ms after last keystroke before firing
    clearTimeout(debounceRef.current);
    debounceRef.current = setTimeout(async () => {
      setIsLoading(true);
      try {
        const res = await fetch('/api/trpc/geo.autocomplete', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            input: value,
            sessionToken: sessionTokenRef.current, // Same token for all keystrokes
            bias,
          }),
        });
        const data = await res.json();
        setPredictions(data.result ?? []);
      } finally {
        setIsLoading(false);
      }
    }, 300);
  }, [bias]);

  const handleSelect = useCallback(async (prediction: Prediction) => {
    setQuery(prediction.description);
    setPredictions([]);

    // Terminating call — closes the session, same token
    const res = await fetch('/api/trpc/geo.getPlaceDetails', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        placeId: prediction.placeId,
        sessionToken: sessionTokenRef.current, // Closes this session
      }),
    });
    const data = await res.json();

    // Rotate token immediately after session closes
    sessionTokenRef.current = uuidv4();

    onSelect(data.result);
  }, [onSelect]);

  return (
    <div className="relative">
      <input
        type="text"
        value={query}
        onFocus={handleFocus}
        onChange={(e) => handleChange(e.target.value)}
        placeholder={placeholder ?? 'Enter address'}
        className="w-full px-3 py-2 border border-gray-200 rounded-lg text-sm"
      />
      {predictions.length > 0 && (
        <ul className="absolute z-50 w-full mt-1 bg-white border border-gray-200 rounded-lg shadow-lg">
          {predictions.map((p) => (
            <li
              key={p.placeId}
              onClick={() => handleSelect(p)}
              className="px-3 py-2 text-sm cursor-pointer hover:bg-gray-50"
            >
              {p.description}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**tRPC geo router procedures:**

```typescript
// src/server/api/routers/geo.ts

export const geoRouter = createTRPCRouter({

  // Autocomplete suggestions — passes session token to Google
  autocomplete: publicProcedure
    .input(z.object({
      input: z.string().min(3),
      sessionToken: z.string().uuid(),
      bias: z.object({ lat: z.number(), lng: z.number() }).optional(),
    }))
    .query(async ({ input }) => {
      const url = new URL('https://maps.googleapis.com/maps/api/place/autocomplete/json');
      url.searchParams.set('input', input.input);
      url.searchParams.set('key', process.env.GOOGLE_PLACES_API_KEY!);
      url.searchParams.set('sessiontoken', input.sessionToken); // ← session token
      url.searchParams.set('components', 'country:il');          // Israel only
      url.searchParams.set('language', 'iw');                    // Hebrew-first results
      if (input.bias) {
        url.searchParams.set('location', `${input.bias.lat},${input.bias.lng}`);
        url.searchParams.set('radius', '50000'); // 50km bias radius
      }

      const res = await fetch(url.toString());
      const data = await res.json();

      return (data.predictions ?? []).map((p: any) => ({
        placeId: p.place_id,
        description: p.description,
      }));
    }),

  // Place details (terminating call — closes the session)
  getPlaceDetails: publicProcedure
    .input(z.object({
      placeId: z.string(),
      sessionToken: z.string().uuid(),
    }))
    .query(async ({ ctx, input }) => {
      const url = new URL('https://maps.googleapis.com/maps/api/place/details/json');
      url.searchParams.set('place_id', input.placeId);
      url.searchParams.set('key', process.env.GOOGLE_PLACES_API_KEY!);
      url.searchParams.set('sessiontoken', input.sessionToken); // ← closes session
      url.searchParams.set('fields', 'geometry,formatted_address,address_components'); // address_components needed for service area matching — still Essentials SKU

      const res = await fetch(url.toString());
      const data = await res.json();
      const result = data.result;

      // Log metering event (session = 1 billable event regardless of keystroke count)
      // businessId may be null for public booking flow — log anyway for platform-level tracking
      await db.apiUsageEvent.create({
        data: {
          businessId: ctx.session?.businessId ?? 'platform',
          provider: 'google_places',
          operation: 'autocomplete_session',
          callCount: 1,
        },
      });

      return {
        address: result.formatted_address,
        lat: result.geometry.location.lat,
        lng: result.geometry.location.lng,
        placeId: input.placeId,
        addressComponents: result.address_components as GoogleAddressComponent[], // ← used by validateServiceArea
      };
    }),

  // Service area validation — called BEFORE getAvailableSlots in the booking flow.
  // Returns inArea: true/false. If false, booking flow shows out-of-area screen.
  // getAvailableSlots assumes this has already been called and passed.
  validateServiceArea: publicProcedure
    .input(z.object({
      businessId: z.string(),
      addressComponents: z.array(z.object({
        long_name: z.string(),
        short_name: z.string(),
        types: z.array(z.string()),
      })),
    }))
    .query(async ({ input }) => {
      const inArea = await geoService.isInServiceArea(
        input.businessId,
        input.addressComponents
      );
      return { inArea };
    }),
});
```

**Key implementation rules enforced by this design:**

|Rule|How it's enforced|
|---|---|
|Fresh token per session|`handleFocus` rotates `sessionTokenRef` on input focus|
|Minimum 3 chars before firing|`handleChange` early-returns if `value.length < 3`|
|300ms debounce|`setTimeout` clears on each keystroke|
|Session closes on selection|`handleSelect` passes same token to `getPlaceDetails` then immediately rotates|
|Fields restricted to Essentials + address_components|`fields=geometry,formatted_address,address_components` — address_components needed for service area matching; still Essentials SKU|
|Metering logged per closed session|`ApiUsageEvent` created in `getPlaceDetails`, not in `autocomplete`|

**Where `AddressInput` is used:**

|Location|Bias anchor|
|---|---|
|Customer booking flow — Step 1|First active resource home base coordinates|
|IC-create appointment modal|First active resource home base coordinates|
|Onboarding Step 2 — Business home base|Israel center (31.7683, 35.2137)|
|Onboarding Step 5 — Resource home base|Business home base coordinates|
|Settings — Business home base|Existing home base coordinates|

---

#### 6.4.5 Service Area — GeoService & ServiceAreaPicker

**Service area validation (GeoService.isInServiceArea):**

When a customer's address is resolved via Google Places Details, Google returns `address_components` — a structured array of named geographic components with their types. The service area check compares these components against the business's stored regions. Both sides come from Google, so naming is guaranteed consistent.

```typescript
// src/server/services/geo/geo.service.ts

export class GeoService {

  /**
   * Check whether a customer's address falls within a business's service area.
   * Compares Google address_components against stored ServiceAreaRegion componentValues.
   * No external API call — pure DB lookup + in-memory comparison.
   */
  async isInServiceArea(
    businessId: string,
    customerComponents: GoogleAddressComponent[]
  ): Promise<boolean> {
    const regions = await db.businessServiceArea.findMany({
      where: { businessId },
      include: { region: true },
    });

    if (regions.length === 0) return false; // Business has no service area configured

    return regions.some(({ region }) =>
      customerComponents.some(
        (comp) =>
          comp.types.includes(region.componentType) &&
          comp.long_name === region.componentValue
      )
    );
  }

  /**
   * Resolve a Google Place ID to a ServiceAreaRegion row.
   * Cache-first: if the region is already stored, return it immediately (zero API calls).
   * On cache miss: call Google Places Details to get address_components, store, return.
   * Called once per unique region ever selected on the platform.
   */
  async resolveOrCreateRegion(
    placeId: string,
    label: string,
    sessionToken: string
  ): Promise<ServiceAreaRegion> {
    // 1. Cache hit — return existing
    const existing = await db.serviceAreaRegion.findUnique({
      where: { googlePlaceId: placeId },
    });
    if (existing) return existing;

    // 2. Cache miss — fetch from Google
    const url = new URL('https://maps.googleapis.com/maps/api/place/details/json');
    url.searchParams.set('place_id', placeId);
    url.searchParams.set('key', process.env.GOOGLE_PLACES_API_KEY!);
    url.searchParams.set('sessiontoken', sessionToken);
    url.searchParams.set('fields', 'address_components,name');
    url.searchParams.set('language', 'iw'); // Hebrew-first

    const res = await fetch(url.toString());
    const data = await res.json();
    const components: GoogleAddressComponent[] = data.result.address_components;

    // Extract the most specific matching component
    const targetTypes = ['locality', 'administrative_area_level_2', 'administrative_area_level_1'];
    const primaryComponent = components.find((c) =>
      c.types.some((t) => targetTypes.includes(t))
    );

    if (!primaryComponent) throw new Error(`No mappable component for placeId: ${placeId}`);

    const componentType = primaryComponent.types.find((t) => targetTypes.includes(t))!;

    // 3. Store and return
    return db.serviceAreaRegion.create({
      data: {
        googlePlaceId: placeId,
        label,                                      // User-facing display label
        labelHe: data.result.name,                  // Hebrew name from Google
        componentType,
        componentValue: primaryComponent.long_name, // Exact value used for matching
        countryCode: 'IL',
      },
    });
  }
}
```

**tRPC serviceArea router:**

```typescript
// src/server/api/routers/service-area.ts

export const serviceAreaRouter = createTRPCRouter({

  // Return all regions currently selected by the business
  list: protectedProcedure.query(async ({ ctx }) => {
    return db.businessServiceArea.findMany({
      where: { businessId: ctx.session.businessId },
      include: { region: true },
      orderBy: { region: { label: 'asc' } },
    });
  }),

  // Add a region (resolves or creates ServiceAreaRegion, then links to business)
  add: protectedProcedure
    .input(z.object({
      placeId: z.string(),
      label:   z.string(),
      sessionToken: z.string().uuid(),
    }))
    .mutation(async ({ ctx, input }) => {
      const region = await geoService.resolveOrCreateRegion(
        input.placeId,
        input.label,
        input.sessionToken
      );
      return db.businessServiceArea.upsert({
        where: { businessId_regionId: { businessId: ctx.session.businessId, regionId: region.id } },
        create: { businessId: ctx.session.businessId, regionId: region.id },
        update: {}, // already exists — no-op
      });
    }),

  // Remove a region from the business
  remove: protectedProcedure
    .input(z.object({ regionId: z.string() }))
    .mutation(async ({ ctx, input }) => {
      return db.businessServiceArea.delete({
        where: {
          businessId_regionId: {
            businessId: ctx.session.businessId,
            regionId: input.regionId,
          },
        },
      });
    }),
});
```

**ServiceAreaPicker component spec:**

Used in Onboarding Step 5 and Settings → Booking Behaviour. Not used in Resource view at MVP (read-only chip there links to Settings).

```
┌─────────────────────────────────────────────────────┐
│ Search for cities or districts...                   │  ← Google Places Autocomplete
│                                                     │     types: ['locality',
│  [Tel Aviv-Yafo ×]  [Herzliya ×]  [Netanya ×]      │            'administrative_area_level_1',
│                                                     │            'administrative_area_level_2']
│  + Add area                                         │     country: IL
└─────────────────────────────────────────────────────┘     language: iw
```

Behaviour:

- Uses a separate `RegionInput` component (variant of `AddressInput`) filtered to region-level types only — not full addresses
- Session token lifecycle identical to `AddressInput` (rotate on focus, close on selection)
- On selection: calls `serviceArea.add` tRPC mutation; chip appears immediately (optimistic)
- On chip dismiss (×): calls `serviceArea.remove` mutation; chip disappears immediately (optimistic)
- Minimum 1 region required before onboarding can advance
- No maximum enforced at MVP (reasonable self-limiting — ICs won't select 50 regions)
- Read-only mode (Settings context): same component, chips not dismissible, edit button opens modal

**RegionInput autocomplete tRPC procedure (addition to geoRouter):**

```typescript
// Filters to region-level types only — not full addresses
regionAutocomplete: protectedProcedure
  .input(z.object({
    input: z.string().min(2),
    sessionToken: z.string().uuid(),
  }))
  .query(async ({ input }) => {
    const url = new URL('https://maps.googleapis.com/maps/api/place/autocomplete/json');
    url.searchParams.set('input', input.input);
    url.searchParams.set('key', process.env.GOOGLE_PLACES_API_KEY!);
    url.searchParams.set('sessiontoken', input.sessionToken);
    url.searchParams.set('components', 'country:il');
    url.searchParams.set('language', 'iw');
    url.searchParams.set('types', '(regions)'); // Cities and districts only — not addresses

    const res = await fetch(url.toString());
    const data = await res.json();

    return (data.predictions ?? []).map((p: any) => ({
      placeId: p.place_id,
      description: p.description,
    }));
  }),
```

---

### 6.5 Magic Link — Customer Appointment Page

Customers receive a signed magic link in all booking notifications. This link opens a read-only appointment page where they can view details and cancel/reschedule (when enabled).

#### 6.5.1 Token Generation

```typescript
// src/server/services/tokens/appointment-token.ts
import crypto from 'crypto';

export function generateAppointmentToken(appointmentId: string): string {
  const secret = process.env.APPOINTMENT_TOKEN_SECRET!;
  const payload = `${appointmentId}:${Date.now()}`;
  const hmac = crypto.createHmac('sha256', secret).update(payload).digest('hex');
  return `${Buffer.from(payload).toString('base64url')}.${hmac.slice(0, 16)}`;
}

export function createAppointmentToken(appointmentId: string, scheduledStart: Date) {
  const token = generateAppointmentToken(appointmentId);
  const expiresAt = addDays(scheduledStart, 7); // Expires 7 days after appointment
  return { token, expiresAt };
}
```

#### 6.5.2 Token Router

```typescript
// src/server/api/routers/appointment-token.ts

export const appointmentTokenRouter = createTRPCRouter({
  // Public — validate token and return appointment details
  getByToken: publicProcedure
    .input(z.object({ token: z.string() }))
    .query(async ({ ctx, input }) => {
      const record = await db.appointmentToken.findUnique({
        where: { token: input.token },
        include: {
          appointment: {
            include: { business: true, service: true, resource: true }
          }
        }
      });
      if (!record) throw new TRPCError({ code: 'NOT_FOUND' });
      if (record.expiresAt < new Date()) throw new TRPCError({ code: 'UNAUTHORIZED', message: 'Link expired' });
      return record.appointment;
    }),

  // Public — customer cancels via magic link
  cancelByToken: publicProcedure
    .input(z.object({ token: z.string(), reason: z.string().optional() }))
    .mutation(async ({ ctx, input }) => {
      // Validate token, check cancellation window, cancel appointment, notify IC
    }),
});
```

#### 6.5.3 Route

Magic link URL pattern: `/appointment/[token]`

Page: `src/app/(public)/appointment/[token]/page.tsx`

Mockup reference: `journiq-appointment-page.jsx`

---

### 6.6 IC-Created Appointment Flow

ICs can create appointments directly from the dashboard (Schedule view → `+ Create ▾ → New Appointment`). This bypasses the slot engine for feasibility checking — the IC takes responsibility for route validity.

#### 6.6.1 Phone-First Customer Lookup

```typescript
// Appointment router — IC creates appointment
icCreate: protectedProcedure
  .input(z.object({
    phone: z.string(),
    customerName: z.string().min(2),
    customerEmail: z.string().email().optional(),
    customerAddress: z.string(),
    serviceId: z.string(),
    resourceId: z.string(),
    scheduledStart: z.string(), // ISO datetime
    icNotes: z.string().max(500).optional(),
  }))
  .mutation(async ({ ctx, input }) => {
    // 1. Look up returning customer by phone within this business's appointments
    // 2. Geocode customer address via geocoding layer
    // 3. Create appointment with status CONFIRMED, createdByIc: true
    // 4. Generate AppointmentToken
    // 5. Send WhatsApp confirmation to customer
    // 6. Log ApiUsageEvent for geocode call
  }),

// Look up existing customer by phone (for returning customer banner)
lookupCustomerByPhone: protectedProcedure
  .input(z.object({ phone: z.string() }))
  .query(async ({ ctx, input }) => {
    const previous = await db.bookedAppointment.findFirst({
      where: { businessId: ctx.business.id, customerPhone: input.phone },
      orderBy: { createdAt: 'desc' },
      select: { customerName: true, customerEmail: true, customerAddress: true, customerLat: true, customerLng: true }
    });
    return previous ?? null; // null = new customer
  }),
```

#### 6.6.2 IC-Create Rules

- **Always auto-confirmed** — IC-created appointments skip the approval flow entirely
- **No slot engine validation** — IC takes responsibility for route feasibility
- `createdByIc: true` on the `BookedAppointment` record
- Customer receives WhatsApp confirmation immediately after IC saves
- `icNotes` field is internal — never exposed to customer via magic link or notifications
- CustomerFlag check still applies — ⚠️ badge shown in step 1 if phone matches a flagged customer

---

### 6.7 Anthropic Claude Integration

**Model:** `claude-sonnet-4-20250514`

**Usage:**

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 1024,
  system: systemPrompt,
  messages: [{ role: 'user', content: userMessage }],
});
```

### 6.8 Supabase Integration

**Database:** PostgreSQL via Prisma ORM

**Storage:** IC logo uploads only

```typescript
const { data, error } = await supabase.storage
  .from('logos')
  .upload(`${businessId}/logo.png`, file, {
    contentType: 'image/png',
    upsert: true,
  });

const publicUrl = supabase.storage
  .from('logos')
  .getPublicUrl(`${businessId}/logo.png`).data.publicUrl;
```

---

## 7. Security Considerations

### 7.1 Authentication

- Email + password with bcrypt (cost 12)
- JWT tokens in httpOnly cookies
- 7-day session expiry
- Session table for token revocation

### 7.2 Authorization

- All dashboard routes require authentication
- Ownership validation on all resource access
- Public booking endpoints have rate limiting

### 7.3 Subscription Tier Gates

Feature gates are enforced **server-side** in tRPC procedures. Client-side hiding is for UX only — never a security boundary. The gate helper lives in `src/server/lib/tierGates.ts`.

```typescript
// src/server/lib/tierGates.ts

import { TRPCError } from '@trpc/server';
import { PlanTier } from '@prisma/client';

export const TIER_LIMITS = {
  SOLO: {
    maxResources: 1,
    maxServices: 5,
    aiChat: false,
    optimizedScheduling: false,
    resourceServiceAreaOverride: false,
    crossResourceOptimization: false,
  },
  PROFESSIONAL: {
    maxResources: 3,
    maxServices: Infinity,
    aiChat: true,
    optimizedScheduling: true,
    resourceServiceAreaOverride: true,
    crossResourceOptimization: false,
  },
  TEAM: {
    maxResources: 5,
    maxServices: Infinity,
    aiChat: true,
    optimizedScheduling: true,
    resourceServiceAreaOverride: true,
    crossResourceOptimization: true,
  },
} as const satisfies Record<PlanTier, object>;

type GateFeature = keyof typeof TIER_LIMITS.SOLO;
type GateLimit = 'maxResources' | 'maxServices';

// Throws FORBIDDEN if the business tier does not have access to a boolean feature
export function assertTierFeature(
  tier: PlanTier,
  feature: GateFeature,
  label: string
) {
  if (!TIER_LIMITS[tier][feature]) {
    throw new TRPCError({
      code: 'FORBIDDEN',
      message: `${label} is not available on the ${tier} plan. Upgrade to access this feature.`,
    });
  }
}

// Throws FORBIDDEN if the business has reached its tier limit for a countable resource
export function assertTierLimit(
  tier: PlanTier,
  limit: GateLimit,
  currentCount: number,
  label: string
) {
  const max = TIER_LIMITS[tier][limit];
  if (currentCount >= max) {
    throw new TRPCError({
      code: 'FORBIDDEN',
      message: `${label} limit (${max}) reached for the ${tier} plan. Upgrade to add more.`,
    });
  }
}

// Returns effective tier — TRIAL businesses get PROFESSIONAL feature access
// regardless of their planTier until trialEndsAt
export function effectiveTier(business: {
  planTier: PlanTier;
  subscriptionStatus: string;
}): PlanTier {
  if (business.subscriptionStatus === 'TRIAL') return 'PROFESSIONAL';
  return business.planTier;
}
```

**Usage examples in procedures:**

```typescript
// resource.create — enforce resource count limit
assertTierLimit(effectiveTier(business), 'maxResources', currentCount, 'Resource');

// service.create — enforce service count limit
assertTierLimit(effectiveTier(business), 'maxServices', currentCount, 'Service');

// chat — enforce AI chat access
assertTierFeature(effectiveTier(business), 'aiChat', 'AI Chat');

// appointment.getAvailableSlots — enforce window booking access
if (service.schedulingMode === 'OPTIMIZE') {
  assertTierFeature(effectiveTier(business), 'optimizedScheduling', 'Optimized Scheduling');
}
```

**LOCKED status:** A separate middleware check handles LOCKED accounts — all mutating procedures throw `FORBIDDEN` with message _"Your account is locked. Please choose a subscription plan to continue."_ Read-only procedures (dashboard data, schedule view) remain accessible so the IC can still see their data.

### 7.4 Input Validation

- All inputs validated with Zod schemas
- Coordinate bounds checking
- String length limits
- SQL injection prevented by Prisma

### 7.5 Rate Limiting

- 100 req/min per IP (public endpoints)
- 10 bookings/hour per phone number
- OpenRouteService calls go direct (no cache layer — see Section 6.1)

---

## 8. Implementation Phases

### Phase 1: Foundation (Week 1-2)

- [ ] Supabase project setup
- [ ] Prisma schema and migrations
- [ ] Auth router (register, login, logout, getSession)
- [ ] Business router (create, update, getBySlug)
- [ ] Service router (CRUD)
- [ ] Resource router (CRUD: create, update, deactivate, reactivate, delete, list, getById)

### Phase 2: Availability, Absences & Routing (Week 3-4)

- [ ] Working hours management (availability router: addShift, updateShift, deleteShift, setDayShifts, setWeekSchedule, getSchedule, applyTemplate, copySchedule)
- [ ] Absence management (absence router: create, update, delete, list)
- [ ] Geo service (isInServiceArea — address_component matching, resolveOrCreateRegion)
- [ ] ServiceAreaRegion + BusinessServiceArea tables and serviceArea router (list, add, remove)
- [ ] OpenRouteService integration (direct calls, no cache — departureTime-ready interface)

### Phase 3: Slot Generation & Booking (Week 5-6)

- [ ] Full slot generation algorithm
- [ ] getAvailableSlots endpoint
- [ ] Create booking with optimistic locking
- [ ] Cancel booking flows

### Phase 4: Dashboard UI — Resources & Schedule (Week 7-8)

- [ ] `/dashboard/resources` page: resource list with expandable panels
    - [ ] Profile tab (add/edit resource form)
    - [ ] Schedule tab (working hours per day, shift management, templates)
    - [ ] Absences tab (recurring + one-time absence management)
- [ ] `/dashboard/schedule` page: Schedule View
    - [ ] schedule router (`getForDateRange`, `getDayForResource`)
    - [ ] Week Overview mode (Gantt grid — resources × days)
    - [ ] Day Detail mode (timeline — resources × time)
    - [ ] Inline break quick-add (click empty slot → add recurring/one-off break)
    - [ ] Appointment detail side panel (click appointment block)
    - [ ] Week/day navigation and date picker

### Phase 5: Notifications & Polish (Week 9-10)

- [ ] SendGrid email integration
- [ ] All notification templates
- [ ] Manual approval workflow
- [ ] Approval timeout automation
- [ ] AI chat integration

### Phase 6: Production Ready (Week 11)

- [ ] Unit tests for critical paths (slot generation, booking creation, schedule query)
- [ ] Error monitoring setup
- [ ] Performance optimization
- [ ] AWS deployment

---

## 9. Project Structure

```
journiq/
├── CLAUDE.md                         # Claude Code project instructions
├── docs/
│   ├── PRD.md                        # Product Requirements Document
│   └── technical-design.md           # This document
├── mockups/
│   ├── journiq-landing-page.jsx      # Marketing landing page mockup
│   ├── journiq-dashboard.jsx         # IC dashboard mockup
│   ├── journiq-customer-booking.jsx  # Customer booking interface mockup
│   ├── journiq-onboarding.jsx        # Onboarding flow mockup
│   ├── journiq-booking-journey.jsx   # Complete booking journey mockup
│   └── journiq-final-brand.jsx       # Brand/design system reference
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts                       # Development seed data
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx              # Dashboard home
│   │   │   ├── appointments/
│   │   │   │   ├── page.tsx          # Appointment list
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx      # Appointment detail
│   │   │   ├── resources/
│   │   │   │   └── page.tsx          # Resource management (list + expandable panels)
│   │   │   ├── schedule/
│   │   │   │   └── page.tsx          # Schedule View (week overview + day detail)
│   │   │   ├── services/
│   │   │   │   └── page.tsx
│   │   │   ├── availability/
│   │   │   │   └── page.tsx
│   │   │   └── settings/
│   │   │       └── page.tsx
│   │   ├── (public)/
│   │   │   ├── [slug]/
│   │   │   │   └── page.tsx          # Customer booking page (Business Storefront)
│   │   │   ├── booking/
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx      # Booking confirmation
│   │   │   └── appointment/
│   │   │       └── [token]/
│   │   │           └── page.tsx      # Customer magic link appointment page
│   │   ├── onboarding/
│   │   │   └── page.tsx
│   │   ├── api/
│   │   │   └── trpc/
│   │   │       └── [trpc]/
│   │   │           └── route.ts
│   │   ├── layout.tsx
│   │   └── page.tsx                  # Landing page
│   ├── components/
│   │   ├── ui/                       # shadcn/ui components
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   ├── booking/
│   │   │   ├── SlotPicker.tsx
│   │   │   ├── ResourceSelector.tsx
│   │   │   ├── ServiceSelector.tsx
│   │   │   ├── AddressInput.tsx
│   │   │   └── BookingConfirmation.tsx
│   │   ├── dashboard/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── AppointmentCard.tsx
│   │   │   └── StatsCard.tsx
│   │   ├── resources/
│   │   │   ├── ResourceList.tsx           # List of resource rows
│   │   │   ├── ResourcePanel.tsx          # Expandable panel with tabs
│   │   │   ├── ResourceProfileTab.tsx     # Profile form (name, role, home base, etc.)
│   │   │   ├── ResourceScheduleTab.tsx    # Working hours editor per day
│   │   │   ├── ResourceAbsencesTab.tsx    # Recurring + one-time absence list + forms
│   │   │   ├── ShiftEditor.tsx            # Add/edit shift time block
│   │   │   └── AbsenceForm.tsx            # Add/edit absence form
│   │   ├── schedule/
│   │   │   ├── ScheduleView.tsx           # Top-level: manages week/day mode toggle + state
│   │   │   ├── WeekOverview.tsx           # Gantt grid (resources × days)
│   │   │   ├── WeekCell.tsx               # Single cell in the Gantt (resource × day summary)
│   │   │   ├── DayDetail.tsx              # Full timeline view for a selected date
│   │   │   ├── ResourceTimeline.tsx       # Single row in DayDetail (one resource's day)
│   │   │   ├── AppointmentBlock.tsx       # Appointment block rendered on timeline
│   │   │   ├── AbsenceBlock.tsx           # Break/absence block rendered on timeline
│   │   │   ├── InlineBreakPopup.tsx       # Quick-add break popup (click empty slot)
│   │   │   ├── AppointmentSidePanel.tsx   # Detail panel for clicked appointment
│   │   │   └── ScheduleNavbar.tsx         # Date nav, view toggle, resource filter
│   │   └── onboarding/
│   │       ├── StepIndicator.tsx
│   │       ├── BusinessProfileStep.tsx
│   │       ├── ServicesStep.tsx
│   │       ├── ServiceAreaStep.tsx
│   │       └── WorkingHoursStep.tsx
│   ├── server/
│   │   ├── api/
│   │   │   ├── root.ts               # Root router
│   │   │   ├── trpc.ts               # tRPC setup
│   │   │   └── routers/
│   │   │       ├── auth.ts
│   │   │       ├── business.ts
│   │   │       ├── resource.ts       # Resource management
│   │   │       ├── service.ts
│   │   │       ├── availability.ts   # Working hours
│   │   │       ├── absence.ts        # Resource absences
│   │   │       ├── schedule.ts       # Schedule view queries
│   │   │       ├── appointment.ts         # Slots + booking + IC-create
│   │   │       ├── appointment-token.ts   # Magic link token validation
│   │   │       └── chat.ts
│   │   ├── services/
│   │   │   ├── routing/
│   │   │   │   ├── interface.ts            # Provider interface (departureTime-ready)
│   │   │   │   ├── openroute.ts            # OpenRouteService implementation
│   │   │   │   └── index.ts                # Provider export — swap here to change provider
│   │   │   ├── slots/
│   │   │   │   └── generator.ts
│   │   │   ├── geo/
│   │   │   │   ├── geocoding-interface.ts  # Provider-agnostic interface
│   │   │   │   ├── google-places.ts        # Google Places implementation (MVP)
│   │   │   │   └── service.ts              # GeoService: isInServiceArea, resolveOrCreateRegion
│   │   │   ├── notifications/
│   │   │   │   ├── service.ts              # Notification orchestration
│   │   │   │   └── whatsapp.ts             # Twilio WhatsApp BSP integration
│   │   │   ├── tokens/
│   │   │   │   └── appointment-token.ts    # Magic link token generation/validation
│   │   │   ├── jobs/
│   │   │   │   └── expire-approvals.ts     # Cron: expire stale manual approvals
│   │   │   └── ai/
│   │   │       └── chat.ts
│   │   └── db.ts                     # Prisma client
│   ├── lib/
│   │   ├── utils.ts
│   │   ├── constants.ts              # App constants
│   │   └── trpc.ts                   # tRPC client
│   └── types/
│       ├── index.ts                  # Shared TypeScript types
│       └── schedule.ts               # Schedule view types (ScheduleViewResponse, ResourceDaySchedule, etc.)
├── public/
│   └── ...
├── .env.local
├── .env.example
├── next.config.js
├── tailwind.config.js
├── tsconfig.json
└── package.json
```

---

## 10. Environment Configuration

```bash
# .env.example

# ============================================================================
# DATABASE
# ============================================================================
DATABASE_URL="postgresql://postgres:[PASSWORD]@db.[PROJECT].supabase.co:5432/postgres"
DIRECT_URL="postgresql://postgres:[PASSWORD]@db.[PROJECT].supabase.co:5432/postgres"

# ============================================================================
# SUPABASE
# ============================================================================
NEXT_PUBLIC_SUPABASE_URL="https://[PROJECT].supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
SUPABASE_SERVICE_ROLE_KEY="eyJ..."

# ============================================================================
# ROUTING
# ============================================================================
OPENROUTE_API_KEY="your-openrouteservice-api-key"

# ============================================================================
# AI
# ============================================================================
ANTHROPIC_API_KEY="sk-ant-..."

# ============================================================================
# NOTIFICATIONS
# ============================================================================
SENDGRID_API_KEY="SG...."
SENDGRID_FROM_EMAIL="noreply@journiq.ai"

# ============================================================================
# WHATSAPP (Twilio BSP)
# ============================================================================
# Get from: https://console.twilio.com → WhatsApp Senders
TWILIO_ACCOUNT_SID="ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
TWILIO_AUTH_TOKEN="your-twilio-auth-token"
TWILIO_WHATSAPP_NUMBER="+14155238886"  # Twilio sandbox or approved number

# ============================================================================
# GEOCODING (Google Places API)
# ============================================================================
# Get from: https://console.cloud.google.com → APIs & Services → Credentials
# Enable: Places API, Geocoding API
GOOGLE_PLACES_API_KEY="AIza..."

# ============================================================================
# MAGIC LINKS
# ============================================================================
# Generate: openssl rand -base64 32
APPOINTMENT_TOKEN_SECRET="your-random-secret-min-32-chars"

# ============================================================================
# SCHEDULED JOBS (pg_cron authentication)
# ============================================================================
# Generate: openssl rand -base64 32
# Also set in Supabase: ALTER DATABASE postgres SET app.cron_secret = '...';
CRON_SECRET="your-random-secret-min-32-chars"

# ============================================================================
# AUTH
# ============================================================================
NEXTAUTH_SECRET="your-random-secret-min-32-chars"
NEXTAUTH_URL="http://localhost:3000"

# ============================================================================
# ENVIRONMENT
# ============================================================================
NODE_ENV="development"
```

---

## 11. Deployment Guide

### 11.1 Prerequisites

- AWS Account
- Supabase Project
- Domain name (optional but recommended)
- API keys for all services

### 11.2 Database Setup

```bash
# 1. Create Supabase project at supabase.com

# 2. Get connection strings from Settings > Database

# 3. Run migrations
npx prisma migrate deploy

# 4. Generate Prisma Client
npx prisma generate
```

### 11.3 AWS Amplify Deployment (Recommended)

```bash
# 1. Install Amplify CLI
npm install -g @aws-amplify/cli

# 2. Configure Amplify
amplify configure

# 3. Initialize project
amplify init

# 4. Add hosting
amplify add hosting

# 5. Deploy
amplify publish
```

**Environment Variables in Amplify:**

- Set all env vars in Amplify Console > App settings > Environment variables
- Mark sensitive values as "secret"

### 11.4 AWS ECS/Fargate Deployment (Alternative)

**Dockerfile:**

```dockerfile
FROM node:20-alpine AS base

FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx prisma generate
RUN npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma

EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
```

**Deploy Steps:**

1. Build and push Docker image to ECR
2. Create ECS cluster
3. Create task definition with env vars
4. Create service with ALB
5. Configure SSL certificate
6. Set up Route 53 for domain

### 11.5 Post-Deployment Checklist

- [ ] Verify database connection
- [ ] Test auth flow (register, login)
- [ ] Test booking flow end-to-end
- [ ] Verify email delivery
- [ ] Test AI chat responses
- [ ] Monitor error rates
- [ ] Set up alerts for critical errors

---

## Appendix A: Region Boundaries (Israel)

```typescript
// src/server/services/geo/regions.ts

export const REGION_BOUNDARIES: Record<string, Coordinate[]> = {
  'Tel Aviv': [
    { lat: 32.1533, lng: 34.7553 },
    { lat: 32.1533, lng: 34.8053 },
    { lat: 32.0533, lng: 34.8053 },
    { lat: 32.0533, lng: 34.7553 },
  ],
  'Ramat Gan': [
    { lat: 32.0900, lng: 34.8000 },
    { lat: 32.0900, lng: 34.8400 },
    { lat: 32.0500, lng: 34.8400 },
    { lat: 32.0500, lng: 34.8000 },
  ],
  'Givatayim': [
    { lat: 32.0750, lng: 34.8000 },
    { lat: 32.0750, lng: 34.8200 },
    { lat: 32.0550, lng: 34.8200 },
    { lat: 32.0550, lng: 34.8000 },
  ],
  'Herzliya': [
    { lat: 32.1900, lng: 34.8200 },
    { lat: 32.1900, lng: 34.8700 },
    { lat: 32.1500, lng: 34.8700 },
    { lat: 32.1500, lng: 34.8200 },
  ],
  // Add more regions as needed
};
```

---

## Appendix B: Error Codes

|Code|HTTP Status|Description|
|---|---|---|
|`OUTSIDE_SERVICE_AREA`|400|Customer location not in IC's service area|
|`ROUTING_UNAVAILABLE`|503|Cannot calculate travel times|
|`SLOT_NO_LONGER_AVAILABLE`|409|Slot was taken by another customer|
|`BUSINESS_NOT_FOUND`|404|Invalid business ID or slug|
|`SERVICE_NOT_FOUND`|404|Invalid service ID|
|`SERVICE_INACTIVE`|400|Service is deactivated|
|`VALIDATION_ERROR`|400|Input validation failed|
|`UNAUTHORIZED`|401|Authentication required|
|`FORBIDDEN`|403|Not authorized for this resource|

---

## Appendix C: Testing Strategy

### Unit Tests (Critical Paths)

```typescript
// __tests__/services/slots/generator.test.ts

describe('SlotGeneratorService', () => {
  describe('generateSlotsInGap', () => {
    it('returns empty array when gap is too small', () => {
      // Test case where earliestStart > latestStart
    });

    it('generates correct slots for empty day', () => {
      // Test full day availability
    });

    it('respects buffer times', () => {
      // Test buffer is added correctly
    });

    it('handles multiple bookings correctly', () => {
      // Test slot generation between bookings
    });
  });

  describe('buildAnchors', () => {
    it('sorts anchors by start time', () => {
      // Test anchor ordering
    });

    it('includes all booking types', () => {
      // Test all anchor types present
    });
  });
});

// __tests__/services/geo/service.test.ts

describe('GeoService', () => {
  describe('isPointInPolygon', () => {
    it('returns true for customer address matching a business region', () => {});
    it('returns false for customer address outside all business regions', () => {});
    it('handles edge cases correctly', () => {});
  });
});
```

---

_Document Version: 2.0_ _Last Updated: January 2025_ _Status: Ready for Implementation_

---

## 12. Project Setup Guide

This section provides instructions for setting up the JOURN project from scratch using Claude Code.

### 12.1 Prerequisites

Before starting, ensure you have:

1. **Node.js 18+** installed
2. **pnpm** (recommended) or npm
3. **Git** for version control
4. **Claude Code** CLI installed (`npm install -g @anthropic-ai/claude-code`)

### 12.2 External Service Accounts

You will need to create accounts and obtain API keys for the following services:

|Service|Purpose|Free Tier|Setup URL|
|---|---|---|---|
|**Supabase**|PostgreSQL database + file storage|Yes (generous)|https://supabase.com|
|**OpenRouteService**|Routing/travel time API|Yes (2,000 req/day)|https://openrouteservice.org|
|**Anthropic**|AI chat (Claude)|Paid only|https://console.anthropic.com|
|**Twilio SendGrid**|Email notifications|Yes (100/day)|https://sendgrid.com|

### 12.3 Environment Variables

Create a `.env.local` file with the following variables:

```bash
# ============================================================================
# DATABASE (Supabase)
# ============================================================================
# Get from: Supabase Dashboard → Settings → Database → Connection string
DATABASE_URL="postgresql://postgres:[PASSWORD]@db.[PROJECT].supabase.co:5432/postgres?pgbouncer=true"
DIRECT_URL="postgresql://postgres:[PASSWORD]@db.[PROJECT].supabase.co:5432/postgres"

# ============================================================================
# SUPABASE CLIENT
# ============================================================================
# Get from: Supabase Dashboard → Settings → API
NEXT_PUBLIC_SUPABASE_URL="https://[PROJECT].supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
SUPABASE_SERVICE_ROLE_KEY="eyJ..."

# ============================================================================
# ROUTING API (OpenRouteService)
# ============================================================================
# Get from: https://openrouteservice.org/dev/#/home → Sign up → Dashboard
OPENROUTE_API_KEY="your-openrouteservice-api-key"

# ============================================================================
# AI (Anthropic Claude)
# ============================================================================
# Get from: https://console.anthropic.com → API Keys
ANTHROPIC_API_KEY="sk-ant-..."

# ============================================================================
# EMAIL (Twilio SendGrid)
# ============================================================================
# Get from: https://app.sendgrid.com → Settings → API Keys
SENDGRID_API_KEY="SG...."
SENDGRID_FROM_EMAIL="noreply@yourdomain.com"

# ============================================================================
# AUTH
# ============================================================================
# Generate: openssl rand -base64 32
NEXTAUTH_SECRET="your-random-secret-min-32-chars"
NEXTAUTH_URL="http://localhost:3000"

# ============================================================================
# ENVIRONMENT
# ============================================================================
NODE_ENV="development"
```

### 12.4 Claude Code Setup Prompt

Use this prompt with Claude Code to initialize and set up the entire project:

```
I'm building JOURN - a route-aware booking platform for field service businesses in Israel.

Please help me set up the project from scratch. Here's what I need:

1. INITIALIZE PROJECT
   - Create a new T3 app (Next.js 14 + tRPC + Prisma + Tailwind)
   - Use TypeScript strict mode
   - Use pnpm as package manager
   - Set up shadcn/ui for components

2. PROJECT STRUCTURE
   Set up the folder structure as defined in the technical design:
   - /docs folder with PRD.md and technical-design.md
   - /mockups folder (I'll add mockup files later)
   - /prisma with schema and seed.ts
   - /src with app router, components, server, lib, types

3. DATABASE SCHEMA
   Implement the Prisma schema with these models:
   
   - User (id, email, passwordHash, role: OWNER|RESOURCE)
   
   - Business (id, ownerId, name, slug, timezone, serviceArea*, settings)
     * serviceArea is REQUIRED (type: POLYGON|REGIONS, data: Json, displayName)
   
   - Resource (id, businessId, userId?, name, email?, phone?, role?, 
               homeBaseLat?, homeBaseLng?, travelMode?, parkingBuffer?, 
               bufferMinutes?, isActive, sortOrder)
     - Max 5 resources per business
     - Settings override business defaults if set
   
   - WorkingHours (id, resourceId, dayOfWeek: 0-6, startTime: DateTime, endTime: DateTime)
     - Separate table, NOT embedded JSON
     - Multiple rows per day allowed (split shifts)
     - No unique constraint on [resourceId, dayOfWeek]
     - All times in business timezone (MVP)
   
   - ResourceAbsence (id, resourceId, type: RECURRING|ONE_TIME, 
                      dayOfWeek?, date?, startTime: DateTime, endTime: DateTime, reason?)
   
   - Service (id, businessId, name, description, durationMinutes,
              visitFee, visitFeePolicy, jobFeeMin, jobFeeMax, isActive)
   
   - BookedAppointment (id, businessId, resourceId, serviceId,
                        scheduledStart: DateTime, scheduledEnd: DateTime,
                        customer fields, status: PENDING|CONFIRMED|CANCELLED|COMPLETED)
   
   - ServiceAreaRegion (googlePlaceId unique, label, labelHe, componentType, componentValue)
   
   - BusinessServiceArea (businessId, regionId — join table, unique constraint)

4. tRPC ROUTERS
   Create the router structure:
   - auth.ts (register, login, logout, me)
   - business.ts (create, update, getById, getBySlug)
   - resource.ts (CRUD, max 5 validation)
   - service.ts (CRUD)
   - serviceArea.ts (list, add, remove — links business to ServiceAreaRegion)
   - availability.ts (addShift, updateShift, deleteShift, setDayShifts, 
                      setWeekSchedule, getSchedule, copySchedule, applyTemplate)
   - absence.ts (create, update, delete, list - recurring/one-time)
   - appointment.ts (getAvailableSlots, create, approve, decline, cancel, list)
   - geo.ts (autocomplete, getPlaceDetails, regionAutocomplete)
   - chat.ts (AI chat endpoint)

5. SERVICE LAYER
   Create the core services:
   - SlotGeneratorService (route-aware algorithm with multi-shift support)
     * Available Time = Shifts - Absences - Appointments
     * Process each shift window independently
     * Build anchors, find gaps, generate slots
   - RoutingService (OpenRouteService integration)
   - GeoService (isInServiceArea via address_component matching, resolveOrCreateRegion)
   - NotificationService (SendGrid emails)
   - AIService (Claude chat)

6. CONFIGURATION
   - Set up environment variables validation with zod
   - Configure Prisma client
   - Set up tRPC context with auth

KEY DESIGN DECISIONS:
- WorkingHours is a separate table (not embedded JSON) for query flexibility
- Multiple shifts per day supported (e.g., 08:00-12:00 and 14:00-18:00)
- No row in WorkingHours = resource doesn't work that day
- All DateTime fields use Date type (not strings)
- All times stored in business timezone (multi-timezone is post-MVP)

Please start by creating the project structure and Prisma schema.
I have the following environment variables ready: [list your env vars]

Reference the CLAUDE.md file and /docs/technical-design.md for detailed specifications.
```

### 12.5 Step-by-Step Setup

If you prefer manual setup:

```bash
# 1. Create T3 app
pnpm create t3-app@latest journiq --noGit

# When prompted:
# - TypeScript: Yes
# - tRPC: Yes  
# - Prisma: Yes
# - Tailwind: Yes
# - NextAuth: No (we'll use custom auth)
# - App Router: Yes

# 2. Navigate to project
cd journiq

# 3. Add additional dependencies
pnpm add @supabase/supabase-js zod date-fns
pnpm add @sendgrid/mail @anthropic-ai/sdk
pnpm add -D @types/node

# 4. Initialize shadcn/ui
pnpm dlx shadcn-ui@latest init

# 5. Add shadcn components
pnpm dlx shadcn-ui@latest add button card input label
pnpm dlx shadcn-ui@latest add dialog sheet tabs toast
pnpm dlx shadcn-ui@latest add calendar select checkbox

# 6. Create folder structure
mkdir -p docs mockups
mkdir -p prisma
mkdir -p src/server/services/{routing,slots,geo,notifications,ai}
mkdir -p src/server/api/routers
mkdir -p src/types
mkdir -p src/components/{ui,booking,dashboard,onboarding}
mkdir -p src/lib

# 7. Copy your documentation
cp /path/to/PRD.md docs/
cp /path/to/technical-design.md docs/

# 8. Copy CLAUDE.md to project root
cp /path/to/CLAUDE.md .

# 9. Copy mockups (optional - for reference)
cp /path/to/mockups/*.jsx mockups/

# 10. Set up environment
cp .env.example .env.local
# Edit .env.local with your API keys

# 11. Create Prisma schema
# Copy the schema from Section 3.3 of technical-design.md to prisma/schema.prisma
# Key models: User, Business, Resource, WorkingHours, ResourceAbsence,
#             Service, BookedAppointment, ServiceAreaRegion, BusinessServiceArea,
#             AppointmentToken, CustomerFlag, ApiUsageEvent

# 12. Initialize database
pnpm prisma generate
pnpm prisma db push

# 13. (Optional) Create seed file for development data
touch prisma/seed.ts

# 14. Start development
pnpm dev
```

### 12.6 Category Seed Data

The Category table must be seeded before businesses can be created. Add this to your seed file:

```typescript
// prisma/seed.ts

import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

const categories = [
  {
    slug: "plumbing",
    icon: "🔧",
    sortOrder: 1,
    names: { en: "Plumbing", he: "אינסטלציה" }
  },
  {
    slug: "electrical",
    icon: "⚡",
    sortOrder: 2,
    names: { en: "Electrical", he: "חשמל" }
  },
  {
    slug: "hvac",
    icon: "❄️",
    sortOrder: 3,
    names: { en: "HVAC", he: "מיזוג אוויר" }
  },
  {
    slug: "locksmith",
    icon: "🔐",
    sortOrder: 4,
    names: { en: "Locksmith", he: "מנעולן" }
  },
  {
    slug: "cleaning",
    icon: "🧹",
    sortOrder: 5,
    names: { en: "Cleaning", he: "ניקיון" }
  },
  {
    slug: "handyman",
    icon: "🔨",
    sortOrder: 6,
    names: { en: "Handyman", he: "איש תחזוקה" }
  },
  {
    slug: "appliance-repair",
    icon: "🔌",
    sortOrder: 7,
    names: { en: "Appliance Repair", he: "תיקון מכשירי חשמל" }
  },
  {
    slug: "moving",
    icon: "🚚",
    sortOrder: 8,
    names: { en: "Moving", he: "הובלות" }
  },
  {
    slug: "painting",
    icon: "🎨",
    sortOrder: 9,
    names: { en: "Painting", he: "צביעה" }
  },
  {
    slug: "pest-control",
    icon: "🐜",
    sortOrder: 10,
    names: { en: "Pest Control", he: "הדברה" }
  },
  {
    slug: "other",
    icon: "📦",
    sortOrder: 99,
    names: { en: "Other", he: "אחר" }
  }
];

async function seedCategories() {
  console.log('Seeding categories...');
  
  for (const cat of categories) {
    await prisma.category.upsert({
      where: { slug: cat.slug },
      create: {
        slug: cat.slug,
        icon: cat.icon,
        sortOrder: cat.sortOrder,
        names: cat.names,
        isActive: true
      },
      update: {
        icon: cat.icon,
        sortOrder: cat.sortOrder,
        names: cat.names
      }
    });
    console.log(`  ✓ ${cat.slug}`);
  }
  
  console.log('Categories seeded successfully!');
}

async function seedTestUser() {
  console.log('Seeding test user...');

  const email = 'test@journiq.dev';
  const passwordHash = await bcrypt.hash('journiq_test_2024', 12);

  const user = await prisma.user.upsert({
    where: { email },
    create: {
      email,
      passwordHash,
      role: 'OWNER',
    },
    update: {},
  });

  // Find the HVAC category for the test business
  const hvacCategory = await prisma.category.findUnique({ where: { slug: 'hvac' } });
  if (!hvacCategory) throw new Error('Categories must be seeded before test user');

  const business = await prisma.business.upsert({
    where: { slug: 'test-journiq' },
    create: {
      ownerId: user.id,
      name: 'JOURN Test Business',
      slug: 'test-journiq',
      categoryId: hvacCategory.id,
      tagline: 'Test business for development and QA',
      // defaultHomeBaseLat removed — set per Resource

      timezone: 'Asia/Jerusalem',
      fallbackContactMethod: 'WHATSAPP',
      fallbackContactValue: '+972501234567',
      currency: 'ILS',

      // TEAM tier — all features unlocked for development and QA
      // trialEndsAt set to far future so no tier gates ever block during testing
      planTier: 'TEAM',
      subscriptionStatus: 'ACTIVE',
      trialEndsAt: new Date('2099-01-01'),
    },
    update: {
      // Keep tier at TEAM on re-seed — never downgrade test user
      planTier: 'TEAM',
      subscriptionStatus: 'ACTIVE',
      trialEndsAt: new Date('2099-01-01'),
    },
  });

  console.log(`  ✓ Test user: ${email}`);
  console.log(`  ✓ Test business: ${business.slug} (TEAM tier, expires 2099)`);
  console.log('  ✓ Storefront: http://localhost:3000/b/test-journiq');
}

async function main() {
  await seedCategories();
  await seedTestUser();
  // Add other seed functions here
}

// ============================================================================
// TEST USER
// Always provisioned at TEAM tier with a non-expiring trial so all features
// are accessible during development and QA without hitting tier gates.
// Credentials: test@journiq.dev / test1234
// ============================================================================
async function seedTestUser() {
  console.log('Seeding test user...');

  const user = await prisma.user.upsert({
    where: { email: 'test@journiq.dev' },
    create: {
      email: 'test@journiq.dev',
      passwordHash: await hashPassword('test1234'),
      role: 'OWNER',
    },
    update: {},
  });

  const plumbingCategory = await prisma.category.findFirst({
    where: { slug: 'plumbing' },
  });

  await prisma.business.upsert({
    where: { slug: 'test-business' },
    create: {
      ownerId: user.id,
      name: 'Test Business',
      slug: 'test-business',
      categoryId: plumbingCategory!.id,
      tagline: 'Development test business',
      // defaultHomeBaseLat removed — set per Resource

      timezone: 'Asia/Jerusalem',
      fallbackContactMethod: 'WHATSAPP',
      fallbackContactValue: '+972501234567',
      planTier: 'TEAM',
      subscriptionStatus: 'TRIAL',
      trialEndsAt: new Date('2099-01-01'), // Never expires in dev
    },
    update: {
      planTier: 'TEAM',
      subscriptionStatus: 'TRIAL',
      trialEndsAt: new Date('2099-01-01'),
    },
  });

  console.log('  ✓ test@journiq.dev / test1234 (TEAM tier, trial never expires)');
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

**Run seed:**

```bash
pnpm prisma db seed
```

**Add to package.json:**

```json
{
  "prisma": {
    "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
  }
}
```

### 12.8 Key Schema Notes

When implementing the Prisma schema, remember these key points:

```prisma
// WorkingHours - SEPARATE TABLE, not embedded JSON
model WorkingHours {
  id         String   @id @default(cuid())
  resourceId String
  dayOfWeek  Int      // 0 = Sunday, 6 = Saturday
  startTime  DateTime // Time portion only
  endTime    DateTime // Time portion only
  
  resource Resource @relation(fields: [resourceId], references: [id], onDelete: Cascade)
  
  // NO unique constraint - allows multiple shifts per day!
  @@index([resourceId, dayOfWeek])
  @@index([resourceId])
  @@map("working_hours")
}

// Resource - links to WorkingHours, NOT embedded
model Resource {
  // ... other fields
  workingHours WorkingHours[]  // One-to-many relation
  // ...
}
```

**Why separate table instead of embedded JSON?**

- Supports multiple shifts per day (split schedules)
- Can query "all resources working on Sunday"
- DB-level validation
- No row = day off (sparse, efficient storage)

### 12.9 External Service Setup Details

#### Supabase Setup

1. Create new project at https://supabase.com
2. Go to **Settings → Database**
3. Copy connection strings (both pooled and direct)
4. Go to **Settings → API**
5. Copy project URL and anon/service role keys
6. (Optional) Set up Storage bucket for logos/avatars

#### OpenRouteService Setup

1. Sign up at https://openrouteservice.org/dev/#/home
2. Verify email and go to Dashboard
3. Create new API key (free tier: 2,000 requests/day)
4. Note: Use the Matrix API for travel times between multiple points

#### SendGrid Setup

1. Sign up at https://sendgrid.com
2. Go to **Settings → API Keys**
3. Create API key with "Mail Send" permissions
4. Set up **Sender Authentication** (domain or single sender)
5. Create email templates for:
    - Booking confirmation (customer)
    - New booking notification (business)
    - Booking approved/declined
    - Reminder emails

#### Anthropic Setup

1. Sign up at https://console.anthropic.com
2. Add payment method (no free tier for API)
3. Create API key
4. Note: Uses Claude claude-sonnet-4-20250514 for chat

### 12.10 Verification Checklist

After setup, verify:

- [ ] `pnpm dev` starts without errors
- [ ] Database connection works (`pnpm prisma studio`)
- [ ] tRPC playground accessible at `/api/trpc`
- [ ] Environment variables loaded (check via console.log in a route)
- [ ] Tailwind styles working (test with a colored button)

---

## Summary of Multi-Resource Architecture

This document describes the JOURN MVP with full multi-resource support:

|Feature|Description|
|---|---|
|**Multi-Resource**|Each business can have 1-5 Resources (service providers)|
|**Flexible Working Hours**|Separate WorkingHours table supporting multiple shifts per day|
|**Per-Resource Scheduling**|WorkingHours, ResourceAbsence, and BookedAppointment tied to individual Resources|
|**Resource Settings**|Each resource can override business defaults for home base, travel mode, and buffer times|
|**Service Area**|Required at business level; resource-level override is post-MVP|
|**Slot Generation**|Algorithm runs per-resource per-shift, returning availability grouped by resource|
|**DateTime Types**|All time fields use DateTime type for consistency|
|**Timezone**|All times in business timezone (MVP); multi-timezone support is post-MVP|

### Working Hours Design

The WorkingHours model supports maximum flexibility:

```
┌─────────────────────────────────────────────────────────────────┐
│ Example: Split shift on Monday, full day Tuesday, half Wed     │
├────────────┬───────────┬───────────┬───────────────────────────┤
│ resourceId │ dayOfWeek │ startTime │ endTime                   │
├────────────┼───────────┼───────────┼───────────────────────────┤
│ res_abc    │ 1 (Mon)   │ 08:00     │ 12:00   ← Morning shift   │
│ res_abc    │ 1 (Mon)   │ 14:00     │ 18:00   ← Afternoon shift │
│ res_abc    │ 2 (Tue)   │ 08:00     │ 17:00   ← Full day        │
│ res_abc    │ 3 (Wed)   │ 08:00     │ 13:00   ← Half day        │
│ (no rows)  │ 4 (Thu)   │   -       │   -     ← Day off         │
└────────────┴───────────┴───────────┴───────────────────────────┘
```

### Terminology

|Term|Definition|
|---|---|
|Resource|A service provider/technician working for a business|
|WorkingHours|Separate table with multiple shifts per day allowed|
|Shift|A single working window (startTime to endTime)|
|ResourceAbsence|Time when a resource is unavailable|
|BookedAppointment|A confirmed or pending customer appointment|