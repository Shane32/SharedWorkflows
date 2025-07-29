# Configuring Azure Permissions for GitHub Actions

This guide explains how to set up the necessary permissions and credentials to allow GitHub Actions workflows to deploy to Azure resources.

## Prerequisites

- Access to Azure Portal with permissions to create managed identities and assign roles
- Access to GitHub repository settings to configure environments and secrets
- Azure subscription where the resources are deployed

## Steps for Production Environment

1. Create User-Assigned Managed Identity
   - Navigate to Azure Portal
   - Search for "Managed Identities" in the search bar
   - Click "Create"
   - Select your subscription and resource group
   - Give it a name (e.g., `myapp-identity`)
   - Select the same region as your resources
   - Click "Create"

:warning: The subscription must match the resource you will be publishing to :warning:

2. Configure Role Assignments

   ### For Azure Web App
   - Navigate to your Azure Web App in the Azure Portal
   - Click on "Access control (IAM)"
   - Click "Add" then "Add role assignment"
   - Select the "Contributor" role
   - In the Members tab, select "Managed identity" for Assign access to
   - Select your managed identity from the list
   - Click "Review + assign"

   ### For Azure Blob Storage
   - Navigate to your Storage Account in the Azure Portal
   - Click on "Containers" and select the specific container you want to grant access to
   - Click on "Access Control (IAM)"
   - Click "Add" then "Add role assignment"
   - Select the "Storage Blob Data Contributor" role
   - In the Members tab, select "Managed identity" for Assign access to
   - Select your managed identity from the list
   - Click "Review + assign"

3. Configure Federated Credentials
   - In the managed identity, go to "Federated credentials"
   - Click "Add credential"
   - Select "GitHub Actions deploying Azure resources" as the scenario
   - Fill in the details:
     - Organization: Your GitHub organization name
     - Repository: Your repository name
     - Entity type: Environment
     - Environment name: `Production` (or as needed by the workflow)
     - Name: A descriptive name (e.g., `myapp-identity-cred`)
   - Click "Add"

4. Configure GitHub Environment
   - Go to your GitHub repository
   - Navigate to Settings > Environments
   - Click "New environment"
   - Name it `Production` (or as needed by the workflow)
   - Add any required environment protection rules
   - Add the following environment variables:
     - `AZURE_CLIENT_ID`: The client ID of the managed identity (found in Managed Identity > Overview)
     - `AZURE_SUBSCRIPTION_ID`: Your Azure subscription ID (found in Managed Identity > Overview)
     - `AZURE_TENANT_ID`: Your Azure tenant ID (found in Managed Identity > Settings > Properties)
     - `AZURE_WEBAPP_NAME`: The name of your Azure Web App, if applicable
     - `AZURE_WEBAPP_SLOT_NAME`: The slot name of your Azure Web App (defaults to `Production`), if applicable
     - `AZURE_STORAGE_ACCOUNT`: The storage account name to publish the SPA to, if applicable
     - `AZURE_STORAGE_CONTAINER`: The storage container name to publish the SPA to, if applicable

## Steps for Development Environment

Repeat the above steps with these modifications:

1. Create a separate managed identity (e.g., `myapp-dev-identity`) in the proper subscription matching the resources
2. Configure the same role assignments as needed
3. Create federated credentials using:
   - Environment name: `Development`
4. In GitHub:
   - Create a new environment named `Development`
   - Add the same environment variables but with the development managed identity details

## Workflow Configuration

Example workflow configurations using the [Publish Application Workflow](publish-app.md):

### Development Environment (on push to master)

```yaml
name: Deploy to Development

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
    with:
      environment_name: Development
      dotnet_folder: '.'
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
```

### Production Environment (on release)

```yaml
name: Deploy to Production

on:
  release:
    types:
      - published

jobs:
  publish-application:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app.yml@v2
    with:
      environment_name: Production
      dotnet_folder: '.'
    secrets:
      NUGET_ORG_USER: ${{ secrets.NUGET_ORG_USER }}
      NUGET_ORG_TOKEN: ${{ secrets.NUGET_ORG_TOKEN }}
```

## Verification

To verify the setup:
1. Create a test workflow using the credentials
2. Trigger the workflow
3. Check the workflow logs to ensure authentication succeeds
4. Verify the deployment completes successfully

## Troubleshooting

Common issues and solutions:
- If authentication fails, verify the federated credential configuration matches exactly
- Ensure the managed identity has the necessary role assignments
- Check that environment variables are correctly set in GitHub
- Verify the environment name in the workflow matches the federated credential configuration
- Ensure that the resource is in the same subscription as the managed identity
