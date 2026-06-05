# 07 — DevOps & Deployment

> **Phase:** 1 MVP · **Budget share:** ~22M VND (10%)  
> **Target:** Staging + Production on AWS; Docker Compose local

---

## 6.1 Environment Overview

| Environment | Purpose | URL pattern |
|-------------|---------|-------------|
| **Local** | Developer machines | `localhost:3000` (shop), `localhost:3001` (admin), `localhost:8080` (API) |
| **Staging** | Client UAT, integration testing | `staging.shop.{domain}`, `staging.admin.{domain}`, `staging.api.{domain}` |
| **Production** | Live MVP launch | `shop.{domain}`, `admin.{domain}`, `api.{domain}` |

---

## 6.2 Infrastructure Diagram (MVP)

```
                    ┌──────────────┐
                    │  Route 53    │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │  CloudFront  │  (Shop + Admin static assets)
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ EC2 #1   │ │ EC2 #2   │ │   S3     │
        │ Shop     │ │ Admin    │ │  images  │
        │ Next.js  │ │ Next.js  │ │          │
        └────┬─────┘ └────┬─────┘ └──────────┘
             │            │
             └─────┬──────┘
                   ▼
            ┌──────────────┐
            │     ALB      │
            └──────┬───────┘
                   ▼
            ┌──────────────┐
            │   EC2 #3     │
            │ Quarkus API  │
            │  (Docker)    │
            └──────┬───────┘
                   │
     ┌─────────────┼─────────────┐
     ▼             ▼             ▼
┌─────────┐  ┌──────────┐  ┌─────────┐
│ RDS     │  │ Secrets  │  │  SES    │
│ Postgres│  │ Manager  │  │ (email) │
└─────────┘  └──────────┘  └─────────┘
```

> **Cost optimization:** Shop and Admin can share one EC2 (different ports + reverse proxy) in staging; split in production if load requires.

---

## 6.3 AWS Resource Sizing (MVP)

| Resource | Staging | Production | Est. monthly cost (USD) |
|----------|---------|------------|-------------------------|
| EC2 (app) | t3.small × 1 | t3.medium × 1–2 | $15–40 |
| RDS PostgreSQL | db.t3.micro | db.t3.small | $15–30 |
| S3 | < 50 GB | < 100 GB | $1–5 |
| CloudFront | Low traffic | Low traffic | $1–10 |
| SES | < 10k emails/mo | — | < $1 |
| Secrets Manager | 5 secrets | — | $2 |
| **Total estimate** | | | **~$35–90 USD (~1–2.2M VND/mo)** |

---

## 6.4 Docker Setup

### Local — `docker-compose.yml`

| Service | Image | Port |
|---------|-------|------|
| `postgres` | postgres:16-alpine | 5432 |
| `api` | Built from `Dockerfile` | 8080 |
| `mailhog` | mailhog/mailhog (dev email) | 8025 (UI) |

Shop and Admin run via `npm run dev` on host (hot reload).

### Production — API `Dockerfile` (multi-stage)

1. **Build stage:** Maven package Quarkus JVM jar  
2. **Runtime stage:** `eclipse-temurin:21-jre-alpine` + non-root user  
3. Health check: `GET /q/health/ready`

---

## 6.5 CI/CD Pipeline (GitHub Actions)

### On Pull Request

```yaml
jobs:
  backend:
    - checkout
    - setup-java 21
    - mvn verify (unit + integration tests)
    - openapi diff check (optional)
  frontend-shop:
    - npm ci && npm run lint && npm run build
  frontend-admin:
    - npm ci && npm run lint && npm run build
```

### On merge to `main`

```yaml
jobs:
  deploy-staging:
    - build & push API image to ECR
    - build Shop & Admin → upload to S3 / deploy to EC2
    - SSH or SSM → pull latest → docker compose up -d
    - smoke test: GET /q/health/ready
```

### Production deploy

- Manual approval gate (GitHub environment `production`)
- Tag release `v1.x.x`
- Database migration runs before app rollout
- Rollback: redeploy previous image tag (keep last 3 images in ECR)

---

## 6.6 Database Management

| Task | Approach |
|------|----------|
| Schema migrations | Flyway (`src/main/resources/db/migration`) |
| Seeding | `V999__seed_dev_data.sql` (categories, sample products, admin user) |
| Backups | RDS automated daily; 7-day retention |
| Restore test | Once before production launch |

### Initial admin seed

| Field | Value |
|-------|-------|
| Email | `admin@store.local` (replace at deploy) |
| Role | Admin |
| Password | Set via env var `ADMIN_INITIAL_PASSWORD` — force change on first login (Phase 2 nice-to-have) |

---

## 6.7 Secrets & Configuration

| Secret | Storage | Used by |
|--------|---------|---------|
| `DATABASE_URL` | Secrets Manager | API |
| `JWT_PRIVATE_KEY` / `JWT_PUBLIC_KEY` | Secrets Manager | API |
| `STRIPE_SECRET_KEY` | — | Removed from MVP |
| `STRIPE_WEBHOOK_SECRET` | — | Removed from MVP |
| `AWS_S3_BUCKET` | Env / SSM | API |
| `SES_FROM_EMAIL` | SSM | API |
| `NEXT_PUBLIC_API_URL` | Build-time env | Shop, Admin |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | — | Removed from MVP |

**Never commit secrets.** `.env.example` documents required keys.

---

## 6.8 Networking & Security

| Control | Implementation |
|---------|----------------|
| TLS | ACM certificate on ALB + CloudFront |
| Security groups | ALB → EC2 only; EC2 → RDS only on 5432 |
| RDS | Private subnet; no public access |
| WAF | AWS WAF basic rules (optional production) |
| Rate limiting | Nginx/ALB rate limit: 100 req/s per IP on `/v1/auth/*` |
| CORS | Whitelist shop + admin domains |
| Headers | `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options: DENY` on admin |

---

## 6.9 Monitoring & Alerting (MVP)

| Signal | Tool | Alert |
|--------|------|-------|
| App logs | CloudWatch Logs | — |
| API errors | CloudWatch metric filter (5xx count) | Email if > 10 in 5 min |
| RDS CPU | CloudWatch | Email if > 80% for 10 min |
| Disk | CloudWatch | Email if < 20% free |
| Uptime | UptimeRobot (free) or Route53 health check | Email on downtime |

---

## 6.10 Deployment Runbook (Summary)

### Deploy to staging

1. Merge PR to `main` → CI passes → auto-deploy staging  
2. Run Flyway migrations (automatic on API start)  
3. Smoke test: login admin → list products → shop COD checkout  
4. Notify client for UAT

### Deploy to production

1. Create release tag `v1.0.0`  
2. Approve production environment in GitHub Actions  
3. RDS snapshot before migration  
4. Deploy API → wait healthy → deploy frontends  
5. Verify bank account details in system settings  
6. Monitor CloudWatch for 30 minutes

### Rollback

1. Revert to previous ECR image tag on API EC2  
2. If migration failed: restore RDS snapshot (document RTO: 30 min)  
3. Revert frontend S3/EC2 to previous build artifact

---

## 6.11 DevOps Deliverables Checklist

- [ ] `docker-compose.yml` for local development
- [ ] `Dockerfile` for API (multi-stage)
- [ ] GitHub Actions workflows (PR + staging + production)
- [ ] Terraform or documented manual AWS setup script
- [ ] Flyway migrations for all schemas
- [ ] Secrets Manager entries documented
- [ ] `.env.example` for all apps
- [ ] Deploy & rollback runbook (this document)
- [ ] RDS backup verified
- [ ] HTTPS verified on all environments

---

## 6.12 Out of Scope (DevOps — Phase 2)

| Item | Notes |
|------|-------|
| Kubernetes (EKS) | Use when > 500 RPS or multi-region |
| Kafka / MSK | Event bus for microservices split |
| ArgoCD GitOps | When cluster-based |
| Multi-region DR | Cross-region RDS replica |
| Auto-scaling | CloudWatch + ASG when traffic grows |
| Penetration test | Recommended before public launch; client-owned |
