# Phase 5 — Backend & DevOps

> **Owner:** BE dev + DevOps · **Consumed by:** [frontend.md](./frontend.md)

---

## Service split plan

| # | Service | Database | Wave |
|---|---------|----------|------|
| 1 | identity-service | identity_db | W1 |
| 2 | catalog-service | catalog_db | W1 |
| 3 | inventory-service | inventory_db | W2 |
| 4 | pricing-service | pricing_db | W2 |
| 5 | order-service | order_db | W3 |
| 6 | payment-service | payment_db | W3 |
| 7 | fulfillment-service | fulfillment_db | W3 |
| 8 | store-pos-service | store_db | W4 |
| 9 | crm-loyalty-service | crm_db | W4 |
| 10 | notification-service | notification_db | W4 |
| + | search-indexer-service | OpenSearch only | W5 |

---

## Repository layout (target)

```
/services
  /identity-service … /notification-service
  /search-indexer-service
/contracts
  /avro-schemas
  /openapi
/platform
  /temporal-workflows
  /shared-libs          # otel, outbox, security, rest-client
/web /shop-next /admin-next
/mobile /retail-app
/infra /terraform /helm /argocd
```

---

## Event catalog (Kafka topics)

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

Transport: Avro + Schema Registry; outbox relay via Debezium CDC.

---

## Infrastructure (AWS)

| Resource | Spec |
|----------|------|
| EKS | 3 AZs; node groups: general, kafka-consumers, temporal |
| RDS | Postgres 16 per service; read replicas for Catalog, Inventory |
| MSK | 3 brokers, kafka.m7g.large |
| OpenSearch | Scale up from Phase 4 |
| ElastiCache Redis | Cart, rate limits, idempotency |
| APISIX | EKS ingress — rate limit, OIDC introspection |
| Keycloak | Separate cluster or EKS namespace |
| CI/CD | GitHub Actions → ECR → ArgoCD; Trivy, OpenAPI lint |
| Native image | GraalVM `mvn package -Pnative` per service |

---

## Keycloak (BE-owned setup)

| Realm | Users | JWT validation |
|-------|-------|----------------|
| `customers` | Shop + Mobile | JWKS per service |
| `staff` | Admin + POS | JWKS per service |

Services validate JWT via JWKS; remove custom JWT issuance from monolith.

---

## Workstreams & estimates

| Workstream | Duration | Team |
|------------|----------|------|
| Terraform + EKS + MSK | 4 weeks | DevOps |
| Shared libs + outbox relay | 3 weeks | BE |
| Service extraction (5 waves) | 12 weeks | 2–3 BE |
| Keycloak + APISIX | 3 weeks | BE + DevOps |
| Observability + SLOs | 2 weeks | DevOps |
| Load + chaos testing | 2 weeks | BE + DevOps |
| Security audit + pen test | 2 weeks | External + team |

---

## BE / DevOps tasks

| ID | Task | Est. |
|----|------|------|
| P5-BE-01 | Terraform VPC + EKS + MSK | 15d |
| P5-BE-02 | Shared libs (outbox, otel, security) | 10d |
| P5-BE-03 | Wave 1: Identity + Catalog extract | 12d |
| P5-BE-04 | Wave 2: Inventory + Pricing extract | 12d |
| P5-BE-05 | Wave 3: Order + Payment + Fulfillment | 18d |
| P5-BE-06 | Wave 4: POS + CRM + Notification | 15d |
| P5-BE-07 | Wave 5: Search indexer on Kafka | 8d |
| P5-BE-08 | Keycloak + APISIX config | 10d |
| P5-BE-09 | Pact + k6 + chaos test suite | 10d |
| P5-BE-10 | GraalVM native images | 8d |

**Total:** ~118 dev-days (parallel workstreams reduce calendar time)
