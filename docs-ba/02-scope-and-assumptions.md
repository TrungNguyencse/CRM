# 02 — Scope, Assumptions & Acceptance

## 2.1 In Scope (MVP)

### Functional

- [ ] Customer registration, login, logout, password reset (email link)
- [ ] Browse products by category (max 2-level category tree)
- [ ] Product detail with images, price, stock availability indicator
- [ ] Search products by name or SKU
- [ ] Guest cart (browser session) and logged-in cart (persisted)
- [ ] Checkout with shipping address and payment method selection (**COD** or **bank transfer**)
- [ ] Bank transfer: display store bank account details; admin confirms payment manually
- [ ] Order confirmation email (COD: immediate; bank transfer: after admin confirms)
- [ ] Customer order history and status tracking
- [ ] Admin/staff login with roles: **Admin**, **StoreManager**
- [ ] Admin product CRUD with image upload (max 5 images/product)
- [ ] Admin category CRUD
- [ ] Admin inventory view and manual stock adjustment
- [ ] Admin order list with filters; status updates: `PENDING` → `CONFIRMED` → `SHIPPED` → `DELIVERED` → `CANCELLED`
- [ ] Admin dashboard with basic KPIs
- [ ] Admin confirms bank transfer payments (AWAITING_PAYMENT → CONFIRMED)
- [ ] Stock reservation at checkout (released on cancel/timeout)
- [ ] Admin can cancel orders and restore stock (no automated refund — manual for COD/bank transfer)

### Non-functional

- [ ] Responsive Shop UI (mobile-first, 320px+)
- [ ] Admin UI usable on desktop (1280px+)
- [ ] API response p95 < 1s under normal load (< 50 concurrent users)
- [ ] HTTPS everywhere (TLS via CloudFront / ALB)
- [ ] Staging + production environments
- [ ] Automated CI on pull requests
- [ ] Basic CloudWatch logging

---

## 2.2 Out of Scope (Phase 2+)

| Item | Reason |
|------|--------|
| Mobile app (React Native / Expo) | ~2–3 additional person-months |
| POS terminal & cashier sessions | Hardware + separate UX |
| Loyalty points, tiers, coupons, promotions | Pricing engine complexity |
| Multi-store / multi-warehouse | Single default warehouse in MVP |
| Inter-store stock transfers | Inventory module simplification |
| Click-and-collect / pickup slots | Fulfillment simplification |
| Carrier integration (tracking APIs) | Manual tracking number entry only |
| Kafka, Temporal, OpenSearch, EKS | Infrastructure cost & ops burden |
| Keycloak / SSO | JWT sufficient for MVP |
| VNPay, MoMo, Stripe, bank payment APIs | Dev cost + 1.5–3% transaction fees; Phase 2 |
| Automated online payment capture | Replaced by COD + manual bank transfer |
| Vietnamese UI (i18n) | English UI; i18n framework ready |
| Advanced reports & analytics | Basic dashboard only |
| Customer segmentation & CRM | Phase 2 |
| SMS / push notifications | Email only in MVP |
| Multi-currency | VND only |
| Tax engine (Avalara) | Flat tax rate configurable in admin settings |
| Franchise / multi-tenant SaaS | Single tenant |
| Product variants (size/color) | Single SKU per product in MVP |
| Guest checkout without account | Must register/login before placing order |
| Product reviews & ratings | Phase 2 |

---

## 2.3 Assumptions

| # | Assumption | Impact if wrong |
|---|------------|-----------------|
| A1 | Client provides store bank account details for transfer display | Bank transfer option blocked |
| A2 | Client provides AWS account or approves sub-account | Deployment blocked |
| A3 | Client provides domain name and DNS access | Production URL delayed |
| A4 | Product catalog seed ≤ 500 SKUs at launch | Performance tuning may be needed |
| A5 | Single warehouse / fulfillment location | Multi-location needs scope change |
| A6 | Client reviews and approves specs within 3 business days per milestone | Timeline slips |
| A7 | Content (logo, brand colors, sample products) provided by Week 2 | UI uses placeholders |
| A8 | No legacy system integration required | N/A |
| A9 | Flat VAT/tax rate (e.g. 10%) applied at checkout | Complex tax rules need change request |
| A10 | COD is acceptable as primary payment for launch market | If online payment required at launch → scope change (Phase 2 quote) |
| A11 | English-only UI copy; client provides final marketing text | Copy changes are CR |

---

## 2.4 Roles & Permissions (MVP)

| Permission | Customer | StoreManager | Admin |
|------------|:--------:|:------------:|:-----:|
| Browse shop | ✅ | ✅ | ✅ |
| Place orders | ✅ | — | — |
| View own orders | ✅ | — | — |
| Admin login | — | ✅ | ✅ |
| View dashboard | — | ✅ | ✅ |
| Manage products | — | ✅ | ✅ |
| Manage categories | — | — | ✅ |
| Adjust inventory | — | ✅ | ✅ |
| Manage orders | — | ✅ | ✅ |
| Manage staff users | — | — | ✅ |
| System settings (tax rate) | — | — | ✅ |
| Process refunds | — | — | Manual (record only; no gateway) |

---

## 2.5 Order Status Lifecycle

### COD flow
```
Place order (COD) ──► CONFIRMED ──► SHIPPED ──► DELIVERED
                          │
                          └──► CANCELLED (admin/customer before ship)
```

### Bank transfer flow
```
Place order (transfer) ──► AWAITING_PAYMENT ──► CONFIRMED ──► SHIPPED ──► DELIVERED
                                │                    │
                                │ timeout 48h        └──► CANCELLED
                                └──► CANCELLED
```

### Combined diagram

```
                         ┌───────────────────┐
                         │ AWAITING_PAYMENT  │  (bank transfer only)
                         └─────────┬─────────┘
                                   │ admin confirms
     COD ──────────────────────────┼──────────────► ┌──────────────┐
     (direct)                      │                │  CONFIRMED   │
                                    └───────────────►└──────┬───────┘
                                                            │ ship
                         cancel                             ▼
                    ┌──────────────┐                 ┌──────────────┐
                    │  CANCELLED   │◄────────────────│   SHIPPED    │
                    └──────────────┘                 └──────┬───────┘
                                                            │ deliver
                                                            ▼
                                                     ┌──────────────┐
                                                     │  DELIVERED   │
                                                     └──────────────┘
```

**Rules:**
- Customer can cancel `AWAITING_PAYMENT` orders anytime before admin confirms.
- Customer can cancel `CONFIRMED` orders only if not yet `SHIPPED` (store policy — MVP enabled).
- Admin can cancel any non-delivered order; stock is restored.
- Bank transfer orders auto-cancel after **48 hours** if payment not confirmed.
- No automated refund API — admin records cancellation; actual money return is manual (COD: N/A; bank: client processes offline).

---

## 2.6 Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Scope creep during development | High | High | Change request process; Phase 2 backlog |
| COD payment disputes / no-shows | Medium | Medium | Clear COD policy on checkout; admin can cancel |
| Bank transfer not confirmed in time | Medium | Low | 48h auto-cancel + reminder email at 24h |
| Single BE dev bottleneck | Medium | High | Modular monolith; FE uses mock API Weeks 1–4 |
| AWS cost overrun | Low | Medium | t3.small/micro sizing; alerts at 80% budget |
| Client delay on content/review | Medium | Medium | Placeholder content; strict review SLA in contract |

---

## 2.7 Acceptance Criteria (Sign-off Checklist)

### Shop

- [ ] Customer can register, log in, and update profile
- [ ] Customer can browse categories and view product details
- [ ] Customer can search and add items to cart
- [ ] Customer can complete checkout with **COD** — order immediately CONFIRMED
- [ ] Customer can complete checkout with **bank transfer** — sees bank details; order AWAITING_PAYMENT until admin confirms
- [ ] Customer receives order confirmation email within 2 minutes (COD) or after admin confirms (bank transfer)
- [ ] Customer can view order history and current status
- [ ] Shop is responsive on mobile (375px viewport tested)

### Admin

- [ ] Staff can log in; StoreManager cannot access category management
- [ ] Admin can create/edit/delete products with images
- [ ] Admin can manage categories (2 levels)
- [ ] Admin can view and adjust inventory; low-stock items highlighted
- [ ] Admin can confirm bank transfer payment (AWAITING_PAYMENT → CONFIRMED)
- [ ] Admin can filter orders and update status
- [ ] Admin can cancel orders and stock is restored
- [ ] Dashboard shows order count and revenue for last 30 days

### Backend

- [ ] All API endpoints documented in OpenAPI spec
- [ ] Input validation returns RFC 7807 Problem Details errors
- [ ] Stock cannot go negative; concurrent checkout handled safely
- [ ] Idempotency on order creation endpoint

### DevOps

- [ ] CI pipeline passes on main branch
- [ ] Staging environment accessible to client for UAT
- [ ] Production deployed with HTTPS
- [ ] Database daily automated backup enabled
- [ ] Deploy rollback procedure documented

---

## 2.8 Change Request Process

Any request outside Section 2.1 requires a written change request with:

1. Description of change  
2. Impact on timeline (days)  
3. Impact on budget (VND)  
4. Client approval before work starts  

**Guideline:** 1 person-day ≈ 3,500,000–4,000,000 VND (based on 220M / ~55–60 dev-days after overhead).
