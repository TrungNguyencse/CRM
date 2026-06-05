# 06 — Backend Specifications

> **Runtime:** Quarkus 3 · Java 21 · Modular monolith · PostgreSQL 16  
> **API:** REST JSON · OpenAPI 3 · RFC 7807 · `/v1`  
> **Screens:** [03-shop](./03-shop-screen-specifications.md) · [04-admin](./04-admin-screen-specifications.md) · **FE:** [05-frontend](./05-frontend-specifications.md)

---

## 5.1 Module Overview

| Module | Package | Responsibility |
|--------|---------|----------------|
| `identity` | `com.store.identity` | Auth, users, roles, profiles |
| `catalog` | `com.store.catalog` | Products, categories, media |
| `inventory` | `com.store.inventory` | Stock levels, reservations, movements |
| `order` | `com.store.order` | Cart, orders, checkout, payment method (COD / bank transfer) |

Shared: `common` (exceptions, pagination, security filters), `config` (application.yaml).

> **No `payment` module in MVP.** Phase 2 will add a pluggable gateway adapter (VNPay, MoMo, Stripe).

---

## 5.2 API Endpoint Inventory (~35 endpoints)

### Auth & Identity (8)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/v1/auth/register` | Public | Customer registration |
| POST | `/v1/auth/login` | Public | Login (customer or staff) |
| POST | `/v1/auth/refresh` | Refresh cookie | New access token |
| POST | `/v1/auth/logout` | User | Revoke refresh token |
| POST | `/v1/auth/forgot-password` | Public | Send reset email |
| POST | `/v1/auth/reset-password` | Public | Reset with token |
| POST | `/v1/auth/change-password` | User | Change password |
| GET | `/v1/customers/me` | Customer | Profile |
| PATCH | `/v1/customers/me` | Customer | Update profile |

### Addresses (3)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/customers/me/addresses` | Customer |
| POST | `/v1/customers/me/addresses` | Customer |
| PATCH | `/v1/customers/me/addresses/{id}` | Customer |
| DELETE | `/v1/customers/me/addresses/{id}` | Customer |

### Catalog — Public (5)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/categories` | Public |
| GET | `/v1/categories/{slug}` | Public |
| GET | `/v1/products` | Public |
| GET | `/v1/products/{slug}` | Public |
| GET | `/v1/products/search` | Public |

### Cart (4)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/cart` | Session / User |
| POST | `/v1/cart/items` | Session / User |
| PATCH | `/v1/cart/items/{sku}` | Session / User |
| DELETE | `/v1/cart/items/{sku}` | Session / User |

### Orders — Customer (4)

| Method | Path | Auth |
|--------|------|------|
| POST | `/v1/orders/checkout` | Customer |
| GET | `/v1/orders/me` | Customer |
| GET | `/v1/orders/me/{id}` | Customer |
| POST | `/v1/orders/me/{id}/cancel` | Customer |

### Settings (2)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/settings/tax-rate` | Public |
| GET | `/v1/settings/payment-info` | Public |

### Admin — Products (6)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/admin/products` | Staff |
| POST | `/v1/admin/products` | Staff |
| PUT | `/v1/admin/products/{id}` | Staff |
| POST | `/v1/admin/products/{id}/images` | Staff |
| DELETE | `/v1/admin/products/{id}/images/{imageId}` | Staff |

### Admin — Categories (4)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/admin/categories` | Admin |
| POST | `/v1/admin/categories` | Admin |
| PUT | `/v1/admin/categories/{id}` | Admin |
| DELETE | `/v1/admin/categories/{id}` | Admin |

### Admin — Inventory (3)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/admin/inventory` | Staff |
| POST | `/v1/admin/inventory/adjust` | Staff |
| GET | `/v1/admin/inventory/{sku}/movements` | Staff |

### Admin — Orders (6)

| Method | Path | Auth |
|--------|------|------|
| GET | `/v1/admin/dashboard/summary` | Staff |
| GET | `/v1/admin/orders` | Staff |
| GET | `/v1/admin/orders/{id}` | Staff |
| PATCH | `/v1/admin/orders/{id}/status` | Staff |
| POST | `/v1/admin/orders/{id}/confirm-payment` | Staff |
| POST | `/v1/admin/orders/{id}/cancel` | Staff |

### Admin — Settings (1)

| Method | Path | Auth |
|--------|------|------|
| PUT | `/v1/admin/settings/payment-info` | Admin |

---

## 5.3 Data Model (MVP Tables)

Schemas separated by module prefix in single PostgreSQL database.

### identity schema

```sql
users (id, email UK, password_hash, type ENUM, status, created_at, updated_at)
customer_profiles (user_id PK/FK, full_name, phone, dob, marketing_opt_in)
staff_profiles (user_id PK/FK, employee_code, role ENUM)
refresh_tokens (id, user_id, token_hash, expires_at, revoked_at)
password_reset_tokens (id, user_id, token_hash, expires_at, used_at)
customer_addresses (id, user_id, full_name, phone, line1, line2, city, district, postal_code, is_default)
```

### catalog schema

```sql
categories (id, parent_id, name, slug UK, image_url, sort_order)
products (id, sku UK, slug UK, name, brand, description, base_price, status, featured, reorder_point)
product_categories (product_id, category_id)
product_images (id, product_id, url, sort_order)
```

### inventory schema

```sql
stock_levels (id, sku UK, quantity_on_hand, quantity_reserved, reorder_point)
stock_movements (id, sku, delta, reason, reference_type, reference_id, actor_id, created_at)
stock_reservations (id, sku, quantity, order_id, status, expires_at)
```

### order schema

```sql
carts (id, user_id NULL, session_id, expires_at)
cart_items (id, cart_id, sku, quantity, unit_price_snapshot)
orders (id, order_number UK, user_id, status, payment_method ENUM[COD,BANK_TRANSFER], transfer_reference, subtotal, tax_amount, shipping_amount, total, currency, placed_at, payment_confirmed_at)
order_items (id, order_id, sku, product_name, quantity, unit_price, line_total)
order_addresses (id, order_id, type, full_name, phone, line1, line2, city, district, postal_code)
order_status_history (id, order_id, status, note, changed_by, changed_at)
```

### settings schema

```sql
system_settings (key PK, value, updated_at)
-- keys: tax_rate (default 10), shipping_flat_fee (default 30000),
--       bank_name, bank_account_number, bank_account_holder
```

---

## 5.4 Validation Rules by Module

### Identity

| Rule ID | Rule | HTTP |
|---------|------|------|
| ID-01 | Email unique, case-insensitive | 409 |
| ID-02 | Password bcrypt, min 8 chars with complexity | 422 |
| ID-03 | Login rate limit 5/15min/IP | 429 |
| ID-04 | Refresh token single-use rotation | 401 |
| ID-05 | Password reset token expires 1 hour | 400 |
| ID-06 | Max 3 addresses per customer | 422 |
| ID-07 | Staff login rejects type=CUSTOMER for admin context | 403 |

### Catalog

| Rule ID | Rule | HTTP |
|---------|------|------|
| CAT-01 | SKU unique | 409 |
| CAT-02 | Slug unique, auto-generated | 409 |
| CAT-03 | Active product requires ≥1 image | 422 |
| CAT-04 | Category max depth 2 | 422 |
| CAT-05 | Cannot delete category with products | 409 |
| CAT-06 | Image max 5MB, types jpg/png/webp | 422 |
| CAT-07 | Price positive integer (VND, no decimals) | 422 |

### Inventory

| Rule ID | Rule | HTTP |
|---------|------|------|
| INV-01 | quantity_on_hand >= quantity_reserved always | 409 |
| INV-02 | Reserve only if available >= requested | 409 |
| INV-03 | Reservation expires 30 minutes | — (scheduled job) |
| INV-04 | Movement ledger append-only (no delete) | — |
| INV-05 | REMOVE adjustment <= available | 422 |

### Order

| Rule ID | Rule | HTTP |
|---------|------|------|
| ORD-01 | Checkout requires non-empty cart | 422 |
| ORD-02 | Re-validate stock at checkout | 409 |
| ORD-03 | Idempotency-Key header on checkout (UUID) | 409 duplicate |
| ORD-04 | Customer cancel AWAITING_PAYMENT or CONFIRMED (not shipped) | 422 |
| ORD-05 | Order number format: `ORD-{YYYYMMDD}-{6-digit-seq}` | — |
| ORD-06 | Status transitions per state machine | 422 |
| ORD-07 | Merge guest cart on login/register | — |
| ORD-08 | COD → CONFIRMED immediately; BANK_TRANSFER → AWAITING_PAYMENT | — |
| ORD-09 | Admin confirm-payment only from AWAITING_PAYMENT | 422 |
| ORD-10 | Auto-cancel AWAITING_PAYMENT after 48h | — (scheduled job) |

### Payment (MVP — offline only)

| Rule ID | Rule | HTTP |
|---------|------|------|
| PAY-01 | paymentMethod required: COD or BANK_TRANSFER | 422 |
| PAY-02 | Bank details served from system_settings (not hardcoded) | — |
| PAY-03 | No external payment API calls in MVP | — |

---

## 5.5 Checkout Flow (Offline Payment)

### COD flow

```
Client                 Order Module              Inventory
  |                         |                       |
  |-- POST /checkout ------>|  (paymentMethod=COD)  |
  |                         |-- validate stock ---->|
  |                         |-- deduct stock ------>|
  |                         |-- order CONFIRMED     |
  |                         |-- send email (SES)    |
  |<-- 201 orderId ---------|                       |
```

### Bank transfer flow

```
Client                 Order Module              Inventory        Admin
  |                         |                       |              |
  |-- POST /checkout ------>|  (paymentMethod=     |              |
  |                         |   BANK_TRANSFER)      |              |
  |                         |-- reserve stock ----->|              |
  |                         |-- AWAITING_PAYMENT    |              |
  |                         |-- send awaiting email |              |
  |<-- 201 + bank details --|                       |              |
  |                         |                       |              |
  |            (customer transfers offline)         |              |
  |                         |                       |              |
  |                         |<-- POST /confirm-payment ------------|
  |                         |-- deduct stock ------>|
  |                         |-- order CONFIRMED     |
  |                         |-- send confirmed email|
```

**Auto-cancel:** AWAITING_PAYMENT orders not confirmed within 48h → CANCELLED, stock reservation released.

---

## 5.6 Security

| Concern | Implementation |
|---------|----------------|
| Authentication | JWT RS256; access 15m; refresh 7d |
| Authorization | `@RolesAllowed` + custom ownership check |
| CORS | Shop + Admin origins whitelist |
| Input sanitization | Bean Validation (Hibernate Validator) |
| SQL injection | Parameterized queries (Panache) |
| File upload | Content-type check + size limit; S3 presigned upload |
| Secrets | AWS Secrets Manager; never in git |
| HTTPS | TLS termination at CloudFront/ALB |

### JWT Claims

```json
{
  "sub": "user-uuid",
  "email": "user@example.com",
  "type": "CUSTOMER|STAFF",
  "roles": ["Customer"] | ["Admin"] | ["StoreManager"],
  "iat": 0,
  "exp": 0
}
```

---

## 5.7 Error Response Format (RFC 7807)

```json
{
  "type": "https://api.store.example/problems/validation-error",
  "title": "Validation Failed",
  "status": 422,
  "detail": "One or more fields are invalid",
  "instance": "/v1/orders/checkout",
  "errors": [
    { "field": "shippingAddress.phone", "message": "Enter a valid phone number" }
  ]
}
```

---

## 5.8 Pagination Standard

Query: `?page=1&size=20` (1-based page, default size 20, max 100).

Response wrapper:

```json
{
  "data": [],
  "meta": { "page": 1, "size": 20, "totalItems": 150, "totalPages": 8 }
}
```

---

## 5.9 Scheduled Jobs

| Job | Schedule | Action |
|-----|----------|--------|
| Expire cart reservations | Every 5 min | Release reservations past `expires_at` |
| Cancel stale AWAITING_PAYMENT orders | Every 15 min | Cancel > 48h; release stock |
| Payment reminder email | Every 6 hours | Remind at 24h for AWAITING_PAYMENT |
| Clean expired guest carts | Daily 02:00 | Delete carts past TTL |

---

## 5.10 Email Templates (SES)

| Template code | Trigger | Recipient |
|---------------|---------|-----------|
| `ORDER_CONFIRMED` | COD checkout or admin confirms bank transfer | Customer |
| `ORDER_AWAITING_PAYMENT` | Bank transfer checkout | Customer |
| `ORDER_SHIPPED` | Status → SHIPPED | Customer |
| `ORDER_CANCELLED` | Cancel | Customer |
| `PAYMENT_REMINDER` | AWAITING_PAYMENT > 24h | Customer |
| `PASSWORD_RESET` | Forgot password | User |

---

## 5.11 Non-Functional Requirements

| Metric | Target |
|--------|--------|
| API p95 latency | < 800ms (excl. image upload) |
| Concurrent users | 50 (MVP load) |
| DB connection pool | max 20 |
| Test coverage | ≥ 60% on service layer (unit + integration) |
| OpenAPI | Published at `/q/openapi` (dev/staging only) |

---

## 5.12 Testing Strategy

| Layer | Tool | Scope |
|-------|------|-------|
| Unit | JUnit 5 + Mockito | Business rules per module |
| Integration | Testcontainers (Postgres) | Repository + REST |
| Contract | Manual OpenAPI validation | FE-BE alignment |
| E2E | Playwright (optional Week 12) | Critical checkout path |
