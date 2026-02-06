# Cleanup Dev Stack GitHub Action

A composite GitHub Action that automates the cleanup of development environments by removing both Kubernetes Helm releases and AWS CloudFormation stacks.

## Overview

This action provides a safe and automated way to tear down development stacks that were deployed using Helm (on EKS) and CloudFormation. It includes validation to ensure only development environments (prefixed with `dev-`) can be cleaned up.

## Features

- Validates release names to prevent accidental production cleanup
- AWS authentication via role assumption
- Helm release uninstallation from EKS
- CloudFormation stack deletion with wait
- Resource existence verification (fails if nothing found)
- Idempotent cleanup operations

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `release-name` | Release name (must start with `dev-`) | Yes | - |
| `service-name` | Service name (e.g., `hello-k8s`) | Yes | - |
| `namespace` | Kubernetes namespace | No | `bisnow-apps` |
| `eks-cluster` | EKS cluster name | No | `bisnow-non-prod-eks` |
| `aws-account` | AWS account for role assumption | No | `bisnow` |
| `aws-region` | AWS region | No | `us-east-1` |

## Outputs

| Output | Description |
|--------|-------------|
| `cleanup-status` | Status of cleanup operation (`success` or `not-found`) |

## Usage

### Basic Example

```yaml
- name: Cleanup dev environment
  uses: bisnow/github-actions-cleanup-dev-stack@v1
  with:
    release-name: dev-123
    service-name: hello-k8s
```

### Full Example with Custom Configuration

```yaml
- name: Cleanup custom dev environment
  uses: bisnow/github-actions-cleanup-dev-stack@v1
  with:
    release-name: dev-feature-branch
    service-name: my-service
    namespace: custom-namespace
    eks-cluster: my-eks-cluster
    aws-account: my-aws-account
    aws-region: us-west-2
```

### Complete Workflow Example

```yaml
name: Cleanup PR Environment

on:
  pull_request:
    types: [closed]

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Cleanup dev stack
        uses: bisnow/github-actions-cleanup-dev-stack@v1
        with:
          release-name: dev-${{ github.event.pull_request.number }}
          service-name: my-app
```

## How It Works

1. **Validation**: Ensures the release name starts with `dev-` to prevent accidental production cleanup
2. **AWS Authentication**: Assumes the appropriate AWS role for the specified account
3. **Kubernetes Configuration**: Updates kubeconfig for the target EKS cluster
4. **Resource Cleanup**:
   - Checks for and uninstalls the Helm release: `{release-name}-{service-name}`
   - Checks for and deletes the CloudFormation stack: `{release-name}-{service-name}`
   - Waits for complete stack deletion
5. **Verification**: Fails if no resources were found (prevents silent failures)

## Resource Naming Convention

Resources are named using the pattern: `{release-name}-{service-name}`

Example: If `release-name=dev-123` and `service-name=hello-k8s`, the action will look for:
- Helm release: `dev-123-hello-k8s`
- CloudFormation stack: `dev-123-hello-k8s`

## Error Handling

The action will fail if:
- Release name doesn't start with `dev-`
- No Helm release or CloudFormation stack is found (nothing to clean up)
- AWS authentication fails
- Helm or CloudFormation operations fail

## Versioning

This action uses rolling major version tags. You can pin to:

- A specific version: `@v3.1.0` (exact, never changes)
- A major version: `@v3` (recommended, gets bug fixes and new features)

When a new semantic version tag (e.g., `v3.2.0`) is pushed, a GitHub Actions workflow automatically updates the corresponding major version tag (`v3`) to point to the new release.