# TASK: Backend — Phase 3 — Visitor Module

**Phase:** 3 (Important)  
**Module:** Security  
**Priority:** 🟡 Important  
**Estimate:** 2 days  
**Depends on:** PHASE_2_owner_tenant_module  

---

## Objective

Implement visitor registration with QR code generation, vehicle linkage, expiry management, and access logging.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/apartments/{aptId}/visitors` | Owner, Tenant, Admin | List visitors |
| POST | `/api/v1/apartments/{aptId}/visitors` | Owner, Tenant, Admin | Register visitor |
| GET | `/api/v1/visitors/{id}` | Owner(linked), Admin | Visitor detail + QR |
| PUT | `/api/v1/visitors/{id}` | Owner(linked), Admin | Update visitor |
| DELETE | `/api/v1/visitors/{id}` | Owner(linked), Admin | Deactivate visitor |
| GET | `/api/v1/visitors/by-qr/{qrCode}` | Guard | Look up by QR code |
| GET | `/api/v1/visitors/by-id/{idNumber}` | Guard | Look up by ID number |
| POST | `/api/v1/visitors/{id}/authorize-entry` | Guard | Log visitor entry |

---

## DTOs

### CreateVisitorDto
```csharp
public record CreateVisitorDto(
    string FirstName,        // Required, max 100
    string LastName,         // Required, max 100
    string? IdType,          // Optional: CC|CE|Passport
    string? IdNumber,        // Optional, max 30
    VisitorType VisitorType, // Required: temporary|permanent
    DateTimeOffset? ExpiresAt, // Required if temporary; must be future
    CreateVisitorVehicleDto? Vehicle  // Optional
);

public record CreateVisitorVehicleDto(
    string LicensePlate,  // Required, max 20, regex validated
    string? Make,
    string? Model,
    string? Color
);
```

### VisitorDto (response)
```csharp
public record VisitorDto(
    Guid Id,
    string FullName,
    string? IdNumber,
    VisitorType VisitorType,
    DateTimeOffset? ExpiresAt,
    bool IsActive,
    bool IsExpired,
    string QrCode,      // The QR code string (frontend renders as image)
    string QrImageUrl,  // Pre-generated QR image URL from Blob
    VehicleDto? Vehicle,
    string ApartmentNumber,
    string RegisteredByName,
    DateTimeOffset CreatedAt
);
```

### GuardVisitorLookupDto
```csharp
public record GuardVisitorLookupDto(
    Guid Id,
    string FullName,
    string? PhotoUrl,
    string? IdNumber,
    bool IsAuthorized,      // true if active and not expired
    string AuthorizationStatus, // "authorized"|"expired"|"deactivated"
    string ApartmentNumber,
    string OwnerName,
    VisitorType VisitorType,
    DateTimeOffset? ExpiresAt,
    VehicleDto? Vehicle
);
```

---

## QR Code Generation

```csharp
// On visitor creation, generate unique QR code content:
var qrContent = $"REXI-VISITOR-{visitor.Id}-{Guid.NewGuid():N}";
visitor.QrCode = qrContent;

// Generate QR image using QRCoder library:
var qrGenerator = new QRCodeGenerator();
var qrData = qrGenerator.CreateQrCode(qrContent, QRCodeGenerator.ECCLevel.Q);
var pngQr = new PngByteQRCode(qrData);
var qrBytes = pngQr.GetGraphic(20);

// Upload to Azure Blob:
var qrUrl = await _blobService.UploadAsync("visitor-qr", $"{visitor.Id}.png", qrBytes);
visitor.QrImageUrl = qrUrl;
```

---

## Guard Entry Authorization

```csharp
// When guard scans QR:
public async Task<Result<GuardVisitorLookupDto>> AuthorizeEntryAsync(string qrCode, Guid guardId)
{
    var visitor = await _uow.Visitors.GetByQrCodeAsync(qrCode);
    
    if (visitor is null)
        return Result.Failure<GuardVisitorLookupDto>("QR code not found");
    
    var isAuthorized = visitor.IsActive && 
        (visitor.VisitorType == VisitorType.Permanent || 
         visitor.ExpiresAt > DateTimeOffset.UtcNow);
    
    // Log access attempt regardless of authorization
    await _uow.AccessLogs.AddAsync(new AccessLog
    {
        ComplexId = visitor.ComplexId,
        GuardId = guardId,
        VisitorId = visitor.Id,
        EntryType = "entry",
        Method = "qr",
        IsAuthorized = isAuthorized
    });
    
    // Notify owner/tenant if authorized
    if (isAuthorized)
        await _notificationService.SendVisitorArrivedAsync(visitor);
    
    return Result.Success(MapToGuardDto(visitor, isAuthorized));
}
```

---

## Background Job: Visitor Expiry

```csharp
// Jobs/VisitorExpiryJob.cs (runs daily at 00:00)
// Sets IsActive = false for all temporary visitors where ExpiresAt < NOW()
// QR code becomes invalid automatically (checked at lookup time)
```

---

## Business Rules

- `VisitorType = temporary` requires `ExpiresAt` in the future
- `VisitorType = permanent` → `ExpiresAt` is null
- Owner/tenant can only register visitors for their own apartment
- Guard sees clearly: ✅ Authorized (green) / ❌ Not authorized (red)
- Notification sent to apartment resident when visitor enters
- Visitor vehicles are searched by plate by guards

---

## Acceptance Criteria

- [ ] Creating a visitor generates unique QR code and image
- [ ] QR lookup returns authorization status clearly
- [ ] Expired QR returns "expired" status (not "not found")
- [ ] Access log entry created on each guard lookup
- [ ] Owner notified when their visitor checks in
- [ ] Expiry job correctly marks expired visitors as inactive
- [ ] Vehicle plate searchable by guard

---

## Unit Tests

- [ ] QR code uniqueness guaranteed
- [ ] `AuthorizeEntryAsync` — authorized when active + not expired
- [ ] `AuthorizeEntryAsync` — rejected when expired
- [ ] `AuthorizeEntryAsync` — rejected when deactivated
- [ ] `VisitorExpiryJob` — marks correct visitors inactive

---

## Integration Tests

- [ ] POST `/apartments/{id}/visitors` → 201 with QR code
- [ ] GET `/visitors/by-qr/{code}` → 200 authorized visitor
- [ ] GET `/visitors/by-qr/{expiredCode}` → 200 with status "expired"
- [ ] GET `/visitors/by-qr/{unknownCode}` → 404
