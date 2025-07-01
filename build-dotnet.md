# Build .NET Application Workflow

This reusable GitHub Actions workflow automates the process of building .NET applications. It handles package restoration, compilation, and artifact creation with support for version numbering and private NuGet sources.

## Functionality Summary

The **Build .NET Application** workflow includes the following features:

- **Version Management**:
  - Automatically generates version numbers based on tags or run numbers.
  - For tagged releases: uses tag value with `.0` suffix.
  - For other builds: uses format `0.0.0.{run-number}`.

- **NuGet Package Management**:
  - Restores NuGet packages from configured sources.
  - Supports adding private NuGet sources based on provided accounts and secrets.

- **Build Process**:
  - Sets up .NET SDK based on `global.json` configuration.
  - Restores project dependencies.
  - Compiles and publishes the application with version information.
  - Creates build artifacts for deployment.

- **Artifact Management**:
  - Uploads build output as a downloadable artifact.
  - Configurable artifact naming.

## Configuration Options

### Workflow Inputs

| **Input Name**          | **Description**                                          | **Required** | **Default Value**   | **Comments** |
|------------------------|-----------------------------------------------------------|--------------|---------------------|--------------|
| `dotnet_folder`        | Path to the .NET project folder                           | Yes          | None                |              |
| `global_json_folder`   | Path to the folder containing `global.json`               | No           | `.`                 | Used for SDK version |
| `nuget_accounts`       | Comma-separated list of GitHub accounts for NuGet sources | No           | Repository owner    | Skipped if tokens not provided |
| `dotnet_configuration` | .NET build configuration (e.g., `Release`, `Debug`)       | No           | `Release`           |              |
| `artifact_name`        | Name of the artifact to upload                            | No           | `.net-app`          |              |
| `environment_name`     | Environment name for deployment                           | No           | None                |              |

### Environment Variables

If an environment is configured, variables configured for the specified environment are used as environment variables when building the SPA.

### Secrets

| **Secret Name**       | **Description**                                  | **Required** |
|-----------------------|--------------------------------------------------|--------------|
| `NUGET_ORG_USER`      | Username for private NuGet source                | No           |
| `NUGET_ORG_TOKEN`     | Token for private NuGet source                   | No           |

### Outputs

| **Output Name** | **Description**                    | **Example Value** |
|-----------------|------------------------------------|-------------------|
| `version`       | The version number used for build  | `1.2.3.0` or `0.0.0.123` |

## Example Usage Scripts

### 1. Basic Build Configuration

```yaml
name: Build .NET App

on:
  push:
    branches:
      - master

jobs:
  build:
    uses: Shane32/SharedWorkflows/.github/workflows/build-dotnet.yml@v1
    with:
      dotnet_folder: 'MyAppServer'
    secrets: inherit
```

### 2. Advanced Build Configuration

```yaml
name: Build .NET App

on:
  push:
    branches:
      - master

jobs:
  build:
    uses: Shane32/SharedWorkflows/.github/workflows/build-dotnet.yml@v1
    with:
      dotnet_folder: 'src/MyApp'
      global_json_folder: 'src'
      dotnet_configuration: 'Debug'
      artifact_name: 'my-custom-app'
      environment_name: 'Development'
      nuget_accounts: 'Shane32'
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
```

## Notes

- `global.json` is required in the specified `global_json_folder`
- Version numbering follows the pattern:
  - For tags: `{tag}.0` (e.g., `1.2.3` becomes `1.2.3.0`)
  - For regular builds: `0.0.0.{run-number}`
- The workflow runs on Ubuntu latest
- Build artifacts are automatically uploaded and available for subsequent workflow steps or manual download
- Private NuGet sources are only configured if both username and token secrets are provided
