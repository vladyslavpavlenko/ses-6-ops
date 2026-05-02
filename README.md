# Genesis SES 6.0 Ops Config

Central CI and tooling configuration for Genesis SE School Go projects. Provides:

- GitHub Actions CI workflow (runs automatically in student projects)
- golangci-lint configuration with curated rules for Go best practices

All job definitions live in this repo. When a job is added or updated here, it runs in every project on the next push — no changes needed on your side.

---

## Student setup

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

### 3. Add .ops to .gitignore

```bash
echo ".ops" >> .gitignore
```

### 4. Commit the CI workflow

```bash
git add .github/workflows/ci.yml
git commit -m "chore: add CI"
git push
```

GitHub Actions will now run CI on every push and pull request.

---

## Local development tasks

After setup, the following tasks are available in your project:

| Task | Description |
|---|---|
| `task ops:lint` | Run golangci-lint |
| `task ops:lint:fix` | Run golangci-lint with auto-fix |
| `task ops:lint:install` | Install latest golangci-lint |

---

## Updating

Re-run `task ops:setup` at any time to pull the latest config and workflow updates.
