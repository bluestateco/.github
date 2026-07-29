# bluestateco/.github

Organization-wide shared configuration for the Blue State GitHub org.

## Renovate

`renovate.json` provides org-level default configuration for the
[Renovate](https://docs.renovatebot.com/) dependency updater.

### What it manages

| Manager          | Scope                                                   | Auto-merge       |
| ---------------- | ------------------------------------------------------- | ---------------- |
| `terraform`      | `?ref=vX.Y.Z` pins in `terraform/main.tf` (spoke repos)| Patch only       |
| `github-actions` | SHA-pinned actions across all repos                     | Patch + digest   |

Minor and major bumps require human review.

### Prerequisites

The **Renovate GitHub App** must be installed at the org level:

1. An org admin goes to <https://github.com/apps/renovate>
2. Click **Install** and select the `bluestateco` organization
3. Grant access to **all repositories** (or at minimum: all `gather-*` spoke repos,
   `better-data-initiative`, `gather-starter`, `dbt-gather-functions`)
4. Renovate will create an onboarding PR in each repo; the org-level config here
   serves as the default for repos that don't have their own `renovate.json`

### Dependabot overlap

Some repos (BDI, gather-starter) already use Dependabot for GitHub Actions.
Once Renovate is live, either:

- Remove Dependabot from those repos (preferred — single tool), or
- Add a per-repo `renovate.json` to disable the `github-actions` manager:
  ```json
  {
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "enabledManagers": ["terraform"]
  }
  ```

### Relationship to update-client-repos.yml

`gather-starter/.github/workflows/update-client-repos.yml` is a push-based
Terraform module updater (hub pushes version bumps to spokes on new tag).
It is currently broken because `auto-tag.yml` uses `GITHUB_TOKEN`, which
does not trigger downstream `on: push: tags` workflows
(see [gather-starter#330](https://github.com/bluestateco/gather-starter/issues/330)).

Renovate is pull-based (periodically scans spoke repos for outdated refs).
Once Renovate is confirmed working:

- **Retire `update-client-repos.yml`** — Renovate handles the same job
- **Keep or fix `auto-tag.yml`** — still needed for creating semver tags on merge
- Remove the `TERRAFORM_UPDATER_APP_ID` / `TERRAFORM_UPDATER_PRIVATE_KEY`
  secrets from gather-starter (no longer needed)
