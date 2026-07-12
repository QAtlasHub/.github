# QAtlasHub

A self-hosted home for the **QAtlas** atlas family and its supporting
infrastructure — declarative physics knowledge, its independent numerical
oracles, and the reproducible-research tooling around them.

## Packages

- **AbstractQAtlas.jl** — model-independent physics vocabulary + relations (the verify-engine base)
- **QAtlas.jl** — the model atlas: declared values, closed forms, constraint kernel
- **QAtlasRegistry** — cross-validated evidence registry (oracles report here)
- **ClassicalMonteCarlo.jl** — independent classical numerical oracle
- **LatticeCore / Lattice2D / QuasiCrystal** — lattice geometry
- **ParamIO.jl / DataVault.jl** — parameter & data management for compute campaigns
- **Pinax.jl** — figure galleries → self-contained annotated HTML
- **doiget** — OA-first DOI/arXiv fetcher + citation-check action

## Shared CI

This org's [`.github`](https://github.com/QAtlasHub/.github) repo hosts the
reusable workflows and actions every package uses via thin `uses:` callers —
CI (optional sharding), formatting, versioning, CompatHelper/AutoMerge, docs
deploy/preview, rich release notes, and citation checking.
