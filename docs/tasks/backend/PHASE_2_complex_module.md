# TASK: Backend — Phase 2 — Complex Management Module

**Phase:** 2 (Core MVP)  
**Module:** Residential  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  
**Depends on:** PHASE_1_database_erd  

---

## Objective

Implement full CRUD for complexes, towers, and apartments including bulk upload.

---

## Endpoints

### Complexes
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes` | SuperAdmin | List all complexes (paginated) |
| POST | `/api/v1/complexes` | SuperAdmin | Create complex |
| GET | `/api/v1/complexes/{id}` | Admin+ | Get complex detail |
| PUT | `/api/v1/complexes/{id}` | Admin+ | Update complex |
| DELETE | `/api/v1/complexes/{id}` | SuperAdmin | Soft-delete complex |
| GET | `/api/v1/complexes/{id}/summary` | Admin | Dashboard stats |

### Towers
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/towers` | Admin+ | List towers |
| POST | `/api/v1/complexes/{cid}/towers` | Admin | Create tower |
| PUT | `/api/v1/complexes/{cid}/towers/{tid}` | Admin | Update tower |
| DELETE | `/api/v1/complexes/{cid}/towers/{tid}` | Admin | Delete tower (only if no apartments) |

### Apartments
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/towers/{tid}/apartments` | Admin | List apartments (paginated + filters) |
| POST | `/api/v1/towers/{tid}/apartments` | Admin | Create apartment |
| PUT | `/api/v1/apartments/{id}` | Admin | Update apartment |
| POST | `/api/v1/towers/{tid}/apartments/bulk-upload` | Admin | CSV/Excel upload |
| GET | `/api/v1/complexes/{cid}/coefficient-check` | Admin | Verify coefficients sum = 1.0 |

---

## DTOs

### CreateComplexDto
```csharp
public record CreateComplexDto(
    string Name,           // Required, max 200, unique
    string? Nit,           // Max 20
    string Address,        // Required
    string City,           // Required, max 100
    string? Phone,         // Max 20, valid phone
    string? Email,         // Valid email
    decimal AnnualBudget,  // Required, > 0
    string FeeCalculationMethod, // coefficient|fixed (Required)
    decimal FixedAdminFeeAmount, // Required if method = fixed, > 0
    decimal LateInterestRate, // 0-100 (percentage)
    string LateInterestType,  // percentage|fixed (Required)
    int PaymentDeadlineDay    // 1-28
);
```

### CreateApartmentDto
```csharp
public record CreateApartmentDto(
    string Number,          // Required, max 20, unique per tower
    int Floor,              // Required, > 0
    decimal? AreaSqm,       // Optional, > 0
    decimal Coefficient,    // Required, 0 < value < 1
    ParkingType ParkingType, // assigned|communal|none
    decimal ParkingCoefficient, // >= 0
    decimal StorageCoefficient  // >= 0
);
```

### BulkUploadApartmentsDto
```csharp
public record BulkUploadApartmentsDto(IFormFile File);
// File: CSV or XLSX
// Columns: Number, Floor, AreaSqm, Coefficient, ParkingType, ParkingCoefficient, StorageCoefficient
```

---

## Business Rules

- Complex name must be unique across the platform
- Annual budget must be > 0
- `PaymentDeadlineDay` must be between 1 and 28
- Apartment `Number` must be unique within the same tower
- Sum of all apartment coefficients (apt + parking + storage) per complex must equal 1.0 (validated via endpoint, not hard enforced on individual saves)
- Tower cannot be deleted if it has apartments with active owners or tenants
- Complex cannot be deleted if it has active financial records

---

## Validation (FluentValidation)

```csharp
public class CreateComplexValidator : AbstractValidator<CreateComplexDto>
{
    public CreateComplexValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(200);
        RuleFor(x => x.Address).NotEmpty();
        RuleFor(x => x.City).NotEmpty().MaximumLength(100);
        RuleFor(x => x.AnnualBudget).GreaterThan(0);
        RuleFor(x => x.LateInterestRate).InclusiveBetween(0, 100);
        RuleFor(x => x.PaymentDeadlineDay).InclusiveBetween(1, 28);
        RuleFor(x => x.Email).EmailAddress().When(x => x.Email != null);
    }
}
```

---

## Bulk Upload Logic

1. Accept CSV or XLSX file
2. Parse using `ClosedXML` (Excel) or `CsvHelper` (CSV)
3. Validate each row against `CreateApartmentDto` rules
4. Return preview DTO: `{ validRows: [], invalidRows: [{ row, errors }] }`
5. On confirm: insert all valid rows in a transaction
6. Return: `{ imported: N, skipped: M, errors: [...] }`

---

## Complex Summary Response

```json
{
  "complexId": "uuid",
  "name": "Las Palmas",
  "totalTowers": 2,
  "totalApartments": 60,
  "occupiedApartments": 55,
  "annualBudget": 120000000,
  "monthlyBudget": 10000000,
  "coefficientsSum": 1.0,
  "coefficientsComplete": true
}
```

---

## Acceptance Criteria

- [ ] All CRUD endpoints return correct status codes
- [ ] Unique name validation works across complexes
- [ ] Bulk upload processes valid CSV/XLSX and rejects bad rows with row-level errors
- [ ] Soft delete prevents data loss but hides from active lists
- [ ] Coefficient sum validation endpoint returns accurate result
- [ ] Pagination and filters work on apartment list (filter by floor, parking type)

---

## Unit Tests

- [ ] `CreateComplexCommandHandler` — happy path
- [ ] `CreateComplexCommandHandler` — duplicate name returns failure
- [ ] `BulkUploadApartmentsCommandHandler` — valid CSV imports correctly
- [ ] `BulkUploadApartmentsCommandHandler` — mixed valid/invalid rows returns partial success
- [ ] `GetComplexSummaryQueryHandler` — returns correct statistics

---

## Integration Tests

- [ ] POST `/complexes` → 201 with valid data
- [ ] POST `/complexes` → 422 with duplicate name
- [ ] POST `/towers/{tid}/apartments/bulk-upload` → 200 with valid CSV
- [ ] GET `/complexes/{cid}/coefficient-check` → 200 with sum result
