# TASK: Frontend — Phase 2 — Fee & Payment Screens

**Phase:** 2 (Core MVP)  
**Module:** Finance  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  
**Depends on:** PHASE_2_admin_dashboard  

---

## Objective

Build all fee and payment management screens for admin, owner, and tenant roles.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/fees` | Admin | All fees (paginated) |
| `/fees/generate` | Admin | Generate fees for a period |
| `/fees/[id]` | Admin, Owner, Tenant | Fee detail |
| `/my/fees` | Owner, Tenant | My fees list |
| `/payments/report` | Admin | Payment report |

---

## Admin: Fees List Page (`/fees`)

**Columns:**
| Column | Notes |
|--------|-------|
| Apartment | Link to apartment detail |
| Tower | Text |
| Owner | Text |
| Period | "April 2026" formatted |
| Total | Currency |
| Paid | Currency |
| Remaining | Currency |
| Status | StatusBadge: pending/paid/overdue/partial |
| Due Date | Date |
| Days Overdue | Number (red if > 0) |
| Actions | View, Record Payment |

**Filters:**
- Period: Month + Year selectors
- Status: multi-select chips
- Tower: select dropdown
- Search: apartment number or owner name

**Bulk action:** Select multiple → "Send Reminder" → sends payment reminder notifications

**Top actions:**
- "Generate Fees" button → opens dialog:
  - Select: Year + Month
  - Preview: "Will generate fees for 60 active apartments"
  - Confirm → loading state → success message with count

---

## Fee Detail Page (`/fees/[id]`)

**Header:** Apartment number, Owner name, Period (e.g., "April 2026"), Status badge

**Section 1: Fee Breakdown**
```
Administration Base:     $ 350,000
Parking (Torre A-P01):  $  52,000
Storage (D-05):         $  18,000
Extraordinary Fee:      $ 100,000 (Roof Repair)
Fines charged:          $  50,000
─────────────────────────────────
TOTAL:                  $ 570,000

Due Date: April 10, 2026
Days Overdue: 5 ⚠️
Late Interest (Configurable Rate): $35,250
```

**Section 2: Payments Made**
Table: Date | Amount | Method | Reference | Recorded By

"Record Payment" button → opens slide-over:
| Field | Type | Validation |
|-------|------|-----------|
| Amount | Currency input | Required, > 0 |
| Payment Date | Date picker | Required, not future |
| Payment Method | Select | cash/transfer/card/online |
| Reference # | Text | Optional |
| Notes | Textarea | Optional |

**Section 3: PDF Receipt**
- "Download PDF" button
- "Re-send Email" button (admin only)

---

## Owner/Tenant: My Fees Page (`/my/fees`)

Simplified view for residents:

**Summary bar:** "You have N overdue fee(s)" (if any) — orange banner

**Fees table:**
| Column | Notes |
|--------|-------|
| Period | "April 2026" |
| Apartment | If owner with multiple |
| Total | Currency |
| Paid | Currency |
| Status | StatusBadge |
| Due Date | Date |
| Actions | View, Download PDF |

**Click "View"** → Fee detail (read-only for residents, no record payment button)

---

## Payment Report Page (`/payments/report`)

**Filters (sticky top bar):**
- Date Range: Start date + End date (required)
- Status: All / Paid / Overdue / Partial / Pending
- Tower: All / Tower A / Tower B

**Summary cards row:**
- Total Expected | Total Collected | Collection Rate | Total Overdue

**Results Table:**
Same columns as fees list, with date range applied

**Export buttons:** `[Export Excel]` `[Export PDF]`

---

## Currency Input Component

```typescript
// components/forms/CurrencyInput.tsx
// Displays formatted: "$ 12,000,000"
// Stores as number in form state
// Locale-aware formatting (es-CO: . as thousands separator, , as decimal)
// Keyboard: only allows digits
```

---

## Status Color Mapping

| Status | Color | Icon |
|--------|-------|------|
| `pending` | Gray | Clock |
| `paid` | Green | Check circle |
| `overdue` | Red | Alert circle |
| `partial` | Yellow | Minus circle |

---

## Acceptance Criteria

- [ ] Fees list shows correct data with all columns
- [ ] Period filter (month + year) works correctly
- [ ] Fee generation dialog shows preview count and confirms
- [ ] Fee generation shows error if already generated for period
- [ ] Record payment slide-over validates and submits correctly
- [ ] Fee status updates after payment recorded (optimistic update)
- [ ] PDF download opens/saves correctly
- [ ] Re-send email shows success toast
- [ ] Payment report filters work and show correct totals
- [ ] Excel and PDF export download correctly
- [ ] My Fees page shows only the current user's fees

---

## Component IDs

```
fees-list-table
fees-period-month-select
fees-period-year-select
fees-status-filter
fees-generate-btn
fees-generate-dialog
fees-generate-confirm-btn
fee-breakdown-card
fee-payments-table
fee-record-payment-btn
fee-payment-form
fee-payment-amount
fee-payment-date
fee-payment-method
fee-payment-submit
fee-download-pdf-btn
fee-resend-email-btn
report-date-start
report-date-end
report-export-excel
report-export-pdf
```
