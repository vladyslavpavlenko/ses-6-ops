# Genesis SES 6.0 Ops Config

Central CI and tooling configuration for Genesis SE School Go projects. Provides:

- GitHub Actions CI workflow (runs automatically in projects)
- golangci-lint configuration with curated rules for Go best practices
- CodeRabbit AI code review configuration

All job definitions live in this repo. When a job is added or updated here, it runs in every project on the next push — no changes needed on your side.

## Setup

### Prerequisites

- [Task](https://taskfile.dev/installation/) — task runner
- [golangci-lint](https://golangci-lint.run/welcome/install/) — or install via `task ops:lint:install` after setup

### 1. Add to your Taskfile.yml

If you don't have a `Taskfile.yml` yet, create one in your project root. Add the following:

```yaml
includes:
  ops:
    taskfile: .ops/Taskfile.yml
    optional: true

tasks:
  ops:setup:
    desc: Sync ops config
    cmds:
      - |
        if [ -d .ops ]; then
          git -C .ops pull
        else
          git clone --branch {{.OPS_BRANCH}} https://github.com/vladyslavpavlenko/ses-6-ops.git .ops
        fi
      - mkdir -p .github/workflows
      - cp .ops/workflows/*.yml .github/workflows/
      - cp .ops/workflows/.coderabbit.yaml .coderabbit.yaml
    vars:
      OPS_BRANCH: '{{.OPS_BRANCH | default "main"}}'
```

### 2. Run setup

```bash
task ops:setup
```

This will:
1. Clone this repo into `.ops/` inside your project
2. Copy `ci.yml` into `.github/workflows/`
3. Copy `.coderabbit.yaml` into your project root

### 3. Add .ops to .gitignore

```bash
echo ".ops" >> .gitignore
```

### 4. Commit the CI workflow and CodeRabbit config

```bash
git add .github/workflows/ci.yml .coderabbit.yaml
git commit -m "chore: add CI and CodeRabbit"
git push
```

GitHub Actions will now run CI on every push and pull request.

### 5. Install CodeRabbit on your repository

CodeRabbit works as a GitHub App — install it once per repository:

1. Go to [github.com/apps/coderabbit-ai](https://github.com/apps/coderabbit-ai)
2. Click **Install** and select your repository
3. CodeRabbit will now post AI review comments on every pull request automatically

No additional configuration is needed — `.coderabbit.yaml` is already in your project root.

## CodeRabbit

[CodeRabbit](https://coderabbit.ai) is an AI code reviewer that comments on pull requests. After running `task ops:setup`, you'll find `.coderabbit.yaml` in your project root.

### What the template covers

The distributed `.coderabbit.yaml` is a **template** tuned for Go projects:

- **Tools enabled**: golangci-lint, hadolint, gitleaks, actionlint, yamllint, markdownlint, shellcheck
- **Layer-specific reviews** for `cmd/`, `internal/api/`, `internal/service/`, `internal/repository/`, `internal/client/`
- **PostgreSQL migration** review rules (TIMESTAMPTZ, partial unique indexes, etc.)
- **Test review** rules (testcontainers-go, mocking via interfaces, soft-delete checks)
- **Dockerfile, docker-compose, Makefile, .env.example** path-specific rules
- **Auto-labeling** based on changed paths (database, api, security, infrastructure, etc.)
- **Pre-merge checks** for PR title prefix and description sections

### Customizing the template

The template assumes a **clean architecture** with `internal/api`, `internal/service`, `internal/repository`, `internal/client` directories. If your project uses a different structure:

1. Open `.coderabbit.yaml` in your project root
2. Edit the `path_instructions` blocks to match your directory layout
3. Remove rules that don't apply (e.g., scanner block if you have no background workers)
4. Commit the changes

The reference example tuned for the GitHub Release Notification API project lives in this ops repo as [`.coderabbit.project.yaml`](workflows/.coderabbit.project.yaml) — copy ideas from it.

### Required GitHub setup

CodeRabbit only reviews PRs that have the **`ready-for-review`** label. This avoids burning tokens on work-in-progress PRs.

1. Install the CodeRabbit GitHub App: [github.com/apps/coderabbit-ai](https://github.com/apps/coderabbit-ai)
2. Create the required labels on your repository — easiest via `gh`:

   ```bash
   for label in ready-for-review database api security breaking-change infrastructure tests documentation dependencies; do
     gh label create "$label" 2>/dev/null
   done
   ```

3. When your PR is ready, add the `ready-for-review` label to trigger the review

### Updating

Re-run `task ops:setup` at any time to pull the latest template. **Note**: this will overwrite your customizations in `.coderabbit.yaml`. Keep a local diff or branch if you've edited the template heavily.

## Local development tasks

After setup, the following tasks are available in your project:

| Task | Description |
|---|---|
| `task ops:lint` | Run golangci-lint |
| `task ops:lint:fix` | Run golangci-lint with auto-fix |
| `task ops:lint:install` | Install latest golangci-lint |

## Updating

Re-run `task ops:setup` at any time to pull the latest config and workflow updates.
