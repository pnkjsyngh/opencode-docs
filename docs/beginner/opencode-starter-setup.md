# OpenCode Starter Setup
> A opinionated guide to setting up OpenCode for Python and TypeScript development.

This repo documents a production-ready OpenCode configuration — covering code intelligence, formatting, permissions, session continuity, MCP integrations, and model selection. Fork it, adapt it to your stack, and use it as your baseline for every new project.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Global Configuration](#global-configuration)
  - [Karpathy's Agent Principles](#1-karpathys-agent-principles)
  - [opencode.json](#2-opencodejson)
- [LSP Setup](#lsp-setup)
- [Formatter Integration](#formatter-integration)
- [Permissions](#permissions)
- [Custom Commands](#custom-commands)
  - [/project-setup](#project-setup)
  - [/test](#test)
  - [/review](#review)
  - [/prime-context](#prime-context)
  - [/handoff](#handoff)
- [MCP Servers](#mcp-servers)
- [Model Pinning](#model-pinning)
- [Session Workflow](#session-workflow)
- [Future Enhancements](#future-enhancements)

---

## Prerequisites

```bash
# Python tooling
pip install python-lsp-server black isort

# TypeScript tooling
npm install -g typescript-language-server typescript prettier
```

---

## Project Structure

```
~/.config/opencode/
  AGENTS.md              ← global agent behaviour principles
  opencode.json          ← global config (LSP, formatters, MCP, models)
  commands/
    project-setup.md     ← initialize a new project
    test.md              ← generate pytest test files
    review.md            ← structured code review
    prime-context.md     ← load session context
    handoff.md           ← save session summary

your-project/
  AGENTS.md              ← project-specific conventions (generated)
  .opencodeignore        ← files to exclude from agent context (generated)
  .opencode/
    session-handoff.md   ← continuity between sessions (generated)
```

---

## Global Configuration

### 1. Karpathy's Agent Principles

Create `~/.config/opencode/AGENTS.md` with the following. These four principles address the most common LLM failure modes and apply universally across all projects.

> **Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

```markdown
# Global Agent Guidelines

## 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

## 4. Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

> Original source: [Andrej Karpathy's CLAUDE.md](https://github.com/forrestchang/andrej-karpathy-skills/blob/main/CLAUDE.md)

---

### 2. opencode.json

Create `~/.config/opencode/opencode.json`. This is your complete global config — copy it and replace the placeholder values.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "lsp": true,
  "model": "anthropic/claude-sonnet-4-6",
  "permission": {
    "bash": {
      "*": "ask",
      "git *": "allow",
      "grep *": "allow",
      "pytest *": "allow",
      "black *": "allow",
      "isort *": "allow",
      "rm *": "ask"
    }
  },
  "formatter": {
    "black": {
      "command": ["black", "$FILE"],
      "extensions": [".py"]
    },
    "isort": {
      "command": ["isort", "$FILE"],
      "extensions": [".py"]
    },
    "prettier": {
      "command": ["npx", "prettier", "--write", "$FILE"],
      "extensions": [".ts", ".tsx", ".js", ".jsx"]
    }
  },
  "mcp": {
    "github": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
      }
    },
    "filesystem": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-filesystem", "/home"]
    }
  },
  "agent": {
    "explore": { "disable": true },
    "general": { "disable": true },
    "build": { "model": "opencode/minimax-m2.5-free" },
    "plan":  { "model": "opencode/nemotron-3-super-free" }
  }
}
```

---

## LSP Setup

OpenCode has built-in LSP support for Python and TypeScript. Setting `"lsp": true` in your config is all that's needed — OpenCode automatically spins up the right language server based on your file types.

**What LSP gives you:**

- Type-aware code intelligence — the agent understands types, not just text
- Real-time diagnostics — type errors, unused imports, undefined references
- Precise edits — the agent knows if a change breaks something before saving

**Verify it's working:** Open a Python file and introduce a deliberate type error. Ask OpenCode to check for type issues — it should catch the error with the exact line number and argument context.

---

## Formatter Integration

Formatters run automatically on every file OpenCode edits. You never need to invoke them manually.

| Language | Formatter | What it does |
|----------|-----------|--------------|
| Python | `black` | Code formatting |
| Python | `isort` | Import sorting |
| TypeScript/JS | `prettier` | Code formatting |

The `formatter` block in `opencode.json` wires these up. Every AI-edited file comes out clean regardless of what you asked it to do.

---

## Permissions

The permission system controls which actions require your approval before running.

**Philosophy: Maximum Autonomy with Visibility**

- Routine commands (`git`, `pytest`, formatters) run silently — no friction
- Unfamiliar or destructive commands (`rm`) pause for your approval
- You see exactly what the agent wants to run before it executes

**Three permission levels:**
- `allow` — runs immediately, no interruption
- `ask` — pauses and shows you the command, waits for approval
- `deny` — blocked entirely

---

## Custom Commands

Commands live in `~/.config/opencode/commands/` as Markdown files. Invoke with `/commandname`.

Use `!` before a shell command to execute it and inject the output into the prompt automatically.

---

### /project-setup

Initializes a new project. Run this instead of `/init` for every new project.

**What it does:**
1. Runs `/init` to generate a project-specific `AGENTS.md`
2. Creates `.opencodeignore` with sensible defaults for Python/TypeScript

**Create** `~/.config/opencode/commands/project-setup.md`:

```markdown
---
description: Initialize project with AGENTS.md and .opencodeignore
---

Do the following two steps in order:

Step 1: Run /init to generate AGENTS.md for this project.

Step 2: Create a file called .opencodeignore in the current working directory with exactly this content:

# Python
__pycache__/
*.pyc
*.pyo
.venv/
venv/
.mypy_cache/
.pytest_cache/
*.egg-info/
dist/
build/

# TypeScript / Node
node_modules/
dist/
build/
.next/
*.lock

# General
.git/
.DS_Store
*.log
.env
.env.*

After both steps, confirm the full path of each file created.
```

---

### /test

Generates a complete pytest test file for a Python source file. Follows the one-test-file-per-source-file convention, grouping tests by function using `TestFunctionName` classes.

**Usage:** `/test FILE=src/billing.py`

**Create** `~/.config/opencode/commands/test.md`:

```markdown
---
description: Generate a complete pytest test file for a Python source file
---

!`cat $FILE`

Before writing any tests, do the following checks:

Step 1: Check if the tests/ directory exists. If not, create it.
Step 2: Check if tests/__init__.py exists. If not, create it as an empty file.
Step 3: Generate a complete pytest test file for $FILE.

Follow these conventions:
- Save the output as tests/test_$(basename $FILE)
- Group tests by function using classes named Test<FunctionName>
- For each function include:
  - Happy path cases
  - Edge cases and boundary values
  - Error and exception cases
- Use pytest fixtures for repeated setup
- Add type hints to all test functions
- Import only what is needed from $FILE

After writing the tests:
- Run pytest tests/test_$(basename $FILE) and fix any failures
- Report how many tests passed
```

---

### /review

Structured code review. Read-only — does not modify files.

**Usage:** `/review FILE=auth.py`

**Create** `~/.config/opencode/commands/review.md`:

```markdown
---
description: Review a Python or TypeScript file for issues
---

!`cat $FILE`

Review the code in $FILE for:
- Type safety issues
- Missing error handling
- Performance problems
- PEP 8 violations (Python) or TypeScript strict mode violations
- Missing docstrings or type hints

Provide specific fixes with line numbers. Do not modify the file unless asked.
```

---

### /prime-context

Run at the start of every session. Loads git history, file structure, README, AGENTS.md, and the previous session handoff — so the agent starts fully oriented rather than exploring blindly.

**Create** `~/.config/opencode/commands/prime-context.md`:

```markdown
---
description: Load project context at the start of a session
---

!`git log --oneline -10`
!`find . -name "*.py" -not -path "./.venv/*" -not -path "./__pycache__/*" | head -30`
!`find . -name "*.ts" -not -path "./node_modules/*" | head -20`
!`cat README.md 2>/dev/null || echo "No README.md found"`
!`cat AGENTS.md 2>/dev/null || echo "No AGENTS.md found"`
!`cat .opencode/session-handoff.md 2>/dev/null || echo "No previous session handoff found"`

You now have full context on this project. Summarize:
1. What the project does
2. Key files and structure
3. What was last worked on (from git log and handoff)
4. Suggested next steps

Then ask what we are working on today.
```

---

### /handoff

Run at the end of every session. Saves a structured summary to `.opencode/session-handoff.md` which is picked up automatically by `/prime-context` next session.

**Create** `~/.config/opencode/commands/handoff.md`:

```markdown
---
description: Summarize the session and save a handoff for next time
---

!`git log --oneline -5`
!`git diff --stat HEAD`

Summarize this session and save it to .opencode/session-handoff.md with exactly this structure:

## Last Session Summary
Date: <today's date>

## What We Worked On
<brief description of the task>

## Files Modified
<list of files changed and why>

## Decisions Made
<key decisions and their reasoning>

## What Was Left Incomplete
<anything that was not finished>

## Next Steps
<concrete next actions to take next session>

After saving the file confirm it was written successfully.
```

---

## MCP Servers

MCP (Model Context Protocol) servers extend OpenCode with external tool access. Add them to the `mcp` block in `opencode.json`.

### GitHub MCP

Lets the agent read issues, list repos, create branches, and interact with PRs directly from your session.

**Setup:**

1. Go to `https://github.com/settings/tokens`
2. Generate a classic token with `repo`, `read:org`, `read:user` scopes
3. Add to your `opencode.json` `mcp` block (see complete config above)

### Filesystem MCP

Structured file operations beyond basic read/write — useful for navigating large or complex directory structures.

The `args` path controls which directories the agent can access. Narrow it to your projects folder for tighter control:

```json
"filesystem": {
  "type": "local",
  "command": ["npx", "-y", "@modelcontextprotocol/server-filesystem", "/home/yourname/projects"]
}
```

### Built-in: webfetch + websearch

No configuration needed. OpenCode includes these natively:

- **webfetch** — give the agent a URL and it reads the full page content
- **websearch** — web search via Exa AI, no API key required

**Check all available tools:** type `/mcp` inside OpenCode, or ask the agent "What tools do you have available?"

---

## Model Pinning

Explicitly pinning models prevents OpenCode updates from silently shifting agent behaviour.

```json
{
  "model": "anthropic/claude-sonnet-4-6",
  "agent": {
    "build": { "model": "opencode/minimax-m2.5-free" },
    "plan":  { "model": "opencode/nemotron-3-super-free" }
  }
}
```

| Agent | Model | Reason |
|-------|-------|--------|
| Build | MiniMax M2.5 | 80.2% SWE-bench, strong real-world coding performance |
| Plan | Nemotron 3 Super | 2.2x throughput, fast analysis without file modification |
| Fallback | claude-sonnet-4-6 | Reliable baseline for everything else |

**Find exact model IDs:** run `/models` or `opencode models` in your terminal.

> **Note on free models:** Free tier models (MiniMax M2.5 Free, Nemotron 3 Super Free) log prompts and outputs for model improvement. Avoid submitting proprietary or sensitive code on free tiers.

**Why `explore` and `general` subagents are disabled:**

OpenCode's built-in `explore` and `general` subagents can be invoked automatically by primary agents, increasing token usage and making sessions less predictable. Disabling them keeps a single agent in control — simpler to reason about while getting started. Re-enable `explore` when your projects get large enough that codebase navigation becomes slow.

---

## Session Workflow

```
# Start your day
opencode --continue          # resume last session (or use /sessions to pick one)
/prime-context               # load project context and previous handoff

# Do your work
/test FILE=src/feature.py    # generate tests
/review FILE=src/feature.py  # code review

# End your session
/handoff                     # save summary for next time
```

**Session resumption options:**
- `opencode --continue` — instantly resumes the most recent session
- `/sessions` inside OpenCode — interactive list of all past sessions
- `opencode -s <session-id>` — resume a specific session by ID

---

## Future Enhancements

### Per-Agent AGENTS.md
When your shared `AGENTS.md` starts getting cluttered with "if you're agent X, do Y" conditionals, split into agent-specific files:

```
.opencode/agents/
  testwriter.md    ← pytest only, never touch source files
  reviewer.md      ← read-only, structured feedback format
  builder.md       ← full tool access, implements features
```

### oh-my-opencode-slim
Multi-agent orchestration that routes tasks to specialized agents (Explorer, Librarian, Fixer). Worth adding when you hit context quality issues on large tasks or token costs become a concern.

```bash
bunx oh-my-opencode-slim@latest install
```

### Snapshot Management
OpenCode snapshots file changes per session to enable `/undo`. For large repos this can cause slow indexing — disable if needed:

```json
{ "snapshot": false }
```

Note: disabling means Git becomes your only rollback mechanism.

---

## Contributing

Found a better config pattern? Discovered a useful command? PRs welcome.

---

## References

- [OpenCode Docs](https://opencode.ai/docs)
- [Andrej Karpathy's CLAUDE.md](https://github.com/forrestchang/andrej-karpathy-skills/blob/main/CLAUDE.md)
- [MCP Servers Registry](https://github.com/modelcontextprotocol/servers)
- [OpenCode Zen Models](https://opencode.ai/docs/zen)
