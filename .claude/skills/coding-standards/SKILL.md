---
name: coding-standards
description: Baseline cross-project coding conventions for naming, readability, immutability, and code-quality review.
origin: ECC
---

# Coding Standards & Best Practices

Baseline coding conventions applicable across projects.

## Core Principles

1. **Readability First** — code is read more than written
2. **KISS** — simplest solution that works; avoid over-engineering
3. **DRY** — extract common logic; avoid copy-paste
4. **YAGNI** — don't build features before needed

## Naming

```typescript
// GOOD
const marketSearchQuery = 'election'
const isUserAuthenticated = true
async function fetchMarketData(marketId: string) {}
function isValidEmail(email: string): boolean {}

// BAD
const q = 'election'
const flag = true
async function market(id: string) {}
```

## Immutability (CRITICAL)

```typescript
// ALWAYS use spread
const updatedUser = { ...user, name: 'New Name' }
const updatedArray = [...items, newItem]

// NEVER mutate directly
user.name = 'New Name'  // BAD
items.push(newItem)     // BAD
```

## Error Handling

```typescript
// GOOD
async function fetchData(url: string) {
  try {
    const response = await fetch(url)
    if (!response.ok) throw new Error(`HTTP ${response.status}`)
    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}
```

## Async Best Practices

```typescript
// GOOD: parallel
const [users, markets] = await Promise.all([fetchUsers(), fetchMarkets()])

// BAD: unnecessary sequential
const users = await fetchUsers()
const markets = await fetchMarkets()
```

## Code Smells

**Long functions (>50 lines):** Extract into smaller named functions.

**Deep nesting (>4 levels):** Use early returns.
```typescript
// BAD: nested ifs
if (user) { if (user.isAdmin) { if (market) { ... } } }

// GOOD: early returns
if (!user) return
if (!user.isAdmin) return
if (!market) return
// Do something
```

**Magic numbers:**
```typescript
const MAX_RETRIES = 3         // GOOD
if (retryCount > 3) {}        // BAD
```

**Type safety:**
```typescript
// BAD
function getMarket(id: any): Promise<any>

// GOOD
function getMarket(id: string): Promise<Market>
```

## File Naming

```
components/Button.tsx     # PascalCase for components
hooks/useAuth.ts          # camelCase + 'use' prefix
lib/formatDate.ts         # camelCase for utilities
types/market.types.ts     # .types suffix
```

## Comments

```typescript
// GOOD: explain WHY
// Exponential backoff to avoid overwhelming API during outages
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// BAD: state the obvious
count++ // increment counter
```
