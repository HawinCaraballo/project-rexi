# Rexi — Database Architecture Proposal

**Engine:** PostgreSQL 16+  
**ORM:** Entity Framework Core 10  
**Strategy:** Schema-per-module (bounded context isolation, microservices-ready)  

---

## 1. Schema Organization

Each backend module has its own PostgreSQL schema, providing:
- Clear bounded context boundaries
- Easy extraction into a separate database if scaling to microservices
- Simplified permission management

| Schema | Description |
|--------|-------------|
| `public` | Platform-level: tenants (complexes), users |
| `identity` | Authentication: users, roles, refresh tokens |
| `residential` | Complexes, towers, apartments |
| `people` | Owners, tenants, family members |
| `finance` | Fees, payments, late interest |
| `security` | Visitors, vehicles, access logs, fines |
| `facilities` | Common areas, bookings |
| `communication` | Notifications, notification templates |
| `documents` | Uploaded documents, document chunks (RAG) |
| `incidents` | Issue reports, emergency reports |

---

## 2. Entity-Relationship Diagram (ERD)

### 2.1 Core Entities Overview

```
Complex ──< Tower ──< Apartment ──< AdministrationFee ──< Payment
   │                    │
   │                    ├──< Owner ──< Tenant
   │                    ├── ParkingSlot (assigned/communal)
   │                    └── StorageUnit
   │
   ├──< CommonArea ──< Booking
   ├──< Document
   ├──< Notification
   └──< Budget
```

---

## 3. Table Definitions

### Schema: `identity`

```sql
-- identity.users
CREATE TABLE identity.users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email           VARCHAR(255) NOT NULL UNIQUE,
    password_hash   VARCHAR(512) NOT NULL,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    is_email_confirmed BOOLEAN NOT NULL DEFAULT FALSE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- identity.user_roles
CREATE TABLE identity.user_roles (
    user_id     UUID NOT NULL REFERENCES identity.users(id),
    role        VARCHAR(50) NOT NULL,  -- SuperAdmin|Admin|Owner|Tenant|Guard
    complex_id  UUID,  -- NULL for SuperAdmin
    PRIMARY KEY (user_id, role, COALESCE(complex_id, '00000000-0000-0000-0000-000000000000'::UUID))
);

-- identity.refresh_tokens
CREATE TABLE identity.refresh_tokens (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES identity.users(id),
    token       VARCHAR(512) NOT NULL UNIQUE,
    expires_at  TIMESTAMPTZ NOT NULL,
    revoked_at  TIMESTAMPTZ,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### Schema: `residential`

```sql
-- residential.complexes
CREATE TABLE residential.complexes (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(200) NOT NULL,
    nit             VARCHAR(20),
    address         TEXT NOT NULL,
    city            VARCHAR(100) NOT NULL,
    phone           VARCHAR(20),
    email           VARCHAR(255),
    logo_url        TEXT,
    annual_budget   NUMERIC(18,2) NOT NULL DEFAULT 0,
    late_interest_rate NUMERIC(5,4) NOT NULL DEFAULT 0.015,  -- 1.5% daily
    payment_deadline_day INT NOT NULL DEFAULT 10,  -- day of month
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- residential.towers
CREATE TABLE residential.towers (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id  UUID NOT NULL REFERENCES residential.complexes(id),
    name        VARCHAR(100) NOT NULL,
    floors      INT NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (complex_id, name)
);

-- residential.apartments
CREATE TABLE residential.apartments (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tower_id                UUID NOT NULL REFERENCES residential.towers(id),
    number                  VARCHAR(20) NOT NULL,
    floor                   INT NOT NULL,
    area_sqm                NUMERIC(8,2),
    coefficient             NUMERIC(10,8) NOT NULL,  -- horizontal property coefficient
    parking_type            VARCHAR(20) NOT NULL DEFAULT 'none',  -- assigned|communal|none
    parking_coefficient     NUMERIC(10,8) NOT NULL DEFAULT 0,
    storage_coefficient     NUMERIC(10,8) NOT NULL DEFAULT 0,
    is_active               BOOLEAN NOT NULL DEFAULT TRUE,
    created_at              TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at              TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (tower_id, number)
);

-- residential.parking_slots (for assigned parking)
CREATE TABLE residential.parking_slots (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    slot_number     VARCHAR(20) NOT NULL,
    apartment_id    UUID REFERENCES residential.apartments(id),  -- NULL if communal
    is_communal     BOOLEAN NOT NULL DEFAULT FALSE,
    UNIQUE (complex_id, slot_number)
);

-- residential.storage_units
CREATE TABLE residential.storage_units (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    unit_number     VARCHAR(20) NOT NULL,
    apartment_id    UUID REFERENCES residential.apartments(id),
    UNIQUE (complex_id, unit_number)
);
```

---

### Schema: `people`

```sql
-- people.owners
CREATE TABLE people.owners (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES identity.users(id),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    first_name      VARCHAR(100) NOT NULL,
    last_name       VARCHAR(100) NOT NULL,
    id_type         VARCHAR(20) NOT NULL,  -- CC|CE|NIT|Passport
    id_number       VARCHAR(30) NOT NULL,
    phone           VARCHAR(20),
    photo_url       TEXT,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (complex_id, id_number)
);

-- people.owner_apartments (owner can have multiple apartments)
CREATE TABLE people.owner_apartments (
    owner_id        UUID NOT NULL REFERENCES people.owners(id),
    apartment_id    UUID NOT NULL REFERENCES residential.apartments(id),
    since_date      DATE NOT NULL DEFAULT CURRENT_DATE,
    PRIMARY KEY (owner_id, apartment_id)
);

-- people.tenants
CREATE TABLE people.tenants (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES identity.users(id),
    owner_id        UUID NOT NULL REFERENCES people.owners(id),
    apartment_id    UUID NOT NULL REFERENCES residential.apartments(id),
    first_name      VARCHAR(100) NOT NULL,
    last_name       VARCHAR(100) NOT NULL,
    id_type         VARCHAR(20) NOT NULL,
    id_number       VARCHAR(30) NOT NULL,
    phone           VARCHAR(20),
    photo_url       TEXT,
    lease_start     DATE,
    lease_end       DATE,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- people.family_members
CREATE TABLE people.family_members (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id        UUID REFERENCES people.owners(id),
    tenant_id       UUID REFERENCES people.tenants(id),
    first_name      VARCHAR(100) NOT NULL,
    last_name       VARCHAR(100) NOT NULL,
    relationship    VARCHAR(50) NOT NULL,  -- spouse|child|parent|other
    birth_date      DATE,
    CHECK (owner_id IS NOT NULL OR tenant_id IS NOT NULL)
);

-- people.face_enrollments (facial recognition - optional)
CREATE TABLE people.face_enrollments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id        UUID REFERENCES people.owners(id),
    tenant_id       UUID REFERENCES people.tenants(id),
    face_id         VARCHAR(255) NOT NULL,  -- Azure Face API face ID
    enrolled_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    CHECK (owner_id IS NOT NULL OR tenant_id IS NOT NULL)
);
```

---

### Schema: `finance`

```sql
-- finance.administration_fees
CREATE TABLE finance.administration_fees (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    apartment_id    UUID NOT NULL REFERENCES residential.apartments(id),
    year            INT NOT NULL,
    month           INT NOT NULL CHECK (month BETWEEN 1 AND 12),
    base_amount     NUMERIC(18,2) NOT NULL,
    parking_amount  NUMERIC(18,2) NOT NULL DEFAULT 0,
    storage_amount  NUMERIC(18,2) NOT NULL DEFAULT 0,
    fines_amount    NUMERIC(18,2) NOT NULL DEFAULT 0,
    total_amount    NUMERIC(18,2) GENERATED ALWAYS AS (base_amount + parking_amount + storage_amount + fines_amount) STORED,
    due_date        DATE NOT NULL,
    status          VARCHAR(20) NOT NULL DEFAULT 'pending',  -- pending|paid|overdue|partial
    pdf_url         TEXT,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (apartment_id, year, month)
);

-- finance.payments
CREATE TABLE finance.payments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    fee_id          UUID NOT NULL REFERENCES finance.administration_fees(id),
    amount          NUMERIC(18,2) NOT NULL,
    payment_date    DATE NOT NULL,
    payment_method  VARCHAR(50) NOT NULL,  -- cash|transfer|card|online
    reference       VARCHAR(100),
    notes           TEXT,
    recorded_by     UUID NOT NULL REFERENCES identity.users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- finance.late_interests
CREATE TABLE finance.late_interests (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    fee_id          UUID NOT NULL REFERENCES finance.administration_fees(id),
    days_overdue    INT NOT NULL,
    rate            NUMERIC(5,4) NOT NULL,
    amount          NUMERIC(18,2) NOT NULL,
    calculated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### Schema: `security`

```sql
-- security.visitors
CREATE TABLE security.visitors (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    registered_by   UUID NOT NULL REFERENCES identity.users(id),
    apartment_id    UUID NOT NULL REFERENCES residential.apartments(id),
    first_name      VARCHAR(100) NOT NULL,
    last_name       VARCHAR(100) NOT NULL,
    id_type         VARCHAR(20),
    id_number       VARCHAR(30),
    photo_url       TEXT,
    visitor_type    VARCHAR(20) NOT NULL DEFAULT 'temporary',  -- temporary|permanent
    expires_at      TIMESTAMPTZ,  -- NULL for permanent
    qr_code         VARCHAR(512) NOT NULL UNIQUE,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- security.vehicles
CREATE TABLE security.vehicles (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    license_plate   VARCHAR(20) NOT NULL,
    make            VARCHAR(50),
    model           VARCHAR(50),
    color           VARCHAR(30),
    year            INT,
    owner_id        UUID REFERENCES people.owners(id),
    tenant_id       UUID REFERENCES people.tenants(id),
    visitor_id      UUID REFERENCES security.visitors(id),
    is_authorized   BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (complex_id, license_plate)
);

-- security.access_logs
CREATE TABLE security.access_logs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    guard_id        UUID NOT NULL REFERENCES identity.users(id),
    owner_id        UUID REFERENCES people.owners(id),
    tenant_id       UUID REFERENCES people.tenants(id),
    visitor_id      UUID REFERENCES security.visitors(id),
    vehicle_id      UUID REFERENCES security.vehicles(id),
    entry_type      VARCHAR(20) NOT NULL,  -- entry|exit
    method          VARCHAR(30) NOT NULL,  -- qr|plate|facial|manual
    timestamp       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    notes           TEXT
);

-- security.fines
CREATE TABLE security.fines (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    apartment_id    UUID NOT NULL REFERENCES residential.apartments(id),
    issued_by       UUID NOT NULL REFERENCES identity.users(id),  -- guard
    infraction_type VARCHAR(100) NOT NULL,
    description     TEXT NOT NULL,
    rule_reference  VARCHAR(200),
    amount          NUMERIC(18,2) NOT NULL,
    evidence_urls   TEXT[],
    status          VARCHAR(20) NOT NULL DEFAULT 'pending_approval',
    -- pending_approval|approved|rejected|appealed|appeal_approved|appeal_rejected
    approved_by     UUID REFERENCES identity.users(id),
    appeal_reason   TEXT,
    appeal_filed_at TIMESTAMPTZ,
    resolved_at     TIMESTAMPTZ,
    fee_id          UUID REFERENCES finance.administration_fees(id),  -- linked when charged
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- security.packages
CREATE TABLE security.packages (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    apartment_id    UUID NOT NULL REFERENCES residential.apartments(id),
    received_by     UUID NOT NULL REFERENCES identity.users(id),
    sender          VARCHAR(100),
    description     TEXT,
    photo_url       TEXT,
    received_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    picked_up_at    TIMESTAMPTZ,
    status          VARCHAR(20) NOT NULL DEFAULT 'waiting'  -- waiting|picked_up
);
```

---

### Schema: `facilities`

```sql
-- facilities.common_areas
CREATE TABLE facilities.common_areas (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    name            VARCHAR(100) NOT NULL,
    description     TEXT,
    capacity        INT,
    fee             NUMERIC(10,2) NOT NULL DEFAULT 0,
    photo_urls      TEXT[],
    open_time       TIME NOT NULL DEFAULT '06:00',
    close_time      TIME NOT NULL DEFAULT '22:00',
    advance_booking_hours INT NOT NULL DEFAULT 24,
    requires_approval BOOLEAN NOT NULL DEFAULT FALSE,
    blocks_on_debt  BOOLEAN NOT NULL DEFAULT TRUE,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- facilities.bookings
CREATE TABLE facilities.bookings (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    common_area_id  UUID NOT NULL REFERENCES facilities.common_areas(id),
    booked_by       UUID NOT NULL REFERENCES identity.users(id),
    apartment_id    UUID NOT NULL REFERENCES residential.apartments(id),
    start_time      TIMESTAMPTZ NOT NULL,
    end_time        TIMESTAMPTZ NOT NULL,
    status          VARCHAR(20) NOT NULL DEFAULT 'confirmed',  -- confirmed|pending|cancelled
    notes           TEXT,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    -- Prevent double-booking
    EXCLUDE USING gist (
        common_area_id WITH =,
        tstzrange(start_time, end_time) WITH &&
    ) WHERE (status != 'cancelled')
);
```

---

### Schema: `communication`

```sql
-- communication.notifications
CREATE TABLE communication.notifications (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    user_id         UUID NOT NULL REFERENCES identity.users(id),
    type            VARCHAR(50) NOT NULL,
    title           VARCHAR(200) NOT NULL,
    body            TEXT NOT NULL,
    data            JSONB,
    is_read         BOOLEAN NOT NULL DEFAULT FALSE,
    sent_email      BOOLEAN NOT NULL DEFAULT FALSE,
    sent_push       BOOLEAN NOT NULL DEFAULT FALSE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- communication.announcement_templates
CREATE TABLE communication.announcement_templates (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id  UUID NOT NULL REFERENCES residential.complexes(id),
    name        VARCHAR(100) NOT NULL,
    subject     VARCHAR(200) NOT NULL,
    body        TEXT NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### Schema: `documents`

```sql
-- documents.complex_documents
CREATE TABLE documents.complex_documents (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    name            VARCHAR(200) NOT NULL,
    description     TEXT,
    file_url        TEXT NOT NULL,
    file_type       VARCHAR(20) NOT NULL,  -- pdf|word|other
    version         INT NOT NULL DEFAULT 1,
    uploaded_by     UUID NOT NULL REFERENCES identity.users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- documents.document_chunks (for RAG/AI bot)
CREATE TABLE documents.document_chunks (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id     UUID NOT NULL REFERENCES documents.complex_documents(id),
    chunk_index     INT NOT NULL,
    content         TEXT NOT NULL,
    embedding       VECTOR(1536),  -- pgvector extension
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### Schema: `incidents`

```sql
-- incidents.reports
CREATE TABLE incidents.reports (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    reported_by     UUID NOT NULL REFERENCES identity.users(id),
    category        VARCHAR(50) NOT NULL,  -- maintenance|security|noise|other
    description     TEXT NOT NULL,
    photo_urls      TEXT[],
    ai_priority     VARCHAR(20),  -- critical|high|medium|low
    manual_priority VARCHAR(20),
    status          VARCHAR(20) NOT NULL DEFAULT 'open',  -- open|in_progress|resolved
    assigned_to     UUID REFERENCES identity.users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- incidents.emergency_reports
CREATE TABLE incidents.emergency_reports (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    complex_id      UUID NOT NULL REFERENCES residential.complexes(id),
    reported_by     UUID NOT NULL REFERENCES identity.users(id),
    emergency_type  VARCHAR(50) NOT NULL,  -- fire|police|ambulance|power|water|gas
    conversation    JSONB NOT NULL,
    ai_report       TEXT NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 4. Indexes

```sql
-- High-frequency lookups
CREATE INDEX idx_vehicles_plate ON security.vehicles (complex_id, license_plate);
CREATE INDEX idx_visitors_qr ON security.visitors (qr_code);
CREATE INDEX idx_fees_apartment_period ON finance.administration_fees (apartment_id, year, month);
CREATE INDEX idx_notifications_user ON communication.notifications (user_id, is_read, created_at DESC);
CREATE INDEX idx_access_logs_complex_ts ON security.access_logs (complex_id, timestamp DESC);

-- Vector search for AI bot
CREATE INDEX idx_doc_chunks_embedding ON documents.document_chunks 
    USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

---

## 5. Required PostgreSQL Extensions

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";  -- UUID generation
CREATE EXTENSION IF NOT EXISTS "pgcrypto";   -- Hashing
CREATE EXTENSION IF NOT EXISTS "btree_gist"; -- Exclude constraint for booking overlap
CREATE EXTENSION IF NOT EXISTS "vector";     -- pgvector for AI embeddings
```

---

## 6. Audit Trail

All mutable tables inherit from an audit pattern:

```sql
-- Audit trigger applied to all major tables
CREATE TABLE audit.audit_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    table_name      TEXT NOT NULL,
    record_id       UUID NOT NULL,
    action          VARCHAR(10) NOT NULL,  -- INSERT|UPDATE|DELETE
    old_values      JSONB,
    new_values      JSONB,
    changed_by      UUID REFERENCES identity.users(id),
    changed_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 7. Microservices Migration Path

When extracting a module to a microservice, the schema becomes its own database:

```
Phase 1 (Monolith):  Single PostgreSQL DB, multiple schemas
Phase 2 (Hybrid):    Notifications → own DB; AI → own DB
Phase 3 (Micro):     Each bounded context has its own DB
                     Cross-service data via events (Azure Service Bus)
```
