# BA Documentation — Phase 1 MVP

> **220,000,000 VND** · 1 FE + 1 BE · 3 months · 18 screens

---

## Documents (read in order)

| # | File | Who reads it | Contents |
|---|------|--------------|----------|
| **01** | [project-overview](./01-project-overview.md) | Client, everyone | Summary, stack, screens, budget, architecture |
| **02** | [scope-and-assumptions](./02-scope-and-assumptions.md) | Client, QA | In/out scope, rules, **UAT checklist** |
| **03** | [shop-screens](./03-shop-screen-specifications.md) | BA, FE | S01–S10: UI, flows, validation |
| **04** | [admin-screens](./04-admin-screen-specifications.md) | BA, FE | A01–A08: UI, flows, validation |
| **05** | [frontend-specs](./05-frontend-specifications.md) | **FE dev** | Components, tasks, routes, DoD |
| **06** | [backend-specs](./06-backend-specifications.md) | **BE dev** | APIs, data model, business rules |
| **07** | [devops](./07-devops-and-deployment.md) | BE, DevOps | AWS, CI/CD |
| **08** | [implementation-plan](./08-implementation-plan.md) | PM, all | 6 sprints, milestones |
| — | [mockups/index.html](./mockups/index.html) | Client, BA | Wireframes |

### Why this order?

```
01–02  Business context (client kickoff)
  ↓
03–04  What each screen does (BA + FE validation)
  ↓
05     How to build FE (right after screens — not last)
  ↓
06–07  Backend + infrastructure
  ↓
08     Timeline & delivery
```

> Old `08-frontend` was confusing — FE spec now **05**, directly after screen specs.

---

## Quick links

| Role | Start here |
|------|------------|
| Client | `01` → `02` → mockups |
| FE dev | `05` + `03` + `04` |
| BE dev | `06` + `07` |
| PM | `08` + `02` § 2.7 |

**Post-MVP:** [docs-phases/](../docs-phases/README.md) · [Pricing](../docs-phases/00-pricing-estimates.md)
