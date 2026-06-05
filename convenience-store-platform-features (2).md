# Omnichannel Convenience Store Platform — Features by Module

---

## Backend Microservices

### 1. Identity & Auth Service
- User registration & deactivation
- Role & permission management (RBAC)
- Session management (refresh tokens, revocation)
- Keycloak OIDC integration
- Staff profiles (employee code, store assignment)
- Customer profiles (name, DOB, gender, marketing opt-in)
- Audit logging (action, IP, user-agent tracking)
- JWT token issuance with `roles` claim

### 2. Product & Catalog Service
- Product CRUD (SKU, GTIN, name, brand, description, pricing)
- Category management with hierarchy (Postgres `ltree`)
- Product variants (attributes, price overrides)
- Product media upload (S3 integration)
- EAV-based flexible product attributes
- Product status lifecycle
- Event emission for search indexing

### 3. Inventory Service
- Stock level tracking per SKU per location (store/warehouse)
- Stock reservations (with expiry) for checkout
- Stock movements ledger (append-only: sale, receipt, adjust, transfer, return)
- Stock transfers between locations
- Reorder point & low-stock alerts
- Reservation release on order cancellation
- Monthly partitioning on movements

### 4. Pricing & Promotions Service
- Price lists (currency, validity period, location scope)
- Price list items per SKU
- Promotions engine (percent off, fixed off, BOGO, bundle)
- Rule-based promotion definitions (JSONB)
- Coupon generation & management (max uses, per-user limits, targeted coupons)
- Coupon redemption tracking
- Campaign activation/deactivation

### 5. Order & Checkout Service
- Shopping cart management (guest + authenticated, expiry)
- Cart item CRUD
- Order placement (multi-channel: online, POS, mobile)
- Order status lifecycle & history tracking
- Billing & shipping address management
- Saga state tracking (Temporal workflow reference)
- Order number generation (unique)
- Idempotency key support

### 6. Payment Service
- Payment intent creation (idempotency key)
- Multi-provider support (Stripe + abstract SPI for local providers)
- Authorization, capture, void flows
- Refund processing
- Tokenized payment method storage (last4, brand)
- Transaction ledger (per intent)
- Multi-currency ready

### 7. Shipping & Fulfillment Service
- Shipment creation (delivery & click-and-collect)
- Carrier assignment & tracking number
- Shipment status lifecycle (created → dispatched → delivered)
- Pickup slot management (capacity-based)
- Pickup booking
- Delivery attempt tracking (outcome, notes)
- Shipment item-level tracking

### 8. Store & POS Service
- Store management (code, name, opening hours)
- Register management (status tracking)
- Cashier session open/close (float tracking)
- POS sale recording (items, tender, change)
- Cash drop recording
- Customer linking to POS sale (optional)
- Real-time product/price sync (consumes catalog events)

### 9. CRM & Loyalty Service
- Customer profile (lifetime spend, tier, points balance)
- Loyalty tiers (Bronze/Silver/Gold/Platinum) with perks
- Points ledger (append-only: earn, redeem, adjust, expire)
- Points expiry (24-month default)
- Nightly tier re-evaluation job
- Rule-based customer segmentation
- Segment membership tracking
- Customer interaction logging (clicks, opens, store visits)
- Points earned from orders + POS sales

### 10. Notification Service
- Template management (email/SMS/push, locale, variables)
- Multi-channel delivery (SES email, SNS SMS, FCM/APNs push)
- Notification status tracking (sent, failed + reason)
- User subscription/opt-in management
- Device token registry (iOS, Android, Web)
- Event-driven triggers (order confirmed, shipped, tier changed, low stock)

### 11. Search (Read-Model)
- OpenSearch-backed product search
- Customer search
- Order search
- Real-time index updates from Kafka events (product, price, stock changes)

---

## Frontend Applications

### 12. Public Shop (Next.js SSR)
- Home page
- Category browsing
- Product detail page (PDP)
- Shopping cart
- Checkout flow
- User account management
- Order tracking
- Loyalty dashboard
- SSR for SEO, ISR for catalog, CSR for account
- NextAuth + Keycloak OIDC

### 13. Staff CRM/POS Dashboard (Next.js CSR)
- Admin dashboard
- Customer 360 view
- Order operations
- Inventory management views
- POS terminal interface
- Loyalty administration
- Campaign management
- Reports
- Role-gated routes (Admin/StoreManager/Cashier)
- Real-time updates via SSE

### 14. Mobile App (React Native / Expo)
- Shop tab (browse & purchase)
- Scan tab (barcode/QR for in-store loyalty)
- Wallet tab (coupons, points, tier status)
- Orders tab (history & tracking)
- Profile tab
- Push notifications (Expo Push → FCM/APNs)
- PKCE OIDC auth flow
- Offline cart cache (MMKV)

---

## Cross-Cutting / Platform

### 15. Temporal Workflows
- `PlaceOrderWorkflow` (reserve → authorize → redeem coupon → confirm → ship → capture)
- Refund saga (refund payment → update order → restock → reverse points → notify)
- Compensation (rollback) for each step on failure

### 16. Shared Libraries
- OpenTelemetry starter
- Outbox relay (Debezium CDC → Kafka)
- Security/JWT utilities
- REST client wrapper (with fault tolerance)

### 17. API Gateway (APISIX)
- Rate limiting
- OIDC token introspection
- Request routing

### 18. Infrastructure
- Terraform (VPC, EKS, RDS, MSK, OpenSearch, Secrets Manager, Route53)
- Helm charts per service
- ArgoCD GitOps deployment
- CI/CD via GitHub Actions (build, native-image, Trivy scan, OpenAPI lint)
