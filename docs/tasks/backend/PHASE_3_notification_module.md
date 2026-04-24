# TASK: Backend — Phase 3 — Notification Module

**Phase:** 3 (Important)  
**Module:** Communication  
**Priority:** 🟡 Important  
**Estimate:** 2 days  
**Depends on:** PHASE_1_auth_module  

---

## Objective

Build a unified multi-channel notification system supporting in-app, email, and push notifications with user preferences.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/notifications` | Auth | Get own notifications (paginated) |
| PUT | `/api/v1/notifications/{id}/read` | Auth | Mark as read |
| PUT | `/api/v1/notifications/read-all` | Auth | Mark all as read |
| GET | `/api/v1/notifications/unread-count` | Auth | Unread count (for bell badge) |
| POST | `/api/v1/complexes/{cid}/announcements` | Admin | Send general announcement |
| GET | `/api/v1/users/notification-preferences` | Auth | Get preferences |
| PUT | `/api/v1/users/notification-preferences` | Auth | Update preferences |

---

## DTOs

### NotificationDto
```csharp
public record NotificationDto(
    Guid Id,
    string Type,      // package|payment_reminder|visitor_arrived|announcement|fine|booking
    string Title,
    string Body,
    Dictionary<string, object>? Data,  // Extra context (feeId, visitorId, etc.)
    bool IsRead,
    DateTimeOffset CreatedAt
);
```

### SendAnnouncementDto
```csharp
public record SendAnnouncementDto(
    string Subject,        // Required, max 200
    string Body,           // Required, rich text HTML
    AnnouncementTarget Target, // all|per_tower|per_apartment
    List<Guid>? TowerIds,      // If target = per_tower
    List<Guid>? ApartmentIds,  // If target = per_apartment
    bool SendEmail,
    bool SendPush,
    bool SendInApp
);
```

### UserNotificationPreferencesDto
```csharp
public record UserNotificationPreferencesDto(
    bool InAppEnabled,
    bool EmailEnabled,
    bool PushEnabled,
    bool PaymentReminders,
    bool VisitorAlerts,
    bool PackageAlerts,
    bool Announcements,
    bool FineAlerts
);
```

---

## Notification Service Interface

```csharp
public interface INotificationService
{
    Task SendAsync(Guid userId, string type, string title, string body,
                   Dictionary<string, object>? data = null,
                   bool email = true, bool push = true, bool inApp = true);
    
    Task SendToApartmentAsync(Guid apartmentId, string type, string title, string body,
                              Dictionary<string, object>? data = null);
    
    Task SendAnnouncementAsync(SendAnnouncementDto dto, Guid complexId);
    
    // Specific helpers:
    Task SendPaymentReminderAsync(AdministrationFee fee, int daysUntilDue);
    Task SendPackageArrivedAsync(Package package);
    Task SendVisitorArrivedAsync(Visitor visitor);
    Task SendFineIssuedAsync(Fine fine);
    Task SendBookingConfirmedAsync(Booking booking);
}
```

---

## Notification Types

| Type Constant | Trigger | Recipients |
|---------------|---------|-----------|
| `package.arrived` | Guard logs package | Owner + active tenant of apartment |
| `payment.reminder_7d` | Reminder job (7 days before) | Owner |
| `payment.reminder_3d` | Reminder job (3 days before) | Owner |
| `payment.due_today` | Reminder job (due date) | Owner |
| `payment.overdue` | Daily overdue job | Owner |
| `visitor.arrived` | Guard authorizes visitor entry | Owner + tenant |
| `fine.issued` | Fine approved by admin | Owner |
| `fine.appeal.resolved` | Admin resolves appeal | Owner |
| `booking.confirmed` | Booking confirmed | Booker |
| `booking.cancelled` | Admin cancels booking | Booker |
| `announcement` | Admin sends announcement | Selected residents |
| `issue.status_changed` | Admin updates issue | Reporter |

---

## Push Notification (Firebase FCM)

```csharp
// Infrastructure/Services/PushNotificationService.cs
public async Task SendPushAsync(string fcmToken, string title, string body, 
                                 Dictionary<string, string>? data = null)
{
    var message = new Message
    {
        Token = fcmToken,
        Notification = new Notification { Title = title, Body = body },
        Data = data ?? new(),
        Android = new AndroidConfig { Priority = Priority.High },
        Apns = new ApnsConfig { /* iOS priority */ }
    };
    await FirebaseMessaging.DefaultInstance.SendAsync(message);
}
```

Users register their FCM device token via:
`POST /api/v1/users/device-token` → `{ "token": "fcm-device-token" }`

---

## Real-time Notifications (SignalR)

Use **SignalR** to push notifications instantly to active web/mobile clients.

**Hub Definition:**
```csharp
public class NotificationHub : Hub
{
    public override async Task OnConnectedAsync()
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, Context.UserIdentifier);
        await base.OnConnectedAsync();
    }
}
```

**Sending via SignalR:**
Whenever an in-app notification is created, broadcast it via the `NotificationHub` to the specific user's group.

---

## Email Templates

All emails use HTML templates with complex logo + branding. Templates stored as embedded resources or in Blob storage.

Template variables: `{{ownerName}}`, `{{complexName}}`, `{{amount}}`, `{{dueDate}}`, etc.

---

## Preferences Enforcement

Before sending any notification:
```csharp
var prefs = await _uow.UserPreferences.GetAsync(userId);
var sendEmail = dto.SendEmail && prefs.EmailEnabled && prefs.PaymentReminders;
var sendPush  = dto.SendPush  && prefs.PushEnabled  && prefs.PaymentReminders;
var sendInApp = prefs.InAppEnabled; // always stored
```

---

## Acceptance Criteria

- [ ] In-app notifications stored and returned via API
- [ ] Unread count updates in real-time (via polling or SignalR)
- [ ] Email sent through SendGrid with correct template
- [ ] Push notification delivered to FCM (test with device token)
- [ ] User preferences respected per notification type
- [ ] Admin announcement reaches correct target audience
- [ ] Notification history paginated correctly
- [ ] Mark as read / mark all as read works

---

## Unit Tests

- [ ] `SendAnnouncementCommandHandler` — resolves correct user IDs per target
- [ ] `NotificationService.SendAsync` — respects user preferences
- [ ] `PaymentReminderJob` — calls notification service with correct days

---

## Integration Tests

- [ ] POST `/announcements` → notifications created for target users
- [ ] GET `/notifications` → returns paginated notifications for current user
- [ ] PUT `/notifications/{id}/read` → marks as read
- [ ] GET `/notifications/unread-count` → decrements after read
