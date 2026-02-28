# PG Atlas SBOM Action

This is an early experiment. Take everything you read here with a grain of salt. It may or may not
work as described.

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
      contents: read # for GitHub Dependency Graph API
      id-token: write # for OIDC authentication to PG Atlas
    steps:
      - uses: SCF-Public-Goods-Maintenance/pg-atlas-sbom-action@3b49bfea5a8b78d04f00ad7bd633e107241a3485
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

| Input             | Required | Default                      | Description                                                                                                                  |
| ----------------- | -------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `api-url`         | No       | Production PG Atlas endpoint | Base URL of the PG Atlas ingestion API. Override to point at a staging environment, or your weekend experiment.              |
| `submission-path` | No       | `/ingest/sbom`               | API endpoint path appended to `api-url` for SBOM ingestion. Override if the server exposes the endpoint at a different path. |
| `dry-run`         | No       | `false`                      | If `true`, fetch the SBOM and obtain the OIDC token but skip the final submission. Useful for testing your setup.            |

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

### Conventional Commits (pre-commit hook)

Commits must follow [Conventional Commits](https://www.conventionalcommits.org/). Install the git
hook so violations are caught before they reach CI:

```bash
pip install pre-commit  # or: brew install pre-commit
pre-commit install --hook-type commit-msg
```

Releases and `CHANGELOG.md` are managed automatically by
[release-please](https://github.com/googleapis/release-please) on every push to `main`.

### Linting action.yml and workflow files

[actionlint](https://github.com/rhysd/actionlint) validates GitHub Actions syntax well beyond plain
YAML parsing:

```bash
# macOS / Linux with Homebrew
brew install actionlint

# Cross-platform with Go
go install github.com/rhysd/actionlint/cmd/actionlint@latest
```

Then run from the repo root:

```bash
actionlint -verbose
```

### Running the workflow locally with act

[act](https://nektosact.com) lets you run the CI workflow on your machine. The easiest install on
Linux is as a `gh` extension:

```bash
gh extension install https://github.com/nektos/gh-act
```

Dry-run the self-test (fetches your repo's real SBOM but skips submission):

```bash
gh act push -W .github/workflows/ci.yml
```

## License

[MIT](LICENSE)
