# Code Style Rules — TypeScript

These rules apply to all TypeScript files in this project.

---

## TypeScript Conventions

- **Strict mode is enabled** — no `any`, no non-null assertions (`!`) unless you have a comment explaining why
- **Prefer `const` over `let`** — use `let` only when reassignment is required
- **Explicit return types on public methods** of page classes
- **Use `type` for data shapes**, `interface` for class contracts

```typescript
// Correct — explicit return type, no any
async login(email: string, password: string): Promise<void> { ... }

// Correct — type for data
type LoginCase = {
  email: string
  password: string
  expected: { redirectTo: string }
}
```

---

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Page classes | PascalCase + `Page` suffix | `LoginPage`, `ProfileInfoPage` |
| Fixture files | kebab-case + `.fixture.ts` | `patient.fixture.ts` |
| Spec files | kebab-case + `.spec.ts` | `profile-info.spec.ts` |
| Test data files | kebab-case + `.json` | `profile-info.json` |
| Env var names | UPPER_SNAKE_CASE | `PATIENT_URL` |
| Private locator getters | camelCase | `emailInput`, `submitButton` |
| Public methods | camelCase verb | `goto()`, `login()`, `fillForm()` |

---

## Import Order

1. Node.js built-ins
2. Third-party packages (`@playwright/test`, etc.)
3. Internal imports (src/ paths)

Separate each group with a blank line. Use relative imports for src/ files.

---

## File Organization

- One page class per file
- One fixture composition per file
- Export only what is needed — avoid `export *`
- Keep fixture files lean — delegate logic to page classes

---

## Comments

- **Do not add JSDoc** to obvious methods like `goto()` or simple getters
- **Add comments only when the logic is non-obvious** (e.g., why `Promise.all` is needed)
- **TODO comments must include a ticket/issue reference**: `// TODO(ISSUE-123): ...`

---

## Async Patterns

- All Playwright actions are `async` — always `await` them
- Avoid nested `.then()` chains — use `async/await` throughout
- Group independent async setup with `Promise.all` when performance matters:

```typescript
// ✅ Parallel setup
await Promise.all([
  loginPage.goto(),
  page.setExtraHTTPHeaders({ 'Accept-Language': 'ja' })
])

// ❌ Sequential when unnecessary
await loginPage.goto()
await page.setExtraHTTPHeaders({ 'Accept-Language': 'ja' })
```
