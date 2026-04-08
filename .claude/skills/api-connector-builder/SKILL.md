# API Connector Builder Skill

Build clean, reusable API connectors that integrate third-party services into the codebase following repo-native patterns.

## Discovery Phase

Before writing a connector, scan the codebase:

```bash
# Find existing HTTP clients or API patterns
grep -r "axios\|fetch\|httpx\|requests\|got" src/ --include="*.ts" -l
grep -r "class.*Client\|class.*Api\|class.*Service" src/ --include="*.ts" -l

# Find existing connector patterns to match
ls src/lib/integrations/ src/services/ src/connectors/ 2>/dev/null
```

Match the style of existing connectors — don't introduce new patterns when conventions already exist.

## Connector Structure (TypeScript)

```typescript
// src/lib/integrations/stripe/client.ts

import axios, { AxiosInstance } from 'axios';

interface StripeConfig {
  apiKey: string;
  baseURL?: string;
  timeout?: number;
}

export class StripeClient {
  private readonly http: AxiosInstance;

  constructor(config: StripeConfig) {
    this.http = axios.create({
      baseURL: config.baseURL ?? 'https://api.stripe.com/v1',
      timeout: config.timeout ?? 10_000,
      headers: {
        Authorization: `Bearer ${config.apiKey}`,
        'Content-Type': 'application/x-www-form-urlencoded',
      },
    });

    this.http.interceptors.response.use(
      (res) => res,
      (err) => { throw this.normalizeError(err); }
    );
  }

  async createCustomer(email: string, name: string) {
    const { data } = await this.http.post('/customers', { email, name });
    return data;
  }

  async getCustomer(id: string) {
    const { data } = await this.http.get(`/customers/${id}`);
    return data;
  }

  private normalizeError(err: unknown): Error {
    if (axios.isAxiosError(err) && err.response) {
      const { status, data } = err.response;
      return Object.assign(new Error(data?.error?.message ?? 'API error'), { status, data });
    }
    return err instanceof Error ? err : new Error(String(err));
  }
}
```

## Environment Config Pattern

```typescript
// src/lib/integrations/stripe/index.ts
import { StripeClient } from './client';

export const stripe = new StripeClient({
  apiKey: process.env.STRIPE_SECRET_KEY!,
});
```

Always read credentials from `process.env` — never hardcode or pass from calling code.

## Python Connector Pattern

```python
# tools/integrations/stripe_client.py
import os
import requests
from typing import Any

class StripeClient:
    BASE_URL = "https://api.stripe.com/v1"

    def __init__(self):
        self.api_key = os.environ["STRIPE_SECRET_KEY"]
        self.session = requests.Session()
        self.session.auth = (self.api_key, "")

    def create_customer(self, email: str, name: str) -> dict[str, Any]:
        resp = self.session.post(f"{self.BASE_URL}/customers", data={"email": email, "name": name})
        resp.raise_for_status()
        return resp.json()

    def get_customer(self, customer_id: str) -> dict[str, Any]:
        resp = self.session.get(f"{self.BASE_URL}/customers/{customer_id}")
        resp.raise_for_status()
        return resp.json()
```

## Rate Limiting & Retry

```typescript
import pRetry from 'p-retry';

async function withRetry<T>(fn: () => Promise<T>): Promise<T> {
  return pRetry(fn, {
    retries: 3,
    onFailedAttempt: (err) => {
      if (err.response?.status === 429) {
        const retryAfter = err.response.headers['retry-after'];
        // respect rate limit header
      }
    },
  });
}
```

## Checklist

Before finalizing a connector:

- [ ] Credentials come from env vars only
- [ ] Base URL is configurable (for test/sandbox environments)
- [ ] Errors are normalized to a consistent shape
- [ ] Timeout is set explicitly (never rely on defaults)
- [ ] Retry logic handles 429 and 5xx
- [ ] Interface is typed (TypeScript) or annotated (Python)
- [ ] Singleton export matches repo patterns
- [ ] `.env.example` updated with new variable names

## .env.example Update

Always add new variables to `.env.example`:

```bash
# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```
