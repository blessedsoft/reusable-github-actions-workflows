# Quick start

This guide sets up the workflow library and connects a consuming repository to all three reusable workflows.

## 1. Publish the workflow library

1. Create a GitHub repository for this project, or push the project to the organization's existing repository.
2. Confirm that these files exist in the default branch:

   ```text
   .github/workflows/build-and-test.yml
   .github/workflows/docker-build-push.yml
   .github/workflows/deploy-to-kubernetes.yml
   ```

3. Enable Actions in the repository settings.
4. Create and push a compatible release tag. The caller examples use `v1`:

   ```powershell
   git add .
   git commit -m "Add reusable GitHub Actions workflow library"
   git push origin main
   git tag v1
   git push origin v1
   ```

   If the repository uses another default branch, replace `main` with that branch name.

## 2. Prepare the consuming repository

The consuming repository should contain:

- `package.json` and `package-lock.json` for build and test;
- a `Dockerfile` for image publishing; and
- a Helm chart plus environment-specific values file for deployment.

Create the caller workflow directory if it does not already exist:

```text
.github/workflows/
```

Copy one or more files from this repository's `caller-example/` directory into that directory.

## 3. Configure the library reference

In each copied file, replace:

```text
YOUR-ORG/reusable-github-actions-workflows
```

with the actual organization and repository name. Keep the `@v1` suffix to consume the version-1 workflow contract.

For a private library repository, configure the consuming repository's Actions access under the library repository's settings so the caller is allowed to use its reusable workflows.

## 4. Configure Docker publishing

The Docker caller requires package publishing permission and registry credentials. For GitHub Container Registry, the example uses the caller's `GITHUB_TOKEN`:

```yaml
permissions:
  contents: read
  packages: write
```

Update `image-name` to the target image path, for example `acme/orders-api`. The resulting image tag is based on the caller commit SHA.

If using a registry other than GHCR, create appropriate repository or organization secrets and pass them as `registry-username` and `registry-password`.

## 5. Configure Kubernetes deployment

1. Add the target cluster's kubeconfig as an encrypted repository or environment secret named `KUBECONFIG`.
2. Update `chart-path`, `namespace`, and `values-file` in `caller-example/deploy-to-kubernetes.yml`.
3. Review the deployment environment protection rules before enabling automatic deployment.

The shared workflow passes the secret to Helm through a temporary kubeconfig file on the runner. The file is created with restrictive permissions and is not committed to the repository.

## 6. Run the workflows

- Build and test runs on `push`, `pull_request`, and manual dispatch.
- Docker build and push runs on pushes to `main` and manual dispatch.
- Kubernetes deployment is manual by default through `workflow_dispatch`.

To run a manually-triggered workflow, open the consuming repository on GitHub, select **Actions**, choose the workflow, and select **Run workflow**.

## 7. Verify results

1. Open the **Actions** tab in the consuming repository.
2. Confirm that the caller job completed successfully.
3. For Docker publishing, verify the image exists in the configured registry with the expected commit-SHA tag.
4. For Kubernetes deployment, verify the Helm release and workload in the target namespace.

## 8. Pin production consumers

After validating `v1`, production teams should replace the major tag with the full commit SHA of the approved library release when their security policy requires immutable dependencies:

```yaml
uses: YOUR-ORG/reusable-github-actions-workflows/.github/workflows/build-and-test.yml@COMMIT_SHA
```

Maintain the `v1` tag for compatible fixes. Introduce a new `v2` tag for breaking input, permission, or behavior changes and migrate teams deliberately.
