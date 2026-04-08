---
name: e2e-testing
description: Playwright E2E testing patterns, Page Object Model, configuration, CI/CD integration, artifact management, and flaky test strategies.
origin: ECC
---

# E2E Testing Patterns

Comprehensive Playwright patterns for building stable, fast, and maintainable E2E test suites.

## Page Object Model (POM)

```typescript
export class ItemsPage {
  readonly page: Page
  readonly searchInput: Locator
  readonly itemCards: Locator

  constructor(page: Page) {
    this.page = page
    this.searchInput = page.locator('[data-testid="search-input"]')
    this.itemCards = page.locator('[data-testid="item-card"]')
  }

  async goto() {
    await this.page.goto('/items')
    await this.page.waitForLoadState('networkidle')
  }

  async search(query: string) {
    await this.searchInput.fill(query)
    await this.page.waitForResponse(resp => resp.url().includes('/api/search'))
  }
}
```

## Test Structure

```typescript
test.describe('Item Search', () => {
  let itemsPage: ItemsPage

  test.beforeEach(async ({ page }) => {
    itemsPage = new ItemsPage(page)
    await itemsPage.goto()
  })

  test('should search by keyword', async ({ page }) => {
    await itemsPage.search('test')
    await expect(itemsPage.itemCards.first()).toContainText(/test/i)
  })
})
```

## Playwright Configuration

```typescript
export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10000,
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
  ],
})
```

## Flaky Test Handling

```typescript
// Quarantine flaky test
test('flaky: complex flow', async ({ page }) => {
  test.fixme(true, 'Flaky - Issue #123')
})

// Skip in CI
test('visual test', async ({ page }) => {
  test.skip(!!process.env.CI, 'Flaky in CI - Issue #456')
})
```

## Common Race Condition Fixes

```typescript
// BAD: arbitrary timeout
await page.waitForTimeout(5000)

// GOOD: wait for specific network event
await page.waitForResponse(resp => resp.url().includes('/api/data'))

// GOOD: wait for element stability before clicking
await page.locator('[data-testid="btn"]').waitFor({ state: 'visible' })
await page.waitForLoadState('networkidle')
await page.locator('[data-testid="btn"]').click()
```

## CI/CD Integration

```yaml
- run: npx playwright install --with-deps
- run: npx playwright test
  env:
    BASE_URL: ${{ vars.STAGING_URL }}
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: playwright-report
    path: playwright-report/
```

## Stability Standards

| Metric | Target |
|--------|--------|
| Critical journeys pass rate | 100% |
| Overall pass rate | >95% |
| Flaky test rate | <5% |
| Total duration | <10 minutes |
