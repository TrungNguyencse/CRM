# Phase 2 — Specification: Commerce & Payments

> Maps to original plan: **Phase 2 — Commerce Loop** (weeks 8–12) + payment gateway expansion

---

## 1. Prerequisites

- Phase 1 MVP in production
- Client merchant accounts: VNPay and/or MoMo (and Stripe if international cards needed)
- `PaymentProvider` SPI interface stub from Phase 1

---

## 2. Scope

### In scope

| Module | Features (from original docs) |
|--------|----------------------------|
| **Pricing & Promotions** | Price lists, promotions (% off, fixed, BOGO, bundle), coupons, campaigns |
| **Payment** | Payment intents, VNPay/MoMo/Stripe SPI, auth/capture/void, refunds, tokenized methods |
| **Fulfillment** | Shipments, click-and-collect, pickup slots, delivery attempts |
| **Order** | Temporal `PlaceOrderWorkflow`, saga state, multi-step checkout |
| **Shop FE** | Coupon at checkout, online payment UI, delivery vs pickup |
| **Admin FE** | Promotion/campaign management, shipment ops |

### Out of scope (later phases)

- POS terminal (Phase 3)
- Loyalty points redemption at checkout (Phase 3)
- Mobile app (Phase 4)
- Microservice split / Kafka (Phase 5)

---

## 3. New & Updated Screens

### Shop (+2 updated, 1 new flow)

| ID | Screen | Change |
|----|--------|--------|
| S05 | Cart | Enable promo code field |
| S06 | Checkout | Payment method: COD, bank transfer, **VNPay**, **MoMo**, (Stripe); delivery vs **click-collect** |
| S06b | Payment return | New — handle gateway redirect callback `/checkout/return` |

### Admin (+3 screens)

| ID | Screen | Route |
|----|--------|-------|
| A09 | Promotions list | `/admin/promotions` |
| A10 | Promotion / coupon editor | `/admin/promotions/new`, `/admin/promotions/[id]` |
| A11 | Fulfillment / shipments | `/admin/shipments` |

---

## 4. Backend Modules

### 4.1 Pricing & Promotions (`pricing` package)

**Tables:** `price_lists`, `price_list_items`, `promotions`, `coupons`, `coupon_redemptions`, `outbox`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/products/{slug}/price` | GET | Effective price (list + promotion) |
| `POST /v1/cart/apply-coupon` | POST | Validate and apply coupon |
| `DELETE /v1/cart/coupon` | DELETE | Remove coupon |
| `GET /v1/admin/promotions` | GET | List promotions |
| `POST /v1/admin/promotions` | POST | Create promotion |
| `POST /v1/admin/coupons/generate` | POST | Bulk coupon codes |

**Events (outbox, relay in Phase 5):** `CouponIssued`, `CouponRedeemed`, `PromotionActivated`, `PriceChanged`

### 4.2 Payment (`payment` package)

**Tables:** `payment_intents`, `payment_transactions`, `payment_methods`, `refunds`, `outbox`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `POST /v1/payments/intents` | POST | Create intent (idempotency key) |
| `POST /v1/payments/{id}/confirm` | POST | Confirm after gateway callback |
| `POST /v1/payments/{id}/void` | POST | Void authorization |
| `POST /v1/webhooks/vnpay` | POST | VNPay IPN |
| `POST /v1/webhooks/momo` | POST | MoMo IPN |
| `POST /v1/webhooks/stripe` | POST | Stripe webhook |
| `POST /v1/admin/orders/{id}/refund` | POST | Admin refund |

**SPI interface:**

```java
interface PaymentProvider {
  PaymentIntentResponse createIntent(CreateIntentRequest req);
  PaymentStatus queryStatus(String providerRef);
  RefundResult refund(String providerRef, BigDecimal amount);
}
```

Implementations: `VNPayProvider`, `MoMoProvider`, `StripeProvider`

### 4.3 Fulfillment (`fulfillment` package)

**Tables:** `shipments`, `shipment_items`, `pickup_slots`, `pickup_bookings`, `delivery_attempts`, `outbox`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/fulfillment/pickup-slots` | GET | Available slots by location |
| `POST /v1/admin/shipments` | POST | Create shipment |
| `PATCH /v1/admin/shipments/{id}` | PATCH | Update status, tracking |
| `GET /v1/admin/shipments` | GET | List/filter |

### 4.4 Temporal — `PlaceOrderWorkflow`

Steps (from original plan):

1. `CreateOrder` → PENDING  
2. `ReserveStock` — compensate: `ReleaseStock`  
3. `AuthorizePayment` — compensate: `VoidPayment`  
4. `ApplyCouponRedemption` — compensate: `RevertRedemption`  
5. `ConfirmOrder` → emit `OrderConfirmed`  
6. `CreateShipment` — compensate: cancel shipment  
7. `CapturePayment` → `OrderCompleted`  

**Refund workflow (basic):** refund payment → update order → restock → notify

---

## 5. Frontend Tasks (summary)

| Task ID | Task | Est. |
|---------|------|------|
| P2-FE-01 | Coupon input on S05/S06 + validation | 2d |
| P2-FE-02 | Payment method selector (gateway redirect) | 4d |
| P2-FE-03 | Payment return/callback page S06b | 2d |
| P2-FE-04 | Click-collect slot picker on S06 | 3d |
| P2-FE-05 | A09–A10 promotions admin | 5d |
| P2-FE-06 | A11 shipments admin | 4d |
| P2-FE-07 | Integration tests checkout paths | 3d |

---

## 6. Backend Tasks (summary)

| Task ID | Task | Est. |
|---------|------|------|
| P2-BE-01 | Pricing module + coupon engine | 10d |
| P2-BE-02 | Payment SPI + VNPay integration | 8d |
| P2-BE-03 | MoMo integration (if scoped) | 5d |
| P2-BE-04 | Fulfillment module + pickup slots | 8d |
| P2-BE-05 | Temporal setup + PlaceOrderWorkflow | 10d |
| P2-BE-06 | Refund workflow | 5d |
| P2-BE-07 | Redis for idempotency + sessions | 3d |

---

## 7. Infrastructure

| Resource | Purpose |
|----------|---------|
| Redis (ElastiCache) | Idempotency keys, cart session cache |
| Temporal Server | Self-hosted on EC2 or Temporal Cloud |
| Secrets Manager | VNPay/MoMo merchant keys |

---

## 8. Sprint Plan (14 weeks)

| Sprint | Weeks | Focus |
|--------|-------|-------|
| P2-S1 | 1–2 | Pricing module + admin UI shell |
| P2-S2 | 3–4 | Coupon engine + shop apply coupon |
| P2-S3 | 5–6 | Payment SPI + VNPay sandbox |
| P2-S4 | 7–8 | MoMo + checkout payment UI |
| P2-S5 | 9–10 | Fulfillment + click-collect |
| P2-S6 | 11–12 | Temporal PlaceOrderWorkflow |
| P2-S7 | 13–14 | Refund saga, UAT, production |

---

## 9. Acceptance Criteria

- [ ] Customer completes checkout with VNPay sandbox payment
- [ ] Coupon % discount applied correctly at checkout
- [ ] Click-collect: customer selects pickup slot; admin sees booking
- [ ] PlaceOrderWorkflow compensates on simulated payment failure (stock released)
- [ ] Admin can create BOGO promotion and generate 100 coupon codes
- [ ] Admin can refund paid order via gateway API
- [ ] p95 checkout API < 1s (excl. gateway redirect)

---

## 10. Reference

Original modules: [Features doc](../../convenience-store-platform-features%20(2).md) §4, §6, §7, §15
