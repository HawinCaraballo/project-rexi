# TASK: Frontend — Phase 4 — Reports Screens

**Phase:** 4 (Advanced)  
**Module:** Reports  
**Priority:** 🟢 Advanced  
**Estimate:** 2 days  
**Depends on:** PHASE_2_fee_payment_screens  

---

## Objective

Build comprehensive admin reporting screens with filtering, visualization, and export for payments, fines, access logs, and incidents.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/reports` | Admin | Reports hub |
| `/reports/payments` | Admin | Payment & fee report |
| `/reports/fines` | Admin | Fines report |
| `/reports/access` | Admin | Access log report |
| `/reports/incidents` | Admin | Incident reports |
| `/reports/assembly` | Admin | Assembly & Financial Statement Base |

---

## Reports Hub Page (`/reports`)

**Layout:** Grid of report cards

```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ 💰 Payments      │  │ 🚫 Fines         │  │ 🚗 Access Log   │
│ Fees & payments  │  │ Issued, appeals  │  │ Entry/exit log  │
│ [Open Report →]  │  │ [Open Report →]  │  │ [Open Report →] │
└──────────────────┘  └──────────────────┘  └──────────────────┘
┌──────────────────┐  ┌──────────────────┐
│ ⚠️ Incidents     │  │ 📈 Assembly      │
│ Reports & issues │  │ Collection/Exp.  │
│ [Open Report →]  │  │ [Open Report →]  │
└──────────────────┘  └──────────────────┘
```

---

## Payments Report (`/reports/payments`)

**Filter bar:**
| Filter | Type |
|--------|------|
| Date Range | Date pickers (required) |
| Status | Multi-select: paid/overdue/partial/pending |
| Tower | Select |

**Summary cards:**
- Total Expected | Total Collected | Total Overdue | Collection Rate %

**Results Table:**
| Column | Notes |
|--------|-------|
| Period | Month/Year |
| Apartment | Tower + number |
| Owner | Full name |
| Base Fee | Currency |
| Parking | Currency |
| Storage | Currency |
| Fines | Currency |
| Total | Currency |
| Paid | Currency |
| Remaining | Currency |
| Status | StatusBadge |
| Due Date | Date |
| Days Overdue | Number (red if > 0) |
| Payment Date | Date (last payment) |
| Payment Method | Text |

**Exports:** [Excel] [PDF]

---

## Fines Report (`/reports/fines`)

**Filter bar:** Date range | Status | Tower | Guard (user)

**Summary:**
- Total fines issued | Total amount | By status breakdown (mini donut)

**Table columns:** Date | Apartment | Infraction | Amount | Guard | Status | Appeal | Charged in Fee

**Exports:** [Excel] [PDF]

---

## Access Log Report (`/reports/access`)

**Filter bar:** Date range | Entry type (entry/exit) | Method (qr/plate/facial/manual) | Tower

**Table columns:** Date/Time | Name | Type | Apartment | Method | Vehicle Plate | Guard | Status (authorized/denied)

**Exports:** [Excel] [PDF]

---

## Incidents Report (`/reports/incidents`)

**Filter bar:** Date range | Category | Priority | Status | Assigned To

**Table columns:** Date | Category | Priority badge | Description | Reported By | Apt | Status | Assigned To | Resolved Date

**Exports:** [Excel] [PDF]

---

## Shared Report Components

### DateRangePicker
```typescript
// components/forms/DateRangePicker.tsx
// Two date inputs: Start and End
// Validates: end >= start
// Quick presets: "This month", "Last month", "Last 3 months", "This year"
```

### ExportButton
```typescript
// components/reports/ExportButton.tsx
// Props: format ('excel'|'pdf'), endpoint, filters
// Shows loading spinner during download
// Triggers file download from blob response
```

### SummaryCards
```typescript
// Reusable 4-card row used by all report pages
// Props: Array of { label, value, format ('currency'|'number'|'percent'), color }
```

---

## Acceptance Criteria

- [ ] Reports hub shows all 4 report types
- [ ] All report pages have working date range filter
- [ ] Summary cards show correct totals for filtered data
- [ ] Tables paginate correctly with all columns
- [ ] Excel export downloads valid .xlsx file
- [ ] PDF export downloads valid .pdf file
- [ ] Quick date presets work correctly
- [ ] All filter combinations work

---

## Component IDs

```
reports-hub-grid
report-payments-link
report-fines-link
report-access-link
report-incidents-link
payment-report-date-start
payment-report-date-end
payment-report-status-filter
payment-report-table
payment-report-summary
payment-report-export-excel
payment-report-export-pdf
fines-report-table
fines-report-export-excel
access-report-table
incidents-report-table
date-range-picker
date-range-preset-this-month
date-range-preset-last-month
```
