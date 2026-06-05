# Phase 3 — General

> **Owner:** PM / either dev · **Price:** 295,000,000 VND · **Duration:** ~4 months

---

## Goals

| # | Goal |
|---|------|
| G1 | Web POS terminal for in-store sales |
| G2 | Loyalty points + 4 tiers (Bronze–Platinum) |
| G3 | Customer segments + admin campaigns |
| G4 | Email, SMS, push notifications |
| G5 | Refund saga with points reversal |

---

## Prerequisites

- Phase 2 complete (online payment + Temporal order flow)
- At least 1 physical store with registers configured
- POS hardware optional: barcode scanner, receipt printer

---

## Scope summary

| In | Out (later phases) |
|----|-------------------|
| Store & POS, CRM & Loyalty, Notification | Mobile app (Ph4), OpenSearch (Ph4), franchise portal |

Detail: [frontend.md](./frontend.md) · [backend.md](./backend.md)

---

## Sprint plan (14 weeks)

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

## Acceptance criteria

- [ ] Cashier completes POS sale; stock decrements; receipt email/SMS sent
- [ ] Linked customer earns points; tier upgrades after threshold
- [ ] Customer views loyalty dashboard with correct balance
- [ ] Admin creates segment rule; memberships update nightly
- [ ] Refund on delivered order reverses points and restocks
- [ ] Low-stock alert triggers notification to admin

---

## Client-owned costs

SMS (~500–2,000 VND/SMS), optional POS hardware.

→ [Full pricing](../00-pricing-estimates.md#phase-3--omnichannel--loyalty--295000000-vnd)

---

## Reference

[Features doc](../../convenience-store-platform-features%20(2).md) §8, §9, §10, §13 (POS)
