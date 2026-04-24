# TASK: Frontend — Phase 2 — Owner & Tenant Screens

**Phase:** 2 (Core MVP)  
**Module:** People  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  
**Depends on:** PHASE_2_complex_management  

---

## Objective

Build screens for managing owners and tenants: list, search, detail, create/edit forms, and family member management.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/owners` | Admin | Owner list |
| `/owners/new` | Admin | Create owner |
| `/owners/[id]` | Admin, Owner(self) | Owner detail |
| `/owners/[id]/edit` | Admin | Edit owner |
| `/owners/[id]/tenants/new` | Admin, Owner | Create tenant |
| `/tenants/[id]` | Admin, Owner(linked), Tenant(self) | Tenant detail |
| `/search` | Admin, Guard | Universal search |

---

## Owner List Page (`/owners`)

**Columns:** Name | ID | Phone | Apartments | Status | Actions  
**Filters:** Search text (name/ID/email), Tower, Status (active/inactive)  
**Row actions:** View, Edit  

---

## Owner Detail Page (`/owners/[id]`)

**Layout:** 3-column header (avatar + name + apartment badges) + tabs

### Tab 1: Personal Info
- Full name, ID type + number, email, phone
- Profile photo (editable inline)
- Registration date
- "Edit" button

### Tab 2: Apartments
- List of assigned apartments with tower, floor, monthly fee
- "Assign Apartment" button (admin only)
- "Remove from apartment" (admin only, with confirmation)

### Tab 3: Family Members
- Table: Name | Relationship | Age
- "Add Member" button → inline form (name, relationship, birth date)
- Delete icon per row

### Tab 4: Tenant(s)
- For each apartment owned: show active tenant (if any)
- "Register Tenant" button → `/owners/[id]/tenants/new`

---

## Create Owner Form (`/owners/new`)

**Section 1: Personal Information**
| Field | Type | Validation |
|-------|------|-----------|
| First Name | Text | Required, 2-100 chars |
| Last Name | Text | Required, 2-100 chars |
| ID Type | Select | CC / CE / NIT / Passport |
| ID Number | Text | Required, 5-20 chars |
| Email | Email | Required, unique (async validation) |
| Phone | Tel | Optional |
| Profile Photo | FileUpload | Optional, image |

**Section 2: Apartment Assignment**
- Multi-select autocomplete: search and select apartments from the complex
- Shows: `[Tower A] - Apt 101 (Floor 1)`
- Minimum 1 apartment required

**Section 3: Family Members (optional)**
- Dynamic list: each row has First Name, Last Name, Relationship, Birth Date
- "Add another family member" button
- Remove button per row

**Footer:** Cancel | Save buttons

**Async email validation:**
```typescript
// On email blur, call GET /api/v1/users/check-email?email=X
// Show inline "Email already registered" if taken
```

---

## Create Tenant Form (`/owners/[id]/tenants/new`)

Similar to owner form with additions:
- **Apartment:** Pre-selected from owner's apartment (if only 1); otherwise select from owner's apartments
- **Lease Start / End:** Date pickers (optional)
- **Note:** "Tenant will receive an email to set their password."

---

## Universal Search Page (`/search`)

**For admin and guard:**

Large search bar with category tabs:
- **All** | **Owners** | **Tenants** | **Apartments** | **Vehicles**

Real-time results as user types (debounced 300ms):

**Owner result card:**
```
[Avatar] Carlos García    [Owner badge]
ID: CC 12345678  |  📞 310-555-1234
Apartments: 101-A, 204-B
[View Profile →]
```

**Tenant result card:**
Similar with "Tenant" badge + which owner registered them.

**Apartment result card:**
```
[Building icon] Apartment 101-A | Tower A | Floor 1
Owner: Carlos García (CC 12345678)
Tenant: (none)
Fee: $470,000/month
[View →]
```

---

## Profile Photo Upload

```typescript
// FileUpload component configured for images:
// - Accept: image/jpeg, image/png, image/webp
// - Max size: 2MB
// - Shows circular preview on upload
// - Calls POST /api/v1/media/upload → gets blob URL
// - Stored in form state as URL
```

---

## Acceptance Criteria

- [ ] Owner list paginates and filters by text/tower/status
- [ ] Async email validation shows error before form submission
- [ ] Creating owner sends welcome email (verified via success toast)
- [ ] Owner detail shows all tabs with correct data
- [ ] Family members can be added and removed
- [ ] Tenant can be created linked to owner's apartment
- [ ] Tenant creation shows apartment dropdown with only owner's apartments
- [ ] Universal search returns results across all types
- [ ] Guard-role users see search but not create/edit buttons

---

## Component IDs

```
owner-list-table
owner-search-input
owner-new-btn
owner-form
owner-form-email
owner-form-apartment-select
owner-form-add-family-btn
owner-detail-tabs
owner-tab-apartments
owner-tab-family
tenant-form
search-input
search-results-list
```
