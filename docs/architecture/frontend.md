# Rexi — Frontend Architecture Proposal

**Stack:** React 19 + Next.js 15 (App Router) + TypeScript  
**Target:** Multi-tenant SaaS, multi-language (ES/EN), mobile-first  
**Scalability Target:** 100 complexes / ~15,000 users in Year 1  

---

## 1. Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Framework | Next.js 15 (App Router) | SSR/SSG, file-based routing, built-in i18n, Vercel/Azure deployment |
| Language | TypeScript 5+ | Type safety, better DX, maintainability |
| State Management | Zustand + React Query (TanStack Query v5) | Lightweight global state + async server state |
| UI Component Library | Shadcn/ui + Radix UI primitives | Accessible, headless, fully customizable |
| Styling | Tailwind CSS v4 | Utility-first, design tokens, responsive |
| Forms | React Hook Form + Zod | Performant forms with schema-based validation |
| Internationalization | next-intl | File-based translations, SSR-compatible |
| Charts / Analytics | Recharts | Composable, responsive charting |
| Tables | TanStack Table v8 | Server-side pagination, filtering, sorting |
| PDF Generation | React PDF (@react-pdf/renderer) | Client/server PDF rendering |
| File Upload | react-dropzone + Azure Blob SDK | Direct-to-blob upload |
| QR Codes | qrcode.react | QR generation |
| Camera / Face | react-webcam | Facial enrollment UI |
| Testing | Vitest + React Testing Library + Playwright | Unit + E2E |
| Linting | ESLint + Prettier | Code quality |
| CI/CD | GitHub Actions + Azure Static Web Apps or Vercel | Automated deploy |

---

## 2. Project Structure

```
rexi-frontend/
├── public/
│   └── locales/              # Static assets
├── src/
│   ├── app/                  # Next.js App Router pages
│   │   ├── (auth)/           # Auth group (login, register, reset)
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── reset-password/
│   │   ├── (portal)/         # Authenticated portal (role-gated)
│   │   │   ├── dashboard/
│   │   │   ├── complexes/
│   │   │   ├── owners/
│   │   │   ├── tenants/
│   │   │   ├── apartments/
│   │   │   ├── fees/
│   │   │   ├── payments/
│   │   │   ├── visitors/
│   │   │   ├── vehicles/
│   │   │   ├── common-areas/
│   │   │   ├── fines/
│   │   │   ├── notifications/
│   │   │   ├── reports/
│   │   │   ├── documents/
│   │   │   ├── ai-bot/
│   │   │   ├── emergency/
│   │   │   └── settings/
│   │   ├── api/              # Next.js API routes (BFF layer if needed)
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/               # Shadcn/ui base components (Button, Input, etc.)
│   │   ├── layout/           # Shell, Sidebar, TopBar, Breadcrumbs
│   │   ├── forms/            # Reusable form components
│   │   ├── tables/           # DataTable wrappers with filters/pagination
│   │   ├── charts/           # Chart wrappers (BarChart, LineChart, etc.)
│   │   ├── notifications/    # Notification bell, dropdown, center
│   │   ├── ai/               # ChatBot UI, emergency chat
│   │   └── shared/           # Avatar, StatusBadge, FileUpload, QRDisplay
│   ├── modules/              # Feature modules (co-located logic)
│   │   ├── auth/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   └── store/
│   │   ├── complexes/
│   │   ├── owners/
│   │   ├── tenants/
│   │   ├── fees/
│   │   ├── payments/
│   │   ├── visitors/
│   │   ├── vehicles/
│   │   ├── common-areas/
│   │   ├── fines/
│   │   └── notifications/
│   ├── services/             # API client layer
│   │   ├── api.ts            # Axios/fetch base client with interceptors
│   │   └── endpoints.ts      # Typed endpoint definitions
│   ├── store/                # Global Zustand stores
│   │   ├── authStore.ts
│   │   └── uiStore.ts
│   ├── hooks/                # Shared hooks
│   │   ├── usePermissions.ts
│   │   ├── usePagination.ts
│   │   └── useDebounce.ts
│   ├── lib/                  # Utilities
│   │   ├── utils.ts
│   │   ├── validators.ts     # Zod schemas
│   │   ├── formatters.ts     # Currency, dates
│   │   └── constants.ts
│   ├── i18n/                 # next-intl configuration
│   │   ├── routing.ts
│   │   ├── request.ts
│   │   └── messages/
│   │       ├── en.json
│   │       └── es.json
│   ├── types/                # Shared TypeScript interfaces
│   │   ├── api.ts
│   │   ├── models.ts
│   │   └── permissions.ts
│   └── middleware.ts         # Auth + i18n middleware
├── tests/
│   ├── unit/
│   └── e2e/
├── .env.local
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── Dockerfile
```

---

## 3. Key Design Decisions

### 3.1 Role-Based Access Control (RBAC) on Frontend

```typescript
// types/permissions.ts
export type Role = 'SuperAdmin' | 'Admin' | 'Owner' | 'Tenant' | 'Guard';

export const PERMISSIONS = {
  'complexes:manage': ['SuperAdmin', 'Admin'],
  'owners:manage': ['SuperAdmin', 'Admin'],
  'tenants:manage': ['SuperAdmin', 'Admin', 'Owner'],
  'fees:view': ['SuperAdmin', 'Admin', 'Owner', 'Tenant'],
  'vehicles:search': ['SuperAdmin', 'Admin', 'Guard'],
  'fines:issue': ['Guard'],
  'fines:approve': ['Admin'],
  // ...
} as const;
```

- `usePermissions()` hook checks role from auth token
- `<PermissionGate permission="fines:issue">` wrapper component
- Navigation items filtered by role

### 3.2 Server-Side State with TanStack Query

```typescript
// All API calls use React Query for:
// - Automatic caching
// - Background refresh
// - Optimistic updates
// - Stale-while-revalidate
const { data: apartments, isLoading } = useQuery({
  queryKey: ['apartments', complexId, { page, filters }],
  queryFn: () => apartmentService.list(complexId, { page, filters }),
  staleTime: 30_000,
});
```

### 3.3 Form Architecture

```typescript
// Every form uses React Hook Form + Zod schema
const apartmentSchema = z.object({
  number: z.string().min(1).max(10),
  floor: z.number().int().positive(),
  coefficient: z.number().min(0).max(1),
  parkingType: z.enum(['assigned', 'communal', 'none']),
  // ...
});

type ApartmentForm = z.infer<typeof apartmentSchema>;
```

### 3.4 Internationalization

```
// i18n/messages/es.json
{
  "nav": { "dashboard": "Panel", "owners": "Propietarios" },
  "common": { "save": "Guardar", "cancel": "Cancelar" },
  "fees": { "title": "Cuotas de Administración", ... }
}
```

- URL-based locale switching: `/es/dashboard`, `/en/dashboard`
- All dates and currencies formatted per locale

### 3.5 Pagination & Filtering Pattern

All list pages use a **server-side cursor/offset** pattern:
- `page`, `pageSize`, `sortBy`, `sortDir` query params
- Filters as query params (persisted to URL for bookmarking/sharing)
- TanStack Table handles client-side column visibility

---

## 4. Component Architecture

### 4.1 DataTable (Reusable)

```
<DataTable
  columns={apartmentColumns}
  queryKey={['apartments']}
  queryFn={apartmentService.list}
  filters={<ApartmentFilters />}
  actions={<CreateApartmentButton />}
  exportable
/>
```

### 4.2 Page Layout Pattern

```
<PageLayout
  title="Apartments"
  breadcrumbs={[...]}
  actions={<Button>Add Apartment</Button>}
>
  <DataTable ... />
</PageLayout>
```

---

## 5. Performance Strategy

| Technique | Application |
|-----------|-------------|
| Route-based code splitting | Next.js automatic |
| Image optimization | next/image with Azure CDN |
| React Query caching | 30s stale time for list data |
| Virtualized lists | react-virtual for large lists |
| Lazy loading modals | next/dynamic for heavy modals |
| Bundle analysis | @next/bundle-analyzer in CI |

---

## 6. Docker

```dockerfile
# Dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

---

## 7. Deployment (Azure)

```
Azure Static Web Apps  ← Option A (simpler, CDN-backed)
Azure App Service       ← Option B (full Node.js runtime, recommended for SSR)
Azure Container Apps    ← Option C (Docker, best for scaling)
```

**Recommended:** Azure Container Apps (Docker) — aligned with backend containerization strategy.

---

## 8. Scalability Projection

| Year | Complexes | Users | Strategy |
|------|-----------|-------|----------|
| 1 | 100 | 15,000 | Single Next.js app, Azure Container Apps |
| 2 | 500 | 75,000 | Add CDN caching, horizontal scaling |
| 3+ | 1,000+ | 150,000+ | Micro-frontend consideration per role |
