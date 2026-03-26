# Claude Code Guideline

A living collection of knowledge and skills for working with Claude Code effectively.

---

## Structure

```
claude-code-guideline/
  knowledge/
    claude-code-core.md     ← Core principles distilled from Claude Code docs
  skills/
    playwright/
      SKILL.md              ← Playwright testing skill (example)
```

---

## How to Use

### 1. Knowledge files

Reference these when setting up CLAUDE.md for a project or when onboarding a new codebase. They distill the Claude Code documentation into actionable guidelines.

- [knowledge/claude-code-core.md](knowledge/claude-code-core.md) — Memory system, workflows, context management, prompting patterns, permission modes

### 2. Skill files

Skills are `SKILL.md` files placed in `.claude/skills/<name>/SKILL.md` inside a project. Claude Code loads them on-demand when relevant to your prompt, or you can invoke them explicitly with `/skill-name`.

**To use a skill in a project:**

```bash
mkdir -p .claude/skills/playwright
cp /path/to/claude-code-guideline/skills/playwright/SKILL.md .claude/skills/playwright/SKILL.md
```

Or symlink for shared maintenance:

```bash
ln -s /path/to/claude-code-guideline/skills/playwright .claude/skills/playwright
```

### Available skills

| Skill                                    | Description                                                                                                       |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| [playwright](skills/playwright/SKILL.md) | Write, extend, and debug Playwright tests. Covers POM, fixtures, data-driven cases, auth setup, locator strategy. |

---

## Adding a New Skill

1. Create a directory: `skills/<skill-name>/`
2. Create `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: One-sentence description of when Claude should apply this skill.
---

# Skill Title

## Context
What problem this skill solves.

## Patterns
The key patterns, conventions, and code examples.

## Checklist
Step-by-step workflow for common tasks in this domain.
```

Keep skills **specific and verifiable**. Include real code examples from the project. Avoid general advice that Claude already knows.

---

## Core Principles (from Claude Code docs)

1. **Context window is the constraint** — everything degrades when it fills
2. **CLAUDE.md under 200 lines** — longer files cause rules to be ignored
3. **Explore → Plan → Implement → Commit** — never code without understanding first
4. **Always give Claude a way to verify** — tests, screenshots, expected outputs
5. **Skills over CLAUDE.md** — domain knowledge loads on demand, not every session
6. **Subagents for research** — they run in separate context, keeping yours clean
7. **/clear between unrelated tasks** — stale context causes mistakes
