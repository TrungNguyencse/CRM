# Pricing Estimates — All Phases

> **Anchor:** Phase 1 MVP = **220,000,000 VND** for **1 FE + 1 BE × 3 months**  
> **Audience:** Client proposals, contract scoping, roadmap meetings

---

## 1. Base Rate (derived from Phase 1)

| Metric | Value | Formula |
|--------|-------|---------|
| Phase 1 total | **220,000,000 VND** | Fixed anchor |
| Team | 1 FE + 1 BE | |
| Duration | 3 months | |
| Person-months | **6 PM** | 2 people × 3 months |
| **Rate per person-month** | **~36,700,000 VND** | 220M ÷ 6 |
| **Rate per dev-day** | **~1,830,000 VND** | 220M ÷ 120 dev-days† |

†Assumes ~20 billable dev-days per person per month (sprints, meetings, QA included in phase price).

### Role rates (for mixed teams)

| Role | Person-month (VND) | Notes |
|------|-------------------|-------|
| Frontend Developer | 36,700,000 | Base rate |
| Backend Developer | 36,700,000 | Base rate |
| Mobile Developer (React Native) | 40,000,000 | +9% vs base |
| DevOps Engineer | 42,000,000 | +14% vs base |

---

## 2. Summary — All Phases

### Standard plan (cost-optimised: mostly 1 FE + 1 BE)

| Phase | Name | Team | Duration | Person-months | **Price (VND)** |
|-------|------|------|----------|---------------|-----------------|
| **1** | MVP — Online Store | 1 FE + 1 BE | 3 months | 6.0 | **220,000,000** |
| **2** | Commerce & Payments | 1 FE + 1 BE | 4 months | 8.0 | **295,000,000** |
| **3** | Omnichannel & Loyalty | 1 FE + 1 BE | 4 months | 8.0 | **295,000,000** |
| **4** | Mobile & Search | 1 FE + 1 BE + 1 Mobile | 3.5 months | 10.5 | **390,000,000** |
| **5** | Platform Scale | 2 BE + 1 DevOps + 1 FE | 6 months | 18.0 | **660,000,000** |
| | **TOTAL (Phase 1–5)** | | **~20.5 months** | **50.5 PM** | **~1,860,000,000** |

> **~1.86 tỷ VNĐ** for the full platform (standard plan). Phase 1 is fixed at 220M; Phases 2–5 are estimates ±10%.

### Accelerated plan (faster delivery, larger team)

| Phase | Team | Duration | **Price (VND)** | vs Standard |
|-------|------|----------|-----------------|-------------|
| 2 | 2 FE + 2 BE | 2.5 months | **370,000,000** | +75M, −1.5 months |
| 3 | 2 FE + 2 BE | 2.5 months | **370,000,000** | +75M, −1.5 months |
| 4 | 2 FE + 1 BE + 1 Mobile | 3 months | **450,000,000** | +60M, −0.5 months |
| 5 | 3 BE + 1 DevOps + 1 FE | 5 months | **730,000,000** | +70M, −1 month |

---

## 3. Phase-by-Phase Breakdown

### Phase 1 — MVP · **220,000,000 VND** ✓ Fixed

| Item | Amount (VND) | % |
|------|-------------|---|
| Frontend (Shop + Admin, 18 screens) | 72,600,000 | 33% |
| Backend (4 modules, ~35 APIs) | 72,600,000 | 33% |
| DevOps (Docker, CI/CD, AWS lean) | 22,000,000 | 10% |
| QA & stabilisation | 30,800,000 | 14% |
| PM, BA docs, client meetings | 11,000,000 | 5% |
| AWS infra (3 months)‡ | 11,000,000 | 5% |
| **Total** | **220,000,000** | 100% |

‡Can be excluded from dev contract if client pays AWS directly (~3–4M VND/month).

**Deliverables:** [docs-ba/](../docs-ba/README.md) — 18 screens, COD + bank transfer, monolith.

---

### Phase 2 — Commerce & Payments · **295,000,000 VND**

| Item | Amount (VND) | Notes |
|------|-------------|-------|
| Frontend | 95,000,000 | Coupon UI, payment redirect, click-collect, 3 admin screens |
| Backend | 130,000,000 | Pricing, Payment SPI (VNPay/MoMo), Fulfillment, Temporal |
| DevOps | 25,000,000 | Redis, Temporal server |
| QA & integration testing | 30,000,000 | Payment sandbox, saga compensation tests |
| PM & documentation | 15,000,000 | |
| **Subtotal dev** | **295,000,000** | 1 FE + 1 BE × 4 months |

**Effort basis:** ~72 new dev-days (BE 49 + FE 23) × complexity factor 1.35 for payment/Temporal ≈ 8 PM.

| Option | Price (VND) | Team | Duration |
|--------|-------------|------|----------|
| **Standard** | **295,000,000** | 1 FE + 1 BE | 4 months |
| Accelerated | 370,000,000 | 2 FE + 2 BE | 2.5 months |

**Client-owned extras (not in dev price):**

| Item | Est. cost |
|------|-----------|
| VNPay merchant setup | 5–30M VND (one-time) |
| MoMo merchant setup | 5–20M VND (one-time) |
| Transaction fees | ~1.5–2% per order (ongoing) |
| Temporal Cloud (optional) | ~$200 USD/month |

---

### Phase 3 — Omnichannel & Loyalty · **295,000,000 VND**

| Item | Amount (VND) | Notes |
|------|-------------|-------|
| Frontend | 100,000,000 | POS terminal, Customer 360, loyalty admin, shop loyalty page |
| Backend | 125,000,000 | Store/POS, CRM, Notification (SMS/push) |
| DevOps | 20,000,000 | SNS SMS config, push certificates |
| QA & UAT | 35,000,000 | POS flow, points accuracy, refund saga |
| PM & documentation | 15,000,000 | |
| **Subtotal dev** | **295,000,000** | 1 FE + 1 BE × 4 months |

**Effort basis:** ~77 dev-days (BE 47 + FE 30) ≈ 8 PM.

| Option | Price (VND) | Team | Duration |
|--------|-------------|------|----------|
| **Standard** | **295,000,000** | 1 FE + 1 BE | 4 months |
| Accelerated | 370,000,000 | 2 FE + 2 BE | 2.5 months |

**Client-owned extras:**

| Item | Est. cost |
|------|-----------|
| SMS (SNS) | ~500–2,000 VND/SMS |
| POS hardware (optional) | 5–15M VND/device |
| Apple/Google push certs | Free (dev accounts required) |

---

### Phase 4 — Mobile & Search · **390,000,000 VND**

| Item | Amount (VND) | Notes |
|------|-------------|-------|
| Mobile (Expo) | 140,000,000 | 5 tabs, ~12 screens, push, offline cart |
| Frontend (Admin) | 55,000,000 | Reports, global search, SSE |
| Backend | 110,000,000 | OpenSearch, indexer, SSE, search APIs |
| DevOps | 35,000,000 | OpenSearch cluster, app store pipeline |
| QA & device testing | 35,000,000 | iOS + Android matrix |
| PM & store submission | 15,000,000 | TestFlight, Play Internal |
| **Subtotal dev** | **390,000,000** | 1 FE + 1 BE + 1 Mobile × 3.5 months |

**Effort basis:** Mobile 49d + BE 23d + FE 11d; mobile rate premium → 10.5 PM blended.

| Option | Price (VND) | Team | Duration |
|--------|-------------|------|----------|
| **Standard** | **390,000,000** | 1 FE + 1 BE + 1 Mobile | 3.5 months |
| Accelerated | 450,000,000 | 2 FE + 1 BE + 1 Mobile | 3 months |

**Client-owned extras:**

| Item | Est. cost |
|------|-----------|
| Apple Developer Program | $99 USD/year |
| Google Play Console | $25 USD one-time |
| OpenSearch (AWS) | ~$50–150 USD/month |

---

### Phase 5 — Platform Scale · **660,000,000 VND**

| Item | Amount (VND) | Notes |
|------|-------------|-------|
| Backend (service extraction) | 320,000,000 | 10 microservices, Kafka, outbox relay |
| DevOps | 180,000,000 | EKS, MSK, Terraform, Helm, ArgoCD, APISIX |
| Frontend (auth migration) | 45,000,000 | Keycloak OIDC on Shop/Admin/Mobile |
| QA (Pact, k6, chaos) | 70,000,000 | Load test 200 RPS, chaos experiments |
| PM, runbooks, pilot rollout | 45,000,000 | 5 → 50 stores pilot support |
| **Subtotal dev** | **660,000,000** | 2 BE + 1 DevOps + 1 FE × 6 months |

**Effort basis:** ~18 person-months (largest phase; aligns with original 24-week infra plan).

| Option | Price (VND) | Team | Duration |
|--------|-------------|------|----------|
| **Standard** | **660,000,000** | 2 BE + 1 DevOps + 1 FE | 6 months |
| Accelerated | 730,000,000 | 3 BE + 1 DevOps + 1 FE | 5 months |
| Minimal (infra only, no split) | 450,000,000 | 1 DevOps + 1 BE | 4 months |

**Client-owned extras (significant):**

| Item | Est. cost |
|------|-----------|
| AWS EKS + MSK + RDS×10 | ~$800–2,000 USD/month at medium scale |
| Keycloak hosting | ~$50–100 USD/month |
| Security pen test (external) | 50–150M VND |

---

## 4. Cumulative Investment

| After phase | Cumulative dev cost (VND) | Cumulative calendar |
|-------------|---------------------------|---------------------|
| Phase 1 | 220,000,000 | 3 months |
| + Phase 2 | 515,000,000 | 7 months |
| + Phase 3 | 810,000,000 | 11 months |
| + Phase 4 | 1,200,000,000 | 14.5 months |
| + Phase 5 | **1,860,000,000** | **~20.5 months** |

```
Phase 1   ████                          220M
Phase 2   █████                         295M
Phase 3   █████                         295M
Phase 4   ███████                       390M
Phase 5   ████████████                  660M
          ─────────────────────────────────────
          Total                         ~1.86B VND
```

---

## 5. What's Included vs Excluded

### Included in phase price

- Development, integration, unit/integration testing
- BA specs, API documentation, basic runbooks
- Staging deployment support
- 2 weeks hypercare after each phase go-live
- Bug fixes for in-scope features during phase timeline

### Excluded (client or separate SOW)

| Item | Typical handling |
|------|------------------|
| AWS / cloud running costs | Client account |
| Payment gateway merchant fees | Client contract with VNPay/MoMo |
| Domain, SSL (beyond setup) | Client |
| Content creation (copy, photos) | Client or separate quote |
| Penetration testing | Phase 5 optional add-on |
| Scope changes | Change request per [docs-ba/02](../docs-ba/02-scope-and-assumptions.md) |
| Training beyond 2 handover sessions | Separate quote |

---

## 6. Payment Schedule (suggested)

| Milestone | % | Phase 1 example |
|-----------|---|-----------------|
| Contract signing | 30% | 66,000,000 |
| Mid-phase demo (Sprint 3) | 30% | 66,000,000 |
| UAT complete | 30% | 66,000,000 |
| Production go-live + 1 week | 10% | 22,000,000 |

Same 30/30/30/10 pattern applies to Phases 2–5.

---

## 7. Change Request Rate

When scope exceeds phase SOW:

| Unit | Rate (VND) |
|------|------------|
| Dev-day (FE/BE) | 1,830,000 |
| Dev-day (Mobile) | 2,000,000 |
| Dev-day (DevOps) | 2,100,000 |

Minimum change request: **5 dev-days** (~9–10.5M VND).

---

## 8. Assumptions & Validity

| # | Assumption |
|---|------------|
| P1 | Rates valid for contracts signed within 6 months of May 2026 |
| P2 | Phase 1 anchor 220M is fixed; Phases 2–5 ±10% after detailed discovery |
| P3 | Client provides timely review (≤ 3 business days per sprint) |
| P4 | One concurrent phase at a time (standard plan) |
| P5 | VNĐ amounts exclude VAT (if applicable per contract entity) |

---

## 9. Quick Reference Links

| Phase | Spec | Price |
|-------|------|-------|
| 1 | [docs-ba/](../docs-ba/README.md) | **220M** (see [01-overview](../docs-ba/01-project-overview.md)) |
| 2 | [phase-2/specification.md](./phase-2-commerce-payments/specification.md) | **295M** |
| 3 | [phase-3/specification.md](./phase-3-omnichannel-loyalty/specification.md) | **295M** |
| 4 | [phase-4/specification.md](./phase-4-mobile-search/specification.md) | **390M** |
| 5 | [phase-5/specification.md](./phase-5-platform-scale/specification.md) | **660M** |
