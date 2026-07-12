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


### Documentation (Documenter.jl)

**Deploy** (live site on push to main) — `.github/workflows/Documentation.yml`:
```yaml
name: Documentation
on: { push: { branches: [main], tags: ['*'] }, workflow_dispatch: {} }
jobs:
  docs:
    uses: QAtlasHub/.github/.github/workflows/docs-deploy.yml@main
    secrets: inherit
    # with: { pre-build: docs/atlas/generate.jl }   # optional pre-make hook
```

**Preview** (PR preview + sticky URL comment) — `.github/workflows/DocsPreview.yml`:
```yaml
name: Docs Preview
on: { pull_request: {} }
jobs:
  preview:
    uses: QAtlasHub/.github/.github/workflows/docs-preview.yml@main
    secrets: inherit
    # with: { preview-base: https://codes.sota-shimozono.com }
```

**Delete preview** (remove previews/PR<N> on close) — `.github/workflows/CleanupPreview.yml`:
```yaml
name: Cleanup Preview
on: { pull_request: { types: [closed] } }
jobs:
  cleanup:
    uses: QAtlasHub/.github/.github/workflows/docs-cleanup-preview.yml@main
```

Inputs: `julia-version` (default `1`), `pre-build` (optional Julia script before `make.jl` — e.g. an atlas generator), `preview-base` (docs host; preview link is `<base>/<repo>/previews/PR<N>/`).


### Julia CI with optional sharding

One `shards` input toggles sharding — `shards: 1` (default) runs a single
unsharded `Pkg.test`; `shards: 16` fans out into 16 parallel shards.

```yaml
name: CI
on: { push: { branches: [main] }, pull_request: {} }
jobs:
  ci:
    uses: QAtlasHub/.github/.github/workflows/julia-ci.yml@main
    with:
      shards: 16          # omit or set 1 for a plain single-job Pkg.test
```

**Sharding contract** (only when `shards > 1`): `test/ci/plan_shards.jl <N> [timings.tsv]`
prints `{"include":[{"sid":"s01","cases":"…"}, …]}`; the suite reads env `CI_CASES`
(`""` = all). LPT planning activates when a `ci-timings` orphan branch exists,
else round-robin. Absent `plan_shards.jl` → safe unsharded fallback.


### Release automation (labeler → drafter → autoregister → tagbot)

The label→version→register→tag→notes chain, all reusable:

```yaml
# PRLabeler.yml
on: { pull_request: { types: [opened, edited, synchronize, reopened] } }
jobs: { label: { uses: QAtlasHub/.github/.github/workflows/labeler.yml@main } }
```
```yaml
# ReleaseDrafter.yml   (repo also ships .github/release-drafter.yml config)
on: { push: { branches: [main] }, pull_request: { types: [opened, reopened, synchronize, edited] } }
jobs: { draft: { uses: QAtlasHub/.github/.github/workflows/release-drafter.yml@main } }
```
```yaml
# AutoRegister.yml
on: { workflow_run: { workflows: ["Release Drafter"], types: [completed], branches: [main] } }
jobs: { register: { uses: QAtlasHub/.github/.github/workflows/autoregister.yml@main } }
```
```yaml
# TagBot.yml
on: { issue_comment: { types: [created] }, workflow_dispatch: { inputs: { lookback: { default: "3" } } } }
jobs: { tagbot: { uses: QAtlasHub/.github/.github/workflows/tagbot.yml@main, secrets: inherit } }
```

- **labeler** feeds `enhancement`/`bug`/… labels from the PR title + Type-of-Change checkboxes → drives the version-resolver and release-notes categories.
- **release-drafter** resolves the next version from labels + keeps a draft (needs the repo's `.github/release-drafter.yml`).
- **autoregister** posts `@JuliaRegistrator register` on a version change (minimal `## Changelog` stub — the rich notes come from the release-notes action).
- **tagbot** tags + a **minimal changelog** (the release-notes action owns the body → single source).

> Formatting is standardised on **format-check** (JuliaFormatter v2 verify), replacing the older FormatFix auto-commit flow.

## Per-repo only (cannot be centralised)
- **`dependabot.yml`** — Dependabot config is per-repository; copy this repo's
  `.github/dependabot.yml` into each repo. (There is no org-level default.)

## One-time org / repo settings
- Org → Settings → Actions → **Workflow permissions = Read and write** (done).
- Org → Settings → Actions → **Allow GitHub Actions to create and approve pull requests** (needed for AutoMerge).
- Each repo → Settings → General → **Allow auto-merge**, + branch protection with
  required status checks (so AutoMerge's `--auto` waits for green).
