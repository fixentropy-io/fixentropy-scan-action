# Fixentropy Scan Action

A GitHub Action to scan your code with [Fixentropy.io](https://fixentropy.io). This action uses Dagger under the hood to run the scanning pipeline and report the results to the Fixentropy backend.

## Features

- Scans your codebase using specific Fixentropy asserters.
- Securely authenticates with Fixentropy using GitHub OIDC tokens limitating the need for long-lived secrets.
- Uses bundled CLI binaries from `bin/` through `bin/cli.sh` (no build/download during action run).
- Runs in an isolated environment powered by Dagger.

## Usage

To use this action in your repository, you need to create a workflow file (e.g., `.github/workflows/fixentropy-scan.yml`) and add the following configuration:

### Basic Example

```yaml
name: Fixentropy Scan

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      id-token: write # Required for OIDC authentication with Fixentropy
      contents: read # Required to checkout the code and read contents

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Run Fixentropy Scan
        uses: fixentropy-io/scan-action@v1 # Replace with the correct reference/version
        with:
          asserter: "my-org/my-asserter" # Optional: Specify the asserter if needed
```

## Inputs

| Parameter     | Type       | Required | Default                         | Description                                                                                                                     |
| :------------ | :--------- | :------- | :------------------------------ | :------------------------------------------------------------------------------------------------------------------------------ |
| `asserter`    | **string** | No       | N/A                             | The asserter to use, formatted as `<org>/<asserter>`. If omitted, default asserters may be used based on project configuration. |
| `source`      | **string** | No       | `${{ github.workspace }}`       | The source directory to scan.                                                                                                   |
| `backend-url` | **string** | No       | `https://app.fixentropy.io/api` | The Fixentropy backend URL. You usually don't need to change this unless you are using a self-hosted instance.                  |

## Permissions

For the action to securely communicate with the Fixentropy backend using OIDC (OpenID Connect), the job running the action **must** have the following permissions:

```yaml
permissions:
  id-token: write
  contents: read
```

- `id-token: write`: Allows the action to request an OIDC JWT token from GitHub which is then used to authenticate with Fixentropy.io securely without needing a static API key.
- `contents: read`: Allows the workflow to read the repository contents (and the Dagger module).

## Incorporation in Project Context

### 1. Identify what to scan

Decide which directory in your project requires scanning. This will be the value for the `from` input. For monolithic repositories, you can run multiple scan steps or matrix jobs for different components.

### 2. Add Workflow File

Create a new file at `.github/workflows/fixentropy.yml` in your project's repository.

### 3. Provide the Asserter (If applicable)

If you are using custom or specific community asserters (e.g., `fixentropy-io/hexagonal-asserter`), define it using the `asserter` input so that the scan is accurately tailored to your architecture.

### 4. Trigger on PRs

It is highly recommended to run this action on `pull_request` events to catch architectural drifts or issues before they are merged into your main branch.

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
```

## Troubleshooting

- **OIDC Token Error:** Ensure your job explicitly declares `permissions: { id-token: write, contents: read }`.
- **Dagger Execution Failed:** The action relies on Dagger. Make sure your GitHub runners can execute Docker containers, as Dagger uses containers to isolate the pipeline execution.
- **Binary Not Executable:** Ensure files in `bin/` keep executable permissions, especially `bin/cli.sh` and platform binaries.
