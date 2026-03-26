# Care Hub — Automation Test (CLAUDE.md Demo)

> This file is loaded automatically by Claude Code when you open this project.
> It tells Claude everything it needs to know to work effectively in this codebase.

---

## Project Overview

TypeScript + Playwright automation test suite for the **Care Hub** clinic platform.
Covers **3 user roles**: patient, doctor, clinic-admin.

**Tech stack:** Playwright 1.52, TypeScript 5, Node >=22

---

## Essential Commands

```bash
# 1. Install dependencies
npm ci

# 2. Copy env and fill in credentials
cp .env.example .env

# 3. Generate auth state for all roles (MUST run before tests)
npm run auth:setup

# 4. Run tests
npm run test:smoke:patient        # Patient smoke tests
npm run test:smoke:doctor         # Doctor smoke tests
npm run test:smoke:clinic         # Clinic-admin smoke tests

# 5. Validate (typecheck + lint + format check)
npm run validate

# Individual checks
npm run typecheck
npm run lint
npm run format:check
npm run format:write              # Auto-fix formatting
```

---

## Project Structure

```
src/
  pages/            Page Object Model — one class per page/role
    base.page.ts    Abstract base, all pages extend this
    patient/        Patient-specific pages
    doctor/         Doctor-specific pages
    clinic/         Clinic-admin pages

  fixtures/         Playwright fixture composition
    data-driven.fixture.ts   registerCase() + screenshot on complete
    patient.fixture.ts       Extends with POM instances for patient role
    doctor.fixture.ts        Extends with POM instances for doctor role
    clinic.fixture.ts        Extends with POM instances for clinic role

  data-driven/      Test data loading utilities
    schema.ts        DataDrivenCase type definition
    loaders.ts       loadJsonCases(), loadCsvCases()

  auth/
    storage-state.ts  getStorageStatePath(), getBaseUrl() per role

  reporters/        Custom Playwright reporters

tests/
  auth/             Storage state setup (run once)
    patient.setup.ts
    doctor.setup.ts
    clinic.setup.ts

  smoke/            Smoke test suites
    patient/
    doctor/
    clinic/

test-data/
  cases/            JSON test case files per spec

config/
  env.ts            Env var loading + validation
  projects.ts       Playwright project definitions

.auth/              Storage state files (gitignored)
  patient.json
  doctor.json
  clinic.json
```

---

## Code Conventions

### Page Object Model

- All pages extend `BasePage` (which receives `page: Page` in constructor)
- Locators are **private getters** — never public fields
- Public methods: `goto()`, action methods (e.g. `login()`), assertion methods (e.g. `expectLoginError()`)

```typescript
// ✅ Correct
export class LoginPage extends BasePage {
  private get emailInput() { return this.page.getByLabel('Email') }
  private get submitButton() { return this.page.getByRole('button', { name: 'ログイン' }) }

  async goto() { await this.page.goto('/sign-in') }
  async login(email: string, password: string) { ... }
  async expectLoginError(message: string) { ... }
}

// ❌ Wrong — public locators, no goto()
export class LoginPage extends BasePage {
  emailInput = this.page.locator('input[type=email]')
}
```

### Fixtures

- Import from the **role-specific fixture** (not directly from `@playwright/test`)
- Use `registerCase()` to attach the test case to the fixture context

```typescript
import { test, expect } from '../../fixtures/patient.fixture';

test.describe('Profile info', () => {
  for (const tc of loadJsonCases('profile-info')) {
    test(tc.name, async ({ page, loginPage, registerCase }) => {
      registerCase(tc);
      // ... test body
    });
  }
});
```

### Data-Driven Cases (JSON format)

```json
[
  {
    "name": "valid login",
    "data": { "email": "test@example.com", "password": "secret" },
    "expected": { "redirectTo": "/patient/dashboard" }
  }
]
```

### Auth Setup Pattern

```typescript
// tests/auth/patient.setup.ts
test('setup patient storage state', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login(
    process.env.PATIENT_USERNAME!,
    process.env.PATIENT_PASSWORD!,
  );
  await page.context().storageState({ path: getStorageStatePath('patient') });
});
```

---

## Environment Variables

| Variable           | Required | Description                                      |
| ------------------ | -------- | ------------------------------------------------ |
| `AUTH_STATE_DIR`   | No       | Default `.auth`. Dir for storage state files     |
| `BROWSERS`         | No       | Default `chromium`. Comma-separated browser list |
| `HEADLESS`         | No       | Default `true`. Set `false` for headed mode      |
| `CI`               | No       | Default `false`. Adjusts timeouts for CI         |
| `PATIENT_URL`      | **Yes**  | Base URL for patient portal                      |
| `PATIENT_USERNAME` | **Yes**  | Patient login email                              |
| `PATIENT_PASSWORD` | **Yes**  | Patient login password                           |
| `DOCTOR_URL`       | **Yes**  | Base URL for doctor portal                       |
| `DOCTOR_USERNAME`  | **Yes**  | Doctor login email                               |
| `DOCTOR_PASSWORD`  | **Yes**  | Doctor login password                            |
| `CLINIC_URL`       | **Yes**  | Base URL for clinic-admin portal                 |
| `CLINIC_USERNAME`  | **Yes**  | Clinic login email                               |
| `CLINIC_PASSWORD`  | **Yes**  | Clinic login password                            |

---

## Important Gotchas

1. **Always run `npm run auth:setup` before tests** — tests rely on storage state in `.auth/`
2. **Never commit `.auth/*.json` or `.env`** — they contain live session tokens and credentials
3. **Use `Promise.all` for click + navigation** — avoids race condition:
   ```typescript
   await Promise.all([page.waitForNavigation(), loginPage.clickSubmit()]);
   ```
4. **Locators use Japanese text** — this is a Japanese clinic app; selectors like `getByRole('button', { name: 'ログイン' })` are correct
5. **`test:smoke:*` scripts use `--project` flag** — see `config/projects.ts` for project names

---

## When Adding a New Test

1. Add test case data to `test-data/cases/<spec-name>.json`
2. Create or extend the spec file in `tests/smoke/<role>/`
3. Import from the correct role fixture (not `@playwright/test`)
4. Use `registerCase(tc)` before assertions
5. Run `npm run validate` before committing
