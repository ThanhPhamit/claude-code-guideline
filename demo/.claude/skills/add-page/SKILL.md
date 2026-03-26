---
name: Add Page Object
description: "Create a new Page Object class for a new page or screen. Use when asked to create a new page class, add a POM, or scaffold a page object."
---

# Skill: Add Page Object

Create a new Page Object Model class for a page in this project.

## Instructions

### Step 1 — Identify the role
The user will specify which role this page belongs to: `patient`, `doctor`, or `clinic`.

Page file goes in: `src/pages/<role>/<page-name>.page.ts`

### Step 2 — Scaffold the class

Use this exact template:

```typescript
import { Page } from '@playwright/test'
import { BasePage } from '../base.page'

export class $NAME extends BasePage {
  constructor(page: Page) {
    super(page)
  }

  // --- Navigation ---

  async goto(): Promise<void> {
    await this.page.goto('$PATH')
  }

  // --- Locators (private) ---

  // private get someInput() { return this.page.getByLabel('...') }

  // --- Actions ---

  // async doSomething(): Promise<void> { ... }

  // --- Assertions ---

  // async expectSomething(): Promise<void> { ... }
}
```

Replace:
- `$NAME` with PascalCase + `Page` suffix (e.g., `ProfileInfoPage`)
- `$PATH` with the route path (e.g., `/patient/profile-info`)

### Step 3 — Register in the fixture

Open `src/fixtures/<role>.fixture.ts` and add:

```typescript
import { $NAME } from '../pages/<role>/<page-name>.page'

// Inside the fixture extend() block, add:
$instanceName: new $NAME(page),
```

Use camelCase for instance name (e.g., `profileInfoPage`).

### Step 4 — Validate

Run: `npm run typecheck`

Fix any import or type errors before finishing.

## Example

**User prompt:** "Add a page class for the patient delivery address page"

**You will:**
1. Create `src/pages/patient/delivery-address.page.ts` with class `DeliveryAddressPage`
2. Set `goto()` path to `/patient/delivery-address`
3. Add private locators for the address form fields
4. Add `fillAddressForm()` action method
5. Register `deliveryAddressPage` in `patient.fixture.ts`
6. Run `npm run typecheck`
