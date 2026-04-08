---
name: frontend-patterns
description: Frontend development patterns for React, Next.js, state management, performance optimization, and UI best practices.
origin: ECC
---

# Frontend Development Patterns

Modern frontend patterns for React, Next.js, and performant user interfaces.

## When to Activate

- Building React components (composition, props, rendering)
- Managing state (useState, useReducer, Zustand, Context)
- Implementing data fetching (SWR, React Query, server components)
- Optimizing performance (memoization, virtualization, code splitting)
- Working with forms (validation, controlled inputs, Zod schemas)
- Building accessible, responsive UI patterns

## Component Patterns

### Composition Over Inheritance

```typescript
interface CardProps { children: React.ReactNode; variant?: 'default' | 'outlined' }

export function Card({ children, variant = 'default' }: CardProps) {
  return <div className={`card card-${variant}`}>{children}</div>
}
export function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>
}
```

### Custom Hooks

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)
  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])
  return debouncedValue
}

export function useToggle(initialValue = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue)
  const toggle = useCallback(() => setValue(v => !v), [])
  return [value, toggle]
}
```

### Async Data Fetching Hook

```typescript
export function useQuery<T>(key: string, fetcher: () => Promise<T>, options?: { enabled?: boolean }) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Error | null>(null)
  const [loading, setLoading] = useState(false)

  const refetch = useCallback(async () => {
    setLoading(true)
    try { setData(await fetcher()) }
    catch (err) { setError(err as Error) }
    finally { setLoading(false) }
  }, [fetcher])

  useEffect(() => { if (options?.enabled !== false) refetch() }, [key])
  return { data, error, loading, refetch }
}
```

## State Management

### Context + Reducer

```typescript
type Action = { type: 'SET_DATA'; payload: any } | { type: 'SET_LOADING'; payload: boolean }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_DATA': return { ...state, data: action.payload }
    case 'SET_LOADING': return { ...state, loading: action.payload }
    default: return state
  }
}
```

## Performance Optimization

```typescript
// Memoize expensive computations
const sortedItems = useMemo(() => items.sort((a, b) => b.score - a.score), [items])

// Memoize callbacks
const handleSearch = useCallback((q: string) => setQuery(q), [])

// Memoize pure components
export const ItemCard = React.memo<ItemCardProps>(({ item }) => <div>{item.name}</div>)
```

### Code Splitting

```typescript
const HeavyChart = lazy(() => import('./HeavyChart'))

<Suspense fallback={<Spinner />}>
  <HeavyChart data={data} />
</Suspense>
```

### Virtualization for Long Lists

```typescript
import { useVirtualizer } from '@tanstack/react-virtual'

const virtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 100,
  overscan: 5,
})
```

## Error Boundary

```typescript
export class ErrorBoundary extends React.Component<{ children: React.ReactNode }, { hasError: boolean }> {
  state = { hasError: false }
  static getDerivedStateFromError() { return { hasError: true } }
  componentDidCatch(error: Error, info: React.ErrorInfo) { console.error(error, info) }
  render() {
    if (this.state.hasError) return <div>Something went wrong. <button onClick={() => this.setState({ hasError: false })}>Retry</button></div>
    return this.props.children
  }
}
```

## Accessibility

- `role="combobox"`, `aria-expanded`, `aria-haspopup` for custom dropdowns
- Keyboard navigation: ArrowUp/Down, Enter to select, Escape to close
- Focus management in modals: save previous focus, restore on close
- All interactive elements have accessible labels
