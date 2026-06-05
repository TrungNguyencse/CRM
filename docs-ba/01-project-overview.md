# 01 — Project Overview

> **Price:** 220,000,000 VND · **Team:** 1 FE + 1 BE · **Duration:** 3 months · **Screens:** 18

---

## 1.1 Summary

MVP omnichannel convenience store: customers order online; staff manage catalog, inventory, and orders.

**Flow:** browse → cart → checkout → track order.  
**Payment:** COD + bank transfer (no gateway fees).  
**Deferred:** online payment, POS, mobile, microservices → [docs-phases](../docs-phases/README.md).

| Attribute | Value |
|-----------|-------|
| Budget | **220,000,000 VND** (fixed) |
| Team | 1 FE + 1 BE |
| Duration | 3 months |
| UI | English · Currency VND |

---

## 1.2 Business Goals

| # | Goal | Success indicator |
|---|------|-------------------|
| G1 | Working online store | End-to-end purchase |
| G2 | Catalog & stock management | Product CRUD + inventory |
| G3 | Offline payment | COD + bank transfer |
| G4 | Order visibility | Customer + admin tracking |
| G5 | Production deploy | Staging + prod on AWS |

---

## 1.3 Tech Stack

| Layer | Technology |
|-------|------------|
| Shop | Next.js 14 (SSR/ISR) |
| Admin | Next.js 14 (CSR) |
| UI | Tailwind, shadcn/ui, React Query, Zod |
| Backend | Quarkus 3, Java 21, PostgreSQL 16 |
| Auth | JWT (Keycloak → Phase 5) |
| Payment | COD + transfer in Order module |
| Hosting | EC2, RDS, S3, CloudFront |

**Not in MVP:** EKS, Kafka, Temporal, OpenSearch, Keycloak, mobile, POS.

Detail → [07-devops](./07-devops-and-deployment.md) · APIs → [06-backend](./06-backend-specifications.md)

---

## 1.4 Screen Inventory

### Shop — 10 screens → [03-shop](./03-shop-screen-specifications.md)

| ID | Screen | Route |
|----|--------|-------|
| S01 | Home | `/` |
| S02 | Category listing | `/categories/[slug]` |
| S03 | Product detail | `/products/[slug]` |
| S04 | Search | `/search` |
| S05 | Cart | `/cart` |
| S06 | Checkout | `/checkout` |
| S07 | Login / Register | `/auth/*` |
| S08 | Account profile | `/account/profile` |
| S09 | Order history | `/account/orders` |
| S10 | Order detail | `/account/orders/[id]` |

### Admin — 8 screens → [04-admin](./04-admin-screen-specifications.md)

| ID | Screen | Route |
|----|--------|-------|
| A01 | Login | `/admin/login` |
| A02 | Dashboard | `/admin` |
| A03 | Product list | `/admin/products` |
| A04 | Product edit | `/admin/products/*` |
| A05 | Categories | `/admin/categories` |
| A06 | Inventory | `/admin/inventory` |
| A07 | Order list | `/admin/orders` |
| A08 | Order detail | `/admin/orders/[id]` |

Wireframes → [mockups/index.html](./mockups/index.html)

---

## 1.5 Core Features

**Shop:** browsing, search, cart, checkout (COD/transfer), account, orders, email.  
**Admin:** dashboard, products, categories, inventory, orders, bank confirm.  
**Backend modules:** Identity, Catalog, Inventory, Order (4 modules).

---

## 1.6 Budget (220M VND)

| Workstream | VND | % |
|------------|-----|---|
| Frontend | 72,600,000 | 33% |
| Backend | 72,600,000 | 33% |
| DevOps | 22,000,000 | 10% |
| QA | 30,800,000 | 14% |
| PM & docs | 11,000,000 | 5% |
| AWS (3 mo) | 11,000,000 | 5% |

Phases 2–5 → [pricing](../docs-phases/00-pricing-estimates.md)

---

## 1.7 Architecture

```
Shop + Admin (Next.js) ──REST──► Quarkus Monolith
                                  (Identity|Catalog|Inventory|Order)
                                        │
                                   PostgreSQL + S3
```

**Payment:** offline in Order module; `PaymentProvider` SPI ready for Phase 2.

| Method | Status after order | Stock |
|--------|-------------------|-------|
| COD | CONFIRMED | Deduct immediately |
| Bank transfer | AWAITING_PAYMENT | Deduct on admin confirm |

Lifecycle detail → [02-scope § 2.5](./02-scope-and-assumptions.md#25-order-status-lifecycle)

---

## 1.8 MVP vs Full Platform

| Component | MVP | Later phase |
|-----------|-----|-------------|
| Payment | Offline | Phase 2 gateway |
| POS / CRM | — | Phase 3 |
| Mobile | — | Phase 4 |
| Microservices | Monolith | Phase 5 |

→ [docs-phases](../docs-phases/README.md)
