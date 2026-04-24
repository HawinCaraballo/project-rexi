# TASK: Backend — Phase 3 — Common Areas Module

**Phase:** 3 (Important)  
**Module:** Facilities  
**Priority:** 🟡 Important  
**Estimate:** 2 days  
**Depends on:** PHASE_2_owner_tenant_module, PHASE_3_notification_module  

---

## Objective

Implement common area CRUD and booking system with conflict prevention, fee support, and approval workflow.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/common-areas` | Auth | List common areas |
| POST | `/api/v1/complexes/{cid}/common-areas` | Admin | Create common area |
| PUT | `/api/v1/common-areas/{id}` | Admin | Update common area |
| DELETE | `/api/v1/common-areas/{id}` | Admin | Soft delete |
| GET | `/api/v1/common-areas/{id}/availability` | Auth | Get availability calendar |
| POST | `/api/v1/common-areas/{id}/bookings` | Owner, Tenant | Create booking |
| GET | `/api/v1/common-areas/{id}/bookings` | Admin | List bookings for area |
| GET | `/api/v1/my/bookings` | Owner, Tenant | My bookings |
| PUT | `/api/v1/bookings/{id}/cancel` | Owner, Tenant, Admin | Cancel booking |
| PUT | `/api/v1/bookings/{id}/approve` | Admin | Approve pending booking |

---

## DTOs

### CreateCommonAreaDto
```csharp
public record CreateCommonAreaDto(
    string Name,             // Required, max 100
    string? Description,
    int? Capacity,           // Optional, > 0
    decimal Fee,             // Required, >= 0
    TimeOnly OpenTime,       // Required
    TimeOnly CloseTime,      // Required, must be > OpenTime
    int AdvanceBookingHours, // Required, >= 1
    bool RequiresApproval,
    bool BlocksOnDebt
);
```

### CreateBookingDto
```csharp
public record CreateBookingDto(
    DateTimeOffset StartTime,  // Required, must be future
    DateTimeOffset EndTime,    // Required, must be > StartTime
    string? Notes              // Optional, max 500
);
```

### AvailabilityDto
```csharp
public record AvailabilityDto(
    DateOnly Date,
    List<TimeSlotDto> OccupiedSlots,
    List<TimeSlotDto> AvailableSlots
);

public record TimeSlotDto(TimeOnly Start, TimeOnly End, bool IsFree);
```

---

## Business Rules

- Time slot must be within area's `open_time` to `close_time`
- Booking must be made at least `advance_booking_hours` before start
- No overlapping confirmed bookings for same area (enforced by DB exclusion constraint + application-level check)
- If `blocks_on_debt = true`, resident with overdue fee cannot book
- If `requires_approval = true`, booking starts as "pending" and notifies admin
- Fee > 0 → future: integrate payment before confirmation (Phase 4 enhancement)
- Cancellation allowed up to 2 hours before start (configurable)

---

## Overlap Check (Application Level)

```csharp
var conflicts = await _uow.Bookings.GetConflictingAsync(
    commonAreaId: dto.CommonAreaId,
    start: dto.StartTime,
    end: dto.EndTime,
    excludeStatuses: [BookingStatus.Cancelled]
);

if (conflicts.Any())
    return Result.Failure<BookingDto>("Time slot is already booked");
```

---

## Debt Check

```csharp
var hasDebt = area.BlocksOnDebt &&
    await _uow.AdministrationFees.HasOverdueAsync(apartmentId);

if (hasDebt)
    return Result.Failure<BookingDto>("Cannot book: outstanding administration fee");
```

---

## Availability Calendar Response

For a 7-day window, return hour-by-hour slots with availability status.

---

## Acceptance Criteria

- [ ] Common area CRUD works with correct validations
- [ ] Overlapping bookings rejected (both DB constraint and app-level)
- [ ] Debt check blocks booking when configured
- [ ] Approval workflow: pending → admin notified → approved/rejected → booker notified
- [ ] My bookings returns only current user's bookings
- [ ] Cancellation within allowed window works
- [ ] Availability calendar reflects actual bookings

---

## Unit Tests

- [ ] Overlap detection logic
- [ ] Debt check logic
- [ ] Advance booking hours validation

---

## Integration Tests

- [ ] POST `/common-areas/{id}/bookings` → 201 confirmed booking
- [ ] POST conflicting booking → 422
- [ ] POST when in debt + blocks_on_debt=true → 422
- [ ] PUT `/bookings/{id}/approve` → 200, booker notified
