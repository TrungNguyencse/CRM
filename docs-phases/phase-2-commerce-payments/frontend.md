# Phase 2 — Frontend

> **Owner:** FE dev · **Depends on:** [backend.md](./backend.md) APIs

---

## Updated screens

| ID | Screen | Change |
|----|--------|--------|
| S05 | Cart | Enable promo code field |
| S06 | Checkout | COD, transfer, **VNPay**, **MoMo**, Stripe; delivery vs **click-collect** |
| S06b | Payment return | **New** — `/checkout/return` gateway callback |

## New admin screens

| ID | Screen | Route |
|----|--------|-------|
| A09 | Promotions list | `/admin/promotions` |
| A10 | Promotion / coupon editor | `/admin/promotions/new`, `[id]` |
| A11 | Fulfillment / shipments | `/admin/shipments` |

---

## FE tasks

| ID | Task | Est. | Notes |
|----|------|------|-------|
| P2-FE-01 | Coupon on S05/S06 | 2d | Validate via `POST /v1/cart/apply-coupon` |
| P2-FE-02 | Payment method + gateway redirect | 4d | VNPay/MoMo redirect URLs |
| P2-FE-03 | S06b payment return page | 2d | Parse callback; call confirm API |
| P2-FE-04 | Click-collect slot picker | 3d | `GET /v1/fulfillment/pickup-slots` |
| P2-FE-05 | A09–A10 promotions admin | 5d | CRUD + bulk coupon generate UI |
| P2-FE-06 | A11 shipments admin | 4d | List, status, tracking |
| P2-FE-07 | Checkout integration tests | 3d | Playwright optional |

**Total:** ~23 dev-days

---

## Validation (key rules)

| Area | Rule |
|------|------|
| Promo code | Show error from BE; disable if invalid/expired |
| Payment method | Mutual exclusive with offline if gateway selected |
| Pickup slot | Required when delivery type = click-collect |
| S06b | Handle cancel/fail from gateway; show retry |

---

## Components to add

- `CouponInput` — cart + checkout
- `PaymentGatewaySelector` — replaces offline-only selector from Phase 1
- `PickupSlotPicker` — date/time slots
- `PromotionForm` — admin A10
- `ShipmentTable` — admin A11

Base patterns → [docs-ba/05-frontend](../../docs-ba/05-frontend-specifications.md)
