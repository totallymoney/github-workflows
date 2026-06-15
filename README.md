# GitHub Workflows

Reusable GitHub Actions workflows and composite actions for Node.js projects deployed with AWS CDK.

## Contents

- [Usage](#usage)
- [Contributing](#contributing)
- [Versioning](#versioning)
- [Reference](#reference)

---

## Usage

Copy the relevant example from `examples/` into your project's `.github/workflows/` directory and replace `your-org` with your GitHub organisation name.

Set the following repository variables in the calling repo:

| Variable | Description |
|---|---|
| `NODE_VERSION` | Node.js version (e.g. `24`) |
| `AWS_REGION` | AWS region to deploy to |
| `OIDC_ROLE_ARN` | IAM role ARN to assume via OIDC |
| `CDK_STACK_NAME` | Base name of the CDK stack |

Pin callers to a major version tag so they receive backwards-compatible updates automatically but are protected from breaking changes:

```yaml
jobs:
  ci:
    uses: your-org/github-workflows/.github/workflows/node-cdk.pull-request.yml@v1
    with:
      node-version: ${{ vars.NODE_VERSION }}
      aws-region: ${{ vars.AWS_REGION }}
      oidc-role-arn: ${{ vars.OIDC_ROLE_ARN }}
      cdk-stack-name: ${{ vars.CDK_STACK_NAME }}
```

To pin to an exact release instead, use a full version tag (e.g. `@v1.2.3`).

---

## Contributing

### Making changes

Work on a feature branch and open a PR against `main`. Changes to workflows and actions are tested by the repos that consume them, so consider testing against a real caller before merging.

### Releasing

After merging to `main`, decide the appropriate version bump:

| Change type | Example | Version bump |
|---|---|---|
| Bug fix or internal improvement | Faster runner, fixed edge case | Patch — `v1.0.0 → v1.0.1` |
| New input with a default, new optional behaviour | New optional input | Minor — `v1.0.0 → v1.1.0` |
| Renamed/removed input, changed behaviour | Input renamed, step removed | Major — `v1.0.0 → v2.0.0` |

Then push the tag:

```bash
git tag v1.2.3
git push origin v1.2.3
```

CI will automatically create a GitHub release with generated notes and move the floating major tag (`v1`) to point at the new release. Callers pinned to `@v1` will receive the update on their next run.

### Breaking changes (major version bumps)

When making a breaking change, update all `@v1` action references inside the workflow files to `@v2` (or whatever the new major version is) as part of your PR, before tagging. This keeps the action versions in sync with the workflow version callers are pinned to.

```yaml
# Before
- uses: your-org/github-workflows/.github/actions/node/run-unit-tests@v1

# After
- uses: your-org/github-workflows/.github/actions/node/run-unit-tests@v2
```

Callers on `@v1` are unaffected — they continue using the workflow and actions at `v1`. Callers can opt in to `@v2` when ready.

---

## Versioning

This repo uses semantic versioning via git tags. Two tag types are maintained:

- **Floating major tag** (e.g. `v1`) — always points to the latest release within that major version. Callers on `@v1` automatically receive backwards-compatible updates.
- **Exact tags** (e.g. `v1.2.3`) — immutable. Use these to pin to a specific release permanently.

Breaking changes always increment the major version, so callers pinned to `@v1` are never affected by a `v2` release.

---

## Reference

### node-cdk

A Node.js project deployed with AWS CDK.

#### Workflows

| Workflow | Trigger | Description |
|---|---|---|
| `node-cdk.pull-request.yml` | `pull_request` | Checks formatting, runs tests, deploys a feature environment, runs E2E tests |
| `node-cdk.push-main.yml` | `push` to `main` | Runs tests, creates a release, deploys to stage, runs E2E tests |
| `node-cdk.manually-deploy-env.yml` | `workflow_dispatch` | Deploys a specific release version to a chosen environment |
| `node-cdk.scheduled-e2e-tests.yml` | `schedule` | Runs E2E tests against stage and prod |
| `node-cdk.teardown-feature.yml` | `pull_request` closed | Destroys the feature environment for a closed PR |

##### node-cdk.pull-request.yml

| Input | Required | Default | Description |
|---|---|---|---|
| `node-version` | yes | | Node.js version (e.g. `24`) |
| `aws-region` | yes | | AWS region to deploy to |
| `oidc-role-arn` | yes | | IAM role ARN to assume via OIDC |
| `cdk-stack-name` | yes | | Base name of the CDK stack |
| `cdk-dir` | no | `app-stack` | Directory containing CDK app |
| `api-collection-dir` | no | `API Collection` | Directory containing Bruno API collection |

##### node-cdk.push-main.yml

| Input | Required | Default | Description |
|---|---|---|---|
| `node-version` | yes | | Node.js version (e.g. `24`) |
| `aws-region` | yes | | AWS region to deploy to |
| `oidc-role-arn` | yes | | IAM role ARN to assume via OIDC |
| `cdk-stack-name` | yes | | Base name of the CDK stack |
| `version-prefix` | yes | | Version prefix for releases (e.g. `0.2` produces `0.2.{run_number}`) |
| `cdk-dir` | no | `app-stack` | Directory containing CDK app |
| `api-collection-dir` | no | `API Collection` | Directory containing Bruno API collection |

##### node-cdk.manually-deploy-env.yml

| Input | Required | Default | Description |
|---|---|---|---|
| `version` | yes | | Release version to deploy (e.g. `0.1.42`) |
| `environment` | yes | | Environment to deploy to (e.g. `stage`, `prod`) |
| `node-version` | yes | | Node.js version (e.g. `24`) |
| `aws-region` | yes | | AWS region to deploy to |
| `oidc-role-arn` | yes | | IAM role ARN to assume via OIDC |
| `cdk-stack-name` | yes | | Base name of the CDK stack |
| `cdk-dir` | no | `app-stack` | Directory containing CDK app |
| `api-collection-dir` | no | `API Collection` | Directory containing Bruno API collection |
| `prod-tag` | no | `prod-safe` | Bruno tag used to filter tests when running against prod |

##### node-cdk.scheduled-e2e-tests.yml

| Input | Required | Default | Description |
|---|---|---|---|
| `aws-region` | yes | | AWS region to deploy to |
| `oidc-role-arn` | yes | | IAM role ARN to assume via OIDC |
| `api-collection-dir` | no | `API Collection` | Directory containing Bruno API collection |
| `prod-tag` | no | `prod-safe` | Bruno tag used to filter tests when running against prod |

##### node-cdk.teardown-feature.yml

| Input | Required | Default | Description |
|---|---|---|---|
| `node-version` | yes | | Node.js version (e.g. `24`) |
| `aws-region` | yes | | AWS region to deploy to |
| `oidc-role-arn` | yes | | IAM role ARN to assume via OIDC |
| `cdk-stack-name` | yes | | Base name of the CDK stack |
| `cdk-dir` | no | `app-stack` | Directory containing CDK app |

#### Composite actions

These are used internally by the workflows above and do not need to be referenced directly in most cases.

| Action | Description |
|---|---|
| `.github/actions/node/check-formatting` | Runs `npm run format:check` |
| `.github/actions/node/run-unit-tests` | Runs TypeScript validation and `npm run test:unit` |
| `.github/actions/node/run-integration-tests` | Runs `npm run test:integration` |
| `.github/actions/node/create-release` | Creates a GitHub release with auto-generated notes |
| `.github/actions/cdk/deploy-env` | Deploys a CDK stack to the given environment |
| `.github/actions/cdk/destroy-env` | Destroys a CDK stack for the given environment |
| `.github/actions/bruno/run-e2e-tests` | Runs a Bruno API collection against the given environment |
