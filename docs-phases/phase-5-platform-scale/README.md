# Phase 5 — Platform Scale

> **Duration:** 4–6 months · **Team:** 3 BE + 1 DevOps + 1 FE · **Prerequisite:** Phase 4 complete

Evolve to the **full target architecture** from the original technical plan — 10 microservices, Kafka, EKS, Keycloak, APISIX, observability, production hardening.

---

## Goals

| # | Goal |
|---|------|
| G1 | Split monolith into 10 deployable microservices (DB-per-service) |
| G2 | Event-driven integration via Kafka (MSK) + outbox relay (Debezium) |
| G3 | Deploy on AWS EKS with Helm + ArgoCD GitOps |
| G4 | Keycloak OIDC for customers and staff |
| G5 | Pass load test: 200 RPS checkout, 500 stores, 1M customers (medium scale) |

---

## Key Deliverables

| Layer | Deliverable |
|-------|-------------|
| **Services** | 10 Quarkus services + search-indexer + temporal-workflows |
| **Platform** | APISIX gateway, MSK, OpenTelemetry, shared libs |
| **Infra** | Terraform (VPC, EKS, RDS×10, MSK, OpenSearch), Helm per service |
| **Quality** | Pact contract tests, chaos tests, k6 load tests, OWASP ASVS L2 |

---

## Documents

| File | Contents |
|------|----------|
| [specification.md](./specification.md) | Service split plan, migration steps, infra, acceptance |

---

## Pricing

| Option | Team | Duration | **Price (VND)** |
|--------|------|----------|-----------------|
| **Standard** | 2 BE + 1 DevOps + 1 FE | 6 months | **660,000,000** |
| Accelerated | 3 BE + 1 DevOps + 1 FE | 5 months | 730,000,000 |
| Infra-only (no service split) | 1 DevOps + 1 BE | 4 months | 450,000,000 |

| Breakdown (standard) | Amount (VND) |
|----------------------|-------------|
| Backend (10 microservices) | 320,000,000 |
| DevOps (EKS, MSK, Terraform) | 180,000,000 |
| Frontend (Keycloak migration) | 45,000,000 |
| QA (k6, chaos, Pact) + PM | 115,000,000 |

**Client-owned extras:** AWS ~$800–2,000 USD/month, pen test 50–150M VND.

→ [Full pricing](../00-pricing-estimates.md#phase-5--platform-scale--660000000-vnd)

---

## Reference

Full 24-week original roadmap: [plan-convenienceStorePlatform 1 (1).md](../../plan-convenienceStorePlatform%201%20(1).md) Sections 6–11.
