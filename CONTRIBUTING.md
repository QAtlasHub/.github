# Contributing

1. Open an issue (bug / wish) before large changes.
2. One focused change per PR. Fill in the PR template — the **Proposed Changes**
   and **Usage or Results** sections become the release notes.
3. Write tests that check an *independent* expectation (closed form,
   cross-method, structural law), not tautological self-consistency.
4. Bump `Project.toml` version by exactly one semver step (patch / minor / major).
5. Format with JuliaFormatter v2 (`julia -e 'using JuliaFormatter; format(".")'`).
CI runs format + version + tests; bot PRs (CompatHelper/dependabot) auto-merge on green.
