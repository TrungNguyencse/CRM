# Phase 2 — Backend

> **Owner:** BE dev · **Consumed by:** [frontend.md](./frontend.md)

---

## Modules

### Pricing & Promotions (`pricing`)

**Tables:** `price_lists`, `price_list_items`, `promotions`, `coupons`, `coupon_redemptions`, `outbox`

| Endpoint | Method |
|----------|--------|
| `GET /v1/products/{slug}/price` | GET |
| `POST /v1/cart/apply-coupon` | POST |
| `DELETE /v1/cart/coupon` | DELETE |
| `GET /v1/admin/promotions` | GET |
| `POST /v1/admin/promotions` | POST |
| `POST /v1/admin/coupons/generate` | POST |

**Events (outbox):** `CouponIssued`, `CouponRedeemed`, `PromotionActivated`, `PriceChanged`

### Payment (`payment`)

**Tables:** `payment_intents`, `payment_transactions`, `payment_methods`, `refunds`, `outbox`

| Endpoint | Method |
|----------|--------|
| `POST /v1/payments/intents` | POST |
| `POST /v1/payments/{id}/confirm` | POST |
| `POST /v1/payments/{id}/void` | POST |
| `POST /v1/webhooks/vnpay` | POST |
| `POST /v1/webhooks/momo` | POST |
| `POST /v1/webhooks/stripe` | POST |
| `POST /v1/admin/orders/{id}/refund` | POST |

```java
interface PaymentProvider {
  PaymentIntentResponse createIntent(CreateIntentRequest req);
  PaymentStatus queryStatus(String providerRef);
  RefundResult refund(String providerRef, BigDecimal amount);
}
```

Implementations: `VNPayProvider`, `MoMoProvider`, `StripeProvider`

### Fulfillment (`fulfillment`)

**Tables:** `shipments`, `shipment_items`, `pickup_slots`, `pickup_bookings`, `delivery_attempts`, `outbox`

| Endpoint | Method |
|----------|--------|
| `GET /v1/fulfillment/pickup-slots` | GET |
| `POST /v1/admin/shipments` | POST |
| `PATCH /v1/admin/shipments/{id}` | PATCH |
| `GET /v1/admin/shipments` | GET |

### Temporal — `PlaceOrderWorkflow`

1. CreateOrder → PENDING  
2. ReserveStock — compensate: ReleaseStock  
3. AuthorizePayment — compensate: VoidPayment  
4. ApplyCouponRedemption — compensate: RevertRedemption  
5. ConfirmOrder  
6. CreateShipment — compensate: cancel  
7. CapturePayment → OrderCompleted  

**Refund workflow:** refund → order update → restock → notify

---

## BE tasks

| ID | Task | Est. |
|----|------|------|
| P2-BE-01 | Pricing + coupon engine | 10d |
| P2-BE-02 | Payment SPI + VNPay | 8d |
| P2-BE-03 | MoMo integration | 5d |
| P2-BE-04 | Fulfillment + pickup slots | 8d |
| P2-BE-05 | Temporal + PlaceOrderWorkflow | 10d |
| P2-BE-06 | Refund workflow | 5d |
| P2-BE-07 | Redis idempotency + sessions | 3d |

**Total:** ~49 dev-days

---

## Infrastructure (BE-owned)

| Resource | Purpose |
|----------|---------|
| Redis (ElastiCache) | Idempotency, cart cache |
| Temporal Server | EC2 or Temporal Cloud |
| Secrets Manager | VNPay/MoMo keys |

---

## Business rules

| Rule | Module |
|------|--------|
| Coupon max uses / per-user limit | Pricing |
| Webhook signature verification | Payment |
| Pickup slot capacity | Fulfillment |
| Saga compensation on any step fail | Temporal |
