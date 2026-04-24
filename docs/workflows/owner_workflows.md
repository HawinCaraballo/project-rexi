# Rexi — Owner User Workflows

---

## WF-OWNER-01: First Login & Password Setup

```
[Email received: "Welcome to Rexi"]
    │
    └── Click "Set password" link → Set New Password screen
            │
            ├── New password* (min 8 chars, uppercase, number)
            ├── Confirm password*
            └── [Save] → Redirect to Owner Dashboard
```

---

## WF-OWNER-02: View My Fees & Payments

```
[Owner Dashboard]
    │
    └── My Fees tab
            │
            ├── List: Month | Total | Status | Due Date | Actions
            │
            └── Click fee row → Fee Detail
                    │
                    ├── Breakdown: base + parking + storage + fines
                    ├── Payment history for this period
                    └── [Download PDF Receipt]
```

---

## WF-OWNER-03: Register Tenant

```
[My Apartments] → Select apartment → Add Tenant
    │
    ├── Form:
    │   ├── First name*, Last name*
    │   ├── ID type*, ID number*
    │   ├── Email* (becomes login)
    │   ├── Phone
    │   ├── Profile photo
    │   ├── Lease start date, Lease end date
    │   └── Family members (optional, repeatable)
    │
    └── [Save]
            │
            ├── Tenant user account created
            ├── Tenant receives email with credentials
            └── Apartment status shows "Rented"
```

---

## WF-OWNER-04: Register Visitor

```
[Visitors] → Add Visitor
    │
    ├── First name*, Last name*
    ├── ID type, ID number
    ├── Photo (optional)
    ├── Visitor type:
    │   ├── Temporary → select expiry date*
    │   └── Permanent
    ├── Vehicle? [Yes/No]
    │   └── Yes → Plate*, Make, Model, Color
    │
    └── [Save] → QR code generated
            │
            └── QR code screen:
                    ├── [Download QR]
                    ├── [Share via WhatsApp]
                    └── [Copy visitor link]
```

---

## WF-OWNER-05: Book Common Area

```
[Common Areas]
    │
    ├── List of available areas (name, capacity, fee, hours)
    │
    └── Select area → View availability calendar
            │
            └── Click available slot
                    │
                    ├── Confirm time slot
                    ├── Notes (optional)
                    └── [Book]
                            │
                            ├── [Blocked - outstanding debt] → "Pay pending fee first"
                            ├── [Requires approval] → Status "pending" → Admin notified
                            └── [Confirmed] → Booking confirmation + email notification
```

---

## WF-OWNER-06: Cancel Booking

```
[My Bookings]
    │
    └── Select upcoming booking → [Cancel]
            │
            ├── Confirm cancellation modal
            └── [Yes, cancel] → Booking cancelled → Email confirmation
```

---

## WF-OWNER-07: File Fine Appeal

```
[Notifications] or [My Fines]
    │
    └── Select approved fine
            │
            ├── View: infraction, amount, evidence
            │
            └── [File Appeal] (only available within appeal window)
                    │
                    ├── Appeal reason* (text area)
                    ├── Upload supporting documents (optional)
                    └── [Submit Appeal]
                            │
                            ├── Fine status → "appealed"
                            └── Admin notified
```

---

## WF-OWNER-08: Use AI Document Bot

```
[Help / Documents]
    │
    └── Chat with Rexi Bot
            │
            ├── Type question (any language)
            ├── Bot responds with answer sourced from complex documents
            ├── Bot shows source document reference
            │
            └── [Not helpful?] → Escalate to admin
                    └── Admin receives message in notification center
```

---

## WF-OWNER-09: Report Issue

```
[Report Issue]
    │
    ├── Category*: maintenance / security / noise / other
    ├── Description*
    ├── Photos (optional)
    └── [Submit]
            │
            ├── AI assigns priority (Critical/High/Medium/Low)
            ├── Admin notified with category and priority
            └── Owner sees tracking number and current status
```

---

## WF-OWNER-10: Enroll Facial Recognition (Optional)

```
[Profile] → Facial Recognition → Enroll
    │
    ├── Consent screen (privacy notice)
    ├── [I Agree] → Open camera
    │
    ├── Take 3 photos from different angles
    ├── [Submit enrollment]
    │
    ├── [Success] → Face enrolled, entry without QR/guard enabled
    └── [Remove enrollment] → Face data deleted from system
```
