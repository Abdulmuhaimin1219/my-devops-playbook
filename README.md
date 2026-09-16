# DevOps Playbook

Reusable GitHub Actions workflows for building, pushing, and deploying applications to Azure Kubernetes Service (AKS).

The goal is to keep CI/CD logic reusable across different application repositories while keeping environment-specific configuration in the application repository.

## Architecture

 The Architecture will be added later

        
      

## Reusable Workflows

### Build & Push

`reusable-build-push-acr.yml`

Handles:

* Application tests
* Docker image build
* Azure authentication
* ACR login
* Image push

The workflow currently uses the GitHub Actions `run_number` as the image tag.

Example:

```text
abdul.azurecr.io/go-aks-demo:17
```

### Deploy to AKS

`reusable-deploy-aks.yml`

Handles:

* Azure authentication
* AKS access
* Container image validation
* Helm linting
* Helm deployment
* Deployment verification

The deployment workflow **does not rebuild the application**. It deploys the image tag provided to it.

## Environments

The same deployment workflow can be used for:

```text
DEV
STAGING
PROD
```

Environment-specific differences are passed through workflow inputs and Helm values files.

For example:

```text
values-dev.yaml
values-staging.yaml
values-prod.yaml
```

Production follows the existing release process:

```text
Build → ACR → QA → Team Lead → Production
```

Production deployment is not automatically released without the required approval process.

## Repository Structure

```text
.github/
└── workflows/
    ├── reusable-build-push-acr.yml
    └── reusable-deploy-aks.yml
```

The workflows are maintained in this repository and can be consumed by application repositories.

Example:

```yaml
jobs:
  build:
    uses: Abdulmuhaimin1219/my-devops-playbook/.github/workflows/reusable-build-push-acr.yml@v1
```

## Current Azure Setup

The workflows are currently designed around:

* Azure Container Registry (ACR)
* Azure Kubernetes Service (AKS)
* GitHub Actions
* Azure OIDC authentication
* Helm
* Self-hosted GitHub Actions runners

Example in azure

```text
ACR: abdl-gitacr
AKS: abdl-gitaks
Resource Group: abdl-gitrg
```

## Security

The GitHub Actions identity currently has:

```text
AcrPush
Azure Kubernetes Service Cluster User Role
```

on the required Azure resources.

Secrets such as Azure credentials are passed through GitHub Actions secrets, while authentication uses OIDC rather than long-lived Azure client secrets.

## Future Improvements

Planned improvements include:

* Evaluate `kubelogin` for non-admin AKS authentication.
* Move from image tags to immutable image digests.
* Add stronger deployment concurrency controls.
* Add additional security and policy gates where required.

## Principle

> **CI creates the artifact. CD promotes the exact artifact.**

The deployment workflow should never rebuild an application that has already passed CI.
