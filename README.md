# GitHub Workflows

Reusable GitHub Actions workflows and composite actions.

## Contents

- [Usage](#usage)
- [Contributing](#contributing)
- [Versioning](#versioning)

---

## Usage

Browse the available workflows in `.github/workflows/` and composite actions in `.github/actions/`. Example caller workflows for each scenario are in `examples/`.

To use a workflow, copy the relevant example into your project's `.github/workflows/` directory and pin to a major version tag:

```yaml
jobs:
  ci:
    uses: totallymoney/github-workflows/.github/workflows/node-cdk.pull-request.yml@v1
    with:
      ...
```

Callers pinned to `@v1` automatically receive backwards-compatible updates. To pin to an exact release, use a full version tag (e.g. `@v1.2.3`).

---

## Contributing

### Making changes

Work on a feature branch and open a PR against `main`. Changes to workflows and actions are tested by the repos that consume them, so consider testing against a real caller before merging.

### Releasing

After merging to `main`, decide the appropriate version bump:

| Change type | Version bump |
|---|---|
| Bug fix or internal improvement | Patch — `v1.0.0 → v1.0.1` |
| New input with a default, new optional behaviour | Minor — `v1.0.0 → v1.1.0` |
| Renamed/removed input, changed behaviour | Major — `v1.0.0 → v2.0.0` |

Then push the tag:

```bash
git tag v1.2.3
git push origin v1.2.3
```

CI will automatically create a GitHub release with generated notes and move the floating major tag (`v1`) to point at the new release.

### Breaking changes (major version bumps)

When making a breaking change, update all action references inside the workflow files from the old major version to the new one as part of your PR:

```yaml
# Before
- uses: totallymoney/github-workflows/.github/actions/node/run-unit-tests@v1

# After
- uses: totallymoney/github-workflows/.github/actions/node/run-unit-tests@v2
```

Callers on `@v1` are unaffected and can opt in to `@v2` when ready.

---

## Versioning

This repo uses semantic versioning via git tags:

- **Floating major tag** (e.g. `v1`) — always points to the latest release within that major version. Callers receive backwards-compatible updates automatically.
- **Exact tag** (e.g. `v1.2.3`) — immutable. Use to pin to a specific release permanently.

Breaking changes always increment the major version, so callers on `@v1` are never affected by a `v2` release.
