# Phase 5 — Specification: Platform Scale

> Maps to original plan: **Phase 0 Foundations** + **Phase 5 Production Readiness** + microservices extraction

---

## 1. Prerequisites

- Phases 1–4 complete and stable in production
- Traffic or org scale justification for EKS/Kafka (e.g. > 50 stores, event-driven ops)
- DevOps capacity for cluster operations

---

## 2. Scope

### In scope

| Area | Deliverable (from original docs) |
|------|----------------------------------|
| **Microservices** | Split monolith → 10 services + search-indexer |
| **Event bus** | MSK (Kafka) + Avro Schema Registry + Debezium outbox relay |
| **Orchestration** | Temporal cluster; all sagas in `platform/temporal-workflows` |
| **Auth** | Keycloak OIDC (realms: customers, staff) |
| **Gateway** | Apache APISIX (rate limit, OIDC introspection, routing) |
| **Infra** | Terraform: VPC, EKS, RDS×10, MSK, OpenSearch, Route53 |
| **GitOps** | Helm per service + ArgoCD |
| **Observability** | OpenTelemetry → CloudWatch/Grafana/Tempo/Loki |
| **Quality** | Pact, k6 load test, chaos tests, OWASP ASVS L2 |

### Out of scope (original doc exclusions)

- ERP integrations (SAP, Oracle)
- Cryptocurrency payments
- Franchise self-service portal
- Computer-vision scan-and-go

---

## 3. Service Split Plan

### Target services (from original plan)

| # | Service | Database | Extract from monolith |
|---|---------|----------|----------------------|
| 1 | identity-service | identity_db | Phase 5.1 |
| 2 | catalog-service | catalog_db | Phase 5.1 |
| 3 | inventory-service | inventory_db | Phase 5.2 |
| 4 | pricing-service | pricing_db | Phase 5.2 |
| 5 | order-service | order_db | Phase 5.3 |
| 6 | payment-service | payment_db | Phase 5.3 |
| 7 | fulfillment-service | fulfillment_db | Phase 5.3 |
| 8 | store-pos-service | store_db | Phase 5.4 |
| 9 | crm-loyalty-service | crm_db | Phase 5.4 |
| 10 | notification-service | notification_db | Phase 5.4 |
| + | search-indexer-service | OpenSearch only | Phase 5.5 |

### Migration strategy (strangler fig)

```
Step 1: Extract read-only APIs (catalog, search) — lowest risk
Step 2: Extract inventory + pricing — event consumers added
Step 3: Extract order + payment + fulfillment — Temporal workers move
Step 4: Extract POS + CRM + notification
Step 5: Decommission monolith deployment
```

Each step:
1. Create service repo/module with own DB (migrate schema)
2. Dual-write or outbox sync during transition
3. Switch APISIX route to new service
4. Remove code from monolith

---

## 4. Repository Layout (target)

```
/services
  /identity-service
  /catalog-service
  /inventory-service
  /pricing-service
  /order-service
  /payment-service
  /fulfillment-service
  /store-pos-service
  /crm-loyalty-service
  /notification-service
  /search-indexer-service
/contracts
  /avro-schemas
  /openapi
/platform
  /temporal-workflows
  /shared-libs          # otel, outbox, security, rest-client
/web
  /shop-next
  /admin-next
/mobile
  /retail-app
/infra
  /terraform
  /helm
  /argocd
```

---

## 5. Event Catalog (Kafka topics)

| Event | Producer | Consumers |
|-------|----------|-----------|
| `UserRegistered` | Identity | CRM, Notification |
| `ProductCreated` | Catalog | Search |
| `PriceChanged` | Pricing | Catalog, Search, POS |
| `StockAdjusted` | Inventory | Search, Notification |
| `OrderPlaced` | Order | Inventory, Payment, Notification |
| `OrderCompleted` | Order | CRM, Notification |
| `PaymentCaptured` | Payment | Order |
| `POSSaleCompleted` | Store | Inventory, CRM, Notification |
| `LoyaltyTierChanged` | CRM | Pricing, Notification |
| `LowStockAlert` | Inventory | Notification |

Transport: Avro + Confluent Schema Registry; outbox relay via Debezium CDC.

---

## 6. Infrastructure (AWS)

| Resource | Spec |
|----------|------|
| EKS | 3 AZs; node groups: general (t4g.medium), kafka-consumers, temporal |
| RDS | Postgres 16 per service; read replicas for Catalog, Inventory |
| MSK | 3 brokers, kafka.m7g.large |
| OpenSearch | Managed cluster (already from Phase 4, scale up) |
| ElastiCache Redis | Cart, rate limits, idempotency |
| APISIX | On EKS ingress |
| Keycloak | Separate cluster or EKS namespace |
| CI/CD | GitHub Actions → ECR → ArgoCD; Trivy scan, OpenAPI lint |
| Native image | GraalVM `mvn package -Pnative` per service |

---

## 7. Keycloak Migration

| Realm | Users | Roles |
|-------|-------|-------|
| `customers` | Shop + Mobile | Customer |
| `staff` | Admin + POS | Admin, StoreManager, Cashier |

**Migration steps:**
1. Deploy Keycloak; configure realms
2. Shop/Admin/Mobile switch to NextAuth/Expo OIDC provider
3. Services validate JWT via JWKS (remove custom JWT issuance)
4. Migrate existing users (batch import)

---

## 8. Workstreams & Estimates

| Workstream | Duration | Team |
|------------|----------|------|
| Terraform + EKS + MSK | 4 weeks | 1 DevOps |
| Shared libs + outbox relay | 3 weeks | 1 BE |
| Service extraction (5 waves) | 12 weeks | 3 BE |
| Keycloak + APISIX | 3 weeks | 1 BE + DevOps |
| Observability + SLOs | 2 weeks | DevOps |
| Load + chaos testing | 2 weeks | 1 BE + DevOps |
| Security audit + pen test | 2 weeks | External + team |

**Total:** ~18–24 weeks with parallel workstreams

---

## 9. Verification (from original plan §11)

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

## 10. Sprint Plan (24 weeks, high level)

| Wave | Weeks | Extract |
|------|-------|---------|
| P5-W0 | 1–4 | Infra: EKS, MSK, Terraform, ArgoCD |
| P5-W1 | 5–7 | Catalog + Identity services |
| P5-W2 | 8–10 | Inventory + Pricing services |
| P5-W3 | 11–14 | Order + Payment + Fulfillment + Temporal |
| P5-W4 | 15–17 | POS + CRM + Notification |
| P5-W5 | 18–20 | Search indexer on Kafka; APISIX; Keycloak |
| P5-W6 | 21–24 | Load test, chaos, security, pilot rollout 5→50 stores |

---

## 11. Acceptance Criteria

- [ ] All 10 services deployed on EKS via ArgoCD
- [ ] No cross-service DB joins; events via Kafka only
- [ ] Keycloak OIDC login on Shop, Admin, Mobile
- [ ] APISIX rate limiting active on public routes
- [ ] k6 load test passes at 200 RPS
- [ ] Outbox replay verified after simulated broker failure
- [ ] Pilot: 5 stores live on new platform → expand to 50
- [ ] Runbooks, on-call rotation, DR tested (RDS PITR)

---

## 12. Reference

- [Full technical plan](../../plan-convenienceStorePlatform%201%20(1).md) — Sections 1–11
- [Features by module](../../convenience-store-platform-features%20(2).md)
