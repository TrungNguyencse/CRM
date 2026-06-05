# BA Documentation — Omnichannel Convenience Store Platform (MVP)

> **Version:** 1.0 · **Date:** 30 May 2026  
> **Audience:** Client, Product Owner, Development team (meetings & sign-off)  
> **Budget:** 220,000,000 VND · **Team:** 1 FE + 1 BE · **Duration:** 3 months

---

## Purpose

This documentation set translates the original architecture-first technical plan into **business-oriented, screen-level specifications** so the client can understand *what will be built, who uses it, and what rules apply* before development starts. Scope has been **deliberately reduced** to fit budget and timeline.

---

## Document Structure

| # | File | Contents |
|---|------|----------|
| 1 | [01-project-overview.md](./01-project-overview.md) | Executive summary, tech stack, screen count, core features, budget allocation |
| 2 | [02-scope-and-assumptions.md](./02-scope-and-assumptions.md) | In/out of scope, assumptions, risks, acceptance criteria |
| 3 | [03-shop-screen-specifications.md](./03-shop-screen-specifications.md) | Detailed specs for 10 public shop screens (FE + validation) |
| 4 | [04-admin-screen-specifications.md](./04-admin-screen-specifications.md) | Detailed specs for 8 admin screens (FE + validation) |
| 5 | [05-backend-specifications.md](./05-backend-specifications.md) | APIs, business rules, BE validation by module |
| 6 | [06-devops-and-deployment.md](./06-devops-and-deployment.md) | MVP infrastructure, CI/CD, environments, operational security |
| 7 | [07-implementation-plan.md](./07-implementation-plan.md) | 3-month roadmap, milestones, deliverables, meeting cadence |
| — | [mockups/index.html](./mockups/index.html) | Wireframe mockups for key screens (open in browser) |

---

## Quick Reference (Executive Summary)

| Item | Value |
|------|-------|
| **Total screens** | **18** (10 public Shop + 8 Admin) |
| **Backend** | 1 Quarkus modular monolith, 4 business modules (no payment gateway) |
| **Frontend** | 2 Next.js 14 apps (Shop SSR + Admin CSR) |
| **Mobile / POS** | ❌ Out of MVP scope (Phase 2) |
| **Payments** | COD + manual bank transfer (no payment gateway fees in MVP) |
| **Deployment** | Docker + lean AWS (EC2/RDS/S3) — no EKS/MSK |

---

## How to Use in Meetings

1. **Kickoff:** Read `01` + `02` → align on scope and exclusions.  
2. **UX review:** Open `mockups/index.html` alongside `03` and `04`.  
3. **Technical review:** `05` + `06` for developers; client only needs the summary in `01`.  
4. **Sign-off:** Accept deliverables using the checklist in `02` and milestones in `07`.
