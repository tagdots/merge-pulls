# Merge-Pulls

[![CI](https://github.com/tagdots/merge-pulls/actions/workflows/ci.yaml/badge.svg)](https://github.com/tagdots/merge-pulls/actions/workflows/ci.yaml)
[![marketplace](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/tagdots/merge-pulls/refs/heads/badges/badges/marketplace.json)](https://github.com/marketplace/actions/merge-pulls)
[![coverage](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/tagdots/merge-pulls/refs/heads/badges/badges/coverage.json)](https://github.com/tagdots/merge-pulls/actions/workflows/cron-tasks.yaml)

#### Automate your pull request merging process with key featues below

- Multi-Repository Support (_Process PRs across accessible repositories, and organizations(**1**)_)
- Filtering Capabilities (_Filter PRs by base branch, ex/include labels(**2**), title prefix, and repo owner_)
- Merge Readiness Evaluation (_Check CI status, and code review approvals before merging_)
- Flexible Merge Methods (_Support merge, rebase, and squash (**3**)_)
- Bypass Option (_Bypass required review count (**4**)_)
- Dry-Run Mode (_Preview which PRs would be merged without performing the actual merge_)

> **Note**
>
> 1. _Multiple organizations access requires Classic Personal Access Token._
> 1. _In filtering PRs, `exclude-labels` is applied first, followed by `include-labels`._
> 1. _Review repository | settings | pull requests on allowed PR merge strategies._
> 1. _The token owner must be allowed to bypass rule to merge PR._

<br>

### 📋 Prerequisites

Set up a GitHub personal access token (PAT) with the required permissions.

| Token Type       | Required Permissions                                     | Notes                                 |
| ---------------- | -------------------------------------------------------- | ------------------------------------- |
| Classic PAT      | `repo` scope (full)                                      | Supports multi-organization access    |
| Fine-grained PAT | `Repository`: Read & Write on Contents and Pull Requests | Limited to single organization access |

> **Note**
>
> 1. _If you are using GitHub Actions, the default `GITHUB_TOKEN` lacks the admin permissions needed to read branch protection settings._<br>
> 1. _GitHub Apps do not support GitHub REST API endpoints /users and /user/repos. Thus, we only support the use of classic and fine-grained perosnal access token_.

<br>

### 🔧 Configuration Options

<br>

| Option                | Description                                           | Default |
| --------------------- | ----------------------------------------------------- | ------- |
| `base-branch`         | merge the head branch to the base branch              | `main`  |
| `bypass-review-count` | bypass required review count rule                     | `false` |
| `dry-run`             | preview (true); merge (false)                         | `true`  |
| `exclude-labels`      | filter out PRs from processing (comma/space seprated) | `""`    |
| `include-labels`      | filter in PRs to process (comma/space seprated)       | `""`    |
| `merge-method`        | merge strategy: merge, rebase, or squash              | `merge` |
| `owner`               | filter repos to specific user/org                     | `""`    |
| `prefix`              | filter PRs with title prefix                          | `""`    |
| `repo-type`           | filter repos type from all, private, or public        | `all`   |

<br>

---

### 👥 Usage For GitHub Actions Users

Use this tool as a GitHub Action in your workflow file. An example workflow below:

- Runs `on schedule` at `3:30 AM UTC (Monday-Friday)` or `on demand`.
- Configures `merge-pulls` to:
  - `bypass-review-count` - bypass required review count.
  - `dry-run` - perform actual merge.
  - `exclude-labels` - filter out PRs with `stale` or `test` labels.
  - `include-labels` - filter in PRs with `dependencies` labels.
  - use **default** values for the rest of the options.

```
---
name: merge-pulls

on:
  schedule:
    - cron: "30 3 * * 1-5"

  workflow_dispatch:

permissions:
  contents: read
  pull-requests: read

jobs:
  merge-pulls:
    runs-on: ubuntu-latest

    permissions:
      contents: write
      pull-requests: write

    steps:
      - name: Run merge-pulls # https://github.com/marketplace/actions/merge-pulls
        uses: tagdots/merge-pulls@<commit sha> # <version>
        env:
          GH_TOKEN: ${{ secrets.SECRET_GITHUB_PAT }}
        with:
          # override default
          bypass-review-count: true
          dry-run: false
          exclude-labels: "stale,test"
          include-labels: "dependencies"
          # # defaults
          # base-branch: "main"
          # merge-method: "merge"
          # owner: ""
          # prefix: ""
          # repo-type: "all"
```

<br>

---

### 👨‍💻 Usage For CLI Users

Use this as CLI tool in a Python virtual environment. An example below:

- Installs `merge-pulls`.
- Exports `GH_TOKEN` environment variable (classic PAT).
- Runs `merge-pulls` on pull requests that has `dependencies` or `merge-ready` label.
- Merges 4 of 6 open pull requests.

```
1. ~/work/<your project> $ uv pip install -U merge-pulls
2. ~/work/<your project> $ export GH_TOKEN="ghp_xxxxxxxxxxxxx"
3. ~/work/<your project> $ merge-pulls --include-labels "merge-ready,dependencies" --dry-run false
🚀 Starting Merge-Pulls (1.0.0)

📚 Loading configuration from: ./config/default.yaml
📚 Configuration (merged YAML and CLI):
   base-branch: main
   bypass-review-count: False
   dry-run: False
   exclude-labels: set()
   include-labels: { 'dependencies', 'merge-ready' }
   merge-method: merge
   owner:
   prefix:
   repo-type: all

✅ Token Owner User Information :: login = Jack Nolan, Org. Access = ['org1', 'org2']

✅ Finding Open Pull Request...
   ▪ PR: https://github.com/org1/helloworld/pulls/101 (title: [GITHUB-ACTIONS] Bump the github-actions group with 7 updates)
   ▪ PR: https://github.com/org1/dotfiles/pulls/34 (title: [UV] Bump the uv group with 2 updates)
   ▪ PR: https://github.com/org1/dotfiles/pulls/35 (title: fix: memory leak caused by loop)
   ▪ PR: https://github.com/org2/rainer/pulls/21 (title: feat: add new CLI option)
   ▪ PR: https://github.com/org2/tac/pulls/2 (title: [PIP] Bump the pip group with 1 updates)
   ▪ PR: https://github.com/org2/tac/pulls/3 (title: refactor: generator expression)
   Total Number of Open PR :: 6

✅ Finding PR Ready For Merge...
   ▪ Repo: org1/dotfiles (PR #34) -> Mergeable: True, Rebaseable: True, CI status checks: ok-for-merge, Approval Review checks: ok-for-merge
   ▪ Repo: org1/dotfiles (PR #35) -> Mergeable: True, Rebaseable: True, CI status checks: ok-for-merge, Approval Review checks: ok-for-merge
   ▪ Repo: org2/rainer (PR #21) -> Mergeable: True, Rebaseable: True, CI status checks: ok-for-merge, Approval Review checks: ok-for-merge
   ▪ Repo: org2/tac (PR #2) -> Mergeable: True, Rebaseable: False, CI status checks: ok-for-merge, Approval Review checks: ok-for-merge
   Total Number of Mergeable PR :: 4

✅ Final Merged Open PR Info...
   ▪ PR: https://github.com/org1/dotfiles/pulls/34 (title: [UV] Bump the uv group with 2 updates)
   ▪ PR: https://github.com/org1/dotfiles/pulls/35 (title: fix: memory leak caused by loop)
   ▪ PR: https://github.com/org2/rainer/pulls/21 (title: feat: add new CLI option)
   ▪ PR: https://github.com/org2/tac/pulls/2 (title: [PIP] Bump the pip group with 1 updates)
   Total Number of Merged PR :: 4

🎉 Merge-Pulls Summary :: {'open-prs': 6, 'mergeable-prs': 4, 'merged-prs': 4}
```

<br>

---

## 😕 Troubleshooting

Open an [issue][issues]

<br>

## Contributing

See [Contributing][contributing]

<br>

## License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/tagdots/merge-pulls/blob/main/LICENSE) file for details.

<br>

## 🙌 Appreciation

If you find this project helpful, please ⭐ star it. **Thank you**.

<br>

## References

[GitHub REST API](https://docs.github.com/en/rest?apiVersion=2026-03-10)

[PyGithub User Authentication](https://pygithub.readthedocs.io/en/stable/examples/Authentication.html)

[User access token by GitHub App => 403 ](https://github.com/actions/create-github-app-token/issues/258)

<br>

[contributing]: https://github.com/tagdots/merge-pulls/blob/main/CONTRIBUTING.md
[issues]: https://github.com/tagdots/merge-pulls/issues
