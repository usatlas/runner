# 🛠️ Alma9 GitHub Actions Runner

This repository manages the list of GitHub Actions **self-hosted runner
versions** used by the USATLAS organization. It includes an automated workflow
that periodically checks for new runner releases and opens a pull request to
update the pinned versions.

## 🔧 Configuration

The `.env` file stores the pinned versions used in the docker image, for
example:

```
RUNNER_VERSION=2.329.0
RUNNER_CONTAINER_HOOKS_VERSION=0.7.0
```

These values are automatically updated by the automated version update workflow.

## 🐳 Docker Image Publishing

Docker images for **Alma9-based GitHub Actions self-hosted runners** are
automatically built and published when changes are pushed to the `main` branch.
These provide an alternative to the standard Ubuntu/Debian-based runners. Pull
requests trigger test builds to validate the Dockerfile, but images are only
published from `main`.

### Published Registries

Images are published to two container registries:

- **CERN GitLab Container Registry**: `gitlab-registry.cern.ch/usatlas/runner`
- **GitHub Container Registry**: `ghcr.io/usatlas/actions-runner`

### Available Tags

Multiple tags are created for each build to support different use cases:

- **`latest`** - Always points to the most recent build from `main`
- **`sha-{hash}`** - Immutable reference to a specific commit (e.g.,
  `sha-592ad7bd`)
- **`{RUNNER_VERSION}-latest`** - Latest build for a specific runner version
  (e.g., `2.329.0-latest`)
- **`{RUNNER_VERSION}-{HOOKS_VERSION}`** - Fully pinned, reproducible version
  (e.g., `2.329.0-0.8.0`)

### Versioning Concepts

The Docker images use two independent version numbers:

- **Runner Version** (`RUNNER_VERSION`): The version of the GitHub Actions
  runner software itself, from
  [actions/runner](https://github.com/actions/runner/releases)
- **Runner Container Hooks Version** (`RUNNER_CONTAINER_HOOKS_VERSION`): The
  version of container hooks for Kubernetes integration, from
  [actions/runner-container-hooks](https://github.com/actions/runner-container-hooks/releases)

Both versions are stored in the `.env` file and automatically updated by the
version update workflow.

### Usage Examples

Pull the latest image:

```bash
docker pull ghcr.io/usatlas/actions-runner:latest
```

Pull a specific pinned version:

```bash
docker pull ghcr.io/usatlas/actions-runner:2.329.0-0.8.0
```

### Multi-Platform Support

Images are built for both `linux/amd64` and `linux/arm64` architectures.

## 🚀 Automated Version Updates

The regularly-scheduled GitHub Actions workflow, powered by `gh` CLI for
interacting with GitHub's API, runs the following steps:

1. **Fetch latest runner releases** from the official GitHub runners repository.
2. **Compare versions** against what is currently stored in `.env`.
3. If any versions have changed:
   - A new branch named `update-runner-versions` is created.
   - `.env` is updated with the new versions.
   - A commit is created with a message summarizing exactly what changed.
   - A pull request is automatically opened.

### 📝 Example Commit Message

When versions change, the commit message looks like:

```
Update GitHub Actions runner versions

- Container Hooks: [0.7.0](https://github.com/actions/runner-container-hooks/releases/tag/v0.7.0) → [0.8.0](https://github.com/actions/runner-container-hooks/releases/tag/v0.8.0)
```

Commit messages only include entries for components that actually changed.
