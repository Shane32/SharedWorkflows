# Build Check Workflow

This reusable GitHub Actions workflow is designed to streamline building, testing, and formatting .NET and SPA (Single Page Application) projects. It provides flexibility and modularity, allowing you to enable or disable components as needed. By configuring appropriate options, the workflow can handle complex build pipelines involving multiple project types.

## Functionality Summary

The `Build Check` workflow includes:
- **.NET Support:**
  - Builds and tests .NET projects and/or solutions.
  - Executes tests with optional test settings and generates code coverage reports.
  - Supports uploading code coverage data to Codecov and generating artifacts like HTML summary and Clover reports.
- **SPA Support:**
  - Installs dependencies and builds JavaScript-based SPA projects.
  - Executes tests with optional test scripts.
  - Runs optional analysis scripts and uploads analysis artifacts.
- **Formatting Checks:**
  - Validates .NET project formatting using `dotnet format` with configurable severity.
  - Runs linting and Prettier checks for JavaScript projects.
- **Secrets-Driven Functionality:**
  - Automatically configures private NuGet sources if secrets are provided.
  - Uploads code coverage data or monitors coverage metrics if respective secrets are available.
- **Conditional Execution:**
  - Skips .NET or SPA steps if respective folders are not defined, allowing partial configurations.
  - Skips formatting checks when explicitly disabled.

## Configuration Options

### Generic Options

| **Input Name**          | **Description**                           | **Required** | **Default Value**       | **Comments**                               |
|-------------------------|-------------------------------------------|--------------|-------------------------|--------------------------------------------|
| `build`                 | Enable or disable build checks            | No           | `true`                  |                                            |
| `format`                | Enable or disable formatting checks       | No           | Automatic               | By default only runs for pull requests     |

### .NET Options

| **Input Name**          | **Description**                           | **Required** | **Default Value**       | **Comments**                                   |
|-------------------------|-------------------------------------------|--------------|-------------------------|------------------------------------------------|
| `dotnet_folder`         | Path to the .NET project folder           | No           | None                    | Omitting skips .NET-related steps              |
| `dotnet_build_runner`   | Runner for .NET builds                    | No           | `ubuntu-latest`         | Specifies the runner for .NET builds and tests |
| `dotnet_format_severity`| .NET format severity level                | No           | `error`                 | Controls formatting validation severity        |
| `global_json_folder`    | Path to the folder containing global.json | No           | `.`                     | Ignores `dotnet_folder` when resolving path    |
| `nuget_accounts`        | NuGet Accounts to configure               | No           | Repository owner        | Skipped if tokens are not provided             |
| `dotnet_test_settings`  | .NET test settings XML file location      | No           | `testsettings.xml`      | Only used if file exists                       |
| `data_collector`        | Data collector for dotnet test            | No           | `XPlat Code Coverage`   | If blank then setting is not applied           |
| `coveralls`             | Enable Coveralls support                  | No           | `false`                 |                                                |
| `coverage_alert_threshold` | Coverage alert threshold               | No           | 20                      |                                                |
| `coverage_warning_threshold` | Coverage warning threshold           | No           | 80                      |                                                |
| `coverage_report`       | Enable or disable local coverage report   | No           | Automatic               | By default it is disabled when codecov is used |

### SPA Options

| **Input Name**          | **Description**                           | **Required** | **Default Value**       | **Comments**                               |
|-------------------------|-------------------------------------------|--------------|-------------------------|--------------------------------------------|
| `spa_folder`            | Path to the SPA project folder            | No           | None                    | Omitting skips SPA-related steps           |
| `npm_build_script`      | NPM build script                          | No           | `build`                 |                                            |
| `npm_test_script`       | NPM test script                           | No           | `test`                  | Pass blank value to skip test              |
| `npm_lint_script`       | NPM lint script                           | No           | `lint`                  | Pass blank value to skip linting checks    |
| `npm_prettier_script`   | NPM Prettier check script                 | No           | `prettier:check`        | Pass blank value to skip prettier checks   |
| `npm_analyze_script`    | NPM analyze script to run after build     | No           | None                    | Script to run for post-build analysis      |
| `analysis_artifacts`    | Full path to files to upload as artifacts | No           | None                    | Glob pattern for analysis output files     |

## Secrets

The workflow supports the following secrets, which are optional but enhance functionality:

- `NUGET_ORG_USER`: NuGet user for private repositories.
- `NUGET_ORG_TOKEN`: NuGet token for private repositories.
- `CODECOV_TOKEN`: Token for uploading code coverage to Codecov.

## Example Usage Scripts

### 1. PR Workflow for .NET Only

This script triggers a PR workflow for a .NET project using a Windows runner.

```yaml
name: PR Tests

on:
  pull_request:

concurrency:
  group: pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  build-check:
    uses: Shane32/SharedWorkflows/.github/workflows/build-check.yml@v2
    with:
      dotnet_folder: '.'
      dotnet_build_runner: 'windows-latest'
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

### 2. Build Workflow Skipping Formatting Checks

This script demonstrates a build workflow for uploading code coverage.

```yaml
name: Upload Coverage

on:
  push:
    - master

jobs:
  build-check:
    uses: Shane32/SharedWorkflows/.github/workflows/build-check.yml@v2
    with:
      dotnet_folder: '.'
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

### 3. PR Workflow for .NET and SPA with Analysis

This script triggers a PR workflow for a solution where the .NET project is in the current folder (`.`) and the SPA project is in `ReactApp`, including custom analysis.

```yaml
name: PR Tests

on:
  pull_request:

concurrency:
  group: pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  build-check:
    uses: Shane32/SharedWorkflows/.github/workflows/build-check.yml@v2
    with:
      dotnet_folder: '.'
      spa_folder: 'ReactApp'
      npm_analyze_script: 'analyze'
      analysis_artifacts: 'ReactApp/stats.html'
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

## Notes

- The necessary secrets should already be configured as organization secrets.
- `global.json` is required
- Cannot override .NET SDK with another version or install multiple SDKs
- Test settings XML file is only used if it exists (defaults to `testsettings.xml`)
