# KidStreet — Provider Portal
## Business Requirements Document (BRD)
### Version 1.0 · Phase 1 · May 2026

---

## Document Control

| Field | Detail |
|---|---|
| Product | KidStreet Provider Portal |
| Phase | Phase 1 — Core Provider MVP |
| Version | 1.0 |
| Author | KidStreet Product Team |
| Status | Draft — For Developer Review |
| Target Release | TBC |

---


## 1. Executive Summary

KidStreet (brand TBA) is a two-sided marketplace connecting parents with local kids' activity providers — sports academies, dance studios, music schools, arts workshops, tutoring centres, and more. This document defines Phase 1 requirements for the **Provider Portal**, the web application through which businesses create an account, set up their offering, manage classes, handle enrollments and payments, and communicate with parents.

### Design Principles

- **5-minute onboarding** is a non-negotiable success criterion. A new provider must be able to go live — with at least one class published and payments enabled — within 5 minutes of starting the sign-up flow.
- **Self-service first.** Providers should be able to do everything without contacting KidStreet support.
- **AI-assisted.** An embedded AI assistant guides providers through onboarding and common tasks.
- **Parent money flows directly to the provider** via Stripe Connect. KidStreet takes a platform fee per transaction.

### Out of Scope for Phase 1

- Mobile App for iOS and Android
- Billing and invoicing module (Enhanced)
- Customer Engagement (Bulks SMS/Emails, Push Notifications in App)
- Staff Management (Rostering)
- Provider booking integrations (Roller, Omnify)
- Reporting (Attendance etc.)

---

## 2. User Personas

### 2.1 Provider — Micro Business (Primary)
A sole trader or small team running a kids' activity (e.g. a soccer coach, a dance teacher). Often not highly technical. Wants to spend minimal time on admin. Likely using a mix of spreadsheets, group texts and Facebook for current bookings.

### 2.2 Provider — Small Business (Secondary)
A business with 2–10 staff, multiple coaches, several class types across 1–2 venues. May have a bookkeeper. Wants consolidated visibility of all classes, revenue, and messages.

### 2.3 Provider Admin / Staff
A staff member (e.g. receptionist or coordinator) who manages day-to-day operations on behalf of the business owner. Needs role-based access controls (read-only vs. admin) — defined in detail in Phase 2; Phase 1 supports a single admin user per account.

---

## 3. System Overview

```
Provider Portal (Web App)
│
├── Onboarding Wizard - Business Registration (5-step, < 5 min)
├── Dashboard
├── Class Schedule Manager
├── Enrollments Manager
├── Payments Tracker
├── Inbox (Messaging)
├── Business Profile Editor
├── Coach Manager
└── Stripe / Payments Setup
└── Analytics and Reporting
```

The Provider Portal is a **responsive web application** (both desktop & mobile-responsive). The parent-facing portal (View - prototype) reads class data published by providers and exposes it as a public-facing class availability view.

---

## 4. Functional Requirements

---

### FR-01: Provider Onboarding Wizard

**Priority:** P0 — Critical Path

**Goal:** A new provider can go live with ≥1 published class and Stripe connected in ≤ 5 minutes.

#### FR-01.1 — Account Creation
- Provider registers with email + password, or via Google / Apple OAuth.
- On first login, an onboarding wizard launches automatically.
- An elapsed time indicator ("You've been going 4 min 32 sec — almost there!") motivates completion.

#### FR-01.2 — Step 1: Account
- Fields: Full name, email, password (min 8 chars, 1 uppercase, 1 number), mobile number.
- Email verification is required before proceeding beyond Step 1.
- Google / Apple OAuth skips email verification.

#### FR-01.3 — Step 2: Business Information
- Fields: Business name, activity category (multi-select from defined taxonomy — see §8), ABN (validated against ABN Lookup API), business phone, business address / suburb, brief description (≥50 chars), website URL (optional).
- The AI assistant pre-fills the business description if the provider pastes in a URL or selects their category.

#### FR-01.4 — Step 3: Create First Class
- Provider creates at least one class to proceed.
- Fields: Class name, activity category, age group (dropdown), class capacity, price per session, block/term price (optional), day(s) of week, start time, duration, coach (assign from list or add new inline), venue / location, class description.
- Class type: Regular (recurring) or One-off (single session).
- The system auto-generates a recurring schedule for the term based on the selected day(s) and a term-end date.

#### FR-01.5 — Step 4: Payments Setup (Stripe Connect)
- Provider is presented with a clear explanation of how payments work: parents pay at booking; money goes directly to the provider's bank account; KidStreet deducts a platform fee per transaction.
- Platform fee displayed prominently: **1.4% + $0.30 per transaction** (Starter Plan).
- Provider is redirected to Stripe Connect's hosted onboarding flow (OAuth). KidStreet must never collect or store raw banking details directly.
- On return from Stripe, the portal displays a confirmation: "Stripe connected ✓".
- Provider may skip this step and add payments later, but their listing will show "Enquire Only" on the parent side until Stripe is connected.

#### FR-01.6 — Step 5: Go Live
- Summary screen showing: business name, classes created, Stripe status, public profile URL.
- Toggle: "Publish my listing — make it visible to parents" (on by default if Stripe is connected).
- One-click share buttons for the provider's public profile URL.
- AI assistant congratulates provider and surfaces 3 recommended next actions (e.g. "Add photos to your profile", "Add more classes", "Share your profile link").

#### FR-01.7 — Onboarding Progress Persistence
- Progress is saved at each step. If the provider closes the browser, they return to where they left off.
- Incomplete onboarding shows as a notification/banner in the main portal until completed.

---

### FR-02: Provider Dashboard

**Priority:** P0

The dashboard is the provider's home screen after onboarding is complete.

#### FR-02.1 — KPI Summary Cards
The following metrics are displayed as summary cards at the top of the dashboard:
- Classes this week (count)
- Active enrolments (count, with delta vs. prior week)
- Revenue this month (AUD, with % change vs. prior month)
- Kids on waitlist (count across all classes)
- Unread messages (count)

#### FR-02.2 — Today's Classes Panel
- A table showing all classes scheduled for today: time, class name, coach, enrolled/capacity, status (Open / Full / Cancelled).
- Each row links through to the class detail / roster view.

#### FR-02.3 — Action Required Alerts
- Unpaid enrolments (with count and family names).
- Unanswered enquiries older than 24 hours.
- Profile completeness warning (if below 80%).
- Stripe not connected warning (if applicable).

#### FR-02.4 — Revenue Trend Chart
- Bar chart showing revenue for the last 4 rolling weeks.
- Data source: confirmed Stripe payments only.

#### FR-02.5 — Quick Actions
- Buttons: "+ New Class", "Send Bulk Message", "View All Payments".

---

### FR-03: Class Schedule Manager

**Priority:** P0

#### FR-03.1 — Views
The schedule can be viewed in four modes, toggled via a toolbar:
1. **Week View** — 7-column calendar grid, time-slotted rows. Classes appear as colour-coded event blocks.
2. **Month View** — Standard monthly calendar with event dots/blocks per day.
3. **Coach View** — List of coaches, each showing their assigned classes for the current week as coloured pills.
4. **List View** — Tabular list of all upcoming sessions, sortable by date/class/coach.

#### FR-03.2 — Class Event Block (Week/Month view)
Each event block displays: class name, coach name, enrolled/capacity count, and a colour consistent with that class across all views.

A spot count indicator communicates availability:
- Green: spots available
- Orange: ≤ 2 spots remaining
- Red: Full / Waitlist only

Clicking an event opens the Class Detail side panel.

#### FR-03.3 — Create / Edit Class
A modal or side panel with fields (same as FR-01.4). Additional fields available post-onboarding:
- Class type: Regular (recurring) / One-off / Catch-up / Camp / Holiday Program.
- Start date and end date for a term/block.
- Waitlist enabled (yes/no toggle) and maximum waitlist size.
- Visibility: Public (parents can find and book) / Private (invite-only, not listed on parent portal) / Enquire Only (no online payment; parent submits enquiry).
- Cancellation/rescheduling policy (free-text or template).
- Internal notes (not visible to parents).
- Donate class feature (optional)

#### FR-03.4 — Recurring Class Management
- Provider can edit a single session or all future sessions in a recurring series.
- Cancelling a single session triggers automatic notifications to enrolled parents (via in-app message and, if opted in, email).
- Cancellation of a paid session triggers an automatic refund prompt (provider can choose: full refund, credit, or no refund per their policy).

#### FR-03.5 — Coach View
- Displays all coaches as rows with their class assignments for the week as pills.
- Useful for scheduling and conflict detection.
- System prevents assigning the same coach to two overlapping sessions (warning prompt, not hard block in Phase 1).

#### FR-03.6 — Public Calendar Feed
- Each provider's published class schedule generates a read-only feed consumed by the parent-facing portal.
- Feed exposes: class name, age group, day/time, venue, available spots, waitlist status, price, coach name.
- This feed is the data source for the parent portal's class discovery and booking flows.

---

### FR-04: Enrollments & Roster Management

**Priority:** P0

#### FR-04.1 — Enrollment List
- Filterable table of all enrollments across all classes.
- Filters: Class, Status (Confirmed / Pending / Waitlisted / Cancelled), Payment Status (Paid / Unpaid / Partial), Date range.
- Search: by child name, parent name, or email.
- Columns: Child name & age, Parent name, Class, Coach, Enrolled date, Payment status, Enrollment status, Actions.

#### FR-04.2 — Enrollment Actions
Per enrollment row, the provider can:
- View child and parent details.
- Move the child to a different class (triggers parent notification).
- Mark as cancelled (triggers refund prompt and parent notification).
- Send a direct message to the parent.
- Chase payment (sends a payment reminder notification/email to the parent).

#### FR-04.3 — Manual Enrollment
- Provider can manually enroll a child (e.g. for cash/EFTPOS payments taken offline).
- Manual enrollments are flagged as "Manually Added" and payment status can be set to "Paid Offline".

#### FR-04.4 — Class Roster (per class)
- Accessible from the Class Detail view.
- Shows all children enrolled in a specific class session or recurring class.
- Provider can mark attendance (present / absent / excused) per session.
- Attendance is saved and visible for provider records.

#### FR-04.5 — Waitlist Management
- Classes with waitlist enabled show a waitlist tab alongside the roster.
- Waitlisted families are ordered by join date/time (FIFO).
- When a spot opens (cancellation or capacity increase), the system automatically notifies the first person on the waitlist via in-app notification and email.
- Provider can manually offer a spot to any waitlisted family, bypassing FIFO order.
- Waitlisted families have 24 hours to accept before the offer is passed to the next person.

#### FR-04.6 — Export
- Providers can export the enrollment list or class roster to CSV.

---

### FR-05: Payments & Financial Tracking

**Priority:** P0

#### FR-05.1 — Payment Flow
- When a parent books a class on the parent portal, payment is processed via Stripe at the time of booking.
- Funds are held by Stripe and paid out to the provider's bank account on the configured payout schedule (daily, weekly, or monthly — provider's choice).
- KidStreet deducts its platform fee automatically at payout via Stripe Connect's application fee mechanism.
- KidStreet never holds client funds beyond the Stripe payout cycle.

#### FR-05.2 — Transaction List
- Filterable table: Class, Date range, Payment status (Paid / Pending / Failed / Partial / Refunded).
- Columns: Date, Parent name, Child name, Class, Amount, Payment method (card type), Status, Actions.
- Each row: "View Receipt" button generates a printable/downloadable PDF receipt.

#### FR-05.3 — KPI Summary (Payments Page)
- This Month revenue (gross)
- Total collected
- Outstanding (unpaid enrollments)
- Net paid out to provider's account (after platform fee)

#### FR-05.4 — Payment Reminders
- Provider can send a payment reminder to any parent with an unpaid or partial enrollment (one click).
- The reminder is sent as an in-app message and email.
- Reminders can be sent manually or scheduled automatically (e.g. 48 hrs after enrollment if unpaid).

#### FR-05.5 — Refunds
- Provider can initiate a full or partial refund from the Transaction list.
- Refund is processed via Stripe API.
- Parent receives an in-app notification and email confirmation of the refund.
- KidStreet platform fee is not refunded on cancellations initiated by the provider (configurable policy — TBC with legal).

#### FR-05.6 — Pricing Models Supported (Phase 1)
- Per session (pay-as-you-go)
- Term / block pricing (fixed price for a set number of sessions)
- Free trial session (zero-dollar booking with Stripe card capture for future payments)

---

### FR-06: Inbox — Messaging

**Priority:** P0

#### FR-06.1 — Thread Types
The inbox handles two distinct thread types:

| Type | Trigger | Behaviour |
|---|---|---|
| **Enquiry** | A parent submits an enquiry from the provider's public profile (pre-booking) | New thread with tag "Enquiry". Provider can respond and convert the parent to a booking. |
| **Booking Message** | A parent with an active enrollment sends a message (e.g. class move, catch-up, absence) | Thread tagged by topic (Schedule Change / Catch-up Request / Absence Note / General). |

#### FR-06.2 — Inbox Layout
- Split-pane: thread list on the left, active conversation on the right.
- Thread list shows: parent avatar, name, message preview, timestamp, unread indicator, and thread type tag.
- Threads are sorted by most recent activity (unread threads float to the top).
- Tabs: All / Enquiries / Bookings (filtered views).

#### FR-06.3 — Conversation Actions
From the chat header, provider can:
- View the parent's enrollment details.
- Offer a class swap (opens a class-picker modal, generates a structured "Class Swap Offer" message to the parent).
- Mark conversation as resolved / archived.

#### FR-06.4 — Email Passthrough (Off-Platform Replies)
- Provider can configure their Outlook or Gmail address in account settings.
- When enabled, every new in-app message from a parent also forwards a copy to the provider's external email.
- Providers can reply via email; the reply is captured and posted back into the KidStreet thread (via email parsing / webhook — technical implementation TBD with developer team).
- The parent's message to the provider always arrives in-app first; email is supplementary.
- The provider can toggle this setting per-conversation or globally.

#### FR-06.5 — Parent Email Copy
- When the provider sends a message to a parent, a checkbox allows them to also send the message to the parent's registered email. This is pre-ticked by default.
- Parents can opt out of email copies in their own account settings.

#### FR-06.6 — Bulk Messaging
- Provider can send a message to all enrolled families in a specific class (e.g. "Class cancelled this Saturday due to rain").
- Bulk messages are delivered as individual threads per parent, not a group chat.
- Bulk messages are also sent via email to all enrolled parents (cannot be disabled for class-wide notifications for safety/compliance reasons).

#### FR-06.7 — Enquiry Conversion
- From an Enquiry thread, the provider can click "Convert to Booking" which opens a pre-filled enrollment modal (class selector, child details) and sends the parent a direct booking link.

---

### FR-07: Business Profile & Public Listing

**Priority:** P0

#### FR-07.1 — Profile Fields
| Field | Required | Notes |
|---|---|---|
| Business name | Yes | |
| Activity category (primary) | Yes | From taxonomy §8 |
| Activity categories (secondary) | No | Up to 3 additional |
| ABN | Yes | Validated via ABN Lookup API |
| Business address / suburb | Yes | Used for geo-search on parent portal |
| Contact phone | Yes | |
| Contact email | Yes | |
| Website URL | No | |
| About / description | Yes | 50–1000 chars |
| Cover photo | Yes | Min 1200×630px recommended |
| Gallery photos | No | Up to 20 photos |
| Intro video | No | URL (YouTube/Vimeo embed) or direct upload |
| Age range served | Yes | e.g. 3–16 |
| Languages offered | No | |
| WWCC / Working with Children Check | Yes | Checkbox + upload |
| Public liability insurance | Yes | Checkbox + upload |
| Cancellation / refund policy | Yes | Text or template |

#### FR-07.2 — Profile Completeness Score
- A % completeness indicator is shown on the Business Details page.
- Completeness drives search ranking on the parent portal (higher completeness = higher default sort position).
- Specific milestones unlock features (e.g. "Add a video to unlock Featured Provider badge").

#### FR-07.3 — Listing Status Toggle
- A sidebar toggle switches the listing between Live (visible on parent portal) and Paused (hidden from search).
- Pausing does not affect existing enrollments.

#### FR-07.4 — Multiple Venues
- A provider can add multiple venues/locations, each with a name, full address, and optional URL (e.g. Google Maps link).
- Each class is assigned to one venue.

---

### FR-08: Coach Management

**Priority:** P1

#### FR-08.1 — Coach Profiles
- Provider can add coaches with: name, role/title, years of experience, short bio, profile photo.
- Coach profiles are visible on the provider's public listing and on individual class pages (parent portal).

#### FR-08.2 — Coach Assignment
- Each class is assigned to one primary coach.
- The Coach View in the Schedule (FR-03.5) provides a per-coach weekly view.

#### FR-08.3 — Coach Conflict Detection
- The system warns (but does not hard-block) when a coach is assigned to two overlapping sessions.

---

### FR-09: Payments Setup (Stripe Connect)

**Priority:** P0

#### FR-09.1 — Stripe Connect Integration
- KidStreet uses **Stripe Connect (Standard or Express)** to connect provider bank accounts.
- All raw banking data is handled exclusively by Stripe. KidStreet stores only the Stripe `account_id`.
- Stripe onboarding is initiated from the Provider Portal and hosted by Stripe (redirect or embedded, TBD).

#### FR-09.2 — Platform Fee
- Phase 1 Starter Plan: **1.4% + $0.30 per transaction** (KidStreet application fee).
- Stripe's own processing fee is charged separately by Stripe to the provider (currently ~1.75% + $0.30 for Australian cards on Stripe).
- The combined fee is clearly displayed in the portal and during onboarding.

#### FR-09.3 — Payout Schedule
- Provider selects payout frequency: daily, weekly (default), or monthly.
- Configured via the Stripe Connect API; displayed in the KidStreet UI.

#### FR-09.4 — Refund Policy Configuration
- Provider sets a default refund policy from a dropdown:
  - Full refund up to 48 hrs before class (default)
  - Full refund up to 7 days before class
  - Credit only (no cash refunds)
  - No refunds
  - Custom (free text)
- This policy is displayed to parents at the point of booking.

#### FR-09.5 — Payments Not Connected
- If Stripe is not connected, the provider's classes are listed as "Enquire Only" on the parent portal.
- A persistent banner in the portal prompts the provider to complete Stripe setup.

---

### FR-10: Parent-Facing Class Availability (Data Publishing)

**Priority:** P0

This section describes what the Provider Portal publishes to the parent-facing side. The parent portal UI itself is a separate specification document.

#### FR-10.1 — Published Data Per Class
| Field | Source |
|---|---|
| Class name | Provider input |
| Activity category | Provider input |
| Age group | Provider input |
| Schedule (days/times/dates) | Provider input |
| Venue name + address | Provider input |
| Coach name + photo | Coach profile |
| Capacity | Provider input |
| Available spots | Calculated (capacity minus confirmed enrollments) |
| Waitlist status | Calculated (waitlist enabled + spots remaining = 0) |
| Price (per session / term) | Provider input |
| Class description | Provider input |
| Provider profile (name, logo, rating) | Business profile |
| Booking URL | Generated by system |

#### FR-10.2 — Real-Time Updates
- Spot counts update in real-time as bookings occur.
- If a class fills up, the parent-facing view switches from "Book Now" to "Join Waitlist" automatically.
- Cancelled sessions are removed from the parent view with a notification to enrolled parents.

---

## 5. Non-Functional Requirements

| Ref    | Requirement                   | Target                                                              |
|--------|-------------------------------|---------------------------------------------------------------------|
| NFR-01 | Onboarding completion time    | ≤ 10 minutes for a new provider with no prior data                  |
| NFR-02 | Page load time                | < 2s for dashboard and schedule pages (p95)                         |
| NFR-03 | Uptime                        | 99.5% monthly uptime SLA                                            |
| NFR-04 | Responsive design             | Fully usable on desktop (primary), tablet (secondary)               |
| NFR-05 | Browser support               | Chrome, Firefox, Safari, Edge (latest 2 versions)                   |
| NFR-06 | Data security                 | All PII encrypted at rest (AES-256) and in transit (TLS 1.2+)       |
| NFR-07 | Payment security              | PCI-DSS compliant via Stripe; no raw card data handled by KidStreet |
| NFR-08 | GDPR / Privacy Act compliance | Australian Privacy Act 1988 compliant; privacy policy required      |
| NFR-09 | Email deliverability          | Transactional emails via SendGrid or equivalent with DKIM/SPF       |

---

## 6. AI Assistant Requirements

**Priority:** P1

#### AI-01 — Embedded Assistant (Floating Widget)
- An AI assistant widget ("KidStreet Assistant") is accessible from all pages via a floating action button (✨).
- The assistant can answer questions about how to use the portal, guide the provider through tasks, and suggest next actions.
- In onboarding, the assistant appears inline with contextual tips at each step.

#### AI-02 — Onboarding Guidance
- During onboarding, the AI proactively offers to:
  - Auto-fill the business description based on category and name.
  - Suggest pricing benchmarks for the selected activity category and location.
  - Suggest a schedule template (e.g. "Most soccer academies in Melbourne run Saturday 9am and Wednesday 4pm classes").

#### AI-03 — Operational Suggestions
- After onboarding, the AI surfaces suggestions such as:
  - "You have 3 unpaid enrollments — want me to send reminders to all of them?"
  - "Your Development U8 class is full — want to open a waitlist or add a second session?"
  - "You haven't replied to 2 enquiries in 24+ hours — respond now to improve your conversion rate."

#### AI-04 — Draft Message Assistant
- In the Inbox, the AI can draft a reply based on the thread context (e.g. "Draft a polite response offering to move Liam to Wednesday's class").

#### AI-05 — Phase 1 Scope Limitation
- Phase 1 AI is implemented as a guided chatbot with pre-defined intents and GPT-4o for free-text responses. Full autonomous action (e.g. AI sends a message on the provider's behalf) is Phase 2.

---

## 7. Notifications & Alerts

| Event | Channel | Recipient |
|---|---|---|
| New enquiry received | In-app + email | Provider |
| New booking (parent pays) | In-app + email | Provider |
| Class full / waitlist triggered | In-app | Provider |
| Unpaid enrollment (24h reminder) | In-app + email | Provider |
| Message from parent | In-app + email (if configured) | Provider |
| Spot offered to waitlisted parent | In-app + email | Parent |
| Booking confirmed | In-app + email | Parent |
| Payment reminder | In-app + email | Parent |
| Class cancelled by provider | In-app + email | All enrolled parents |
| Refund processed | In-app + email | Parent |
| Stripe payout | Email (from Stripe) | Provider |

---

## 8. Activity Category Taxonomy (Phase 1)

| Category | Sub-categories (examples) |
|---|---|
| ⚽ Sports | Soccer, Swimming, Tennis, Gymnastics, Basketball, Martial Arts, Cricket, AFL, Rugby, Athletics, Netball, Dance Sport |
| 🎨 Arts & Crafts | Drawing & Painting, Pottery, Sculpture, Craft, Sewing |
| 🎵 Music | Guitar, Piano, Drums, Singing, Music Theory, Band |
| 💃 Dance | Ballet, Hip Hop, Jazz, Contemporary, Tap |
| 🧠 Academic | Maths Tutoring, English/Literacy, Science, Coding & Robotics, Chess |
| 🎭 Drama & Performance | Acting, Improv, Musical Theatre, Public Speaking |
| 🌿 Nature & Outdoors | Rock Climbing, Bushcraft, Environment, Gardening |
| 🍳 Cooking & Life Skills | Kids Cooking, First Aid for Kids |
| 🧘 Wellbeing | Yoga for Kids, Mindfulness, Martial Arts (wellbeing focus) |
| 🎮 STEM & Technology | Coding, Robotics, 3D Printing, Game Design |

---

## 8.1. Activity Attributes (Phase 1)

| Attribute               | (examples)                       |
|-------------------------|----------------------------------|
| Donated                 |                                  |
| Paid /Trial /Free       |                                  |
| Neuro-Diversity Support | Autism Friendly,Sensory Friendly |
| Online/Offline          | Delivery Method                  |
| Accessiblity            | Wheelchair, Hearing Loop         |

## 9. User Stories — Priority Matrix

| ID    | User Story                                                                 | Priority | Notes |
|-------|----------------------------------------------------------------------------|---|---|
| US-01 | As a provider, I can sign up and go live with a class in under 5 minutes   | P0 | Core success metric |
| US-02 | As a provider, I can create recurring weekly classes for a term            | P0 | |
| US-03 | As a provider, I can see a weekly calendar view of all my classes          | P0 | |
| US-04 | As a provider, I can view which kids are enrolled in each class            | P0 | |
| US-05 | As a provider, I can see which enrollments are paid and which are not      | P0 | |
| US-06 | As a provider, I can send a payment reminder to an unpaid family           | P0 | |
| US-07 | As a provider, I can receive and reply to messages from parents            | P0 | |
| US-08 | As a provider, I can have parent messages forwarded to my email            | P0 | |
| US-09 | As a provider, I can connect my bank account via Stripe                    | P0 | |
| US-10 | As a provider, I can view revenue and payout summaries                     | P0 | |
| US-11 | As a provider, I can manage a waitlist and offer spots to waiting families | P0 | |
| US-12 | As a provider, I can view my classes by coach/person                       | P1 | Coach View |
| US-13 | As a provider, I can assign coaches to classes and manage coach profiles   | P1 | |
| US-14 | As a provider, I can edit my public business profile and upload photos     | P0 | |
| US-15 | As a provider, I can cancel a single session and notify enrolled parents   | P1 | |
| US-16 | As a provider, I can process a refund for a cancelled class                | P1 | |
| US-17 | As a provider, I can convert an enquiry into a booking                     | P1 | |
| US-18 | As a provider, I can send a bulk message to all families in a class        | P1 | |
| US-19 | As a provider, I can use an AI assistant for guidance during onboarding    | P1 | |
| US-20 | As a provider, I can export my enrollment list to CSV                      | P1 | |
| US-21 | As a provider, I can set a custom refund policy                            | P1 | |

---

## 10. Integration Requirements

| Integration                              | Purpose                                                | Phase |
|------------------------------------------|--------------------------------------------------------|---|
| **Stripe Connect**                       | Payment processing and direct payouts to providers     | P0 |
| **ABN Lookup API** (abr.business.gov.au) | Validate provider's ABN on registration                | P0 |
| **Google Maps / Places API**             | Venue address autocomplete, map on public profile      | P0 |
| **SendGrid (or equivalent)**             | Transactional email delivery                           | P0 |
| **Google OAuth**                         | Provider sign-in with Google                           | P0 |
| **Apple OAuth**                          | Provider sign-in with Apple                            | P1 |
| **Gmail API**                            | Off-platform email reply capture for inbox passthrough | P1 |
| **Vimeo / YouTube embed**                | Provider intro video embed on public profile           | P1 |
| **Roller and TBA**                       | Provider Booking Systems                               | P2 |

---

## 11. Roles & Permissions (Phase 1)

Phase 1 supports a single **Admin** role per provider account. Phase 2 will add role-based access control (Owner / Admin / Coach / View Only).

| Action | Phase 1 |
|---|---|
| Edit business profile | Admin ✓ |
| Create / edit classes | Admin ✓ |
| View enrollments | Admin ✓ |
| Process refunds | Admin ✓ |
| Send messages | Admin ✓ |
| Connect Stripe | Admin ✓ |
| View financial reports | Admin ✓ |

---

## 12. Data Model — Key Entities (High Level)

```
Provider
 └── has many: Classes
 └── has many: Coaches
 └── has many: Venues
 └── has one: StripeAccount
 └── has many: InboxThreads

Class
 └── belongs to: Provider, Coach, Venue
 └── has many: Sessions (individual occurrences)
 └── has many: Enrollments
 └── has one: WaitlistQueue

Session
 └── belongs to: Class
 └── has many: AttendanceRecords

Enrollment
 └── belongs to: Class, Parent, Child
 └── has one: Payment

Payment
 └── belongs to: Enrollment
 └── references: Stripe PaymentIntent / Charge

InboxThread
 └── belongs to: Provider, Parent
 └── has many: Messages
 └── type: Enquiry | Booking

Message
 └── belongs to: InboxThread
 └── sent_by: Provider | Parent
```

---

## 13. Acceptance Criteria — Critical Flows

### AC-01: Onboarding
- [ ] A new provider can complete all steps in < 5 minutes on a desktop browser.
- [ ] Stripe connection succeeds and returns the provider to the portal with a "Connected ✓" confirmation.
- [ ] At least one class is visible on the parent-facing portal within 60 seconds of going live.
- [ ] Provider receives a welcome email with their public profile URL.

### AC-02: Class Creation
- [ ] Provider creates a recurring weekly class and 12 weeks of sessions are auto-generated.
- [ ] Class appears in the weekly calendar view with the correct day/time.
- [ ] Class is visible on the parent portal within 60 seconds of publishing.

### AC-03: Enrollment & Payment
- [ ] A parent books and pays for a class; the enrollment appears in the provider's Enrollments list as "Confirmed / Paid" within 30 seconds.
- [ ] The parent's payment is attributed to the correct class and provider in the Stripe dashboard.

### AC-04: Waitlist
- [ ] When a class is full, the parent portal shows "Join Waitlist" instead of "Book Now".
- [ ] When a provider cancels an enrollment, the first waitlisted parent receives a notification within 60 seconds.

### AC-05: Messaging
- [ ] A parent sends a message; it appears in the provider's Inbox within 5 seconds.
- [ ] Provider replies; the parent sees the reply in their portal within 5 seconds.
- [ ] If email passthrough is enabled, the parent's message also arrives at the provider's external email.

---

## 14. Open Questions for Developer Team

| # | Question | Owner | Due |
|---|---|---|---|
| OQ-01 | Stripe Connect: Express vs Standard for Australian providers? Express has less friction but Standard gives providers more control. Recommend Express for Phase 1. | Tech Lead | — |
| OQ-02 | Email reply-to-thread (inbox passthrough): use an inbound email parsing service (Postmark Inbound, Mailgun Routes) or a Gmail/Outlook webhook? | Tech Lead | — |
| OQ-03 | Real-time messaging: WebSockets (Socket.io) or polling? Recommend WebSockets for < 5s latency target. | Tech Lead | — |
| OQ-04 | AI assistant: OpenAI GPT-4o with function calling, or a simpler intent-based chatbot for Phase 1? | Product Lead | — |
| OQ-05 | ABN Lookup API: does this require a registered API key with the ATO? | Tech Lead | — |
| OQ-06 | Refund policy: does KidStreet retain the platform fee on a provider-initiated refund? Requires legal sign-off. | Legal / Product | — |
| OQ-07 | Multi-timezone support: MVP is Melbourne-focused — UTC+10/+11. Flag for national rollout. | Tech Lead | — |
| OQ-08 | Class cancellation notification — SLA is "within 60 seconds". Confirm with infra team. | Tech Lead | — |

---

## 15. Glossary

| Term           | Definition |
|---|---|
| Provider       | A business or individual offering kids' activities on KidStreet                 |
| Parent         | The adult user on the parent-facing portal who books activities for their child |
| Class          | A recurring or one-off activity offered by a provider                           |
| Session        | A single occurrence of a Class (e.g. one Saturday morning session)              |
| Enrollment     | A confirmed booking linking a Child to a Class                                  |
| Waitlist       | A queue of parents wishing to enroll when a class is full                       |
| Stripe Connect | Stripe's product for marketplace/platform payment routing                       |
| Platform Fee   | KidStreet's charge per transaction (1.4% + $0.30, Starter Plan)                 |
| Coach          | A person assigned to deliver a class on behalf of a provider                    |
| Enquiry        | A pre-booking message from a parent expressing interest in a class              |
| Listing        | A provider's public profile and class catalogue on the parent-facing portal     |
| ABN            | Australian Business Number — required for all provider accounts                 |
| WWCC           | Working with Children Check — required documentation for providers              |
| Donate Classes | Provider can choose to donate classes to the community                          |
---

*End of Document — KidStreet Provider Portal BRD v1.0*
*For questions, contact the KidStreet Product Team.*
