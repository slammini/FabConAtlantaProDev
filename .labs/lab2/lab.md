# Lab - Power BI and CI/CD

⏱️ Duration: 110 minutes

In this lab you will set up a complete CI/CD pipeline for a Power BI project. You will work with a pre-configured repository template that includes Power BI project files (PBIP), a Python deployment script powered by `fabric-cicd`, and GitHub Actions workflows for automated deployment and quality checks.

## Goals

- Set up a working repository from a template and configure deployment credentials
- Understand `fabric-cicd` as the recommended deployment tool for PBIP projects
- Run a local deployment and verify results in a Fabric workspace
- Automate deployments via GitHub Actions on every push to `main`
- Use the Best Practice Analyzer (BPA) to enforce quality standards on Pull Requests
- Experience a complete Pull Request workflow with automated BPA quality gates

## 0. Setup

**Goal**: Get a working copy of the lab repository on your machine with all prerequisites in place.

### Create a new GitHub repository from the template

1. Navigate to the template repository: **<https://github.com/RuiRomano/workshops-cicd-demo>**

2. Click the green **Use this template** button, then select **Create a new repository**.

    ![use-template](resources/img/gh-template-use.png)

3. Choose an **Owner**, give your repository a **name** (for example, `workshop-cicd`), and select the desired **visibility** (Public or Private). Then click **Create repository**.

    ![new-repo](resources/img/gh-template-new-repo.png)

> [!TIP]
> Creating a repository from a template gives you a clean copy with the full directory structure and all files, but without the template's commit history. Learn more in the [GitHub docs on template repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template).

### Clone the repository to your machine

1. On your new repository's page, click the green **Code** button and copy the HTTPS URL.

    ![clone-url](resources/img/gh-new-repo-code.png)

2. Open a terminal and clone the repository:

    ```bash
    git clone https://github.com/<YOUR-USER>/<YOUR-REPO>.git
    ```

3. Change into the repository directory and open it in **Visual Studio Code**:

    ```bash
    cd <YOUR-REPO>
    code .
    ```

### Set up the Python extension in VS Code

The deployment script requires **Python 3.12**. The easiest way to manage Python versions and run scripts in VS Code is through the official **Python extension**.

1. Open the **Extensions** panel (`Ctrl+Shift+X`) and search for **Python**. Install the extension published by **Microsoft** if it is not already installed.

    ![python-extension](resources/img/vscode-ext-python.png)

2. After installation, the extension automatically detects all Python versions available on your system. You can see the currently selected interpreter in the **VS Code status bar** (bottom-right) whenever a `.py` file is open (for instance, `scripts/deploy.py`).

    ![python-interpreter](resources/img/vscode-py-current-interpreter.png)

3. Click the Python version in the status bar (or press `Ctrl+Shift+P` and type **Python: Select Interpreter**) to open the interpreter picker. Select **Python 3.12.x** from the list.

    ![select-interpreter](resources/img/vscode-py-select-interpreter.png)

> [!TIP]
> If you have multiple Python versions installed (e.g., 3.10, 3.11, 3.12), the Python extension makes it easy to switch between them. The selected interpreter applies to the integrated terminal and any scripts you run from VS Code — no need to manage `PATH` or virtual environments manually.

### Prepare the AZURE_CREDENTIALS secret

Automated deployments via GitHub Actions require a **service principal** — an identity in Microsoft Entra ID (formerly Azure AD) that represents an application rather than a human user. Your workshop environment should already have a service principal provisioned for you.

> [!IMPORTANT]
> If a service principal has **not** been set up yet, follow the step-by-step instructions in **Appendix A** at the end of this lab before continuing.

Using your service principal's details, prepare the following JSON — you will need it in Section 3 when configuring GitHub Actions:

```json
{
  "clientId": "<Application (client) ID>",
  "clientSecret": "<Client secret value>",
  "tenantId": "<Directory (tenant) ID>"
}
```

> [!TIP]
> Keep this JSON in a temporary text file or clipboard manager — you will paste it into GitHub later.

### A note on PowerShell

The repository includes BPA (Best Practice Analyzer) scripts written in **PowerShell**. These run automatically in GitHub Actions, but if you want to run them locally you need PowerShell installed on your machine.

- **Windows**: PowerShell is built in — no action needed.
- **macOS / Linux**: Install PowerShell by following the [official installation guide](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell).

## 1. Introduction to fabric-cicd

How does source code in a GitHub repository actually get deployed to a Fabric workspace? The answer is [`fabric-cicd`](https://microsoft.github.io/fabric-cicd/latest/) — a Python library developed by Microsoft that is the recommended tool for deploying Fabric items from source control.

`fabric-cicd` takes the PBIP definition files in your repository and publishes them to a target workspace via the Fabric REST APIs. Rather than calling APIs directly, you write a short Python script that configures `fabric-cicd` and lets the library handle the heavy lifting — API calls, retries, status polling, and error handling.

A key design principle is that **the same script works identically on your local machine and inside a GitHub Actions workflow**. This means you can test deployments locally first and reproduce CI/CD failures on your workstation if needed.

### Why fabric-cicd?

| Advantage | Description |
| --- | --- |
| **Fabric-native REST APIs** | Built on official Fabric APIs, ensuring long-term compatibility |
| **Python-native** | Integrates naturally with modern DevOps workflows |
| **Parameterization** | Built-in support for environment-specific values via `parameter.yml` |
| **Flexible deployment control** | Deploy specific item types (e.g., only semantic models) |
| **Reliable authentication** | Uses the Azure Identity SDK — browser login locally, service principal in CI/CD |

### How it fits into the deployment pipeline

```text
PBIP Source Files (in your repo)
       │
       ▼
   deploy.py              ← Python script included in the template
       │
       ├── Run locally     → fabric-cicd ──► Fabric Workspace
       │
       └── Run via GitHub Actions (automated)
               │
               └── fabric-cicd ──► Fabric Workspace
```

In the following sections you will run a local deployment first, then see how GitHub Actions automates the process on every push.

> [!NOTE]
> This workshop uses a single target workspace per environment (DEV and PRD). In production scenarios, `fabric-cicd` supports advanced multi-environment parameterization via `parameter.yml`. See the [fabric-cicd documentation](https://microsoft.github.io/fabric-cicd/latest/) for details.

## 2. Local deployment

**Goal**: Understand the deployment script, configure your target workspaces, and run your first deployment from your own machine.

### Explore the repository structure

The template repository is organized like this:

```text
your-repo/
├── src/                              ← Power BI project files (PBIP)
│   ├── Sales.Report/
│   ├── Sales.SemanticModel/
│   ├── AnotherReport.Report/
│   ├── Sales.pbip
│   └── parameter.yml                 ← environment parameterization
├── scripts/
│   ├── deploy.py                     ← deployment script
│   ├── deploy.config                 ← workspace name configuration (for CI/CD)
│   └── bpa/                          ← BPA scripts and rule files
│       ├── bpa.ps1
│       ├── bpa-rules-semanticmodel.json
│       └── bpa-rules-report.json
├── .github/
│   └── workflows/
│       ├── deploy.yml                ← deployment workflow
│       └── bpa.yml                   ← quality checks workflow
├── requirements.txt                  ← Python dependencies
└── .gitignore
```

All scripts and workflows are already in place — you will **not** need to create any of these files.

### Understanding `scripts/deploy.py`

Open `scripts/deploy.py` in VS Code and take a moment to read through it. The script accepts three command-line arguments:

| Argument | Purpose |
| --- | --- |
| `--workspace_name` | Target Fabric workspace name. If omitted, the script resolves it from configuration files (see below). |
| `--environment` | An environment label (`DEV` or `PRD`). Used by `fabric-cicd` to apply the correct values from `parameter.yml`. Defaults to `DEV`. |
| `--spn-auth` | When `True`, authenticates via the Azure CLI session (service principal — used in GitHub Actions). When `False` (default), opens a **browser window for interactive login** — this is what you use locally. |

### Configuration files: `deploy.config` vs `.env`

The script resolves the target workspace name in this order of priority:

| Priority | Source | Committed to Git? | Purpose |
| --- | --- | --- | --- |
| 1 | `--workspace_name` CLI argument | n/a | Explicit override for one-off runs |
| 2 | `.env` file in the repository root | **No** (listed in `.gitignore`) | Personal overrides for local development — optional |
| 3 | `scripts/deploy.config` | **Yes** | Shared workspace configuration used by CI/CD and the team |

In practice:

- **`scripts/deploy.config`** is the main configuration file. It is committed to the repository and used by GitHub Actions. Edit this file to set the workspace names that the team and CI/CD pipeline should use.
- **`.env`** (in the repository root) is an optional file for local overrides. It is listed in `.gitignore` and never pushed to GitHub. If you need to deploy to a personal workspace that differs from the team default, create a `.env` file rather than editing `deploy.config`.

Both files use the same format — one variable per line:

```ini
PBI_WORKSPACE_PRD=Workshop - Lab 2
PBI_WORKSPACE_DEV=Workshop - Lab 2 (DEV)
```

> [!IMPORTANT]
> * The target workspaces **must already exist** in Microsoft Fabric before you can deploy to them. `fabric-cicd` does not create workspaces — it only publishes items to existing ones.
> * For automated deployments (Section 3), the **service principal** must be added to each workspace with at least **Contributor** permissions. See **Appendix A** for details.

### Configure your workspace names

Open `scripts/deploy.config` in VS Code. Replace the default workspace names with your actual Fabric workspace names:

```ini
PBI_WORKSPACE_PRD=<your production workspace name>
PBI_WORKSPACE_DEV=<your development workspace name>
```

> [!NOTE]
> The workspace names you enter here will be persisted with your project in git and will be used by GitHub Actions for automated deployments. If you need to deploy to a different workspace locally, use the `.env` file for overrides or the `--workspace_name` CLI argument for one-off runs.

### Install Python dependencies

Open the integrated terminal in VS Code (`Ctrl + '`). Verify that the terminal is using **Python 3.12** — check the status bar at the bottom or run `python --version`.

Then install the required packages:

```bash
pip install -r requirements.txt
```

This installs `fabric-cicd` and `python-dotenv` as well as any other transient dependencies into your active Python environment.

> [!TIP]
> If you have multiple Python versions installed and the `pip` command above installs packages into the wrong environment, use the following alternative to target a specific interpreter:
>
> ```bash
> python -m pip install -r requirements.txt
> ```
>
> Or, if you need to target Python 3.12 explicitly by its full path (common on Windows where both Python 3.11 and 3.12 are installed side-by-side), use:
>
> ```bash
> # Windows example
> C:\Users\<YourUser>\AppData\Local\Programs\Python\Python312\python.exe -m pip install -r requirements.txt
>
> # macOS / Linux example
> /usr/local/bin/python3.12 -m pip install -r requirements.txt
> ```
>
> `fabric-cicd` requires **Python 3.12**, so make sure the interpreter you use is 3.12.

> [!TIP]
> If you have multiple Python versions installed and want to avoid confusion, consider using a virtual environment for this project. The Python extension in VS Code can automatically create and manage **virtual environments** for you. Learn more in the [VS Code Python documentation](https://code.visualstudio.com/docs/python/environments).

### Run your first local deployment

There are two ways to run the deployment script. The recommended approach for this workshop uses the VS Code Python Extension; the command-line alternative is listed at the end.

#### Option A: Using the VS Code Extension

1. Make sure **Python 3.12** is selected in the status bar (see above).

2. Open the `deploy.py` script in VS Code.

3. Select the **Run Python File** play button in the top-right corner of the editor:

    ![run-py-file](resources/img/vscode-run-py-file.png)

    The Python extension launches a new terminal and runs the script.
    The script reads your workspace name from `deploy.config` (or `.env` if present) and starts the deployment.
    Since you cannot specify any command-line arguments from the VS Code run button, it defaults to the `DEV` environment.

4. A **browser window** will open asking you to sign in with your Microsoft account. This is the interactive authentication flow — it is only used for local runs, not in CI/CD.

5. After sign-in, `fabric-cicd` publishes the items to your workspace. You should see output indicating each semantic model and report being deployed.

    ![local deployment output](resources/img/local-tool-output.png)

6. Once the script completes, open your **DEV** workspace in the Fabric portal and verify that the *Sales* semantic model and reports have appeared.

#### Option B: Command line (advanced)

If you prefer working directly on the command-line, open any terminal (for instance, the one included in VS Code), navigate to the repository root, and run:

```bash
python scripts/deploy.py --environment DEV
```

You can also override the workspace name directly:

```bash
python scripts/deploy.py --workspace_name "My Custom Workspace"
```

The advantage of this approach is that you can specify command-line arguments, but the VS Code method is more streamlined for local development and testing.

> [!TIP]
> Depending on your installation you might have to use `python3` instead of `python`. If the command points to a different version than 3.12, use the full path to your Python 3.12 installation or use the VS Code approach above, which makes switching interpreters straightforward.

### Data refresh after first deployment

After the **first** deployment to a workspace, the report visuals will not load because the semantic model has no data yet. `fabric-cicd` deploys only **definitions** (metadata), not data.

To load data:

1. In the Fabric portal, navigate to your deployed semantic model's **Settings** and take over ownership from the deployment principal.
2. Under **Data Source Credentials**, configure the `Web` data source with **Anonymous** authentication.
3. Trigger a **manual refresh**.

> [!IMPORTANT]
> This one-time setup is only needed after the **first** deployment to a workspace. Subsequent deployments will not require re-configuring credentials or triggering a manual refresh.

### Commit your configuration changes

Commit your `deploy.config` changes:

```bash
git add scripts/deploy.config
git commit -m "Configure workspace names for my environment"
```

Do **not** push to GitHub yet — we will do that in the next section after setting up the GitHub Actions secret.

## 3. GitHub Actions

**Goal**: Configure the deployment secret, understand the automated workflow, and trigger your first automated deployment.

### Add the AZURE_CREDENTIALS secret

GitHub Actions needs a single secret to authenticate as the service principal. This is the only manual configuration step on GitHub — everything else comes from the template.

1. In your GitHub repository, navigate to **Settings > Security > Secrets and variables > Actions**.

2. Click **New repository secret**.

    ![new-secret](resources/img/gh-actions-new-secret.png)

3. Set the **Name** to `AZURE_CREDENTIALS` and paste the JSON you prepared in Section 0 as the **Secret** value. Click **Add secret**.

4. Verify that the secret appears in the repository secrets list.

    ![secret-added](resources/img/gh-actions-secrets-list.png)

> [!WARNING]
> The `clientSecret` inside this JSON has an **expiration date** set when the service principal was created. When it expires, **all automated deployments will fail** with an authentication error. Track the expiration date, generate a new client secret in Entra ID **before** it runs out, and update the `AZURE_CREDENTIALS` secret in GitHub.

> [!NOTE]
> When you created the repository from the template, the deployment workflow may have run automatically. That run will have failed because `AZURE_CREDENTIALS` did not exist yet — this is expected. You can safely ignore or delete the failed run in the **Actions** tab.

### Overview of the deployment workflow

Open `.github/workflows/deploy.yml` in VS Code. This workflow automates the same `deploy.py` script you just ran locally. Here is when it runs:

| Trigger | Environment | What happens |
| --- | --- | --- |
| **Push to `main`** | `PRD` | Deploys to the production workspace |
| **Pull Request to `main`** | `DEV` | Deploys to the development workspace for validation |
| **Manual dispatch** | Your choice | You pick the workspace and environment in the GitHub UI |

The workflow steps are:

1. **Check out** the repository code
2. **Set up Python 3.12**
3. **Install dependencies** from `requirements.txt`
4. **Azure Login** using the `AZURE_CREDENTIALS` secret (via the [`azure/login`](https://github.com/marketplace/actions/azure-login) action)
5. **Run `deploy.py`** with `--spn-auth True` — this uses the Azure CLI session established by `azure/login` instead of interactive browser login

> [!TIP]
> The `allow-no-subscriptions: true` flag in the Azure Login step is needed because Fabric does not require an Azure subscription — only an Entra ID tenant.

### Requirements for automated deployment

Before GitHub Actions can deploy successfully, all of the following must be in place:

- [ ] **DEV and PRD Fabric workspaces** created with sufficient capacity (Premium or Fabric)
- [ ] **Workspace names configured** in `scripts/deploy.config`
- [ ] **Service principal** created in Entra ID with a client secret (see **Appendix A**)
- [ ] **Service principal authorized** to call Fabric APIs (Fabric Admin portal setting)
- [ ] **Service principal added** to both workspaces with Contributor (or higher) permissions
- [ ] **`AZURE_CREDENTIALS` secret** configured in the GitHub repository (above)

### Trigger your first automated deployment

Push your local changes (the `deploy.config` edits from Section 2) to GitHub:

```bash
git push
```

Since this is a push to `main`, the deployment workflow will trigger automatically.

1. Navigate to the **Actions** tab in your GitHub repository.

    ![actions tab with workflow run](resources/img/gh-published-actions.png)

2. Click on the running workflow to see its progress.

    ![deployment in progress](resources/img/gh-deploying-to-PRD.png)

3. Once complete, the run should show a green checkmark.

    ![deployment succeeded](resources/img/gh-deploy-succeeded.png)

### Verify the deployment

Open your **PRD** workspace in the Fabric portal. You should see the *Sales* report and semantic model:

![Fabric workspace with deployed items](resources/img/fabric-sales.png)

If this is the first deployment to the PRD workspace, you will need to configure data source credentials and trigger a manual refresh — the same steps described at the end of Section 2.

![report with no data](resources/img/pbi-report-needs-refresh.png)

![data load error](resources/img/pbi-couldnt-load-data.png)

1. Take over semantic model ownership in **Settings**.
2. Configure the `Web` data source with **Anonymous** authentication.
3. Trigger a **manual refresh**.

    ![trigger manual refresh](resources/img/pbi-manual-refresh.png)

4. The report should now display data.

    ![report with refreshed data](resources/img/pbi-report-refreshed.png)

> [!TIP]
> To learn more about GitHub Actions, see the [GitHub Actions Quickstart](https://docs.github.com/en/actions/get-started/quickstart).

## 4. Best Practice Analyzer (BPA)

**Goal**: Understand the automated quality checks that validate your Power BI project.

### What is BPA?

The repository includes automated quality checks powered by two community tools:

- **[Tabular Editor Best Practice Analyzer](https://docs.tabulareditor.com/te2/Best-Practice-Analyzer.html)** — scans **semantic models** for common issues such as missing format strings, unused columns, default summarization on numeric columns, and DAX anti-patterns.
- **[PBI-InspectorV2](https://github.com/NatVanG/PBI-InspectorV2)** — validates **Power BI reports** against configurable rules covering visual density, accessibility (alt text), theme compliance, and more.

Together, these tools act as an automated code review for your Power BI project.

### Severity levels

Both tools classify rule violations into severity levels. The severity determines whether the CI/CD pipeline passes or fails:

| Severity | Effect on pipeline | When to use |
| --- | --- | --- |
| **Error** | Pipeline **fails** — the check is blocked until the issue is fixed | Critical issues that must be resolved (e.g., missing descriptions on visible columns) |
| **Warning** | Pipeline **passes** but the issue is flagged in the log | Best-practice recommendations worth reviewing (e.g., redundant DAX expressions) |
| **Info** | Logged for informational purposes only, no impact | Suggestions and style notes |

> [!IMPORTANT]
> When a BPA rule at **Error** severity is violated, the GitHub Actions workflow step returns a non-zero exit code, which fails the pipeline. **Warning** and **Info** violations are logged but do not block the pipeline.

### The BPA workflow

Open `.github/workflows/bpa.yml` to see how BPA is triggered:

- **Pull Requests to `main`** — runs automatically whenever files under `src/` are changed
- **Manual dispatch** — can be triggered manually from the **Actions** tab at any time

The workflow runs on a **Windows** runner (required by the BPA tools) and executes two steps:

1. **BPA Semantic Models** — runs `scripts/bpa/bpa.ps1` against all `.SemanticModel` folders
2. **BPA Reports** — runs `scripts/bpa/bpa.ps1` against all `.Report` folders

> [!NOTE]
> The report validation step runs even if the semantic model step fails, so you always get the complete picture in a single run.

### BPA scripts and rule files

The `scripts/bpa/` directory contains:

| File | Purpose |
| --- | --- |
| `bpa.ps1` | PowerShell script that downloads the BPA tools (on first run) and executes them against your source files |
| `bpa-rules-semanticmodel.json` | Rule definitions for semantic model validation (Tabular Editor format) |
| `bpa-rules-report.json` | Rule definitions for report validation (PBI Inspector format) |

The tools themselves (Tabular Editor CLI and PBI Inspector CLI) are automatically downloaded and cached in `scripts/bpa/_tools/` on first run — you do not need to install them manually.

> [!TIP]
> You can customize the BPA rules by editing the JSON rule files. For example, you might downgrade a rule from **Error** to **Warning** if it is too strict for your team, or add entirely new rules. See the [Tabular Editor BPA documentation](https://docs.tabulareditor.com/common/using-bpa.html) and the [PBI-InspectorV2 rules reference](https://github.com/NatVanG/PBI-InspectorV2) for details.

### Running BPA locally (optional)

If you have **PowerShell** installed (see Section 0), you can run BPA on your machine before pushing changes:

```powershell
.\scripts\bpa\bpa.ps1 -src @("./src/*.SemanticModel")
.\scripts\bpa\bpa.ps1 -src @("./src/*.Report")
```

This is useful for catching issues early, before they reach a Pull Request.

## 5. Branch protection setup

> [!NOTE]
> **Advanced / Optional** — This is a one-time administrative setup step. If branch protection rules are already configured in your repository, skip ahead to [Section 6: Pull Request workflow](#6-pull-request-workflow).

**Goal**: Protect the `main` branch so that changes can only land through a reviewed and quality-checked Pull Request.

### Set up branch protection rules

Since any push to `main` triggers an automated deployment to production, it is critical to protect this branch. With branch protection enabled, changes can only reach `main` through a **Pull Request**, which triggers the BPA quality checks first.

1. In your GitHub repository, navigate to **Settings > Branches** and click **Add branch ruleset**.

    ![branch settings](resources/img/gh-settings-branches.png)

2. Provide a name for the ruleset (e.g., "Protect main") and under **Targets**, select **Default branch**.

    ![create branch ruleset](resources/img/gh-branchprotection-create.png)

3. Enable the rules shown below. Under **Require status checks to pass**, search for and select the **bpa** workflow.

    ![branch protection rules](resources/img/gh-branchprotection-rules.png)

4. Save the ruleset. The `main` branch now shows a protected icon in the **Branches** overview.

    ![branch protection applied](resources/img/gh-branches-protection-applied.png)

> [!TIP]
> By requiring the **bpa** status check to pass, Pull Requests with BPA **Error**-level violations cannot be merged. This enforces quality standards automatically — no manual review needed to catch common issues.

## 6. Pull Request workflow

**Goal**: Experience the complete Pull Request workflow — from branching to merge — with BPA quality gates running automatically at every step.

This is the day-to-day development workflow when branch protection is active. Every change to `main` goes through a Pull Request, which triggers the BPA checks and a deployment to the **DEV** workspace. Only when all checks pass can the change be merged — and the production deployment triggered.

### Start work in a new branch

Create a new branch, make a minor change (for example, replace all occurrences of "MyCompany" in the report with your company name), commit the change, and push the new branch to GitHub.

### Create a Pull Request

On github.com, your repository should now show a banner offering to create a Pull Request for your newly pushed branch:

![create PR banner](resources/img/gh-create-pr-banner.png)

Fill in a **title** and optional **description**, then click **Create pull request**.

This triggers two workflows:
- The **BPA** workflow runs quality checks on the changed files
- The **deploy** workflow deploys to the **DEV** workspace (for validation)

![PR checks running](resources/img/gh-pr-checks-running.png)

### Fix BPA issues

The BPA check is expected to **fail** at this point — one of the semantic model rules has an **Error**-level violation. This blocks the Pull Request from being merged.

![PR blocked](resources/img/gh-pr-blocked.png)

Navigate to the failed check run to see the details. Then fix the issue in VS Code, commit, and push to the same branch. The Pull Request will pick up the new commit and re-run the checks.

> [!TIP]
> See the screenshot below for a hint of what needs fixing.

![BPA fix hint](resources/img/bpa-fix-semanticmodel.png)

After pushing the fix, the BPA check should pass:

![PR checks passed](resources/img/gh-pr-checks-passed.png)

### Merge the Pull Request

Once the checks pass and you (or your reviewer) are satisfied, click the green **Merge pull request** button. This merges your changes into `main` and **automatically triggers the deployment workflow** to production.

After merging, delete the branch — the Pull Request page shows a **Delete branch** button for this purpose.

> [!NOTE]
> Deleting a branch on GitHub does **not** automatically delete it on your local machine. Remember to clean up locally with `git branch -d <branch-name>` and `git fetch --prune`.

## Wrap-up

You've now:

- Created a repository from a template with a complete CI/CD setup
- Configured a service principal and GitHub secret for automated authentication
- Deployed Power BI items locally using `fabric-cicd` and interactive browser login
- Triggered automated deployments to production via GitHub Actions
- Used BPA quality gates to catch issues before they reach production
- Completed a full Pull Request workflow — from branch to merge — with automated quality gates enforced at every step

### Next steps

If you want to explore further after the workshop:

- **Multi-environment parameterization** — use `src/parameter.yml` to swap workspace IDs, connection strings, or report settings per environment. See the [fabric-cicd parameterization docs](https://microsoft.github.io/fabric-cicd/latest/how_to/parameterization/).
- **Custom BPA rules** — tailor the rule files in `scripts/bpa/` to match your team's standards.
- **GitHub Environments** — use [GitHub environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/managing-environments-for-deployment) for approval gates and environment-specific secrets.
- **Azure DevOps** — `fabric-cicd` works equally well with Azure Pipelines. See the [official Microsoft docs](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-deploy-fabric-cicd).

## Useful links

- [Reference repository: workshops-cicd-demo](https://github.com/RuiRomano/workshops-cicd-demo) — the template used for this lab
- [fabric-cicd documentation](https://microsoft.github.io/fabric-cicd/latest/)
- [Deploy Power BI projects using fabric-cicd](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-deploy-fabric-cicd)
- [Tabular Editor Best Practice Analyzer](https://docs.tabulareditor.com/te2/Best-Practice-Analyzer.html)
- [PBI-InspectorV2](https://github.com/NatVanG/PBI-InspectorV2)
- [GitHub Actions Quickstart](https://docs.github.com/en/actions/get-started/quickstart)
- [GitHub: About Pull Requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
- [GitHub: Protected Branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)

---

## Appendix A: Service Principal Setup

> Detailed step-by-step instructions for this one-time setup are in a separate document: **[appendix-a-service-principal-setup.md](appendix-a-service-principal-setup.md)**.
