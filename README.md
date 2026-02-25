# PG Atlas SBOM Action

This is an early experiment. Take everything you read here with a grain of salt.
It may or may not work as described.

A GitHub Action that fetches your repository's SBOM from the
[GitHub Dependency Graph API](https://docs.github.com/en/rest/dependency-graph/sboms) and submits it
to the [PG Atlas](https://scf-public-goods-maintenance.github.io/pg-atlas) ingestion endpoint.
Authentication is handled via GitHub OIDC — no secrets or API keys need to be configured.

## Usage

```yaml
jobs:
  sbom:
    runs-on: ubuntu-latest
    permissions:
      contents: read   # for GitHub Dependency Graph API
      id-token: write  # for OIDC authentication to PG Atlas
    steps:
      - uses: SCF-Public-Goods-Maintenance/pg-atlas-sbom-action@957702113e3aa1f3d690c2a7b1b11dbc2a55493e
```

That's it. The action uses the default production API URL; no other inputs are required.

### Recommended trigger

```yaml
on:
  push:
    branches: [main]
  release:
    types: [published]
```

Running on push to `main` keeps the dependency graph in PG Atlas up to date as your project evolves.
Running on release ensures a snapshot is recorded at each version boundary.

## Prerequisites

The action fetches the SBOM from GitHub's Dependency Graph API. This requires the dependency graph
feature to be enabled for your repository:

> Insights → Dependency graph → if you don't see it: Enable.

GitHub's dependency graph is populated from your lock files, also known as package manifests (e.g.
`Cargo.lock`, `package-lock.json`, `go.sum`, `requirements.txt`, `poetry.lock`). Ensure at least one
lock file is committed for the SBOM to contain dependency data.

## Inputs

| Input     | Required | Default                      | Description                                                                                                       |
| --------- | -------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `api-url` | No       | Production PG Atlas endpoint | Full URL of the PG Atlas ingestion API. Override to point at a staging environment, or your weekend experiment.   |
| `dry-run` | No       | `false`                      | If `true`, fetch the SBOM and obtain the OIDC token but skip the final submission. Useful for testing your setup. |

## Outputs

| Output      | Description                                                             |
| ----------- | ----------------------------------------------------------------------- |
| `sbom-path` | Path to the fetched SBOM file (`sbom.spdx.json`, SPDX 2.3 JSON format). |

## Authentication

The action uses
[GitHub OIDC](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
to authenticate with the PG Atlas API. A short-lived, signed JWT is issued by GitHub's OIDC provider
and sent as a `Bearer` token. The PG Atlas API verifies the JWT's RS256 signature against GitHub's
public JWKS and checks the `repository` claim to identify the submitting repo.

This means:

- **No secrets to configure.** The only requirement is `id-token: write` in your workflow's
  `permissions` block.
- **No API keys or tokens to rotate.**
- The submission is cryptographically tied to your repository — the API knows which repo submitted
  the SBOM.

### Trust model

OIDC proves the _identity_ of the submitting repository. It does not independently verify the
_content_ of the SBOM: the repository owner/contributor could modify the submitted payload. The
principal mitigations are:

1. The PG Atlas reference graph cross-check flags dependency declarations that diverge significantly
   from the inferred graph, making inflation detectable.
2. All submissions are logged with the `repository` and `actor` (triggering GitHub user) OIDC claims,
   making falsification an attributable and auditable act.
3. Community review and the public leaderboard create social accountability.

Both GitHub-hosted and self-hosted runners are supported; there is no meaningful difference in the
integrity guarantees between the two.

## Troubleshooting

### `id-token: write is not set`

Add `id-token: write` to your job's `permissions` block:

```yaml
permissions:
  contents: read
  id-token: write
```

If your workflow has a top-level `permissions` block, it must also include `id-token: write`.

### `Dependency graph not accessible (403/404)`

The GitHub Dependency Graph API returned 403 or 404 for your repository. Steps to resolve:

1. Ensure the dependency graph is enabled (see [Prerequisites](#prerequisites)).
2. Ensure the workflow has `contents: read` permission (required to call the API).
3. For organization-owned private repos, confirm the org's security settings allow dependency graph
   access.

### SBOM has no packages / very few packages

The dependency graph is populated from lock files. If your SBOM is sparse:

- Commit your lock file (`Cargo.lock`, `package-lock.json`, `go.sum`, etc.) — do not add it to
  `.gitignore`.
- For monorepos, all lock files in any subdirectory are detected automatically.

### `Authentication failed (401)`

The OIDC token was rejected by the PG Atlas API. This should be rare; if it occurs, open an issue in
this repository.

## Local development

```bash
# Validate action.yml syntax
gh act --list

# Dry-run the action against your own repo (requires github CLI and act)
gh act push -W .github/workflows/ci.yml
```

## License

[MIT](LICENSE)

## TODO

- Enforce Conventional Commits once the MVP is tested with external repos
- Check in stellarcarbon/sc-audit which changelog action we use there
