# Phase 4 — General

> **Owner:** PM / either dev · **Price:** 390,000,000 VND · **Duration:** ~3.5 months

---

## Goals

| # | Goal |
|---|------|
| G1 | iOS + Android app (Shop, Scan, Wallet, Orders, Profile) |
| G2 | OpenSearch for product/customer/order search |
| G3 | Admin reports + global search |
| G4 | Real-time admin updates via SSE |
| G5 | Push notifications (Expo → FCM/APNs) |

---

## Prerequisites

- Phase 3 complete (loyalty, notifications, device token registry)
- Apple Developer + Google Play accounts (client-owned)
- OpenSearch cluster provisioned

---

## Scope summary

| In | Out (later phases) |
|----|-------------------|
| Expo mobile, OpenSearch, admin reports, SSE | Microservice split (Ph5), GraalVM native (Ph5), chaos engineering (Ph5) |

Detail: [frontend.md](./frontend.md) · [backend.md](./backend.md)

---

## Sprint plan (12 weeks)

| Sprint | Weeks | Focus |
|--------|-------|-------|
| P4-S1 | 1–2 | OpenSearch + indexer |
| P4-S2 | 3–4 | Mobile auth + Shop tab |
| P4-S3 | 5–6 | Mobile checkout + Orders |
| P4-S4 | 7–8 | Scan + Wallet tabs |
| P4-S5 | 9–10 | Push, offline cart, admin reports |
| P4-S6 | 11–12 | SSE, store submission, UAT |

---

## Acceptance criteria

- [ ] Mobile app on TestFlight + Play Internal Testing
- [ ] Mobile purchase with VNPay/MoMo
- [ ] Barcode scan resolves product + loyalty eligibility
- [ ] Product search OpenSearch p95 < 200ms
- [ ] Admin customer search by email/phone
- [ ] New order on admin dashboard via SSE within 5s
- [ ] Reports show online vs POS revenue split

---

## Client-owned costs

Apple Dev $99/yr, OpenSearch ~$50–150 USD/month.

→ [Full pricing](../00-pricing-estimates.md#phase-4--mobile--search--390000000-vnd)

---

## Reference

[Features doc](../../convenience-store-platform-features%20(2).md) §11, §14
