# Rexi — Backend Architecture Proposal

**Stack:** .NET 10 + ASP.NET Core + Clean Architecture  
**Pattern:** Modular Monolith → Microservices-ready  
**ORM:** Entity Framework Core 10  
**Patterns:** CQRS, MediatR, Repository, Unit of Work  

---

## 1. Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Runtime | .NET | 10 |
| Web API | ASP.NET Core Web API | 10 |
| ORM | Entity Framework Core | 10 |
| CQRS / Mediator | MediatR | 12+ |
| Validation | FluentValidation | 11+ |
| Auth | ASP.NET Core Identity + JWT Bearer | - |
| Logging | Serilog → Azure Application Insights | - |
| Mapping | Mapster | 7+ |
| API Documentation | Scalar / Swagger (OpenAPI 3) | - |
| Background Jobs | Hangfire (PostgreSQL backend) | - |
| File Storage | Azure Blob Storage SDK | - |
| Email | SendGrid SDK | - |
| Push Notifications | Firebase Admin SDK | - |
| AI / LLM | Azure OpenAI SDK / Semantic Kernel | - |
| Caching | IMemoryCache + Redis (Azure Cache) | - |
| Testing | xUnit + Moq + Testcontainers | - |
| Containerization | Docker | - |

---

## 2. Clean Architecture Layers

```
┌──────────────────────────────────────────────────────────┐
│                     Presentation Layer                    │
│           ASP.NET Core Web API (Controllers)              │
│            Middleware, Filters, Model Binding             │
├──────────────────────────────────────────────────────────┤
│                   Application Layer                       │
│     Commands, Queries (MediatR), DTOs, Validators        │
│               Application Services, Mappers              │
├──────────────────────────────────────────────────────────┤
│                     Domain Layer                          │
│       Entities, Value Objects, Aggregates, Events        │
│           Domain Services, Interfaces, Enums             │
├──────────────────────────────────────────────────────────┤
│                  Infrastructure Layer                     │
│   EF Core DbContext, Repositories, Unit of Work           │
│   External Services (Email, Blob, AI, Push, Payments)    │
└──────────────────────────────────────────────────────────┘
```

---

## 3. Project Structure

```
rexi-backend/
├── src/
│   ├── Rexi.API/                        # Presentation Layer
│   │   ├── Controllers/
│   │   │   ├── ComplexesController.cs
│   │   │   ├── OwnersController.cs
│   │   │   ├── TenantsController.cs
│   │   │   ├── ApartmentsController.cs
│   │   │   ├── FeesController.cs
│   │   │   ├── PaymentsController.cs
│   │   │   ├── VisitorsController.cs
│   │   │   ├── VehiclesController.cs
│   │   │   ├── CommonAreasController.cs
│   │   │   ├── FinesController.cs
│   │   │   ├── NotificationsController.cs
│   │   │   ├── DocumentsController.cs
│   │   │   ├── ReportsController.cs
│   │   │   └── AiController.cs
│   │   ├── Middleware/
│   │   │   ├── ExceptionHandlingMiddleware.cs
│   │   │   └── TenantResolutionMiddleware.cs
│   │   ├── Filters/
│   │   └── Program.cs
│   │
│   ├── Rexi.Application/                # Application Layer
│   │   ├── Common/
│   │   │   ├── Behaviors/
│   │   │   │   ├── ValidationBehavior.cs
│   │   │   │   ├── LoggingBehavior.cs
│   │   │   │   └── AuditBehavior.cs
│   │   │   ├── Interfaces/
│   │   │   │   ├── ICurrentUser.cs
│   │   │   │   ├── IEmailService.cs
│   │   │   │   ├── IStorageService.cs
│   │   │   │   ├── IPushNotificationService.cs
│   │   │   │   └── IAiService.cs
│   │   │   ├── Pagination.cs
│   │   │   └── Result.cs
│   │   ├── Complexes/
│   │   │   ├── Commands/
│   │   │   │   ├── CreateComplex/
│   │   │   │   ├── UpdateComplex/
│   │   │   │   └── DeleteComplex/
│   │   │   └── Queries/
│   │   │       ├── GetComplexById/
│   │   │       └── ListComplexes/
│   │   ├── Owners/
│   │   ├── Tenants/
│   │   ├── Apartments/
│   │   ├── Fees/
│   │   ├── Payments/
│   │   ├── Visitors/
│   │   ├── Vehicles/
│   │   ├── CommonAreas/
│   │   ├── Fines/
│   │   ├── Notifications/
│   │   └── Documents/
│   │
│   ├── Rexi.Domain/                     # Domain Layer
│   │   ├── Common/
│   │   │   ├── BaseEntity.cs
│   │   │   ├── AuditableEntity.cs
│   │   │   └── IDomainEvent.cs
│   │   ├── Entities/
│   │   │   ├── Complex.cs
│   │   │   ├── Tower.cs
│   │   │   ├── Apartment.cs
│   │   │   ├── Owner.cs
│   │   │   ├── Tenant.cs
│   │   │   ├── Visitor.cs
│   │   │   ├── Vehicle.cs
│   │   │   ├── AdministrationFee.cs
│   │   │   ├── Payment.cs
│   │   │   ├── CommonArea.cs
│   │   │   ├── Booking.cs
│   │   │   ├── Fine.cs
│   │   │   ├── Notification.cs
│   │   │   └── Document.cs
│   │   ├── ValueObjects/
│   │   │   ├── Money.cs
│   │   │   ├── LicensePlate.cs
│   │   │   ├── Address.cs
│   │   │   └── Coefficient.cs
│   │   ├── Enums/
│   │   │   ├── Role.cs
│   │   │   ├── ParkingType.cs
│   │   │   ├── FineStatus.cs
│   │   │   ├── PaymentStatus.cs
│   │   │   └── VisitorType.cs
│   │   └── Events/
│   │       ├── PaymentReceivedEvent.cs
│   │       ├── FineIssuedEvent.cs
│   │       └── PackageArrivedEvent.cs
│   │
│   └── Rexi.Infrastructure/             # Infrastructure Layer
│       ├── Persistence/
│       │   ├── RexiDbContext.cs
│       │   ├── Configurations/          # EF Core Fluent API configs
│       │   ├── Repositories/
│       │   │   ├── ComplexRepository.cs
│       │   │   ├── OwnerRepository.cs
│       │   │   └── ... (per entity)
│       │   ├── UnitOfWork.cs
│       │   └── Migrations/
│       ├── Services/
│       │   ├── EmailService.cs          # SendGrid
│       │   ├── BlobStorageService.cs    # Azure Blob
│       │   ├── PushNotificationService.cs # Firebase
│       │   ├── FeeCalculationService.cs
│       │   ├── PdfGenerationService.cs
│       │   ├── AiDocumentService.cs     # Azure OpenAI / Semantic Kernel
│       │   └── FacialRecognitionService.cs # Azure Face API
│       ├── Jobs/                        # Hangfire background jobs
│       │   ├── MonthlyFeeGenerationJob.cs
│       │   ├── PaymentReminderJob.cs
│       │   └── VisitorExpiryJob.cs
│       └── Identity/
│           ├── AppUser.cs
│           └── TokenService.cs
│
└── tests/
    ├── Rexi.UnitTests/
    ├── Rexi.IntegrationTests/
    └── Rexi.ArchTests/                  # Architecture guard tests
```

---

## 4. CQRS Pattern with MediatR

### Command Example

```csharp
// Application/Fees/Commands/GenerateMonthlyFees/GenerateMonthlyFeesCommand.cs
public record GenerateMonthlyFeesCommand(Guid ComplexId, int Year, int Month) 
    : IRequest<Result<int>>;

public class GenerateMonthlyFeesCommandHandler 
    : IRequestHandler<GenerateMonthlyFeesCommand, Result<int>>
{
    private readonly IUnitOfWork _uow;
    private readonly IFeeCalculationService _feeService;

    public async Task<Result<int>> Handle(
        GenerateMonthlyFeesCommand request, 
        CancellationToken ct)
    {
        var complex = await _uow.Complexes.GetWithApartmentsAsync(request.ComplexId, ct);
        if (complex is null) return Result.Failure<int>("Complex not found");

        var fees = _feeService.CalculateAll(complex, request.Year, request.Month);
        await _uow.AdministrationFees.AddRangeAsync(fees, ct);
        await _uow.SaveChangesAsync(ct);

        return Result.Success(fees.Count);
    }
}
```

### Query Example

```csharp
// Application/Owners/Queries/GetOwnerById/GetOwnerByIdQuery.cs
public record GetOwnerByIdQuery(Guid OwnerId) : IRequest<Result<OwnerDto>>;

public class GetOwnerByIdQueryHandler 
    : IRequestHandler<GetOwnerByIdQuery, Result<OwnerDto>>
{
    private readonly IOwnerRepository _repo;
    private readonly IMapper _mapper;

    public async Task<Result<OwnerDto>> Handle(
        GetOwnerByIdQuery request, 
        CancellationToken ct)
    {
        var owner = await _repo.GetByIdAsync(request.OwnerId, ct);
        return owner is null
            ? Result.Failure<OwnerDto>("Owner not found")
            : Result.Success(_mapper.Map<OwnerDto>(owner));
    }
}
```

---

## 5. Multi-Tenancy

Multi-tenancy is implemented at the **schema level** (PostgreSQL schemas per complex) for data isolation, with a `TenantResolutionMiddleware` that extracts the `ComplexId` from:
1. JWT claim (`complex_id`)
2. Subdomain (`complex-slug.rexi.app`)
3. Request header (`X-Complex-Id`)

```csharp
// Middleware/TenantResolutionMiddleware.cs
public class TenantResolutionMiddleware(RequestDelegate next)
{
    public async Task InvokeAsync(HttpContext context, ITenantContext tenantContext)
    {
        var complexId = ExtractComplexId(context);
        tenantContext.SetComplexId(complexId);
        await next(context);
    }
}
```

---

## 6. Repository & Unit of Work

```csharp
// Domain interfaces
public interface IRepository<T> where T : BaseEntity
{
    Task<T?> GetByIdAsync(Guid id, CancellationToken ct = default);
    Task<PagedResult<T>> ListAsync(PaginationParams pagination, CancellationToken ct = default);
    Task AddAsync(T entity, CancellationToken ct = default);
    void Update(T entity);
    void Delete(T entity);
}

public interface IUnitOfWork
{
    IComplexRepository Complexes { get; }
    IOwnerRepository Owners { get; }
    ITenantRepository Tenants { get; }
    IApartmentRepository Apartments { get; }
    IAdministrationFeeRepository AdministrationFees { get; }
    IPaymentRepository Payments { get; }
    IVisitorRepository Visitors { get; }
    IVehicleRepository Vehicles { get; }
    IFineRepository Fines { get; }
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}
```

---

## 7. Background Jobs (Hangfire)

| Job | Schedule | Description |
|-----|----------|-------------|
| `MonthlyFeeGenerationJob` | 1st of month, 06:00 | Generate fees for all active complexes |
| `PaymentReminderJob` | Daily 09:00 | Send reminders: 7d, 3d, 0d before deadline |
| `VisitorExpiryJob` | Daily 00:00 | Mark expired visitors as inactive |
| `LateInterestCalculationJob` | Daily 00:01 | Add late interest to overdue fees |
| `DocumentIndexingJob` | On upload | Index documents for AI bot RAG |

---

## 8. API Design Conventions

### URL Pattern
```
GET    /api/v1/complexes
POST   /api/v1/complexes
GET    /api/v1/complexes/{id}
PUT    /api/v1/complexes/{id}
DELETE /api/v1/complexes/{id}

GET    /api/v1/complexes/{id}/towers
GET    /api/v1/complexes/{id}/apartments
POST   /api/v1/complexes/{id}/fees/generate
```

### Pagination Response
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### Error Response
```json
{
  "type": "https://tools.ietf.org/html/rfc7807",
  "title": "Validation Error",
  "status": 422,
  "errors": {
    "coefficient": ["Coefficient must be between 0 and 1"],
    "email": ["Email already registered"]
  }
}
```

---

## 9. Authentication & Authorization

- **JWT Bearer tokens** with refresh token rotation
- **Roles:** SuperAdmin, Admin, Owner, Tenant, Guard
- **Policy-based auth:**

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanManageComplex", policy =>
        policy.RequireRole("SuperAdmin", "Admin"));
    options.AddPolicy("CanIssueFines", policy =>
        policy.RequireRole("Guard"));
    options.AddPolicy("CanApproveFines", policy =>
        policy.RequireRole("Admin"));
});
```

---

## 10. Docker

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore
RUN dotnet publish src/Rexi.API/Rexi.API.csproj -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "Rexi.API.dll"]
```

---

## 11. Scalability Path

```
Phase 1: Modular Monolith (single deployable unit)
         → All modules in one solution, separate namespaces/schemas

Phase 2: Extract high-load modules
         → Notifications service (high fan-out)
         → AI service (GPU-bound, separate scaling)

Phase 3: Full microservices (if needed)
         → Independent deploy per bounded context
         → API Gateway (Azure API Management)
         → Service Mesh (Dapr)
```
