# TASK: Backend — Phase 4 — Fines Module

**Phase:** 4 (Advanced)  
**Module:** Security  
**Priority:** 🟢 Advanced  
**Estimate:** 2 days  
**Depends on:** PHASE_2_fee_calculation_module, PHASE_3_notification_module  

---

## Objective

Implement the full fines lifecycle: guard issues fine → admin approves → owner/tenant notified → optional appeal → fine added to fee cycle.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/fines` | Admin | List fines (paginated, filterable) |
| POST | `/api/v1/complexes/{cid}/fines` | Guard | Issue fine |
| GET | `/api/v1/fines/{id}` | Admin, Guard, Owner(linked) | Fine detail |
| PUT | `/api/v1/fines/{id}/approve` | Admin | Approve fine |
| PUT | `/api/v1/fines/{id}/reject` | Admin | Reject fine |
| PUT | `/api/v1/fines/{id}/appeal` | Owner, Tenant | File appeal |
| PUT | `/api/v1/fines/{id}/resolve-appeal` | Admin | Approve/reject appeal |
| GET | `/api/v1/complexes/{cid}/fines/report` | Admin | Fines report (date range) |
| GET | `/api/v1/apartments/{aptId}/fines` | Admin, Owner, Tenant | Apartment's fine history |

---

## DTOs

### IssueFineDto
```csharp
public record IssueFineDto(
    Guid ApartmentId,       // Required
    string InfractionType,  // Required, max 100
    string Description,     // Required, max 1000
    string? RuleReference,  // Optional, max 200
    decimal Amount,         // Required, > 0
    List<string> EvidenceUrls  // Required, min 1 URL (uploaded to blob beforehand)
);
```

### ApproveFineDto
```csharp
public record ApproveFineDto(
    bool IsApproved,
    string? RejectionReason  // Required if IsApproved = false
);
```

### FileAppealDto
```csharp
public record FileAppealDto(
    string Reason,            // Required, max 1000
    List<string>? DocumentUrls // Optional supporting documents
);
```

### ResolveAppealDto
```csharp
public record ResolveAppealDto(
    bool IsApproved,
    string? Notes  // Optional
);
```

### FineDto (response)
```csharp
public record FineDto(
    Guid Id,
    string ApartmentNumber,
    string OwnerName,
    string IssuedByName,
    string InfractionType,
    string Description,
    string? RuleReference,
    decimal Amount,
    List<string> EvidenceUrls,
    FineStatus Status,
    string? ApprovedByName,
    string? RejectionReason,
    string? AppealReason,
    DateTimeOffset? AppealFiledAt,
    DateTimeOffset? ResolvedAt,
    bool ChargedInFee,
    DateTimeOffset CreatedAt
);
```

---

## Fine Status State Machine

```
issued
  └─ pending_approval (guard submits)
        ├─ approved (admin approves)
        │     ├─ charged (included in fee cycle)
        │     └─ appealed (owner files appeal within deadline)
        │           ├─ appeal_approved (fine waived)
        │           └─ appeal_rejected (fine stands, charged)
        └─ rejected (admin rejects)
```

---

## Business Rules

- Guard must upload at least 1 evidence photo before submission
- Evidence photos uploaded via `POST /media/upload` → URLs included in IssueFineDto
- Appeal must be filed within configurable window (default: 5 days from approval)
- Approved fines automatically included in the NEXT month's administration fee
- Appeal deadline enforced at application level:

```csharp
var appealDeadline = fine.ResolvedAt!.Value.AddDays(complex.AppealDeadlineDays);
if (DateTimeOffset.UtcNow > appealDeadline)
    return Result.Failure<FineDto>("Appeal deadline has passed");
```

---

## Adding Fine to Fee Cycle

When a fine is charged (status = `charged`), add its amount to the current month's `fines_amount` for the apartment's administration fee:

```csharp
var currentFee = await _uow.AdministrationFees.GetCurrentAsync(fine.ApartmentId);
if (currentFee is not null)
{
    currentFee.FinesAmount += fine.Amount;
    fine.FeeId = currentFee.Id;
    fine.Status = FineStatus.Charged;
}
```

---

## Fines Report

**Filters:** date range (required), status, tower

**Columns:** Date | Apartment | Tower | Infraction | Amount | Status | Guard | Admin | Resolution

**Export:** Excel and PDF

**Summary section:**
```json
{
  "totalFines": 15,
  "totalAmount": 750000,
  "byStatus": {
    "pending_approval": 2,
    "approved": 8,
    "charged": 5,
    "rejected": 2,
    "appealed": 1
  }
}
```

---

## Notification Triggers

| Event | Recipient | Message |
|-------|-----------|---------|
| Fine approved | Owner/Tenant | "A fine of $X has been issued for [infraction]. You can appeal within [N] days." |
| Fine rejected | Guard (info) | "Fine for apartment X was not approved." |
| Appeal approved | Owner/Tenant | "Your appeal was approved. The fine has been waived." |
| Appeal rejected | Owner/Tenant | "Your appeal was denied. The fine of $X will be charged." |

---

## Acceptance Criteria

- [ ] Guard cannot submit fine without evidence photo
- [ ] Status transitions follow state machine (no illegal transitions)
- [ ] Appeal deadline enforced
- [ ] Approved fine included in next fee cycle
- [ ] Fine waived (appeal approved) → removed from fee cycle if already there
- [ ] Admin report exports correct data
- [ ] All notifications sent at correct transitions

---

## Unit Tests

- [ ] `IssueFineCommandHandler` — rejects submission without evidence
- [ ] `FileAppealCommandHandler` — rejects if past deadline
- [ ] `ApproveFineCommandHandler` — triggers notification to owner
- [ ] Fine amount correctly added to administration fee

---

## Integration Tests

- [ ] POST `/fines` (no evidence) → 422
- [ ] PUT `/fines/{id}/approve` → 200, fine status = approved
- [ ] PUT `/fines/{id}/appeal` (past deadline) → 422
- [ ] GET `/complexes/{cid}/fines/report?start=X&end=Y` → 200 with data
