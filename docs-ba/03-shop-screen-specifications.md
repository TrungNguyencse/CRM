# 03 — Public Shop Screen Specifications

> **App:** Next.js 14 (SSR/ISR) · **Screens:** 10 · **Users:** Guest, Registered Customer

Each section follows: **Purpose → Wireframe ref → UI elements → User flows → FE validation → BE integration → Error states**

Wireframe references: [mockups/index.html](./mockups/index.html)

---

## S01 — Home

### Purpose
Landing page showcasing featured products and top-level categories. Primary entry point for SEO and marketing.

### Route
`/` (SSR)

### UI Elements

| Element | Type | Description |
|---------|------|-------------|
| Header | Component | Logo, search bar, cart icon (badge count), login/account link |
| Hero banner | Static image | Client-provided banner; single slide MVP |
| Category grid | Grid (max 8) | Top-level categories with image + name |
| Featured products | Product card grid | 8 products flagged `featured=true` |
| Footer | Component | Links, contact, copyright |

### User Flows

1. Guest lands on home → browses categories or featured products → navigates to S02 or S03.
2. Returning customer → sees cart badge if items exist.

### FE Validation

| Field / Action | Rule | Error message |
|----------------|------|---------------|
| Search (header) | Min 2 chars to submit | "Enter at least 2 characters" |
| Search (header) | Max 100 chars | Truncate silently |

### BE Integration

| Endpoint | Method | Notes |
|----------|--------|-------|
| `GET /v1/categories?level=0&limit=8` | GET | Top categories |
| `GET /v1/products?featured=true&limit=8` | GET | Featured products |
| `GET /v1/cart` | GET | Cart count (cookie/session) |

### Error States

- API failure → show skeleton then inline retry banner: "Unable to load products. Please refresh."
- Empty featured → hide section gracefully.

---

## S02 — Category Listing

### Purpose
Display paginated products within a category (and subcategories if level-2).

### Route
`/categories/[slug]` (ISR, revalidate 60s)

### UI Elements

| Element | Type | Description |
|---------|------|-------------|
| Breadcrumb | Nav | Home > Category > Subcategory |
| Subcategory chips | Filter | Visible if parent category has children |
| Sort dropdown | Select | `newest`, `price_asc`, `price_desc`, `name_asc` |
| Product grid | Cards | Image, name, price, "Add to cart" / "Out of stock" |
| Pagination | Controls | 20 items per page |

### User Flows

1. User selects category from home → sees product grid.
2. User clicks subcategory chip → filters within branch.
3. User clicks product card → S03.

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Page query param | Integer ≥ 1 | Reset to page 1 |
| Sort | Enum whitelist | Default `newest` |

### BE Integration

| Endpoint | Notes |
|----------|-------|
| `GET /v1/categories/{slug}` | Category metadata + children |
| `GET /v1/products?category={slug}&sort=&page=&size=20` | Paginated list |

### Business Rules (display)

- Show **"Out of stock"** badge and disable "Add to cart" when `stockQuantity <= 0`.
- Prices displayed in VND format: `₫150,000` (no decimals).

---

## S03 — Product Detail (PDP)

### Purpose
Show full product information and allow add-to-cart.

### Route
`/products/[slug]` (ISR, revalidate 60s)

### UI Elements

| Element | Description |
|---------|-------------|
| Image gallery | Primary + thumbnails (max 5) |
| Product name, brand, SKU | Text |
| Price | Single price (MVP — no variants) |
| Stock indicator | "In stock" / "Only X left" (if ≤ 10) / "Out of stock" |
| Quantity selector | Stepper, min 1, max = min(stock, 99) |
| Add to cart button | Primary CTA |
| Description | Rich text (plain text MVP) |
| Breadcrumb | Home > Category > Product |

### User Flows

1. View product → select quantity → Add to cart → toast "Added to cart" → option to go to S05 or continue shopping.
2. Out of stock → Add to cart disabled.

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Quantity | Integer 1–99 | "Quantity must be between 1 and 99" |
| Quantity | ≤ available stock | "Only {n} items available" |

### BE Integration

| Endpoint | Notes |
|----------|-------|
| `GET /v1/products/{slug}` | Full product + stock |
| `POST /v1/cart/items` | Body: `{ sku, quantity }` |

### BE Business Rules

- Reject add-to-cart if `quantity > stockQuantity`.
- If guest: cart keyed by `session_id` cookie (HttpOnly, 7-day expiry).
- If logged in: cart keyed by `user_id`.

---

## S04 — Search Results

### Purpose
Full-text search across product name, brand, and SKU.

### Route
`/search?q={query}` (SSR)

### UI Elements

Same product grid as S02, plus:
- Search query displayed: "Results for **{query}**"
- Result count
- Empty state illustration if no results

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| `q` | Min 2 chars | Redirect to home with message |
| `q` | Max 100 chars | Truncate |
| Page, sort | Same as S02 | — |

### BE Integration

`GET /v1/products/search?q={query}&page=&size=20&sort=`

### BE Business Rules

- PostgreSQL `tsvector` search on `name`, `brand`, `sku`.
- Return empty array (not 404) when no matches.

---

## S05 — Shopping Cart

### Purpose
Review cart items, update quantities, proceed to checkout.

### Route
`/cart` (CSR)

### UI Elements

| Element | Description |
|---------|-------------|
| Line items table | Image, name, unit price, qty stepper, line total, remove |
| Order summary sidebar | Subtotal, tax, shipping (flat fee), total |
| Promo code field | **Disabled in MVP** — show "Coming soon" tooltip |
| Checkout button | Navigate to S06 |
| Continue shopping link | → S01 |

### User Flows

1. Update quantity → debounced API call → recalculate totals.
2. Remove item → confirm dialog → remove.
3. Empty cart → empty state with CTA to shop.
4. Click Checkout → if not logged in, redirect to S07 with `?redirect=/checkout`.

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Quantity per line | Integer 1–99 | Same as S03 |
| Quantity per line | ≤ stock | "Only {n} available for {productName}" |

### BE Integration

| Endpoint | Method | Notes |
|----------|--------|-------|
| `GET /v1/cart` | GET | Full cart with line totals |
| `PATCH /v1/cart/items/{sku}` | PATCH | Update quantity |
| `DELETE /v1/cart/items/{sku}` | DELETE | Remove line |
| `GET /v1/settings/tax-rate` | GET | Tax % for display |

### BE Business Rules

- Cart expires after **7 days** (guest) or **30 days** (logged-in).
- Prices snapshotted at add-to-cart time; rechecked at checkout (price change warning if diff > 0).
- Shipping fee: flat rate from system settings (default ₫30,000).

---

## S06 — Checkout

### Purpose
Collect shipping address, payment method, and order summary. **No online payment gateway** — customer pays via COD or bank transfer offline.

### Route
`/checkout` (CSR) — **Requires authentication**

### UI Elements

| Section | Elements |
|---------|----------|
| Shipping address | Full name, phone, address line 1, address line 2 (opt), city, district, postal code |
| Saved addresses | Radio select if customer has saved address (max 3) |
| Payment method | Radio: **Cash on Delivery (COD)** / **Bank Transfer** |
| Bank transfer info panel | Shown when bank transfer selected: bank name, account number, account holder, transfer amount, order reference note |
| Transfer reference (optional) | Text input — customer enters their transfer memo/reference |
| Order summary | Line items (read-only), subtotal, tax, shipping, total |
| Place order button | Disabled until form valid |
| Terms checkbox | "I agree to Terms & Conditions" (required) |

### User Flows

1. **COD:** Customer selects COD → Place order → order `CONFIRMED` → redirect to S10 with success banner + confirmation email.
2. **Bank transfer:** Customer selects bank transfer → sees store bank details → optionally enters transfer reference → Place order → order `AWAITING_PAYMENT` → redirect to S10 with instructions to complete transfer.
3. Stock insufficient at submit → error listing affected SKUs → redirect to S05.

### FE Validation

| Field | Rule | Error message |
|-------|------|---------------|
| Full name | Required, 2–100 chars | "Full name is required" |
| Phone | Required, VN format `0[3|5|7|8|9]XXXXXXXX` or `+84...` | "Enter a valid phone number" |
| Address line 1 | Required, 5–200 chars | "Address is required" |
| Address line 2 | Optional, max 200 chars | — |
| City | Required, from dropdown | "Select a city" |
| District | Required, from dropdown | "Select a district" |
| Postal code | Optional, 5–6 digits | "Invalid postal code" |
| Payment method | Required, enum: `COD`, `BANK_TRANSFER` | "Select a payment method" |
| Transfer reference | Optional, max 100 chars | — |
| Terms checkbox | Must be checked | "You must accept the terms" |

### BE Integration

| Endpoint | Method | Notes |
|----------|--------|-------|
| `POST /v1/orders/checkout` | POST | Creates order with payment method |
| `GET /v1/customers/me/addresses` | GET | Saved addresses |
| `GET /v1/settings/payment-info` | GET | Bank account details for transfer display |

### Request body — `POST /v1/orders/checkout`

```json
{
  "shippingAddress": {
    "fullName": "string",
    "phone": "string",
    "line1": "string",
    "line2": "string|null",
    "city": "string",
    "district": "string",
    "postalCode": "string|null"
  },
  "paymentMethod": "COD",
  "transferReference": "string|null",
  "idempotencyKey": "uuid"
}
```

### BE Business Rules

1. Verify cart not empty.
2. Re-validate stock for all lines.
3. Calculate: `subtotal + tax + shipping = total`.
4. **If COD:** create order `CONFIRMED`, deduct stock, send confirmation email.
5. **If BANK_TRANSFER:** create order `AWAITING_PAYMENT`, reserve stock (not deduct until confirmed), send "awaiting payment" email with bank details.
6. Idempotency key prevents duplicate orders on double-click (24h window).
7. Auto-cancel `AWAITING_PAYMENT` orders after **48 hours** if not confirmed by admin.

### Error States

| Code | Scenario | User message |
|------|----------|--------------|
| 409 | Stock insufficient | "Some items are no longer available" |
| 422 | Validation error | Field-level messages |

---

## S07 — Login / Register

### Purpose
Authenticate customers; support redirect back to checkout.

### Routes
- `/auth/login`
- `/auth/register`
- `/auth/forgot-password`

### UI Elements — Login

| Field | Type |
|-------|------|
| Email | Email input |
| Password | Password input |
| Remember me | Checkbox (extends refresh token) |
| Login button | Submit |
| Register link | → register tab/page |
| Forgot password link | → forgot password |

### UI Elements — Register

| Field | Type |
|-------|------|
| Email | Email input |
| Password | Password input |
| Confirm password | Password input |
| Full name | Text input |
| Phone | Tel input |
| Marketing opt-in | Checkbox (optional) |
| Register button | Submit |

### FE Validation — Login

| Field | Rule | Error message |
|-------|------|---------------|
| Email | Required, valid email | "Enter a valid email" |
| Password | Required, min 8 chars | "Password is required" |

### FE Validation — Register

| Field | Rule | Error message |
|-------|------|---------------|
| Email | Required, valid email, max 255 | "Enter a valid email" |
| Password | Min 8 chars, 1 upper, 1 lower, 1 digit | "Password must be at least 8 characters with upper, lower, and number" |
| Confirm password | Must match password | "Passwords do not match" |
| Full name | Required, 2–100 chars | "Full name is required" |
| Phone | VN phone format | "Enter a valid phone number" |

### BE Integration

| Endpoint | Notes |
|----------|-------|
| `POST /v1/auth/register` | Creates customer user |
| `POST /v1/auth/login` | Returns access + refresh tokens |
| `POST /v1/auth/forgot-password` | Sends reset email |
| `POST /v1/auth/reset-password` | Token + new password |
| `POST /v1/auth/refresh` | Refresh access token |

### BE Business Rules

- Email unique (case-insensitive).
- Password hashed bcrypt cost 12.
- Rate limit login: 5 attempts / 15 min per IP.
- On register: merge guest cart into user cart if session exists.
- JWT access token TTL: 15 min; refresh: 7 days (30 days if remember me).

---

## S08 — Account Profile

### Purpose
View and edit customer profile.

### Route
`/account/profile` (CSR) — **Auth required**

### UI Elements

| Field | Editable |
|-------|----------|
| Email | Read-only |
| Full name | Yes |
| Phone | Yes |
| Date of birth | Yes (optional) |
| Marketing opt-in | Yes (checkbox) |
| Change password section | Link to modal |
| Saved addresses | List + add/edit (max 3) |

### FE Validation

Same rules as S06 address fields + S07 password rules for change password modal.

### BE Integration

| Endpoint | Method |
|----------|--------|
| `GET /v1/customers/me` | GET |
| `PATCH /v1/customers/me` | PATCH |
| `POST /v1/customers/me/addresses` | POST |
| `PATCH /v1/customers/me/addresses/{id}` | PATCH |
| `DELETE /v1/customers/me/addresses/{id}` | DELETE |
| `POST /v1/auth/change-password` | POST |

### BE Business Rules

- Customer can only access own profile (`sub` match).
- Max 3 saved addresses; default address flagged.

---

## S09 — Order History

### Purpose
List customer's past orders.

### Route
`/account/orders` (CSR) — **Auth required**

### UI Elements

| Element | Description |
|---------|-------------|
| Order cards/table | Order #, date, status badge, total, item count |
| Status filter | All, Pending, Confirmed, Shipped, Delivered, Cancelled |
| Pagination | 10 orders per page |
| Empty state | "No orders yet" + CTA to shop |

### FE Validation

| Param | Rule |
|-------|------|
| status filter | Enum whitelist |
| page | Integer ≥ 1 |

### BE Integration

`GET /v1/orders/me?status=&page=&size=10`

---

## S10 — Order Detail / Tracking

### Purpose
Show order line items, shipping address, payment summary, and status timeline.

### Route
`/account/orders/[id]` (CSR) — **Auth required**

### UI Elements

| Section | Content |
|---------|---------|
| Order header | Order #, placed date, status badge |
| Status timeline | Visual stepper: Placed → Confirmed → Shipped → Delivered |
| Line items | Product name, SKU, qty, unit price, line total |
| Shipping address | Full address block |
| Payment summary | Method (COD / Bank transfer), status, transfer reference if any |
| Actions | "Cancel order" (if AWAITING_PAYMENT or CONFIRMED not shipped); "Continue shopping" |

### User Flows

1. Post-checkout redirect lands here with success banner.
2. Customer cancels AWAITING_PAYMENT or CONFIRMED (not shipped) order → confirmation → status CANCELLED.

### FE Validation

- Order ID must be UUID format; 404 page if not found or not owned.

### BE Integration

| Endpoint | Notes |
|----------|-------|
| `GET /v1/orders/me/{id}` | Full order detail |
| `POST /v1/orders/me/{id}/cancel` | Customer cancel (AWAITING_PAYMENT or CONFIRMED if not shipped) |

### BE Business Rules

- Customer can only view own orders.
- Cancel releases stock reservation (bank transfer) or restores stock (COD if not shipped).
- Tracking number shown when status ≥ SHIPPED (manual entry by admin).

---

## Cross-Cutting Shop Requirements

### Authentication

- Access token in memory; refresh token in HttpOnly secure cookie.
- Protected routes redirect to S07 with return URL.

### Responsive Breakpoints

| Breakpoint | Layout |
|------------|--------|
| < 640px | Single column, hamburger nav |
| 640–1024px | 2-column product grid |
| > 1024px | 3–4 column grid, full header |

### Accessibility (MVP baseline)

- All form inputs have labels.
- Focus visible on interactive elements.
- Status badges include text (not color-only).
- Alt text on product images (product name).

### Performance

- LCP target < 2.5s on 4G for S01, S03 (ISR).
- Images via Next.js Image + CloudFront CDN.
