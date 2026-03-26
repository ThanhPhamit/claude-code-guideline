# Demo — Claude Code `.claude/` Folder Structure

This folder is a **complete working demo** of how to set up a Claude Code project for a Playwright TypeScript automation test suite.

It can be copied as-is into any Playwright project root.

---

## Folder Structure

```
demo/
  CLAUDE.md                    ← Root project context (auto-loaded by Claude Code)
  .claude/
    settings.json              ← Permissions + hooks
    rules/
      testing.md               ← Playwright/POM coding rules
      code-style.md            ← TypeScript style rules
      security.md              ← Security rules (credentials, secrets)
    agents/
      test-reviewer.md         ← Review test files for best practices
      add-test-case.md         ← Add a data-driven test case
      fix-failing-test.md      ← Debug and fix a failing test
    skills/
      add-page/
        SKILL.md               ← Scaffold a new Page Object class
      add-auth-setup/
        SKILL.md               ← Add auth storage state setup for a new role
```

---

## How Each File Is Used

### `CLAUDE.md`

Loaded automatically when Claude Code opens the project. Contains:

- Build commands (what to run to test, validate, etc.)
- Project structure overview
- Code conventions summary
- Env variable reference
- Common gotchas

### `.claude/settings.json`

Defines **what Claude is allowed to do** (permissions) and **what runs automatically** (hooks).

In this demo:

- `allow`: safe commands (typecheck, lint, playwright test, etc.)
- `deny`: destructive/sensitive commands (rm -rf, git push, cat .env)
- `PostToolUse` hook: runs `npm run typecheck` after every file edit — catches errors immediately
- `PreToolUse` hook: logs bash commands for transparency

### `.claude/rules/*.md`

**Always-loaded context** that Claude reads for every request in this project.

- `testing.md` — POM rules, fixture rules, locator strategy, async patterns
- `code-style.md` — TypeScript conventions, naming, import order
- `security.md` — Credential rules, .env rules, test data rules

### `.claude/agents/*.md`

**Subagents** — specialized Claude configurations invoked for specific tasks.
Each has a `description:` frontmatter field that enables auto-invocation when the user's request matches.

| Agent              | Auto-invokes when user says...                                        |
| ------------------ | --------------------------------------------------------------------- |
| `test-reviewer`    | "review this test", "audit test quality", "check this page class"     |
| `add-test-case`    | "add a test case", "add a scenario to ...", "extend coverage for ..." |
| `fix-failing-test` | "this test is failing", "fix the error", "test is flaky"              |

### `.claude/skills/*.md`

**Reusable workflows** Claude follows step-by-step for recurring tasks.
Skills differ from agents in that they are **procedural** (do these steps), not just **contextual** (know these rules).

| Skill            | Auto-invokes when user says...                                             |
| ---------------- | -------------------------------------------------------------------------- |
| `add-page`       | "create a page class", "add a page object", "scaffold a POM"               |
| `add-auth-setup` | "add auth setup for ...", "create storage state for ...", "new role setup" |

---

## How to Use This Demo

### Option A: Copy to your project root

```bash
cp -r demo/CLAUDE.md /path/to/your-playwright-project/
cp -r demo/.claude /path/to/your-playwright-project/
```

Then edit `CLAUDE.md` to match your project's actual commands, structure, and env vars.

### Option B: Study the patterns

Use this demo to understand:

1. What goes in `CLAUDE.md` vs `rules/` vs `agents/`
2. How to write agent `description:` for auto-invocation
3. How to write a skill as a step-by-step procedure
4. How `settings.json` permissions protect against destructive commands

---

## Key Principles Applied

| Principle                           | How it's applied                                                                  |
| ----------------------------------- | --------------------------------------------------------------------------------- |
| **Context over instructions**       | `CLAUDE.md` gives architecture and gotchas upfront — Claude doesn't have to guess |
| **Rules are always loaded**         | All 3 rules files apply to every request, no invocation needed                    |
| **Agents are specialized**          | Each agent does one thing well, with its own expertise                            |
| **Skills are procedural**           | Skills list exact steps to follow — reduces errors in multi-step tasks            |
| **Permissions are least-privilege** | Only explicitly safe commands are allowed; secrets are blocked                    |
| **Hooks catch errors immediately**  | typecheck runs after every edit, not just at commit time                          |
