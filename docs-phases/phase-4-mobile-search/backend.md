# Phase 4 — Backend

> **Owner:** BE dev · **Consumed by:** [frontend.md](./frontend.md)

---

## Search service (`search-indexer`)

### OpenSearch indices

| Index | Fields | Source events |
|-------|--------|---------------|
| `products` | name, sku, brand, category, price, stock, status | ProductCreated/Updated, PriceChanged, StockAdjusted |
| `customers` | name, email, phone, tier | UserRegistered, LoyaltyTierChanged |
| `orders` | orderNumber, customerEmail, status, total | OrderPlaced, OrderCompleted |

### Indexer strategy

**Option A (recommended):** Poll outbox tables + bulk index (no Kafka yet)  
**Option B:** Kafka consumer if MSK provisioned early

Migrate to Kafka in Phase 5.

### APIs

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/products/search` | GET | OpenSearch-backed (replaces Postgres FTS) |
| `GET /v1/admin/customers/search` | GET | Customer search |
| `GET /v1/admin/orders/search` | GET | Order search |

---

## SSE admin events

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/admin/events/stream` | GET | SSE: `order.placed`, `low_stock`, `pos.sale` |

---

## BE tasks

| ID | Task | Est. |
|----|------|------|
| P4-BE-01 | OpenSearch cluster + index mappings | 4d |
| P4-BE-02 | Search indexer service | 8d |
| P4-BE-03 | Migrate product search to OpenSearch | 3d |
| P4-BE-04 | Customer/order search APIs | 4d |
| P4-BE-05 | SSE admin events endpoint | 4d |

**Total:** ~23 dev-days

---

## Infrastructure (BE-owned)

| Resource | Purpose |
|----------|---------|
| OpenSearch (managed) | Product, customer, order indices |
| CloudFront | Mobile asset CDN (optional) |

---

## Performance targets

| Metric | Target |
|--------|--------|
| Product search p95 | < 200ms |
| SSE delivery | < 5s from event emit |
