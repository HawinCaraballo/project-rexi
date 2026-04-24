# TASK: Frontend — Phase 2 — Complex Management Screens

**Phase:** 2 (Core MVP)  
**Module:** Residential  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  
**Depends on:** PHASE_1_design_system  

---

## Objective

Build screens for managing complexes, towers, and apartments including list views, detail pages, forms, and bulk upload.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/complexes` | SuperAdmin | List all complexes |
| `/complexes/new` | SuperAdmin | Create complex |
| `/complexes/[id]` | Admin | Complex detail (tabs) |
| `/complexes/[id]/edit` | Admin | Edit complex |
| `/complexes/[id]/towers/new` | Admin | Create tower |
| `/complexes/[id]/apartments` | Admin | Apartment list |
| `/apartments/[id]/edit` | Admin | Edit apartment |

---

## Complex List Page (`/complexes`)

**Layout:** PageLayout + DataTable

**Columns:**
| Column | Type |
|--------|------|
| Name | Text + link |
| City | Text |
| Total Apartments | Number |
| Annual Budget | Currency |
| Status | StatusBadge (active/inactive) |
| Actions | Edit, View |

**Filters:** Search by name, city; Status toggle

**Top actions:** "New Complex" button → `/complexes/new`

---

## Complex Detail Page (`/complexes/[id]`)

**Layout:** PageLayout with 4 tabs:

### Tab 1: Overview
- Info card: name, address, NIT, phone, email, logo
- Budget card: annual budget, monthly budget (÷12), payment deadline day, late interest rate
- Stats: total towers, total apartments, occupied %, coefficient sum indicator

**Coefficient Sum Indicator:**
```
Coefficients: 1.0000 ✅ Complete
Coefficients: 0.9750 ⚠️ Missing 0.025 — fees cannot be generated
```

### Tab 2: Towers
- List of towers with: name, floors, apartment count
- "Add Tower" button
- Click tower → expand to show apartments in that tower

### Tab 3: Apartments
- Full DataTable of apartments
- Filters: Tower (select), Floor (number), Parking Type, Status
- Columns: Number | Tower | Floor | Coefficient | Parking | Storage | Owner | Status

**Bulk Upload button:** Opens drawer with steps:
1. Download CSV template
2. Fill and upload file
3. Preview table (valid rows + error rows highlighted)
4. Confirm import

### Tab 4: Settings
- Edit complex information form
- Danger zone: deactivate complex (with confirmation)

---

## Create / Edit Complex Form

**Fields:**
| Field | Type | Validation |
|-------|------|-----------|
| Complex Name | Text input | Required, max 200 |
| NIT | Text input | Optional |
| Address | Textarea | Required |
| City | Text input | Required |
| Phone | Tel input | Optional |
| Email | Email input | Optional |
| Logo | FileUpload | Optional, image |
| Annual Budget | Currency input | Required, > 0 |
| Late Interest Rate | Number input (%) | Required, 0-100 |
| Payment Deadline Day | Number input | Required, 1-28 |

**Currency input:** displays formatted (e.g., `$12,000,000`), stores as number

---

## Create Tower Modal

**Fields:**
| Field | Validation |
|-------|-----------|
| Tower Name | Required, max 100 |
| Number of Floors | Required, integer > 0 |

Opens as a Dialog (not full page).

---

## Create / Edit Apartment Form

**Fields:**
| Field | Type | Validation |
|-------|------|-----------|
| Apartment Number | Text | Required, unique in tower |
| Floor | Number | Required, > 0 |
| Area (m²) | Decimal | Optional, > 0 |
| Coefficient | Decimal (precision 8) | Required, 0 < x < 1 |
| Parking Type | Select | assigned / communal / none |
| Parking Coefficient | Decimal | Required if parking is assigned, else 0 |
| Storage Coefficient | Decimal | Required if storage unit, else 0 |

---

## Bulk Upload Flow

**Template CSV columns:**
```
number,floor,area_sqm,coefficient,parking_type,parking_coefficient,storage_coefficient
101,1,85.5,0.015,assigned,0.003,0.001
102,1,72.0,0.012,communal,0,0
```

**Upload UI:**
1. Drag-drop or click to upload (accept .csv, .xlsx)
2. Parse and show preview table:
   - Valid rows: green row indicator
   - Invalid rows: red row indicator + error tooltip per cell
3. Summary: "45 valid, 3 invalid — fix errors or import valid only"
4. "Import 45 rows" button → calls API → shows success/error summary

---

## Acceptance Criteria

- [ ] Complex list paginates and filters correctly
- [ ] Create complex form validates all fields
- [ ] Complex detail tabs navigate correctly
- [ ] Coefficient sum indicator is accurate
- [ ] Tower list shows apartment count
- [ ] Apartment list filters by tower and parking type
- [ ] Create apartment validates uniqueness (real-time via API)
- [ ] Bulk upload parses CSV/XLSX and shows preview with errors
- [ ] Bulk upload confirms and imports correctly
- [ ] Logo upload shows preview before save

---

## Component IDs

```
complex-list-table
complex-new-btn
complex-form
complex-form-name
complex-form-budget
complex-form-submit
tower-create-dialog
apartment-list-table
apartment-filter-tower
apartment-bulk-upload-btn
apartment-upload-preview-table
apartment-upload-confirm-btn
```
