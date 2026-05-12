# Git Workflow

> Proposes adding a **Feature Branch Workflow** (section `## 5`) to the AGENTS.md you created in the
> [beginner guide](../beginner/opencode-starter-setup.md#1-karpathys-agent-principles).

## Why This Matters

The [beginner AGENTS.md](../beginner/opencode-starter-setup.md#1-karpathys-agent-principles) covers principles 1–4: thinking before coding, keeping it simple, surgical edits, and goal-driven execution.
Sections 1–4 teach OpenCode _how_ to write code.

This section adds principle 5 — teaching OpenCode _how_ to commit and ship. Without it, the agent will:

- Commit directly to `main`
- Use random branch names
- Never open pull requests
- Mangle release versioning

## Proposed Addition

Add the following as `## 5` in your `~/.config/opencode/AGENTS.md`, right after section 4 (Goal-Driven Execution):

```markdown
## 5. Feature Branch Workflow

**Never commit directly to `main`. Always use a feature branch.**

### Branch + PR lifecycle

1. **Create branch** — Named with the conventional commit prefix:
   - `feat/<description>` for new features
   - `fix/<description>` for bug fixes
   - `chore/<description>` for maintenance
2. **Develop** — Make changes on the branch. Commit messages use conventional format.
3. **Open the PR** — Push the branch and create a pull request.
4. **Wait** — Stop and wait for the user to review and squash-merge. Do not merge yourself.

### When to open the PR (judgment call)

Open a PR when the work is **done and self-contained**:

- **One logical change per PR.** Do not batch unrelated changes into one PR. Do not split one logical change across multiple PRs.
- **Ready means:** all tests pass locally, no WIP or TODO stubs remain, the change does what was asked and nothing more.
- **If one task requires two related fixes** (e.g. "fix: correct date parsing" + "fix: update tests for the same parser"), put them in the same PR with a single `fix:` title.
- **If two tasks are independent** (e.g. `feat: add charts` + `fix: login redirect`), open separate PRs so each gets its own release line.
- **If the user asks for an unrelated change while work is in progress**, open a PR for what you have, then start a fresh branch for the new request.
- **When unsure whether work is complete**, ask the user: "Is this ready for review, or should I continue adding X?"

### What to squash within one PR

The branch may accumulate commits of different types. Squash them all into one PR — the PR title uses the **highest-impact prefix** found on the branch.

| Commits on branch | PR title becomes |
|---|---|
| `fix: A` + `docs: B` | `fix: A` |
| `feat: C` + `chore: D` + `docs: E` | `feat: C` |
| `docs: F` + `ci: G` (no feat/fix) | `docs: F` |

**Split rule:** When a `feat:` or `fix:` commit arrives that represents a **different logical change** from what the branch was created for, open the current work as a PR and start a fresh branch.

### PR title determines the release bump

The PR title becomes the commit message on `main` after squash-merge. Release-please reads it to decide the version:

| PR title prefix | Version impact |
|---|---|
| `feat:` | Minor bump — `1.1.0` → `1.2.0` |
| `fix:` | Patch bump — `1.1.0` → `1.1.1` |
| `chore:`, `docs:`, `ci:` | No bump |

### Exception

Skip the branch workflow (e.g. edit directly on `main`) only when the user explicitly says to. Never decide this yourself.
```

## How It Works

| Concept | What OpenCode learns |
|---|---|
| Branch naming | Branches use `feat/`, `fix/`, or `chore/` prefixes |
| No direct-to-main | Agent will never commit to `main` unless told to |
| PR lifecycle | Agent opens a PR when work is done, then waits for review |
| Squash rules | PR title uses the highest-impact prefix from branch commits |
| Release bumps | `feat:` triggers a minor bump, `fix:` triggers a patch bump |
