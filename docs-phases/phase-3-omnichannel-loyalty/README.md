# Phase 3 — Omnichannel & Loyalty

> **Duration:** 3–4 months · **Team:** 2 FE + 2 BE · **Prerequisite:** Phase 2 complete

Implement in-store POS, CRM, loyalty program, and multi-channel notifications per **original plan Phase 3**.

---

## Goals

| # | Goal |
|---|------|
| G1 | Staff can run in-store sales via web POS terminal |
| G2 | Customers earn/redeem loyalty points (4 tiers) |
| G3 | Admin manages segments and basic campaigns |
| G4 | Notifications via email, SMS, and push (foundation for mobile) |
| G5 | Refund saga via Temporal (payment → order → stock → points reversal) |

---

## Key Deliverables

| Layer | Deliverable |
|-------|-------------|
| **BE** | Store & POS module, CRM & Loyalty module, Notification module |
| **FE Admin** | POS terminal, Customer 360, loyalty admin, segment management |
| **FE Shop** | Loyalty dashboard (points, tier, history) |
| **Infra** | SNS SMS, SES templates expanded, device token registry |

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
| Frontend (POS, Customer 360) | 100,000,000 |
| Backend (POS, CRM, Notification) | 125,000,000 |
| DevOps + QA + PM | 70,000,000 |

**Client-owned extras:** SMS costs (~500–2,000 VND/SMS), POS hardware optional.

→ [Full pricing](../00-pricing-estimates.md#phase-3--omnichannel--loyalty--295000000-vnd)

---

## Next Phase

→ [Phase 4 — Mobile & Search](../phase-4-mobile-search/README.md)
