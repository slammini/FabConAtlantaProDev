# Lab - Power BI and CI/CD

⏱️ Duration: 90 minutes

In this lab, you will learn how to build Power BI solutions in a more reliable, scalable, and team‑friendly way by using CI/CD (Continuous Integration and Continuous Deployment). Instead of manually updating reports and models, you’ll see how changes can be automatically tested and deployed using standard development practices. This approach helps reduce mistakes, makes collaboration easier, and ensures that what you build in development is safely promoted to test and production environments. This is especially useful for teams working on complex Power BI projects or managing enterprise‑scale solutions.

This lab guides you through building a production-grade CI/CD pipeline for a Power BI project on GitHub. Starting from a pre-configured repository template, you will configure automated deployments using GitHub Actions, enforce quality gates with the Best Practice Analyzer, and practice the Pull Request workflow that ties it all together.

No local setup is required for this lab - all you need is a GitHub account and the provided service principal credentials. An optional [appendix-local-deployment](appendix-local-deployment.md) is available if you want to run deployments from your local machine.

## What you will learn

- How to use **`fabric-cicd`** - Microsoft's recommended tool for code deployment of Power BI Project (PBIP) files to Fabric workspaces
- How **GitHub Actions** automates deployments on every push to `main`
- How the **Best Practice Analyzer (BPA)** catches semantic model and report issues automatically on Pull Requests
- How to work within a **branch-and-PR workflow** that enforces quality standards before any change reaches production

## Lab structure

| # | Section | Notes |
| - | ------- | ----- |
| - | [Prerequisites](#-prerequisites) | GitHub account and service principal credentials  |
| - | [Introduction to fabric-cicd](#introduction-to-fabric-cicd) | Overview of the Microsoft-recommended deployment library and how it works |
| 1 | [Setup GitHub Repository from template](#1-setup-github-repository-from-template) | Create your repo from the template, configure the `AZURE_CREDENTIALS` secret, and set up Fabric workspaces |
| 2 | [GitHub Actions workflows](#2-github-actions-workflows) | Explore the pre-built `deploy` and `bpa` workflows; trigger your first automated deployment |
| 3 | [Pull Request workflow](#3-pull-request-workflow) | Branch, edit a TMDL file, open a PR, fix a BPA violation, and merge to trigger a production deployment |

## 🛠️ Prerequisites

- [GitHub account](https://github.com/signup)
- Service principal

## Introduction to fabric-cicd

How does source code in a GitHub repository actually get deployed to a Fabric workspace? The answer is [`fabric-cicd`](https://microsoft.github.io/fabric-cicd/latest/) - a Python library developed by Microsoft that is the recommended tool for deploying Fabric items from code repositories.

`fabric-cicd` takes the PBIP definition files in your repository and publishes them to a target workspace via the Fabric REST APIs. Rather than calling APIs directly, you write a short Python script that configures `fabric-cicd` and lets the library handle the heavy lifting - API calls, retries, status polling, and error handling.

A key design principle is that **the same script works identically on your local machine and inside a GitHub Actions workflow**. This means you can test deployments locally first and reproduce CI/CD failures on your workstation if needed.

### Why fabric-cicd?

| Advantage                       | Description                                                                     |
| ------------------------------- | ------------------------------------------------------------------------------- |
| **Fabric-native REST APIs**     | Built on official Fabric APIs, ensuring long-term compatibility                 |
| **Python-native**               | Integrates naturally with modern DevOps workflows                               |
| **Parameterization**            | Built-in support for environment-specific values via `parameter.yml`            |
| **Flexible deployment control** | Deploy specific item types (e.g., only semantic models)                         |
| **Reliable authentication**     | Uses the Azure Identity SDK - browser login locally, service principal in CI/CD |

In the next section you will see how GitHub Actions automates the deployment process on every push.

> [!NOTE]
> - This workshop uses a single target workspace per environment (DEV and PRD). In production scenarios, `fabric-cicd` supports advanced multi-environment parameterization via `parameter.yml`. See the [fabric-cicd documentation](https://microsoft.github.io/fabric-cicd/latest/) for details.
> - If you want to try running deployments from your local machine, see **[Appendix: Local Deployment](appendix-local-deployment.md)**.

## 1. Setup GitHub Repository from template

✅ **Goal**: Create your own copy of the lab repository on GitHub and prepare the deployment credentials.

### Create a new GitHub repository from the template

1. Navigate to the template repository: **<https://github.com/RuiRomano/workshops-cicd-demo>**

2. Click the green **Use this template** button, then select **Create a new repository**.

    ![use-template](resources/img/gh-template-use.png)

3. Choose an **Owner**, give your repository a **name** (for example, `workshop-cicd`), and select the desired **visibility** (Public or Private). Then click **Create repository**.

    ![new-repo](resources/img/gh-template-new-repo.png)

> [!TIP]
> Creating a repository from a template gives you a clean copy with the full directory structure and all files, but without the template's commit history. Learn more in the [GitHub docs on template repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template).

### Explore the repository structure

Take a moment to browse the files in your newly created repository on GitHub. The template is organized like this:

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

> [!NOTE]
> - All scripts and workflows are already in place - you will **not** need to create any of these files.
> - More details about the scripts in the [workshops-cicd-demo](https://github.com/RuiRomano/workshops-cicd-demo) repository.

### Configure the Service Principal secret

Automated deployments via GitHub Actions require a **service principal** - an identity in Microsoft Entra ID (formerly Azure AD) that represents an application rather than a human user. Since this identity provides tenant access, it must be stored in a secure and encrypted location.

> [!IMPORTANT]
> In this workshop, a preconfigured service principal is provided for use in the workshop tenant. If you want to try these steps in your own tenant, follow the instructions in **[Appendix: Service Principal Setup](appendix-service-principal-setup.md)** to learn how to create and authorize a service principal.


1. In your GitHub repository, navigate to **Settings > Security > Secrets and variables > Actions**.
2. Click **New repository secret**
3. Set the **Name** to `AZURE_CREDENTIALS`
4. Open the URL [Service Principal Details](https://1drv.ms/t/c/5d0350bbe4220916/IQAhwo1rpf_4RI0GTma930yBAWV77VYFvbnpDhFU3KafIBs?e=8BCNzb) and copy the JSON in `## AZURE_CREDENTIALS.json`
5. Paste the JSON to the **Secret** value.     

    ![new-secret](resources/img/gh-actions-new-secret.png)

6. Click **Add secret**.

> [!NOTE]
> When you created the repository from the template, the deployment workflow may have run automatically. That run will have failed because `AZURE_CREDENTIALS` did not exist yet - this is expected. You can safely ignore or delete the failed run in the **Actions** tab.

### Create the environment workspaces in Fabric

The target workspaces **must already exist** in Microsoft Fabric before you can deploy to them. **fabric-cicd** does not create workspaces - it only publishes items to existing ones.

1. In Microsoft Fabric tenant, create two workspaces.

    - Development Workspace: `Workshop - Lab 2 (DEV) - <Your Initials>`
    - Production Workspace: `Workshop - Lab 2 (PRD) - <Your Initials>`
  
    **Note:** Use your initials as suffix to avoid conflicts with other attendees.

2. Authorize the service principal with at least **Contributor** permissions to both workspaces.

    Get the Service Principal name from [Service Principal Details](https://1drv.ms/t/c/5d0350bbe4220916/IQAhwo1rpf_4RI0GTma930yBAWV77VYFvbnpDhFU3KafIBs?e=8BCNzb).

    ![add SP to workspace](resources/img/fabric-workspace-add-service-principal.png)

### Configure your environment workspaces in `deploy.config`

Before the deployment workflow can run, you must specify the Fabric workspaces to deploy to. The template repository you forked to your GitHub account uses a predefined configuration that must be updated, particularly if it runs in the same tenant as other attendees, to avoid conflicts.

1. In your GitHub repository, navigate to the file `scripts/deploy.config` and click the **pencil icon** (✏️) to edit it directly on GitHub.

2. Replace the default workspace names with your actual Fabric workspace names:

    ```ini
    PBI_WORKSPACE_PRD=<your production workspace name>
    PBI_WORKSPACE_DEV=<your development workspace name>
    ```

    ![change deploy.config](resources/img/gh-deploy-config.png)

3. Click **Commit changes**, add a commit message (e.g., "Configure workspace names for my environment"), and commit directly to `main`.

## 2. GitHub Actions workflows

✅ **Goal**: Understang GitHub Actions workflows and trigger your first automated deployment.

[GitHub Actions workflows](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflows#about-workflows) are a configurable automated processes that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule. 

The repo includes two pre-configured workflows:

| Workflow   | File                           | Description                                                                                                                                                                                             | 
| ---------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | 
| **deploy** | `.github/workflows/deploy.yml` | Deploys PBIP source files in `src/` to a Fabric workspace using `fabric-cicd`. <br/>Targets **PRD** environment on push to `main`, **DEV** on Pull Requests, and a user-chosen environment on manual run. | 
| **bpa**    | `.github/workflows/bpa.yml`    | Runs automated quality checks against the Power BI project. It's powered by community tools [**Tabular Editor**](https://github.com/TabularEditor/TabularEditor) for semantic models and [**PBI-InspectorV2**](https://github.com/NatVanG/PBI-InspectorV2) for reports. <br/>Run automatically on every pull-request to `main`.<br/>BPA rules are defined in `scripts/bpa/bpa-rules-*.json`. These rules can be customized. See the [Tabular Editor BPA documentation](https://docs.tabulareditor.com/common/using-bpa.html) and the [PBI-InspectorV2 rules reference](https://github.com/NatVanG/PBI-InspectorV2) for details. | 

Take some time to review the YAML code of both workflows. To learn more about GitHub Actions, see the [GitHub Actions Quickstart](https://docs.github.com/en/actions/get-started/quickstart).

### Manual run `deploy` workflow

1. Navigate to the **Actions** tab in your GitHub repository
2. Open the `deploy` workflow

    ![gh-actions-open-deploy](resources/img/gh-actions-open-deploy.png)    

3. Click on **Run workflow**, type the name of your **DEV** workspace and click Run

    ![gh-actions-run-deploy](resources/img/gh-actions-run-deploy.png)

4. The workflow will start running and you can monitor its progress

    ![gh-actions-running](resources/img/gh-actions-running.png)

5. Once complete, the run should show a green checkmark.

    ![gh-deploy-succeeded](resources/img/gh-deploy-succeeded.png)

    Click the `deploy` job to view more details about the deployment with **fabric-cicd**:

    ![gh-deploy-succeeded-details](resources/img/gh-deploy-succeeded-details.png)

    In case of errors, confirm the following:

   - [ ] **Workspace configured in Run Workflow** exists in your Fabric tenant   
   - [ ] **Service Principal** have at least **Contributor** permissions to the workspace     
   - [ ] **`AZURE_CREDENTIALS` secret** configured in the GitHub repository
   
6. Open your **DEV** workspace in the Fabric portal. You should see the reports and semantic model deployed.

    ![Fabric workspace with deployed items](resources/img/fabric-sales.png)

7. Open the Sales Report

    PBIP deployment does not publish data by design. As a result, the report visuals display errors.

    ![report with no data](resources/img/pbi-report-needs-refresh.png)

8. Open the semantic model settings

    Take the ownership of the semantic model:

    ![take ownership](resources/img/pbi-model-takeover.png)

    Configure the credentials with Authentication: **Anonymous** and **Skip test connection**:

    ![pbi-model-config-credentials](resources/img/pbi-model-config-credentials.png)

9. Refresh the semantic model
    
10. Open the Sales report and refresh the page. The report should now display data.

    ![report with refreshed data](resources/img/pbi-report-refreshed.png)

### Manual run `bpa` workflow

1. Navigate to the **Actions** tab in your GitHub repository
2. Open the `bpa` workflow and click on **Run workflow**
   
   ![gh-actions-bpa-workflow](resources/img/gh-actions-bpa-workflow.png)    

3. Wait for the workflow to complete
4. Select the **BPA** job to view the detailed results of the Best Practice Analyzer run on reports and semantic models located in the `src/` folder.

    Expand the **BPA Semantic Models** and **BPA Reports** and review all the detected issues:

    ![gh-actions-bpa-warnings](resources/img/gh-actions-bpa-warnings.png)    
    
    Both BPA tools classify rule violations into severity levels. The severity determines whether the CI/CD pipeline passes or fails:

    | Severity    | Effect on pipeline                                                 | When to use                                                                           |
    | ----------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
    | **Error**   | Pipeline **fails** — the check is blocked until the issue is fixed | Critical issues that must be resolved (e.g., missing descriptions on visible columns) |
    | **Warning** | Pipeline **passes** but the issue is flagged in the log            | Best-practice recommendations worth reviewing (e.g., redundant DAX expressions)       |
    | **Info**    | Logged for informational purposes only, no impact                  | Suggestions and style notes                                                           |


## 3. Pull Request workflow

✅ **Goal**: Experience the complete Pull Request workflow - from branching to merge - with BPA quality gates running automatically at every step.

This is and example of the day-to-day development workflow. Every change to `main` goes through a Pull Request, which triggers the BPA checks and a deployment to the **DEV** workspace. Only when all checks pass can the change be merged - and the production deployment triggered to the **PRD** workspace.

### Start work in a new branch

1. Create a new branch in GitHub

    Give it a name and select **Create branch...**

    ![gh-create-branch](resources/img/gh-create-branch.png)

2. Make some changes to the semantic model by editing a TMDL file in GitHub.

    Go to `src\Sales.SemanticModel\definition\tables\Sales.tmdl` and click the **pencil icon** (✏️) to edit it directly on GitHub.

    Add the following measure using TMDL:

    ```tmdl
    measure 'Sales Qty (LY)' = CALCULATE([Sales Qty], SAMEPERIODLASTYEAR('Calendar'[Date]))
		lineageTag: 019cddab-955e-72e4-afbd-612d555de377
    ```
    
    ![gh-branch-edit-model-tmdl](resources/img/gh-branch-edit-model-tmdl.png)

3. Commit the change

    ![gh-branch-commit-change](resources/img/gh-branch-commit-change.png)


### Create a Pull Request

Your repository should now show a banner offering to create a Pull Request (PR) for your newly pushed branch:

![create PR banner](resources/img/gh-create-pr-banner.png)

1. Click the banner button **Compare & pull request**
2. Fill in a **title** and optional **description**, then click **Create pull request**.

    ![gh-create-pr](resources/img/gh-create-pr.png)

3. The PR is created and will trigger two workflows
   
   - The **bpa** workflow runs quality checks on the changed files
   - The **deploy** workflow deploys to the **DEV** workspace (for validation)

    ![PR checks running](resources/img/gh-pr-checks-running.png)

### Confirm deploy to DEV

It's expected the deployment to **DEV** to be successfull and the BPA check to **fail**.

1. Confirm the deployment to **DEV** was successfull

    Open the deploy workflow directly from the PR:
    
    ![gh-pr-dev-success-check](resources/img/gh-pr-dev-success-check.png)

    Confirm in the logs the deploy was executed against **DEV** environment:

    ![gh-pr-dev-success-details](resources/img/gh-pr-dev-success-details.png)

2. Open the semantic model in **DEV** workspace and confirm the new measure is there.

    ![gh-pr-dev-model-measure](resources/img/gh-pr-dev-model-measure.png)

    This allows developers to validate their changes in a development environment, even if the changes violate a BPA rule.

### Fix BPA issues

The previous change introduced a violation of the modeling rule `PROVIDE_FORMAT_STRING_FOR_MEASURES`, defined in the semantic model BPA rules file `scripts\bpa\bpa-rules-semanticmodel.json`.

> [!IMPORTANT]
> Even when BPA checks fail, it is still possible to merge changes into the `main` branch. To prevent this, configure [GitHub rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) to block merges to `main` when required checks fail. See [appendix-branch-protection](appendix-branch-protection.md) for guidance, for private repositories a GitHub Pro, GitHub Team, and GitHub Enterprise Cloud is required.

1. Navigate to the failed check run to see the details. 
   
   ![gh-pr-dev-open-bpa](resources/img/gh-pr-dev-open-bpa.png)   
   
   Note that the BPA analysis reports a failure because the measure `Sales Qty (LY)` does not include a `formatString`.

   ![gh-pr-dev-open-bpa-details](resources/img/gh-pr-dev-open-bpa-details.png)

2. Edit the `src\Sales.SemanticModel\definition\tables\Sales.tmdl` file again in the development branch and add the `formatString`.

    ```tmdl
    measure 'Sales Qty (LY)' = CALCULATE([Sales Qty], SAMEPERIODLASTYEAR('Calendar'[Date]))
		formatString: #,##0
		lineageTag: 019cddab-955e-72e4-afbd-612d555de377
    ```
    
    ![gh-branch-edit-model-tmdl-fixed](resources/img/gh-branch-edit-model-tmdl-fixed.png)

3. Commit the changes, go back to the pull request and notice the deployment and BPA validations will execute again.

    This time, the pull request should show all checks passing, and you can merge the changes into the `main` branch.

    ![gh-pr-checks-passed](resources/img/gh-pr-checks-passed.png)
   
### Merge the Pull Request

1. Click the green **Merge pull request** button. 
   
   ![gh-pr-checks-passed](resources/img/gh-pr-checks-passed.png)

   This merges your changes into `main` and **automatically triggers the deployment workflow** to **PRD**.

2. A merge into the `main` branch triggers the `deploy` workflow because it is configured to run on pushes to the `main` branch.
3. Navigate to the **Actions** tab in your GitHub repository and note that the `deploy` workflow should be running.

    ![gh-pr-deploy-prd-running](resources/img/gh-pr-deploy-prd-running.png)
4. The deployment should be successful. Select the **deploy** job and confirm that it was deployed to the **PRD** environment.

    ![gh-pr-prd-success-details](resources/img/gh-pr-prd-success-details.png)

5. Confirm the measure created in the development branch is now available in the **PRD** semantic model.
   

## ✅ Wrap-up

You've now:

- Created a repository from a template with a complete CI/CD setup
- Configured a service principal and GitHub secret for automated authentication
- Triggered automated deployments to development and production environments via GitHub Actions
- Used BPA quality gates to catch issues before they reach production
- Completed a full Pull Request workflow - from branch to merge - with automated quality gates

### Next steps

If you want to explore further after the workshop:

- **Local deployment** - run `fabric-cicd` directly from your machine using interactive browser login. See **[Appendix: Local Deployment](appendix-local-deployment.md)** for step-by-step instructions.
- **Multi-environment parameterization** - use `src/parameter.yml` to swap workspace IDs, connection strings, or report settings per environment. See the [fabric-cicd parameterization docs](https://microsoft.github.io/fabric-cicd/latest/how_to/parameterization/).
- **Custom BPA rules** - tailor the rule files in `scripts/bpa/` to match your team's standards.
- **GitHub Environments** - use [GitHub environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/managing-environments-for-deployment) for approval gates and environment-specific secrets.
- **Azure DevOps** - `fabric-cicd` works equally well with Azure Pipelines. See the [official Microsoft docs](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-deploy-fabric-cicd).

## Useful links

- [Reference repository: workshops-cicd-demo](https://github.com/RuiRomano/workshops-cicd-demo) - the template used for this lab
- [fabric-cicd documentation](https://microsoft.github.io/fabric-cicd/latest/)
- [tutorial-fabric-cicd-azure-devops](https://learn.microsoft.com/en-us/fabric/cicd/tutorial-fabric-cicd-azure-devops)
- [Deploy Power BI projects using fabric-cicd](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-deploy-fabric-cicd)
- [Tabular Editor Best Practice Analyzer](https://docs.tabulareditor.com/te2/Best-Practice-Analyzer.html)
- [PBI-InspectorV2](https://github.com/NatVanG/PBI-InspectorV2)
- [GitHub Actions Quickstart](https://docs.github.com/en/actions/get-started/quickstart)
- [GitHub: About Pull Requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
- [GitHub: Protected Branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
