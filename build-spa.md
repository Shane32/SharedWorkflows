# Build SPA Application Workflow

This reusable GitHub Actions workflow automates the process of building Single Page Applications (SPA). It handles dependency installation, building the application, and artifact management, with support for version environment variables and persisted documents.

## Functionality Summary

The **Build SPA** workflow includes the following features:

- **Version Management**:
  - Automatically generates version numbers based on tags or run numbers.
  - For tagged releases: uses tag value with `.0` suffix.
  - For other builds: uses format `0.0.0.{run-number}`.
  - Configurable environment variable name for version information

- **Node.js Setup**:
  - Automatically detects and uses Node.js version from package.json
  - Ensures consistent build environment

- **Build Process**:
  - Installs NPM dependencies using `npm ci` for reliable builds
  - Executes configurable NPM build script
  - Supports custom distribution folder paths

- **Artifact Management**:
  - Uploads built SPA as an artifact
  - Optional support for persisted documents file
  - Configurable artifact names

- **Analysis Support**:
  - Optional post-build analysis script execution
  - Uploads analysis artifacts separately
  - Configurable analysis file patterns

## Configuration Options

### Workflow Inputs

| **Input Name**                 | **Description**                                                  | **Required** | **Default Value**     | **Comments** |
|-------------------------------|-------------------------------------------------------------------|--------------|-----------------------|--------------|
| `spa_folder`                  | Path to the SPA project folder                                    | Yes          | None                  |              |
| `spa_version_env`             | Environment variable name to store SPA version                    | No           | `VITE_VERSION`        | e.g. `REACT_APP_VERSION` for React |
| `npm_build_script`            | NPM script used to build the SPA application                      | No           | `build`               |              |
| `npm_dist_folder`             | Path to the SPA distribution folder                               | No           | `dist`                |              |
| `artifact_name`               | Name of the artifact to upload                                    | No           | `.spa-app`            |              |
| `persisted_documents_file`    | Path to the persisted documents file                              | No           | None                  |              |
| `persisted_documents_artifact`| Name of the persisted documents artifact                          | No           | `persisted-documents` |              |
| `environment_name`            | Environment name for deployment                                   | No           | None                  |              |
| `npm_analyze_script`          | NPM analyze script to run after build                             | No           | None                  |              |
| `analysis_files`              | Full path to files to upload as analysis artifacts (glob pattern) | No           | None                  |              |
| `analysis_artifact`           | Name of the analysis artifact to upload                           | No           | `analysis`            |              |
| `npm_accounts`                | Comma-separated list of GitHub accounts to add as NPM sources     | No           | Repository owner      | Skipped if tokens are not provided |

### Secrets

The workflow supports the following secrets, which are optional but enhance functionality:

- `NPM_TOKEN`: GitHub Personal Access Token for accessing private npm packages.

### Environment Variables

If an environment is configured, variables configured for the specified environment are used as environment variables when building the SPA.

## Example Usage Scripts

### 1. Basic SPA Build

```yaml
name: Build SPA

on:
  push:
    branches:
      - master

jobs:
  build-spa:
    uses: Shane32/SharedWorkflows/.github/workflows/build-spa.yml@v1
    with:
      spa_folder: '.'
```

### 2. Build with Custom Configuration

```yaml
name: Build SPA with Custom Config

on:
  push:
    branches:
      - develop

jobs:
  build-spa:
    uses: Shane32/SharedWorkflows/.github/workflows/build-spa.yml@v1
    with:
      spa_folder: 'ReactApp'
      spa_version_env: 'REACT_APP_VERSION'
      npm_build_script: 'build:prod'
      npm_dist_folder: 'build'
      artifact_name: 'spa-production-build'
      environment_name: 'Development'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### 3. Build with Persisted Documents

```yaml
name: Build SPA with Persisted Documents artifact

on:
  push:
    branches:
      - master

jobs:
  build-spa:
    uses: Shane32/SharedWorkflows/.github/workflows/build-spa.yml@v1
    with:
      spa_folder: 'ClientApp'
      persisted_documents_file: 'ClientApp/src/gql/persisted-documents.json'
      persisted_documents_artifact: 'persisted-docs'
```

### 4. Build with Analysis

```yaml
name: Build SPA with Analysis

on:
  push:
    branches:
      - master

jobs:
  build-spa:
    uses: Shane32/SharedWorkflows/.github/workflows/build-spa.yml@v1
    with:
      spa_folder: 'ClientApp'
      npm_analyze_script: 'analyze'
      analysis_files: 'ClientApp/analysis/**/*'
      analysis_artifact: 'bundle-analysis'
```

## Notes

- The workflow automatically detects the Node.js version from the project's package.json file
- Version numbering follows the pattern:
  - For tags: `{tag}.0` (e.g., `1.2.3` becomes `1.2.3.0`)
  - For regular builds: `0.0.0.{run-number}`
- When using `persisted_documents_file`, the workflow will fail if the specified file is not found
- The workflow runs in the specified environment context so custom environment variables can be added within GitHub's environment configuration
- Analysis artifacts are only uploaded if both `npm_analyze_script` and `analysis_files` are provided
