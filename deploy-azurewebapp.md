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

### Required Secrets

| **Secret Name**          | **Description**                        | **Required** | **Default Value** |
|--------------------------|----------------------------------------|--------------|-------------------|
| `AZURE_CLIENT_ID`        | Client ID for Azure deployment         | Yes          | None              |
| `AZURE_TENANT_ID`        | Tenant ID for Azure deployment         | Yes          | None              |
| `AZURE_SUBSCRIPTION_ID`  | Subscription ID for Azure deployment   | Yes          | None              |
| `AZURE_WEBAPP_NAME`      | Azure Web App name for deployment      | Yes          | None              |
| `AZURE_WEBAPP_SLOT_NAME` | Azure Web App slot name for deployment | No           | `Production`      |

:warning: To use environment-scoped secrets, you must use `secrets: inherit`. :warning:

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
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-azurewebapp.yml@v1
    with:
      environment_name: Production
    secrets: inherit
```

The above example assumes that the necessary secrets are stored in GitHub's environment secrets.

### 2. Deploy Full Stack Application to Staging Slot

```yaml
name: Deploy to Staging

on:
  pull_request:
    branches:
      - main

jobs:
  deploy:
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-azurewebapp.yml@v1
    with:
      environment_name: Staging
      artifact_name: backend-build
      spa_artifact: frontend-build
      persisted_documents_artifact: persisted-docs
    secrets:
      AZURE_WEBAPP_NAME: my-production-app
      AZURE_WEBAPP_SLOT_NAME: Staging
      AZURE_CLIENT_ID: ${{ secrets.AZURE_STAGING_CLIENT_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_STAGING_TENANT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_STAGING_SUBSCRIPTION_ID }}
```

The above example assumes that the necessary secrets are stored in the repository secrets.

## Notes

- The workflow uses OIDC (OpenID Connect) for secure authentication with Azure
- SPA artifacts are automatically deployed to the wwwroot folder when specified
- Perisisted document artifacts are automatically deployed to the root folder when specified
- All artifacts must be previously created and available in the workflow run
