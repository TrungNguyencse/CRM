# Phase 3 — Frontend

> **Owner:** FE dev · **Depends on:** [backend.md](./backend.md) APIs

---

## New admin screens

| ID | Screen | Route | Users |
|----|--------|-------|-------|
| A12 | POS terminal | `/admin/pos` | Cashier, StoreManager |
| A13 | Cashier session | `/admin/pos/session` | Cashier |
| A14 | Customer 360 | `/admin/customers/[id]` | Admin, StoreManager |
| A15 | Customer list / search | `/admin/customers` | Admin, StoreManager |
| A16 | Loyalty tiers admin | `/admin/loyalty/tiers` | Admin |
| A17 | Segments | `/admin/loyalty/segments` | Admin |

## New shop screen

| ID | Screen | Route |
|----|--------|-------|
| S11 | Loyalty dashboard | `/account/loyalty` |

---

## POS terminal UX (A12)

| Area | Elements |
|------|----------|
| Product search | Barcode scan input + name/SKU search |
| Cart | Line items, qty, discounts (pricing service) |
| Customer link | Optional — phone/email lookup for loyalty |
| Tender | Cash, card (record only), change calculation |
| Actions | Complete sale, void line, open/close session |

**Roles:** Cashier sells; StoreManager voids; Admin full access.

---

## Customer 360 (A14)

| Section | Data source |
|---------|-------------|
| Profile | Identity |
| Order history | Order (online + POS) |
| Points & tier | CRM |
| Segments | CRM |
| Interactions | CRM interaction log |
| Coupons used | Pricing |

---

## FE tasks

| ID | Task | Est. |
|----|------|------|
| P3-FE-01 | POS terminal A12–A13 | 12d |
| P3-FE-02 | Customer 360 A14–A15 | 6d |
| P3-FE-03 | Loyalty admin A16–A17 | 5d |
| P3-FE-04 | Shop loyalty dashboard S11 | 4d |
| P3-FE-05 | Notification template admin | 3d |

**Total:** ~30 dev-days

---

## Components to add

- `POSTerminal` — barcode input, cart, tender
- `CashierSessionBar` — open/close float
- `Customer360Panel` — tabbed aggregate view
- `LoyaltyTierEditor` — admin A16
- `SegmentRuleBuilder` — admin A17
- `LoyaltyDashboard` — shop S11

Base patterns → [docs-ba/05-frontend](../../docs-ba/05-frontend-specifications.md)
