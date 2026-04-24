# TASK: Backend — Phase 2 — Fee Calculation Module

**Phase:** 2 (Core MVP)  
**Module:** Finance  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  
**Depends on:** PHASE_2_complex_module, PHASE_2_owner_tenant_module  

---

## Objective

Implement automatic monthly administration fee calculation per apartment, PDF generation, and emailing receipts to owners.

---

## Fee Formula

**Option A: Coefficient-based**
```
Administration Fee = (AptCoefficient + ParkingCoefficient + StorageCoefficient) × (AnnualBudget / 12)
```

**Option B: Fixed Fee**
```
Administration Fee = Complex.FixedAdminFeeAmount
```

Extraordinary fees and fines charged in the current month are added:
```
Total = base_amount + parking_amount + storage_amount + extraordinary_amount + fines_amount
```

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/fees` | Admin | List fees (paginated, filterable by period/status) |
| POST | `/api/v1/complexes/{cid}/fees/generate` | Admin | Manually trigger fee generation |
| GET | `/api/v1/fees/{id}` | Admin, Owner(linked) | Get fee detail with breakdown |
| GET | `/api/v1/apartments/{aptId}/fees` | Admin, Owner, Tenant | Fee history for apartment |
| GET | `/api/v1/fees/{id}/pdf` | Admin, Owner(linked) | Download PDF receipt |
| POST | `/api/v1/fees/{id}/send-email` | Admin | Re-send PDF receipt by email |
| GET | `/api/v1/complexes/{cid}/fees/summary` | Admin | Financial summary by period |

---

## Fee Generation Logic

```csharp
// FeeCalculationService.cs
public List<AdministrationFee> CalculateAll(Complex complex, int year, int month)
{
    var monthlyBudget = complex.AnnualBudget / 12;
    var dueDate = new DateOnly(year, month, complex.PaymentDeadlineDay);
    var isFixed = complex.FeeCalculationMethod == "Fixed";
    
    return complex.Towers
        .SelectMany(t => t.Apartments)
        .Where(a => a.IsActive)
        .Select(apt => new AdministrationFee
        {
            ApartmentId = apt.Id,
            Year = year,
            Month = month,
            BaseAmount = isFixed ? complex.FixedAdminFeeAmount : (apt.Coefficient * monthlyBudget),
            ParkingAmount = isFixed ? 0 : (apt.ParkingCoefficient * monthlyBudget),
            StorageAmount = isFixed ? 0 : (apt.StorageCoefficient * monthlyBudget),
            ExtraordinaryAmount = GetExtraordinaryFeesAmount(complex.Id, year, month),
            FinesAmount = GetPendingFinesAmount(apt.Id, year, month),
            DueDate = dueDate,
            Status = FeeStatus.Pending
        }).ToList();
}
```

**Protection:** If fees for the given month already exist → return error "Fees already generated for this period."

---

## PDF Generation

Use `QuestPDF` library for PDF receipts.

**PDF Layout:**
```
[Complex Logo]       [Complex Name]
                     [Address, City]
                     [NIT]

ADMINISTRATION FEE RECEIPT
Period: April 2026 | Apartment: 101-A | Tower: A

Owner: Carlos García
ID: CC 123456789

BREAKDOWN:
Base administration:   $ 350,000
Parking:               $  52,000
Storage:               $  18,000
Extraordinary Fee:     $ 100,000 (Roof Repair)
Fines:                 $  50,000
─────────────────────────────────
TOTAL:                 $ 570,000

Due Date: April 10, 2026
Reference: REXI-2026-04-101A

Payment methods: [configured payment info]
```

**After PDF generation:** Upload to Azure Blob → store URL in `pdf_url` field.

---

## Email Sending

```csharp
// After PDF generated and uploaded:
await _emailService.SendFeeReceiptAsync(
    to: owner.Email,
    ownerName: owner.FullName,
    apartmentNumber: apartment.Number,
    period: "April 2026",
    totalAmount: fee.Total,
    dueDate: fee.DueDate,
    pdfUrl: fee.PdfUrl
);
```

Email template: "Your administration fee for April 2026 is ready. Amount: $470,000. Due: April 10. [View/Download Receipt]"

---

## Background Job: Monthly Auto-Generation (Hangfire)

Use **Hangfire** for scheduling and processing background jobs.

```csharp
// Jobs/MonthlyFeeGenerationJob.cs
public async Task Execute()
{
    var complexes = await _uow.Complexes.GetAllActiveAsync();
    foreach (var complex in complexes)
    {
        var (year, month) = GetCurrentPeriod();
        if (await _uow.AdministrationFees.ExistsAsync(complex.Id, year, month))
            continue; // Already generated
        
        var command = new GenerateMonthlyFeesCommand(complex.Id, year, month);
        await _mediator.Send(command);
    }
}
```

**Scheduled:** Recurring job configured in Hangfire to run the 1st of every month at 06:00 AM (UTC-5).

---

## Financial Summary Response

```json
{
  "period": "2026-04",
  "totalApartments": 60,
  "totalExpected": 28200000,
  "totalCollected": 23500000,
  "totalOverdue": 4700000,
  "totalPending": 0,
  "collectionRate": 83.3,
  "overdueApartments": [
    { "apartment": "101-A", "amount": 470000, "daysOverdue": 5 }
  ]
}
```

---

## Late Interest Calculation Job (Parametrizable)

```csharp
// Jobs/LateInterestCalculationJob.cs
// Runs daily at 00:01
// For each overdue fee (status = 'overdue', past due date):
// 1. Get Complex configuration: InterestRate (%), InterestType (fixed/percentage), InterestFrequency (daily)
// 2. DaysOverdue = TODAY - DueDate
// 3. if (type == percentage) 
//      Interest = TotalFee * (Rate/100) * DaysOverdue
//    else 
//      Interest = FixedAmount * DaysOverdue
// 4. Update late_interests table
```

---

## Validation Rules

| Rule | Detail |
|------|--------|
| Period uniqueness | Only one fee set per apartment per year/month |
| Budget required | Complex must have `annual_budget > 0` |
| Active apartments only | Inactive apartments excluded from generation |
| Coefficient completeness | Warn if coefficients don't sum to 1.0 (don't block) |

---

## Acceptance Criteria

- [ ] Fee generation creates one record per active apartment
- [ ] Fee amounts match formula exactly
- [ ] PDF is generated with correct layout and amounts
- [ ] PDF uploaded to Blob storage and URL stored
- [ ] Email sent to owner with PDF attachment or link
- [ ] Re-send email endpoint works
- [ ] Monthly job runs automatically and skips already-generated periods
- [ ] Late interest calculated daily for overdue fees
- [ ] Financial summary returns accurate totals

---

## Unit Tests

- [ ] `FeeCalculationService.CalculateAll` — formula accuracy for various coefficients
- [ ] `FeeCalculationService.CalculateAll` — excludes inactive apartments
- [ ] `FeeCalculationService.CalculateAll` — includes pending fines
- [ ] `GenerateMonthlyFeesCommandHandler` — skips if period exists
- [ ] `LateInterestCalculationJob` — correct days overdue and amount

---

## Integration Tests

- [ ] POST `/complexes/{cid}/fees/generate` → 200, creates fees for all active apartments
- [ ] POST `/complexes/{cid}/fees/generate` (second time same period) → 422
- [ ] GET `/fees/{id}/pdf` → 200 with PDF content-type
- [ ] GET `/complexes/{cid}/fees/summary` → correct totals
