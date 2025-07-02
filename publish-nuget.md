# Publish NuGet Package Workflow

This reusable GitHub Actions workflow automates the process of building and publishing .NET NuGet packages. It leverages the existing `build-dotnet` workflow to create NuGet packages and then intelligently publishes them to either NuGet.org or GitHub Packages based on the build context and configuration.

## Functionality Summary

The **Publish NuGet Package** workflow includes the following features:

- **NuGet Package Build**:
  - Uses the existing `build-dotnet` workflow with `pack: 'true'` to create NuGet packages.
  - Supports all build-dotnet features including version management, private NuGet sources, and environment variables.
  - Automatically handles NuGet package versioning based on tags or run numbers.

- **Smart Publishing Logic**:
  - **Release Builds**: Automatically publishes to NuGet.org when `NUGET_AUTH_TOKEN` is available, falls back to GitHub Packages otherwise.
  - **Non-Release Builds**: Publishes to GitHub Packages by default for safe development workflows.
  - **Override Option**: Provides `publish_to_nuget` input to force NuGet.org publishing for non-release builds.
  - **Safe with `secrets: inherit`**: Won't accidentally publish development builds to NuGet.org.

- **Multi-Package Support**:
  - Publishes all `.nupkg` files found in the build output.
  - Uses `--skip-duplicate` to handle cases where packages already exist.

- **Integrated Authentication**:
  - Configures .NET Core with the appropriate source URL and authentication token.
  - Uses `NUGET_AUTH_TOKEN` environment variable for seamless authentication.

## Configuration Options

### Workflow Inputs

| **Input Name**          | **Description**                                          | **Required** | **Default Value**   | **Comments** |
|------------------------|-----------------------------------------------------------|--------------|---------------------|--------------|
| `dotnet_folder`        | Path to the .NET project folder                           | Yes          | None                |              |
| `global_json_folder`   | Path to the folder containing `global.json`               | No           | `.`                 | Used for SDK version |
| `nuget_accounts`       | Comma-separated list of GitHub accounts for NuGet sources | No           | `default`           | Used during build for private packages |
| `dotnet_configuration` | .NET build configuration (e.g., `Release`, `Debug`)       | No           | `Release`           |              |
| `environment_name`     | Environment name for deployment                           | No           | None                |              |
| `publish_to_nuget`     | Force publishing to NuGet.org instead of GitHub Packages  | No           | `false`             | Requires `NUGET_AUTH_TOKEN` |

### Environment Variables

If an environment is configured, variables configured for the specified environment are used as environment variables when building the NuGet packages.

### Secrets

| **Secret Name**       | **Description**                                 | **Required** | **Comments** |
|-----------------------|-------------------------------------------------|--------------|--------------|
| `NUGET_AUTH_TOKEN`    | Token for publishing to NuGet.org               | No           | Used for releases or when `publish_to_nuget=true` |
| `NUGET_ORG_USER`      | Username for private NuGet source (build only)  | No           |              |
| `NUGET_ORG_TOKEN`     | Token for private NuGet source (build only)     | No           |              |

### Outputs

| **Output Name** | **Description**                    | **Example Value** |
|-----------------|------------------------------------|-------------------|
| `version`       | The assembly version number used for build  | `1.2.3.0` or `0.0.0.123` |

## Publishing Behavior

The workflow uses intelligent logic to determine the publishing target:

### Default Behavior
- **Release Events** (e.g., `on: release: types: [published]`):
  - If `NUGET_AUTH_TOKEN` is available → Publishes to **NuGet.org**
  - If `NUGET_AUTH_TOKEN` is not available → Publishes to **GitHub Packages**

- **Non-Release Events** (e.g., `on: push`, `on: workflow_dispatch`):
  - Always publishes to **GitHub Packages** (safe for development builds)

### Override Behavior
- **When `publish_to_nuget: true`**:
  - Forces publishing to **NuGet.org** regardless of event type
  - Requires `NUGET_AUTH_TOKEN` to be provided (fails if missing)

### Publishing Targets

**NuGet.org Publishing:**
- URL: `https://api.nuget.org/v3/index.json`
- Authentication: Uses `NUGET_AUTH_TOKEN`
- Suitable for: Public packages or private packages with NuGet.org subscription

**GitHub Packages Publishing:**
- URL: `https://nuget.pkg.github.com/{repository_owner}/index.json`
- Authentication: Uses `GITHUB_TOKEN` (automatically provided by GitHub Actions)
- Repository owner: Automatically extracted from `${{ github.repository_owner }}`
- Suitable for: Private packages within GitHub organizations

## Example Usage Script

This safely publishes push events to GitHub Packages and release events to NuGet.org (if `NUGET_AUTH_TOKEN` is available), otherwise to GitHub Packages:

```yaml
name: Build and Publish

on:
  push:
    branches: [main]
  release:
    types: [published]

jobs:
  publish:
    uses: Shane32/SharedWorkflows/.github/workflows/publish-nuget.yml@v1
    with:
      dotnet_folder: '.'
    secrets: inherit
```

## NuGet Package Versioning

The workflow uses the same versioning strategy as the `build-dotnet` workflow:

### Version Prefix Setup

NuGet packages should have a `VersionPrefix` property in their `Directory.Build.props` file:

```xml
<Project>
  <PropertyGroup>
    <VersionPrefix>1.0.0-preview</VersionPrefix>
  </PropertyGroup>
</Project>
```

### Versioning Behavior

- **For tagged releases** (e.g., tag `v3.0.5`):
  - Uses `-p:Version="3.0.5"` to override the VersionPrefix
  - Results in NuGet package version: `3.0.5`

- **For non-release builds** (e.g., run number 304):
  - Uses `-p:VersionSuffix="304"` to append to the VersionPrefix
  - With VersionPrefix `1.0.0-preview`, results in NuGet package version: `1.0.0-preview-304`

## Directory.Build.props Configuration

For optimal NuGet package publishing, consider using a comprehensive `Directory.Build.props` file at your repository root:

```xml
<Project>

  <PropertyGroup>
    <VersionPrefix>1.0.0-preview</VersionPrefix>
    <Copyright>Your Name</Copyright>
    <Authors>Your Name</Authors>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageIcon>logo.64x64.png</PackageIcon>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <RepositoryType>git</RepositoryType>
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
    <Deterministic>true</Deterministic>
    <ContinuousIntegrationBuild Condition="'$(GITHUB_ACTIONS)' == 'true'">True</ContinuousIntegrationBuild>
    <DebugType>embedded</DebugType>
    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <None Include="..\..\logo.64x64.png" Pack="true" PackagePath="\" Condition="'$(IsPackable)' == 'true'"/>
    <None Include="..\..\README.md" Pack="true" PackagePath="\" Condition="'$(IsPackable)' == 'true'"/>
  </ItemGroup>

</Project>
```

> **Note**: The above sample assumes that a logo and README file are located at the repository root while the project is in `src/MyLibrary`. Adjust the relative paths (`..\..\logo.64x64.png` and `..\..\README.md`) based on your actual project structure.

### Key Properties for NuGet Packages

- **`VersionPrefix`**: Base version used by the workflow's versioning logic
- **`IsPackable`**: Set to `true` in individual project files that should be packaged as NuGet packages
- **`Authors`** and **`Copyright`**: Package metadata for attribution
- **`PackageLicenseExpression`**: License information (e.g., `MIT`, `Apache-2.0`)
- **`PackageIcon`** and **`PackageReadmeFile`**: Visual branding and documentation
- **`GenerateDocumentationFile`**: Creates XML documentation for IntelliSense
- **`PublishRepositoryUrl`**: Enables source linking for debugging
- **`Deterministic`** and **`ContinuousIntegrationBuild`**: Ensures reproducible builds in CI/CD
- **`DebugType`**: Set to `embedded` to include debug symbols directly in the assembly for better debugging experience

### Sample Project File

For individual library projects that should be packaged, set `IsPackable` to `true` in the `.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <IsPackable>true</IsPackable>
    <Description>A useful library for doing amazing things</Description>
    <PackageTags>library;utility;awesome</PackageTags>
  </PropertyGroup>

</Project>
```

**Key Project-Specific Properties:**
- **`IsPackable`**: Must be `true` to create a NuGet package for this project
- **`Description`**: Brief description of what the package does
- **`PackageTags`**: Semicolon-separated tags for discoverability

## Notes

- `global.json` is required in the specified `global_json_folder`
- The workflow automatically publishes all `.nupkg` files found in the build output
- Uses `--skip-duplicate` to prevent errors when packages already exist
- `NUGET_ORG_USER` and `NUGET_ORG_TOKEN` are only used during the build phase for accessing private NuGet sources
- The workflow runs on Ubuntu latest
- Authentication is automatically configured based on the target repository
- For GitHub Packages, the repository owner is automatically extracted from the GitHub context
