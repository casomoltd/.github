# .github

Shared GitHub Actions workflows and org-level configuration for casomoltd.

## Setup

### Repo access for reusable workflows

This repo is **private**, so other org repos cannot reference its reusable
workflows unless access is explicitly granted.

**Required setting:** This repo's Actions access level must be set to
`organization`. Without this, caller workflows in other repos will fail at
the file resolution stage (zero jobs created, "workflow file issue" error).

To check or set via API:

```bash
# Check current access level
gh api repos/casomoltd/.github/actions/permissions/access

# Grant access to all org repos
gh api repos/casomoltd/.github/actions/permissions/access \
  -X PUT -f access_level=organization
```

Or via UI: Repo **Settings > Actions > General > Access** — select
"Accessible from repositories in the 'casomoltd' organization".

## Workflows

### Add to Casomo Workboard

**File:** `.github/workflows/add-to-project.yml`

Reusable workflow that automatically adds issues to the
[Casomo Workboard](https://github.com/orgs/casomoltd/projects/3) project.

**Triggers:** Called by per-repo caller workflows on `issues: [opened, transferred, reopened]`.

**Auth:** Uses the **Casomo Bot** GitHub App (App ID stored in `WORKBOARD_APP_ID` org secret)
to mint short-lived tokens at runtime. No PATs required.

#### Caller workflow

Each repo in the org has this caller workflow at `.github/workflows/add-to-project.yml`:

```yaml
name: Add to Casomo Workboard

on:
  issues:
    types: [opened, transferred, reopened]

jobs:
  add:
    uses: casomoltd/.github/.github/workflows/add-to-project.yml@main
    secrets: inherit
```

#### Adding to a new repo

1. Create `.github/workflows/add-to-project.yml` in the new repo with the
   caller workflow above. The `secrets: inherit` directive passes the org-level
   secrets to the reusable workflow automatically.
2. Verify this repo's Actions access level is set to `organization`
   (see [Setup](#setup) above).
3. **Private repos only:** Set the repo-level secrets (see below). Org secrets
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
