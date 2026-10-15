# Reusable GitHub Actions Workflow library

This repository centralises the common build, container publishing, and Helm deployment steps. Consumer repositories call the workflows from their own workflow files; the caller owns the event trigger and supplies repository-specific inputs and secrets.

## Versioning

The shared workflows are released with Git tags such as `v1`, `v1.1.0`, and `v2`. Teams should pin their `uses:` reference to an immutable release tag (preferably a full commit SHA for the strongest supply-chain protection). The examples use the major-line tag `@v1` to make the intended compatibility contract clear.

Changes that preserve the existing inputs and behaviour can be released within `v1`. A breaking change gets a new major version, such as `v2`, while `v1` remains available for existing consumers. Teams can migrate from `v1` to `v2` on their own schedule, so a change does not disrupt all 15 teams at once.

## Secrets

Secrets are declared under `on.workflow_call.secrets` and passed explicitly by each caller. The Docker workflow receives registry credentials and the Kubernetes workflow receives a kubeconfig; neither is stored in this repository or passed as a normal input. GitHub masks secret values in logs, and the deployment writes the kubeconfig only to the runner's temporary directory with restrictive permissions. Callers should use environment- or organization-level secrets with the minimum required permissions.

The caller also grants only the permissions needed by the selected workflow. For example, image publishing needs `packages: write`, while testing and deployment only need repository read access (plus the cluster credentials supplied as a secret).
