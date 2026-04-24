# TASK: Backend — Phase 3 — Vehicle Module

**Phase:** 3 (Important)  
**Module:** Security  
**Priority:** 🟡 Important  
**Estimate:** 1 day  
**Depends on:** PHASE_2_owner_tenant_module, PHASE_3_visitor_module  

---

## Objective

Register and manage vehicles for residents and visitors, with plate-based lookup for guards.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/vehicles` | Admin | List all vehicles (paginated) |
| POST | `/api/v1/complexes/{cid}/vehicles` | Admin | Register resident vehicle |
| GET | `/api/v1/vehicles/{id}` | Admin, Guard | Vehicle detail |
| PUT | `/api/v1/vehicles/{id}` | Admin | Update vehicle |
| DELETE | `/api/v1/vehicles/{id}` | Admin | Remove vehicle |
| GET | `/api/v1/vehicles/by-plate/{plate}` | Guard, Admin | Plate lookup |
| POST | `/api/v1/users/vehicles` | Owner, Tenant | Register own vehicle |
| GET | `/api/v1/users/vehicles` | Owner, Tenant | My vehicles |
| DELETE | `/api/v1/users/vehicles/{id}` | Owner, Tenant | Remove own vehicle |

---

## DTOs

### RegisterVehicleDto
```csharp
public record RegisterVehicleDto(
    string LicensePlate,  // Required, regex: [A-Z]{3}[0-9]{3} or [A-Z]{3}[0-9]{2}[A-Z]
    string? Make,         // Optional, max 50
    string? Model,        // Optional, max 50
    string? Color,        // Optional, max 30
    int? Year,            // Optional, 1900-currentYear+1
    // Admin only - link to owner or tenant:
    Guid? OwnerId,
    Guid? TenantId
);
```

### PlateLookupDto (Guard response)
```csharp
public record PlateLookupDto(
    Guid VehicleId,
    string LicensePlate,
    string? Make, string? Model, string? Color, int? Year,
    string OwnerType,     // "owner"|"tenant"|"visitor"
    string OwnerName,
    string? PhotoUrl,
    string ApartmentNumber,
    bool IsAuthorized,    // true for registered resident vehicles
    string AuthorizationStatus  // "authorized"|"visitor"|"not_registered"
);
```

---

## Business Rules

- License plate must be unique per complex (residents and visitors tracked separately)
- Plate format validated by regex (Colombia: ABC123 or ABC12D)
- Owner/tenant can register their own vehicles (max 3 per apartment, configurable)
- Guard plate search returns: resident info + authorization status
- Visitor vehicles: linked through visitor registration (see visitor module)
- Guard access log entry created on each plate lookup

---

## Plate Lookup Logic

```csharp
public async Task<Result<PlateLookupDto>> LookupByPlateAsync(string plate, Guid complexId)
{
    var vehicle = await _uow.Vehicles.GetByPlateAsync(complexId, plate.ToUpper());
    
    if (vehicle is null)
        return Result.Success(new PlateLookupDto(
            LicensePlate: plate,
            IsAuthorized: false,
            AuthorizationStatus: "not_registered"
        ));
    
    return Result.Success(MapToPlateLookup(vehicle));
}
```

---

## Acceptance Criteria

- [ ] Plate format validation rejects invalid formats
- [ ] Plate uniqueness enforced per complex
- [ ] Guard plate lookup returns owner/tenant info and authorization
- [ ] Guard plate lookup returns visitor info if visitor vehicle
- [ ] Not-found plates return "not_registered" status (not 404)
- [ ] Access log entry created on each plate search

---

## Unit Tests

- [ ] Plate format validation regex
- [ ] `LookupByPlateAsync` — returns correct owner info
- [ ] `LookupByPlateAsync` — not found returns not_registered

---

## Integration Tests

- [ ] GET `/vehicles/by-plate/ABC123` → 200 with owner info
- [ ] GET `/vehicles/by-plate/UNKNOWN` → 200 with not_registered status
- [ ] POST `/users/vehicles` → 422 with invalid plate format
