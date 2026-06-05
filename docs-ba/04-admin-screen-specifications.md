# 04 — Admin Screen Specifications

> **App:** Next.js 14 (CSR) · **Screens:** A01–A08 · **Wireframes:** [mockups/index.html](./mockups/index.html)

Each screen: **Purpose → UI → Flows → FE validation → BE integration → Errors**

---

## A01 — Admin Login

### Purpose
Authenticate staff users for admin dashboard access.

### Route
`/admin/login`

### UI Elements

| Element | Description |
|---------|-------------|
| Logo + title | "Store Admin" |
| Email | Staff email |
| Password | Masked input |
| Login button | Submit |
| Error alert | Invalid credentials message |

### User Flows

1. Staff enters credentials → success → redirect to A02.
2. StoreManager/Admin roles routed to same dashboard; permissions differ per route.
3. Customer accounts cannot log in here (BE rejects non-staff type).

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Email | Required, valid email | "Enter a valid email" |
| Password | Required | "Password is required" |

### BE Integration

`POST /v1/auth/login` — same endpoint as shop; response includes `roles[]`.

### BE Business Rules

- Only users with `type=STAFF` and role `Admin` or `StoreManager` may access admin routes.
- Return generic "Invalid email or password" (no user enumeration).
- Rate limit: 5 attempts / 15 min per IP.

---

## A02 — Dashboard

### Purpose
At-a-glance KPIs and quick links for daily operations.

### Route
`/admin` — **Auth required (Admin, StoreManager)**

### UI Elements

| Widget | Description |
|--------|-------------|
| Orders today | Count + comparison to yesterday |
| Revenue (30 days) | Sum of CONFIRMED+ orders in VND |
| Pending orders | Count with link to A07 filtered |
| Low stock alerts | Top 5 SKUs below reorder point |
| Recent orders table | Last 10 orders (order #, customer, total, status) |
| Quick actions | "Add product", "View orders" buttons |

### User Flows

1. Admin lands after login → scans KPIs → clicks through to relevant screens.

### BE Integration

| Endpoint | Notes |
|----------|-------|
| `GET /v1/admin/dashboard/summary` | Aggregated KPIs |
| `GET /v1/admin/orders?size=10&sort=newest` | Recent orders |
| `GET /v1/admin/inventory/low-stock?limit=5` | Low stock items |

### Business Rules

- Revenue excludes CANCELLED and REFUNDED orders.
- Low stock: `quantityOnHand <= reorderPoint` (default reorder point: 10).

---

## A03 — Product List

### Purpose
Browse, search, filter, and manage products.

### Route
`/admin/products` — **Auth required**

### UI Elements

| Element | Description |
|---------|-------------|
| Search bar | Search by name or SKU |
| Status filter | All, Active, Draft, Archived |
| Category filter | Dropdown |
| Data table | Image thumb, name, SKU, category, price, stock, status, actions |
| Pagination | 25 per page |
| Add product button | → A04 |
| Row actions | Edit, Archive (soft delete) |

### User Flows

1. Search/filter products → click Edit → A04.
2. Click "Add product" → A04 (create mode).
3. Archive product → confirm dialog → product hidden from shop.

### FE Validation

| Param | Rule |
|-------|------|
| search | Max 100 chars |
| status | Enum: ACTIVE, DRAFT, ARCHIVED |
| page | ≥ 1 |

### BE Integration

`GET /v1/admin/products?search=&status=&category=&page=&size=25`

### BE Business Rules

- Archived products not visible on shop.
- Cannot archive product with open PENDING orders containing that SKU.

---

## A04 — Product Create / Edit

### Purpose
Create or update product details and images.

### Routes
- `/admin/products/new`
- `/admin/products/[id]`

### UI Elements

| Field | Type | Required |
|-------|------|----------|
| Name | Text | Yes |
| SKU | Text (unique) | Yes |
| Brand | Text | No |
| Category | Multi-select (max 3) | Yes (≥1) |
| Description | Textarea | No |
| Base price (VND) | Number | Yes |
| Status | Select: Draft / Active | Yes |
| Featured | Checkbox | No |
| Reorder point | Number | No (default 10) |
| Images | Upload (drag-drop) | Yes (≥1 for Active) |

### User Flows

1. Fill form → Save as Draft → product not on shop.
2. Fill form → Publish (Active) → visible on shop (requires ≥1 image, stock ≥ 0 optional).
3. Upload images → preview thumbnails → reorder → delete.

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Name | Required, 2–200 chars | "Product name is required" |
| SKU | Required, 3–50 chars, alphanumeric + hyphen | "SKU must be 3–50 alphanumeric characters" |
| Brand | Max 100 chars | — |
| Description | Max 5000 chars | — |
| Base price | Required, integer > 0, max 999,999,999 | "Enter a valid price" |
| Category | At least 1 selected | "Select at least one category" |
| Images | Max 5 files | "Maximum 5 images" |
| Images | JPG/PNG/WebP, max 5MB each | "File must be JPG/PNG/WebP under 5MB" |
| Reorder point | Integer 0–9999 | "Invalid reorder point" |

### BE Integration

| Endpoint | Method |
|----------|--------|
| `POST /v1/admin/products` | POST |
| `PUT /v1/admin/products/{id}` | PUT |
| `POST /v1/admin/products/{id}/images` | POST (multipart) |
| `DELETE /v1/admin/products/{id}/images/{imageId}` | DELETE |
| `GET /v1/admin/categories` | GET (for dropdown) |

### BE Business Rules

- SKU unique (case-insensitive).
- On create: auto-create inventory record with `quantityOnHand = 0`.
- Image upload to S3 path: `products/{productId}/{uuid}.{ext}`.
- Only Admin and StoreManager can create/edit products.
- Slug auto-generated from name; regenerate if name changes and slug not manually set.

---

## A05 — Category Management

### Purpose
Manage product category hierarchy (max 2 levels in MVP).

### Route
`/admin/categories` — **Auth required (Admin only)**

### UI Elements

| Element | Description |
|---------|-------------|
| Tree table | Category name, slug, parent, product count, actions |
| Add category modal | Name, parent (optional), image upload |
| Edit / Delete actions | Delete blocked if products assigned |

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Name | Required, 2–100 chars | "Category name is required" |
| Slug | Auto from name; editable, 2–100, lowercase hyphen | "Invalid slug format" |
| Parent | Optional; cannot be self or child | "Invalid parent category" |
| Image | Optional, JPG/PNG, max 2MB | File type message |

### BE Integration

| Endpoint | Method |
|----------|--------|
| `GET /v1/admin/categories?tree=true` | GET |
| `POST /v1/admin/categories` | POST |
| `PUT /v1/admin/categories/{id}` | PUT |
| `DELETE /v1/admin/categories/{id}` | DELETE |

### BE Business Rules

- Max depth: 2 levels (root → child). Reject deeper nesting.
- Slug unique across all categories.
- Cannot delete category with products; must reassign first.
- StoreManager role → 403 Forbidden on all category endpoints.

---

## A06 — Inventory Management

### Purpose
View and adjust stock levels; monitor low-stock items.

### Route
`/admin/inventory` — **Auth required**

### UI Elements

| Element | Description |
|---------|-------------|
| Search / filter | By SKU or product name; filter "Low stock only" |
| Data table | SKU, product name, on-hand, reserved, available, reorder point, last adjusted |
| Adjust stock modal | SKU (read-only), adjustment type (+/-), quantity, reason (required) |
| Adjustment history link | Modal with last 20 movements for SKU |

### User Flows

1. Staff searches SKU → opens Adjust modal → enters +50 "Stock receipt" → saves → table updates.
2. Filter low stock → export CSV (optional nice-to-have; **out of MVP** if time constrained).

### FE Validation — Adjust Modal

| Field | Rule | Error message |
|-------|------|---------------|
| Adjustment type | Enum: ADD, REMOVE | Required |
| Quantity | Integer 1–9999 | "Quantity must be 1–9999" |
| Reason | Required, 3–200 chars | "Reason is required" |
| REMOVE quantity | ≤ available (on-hand minus reserved) | "Cannot remove more than available stock" |

### BE Integration

| Endpoint | Method |
|----------|--------|
| `GET /v1/admin/inventory?search=&lowStock=&page=` | GET |
| `POST /v1/admin/inventory/adjust` | POST |
| `GET /v1/admin/inventory/{sku}/movements?limit=20` | GET |

### Request — Adjust Stock

```json
{
  "sku": "SKU-001",
  "type": "ADD",
  "quantity": 50,
  "reason": "Weekly stock receipt"
}
```

### BE Business Rules

- Append-only movement ledger entry for every adjustment.
- `available = quantityOnHand - quantityReserved`.
- REMOVE cannot drop below reserved quantity.
- Admin and StoreManager can adjust.

---

## A07 — Order List

### Purpose
View and filter all customer orders.

### Route
`/admin/orders` — **Auth required**

### UI Elements

| Element | Description |
|---------|-------------|
| Filters | Status, date range (from/to), search by order # or customer email |
| Data table | Order #, date, customer, items count, total, status, actions |
| Pagination | 25 per page |
| Export | **Out of MVP** (Phase 2) |

### FE Validation

| Field | Rule |
|-------|------|
| Date from/to | Valid dates; from ≤ to |
| Status | Enum whitelist |
| Search | Max 100 chars |

### BE Integration

`GET /v1/admin/orders?status=&from=&to=&search=&page=&size=25`

---

## A08 — Order Detail

### Purpose
View full order; update status; enter tracking number; cancel or refund.

### Route
`/admin/orders/[id]` — **Auth required**

### UI Elements

| Section | Content |
|---------|---------|
| Order header | Order #, status (editable dropdown), placed date |
| Customer info | Name, email, phone (read-only) |
| Shipping address | Full address (read-only) |
| Line items | SKU, name, qty, price, line total |
| Payment info | Method (COD / Bank transfer), status, transfer reference |
| Status history | Timeline with timestamp + actor |
| Tracking number | Text input (visible when status → SHIPPED) |
| Action buttons | Update status, **Confirm payment** (bank transfer only), Cancel order |

### User Flows

1. **Confirm bank transfer:** Order is AWAITING_PAYMENT → Admin clicks "Confirm payment" → status CONFIRMED → stock deducted → confirmation email sent.
2. **Confirm → Shipped:** Admin selects SHIPPED, enters tracking number, saves.
3. **Shipped → Delivered:** Admin marks delivered.
4. **Cancel:** Confirm dialog → cancel → restore stock → email customer.

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Status transition | Valid per state machine (see [02 § 2.5](./02-scope-and-assumptions.md#25-order-status-lifecycle)) | "Invalid status transition" |
| Tracking number | Required when status=SHIPPED | "Tracking number is required" |
| Tracking number | 5–50 chars alphanumeric | "Invalid tracking number" |

### BE Integration

| Endpoint | Method |
|----------|--------|
| `GET /v1/admin/orders/{id}` | GET |
| `PATCH /v1/admin/orders/{id}/status` | PATCH |
| `POST /v1/admin/orders/{id}/confirm-payment` | POST | Bank transfer only |
| `POST /v1/admin/orders/{id}/cancel` | POST |

### Request — Update Status

```json
{
  "status": "SHIPPED",
  "trackingNumber": "VN123456789",
  "note": "Shipped via local courier"
}
```

### BE Business Rules

| From | Allowed to |
|------|------------|
| AWAITING_PAYMENT | CONFIRMED, CANCELLED |
| CONFIRMED | SHIPPED, CANCELLED |
| SHIPPED | DELIVERED |
| DELIVERED | — (terminal) |
| CANCELLED | — (terminal) |

- **Confirm payment:** only from `AWAITING_PAYMENT`; deducts stock and sends confirmation email.
- Cancel AWAITING_PAYMENT or CONFIRMED order: restore stock + email customer.
- Actual money return (bank transfer) is processed offline by client — system records cancellation only.
- All status changes logged in `order_status_history`.

---

## Cross-Cutting Admin Requirements

### Layout

- Sidebar navigation: Dashboard, Products, Categories (Admin only), Inventory, Orders.
- Top bar: staff name, role badge, logout.

### Authorization (FE route guards)

| Route | Admin | StoreManager |
|-------|:-----:|:------------:|
| A02–A04, A06–A08 | ✅ | ✅ |
| A05 Categories | ✅ | ❌ |
| Refund action | — | — |
| Confirm bank transfer | ✅ | ✅ |

### Session

- Same JWT mechanism as shop.
- Inactivity timeout: 30 min (refresh token revoked on logout).

### UI Framework

- shadcn/ui DataTable, Dialog, Form components.
- Toast notifications on save success/failure.
