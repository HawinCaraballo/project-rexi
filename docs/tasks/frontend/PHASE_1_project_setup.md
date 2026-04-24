# TASK: Frontend — Phase 1 — Project Setup

**Phase:** 1 (Foundation)  
**Module:** Infrastructure  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 2 days  

---

## Objective

Initialize the Next.js 15 frontend project with TypeScript, Tailwind CSS, design system configuration, i18n setup, and routing.

---

## Commands to Run

```bash
# In the rexi-frontend directory:
npx create-next-app@latest ./ \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --no-turbopack

# Install dependencies
npm install \
  @tanstack/react-query @tanstack/react-table \
  zustand \
  react-hook-form zod @hookform/resolvers \
  next-intl \
  axios \
  recharts \
  qrcode.react \
  react-webcam \
  react-dropzone \
  @react-pdf/renderer \
  date-fns \
  lucide-react \
  clsx tailwind-merge

# Shadcn/ui init
npx shadcn@latest init
# Select: New York style, Slate color, CSS variables: yes

# Add core shadcn components
npx shadcn@latest add button input label form select table \
  dialog sheet dropdown-menu toast badge avatar card \
  separator breadcrumb tabs sidebar skeleton alert
```

---

## Environment Variables (.env.local)

```
NEXT_PUBLIC_API_URL=http://localhost:5000/api/v1
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_AZURE_BLOB_URL=
```

---

## Folder Structure to Create

Follow the structure in `docs/architecture/frontend.md`.

Key files to create manually:

**`src/lib/utils.ts`** — `cn()` helper from shadcn
**`src/lib/formatters.ts`** — currency and date formatters (ES/EN locale-aware)
**`src/lib/constants.ts`** — APP_NAME, routes, roles, etc.
**`src/types/models.ts`** — TypeScript interfaces matching API DTOs
**`src/types/api.ts`** — `ApiResponse<T>`, `PagedResult<T>`, `ApiError`
**`src/services/api.ts`** — Axios instance with auth interceptor

---

## API Client Setup

```typescript
// src/services/api.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  headers: { 'Content-Type': 'application/json' },
});

// Request interceptor: add JWT token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Response interceptor: handle 401 → refresh or redirect to login
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Try refresh token
      // If fails → clear auth + redirect to /login
    }
    return Promise.reject(error);
  }
);
```

---

## Auth Store (Zustand)

```typescript
// src/store/authStore.ts
interface AuthState {
  user: UserProfile | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  setAuth: (user: UserProfile, token: string) => void;
  clearAuth: () => void;
}
```

---

## i18n Configuration

```typescript
// src/i18n/routing.ts
export const routing = defineRouting({
  locales: ['es', 'en'],
  defaultLocale: 'es',
});

// src/middleware.ts
export default createMiddleware(routing);
export const config = { matcher: ['/((?!api|_next|.*\\..*).*)'] };
```

Translation files:
- `src/i18n/messages/es.json` — Spanish (default)
- `src/i18n/messages/en.json` — English

---

## next.config.ts

```typescript
import createNextIntlPlugin from 'next-intl/plugin';
const withNextIntl = createNextIntlPlugin();

export default withNextIntl({
  images: {
    remotePatterns: [{ hostname: '*.blob.core.windows.net' }],
  },
});
```

---

## App Layout Structure

```
app/
├── (auth)/
│   └── layout.tsx    → centered card layout (no sidebar)
└── (portal)/
    └── layout.tsx    → sidebar + topbar layout (role-filtered nav)
```

---

## Acceptance Criteria

- [ ] `npm run dev` starts on port 3000
- [ ] TypeScript compiles without errors
- [ ] ESLint passes with no errors
- [ ] Tailwind CSS classes work
- [ ] Shadcn components render correctly
- [ ] i18n routing works: `/es/...` and `/en/...`
- [ ] Axios client adds Authorization header when token present
- [ ] Docker build succeeds: `docker build -t rexi-frontend .`

---

## Dockerfile

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
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
