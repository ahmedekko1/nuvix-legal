---
name: backend-patterns
description: Backend architecture patterns, API design, database optimization, and server-side best practices for Node.js, Express, and Next.js API routes.
origin: ECC
---

# Backend Development Patterns

Backend architecture patterns and best practices for scalable server-side applications.

## When to Activate

- Designing REST or GraphQL API endpoints
- Implementing repository, service, or controller layers
- Optimizing database queries (N+1, indexing, connection pooling)
- Adding caching (Redis, in-memory, HTTP cache headers)
- Setting up background jobs or async processing
- Structuring error handling and validation for APIs

## Repository Pattern

```typescript
interface MarketRepository {
  findAll(filters?: MarketFilters): Promise<Market[]>
  findById(id: string): Promise<Market | null>
  create(data: CreateMarketDto): Promise<Market>
  update(id: string, data: UpdateMarketDto): Promise<Market>
  delete(id: string): Promise<void>
}
```

## Service Layer Pattern

Business logic separated from data access. The service receives the repository via dependency injection.

## N+1 Query Prevention

```typescript
// BAD: N+1
const markets = await getMarkets()
for (const market of markets) {
  market.creator = await getUser(market.creator_id) // N queries
}

// GOOD: batch fetch
const creatorIds = markets.map(m => m.creator_id)
const creators = await getUsers(creatorIds) // 1 query
const creatorMap = new Map(creators.map(c => [c.id, c]))
markets.forEach(m => { m.creator = creatorMap.get(m.creator_id) })
```

## Caching (Redis)

```typescript
async findById(id: string): Promise<Market | null> {
  const cached = await this.redis.get(`market:${id}`)
  if (cached) return JSON.parse(cached)

  const market = await this.baseRepo.findById(id)
  if (market) await this.redis.setex(`market:${id}`, 300, JSON.stringify(market))
  return market
}
```

## Error Handling

```typescript
class ApiError extends Error {
  constructor(public statusCode: number, public message: string) {
    super(message)
  }
}

export function errorHandler(error: unknown): Response {
  if (error instanceof ApiError) {
    return NextResponse.json({ error: error.message }, { status: error.statusCode })
  }
  if (error instanceof z.ZodError) {
    return NextResponse.json({ error: 'Validation failed', details: error.errors }, { status: 400 })
  }
  console.error('Unexpected error:', error)
  return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
}
```

## Retry with Exponential Backoff

```typescript
async function fetchWithRetry<T>(fn: () => Promise<T>, maxRetries = 3): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try { return await fn() }
    catch (error) {
      if (i < maxRetries - 1) await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000))
      else throw error
    }
  }
  throw new Error('unreachable')
}
```

## Structured Logging

```typescript
const logger = {
  info: (msg: string, ctx?: object) => console.log(JSON.stringify({ level: 'info', msg, ...ctx, ts: new Date().toISOString() })),
  error: (msg: string, err: Error, ctx?: object) => console.log(JSON.stringify({ level: 'error', msg, error: err.message, stack: err.stack, ...ctx, ts: new Date().toISOString() })),
}
```
