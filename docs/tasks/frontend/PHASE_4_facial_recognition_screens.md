# TASK: Frontend — Phase 4 — Facial Recognition Screens

**Phase:** 4 (Advanced)  
**Module:** Security / AI  
**Priority:** 🟢 Advanced  
**Estimate:** 2 days  
**Depends on:** PHASE_1_design_system  

---

## Objective

Build the resident facial enrollment interface and guard station recognition display.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/settings/facial-recognition` | Owner, Tenant | Enroll / manage face |
| `/guard/camera` | Guard | Guard station camera view |

---

## Facial Enrollment Page (`/settings/facial-recognition`)

### State: Not Enrolled

```
┌──────────────────────────────────────────────────────────────┐
│  Facial Recognition                            [Optional]    │
│  ─────────────────                                           │
│  [Face icon illustration]                                    │
│                                                             │
│  Enable contactless entry to your complex.                  │
│  Your face is enrolled once and recognized automatically    │
│  at the entrance.                                           │
│                                                             │
│  🔒 Privacy: Your biometric data is stored securely in     │
│     Azure and can be deleted at any time.                   │
│                                                             │
│  📋 Read Privacy Notice                                     │
│                                                             │
│  [ ] I have read and accept the Privacy Notice              │
│                                                             │
│  [Start Enrollment] (disabled until checkbox checked)       │
└──────────────────────────────────────────────────────────────┘
```

### Enrollment Flow (Step-by-step)

**Step 1: Camera Access**
- Request camera permission
- If denied → show instructions to enable camera in browser settings

**Step 2: Photo Capture (3 photos)**

```
┌──────────────────────────────────────────────────────────────┐
│  Step 2 of 3 ━━━━━━━━━━━━━━━━━━━━━━░░░░░░░░░░░░░░         │
│                                                             │
│  Photo 2: Turn slightly to the right                        │
│                                                             │
│  ┌────────────────────────────────────┐                    │
│  │                                    │                    │
│  │     [Live camera preview]          │                    │
│  │     [Face outline guide]           │                    │
│  │                                    │                    │
│  └────────────────────────────────────┘                    │
│                                                             │
│  ✅ Photo 1 taken    ⏳ Taking photo 2...   ⬜ Photo 3     │
│                                                             │
│  [📷 Take Photo]                                            │
└──────────────────────────────────────────────────────────────┘
```

Instructions per photo:
1. "Look straight at the camera"
2. "Turn slightly to the left"
3. "Turn slightly to the right"

**Face guide overlay:** SVG oval face guide on camera preview

**Step 3: Processing**
```
Uploading photos...  [████████░░] 80%
Registering with biometric system...
✅ Enrollment complete!
```

### State: Enrolled

```
┌──────────────────────────────────────────────────────────────┐
│  Facial Recognition                         ✅ Enrolled      │
│  Enrolled on: April 22, 2026                                │
│                                                             │
│  [Face icon with checkmark]                                 │
│                                                             │
│  Your face is enrolled. You can now enter the complex       │
│  automatically at the security cameras.                     │
│                                                             │
│  [Remove Enrollment] (red, outlined button)                 │
└──────────────────────────────────────────────────────────────┘
```

**Remove Enrollment dialog:**
"Are you sure? Your biometric data will be permanently deleted and you will no longer have contactless entry."
[Cancel] [Remove] (red button)

---

## Guard Station Camera View (`/guard/camera`)

**Full-screen design, optimized for mounted tablets/monitors:**

```
┌──────────────────────────────────────────────────────────────┐
│  REXI SECURITY                          🕐 09:45 AM          │
│                                         [Manual Search]      │
├──────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                                                        │  │
│  │           [Live camera feed]                           │  │
│  │                                                        │  │
│  │     Scanning for residents...                          │  │
│  │                                                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
├──────────────────────────────────────────────────────────────┤
│  Ready to scan              📷 Camera active                │
└──────────────────────────────────────────────────────────────┘
```

**On Recognition (Authorized):**
```
┌──────────────────────────────────────────────────────────────┐
│  ✅ AUTHORIZED                                               │
│  ─────────────────────                                       │
│  [Large photo]  Carlos García                               │
│                 Apartment: 101-A | Tower A                   │
│                 Owner                                        │
│                 Confidence: 97%                              │
│                                                             │
│  [Log Entry]              [Log Exit]                        │
│                                                             │
│  ← Back to scanning (auto in 5s)                            │
└──────────────────────────────────────────────────────────────┘
```

**On Not Recognized:**
```
┌──────────────────────────────────────────────────────────────┐
│  ❌ NOT RECOGNIZED                                           │
│                                                             │
│  Face not found in the system.                              │
│  Proceed with manual verification.                          │
│                                                             │
│  [Manual Search]     ← Back to scanning                    │
└──────────────────────────────────────────────────────────────┘
```

**On Low Confidence (< 75%):**
```
⚠️ LOW CONFIDENCE (68%)
Manual verification required.
[Photo]  Possible match: Carlos García  Apt 101-A
[Confirm Entry]  [Deny]  [Search Manually]
```

---

## Camera Integration

```typescript
// Uses react-webcam for camera access
// For recognition: captures frame every 2 seconds, sends to POST /face-recognition/identify
// Shows loading indicator between frames
// Auto-resets to scanning mode 5 seconds after showing result
```

---

## Acceptance Criteria

- [ ] Privacy notice checkbox required before enrollment starts
- [ ] Camera permission requested and handled correctly
- [ ] 3 photos captured with correct instructions per step
- [ ] Upload progress shown
- [ ] Success state shows enrollment date
- [ ] Remove enrollment shows confirmation and deletes data
- [ ] Guard camera view shows live feed
- [ ] Recognized resident shown with green AUTHORIZED overlay
- [ ] Unknown face shows red NOT RECOGNIZED overlay
- [ ] Low confidence shows yellow warning with manual option
- [ ] Auto-reset after 5 seconds

---

## Component IDs

```
facial-enrollment-consent-checkbox
facial-enrollment-privacy-link
facial-enrollment-start-btn
facial-enrollment-camera-view
facial-enrollment-capture-btn
facial-enrollment-photo-1
facial-enrollment-photo-2
facial-enrollment-photo-3
facial-enrollment-progress
facial-enrollment-success
facial-enrollment-remove-btn
guard-camera-feed
guard-camera-result-card
guard-camera-result-status
guard-camera-log-entry-btn
guard-camera-log-exit-btn
guard-camera-manual-search-btn
```
