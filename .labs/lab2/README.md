# Lab 2 — Power BI & CI/CD

⏱️ Duration: 110 minutes

This lab guides you through building a production-grade CI/CD pipeline for a Power BI project on GitHub. Starting from a pre-configured repository template, you will configure automated deployments, enforce quality gates with the Best Practice Analyzer, and practice the Pull Request workflow that ties it all together.

## What you will learn

- How to use **`fabric-cicd`** — Microsoft's recommended tool for deploying Power BI Project (PBIP) files to Fabric workspaces
- How **GitHub Actions** automates deployments on every push to `main`
- How the **Best Practice Analyzer (BPA)** catches semantic model and report issues automatically on Pull Requests
- How to work within a **branch-and-PR workflow** that enforces quality standards before any change reaches production

## Files in this folder

| File | Description |
| ---- | ----------- |
| [lab.md](lab.md) | **Main lab document** — work through this from top to bottom |
| [appendix-a-service-principal-setup.md](appendix-a-service-principal-setup.md) | Step-by-step instructions for setting up a service principal in Microsoft Entra ID. Only needed if a service principal has not already been provisioned for your workshop environment. |

## Lab structure

The main lab document is organized into the following sections:

| # | Section | Notes |
| - | ------- | ----- |
| 0 | **Setup** | Create your repository from the template, clone it, and prepare credentials |
| 1 | **Introduction to fabric-cicd** | Conceptual overview of the deployment tool |
| 2 | **Local deployment** | Configure workspace names and run your first deployment from your machine |
| 3 | **GitHub Actions** | Set up the deployment secret and trigger automated deployments |
| 4 | **Best Practice Analyzer (BPA)** | Understand the automated quality checks included in the template |
| 5 | **Branch protection setup** | ⚙️ *Advanced / Optional* — One-time configuration to enforce the PR workflow |
| 6 | **Pull Request workflow** | ⭐ *Core exercise* — Work through a complete branch → PR → BPA check → merge cycle |

> [!TIP]
> If you are short on time, sections 5 and 6 are the most representative of real-world team workflows. Section 5 can be skipped if branch protection is already configured; section 6 is the hands-on highlight of the lab.
