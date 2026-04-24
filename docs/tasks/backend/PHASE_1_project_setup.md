# TASK: Backend — Phase 1 — Project Setup

**Phase:** 1 (Foundation)  
**Module:** Infrastructure  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  

---

## Objective

Initialize the .NET 10 backend solution with Clean Architecture, configure all dependencies, and prepare for development.

---

## Solution Structure to Create

```
rexi-backend/
├── Rexi.sln
├── src/
│   ├── Rexi.API/
│   ├── Rexi.Application/
│   ├── Rexi.Domain/
│   └── Rexi.Infrastructure/
└── tests/
    ├── Rexi.UnitTests/
    ├── Rexi.IntegrationTests/
    └── Rexi.ArchTests/
```

---

## Steps

### 1. Create Solution & Projects

```bash
dotnet new sln -n Rexi
dotnet new webapi -n Rexi.API -o src/Rexi.API --use-controllers
dotnet new classlib -n Rexi.Application -o src/Rexi.Application
dotnet new classlib -n Rexi.Domain -o src/Rexi.Domain
dotnet new classlib -n Rexi.Infrastructure -o src/Rexi.Infrastructure
dotnet new xunit -n Rexi.UnitTests -o tests/Rexi.UnitTests
dotnet new xunit -n Rexi.IntegrationTests -o tests/Rexi.IntegrationTests
dotnet new xunit -n Rexi.ArchTests -o tests/Rexi.ArchTests
dotnet sln add src/**/*.csproj tests/**/*.csproj
```

### 2. Add Project References

```
Rexi.API → Rexi.Application, Rexi.Infrastructure
Rexi.Application → Rexi.Domain
Rexi.Infrastructure → Rexi.Application
Rexi.UnitTests → Rexi.Application, Rexi.Domain
Rexi.IntegrationTests → Rexi.API
Rexi.ArchTests → (all projects)
```

### 3. Install NuGet Packages

**Rexi.Application:**
```
MediatR 12+
FluentValidation.DependencyInjectionExtensions
Mapster
```

**Rexi.Infrastructure:**
```
Microsoft.EntityFrameworkCore.Design
Npgsql.EntityFrameworkCore.PostgreSQL
MediatR (for DI registration)
Hangfire.Core + Hangfire.PostgreSql
Serilog.AspNetCore
Azure.Storage.Blobs
SendGrid
FirebaseAdmin
Azure.AI.OpenAI
Microsoft.SemanticKernel
Pgvector
```

**Rexi.API:**
```
Scalar.AspNetCore (OpenAPI)
Serilog.AspNetCore
Microsoft.AspNetCore.Authentication.JwtBearer
```

**Tests:**
```
Moq / NSubstitute
Testcontainers.PostgreSql
FluentAssertions
NetArchTest.Rules (ArchTests)
```

### 4. Configure appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=rexi;Username=rexi_user;Password=secret;"
  },
  "Jwt": {
    "Secret": "<256-bit-secret>",
    "Issuer": "rexi-api",
    "Audience": "rexi-clients",
    "ExpiresInMinutes": 60,
    "RefreshExpiresInDays": 30
  },
  "Azure": {
    "BlobStorage": { "ConnectionString": "", "ContainerName": "rexi-files" },
    "OpenAI": { "Endpoint": "", "ApiKey": "", "DeploymentName": "" }
  },
  "SendGrid": { "ApiKey": "" },
  "Firebase": { "ProjectId": "" },
  "Hangfire": { "Dashboard": "/jobs" }
}
```

### 5. Configure Program.cs

Register in order:
1. Serilog logging
2. DbContext (Npgsql + EF Core)
3. MediatR (scan Application assembly)
4. FluentValidation (scan Application assembly)
5. Mapster configuration
6. Repository + UoW (via DI extension method)
7. JWT Bearer auth
8. Authorization policies
9. Hangfire server + dashboard
10. Scalar/OpenAPI
11. CORS (frontend URL)
12. Exception handling middleware
13. Tenant resolution middleware

### 6. Base Classes to Create

**Domain/Common/BaseEntity.cs:**
```csharp
public abstract class BaseEntity
{
    public Guid Id { get; init; } = Guid.NewGuid();
}

public abstract class AuditableEntity : BaseEntity
{
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public Guid? CreatedBy { get; set; }
    public Guid? UpdatedBy { get; set; }
}
```

**Application/Common/Result.cs:**
```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public string? Error { get; }
    // Factory methods: Success(T), Failure(string)
}
```

**Application/Common/Pagination.cs:**
```csharp
public record PaginationParams(int Page = 1, int PageSize = 20, string? SortBy = null, string? SortDir = "asc");
public record PagedResult<T>(IEnumerable<T> Data, int Page, int PageSize, int Total);
```

### 7. Configure Hangfire Recurring Jobs

```csharp
RecurringJob.AddOrUpdate<MonthlyFeeGenerationJob>("monthly-fees", j => j.Execute(), Cron.Monthly(1, 6));
RecurringJob.AddOrUpdate<PaymentReminderJob>("payment-reminders", j => j.Execute(), Cron.Daily(9));
RecurringJob.AddOrUpdate<VisitorExpiryJob>("visitor-expiry", j => j.Execute(), Cron.Daily());
RecurringJob.AddOrUpdate<LateInterestCalculationJob>("late-interest", j => j.Execute(), Cron.Daily(0, 1));
```

---

## Acceptance Criteria

- [ ] Solution builds without errors
- [ ] All projects reference correctly (no circular dependencies)
- [ ] API starts on `http://localhost:5000`
- [ ] Scalar/OpenAPI available at `/scalar`
- [ ] Hangfire dashboard at `/jobs`
- [ ] Architecture tests enforce Clean Architecture rules (no Infrastructure → Domain direct deps, etc.)
- [ ] Docker Compose with PostgreSQL + API running via `docker-compose up`

---

## Docker Compose (local dev)

```yaml
services:
  postgres:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: rexi
      POSTGRES_USER: rexi_user
      POSTGRES_PASSWORD: secret
    ports: ["5432:5432"]
    volumes: ["pgdata:/var/lib/postgresql/data"]

  api:
    build: .
    ports: ["5000:8080"]
    depends_on: [postgres]
    environment:
      ConnectionStrings__DefaultConnection: "Host=postgres;..."

volumes:
  pgdata:
```
