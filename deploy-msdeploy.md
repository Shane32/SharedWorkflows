# Deploy with MSDeploy Workflow

This reusable GitHub Actions workflow automates the process of deploying applications to IIS servers using Microsoft Web Deploy (MSDeploy). It supports deploying both .NET applications and SPA (Single Page Application) projects, with the ability to handle persisted documents.

## Functionality Summary

The **Deploy with MSDeploy** workflow includes the following features:

- **Artifact Deployment**:
  - Downloads and deploys artifacts to the root of the IIS site
  - Optionally deploys SPA artifacts to the wwwroot subfolder (for combined .NET+SPA deployments)
  - Optionally deploys persisted documents file(s) to the root folder
  - Supports SPA-only deployments by using `artifact_name` instead of `spa_artifact`

- **IIS Integration**:
  - Uses MSDeploy (Microsoft Web Deploy) for deployment
  - Deploys to specified IIS site
  - Supports Basic authentication
  - Configurable SSL certificate validation

- **Environment Management**:
  - Configurable environment names for deployment
  - Supports different configurations per environment

## Configuration Options

### Workflow Inputs

| **Input Name**                 | **Description**                                            | **Required** | **Default Value** |
|--------------------------------|------------------------------------------------------------|--------------|-------------------|
| `environment_name`             | Environment name for deployment                            | Yes          | None              |
| `artifact_name`                | Name of the artifact to deploy to the root of the site     | No           | `.net-app`        |
| `spa_artifact`                 | Name of the SPA artifact to deploy to wwwroot subfolder (for combined .NET+SPA) | No | None |
| `persisted_documents_artifact` | Name of the persisted documents artifact to deploy to root | No           | None              |
| `msdeploy_server_url`          | MSDeploy server URL for deployment                         | No           | None              |
| `msdeploy_site_name`           | IIS site name for deployment                               | No           | None              |
| `msdeploy_username`            | Username for MSDeploy authentication                       | No           | None              |
| `msdeploy_allow_untrusted`     | Allow untrusted SSL certificates                           | No           | `false`           |

### Required Variables, Inputs, or Secrets

The workflow requires the following MSDeploy configuration values to be provided either as:

- **Workflow inputs** (as shown in the table above)
- **Repository/Environment variables** with these names
- **Secrets** (for sensitive values like passwords)

| **Variable Name**       | **Description**                        | **Required** | **Default Value** |
|-------------------------|----------------------------------------|--------------|-------------------|
| `MSDEPLOY_SERVER_URL`   | MSDeploy server URL for deployment     | Yes          | None              |
| `MSDEPLOY_SITE_NAME`    | IIS site name for deployment           | Yes          | None              |
| `MSDEPLOY_USERNAME`     | Username for MSDeploy authentication   | Yes          | None              |
| `MSDEPLOY_PASSWORD`     | Password for MSDeploy authentication   | Yes          | None              |

The workflow will first check for values provided as inputs or secrets, and if not found, will fall back to repository or environment variables. If neither is available, the workflow will fail with a validation error.

### Secrets

| **Secret Name**           | **Description**                                       | **Required** | **Default Value** |
|---------------------------|-------------------------------------------------------|--------------|-------------------|
| `msdeploy_password`       | Password for MSDeploy authentication                  | No           | None              |

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
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-msdeploy.yml@v2
    permissions:
      contents: read
    with:
      environment_name: Production
```

The above example assumes that the necessary MSDeploy configuration values are stored as GitHub environment variables.

### 2. Deploy Full Stack Application with Custom Settings

```yaml
name: Deploy to Staging

on:
  pull_request:
    branches:
      - main

jobs:
  deploy:
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-msdeploy.yml@v2
    permissions:
      contents: read
    with:
      environment_name: Staging
      artifact_name: backend-build
      spa_artifact: frontend-build
      persisted_documents_artifact: persisted-docs
      msdeploy_server_url: https://staging-server.example.com:8172/msdeploy.axd
      msdeploy_site_name: StagingSite
      msdeploy_username: deploy-user
      msdeploy_allow_untrusted: true
    secrets:
      msdeploy_password: ${{ secrets.MSDEPLOY_STAGING_PASSWORD }}
```

The above example shows how to provide MSDeploy configuration values as workflow inputs, with the password passed as a secret.

### 3. Deploy with Custom Artifact Names

```yaml
name: Deploy Custom Build

on:
  workflow_dispatch:

jobs:
  deploy:
    uses: Shane32/SharedWorkflows/.github/workflows/deploy-msdeploy.yml@v2
    permissions:
      contents: read
    with:
      environment_name: Development
      artifact_name: api-artifact
      spa_artifact: web-ui-artifact
    secrets:
      msdeploy_password: ${{ secrets.MSDEPLOY_PASSWORD }}
```

## Notes

- The workflow runs on Windows runners as MSDeploy is a Windows-based tool
- Basic authentication is used for MSDeploy connections
- The `msdeploy_allow_untrusted` option should only be set to `true` for development/staging environments with self-signed certificates
- The main artifact (via `artifact_name`) deploys to the root of the IIS site
- SPA artifacts (via `spa_artifact`) deploy to the wwwroot subfolder - use this for combined .NET+SPA deployments
- For SPA-only deployments, use `artifact_name` to deploy the SPA to the root instead of using `spa_artifact`
- Persisted document artifacts automatically deploy to the root folder when specified
- All artifacts must be previously created and available in the workflow run
- The workflow uses DoNotDeleteRule to preserve files not in the source package
- The workflow uses AppOffline rule to take the application offline during deployment

## MSDeploy Server URL Format

The MSDeploy server URL typically follows this format:

```text
https://server-name:8172/msdeploy.axd
```

Where:
- `server-name` is your IIS server hostname or IP address
- `8172` is the default MSDeploy port (may vary based on your configuration)
- `/msdeploy.axd` is the MSDeploy handler endpoint

## IIS Site Name

The IIS site name should match the exact name of the website or application in IIS where you want to deploy. For deploying to a specific application within a site, use the format:

```text
SiteName/ApplicationName
```
