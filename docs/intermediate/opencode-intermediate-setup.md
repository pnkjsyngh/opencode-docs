# OpenCode Intermediate Setup

> Builds on the [beginner setup](../beginner/opencode-starter-setup.md) to add workflow discipline
> and CI/CD infrastructure.

## What You'll Add

Each intermediate guide proposes a concrete addition to your OpenCode configuration:

| Guide | Adds to | What it does |
|-------|---------|-------------|
| [Git Workflow](./git-workflow.md) | `~/.config/opencode/AGENTS.md` (Section 5) | Teaches OpenCode to use feature branches, conventional commits, and PR lifecycle |
| [Project Setup](./project-setup.md) | `~/.config/opencode/commands/project-setup.md` (Step 3) | Adds release-please workflow to every new project |

## Guide Summaries

### Git Workflow

Adds `## 5. Feature Branch Workflow` to your `AGENTS.md`. This teaches OpenCode:
- Never commit directly to `main`
- Use `feat/`, `fix/`, `chore/` branch prefixes
- Open a PR when work is done
- PR titles determine release version bumps

### Project Setup

Adds a Step 3 to the `/project-setup` custom command. This automates:
- Detecting the default branch (`main` or `master`)
- Creating `.github/workflows/release-please.yml`
- Setting up permissions for GitHub Actions to create PRs

## Getting Started

If you've completed the [beginner guide](../beginner/opencode-starter-setup.md), start with either guide — they're independent.
For the full release workflow, follow both: git workflow (commits + PRs) followed by project setup (release automation).
