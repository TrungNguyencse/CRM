# Phase 4 — Specification: Mobile & Search

> Maps to original plan: **Phase 4 — Mobile + Hardening** (partial) + Search read-model

---

## 1. Prerequisites

- Phase 3 complete (loyalty, notifications, device token registry)
- Apple Developer + Google Play accounts (client-owned)
- OpenSearch cluster provisioned

---

## 2. Scope

### In scope

| Module | Features (from original docs) |
|--------|----------------------------|
| **Mobile App (Expo)** | Shop, Scan, Wallet, Orders, Profile tabs |
| **Search** | OpenSearch indices: products, customers, orders |
| **Admin FE** | Customer search, reports, SSE realtime |
| **Auth** | PKCE OIDC flow (Keycloak-ready or JWT) |

### Out of scope

- Microservice split (Phase 5)
- GraalVM native image (Phase 5)
- Full chaos engineering (Phase 5)

---

## 3. Mobile App Screens

### Tab structure (5 tabs, ~12 screens)

| Tab | Screens | Key features |
|-----|---------|--------------|
| **Shop** | Home, Category, PDP, Cart | Browse, add to cart, share product |
| **Scan** | Barcode scanner, Scan result | GTIN lookup, loyalty check-in |
| **Wallet** | Points, Tier, Coupons list | Loyalty from Phase 3 |
| **Orders** | Order list, Order detail | Track status, cancel if allowed |
| **Profile** | Profile, Settings, Notifications | PKCE login, push opt-in |

### Mobile-specific requirements

| Topic | Spec |
|-------|------|
| Auth | PKCE OIDC with Keycloak or JWT refresh |
| Offline | MMKV cart cache (sync on reconnect) |
| Push | Expo Push → Notification service → FCM/APNs |
| Deep links | `store://orders/{id}`, `store://products/{slug}` |
| Min OS | iOS 15+, Android 8+ |

---

## 4. Search Service (`search-indexer`)

### OpenSearch indices

| Index | Fields | Source events |
|-------|--------|---------------|
| `products` | name, sku, brand, category, price, stock, status | ProductCreated/Updated, PriceChanged, StockAdjusted |
| `customers` | name, email, phone, tier | UserRegistered, LoyaltyTierChanged |
| `orders` | orderNumber, customerEmail, status, total | OrderPlaced, OrderCompleted |

### Indexer options (Phase 4)

**Option A (simpler):** Poll outbox tables + bulk index (no Kafka yet)  
**Option B:** Kafka consumer if MSK already provisioned early

Recommend **Option A** for Phase 4; migrate to Kafka in Phase 5.

### APIs

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/products/search` | GET | OpenSearch-backed (replaces Postgres FTS) |
| `GET /v1/admin/customers/search` | GET | Customer search |
| `GET /v1/admin/orders/search` | GET | Order search |

---

## 5. Admin — New Screens

| ID | Screen | Route |
|----|--------|-------|
| A18 | Reports dashboard | `/admin/reports` |
| A19 | Global search | `/admin/search` |

### Reports (A18) — MVP metrics

| Report | Metrics |
|--------|---------|
| Sales | Revenue by day/week, online vs POS |
| Products | Top sellers, low stock |
| Customers | New registrations, tier distribution |
| Orders | Status breakdown, avg order value |

### SSE Realtime (Admin)

| Event | UI update |
|-------|-----------|
| `order.placed` | Dashboard order count +1 |
| `low_stock` | Alert badge on inventory nav |
| `pos.sale` | Recent activity feed |

Endpoint: `GET /v1/admin/events/stream` (SSE)

---

## 6. Mobile Tasks (summary)

| Task ID | Task | Est. |
|---------|------|------|
| P4-MOB-01 | Expo project bootstrap + navigation | 3d |
| P4-MOB-02 | Auth PKCE + token storage | 5d |
| P4-MOB-03 | Shop tab (browse, PDP, cart) | 10d |
| P4-MOB-04 | Checkout (reuse payment from Phase 2) | 6d |
| P4-MOB-05 | Scan tab + barcode | 4d |
| P4-MOB-06 | Wallet tab (loyalty) | 4d |
| P4-MOB-07 | Orders + Profile tabs | 5d |
| P4-MOB-08 | Push notifications | 4d |
| P4-MOB-09 | Offline cart MMKV | 3d |
| P4-MOB-10 | App Store / Play Store submission | 5d |

---

## 7. Backend & FE Tasks (summary)

| Task ID | Task | Est. |
|---------|------|------|
| P4-BE-01 | OpenSearch cluster + index mappings | 4d |
| P4-BE-02 | Search indexer service | 8d |
| P4-BE-03 | Migrate product search to OpenSearch | 3d |
| P4-BE-04 | Customer/order search APIs | 4d |
| P4-BE-05 | SSE admin events endpoint | 4d |
| P4-FE-01 | Admin reports A18 | 5d |
| P4-FE-02 | Admin global search A19 | 3d |
| P4-FE-03 | SSE integration in admin dashboard | 3d |

---

## 8. Shared Types Package

```
packages/shared/
  types/       # Order, Product, Customer DTOs
  schemas/     # Zod schemas shared web + mobile
```

Used by Shop, Admin, and Mobile for contract alignment.

---

## 9. Sprint Plan (12 weeks)

| Sprint | Weeks | Focus |
|--------|-------|-------|
| P4-S1 | 1–2 | OpenSearch + indexer |
| P4-S2 | 3–4 | Mobile auth + Shop tab |
| P4-S3 | 5–6 | Mobile checkout + Orders |
| P4-S4 | 7–8 | Scan + Wallet tabs |
| P4-S5 | 9–10 | Push, offline cart, admin reports |
| P4-S6 | 11–12 | SSE, store submission, UAT |

---

## 10. Acceptance Criteria

- [ ] Mobile app published to TestFlight + Play Internal Testing
- [ ] Customer completes purchase on mobile with VNPay/MoMo
- [ ] Barcode scan resolves product and shows loyalty eligibility
- [ ] Product search uses OpenSearch with < 200ms p95
- [ ] Admin customer search finds by email/phone
- [ ] New order appears on admin dashboard via SSE within 5s
- [ ] Reports show online vs POS revenue split

---

## 11. Reference

Original: [Features doc](../../convenience-store-platform-features%20(2).md) §11, §14
