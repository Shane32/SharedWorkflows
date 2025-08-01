# Publish Application Workflow

This reusable GitHub Actions workflow automates the process of building and deploying .NET and/or SPA (Single Page Application) projects to Azure. It provides flexibility to build and deploy either or both project types, supporting deployment to Azure Web Apps and/or Azure Storage.

## Functionality Summary

The **Publish Application** workflow includes the following features:

- **.NET Backend Build**:
  - Restores NuGet packages and builds the .NET application.
  - Publishes the .NET application to a build output directory.
  - Supports adding private NuGet sources based on provided accounts and secrets.
  - Adds a GitHub release asset, if applicable, with the .NET application

- **SPA Frontend Build**:
  - Installs NPM dependencies and builds the SPA application using a specified NPM script.
  - Uploads the built SPA artifact and optionally a persisted documents file.
  - Adds a GitHub release asset, if applicable, with the SPA application

- **Deployment to Azure**:
  - Deploys the .NET and/or SPA application to Azure Web App.
  - Deploys the SPA application to Azure Storage for static website hosting.
  - Supports deployment methods: `azure-webapp` and `azure-storage`.

- **Conditional Execution**:
  - Workflow steps are conditionally executed based on the inputs provided.
  - Skips .NET steps if `dotnet_folder` is not specified.
  - Skips SPA steps if `spa_folder` is not specified.
  - Deployment jobs run based on the `deployment_method` input.

## Configuration Options

### Common Inputs

| **Input Name**         | **Description**                                                                     | **Required** | **Default Value**    |
|------------------------|-------------------------------------------------------------------------------------|--------------|----------------------|
| `environment_name`     | Specifies the GitHub environment to use while building and for Azure authentication | Yes          | None                 |

### .NET Build Inputs

| **Input Name**         | **Description**                                           | **Required** | **Default Value**    | **Comments** |
|------------------------|-----------------------------------------------------------|--------------|----------------------|--------------|
| `dotnet_folder`        | Path to the .NET project folder                           | No           | None                 | Omitting skips .NET-related steps |
| `global_json_folder`   | Path to the folder containing `global.json`               | No           | `.`                  | Ignores `dotnet_folder` when resolving path |
| `nuget_accounts`       | Comma-separated list of GitHub accounts for NuGet sources | No           | Repository owner     | Skipped if tokens are not provided |
| `dotnet_configuration` | .NET build configuration (e.g., `Release`, `Debug`)       | No           | `Release`            |              |

### SPA Build Inputs

| **Input Name**             | **Description**                                                   | **Required** | **Default Value** | **Comments** |
|----------------------------|-------------------------------------------------------------------|--------------|-------------------|--------------|
| `spa_folder`               | Path to the SPA project folder                                    | No           | None              | Omitting skips SPA-related steps |
| `spa_deployment_method`    | SPA deployment method: `azure-webapp` or `azure-storage`          | No           | None              | Required to deploy |
| `spa_version_env`          | Environment variable to set with the version number               | No           | `VITE_VERSION`    | e.g. `REACT_APP_VERSION` for React |
| `npm_build_script`         | NPM script used to build the SPA application                      | No           | `build`           |              |
| `npm_dist_folder`          | Path to the SPA distribution folder (relative to `spa_folder`)    | No           | `dist`            |              |
| `persisted_documents_file` | Path to the persisted documents file to copy to .NET build folder | No           | None              |              |
| `npm_analyze_script`       | NPM script to run after build for analysis                        | No           | None              |              |
| `analysis_artifacts`       | Full path to files to upload as analysis artifacts (glob pattern) | No           | None              |              |

### Azure Configuration Inputs

| **Input Name**            | **Description**                                   | **Required** | **Default Value** |
|---------------------------|---------------------------------------------------|--------------|-------------------|
| `azure_client_id`         | Client ID for Azure deployment                    | No           | None              |
| `azure_tenant_id`         | Tenant ID for Azure deployment                    | No           | None              |
| `azure_subscription_id`   | Subscription ID for Azure deployment              | No           | None              |
| `azure_webapp_name`       | Azure Web App name for deployment                 | No           | None              |
| `azure_webapp_slot_name`  | Azure Web App slot name for deployment            | No           | None              |
| `azure_storage_account`   | Azure Storage account name for deployment         | No           | None              |
| `azure_storage_container` | Azure Storage container name for deployment       | No           | None              |

### Environment Variables

If an environment is configured, variables configured for the specified environment are used as shell environment variables when building the code.

### Required Variables or Inputs

The workflow requires the following Azure configuration values to be provided either as:

- **Workflow inputs** (as shown in the table above), or
- **Repository/Environment variables** with these names:

| **Variable Name**         | **Description**                                   | **Required**                                | **Default Value** |
|---------------------------|---------------------------------------------------|---------------------------------------------|-------------------|
| `AZURE_CLIENT_ID`         | Client ID for Azure deployment                    | No (will skip deployment if not specified)  | None              |
| `AZURE_TENANT_ID`         | Tenant ID for Azure deployment                    | No (will skip deployment if not specified)  | None              |
| `AZURE_SUBSCRIPTION_ID`   | Subscription ID for Azure deployment              | No (will skip deployment if not specified)  | None              |
| `AZURE_WEBAPP_NAME`       | Azure Web App name for deployment                                    | Required if `deployment_method` is `azure-webapp`  | None         |
| `AZURE_WEBAPP_SLOT_NAME`  | Azure Web App slot name for deployment                               | No                                                 | `Production` |
| `AZURE_STORAGE_ACCOUNT`   | Azure Storage account name for deployment (`azure-storage` method)   | Required if `deployment_method` is `azure-storage` | None         |
| `AZURE_STORAGE_CONTAINER` | Azure Storage container name for deployment (`azure-storage` method) | Required if `deployment_method` is `azure-storage` | None         |

The workflow will first check for values provided as inputs, and if not found, will fall back to repository or environment variables. If neither is available, the workflow will skip deployment steps.

### Secrets

| **Secret Name**           | **Description**                                   | **Required** | **Default Value** |
|---------------------------|---------------------------------------------------|--------------|-------------------|
| `NUGET_ORG_USER`          | Username for private NuGet source (if applicable) | No           | None              |
| `NUGET_ORG_TOKEN`         | Token for private NuGet source (if applicable)    | No           | None              |

## Example Usage Scripts

### 1. Deploy .NET Application to Azure Web App for development

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
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app.yml@v2
    permissions:
      contents: write
      id-token: write
    with:
      dotnet_folder: '.'
      environment_name: Development
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
```

The above example assumes that the necessary Azure configuration values are stored as GitHub environment variables for the Development environment.

### 2. Deploy SPA to Azure Storage for development

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
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app.yml@v2
    permissions:
      contents: write
      id-token: write
    with:
      spa_folder: ReactApp
      spa_deployment_method: azure-storage
      environment_name: Development
      azure_storage_account: mystorageaccount
      azure_storage_container: public
      azure_client_id: ${{ secrets.AZURE_DEV_CLIENT_ID }}
      azure_tenant_id: ${{ secrets.AZURE_DEV_TENANT_ID }}
      azure_subscription_id: ${{ secrets.AZURE_DEV_SUBSCRIPTION_ID }}
```

The above example shows how to provide Azure configuration values as workflow inputs, pulling from repository secrets.

### 3. Deploy Full Application (Backend and SPA) to Azure Web App for production

```yaml
name: Deploy Full Application

on:
  release:
    types:
      - published

jobs:
  publish-application:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app.yml@v2
    permissions:
      contents: write
      id-token: write
    with:
      dotnet_folder: '.'
      spa_folder: ReactApp
      spa_deployment_method: azure-webapp
      environment_name: Production
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
```

The above example assumes that the necessary Azure configuration values are stored as GitHub environment variables, and only passes the required NuGet secrets.

## Notes

- The `permissions: id-token: write` is required for Azure deployment workflows to enable OIDC authentication with Azure.
- `NUGET_ORG_USER` and `NUGET_ORG_TOKEN` should already be configured as organization secrets but need to be passed in.
- `global.json` is required
- Cannot override .NET SDK with another version or install multiple SDKs
- `permissions: contents: write` is necessary to upload the compiled application as a release asset
