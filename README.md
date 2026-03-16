# .github

Shared GitHub Actions workflows and org-level configuration for casomoltd.

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

Create `.github/workflows/add-to-project.yml` in the new repo with the caller
workflow above. The `secrets: inherit` directive passes the org-level secrets
to the reusable workflow automatically.

## Org secrets

| Secret | Purpose |
|---|---|
| `WORKBOARD_APP_ID` | Casomo Bot GitHub App ID |
| `WORKBOARD_APP_PRIVATE_KEY` | Casomo Bot private key for minting tokens |
