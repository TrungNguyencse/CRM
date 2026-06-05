# Phase 3 — Specification: Omnichannel & Loyalty

> Maps to original plan: **Phase 3 — Omnichannel & Loyalty** (weeks 13–16)

---

## 1. Prerequisites

- Phase 2 complete (online payment + Temporal order flow)
- At least 1 physical store configured with registers
- POS hardware decision (optional): barcode scanner, receipt printer

---

## 2. Scope

### In scope

| Module | Features (from original docs) |
|--------|----------------------------|
| **Store & POS** | Stores, registers, cashier sessions, POS sales, cash drops |
| **CRM & Loyalty** | Tiers (Bronze–Platinum), points ledger, segments, interactions |
| **Notification** | Email + SMS + push templates; device tokens; event triggers |
| **Order** | POS channel orders; refund saga with points reversal |
| **Admin FE** | POS terminal, Customer 360, loyalty admin |
| **Shop FE** | Loyalty dashboard |

### Out of scope

- Mobile app (Phase 4) — but device token registry prepared
- OpenSearch (Phase 4)
- Multi-store franchise portal

---

## 3. New Screens

### Admin (+6 screens)

| ID | Screen | Route | Users |
|----|--------|-------|-------|
| A12 | POS terminal | `/admin/pos` | Cashier, StoreManager |
| A13 | Cashier session | `/admin/pos/session` | Cashier |
| A14 | Customer 360 | `/admin/customers/[id]` | Admin, StoreManager |
| A15 | Customer list / search | `/admin/customers` | Admin, StoreManager |
| A16 | Loyalty tiers admin | `/admin/loyalty/tiers` | Admin |
| A17 | Segments | `/admin/loyalty/segments` | Admin |

### Shop (+1 screen)

| ID | Screen | Route |
|----|--------|-------|
| S11 | Loyalty dashboard | `/account/loyalty` |

---

## 4. Backend Modules

### 4.1 Store & POS (`store` package)

**Tables:** `stores`, `registers`, `cashier_sessions`, `pos_sales`, `pos_sale_items`, `cash_drops`, `outbox`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `POST /v1/pos/sessions/open` | POST | Open cashier session |
| `POST /v1/pos/sessions/close` | POST | Close session + float |
| `POST /v1/pos/sales` | POST | Record POS sale |
| `POST /v1/pos/cash-drops` | POST | Record cash drop |
| `GET /v1/admin/stores` | GET | List stores |
| `GET /v1/admin/registers` | GET | Registers by store |

**Events:** `POSSaleCompleted`, `POSShiftOpened`, `POSShiftClosed`, `CashDropRecorded`

**Choreography (from original plan):**
- POS emits `POSSaleCompleted` → Inventory decrements stock → CRM grants points → Notification sends receipt

### 4.2 CRM & Loyalty (`crm` package)

**Tables:** `customers`, `loyalty_tiers`, `points_ledger`, `segments`, `segment_memberships`, `interactions`, `outbox`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/customers/me/loyalty` | GET | Points, tier, perks |
| `GET /v1/admin/customers` | GET | Search customers |
| `GET /v1/admin/customers/{id}` | GET | Customer 360 aggregate |
| `GET /v1/admin/loyalty/tiers` | GET | Tier config |
| `PUT /v1/admin/loyalty/tiers/{id}` | PUT | Update tier perks |
| `POST /v1/admin/segments` | POST | Create segment rule |
| `POST /v1/admin/customers/{id}/points/adjust` | POST | Manual adjust |

**Defaults:** 1 VND = 1 point; 24-month expiry; nightly tier re-evaluation job

**Events:** `LoyaltyPointsEarned`, `LoyaltyPointsRedeemed`, `LoyaltyTierChanged`, `SegmentMembershipChanged`

### 4.3 Notification (`notification` package)

**Tables:** `templates`, `notifications`, `subscriptions`, `device_tokens`, `outbox`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/admin/notifications/templates` | GET | Template CRUD |
| `POST /v1/notifications/subscribe` | POST | Channel opt-in |
| `POST /v1/notifications/device-tokens` | POST | Register device (web/mobile) |

**Channels:** SES (email), SNS (SMS), FCM/APNs (push via device token)

**Triggers:** `OrderConfirmed`, `OrderCompleted`, `ShipmentDispatched`, `LoyaltyTierChanged`, `LowStockAlert`

### 4.4 Refund Saga (Temporal)

Coordinates: Payment refund → Order status → Inventory restock → Loyalty point reversal → Notification

---

## 5. POS Terminal UX (A12)

| Area | Elements |
|------|----------|
| Product search | Barcode scan input + name/SKU search |
| Cart | Line items, qty, discounts (from pricing service) |
| Customer link | Optional — phone/email lookup for loyalty |
| Tender | Cash, card (records only MVP), change calculation |
| Actions | Complete sale, void line, open/close session |

**Roles:** Cashier can sell; StoreManager can void; Admin full access

---

## 6. Customer 360 (A14)

| Section | Data source |
|---------|-------------|
| Profile | Identity |
| Order history | Order (online + POS) |
| Points & tier | CRM |
| Segments | CRM |
| Interactions | CRM interaction log |
| Coupons used | Pricing |

---

## 7. Frontend Tasks (summary)

| Task ID | Task | Est. |
|---------|------|------|
| P3-FE-01 | POS terminal A12–A13 | 12d |
| P3-FE-02 | Customer 360 A14–A15 | 6d |
| P3-FE-03 | Loyalty admin A16–A17 | 5d |
| P3-FE-04 | Shop loyalty dashboard S11 | 4d |
| P3-FE-05 | Notification template admin | 3d |

---

## 8. Backend Tasks (summary)

| Task ID | Task | Est. |
|---------|------|------|
| P3-BE-01 | Store & POS module | 12d |
| P3-BE-02 | CRM & loyalty module | 12d |
| P3-BE-03 | Notification module (SMS + push) | 8d |
| P3-BE-04 | Refund saga + points reversal | 6d |
| P3-BE-05 | Nightly tier job + segment evaluator | 4d |
| P3-BE-06 | Multi-location inventory link | 5d |

---

## 9. Sprint Plan (14 weeks)

| Sprint | Weeks | Focus |
|--------|-------|-------|
| P3-S1 | 1–2 | Store/register setup + admin |
| P3-S2 | 3–5 | POS terminal FE + BE |
| P3-S3 | 6–7 | CRM schema + points earn on order/POS |
| P3-S4 | 8–9 | Loyalty dashboard + tier job |
| P3-S5 | 10–11 | Notification SMS/push + templates |
| P3-S6 | 12–13 | Customer 360 + segments |
| P3-S7 | 14 | Refund saga UAT |

---

## 10. Acceptance Criteria

- [ ] Cashier completes POS sale; stock decrements; receipt email/SMS sent
- [ ] Linked customer earns points; tier upgrades after threshold
- [ ] Customer views loyalty dashboard with correct balance
- [ ] Admin creates segment rule; memberships update nightly
- [ ] Refund on delivered order reverses points and restocks
- [ ] Low-stock alert triggers notification to admin

---

## 11. Reference

Original modules: [Features doc](../../convenience-store-platform-features%20(2).md) §8, §9, §10, §13 (POS)
