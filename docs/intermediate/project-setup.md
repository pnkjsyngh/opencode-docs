# Project Setup — Extending /project-setup

> Proposes adding a **Step 3** to the `/project-setup` custom command you created in the
> [beginner guide](../beginner/opencode-starter-setup.md#project-setup).
> This step sets up release-please on every new project.

## Why This Matters

The [beginner `/project-setup`](../beginner/opencode-starter-setup.md#project-setup) initializes a project with `AGENTS.md` and `.opencodeignore`.
That gets OpenCode ready to write code — but it doesn't set up release infrastructure.

Without automated releases, you're left creating GitHub Releases and changelogs by hand.
Adding release-please during project setup means every project ships with versioned releases
from day one, driven by the [conventional commit workflow](git-workflow.md) already documented.

## Proposed Addition

Update your `~/.config/opencode/commands/project-setup.md` to include Step 3.
The full updated command becomes:

```markdown
---
description: Initialize project with AGENTS.md, .opencodeignore, and release-please
---

Do the following three steps in order:

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

Step 3: Detect the default branch name (main or master) — for example, you can try:
`git branch -a | grep -Eo '(main|master)' | head -1`.

Then create .github/workflows/release-please.yml with exactly this content,
replacing <DETECTED_BRANCH> with the detected branch name:

name: release-please
on:
  push:
    branches: [<DETECTED_BRANCH>]
permissions:
  contents: write
  pull-requests: write
jobs:
  release-please:
    runs-on: ubuntu-latest
    env:
      FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          release-type: simple
```

## Post-Setup: One-Time Repo Setting

After `/project-setup` runs, you need to enable one repository setting (this can't be automated from a local command):

1. Go to `https://github.com/<owner>/<repo>/settings/actions`
2. Under **Workflow permissions**, enable:
   - ✅ **Allow GitHub Actions to create and approve pull requests**

Without this, release-please will fail with `Resource not accessible by integration`.

## How Release Versioning Works

Once set up, every PR title determines the version bump.
See the [Git Workflow guide](git-workflow.md#pr-title-determines-the-release-bump) for the full prefix → bump table.
