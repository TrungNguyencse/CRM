# 02 — Scope, Assumptions & Acceptance

---

## 2.1 In Scope

### Functional
- Customer auth, browse, search, cart, checkout (COD / bank transfer)
- Order history, confirmation email
- Admin: products, categories, inventory, orders, dashboard
- Bank transfer confirmation; stock reservation

### Non-functional
- Shop responsive 320px+; Admin 1280px+
- API p95 < 1s; HTTPS; staging + prod; CI; CloudWatch

---

## 2.2 Out of Scope

| Item | Phase |
|------|-------|
| Mobile, POS, loyalty, coupons | 2–4 |
| VNPay, MoMo, Stripe | 2 |
| Kafka, Temporal, EKS | 5 |
| Multi-store, variants, i18n | 2+ |

→ [docs-phases](../docs-phases/README.md)

---

## 2.3 Assumptions

| # | Assumption |
|---|------------|
| A1 | Client provides bank account for transfer display |
| A2 | AWS account access |
| A3 | Domain + DNS |
| A4 | ≤ 500 SKUs at launch |
| A5 | Single warehouse |
| A6 | Review ≤ 3 business days per milestone |
| A10 | COD acceptable for launch |

---

## 2.4 Roles & Permissions

| Permission | Customer | StoreManager | Admin |
|------------|:--------:|:------------:|:-----:|
| Shop browse/order | ✅ | — | — |
| Admin login | — | ✅ | ✅ |
| Products & inventory | — | ✅ | ✅ |
| Categories | — | — | ✅ |
| Confirm bank transfer | — | ✅ | ✅ |

---

## 2.5 Order Status Lifecycle

**COD:** CONFIRMED → SHIPPED → DELIVERED  
**Bank transfer:** AWAITING_PAYMENT → CONFIRMED → SHIPPED → DELIVERED

- Cancel AWAITING_PAYMENT anytime before admin confirms
- Cancel CONFIRMED if not SHIPPED
- Auto-cancel transfer after **48h**
- Refunds manual (no gateway)

---

## 2.6 Risks

| Risk | Mitigation |
|------|------------|
| Scope creep | Change request; Phase 2 backlog |
| COD no-shows | Policy on checkout |
| Transfer timeout | 48h cancel + 24h reminder |

---

## 2.7 Acceptance Criteria (UAT Sign-off)

### Shop
- [ ] Register, login, profile
- [ ] Browse, search, cart
- [ ] Checkout COD → CONFIRMED
- [ ] Checkout bank transfer → AWAITING_PAYMENT → admin confirms
- [ ] Confirmation email; order history
- [ ] Mobile responsive (375px)

### Admin
- [ ] Login; StoreManager blocked from categories
- [ ] Product CRUD; categories (2 levels)
- [ ] Inventory adjust; low-stock alerts
- [ ] Confirm transfer; order status; cancel restores stock
- [ ] Dashboard KPIs

### Backend
- [ ] OpenAPI; RFC 7807 errors
- [ ] Safe stock; idempotent order create

### DevOps
- [ ] CI green; staging UAT; prod HTTPS; RDS backup; rollback runbook

---

## 2.8 Change Requests

Outside §2.1 requires written CR: description, timeline, budget, client approval.  
**Rate:** ~1,830,000 VND / dev-day · min 5 days.
