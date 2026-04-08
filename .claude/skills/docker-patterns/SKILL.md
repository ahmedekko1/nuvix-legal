---
name: docker-patterns
description: Docker and Docker Compose patterns for local development, container security, networking, volume strategies, and multi-service orchestration.
origin: ECC
---

# Docker Patterns

Docker and Docker Compose best practices for containerized development.

## Multi-Stage Dockerfile (Node.js)

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

FROM node:22-alpine AS dev
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

FROM node:22-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build && npm prune --production

FROM node:22-alpine AS production
WORKDIR /app
RUN addgroup -g 1001 -S appgroup && adduser -S appuser -u 1001
USER appuser
COPY --from=build --chown=appuser:appgroup /app/dist ./dist
COPY --from=build --chown=appuser:appgroup /app/node_modules ./node_modules
ENV NODE_ENV=production
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

## Docker Compose (Local Dev)

```yaml
services:
  app:
    build: { context: ., target: dev }
    ports: ["3000:3000"]
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/app_dev
    depends_on:
      db: { condition: service_healthy }

  db:
    image: postgres:16-alpine
    ports: ["5432:5432"]
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_dev
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes: [redisdata:/data]

volumes:
  pgdata:
  redisdata:
```

## Container Security

```yaml
services:
  app:
    security_opt: [no-new-privileges:true]
    read_only: true
    tmpfs: [/tmp]
    cap_drop: [ALL]
```

## .dockerignore

```
node_modules
.git
.env
.env.*
dist
coverage
*.log
.next
```

## Debug Commands

```bash
docker compose logs -f app             # Follow logs
docker compose exec app sh             # Shell into container
docker compose exec db psql -U postgres  # Postgres CLI
docker compose up --build              # Rebuild images
docker compose down -v                 # Stop + remove volumes (DESTRUCTIVE)
docker system prune                    # Remove unused images
```

## Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| `:latest` tags | Pin to specific versions |
| Running as root | Create non-root user |
| Secrets in Dockerfile `ENV` | Use runtime env vars or Docker secrets |
| One giant container | One process per container |
| Data without volumes | Containers are ephemeral |
