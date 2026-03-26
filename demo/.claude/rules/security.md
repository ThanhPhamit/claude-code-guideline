# Security Rules

These rules apply to ALL files in this project. Claude must enforce these at all times.

---

## Credentials — NEVER

- **Never hardcode credentials** (email, password, token, API key) in any source file
- **Never hardcode base URLs** for patient/doctor/clinic portals — always read from `process.env.*`
- **Never log or print credentials** — not in console.log, not in error messages
- **Never commit `.env` files** — only `.env.example` with placeholder values belongs in git
- **Never commit `.auth/*.json`** — storage state files contain live session tokens

```typescript
// ❌ WRONG — hardcoded credentials
await loginPage.login('admin@clinic.jp', 'password123')

// ✅ CORRECT — from environment variables
await loginPage.login(process.env.PATIENT_USERNAME!, process.env.PATIENT_PASSWORD!)
```

---

## Environment Variables

- All sensitive values (URLs, usernames, passwords, API keys) must come from `process.env`
- Use `config/env.ts` to load and validate env vars — do not read `process.env` directly in page classes or spec files
- All required env vars must appear in `.env.example` with placeholder values

---

## Test Data

- **Never use real patient data** (real names, real emails, real phone numbers) in test cases
- Use clearly fake/test data: `test-patient@example.com`, `テスト太郎`, `090-0000-0000`
- If a test requires a real-looking email, use a dedicated test inbox (e.g., MailSlurp)

---

## Storage State Files

- `.auth/` directory is in `.gitignore` — verify this before adding files
- Storage state JSON files are equivalent to session cookies — treat them as secrets
- Do not copy storage state files outside the project directory
- Rotate test account credentials if storage state is accidentally committed

---

## Dependency Security

- Run `npm audit` before adding new packages
- Pin major versions in `package.json` — do not use `*` or `latest`
- Review any package that requires network access or file system access

---

## CI/CD

- In CI environments, credentials are injected as environment secrets — never echo them
- Never disable SSL verification in Playwright config
- Review `playwright.config.ts` before modifying `ignoreHTTPSErrors` — it must remain `false`
