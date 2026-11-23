# business-data-actions

Shared GitHub Actions reusable workflows and composite actions for the Business Data teams.

## Catalog

### Reusable workflows

| Workflow | Description |
|----------|-------------|

### Composite actions

| Action | Description |
|--------|-------------|

## Usage

Always pin by full commit SHA and keep the version in a trailing comment. Dependabot updates both
the SHA and the comment.

Reusable workflow (replaces a job):

```yaml
jobs:
  example:
    uses: canonical/business-data-actions/.github/workflows/<name>.yaml@<sha> # vX.Y.Z
```

Composite action (used as a step):

```yaml
steps:
  - uses: canonical/business-data-actions/actions/<name>@<sha> # vX.Y.Z
```

To let Dependabot keep these pins current, add to your repository's `.github/dependabot.yaml`:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

## Versioning

The whole repository shares one [semantic version](https://semver.org/). Tags are informational:

- `vX.Y.Z` marks a release and never moves.
- `vX` moves to the latest `vX.Y.Z` release.

Pin by SHA rather than by tag; tags can be moved.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
