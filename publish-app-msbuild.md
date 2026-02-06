# Publish Application with MSDeploy Workflow

This reusable GitHub Actions workflow automates the process of building and deploying .NET and/or SPA (Single Page Application) projects to IIS servers using Microsoft Web Deploy (MSDeploy). It provides flexibility to build and deploy either or both project types, combining artifacts when both are present.

## Functionality Summary

The **Publish Application with MSDeploy** workflow includes the following features:

- **.NET Backend Build**:
  - Restores NuGet packages and builds the .NET application.
  - Publishes the .NET application to a build output directory.
  - Supports adding private NuGet sources based on provided accounts and secrets.
  - Adds a GitHub release asset, if applicable, with the .NET application

- **SPA Frontend Build**:
  - Installs NPM dependencies and builds the SPA application using a specified NPM script.
  - Uploads the built SPA artifact and optionally a persisted documents file.
  - Adds a GitHub release asset, if applicable, with the SPA application

- **Deployment to IIS with MSDeploy**:
  - Deploys the .NET application to IIS via MSDeploy.
  - When both .NET and SPA are present, combines them before deployment (SPA goes to wwwroot).
  - Supports Basic authentication and configurable SSL certificate validation.

- **Conditional Execution**:
  - Workflow steps are conditionally executed based on the inputs provided.
  - Skips .NET steps if `dotnet_folder` is not specified.
  - Skips SPA steps if `spa_folder` is not specified.
  - Deployment jobs run based on what was built.

## Configuration Options

### Common Inputs

| **Input Name**         | **Description**                                                                     | **Required** | **Default Value**    |
|------------------------|-------------------------------------------------------------------------------------|--------------|----------------------|
| `environment_name`     | Specifies the GitHub environment to use while building and for MSDeploy authentication | Yes       | None                 |

### .NET Build Inputs

| **Input Name**         | **Description**                                           | **Required** | **Default Value**    | **Comments** |
|------------------------|-----------------------------------------------------------|--------------|----------------------|--------------|
| `dotnet_folder`        | Path to the .NET project folder                           | No           | None                 | Omitting skips .NET-related steps |
| `global_json_folder`   | Path to the folder containing `global.json`               | No           | `.`                  | Ignores `dotnet_folder` when resolving path |
| `nuget_accounts`       | Comma-separated list of GitHub accounts for NuGet sources | No           | Repository owner     | Skipped if tokens are not provided |
| `npm_accounts`         | Comma-separated list of GitHub accounts for NPM sources   | No           | Repository owner     | Skipped if tokens are not provided |
| `dotnet_configuration` | .NET build configuration (e.g., `Release`, `Debug`)       | No           | `Release`            |              |

### SPA Build Inputs

| **Input Name**             | **Description**                                                   | **Required** | **Default Value** | **Comments** |
|----------------------------|-------------------------------------------------------------------|--------------|-------------------|--------------|
| `spa_folder`               | Path to the SPA project folder                                    | No           | None              | Omitting skips SPA-related steps |
| `spa_version_env`          | Environment variable to set with the version number               | No           | `VITE_VERSION`    | e.g. `REACT_APP_VERSION` for React |
| `npm_build_script`         | NPM script used to build the SPA application                      | No           | `build`           |              |
| `npm_dist_folder`          | Path to the SPA distribution folder (relative to `spa_folder`)    | No           | `dist`            |              |
| `persisted_documents_file` | Path to the persisted documents file to copy to .NET build folder | No           | None              |              |
| `npm_analyze_script`       | NPM script to run after build for analysis                        | No           | None              |              |
| `analysis_artifacts`       | Full path to files to upload as analysis artifacts (glob pattern) | No           | None              |              |

### MSDeploy Configuration Inputs

| **Input Name**              | **Description**                           | **Required** | **Default Value** |
|-----------------------------|-------------------------------------------|--------------|-------------------|
| `msdeploy_server_url`       | MSDeploy server URL for deployment        | No           | None              |
| `msdeploy_site_name`        | IIS site name for deployment              | No           | None              |
| `msdeploy_username`         | Username for MSDeploy authentication      | No           | None              |
| `msdeploy_password`         | Password for MSDeploy authentication      | No           | None              |
| `msdeploy_allow_untrusted`  | Allow untrusted SSL certificates          | No           | `false`           |

### Environment Variables

If an environment is configured, variables configured for the specified environment are used as shell environment variables when building the code.

### Required Variables or Inputs

The workflow requires the following MSDeploy configuration values to be provided either as:

- **Workflow inputs** (as shown in the table above), or
- **Repository/Environment variables** with these names:

| **Variable Name**       | **Description**                        | **Required**                               | **Default Value** |
|-------------------------|----------------------------------------|--------------------------------------------|-------------------|
| `MSDEPLOY_SERVER_URL`   | MSDeploy server URL for deployment     | No (will skip deployment if not specified) | None              |
| `MSDEPLOY_SITE_NAME`    | IIS site name for deployment           | No (will skip deployment if not specified) | None              |
| `MSDEPLOY_USERNAME`     | Username for MSDeploy authentication   | No (will skip deployment if not specified) | None              |
| `MSDEPLOY_PASSWORD`     | Password for MSDeploy authentication   | No (will skip deployment if not specified) | None              |

The workflow will first check for values provided as inputs, and if not found, will fall back to repository or environment variables. If neither is available, the workflow will skip deployment steps.

### Secrets

| **Secret Name**           | **Description**                                       | **Required** | **Default Value** |
|---------------------------|-------------------------------------------------------|--------------|-------------------|
| `NUGET_ORG_USER`          | Username for private NuGet source (if applicable)     | No           | None              |
| `NUGET_ORG_TOKEN`         | Token for private NuGet source (if applicable)        | No           | None              |
| `NPM_TOKEN`               | GitHub Personal Access Token for private npm packages | No           | None              |

## Example Usage Scripts

### 1. Deploy .NET Application to IIS for development

```yaml
name: Deploy .NET Application

on:
  push:
    branches:
      - master

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  publish-application:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app-msbuild.yml@v2
    permissions:
      contents: write
    with:
      dotnet_folder: '.'
      environment_name: Development
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
```

The above example assumes that the necessary MSDeploy configuration values are stored as GitHub environment variables for the Development environment.

### 2. Deploy SPA Only to IIS for development

```yaml
name: Deploy SPA Application

on:
  push:
    branches:
      - master

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  publish-application:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app-msbuild.yml@v2
    permissions:
      contents: write
    with:
      spa_folder: ReactApp
      environment_name: Development
      msdeploy_server_url: https://dev-server.example.com:8172/msdeploy.axd
      msdeploy_site_name: DevSite
      msdeploy_username: deploy-user
      msdeploy_password: ${{ vars.MSDEPLOY_DEV_PASSWORD }}
```

The above example shows how to provide MSDeploy configuration values as workflow inputs, pulling from repository variables.

### 3. Deploy Full Application (Backend and SPA) to IIS for production

```yaml
name: Deploy Full Application

on:
  release:
    types:
      - published

jobs:
  publish-application:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app-msbuild.yml@v2
    permissions:
      contents: write
    with:
      dotnet_folder: '.'
      spa_folder: ReactApp
      environment_name: Production
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

The above example assumes that the necessary MSDeploy configuration values are stored as GitHub environment variables, and only passes the required NuGet secrets. When both .NET and SPA are specified, the SPA will be automatically deployed to the wwwroot folder within the IIS site.

## Notes

- Unlike the Azure version, this workflow does not have a `spa_deployment_method` input
- When both .NET and SPA are present, they are automatically combined before deployment (SPA goes to wwwroot)
- The workflow runs on Windows runners for deployment due to MSDeploy requirements
- `NUGET_ORG_USER`, `NUGET_ORG_TOKEN`, and `NPM_TOKEN` should already be configured as organization secrets but need to be passed in.
- `global.json` is required
- Cannot override .NET SDK with another version or install multiple SDKs
- `permissions: contents: write` is necessary to upload the compiled application as a release asset
- The `msdeploy_allow_untrusted` option should only be set to `true` for development/staging environments with self-signed certificates

## MSDeploy Server URL Format

The MSDeploy server URL typically follows this format:

```
https://server-name:8172/msdeploy.axd
```

Where:
- `server-name` is your IIS server hostname or IP address
- `8172` is the default MSDeploy port (may vary based on your configuration)
- `/msdeploy.axd` is the MSDeploy handler endpoint

## IIS Site Name

The IIS site name should match the exact name of the website or application in IIS where you want to deploy. For deploying to a specific application within a site, use the format:

```
SiteName/ApplicationName
```
