# Backport Workflow Module

This repository contains a reusable GitHub Actions workflow for automatically backporting pull requests to other branches.

## Features

- 🏷️ Trigger backports by adding labels (e.g., `backport production`)
- 🔄 Support multiple labels on the same PR
- 💬 Automatic comment with instructions when conflicts occur
- ✨ Provides ready-to-use commands with actual branch names
- 🚀 Easy to use across multiple repositories

## Usage

### In Other Repositories

To use this backport workflow in any repository, create `.github/workflows/backport.yml`:

```yaml
name: Backport merged pull request

on:
  pull_request_target:
    types: [closed, labeled]
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  backport:
    if: github.event.pull_request.merged == true
    uses: humanmade/backport/.github/workflows/backport-module.yml@app
```

### Customizing the Label Pattern

If you want to use a different label pattern:

```yaml
jobs:
  backport:
    if: github.event.pull_request.merged == true
    uses: humanmade/backport/.github/workflows/backport-module.yml@app
    with:
      label_pattern: '^cherry-pick-to ([^ ]+)$'
```

## How to Backport a PR

1. Create a pull request targeting your main branch
2. Add a label like `backport production` or `backport staging` to specify the target branch
3. Merge the pull request
4. The backport action will automatically:
   - Create a new PR with the changes cherry-picked to the target branch
   - Or, if conflicts occur, post detailed instructions for manual resolution

## Multiple Backports

You can add multiple backport labels (e.g., `backport production` and `backport staging`) and the action will create separate PRs for each target branch.

## Conflict Resolution

When conflicts occur, the action posts two comments:

1. **Initial instructions** from the backport action with the commands to start resolving
2. **Continuation instructions** with the exact branch names and a ready-to-use `gh` command

Example continuation instructions:
```bash
# Add the resolved files
git add .
# Continue the cherry-pick
git cherry-pick --continue
# Push the branch to GitHub
git push --set-upstream origin backport-18-to-prod
# Go back to the original working tree
cd ../..
# Delete the working tree
git worktree remove .worktree/backport-18-to-prod
```

Then create the PR:
```bash
gh pr create --base prod --head backport-18-to-prod --title "[Backport prod] Fix auth bug" --body "Backport of #18 to \`prod\`."
```

## Maintenance

All the backport logic is maintained in this central repository. To update all repositories using this workflow:

1. Update `backport-module.yml` in this repo
2. Push to the `app` branch
3. All repositories will automatically use the updated workflow on their next backport

No need to update individual repositories!

## Requirements

- Target branches must exist before backporting
- Repository must have GitHub Actions enabled
- Workflows must have `contents: write` and `pull-requests: write` permissions
