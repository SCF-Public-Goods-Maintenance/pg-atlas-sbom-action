# Ecosystem support guide

This guide explains the setup used by the [ecosystem examples](https://github.com/pgm-testing). Use
the demo for your ecosystem as a starting point, then check your own exported SBOM and Atlas
dependency relationships. The examples cover specific fixtures, not every package-manager
configuration or project.

## Enable dependency collection

GitHub's **Dependency graph** must be enabled for the repository before the Action can export its
SBOM. This setting is separate from **Automatic dependency submission**.

For Gradle, enable **Automatic dependency submission** and allow GitHub's dependency collection job
to finish before exporting the SBOM. The Gradle demo does not submit a custom dependency snapshot.
The other demo setups do not require that additional setting.

GitHub updates dependency graphs asynchronously. After changing dependencies, wait for collection to
finish before checking whether the export reflects the new versions.

## Configure the Action

Make sure that your project uses an Action revision and package manager version that are at least as
recent as the ones in the relevant demo. See the [README workflow example](../README.md#usage) for
how to add the Action to your repository.

## Tested ecosystem support

Commit the dependency files used by your ecosystem so GitHub can collect the dependency graph. The
files below describe the demo fixtures; they are not a universal list of files supported by GitHub.

| Ecosystem                                                               | Demo dependency files                                        | Additional setup                       | Verification                                                                                                                                                                                            |
| ----------------------------------------------------------------------- | ------------------------------------------------------------ | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Python / uv](https://github.com/pgm-testing/sbom-demo-python)          | `pyproject.toml`, `uv.lock`                                  | —                                      | [![Verification](https://github.com/pgm-testing/sbom-demo-python/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-python/actions/workflows/verify-atlas.yml)     |
| [JavaScript / npm](https://github.com/pgm-testing/sbom-demo-npm)        | `package.json`, `package-lock.json`                          | —                                      | [![Verification](https://github.com/pgm-testing/sbom-demo-npm/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-npm/actions/workflows/verify-atlas.yml)           |
| [JVM / Maven](https://github.com/pgm-testing/sbom-demo-maven)           | `pom.xml` with pinned direct versions                        | —                                      | [![Verification](https://github.com/pgm-testing/sbom-demo-maven/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-maven/actions/workflows/verify-atlas.yml)       |
| [JVM / Gradle](https://github.com/pgm-testing/sbom-demo-gradle)         | `build.gradle.kts`, `settings.gradle.kts`, `gradle.lockfile` | Enable Automatic dependency submission | [![Verification](https://github.com/pgm-testing/sbom-demo-gradle/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-gradle/actions/workflows/verify-atlas.yml)     |
| [.NET / NuGet](https://github.com/pgm-testing/sbom-demo-dotnet)         | `Demo.csproj`, `packages.lock.json`                          | —                                      | [![Verification](https://github.com/pgm-testing/sbom-demo-dotnet/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-dotnet/actions/workflows/verify-atlas.yml)     |
| [Swift Package Manager](https://github.com/pgm-testing/sbom-demo-swift) | `Package.swift`, `Package.resolved`                          | None for this example                  | [![Verification](https://github.com/pgm-testing/sbom-demo-swift/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-swift/actions/workflows/verify-atlas.yml)       |
| [Dart / Flutter](https://github.com/pgm-testing/sbom-demo-flutter)      | `pubspec.yaml`, `pubspec.lock`                               | —                                      | [![Verification](https://github.com/pgm-testing/sbom-demo-flutter/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-flutter/actions/workflows/verify-atlas.yml)   |
| [PHP / Composer](https://github.com/pgm-testing/sbom-demo-composer)     | `composer.json`, `composer.lock`                             | —                                      | [![Verification](https://github.com/pgm-testing/sbom-demo-composer/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-composer/actions/workflows/verify-atlas.yml) |
| [Go modules](https://github.com/pgm-testing/sbom-demo-go)               | `go.mod`, `go.sum`                                           | None for this example                  | [![Verification](https://github.com/pgm-testing/sbom-demo-go/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-go/actions/workflows/verify-atlas.yml)             |
| [Rust / Cargo](https://github.com/pgm-testing/sbom-demo-cargo)          | `Cargo.toml`, `Cargo.lock`                                   | —                                      | [![Verification](https://github.com/pgm-testing/sbom-demo-cargo/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-cargo/actions/workflows/verify-atlas.yml)       |

The Maven and Gradle examples cover JVM dependencies. The Swift example includes `Package.resolved`.
Use the example that matches your package manager; other targets and configurations need their own
verification.

The badges link to the verification runs for the examples. For the test setup, limitations, and
interpretation of those runs, see [end-to-end testing](e2e-testing.md).

## Check your submission

After the Action submits an SBOM, allow Atlas to finish processing it (see
[SBOM Queue](https://github.com/SCF-Public-Goods-Maintenance/pg-atlas-backend/actions/workflows/sbom-queue.yml)),
then check your repository's dependency relationships in Atlas. A successful submission alone does
not establish that every dependency was collected or linked correctly.

View your SBOM submissions in the API:

- Processing status — `https://api.pgatlas.xyz/ingest/sbom?repository=owner%2Frepo`
- Artifact read from IPFS — `https://api.pgatlas.xyz/ingest/sbom/{submission_id}`

## How to report issues

Open an issue if the verification run for your ecosystem is passing, you have followed the guidance,
and the submitted SBOM or dependency relationships in Atlas are still lacking. Please describe your
expectations and findings by opening an issue in the relevant demo repository. It is helpful if you
can also create a PR with a minimal reproduction of the flaw.

The demos are not automatically updated to use the latest version of the Action or ecosystem build
tools. If an upgrade on your end led to errors in SBOM processing, please include links to your last
successful workflow run, and the first one that demonstrates the flaw. If your workflow runs aren't
public, we'll need a full description of the changes and the corresponding SBOM submission IDs.
