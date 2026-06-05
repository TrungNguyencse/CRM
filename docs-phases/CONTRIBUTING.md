# Contributing to Phase Docs

Each phase folder uses **3 owner files** so FE and BE can update in parallel without merge conflicts.

## File ownership

| File | Owner | Edit when… |
|------|-------|------------|
| `general.md` | **PM / either dev** | Scope, sprint plan, acceptance, pricing notes change |
| `frontend.md` | **FE dev** | Screens, UI, FE tasks, components, mobile (Phase 4) |
| `backend.md` | **BE dev** | APIs, modules, BE tasks, infra, Temporal, events |

`README.md` is a thin index — update only when adding files or changing phase summary.

## Rules

1. **Do not** put FE tasks in `backend.md` or API specs in `frontend.md`.
2. **Cross-references:** link to the other file instead of duplicating content.
3. **Acceptance criteria** live in `general.md` only.
4. **Pricing** stays in [00-pricing-estimates.md](./00-pricing-estimates.md) — phase README links to it.
5. Phase 1 MVP detail lives in [docs-ba/](../docs-ba/README.md) — do not duplicate full specs in `docs-phases/phase-1-mvp/`.

## Suggested git workflow

```bash
# FE updating Phase 2
git checkout -b docs/p2-frontend
# edit only phase-2-commerce-payments/frontend.md

# BE updating Phase 2
git checkout -b docs/p2-backend
# edit only phase-2-commerce-payments/backend.md
```

## Phase 1 (MVP)

Use [docs-ba/05](../docs-ba/05-frontend-specifications.md) and [docs-ba/06](../docs-ba/06-backend-specifications.md) as the live specs. `phase-1-mvp/` only has pointers.
