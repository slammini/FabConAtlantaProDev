# Lab 2 — Power BI & CI/CD

This lab guides you through building a production-grade CI/CD pipeline for a Power BI project on GitHub. Starting from a pre-configured repository template, you will configure automated deployments using GitHub Actions, enforce quality gates with the Best Practice Analyzer, and practice the Pull Request workflow that ties it all together.

No local setup is required — all you need is a GitHub account and the provided service principal credentials.

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
| [appendix-local-deployment.md](appendix-local-deployment.md) | Instructions for running deployments from your local machine using interactive browser login. Optional — the main lab uses GitHub Actions for all deployments. |

## Lab structure

The main lab document is organized into the following sections:

| # | Section | Notes |
| - | ------- | ----- |
| 0 | **Setup** | Create your repository from the template and prepare credentials |
| 1 | **Introduction to fabric-cicd** | Conceptual overview of the deployment tool |
| 2 | **GitHub Actions** | Configure workspace names, set up the deployment secret, and trigger automated deployments |
| 3 | **Best Practice Analyzer (BPA)** | Understand the automated quality checks included in the template |
| 4 | **Branch protection setup** | ⚙️ *Advanced / Optional* — One-time configuration to enforce the PR workflow |
| 5 | **Pull Request workflow** | ⭐ *Core exercise* — Work through a complete branch → PR → BPA check → merge cycle |

> [!TIP]
> If you are short on time, sections 4 and 5 are the most representative of real-world team workflows. Section 4 can be skipped if branch protection is already configured; section 5 is the hands-on highlight of the lab.
