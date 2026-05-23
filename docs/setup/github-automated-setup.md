# GitHub Actions Automated Setup

> If you prefer to configure everything manually, see the [GitHub Actions Setup Guide](github-setup.md).

The `setup-github.ps1` script automates the GitHub Actions setup for ALM4Dataverse.
It mirrors the same automation available for Azure DevOps (`setup-azdo.ps1`).

---

## Limitations

- The account you use for setup must be in the same Entra ID tenant as the Dataverse environments.
- The process works for the standard `Azure Cloud` (`Commercial`) cloud and not `GCC` etc.
- The GitHub CLI (`gh`) must be installed before running the script.
- App Registrations created automatically will be named `{repo-name} - {env-name} - deployment`.
  You can safely rename them afterwards.
- You will be prompted to choose between two authentication types per environment:
  - **Workload Identity Federation (recommended)**: No secrets to manage or rotate.
  - **Service Principal with Secret (traditional)**: Uses a client secret that expires.

---

## Pre-requisites

Before you start, you need:

### 1) A GitHub repository

Create or use an existing GitHub repository for your Dataverse application source code.

### 2) GitHub CLI installed

Download and install the GitHub CLI from <https://cli.github.com/>.

Verify installation:

```powershell
gh --version
```

### 3) Entra ID access

You need permission to create App Registrations in the Entra ID tenant that hosts your
Dataverse environments. The setup script will create them automatically if you have that
permission, or you can provide existing App Registration details.

### 4) Application user in each Dataverse environment

For each environment, the setup script will automatically create an application user for
the selected App Registration and grant it the **System Administrator** role.

---

## Running Setup

1. Open **Windows PowerShell** from the Start menu (or PowerShell 7+).

2. Paste this in and press Enter:

   ```powershell
   iwr https://github.com/ALM4Dataverse/ALM4Dataverse/releases/latest/download/setup-github.ps1 | iex
   ```

   > If you would like to review the script first (good practice), download it from
   > <https://github.com/ALM4Dataverse/ALM4Dataverse/releases/latest/download/setup-github.ps1>

3. Follow the on-screen instructions.

---

## What Setup Does

1. **Authenticates with GitHub** — signs in via the GitHub CLI if not already logged in.
2. **Authenticates with Azure** — opens a browser to sign in to your Entra ID tenant.
3. **Selects your GitHub repository** — lists repos you have write access to.
4. **Copies workflow templates** — copies the `copy-to-your-repo/` files into your repository
   and pushes them. `DEPLOY-main.yml` is renamed to match your default branch.
5. **Configures solutions** — connects to your Dataverse DEV environment, lists unmanaged
   solutions, lets you select them in dependency order, and updates `alm-config.psd1`.
6. **Sets up the Dev environment** — creates a GitHub environment named `Dev-{branch}`,
   creates or reuses an Entra ID App Registration, optionally configures a WIF federated
   credential, sets `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `DATAVERSE_URL`, and
   `DATAVERSESERVICEACCOUNTUPN` in the environment, and creates the Dataverse application user.
7. **Sets up each deployment environment** — repeats step 6 for every environment you add
   (TEST-main, PROD, UAT, etc.).

---

## Post-Setup Steps

After the script completes:

1. Go to **Actions** in your repository — the `BUILD`, `EXPORT`, `IMPORT`, and `DEPLOY-main`
   workflows are ready.
2. If you want environment protection rules (required reviewers, wait timer) on TEST or PROD,
   go to **Settings** > **Environments** and configure them.
   See [GitHub licence limitations](github-setup.md#github-licence-limitations) for details.
3. Review the `DEPLOY-{branch}.yml` workflow in your repository.
   The default uses **Strategy A** (GitHub Free — manual re-trigger + gate tags).
   If you have GitHub Pro/Team/Enterprise and want auto-chained approvals, switch to Strategy B
   as described in the [GitHub Actions Setup Guide](github-setup.md#deployment-gates-for-github-free).

---

## Adding Environments Later

Re-run the script at any time to add more deployment environments. Previously entered
credentials are offered as re-use options to avoid re-entering them.

---

## See Also

- [GitHub Actions Setup Guide](github-setup.md) — full manual setup instructions
- [GitHub Secrets & Variables Reference](../config/github-secrets.md)
