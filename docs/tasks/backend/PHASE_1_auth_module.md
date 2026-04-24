# TASK: Backend — Phase 1 — Authentication Module

**Phase:** 1 (Foundation)  
**Module:** Identity / Auth  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 2 days  
**Depends on:** PHASE_1_project_setup, PHASE_1_database_erd  

---

## Objective

Implement JWT-based authentication with role-based authorization, password reset flow, and refresh token rotation.

---

## Endpoints

| Method | Route | Access | Description |
|--------|-------|--------|-------------|
| POST | `/api/v1/auth/login` | Public | Login with email/password |
| POST | `/api/v1/auth/refresh` | Public | Refresh JWT using refresh token |
| POST | `/api/v1/auth/logout` | Auth | Revoke refresh token |
| POST | `/api/v1/auth/forgot-password` | Public | Send reset email |
| POST | `/api/v1/auth/reset-password` | Public | Reset password with token |
| POST | `/api/v1/auth/change-password` | Auth | Change own password |
| GET  | `/api/v1/auth/me` | Auth | Get current user profile |

---

## Request / Response DTOs

### POST /auth/login
```json
// Request
{ "email": "owner@example.com", "password": "Secret123!" }

// Response 200
{
  "accessToken": "eyJ...",
  "refreshToken": "abc123...",
  "expiresAt": "2026-04-22T14:00:00Z",
  "user": {
    "id": "uuid",
    "email": "owner@example.com",
    "fullName": "Carlos García",
    "role": "Owner",
    "complexId": "uuid"
  }
}

// Response 401
{ "title": "Invalid credentials", "status": 401 }
```

### POST /auth/refresh
```json
// Request
{ "refreshToken": "abc123..." }

// Response 200: same as login response
// Response 401: { "title": "Invalid or expired refresh token" }
```

### POST /auth/forgot-password
```json
// Request
{ "email": "owner@example.com" }

// Response 200: always (to prevent email enumeration)
{ "message": "If the email exists, a reset link was sent." }
```

### POST /auth/reset-password
```json
// Request
{ "token": "reset-token-from-email", "newPassword": "NewSecret123!" }

// Response 200: { "message": "Password updated successfully" }
// Response 400: { "errors": { "token": ["Token expired or invalid"] } }
```

---

## Implementation

### TokenService.cs (Infrastructure/Identity/)

```csharp
public interface ITokenService
{
    string GenerateAccessToken(AppUser user, string role, Guid? complexId);
    RefreshToken GenerateRefreshToken();
    ClaimsPrincipal? ValidateToken(string token);
}
```

**JWT Claims to include:**
- `sub` → userId
- `email` → user email
- `role` → user role
- `complex_id` → complexId (if applicable)
- `jti` → unique token ID

### AuthController.cs

Use `IMediator` to dispatch:
- `LoginCommand` → `LoginCommandHandler`
- `RefreshTokenCommand` → `RefreshTokenCommandHandler`
- `ForgotPasswordCommand` → `ForgotPasswordCommandHandler`
- `ResetPasswordCommand` → `ResetPasswordCommandHandler`

---

## Validation Rules

| Field | Rules |
|-------|-------|
| `email` | Required, valid email format |
| `password` (login) | Required, min 1 char |
| `newPassword` (reset/change) | Required, min 8 chars, at least 1 uppercase, 1 digit |
| `refreshToken` | Required |
| `token` (reset) | Required |

---

## Security Requirements

- Passwords stored as bcrypt hash via ASP.NET Core Identity
- Refresh tokens stored in DB, rotated on each use (old token revoked)
- Reset password tokens expire in 24 hours
- Failed login attempts: after 5 failures, lock account for 15 minutes (Identity default lockout)
- All tokens transmitted over HTTPS only

---

## Email Templates

**"Welcome to Rexi" (new user):**
```
Subject: Welcome to Rexi - Set your password
Body: Your account has been created. Click below to set your password:
[Set Password Button] (expires in 48h)
```

**"Password Reset":**
```
Subject: Reset your Rexi password
Body: We received a request to reset your password. Click below:
[Reset Password Button] (expires in 24h)
If you didn't request this, ignore this email.
```

---

## User Creation Flow (called by Owner/Tenant registration)

```csharp
public class UserCreationService
{
    public async Task<AppUser> CreateUserAsync(string email, string role, Guid? complexId)
    {
        var user = new AppUser { Email = email, UserName = email };
        var tempPassword = GenerateSecureTemp(); // 12 chars random
        await _userManager.CreateAsync(user, tempPassword);
        await _userManager.AddToRoleAsync(user, role);
        var setupToken = await _userManager.GeneratePasswordResetTokenAsync(user);
        await _emailService.SendWelcomeAsync(email, setupToken);
        return user;
    }
}
```

---

## Acceptance Criteria

- [ ] Login returns valid JWT + refresh token
- [ ] Refresh token rotates correctly (old revoked, new issued)
- [ ] Logout revokes refresh token
- [ ] Forgot password sends email with working reset link
- [ ] Reset password validates token expiry
- [ ] JWT claims verified in subsequent requests via `[Authorize]`
- [ ] Role-based policies work (`[Authorize(Policy = "CanManageComplex")]`)
- [ ] Account lockout after 5 failed attempts

---

## Unit Tests

- [ ] `LoginCommandHandler` — valid credentials returns tokens
- [ ] `LoginCommandHandler` — invalid credentials returns failure
- [ ] `RefreshTokenCommandHandler` — valid token rotates correctly
- [ ] `RefreshTokenCommandHandler` — expired token returns failure
- [ ] `TokenService.GenerateAccessToken` — claims are correct
- [ ] `ForgotPasswordCommandHandler` — always returns success (no email enumeration)

---

## Integration Tests

- [ ] POST `/auth/login` → 200 with valid user
- [ ] POST `/auth/login` → 401 with wrong password
- [ ] POST `/auth/refresh` → 200 with new tokens
- [ ] POST `/auth/refresh` → 401 with revoked token
- [ ] GET `/auth/me` → 200 authenticated; 401 unauthenticated
