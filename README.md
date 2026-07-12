# QAtlasHub org defaults & reusable workflows

Central home for organization-wide GitHub configuration.

## Inherited automatically (no per-repo action)
- **`.github/FUNDING.yml`** — the Sponsor button on every QAtlasHub repo that
  lacks its own FUNDING.yml.

## Reusable workflows (thin caller per repo)
Reference with the (intentional) double `.github` path, pinned to `@main`:

**Format check** — `.github/workflows/FormatCheck.yml`:
```yaml
name: Format Check
on: { pull_request: { paths: ['**/*.jl'] } }
jobs:
  format: { uses: QAtlasHub/.github/.github/workflows/format-check.yml@main }
```

**Version check** — `.github/workflows/VersionCheck.yml`:
```yaml
name: VersionCheck
on: { pull_request: { branches: [main] } }
jobs:
  version: { uses: QAtlasHub/.github/.github/workflows/version-check.yml@main }
```

**CompatHelper** — `.github/workflows/CompatHelper.yml`:
```yaml
name: CompatHelper
on: { schedule: [{ cron: '0 */4 * * *' }], workflow_dispatch: {} }
jobs:
  compathelper:
    uses: QAtlasHub/.github/.github/workflows/compathelper.yml@main
    secrets: inherit
```

**AutoMerge** (bot PRs + `automerge` label) — `.github/workflows/AutoMerge.yml`:
```yaml
name: AutoMerge
on: { pull_request: { types: [opened, synchronize, reopened, labeled] } }
jobs:
  automerge:
    uses: QAtlasHub/.github/.github/workflows/automerge.yml@main
    secrets: inherit
```

## Per-repo only (cannot be centralised)
- **`dependabot.yml`** — Dependabot config is per-repository; copy this repo's
  `.github/dependabot.yml` into each repo. (There is no org-level default.)

## One-time org / repo settings
- Org → Settings → Actions → **Workflow permissions = Read and write** (done).
- Org → Settings → Actions → **Allow GitHub Actions to create and approve pull requests** (needed for AutoMerge).
- Each repo → Settings → General → **Allow auto-merge**, + branch protection with
  required status checks (so AutoMerge's `--auto` waits for green).
