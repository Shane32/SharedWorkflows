# Deploy Azure Web App Workflow

This reusable GitHub Actions workflow automates the process of deploying applications to Azure Web App. It supports deploying both .NET applications and SPA (Single Page Application) projects, with the ability to handle persisted documents and multiple deployment slots.

## Functionality Summary

The **Deploy Azure Web App** workflow includes the following features:

- **Artifact Deployment**:
  - Downloads and deploys .NET application artifacts
  - Optionally deploys SPA artifacts to the wwwroot folder
  - Optionally deploys persisted documents file(s) to the root folder

- **Azure Integration**:
  - Authenticates with Azure using OIDC
  - Deploys to specified Azure Web App
  - Supports deployment to different slots (e.g., Production, Staging)

- **Environment Management**:
  - Configurable environment names for deployment
  - Supports different configurations per environment

## Configuration Options

### Workflow Inputs

| **Input Name**                 | **Description**                                           | **Required** | **Default Value** |
|--------------------------------|-----------------------------------------------------------|--------------|-------------------|
| `environment_name`             | Environment name for deployment                           | Yes          | None              |
| `artifact_name`                | Name of the .NET artifact to deploy                       | No           | `.net-app`        |
| `spa_artifact`                 | Name of the SPA artifact to deploy in wwwroot folder      | No           | None              |
| `persisted_documents_artifact` | Name of the persisted documents artifact to deploy        | No           | None              |
| `azure_client_id`              | Client ID for Azure deployment                            | No           | None              |
| `azure_tenant_id`              | Tenant ID for Azure deployment                            | No           | None              |
| `azure_subscription_id`        | Subscription ID for Azure deployment                      | No           | None              |
| `azure_webapp_name`            | Azure Web App name for deployment                         | No           | None              |
| `azure_webapp_slot_name`       | Azure Web App slot name for deployment                    | No           | None              |

### Required Variables or Inputs

The workflow requires the following Azure configuration values to be provided either as:

- **Workflow inputs** (as shown in the table above), or
- **Repository/Environment variables** with these names:

| **Variable Name**        | **Description**                        | **Required** | **Default Value** |
|--------------------------|----------------------------------------|--------------|-------------------|
| `AZURE_CLIENT_ID`        | Client ID for Azure deployment         | Yes          | None              |
| `AZURE_TENANT_ID`        | Tenant ID for Azure deployment         | Yes          | None              |
| `AZURE_SUBSCRIPTION_ID`  | Subscription ID for Azure deployment   | Yes          | None              |
| `AZURE_WEBAPP_NAME`      | Azure Web App name for deployment      | Yes          | None              |
| `AZURE_WEBAPP_SLOT_NAME` | Azure Web App slot name for deployment | No           | `Production`      |

The workflow will first check for values provided as inputs, and if not found, will fall back to repository or environment variables. If neither is available, the workflow will fail with a validation error.

## Example Usage Scripts

### 1. Deploy .NET Application

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy:
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-azurewebapp.yml@v2
    permissions:
      contents: read
      id-token: write
    with:
      environment_name: Production
```

The above example assumes that the necessary Azure configuration values are stored as GitHub environment variables.

### 2. Deploy Full Stack Application to Staging Slot

```yaml
name: Deploy to Staging

on:
  pull_request:
    branches:
      - main

jobs:
  deploy:
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-azurewebapp.yml@v2
    permissions:
      contents: read
      id-token: write
    with:
      environment_name: Staging
      artifact_name: backend-build
      spa_artifact: frontend-build
      persisted_documents_artifact: persisted-docs
      azure_webapp_name: my-production-app
      azure_webapp_slot_name: Staging
      azure_client_id: ${{ secrets.AZURE_STAGING_CLIENT_ID }}
      azure_tenant_id: ${{ secrets.AZURE_STAGING_TENANT_ID }}
      azure_subscription_id: ${{ secrets.AZURE_STAGING_SUBSCRIPTION_ID }}
```

The above example shows how to provide Azure configuration values as workflow inputs, pulling from repository secrets.

## Notes

- The workflow uses OIDC (OpenID Connect) for secure authentication with Azure
- The `permissions: id-token: write` is required for Azure deployment workflows to enable OIDC authentication with Azure.
- SPA artifacts are automatically deployed to the wwwroot folder when specified
- Perisisted document artifacts are automatically deployed to the root folder when specified
- All artifacts must be previously created and available in the workflow run
