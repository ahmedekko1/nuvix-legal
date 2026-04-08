# Frontend Design Skill

UI design principles, component architecture, and visual consistency patterns for React/Next.js applications.

## Design Tokens

Define tokens once, use everywhere. Store in `src/styles/tokens.ts` or `tailwind.config.ts`:

```typescript
// tailwind.config.ts
export default {
  theme: {
    extend: {
      colors: {
        brand: {
          50:  '#f0f9ff',
          500: '#0ea5e9',
          900: '#0c4a6e',
        },
        surface: {
          DEFAULT: '#ffffff',
          muted: '#f8fafc',
          raised: '#ffffff',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      borderRadius: {
        DEFAULT: '0.5rem',
        lg: '0.75rem',
        xl: '1rem',
      },
    },
  },
};
```

Never use raw hex values in components — always reference tokens.

## Component Composition

Prefer small, composable components over large monoliths:

```tsx
// Bad: one big component
function Dashboard() {
  return (
    <div>
      {/* 300 lines of mixed concerns */}
    </div>
  );
}

// Good: composed from focused pieces
function Dashboard() {
  return (
    <DashboardLayout>
      <DashboardHeader />
      <MetricsGrid />
      <ActivityFeed />
    </DashboardLayout>
  );
}
```

## Spacing System

Use consistent spacing scale — don't invent values:

```
4px  → p-1, gap-1
8px  → p-2, gap-2
12px → p-3, gap-3
16px → p-4, gap-4
24px → p-6, gap-6
32px → p-8, gap-8
48px → p-12
64px → p-16
```

## Typography Scale

```tsx
// Headings
<h1 className="text-3xl font-bold tracking-tight">Page Title</h1>
<h2 className="text-2xl font-semibold">Section Title</h2>
<h3 className="text-lg font-medium">Card Title</h3>

// Body
<p className="text-base text-gray-700 leading-relaxed">Body text</p>
<p className="text-sm text-gray-500">Secondary / helper text</p>
<span className="text-xs text-gray-400">Metadata / timestamps</span>
```

## Layout Patterns

**Page layout:**
```tsx
function PageLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen bg-surface-muted">
      <Navbar />
      <main className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 py-8">
        {children}
      </main>
    </div>
  );
}
```

**Card:**
```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="rounded-lg bg-white border border-gray-200 shadow-sm p-6">
      <h3 className="text-lg font-medium text-gray-900 mb-4">{title}</h3>
      {children}
    </div>
  );
}
```

**Two-column form:**
```tsx
<div className="grid grid-cols-1 gap-6 sm:grid-cols-2">
  <FormField label="First name" name="firstName" />
  <FormField label="Last name" name="lastName" />
</div>
```

## Button Hierarchy

```tsx
// Primary — one per page section
<button className="bg-brand-500 text-white px-4 py-2 rounded font-medium hover:bg-brand-600">
  Save changes
</button>

// Secondary
<button className="border border-gray-300 text-gray-700 px-4 py-2 rounded font-medium hover:bg-gray-50">
  Cancel
</button>

// Destructive
<button className="bg-red-600 text-white px-4 py-2 rounded font-medium hover:bg-red-700">
  Delete
</button>

// Ghost / text
<button className="text-brand-500 font-medium hover:text-brand-700">
  Learn more →
</button>
```

## Loading & Empty States

Always handle all states — never leave a blank screen:

```tsx
function DataTable({ data, isLoading, error }) {
  if (isLoading) return <TableSkeleton rows={5} />;
  if (error)     return <ErrorState message={error.message} onRetry={refetch} />;
  if (!data?.length) return <EmptyState message="No records yet" action={<CreateButton />} />;
  return <Table data={data} />;
}
```

## Accessibility Checklist

- [ ] All interactive elements reachable by keyboard (`Tab`, `Enter`, `Space`)
- [ ] Focus rings visible (`focus:ring-2 focus:ring-brand-500`)
- [ ] Color contrast ≥ 4.5:1 for text
- [ ] Images have `alt` text
- [ ] Form inputs have `<label>` associations
- [ ] Modals trap focus and close on `Escape`
- [ ] `aria-label` on icon-only buttons

## Responsive Breakpoints

```
sm:  640px  — large phone / small tablet
md:  768px  — tablet
lg:  1024px — laptop
xl:  1280px — desktop
2xl: 1536px — wide screen
```

Mobile-first: write base styles for mobile, override at larger breakpoints.
