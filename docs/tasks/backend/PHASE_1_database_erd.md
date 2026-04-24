# TASK: Backend — Phase 1 — Database ERD & Migrations

**Phase:** 1 (Foundation)  
**Module:** Persistence  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 3 days  
**Depends on:** PHASE_1_project_setup  

---

## Objective

Create the full Entity Framework Core data model, configure all entity mappings, and generate the initial database migration covering all modules.

---

## PostgreSQL Extensions to Enable

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "btree_gist";
CREATE EXTENSION IF NOT EXISTS "vector";
```

Add to `Infrastructure/Persistence/RexiDbContext.cs` `OnModelCreating`:
```csharp
modelBuilder.HasPostgresExtension("uuid-ossp");
modelBuilder.HasPostgresExtension("btree_gist");
modelBuilder.HasPostgresExtension("vector");
```

---

## Schemas

Create PostgreSQL schemas using migrations:
```
identity, residential, people, finance, security, facilities, communication, documents, incidents, audit
```

```csharp
// In OnModelCreating, set default schema per entity group
modelBuilder.HasDefaultSchema("public");
```

---

## Entities to Create

### Domain Layer (Rexi.Domain/Entities/)

For each entity below, create a C# class inheriting from `AuditableEntity`:

#### Identity (use ASP.NET Core Identity)
- `AppUser : IdentityUser<Guid>` — extend with `IsActive`, `LastLoginAt`
- `AppRole : IdentityRole<Guid>` — extend with `ComplexId` (nullable)

#### Residential
- `Complex` — Id, Name, Nit, Address, City, Phone, Email, LogoUrl, AnnualBudget, LateInterestRate, PaymentDeadlineDay, IsActive
- `Tower` — Id, ComplexId, Name, Floors
- `Apartment` — Id, TowerId, Number, Floor, AreaSqm, Coefficient, ParkingType (enum), ParkingCoefficient, StorageCoefficient, IsActive
- `ParkingSlot` — Id, ComplexId, SlotNumber, ApartmentId (nullable), IsCommunal
- `StorageUnit` — Id, ComplexId, UnitNumber, ApartmentId

#### People
- `Owner` — Id, UserId, ComplexId, FirstName, LastName, IdType, IdNumber, Phone, PhotoUrl
- `OwnerApartment` — OwnerId, ApartmentId, SinceDate (join table)
- `Tenant` — Id, UserId, OwnerId, ApartmentId, FirstName, LastName, IdType, IdNumber, Phone, PhotoUrl, LeaseStart, LeaseEnd, IsActive
- `FamilyMember` — Id, OwnerId (nullable), TenantId (nullable), FirstName, LastName, Relationship, BirthDate
- `FaceEnrollment` — Id, OwnerId (nullable), TenantId (nullable), FaceId, IsActive

#### Finance
- `AdministrationFee` — Id, ApartmentId, Year, Month, BaseAmount, ParkingAmount, StorageAmount, FinesAmount, DueDate, Status (enum), PdfUrl
- `Payment` — Id, FeeId, Amount, PaymentDate, PaymentMethod, Reference, Notes, RecordedBy
- `LateInterest` — Id, FeeId, DaysOverdue, Rate, Amount

#### Security
- `Visitor` — Id, ComplexId, RegisteredBy, ApartmentId, FirstName, LastName, IdType, IdNumber, PhotoUrl, VisitorType (enum), ExpiresAt, QrCode, IsActive
- `Vehicle` — Id, ComplexId, LicensePlate, Make, Model, Color, Year, OwnerId (nullable), TenantId (nullable), VisitorId (nullable), IsAuthorized
- `AccessLog` — Id, ComplexId, GuardId, OwnerId (nullable), TenantId (nullable), VisitorId (nullable), VehicleId (nullable), EntryType, Method, Timestamp, Notes
- `Fine` — Id, ComplexId, ApartmentId, IssuedBy, InfractionType, Description, RuleReference, Amount, EvidenceUrls (string[]), Status (enum), ApprovedBy, AppealReason, AppealFiledAt, ResolvedAt, FeeId (nullable)
- `Package` — Id, ComplexId, ApartmentId, ReceivedBy, Sender, Description, PhotoUrl, ReceivedAt, PickedUpAt, Status

#### Facilities
- `CommonArea` — Id, ComplexId, Name, Description, Capacity, Fee, PhotoUrls, OpenTime, CloseTime, AdvanceBookingHours, RequiresApproval, BlocksOnDebt, IsActive
- `Booking` — Id, CommonAreaId, BookedBy, ApartmentId, StartTime, EndTime, Status (enum), Notes

#### Communication
- `Notification` — Id, ComplexId, UserId, Type, Title, Body, Data (JSONB → Dictionary<string,object>), IsRead, SentEmail, SentPush
- `AnnouncementTemplate` — Id, ComplexId, Name, Subject, Body

#### Documents
- `ComplexDocument` — Id, ComplexId, Name, Description, FileUrl, FileType, Version, UploadedBy
- `DocumentChunk` — Id, DocumentId, ChunkIndex, Content, Embedding (pgvector)

#### Incidents
- `IncidentReport` — Id, ComplexId, ReportedBy, Category, Description, PhotoUrls, AiPriority, ManualPriority, Status, AssignedTo
- `EmergencyReport` — Id, ComplexId, ReportedBy, EmergencyType, Conversation (JSONB), AiReport

---

## EF Core Configuration (Fluent API)

Create one `IEntityTypeConfiguration<T>` per entity in `Infrastructure/Persistence/Configurations/`.

**Example — ApartmentConfiguration.cs:**
```csharp
public class ApartmentConfiguration : IEntityTypeConfiguration<Apartment>
{
    public void Configure(EntityTypeBuilder<Apartment> builder)
    {
        builder.ToTable("apartments", "residential");
        builder.HasKey(x => x.Id);
        builder.Property(x => x.Number).HasMaxLength(20).IsRequired();
        builder.Property(x => x.Coefficient).HasPrecision(10, 8).IsRequired();
        builder.Property(x => x.ParkingType)
               .HasConversion<string>()
               .HasMaxLength(20);
        builder.HasIndex(x => new { x.TowerId, x.Number }).IsUnique();
        builder.HasOne(x => x.Tower)
               .WithMany(x => x.Apartments)
               .HasForeignKey(x => x.TowerId);
    }
}
```

---

## Indexes to Create

```csharp
// In configurations or OnModelCreating:
// vehicles: (complex_id, license_plate) UNIQUE
// visitors: (qr_code) UNIQUE
// administration_fees: (apartment_id, year, month) UNIQUE
// notifications: (user_id, is_read, created_at DESC)
// access_logs: (complex_id, timestamp DESC)
// document_chunks: ivfflat on embedding (pgvector)
```

---

## Booking Overlap Exclusion Constraint

```csharp
// In BookingConfiguration
builder.HasAnnotation("SqlServer:ExcludeConstraint", 
    "EXCLUDE USING gist (common_area_id WITH =, tstzrange(start_time, end_time) WITH &&) WHERE (status != 'cancelled')");
// For PostgreSQL via raw SQL in migration
```

---

## Audit Trigger

Create a database trigger in a migration (raw SQL) for the `audit.audit_log` table:

```sql
CREATE OR REPLACE FUNCTION audit.record_change()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit.audit_log(table_name, record_id, action, old_values, new_values)
  VALUES (TG_TABLE_NAME, COALESCE(NEW.id, OLD.id), TG_OP,
          row_to_json(OLD), row_to_json(NEW));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

## Migration Commands

```bash
# Add initial migration
dotnet ef migrations add InitialCreate --project src/Rexi.Infrastructure --startup-project src/Rexi.API

# Apply migration
dotnet ef database update --project src/Rexi.Infrastructure --startup-project src/Rexi.API
```

---

## Seed Data (for development)

Create `DataSeeder.cs` in Infrastructure that seeds:
- 1 complex: "Conjunto Residencial Las Palmas"
- 2 towers: Torre A (10 floors, 4 apts/floor), Torre B (8 floors, 3 apts/floor)
- 40 apartments with realistic coefficients (sum = 1.0)
- 1 super admin user: admin@rexi.app / Admin123!
- 1 admin user: laspalmas@rexi.app / Admin123!
- 2 owners with apartments
- 1 tenant
- 2 common areas (Gym - free, Social Hall - $50,000)

---

## Acceptance Criteria

- [ ] All entities created with correct properties and types
- [ ] All EF Core configurations applied (schema, table names, indexes, constraints)
- [ ] Migration generated and applies cleanly to empty database
- [ ] Seed data loads without errors
- [ ] All relationships navigable (EF include works)
- [ ] pgvector index created for document_chunks
- [ ] Booking overlap exclusion constraint works (test with conflicting bookings)

---

## Unit Tests

- [ ] `ApartmentCoefficientsTest` — coefficients sum to 1.0 per complex
- [ ] `VisitorExpiryTest` — expired visitors cannot be used for entry
- [ ] `FeeCalculationTest` — formula correctness
