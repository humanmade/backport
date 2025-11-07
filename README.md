# Backport Workflow Module

A reusable GitHub Actions workflow for automatically backporting pull requests to other branches.

## Features

- 🏷️ **Label-based triggers** - Simply add a `backport <branch>` label to any PR
- 🔄 **Multiple backports** - Add multiple labels to backport to several branches at once
- 💬 **Smart conflict handling** - Automatic comments with detailed resolution instructions
- ✨ **Ready-to-use commands** - Generated commands include actual branch names
- 🚀 **Centralized maintenance** - Update once, applies to all repositories
- 🛠️ **GitHub CLI integration** - Includes `gh` commands for easy PR creation

## Quick Start

### Add to Any Repository

Create `.github/workflows/backport.yml` in your repository:

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

That's it! The workflow is now active.

## Usage

1. **Create a PR** targeting your main branch
2. **Add a label** like `backport production` or `backport staging`
3. **Merge the PR** - backport happens automatically

### Multiple Backports

Add multiple labels to backport to several branches:
- `backport production`
- `backport staging`

Both will be processed automatically.

## Conflict Resolution

When conflicts occur, you'll receive detailed instructions with actual branch names:

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

Plus a ready-to-use GitHub CLI command:
```bash
gh pr create --base prod --head backport-18-to-prod --title "[Backport prod] Fix auth bug" --body "Backport of #18 to \`prod\`."
```

## Customization

### Custom Label Pattern

If you prefer different label naming:

```yaml
jobs:
  backport:
    if: github.event.pull_request.merged == true
    uses: humanmade/backport/.github/workflows/backport-module.yml@app
    with:
      label_pattern: '^cherry-pick-to ([^ ]+)$'
```

## Requirements

- Target branches must exist before backporting
- GitHub Actions must be enabled
- Repository needs `contents: write` and `pull-requests: write` permissions

## Maintenance

All backport logic is maintained in this repository. Updates are automatic:

1. Update `backport-module.yml` in this repo
2. Push to the `app` branch
3. All repositories automatically use the new version

No need to update individual repositories!

## Documentation

See [.github/workflows/README.md](.github/workflows/README.md) for detailed documentation.

## Dependencies

This workflow uses the following external GitHub Actions:

- **[actions/checkout@v4](https://github.com/actions/checkout)** - Checks out the repository code
- **[korthout/backport-action@v3](https://github.com/korthout/backport-action)** - Core backport functionality for cherry-picking commits
- **[actions/github-script@v7](https://github.com/actions/github-script)** - Enables custom JavaScript to interact with GitHub API for posting conflict resolution instructions

## License

MIT License - see [LICENSE](LICENSE) file for details.

Copyright (c) 2025 Human Made
