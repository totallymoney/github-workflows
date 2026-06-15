# GitHub Workflows

Reusable GitHub Actions workflows.

## Structure

```
.github/workflows/
  workflows/        # Orchestrators — entry points for callers
    node-cdk/
  jobs/             # Building blocks — composed by orchestrators
    node/
    cdk/
    bruno/
examples/           # Example caller workflows
  node-cdk/
```

## Versioning

This repo uses semantic versioning via git tags. Callers pin to a major version tag (e.g. `@v1`) which automatically receives backwards-compatible updates, or to an exact tag (e.g. `@v1.2.3`) to stay fully pinned.

Breaking changes always bump the major version, so callers on `@v1` are never affected by a `v2` release.

### Cutting a release

```bash
git tag v1.2.3
git push origin v1.2.3
```

CI will automatically move the floating `v1` tag to point at the new release.

## Usage

Copy the relevant example from `examples/` into your project's `.github/workflows/` directory and replace `your-org` with the actual org name.

### node-cdk

A Node.js project deployed with AWS CDK. Requires the following repository variables:

| Variable | Description |
|---|---|
| `NODE_VERSION` | Node.js version (e.g. `24`) |
| `AWS_REGION` | AWS region to deploy to |
| `OIDC_ROLE_ARN` | IAM role ARN to assume via OIDC |
| `CDK_STACK_NAME` | Base name of the CDK stack |

#### Workflows

| Workflow | Trigger | Description |
|---|---|---|
| `workflows/node-cdk/pull-request.yml` | `pull_request` | Checks formatting, runs tests, deploys a feature environment, runs E2E tests |
| `workflows/node-cdk/push-main.yml` | `push` to `main` | Runs tests, creates a GitHub release, deploys to stage, runs E2E tests |
| `workflows/node-cdk/manually-deploy-env.yml` | `workflow_dispatch` | Deploys a specific release version to a chosen environment |
| `workflows/node-cdk/scheduled-e2e-tests.yml` | `schedule` | Runs E2E tests against stage and prod |
| `workflows/node-cdk/teardown-feature.yml` | `pull_request` closed | Destroys the feature environment for a closed PR |

#### Jobs

| Job | Description |
|---|---|
| `jobs/node/job.check-formatting.yml` | Runs `npm run format:check` |
| `jobs/node/job.run-unit-tests.yml` | Runs TypeScript validation and `npm run test:unit` |
| `jobs/node/job.run-integration-tests.yml` | Runs `npm run test:integration` |
| `jobs/node/job.create-release.yml` | Creates a GitHub release with auto-generated notes |
| `jobs/cdk/job.deploy-env.yml` | Deploys a CDK stack to the given environment |
| `jobs/cdk/job.destroy-env.yml` | Destroys a CDK stack for the given environment |
| `jobs/bruno/job.run-e2e-tests.yml` | Runs a Bruno API collection against the given environment |
