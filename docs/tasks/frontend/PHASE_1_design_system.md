# TASK: Frontend — Phase 1 — Design System

**Phase:** 1 (Foundation)  
**Module:** UI / Design  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 2 days  
**Depends on:** PHASE_1_project_setup  

---

## Objective

Establish the visual design language for Rexi: color palette, typography, spacing, reusable layout components, and core UI patterns.

---

## Brand Identity

**Product Name:** Rexi  
**Tagline:** "Tu conjunto, simplificado."  
**Tone:** Professional, trustworthy, modern, accessible  

---

## Color Palette

```css
/* src/app/globals.css */
:root {
  /* Primary - Deep teal (trust + calm) */
  --color-primary-50: #f0fdfa;
  --color-primary-100: #ccfbf1;
  --color-primary-500: #14b8a6;
  --color-primary-600: #0d9488;
  --color-primary-700: #0f766e;
  --color-primary-900: #134e4a;

  /* Secondary - Warm slate */
  --color-secondary-50: #f8fafc;
  --color-secondary-100: #f1f5f9;
  --color-secondary-500: #64748b;
  --color-secondary-700: #334155;
  --color-secondary-900: #0f172a;

  /* Accent - Amber (alerts, CTAs) */
  --color-accent-400: #fbbf24;
  --color-accent-500: #f59e0b;

  /* Semantic */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  --color-info: #3b82f6;

  /* Surface */
  --color-surface: #ffffff;
  --color-surface-raised: #f8fafc;
  --color-border: #e2e8f0;
}

/* Tailwind extend */
```

**Tailwind Config Extension:**
```typescript
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      primary: { /* map to CSS vars */ },
      secondary: { /* ... */ },
    },
    fontFamily: {
      sans: ['Inter', 'system-ui', 'sans-serif'],
      display: ['Outfit', 'sans-serif'],
    },
    borderRadius: { DEFAULT: '0.5rem', lg: '0.75rem' },
  }
}
```

---

## Typography

Import from Google Fonts in `layout.tsx`:
- **Display / Headers:** Outfit (600, 700)
- **Body / UI:** Inter (400, 500, 600)

```typescript
// app/layout.tsx
import { Inter, Outfit } from 'next/font/google';
const inter = Inter({ subsets: ['latin'], variable: '--font-inter' });
const outfit = Outfit({ subsets: ['latin'], variable: '--font-outfit' });
```

**Scale:**
| Token | Size | Weight | Usage |
|-------|------|--------|-------|
| `text-4xl font-display` | 36px | 700 | Page hero title |
| `text-2xl font-display` | 24px | 700 | Page titles |
| `text-xl font-semibold` | 20px | 600 | Section headers |
| `text-base` | 16px | 400 | Body text |
| `text-sm` | 14px | 400 | Labels, helper text |
| `text-xs` | 12px | 400 | Captions, badges |

---

## Core Components to Build

### 1. AppShell (Portal Layout)

```tsx
// components/layout/AppShell.tsx
// - Left sidebar (collapsible on mobile)
// - Top bar with: breadcrumbs, notifications bell, user avatar menu
// - Main content area with padding
// - Role-based navigation items (filtered by usePermissions)
```

**Sidebar Navigation Items (by role):**

| Nav Item | Roles |
|----------|-------|
| Dashboard | All |
| Complexes | SuperAdmin |
| Towers & Apartments | Admin |
| Owners | Admin |
| Tenants | Admin, Owner |
| Fees | Admin, Owner, Tenant |
| Payments | Admin, Owner |
| Visitors | Admin, Owner, Tenant |
| Vehicles | Admin, Guard |
| Common Areas | All |
| Fines | Admin, Guard, Owner, Tenant |
| Notifications | All |
| Documents | All |
| Reports | Admin |
| AI Bot | All |
| Emergency | All |
| Settings | Admin, Owner, Tenant |

### 2. DataTable Component

```tsx
// components/tables/DataTable.tsx
// Props:
// - columns: ColumnDef<T>[]  (TanStack Table)
// - queryKey: QueryKey
// - queryFn: (params: PaginationParams) => Promise<PagedResult<T>>
// - filters?: ReactNode  (slot for filter controls)
// - actions?: ReactNode  (slot for top-right action buttons)
// - exportable?: boolean

// Features:
// - Server-side pagination (page, pageSize selectors)
// - Sortable columns (click header)
// - Column visibility toggle
// - Loading skeleton rows
// - Empty state illustration
// - Export button (calls separate export endpoint)
```

### 3. PageLayout Component

```tsx
// components/layout/PageLayout.tsx
// Props: title, breadcrumbs, actions, children
// Renders: page header with title + actions, breadcrumb trail, content area
```

### 4. StatusBadge Component

```tsx
// components/shared/StatusBadge.tsx
// Statuses: pending, paid, overdue, partial, active, inactive, 
//           approved, rejected, appealed, authorized, expired
// Color mapping: each status → semantic color
```

### 5. ConfirmationDialog Component

```tsx
// components/shared/ConfirmationDialog.tsx
// Props: title, description, onConfirm, onCancel, variant (danger|warning|default)
// Uses: Radix AlertDialog
```

### 6. FileUpload Component

```tsx
// components/shared/FileUpload.tsx
// Uses: react-dropzone
// Supports: images (profile photos), PDFs (documents), CSV/Excel (bulk upload)
// Shows: drag-drop zone, preview thumbnails, upload progress
// On file select: calls POST /media/upload, returns blob URL
```

---

## Responsive Breakpoints

| Breakpoint | Width | Layout |
|------------|-------|--------|
| `sm` | 640px | Sidebar collapses to bottom nav |
| `md` | 768px | Sidebar icon-only mode |
| `lg` | 1024px | Full sidebar |
| `xl` | 1280px | Wide content area |

---

## Animation Guidelines

Use Tailwind's built-in transitions + CSS animations (no heavy animation library):
- Sidebar toggle: `transition-all duration-300`
- Modal open/close: `animate-in fade-in zoom-in-95`
- Loading states: skeleton pulse animation
- Hover on cards: `hover:-translate-y-0.5 transition-transform`
- Button press: `active:scale-95 transition-transform`

---

## Empty States

Each list view must have a designed empty state:
```tsx
// components/shared/EmptyState.tsx
// Props: icon, title, description, action (optional CTA button)
// Example: "No apartments yet" + "Add your first apartment" button
```

---

## Error States

- **Form field errors:** Red border + error message below field
- **API errors:** Toast notification (top-right, auto-dismiss 5s)
- **Page-level errors:** Full-page error card with retry button
- **404:** Custom 404 page with navigation back to dashboard

---

## Acceptance Criteria

- [ ] Color palette applied via CSS variables + Tailwind config
- [ ] Fonts loaded from Google Fonts (no FOUT)
- [ ] AppShell renders correctly with sidebar + topbar
- [ ] Navigation items filter by role
- [ ] DataTable shows loading skeletons, empty state, and data
- [ ] StatusBadge renders all status variants
- [ ] FileUpload handles drag-drop and shows preview
- [ ] Responsive: works on mobile (375px) and desktop (1440px)
- [ ] Dark mode ready (CSS variables swap via `data-theme="dark"`)
