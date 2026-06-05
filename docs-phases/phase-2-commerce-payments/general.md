# Phase 2 — General

> **Owner:** PM / either dev · **Price:** 295,000,000 VND · **Duration:** ~4 months

---

## Goals

| # | Goal |
|---|------|
| G1 | Online payments (VNPay and/or MoMo; Stripe optional) |
| G2 | Promotions, coupons, campaign management |
| G3 | Temporal `PlaceOrderWorkflow` with compensations |
| G4 | Delivery + click-and-collect with pickup slots |
| G5 | Pricing + Payment as separate monolith modules |

---

## Prerequisites

- Phase 1 MVP in production
- Client merchant accounts: VNPay and/or MoMo
- `PaymentProvider` SPI stub from Phase 1 Order module

---

## Scope summary

| In | Out (later phases) |
|----|-------------------|
| Pricing, Payment SPI, Fulfillment, Temporal | POS (Ph3), loyalty redeem (Ph3), mobile (Ph4), Kafka split (Ph5) |

Detail: [frontend.md](./frontend.md) · [backend.md](./backend.md)

---

## Sprint plan (14 weeks)

| Sprint | Weeks | Focus |
|--------|-------|-------|
| P2-S1 | 1–2 | Pricing BE + promotion admin shell |
| P2-S2 | 3–4 | Coupon engine + shop apply |
| P2-S3 | 5–6 | VNPay sandbox |
| P2-S4 | 7–8 | MoMo + checkout payment UI |
| P2-S5 | 9–10 | Fulfillment + click-collect |
| P2-S6 | 11–12 | Temporal PlaceOrderWorkflow |
| P2-S7 | 13–14 | Refund saga, UAT |

---

## Acceptance criteria

- [ ] VNPay sandbox checkout completes end-to-end
- [ ] Coupon % discount at checkout
- [ ] Click-collect: pickup slot selected; admin sees booking
- [ ] PlaceOrderWorkflow compensates on payment failure (stock released)
- [ ] Admin creates BOGO + generates 100 coupon codes
- [ ] Admin refunds paid order via gateway API
- [ ] Checkout API p95 < 1s (excl. gateway redirect)

---

## Client-owned costs

VNPay/MoMo merchant setup (5–30M VND), transaction fees ~1.5–2%/order.

→ [Full pricing](../00-pricing-estimates.md#phase-2--commerce--payments--295000000-vnd)

---

## Reference

[Features doc](../../convenience-store-platform-features%20(2).md) §4, §6, §7, §15
