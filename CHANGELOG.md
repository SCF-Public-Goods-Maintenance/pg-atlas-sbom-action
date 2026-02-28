# Changelog

## 1.0.0 (2026-02-28)


### Bug Fixes

* append route to default api url ([923023c](https://github.com/SCF-Public-Goods-Maintenance/pg-atlas-sbom-action/commit/923023c0b1afbc3d6c34a2b625d812c5bdc0cca4))
* separate base url and endpoint path ([4abc573](https://github.com/SCF-Public-Goods-Maintenance/pg-atlas-sbom-action/commit/4abc573461467267b7a6be691d279a4293a32227))

## Changelog

All notable changes to this project will be documented in this file.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!-- Release notes are generated automatically by release-please. Do not edit by hand. -->

## Pre-release history

### Added

- Initial composite action: fetches SBOM from GitHub Dependency Graph API and submits to PG Atlas via
  GitHub OIDC authentication.
- `api-url` input with production default (current dev API).
- `dry-run` input for testing setup without submitting.
- Clear early-failure messages for missing `id-token: write` permission and disabled dependency
  graph.
- Self-test CI workflow (`.github/workflows/ci.yml`).
