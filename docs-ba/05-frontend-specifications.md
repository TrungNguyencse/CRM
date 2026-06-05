# 05 — Frontend Specifications

> **Owner:** FE Developer · **Apps:** Shop + Admin · **Screens:** 18  
> **Stack:** Next.js 14 · Tailwind · shadcn/ui · React Query · Zod

FE engineering tasks. Screen specs → [03-shop](./03-shop-screen-specifications.md) · [04-admin](./04-admin-screen-specifications.md). APIs → [06-backend](./06-backend-specifications.md).

---

## 5.1 Document Map

| Need | Document |
|------|----------|
| Per-screen UI, flows, validation | [03-shop](./03-shop-screen-specifications.md), [04-admin](./04-admin-screen-specifications.md) |
| FE structure, components, tasks | **This file (05)** |
| API contracts | [06-backend](./06-backend-specifications.md) |
| Wireframes | [mockups/index.html](./mockups/index.html) |
| Screen list | [01 § 1.4](./01-project-overview.md#14-screen-inventory) |

---

## 8.2 Applications Overview

| App | Folder | Rendering | Port (local) | Auth |
|-----|--------|-----------|--------------|------|
| **Shop** | `web/shop-next` | SSR / ISR / CSR mix | 3000 | Customer JWT |
| **Admin** | `web/admin-next` | CSR only | 3001 | Staff JWT + role guard |

Both apps share: TypeScript types, Zod schemas (optional `packages/shared`), API client pattern.

---

## 8.3 Repository Structure (FE)

```
web/
  shop-next/
    app/
      (public)/              # S01–S06, layout with header/footer
        page.tsx               # S01 Home
        categories/[slug]/     # S02
        products/[slug]/       # S03
        search/                # S04
        cart/                  # S05
        checkout/              # S06
      auth/
        login/                 # S07
        register/
        forgot-password/
      account/
        profile/               # S08
        orders/                # S09
        orders/[id]/           # S10
    components/
      layout/                  # Header, Footer, MobileNav
      product/                 # ProductCard, ProductGrid, QuantityStepper
      cart/                    # CartLineItem, OrderSummary
      checkout/                # AddressForm, PaymentMethodSelector
      order/                   # OrderStatusBadge, OrderTimeline
      ui/                      # shadcn primitives
    lib/
      api/                     # fetch wrapper, endpoints
      auth/                    # token storage, session provider
      hooks/                   # useCart, useAuth, useDebounce
      schemas/                 # Zod schemas
      utils/                   # formatVND, formatDate
  admin-next/
    app/
      admin/
        login/                 # A01
        page.tsx               # A02 Dashboard
        products/              # A03, A04
        categories/            # A05
        inventory/             # A06
        orders/                # A07, A08
    components/
      layout/                  # AdminSidebar, AdminTopBar
      data-table/              # reusable DataTable + filters
      forms/                   # ProductForm, CategoryForm, AdjustStockModal
      dashboard/               # KpiCard, RecentOrdersTable
    lib/                       # same pattern as shop
```

---

## 8.4 Foundation Tasks (Sprint 1)

Complete before screen implementation. **~5 dev-days.**

| Task ID | Task | Details | DoD |
|---------|------|---------|-----|
| FE-00 | Monorepo / project bootstrap | `create-next-app` × 2, Tailwind, ESLint, Prettier, path aliases `@/` | Both apps `npm run dev` without errors |
| FE-01 | Install shadcn/ui | Button, Input, Label, Select, Dialog, Table, Toast, Form, Badge, Card, Skeleton | Story-less smoke render in dev |
| FE-02 | API client | `lib/api/client.ts`: base URL from env, `Authorization` header, RFC 7807 error parser, 401 → refresh flow | Unit test error parsing |
| FE-03 | React Query setup | `QueryClientProvider`, default staleTime, devtools (dev only) | Sample query works |
| FE-04 | Zod + react-hook-form | Shared form pattern with `@hookform/resolvers/zod` | Login form prototype |
| FE-05 | Auth — Shop | Context/hook: `useAuth`, login/logout/register, access token in memory, refresh via cookie | Login redirects with `?redirect=` |
| FE-06 | Auth — Admin | Same client; `useStaffAuth`; role in JWT (`Admin` / `StoreManager`) | Non-staff rejected on admin routes |
| FE-07 | Route guards | Shop: middleware or layout check for `/account/*`, `/checkout` | Unauthenticated → `/auth/login` |
| FE-08 | Route guards — Admin | HOC/layout: redirect unauthenticated; hide A05 for StoreManager | Role matrix in [04-admin](./04-admin-screen-specifications.md) |
| FE-09 | Shared formatters | `formatVND(150000)` → `₫150,000`; date/time locale EN | Snapshot tests |
| FE-10 | Env config | `.env.example`: `NEXT_PUBLIC_API_URL` | Documented in README |

---

## 8.5 Shared Component Library

Build once, reuse across screens.

| Component | Used in | Props / behaviour |
|-----------|---------|-------------------|
| `Header` | Shop all | Logo, `SearchBar`, cart badge (React Query), account link |
| `Footer` | Shop public | Static links |
| `MobileNav` | Shop < 640px | Hamburger, drawer |
| `SearchBar` | Header, S04 | Debounce 300ms, min 2 chars, navigates to `/search?q=` |
| `ProductCard` | S01, S02, S04 | image, name, price, stock badge, `onAddToCart` |
| `ProductGrid` | S02, S04 | responsive columns, skeleton loading |
| `QuantityStepper` | S03, S05 | min 1, max stock, disabled at 0 |
| `OrderSummary` | S05, S06 | subtotal, tax, shipping, total (fetch tax/shipping settings) |
| `AddressForm` | S06, S08 | all fields + Zod schema from [03 § S06](./03-shop-screen-specifications.md#s06--checkout) |
| `PaymentMethodSelector` | S06 | Radio COD / Bank transfer; bank panel conditional |
| `BankTransferPanel` | S06, S10 | Fetch `GET /v1/settings/payment-info`, copy-friendly layout |
| `OrderStatusBadge` | S09, S10, Admin | colour + text per status enum |
| `OrderTimeline` | S10 | stepper: Placed → Confirmed → Shipped → Delivered |
| `EmptyState` | Cart, orders, search | illustration + CTA |
| `Pagination` | S02, S04, S09, A03, A07 | page/size query sync with URL |
| `AdminSidebar` | Admin all | nav items; Categories hidden for StoreManager |
| `AdminTopBar` | Admin all | user name, role badge, logout |
| `DataTable` | A03, A06, A07 | sort, filter, pagination (shadcn Table) |
| `ImageUpload` | A04 | drag-drop, preview, max 5, type/size validation |
| `KpiCard` | A02 | title, value, optional trend |
| `ConfirmDialog` | delete, cancel order | title, description, confirm/cancel |
| `Toast` | all forms | success / error via shadcn sonner |

---

## 8.6 Shop Screen Tasks (S01–S10)

**~28 dev-days total.** Detail per screen in [03-shop](./03-shop-screen-specifications.md).

| Task ID | Screen | Route | Render | Est. | Depends | Key FE work |
|---------|--------|-------|--------|------|---------|-------------|
| FE-S01 | Home | `/` | SSR | 2d | FE-00–10 | SSR fetch categories + featured; skeleton |
| FE-S02 | Category listing | `/categories/[slug]` | ISR 60s | 3d | FE-S01, ProductCard | Breadcrumb, subcategory chips, sort, pagination |
| FE-S03 | Product detail | `/products/[slug]` | ISR 60s | 3d | QuantityStepper | Gallery, add-to-cart mutation + toast |
| FE-S04 | Search results | `/search` | SSR | 2d | ProductGrid | Query param `q`, empty state |
| FE-S05 | Cart | `/cart` | CSR | 3d | FE-S03 | Line CRUD, debounced qty update, OrderSummary |
| FE-S06 | Checkout | `/checkout` | CSR | 4d | FE-S05, AddressForm, PaymentMethodSelector | Auth gate, place order mutation, redirect S10 |
| FE-S07 | Login / Register | `/auth/*` | CSR | 3d | FE-05 | Tabs/pages, forgot password, redirect param |
| FE-S08 | Account profile | `/account/profile` | CSR | 3d | FE-07, AddressForm | Profile PATCH, address CRUD (max 3) |
| FE-S09 | Order history | `/account/orders` | CSR | 2d | OrderStatusBadge | Status filter, pagination |
| FE-S10 | Order detail | `/account/orders/[id]` | CSR | 3d | OrderTimeline | Cancel action, bank transfer instructions |

### Shop — Zod schemas (`lib/schemas/`)

| Schema file | Fields |
|-------------|--------|
| `auth.ts` | login, register, changePassword, forgotPassword |
| `address.ts` | fullName, phone, line1, line2, city, district, postalCode |
| `checkout.ts` | shippingAddress + paymentMethod + transferReference + terms |
| `search.ts` | q (min 2, max 100) |

### Shop — React Query keys

| Key | Endpoint |
|-----|----------|
| `['categories', level]` | `GET /v1/categories` |
| `['products', filters]` | `GET /v1/products` |
| `['product', slug]` | `GET /v1/products/{slug}` |
| `['search', q, page, sort]` | `GET /v1/products/search` |
| `['cart']` | `GET /v1/cart` |
| `['settings', 'tax-rate']` | `GET /v1/settings/tax-rate` |
| `['settings', 'payment-info']` | `GET /v1/settings/payment-info` |
| `['orders', 'me', filters]` | `GET /v1/orders/me` |
| `['orders', 'me', id]` | `GET /v1/orders/me/{id}` |
| `['customer', 'me']` | `GET /v1/customers/me` |
| `['addresses']` | `GET /v1/customers/me/addresses` |

### Shop — Mutations

| Mutation | Endpoint | Optimistic update? |
|----------|----------|-------------------|
| `addToCart` | `POST /v1/cart/items` | Invalidate `cart` |
| `updateCartItem` | `PATCH /v1/cart/items/{sku}` | Debounce 400ms |
| `removeCartItem` | `DELETE /v1/cart/items/{sku}` | Invalidate `cart` |
| `checkout` | `POST /v1/orders/checkout` | Idempotency-Key header (uuid) |
| `cancelOrder` | `POST /v1/orders/me/{id}/cancel` | Invalidate order queries |

---

## 8.7 Admin Screen Tasks (A01–A08)

**~22 dev-days total.** Detail per screen in [04-admin](./04-admin-screen-specifications.md).

| Task ID | Screen | Route | Est. | Depends | Key FE work |
|---------|--------|-------|------|---------|-------------|
| FE-A01 | Admin login | `/admin/login` | 1d | FE-06 | Staff-only login, redirect A02 |
| FE-A02 | Dashboard | `/admin` | 3d | FE-A01, KpiCard | 4 KPI widgets + recent orders table |
| FE-A03 | Product list | `/admin/products` | 3d | DataTable | Search, status/category filters, archive |
| FE-A04 | Product create/edit | `/admin/products/new`, `[id]` | 5d | ImageUpload | ProductForm, multipart image upload |
| FE-A05 | Categories | `/admin/categories` | 3d | FE-08 | Tree table, add/edit modal (Admin only) |
| FE-A06 | Inventory | `/admin/inventory` | 3d | DataTable | Low-stock filter, AdjustStockModal |
| FE-A07 | Order list | `/admin/orders` | 2d | DataTable | Date range, status filter |
| FE-A08 | Order detail | `/admin/orders/[id]` | 4d | OrderTimeline | Status dropdown, confirm payment btn, tracking |

### Admin — Zod schemas

| Schema file | Used in |
|-------------|---------|
| `product.ts` | A04 — name, sku, brand, price, categories, images |
| `category.ts` | A05 — name, slug, parentId |
| `inventory-adjust.ts` | A06 — type, quantity, reason |
| `order-status.ts` | A08 — status, trackingNumber, note |

### Admin — Role-based UI rules

| Element | Admin | StoreManager |
|---------|:-----:|:------------:|
| Sidebar: Categories link | Show | **Hide** |
| A05 route | Allow | **403 page** |
| A08: Confirm payment button | Show | Show |
| A08: Cancel order | Show | Show |

---

## 8.8 API Integration Layer

### `lib/api/client.ts` requirements

```typescript
// Pseudocode — implement in TypeScript
apiClient.get/post/patch/delete(path, { body?, headers? })
// Auto-attach: Authorization: Bearer {accessToken}
// On 401: attempt POST /v1/auth/refresh once, retry original request
// On 422: parse errors[] → field errors for react-hook-form
// On 409: show toast with detail message
```

### Environment variables

| Variable | App | Required |
|----------|-----|----------|
| `NEXT_PUBLIC_API_URL` | Both | Yes — e.g. `http://localhost:8080` |

### Mock API (Weeks 1–4 if BE not ready)

| Approach | Scope |
|----------|-------|
| MSW (Mock Service Worker) or JSON fixtures | Cart, products list, auth |
| Switch via `NEXT_PUBLIC_USE_MOCK=true` | Dev only |

---

## 8.9 State & Session

| Concern | Approach |
|---------|----------|
| Server state | React Query (products, cart, orders) |
| Auth state | React Context `AuthProvider` — user, roles, login/logout |
| Form state | react-hook-form (local) |
| Cart count (header) | React Query `['cart']` — `select` item count |
| URL state | `searchParams` for filters, pagination, sort |
| No Redux | Not needed for MVP |

### Token handling

| Token | Storage | Notes |
|-------|---------|-------|
| Access token | Memory (React state / module variable) | Lost on refresh — re-fetch via refresh cookie |
| Refresh token | HttpOnly cookie (set by BE) | `credentials: 'include'` on refresh call |

---

## 8.10 Routing & Middleware

### Shop middleware (`middleware.ts`)

| Path pattern | Rule |
|--------------|------|
| `/account/*` | Require auth |
| `/checkout` | Require auth |
| `/auth/*` | Redirect to `/account/profile` if already logged in |

### Admin layout guard

| Check | Action |
|-------|--------|
| No token / refresh fails | Redirect `/admin/login` |
| `roles` lacks Admin/StoreManager | Show 403 |
| StoreManager visits `/admin/categories` | Show 403 |

---

## 8.11 UI / UX Standards

| Topic | Rule |
|-------|------|
| Currency | Always `formatVND()` — no decimals |
| Dates | `DD MMM YYYY` (e.g. 30 May 2026) |
| Loading | Skeleton for lists; button spinner on submit |
| Errors | Inline field errors (forms); toast for API errors |
| Empty states | Every list screen has EmptyState + CTA |
| Confirm destructive actions | ConfirmDialog before delete/cancel/archive |
| Responsive | Shop mobile-first; Admin desktop 1280px+ |
| Breakpoints | Tailwind defaults: `sm` 640, `md` 768, `lg` 1024 |

---

## 8.12 SEO & Performance (Shop only)

| Page | Strategy | Metadata |
|------|----------|----------|
| S01 Home | SSR | `title`, `description`, OG image |
| S02 Category | ISR 60s | dynamic `title` from category name |
| S03 PDP | ISR 60s | product name, description, OG image from product |
| S04 Search | SSR | `noindex` if empty query |
| Cart, checkout, account | CSR | `noindex` |

| Metric | Target |
|--------|--------|
| LCP (S01, S03) | < 2.5s on 4G |
| Images | `next/image`, sizes prop, CloudFront URL |
| Fonts | `next/font` — single family MVP |

---

## 8.13 FE Testing (MVP)

| Type | Tool | Coverage target |
|------|------|-----------------|
| Unit | Vitest | `formatVND`, Zod schemas, API error parser |
| Component | Vitest + Testing Library | ProductCard, PaymentMethodSelector, OrderTimeline |
| E2E (Week 12) | Playwright (optional) | COD checkout happy path |

### E2E critical path (if Playwright added)

1. Register → login  
2. Browse → add to cart  
3. Checkout COD → order confirmed  
4. Admin login → confirm order → shipped  

---

## 8.14 Definition of Done — Per Screen

A screen is **done** when all items below are checked:

- [ ] Matches UI elements in [03-shop](./03-shop-screen-specifications.md) or [04-admin](./04-admin-screen-specifications.md)
- [ ] All FE validation rules implemented (Zod + inline messages)
- [ ] Loading, empty, and error states handled
- [ ] Responsive at 375px (Shop) or 1280px (Admin)
- [ ] API wired to real backend (or MSW documented)
- [ ] Role guards applied (Admin screens)
- [ ] Toast/feedback on success and failure mutations
- [ ] No console errors in dev tools
- [ ] Peer review passed

---

## 8.15 Sprint Mapping (FE tasks)

| Sprint | Weeks | FE tasks |
|--------|-------|----------|
| 1 | 1–2 | FE-00 → FE-10, FE-S07 (auth UI), layout shells |
| 2 | 3–4 | FE-A01–A06 (admin catalog + inventory) |
| 3 | 5–6 | FE-S01–S05 (shop browse + cart) |
| 4 | 7–8 | FE-S06, FE-S09–S10, FE-S08 (checkout + orders) |
| 5 | 9–10 | FE-A07–A08, polish, responsive pass |
| 6 | 11–12 | E2E, a11y pass, bug fixes, production build |

---

## 8.16 Out of Scope (FE — MVP)

| Item | Phase |
|------|-------|
| Stripe / VNPay / MoMo UI | Phase 2 |
| i18n / Vietnamese | Phase 2 |
| PWA / offline mode | Phase 2 |
| Dark mode | Phase 2 |
| Storybook | Optional |
| Product variants UI | Phase 2 |
| Real-time SSE admin updates | Phase 2 (use manual refresh MVP) |

---

## 8.17 Quick Reference — Where to Look

| Question | Go to |
|----------|-------|
| What fields on checkout form? | [03 § S06](./03-shop-screen-specifications.md#s06--checkout) |
| What API does cart use? | [03 § S05](./03-shop-screen-specifications.md#s05--shopping-cart) or § 5.6 above |
| What components to build first? | Section 8.5 |
| How many days for product edit screen? | Section 8.7 → FE-A04 (5d) |
| Admin role hides categories? | Section 8.7 → Role-based UI rules |
