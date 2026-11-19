# business-data-actions

Shared GitHub Actions reusable workflows and composite actions for the Business Data teams.

## Catalog

### Reusable workflows

| Workflow | Description |
|----------|-------------|
| [`retain-reviewers`](.github/workflows/retain-reviewers.yaml) | Re-requests teams/users on a PR until it has the required approvals |

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

## retain-reviewers

GitHub clears a team's review request once one member reviews. This workflow re-requests the
configured teams and users until the PR has `required-approvals` approvals, unless someone
currently requests changes. See [review-requests.yaml](.github/workflows/review-requests.yaml)
for a complete caller.

```yaml
on:
  pull_request:
    types: [opened, reopened, ready_for_review]
  pull_request_review:
    types: [submitted, dismissed]

permissions: {}

jobs:
  retain-reviewers:
    uses: canonical/business-data-actions/.github/workflows/retain-reviewers.yaml@<sha> # vX.Y.Z
    with:
      required-approvals: 2
      team-reviewers: my-team, other-team # slugs without @org/
      reviewers: some-user                # optional
    secrets:
      REVIEW_REASSIGN_TOKEN: ${{ secrets.REVIEW_REASSIGN_TOKEN }}
```

Do not add `review_requested` to the triggers; the workflow's own requests would re-trigger it.

### Token

`GITHUB_TOKEN` cannot request reviews from organization teams, so create a
[fine-grained personal access token](https://github.com/settings/personal-access-tokens/new):

- **Resource owner:** the organization (an org owner may need to approve the token).
- **Repository access:** only the repositories that use the workflow.
- **Permissions:** repository `Pull requests: Read and write`; organization `Members: Read-only`.
- **Expiration:** set one and record it; rotate the token and update the secret before it expires.

Store it as the `REVIEW_REASSIGN_TOKEN` repository or organization secret. Requests are made as the
token's owner. Fork and Dependabot PRs do not receive the secret, so the workflow skips them.

## Versioning

The whole repository shares one [semantic version](https://semver.org/). Tags are informational:

- `vX.Y.Z` marks a release and never moves.
- `vX` moves to the latest `vX.Y.Z` release.

Pin by SHA rather than by tag; tags can be moved.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
