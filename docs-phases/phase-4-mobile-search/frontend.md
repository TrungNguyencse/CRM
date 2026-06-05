# Phase 4 — Frontend

> **Owner:** FE dev (+ Mobile dev for Expo) · **Depends on:** [backend.md](./backend.md) APIs

---

## Mobile app (Expo) — 5 tabs, ~12 screens

| Tab | Screens | Key features |
|-----|---------|--------------|
| **Shop** | Home, Category, PDP, Cart | Browse, add to cart, share product |
| **Scan** | Barcode scanner, Scan result | GTIN lookup, loyalty check-in |
| **Wallet** | Points, Tier, Coupons list | Loyalty from Phase 3 |
| **Orders** | Order list, Order detail | Track status, cancel if allowed |
| **Profile** | Profile, Settings, Notifications | PKCE login, push opt-in |

### Mobile requirements

| Topic | Spec |
|-------|------|
| Auth | PKCE OIDC (Keycloak-ready or JWT refresh) |
| Offline | MMKV cart cache (sync on reconnect) |
| Push | Expo Push → Notification service → FCM/APNs |
| Deep links | `store://orders/{id}`, `store://products/{slug}` |
| Min OS | iOS 15+, Android 8+ |

---

## New admin screens

| ID | Screen | Route |
|----|--------|-------|
| A18 | Reports dashboard | `/admin/reports` |
| A19 | Global search | `/admin/search` |

### Reports (A18)

| Report | Metrics |
|--------|---------|
| Sales | Revenue by day/week, online vs POS |
| Products | Top sellers, low stock |
| Customers | New registrations, tier distribution |
| Orders | Status breakdown, avg order value |

### SSE realtime (admin)

| Event | UI update |
|-------|-----------|
| `order.placed` | Dashboard order count +1 |
| `low_stock` | Alert badge on inventory nav |
| `pos.sale` | Recent activity feed |

Endpoint: `GET /v1/admin/events/stream` — see [backend.md](./backend.md)

---

## Shared types package

```
packages/shared/
  types/       # Order, Product, Customer DTOs
  schemas/     # Zod schemas shared web + mobile
```

Used by Shop, Admin, and Mobile for contract alignment.

---

## FE / Mobile tasks

| ID | Task | Est. | Owner |
|----|------|------|-------|
| P4-MOB-01 | Expo bootstrap + navigation | 3d | Mobile |
| P4-MOB-02 | Auth PKCE + token storage | 5d | Mobile |
| P4-MOB-03 | Shop tab (browse, PDP, cart) | 10d | Mobile |
| P4-MOB-04 | Checkout (Phase 2 payments) | 6d | Mobile |
| P4-MOB-05 | Scan tab + barcode | 4d | Mobile |
| P4-MOB-06 | Wallet tab (loyalty) | 4d | Mobile |
| P4-MOB-07 | Orders + Profile tabs | 5d | Mobile |
| P4-MOB-08 | Push notifications | 4d | Mobile |
| P4-MOB-09 | Offline cart MMKV | 3d | Mobile |
| P4-MOB-10 | App Store / Play submission | 5d | Mobile |
| P4-FE-01 | Admin reports A18 | 5d | FE |
| P4-FE-02 | Admin global search A19 | 3d | FE |
| P4-FE-03 | SSE integration in admin dashboard | 3d | FE |

**Total:** ~59 dev-days (Mobile ~49d, FE ~11d)

---

## Components to add

**Mobile:** tab navigator, `BarcodeScanner`, `LoyaltyWallet`, `OfflineCartSync`

**Admin:** `ReportsDashboard`, `GlobalSearchBar`, `SSEActivityFeed`

Base patterns → [docs-ba/05-frontend](../../docs-ba/05-frontend-specifications.md)
