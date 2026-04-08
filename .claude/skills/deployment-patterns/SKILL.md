---
name: deployment-patterns
description: Deployment workflows, CI/CD pipeline patterns, Docker containerization, health checks, rollback strategies, and production readiness checklists for web applications.
origin: ECC
---

# Deployment Patterns

Production deployment workflows and CI/CD best practices.

## Deployment Strategies

### Rolling (Default)
Replace instances gradually — old and new versions run simultaneously.
- **Use when**: Standard deployments, backward-compatible changes
- **Requires**: Backward-compatible DB changes

### Blue-Green
Run two identical environments. Switch traffic atomically.
- **Use when**: Critical services, instant rollback needed
- **Cost**: 2x infrastructure during deployment

### Canary
Route a small percentage of traffic to the new version first (5% → 50% → 100%).
- **Use when**: High-traffic services, risky changes
- **Requires**: Traffic splitting infrastructure and monitoring

## Health Check Endpoint

```typescript
app.get("/health", (req, res) => res.status(200).json({ status: "ok" }))

app.get("/health/detailed", async (req, res) => {
  const checks = { database: await checkDatabase(), redis: await checkRedis() }
  const healthy = Object.values(checks).every(c => c.status === "ok")
  res.status(healthy ? 200 : 503).json({ status: healthy ? "ok" : "degraded", checks })
})
```

## GitHub Actions Pipeline

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci && npm run lint && npm run typecheck && npm test

  build:
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

  deploy:
    needs: build
    environment: production
    steps:
      - run: echo "Deploy ${{ github.sha }}"
```

## Environment Config Validation

```typescript
import { z } from "zod"

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "staging", "production"]),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
})

export const env = envSchema.parse(process.env) // Fail fast at startup
```

## Rollback

```bash
kubectl rollout undo deployment/app          # Kubernetes
vercel rollback                              # Vercel
railway up --commit <previous-sha>          # Railway
npx prisma migrate resolve --rolled-back <name>  # DB migration
```

## Production Readiness Checklist

- [ ] All tests pass (unit, integration, E2E)
- [ ] No hardcoded secrets
- [ ] Structured JSON logging (no PII in logs)
- [ ] Health check endpoint operational
- [ ] Docker image builds reproducibly with pinned versions
- [ ] SSL/TLS enabled on all endpoints
- [ ] Rate limiting on public endpoints
- [ ] Dependencies scanned for CVEs (`npm audit`)
- [ ] Rollback plan documented and tested
- [ ] Database migrations tested against production-sized data
- [ ] Monitoring and alerting configured
