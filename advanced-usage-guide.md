# Claude Code — Usage Guide

> **Quick rule of thumb:** Type `/` in the prompt to see all available skills, agents, and commands.

---

## Standard Usage (Quick Reference)

These work out of the box with no extra setup:

| Goal | How |
|---|---|
| Browse / invoke a skill | `/` → pick from menu, or type `/terraform-engineer` directly |
| Delegate to an agent | `use the infra-reviewer agent to review @modules/rds/` |
| Reference a file or folder | `@src/api/` or `@variables.tf` |
| Explore safely without edits | `claude --permission-mode plan` |
| See Claude's reasoning | `Ctrl+O` (verbose) or `Alt+T` (toggle thinking) |
| Increase thinking depth | `/effort high` or include `ultrathink` in the prompt |
| Compress a long session | `/compact focus on terraform changes` |
| Full reset between tasks | `/clear` |
| Undo / restore a checkpoint | `Esc+Esc` → `/rewind` |
| Quick question, no context impact | `/btw your question` |
| Inspect what's loaded in memory | `/memory` |
| Resume a previous session | `claude --continue` or `claude --resume` |

---

## Bundled Skills — What They Actually Do

These are **built-in skills that ship with Claude Code**. They're not in `.claude/skills/` but are available in every session. Type `/` to see them.

### `/batch <instruction>` — Parallel Large-Scale Changes

Fans out changes across many files or services simultaneously. Each agent works in an **isolated git worktree** — no conflicts between agents.

**Use when:**
- Refactoring or migrating many files at once (20+ files)
- Applying a repeated, pattern-based change across the whole codebase

**Avoid when:**
- Changes require cross-file context (e.g. refactoring an interface then updating all callers — sequential is safer)
- Files have ordering dependencies

```
/batch ensure all Terraform modules under environments/ have a required_providers block
      with version constraints matching the root versions.tf

/batch add a standard "ManagedBy = Terraform" tag to every aws_* resource
      that is missing it across all .tf files
```

**How it works internally:** Claude spawns 5–30 sub-agents, each handling a subset of files in their own worktree, then aggregates results back.

---

### `/simplify [focus]` — Parallel Code Quality Review

Spawns **3 review agents in parallel**, each analyzing code from a different angle (structure / logic / readability), then aggregates into a single set of suggestions and applies them.

**Use when:**
- A module is complex and needs a multi-angle review
- Code has deep nesting or unclear control flow
- You're not sure where to start a refactor

```
/simplify the ECS task definition builder in modules/ecs/main.tf

/simplify deeply nested conditionals in the WAF rule logic

/simplify   # no argument = uses current active file/context
```

> `/simplify` targets **code quality** (readability, structure). For security concerns → use the `security-auditor` agent instead.

---

### `/debug [description]` — Session Debug Analysis

Enables verbose debug logging **for the full session**, then analyzes that log to identify what went wrong.

**Use when:**
- Claude is stuck, looping, or behaving unexpectedly
- A tool call is silently failing
- A hook or permission rule isn't triggering as expected

```
/debug why terraform fmt hook didn't run after the last file edit

/debug Claude keeps re-reading the same file instead of progressing to the next step
```

> Different from `/btw`: `/btw` is a side question that doesn't enter context. `/debug` is a real diagnostic command that changes session behavior.

---

### `/loop [interval] <prompt>` — Scheduled Polling Within a Session

Repeats a prompt on a set interval **within the active session**, running in the background while you continue working.

**Use when:**
- Waiting for a CI/CD pipeline to complete
- Health-checking a service during startup
- Monitoring state while doing other work in the session

```
/loop 30s check if ECS service my-app-prod has reached RUNNING state and alert me when it does

/loop 2m check GitHub Actions status for the latest commit on main
      and summarize any failures
```

> `/loop` only runs while the session is open. For persistent scheduled tasks (e.g. overnight or on a schedule) → use Cloud Scheduled Tasks at claude.ai/code or a GitHub Actions cron workflow.

---

### `/claude-api` — Load Claude API Reference

Loads the Claude API documentation into context. **Auto-activates** when your code imports the `anthropic` package, but can also be triggered manually.

**Use when:**
- Writing code that calls the Claude API
- Building a tool or script using the Anthropic SDK
- Need to verify exact method signatures or parameters

```
/claude-api
Now write a Python script that streams a response from claude-3-7-sonnet
with a system prompt about Terraform best practices.
```

---

## Advanced Usage Patterns

Common patterns that unlock more power from Claude Code across real workflows.

### Non-Interactive Mode (CI/Scripting)

Run Claude headlessly inside pipelines or shell scripts using `claude -p`:

```bash
# Pipe logs into Claude for analysis
cat application.log | \
  claude -p "Find all ERROR entries related to database connection timeouts.
    Group by service and suggest whether Aurora scaling is needed." \
  --output-format json | jq '.'

# Bulk update Terraform files
for file in environments/**/*.tf; do
  claude -p "Ensure this file has a required_providers block with pinned versions." \
    < "$file" > "${file}.tmp" && mv "${file}.tmp" "$file"
done

# Typo/lint check in CI
claude -p "Review changes vs main. Report typos: filename + line number on one line,
  description on next. No other output." \
  --output-format json
```

---

### Parallel Features with Git Worktrees

When working on multiple infra changes simultaneously, worktrees prevent Claude sessions from mixing context:

```bash
# Terminal 1: debugging production RDS auth
claude --worktree fix/rds-iam-auth

# Terminal 2: building new EKS module in parallel
claude --worktree feature/eks-blueprints
```

To automatically share gitignored files (env files, AWS config) across worktrees, create `.worktreeinclude` at the project root:

```
.env
~/.aws/credentials
scripts/local-deploy.sh
```

---

### Controlling Thinking Depth

For complex infrastructure decisions (module design, migration strategy, security architecture), raise effort level:

```bash
# In-session
/effort high

# Or inline in the prompt
ultrathink: analyze our networking module for potential routing loops or overlapping subnets

# Via environment variable (persists across CLI invocations)
export CLAUDE_CODE_EFFORT_LEVEL=high
export MAX_THINKING_TOKENS=25000
```

---

### Delegating to Subagents for Isolated Investigations

Use subagents when a task involves reading many files and you don't want to bloat your active context. The agent runs in a separate context and reports a clean summary back:

```
use the security-auditor subagent to audit all IAM role definitions in @modules/iam-role/

ask the cost-optimizer subagent to review state file outputs for any unused NAT Gateways
```

**Writer/Reviewer pattern** — useful after implementing something complex:

```
# Session A: implement
implement a secrets rotation workflow for the RDS module

# Session B: review (fresh context, no bias)
review @modules/rds_secret_rotation/ for edge cases, race conditions,
and consistency with existing rotation patterns
```

---

### Skill Token Budget After `/compact`

After `/compact`, skills are truncated to **5,000 tokens max** (shared budget of 25,000 tokens across all skills). Long skills like `terraform-engineer` (with 7 reference files) may lose their reference section.

**Fix:** Reinvoke the skill explicitly with `/terraform-engineer` to reload it fully.

Check current budget usage: `/context`

---

### `disable-model-invocation` vs `user-invocable: false`

Two opposite settings on skills — easy to confuse:

| Setting | You can invoke? | Claude auto-invokes? | Use for |
|---|---|---|---|
| *(default)* | ✅ | ✅ | Standard reference skills |
| `disable-model-invocation: true` | ✅ | ❌ | Side-effect actions (deploy, commit, send Slack) |
| `user-invocable: false` | ❌ | ✅ | Background knowledge, hidden from `/` menu |

Skills in this repo's `.claude/skills/` use the default — Claude can auto-trigger them when context matches.
