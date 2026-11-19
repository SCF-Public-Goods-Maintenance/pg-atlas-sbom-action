# Ecosystem support guide

This guide explains the setup used by the [ecosystem examples](https://github.com/pgm-testing).
Use the example for your ecosystem as a starting point, then check your own exported SBOM
and Atlas dependency relationships. The examples cover specific fixtures, not every package-manager
configuration or project.

## Enable dependency collection

GitHub's **Dependency graph** must be enabled for the repository before the Action can export its SBOM.
This setting is separate from **Automatic dependency submission**.

For Gradle, enable **Automatic dependency submission** and allow GitHub's dependency collection job
to finish before exporting the SBOM. The Gradle demo does not submit a custom dependency snapshot.
The other demo setups do not require that additional setting.

GitHub updates dependency graphs asynchronously. After changing dependencies, wait for collection
to finish before checking whether the export reflects the new versions.

## Configure the Action

Use the workflow and pinned Action revision shown in the relevant demo. The workflow needs:

```yaml
permissions:
  contents: read
  id-token: write
```

No Atlas secret or API key is needed. See the [README workflow example](../README.md#usage)
for how to add the Action to your repository.

## Ecosystem examples

Commit the dependency files used by your ecosystem so GitHub can collect the dependency graph.
The files below describe the demo fixtures; they are not a universal list of files supported by GitHub.

| Ecosystem | Demo dependency files | Additional setup | Verification |
| --- | --- | --- | --- |
| [Python / uv](https://github.com/pgm-testing/sbom-demo-python) | `pyproject.toml`, `uv.lock` | No additional setting | [![Verification](https://github.com/pgm-testing/sbom-demo-python/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-python/actions/workflows/verify-atlas.yml) |
| [JavaScript / npm](https://github.com/pgm-testing/sbom-demo-npm) | `package.json`, `package-lock.json` | No additional setting | [![Verification](https://github.com/pgm-testing/sbom-demo-npm/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-npm/actions/workflows/verify-atlas.yml) |
| [Maven](https://github.com/pgm-testing/sbom-demo-maven) | `pom.xml` with pinned direct versions | No additional setting | [![Verification](https://github.com/pgm-testing/sbom-demo-maven/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-maven/actions/workflows/verify-atlas.yml) |
| [Gradle](https://github.com/pgm-testing/sbom-demo-gradle) | `build.gradle.kts`, `settings.gradle.kts`, `gradle.lockfile` | Enable Automatic dependency submission | [![Verification](https://github.com/pgm-testing/sbom-demo-gradle/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-gradle/actions/workflows/verify-atlas.yml) |
| [.NET / NuGet](https://github.com/pgm-testing/sbom-demo-dotnet) | `Demo.csproj`, `packages.lock.json` | No additional setting | [![Verification](https://github.com/pgm-testing/sbom-demo-dotnet/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-dotnet/actions/workflows/verify-atlas.yml) |
| [Swift Package Manager](https://github.com/pgm-testing/sbom-demo-swift) | `Package.swift`, `Package.resolved` | No additional setting for this export setup | [![Verification](https://github.com/pgm-testing/sbom-demo-swift/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-swift/actions/workflows/verify-atlas.yml) |
| [Dart / Flutter](https://github.com/pgm-testing/sbom-demo-flutter) | `pubspec.yaml`, `pubspec.lock` | No additional setting | [![Verification](https://github.com/pgm-testing/sbom-demo-flutter/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-flutter/actions/workflows/verify-atlas.yml) |
| [PHP / Composer](https://github.com/pgm-testing/sbom-demo-composer) | `composer.json`, `composer.lock` | No additional setting | [![Verification](https://github.com/pgm-testing/sbom-demo-composer/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-composer/actions/workflows/verify-atlas.yml) |
| [Go modules](https://github.com/pgm-testing/sbom-demo-go) | `go.mod`, `go.sum` | No additional setting for this export setup | [![Verification](https://github.com/pgm-testing/sbom-demo-go/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-go/actions/workflows/verify-atlas.yml) |
| [Rust / Cargo](https://github.com/pgm-testing/sbom-demo-cargo) | `Cargo.toml`, `Cargo.lock` | No additional setting | [![Verification](https://github.com/pgm-testing/sbom-demo-cargo/actions/workflows/verify-atlas.yml/badge.svg)](https://github.com/pgm-testing/sbom-demo-cargo/actions/workflows/verify-atlas.yml) |

The Maven and Gradle examples cover JVM dependencies. The Swift example includes
`Package.resolved`. Use the example that matches your package manager; other targets
and configurations need their own verification.

The badges link to the verification runs for the examples. For the test setup,
limitations and interpretation of those runs, see [end-to-end testing](e2e-testing.md).

## Check your submission

After the Action submits an SBOM, allow Atlas to finish processing it, then check
your repository's dependency relationships in Atlas. A successful submission alone
does not establish that every dependency was collected or linked correctly.
