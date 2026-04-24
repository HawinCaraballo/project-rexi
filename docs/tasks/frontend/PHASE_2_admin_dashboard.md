# TASK: Frontend — Phase 2 — Admin Dashboard

**Phase:** 2 (Core MVP)  
**Module:** Dashboard  
**Priority:** 🔴 Critical (MVP)  
**Estimate:** 2 days  
**Depends on:** PHASE_2_auth_screens  

---

## Objective

Build the admin dashboard with KPI cards, charts, recent activity, and quick actions.

---

## Route

`/dashboard` (Admin, SuperAdmin)

---

## Screen Layout

```
┌──────────────────────────────────────────────────────────────┐
│  Dashboard                                   [Generate Fees]  │
├─────────────┬──────────────┬──────────────┬──────────────────┤
│ Total Fees  │  Collected   │   Overdue    │  Collection Rate │
│  $28.2M     │   $23.5M    │    $4.7M     │     83.3%        │
│  ↑ +3% vs   │   ↑ +5%     │   ↓ -2%     │    ↑ +2pts       │
│  last month │             │              │                   │
├─────────────┴──────────────┴──────────────┴──────────────────┤
│  Payment Trend (6 months)    │  Fee Status Distribution      │
│  [Line Chart]                │  [Donut Chart]                │
│  Collected vs Expected       │  Paid / Partial / Overdue     │
├──────────────────────────────┴───────────────────────────────┤
│  Today's Activity            │  Overdue Apartments           │
│  ─────────────               │  ──────────────────           │
│  📦 Package - Apt 101 (5m)   │  101-A | Carlos G. | $470K   │
│  🚗 Entry - ABC123 (10m)     │  203-B | María R.  | $235K   │
│  👤 Visitor - QR (15m)       │  [View all overdue →]        │
│  [View full activity →]      │                               │
└──────────────────────────────────────────────────────────────┘
```

---

## KPI Cards

Each card shows:
- Label (i18n key)
- Main value (large, formatted as currency or percentage)
- Trend indicator (% change vs last month, with up/down arrow)
- Color: positive trend = green, negative = red (for overdue: reverse)

**Cards:**
1. **Total Expected** — `complex.summary.totalExpected`
2. **Total Collected** — `complex.summary.totalCollected`  
3. **Total Overdue** — `complex.summary.totalOverdue`
4. **Collection Rate** — `complex.summary.collectionRate`%
5. **Active Apartments** — `complex.summary.occupiedApartments`
6. **Visitors Today** — `access_logs` count for today

---

## Charts

### Payment Trend (Line Chart — Recharts)

```typescript
// Data: 6 months of { month, expected, collected }
// Two lines: "Expected" (dashed, secondary) and "Collected" (solid, primary)
// X-axis: month names (locale-aware)
// Y-axis: currency formatted (abbreviate: 1M, 500K)
// Tooltip: shows exact amounts on hover
```

### Fee Status Donut (Donut Chart — Recharts)

```typescript
// Segments: Paid (green), Partial (yellow), Overdue (red), Pending (gray)
// Center: total apartments count
// Legend below chart
```

---

## Recent Activity Feed

```typescript
// GET /api/v1/complexes/{cid}/activity-log?limit=10
// Items show: icon (emoji or lucide), description, relative time ("hace 5 min")
// Types: package, vehicle_entry, visitor_entry, payment, fine
// Clicking an item navigates to the relevant record
```

---

## Overdue Apartments Widget

```typescript
// GET /api/v1/complexes/{cid}/fees/summary (current month)
// Shows top 5 overdue apartments
// Columns: Apartment | Owner | Amount | Days Overdue
// "Send reminder" button per row → triggers notification
// "View all" link → /fees?status=overdue
```

---

## Quick Actions

Floating action area (top-right of page):
- **Generate Fees** button (Admin only) → opens confirmation dialog → calls POST /fees/generate
- **Send Announcement** button → opens slide-over panel with announcement form

---

## Data Fetching

```typescript
// src/modules/dashboard/hooks/useDashboard.ts
export function useDashboard(complexId: string) {
  const summary = useQuery({
    queryKey: ['dashboard-summary', complexId],
    queryFn: () => dashboardService.getSummary(complexId),
    refetchInterval: 60_000, // Auto-refresh every minute
  });

  const trend = useQuery({
    queryKey: ['payment-trend', complexId],
    queryFn: () => dashboardService.getPaymentTrend(complexId, 6),
    staleTime: 5 * 60_000,
  });

  return { summary, trend };
}
```

---

## Responsive Behavior

- **Desktop (lg+):** 4-column KPI grid, 2-column charts, 2-column bottom widgets
- **Tablet (md):** 2-column KPI grid, stacked charts
- **Mobile (sm):** 1-column everything, simplified charts

---

## Acceptance Criteria

- [ ] All 6 KPI cards render with correct data
- [ ] Trend indicators show correct direction and color
- [ ] Line chart renders with 6 months of data
- [ ] Donut chart renders with correct segment colors
- [ ] Recent activity feed shows last 10 events with relative timestamps
- [ ] Overdue apartments widget shows correct apartments
- [ ] "Send reminder" triggers notification and shows success toast
- [ ] Dashboard data auto-refreshes every 60 seconds
- [ ] Generate Fees button shows confirmation dialog before action
- [ ] Responsive layout works on all breakpoints

---

## Component IDs

```
dashboard-kpi-total-expected
dashboard-kpi-collected
dashboard-kpi-overdue
dashboard-kpi-rate
dashboard-chart-payment-trend
dashboard-chart-status-donut
dashboard-activity-feed
dashboard-overdue-list
dashboard-btn-generate-fees
dashboard-btn-announcement
```
