# Power BI for Developers: PBIP and CI/CD Unleashed

Hands-on deep dive into Power BI developer mode and the powerful capabilities unlocked by Power BI Project files (PBIP). In this session, you’ll explore TMDL and PBIR file formats, learning how they open the door to structured, source-controlled development workflows.

By the end of this workshop, you will:
- Gain a solid understanding of Power BI Project file formats (PBIP, TMDL, PBIR) and how they support modern, scalable development workflows.
- Learn how to effectively leverage AI to accelerate and enhance Power BI development, with hands-on, practical examples.
- Understand the value of applying DevOps and CI/CD practices to Power BI projects and walk away with actionable solutions—both out-of-the-box and community-driven.

## Get started

- Download this repository to your machine.
  
  ![clone-repository](.labs/lab1/resources/img/clone-repository.png)
  
- Ensure you met all the [Requirements](#requirements) before starting the labs.
- Open the lab documents in your browser for an improved reading experience.

## Agenda

| Topic                                    | Content                                                                                                    | Labs
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------- | -----------------------------------------------------------------
| **Power BI Project (PBIP) fundamentals** | • PBIP fundamentals<br>• Git basics<br>• Semantic Modeling as code with TMDL<br>• Report as code with PBIR | [Lab 1 - Power BI Project (PBIP) fundamentals](.labs/lab1/lab.md)
| **Power BI & CI/CD**                     | • GitHub Collaboration Workflows<br>• CI/CD Pipelines                                                      | [Lab 2 - Power BI & CI/CD](.labs/lab2/README.md)
| **Power BI Development with AI**         | • Power BI Development with AI <br>• Agentic development                                                   | [Lab 3 - Power BI Development with AI](.labs/lab3/lab.md)

## Requirements

> [!IMPORTANT]
> * We can provide a Fabric Account and GitHub Copilot license for this workshop. Request access in this [form](https://forms.office.com/e/47Jx6CqPSq)

- Licenses
  - Access to a Fabric / Power BI tenant
    - You can use your existing organizational tenant but you must have admin permissions.
      - You can also create your own tenant using the [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program)
    - Power BI Pro license
    - Access to a Fabric Capacity or [Trial](https://learn.microsoft.com/en-us/fabric/fundamentals/fabric-trial)
    - Access to create and authorize a service principal in Fabric for automated CI/CD deployment. See [Create a Microsoft Entra ID app](https://learn.microsoft.com/en-us/rest/api/fabric/articles/get-started/create-entra-app).
  - [GitHub account](https://github.com/signup)
  - [GitHub Copilot Free or Pro](https://github.com/github-copilot/pro)
- Software
  - [Power BI Desktop](https://pbi.onl/download)
  - [Visual Studio Code](https://code.visualstudio.com/download)
  - [Git for Windows](https://gitforwindows.org/)
  - [Python 3.12](https://apps.microsoft.com/detail/9ncvdn91xzqp)