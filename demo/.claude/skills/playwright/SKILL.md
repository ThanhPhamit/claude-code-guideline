---
name: playwright
description: Write, extend, or debug Playwright tests for this project. Use when adding new page objects, fixtures, test specs, auth setups, or data-driven test cases.
---

# Playwright Testing — Project Skill

## Project Architecture

```
src/
  auth/
    storage-state.ts       ← auth state paths + helpers
  data/
    auth.data.ts           ← UserCredentials + role definitions
  data-driven/
    schema.ts              ← DataDrivenCase type + normalization
    loaders.ts             ← loadJsonCases / loadCsvCases
  fixtures/
    data-driven.fixture.ts ← registerCase fixture (attaches screenshot)
    patient.fixture.ts     ← patient page fixtures (extends data-driven)
    doctor.fixture.ts      ← doctor page fixtures (extends data-driven)
  pages/
    base.page.ts           ← abstract BasePage
    patient/
      login.page.ts
      profile.page.ts
      profile-info.page.ts
    doctor/
      ...
  reporters/
    aggregated-results.reporter.ts

config/
  env.ts                   ← typed env vars (BASE_URL, PATIENT_URL, etc.)
  projects.ts              ← Playwright projects per role + browser

tests/
  auth/
    patient.setup.ts       ← save storage state for patient role
    doctor.setup.ts
  smoke/
    patient/
      profile-info.spec.ts ← data-driven spec example
  
test-data/
  cases/
    patient/
      *.cases.json         ← test case JSON files
```

---

## 1. Page Object Model (POM)

### Always extend BasePage

```typescript
// src/pages/base.page.ts
import type { Page } from '@playwright/test';

export abstract class BasePage {
  protected constructor(protected readonly page: Page) {}
}
```

### Page object rules

- **Private Locator getters** — never expose locators publicly; encapsulate selectors
- **`goto()` asserts readiness** — always `expect(...).toBeVisible()` at end of goto
- **`expect`** imported from `@playwright/test`, not custom
- Navigate + click together with `Promise.all` to avoid race conditions

```typescript
// src/pages/patient/example.page.ts
import { expect, type Locator, type Page } from '@playwright/test';
import { BasePage } from '../base.page';

export class ExamplePage extends BasePage {
  constructor(page: Page) {
    super(page);
  }

  // PRIVATE locator getters — one per interactive element
  private get submitButton(): Locator {
    return this.page.getByRole('button', { name: 'ログイン' });
  }

  private get emailInput(): Locator {
    return this.page.locator("input[type='email']");
  }

  // goto: navigate + assert page is ready
  async goto(): Promise<void> {
    await this.page.goto('/some-path');
    await expect(this.emailInput).toBeVisible();
  }

  // Actions return Promise<void>
  async submit(credentials: { username: string; password: string }): Promise<void> {
    await this.emailInput.fill(credentials.username);

    // Navigation + click together to avoid race
    await Promise.all([
      this.page.waitForURL(/\/patient\/dashboard(?:\?.*)?$/, { timeout: 15_000 }),
      this.submitButton.click(),
    ]);
  }

  // Assertions inside page object
  async expectError(message: string): Promise<void> {
    await expect(this.page.locator('[data-test="error"]')).toContainText(message);
  }
}
```

---

## 2. Fixture Composition

Fixtures compose in a chain:  
`@playwright/test` → `data-driven.fixture` → `role.fixture` → spec file

### The registerCase fixture

Every data-driven test MUST call `registerCase(testCase)` — it attaches annotations and captures a full-page screenshot on completion.

```typescript
// Usage in spec
test(`[${testCase.caseId}]`, async ({ profileInfoPage, registerCase }) => {
  registerCase(testCase);  // ← call at the START of every data-driven test
  await profileInfoPage.goto();
  // ...
});
```

### Adding a new fixture

```typescript
// src/fixtures/patient.fixture.ts
import { test as base, expect } from './data-driven.fixture';
import { NewPage } from '../pages/patient/new.page';

type PatientFixtures = {
  loginPage: LoginPage;
  newPage: NewPage;        // ← add here
};

export const test = base.extend<PatientFixtures>({
  // existing fixtures...
  newPage: async ({ page }, use) => {
    await use(new NewPage(page));
  },
});

export { expect };
```

---

## 3. Data-Driven Test Cases

### Case schema (all fields required)

```typescript
type DataDrivenCase = {
  readonly caseId: string;       // e.g. "TC-001"
  readonly title: string;        // Human-readable description
  readonly module: string;       // e.g. "Patient/Profile-Info"
  readonly expected: string;     // Pass/Fail expectation description
  readonly severity: 'low' | 'medium' | 'high' | 'critical';
  readonly steps: readonly string[];  // step descriptions for reporting
  readonly payload: Record<string, string>;  // dynamic data for the test
  // internal: sourceFile, sourceFormat added by loaders
};
```

### JSON case file format

```json
[
  {
    "caseId": "TC-001",
    "title": "Update first name with valid value",
    "module": "Patient/Profile-Info",
    "expected": "Pass",
    "severity": "medium",
    "steps": ["Navigate to profile-info", "Update first name", "Verify updated"],
    "firstName": "田中",
    "lastName": "太郎"
  }
]
```

Non-core fields (`firstName`, `lastName`, etc.) are automatically extracted into `payload`.

### Loading cases in a spec

```typescript
import { loadCaseCollection } from '../../../src/data-driven/loaders';

const cases = loadCaseCollection('test-data/cases/patient/my-feature.cases.json');

test.describe('Smoke|Patient|MyFeature', () => {
  for (const testCase of cases) {
    test(`[${testCase.caseId}]`, async ({ myPage, registerCase }) => {
      registerCase(testCase);
      // use testCase.payload for dynamic values
    });
  }
});
```

---

## 4. Auth Setup (Storage State)

Every role gets its own `tests/auth/<role>.setup.ts`. It logs in manually and saves storage state.

```typescript
// tests/auth/new-role.setup.ts
import { test as setup } from '@playwright/test';
import { ensureAuthStateDirectory, getStorageStatePath } from '../../src/auth/storage-state';
import { resolveUserCredentials } from '../../src/data/auth.data';
import { LoginPage } from '../../src/pages/new-role/login.page';
import { DashboardPage } from '../../src/pages/new-role/dashboard.page';
import { env } from '../../config/env';

setup('Auth|NewRole', async ({ page }) => {
  const loginPage = new LoginPage(page);
  const dashboardPage = new DashboardPage(page);
  const credentials = resolveUserCredentials(
    { username: env.NEW_ROLE_USERNAME, password: env.NEW_ROLE_PASSWORD },
    'new-role',
  );

  await loginPage.goto();
  await loginPage.login(credentials);
  await dashboardPage.expectLoaded();
  await ensureAuthStateDirectory();
  await page.context().storageState({ path: getStorageStatePath('new-role') });
});
```

Then register the project in `config/projects.ts`.

---

## 5. Locator Strategy (Priority Order)

Use locators in this priority order — most resilient first:

| Priority | Strategy | Example |
|----------|----------|---------|
| 1st | Role-based | `page.getByRole('button', { name: 'ログイン' })` |
| 2nd | Test ID | `page.locator('[data-test="submit"]')` |
| 3rd | Label | `page.getByLabel('メールアドレス')` |
| 4th | CSS (structural) | `page.locator("input[type='email']")` |
| Last resort | XPath | Only when nothing else works |

**Never** use nth-child selectors or text content that may change with i18n.

---

## 6. Assertions

```typescript
// URL assertions
await expect(page).toHaveURL(/\/patient\/dashboard(?:\?.*)?$/);      // regex for query params
await expect(page).toHaveURL('/patient/dashboard', { timeout: 15_000 });

// Visibility
await expect(locator).toBeVisible();
await expect(locator).not.toBeVisible();

// Text content
await expect(locator).toContainText('エラーメッセージ');
await expect(locator).toHaveText('完全一致テキスト');

// Input value
await expect(input).toHaveValue('expected value');

// Count
await expect(locator).toHaveCount(3);
```

---

## 7. Configuration

### Environment variables (.env)

```bash
BASE_URL=https://demo.care-hub.lion-garden.com
PATIENT_URL=https://demo.care-hub.lion-garden.com
CLINIC_URL=https://clinic.demo.care-hub.lion-garden.com
DOCTOR_URL=https://doctor.demo.care-hub.lion-garden.com
BASIC_AUTH_USERNAME=...
BASIC_AUTH_PASSWORD=...
PATIENT_USERNAME=...
PATIENT_PASSWORD=...
DOCTOR_USERNAME=...
DOCTOR_PASSWORD=...
AUTH_STATE_DIR=.auth
HEADLESS=true
CI=false
```

### Commands

```bash
# Auth setup (must run before tests)
npx playwright test --project="setup:patient"
npx playwright test --project="setup:doctor"

# Run all tests
npx playwright test

# Run specific role
npx playwright test --project="chromium-patient"

# Run specific file
npx playwright test tests/smoke/patient/profile-info.spec.ts

# Debug mode
npx playwright test --debug

# Headed browser
npx playwright test --headed

# View HTML report
npx playwright show-report playwright-report/<timestamp>
```

---

## 8. Writing a New Test — Checklist

When adding a test for a new feature, follow this order:

1. **Create page object(s)** in `src/pages/<role>/<feature>.page.ts`
   - Extend `BasePage`
   - Private locator getters only
   - `goto()` asserts readiness
   - Actions return `Promise<void>`

2. **Add fixtures** to `src/fixtures/<role>.fixture.ts`
   - Extend the existing fixture chain (not `@playwright/test` directly)

3. **Create test data** in `test-data/cases/<role>/<feature>.cases.json`
   - All required fields: `caseId`, `title`, `module`, `expected`, `severity`, `steps`
   - Dynamic data as extra fields → becomes `payload`

4. **Write the spec** in `tests/smoke/<role>/<feature>.spec.ts`
   - Call `registerCase(testCase)` first
   - Use `test.describe` with format: `Smoke|Patient|FeatureName`
   - Test title: `[${testCase.caseId}]`

5. **Verify**: run the spec in headed mode to confirm it works before committing
   ```bash
   npx playwright test tests/smoke/<role>/<feature>.spec.ts --headed
   ```

---

## 9. Common Mistakes to Avoid

| Mistake | Fix |
|---------|-----|
| `await page.click(selector)` | Use `locator.click()` instead |
| Hard-coded `page.waitForTimeout(2000)` | Use `expect(locator).toBeVisible()` |
| Public locator getters | Make locators `private get` |
| Missing `registerCase(testCase)` | Always call at start of data-driven test |
| Importing `test` from `@playwright/test` directly | Import from the role fixture instead |
| Navigation without waiting for URL | Use `Promise.all([waitForURL, click])` |
| Selector with changing text | Use `data-test` attribute or role instead |
