# TASK: Backend — Phase 2 — Owner & Tenant Module

**Phase:** 2 (Core MVP)  
**Module:** People  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  
**Depends on:** PHASE_2_complex_module, PHASE_1_auth_module  

---

## Objective

Implement full CRUD for owners and tenants, including automatic user account creation and family member management.

---

## Endpoints

### Owners
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/owners` | Admin | List owners (paginated, filterable) |
| POST | `/api/v1/complexes/{cid}/owners` | Admin | Create owner + user account |
| GET | `/api/v1/owners/{id}` | Admin, Owner(self) | Get owner detail |
| PUT | `/api/v1/owners/{id}` | Admin, Owner(self) | Update owner |
| GET | `/api/v1/owners/{id}/apartments` | Admin, Owner(self) | Owner's apartments |
| POST | `/api/v1/owners/{id}/apartments/{aptId}` | Admin | Assign additional apartment |
| DELETE | `/api/v1/owners/{id}/apartments/{aptId}` | Admin | Remove apartment assignment |
| GET | `/api/v1/owners/search` | Admin, Guard | Search by name, ID, email, apartment |

### Tenants
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/apartments/{aptId}/tenants` | Admin, Owner | List tenants for apartment |
| POST | `/api/v1/apartments/{aptId}/tenants` | Admin, Owner | Create tenant + user account |
| GET | `/api/v1/tenants/{id}` | Admin, Owner(linked), Tenant(self) | Tenant detail |
| PUT | `/api/v1/tenants/{id}` | Admin, Owner(linked) | Update tenant |
| DELETE | `/api/v1/tenants/{id}` | Admin, Owner(linked) | Deactivate tenant |
| GET | `/api/v1/tenants/search` | Admin, Guard | Search by name, ID, email, apartment |

### Family Members
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/owners/{id}/family` | Admin, Owner(self) | List family members |
| POST | `/api/v1/owners/{id}/family` | Admin, Owner(self) | Add family member |
| PUT | `/api/v1/owners/{id}/family/{fid}` | Admin, Owner(self) | Update family member |
| DELETE | `/api/v1/owners/{id}/family/{fid}` | Admin, Owner(self) | Remove family member |

---

## DTOs

### CreateOwnerDto
```csharp
public record CreateOwnerDto(
    string FirstName,      // Required, max 100
    string LastName,       // Required, max 100
    string IdType,         // Required: CC|CE|NIT|Passport
    string IdNumber,       // Required, unique in complex
    string Email,          // Required, unique platform-wide
    string? Phone,         // Optional, max 20, valid format
    string? PhotoUrl,      // Optional (upload handled separately)
    List<Guid> ApartmentIds,  // Required, min 1, must belong to complex
    List<CreateFamilyMemberDto> FamilyMembers  // Optional
);

public record CreateFamilyMemberDto(
    string FirstName,      // Required, max 100
    string LastName,       // Required, max 100
    string Relationship,   // Required: spouse|child|parent|sibling|other
    DateOnly? BirthDate    // Optional
);
```

### CreateTenantDto
```csharp
public record CreateTenantDto(
    string FirstName,
    string LastName,
    string IdType,
    string IdNumber,
    string Email,          // Required → user login
    string? Phone,
    DateOnly? LeaseStart,
    DateOnly? LeaseEnd,    // Must be >= LeaseStart if provided
    List<CreateFamilyMemberDto> FamilyMembers
);
```

### OwnerDto (response)
```csharp
public record OwnerDto(
    Guid Id, string FullName, string IdType, string IdNumber,
    string Email, string? Phone, string? PhotoUrl,
    List<ApartmentSummaryDto> Apartments,
    List<FamilyMemberDto> FamilyMembers,
    DateTime CreatedAt
);
```

---

## Business Rules

- Email must be unique across the entire platform (all complexes)
- ID number must be unique within the same complex
- Owner must have at least 1 apartment assigned
- Apartment can only have 1 active tenant at a time
- Owner is responsible for registering tenants for their apartments only
- Deactivating a tenant does NOT delete their user account (for history preservation)
- Lease end date must be >= lease start date

---

## Photo Upload

Photos are NOT stored as base64. Flow:
1. Frontend calls `POST /api/v1/media/upload` → gets blob URL
2. Owner/tenant form includes the returned URL
3. URL stored in `photo_url` field

---

## Search Endpoint

**GET `/api/v1/owners/search?q=Carlos&complexId={cid}`**

Searches across: `first_name`, `last_name`, `id_number`, `email`, `apartment_number`

Response includes:
```json
[
  {
    "id": "uuid",
    "fullName": "Carlos García",
    "idNumber": "123456789",
    "apartments": ["101-A", "204-B"],
    "type": "owner"
  }
]
```

---

## Validation Rules (All Fields)

| Field | Rules |
|-------|-------|
| `firstName` / `lastName` | Required, 2-100 chars, no numbers |
| `idType` | Must be one of: CC, CE, NIT, Passport |
| `idNumber` | Required, 5-20 chars, alphanumeric |
| `email` | Required, valid format, unique |
| `phone` | Optional, 7-15 digits, may start with + |
| `apartmentIds` | Min 1 entry, all must exist in complex |
| `leaseEnd` | Must be >= leaseStart if both provided |
| `relationship` | Must be: spouse, child, parent, sibling, other |

---

## Acceptance Criteria

- [ ] Creating owner automatically creates user account and sends welcome email
- [ ] Owner with duplicate email returns 422 with clear error
- [ ] Owner can have multiple apartments (separate assignment endpoint)
- [ ] Owner can only register tenant for their own apartments (if role = Owner)
- [ ] Tenant deactivation preserves history, disables login
- [ ] Search works across name, ID, email, and apartment number
- [ ] Pagination returns correct totals and pages

---

## Unit Tests

- [ ] `CreateOwnerCommandHandler` — creates user + owner + sends email
- [ ] `CreateOwnerCommandHandler` — duplicate email returns failure
- [ ] `CreateTenantCommandHandler` — validates apartment belongs to owner
- [ ] `SearchOwnersQueryHandler` — returns correct results for partial name match

---

## Integration Tests

- [ ] POST `/owners` → 201 with all fields
- [ ] POST `/owners` → 422 with duplicate email
- [ ] POST `/apartments/{aptId}/tenants` → 422 if apartment already has active tenant
- [ ] GET `/owners/search?q=Car` → returns matching owners
