# TASK: Frontend — Phase 3 — Common Areas Screens

**Phase:** 3 (Important)  
**Module:** Facilities  
**Priority:** 🟡 Important  
**Estimate:** 2 days  
**Depends on:** PHASE_1_design_system  

---

## Objective

Build common area listing, booking calendar, and booking management screens.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/common-areas` | All | Browse common areas |
| `/common-areas/[id]` | All | Area detail + booking calendar |
| `/my/bookings` | Owner, Tenant | My bookings |
| `/common-areas/manage` | Admin | Admin area management |
| `/common-areas/new` | Admin | Create area |

---

## Common Areas Browse Page (`/common-areas`)

**Layout:** Card grid (3 columns desktop, 1 column mobile)

**Area Card:**
```
[Photo]
─────────────────────
Gym                  🟢 Free
Open: 6:00 - 22:00  👥 Cap: 30
"Book Now" button
```

**Filter bar:** Free / Paid | Available Now | Capacity

---

## Area Detail + Booking Page (`/common-areas/[id]`)

**Header:** Large hero photo, name, description, capacity, open hours, fee (or "Free")

**Tabs:**

### Tab 1: Book
- **Date picker** (calendar-style, current month + next)
- Available days highlighted, full days grayed
- **Time slot grid** for selected day:
  ```
  06:00 - 07:00  [Available]
  07:00 - 08:00  [Available]
  08:00 - 10:00  [Booked]
  10:00 - 12:00  [Available]
  ```
- Click available slot → opens booking form modal:
  | Field | Validation |
  |-------|-----------|
  | Start Time | Pre-filled |
  | End Time | Pre-filled or select |
  | Notes | Optional |
- Booking confirmation dialog with fee amount (if any)
- Requires approval badge if area needs admin approval

**Debt block:** If user has overdue fees and area blocks on debt → show warning banner and disable booking button.

### Tab 2: Rules
- Booking rules text (from area description)

---

## My Bookings Page (`/my/bookings`)

**Timeline view + table view toggle:**

**Table columns:** Area | Date | Time | Status | Fee | Actions (Cancel)

**Status badges:**
- `confirmed` → Green
- `pending` → Yellow (awaiting admin approval)
- `cancelled` → Gray

**Cancel button:** Only shows if booking start is > 2h away. Opens confirmation dialog.

---

## Admin: Manage Common Areas (`/common-areas/manage`)

**Admin-only view with full booking management:**

**Pending Approvals section (if areas require approval):**
- List: Area | Resident | Date | Time | Notes | [Approve] [Reject]

**All Areas list:**
- Each area: name, fee, active bookings count today, [Edit] [View Bookings] [Deactivate]

---

## Create/Edit Common Area Form

| Field | Type | Validation |
|-------|------|-----------|
| Name | Text | Required, max 100 |
| Description | Textarea | Optional |
| Capacity | Number | Optional, > 0 |
| Fee | Currency | Required, >= 0 |
| Open Time | Time picker | Required |
| Close Time | Time picker | Required, > open time |
| Min Advance Booking | Number (hours) | Required, >= 1 |
| Requires Approval | Toggle | Default: off |
| Block on Outstanding Debt | Toggle | Default: on |
| Photos | FileUpload (multiple) | Optional |

---

## Acceptance Criteria

- [ ] Area cards show correct info including fee and capacity
- [ ] Date picker highlights available/unavailable days
- [ ] Time slot grid shows occupied and available slots
- [ ] Booking form pre-fills selected time slot
- [ ] Conflict error shown if selected slot becomes occupied mid-flow
- [ ] Debt warning banner shows when user has overdue fees
- [ ] Pending approval state shown to resident after booking
- [ ] Admin approve/reject updates booking status and notifies resident
- [ ] Cancel button only shows when cancellation is still allowed
- [ ] Area creation form validates all fields

---

## Component IDs

```
common-areas-grid
area-card
area-detail-tabs
area-date-picker
area-timeslot-grid
area-booking-form
area-booking-notes
area-booking-submit
area-debt-warning
my-bookings-table
my-bookings-cancel-btn
manage-pending-list
manage-approve-btn
manage-reject-btn
area-form
area-form-name
area-form-fee
area-form-submit
```
