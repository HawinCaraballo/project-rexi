# TASK: Backend — Phase 2 — Payment Module

**Phase:** 2 (Core MVP)  
**Module:** Finance  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 2 days  
**Depends on:** PHASE_2_fee_calculation_module  

---

## Objective

Implement payment recording, history, automated reminders, and overdue management.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| POST | `/api/v1/fees/{feeId}/payments` | Admin | Record a payment |
| GET | `/api/v1/fees/{feeId}/payments` | Admin, Owner | Payment history for fee |
| GET | `/api/v1/apartments/{aptId}/payments` | Admin, Owner | Full payment history |
| GET | `/api/v1/complexes/{cid}/payments` | Admin | All payments (paginated, date range filter) |
| GET | `/api/v1/complexes/{cid}/payments/report` | Admin | Payment report (Excel/PDF export) |
| GET | `/api/v1/fees/{feeId}/interest` | Admin, Owner | Late interest detail |
| GET | `/api/v1/complexes/{cid}/reports/assembly` | Admin | Assembly financial report (Collection vs Expenses) |

---

## DTOs

### RecordPaymentDto
```csharp
public record RecordPaymentDto(
    decimal Amount,          // Required, > 0
    DateOnly PaymentDate,    // Required, cannot be future
    string PaymentMethod,    // Required: cash|transfer|card|online
    string? Reference,       // Optional, max 100
    string? Notes            // Optional, max 500
);
```

### PaymentDto (response)
```csharp
public record PaymentDto(
    Guid Id,
    Guid FeeId,
    decimal Amount,
    DateOnly PaymentDate,
    string PaymentMethod,
    string? Reference,
    string? Notes,
    string RecordedByName,
    DateTime CreatedAt
);
```

### FeePaymentSummaryDto
```csharp
public record FeePaymentSummaryDto(
    Guid FeeId,
    string ApartmentNumber,
    string OwnerName,
    int Year, int Month,
    decimal TotalAmount,
    decimal PaidAmount,
    decimal RemainingAmount,
    FeeStatus Status,
    DateOnly DueDate,
    int DaysOverdue,  // 0 if not overdue
    decimal LateInterestAmount
);
```

---

## Business Rules

- Payment amount must be > 0
- Payment date cannot be in the future
- Multiple payments can be applied to one fee (partial payments supported)
- Fee status auto-updates:
  - `paid_amount >= total_amount` → status = **paid**
  - `paid_amount > 0 AND paid_amount < total_amount` → status = **partial**
  - `today > due_date AND paid_amount < total_amount` → status = **overdue**
- Payments cannot be deleted (audit trail), only voided by admin with reason

---

## Fee Status Update Logic

```csharp
private void UpdateFeeStatus(AdministrationFee fee)
{
    var totalPaid = fee.Payments.Sum(p => p.Amount);
    fee.Status = totalPaid >= fee.TotalAmount
        ? FeeStatus.Paid
        : DateOnly.FromDateTime(DateTime.Today) > fee.DueDate
            ? FeeStatus.Overdue
            : totalPaid > 0 ? FeeStatus.Partial : FeeStatus.Pending;
}
```

---

## Online Payment Integration (Wompi / PayU)

Implement integration with **Wompi** or **PayU** for online payments (PSE, Credit Cards).

**Workflow:**
1. Resident clicks "Pay Online" in the app.
2. Backend generates a unique `PaymentSession` and returns a secure payment URL (from Wompi/PayU API).
3. Resident completes payment on the provider's site.
4. Provider sends a **Webhook** to `/api/v1/payments/webhook`.
5. Backend validates the signature, updates the `Fee` status, and records the `Payment`.

---

## Automated Payment Reminder Job (Hangfire)

Use **Hangfire** to schedule daily reminders.

```csharp
// Jobs/PaymentReminderJob.cs  (runs daily at 09:00)
public async Task Execute()
{
    var today = DateOnly.FromDateTime(DateTime.Today);
    var reminderDays = new[] { 7, 3, 0 };
    
    foreach (var days in reminderDays)
    {
        var targetDate = today.AddDays(days);
        var pendingFees = await _uow.AdministrationFees
            .GetPendingByDueDateAsync(targetDate);
        
        foreach (var fee in pendingFees)
        {
            await _notificationService.SendPaymentReminderAsync(fee, days);
        }
    }
}
```

**Reminder message templates:**
- 7 days: "Reminder: Your fee of $X for apartment Y is due on [date]. Please pay before to avoid late fees."
- 3 days: "Urgent: Your fee of $X is due in 3 days. Late interest applies after [date]."
- 0 days: "Today is the due date for your fee of $X. Pay now to avoid late interest."

Sent via: **Firebase Cloud Messaging (FCM)** + **SendGrid** email.

---

## Payment Report Export (QuestPDF)

**Filters:**
- Date range (required)
- Status (optional): pending|paid|overdue|partial
- Tower (optional)

**Export formats:** PDF (using **QuestPDF**) and Excel (using **ClosedXML**)

**Columns:** Apartment | Tower | Owner | Period | Total | Paid | Remaining | Status | Due Date | Days Overdue

---

## Validation Rules

| Field | Rules |
|-------|-------|
| `amount` | Required, decimal > 0, max 2 decimal places |
| `paymentDate` | Required, must not be in the future |
| `paymentMethod` | Must be: cash, transfer, card, online |
| `reference` | Optional, max 100 chars |

---

## Acceptance Criteria

- [ ] Recording a payment updates fee status correctly
- [ ] Partial payments accumulate correctly
- [ ] Fee status auto-updates to `paid` when fully covered
- [ ] Fee status becomes `overdue` after due date with unpaid balance
- [ ] Payment reminders sent at 7, 3, and 0 days before due date
- [ ] Late interest calculated and stored daily
- [ ] Payment report exports correct data in Excel and PDF
- [ ] Payments cannot be recorded on already-paid fees (or warn admin)

---

## Unit Tests

- [ ] `RecordPaymentCommandHandler` — updates status to paid when full amount
- [ ] `RecordPaymentCommandHandler` — status remains partial with partial payment
- [ ] `UpdateFeeStatus` — overdue when past due date
- [ ] `PaymentReminderJob` — selects correct fees for each reminder day
- [ ] `LateInterestCalculationJob` — correct amount

---

## Integration Tests

- [ ] POST `/fees/{id}/payments` → 201, fee status updated
- [ ] POST `/fees/{id}/payments` (future date) → 422
- [ ] GET `/complexes/{cid}/payments/report?start=2026-01-01&end=2026-04-30` → 200
