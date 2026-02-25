# Changelog

All notable changes to this project will be documented in this file.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Initial composite action: fetches SBOM from GitHub Dependency Graph API and submits to PG Atlas via
  GitHub OIDC authentication.
- `api-url` input with production default (HTTPbin for now, lol).
- `dry-run` input for testing setup without submitting.
- Clear early-failure messages for missing `id-token: write` permission and disabled dependency
  graph.
- Self-test CI workflow (`.github/workflows/ci.yml`).
