# Testing Rules — Playwright Automation

These rules apply when writing, reviewing, or modifying test files in this project.

---

## Page Object Model (POM) Rules

- **Every page gets its own class** that extends `BasePage` from `src/pages/base.page.ts`
- **Locators must be private getters**, never public fields or variables
- **Each class must have a `goto()` method** that navigates to that page's URL
- **Assertion methods start with `expect`** (e.g., `expectLoginError()`, `expectPageTitle()`)
- **Action methods are descriptive** (e.g., `login()`, `fillProfileForm()`, `submitOrder()`)
- Do not mix page navigation logic into spec files — keep it in the page class

```typescript
// Correct POM structure
export class SomePage extends BasePage {
  private get someInput() { return this.page.getByLabel('Label text') }
  private get submitBtn() { return this.page.getByRole('button', { name: 'Submit' }) }

  async goto() { await this.page.goto('/some-path') }
  async fillAndSubmit(value: string) {
    await this.someInput.fill(value)
    await this.submitBtn.click()
  }
  async expectSuccessMessage() {
    await expect(this.page.getByText('成功')).toBeVisible()
  }
}
```

---

## Fixture Rules

- **Import `test` and `expect` from the role-specific fixture**, not from `@playwright/test` directly
  - Patient tests: `import { test, expect } from '../../../fixtures/patient.fixture'`
  - Doctor tests: `import { test, expect } from '../../../fixtures/doctor.fixture'`
- **Call `registerCase(tc)` inside every data-driven test** before any assertion
- **Fixtures are composed** — higher-level fixtures extend lower-level ones; do not duplicate logic

---

## Data-Driven Test Rules

- **Test cases live in `test-data/cases/`** as JSON files, one file per spec
- **JSON file name matches the spec file name** (e.g., `profile-info.spec.ts` → `profile-info.json`)
- **Each case must have `name`, `data`, `expected` fields** per the `DataDrivenCase` schema
- **Iterate with `for...of`** over loaded cases, not `forEach`

```typescript
for (const tc of loadJsonCases('profile-info')) {
  test(tc.name, async ({ ..., registerCase }) => {
    registerCase(tc)
    // test body
  })
}
```

---

## Locator Strategy (priority order)

1. `getByRole()` — preferred for interactive elements (buttons, inputs, links)
2. `getByLabel()` — for form inputs with visible labels
3. `getByText()` — for static text assertions
4. `getByTestId()` — only when other selectors are unstable
5. `locator('css')` — last resort, avoid if possible

**Never** use `locator('xpath=...')` unless absolutely necessary.
**Never** use index-based locators like `.nth(0)` without a comment explaining why.

---

## Auth Setup Rules

- Auth setup tests live in `tests/auth/` — one file per role
- They must call `page.context().storageState({ path: ... })` at the end
- They run in a special Playwright project (`setup`) that runs before smoke tests
- Do not put assertions in setup files beyond confirming successful login

---

## General Test Rules

- **No hardcoded timeouts** — use Playwright's `waitFor` options or `expect().toBeVisible()`
- **No `page.waitForTimeout()`** — always wait for element state, not arbitrary time
- **Use `Promise.all` for navigation triggered by click**:
  ```typescript
  await Promise.all([page.waitForNavigation(), someElement.click()])
  ```
- **Test names must be descriptive** — include the role, action, and outcome
- **One assertion per logical step** — split complex flows into named `test.step()` blocks
