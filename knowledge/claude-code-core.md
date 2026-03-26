# Claude Code — Core Knowledge

Distilled from the official Claude Code documentation (memory, permission-modes, common-workflows, best-practices, skills).

---

## 1. The One Constraint That Drives Everything

Claude's context window fills fast. Every message, every file read, every command output occupies tokens. **When the window fills, performance degrades — Claude starts forgetting earlier instructions and making more mistakes.**

All best practices flow from managing that window efficiently.

---

## 2. Memory System

Claude Code has two complementary memory systems.

### CLAUDE.md (you write it)

| Location                               | Scope                    | Use for                                        |
| -------------------------------------- | ------------------------ | ---------------------------------------------- |
| `/etc/claude-code/CLAUDE.md` (Linux)   | All users in org         | Company coding standards, security policies    |
| `~/.claude/CLAUDE.md`                  | All your projects        | Personal shortcuts, preferences                |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Project (shared via git) | Coding standards, build commands, architecture |
| Child directory `CLAUDE.md`            | Loaded on demand         | Sub-module specific instructions               |

**Rules for writing effective CLAUDE.md:**

- Target **under 200 lines** — longer files cause Claude to ignore rules
- Use markdown headers & bullets — Claude scans structure like a reader
- Write **specific, verifiable** instructions:
  - ✅ `Run npm test before committing`
  - ❌ `Test your changes`
- Check into git so the team benefits; grows in value over time
- If Claude ignores a rule, the file is too long — prune ruthlessly

**What belongs in CLAUDE.md vs what doesn't:**

| Include                                     | Exclude                            |
| ------------------------------------------- | ---------------------------------- |
| Build/test commands Claude can't guess      | Things Claude infers from code     |
| Code style that differs from defaults       | Standard language conventions      |
| Repo etiquette (branch naming, PR format)   | Detailed API docs (link instead)   |
| Architectural decisions specific to project | Long tutorials or explanations     |
| Common gotchas, non-obvious behaviors       | File-by-file codebase descriptions |
| Required env vars                           | "Write clean code" (self-evident)  |

**Importing other files:**

```
# CLAUDE.md
See @README.md for project overview.
- Git workflow: @docs/git-instructions.md
- Personal preferences: @~/.claude/my-overrides.md
```

### .claude/rules/ (structured rules)

For large projects, split instructions into topic-specific files:

```
.claude/
  CLAUDE.md           ← main instructions
  rules/
    code-style.md
    testing.md
    security.md
```

Path-scoped rules only load when Claude works with matching files:

```yaml
---
paths:
  - 'src/api/**/*.ts'
---
# API Development Rules
- All endpoints must include input validation
```

### Auto memory (Claude writes it)

Claude automatically saves build commands, debugging insights, and patterns it discovers. Stored at `~/.claude/projects/<repo>/memory/MEMORY.md`. First 200 lines loaded every session. Run `/memory` to browse or edit.

---

## 3. Skills

Skills follow the [Agent Skills](https://agentskills.io/) open standard. Claude Code extends it with invocation control, subagent execution, and dynamic context injection.

### Storage locations (priority: enterprise > personal > project)

| Level      | Path                               | Scope             |
| ---------- | ---------------------------------- | ----------------- |
| Enterprise | managed settings                   | All users in org  |
| Personal   | `~/.claude/skills/<name>/SKILL.md` | All your projects |
| Project    | `.claude/skills/<name>/SKILL.md`   | This project only |

### Frontmatter fields

```yaml
---
name: skill-name # becomes /slash-command (optional, defaults to dir name)
description: When to use this # RECOMMENDED — Claude uses this to auto-detect
disable-model-invocation: true # Only YOU can invoke (not Claude auto)
user-invocable: false # Only Claude can invoke (hidden from / menu)
allowed-tools: Read, Grep, Glob # Tools Claude can use without asking permission
context: fork # Run in isolated subagent (separate context)
agent: Explore # Which subagent type (Explore|Plan|general-purpose|custom)
effort: high # Effort level override (low|medium|high|max)
argument-hint: '[issue-number]' # Shown in autocomplete
---
```

### Invocation control

| Setting                          | You invoke? | Claude auto-invoke? | When to use                                     |
| -------------------------------- | ----------- | ------------------- | ----------------------------------------------- |
| (default)                        | ✅          | ✅                  | Reference knowledge, conventions                |
| `disable-model-invocation: true` | ✅          | ❌                  | Side-effect actions: deploy, commit, send-slack |
| `user-invocable: false`          | ❌          | ✅                  | Background knowledge, not a user command        |

### Arguments

```yaml
---
name: fix-issue
disable-model-invocation: true
---
Fix GitHub issue $ARGUMENTS.        # $ARGUMENTS = all args

# Or positional:
Migrate $0 from $1 to $2.           # $0 = first arg, $1 = second, etc.
# Also: $ARGUMENTS[0], $ARGUMENTS[1]
```

### Dynamic context injection (!`command`)

Runs shell commands before Claude sees the skill — output replaces the placeholder:

```yaml
---
name: pr-summary
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---
## PR context
- Diff: !`gh pr diff`
- Comments: !`gh pr view --comments`
- Files: !`gh pr diff --name-only`

## Task
Summarize this pull request...
```

### Supporting files

Skills can include more than just `SKILL.md`. Keep `SKILL.md` under 500 lines:

```
.claude/skills/my-skill/
  SKILL.md              ← required, overview + navigation
  reference.md          ← detailed docs, loaded on demand
  examples.md           ← usage examples
  scripts/
    validate.sh         ← scripts Claude can execute
```

Reference from `SKILL.md`:

```markdown
- Full API reference: see [reference.md](reference.md)
- Examples: see [examples.md](examples.md)
```

### Bundled skills (ship with Claude Code)

| Command                     | What it does                                                                                         |
| --------------------------- | ---------------------------------------------------------------------------------------------------- |
| `/batch <instruction>`      | Fan out large-scale changes across codebase in parallel (5–30 agents, each in isolated git worktree) |
| `/simplify [focus]`         | Spawn 3 review agents in parallel, aggregate, then fix code quality issues                           |
| `/debug [description]`      | Enable debug logging + analyze session debug log                                                     |
| `/loop [interval] <prompt>` | Poll/repeat a prompt on a schedule (e.g. `/loop 5m check if deploy finished`)                        |
| `/claude-api`               | Load Claude API reference for your language (auto-activates when code imports `anthropic`)           |

### Troubleshooting skills

| Problem                  | Fix                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------- |
| Skill not triggering     | Check description has keywords users would say. Try `/skill-name` directly.                             |
| Skill triggers too often | Make description more specific. Add `disable-model-invocation: true`.                                   |
| Skills missing           | Many skills exceed the character budget (2% of context window, ~16,000 chars). Run `/context` to check. |

### Key distinction

- **CLAUDE.md / rules/** → loaded every session → persistent standards, build commands
- **Skills** → loaded on demand → domain knowledge, reusable workflows, actions with side effects

---

## 4. The 4-Phase Workflow

```
EXPLORE → PLAN → IMPLEMENT → COMMIT
```

### Phase 1: Explore (Plan Mode — read-only)

```
claude --permission-mode plan

read src/auth/ and understand sessions and login.
look at how secrets are managed.
```

### Phase 2: Plan (Plan Mode — still)

```
I want to add Google OAuth. What files need to change?
What's the session flow? Create a detailed plan.
```

Press `Ctrl+G` to open the plan in your editor and edit before proceeding.

### Phase 3: Implement (Normal Mode)

```
implement the OAuth flow from the plan.
write tests for the callback handler, run the suite and fix failures.
```

### Phase 4: Commit

```
commit with a descriptive message and open a PR
```

**When to skip planning:** task is clear, fix is small (typo, rename, single-line change). If you can describe the diff in one sentence, skip the plan.

---

## 5. Context Management

The context window is the most critical resource.

| Command                 | When to use                                                                            |
| ----------------------- | -------------------------------------------------------------------------------------- |
| `/clear`                | Between unrelated tasks. Resets the full window.                                       |
| `/compact <focus>`      | Compress long sessions while keeping key context. E.g. `/compact Focus on API changes` |
| `Esc + Esc` / `/rewind` | Open rewind menu — restore code/conversation to a checkpoint                           |
| `/btw`                  | Quick side question — answer appears in overlay, never enters context history          |
| Subagents               | Research tasks that read many files — runs in separate context, reports summary back   |

**Common failure patterns:**

1. **Kitchen sink session** — mixing unrelated tasks. Fix: `/clear` between tasks.
2. **Correction loop** — correcting the same issue 3+ times. Fix: `/clear` and rewrite the prompt from scratch incorporating what you learned.
3. **Bloated CLAUDE.md** — rules get lost in noise. Fix: ruthlessly prune.
4. **Trust-then-verify gap** — plausible output that misses edge cases. Fix: always provide tests or a verification command.
5. **Infinite exploration** — "investigate" without scope. Fix: narrow the scope, or use a subagent.

---

## 6. Permission Modes

| Mode                | What Claude can do                  | When to use                         |
| ------------------- | ----------------------------------- | ----------------------------------- |
| `default`           | Read files                          | Sensitive work, getting started     |
| `acceptEdits`       | Read + edit files                   | Iterating on code you're reviewing  |
| `plan`              | Read files only (no edits)          | Exploring, planning complex changes |
| `auto`              | All actions + background classifier | Long tasks, reduce prompt fatigue   |
| `bypassPermissions` | All actions, no checks              | Isolated containers/VMs only        |
| `dontAsk`           | Only pre-approved tools             | Locked-down CI environments         |

**Switch modes:** `Shift+Tab` cycles through modes during a session.

**Start in a specific mode:**

```bash
claude --permission-mode plan
```

**Set as default:**

```json
// .claude/settings.json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

---

## 7. Effective Prompting

### Scope the task

- ✅ `write a test for foo.py covering the edge case where user is logged out. avoid mocks.`
- ❌ `add tests for foo.py`

### Provide verification criteria

- ✅ `implement validateEmail(). test cases: user@example.com → true, invalid → false. run tests after.`
- ❌ `implement a function that validates email`

### Point to sources

- ✅ `look through ExecutionFactory's git history and explain how its API came to be`
- ❌ `why does ExecutionFactory have such a weird API?`

### Reference patterns

- ✅ `look at HotDogWidget.php to understand the pattern. implement a CalendarWidget following the same pattern without extra libraries.`
- ❌ `add a calendar widget`

### Describe the symptom with location

- ✅ `users report login fails after session timeout. check auth flow in src/auth/, especially token refresh. write a failing test first, then fix it.`
- ❌ `fix the login bug`

### Provide rich content

- Use `@file.ts` to reference files — Claude reads before responding
- Paste screenshots/images directly
- Pipe data: `cat error.log | claude`
- Let Claude fetch context: `use Bash commands to gather context`

### Have Claude interview you for large features

```
I want to build [brief description].
Interview me in detail using the AskUserQuestion tool.
Ask about technical implementation, UI/UX, edge cases, tradeoffs.
Don't ask obvious questions — dig into the hard parts.
Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

---

## 8. Subagents

Subagents run in separate context windows and report back summaries. Use them to:

- Investigate codebases (keeps main context clean)
- Review code from a fresh perspective
- Fan out parallel work

```
use subagents to investigate how our authentication system handles token refresh,
and whether we have any existing OAuth utilities I should reuse.
```

**Writer/Reviewer pattern:**

1. Session A: `implement a rate limiter for our API endpoints`
2. Session B: `review the rate limiter in @src/middleware/rateLimiter.ts. look for edge cases, race conditions, and consistency with existing patterns.`
3. Session A: `here's the review feedback: [paste]. address these issues.`

---

## 9. Session Management

```bash
claude --continue         # Resume most recent conversation
claude --resume           # Open picker to select session
claude -n auth-refactor   # Start session with name
/rename auth-refactor     # Rename current session
```

Every Claude action creates a checkpoint. `Esc + Esc` → `/rewind` to restore.

---

## 10. Automation & Scale

**Non-interactive (CI, scripts):**

```bash
claude -p "your prompt" --output-format json
claude -p "fix all lint errors" --permission-mode auto
```

**Fan out across files:**

```bash
for file in $(cat files.txt); do
  claude -p "migrate $file from React to Vue. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

**Verify with Claude in CI:**

```json
// package.json
{
  "scripts": {
    "lint:claude": "claude -p 'look at changes vs. main. report typos: filename + line number on one line, description on next. no other text.'"
  }
}
```

---

## 11. Hooks

Hooks run scripts automatically at specific points in Claude's workflow. Unlike CLAUDE.md (advisory), hooks are **deterministic** — they always execute.

### Hook events

| Event                               | When it fires                                                        |
| ----------------------------------- | -------------------------------------------------------------------- |
| `PreToolUse`                        | Before every tool call — can allow/deny/escalate                     |
| `PostToolUse`                       | After every tool call                                                |
| `Notification`                      | Claude needs attention (idle, permission prompt, auth)               |
| `InstructionsLoaded`                | When CLAUDE.md and rules files are loaded (debug which files loaded) |
| `WorktreeCreate` / `WorktreeRemove` | Custom worktree logic for non-git VCS                                |

### Configure in `.claude/settings.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [{ "type": "command", "command": "npm run lint -- $FILE" }]
      }
    ],
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Needs your attention'"
          }
        ]
      }
    ]
  }
}
```

**Notification matcher values:** `permission_prompt` | `idle_prompt` | `auth_success` | `elicitation_dialog`

### Use cases

- `PostToolUse` on `Edit` → run eslint after every file edit
- `PreToolUse` → block writes to migrations folder
- `Notification` on `idle_prompt` → desktop notification when Claude finishes

Ask Claude to write hooks for you: _"Write a hook that runs eslint after every file edit"_

---

## 12. Subagents

Subagents run in isolated contexts with their own tool sets.

### Where they're defined

```
.claude/agents/
  security-reviewer.md
  test-writer.md
```

### Frontmatter

```yaml
---
name: security-reviewer
description: Reviews code for security vulnerabilities. Use when checking auth, APIs, or data handling.
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer. Review for:
  - Injection vulnerabilities (SQL, XSS, command injection)
  - Auth and authz flaws
  - Secrets or credentials in code
```

### Key fields

| Field       | Options                           | Purpose                               |
| ----------- | --------------------------------- | ------------------------------------- |
| `tools`     | `Read, Grep, Glob, Bash, Edit...` | Restrict to only what the agent needs |
| `model`     | `opus`, `sonnet`, `haiku`         | Different model per agent             |
| `isolation` | `worktree`                        | Agent gets its own git worktree       |

### How to use

```
# Automatic delegation
use a subagent to review this code for edge cases

# Explicit
use the security-reviewer subagent to check the auth module

# Available built-in agents
Explore   ← read-only, fast codebase research
Plan      ← planning only, no edits
general-purpose ← default
```

### Subagent + skill (`context: fork`)

Skill with `context: fork` becomes a subagent task — the SKILL.md content is the prompt:

```yaml
---
name: deep-research
context: fork
agent: Explore # or any custom agent from .claude/agents/
---
Research $ARGUMENTS thoroughly.
Find relevant files, read code, summarize with file references.
```

---

## 13. MCP (Model Context Protocol)

Connect external tools so Claude can query databases, read Figma, search GitHub issues, etc.

```bash
# Add a server interactively
claude mcp add

# Common MCP servers
claude mcp add github    # GitHub repos, issues, PRs
claude mcp add notion    # Notion pages
claude mcp add postgres  # Query databases
```

Once connected, reference MCP resources with `@`:

```
Show me the data from @github:repos/owner/repo/issues
```

Use `/permissions` to allowlist frequently used MCP domains.

---

## 14. Project Setup Checklist

When starting on a new project with Claude Code:

```
1. Run /init                 ← generates starter CLAUDE.md from codebase analysis
2. Refine CLAUDE.md          ← add what /init missed (gotchas, env vars, test commands)
3. Create .claude/rules/     ← split large CLAUDE.md into topic files
4. Add skills                ← domain knowledge + repeatable workflows
5. Create agents             ← specialized subagents for security, testing, etc.
6. Configure hooks           ← automate lint/format after edits
7. Connect MCP servers       ← external tools (GitHub, Figma, DB)
8. Set permission mode       ← defaultMode in .claude/settings.json
```
