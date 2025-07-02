# Shared Workflows

This repository contains shared workflows for building and testing applications.
See each workflow's respective readme for more details:

- [Build Check workflow for pull requests and pushing coverage reports](build-check.md)
- [Build SPA workflow for Single Page Applications](build-spa.md)
- [Build .NET workflow for .NET applications](build-dotnet.md)
- [Publish NuGet Package workflow for building and publishing NuGet packages](publish-nuget.md)
- [Deploy Azure Web App workflow for web application deployments](deploy-azurewebapp.md)
- [Deploy Azure Storage workflow for static content deployments](deploy-azurestorage.md)
- [Combination Build and Deploy Application workflow for both .NET and SPA code](publish-app.md)

## Azure Permissions Setup

Before using the deployment workflows, you'll need to configure the necessary Azure permissions and GitHub environments:

- [Configure Azure Permissions and GitHub Environments](configure-permissions.md)

## To make changes to this repository

1. Make required changes and update readme files, being sure to update version number in all samples if the major version is changing
2. Issue release, bumping major version number for breaking changes
3. Adjust/create 'v#' tag to point to new #.x.x release
4. If updating the `add-nuget-sources` workflow, update references to it within the other workflows to use the new version number
5. If updating the `build-spa`, `build-dotnet`, `deploy-azurewebapp` or `deploy-azurestorage` workflows, update references to them in the `publish-app` workflow to use the new version number
