# TASK: Frontend — Phase 3 — Notification Center

**Phase:** 3 (Important)  
**Module:** Communication  
**Priority:** 🟡 Important  
**Estimate:** 2 days  
**Depends on:** PHASE_1_design_system  

---

## Objective

Build in-app notification bell with dropdown, notification center page, and admin announcement composer.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/notifications` | All | Notification center page |
| `/notifications/compose` | Admin | Compose & send announcement |

---

## Notification Bell (Top Bar Component)

```typescript
// components/notifications/NotificationBell.tsx
// Shows bell icon with unread count badge (red dot with number)
// Polling: GET /notifications/unread-count every 30s
// Click → opens dropdown (max 5 latest, with "View all" link)
```

**Dropdown item:**
```
[📦] Package Received
     "You have a package at reception"          2m ago
     ─────────────────────────────────────────────────
[💰] Payment Reminder
     "Your fee of $470,000 is due tomorrow"    1h ago
     ─────────────────────────────────────────────────
[Mark all as read]        [View all notifications →]
```

**Unread items:** slightly highlighted background, bold title

---

## Notification Center Page (`/notifications`)

**Layout:** Two-column on desktop, single column on mobile

**Left: Filter sidebar**
- All / Unread
- By type: Payment | Package | Visitor | Fine | Announcement | Booking

**Right: Notification list (paginated, 20/page)**

Each notification card:
```
[Icon]  [Type badge] [Time]
[Title]
[Body preview (2 lines)]
[Action button if applicable: "View Fee" / "View Visitor" / "View Fine"]
```

**Mark as read:** Click notification → marks as read, dims background  
**"Mark all read" button** at top of list

---

## Notification Types & Icons

| Type | Icon | Color |
|------|------|-------|
| `package.arrived` | 📦 Package | Blue |
| `payment.reminder_*` | 💰 Money | Orange |
| `payment.overdue` | ⚠️ Alert | Red |
| `visitor.arrived` | 👤 User | Green |
| `fine.issued` | 🚫 Ban | Red |
| `fine.appeal.*` | ⚖️ Scale | Purple |
| `booking.confirmed` | 📅 Calendar | Green |
| `booking.cancelled` | ❌ X | Gray |
| `announcement` | 📢 Speaker | Blue |
| `issue.status_changed` | 🔧 Wrench | Yellow |

---

## Admin: Compose Announcement Page (`/notifications/compose`)

**Layout:** Full-page form with preview panel

**Form (left side):**
| Field | Type | Validation |
|-------|------|-----------|
| Subject | Text | Required, max 200 |
| Body | Rich text editor (Tiptap or TipTap) | Required |
| Target Audience | Radio | All residents / By tower / By apartment |
| Towers (if by tower) | Multi-select | At least 1 required |
| Apartments (if by apartment) | Multi-select autocomplete | At least 1 required |
| Channels | Checkboxes | In-app, Email, Push (at least 1) |

**Preview panel (right side):**
- Live preview of how notification will look
- Channel preview tabs: "In-app" / "Email" / "Push"

**Audience summary:**
"This will be sent to 45 residents in 2 towers"

**Footer:** Cancel | Preview | Send

**Confirmation dialog before send:**
"Send announcement to 45 residents via [in-app, email, push]?"

---

## User Notification Preferences (`/settings/notifications`)

**Toggle switches for:**
- Email notifications enabled
- Push notifications enabled
- In-app enabled (always visible, always on)
- Payment reminders
- Visitor alerts
- Package alerts
- Announcements
- Fine alerts

**Note:** "These preferences apply to non-critical notifications. Payment overdue alerts are always sent."

---

## Notification Data Fetching

```typescript
// src/modules/notifications/hooks/useNotifications.ts
export function useNotifications() {
  return useQuery({
    queryKey: ['notifications'],
    queryFn: notificationService.getAll,
    refetchInterval: 30_000, // Poll every 30s
  });
}

export function useUnreadCount() {
  return useQuery({
    queryKey: ['notifications-unread'],
    queryFn: notificationService.getUnreadCount,
    refetchInterval: 30_000,
  });
}
```

---

## Acceptance Criteria

- [ ] Bell badge shows correct unread count
- [ ] Count decrements as notifications are read
- [ ] Notification dropdown shows latest 5 with relative timestamps
- [ ] "Mark all as read" clears all unread
- [ ] Notification center paginates and filters by type
- [ ] Admin compose form validates all fields
- [ ] Audience summary shows correct count before sending
- [ ] After sending, success toast shown
- [ ] User preferences save and persist
- [ ] Unread count polling works (changes in another tab are reflected)

---

## Component IDs

```
notification-bell
notification-bell-badge
notification-dropdown
notification-dropdown-list
notification-mark-all-btn
notification-center-list
notification-filter-type
notification-filter-unread
compose-form
compose-subject
compose-body-editor
compose-target-all
compose-target-tower
compose-target-apartment
compose-tower-select
compose-apartment-select
compose-channel-email
compose-channel-push
compose-channel-inapp
compose-send-btn
compose-audience-summary
```
