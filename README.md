# Claude Code Guideline

A living collection of knowledge, skills, and a complete demo for working with Claude Code effectively.

---

## Structure

```
claude-code-guideline/
  README.md
  knowledge/
    claude-code-core.md          ← Core principles distilled from Claude Code docs
  demo/
    README.md                    ← Explains the demo in detail
    .claude/
      CLAUDE.md                  ← Root project context (auto-loaded by Claude Code)
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
        playwright/
          SKILL.md               ← Full Playwright testing skill
        add-page/
          SKILL.md               ← Scaffold a new Page Object class
        add-auth-setup/
          SKILL.md               ← Add auth storage state setup for a new role
```

---

## How to Use

### 1. Knowledge files

Reference these when setting up CLAUDE.md for a project or when onboarding a new codebase. They distill the Claude Code documentation into actionable guidelines.

- [knowledge/claude-code-core.md](knowledge/claude-code-core.md) — Memory system, skills, workflows, context management, prompting patterns, hooks, subagents, MCP, permission modes

### 2. Demo — Complete `.claude/` folder

The `demo/` folder is a **ready-to-copy Claude Code project setup** for a TypeScript + Playwright automation test suite.

**Copy into a project:**

```bash
cp demo/.claude/CLAUDE.md /path/to/your-project/
cp -r demo/.claude /path/to/your-project/
```

See [demo/README.md](demo/README.md) for a full explanation of every file and how each is used.

### 3. Skills

Skills are `SKILL.md` files in `.claude/skills/<name>/SKILL.md`. Claude Code loads them when the user's prompt matches the skill's `description`.

| Skill                                                         | Description                                                                                                       |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| [playwright](demo/.claude/skills/playwright/SKILL.md)         | Write, extend, and debug Playwright tests. Covers POM, fixtures, data-driven cases, auth setup, locator strategy. |
| [add-page](demo/.claude/skills/add-page/SKILL.md)             | Scaffold a new Page Object class for a page or screen.                                                            |
| [add-auth-setup](demo/.claude/skills/add-auth-setup/SKILL.md) | Add auth storage state setup for a new user role.                                                                 |

### 4. Agents

Agents are markdown files in `.claude/agents/` with a `description:` frontmatter. Claude auto-invokes them when the request matches.

| Agent                                                       | Auto-invokes when...                     |
| ----------------------------------------------------------- | ---------------------------------------- |
| [test-reviewer](demo/.claude/agents/test-reviewer.md)       | "review this test", "audit test quality" |
| [add-test-case](demo/.claude/agents/add-test-case.md)       | "add a test case", "add a scenario"      |
| [fix-failing-test](demo/.claude/agents/fix-failing-test.md) | "this test is failing", "fix the error"  |

---

## Core Principles (from Claude Code docs)

1. **Context window is the constraint** — everything degrades when it fills
2. **CLAUDE.md under 200 lines** — longer files cause rules to be ignored
3. **Explore → Plan → Implement → Commit** — never code without understanding first
4. **Always give Claude a way to verify** — tests, screenshots, expected outputs
5. **Skills over CLAUDE.md** — domain knowledge loads on demand, not every session
6. **Subagents for research** — they run in separate context, keeping yours clean
7. **/clear between unrelated tasks** — stale context causes mistakes
