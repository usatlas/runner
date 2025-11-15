# 🛠️ Alma9 GitHub Actions Runner

This repository manages the list of GitHub Actions **self-hosted runner versions** used by the USATLAS organization.
It includes an automated workflow that periodically checks for new runner releases and opens a pull request to update the pinned versions.

Any changes to the versions will build a new docker image deployed to ghcr.io.

## 🚀 How It Works

A scheduled GitHub Actions workflow runs the following process:

1. **Fetch latest runner releases** from the official GitHub runners repository.
2. **Compare versions** against what is currently stored in `.env`.
3. If any versions have changed:

   * A new branch named `update-runner-versions` is created.
   * `.env` is updated with the new versions.
   * A commit is created with a message summarizing exactly what changed.
   * A pull request is automatically opened.

## 📝 Example Commit Message

When versions change, the commit message looks like:

```
Update GitHub Actions runner versions

- Container Hooks: [0.7.0](https://github.com/actions/runner-container-hooks/releases/tag/v0.7.0) → [0.8.0](https://github.com/actions/runner-container-hooks/releases/tag/v0.8.0)
```

Commit messages only include entries for components that actually changed.

## 🔧 Configuration

The `.env` file stores the pinned versions, for example:

```
RUNNER_VERSION=2.329.0
RUNNER_CONTAINER_HOOKS_VERSION=0.7.0
```

These values are automatically updated by the workflow.

## 🤖 Automation Workflow

The regularly-scheduled update job is powered by `gh` CLI (preinstalled on runners) for interacting with GitHub.
