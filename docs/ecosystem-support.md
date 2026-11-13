# Ecosystem support guide

This guide records the minimum setup demonstrated by the [D1 test repositories](https://github.com/pgm-testing).
It was checked on 13 November 2026 against the production Atlas API. It describes the tested fixtures,
not a guarantee for every package manager configuration or project.

The goal is simple: when your repository uses a Stellar public good, its dependency should appear in
GitHub's exported SBOM and link to the right repository in Atlas.

## Start with your current settings

Keep settings as they are unless a check shows that something is missing.

The Action needs GitHub's Dependency graph to be enabled. If it is already on, no change is needed.
The new demo repositories initially had it disabled; their first exports returned 404. Enabling it
allowed export. This prerequisite is separate from **Automatic dependency submission**.

Do not enable Automatic dependency submission everywhere. In these tests, **Gradle needed it** and Alex
enabled it in the repository settings. The other demos exported their expected packages without it.

Use the Action's existing workflow and OIDC permissions: `contents: read` and `id-token: write`.
No Atlas secret or API key is needed. The demos use the unchanged Action at
`ed68e4cc4c42a25a88a12f8ac26f520986851ec5`.

## Tested setups

All rows below require Dependency graph to be on. “No extra setting” means no additional
ecosystem-specific GitHub setting was needed in that demo.

| Ecosystem | Committed dependency files | Additional setup | Processed Atlas result |
| --- | --- | --- | --- |
| [Python / uv](https://github.com/pgm-testing/sbom-demo-python) | `pyproject.toml`, `uv.lock` | No extra setting | Three original Stellar probes pass; Tansu links to its historical repository |
| [JavaScript / npm](https://github.com/pgm-testing/sbom-demo-npm) | `package.json`, `package-lock.json` | No extra setting | All five Stellar probes pass |
| [Maven](https://github.com/pgm-testing/sbom-demo-maven) | `pom.xml` with pinned direct versions | No extra setting | Java and KMP probes pass |
| [Gradle](https://github.com/pgm-testing/sbom-demo-gradle) | `build.gradle.kts`, `settings.gradle.kts`, `gradle.lockfile` | Enable Automatic dependency submission | Java and KMP probes pass after the setting change |
| [.NET / NuGet](https://github.com/pgm-testing/sbom-demo-dotnet) | `Demo.csproj`, `packages.lock.json` | No extra setting | Stellar .NET SDK probe passes |
| [Swift Package Manager](https://github.com/pgm-testing/sbom-demo-swift) | `Package.swift`, `Package.resolved` | No extra setting for the tested export | SDK is in the SBOM; Atlas repository linking remains unresolved |
| [Dart / Flutter](https://github.com/pgm-testing/sbom-demo-flutter) | `pubspec.yaml`, `pubspec.lock` | No extra setting | Flutter SDK probe passes |
| [PHP / Composer](https://github.com/pgm-testing/sbom-demo-composer) | `composer.json`, `composer.lock` | No extra setting | PHP SDK probe passes |
| [Go modules](https://github.com/pgm-testing/sbom-demo-go) | `go.mod`, `go.sum` | No extra setting for the tested export | nebu is in the SBOM; Atlas repository linking remains unresolved |
| [Rust / Cargo](https://github.com/pgm-testing/sbom-demo-cargo) | `Cargo.toml`, `Cargo.lock` | No extra setting | All seven Stellar probes pass |

Exact dependency pins, toolchain versions, build commands and expected Atlas repositories are recorded
in each demo's `expected.json`, workflow and lockfiles. Each demo includes a well-known control package.

These examples cover Python 3.12, Node 22.14.0, Java 21 with separate Maven and Gradle 8.14.3 builds,
.NET SDK 8.0.408, the Swift toolchain supplied by macOS 15 runners, Flutter 3.35.7, PHP 8.3,
Go 1.26.11 and Rust 1.95.0. Toolchain output is kept with run evidence. A single JVM artifact is used
for the KMP probe; native KMP targets are not separately tested here.

The Swift result uses a committed resolved file. A manifest-only Swift variant has not yet been
established by this demo. Other Python or JavaScript package managers and private-repository
configurations are also outside the tested matrix.

## Gradle: the setting that made the difference

Open **Settings → Advanced Security → Dependency graph → Automatic dependency submission** and enable
automatic submission when the baseline demonstrates missing Gradle dependencies.

GitHub then runs its dependency collection separately. Allow that job to finish before exporting the
SBOM again. The demo's Gradle setup does not submit a custom dependency snapshot.

The processed submission [2431](https://api.pgatlas.xyz/ingest/sbom/2431) links to both the Java SDK
and the KMP SDK. Its [baseline run](https://github.com/pgm-testing/sbom-demo-gradle/actions/runs/43437541224)
originally stopped because processing was pending; the later API check passed.

## Read the three stages separately

1. **Build and resolve:** the fixture must install or build successfully with its pinned dependencies.
2. **SBOM export:** check expected package identities, versions and dependency paths. A successful export
   or a high package count alone does not establish coverage.
3. **Atlas processing:** wait for the submission to be processed, then check the repository's
   `depends-on` API response for the expected repository IDs.

The demos are not attached to Atlas projects. Their checks also verify that the repository has no
project association.

GitHub updates its dependency graph asynchronously. A run immediately after a dependency change may
export the previous graph. Preserve that run, wait for the graph-update job to finish and repeat the
export before declaring a coverage gap.

The Action's current response does not expose a submission ID. The demos identify the submitted
document by Atlas's semantic hash of the document name, packages and relationships. This handles
deduplication without treating timestamps or export namespaces as dependency changes.

## Waiting without producing false failures

The demo checks poll the submission every 15 seconds for up to five minutes. They check relationships
as soon as processing completes.

If processing is still pending at the deadline, they record `pending-timeout` and
`verification_complete: false`. That delay does not fail the workflow, but **it is not a coverage
pass**. Run the demo's `verify-atlas.yml` workflow later with the saved submission ID, at the same
fixture revision. The baseline and follow-up result should be reviewed together.

A missing relationship after processing still fails. API errors and failed processing remain errors.
The CLI supports `--processing-wait-seconds` and `--poll-interval` to try a longer waiting period.
Submission-record lookup has a separate 90-second budget.

Initial runs took several hours to process; five minutes is a bounded trial, not a promise about
worker turnaround. Tests cover pending-to-processed polling, timeout reporting, processing failures
and missing relationships. Longer live waits can be assessed from subsequent run evidence.

## Links that still need correction

These are package-to-repository attribution findings, not missing packages in the exported SBOM.
There will be backend implementation for fixes; the demos provide reproducible checks for those fixes.

| Case | Received package | Observed Atlas link | Expected repository |
| --- | --- | --- | --- |
| Tansu | `pkg:pypi/tansu@1.0.1` | `pkg:github/tupui/soroban-versioning` | `pkg:github/Consulting-Manao/tansu` |
| Swift SDK | `pkg:swift/github.com/soneso/stellar-ios-mac-sdk@3.14.0` | Unresolved external Swift package | `pkg:github/Soneso/stellar-ios-mac-sdk` |
| nebu | `pkg:golang/github.com/withobsrvr/nebu@v0.7.2` | Unresolved external Go package | `pkg:github/withObsrvr/nebu` |

Evidence: [Python submission 2430](https://api.pgatlas.xyz/ingest/sbom/2430),
[Swift submission 2426](https://api.pgatlas.xyz/ingest/sbom/2426),
and [Go submission 2424](https://api.pgatlas.xyz/ingest/sbom/2424).

OpenGrants and StellarAtlas had package-name mappings that were not supported by publication evidence.
Those names were excluded from the fixtures; actual published package names still need maintainer
confirmation. This does not establish that either project publishes no packages.

## Preserve the evidence

Each baseline uploads the exported SBOM, expectations, submission reference, run/commit references and
check results. Artifacts are retained for 30 days; download important runs before they expire.

The original Python export flattened transitive dependencies onto the root. A later export preserved
the checked `requests → urllib3` relationship. Keep both observations rather than assume all exports
always have the same graph structure.

GitHub's reverse **Dependents** page is not an acceptance check. Dependencies submitted through
snapshots may appear in exported SBOMs without appearing on that page.
