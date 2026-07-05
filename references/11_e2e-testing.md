# 11 — E2E Testing: Playwright & Cypress

---

## Tool Selection

| Criterion | Playwright | Cypress |
|-----------|-----------|---------|
| Multi-browser | Chrome, Firefox, Safari, Edge | Chrome, Firefox, Edge (Safari limited) |
| Multi-tab / iframes | ✅ Native | ⚠️ Limited |
| Mobile emulation | ✅ Built-in device profiles | ⚠️ Viewport only |
| API mocking | ✅ `page.route()` | ✅ `cy.intercept()` |
| Auth state persistence | ✅ `storageState` | ✅ `cy.session()` |
| CI speed | Fast (parallel workers) | Moderate |
| TypeScript | ✅ First-class | ✅ Good |
| **Recommendation** | **Prefer for new projects** | **Good for existing Cypress teams** |

---

## Playwright

### Setup

```bash
npm init playwright@latest
# Select: TypeScript, tests/ folder, GitHub Actions workflow
```

### Project Structure

```
tests/
├── auth/
│   ├── login.spec.ts
│   ├── register.spec.ts
│   └── password-reset.spec.ts
├── [feature]/
│   ├── [feature].spec.ts
│   └── [feature].page.ts       ← Page Object
├── api/
│   └── [endpoint].spec.ts
└── fixtures/
    ├── auth.ts                  ← Authenticated user fixture
    └── test-data.ts
playwright.config.ts
```

### Page Object Model (required for maintainability)

```typescript
// tests/auth/login.page.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

### Test File Pattern

```typescript
// tests/auth/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from './login.page';

test.describe('Login', () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });

  test('TC-001 — valid credentials → redirects to dashboard', async ({ page }) => {
    await loginPage.login(process.env.TEST_USER_EMAIL!, process.env.TEST_USER_PASSWORD!);
    await expect(page).toHaveURL('/dashboard');
  });

  test('TC-002 — invalid password → shows error', async ({ page }) => {
    await loginPage.login('valid@example.com', 'wrongpassword');
    await expect(loginPage.errorMessage).toBeVisible();
    await expect(loginPage.errorMessage).toContainText('Invalid');
  });

  test('TC-003 — empty fields → validation errors', async ({ page }) => {
    await loginPage.submitButton.click();
    await expect(page.getByText('Email is required')).toBeVisible();
  });

  test('TC-004 — RTL layout → correct rendering', async ({ page }) => {
    await page.emulateMedia({ reducedMotion: 'reduce' });
    // Switch to Arabic locale
    await page.goto('/login?lang=ar');
    const htmlDir = await page.locator('html').getAttribute('dir');
    expect(htmlDir).toBe('rtl');
  });
});
```

### Auth Fixture (reuse across all tests)

```typescript
// tests/fixtures/auth.ts
import { test as base, expect } from '@playwright/test';

type AuthFixtures = {
  authenticatedPage: Page;
};

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: 'tests/.auth/user.json',
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },
});

// Generate auth state once (run before test suite):
// npx playwright test tests/fixtures/setup.ts --project=setup
```

### API Mocking

```typescript
test('dashboard shows correct data when API returns empty', async ({ page }) => {
  // Intercept before navigation
  await page.route('**/api/v1/orders', route => {
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({ data: [], total: 0 }),
    });
  });

  await page.goto('/dashboard');
  await expect(page.getByText('No orders yet')).toBeVisible();
  await expect(page.getByRole('button', { name: 'Create first order' })).toBeVisible();
});
```

### playwright.config.ts (production-ready)

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  reporter: [
    ['html'],
    ['github'],                      // GitHub Actions annotations
    ['json', { outputFile: 'playwright-report/results.json' }],
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 7'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 15'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## Cypress

### Setup

```bash
npm install --save-dev cypress
npx cypress open   # interactive
npx cypress run    # headless CI
```

### Project Structure

```
cypress/
├── e2e/
│   ├── auth/
│   │   └── login.cy.ts
│   └── [feature]/
│       └── [feature].cy.ts
├── support/
│   ├── commands.ts         ← Custom commands
│   └── e2e.ts
├── fixtures/
│   └── user.json
└── pages/                  ← Page Objects
    └── login.page.ts
cypress.config.ts
```

### Custom Commands Pattern

```typescript
// cypress/support/commands.ts
Cypress.Commands.add('login', (email: string, password: string) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-testid="email"]').type(email);
    cy.get('[data-testid="password"]').type(password);
    cy.get('[data-testid="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});

Cypress.Commands.add('loginAsAdmin', () => {
  cy.login(Cypress.env('ADMIN_EMAIL'), Cypress.env('ADMIN_PASSWORD'));
});
```

### Test File Pattern

```typescript
// cypress/e2e/auth/login.cy.ts
describe('Login', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('TC-001 — valid credentials → dashboard', () => {
    cy.get('[data-testid="email"]').type(Cypress.env('TEST_EMAIL'));
    cy.get('[data-testid="password"]').type(Cypress.env('TEST_PASSWORD'));
    cy.get('[data-testid="submit"]').click();
    cy.url().should('include', '/dashboard');
  });

  it('TC-002 — wrong password → error visible', () => {
    cy.get('[data-testid="email"]').type('valid@example.com');
    cy.get('[data-testid="password"]').type('wrongpassword');
    cy.get('[data-testid="submit"]').click();
    cy.get('[role="alert"]').should('be.visible');
  });

  it('TC-003 — API error → graceful fallback', () => {
    cy.intercept('POST', '/api/auth/login', {
      statusCode: 500,
      body: { error: 'Internal server error' },
    }).as('loginRequest');

    cy.get('[data-testid="submit"]').click();
    cy.wait('@loginRequest');
    cy.get('[data-testid="error-banner"]').should('contain', 'Something went wrong');
  });
});
```

---

## E2E Test Checklist (run before every release)

### Authentication Flows
- [ ] Login with valid credentials → correct redirect
- [ ] Login with invalid credentials → error message (no credential leak)
- [ ] Password reset end-to-end (email → link → new password → login)
- [ ] Token expiry → redirect to login
- [ ] Logout → session cleared → protected route inaccessible

### Core User Flows
- [ ] Happy path — main job-to-be-done completes end-to-end
- [ ] Error path — API failure → graceful UI state
- [ ] Empty state — new account, no data
- [ ] Form validation — required fields, max length, special chars, Arabic input
- [ ] Pagination / infinite scroll — loads next page without duplicate data

### Cross-Browser
- [ ] Chrome (latest) — full suite
- [ ] Safari (latest) — auth + core flows
- [ ] Firefox (latest) — core flows
- [ ] Mobile Chrome (Pixel 7 emulation) — full suite
- [ ] Mobile Safari (iPhone 15 emulation) — full suite

### RTL / Arabic
- [ ] `dir="rtl"` applied on Arabic locale switch
- [ ] Layout mirrors without overflow
- [ ] Arabic text renders correctly (not fallback font)
- [ ] Form fields accept and display Arabic input

---

## GitHub Actions CI

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium firefox webkit

      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: ${{ secrets.STAGING_URL }}
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7
```

---

## E2E Output Format

```markdown
## E2E Test Suite — [Feature / Release]
**Tool:** Playwright / Cypress  
**Reviewer:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  
**Browser coverage:** Chrome · Firefox · Safari · Mobile Chrome · Mobile Safari  

### Results
| Suite | Tests | Passed | Failed | Skipped |
|-------|-------|--------|--------|---------|
| Auth | XX | XX | XX | XX |
| [Feature] | XX | XX | XX | XX |
| **Total** | **XX** | **XX** | **XX** | **XX** |

### Failures
#### ❌ [Test name] — [Browser]
- **Error:** [exact error message]
- **Screenshot:** [link]
- **Trace:** [link]
- **Linked bug:** BUG-[XXX]

### Verdict
🟢 PASS — ready for release gate / 🔴 FAIL — [X] blocking failures
```
