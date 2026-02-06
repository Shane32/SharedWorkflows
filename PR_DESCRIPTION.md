# Add MSDeploy Deployment Workflows

## Summary

This PR adds new reusable GitHub Actions workflows for deploying applications to IIS servers using Microsoft Web Deploy (MSDeploy). These workflows provide an alternative deployment method for projects that use on-premises or self-hosted IIS infrastructure instead of Azure.

## New Files

### Workflows

1. **`.github/workflows/deploy-msdeploy.yml`** - Reusable workflow for deploying applications to IIS via MSDeploy
   - Supports .NET application deployment
   - Optionally deploys SPA artifacts to wwwroot folder
   - Optionally deploys persisted documents
   - Uses Basic authentication with configurable SSL validation
   - Runs on Windows runners (required for MSDeploy)

2. **`.github/workflows/publish-app-msbuild.yml`** - Complete build and deploy workflow using MSDeploy
   - Builds .NET and/or SPA projects
   - Automatically combines artifacts when both are present
   - Deploys to IIS using MSDeploy
   - Publishes release assets for GitHub releases
   - Version references set to `@2.3.0`

### Documentation

1. **`deploy-msdeploy.md`** - Comprehensive documentation for the MSDeploy deployment workflow
   - Configuration options and inputs
   - Three example usage scenarios
   - MSDeploy server URL format guidance
   - IIS site name configuration tips

2. **`publish-app-msbuild.md`** - Complete guide for the publish-app-msbuild workflow
   - Full workflow capabilities overview
   - Build and deployment configuration
   - Example usage for .NET-only, SPA-only, and full-stack deployments
   - Notes on differences from Azure deployment workflows

## Key Features

### MSDeploy Deployment Workflow
- **Windows-based**: Runs on `windows-latest` runners to support MSDeploy CLI
- **Flexible Configuration**: Accepts deployment settings via inputs or environment variables
- **Artifact Management**: Downloads and combines .NET apps, SPA builds, and persisted documents
- **Configurable Security**: Optional `msdeploy_allow_untrusted` flag for self-signed certificates
- **Consistent with Azure Workflows**: Uses environment variables instead of secrets for credentials

### Publish Application Workflow
- **No Deployment Method Input**: Unlike the Azure version, this workflow doesn't need a `spa_deployment_method` input—it always combines artifacts when both .NET and SPA are present
- **Automatic Combination**: SPA artifacts are automatically placed in wwwroot when deploying alongside .NET apps
- **Conditional Execution**: Skips builds and deployments based on what's configured
- **Release Assets**: Automatically creates and uploads release assets for GitHub releases
- **Configuration Validation**: Checks for required MSDeploy settings before proceeding

## Configuration

### Required Variables/Inputs

The workflows require the following MSDeploy configuration (either as inputs or environment variables):

| Variable | Description | Required |
|----------|-------------|----------|
| `MSDEPLOY_SERVER_URL` | MSDeploy server URL (e.g., `https://server:8172/msdeploy.axd`) | Yes |
| `MSDEPLOY_SITE_NAME` | IIS site name for deployment | Yes |
| `MSDEPLOY_USERNAME` | Username for MSDeploy authentication | Yes |
| `MSDEPLOY_PASSWORD` | Password for MSDeploy authentication | Yes |

### Optional Configuration

| Input | Description | Default |
|-------|-------------|---------|
| `msdeploy_allow_untrusted` | Allow untrusted SSL certificates | `false` |
| `artifact_name` | Name of the .NET artifact | `.net-app` |
| `spa_artifact` | Name of the SPA artifact | None |
| `persisted_documents_artifact` | Name of persisted documents artifact | None |

## Usage Example

```yaml
name: Deploy to Production

on:
  release:
    types:
      - published

jobs:
  publish-application:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app-msbuild.yml@v2.3.0
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

## Design Decisions

1. **Environment Variables Over Secrets**: MSDeploy credentials are configured as environment variables (accessible via `vars`) rather than requiring a dedicated `secrets` block in the workflow, keeping consistency with the Azure Web App deployment workflow.

2. **Simplified Deployment Model**: The publish-app-msbuild workflow doesn't have a `spa_deployment_method` input because MSDeploy deployments always use the combined approach (SPA in wwwroot).

3. **Windows Runners**: MSDeploy is Windows-specific, so the deployment job runs on `windows-latest` while build jobs can still run on `ubuntu-latest`.

4. **Version Tags**: All internal workflow references use `@2.3.0` to ensure compatibility and allow for future versioning.

## Testing Recommendations

- Test with .NET-only projects
- Test with SPA-only projects  
- Test with combined .NET + SPA projects
- Test with persisted documents
- Verify environment variable fallback behavior
- Test with self-signed certificates (`msdeploy_allow_untrusted: true`)

## Breaking Changes

None - these are new workflows that don't affect existing functionality.

## Related Issues

Closes #[issue-number] (if applicable)
