# TASK: Frontend — Phase 2 — Authentication Screens

**Phase:** 2 (Core MVP)  
**Module:** Auth  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 2 days  
**Depends on:** PHASE_1_design_system  

---

## Objective

Build all authentication screens: login, forgot password, reset password, and first-time password setup.

---

## Screens

### 1. Login Screen (`/login`)

**Layout:** Centered card (max-w-md) on a split background (left: brand illustration/color, right: form)

**Fields:**
| Field | Type | Validation |
|-------|------|-----------|
| Email | Email input | Required, valid email format |
| Password | Password input (toggle visibility) | Required, min 1 char |

**Elements:**
- Rexi logo at top
- "Iniciar Sesión" / "Sign In" heading
- Email field with label
- Password field with show/hide toggle button
- "Forgot password?" link (right-aligned under password)
- Sign In button (full width, primary, loading state)
- Language switcher (ES/EN) in top-right corner
- Error message area (below button, red, for invalid credentials)

**Behavior:**
- On submit → call `POST /auth/login`
- On success → store token in auth store → redirect based on role:
  - SuperAdmin → `/complexes`
  - Admin → `/dashboard`
  - Owner → `/my/fees`
  - Tenant → `/my/fees`
  - Guard → `/guard/search`
- On 401 → show "Email o contraseña incorrectos"
- On account locked → show "Cuenta bloqueada. Intente en 15 minutos."
- "Enter" key submits form

**Form ID:** `login-form`

---

### 2. Forgot Password Screen (`/forgot-password`)

**Fields:**
| Field | Type | Validation |
|-------|------|-----------|
| Email | Email input | Required, valid email |

**Elements:**
- Back arrow to login
- "¿Olvidaste tu contraseña?" heading
- Helper text: "Ingresa tu email y te enviaremos un enlace para restablecer tu contraseña."
- Email field
- Send button
- On success: show success message card (same page, replace form)

**Success message:** "Si el email está registrado, recibirás un enlace en los próximos minutos. Revisa también tu carpeta de spam."

---

### 3. Reset Password Screen (`/reset-password?token=...`)

**Fields:**
| Field | Type | Validation |
|-------|------|-----------|
| New Password | Password input | Required, min 8 chars, 1 uppercase, 1 digit |
| Confirm Password | Password input | Must match new password |

**Elements:**
- Password strength indicator (weak/medium/strong)
- List of requirements with checkmarks as typed
- Submit button
- On success → redirect to login with success toast

**Expired token:** Show error card "Este enlace ha expirado. Solicita uno nuevo." with link to forgot-password.

---

### 4. Set Password (First Login) (`/set-password?token=...`)

Same as Reset Password but with a welcome message:
"Bienvenido a Rexi. Crea tu contraseña para activar tu cuenta."

---

### 5. Change Password Screen (`/settings/security`)

**Fields:**
| Field | Validation |
|-------|-----------|
| Current Password | Required |
| New Password | Required, min 8, uppercase, digit |
| Confirm New Password | Must match |

---

## Form Validation

Use `React Hook Form` + `Zod` for all forms:

```typescript
// Login schema
const loginSchema = z.object({
  email: z.string().email('Email inválido'),
  password: z.string().min(1, 'La contraseña es requerida'),
});

// Reset password schema
const resetPasswordSchema = z.object({
  newPassword: z.string()
    .min(8, 'Mínimo 8 caracteres')
    .regex(/[A-Z]/, 'Debe incluir al menos una mayúscula')
    .regex(/[0-9]/, 'Debe incluir al menos un número'),
  confirmPassword: z.string(),
}).refine(d => d.newPassword === d.confirmPassword, {
  message: 'Las contraseñas no coinciden',
  path: ['confirmPassword'],
});
```

---

## Auth Module Structure

```
src/modules/auth/
├── hooks/
│   ├── useLogin.ts          # useMutation for POST /auth/login
│   ├── useForgotPassword.ts
│   └── useResetPassword.ts
├── services/
│   └── authService.ts       # API calls
└── store/
    └── authStore.ts         # Zustand store
```

---

## UX Details

- Passwords hidden by default (eye icon toggle)
- Loading spinner replaces button text during API call
- All error messages in the user's current language
- Auto-focus first field on page load
- ARIA labels on all inputs
- Tab order flows logically

---

## Acceptance Criteria

- [ ] Login works with valid credentials → redirects to correct dashboard per role
- [ ] Login shows error for invalid credentials
- [ ] Forgot password form sends email and shows success message
- [ ] Reset password validates token and password requirements
- [ ] Password strength indicator reflects requirements in real time
- [ ] Confirm password validates match
- [ ] All forms accessible (ARIA, keyboard navigation)
- [ ] Mobile responsive (375px wide)

---

## Component IDs (for testing)

```
login-email-input
login-password-input
login-submit-button
login-forgot-link
login-error-message
forgot-email-input
forgot-submit-button
forgot-success-message
reset-password-input
reset-confirm-input
reset-submit-button
reset-strength-indicator
```
