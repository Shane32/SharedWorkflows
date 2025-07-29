# Deploy to Azure Storage Workflow

This reusable GitHub Actions workflow automates the process of deploying static content to Azure Storage. It's particularly useful for deploying Single Page Applications (SPAs) or other static website content to Azure Storage's static website hosting.

## Functionality Summary

The **Deploy to Azure Storage** workflow includes the following features:

- **Artifact Download**:
  - Downloads the previously built artifact containing the static content.
  - Supports custom artifact names for flexibility in deployment pipelines.

- **Azure Authentication**:
  - Securely authenticates with Azure using OIDC (OpenID Connect).
  - Uses Azure CLI for secure and reliable authentication.

- **Storage Deployment**:
  - Synchronizes content to Azure Storage using AzCopy.
  - Performs recursive synchronization of all files.
  - Cleans up old files in the destination that don't exist in the source, configurable via `delete_destination` input.
  - Maintains file hierarchy during deployment.

## Configuration Options

### Workflow Inputs

| **Input Name**            | **Description**                                       | **Required** | **Default Value** |
|---------------------------|-------------------------------------------------------|--------------|-------------------|
| `environment_name`        | Environment name for deployment                       | Yes          | None              |
| `artifact_name`           | Name of the SPA artifact to deploy                    | No           | `.spa-app`        |
| `delete_destination`      | Whether to delete files in destination not in source  | No           | `true`            |
| `azure_client_id`         | Client ID for Azure deployment                        | No           | None              |
| `azure_tenant_id`         | Tenant ID for Azure deployment                        | No           | None              |
| `azure_subscription_id`   | Subscription ID for Azure deployment                  | No           | None              |
| `azure_storage_account`   | Azure Storage account name for deployment             | No           | None              |
| `azure_storage_container` | Azure Storage container name for deployment           | No           | None              |

### Required Variables or Inputs

The workflow requires the following Azure configuration values to be provided either as:

- **Workflow inputs** (as shown in the table above), or
- **Repository/Environment variables** with these names:

| **Variable Name**         | **Description**                             | **Required** |
|---------------------------|---------------------------------------------|--------------|
| `AZURE_CLIENT_ID`         | Client ID for Azure deployment              | Yes          |
| `AZURE_TENANT_ID`         | Tenant ID for Azure deployment              | Yes          |
| `AZURE_SUBSCRIPTION_ID`   | Subscription ID for Azure deployment        | Yes          |
| `AZURE_STORAGE_ACCOUNT`   | Azure Storage account name for deployment   | Yes          |
| `AZURE_STORAGE_CONTAINER` | Azure Storage container name for deployment | Yes          |

The workflow will first check for values provided as inputs, and if not found, will fall back to repository or environment variables. If neither is available, the workflow will fail with a validation error.

## Example Usage Scripts

### 1. Basic Deployment to Production

```yaml
name: Deploy to Azure Storage

on:
  push:
    branches:
      - main

jobs:
  deploy-to-storage:
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-azurestorage.yml@v2
    permissions:
      contents: read
      id-token: write
    with:
      environment_name: Production
```

The above example assumes that the necessary Azure configuration values are stored as GitHub environment variables.

### 2. Deployment to Staging Environment with Custom Artifact

```yaml
name: Deploy to Staging

on:
  pull_request:
    branches:
      - main

jobs:
  deploy-to-storage:
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-azurestorage.yml@v2
    permissions:
      contents: read
      id-token: write
    with:
      environment_name: Staging
      artifact_name: staging-spa-build
      delete_destination: false
      azure_storage_account: mystagingsite
      azure_storage_container: static
      azure_client_id: ${{ secrets.AZURE_STAGING_CLIENT_ID }}
      azure_tenant_id: ${{ secrets.AZURE_STAGING_TENANT_ID }}
      azure_subscription_id: ${{ secrets.AZURE_STAGING_SUBSCRIPTION_ID }}
```

The above example shows how to provide Azure configuration values as workflow inputs, pulling from repository secrets.

## Notes

- The workflow expects an artifact containing the built static content to be available from a previous job.
- Uses AzCopy's sync command with `--delete-destination` which defaults to `true` but can be overridden as needed.
- Requires appropriate Azure RBAC permissions for the provided service principal (Client ID) to manage blob storage.
- The `permissions: id-token: write` is required for Azure deployment workflows to enable OIDC authentication with Azure.
