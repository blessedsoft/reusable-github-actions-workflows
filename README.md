# Reusable GitHub Actions Workflows

Centralized GitHub Actions workflows for standardizing application testing, Docker image publishing, and Kubernetes deployments across development teams.

## Included workflows

| Workflow | Purpose | Required inputs |
| --- | --- | --- |
| `build-and-test.yml` | Installs Node.js dependencies and runs tests | `node-version`, `test-command` |
| `docker-build-push.yml` | Builds and publishes a Docker image | `image-name`, `registry` |
| `deploy-to-kubernetes.yml` | Installs Helm and deploys a chart | `chart-path`, `namespace`, `values-file` |

The workflows are reusable workflows triggered with `workflow_call`. The calling repository owns the event trigger; this repository owns the shared job implementation.

## Repository layout

```text
.
├── .github/workflows/       # Reusable workflows; GitHub requires this location
├── caller-example/          # Example consumer workflow files
├── DESIGN.md                # Versioning, breaking changes, and secrets
├── quick-start.md            # Step-by-step setup guide
└── req.txt                  # Assignment requirements
```

## Prerequisites

- A GitHub repository containing this project.
- GitHub Actions enabled in the library and consuming repositories.
- A Node.js consumer with `package.json` and `package-lock.json` for the build workflow.
- A consumer repository with a `Dockerfile` for the Docker workflow.
- A Helm chart and values file for the deployment workflow.
- Appropriate repository or organization secrets for registry and Kubernetes access.

## Usage

Copy the relevant file from `caller-example/` into the consuming repository's `.github/workflows/` directory. Replace `YOUR-ORG`, `YOUR-REPOSITORY`, and other example values, then commit the file.

The examples reference this repository with the `v1` major-version tag:

```yaml
uses: YOUR-ORG/reusable-github-actions-workflows/.github/workflows/build-and-test.yml@v1
```

Publish the `v1` tag before using the examples. For stronger supply-chain protection, consumers can pin the reference to a full commit SHA instead.

## Required secrets

The Docker workflow expects `registry-username` and `registry-password`. The Kubernetes workflow expects `kubeconfig`. Secrets are passed explicitly by the caller and are never stored in this repository.

See [DESIGN.md](DESIGN.md) for the security and versioning approach, and [quick-start.md](quick-start.md) for the complete setup procedure.

## Operational notes

- The Docker image tag is the caller's `${{ github.sha }}`.
- The Helm release name is currently `app`.
- The build workflow runs `npm ci`, so a committed lockfile is required.
- The caller examples are templates and require repository-specific configuration before use.
