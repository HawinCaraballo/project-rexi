# Rexi — Guard User Workflows

---

## WF-GUARD-01: Login

```
[Guard Login Screen]
    │
    ├── Email + Password
    └── [Sign In] → Guard Dashboard (simplified, access-control-focused)
            │
            ├── Quick Search bar (plate / QR / ID)
            ├── Today's access log
            └── Pending packages
```

---

## WF-GUARD-02: Vehicle Plate Search

```
[Quick Search] → Enter license plate
    │
    ├── [Not found] → "Vehicle not registered in this complex"
    │
    └── [Found] → Display card:
            │
            ├── Vehicle: plate, make, model, color
            ├── Resident: name, photo, apartment
            ├── Authorization status: ✅ Authorized / ❌ Not authorized
            └── [Log Entry/Exit]
```

---

## WF-GUARD-03: Visitor QR Code Scan

```
[Quick Search] → Scan QR / Enter QR code
    │
    ├── [Not found / Expired] → "Visitor QR invalid or expired" (red screen)
    │
    └── [Found & valid] → Display card:
            │
            ├── Visitor: name, photo, ID
            ├── Apartment they are visiting
            ├── Validity: temporary (expiry date) / permanent
            └── [Authorize entry] → Access log recorded
                    │
                    └── Owner notified: "Your visitor [Name] has arrived"
```

---

## WF-GUARD-04: Search Resident by ID

```
[Quick Search] → Enter ID number
    │
    ├── [Not found] → "No resident found with this ID"
    │
    └── [Found] → Display card:
            │
            ├── Name, Photo, Apartment, Type (Owner/Tenant)
            ├── Active vehicles
            ├── Active visitors
            └── [Log entry/exit]
```

---

## WF-GUARD-05: Log Package Arrival

```
[Packages] → Register Package
    │
    ├── Search apartment (autocomplete)
    ├── Sender (optional)
    ├── Description (optional)
    ├── Photo of package (optional)
    └── [Save]
            │
            ├── Package logged
            └── Owner/Tenant notified:
                    "📦 You have a package at the reception desk"
```

---

## WF-GUARD-06: Issue Fine

```
[Fines] → New Fine
    │
    ├── Search apartment*
    ├── Infraction type* (dropdown: noise, parking violation, trash, etc.)
    ├── Description*
    ├── Rule reference (text field, reference to coexistence rules)
    ├── Evidence photos* (at least 1 required)
    └── [Submit for approval]
            │
            ├── Fine status → "pending_approval"
            └── Admin notified for review
```

---

## WF-GUARD-07: Facial Recognition Entry (if camera system integrated)

```
[Camera station detects face]
    │
    ├── [No match] → Manual override required (Guard confirmation)
    │
    └── [Match found] → Display on guard screen:
            │
            ├── Resident photo, name, apartment
            ├── Access authorized: ✅
            └── Entry logged automatically
```
