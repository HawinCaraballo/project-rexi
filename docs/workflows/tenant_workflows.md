# Rexi — Tenant User Workflows

---

## WF-TENANT-01: First Login & Password Setup

```
[Email received: "Your Rexi account"]
    │
    └── Click "Set password" link → Set New Password screen
            │
            ├── New password*, Confirm password*
            └── [Save] → Tenant Dashboard
```

---

## WF-TENANT-02: View My Fees

```
[Tenant Dashboard] → My Fees
    │
    ├── Note: Fees are owned by the apartment owner
    │   Tenant sees fees as read-only if admin configured this
    │
    └── [Download PDF Receipt] (if visible)
```

---

## WF-TENANT-03: Register Visitor (same as owner WF-OWNER-04)

```
Same as WF-OWNER-04
Tenant registers visitors for their rented apartment
```

---

## WF-TENANT-04: Book Common Area (same as owner WF-OWNER-05)

```
Same as WF-OWNER-05
Tenant can book common areas for their rented apartment
```

---

## WF-TENANT-05: Report Issue (same as owner WF-OWNER-09)

```
Same as WF-OWNER-09
Tenant can report issues for their apartment
```

---

## WF-TENANT-06: Use AI Bot (same as WF-OWNER-08)

```
Same as WF-OWNER-08
Access AI chatbot for complex document Q&A
```

---

## WF-TENANT-07: Update Profile

```
[Profile]
    │
    ├── Edit: phone, photo
    ├── Family members: add/remove
    ├── Vehicle: add/remove plate
    ├── Notification preferences: email / push / in-app
    └── Facial Recognition: enroll / remove
```

---

## WF-TENANT-08: Emergency Report

```
[Emergency Button] (prominent, always visible)
    │
    ├── Select type: Fire | Police | Ambulance | Power | Water | Gas
    │
    └── Chat screen opens:
            │
            ├── Bot asks: "What is happening? Where exactly?"
            ├── Resident describes situation
            ├── Bot asks follow-up questions (injured? how many?)
            │
            └── [Generate Report]
                    │
                    ├── Structured report displayed
                    ├── Admin notified immediately
                    ├── [Call emergency number] button
                    └── Report saved in incident log
```
