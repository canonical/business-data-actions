# retain-reviewers

GitHub clears a team's review request as soon as one member reviews, which drops the pull request
from the team's queue before it has enough approvals. This reusable workflow re-requests the
configured teams and users until the pull request has `required-approvals` approvals.

## Usage

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
      team-reviewers: my-team, other-team
      reviewers: some-user
    secrets:
      REVIEW_REASSIGN_TOKEN: ${{ secrets.REVIEW_REASSIGN_TOKEN }}
```

Do not add `review_requested` to the triggers; the workflow's own requests would re-trigger it.

This repository's [review-requests.yaml](../../.github/workflows/review-requests.yaml) is a
complete caller.

## Inputs

| Name | Default | Description |
|------|---------|-------------|
| `required-approvals` | `2` | Number of approvals after which review requests are no longer renewed |
| `team-reviewers` | `""` | Team slugs without the `@org/` prefix |
| `reviewers` | `""` | GitHub usernames |

`team-reviewers` and `reviewers` accept a comma- or whitespace-separated list, or a JSON array.

## Secrets

| Name | Description |
|------|-------------|
| `REVIEW_REASSIGN_TOKEN` | Fine-grained personal access token; see [Token](#token) |

## Behavior

The workflow does nothing when the pull request is a draft or not open, or when the token is not
available. Otherwise, it takes each reviewer's latest decisive review:

- `COMMENTED` does not override an earlier approval or change request.
- `DISMISSED` clears the reviewer's earlier state.
- The pull request author's reviews are ignored.

If fewer than `required-approvals` reviewers approve and nobody requests changes, it requests the
configured teams and users that are not already pending. It never requests the author or users
who already approved. Once the threshold is reached, it leaves the requests cleared.

## Token

`GITHUB_TOKEN` cannot request reviews from organization teams, so create a
[fine-grained personal access token](https://github.com/settings/personal-access-tokens/new):

- **Resource owner:** the organization (an org owner may need to approve the token).
- **Repository access:** only the repositories that use the workflow.
- **Permissions:** repository `Pull requests: Read and write`; organization `Members: Read-only`.
- **Expiration:** set one and record it; rotate the token and update the secret before it expires.

Store it as the `REVIEW_REASSIGN_TOKEN` repository or organization secret. Requests are made as the
token's owner. Fork and Dependabot pull requests do not receive the secret, so the workflow skips
them.

## Limitations

- Every approval counts toward `required-approvals`, including approvals from people outside the
  configured teams or without write access. The workflow can stop re-requesting before branch
  protection is satisfied.
- For teams with code review auto-assignment enabled, GitHub replaces the team request with
  individual members.
- A manually removed review request comes back only on the next triggering event.
