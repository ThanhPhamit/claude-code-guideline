# Using the `init-project` skill

This skill **bootstraps Claude Code Core for a new project** — auto-detects the tech stack, generates a tailored `CLAUDE.md`, selectively copies the relevant skills/agents/rules, and produces a `.mcp.json`.

---

## 1. Install (once per machine)

The skill lives in the `claude-code-guideline` repo. To make it available from any project on your machine, symlink it into your user-level skills directory:

```bash
mkdir -p ~/.claude/skills
ln -sfn ~/Documents/Devops/claude-code-guideline/.claude/skills/init-project \
        ~/.claude/skills/init-project
```

Verify:

```bash
ls -la ~/.claude/skills/init-project/SKILL.md
```

> **Why a symlink instead of a copy?**
> When you `git pull` the guideline repo to update the skill, every project picks up the new version automatically.

> **Why not `/add-dir`?**
> `/add-dir` only grants read/write access to a directory — it does **not** load the `.claude/` config from that directory into the session. Skills/agents/rules are loaded only from:
> - `~/.claude/` (user level — visible to every project)
> - `<project>/.claude/` (current project only)

---

## 2. Standard workflow when starting a new project

### Step 1: Prepare the project

The project should contain at least one of:
- `README.md` (describes tech stack, CI/CD)
- `package.json` / `go.mod` / `Pipfile` / `Cargo.toml`
- `Dockerfile` / `docker-compose.yml`
- Terraform `.tf` files

→ The skill reads these files to **detect the tech stack**. An empty project produces a generic result.

### Step 2: Open Claude Code in the project directory

```bash
cd /path/to/your-new-project
claude
```

> ⚠️ **You must `cd` into the project before running `claude`** — do not use `/add-dir`. The `.claude/` directory is created at the **session's working directory** at startup.

### Step 3: Invoke the skill

In Claude Code, type:

```
/init-project
```

To write `CLAUDE.md` to a custom path, pass it as an argument:

```
/init-project docs/CLAUDE.md
```

### Step 4: The skill runs six phases automatically

| Phase | What it does |
|---|---|
| 1. Explore | Scans file structure, reads README + manifest files |
| 2. Analyze | Inspects Terraform / Docker / K8s configurations if present |
| 3. Generate CLAUDE.md | Writes a ~100–150 line file tailored to the project |
| 4. Copy Core Files | Copies **only** the relevant skills/agents/rules into `.claude/` |
| 5. Generate .mcp.json | Builds an MCP config with **only** the relevant servers, then adds it to `.gitignore` |
| 6. Summary | Prints a recap of everything it did |

### Step 5: Manual follow-up

Once the skill finishes:

1. **Fill in placeholders in `.mcp.json`:**
   ```bash
   grep -n '<your-' .mcp.json
   ```
   Common placeholders:
   - `<your-aws-profile>` — e.g. `claude-mcp-default` (see [aws-iam-mcp-setup.md](../../../knowledge/aws-iam-mcp-setup.md) for the recommended setup)
   - `<your-aws-region>` — e.g. `ap-southeast-1`
   - `<your-github-pat>` — GitHub Personal Access Token
   - `<your-grafana-url>` / `<your-grafana-api-token>`

2. **Set up the AWS IAM user for MCP** (one-time per developer per AWS account):
   Follow [knowledge/aws-iam-mcp-setup.md](../../../knowledge/aws-iam-mcp-setup.md) to create a dedicated, read-only IAM user with permission boundary, conditional access, and rotation schedule. This is **required before MCP can talk to AWS**.

3. **Review `CLAUDE.md`:**
   - Verify the build/test/deploy commands are accurate
   - Add gotchas and quirks the skill couldn't infer

4. **Restart Claude Code** so the new `.claude/` is loaded:
   ```
   /exit
   claude
   ```

5. **Commit:**
   ```bash
   git add CLAUDE.md .claude/
   git commit -m "chore: add Claude Code guidelines"
   # DO NOT commit .mcp.json — it's already in .gitignore
   ```

---

## 3. Detection table — what the skill copies

| Detected | Skills added | Agents added | Rules added | MCP servers added |
|---|---|---|---|---|
| **Always** | `init-project`, `devops-engineer`, `secure-code-guardian` | — | `security.md` | — |
| **Terraform** | `terraform-engineer`, `cloud-architect` | `infra-reviewer.md` | `terraform.md` | `terraform`, `iac` |
| **Docker** | — | — | `docker.md` | — |
| **Kubernetes / Helm** | `kubernetes-specialist` | — | `kubernetes.md` | `eks` (if EKS) |
| **PostgreSQL / Aurora** | `postgres-pro`, `database-optimizer` | — | — | `aurora-postgresql` |
| **MySQL** | — | — | — | `aurora-mysql` |
| **Redis / ElastiCache** | — | — | — | `elasticache`, `elasticache-valkey` |
| **Lambda / Serverless** | — | — | — | `serverless`, `lambda-tool` |
| **SQS / SNS** | — | — | — | `sns-sqs` |
| **Kafka / MSK** | — | — | — | `msk` |
| **ECS** | — | — | — | `ecs` |
| **AWS (any service)** | — | `cost-optimizer.md`, `incident-responder.md` | — | `aws-api`, `aws-knowledge`, `cloudwatch`, `iam` |
| **CLI tool** | `cli-developer` | — | — | — |
| **Chaos / resilience** | `chaos-engineer` | — | — | — |
| **SRE / SLO** | `sre-engineer` | — | — | — |
| **Security-sensitive** (PII / finance / healthcare) | `security-reviewer` | `security-auditor.md` | — | — |
| **Grafana / Prometheus** | `monitoring-expert` | — | — | `grafana` |
| **GitHub Actions in README** | — | — | `cicd.md` | `github` |
| **GitLab CI in README** | — | — | `cicd.md` | `gitlab` |
| **Jenkins in README** | — | — | `cicd.md` | `jenkins` |
| **New project / design phase** | — | — | — | `well-architected` |

> CI/CD detection reads the **README only**, not `.github/workflows/`. At init time the workflow files may not exist yet.

---

## 4. Troubleshooting

**Typing `/init-project` shows no suggestion?**
- Restart Claude Code (skills are loaded at startup).
- Check the symlink: `ls -la ~/.claude/skills/init-project`
- Check the frontmatter: `head -10 ~/.claude/skills/init-project/SKILL.md` — must include `name: init-project`.

**The skill ran but `.claude/` is empty / missing skills?**
- The skill only copies files that **already exist** under `~/Documents/Devops/claude-code-guideline/.claude/`. Verify:
  ```bash
  ls ~/Documents/Devops/claude-code-guideline/.claude/skills/
  ls ~/Documents/Devops/claude-code-guideline/.claude/agents/
  ls ~/Documents/Devops/claude-code-guideline/.claude/rules/
  ```
- A missing source file is silently skipped (not an error).

**`$GUIDELINE_CLAUDE` in `SKILL.md` resolving wrong?**
- `SKILL.md` uses `$(dirname "$(dirname "$CLAUDE_SKILL_DIR")")`. Claude Code sets `CLAUDE_SKILL_DIR` to the running skill's directory. With a symlink, it resolves through the symlink target — i.e. back into the guideline repo — which is correct.
- If you copied the skill instead of symlinking it, the variable resolves to `~/.claude/`, and the `skills/`, `agents/`, `rules/` directories must exist directly under `~/.claude/` (they don't, by default). This is another reason to symlink.

**`CLAUDE.md` already exists — does re-running the skill overwrite it?**
- Yes, it is overwritten. Either back it up first, or pass a different output path:
  ```
  /init-project CLAUDE.new.md
  ```

**`.mcp.json` already exists — what happens?**
- The skill **skips Phase 5 entirely** and leaves the existing `.mcp.json` untouched.

---

## 5. When to re-run vs. not

✅ **Re-run** when:
- A new tech stack is added to the project (e.g. Terraform added to a Node.js project)
- The guideline repo gains new skills/agents/rules and you want to pull them into an existing project

❌ **Don't re-run** when:
- You've heavily customized `CLAUDE.md` — it will be overwritten
- The change is small and doesn't affect the detected tech stack

→ For one-off file updates, copy directly from `~/Documents/Devops/claude-code-guideline/.claude/`.

---

## 6. Related docs

- [`knowledge/aws-iam-mcp-setup.md`](../../../knowledge/aws-iam-mcp-setup.md) — set up the AWS IAM user that powers all AWS MCP servers
- [`knowledge/mcp-devops-setup.md`](../../../knowledge/mcp-devops-setup.md) — full catalog of supported MCP servers with `.mcp.json` snippets
- [`knowledge/claude-code-core.md`](../../../knowledge/claude-code-core.md) — overview of Claude Code Core (skills/agents/rules system)
