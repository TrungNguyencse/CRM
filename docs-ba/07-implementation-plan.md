# 07 — Implementation Plan (3 Months)

> **Duration:** 12 weeks · **Team:** 1 FE + 1 BE · **Budget:** 220,000,000 VND  
> **Methodology:** 2-week sprints · Client demo at end of each sprint

---

## 7.1 Timeline Overview

```
Week:  1  2  3  4  5  6  7  8  9  10  11  12
       ├── Sprint 1 ──┤├── Sprint 2 ──┤├── Sprint 3 ──┤├── Sprint 4 ──┤├── Sprint 5 ──┤├── Sprint 6 ──┤
       Foundations     Catalog+Admin    Commerce         Orders+Pay       Hardening        UAT+Launch
```

| Sprint | Weeks | Theme | Client demo |
|--------|-------|-------|-------------|
| 1 | 1–2 | Foundations | Local environment walkthrough |
| 2 | 3–4 | Catalog & Admin CRUD | Admin product management |
| 3 | 5–6 | Shop browsing + cart | Browse and add to cart |
| 4 | 7–8 | Checkout + offline payment | End-to-end COD + bank transfer order |
| 5 | 9–10 | Orders, inventory, emails | Admin order management |
| 6 | 11–12 | UAT, staging, production | Production launch |

---

## 7.2 Detailed Sprint Plan

### Sprint 1 — Foundations (Weeks 1–2)

| Task | Owner | Deliverable |
|------|-------|-------------|
| Monorepo setup, branch strategy | BE | Git repo with `/api`, `/web/shop`, `/web/admin` |
| Quarkus project + Postgres + Flyway | BE | API boots, health check passes |
| Identity module: register, login, JWT | BE | Auth endpoints working |
| Docker Compose local env | BE | `docker compose up` documented |
| Next.js shop + admin scaffolding | FE | Layouts, routing shell, Tailwind |
| Auth UI (login/register) | FE | S07 screens (mock API OK Week 1) |
| AWS staging skeleton | BE | EC2 + RDS provisioned |
| CI: PR checks | BE | GitHub Actions green |

**Milestone M1:** Developer environment documented; auth flow works locally.

---

### Sprint 2 — Catalog & Admin (Weeks 3–4)

| Task | Owner | Deliverable |
|------|-------|-------------|
| Catalog module: products, categories, S3 upload | BE | Admin catalog APIs |
| Inventory module: stock levels, adjust | BE | Inventory APIs |
| Admin A03–A06 screens | FE | Product list, create/edit, categories, inventory |
| Admin A01–A02 login + dashboard | FE | Staff login, KPI widgets |
| Seed data script | BE | 20 sample products, 5 categories |
| PostgreSQL full-text search | BE | Search endpoint |

**Milestone M2:** Admin can manage full catalog; client reviews A03–A06 with mockups.

---

### Sprint 3 — Shop Browsing (Weeks 5–6)

| Task | Owner | Deliverable |
|------|-------|-------------|
| Public catalog APIs wired | BE | Performance check on product list |
| Shop S01–S05 screens | FE | Home, category, PDP, search, cart |
| Cart persistence (guest + user) | BE | Cart merge on login |
| Responsive mobile layout | FE | Tested 375px |
| CloudFront + S3 image delivery | BE | CDN URLs on product images |

**Milestone M3:** Customer can browse and manage cart; client UAT on mobile.

---

### Sprint 4 — Checkout & Order Placement (Weeks 7–8)

| Task | Owner | Deliverable |
|------|-------|-------------|
| Order module: checkout (COD + bank transfer) | BE | Checkout flow, no gateway |
| Admin confirm-payment endpoint | BE | Bank transfer confirmation |
| Shop S06 checkout (payment method selector) | FE | COD + bank transfer UI |
| Order confirmation / awaiting emails (SES) | BE | Email templates |
| Shop S09–S10 order views | FE | Order history + detail |
| Staging deployment | BE | Staging URLs live |

**Milestone M4:** End-to-end order on staging — **COD path** + **bank transfer path with admin confirm**.

---

### Sprint 5 — Order Ops & Polish (Weeks 9–10)

| Task | Owner | Deliverable |
|------|-------|-------------|
| Admin A07–A08 order management + confirm payment | FE | Status updates, bank confirm button |
| Cancel flows + stock restore | BE | Cancel + auto-expire AWAITING_PAYMENT |
| Shop S08 account profile | FE | Profile + addresses |
| Scheduled jobs (reservation expiry) | BE | Jobs running |
| Dashboard KPIs | BE + FE | A02 complete |
| Bug fixes from Sprint 4 UAT | Both | UAT log cleared |

**Milestone M5:** Admin full operations; customer account complete.

---

### Sprint 6 — Launch (Weeks 11–12)

| Task | Owner | Deliverable |
|------|-------|-------------|
| Production environment | BE | HTTPS live |
| Security pass (OWASP top 10 checklist) | BE | Checklist signed |
| Performance smoke test | BE | p95 < 1s at 20 concurrent |
| FE accessibility pass | FE | Form labels, focus states |
| Client UAT (3 days) | Client | Signed acceptance (02-scope checklist) |
| Documentation handover | Both | README, API docs, runbooks |
| Hypercare buffer | Both | 3 days post-launch support |

**Milestone M6:** Production launch + acceptance sign-off.

---

## 7.3 Work Allocation by Role

### Frontend Developer (~60 dev-days)

| Area | Days | Screens |
|------|------|---------|
| Project setup, shared components | 5 | — |
| Shop screens S01–S10 | 28 | 10 |
| Admin screens A01–A08 | 22 | 8 |
| Integration, bug fixes, responsive | 5 | — |

### Backend Developer (~60 dev-days)

| Area | Days |
|------|------|
| Project setup, CI, Docker, AWS | 8 |
| Identity module | 8 |
| Catalog + S3 | 10 |
| Inventory | 6 |
| Order + checkout | 10 |
| Payment (offline logic in Order) | 2 |
| Admin APIs + dashboard aggregations | 6 |
| Email, schedulers, hardening | 6 |

---

## 7.4 Meeting Cadence

| Meeting | Frequency | Attendees | Purpose |
|---------|-----------|-----------|---------|
| Sprint planning | Biweekly (Monday W1/W3/…) | Client, PO, Devs | Confirm sprint backlog |
| Demo | Biweekly (Friday) | Client, PO, Devs | Show working software |
| Standup | Daily 15 min | Devs | Blockers only (async OK) |
| Client review | Weekly 30 min (optional) | Client, PO | Scope check, approvals |
| UAT session | Week 11 (3 days) | Client, QA | Acceptance testing |

---

## 7.5 Deliverables Summary

| # | Deliverable | Format | When |
|---|-------------|--------|------|
| D1 | BA documentation (this set) | Markdown + mockups | Week 0 (now) |
| D2 | OpenAPI specification | YAML in repo | Week 4 |
| D3 | Staging environment | URLs | Week 8 |
| D4 | Admin + Shop applications | Deployed | Week 12 |
| D5 | Source code | GitHub repository | Week 12 |
| D6 | Deploy & rollback runbook | Markdown | Week 12 |
| D7 | UAT sign-off sheet | PDF / signed email | Week 12 |

---

## 7.6 Dependencies on Client

| Item | Required by | Blocking |
|------|-------------|----------|
| AWS account access or approved billing | Week 1 | Staging |
| Store bank account details for transfer display | Week 2 | Bank transfer checkout |
| Domain + DNS | Week 7 | Staging URLs |
| Logo, brand colors, hero image | Week 2 | UI polish |
| Sample product data (or approve seed data) | Week 3 | Demos |
| UAT testers (2 people, 3 days) | Week 11 | Sign-off |
| Production go/no-go decision | Week 12 | Launch |

---

## 7.7 Phase 2 Backlog (Post-MVP)

Prioritized for future quoting (not in 220M scope):

| Priority | Feature | Est. effort | Est. 3rd-party cost |
|----------|---------|-------------|---------------------|
| P1 | VNPay integration | 3–4 weeks | ~1.5–2% per txn + setup |
| P2 | MoMo e-wallet integration | 2–3 weeks | ~1.5–2% per txn + setup |
| P3 | Stripe / card payments | 2 weeks | ~2.9% per txn |
| P4 | Promotions & coupon engine | 4 weeks | — |
| P5 | POS terminal (web) | 6–8 weeks | — |
| P6 | Mobile app (Expo) | 8–10 weeks | — |
| P7 | Loyalty points & tiers | 4 weeks | — |
| P8 | Split to microservices + Kafka | 8–12 weeks | — |

---

## 7.8 Risk Buffer

Weeks 11–12 include **explicit buffer** (~10 dev-days combined) for:

- UAT bug fixes  
- Bank transfer confirmation edge cases  
- Mobile responsive issues  
- Deployment issues  

If buffer unused, deliver optional: CSV export on order list (admin) or product duplicate action.
