# Phase 5 — Frontend

> **Owner:** FE dev · **Depends on:** [backend.md](./backend.md) Keycloak + APISIX setup

---

## Keycloak OIDC migration

| Realm | Apps | Roles |
|-------|------|-------|
| `customers` | Shop (Next.js), Mobile (Expo) | Customer |
| `staff` | Admin (Next.js), POS | Admin, StoreManager, Cashier |

### Migration steps

1. Deploy Keycloak; configure realms (BE/DevOps — see [backend.md](./backend.md))
2. Shop: NextAuth OIDC provider → Keycloak
3. Admin: NextAuth OIDC provider → Keycloak
4. Mobile: Expo AuthSession PKCE → Keycloak
5. Remove custom JWT refresh logic; use Keycloak refresh tokens
6. Batch user import (existing Phase 1–4 users)

---

## FE tasks

| ID | Task | Est. | App |
|----|------|------|-----|
| P5-FE-01 | Shop NextAuth → Keycloak | 3d | Shop |
| P5-FE-02 | Admin NextAuth → Keycloak | 3d | Admin |
| P5-FE-03 | Mobile Expo OIDC → Keycloak | 5d | Mobile |
| P5-FE-04 | Role-based route guards update | 2d | Admin + POS |
| P5-FE-05 | APISIX CORS / redirect URI config support | 2d | All |
| P5-FE-06 | Smoke tests post-migration | 3d | All |

**Total:** ~18 dev-days

---

## Minimal UI changes

Phase 5 is primarily platform/infra. FE work is **auth migration only** — no new screens unless client requests franchise portal (out of scope).

### Validation

| Area | Rule |
|------|------|
| Login redirect | Keycloak hosted login page |
| Token refresh | Silent refresh via NextAuth / Expo secure store |
| POS session | Staff realm; Cashier role enforced |
| Logout | Keycloak end-session endpoint |

Base auth patterns → [docs-ba/05-frontend](../../docs-ba/05-frontend-specifications.md)
