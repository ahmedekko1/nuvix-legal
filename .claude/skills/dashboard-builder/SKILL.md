# Dashboard Builder Skill

Build data dashboards with metrics, charts, filters, and real-time updates.

## Dashboard Layout Structure

```tsx
// src/app/dashboard/page.tsx
export default function DashboardPage() {
  return (
    <DashboardLayout>
      <DashboardHeader title="Overview" dateRange={<DateRangePicker />} />
      <MetricsRow />
      <div className="grid grid-cols-1 gap-6 lg:grid-cols-2 mt-6">
        <RevenueChart />
        <ActivityChart />
      </div>
      <div className="mt-6">
        <RecentActivityTable />
      </div>
    </DashboardLayout>
  );
}
```

## Metric Cards

```tsx
interface MetricCardProps {
  label: string;
  value: string | number;
  change?: number;       // percentage change, positive = up
  prefix?: string;       // e.g. "$"
  suffix?: string;       // e.g. "%"
  isLoading?: boolean;
}

function MetricCard({ label, value, change, prefix, suffix, isLoading }: MetricCardProps) {
  if (isLoading) return <MetricCardSkeleton />;

  const isPositive = (change ?? 0) >= 0;

  return (
    <div className="rounded-lg bg-white border border-gray-200 p-6">
      <p className="text-sm font-medium text-gray-500">{label}</p>
      <p className="mt-2 text-3xl font-bold text-gray-900">
        {prefix}{typeof value === 'number' ? value.toLocaleString() : value}{suffix}
      </p>
      {change !== undefined && (
        <p className={`mt-1 text-sm font-medium ${isPositive ? 'text-green-600' : 'text-red-600'}`}>
          {isPositive ? '↑' : '↓'} {Math.abs(change)}% from last period
        </p>
      )}
    </div>
  );
}

function MetricsRow() {
  const { data, isLoading } = useMetrics();
  return (
    <div className="grid grid-cols-2 gap-4 sm:grid-cols-4">
      <MetricCard label="Total Revenue" value={data?.revenue ?? 0} prefix="$" change={data?.revenueChange} isLoading={isLoading} />
      <MetricCard label="Active Users" value={data?.activeUsers ?? 0} change={data?.userChange} isLoading={isLoading} />
      <MetricCard label="Calls Today" value={data?.callsToday ?? 0} isLoading={isLoading} />
      <MetricCard label="Avg. Response Time" value={data?.avgResponseMs ?? 0} suffix="ms" isLoading={isLoading} />
    </div>
  );
}
```

## Charts with Recharts

```tsx
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

function RevenueChart() {
  const { data, isLoading } = useRevenueTimeseries();

  if (isLoading) return <ChartSkeleton />;

  return (
    <div className="rounded-lg bg-white border border-gray-200 p-6">
      <h3 className="text-base font-semibold text-gray-900 mb-4">Revenue Over Time</h3>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={data}>
          <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
          <XAxis dataKey="date" tick={{ fontSize: 12, fill: '#6b7280' }} />
          <YAxis tick={{ fontSize: 12, fill: '#6b7280' }} tickFormatter={(v) => `$${v}`} />
          <Tooltip formatter={(value) => [`$${value}`, 'Revenue']} />
          <Line type="monotone" dataKey="revenue" stroke="#0ea5e9" strokeWidth={2} dot={false} />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}
```

## Date Range Filter

```tsx
'use client';
import { useState } from 'react';

type Range = '7d' | '30d' | '90d' | 'custom';

function DateRangePicker() {
  const [range, setRange] = useState<Range>('30d');

  const options: { label: string; value: Range }[] = [
    { label: 'Last 7 days', value: '7d' },
    { label: 'Last 30 days', value: '30d' },
    { label: 'Last 90 days', value: '90d' },
  ];

  return (
    <div className="flex gap-1 bg-gray-100 rounded-lg p-1">
      {options.map((opt) => (
        <button
          key={opt.value}
          onClick={() => setRange(opt.value)}
          className={`px-3 py-1.5 rounded-md text-sm font-medium transition-colors ${
            range === opt.value
              ? 'bg-white text-gray-900 shadow-sm'
              : 'text-gray-500 hover:text-gray-700'
          }`}
        >
          {opt.label}
        </button>
      ))}
    </div>
  );
}
```

## Data Fetching with SWR

```typescript
// src/hooks/useMetrics.ts
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

export function useMetrics(range = '30d') {
  const { data, error, isLoading } = useSWR(`/api/metrics?range=${range}`, fetcher, {
    refreshInterval: 60_000,   // refresh every minute
    revalidateOnFocus: true,
  });
  return { data, error, isLoading };
}
```

## API Route for Metrics

```typescript
// src/app/api/metrics/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';
import { getAuth } from '@/lib/auth';

export async function GET(req: NextRequest) {
  const session = await getAuth(req);
  if (!session) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });

  const range = req.nextUrl.searchParams.get('range') ?? '30d';
  const days = range === '7d' ? 7 : range === '90d' ? 90 : 30;
  const since = new Date(Date.now() - days * 86400_000);

  const [revenue, users, calls] = await Promise.all([
    db.payment.aggregate({ where: { createdAt: { gte: since }, orgId: session.orgId }, _sum: { amount: true } }),
    db.user.count({ where: { lastActiveAt: { gte: since }, orgId: session.orgId } }),
    db.call.count({ where: { createdAt: { gte: since }, orgId: session.orgId } }),
  ]);

  return NextResponse.json({
    revenue: revenue._sum.amount ?? 0,
    activeUsers: users,
    callsToday: calls,
  });
}
```

## Empty and Loading States

```tsx
function ChartSkeleton() {
  return <div className="h-[300px] rounded-lg bg-gray-100 animate-pulse" />;
}

function MetricCardSkeleton() {
  return (
    <div className="rounded-lg border border-gray-200 p-6 space-y-3">
      <div className="h-4 w-24 bg-gray-100 rounded animate-pulse" />
      <div className="h-8 w-32 bg-gray-100 rounded animate-pulse" />
    </div>
  );
}
```
