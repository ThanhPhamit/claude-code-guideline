# Setup Claude Code Core — New Project Guide

## Concept

```
claude-code-guideline/     ← This repository (Skills, Agents, Rules, MCP)
    .claude/
        skills/            ← 13 skills shared across all projects
        agents/            ← 4 agents (infra-reviewer, security-auditor, ...)
        rules/             ← 5 rules (terraform, k8s, docker, cicd, security)

my-project/                ← Specific project
    CLAUDE.md              ← Generated from /init-project (project-specific)
    .terraform/
    .github/workflows/
    ...
```

**Claude Core** (skills/agents/rules) is loaded from `claude-code-guideline`.
**CLAUDE.md** is created separately for each project, containing specific information about that project.

---

## Prerequisites

`/init-project` automatically sets up everything. You just need:

- Claude Code CLI installed and running in the project directory
- `claude-code-guideline` cloned to your local machine

> **Manual setup** (if you want to set it up before running `/init-project`):

**Method A — VS Code:**

```json
// .vscode/settings.json
{
  "claude.additionalDirectories": [
    "/home/lg-vietnam007/Documents/Devops/claude-code-guideline"
  ]
}
```

**Method B — CLI:**

```bash
claude --add-dir /home/lg-vietnam007/Documents/Devops/claude-code-guideline
```

**Method C — CLAUDE.md import:**

```markdown
@/home/lg-vietnam007/Documents/Devops/claude-code-guideline/.claude/CLAUDE.md
```

---

## Workflow: Setup a New Project

### Step 1: Open the project in VS Code

```bash
cd /path/to/my-project
code .
```

> **Note:** `claude-code-guideline` does not need to be added beforehand — `/init-project` will set it up for you.

### Step 2: Run `/init-project`

In Claude Code, type:

```
/init-project
```

Claude will automatically perform the **entire setup**:

1. **Explore project** — structure, tech stack, dependencies
2. **Read README.md** — identify CI/CD platform (GitHub Actions, GitLab CI, Jenkins, etc.)
3. **Analyze infrastructure** — Terraform, Docker, Kubernetes
4. **Generate CLAUDE.md** — project-specific guidelines
5. **Setup VS Code** — add `claude-code-guideline` to `.vscode/settings.json`
6. **Generate `.mcp.json`** — only includes MCP servers suitable for the tech stack
7. **Update `.gitignore`** — exclude `.mcp.json` (since it contains credentials)

**Default output:**

- `.claude/CLAUDE.md` if the `.claude/` folder exists
- `CLAUDE.md` at root if `.claude/` does not exist yet

**Specify output path:**

```
/init-project .claude/CLAUDE.md
```

### Step 3: Fill credentials in `.mcp.json`

The `.mcp.json` file is generated with placeholder values:

```bash
# Replace placeholder values:
# <your-aws-profile>  → AWS profile name (e.g., my-project-dev)
# <your-aws-region>   → e.g., ap-southeast-1
# <your-github-pat>   → GitHub Personal Access Token
# <your-grafana-*>    → Grafana URL and token (if used)
```

### Step 4: Reload VS Code

After `.vscode/settings.json` is created:

- `Cmd+Shift+P` → **"Developer: Reload Window"**
- Type `/` in Claude → verify that skills appear

### Step 5: Review and customize CLAUDE.md

- Fill in additional project-specific info that Claude couldn't find
- Remove irrelevant sections
- Add gotchas, non-obvious behaviors
- Verify that the commands actually run

### Step 6: Commit to git

```bash
git add CLAUDE.md .vscode/settings.json  # DO NOT commit .mcp.json
git commit -m "chore: add Claude Code guidelines"
```

---

## What does the generated CLAUDE.md look like?

````markdown
# care-hub-backend — Claude Code Guidelines

## Stack

- **Language/Framework:** Python 3.12 / Django
- **Infrastructure:** Terraform (AWS), ECS Fargate + CodeDeploy blue-green
- **Database:** Aurora PostgreSQL 16.4
- **CI/CD:** GitHub Actions with OIDC

## Essential Commands

```bash
# Local dev
docker-compose up -d

# Tests
pipenv run pytest tests/ -v

# Terraform (staging)
cd environments/tokyo-staging && terraform plan
```
````

## Architecture

- ECS Fargate + CodeDeploy blue-green. Task definition managed by CodeDeploy —
  do not run `terraform apply` while the service is deploying.
- Terraform state: S3 `care-hub-tfstate-storage`, key `tokyo-staging/terraform.tfstate`
- Secrets in AWS Secrets Manager — do not use .env in production

## Environments

| Env        | Branch  | Deploy Trigger |
| ---------- | ------- | -------------- |
| develop    | develop | PR merge       |
| staging    | staging | PR merge       |
| production | (any)   | Tag push (v\*) |

## Gotchas

- ECR registry URL contains AWS Account ID → masked in CI logs → pass image tag separately
- `appspec.yaml` is downloaded from S3 then the TaskDefinition ARN is updated before deployment
- RDS rotation rollout = CodeDeploy redeploy after Secrets Manager rotates the password

## Skills Available (claude-code-guideline)

Infrastructure: terraform-engineer, kubernetes-specialist, cloud-architect
DevOps: devops-engineer, monitoring-expert, sre-engineer
Database: postgres-pro, database-optimizer
Security: secure-code-guardian, security-reviewer

```

---

## Skills Available — Quick Reference

| Skill | Invoke | Use when |
|-------|--------|----------|
| `terraform-engineer` | `/terraform-engineer` | Write/review Terraform code |
| `devops-engineer` | `/devops-engineer` | CI/CD, Docker, deployment |
| `kubernetes-specialist` | `/kubernetes-specialist` | K8s manifests, Helm, EKS |
| `postgres-pro` | `/postgres-pro` | Query optimization, replication |
| `cloud-architect` | `/cloud-architect` | AWS architecture design |
| `security-reviewer` | `/security-reviewer` | Security audit code/infra |
| `monitoring-expert` | `/monitoring-expert` | Prometheus, Grafana, alerting |
| `sre-engineer` | `/sre-engineer` | SLO/SLI, incident response |
| `secure-code-guardian` | `/secure-code-guardian` | OWASP, secure coding |
| `database-optimizer` | `/database-optimizer` | DB performance tuning |
| `chaos-engineer` | `/chaos-engineer` | Resilience testing |
| `cli-developer` | `/cli-developer` | Build CLI tools |
| `init-project` | `/init-project` | Generate project CLAUDE.md |

## Agents — Invoke using natural language

| Agent | Invoke | Use when |
|-------|--------|----------|
| `infra-reviewer` | `use the infra-reviewer agent to review this code` | Review Terraform/K8s/Docker PR |
| `security-auditor` | `run a security audit on this project` | Full security audit |
| `incident-responder` | `use the incident-responder agent` | Production incident |
| `cost-optimizer` | `use the cost-optimizer agent` | Analyze AWS costs |

---

## Troubleshooting

**Skills không hiện trong `/` menu:**
- Kiểm tra `claude-code-guideline` đã được add làm additional directory chưa
- Thử `/context` để xem directories đang được load

**CLAUDE.md quá generic:**
- Thêm context bằng cách paste nội dung file cụ thể vào chat trước khi gọi `/init-project`
- Hoặc chạy: `What's the tech stack here? Read package.json, Dockerfile, and .github/workflows/` trước

**Skills không auto-trigger:**
- Skills trigger dựa trên keywords trong conversation
- Nếu không tự trigger, gõ trực tiếp: `/terraform-engineer`
```
