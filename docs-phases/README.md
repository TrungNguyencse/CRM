# Phased Implementation Roadmap — Full Platform

> **Goal:** Evolve from [MVP (Phase 1)](../docs-ba/README.md) to the **full omnichannel platform** described in the original technical documents.  
> **References:** [Features by module](../convenience-store-platform-features%20(2).md) · [Full technical plan](../plan-convenienceStorePlatform%201%20(1).md)

---

## Phase Overview

| Phase | Name | Duration | Team (standard) | **Price (VND)** | Delivers |
|-------|------|----------|-----------------|-----------------|----------|
| **1** | [MVP](./phase-1-mvp/README.md) | 3 mo | 1 FE + 1 BE | **220,000,000** | Shop + Admin, COD/bank transfer |
| **2** | [Commerce & Payments](./phase-2-commerce-payments/README.md) | 4 mo | 1 FE + 1 BE | **295,000,000** | VNPay/MoMo, promotions, Temporal |
| **3** | [Omnichannel & Loyalty](./phase-3-omnichannel-loyalty/README.md) | 4 mo | 1 FE + 1 BE | **295,000,000** | POS, CRM, loyalty, SMS/push |
| **4** | [Mobile & Search](./phase-4-mobile-search/README.md) | 3.5 mo | 1 FE + 1 BE + 1 Mobile | **390,000,000** | Expo app, OpenSearch, reports |
| **5** | [Platform Scale](./phase-5-platform-scale/README.md) | 6 mo | 2 BE + 1 DevOps + 1 FE | **660,000,000** | Microservices, Kafka, EKS |
| | **TOTAL** | **~20.5 mo** | | **~1,860,000,000** | Full platform |

**Pricing detail:** [00-pricing-estimates.md](./00-pricing-estimates.md) · Anchor: **220M = 1 FE + 1 BE × 3 months**

---

## Architecture Evolution

```
Phase 1          Phase 2–3              Phase 4           Phase 5
────────         ─────────              ───────           ───────
Monolith    →    Monolith + Temporal →  + Mobile API  →   10 microservices
EC2/RDS     →    EC2/RDS + Redis     →  + OpenSearch  →   EKS + MSK + APISIX
JWT         →    JWT + Payment SPI   →  + Keycloak?   →   Keycloak OIDC
Postgres FT →    Postgres FT         →  OpenSearch    →   OpenSearch + events
COD/Transfer→    VNPay/MoMo/Stripe   →  same          →   same
1 warehouse →    Multi-location      →  + stores      →   50–500 stores
```

---

## Document Structure

Each phase folder contains:

| File | Purpose |
|------|---------|
| [00-pricing-estimates.md](./00-pricing-estimates.md) | **Pricing all phases** — rates, breakdowns, payment schedule |
| [00-phase-map.md](./00-phase-map.md) | Module coverage matrix |
| `README.md` | Summary, prerequisites, deliverables |
| `specification.md` | Full scope: modules, screens, BE/FE tasks, acceptance |

---

## Dependency Chain

```
Phase 1 (MVP) ──must complete──► Phase 2 (Payments need order flow)
       │
       └──► Phase 2 ──► Phase 3 (POS needs catalog + inventory; loyalty needs orders)
                │
                └──► Phase 3 ──► Phase 4 (Mobile consumes same APIs + loyalty)
                         │
                         └──► Phase 4 ──► Phase 5 (Split services when scale/events needed)
```

---

## Mapping to Original Technical Plan

| Original plan (24-week) | This roadmap |
|-------------------------|--------------|
| Phase 0 — Foundations (EKS, Kafka, Keycloak) | **Phase 5** (deferred for cost) |
| Phase 1 — Catalog & Identity | **Phase 1 MVP** (simplified) |
| Phase 2 — Commerce Loop | **Phase 2** |
| Phase 3 — Omnichannel & Loyalty | **Phase 3** |
| Phase 4 — Mobile + Hardening | **Phase 4** + part of **Phase 5** |
| Phase 5 — Production Readiness | **Phase 5** |

---

## How to Use

1. **Client / PO:** Read this README + each phase `README.md` for roadmap meetings.  
2. **Dev lead:** Use `specification.md` per phase for sprint planning.  
3. **After MVP sign-off:** Kick off Phase 2 scoping with client budget approval.
