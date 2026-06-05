# Phase 3 — Backend

> **Owner:** BE dev · **Consumed by:** [frontend.md](./frontend.md)

---

## Store & POS (`store`)

**Tables:** `stores`, `registers`, `cashier_sessions`, `pos_sales`, `pos_sale_items`, `cash_drops`, `outbox`

| Endpoint | Method |
|----------|--------|
| `POST /v1/pos/sessions/open` | POST |
| `POST /v1/pos/sessions/close` | POST |
| `POST /v1/pos/sales` | POST |
| `POST /v1/pos/cash-drops` | POST |
| `GET /v1/admin/stores` | GET |
| `GET /v1/admin/registers` | GET |

**Events:** `POSSaleCompleted`, `POSShiftOpened`, `POSShiftClosed`, `CashDropRecorded`

**Choreography:** POS → `POSSaleCompleted` → Inventory decrements → CRM grants points → Notification sends receipt

---

## CRM & Loyalty (`crm`)

**Tables:** `customers`, `loyalty_tiers`, `points_ledger`, `segments`, `segment_memberships`, `interactions`, `outbox`

| Endpoint | Method |
|----------|--------|
| `GET /v1/customers/me/loyalty` | GET |
| `GET /v1/admin/customers` | GET |
| `GET /v1/admin/customers/{id}` | GET |
| `GET /v1/admin/loyalty/tiers` | GET |
| `PUT /v1/admin/loyalty/tiers/{id}` | PUT |
| `POST /v1/admin/segments` | POST |
| `POST /v1/admin/customers/{id}/points/adjust` | POST |

**Defaults:** 1 VND = 1 point; 24-month expiry; nightly tier re-evaluation

**Events:** `LoyaltyPointsEarned`, `LoyaltyPointsRedeemed`, `LoyaltyTierChanged`, `SegmentMembershipChanged`

---

## Notification (`notification`)

**Tables:** `templates`, `notifications`, `subscriptions`, `device_tokens`, `outbox`

| Endpoint | Method |
|----------|--------|
| `GET /v1/admin/notifications/templates` | GET |
| `POST /v1/notifications/subscribe` | POST |
| `POST /v1/notifications/device-tokens` | POST |

**Channels:** SES (email), SNS (SMS), FCM/APNs (push)

**Triggers:** `OrderConfirmed`, `OrderCompleted`, `ShipmentDispatched`, `LoyaltyTierChanged`, `LowStockAlert`

---

## Refund saga (Temporal)

Coordinates: Payment refund → Order status → Inventory restock → Loyalty point reversal → Notification

---

## BE tasks

| ID | Task | Est. |
|----|------|------|
| P3-BE-01 | Store & POS module | 12d |
| P3-BE-02 | CRM & loyalty module | 12d |
| P3-BE-03 | Notification module (SMS + push) | 8d |
| P3-BE-04 | Refund saga + points reversal | 6d |
| P3-BE-05 | Nightly tier job + segment evaluator | 4d |
| P3-BE-06 | Multi-location inventory link | 5d |

**Total:** ~47 dev-days

---

## Infrastructure (BE-owned)

| Resource | Purpose |
|----------|---------|
| SNS | SMS delivery |
| SES | Email templates expanded |
| Device token registry | Prepared for Phase 4 mobile push |
