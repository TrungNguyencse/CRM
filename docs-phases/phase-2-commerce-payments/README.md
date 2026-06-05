# Phase 2 — Commerce & Payments

> **Duration:** 3–4 months · **Team:** 2 FE + 2 BE · **Prerequisite:** Phase 1 MVP signed off

Upgrade the commerce loop to match the **original plan Phase 2** — online payments, promotions, fulfillment service, and Temporal order orchestration.

---

## Goals

| # | Goal |
|---|------|
| G1 | Accept online payments (VNPay and/or MoMo; Stripe optional) |
| G2 | Run promotions, coupons, and campaign management |
| G3 | Orchestrate checkout via Temporal `PlaceOrderWorkflow` with compensations |
| G4 | Support delivery + click-and-collect with pickup slots |
| G5 | Extract Pricing and Payment into separate modules (still deployable as monolith JAR modules) |

---

## Key Deliverables

| Layer | Deliverable |
|-------|-------------|
| **BE** | Pricing module, Payment module (SPI), Fulfillment module, Temporal workers |
| **FE Shop** | Coupon field at checkout, payment gateway redirect/iframe, click-collect option |
| **FE Admin** | Promotion CRUD, coupon generator, fulfillment/shipment management |
| **Infra** | Redis (sessions, idempotency), Temporal server (self-hosted or cloud) |

---

## Documents

| File | Contents |
|------|----------|
| [specification.md](./specification.md) | Full scope, screens, APIs, tasks, acceptance |

---

## Pricing

| Option | Team | Duration | **Price (VND)** |
|--------|------|----------|-----------------|
| **Standard** | 1 FE + 1 BE | 4 months | **295,000,000** |
| Accelerated | 2 FE + 2 BE | 2.5 months | 370,000,000 |

| Breakdown (standard) | Amount (VND) |
|----------------------|-------------|
| Frontend | 95,000,000 |
| Backend | 130,000,000 |
| DevOps + QA + PM | 70,000,000 |

**Client-owned extras:** VNPay/MoMo merchant setup (5–30M), transaction fees ~1.5–2%/order.

→ [Full pricing](../00-pricing-estimates.md#phase-2--commerce--payments--295000000-vnd)

---

## Next Phase

→ [Phase 3 — Omnichannel & Loyalty](../phase-3-omnichannel-loyalty/README.md)
