# Rexi Platform — Documentation Index

> **Rexi** is a SaaS platform for managing residential complexes (conjuntos residenciales). It covers owners, tenants, guards, and administrators with features ranging from fee calculation and visitor management to AI-assisted reporting and facial recognition.

---

## 📁 Documentation Structure

```
docs/
├── README.md                        ← This file (index)
├── PRD.md                           ← Product Requirements Document
├── architecture/
│   ├── frontend.md                  ← Frontend architecture proposal
│   ├── backend.md                   ← Backend architecture proposal
│   └── database.md                  ← Database architecture proposal
├── workflows/
│   ├── admin_workflows.md           ← Administrator user flows
│   ├── owner_workflows.md           ← Owner user flows
│   ├── tenant_workflows.md          ← Tenant user flows
│   └── guard_workflows.md           ← Guard user flows
└── tasks/
    ├── backend/
    │   ├── PHASE_1_project_setup.md
    │   ├── PHASE_1_database_erd.md
    │   ├── PHASE_1_auth_module.md
    │   ├── PHASE_2_complex_module.md
    │   ├── PHASE_2_owner_tenant_module.md
    │   ├── PHASE_2_fee_calculation_module.md
    │   ├── PHASE_2_payment_module.md
    │   ├── PHASE_3_visitor_module.md
    │   ├── PHASE_3_notification_module.md
    │   ├── PHASE_3_common_areas_module.md
    │   ├── PHASE_3_vehicle_module.md
    │   ├── PHASE_4_ai_bot_module.md
    │   ├── PHASE_4_ai_report_module.md
    │   ├── PHASE_4_fine_module.md
    │   └── PHASE_4_facial_recognition_module.md
    └── frontend/
        ├── PHASE_1_project_setup.md
        ├── PHASE_1_design_system.md
        ├── PHASE_2_auth_screens.md
        ├── PHASE_2_admin_dashboard.md
        ├── PHASE_2_complex_management.md
        ├── PHASE_2_owner_tenant_screens.md
        ├── PHASE_2_fee_payment_screens.md
        ├── PHASE_3_visitor_screens.md
        ├── PHASE_3_notification_center.md
        ├── PHASE_3_common_areas_screens.md
        ├── PHASE_3_vehicle_screens.md
        ├── PHASE_4_ai_chat_bot.md
        ├── PHASE_4_reports_screens.md
        └── PHASE_4_facial_recognition_screens.md
```

---

## 🗺️ MVP Scope

| Phase | Description | Priority |
|-------|-------------|----------|
| **Phase 1** | Project setup, auth, database schema | 🔴 Essential |
| **Phase 2** | Core management: complexes, owners, fees, payments | 🔴 Essential |
| **Phase 3** | Visitors, vehicles, notifications, common areas | 🟡 Important |
| **Phase 4** | AI features, fines, facial recognition | 🟢 Advanced |

---

## 👥 Roles

| Role | Description |
|------|-------------|
| **Super Admin** | Platform-level access across all complexes |
| **Administrator** | Manages one or more residential complexes |
| **Owner** | Apartment owner; can register tenants |
| **Tenant** | Renter registered by owner |
| **Guard** | Security personnel; handles access control |
