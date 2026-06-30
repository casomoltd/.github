# Workflows

## Add to Casomo Workboard

**File:** `.github/workflows/add-to-project.yml`

Reusable workflow that automatically adds items to the
[Casomo Workboard](https://github.com/orgs/casomoltd/projects/3) project and
sets their status to **Backlog**. `actions/add-to-project` auto-detects whether
the triggering item is an issue or a pull request, so the same reusable workflow
serves both.

**Triggers:** Called by per-repo caller workflows on:
- `issues: [opened, transferred, reopened]` — every new issue.
- `pull_request_target: [opened, reopened]` — **Dependabot PRs only** (the
  caller gates on `github.actor == 'dependabot[bot]'`). This is how Dependabot's
  security-update and grouped `github-actions` bump PRs land on the board. Human
  PRs are intentionally left off — they already track via their linked issue.

**Why `pull_request_target` (not `pull_request`):** a workflow triggered by a
Dependabot `pull_request` event runs with a read-only token and **no access to
Actions secrets**, so `create-github-app-token` could not mint the Workboard App
token. `pull_request_target` runs in the base-branch context with full secret
access. It's safe here because the workflow never checks out the PR's code.

**Auth:** Uses the **Casomo Bot** GitHub App (App ID stored in `WORKBOARD_APP_ID` org secret)
to mint short-lived tokens at runtime. No PATs required.

### Caller workflow

Each repo in the org has this caller workflow at `.github/workflows/add-to-project.yml`:

```yaml
name: Add to Casomo Workboard

on:
  issues:
    types: [opened, transferred, reopened]
  pull_request_target:
    types: [opened, reopened]

jobs:
  add:
    # Issues: always. PRs: only Dependabot's — human PRs track via their
    # linked issue. pull_request_target so the reusable workflow can read
    # the App secrets that Dependabot pull_request runs are denied.
    if: github.event_name == 'issues' || github.actor == 'dependabot[bot]'
    uses: casomoltd/.github/.github/workflows/add-to-project.yml@main
    secrets: inherit
```

### Adding to a new repo

1. Create `.github/workflows/add-to-project.yml` in the new repo with the
   caller workflow above. The `secrets: inherit` directive passes the org-level
   secrets to the reusable workflow automatically.
2. **Private repos only:** Set the repo-level secrets (see below). Org secrets
   are only available to public repos on the GitHub Free plan.

   ```bash
   echo "<APP_ID>" | gh secret set WORKBOARD_APP_ID --repo casomoltd/<repo>
   gh secret set WORKBOARD_APP_PRIVATE_KEY --repo casomoltd/<repo> < path/to/private-key.pem
   ```

## Secrets

| Secret | Purpose |
|---|---|
| `WORKBOARD_APP_ID` | Casomo Bot GitHub App ID |
| `WORKBOARD_APP_PRIVATE_KEY` | Casomo Bot private key for minting tokens |

Org-level secrets are scoped to **public repositories only** (GitHub Free plan
limitation). For **private repos**, these must be set as repo-level secrets.
The App ID and private key can be found/generated at the
[Casomo Bot app settings](https://github.com/organizations/casomoltd/settings/apps).
