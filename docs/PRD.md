# Rexi — Product Requirements Document (PRD)

**Version:** 1.0  
**Date:** 2026-04-22  
**Status:** Draft  

---

## 1. Executive Summary

**Rexi** is a multi-tenant SaaS platform designed for the integral management of residential complexes (*conjuntos residenciales*). It digitizes and automates the administrative, financial, security, and communication processes of residential complexes in Latin America.

The platform targets administrators, property owners, tenants, and security guards. It must be accessible to users of all ages and technological skill levels, with a clean, intuitive UI supporting Spanish and English.

---

## 2. Research & Justification

### 2.1 Why Digitization?
The transition from manual to digital administration addresses several critical pain points in residential management:
- **Common Issues in Manual Management:** Error-prone manual calculations, disorganized or lost physical documentation, inefficient communication, difficult-to-track fee collection, and conflicts due to lack of financial transparency.
- **Benefits of Rexi:**
    - **Automation:** Drastic reduction in repetitive administrative tasks.
    - **Accessibility:** Centralized, instant access to information for all stakeholders.
    - **Transparency:** Full financial visibility and audit trails.
    - **Agility:** Instant communication and automated, data-driven reporting.

### 2.2 Essential Functionalities (Core Pillars)
1. **Financial Health:** Automated fee generation, multi-channel payments, digital receipts, and delinquency control.
2. **Seamless Communication:** Multi-channel notifications (Push/Email), digital bulletin boards, and real-time messaging.
3. **Optimized Common Areas:** 24/7 online booking, automatic confirmations, and usage-linked fee management.
4. **Managed Maintenance:** Ticket tracking, provider assignment, and preventative scheduling.
5. **Unified Financials:** Budgeting, expense control, and assembly-ready financial statements.
6. **Smart Access Control:** QR-based visitor logs, plate recognition, and real-time entry alerts.

---

## 3. Implementation Strategy & Libraries

To ensure a high-performance, low-cost, and scalable product, the following technical components are selected:

| Module | Component / Library | Purpose |
| :--- | :--- | :--- |
| **Backend Core** | .NET 10 | High-performance enterprise-grade framework. |
| **Database** | PostgreSQL 16 | Robust relational data with support for multi-tenant schemas. |
| **Job Scheduling** | **Hangfire** | Automation of monthly fee generation and recurring alerts. |
| **Payments** | **Wompi / PayU** | Integrated Colombian payment gateway (PSE, Cards). |
| **PDF Generation** | **QuestPDF** | High-speed generation of administration fee receipts. |
| **Notifications** | **Firebase (FCM)** | Cross-platform push notifications for residents. |
| **Real-time** | **SignalR** | Instant alerts for visitor arrivals and emergency comms. |
| **AI / RAG** | **Azure AI / OpenAI** | Powering the document bot and emergency triage. |
| **Security** | **Azure Face API** | Optional facial recognition module. |
| **QR Engine** | **QRCoder** | Offline-capable QR generation for visitor access. |

### 3.1 Selection Criteria
- **Ease of Use:** Intuitive interfaces for non-technical users (e.g., older residents, guards).
- **Support:** Spanish-first documentation and local support (especially for payments).
- **Adaptability:** Scalable and customizable per complex.
- **Security:** AES-256 encryption, automatic backups, and Habeas Data (Ley 1581) compliance.

---

## 4. Vision & Goals

| Goal | Description |
|------|-------------|
| Simplify administration | Centralize all complex management in one platform |
| Financial transparency | Automate fee calculation, invoicing, and payment tracking |
| Security & Access Control | QR codes, facial recognition, vehicle plates for access |
| AI-powered assistance | Document Q&A bot, automated emergency reports, issue triage |
| Scalability | Support 100 complexes in year 1, grow without re-architecture |

---

## 5. Users & Roles

### 3.1 Role Definitions

| Role | Description | Key Capabilities |
|------|-------------|-----------------|
| **Super Admin** | Platform operator | Full access; manages all complexes, billing |
| **Administrator** | Complex admin | Manages complex data, owners, fees, reports |
| **Owner** | Apartment owner | Views fees, registers tenants, receives notifications |
| **Tenant** | Renter | Views info, common area bookings, notifications |
| **Guard** | Security officer | Access control, visitor log, vehicle plate lookup, fines |

### 3.2 User Personas

**Ana (Administrator, 42):** Non-technical admin who manages a complex of 120 apartments. She needs clear dashboards, automatic fee invoicing, and easy report generation.

**Carlos (Owner, 55):** Works full-time; wants to receive payment reminders, track his tenant's status, and book the social hall via mobile.

**Sofia (Tenant, 27):** Expects a modern mobile-first experience to manage packages, visitors, and submit maintenance requests.

**Don Pedro (Guard, 60):** Low digital literacy. Needs simple plate/QR search and clear pass/deny indicators.

---

## 6. Feature Modules

### 4.1 Complex & Infrastructure Management

**Description:** CRUD operations for complexes, towers, and apartments.

**Features:**
- Create/edit/delete complexes with name, address, NIT, contact info, annual budget
- Add towers to a complex (name, number of floors, apartments per floor)
- Add apartments with: floor, number, area (m²), horizontal property coefficient, parking assignment (assigned/communal/none), storage assignment
- Bulk upload apartments per tower via Excel/CSV
- View complex summary: total units, total budget, occupancy rate

**Validations:**
- Complex name must be unique per platform
- Annual budget must be a positive decimal
- Coefficients per complex must sum to 1.0 (100%)
- Apartment number must be unique within a tower

---

### 4.2 Owner & Tenant Management

**Description:** Register and manage owners and tenants linked to apartments.

**Features:**
- Register owner: full name, ID number, ID type, email, phone, profile photo, apartment(s), family members (name, relationship, age)
- Auto-generate user credentials on owner creation (email + temporary password)
- Owner can register a tenant for their apartment: same data fields + owner generates tenant credentials from email
- Owner can have multiple apartments
- Search by: name, ID, email, apartment number
- View apartment timeline: owner → tenant history

**Validations:**
- Email must be unique across the platform
- ID number must be unique
- Phone must be valid format
- Owner must be linked to at least one apartment

---

### 4.3 Fee Calculation & Administration

**Description:** Automated monthly administration fee calculation per apartment.

**Formula:**
```
Administration Fee = (Apt Coefficient + Parking Coefficient + Storage Coefficient) × Annual Budget / 12
```

**Fee Calculation Methods:**
- **Coefficient-based:** `(AptCoefficient + ParkingCoefficient + StorageCoefficient) × AnnualBudget / 12`.
- **Fixed Fee:** A flat rate applied to all apartments regardless of area/coefficient.

**Features:**
- Store horizontal property coefficients for apartment, parking, and storage units
- Support for "Fixed Fee" configuration at the complex level
- Monthly automatic fee generation for all apartments
- **Extraordinary Fees:** Admin can create one-off fees for a specific date (e.g., "Roof Repair") that are automatically added to the next administration bill.
- Display fee breakdown per apartment: base + parking + storage + extraordinary + fines
- Manual recalculation trigger when budget changes
- Generate PDF receipt with: complex logo, period, apartment, owner name, amount breakdown, payment deadline
- Email PDF receipt to owner's registered email

**Validations:**
- Budget must be approved before fee generation
- Coefficients must exist for all active apartments
- Extraordinary fees must be approved/assigned to a specific period

---

### 4.4 Payment Management

**Description:** Track payment status, generate history, apply late fees.

**Features:**
- Record payment: date, amount, payment method, reference number
- Payment history per apartment (owner-visible, admin-visible)
- Overdue notifications: automated reminders at 7 days, 3 days, and day-of deadline
- **Parametrizable Late Interest:**
    - Configurable per complex: % of total fee OR fixed value per day.
    - System calculates `Interest = (DaysOverdue) * (Rate/Value)`.
    - Applied automatically when the payment date > due date.
- Admin reports: paid/overdue/total by date range, export to Excel/PDF
- Dashboard widget: payment summary (% paid, % overdue, total collected)

**Validations:**
- Payment amount must match or exceed fee amount
- Payment date cannot be in the future

---

### 4.5 Visitor Management

**Description:** Register visitors, assign QR codes, control access.

**Features:**
- Owner/tenant registers visitor: name, ID, photo, vehicle plate (optional)
- Visitor type: temporary (with expiry date) or permanent
- Auto-generate unique QR code per visitor
- Guard can scan QR or search by ID to allow/deny entry
- Visitor access log (timestamp, guard, entry/exit)
- Owner receives notification when registered visitor enters

**Validations:**
- Temporary visitor expiry date must be in the future
- QR codes expire automatically on visitor expiry date

---

### 4.6 Vehicle Management

**Description:** Register and search vehicles by plate.

**Features:**
- Register vehicle: plate, make, model, color, owner/tenant/visitor link
- Guard plate search: instant lookup returning owner/tenant info + entry authorization status
- Vehicle access log
- Filter vehicles by type (resident vs. visitor)

**Validations:**
- Plate format validated by regex
- Plate must be unique per registered vehicle (residents; visitors may repeat with different visit records)

---

### 4.7 Common Areas Management

**Description:** Manage shared facilities, bookings, and usage fees.

**Features:**
- CRUD for common areas: name, description, capacity, photos, fee (if applicable, $0 = free), available hours, booking rules
- Owners/tenants can book available slots
- Admin approval workflow for bookings (optional per area)
- Booking calendar view per area
- Email/push notification on booking confirmation or cancellation
- Usage history per area and per resident

**Validations:**
- Cannot double-book same area and time slot
- Booking must be made at least N hours in advance (configurable)
- Outstanding debt blocks booking (configurable)

---

### 4.8 Notifications & Communications

**Description:** Platform-wide and targeted notification system.

**Channels:**
- In-app notification center
- Email
- Web push notification

**Types:**
- 📦 Package arrival (guard sends → owner/tenant notified)
- 💰 Payment reminder (automated)
- 📢 General announcements (admin → all or selected residents)
- 🚗 Vehicle/visitor entry alerts
- 📅 Common area booking confirmations
- ⚠️ Fine issued notifications

**Features:**
- Admin compose & send announcement to: all residents, per tower, per apartment
- Notification history per user
- User notification preferences (which channels)

---

### 4.9 Document Management & AI Bot

**Description:** Store complex documents; AI bot answers resident questions.

**Features:**
- Upload complex documents: coexistence rules, regulations, circulars (PDF/Word)
- Document listing with search and version history
- AI chatbot (RAG-based): trained on complex documents, answers resident questions in natural language
- Bot available in-app and optionally via WhatsApp integration
- Bot escalation: if confidence is low, route to admin

**Languages:** Spanish and English (auto-detect)

---

### 4.10 Fines Management

**Description:** Guards issue fines; admin approves; residents can appeal.

**Workflow:**
1. Guard documents infraction (description, photos, rule reference)
2. Admin reviews and approves/rejects fine
3. Owner/tenant notified of approved fine
4. Owner/tenant can file appeal with justification
5. Admin reviews appeal and approves/rejects
6. Approved fines added to monthly administration fee

**Features:**
- Fine CRUD with status tracking (pending → approved → appealed → resolved)
- Fine amount configurable per infraction type
- Fine history per apartment
- Report: fines by date range, status, amount

**Validations:**
- Fine cannot be issued without photo evidence
- Appeal must be filed within configurable deadline (e.g., 5 days)

---

### 4.11 AI Emergency Reporting

**Description:** AI-assisted report generation for emergency services.

**Features:**
- Resident opens emergency chat (Fire, Police, Ambulance, Utilities: Power/Water/Gas)
- Conversational AI collects incident details (what, where, when, who is affected)
- AI generates structured report formatted for the relevant authority
- Admin notified instantly
- Option to call emergency number directly from the app
- Report saved in incident log

---

### 4.12 Issue / Incident Reporting

**Description:** Resident-reported issues with AI-assisted triage.

**Features:**
- Resident submits issue: category, description, photos
- AI categorizes priority: Critical / High / Medium / Low
- Admin receives categorized alert
- Admin assigns issue to staff or external contractor
- Issue status tracking: open → in-progress → resolved
- Resident notified on status updates
- Manual priority override by admin

---

### 6.14 Assembly & Financial Reporting

**Description:** Generate comprehensive reports for the annual/extraordinary assembly.

**Features:**
- **Monthly Collection vs. Expenses Report:** Breakdown of total collection vs. actual common expenses.
- **Financial Statement Base:** Export data in a format suitable for generating balance sheets and income statements.
- **Assembly Pack:** PDF export containing collection rates, top debtors, and expense summaries for the month/year.

---

### 4.13 Facial Recognition (Optional)

**Description:** Residents optionally enroll face data for contactless entry.

**Features:**
- Resident enrolls face via app camera (multiple angles)
- Guard station camera recognizes face → displays resident info
- Entry logged automatically
- Easy opt-in/opt-out by resident
- Data stored with encryption; GDPR/local privacy law compliant

---

## 7. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| **Performance** | Page load < 2s; API response < 500ms (p95) |
| **Scalability** | Support 100 complexes, ~15,000 users in Year 1 |
| **Availability** | 99.5% uptime SLA |
| **Security** | HTTPS, JWT auth, RBAC, data encryption at rest and in transit |
| **Accessibility** | WCAG 2.1 AA; keyboard navigation; readable font sizes |
| **Internationalization** | Spanish (default) and English; date/number locale formatting |
| **Audit Trail** | All data changes logged with user, timestamp, before/after |
| **Compliance** | Local data privacy laws (Colombia Ley 1581); GDPR-ready |

---

## 8. Integrations

| Service | Purpose |
|---------|---------|
| **SendGrid / SES** | Transactional email |
| **Firebase / OneSignal** | Push notifications |
| **Azure Blob Storage** | Document and image storage |
| **Azure AI / OpenAI** | AI bot and document analysis |
| **Azure Face API** | Facial recognition |
| **Stripe / Wompi** | Online payment processing |
| **WhatsApp Business API** | Chatbot channel (Phase 4) |

---

## 9. MVP Definition

The **Minimum Viable Product** includes Phase 1 + Phase 2 features:

| Module | MVP? |
|--------|------|
| Complex/Tower/Apt CRUD | ✅ |
| Owner & Tenant Management | ✅ |
| Fee Calculation & PDF Receipt | ✅ |
| Payment Tracking & Reminders | ✅ |
| Auth (Login, Roles, Password Reset) | ✅ |
| Admin Dashboard | ✅ |
| Visitor Management | 🟡 Phase 3 |
| Vehicle Management | 🟡 Phase 3 |
| Common Areas | 🟡 Phase 3 |
| Notifications | 🟡 Phase 3 |
| AI Document Bot | 🟢 Phase 4 |
| AI Emergency Reports | 🟢 Phase 4 |
| Fines | 🟢 Phase 4 |
| Facial Recognition | 🟢 Phase 4 |

---

## 10. Success Metrics

| Metric | Year 1 Target |
|--------|--------------|
| Active complexes | 100 |
| Registered users | 15,000 |
| Monthly fee collection automation | 80% of complexes |
| NPS (Net Promoter Score) | ≥ 40 |
| Average page load time | < 2s |
| Support tickets / complex / month | < 5 |
