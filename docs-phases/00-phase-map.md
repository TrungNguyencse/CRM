# Phase Map — MVP to Full Platform

Visual mapping between **Phase 1 MVP** deliverables and **original full-platform modules**.

---

## Module Coverage by Phase

| # | Module (original docs) | Phase 1 MVP | Phase 2 | Phase 3 | Phase 4 | Phase 5 |
|---|------------------------|:-----------:|:-------:|:-------:|:-------:|:-------:|
| 1 | Identity & Auth | JWT basic | + audit hardening | + staff store assignment | + Keycloak prep | Keycloak OIDC |
| 2 | Product & Catalog | CRUD, 2-level categories | + variants, EAV attrs | + ltree deep hierarchy | same | + event emit |
| 3 | Inventory | Single location | + multi-location | + transfers | same | + partitioned movements |
| 4 | Pricing & Promotions | Base price | **Full engine** | + campaigns admin | coupons in mobile | + price lists |
| 5 | Order & Checkout | Online only | **+ Temporal saga** | + POS channel | + mobile channel | microservice split |
| 6 | Payment | COD + bank transfer | **VNPay/MoMo/Stripe** | refunds automated | tokenized methods | Payment service |
| 7 | Shipping & Fulfillment | Manual status | **+ click-collect** | + pickup slots | tracking in app | carrier APIs |
| 8 | Store & POS | — | — | **Full POS** | — | multi-store |
| 9 | CRM & Loyalty | — | — | **Full CRM** | loyalty UI mobile | segments |
| 10 | Notification | Email only | + templates admin | **SMS + push** | push via Expo | event-driven |
| 11 | Search | Postgres FTS | — | — | **OpenSearch** | Kafka indexer |
| 12 | Public Shop | 10 screens | + coupons checkout | + loyalty dashboard | — | i18n |
| 13 | Admin Dashboard | 8 screens | + campaigns | + POS + Customer 360 | + reports | SSE realtime |
| 14 | Mobile App | — | — | — | **Expo full app** | — |
| 15 | Temporal | — | **PlaceOrder + Refund** | Return saga | — | all sagas |
| 16 | Shared libs | minimal | outbox prep | — | — | full libs |
| 17 | API Gateway | — | — | — | — | **APISIX** |
| 18 | Infrastructure | EC2/RDS | + Redis | — | OpenSearch | **EKS/MSK/ArgoCD** |

---

## Screen Count by Phase

| Phase | New Shop screens | New Admin screens | New Mobile screens |
|-------|------------------|-------------------|--------------------|
| 1 MVP | 10 | 8 | — |
| 2 | +2 (coupons, payment) | +3 (promotions, campaigns, fulfillment) | — |
| 3 | +1 (loyalty dashboard) | +6 (POS, Customer 360, loyalty admin, segments) | — |
| 4 | — | +2 (reports, customer search) | 5 tabs (~12 screens) |
| **Full total** | **~13** | **~19** | **~12** |

---

## Event / Integration Milestones

| Milestone | Phase | Technology |
|-----------|-------|------------|
| Sync REST only | 1 | HTTP between FE ↔ monolith |
| Outbox table (no relay) | 1 | DB only, future-proof |
| Payment provider SPI | 2 | Interface in order module |
| Temporal workflows | 2 | Order + refund sagas |
| Kafka + Debezium CDC | 5 | Outbox → MSK |
| OpenSearch indexer | 4 | Kafka consumer or direct |
| SSE admin realtime | 4 | HTTP SSE |

---

## Pricing (standard plan)

> Full breakdown: [00-pricing-estimates.md](./00-pricing-estimates.md)

| Phase | Team | Duration | Person-months | **Price (VND)** |
|-------|------|----------|---------------|-----------------|
| 1 MVP | 1 FE + 1 BE | 3 mo | 6.0 | **220,000,000** |
| 2 | 1 FE + 1 BE | 4 mo | 8.0 | **295,000,000** |
| 3 | 1 FE + 1 BE | 4 mo | 8.0 | **295,000,000** |
| 4 | 1 FE + 1 BE + 1 Mobile | 3.5 mo | 10.5 | **390,000,000** |
| 5 | 2 BE + 1 DevOps + 1 FE | 6 mo | 18.0 | **660,000,000** |
| **Total** | | **~20.5 mo** | **50.5 PM** | **~1,860,000,000** |

**Base rate:** ~36.7M VND / person-month (from 220M ÷ 6 PM).

Excludes: ongoing AWS, payment gateway transaction fees, client content. Accelerated (larger team) options in pricing doc.
