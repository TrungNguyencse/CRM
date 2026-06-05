# Phase 5 — General

> **Owner:** PM / either dev · **Price:** 660,000,000 VND · **Duration:** ~6 months

---

## Goals

| # | Goal |
|---|------|
| G1 | Split monolith → 10 microservices (DB-per-service) |
| G2 | Event-driven integration via Kafka (MSK) + Debezium outbox relay |
| G3 | Deploy on AWS EKS with Helm + ArgoCD |
| G4 | Keycloak OIDC for customers and staff |
| G5 | Load test: 200 RPS checkout, 500 stores, 1M customers |

---

## Prerequisites

- Phases 1–4 complete and stable in production
- Traffic or org scale justification for EKS/Kafka (e.g. > 50 stores)
- DevOps capacity for cluster operations

---

## Scope summary

| In | Out |
|----|-----|
| Microservices, Kafka, EKS, Keycloak, APISIX, observability | ERP integrations, crypto payments, franchise portal, CV scan-and-go |

Detail: [frontend.md](./frontend.md) · [backend.md](./backend.md)

---

## Migration strategy (strangler fig)

```
Step 1: Extract read-only APIs (catalog, search) — lowest risk
Step 2: Extract inventory + pricing — event consumers added
Step 3: Extract order + payment + fulfillment — Temporal workers move
Step 4: Extract POS + CRM + notification
Step 5: Decommission monolith deployment
```

Each step:
1. Create service with own DB (migrate schema)
2. Dual-write or outbox sync during transition
3. Switch APISIX route to new service
4. Remove code from monolith

Full service list → [backend.md](./backend.md)

---

## Sprint plan (24 weeks)

| Wave | Weeks | Extract |
|------|-------|---------|
| P5-W0 | 1–4 | Infra: EKS, MSK, Terraform, ArgoCD |
| P5-W1 | 5–7 | Catalog + Identity services |
| P5-W2 | 8–10 | Inventory + Pricing services |
| P5-W3 | 11–14 | Order + Payment + Fulfillment + Temporal |
| P5-W4 | 15–17 | POS + CRM + Notification |
| P5-W5 | 18–20 | Search indexer on Kafka; APISIX; Keycloak |
| P5-W6 | 21–24 | Load test, chaos, security, pilot 5→50 stores |

---

## Verification checklist

| # | Check | Target |
|---|-------|--------|
| V1 | Architecture + event catalog sign-off | Documented |
| V2 | Pact contract tests in CI | All consumer/producer pairs |
| V3 | Temporal saga tests | Happy path + each compensation |
| V4 | k6 load test | 200 RPS checkout 30 min, p95 < 800ms |
| V5 | Native image startup | < 200ms start, RSS < 80MB idle |
| V6 | Chaos tests | Kill broker/RDS/pod; no data loss |
| V7 | OWASP ZAP | Green scan; OIDC verified |

---

## Acceptance criteria

- [ ] All 10 services deployed on EKS via ArgoCD
- [ ] No cross-service DB joins; events via Kafka only
- [ ] Keycloak OIDC login on Shop, Admin, Mobile
- [ ] APISIX rate limiting active on public routes
- [ ] k6 load test passes at 200 RPS
- [ ] Outbox replay verified after simulated broker failure
- [ ] Pilot: 5 stores live → expand to 50
- [ ] Runbooks, on-call rotation, DR tested (RDS PITR)

---

## Client-owned costs

AWS ~$800–2,000 USD/month, pen test 50–150M VND.

→ [Full pricing](../00-pricing-estimates.md#phase-5--platform-scale--660000000-vnd)

---

## Reference

[Full technical plan](../../plan-convenienceStorePlatform%201%20(1).md) Sections 1–11
