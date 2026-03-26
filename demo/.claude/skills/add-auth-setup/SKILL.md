---
name: Add Auth Setup
description: "Create a new storage state setup file for a new role. Use when adding a new authenticated role that needs its own auth state, or when asked to add auth setup for a new user type."
---

# Skill: Add Auth Setup

Scaffold the auth storage state setup for a new role in this project.

## Required Files

Creating auth setup for a new role requires:
1. `tests/auth/<role>.setup.ts` — the setup test
2. Entry in `config/projects.ts` — Playwright project definition
3. Entry in `src/auth/storage-state.ts` — `getStorageStatePath()` support
4. Env vars in `.env.example`

## Step-by-Step

### Step 1 — Create the setup test

File: `tests/auth/<role>.setup.ts`

```typescript
import { test as setup } from '@playwright/test'
import { LoginPage } from '../../src/pages/<role>/login.page'
import { getStorageStatePath } from '../../src/auth/storage-state'

setup('<role> auth setup', async ({ page }) => {
  const loginPage = new LoginPage(page)
  await loginPage.goto()
  await loginPage.login(
    process.env.<ROLE>_USERNAME!,
    process.env.<ROLE>_PASSWORD!
  )
  // Wait for successful login — adjust URL based on role's dashboard path
  await page.waitForURL('**/<role-dashboard-path>')
  await page.context().storageState({ path: getStorageStatePath('<role>') })
})
```

### Step 2 — Register in `src/auth/storage-state.ts`

Add the new role to the `getStorageStatePath()` switch/map:

```typescript
case '<role>':
  return path.join(storageStateDir, '<role>.json')
```

### Step 3 — Add Playwright project in `config/projects.ts`

```typescript
{
  name: '<role>-setup',
  testMatch: 'tests/auth/<role>.setup.ts',
  use: {
    baseURL: process.env.<ROLE>_URL,
  },
},
{
  name: '<role>-smoke',
  testMatch: 'tests/smoke/<role>/**/*.spec.ts',
  dependencies: ['<role>-setup'],
  use: {
    baseURL: process.env.<ROLE>_URL,
    storageState: getStorageStatePath('<role>'),
  },
},
```

### Step 4 — Add env vars to `.env.example`

```bash
# <ROLE> portal
<ROLE>_URL=https://example.com
<ROLE>_USERNAME=test-<role>@example.com
<ROLE>_PASSWORD=changeme
```

### Step 5 — Add npm script in `package.json`

```json
"auth:setup:<role>": "playwright test --project=<role>-setup",
"test:smoke:<role>": "playwright test --project=<role>-smoke"
```

### Step 6 — Validate

```bash
npm run typecheck
npm run auth:setup:<role>
```

## Notes

- The `<role-dashboard-path>` in `waitForURL` must match what the app redirects to after login
- If the role uses a different login page than others, create `src/pages/<role>/login.page.ts`
- Storage state is saved to `.auth/<role>.json` — this file is gitignored
