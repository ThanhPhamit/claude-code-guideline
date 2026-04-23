---
name: update-server-env
description: 'Standardize backend/server env updates from .env and Terraform wiring to ECS runtime across dev/demo/stg environments.'
metadata:
  domain: backend-devops
  triggers: server env, ECS env, Terraform env, tokyo-dev, tokyo-demo, tokyo-stg
  role: implementation
  scope: config
  output-format: checklist
  related-skills: terraform-engineer, devops-engineer, security-reviewer
---

# Update Server Env

Use this skill whenever you add, rename, or remove server environment variables.

## Goal

Guarantee the new server env is fully wired from configuration to runtime container for all non-prod environments (`tokyo-dev`, `tokyo-demo`, `tokyo-stg`).

## Required Workflow

1. Define env contract

- Confirm exact env names and expected type (`bool`, `string`, `number`).
- Decide defaults and required/optional behavior.

2. Update Terraform module interface

- Add variables in module `variables.tf` with clear `description`.
- Pass values through module `main.tf` template arguments.
- Add env entries in ECS container definition template.

3. Update environment layer (all targets)

- Add declarations in each environment `variables.tf`.
- Wire values into module call in each environment `main.tf`.
- Set actual values in each `terraform.tfvars`.
- Ensure no duplicate declarations or duplicate arguments.

4. Validate all environments

- Run `terraform -chdir=environments/tokyo-dev validate`.
- Run `terraform -chdir=environments/tokyo-demo validate`.
- Run `terraform -chdir=environments/tokyo-stg validate`.

5. Optional deployment safety checks

- Run `terraform plan` per environment before apply.
- Verify task definition diff includes expected env updates only.

## Security Rules

- Never store secrets directly in `.tf` files.
- Use Secrets Manager for sensitive server env values.
- Avoid printing secret values in CI logs.

## Output Format

When applying this skill, report:

1. Env keys added/changed
2. Module files updated
3. Environment files updated (`dev/demo/stg`)
4. Validation status per environment
