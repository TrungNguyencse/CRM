# 01 — Project Overview

## 1.1 Project Summary

Build an **MVP omnichannel convenience store platform** that lets customers browse products and place orders online, and lets store staff manage catalog, inventory, and orders through an admin dashboard.

This MVP focuses on the **online commerce loop** (browse → cart → checkout → place order → track order). **Payment is offline-only** (COD + manual bank transfer) to avoid third-party gateway fees and integration cost. Online payment (VNPay, MoMo, Stripe) is Phase 2. In-store POS, mobile app, loyalty tiers, and full microservices architecture are also deferred.

| Attribute | Value |
|-----------|-------|
| Project name | Convenience Store Platform — MVP |
| Budget | **220,000,000 VND** (fixed) |
| Team | **1 Frontend Developer** + **1 Backend Developer** |
| Duration | **3 months** (12 weeks) |
| Total effort | ~6 person-months |
| Target users | End customers (Shop), Store managers & staff (Admin) |
| Primary language (UI) | English (Vietnamese i18n optional Phase 2) |
| Currency | VND |
| Payment (MVP) | **COD** + **bank transfer** (manual admin confirmation) — **no gateway fees** |

---

## 1.2 Business Goals

| # | Goal | Success indicator |
|---|------|-------------------|
| G1 | Launch a working online store | Customers can complete a purchase end-to-end |
| G2 | Enable staff to manage catalog & stock | Admin can CRUD products and adjust inventory |
| G3 | Accept orders with offline payment | COD checkout + bank transfer with admin confirmation |
| G4 | Provide order visibility | Customer sees order status; admin can update status |
| G5 | Deploy to production-ready MVP infra | Staging + production on AWS with CI/CD |

---

## 1.3 Tech Stack

### Frontend

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Public Shop | **Next.js 14** (App Router, SSR/ISR) | SEO for product/category pages |
| Admin Dashboard | **Next.js 14** (CSR) | Internal tool; no SEO requirement |
| UI | Tailwind CSS + shadcn/ui | Fast, consistent components |
| State / data | React Query + Zod | Server state + form validation |
| Auth (Shop) | NextAuth.js + JWT from backend | Standard OIDC-ready pattern |
| Auth (Admin) | Same JWT; role-gated routes | Admin / StoreManager roles |

### Backend

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Runtime | **Quarkus 3** (Java 21) | Matches long-term plan; fast dev cycle |
| Architecture | **Modular monolith** (5 modules) | Fits 3-month budget vs 10 microservices |
| ORM | Hibernate Panache | Quarkus standard |
| Database | **PostgreSQL 16** (single instance, schema-per-module) | One RDS for MVP cost control |
| Auth | JWT (access + refresh tokens) | Self-contained MVP; Keycloak Phase 2 |
| Payment handling | **Order module** (no gateway) | COD + bank transfer; no Stripe/VNPay/MoMo in MVP |
| File storage | AWS S3 | Product images |
| Email | AWS SES | Order confirmation emails |

### DevOps

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Containerization | Docker + Docker Compose | Local dev + prod parity |
| CI/CD | GitHub Actions | Build, test, deploy |
| Hosting | AWS EC2 (app) + RDS (DB) + S3 + CloudFront | Lean vs full EKS |
| Secrets | AWS Secrets Manager | No secrets in repo |
| Monitoring | CloudWatch logs + basic uptime check | MVP observability |

### Explicitly NOT in MVP stack

| Item | Phase 2 rationale |
|------|---------------------|
| Kubernetes (EKS) | Cost & complexity |
| Kafka / MSK | Event bus not needed at MVP scale |
| Temporal | Simple DB transactions for order flow |
| OpenSearch | PostgreSQL full-text search |
| Keycloak | JWT auth sufficient for MVP |
| GraalVM native image | JVM mode for faster delivery |
| React Native mobile app | Separate 2–3 month effort |
| POS terminal | Hardware + UX complexity |

---

## 1.4 Screen & Page Inventory

### Public Shop — 10 screens

| ID | Screen | Route | Primary user |
|----|--------|-------|--------------|
| S01 | Home | `/` | Guest / Customer |
| S02 | Category listing | `/categories/[slug]` | Guest / Customer |
| S03 | Product detail (PDP) | `/products/[slug]` | Guest / Customer |
| S04 | Search results | `/search?q=` | Guest / Customer |
| S05 | Shopping cart | `/cart` | Guest / Customer |
| S06 | Checkout | `/checkout` | Customer (login required; select COD or bank transfer) |
| S07 | Login / Register | `/auth/login`, `/auth/register` | Guest |
| S08 | Account profile | `/account/profile` | Customer |
| S09 | Order history | `/account/orders` | Customer |
| S10 | Order detail / tracking | `/account/orders/[id]` | Customer |

### Admin Dashboard — 8 screens

| ID | Screen | Route | Primary user |
|----|--------|-------|--------------|
| A01 | Admin login | `/admin/login` | Staff |
| A02 | Dashboard | `/admin` | Admin, StoreManager |
| A03 | Product list | `/admin/products` | Admin, StoreManager |
| A04 | Product create / edit | `/admin/products/new`, `/admin/products/[id]` | Admin, StoreManager |
| A05 | Category management | `/admin/categories` | Admin |
| A06 | Inventory management | `/admin/inventory` | Admin, StoreManager |
| A07 | Order list | `/admin/orders` | Admin, StoreManager |
| A08 | Order detail | `/admin/orders/[id]` | Admin, StoreManager |

**Total: 18 screens**

---

## 1.5 Core Features (MVP)

### Customer-facing (Shop)

| Feature | Description |
|---------|-------------|
| Product browsing | Home, categories, PDP with images and pricing |
| Search | Full-text search on product name/SKU (PostgreSQL) |
| Shopping cart | Guest cart (session) + persisted cart when logged in |
| Checkout | Shipping address, payment method (COD / bank transfer), order summary |
| Account | Registration, login, profile edit |
| Order tracking | View order history and status timeline |
| Email notification | Order confirmation email (COD immediately; bank transfer after admin confirms) |

### Staff-facing (Admin)

| Feature | Description |
|---------|-------------|
| Authentication | Staff login with role-based access |
| Dashboard | Order count, revenue summary (last 30 days), low-stock alerts |
| Product management | CRUD products, upload images, assign categories |
| Category management | Create/edit category tree (max 2 levels for MVP) |
| Inventory | View stock per SKU, manual adjust (+/-), low-stock threshold |
| Order management | List/filter orders, confirm bank transfers, update status (Confirmed → Shipped → Delivered) |

### Backend modules (monolith)

| Module | Responsibility |
|--------|----------------|
| Identity | Users, auth, roles, staff/customer profiles |
| Catalog | Products, categories, media (S3) |
| Inventory | Stock levels, reservations, adjustments |
| Order | Cart, checkout, order lifecycle, payment method (COD / bank transfer) |

> **No separate Payment module in MVP.** Online gateway integration (VNPay, MoMo, Stripe, bank API) is scoped to Phase 2 — see Section 1.9.

### DevOps deliverables

| Deliverable | Description |
|-------------|-------------|
| Local environment | Docker Compose (app + Postgres) |
| CI pipeline | Lint, test, build on PR |
| Staging | EC2 + RDS staging instance |
| Production | EC2 + RDS production, S3, CloudFront for static assets |
| Runbook | Basic deploy rollback and backup procedures |

---

## 1.6 Budget Allocation

Estimated split of **220,000,000 VND** across workstreams:

| Workstream | % | Amount (VND) | Owner | Notes |
|------------|---|--------------|-------|-------|
| Frontend (Shop + Admin) | 33% | 72,600,000 | FE Dev | 18 screens, checkout without gateway UI |
| Backend (API + business logic) | 33% | 72,600,000 | BE Dev | 4 modules, ~35 REST endpoints |
| DevOps & infrastructure | 10% | 22,000,000 | BE Dev (shared) | AWS setup, CI/CD, Docker |
| QA & bug fix buffer | 14% | 30,800,000 | Both | +4% freed from removing payment gateway |
| Project management & docs | 5% | 11,000,000 | Both | BA docs, demos, client meetings |
| AWS infra (3 months) | 5% | 11,000,000 | Client-paid or included | EC2 t3.medium, RDS db.t3.micro, S3, SES |

**Budget note:** Removing online payment gateway integration saves ~**8–10 dev-days** (~28–35M VND) and eliminates recurring merchant fees (typically 1.5–3% per transaction + setup/monthly fees from VNPay, MoMo, banks). Savings reallocated to QA buffer.

> **Note:** AWS running costs (~3–4M VND/month for MVP sizing) should be confirmed as client-owned or included in the 220M cap.

---

## 1.7 High-Level Architecture (MVP)

```
┌─────────────┐     ┌─────────────┐
│  Shop       │     │  Admin      │
│  (Next.js)  │     │  (Next.js)  │
└──────┬──────┘     └──────┬──────┘
       │    HTTPS / REST   │
       └─────────┬─────────┘
                 ▼
       ┌─────────────────────┐
       │  Quarkus Monolith   │
       │  ┌───────┬────────┐ │
       │  │Identity│ Catalog│ │
       │  ├───────┼────────┤ │
       │  │Inventory│ Order │ │
       │  └───────┴────────┘ │
       └─────────┬───────────┘
                 │
         ┌───────┴───────┐
         ▼               ▼
    PostgreSQL         AWS S3
      (RDS)           (images)
```

**Payment in MVP:** handled inside Order module — no external payment provider.

---

## 1.9 Payment Strategy & Cost Rationale

### Why offline payment for MVP

| Factor | Online gateway (VNPay/MoMo/Stripe) | MVP (COD + bank transfer) |
|--------|-----------------------------------|---------------------------|
| Dev effort | 3–4 weeks (integration, webhooks, testing) | ~3 days (payment method selector + admin confirm) |
| Setup fees | Often 5–30M VND+ or merchant onboarding | **0 VND** |
| Transaction fees | ~1.5–3% per order + e-wallet fees | **0 VND** |
| Monthly maintenance | Gateway dashboard, reconciliation tools | Admin manual check |
| Legal/merchant account | Business registration, bank merchant ID | Uses existing store bank account |

### MVP payment methods

| Method | Customer experience | Order status after submit | When stock is deducted |
|--------|---------------------|---------------------------|------------------------|
| **COD** (Cash on Delivery) | Select COD → Place order → receive email | `CONFIRMED` immediately | On order placement |
| **Bank transfer** | Select transfer → see store bank details → optional transfer note → Place order | `AWAITING_PAYMENT` | When admin confirms payment |

### Phase 2 — online payment (separate quote)

| Provider | Est. integration | Est. recurring cost |
|----------|------------------|---------------------|
| VNPay | 3–4 weeks | ~1.5–2% + setup |
| MoMo | 2–3 weeks | ~1.5–2% + setup |
| Stripe (cards) | 2 weeks | ~2.9% + 30¢ (international) |
| Bank direct API | 4–6 weeks | Bank-specific |

Architecture will keep a **payment provider interface** in the Order module so Phase 2 plugs in without rewriting checkout.

---

## 1.10 Relationship to Full Platform Plan

The original technical plan describes 10 microservices, mobile app, POS, Kafka, Temporal, and EKS. This MVP **preserves the same domain model and tech choices** (Quarkus, Next.js, PostgreSQL) but delivers a **vertical slice** with **offline payment** to fit budget. Online payment service from the full plan is Phase 2.

| Full plan component | MVP status |
|---------------------|------------|
| Identity & Auth | ✅ Simplified (JWT, no Keycloak) |
| Product & Catalog | ✅ Full CRUD |
| Inventory | ✅ Basic (no inter-store transfers) |
| Pricing & Promotions | ⚠️ Base price only (no coupons/BOGO) |
| Order & Checkout | ✅ Online channel only |
| Payment | ⚠️ Offline only (COD + bank transfer); gateway Phase 2 |
| Shipping & Fulfillment | ⚠️ Simple status flow (no carrier API) |
| Store & POS | ❌ Phase 2 |
| CRM & Loyalty | ❌ Phase 2 |
| Notification | ⚠️ Order confirmation email only |
| Search (OpenSearch) | ⚠️ PostgreSQL full-text |
| Mobile app | ❌ Phase 2 |
