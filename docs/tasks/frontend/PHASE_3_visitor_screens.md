# TASK: Frontend — Phase 3 — Visitor Management Screens

**Phase:** 3 (Important)  
**Module:** Security  
**Priority:** 🟡 Important  
**Estimate:** 2 days  
**Depends on:** PHASE_2_owner_tenant_screens  

---

## Objective

Build visitor registration, QR code display, and guard lookup screens.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/visitors` | Admin | All visitors list |
| `/visitors/new` | Owner, Tenant, Admin | Register visitor |
| `/visitors/[id]` | Owner, Admin | Visitor detail + QR |
| `/guard/search` | Guard | Guard lookup dashboard |
| `/guard/scan` | Guard | QR scanner screen |

---

## My Visitors Page (`/my/visitors` or `/visitors` for admin)

**Columns:** Name | ID | Type | Expires | Vehicle | Status | QR | Actions  
**Filters:** Type (temporary/permanent), Status (active/expired/deactivated), Search name/ID  
**Top action:** "Register Visitor" button

---

## Register Visitor Form (`/visitors/new`)

**Section 1: Visitor Information**
| Field | Type | Validation |
|-------|------|-----------|
| First Name | Text | Required |
| Last Name | Text | Required |
| ID Type | Select | CC / CE / Passport (optional) |
| ID Number | Text | Optional |
| Photo | FileUpload | Optional |
| Visitor Type | Radio | Temporary / Permanent |
| Expiry Date | DateTimePicker | Required if Temporary, must be future |
| Apartment | Select | (for admin: select from list; for owner: pre-filled) |

**Section 2: Vehicle (optional)**
- Toggle: "Does visitor have a vehicle?" [Yes/No]
- If Yes:
  | Field | Validation |
  |-------|-----------|
  | License Plate | Required, validated format |
  | Make | Optional |
  | Model | Optional |
  | Color | Optional |

**Footer:** Cancel | Save & Get QR

**After save → redirect to visitor detail (QR shown immediately)**

---

## Visitor Detail Page (`/visitors/[id]`)

**Layout:** 2-column

**Left column:**
- Visitor photo (if uploaded) or avatar placeholder
- Name, ID, type badge, status badge
- Expiry date (with countdown: "Expires in 3 days")
- Apartment being visited
- Registration date

**Right column:**
- QR Code image (large, downloadable)
- QR code string (copyable text)
- Buttons:
  - "Download QR" → saves PNG
  - "Share via WhatsApp" → generates WhatsApp link with QR URL
  - "Deactivate Visitor" → with confirmation (admin/owner only)

**Vehicle card (if linked):**
- Plate, make, model, color chips

---

## Guard Search Dashboard (`/guard/search`)

**Design:** Large, simple, mobile-optimized (guards use tablets/phones)

```
┌──────────────────────────────────────────────────┐
│            REXI SECURITY                    [🔔]  │
├──────────────────────────────────────────────────┤
│                                                   │
│  🔍 Search by plate, QR, or ID number            │
│  ┌─────────────────────────────────────┐          │
│  │  ABC123  or  scan QR  or  12345678 │  [Search]│
│  └─────────────────────────────────────┘          │
│  [📷 Scan QR]  [🚗 Plate]  [👤 ID Number]       │
│                                                   │
│  Today's Access Log                               │
│  ─────────────────                                │
│  ✅ Carlos García    Apt 101-A   Entry  09:45     │
│  ✅ Visitor: Juan P. Apt 203-B   Entry  09:30     │
│  ❌ Unknown Plate    ABC999      Denied  09:20    │
└──────────────────────────────────────────────────┘
```

**Search Result Card (Authorized):**
```
┌──────────────────────────────────────────────────┐
│ [Photo]  Carlos García                    ✅      │
│          CC: 12345678                  AUTHORIZED │
│          Apartment: 101-A | Tower A               │
│          Owner                                    │
│          Vehicle: ABC123 (Toyota, Blue)           │
│  [Log Entry]  [Log Exit]  [View Profile]         │
└──────────────────────────────────────────────────┘
```

**Search Result Card (Not Authorized):**
```
┌──────────────────────────────────────────────────┐
│ [?]  Vehicle: XYZ999                      ❌      │
│      Not registered in this complex    DENIED     │
│                                                   │
│  [Report Incident]                               │
└──────────────────────────────────────────────────┘
```

---

## QR Scanner Screen (`/guard/scan`)

```typescript
// Uses react-webcam or native camera API
// Continuously scans for QR codes (jsQR library)
// On detect → automatically searches and shows result overlay
// Result overlay: full-screen card with AUTHORIZED (green) or DENIED (red)
// [Close] button to scan next visitor
```

---

## Package Logging (Guard)

Accessible from guard dashboard as secondary action:

**"Log Package" button → Modal:**
| Field | Validation |
|-------|-----------|
| Apartment | Required, autocomplete |
| Sender | Optional |
| Description | Optional |
| Photo | Optional, camera or upload |

**Submit → sends package arrival notification to resident**

---

## Acceptance Criteria

- [ ] Visitor form validates all fields including expiry date
- [ ] After save, QR code image is shown immediately
- [ ] QR download saves correct PNG
- [ ] Guard search works for plate, QR text, and ID number
- [ ] Authorized result shown in green, denied in red (clear visual)
- [ ] QR scanner opens camera and auto-detects QR codes
- [ ] Access log entry shown in today's log after authorization
- [ ] Package logging triggers resident notification
- [ ] Expired visitors shown with expired badge (not authorized)
- [ ] Mobile-optimized guard screen (large tap targets, high contrast)

---

## Component IDs

```
visitor-list-table
visitor-new-btn
visitor-form
visitor-form-type-radio
visitor-form-expiry
visitor-form-vehicle-toggle
visitor-qr-image
visitor-qr-download-btn
visitor-qr-share-btn
guard-search-input
guard-scan-btn
guard-result-card
guard-result-status
guard-log-entry-btn
guard-package-btn
guard-package-form
```
