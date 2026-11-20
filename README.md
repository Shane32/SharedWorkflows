# Shared Workflows

This repository contains shared workflows for building and testing applications.
See each workflow's respective readme for more details:

- [Build Check workflow for pull requests and pushing coverage reports](build-check.md)
- [Build SPA workflow for Single Page Applications](build-spa.md)
- [Build .NET workflow for .NET applications](build-dotnet.md)
- [Publish NuGet Package workflow for building and publishing NuGet packages](publish-nuget.md)
- [Deploy Azure Web App workflow for web application deployments](deploy-azurewebapp.md)
- [Deploy Azure Storage workflow for static content deployments](deploy-azurestorage.md)
- [Deploy with MSDeploy workflow for deploying to IIS servers](deploy-msdeploy.md)
- [Combination Build and Deploy Application workflow for both .NET and SPA code](publish-app.md)
- [Publish Application with MSDeploy workflow for building and deploying .NET/SPA to IIS](publish-app-msbuild.md)

## Artifact retention

The `build-check`, `build-dotnet`, `build-spa`, `publish-app`, `publish-app-msbuild`, and `publish-nuget` workflows accept an optional numeric `intermediate-retention-days` input. For example, keep the intermediate files from a release build for three days:

```yaml
jobs:
  publish:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-app-msbuild.yml@v3
    with:
      dotnet_folder: 'MyAppServer'
      environment_name: Production
      intermediate-retention-days: 3
    secrets:
      msdeploy_password: ${{ secrets.MSDEPLOY_PASSWORD }}
```

Omitting either input or setting it to `0` uses the calling repository's default retention. A positive value controls intermediate Actions artifacts: the `.net-app` build output (or packed NuGet files), `.spa-app` frontend output, `persisted-documents` manifest, and raw `TestResults`, where applicable. The repository and organization retention limits still apply.

The `Code Coverage Artifacts`, `analysis`, and `Analysis Artifacts` uploads keep the repository's default retention. Raw `TestResults` feed the coverage job and may also be useful for diagnosis; their retention follows the intermediate setting.

Publishing workflows forward this setting to their build workflows. It applies to newly uploaded Actions artifacts; existing artifacts keep their expiration dates. It does not expire files attached to GitHub Releases or packages published to a package registry. See [GitHub's artifact retention documentation](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/remove-workflow-artifacts).

The `@v3` examples require the tag to be updated to a release containing this input. Before that release, pin a commit containing the change. Older pinned versions, such as `2.3.0`, do not accept these retention inputs.

## Azure Permissions Setup

Before using the deployment workflows, you'll need to configure the necessary Azure permissions and GitHub environments:

- [Configure Azure Permissions and GitHub Environments](configure-permissions.md)

## To make changes to this repository

1. Make required changes and update readme files, being sure to update version number in all samples if the major version is changing
2. Issue release, bumping major version number for breaking changes
3. Adjust/create 'v#' tag to point to new #.x.x release
4. If updating the `add-nuget-sources` or `add-npm-sources` workflows, update references to them within the other workflows to use the new version number
5. If updating the `deploy-azurewebapp` or `deploy-azurestorage` workflows, update references to them in the `publish-app` workflow to use the new version number
6. Keep the `$/` self repository builder references in `publish-app`, `publish-app-msbuild`, and `publish-nuget` so each publishing workflow uses builders from the same SharedWorkflows commit, including when called from another repository. See [reusable workflow reference syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_iduses).
