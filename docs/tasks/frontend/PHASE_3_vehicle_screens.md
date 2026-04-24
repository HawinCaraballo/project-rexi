# TASK: Frontend — Phase 3 — Vehicle Screens

**Phase:** 3 (Important)  
**Module:** Security  
**Priority:** 🟡 Important  
**Estimate:** 1 day  
**Depends on:** PHASE_3_visitor_screens  

---

## Objective

Build vehicle registration and plate lookup screens for residents and guards.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/my/vehicles` | Owner, Tenant | Manage own vehicles |
| `/vehicles` | Admin | All vehicles list |

Guard vehicle search is part of `/guard/search` (see visitor screens task).

---

## My Vehicles Page (`/my/vehicles`)

**Layout:** Cards grid (simpler than a table)

**Vehicle card:**
```
🚗  ABC 123
Toyota Corolla | Blue | 2020
[Edit]  [Remove]
```

**"Add Vehicle" button** → opens modal/slide-over:

| Field | Validation |
|-------|-----------|
| License Plate | Required, format: AAA000 or AAA00A |
| Make | Optional, max 50 |
| Model | Optional, max 50 |
| Color | Optional, color picker or text |
| Year | Optional, 1900-present+1 |

**Plate validation:** Real-time regex check with format hint:
"Formato: ABC123 (3 letras + 3 números) o ABC12D (3 letras + 2 números + 1 letra)"

**Plate uniqueness:** Async check on blur → shows "Plate already registered" if duplicate.

**Max vehicles warning:** If apartment limit reached → "Maximum vehicles per apartment reached (3). Contact admin."

---

## Admin: All Vehicles Page (`/vehicles`)

**Columns:** Plate | Make/Model | Color | Year | Type | Owner/Tenant | Apartment | Status | Actions  
**Filters:** Type (resident/visitor), Tower, Search by plate  
**Actions:** View, Edit (admin only)

---

## Acceptance Criteria

- [ ] My vehicles shows all user's vehicles
- [ ] Add vehicle validates plate format in real-time
- [ ] Duplicate plate shows inline error
- [ ] Max vehicle limit shows warning
- [ ] Remove vehicle shows confirmation dialog
- [ ] Admin vehicle list filters by type and plate

---

## Component IDs

```
my-vehicles-list
vehicle-add-btn
vehicle-form
vehicle-form-plate
vehicle-form-plate-hint
vehicle-form-submit
vehicle-card
vehicle-remove-btn
admin-vehicles-table
admin-vehicles-plate-filter
```
