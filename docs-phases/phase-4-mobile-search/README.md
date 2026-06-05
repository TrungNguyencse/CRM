# Phase 4 — Mobile & Search

> **Duration:** 3 months · **Team:** 2 FE + 1 BE + 1 Mobile dev · **Prerequisite:** Phase 3 complete

Deliver the **React Native / Expo mobile app** and **OpenSearch** read-model per original docs modules 11 and 14.

---

## Goals

| # | Goal |
|---|------|
| G1 | Launch iOS + Android app (Shop, Scan, Wallet, Orders, Profile tabs) |
| G2 | Replace PostgreSQL FTS with OpenSearch for product/customer/order search |
| G3 | Admin Customer 360 view and basic reports |
| G4 | Real-time admin updates via SSE |
| G5 | Push notifications to mobile via Expo → FCM/APNs |

---

## Key Deliverables

| Layer | Deliverable |
|-------|-------------|
| **Mobile** | Expo app — 5 tabs, PKCE OIDC, offline cart (MMKV), barcode scan |
| **BE** | Search indexer service, customer/order search APIs, SSE endpoints |
| **FE Admin** | Customer 360, reports dashboard, customer search |
| **Infra** | OpenSearch cluster, CloudFront for mobile assets |

---

## Documents

| File | Contents |
|------|----------|
| [specification.md](./specification.md) | Full scope, mobile screens, APIs, tasks, acceptance |

---

## Pricing

| Option | Team | Duration | **Price (VND)** |
|--------|------|----------|-----------------|
| **Standard** | 1 FE + 1 BE + 1 Mobile | 3.5 months | **390,000,000** |
| Accelerated | 2 FE + 1 BE + 1 Mobile | 3 months | 450,000,000 |

| Breakdown (standard) | Amount (VND) |
|----------------------|-------------|
| Mobile (Expo) | 140,000,000 |
| Frontend (Admin reports, SSE) | 55,000,000 |
| Backend (OpenSearch, indexer) | 110,000,000 |
| DevOps + QA + PM | 85,000,000 |

**Client-owned extras:** Apple Dev $99/yr, OpenSearch ~$50–150 USD/month.

→ [Full pricing](../00-pricing-estimates.md#phase-4--mobile--search--390000000-vnd)

---

## Next Phase

→ [Phase 5 — Platform Scale](../phase-5-platform-scale/README.md)
