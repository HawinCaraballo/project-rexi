# Rexi — Administrator User Workflows

---

## WF-ADMIN-01: Login & Dashboard Access

```
[Login Screen]
    │
    ├─ Enter email + password
    ├─ Click "Sign In"
    │
    ├── [Error] Invalid credentials → Show error message → Retry
    ├── [Error] Account inactive → Show "Contact support" message
    │
    └── [Success] → Redirect to Admin Dashboard
                        │
                        ├── Summary cards (fees collected, overdue, visitors today)
                        ├── Recent activity feed
                        ├── Quick actions (Generate fees, Send announcement)
                        └── Navigation sidebar (all modules)
```

---

## WF-ADMIN-02: Create Residential Complex

```
[Dashboard]
    │
    └── Complexes → New Complex
            │
            ├── Fill form:
            │   ├── Name* (unique)
            │   ├── NIT
            │   ├── Address*, City*
            │   ├── Phone, Email
            │   ├── Logo upload
            │   ├── Annual Budget*
            │   ├── Late interest rate %
            │   └── Payment deadline day (1-28)
            │
            ├── [Validate] All required fields → show field-level errors
            │
            └── [Submit] → Complex created
                    │
                    └── Prompt: "Add towers now?" [Yes/Later]
                            │
                            Yes → WF-ADMIN-03
```

---

## WF-ADMIN-03: Add Tower & Apartments (manual)

```
[Complex Detail]
    │
    └── Towers tab → Add Tower
            │
            ├── Name*, Floors*
            └── [Save] → Tower created
                    │
                    └── Add Apartments (per tower)
                            │
                            Option A: Manual entry
                            ├── Number*, Floor*, Area, Coefficient*
                            ├── Parking type (assigned/communal/none)
                            ├── Parking coefficient
                            └── Storage coefficient
                            │
                            Option B: Bulk upload (CSV/Excel)
                            ├── Download template
                            ├── Fill & upload
                            ├── Preview data table
                            ├── [Error rows] Highlighted with reasons
                            └── [Confirm] → All valid rows imported
```

---

## WF-ADMIN-04: Register Owner

```
[Owners module] → New Owner
    │
    ├── Form:
    │   ├── First name*, Last name*
    │   ├── ID type*, ID number* (unique)
    │   ├── Email* (unique → becomes login)
    │   ├── Phone
    │   ├── Profile photo
    │   └── Assign apartment(s)* [multi-select from complex apartments]
    │
    ├── Family members (optional, repeatable):
    │   ├── Name, Relationship, Age
    │   └── [Add another family member]
    │
    └── [Save]
            │
            ├── System creates user account (email + temp password)
            ├── Email sent with credentials
            └── Owner record shown with "User created" badge
```

---

## WF-ADMIN-05: Generate Monthly Fees

```
[Fees module] → Generate Fees
    │
    ├── Select: Year, Month
    ├── Preview: table of all apartments + calculated amounts
    │   (base + parking + storage + pending fines)
    │
    ├── [Confirm Generate]
    │       │
    │       ├── System creates fee records for all active apartments
    │       ├── Generates PDF receipts
    │       ├── Sends email with PDF to each owner
    │       └── Dashboard updated
    │
    └── [Cancel]
```

---

## WF-ADMIN-06: Record Payment

```
[Fees module] → Search apartment or owner
    │
    └── Select fee period → Record Payment
            │
            ├── Amount*, Payment date*, Method*
            ├── Reference number, Notes
            └── [Save]
                    │
                    ├── Fee marked as paid/partial
                    ├── Payment history entry created
                    └── Owner notified (in-app + email)
```

---

## WF-ADMIN-07: Approve / Reject Fine

```
[Fines module] → Pending Approval list
    │
    └── Select fine → Review detail
            │
            ├── View: infraction type, description, evidence photos, guard, apartment
            │
            ├── [Approve]
            │       ├── Fine status → "approved"
            │       ├── Owner notified with fine amount and reason
            │       └── Fine added to next fee cycle
            │
            └── [Reject]
                    ├── Enter rejection reason
                    ├── Fine status → "rejected"
                    └── Guard notified
```

---

## WF-ADMIN-08: Review Appeal

```
[Fines module] → Appealed tab
    │
    └── Select appealed fine
            │
            ├── View: appeal reason, supporting documents (if any), original evidence
            │
            ├── [Approve Appeal]
            │       ├── Fine status → "appeal_approved"
            │       ├── Fine removed from fee cycle
            │       └── Owner notified: "Fine waived"
            │
            └── [Reject Appeal]
                    ├── Fine status → "appeal_rejected"
                    └── Owner notified: "Appeal denied"
```

---

## WF-ADMIN-09: Send General Announcement

```
[Notifications module] → New Announcement
    │
    ├── Subject*, Body* (rich text editor)
    ├── Target audience:
    │   ├── All residents
    │   ├── Per tower (multi-select)
    │   └── Per apartment (multi-select)
    │
    ├── Channels (checkboxes):
    │   ├── In-app notification
    │   ├── Email
    │   └── Push notification
    │
    ├── [Preview] → See how it looks per channel
    │
    └── [Send] → Delivered to selected audience
```

---

## WF-ADMIN-10: Generate Reports

```
[Reports module]
    │
    ├── Report type:
    │   ├── Payment summary (by date range)
    │   ├── Overdue fees
    │   ├── Fines history
    │   ├── Visitor log
    │   └── Common area usage
    │
    ├── Filters: date range, tower, status
    │
    ├── [Generate] → Preview table in screen
    │
    └── Export: [PDF] [Excel]
```
