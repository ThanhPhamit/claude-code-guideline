---
name: update-client-env
description: "Standardize frontend .env updates for Vite client, CI/CD propagation, and feature-flag consistency across environments."
metadata:
  domain: frontend-devops
  triggers: .env, VITE_, frontend env, workflow env, feature flag
  role: implementation
  scope: config
  output-format: checklist
  related-skills: devops-engineer, secure-code-guardian
---

# Update Client Env

Use this skill whenever you add, rename, or remove frontend environment variables (especially `VITE_*`).

## Goal

Ensure a frontend env change is complete, consistent, and deploy-safe across:
- local development
- `.env.example`
- CI/CD workflow build env
- all deployment environments (develop/demo/staging/production)

## Required Workflow

1. Identify the env change
- Confirm exact key names and default values.
- Confirm whether key is boolean/string/number (Vite reads as string).

2. Update source-of-truth env templates
- Update `.env.example` with new keys.
- Keep placeholders for sensitive values, never commit secrets.

3. Update runtime usage
- Add or update env reads in frontend config (for example `src/core/config/features.ts`, `config.ts`, or equivalent).
- For booleans, normalize via helper parsing (`true/false`) instead of raw string checks.

4. Propagate to CI/CD
- Update workflow outputs (setup job) if workflow maps env via outputs.
- Update environment-specific assignment blocks (develop/demo/staging/production).
- Pass env values into build step (`env:` in build job).

5. Validate
- Ensure all target environments include the key.
- Run lint/check commands if available.
- If workflow changed, run YAML diagnostics and validate no missing outputs.

## Security Rules

- Never hardcode credentials or tokens.
- Keep secrets in secret managers / CI secrets only.
- Do not print sensitive env values in logs.

## Output Format

When applying this skill, report:
1. Added/changed env keys
2. Files updated
3. Environment coverage (develop/demo/staging/production)
4. Validation results and any follow-up actions
