# Lab — Power BI Project (PBIP) Fundamentals

⏱️ Duration: 120 minutes

This lab gives you hands-on experience with the **Power BI Project (PBIP)** format. You will learn how PBIP files are organized on disk, set up Git source control for a project, publish to a Fabric workspace, and edit semantic models and reports directly as code using **TMDL** and **PBIR**.

## What you will learn

- How the **PBIP folder structure** maps to the report and semantic model you see in Power BI Desktop and the Fabric service
- How to use **Git** to version-control a Power BI project, track changes, and restore previous versions
- How to **publish a PBIP** to a Fabric workspace 
- How to read and edit a semantic model as code with **TMDL** (Tabular Model Definition Language)
- How to read and edit a report as code with **PBIR** (Power BI Report format)

## Lab structure

| # | Section | Notes |
| - | ------- | ----- |
| - | [Prerequisites](#️-prerequisites) | Enable preview features in Power BI Desktop and install required software |
| 1 | [Save a PBIX to PBIP Format](#1-save-a-pbix-to-pbip-format) | Convert an existing PBIX file into the PBIP folder structure |
| 2 | [Explore PBIP files and folders](#2-explore-pbip-files-and-folders) | Understand the key files and folders that make up a PBIP project |
| 3 | [Version Control your PBIP](#3-version-control-your-pbip) | Initialize a Git repository, commit changes, view history, and restore previous versions |
| 4 | [Publish Power BI Project to workspace](#4-publish-power-bi-project-to-workspace) | Deploy your PBIP project to a Fabric workspace |
| 5 | [Semantic Modeling as Code with TMDL](#5-semantic-modeling-as-code-with-tmdl) | Read and edit the semantic model definition using TMDL |
| 6 | [Report as Code with PBIR](#6-report-as-code-with-pbir) | Read and edit the report definition using PBIR |

## 🛠️ Prerequisites

* Enable the following **Power BI Desktop** preview features:
  * Power BI Project (.pbip) save option
  * Store semantic model using TMDL format
  * Store reports using enhanced metadata format (PBIR)
    
    ![preview features](resources/img/previewfeatures.png)

* Ensure you have the following software installed:    
  * [Visual Studio Code](https://code.visualstudio.com/download)  
    * [TMDL extension](https://marketplace.visualstudio.com/items?itemName=analysis-services.TMDL)
    * [Microsoft Fabric extension](https://marketplace.visualstudio.com/items?itemName=fabric.vscode-fabric)
    * [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  * [Git for Windows](https://gitforwindows.org/)  
  * [Python 3.13](https://apps.microsoft.com/detail/9pnrbtzxmb4z)	

## 1. Save a PBIX to PBIP Format

✅ **Goal**: Save your pbix as a PBIP project.

When you save your work as a Power BI Project (PBIP), both reports and semantic models are stored as individual plain‑text files organized in a clear, intuitive folder structure, making the contents easy to inspect and modify. This project‑based format unlocks text editor support, full transparency into report and model definitions, and first‑class compatibility with source control systems, while also enabling CI/CD pipelines and programmatic generation or editing of items—significantly improving collaboration, automation, and developer‑centric workflows compared to a single binary file.

To learn more about about PBIP: https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview

In this section, we'll see how to save a .pbix file as a PBIP. Let's get started!

1. Open [resources/Sales.pbix](resources/Sales.pbix) in **Power BI Desktop**.
2. Go to **File > Save As**.
3. Choose a folder (e.g. `c:\temp\lab1`) and select **Save as type**: `Power BI Project Files (*.pbip)`
4. Name it: `Sales.pbip` and click Save. **Power BI Desktop** will save your work as a PBIP folder instead of a single PBIX file.

5. You can recognize a Power BI Project by the **expanded title bar** in Desktop, which also allows you to identify and open the PBIP folder directly in Windows Explorer.

    ![flyout](resources/img/flyout.png)

6. Although **Power BI Desktop** presents the experience as if you're working with a single file, you're actually editing two distinct components: the report and the semantic model, each stored in its own folder within the PBIP structure.
   
    ```text
    Lab1/
    ├── Sales.Report/
    ├── Sales.SemanticModel/
    ├── .gitignore
    └── Sales.pbip
    ```
    PBIP mimics the experience you get in the service when you publish a PBIX - unless its a live connect report, you always get two items in the workspace after publishing a PBIX report: a report and semantic model.

> [!TIP]
> * The provided PBIX file uses mock data sourced from CSV files hosted in a public location. When prompted for authentication, select **Anonymous** - the data should refresh without any errors.


<project name>.Semantic model contains a collection of files and folders that represent a Power BI semantic model. To learn more about the files and subfolders and files in here, see [Project Semantic Model folder](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-dataset).

<project name>.Report contains a collection of files and folders that represent a Power BI report. To learn more about the files and subfolders and files in here, see [Project report folder](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report).

.gitIgnore specifies intentionally untracked files Git should ignore for Power BI Project files.

<project name>.pbip contains a pointer to a report folder. Opening a PBIP opens th e targeted report and model for authoring. 

## 2. Explore PBIP files and folders

✅ **Goal**: Understand how Power BI Project is organized and its key files and folders.

Now that our PBIP is ready, let's take a closer look at the folders and files in the project's root folder to understand how it's organized.

1. Open **Visual Studio Code**
1. Go to **File > Open Folder...** and open the saved PBIP folder.
1. Open and explore the following key files and folders:
   
    | File/Folder                            | Description |
    |----------------------------------------|-------------|
    | `*.pbip`                               | Main entry point file for **Power BI Desktop**. Includes a reference to the Report folder. **Optional** file; **Power BI Desktop** can also open a report by opening the `definition.pbir`. |
    | `*.Report/definition.pbir`             | Contains the overall report definition and key configuration settings such as folder version. Includes a reference to the **semantic model** - typically via a **relative byPath reference**, but can also use an **absolute byConnection reference** to connect to a model hosted in Fabric. |
    | `*.Report/definition`                  | Contains the report definition in [**PBIR format**](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report?tabs=v2%2Cdesktop#pbir-format), where each report object (pages, visuals, bookmarks, etc.) is organized into its own **folders** and **JSON files**. |
    | `*.SemanticModel/definition`           | Contains the semantic model definition in [**TMDL file format**](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-dataset#tmdl-format), where each semantic model object (tables, roles, cultures, etc.) is organized into its own **folders** and **TMDL documents** using the [**TMDL language**](https://learn.microsoft.com/en-us/analysis-services/tmdl/tmdl-overview?view=sql-analysis-services-2025). |
    | `*.SemanticModel/.platform`           | Fabric platform file for the semantic model. Contains properties such as `displayName`, `description`, and `logicalId` (required for deployment and Fabric Git integration). |
    | `*.Report/.platform`                  | Same as the semantic model `.platform` file, but the `displayName` property is also used to define the **Power BI Desktop** window title. |
    | `*.SemanticModel/.pbi/cache.abf`      | A **local cached copy** of the semantic model's data stored as an Analysis Services Backup File (ABF). Acts as a **user-specific cache** and **should not be shared** via Git. **Power BI Desktop** can open the PBIP without it, but the model will be empty, requiring a data refresh. You can safely drop this file whenever you wish to share your development without compromising the data in it.|
1. Each JSON file includes a `$schema` property at the top of the document, which specifies the document's version. This property also helps clarify the meaning of other properties and can assist in detecting syntax errors when editing the files manually.

    ![pbip-schemas](resources/img/pbip-schemas.png)

    Learn more about PBIP JSON schemas in the [documentation](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview#json-file-schemas).

> [!TIP]
> * One of the most important concepts to understand when working with PBIP is that ALL changes made by the tool is saved into some readable text file. You can use that to your advantage, specially when dealing with long and repetivive tasks.
> * Simply being able to explore PBIP files using a file explorer or code editor is a significant advantage. For example, you no longer need to wait minutes for **Power BI Desktop** to open a PBIX just to review a measure's DAX code or to confirm which database your semantic model is connected to by inspecting a table's Power Query code - Just open the files and look at the code!
> * All files and folders are documented in [PBIP documentation](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview) including references to their public [JSON-schemas](https://github.com/microsoft/json-schemas/tree/main/fabric). 
  
## 3. Version Control your PBIP

✅ **Goal**: Set up Git, initialize a repository for your PBIP project, track changes, commit, view history, and restore to a previous version.

### Setup Git [skip if you already use Git]
One of the biggest advantages of PBIP is that its plain text files integrate seamlessly with Git — giving you full version history, team collaboration, and CI/CD support with quality gates and automated deployments. 

In this section, we'll set up source control to track and understand the impact of every change throughout the workshop, across both the semantic model and the report. Let's go!

### First time using Git in machine? (skip if you already use Git)

> [!IMPORTANT]
> * You only need to do this once!
> * It is important that these settings are in place before you start using git.

1. Open a new command prompt or terminal and run `git --version` to check that Git is installed:

    ![git version](resources/img/git-version.png)

2. All git commits are tagged with your name and email as an identifier. 
   
    Run the following commands:

    ```bash
    git config user.name
    git config user.email
    ```

    ![git config](resources/img/git-config.png)

    If no values are returned or you want to change them, run:

    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "your@email.com"
    ```

### Initialize local Git Repo for the PBIP folder

1. Open **Visual Studio Code** and go to **File > Open Folder...** to open the PBIP folder.
2. Navigate to the **Source Control** tab (`CTRL+SHIFT+G`) and click **Initialize Repository** button:

    ![vscode git init](resources/img/vscode-git-init.png)

    The Source Control panel now shows untracked changes across a number of files:

    ![vscode git pending changes](resources/img/vscode-git-pending-changes.png)

> [!IMPORTANT]
> Once you initialize a Git repository, a `.git/` folder is created you can view in file explorer. Do not edit, modify, or delete any files inside the `.git` directory. **Visual Studio Code** automatically hides the `.git/` folder for you.

### First commit

1. Provide a descriptive commit message (for instance, "Initial PBIP project") and click **Commit**. Confirm the dialog with **Yes** to stage and commit all changes.

    ![vscode commit all](resources/img/vscode-commit-all.png)

1. Once committed, the Source Control panel clears and shows no further pending changes. Also notice that the **Graph** tab shows your first commit.

    ![vscode clean workspace](resources/img/vsode-clean-workspace.png)

> [!TIP]
> It is a common mistake to forget the commit message. If that happens, **Visual Studio Code** will open a text file named `COMMIT_EDITMSG`. Simply close the COMMIT_EDITMSG tab and try again.
> 
> ![vscode git missing commit message](resources/img/vscode-git-missing-commit-message.png)


### Track your changes and Commit

1. Open `Sales.pbip` in **Power BI Desktop** (if not already opened).
1. Make a report change, for example move any visual to a different position
1. Make a semantic model change, for example edit a measure and add a DAX comment such as `/// Change in Desktop`
1. **Save** in **Power BI Desktop**
1. Return to **Visual Studio Code** and open the **Source Control** tab. You'll see all the updated **PBIR** and **TMDL** files listed there, reflecting the all the changes you made in **Power BI Desktop**.
    
    ![vscode-git-changes](resources/img/vscode-git-changes.png)

1. When you click on any of the files, a clear diff view opens so you can review exactly what was changed.

    ![vscode-git-changes-diff](resources/img/vscode-git-changes-diff.png)

1. Provide a meaningful commit message describing what you changed (e.g., "Updated report layout") and click **Commit**.

    You can also use GitHub Copilot [**Generate Commit Message**](https://code.visualstudio.com/docs/sourcecontrol/overview#_stage-and-commit-changes) feature and use AI to to look into what changed and generate a commit message for you.

    ![vscode-commit-changes](resources/img/vscode-commit-changes.png)

1. After commit, notice that a new entry appears in the **Graph** tab. When you click on it, you can see the commit changes made to each file.

    ![vscode-git-after-commit](resources/img/vscode-git-after-commit.png)

    This is very useful to understand and review changes you made in the past. That's why its an important best practice to always provide a meaningful message to commits - they can help you quickly navigate and understand the purpose of the change.
   
> [!TIP]
> It’s important to pause and reflect on this: using Git lets you see, in detail, exactly what Power BI changes in the underlying code files. Understanding this is key-because if you ever want to automate something in the future, you can simply perform the action in the UI and inspect the resulting changes to learn precisely what needs to be modified.

### Discard changes

1. Go back to **Power BI Desktop** and make some destructive changes, for example drop `Sales` page and drop table `Product`
1. **Save** in **Power BI Desktop**
1. Return to **Visual Studio Code** and open the **Source Control** tab. Notice all the code files that resulted from the destructive operation.
    
    ![vscode-changes-destructive](resources/img/vscode-changes-destructive-2.png)
    
    Normally if working against a PBIX if you make such a change and Save, it's not possible to recover. Not with PBIP, because when using Git you can always discard these changes and it will go back to how it was before the change and perform a full roll back of the entire project to a previous state.

2. Click the **Discard All Changes** button in the Source Control panel to revert all uncommitted changes:

    ![vscode discard changes](resources/img/vscode-discard-changes.png)

    Since this is a destructive action, **Visual Studio Code** requires an explicit confirmation:

    ![vscode discard confirm](resources/img/vscode-discard-confirm.png)

3. The destructive change you performed in the first step has now been undone - **but only in the PBIP code files, not in Power BI Desktop**. 
    
    **Power BI Desktop** does not automatically detect changes made through external tools or direct file edits. To see the updated state reflected in the application, you need to **restart Power BI Desktop**.
    
4. Go back to **Power BI Desktop** and close the application **without saving**. 
    
    If you choose to save, the destructive change will be written back to the code files, because that modification still exists in the current in-memory state of **Power BI Desktop**.

5. Re-Open `Sales.pbip` in **Power BI Desktop**
6. Notice that the destructive change has been reverted. If you deleted a semantic model table, its data is lost because Git does not track the `cache.abf` file. However, all of the table’s metadata is preserved, so you can always refresh the table to restore its data.

> [!IMPORTANT]
> With Git, you can much safely experiment new developments, knowing you can always recover your work if needed. It’s a best practice to commit often, with each commit representing a clear unit of work (e.g., creating or editing a measure). This approach not only helps you track changes but also makes it easy to roll back or recover specific developments.

### Restore to previous commit

1. One of the key benefits of using Git is the ability to switch between different versions of your development effortlessly - no need to maintain multiple PBIX files like `Report.pbix`, `Report-beforeNewMeasureX.pbix`, and so on. Using Git you can quickly navigate the commmit history and move to a previous state of your development.
2. Let's go back to the initial version of your project before any changes. In **Source Control Graph** right-click the first commit and select **Create Branch...**
   
    ![vscode-new-branch-graph](resources/img/vscode-new-branch-graph.png)

    Name your new branch something like "before changes":

    ![vscode-new-branch-name](resources/img/vscode-new-branch-name.png)

3. Notice that now your **Source Control Graph** don't include the changes you did before. Git branches work directly on the folder: when you switch branches it directly changes the folder files.
4. Close **Power BI Desktop** (without saving), then reopen `Sales.pbip` and observe that the Report and Semantic Model no longer include any of the previous changes.
    
    Git replaces the files in your folder with the version stored in that branch, letting you work on different versions without mixing them. 

5. Click the branch name in the bottom-left corner and select `main` to switch back to the `main` branch.

    ![vscode-git-switch-branch](resources/img/vscode-git-switch-branch.png)

    Observe that all your changes have been restored in the PBIP files.

> [!TIP]
> - Git is an essential tool for Power BI developers working with PBIP files. It provides a safety net for experimenting, a clear audit trail of all changes, and the ability to collaborate with teammates. 
> - Git branches let you work on different versions of your project at the same time, and you can easily switch between them without affecting each other. For example, create a branch each time you start a new development giving you the ability to work in isolation and always come back to the original version very esily while keeping your development going in the original branch. Learn more about branches in [Git Branches and Worktrees in VS Code](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees) or [Introduction to Git Branch](https://www.geeksforgeeks.org/git/introduction-to-git-branch/).

## 4. Publish Power BI Project to workspace

✅ **Goal**: Publish your PBIP project to a Fabric workspace using both **Power BI Desktop** and the **Microsoft Fabric VS Code extension**, and understand the differences between them.

### Publish from Desktop

1. Create a new workspace in Fabric, give it a name like `Workshop - Lab 1`.
2. In **Power BI Desktop**, open your `Sales.pbip` if not already open.
3. Publish the report to the workspace using **Power BI Desktop** **Publish** option.
   
   ![desktop publish](resources/img/desktopPublish.png)

4. The workspace should now have a new report and semantic model (with data).

### Publish from Fabric VS Code extension

1. Open **Visual Studio Code** and navigate to the **Microsoft Fabric extension**.
   
   ![microsoft fabric extension](resources/img/microsoftFabric-tab.png)

1. Filter the workshop workspace.
    
    ![microsoft fabric extension filter workspace](resources/img/microsoftFabric-filterworkspace.png)

7. Go to **File > Open Folder...** and open the PBIP folder.
8. In the **Microsoft Fabric extension**, expand the **Local folder** node and confirm that each Report (*.Report) and SemanticModel (*.SemanticModel) folder from your PBIP folder is listed in the tree-view.
   
   ![microsoft fabric extension local folder](resources/img/microsoftFabric-localfolder.png)

9. Right-click the `Sales` semantic model, select **Publish > To specific workspace...** and pick a Fabric workspace.
   
   ![microsoft fabric extension publish model](resources/img/microsoftFabric-publishmodel.png)
   
   ![microsoft fabric extension select workspace](resources/img/microsoftFabric-selectworkspace.png)

    Look for the operation status in the bottom right corner:

    ![microsoft fabric extension success](resources/img/microsoftFabric-success.png)

10. Confirm the semantic model got published to the workspace.
    
    Notice that if you explore the semantic model in the workspace, there is no data in it. 

11. Repeat the publish operation for the Report. 
    
    Unlike semantic models, Power BI reports must be connected to a semantic model. The **Fabric extension** will prompt you to select the target semantic model in the workspace that the report will connect to.

    ![microsoft fabric extension publish report](resources/img/microsoftFabric-publishreport.png)

### Key differences between the two publish methods

|                         | **Power BI Desktop Publish**                                                                                     | **Fabric VS Code Extension Publish**                                                                                         |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **What gets published** | Everything - definitions + data                                                                                  | **Only definitions (metadata)** - no data is pushed                                                                          |
| **Granularity**         | Publishes the report and semantic model together. You cannot publish individually.                                                               | Granular control - you choose to publish the report, the semantic model, or both independently                               |
| **API used**            | Traditional [PBIX publish API](https://learn.microsoft.com/en-us/rest/api/power-bi/imports/post-import-in-group) | [Fabric REST CRUD APIs](https://learn.microsoft.com/en-us/rest/api/fabric/core/items) |
| **Best for**            | Initial publish or when data needs to be pushed                                                                  | Iterative development - e.g. fixing a DAX measure without waiting for a full data refresh or publishing a new report page without affecting the semantic model                                  |

> [!IMPORTANT]
> PBIP publish using **Fabric extension** can significantly **boosts development efficiency**. For example, when making a simple update like fixing a DAX measure or updating a report page, you can publish only the code definition and skip the data entirely. Always publishing the semantic model data may require you to perform a data refresh to avoid impacting end users consuming the semantic model.

### Switch the semantic model of a PBIP report

All Power BI reports must connect to a semantic model. When using the PBIP file format, the semantic model that a report connects to is defined in the `definition.pbir` file. By default, the connection is relative (called `byPath`), but you can also configure a remote connection (called `byConnection`) to target a semantic model hosted in the service.

This exercise demonstrates how to configure a local Power BI report to connect either to a **local semantic model** running in **Power BI Desktop** or to a **remote semantic model** in a Fabric workspace by switching to **Live Connect** mode. 

> [!TIP]
> Using multiple `*.pbir` files is particularly useful for easy toggling between local and remote semantic models, providing flexibility for development, testing, and iterative report design. More details in [definition.pbir](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report?tabs=v2%2Cdesktop#definitionpbir).


1. Open the workspace and copy to an empty notepad the following attributes:
    * Workspace and semantic model name
  
        ![workspace and model name copy](resources/img/workspaceandmodelnames.png)
    
    * Semantic model ID
  
        ![semanticmodel id](resources/img/semanticmodel-id.png)

2.  Go back to **Visual Studio Code** and create a new file `definition-live.pbir` inside `Sales.Report/` folder with the following content:
   
    Replace the placeholders `[WorkspaceName]`, `[SemanticModelName]` and `[SemanticModelId]` with the attributes from the previous step.

    ```json
    {
        "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definitionProperties/2.0.0/schema.json",
        "version": "4.0",
        "datasetReference": {
            "byConnection": {      
                "connectionString": "Data Source=\"powerbi://api.powerbi.com/v1.0/myorg/[WorkspaceName]\";initial catalog=[SemanticModelName];access mode=readonly;integrated security=ClaimsToken;semanticmodelid=[SemanticModelId]"
            }
        }
    }
    ```

    The report folder should now include two `*.pbir` files side by side:

    ```text    
    Lab1/
    ├── Sales.Report/
    |   ├── definition/
    |   ├── definition-live.pbir
    |   └── definition.pbir
    ```
1. Open the `definition-live.pbir` with **Power BI Desktop** and notice that now the semantic model is not opened for edit but instead the report is **Live Connected** to the semantic model in the Fabric workspace.

    ![report live connect](resources/img/report-liveconnect.png)

> [!TIP]
> With this technique you can easily test a report against any semantic model running in the service. For example, test a report against a production semantic model with production data before publishing.

## 5. Semantic Modeling as Code with TMDL

✅ **Goal**: Edit a semantic model using TMDL, reuse shared TMDL components, and edit TMDL directly on a workspace using the [TMDL VS Code extension](https://marketplace.visualstudio.com/items?itemName=analysis-services.TMDL) and [Fabric VS Code extension](https://marketplace.visualstudio.com/items?itemName=fabric.vscode-fabric).

Tabular Model Definition Language (TMDL) is a human‑readable, code‑based language that represents the full structure of a Power BI semantic model as plain text, including tables, measures, relationships, and model properties. In this workshop section, we will take a hands‑on approach to semantic modeling as code by editing our model in Visual Studio Code using the TMDL VS Code extension, exploring how to view and modify TMDL scripts in TMDL View on the web, and editing TMDL directly in a Fabric workspace using the Fabric VS Code extension, covering end‑to‑end workflows from local development to in‑workspace editing.


### Reflect Power BI changes in PBIP Semantic Modeling files

1. Open the `Sales.pbip` file with **Power BI Desktop**.

2. From **Diagram View**, rearrange the position of the tables. This is a visual-only change, but it is still tracked as part of the semantic model.
   <img width="364" height="319" alt="Diagram Layout 1" src="https://github.com/user-attachments/assets/b4ccb8ed-ae58-4ea3-b9ca-f87e4d8ba644" />
   <img width="300" height="326" alt="Diagram Layour 2" src="https://github.com/user-attachments/assets/9bfc3280-42f1-401b-8a23-e5f4db0cdd34" />

   

4. Create a new calculated column in the **Customer** table.
    1. Select the **Customer** table.
    1. Click **New column**.
    1. Enter the following DAX expression:
       
```DAX
    Age Range Order =
    SWITCH(
        TRUE(),
        Customer[Age] < 25, 1,
        Customer[Age] < 35, 2,
        Customer[Age] < 45, 3,
        Customer[Age] < 55, 4,
        Customer[Age] < 65, 5,
        6
    )
```

<img width="957" height="503" alt="new calculated column" src="https://github.com/user-attachments/assets/1311474f-2c12-4853-9bff-bf348c463187" />

> [!TIP]
> Changes are only written to the PBIP files after you explicitly **save** them in Power BI Desktop. Until then, the PBIP files remain unchanged.



4. Save your changes in **Power BI Desktop**.

5. Open **Visual Studio Code** and go to source control. Notice how the changes you made in Power BI Desktop are now reflected in the PBIP files. In green you can see the lines that were added. In red you can see the lines that were deleted:

<img width="955" height="503" alt="Diagram layour changes" src="https://github.com/user-attachments/assets/46c0cccc-f8ed-47cd-8b9c-f0f70da8cecc" />

<img width="959" height="503" alt="calculated column changes" src="https://github.com/user-attachments/assets/62086739-74cf-428e-a11b-e73dd32646ad" />

6. Add a name for the commit, and commit the changes.



### Edit semantic model using TMDL

1. Open file `Sales.SemanticModel/definition/tables/Sales.tmdl`
1. Notice the TMDL code readability features provided by [TMDL extension](https://marketplace.visualstudio.com/items?itemName=analysis-services.TMDL):
   * **Code highlighting** - Easily identify different elements of TMDL code: types, values, expressions
    
        ![tmdl extension - code highlight](resources/img/tmdlextension-highlight.png)

   *  **Breadcrumb navigation** - Easily navigate between TMDL files and objects
        
        ![tmdl extension - breadcrumb](resources/img/tmdlextension-breadcrumb.png)

   * **Code formatting** - Keep your TMDL code clean and consistent
        
        ![tmdl extension - format](resources/img/tmdlextension-format.png)

1. Create two new measures by duplicating the code of the measure `Sales Qty`. Select all the lines of the `Sales Qty` measure and press **SHIFT + ALT + Down arrow**
 
    The TMDL code should look like the following:

     ```TMDL
    table Sales
        ...
        measure 'Sales Qty' = sum('Sales'[Quantity])
            formatString: #,##0
            lineageTag: c2ff8d96-2f03-4005-84df-91458625b73b
        measure 'Sales Qty' = sum('Sales'[Quantity])
            formatString: #,##0
            lineageTag: c2ff8d96-2f03-4005-84df-91458625b73b
        measure 'Sales Qty' = sum('Sales'[Quantity])
            formatString: #,##0
            lineageTag: c2ff8d96-2f03-4005-84df-91458625b73b
        ...
     ```
1. Notice that the TMDL extension highlights errors with red squiggles. You can also view a list of problems in the Problems pane - go to **View > Problems**.

    ![tmdl extension - problems](resources/img/tmdlextension-problems.png)
   
1. Rename the new measures to `Sales Qty (LY)` and `Sales Qty Avg per Day` and change the DAX expression accordingly.

    TMDL code should look like the following:

    ```TMDL
    table Sales
        ...
        measure 'Sales Qty' = sum('Sales'[Quantity])
            formatString: #,##0
            lineageTag: c2ff8d96-2f03-4005-84df-91458625b73b
        
        measure 'Sales Qty (LY)' = IF ([Sales Qty] > 0, CALCULATE([Sales Qty], SAMEPERIODLASTYEAR('Calendar'[Date])))
            formatString: #,##0
            lineageTag: c2ff8d96-2f03-4005-84df-91458625b73b
        
        measure 'Sales Qty Avg per Day' = AVERAGEX(VALUES('Calendar'[Date]), [Sales Qty])
            formatString: #,##0
            lineageTag: c2ff8d96-2f03-4005-84df-91458625b73b
        ...
     ```
    
    The TMDL extension will highlight an error when a duplicate lineage tag is detected. Lineage tags must be unique within their scope - for example, two measures in the same table cannot share the same lineage tag. Fix this by either entering a new unique identifier in the `lineageTag` property or using the TMDL extension's **code action** to generate a new lineage tag automatically.

    ![tmdl extension - new lineage tag](resources/img/tmdlextension-newlineagetag.png)

    Learn more about lineage tags in [lineage-tags-for-power-bi-semantic-models](https://learn.microsoft.com/en-us/analysis-services/tom/lineage-tags-for-power-bi-semantic-models).

2.  Configure the `displayFolder` property on the three Quantity measures. 
    
    You can edit multiple lines at same time, by using VS Code [multi cursor](https://code.visualstudio.com/docs/editing/codebasics#_multiple-selections-multicursor) feature. Press the **ALT** key and click at the end of line of the `lineageTag` property value of each measure.
    
    Notice the IntelliSense help while you type the property name:

    ![tmdl extension - intellisense](resources/img/tmdlextension-intellisense.png)

    TMDL code should look like the following (note: your lineage tags should be different):

    ```TMDL
    table Sales
        ...
        measure 'Sales Qty' = sum('Sales'[Quantity])
            formatString: #,##0
            lineageTag: c2ff8d96-2f03-4005-84df-91458625b73b
            displayFolder: Qty
        measure 'Sales Qty (LY)' = IF ([Sales Qty] > 0, CALCULATE([Sales Qty], SAMEPERIODLASTYEAR('Calendar'[Date])))
            formatString: #,##0
            lineageTag: 01992909-efd5-72c9-9746-d811874a1677
            displayFolder: Qty
        measure 'Sales Qty Avg per Day' = AVERAGEX(VALUES('Calendar'[Date]), [Sales Qty])
            formatString: #,##0
            lineageTag: 0199290c-54fe-747c-8ddf-c496eafd0d88
            displayFolder: Qty
        ...
     ```

3. Save the edited files in **Visual Studio Code** and open the `Sales.pbip` with **Power BI Desktop** to verify the newly created measures.

> [!TIP]
> While this example was simple, it demonstrates the potential of direct TMDL editing for large code changes or simple refactorings. You can take advantage of everything code editors offer, such as keyboard shortcuts, code duplication with copy & paste, advanced find-and-replace with regular expressions, programmatic edits, and AI autocompletion.

 

### Reuse semantic model objects using TMDL files

1. Close **Power BI Desktop**.
1. Copy the [`resources/tmdl/Time Intelligence.tmdl`](resources/tmdl/Time%20Intelligence.tmdl) TMDL file to the `Sales.SemanticModel\definition\tables` folder.
1. Open `Sales.pbip` in **Power BI Desktop** and verify that the semantic model now includes the new **Time Intelligence** calculation group table. A refresh may be required.
   
    ![time intelligence calc group](resources/img/timeintelligence-calcgroup.png)
   
> [!TIP]
> - The **TMDL folder structure makes it easy to reuse and collaborate while developing semantic models**. With TMDL you can maintain shared model components (e.g. calendar tables, calculation groups, roles,...) and quickly apply them to multiple semantic models by simply copy files between semantic model code repositories.

### Edit TMDL using Fabric Extension

1. Open **Visual Studio Code** and navigate to the **Microsoft Fabric extension**.
1. In the workspace tree-view, expand the semantic model that was previously published to the workspace.  
   
   ![microsoftFabric-expand-definition](resources/img/microsoftFabric-expand-definition.png)

1. Navigate to the **definition** node of the semantic model - you should see the TMDL files representing the semantic model code definition directly from the workspace.
   
    ![microsoftFabric-tmdl-definition](resources/img/microsoftFabric-tmdl-definition.png)

1. Open the `definition/tables/Sales.tmdl` file and click select to edit the file
   
    ![microsoftFabric-edit-definition](resources/img/microsoftFabric-edit-definition.png)

    You may need to enable the [**Edit Item Definitions**](vscode://settings/Fabric.EditItemDefinitions) setting in the extension: Open Settings (`Ctrl+,`) and search: `item definitions`. Select **allow editing of item definition files in Fabric Workspaces views**

1. Create a new measure using TMDL and **Save** the file. The **Fabric extension** will push the updated definition directly to the workspace.  

    Notice the notification banner in the bottom-right:

    ![microsoftFabric-save-definition-notification](resources/img/microsoftFabric-save-definition-notification.png)

1. Confirm the change got applied to the semantic model in the workspace

    You can open the semantic model in Fabric workspace from **Visual Studio Code** by right-click the item and select **Open in Fabric...**

    ![microsoftFabric-open-item](resources/img/microsoftFabric-open-item.png)

> [!TIP]
> Editing TMDL through the **Fabric extension** allows you to make quick, targeted changes to a semantic model in a workspace **without needing Power BI Desktop or a local PBIP copy**. This is especially useful for hotfixes or quick adjustments in development/test workspaces or a bulk find & replace operation.


### Edit TMDL Using TMDL View

TMDL View in Power BI Desktop allows you to view, edit, and apply changes to a semantic model directly using Tabular Model Definition Language (TMDL). Unlike editing TMDL files externally in VS Code—where changes require restarting Power BI Desktop to reload the model—TMDL View applies changes directly to the open model.

1. In Power BI Desktop, select the **TMDL View** icon located along the left side of the window.

   <img width="958" height="501" alt="TMDL View" src="https://github.com/user-attachments/assets/bf595ebe-f86b-451f-9ded-bc4c876fa4d2" />

2. Script any semantic model object (such as a table, measure, or column) by selecting objects from the **Data** pane and dragging them onto the code editor.

   <img width="958" height="502" alt="Script TMDL View" src="https://github.com/user-attachments/assets/74259f7a-b98a-44f6-ae36-2529aa6b0c60" />

   This generates a TMDL Script for the selected object.

   Alternatively, you can right-click an object in the Data view and select **Script TMDL to new tab** or **to the clipboard**:

   <img width="332" height="354" alt="Script to TMDL" src="https://github.com/user-attachments/assets/e58393cd-bd3b-4833-b6db-332729a115e8" />

3. Select a different semantic object (a table, measure, or column) and drag it into the editor. A new tab will be created with the TMDL Script for that object. If you save your changes in Power BI Desktop, your scripts will be saved as part of your PBIP under the **TMDLScripts** folder.

4. Now we'll create a new role using TMDL. Script your Roles by dragging **Roles** from the Data pane to your editor. Add a new role for **Australia** and click **Preview** to preview the changes to the semantic model as a TMDL code diff.
   ```TMDL
    role 'Store - Australia'
		    modelPermission: read
      tablePermission Store = [Country] == "Australia"
     ```

   <img width="959" height="503" alt="Role Australia Preview" src="https://github.com/user-attachments/assets/9372f187-b15c-4e88-b281-0b584aded482" />

   Without closing the preview, add a new role for **United Kingdom**. Select **Update preview** to refresh the preview after changes.
    ```TMDL
    role 'Store - United Kingdom'
		    modelPermission: read
      tablePermission Store = [Country] == "United Kingdom"
     ```

   <img width="958" height="502" alt="Preview Out Of Date" src="https://github.com/user-attachments/assets/b79ce403-b838-418b-b070-aba93e3be538" />

6. Click **Apply** to apply changes to your model.

7. Script all your measures by dragging **Measures** from the Data pane to the editor. Some measures use euros while others use dollars. We'll now standardize the currency format so all measures use the dollar format. Click the **Replace** option (`Ctrl+F`) from the ribbon to replace `€` with `$`. Preview your changes and apply them to the model.

   <img width="959" height="502" alt="Standardize currency" src="https://github.com/user-attachments/assets/f2e49027-572a-4506-a972-56ffdf6d2e40" />


















## 6. Report as Code with PBIR

✅ **Goal**: Map a report object to a PBIR file, identify changes, batch apply changes using a script, and edit PBIR directly on a workspace using the [Fabric extension](https://marketplace.visualstudio.com/items?itemName=fabric.vscode-fabric).

In this section, we'll focus on report as code with PBIR. We'll do a deep dive into the folders and subfolders that represent the Power BI report — specifically, the Sales.Report folder within our PBIP project.

To learn more: [Power BI Desktop project report folder](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report?tabs=v2%2Cdesktop)

### Understand PBIR

1. Open `Sales.pbip` with **VS Code** and explore the files and subfolders inside the `<project name>.Report` folder.

   ![PBIR folder structure in VS Code](resources/img/pbir-folder-vscode.png)

2. Open and inspect the **definition** folder. Inside, you'll find:

   - A **pages** folder containing one subfolder for each page in your report
   - Each page folder includes a **visuals** folder with a subfolder for every visual on that page
   - Inside each visual's subfolder, you'll find a `visual.json` file with the full definition of that visual

3. Open and inspect `pages.json`. This file contains the list of all pages in your report.

4. Open and inspect `report.json`. This file contains the report definition in Power BI Report Legacy format (PBIR-Legacy) and doesn't support external editing.

5. Open and inspect `.platform`. This is a Fabric platform file that holds properties vital for establishing and maintaining the connection between Fabric items and Git.

6. Open and inspect `definition.pbir`. This file contains the overall report definition and core settings, including the reference to the semantic model used by the report. Power BI Desktop can open a PBIR file directly, just like opening from a PBIP file. Opening a PBIR file also opens the semantic model if there's a relative reference using `byPath`.

### Reflect Power BI Changes in PBIP Report Files

1. Go to Power BI Desktop and make any change to a visual. For example, resize and reposition the KPI table on the KPI page. Save your changes.

   ![Resized KPI table in Power BI Desktop](resources/img/resize-kpi-table-pbi-desktop.png)

2. Go to VS Code and review the changes. You'll see the modifications in the `visual.json` file for the updated visual. Commit the changes.

   ![VS Code showing visual.json changes](resources/img/vscode-show-visual.png)

3. Go back to Power BI Desktop and create a new page with a new visual.

   ![New page with visual in Power BI Desktop](resources/img/new-page-visual-pbi-desktop.png)

4. Save your changes and return to VS Code to explore the modified files. Inspect the added and changed files.

   ![VS Code Source Control showing new and modified PBIR files](resources/img/vscode-source-control-modified-pbir.png)



### Edit Report using PBIR

Now that we understand the PBIR structure and have seen how changes made in Power BI Desktop are reflected in the PBIR files, it's time to learn how to edit our reports directly using PBIR.

1. Open `Sales.pbip` with **Power BI Desktop**.
1. Go to **File > Options and settings > Options > Report settings** and enable **Copy object names when right clicking on report objects**.
   
   ![settings - copy report object names](resources/img/settings-copypbirobjectname.png)

1. In the `Sales` page, hide the title from the first bar chart.
   
   ![pbirdemo-hidetitle](resources/img/pbirdemo-hidetitle.png)

1. Open the **More options** menu in the top-right corner and select **Copy object name**
   
   ![pbirdemo-copy object name](resources/img/pbirdemo-copyobjectname.png)

1. Save the PBIP.
1. Open **Visual Studio Code**
1. Go to **File > Open Folder...** and open the PBIP folder.
1. Right-click the `Sales.Report` folder, select **Find in folder** and paste the report object name.
   
   ![pbir-find report object menu](resources/img/pbirdemo-findobjectmenu.png)

   ![pbir-find report object](resources/img/pbirdemo-findobjectname.png)
   
   Using **Copy object name** can help you easily identify the location the report object (visual, page, filter, bookmark,...) in the PBIR folder.

1. Open the matched file for edit.
1. Mouse-hover the `height` property and notice that **Visual Studio Code** shows a tooltip that explains the property.

    ![pbir-schematooltip](resources/img/pbirdemo-schematooltip.png)

    This is because all PBIR JSON files include a JSON schema declaration at the top of the document with the `$schema` property. Additionally, it provides built-in IntelliSense and validation when editing with code editors like Visual Studio Code.

    If you misspell a property name, the code editor will flag the error and show an explanatory tooltip or entry in the Problems pane (`CTRL + SHIFT + M`):

    ![pbir-schemaerror](resources/img/pbirdemo-schemaerror.png)

    Learn more about PBIR schemas in [documentation](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report?tabs=v2%2Cdesktop#pbir-json-schemas).

1. Navigate to the JSON property `visual.visualContainerObjects.title` and notice that the property `show` is configured as `false`. 

    ![pbir-title hidden property](resources/img/pbir-titlehiddenproperty.png)

    This is the PBIR JSON configuration that determines that the visual title is hidden.

### Edit with script

Now that you know which property to modify in the PBIR JSON, let’s update it in bulk using a script.

1. Close **Power BI Desktop**
1. Create a new **Python** script file named `HideVisualTitles.py` side by side with the `Sales.pbip`.

    ```text
    Lab1/
    ├── Sales.Report/
    ├── Sales.SemanticModel/    
    ├── HideVisualTitles.py
    └── Sales.pbip
    ```

    Copy and paste the following code to `HideVisualTitles.py`
    
    ```python
    import json
    import os
    from pathlib import Path


    def main():
        current_path = Path(os.path.dirname(os.path.abspath(__file__)))
        report_path = current_path / "Sales.Report"

        visual_files = list(report_path.rglob("visual.json"))

        for file in visual_files:
            print(f"Processing visual in file: {file}")

            with open(file, "r", encoding="utf-8") as f:
                data = json.load(f)

            visual = data.get("visual", {})

            # Ensure the nested structure exists
            if "visualContainerObjects" not in visual:
                visual["visualContainerObjects"] = {}

            container_objects = visual["visualContainerObjects"]

            if "title" not in container_objects:
                container_objects["title"] = [{"properties": {}}]

            title = container_objects["title"]

            # title can be a list (array) - access the first element
            if isinstance(title, list):
                title_obj = title[0]
            else:
                title_obj = title

            if "properties" not in title_obj:
                title_obj["properties"] = {}

            properties = title_obj["properties"]

            if "show" not in properties:
                properties["show"] = {
                    "expr": {
                        "Literal": {
                            "Value": "false"
                        }
                    }
                }
            else:
                properties["show"]["expr"]["Literal"]["Value"] = "false"

            with open(file, "w", encoding="utf-8") as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
                f.write("\n")


    if __name__ == "__main__":
        main()

    ```
1. Click **Run** in the top-right to run the **Python** script

    ![vscode-python-run](resources/img/vscode-python-run.png)

    The following output is expected:

    ![vscode-python-script-run-output](resources/img/vscode-python-script-run-output.png)

1. Open the `Sales.pbip` with **Power BI Desktop** and notice that all visuals on all pages have their titles hidden.

    ![pbir-notitle-report](resources/img/pbir-notitle-report.png)

> [!TIP]
> This example is simple and educational - it could have been more efficient to hide titles using multi-select on each page. However, if you needed to apply the same change across 50 reports with an average of 10 pages each, automating it with a script and running it against all PBIR report files would be far more efficient. And remember that you can use AI to help you write the script code.

### Edit PBIR using Fabric Extension

1. Open **Visual Studio Code** and navigate to the **Microsoft Fabric extension**.
1. In the workspace tree-view, expand the report that was previously published to the workspace.
1. Navigate to the **definition** node of the report - you should see the PBIR JSON files representing the report's pages and visuals directly from the workspace.
   
   ![microsoftFabric-expand-definition-report](resources/img/microsoftFabric-expand-definition-report.png)

1. Open the visual.json of the report title, it should be `definition\pages\ReportSection89a9619c7025093ade1c\visuals\eb5c360e357e8b54eb88\visual.json`, enable file edit and update the title.
   
   ![microsoftFabric-edit-definition-report](resources/img/microsoftFabric-edit-definition-report.png)

    You may need to enable the [**Edit Item Definitions**](vscode://settings/Fabric.EditItemDefinitions) setting in the extension.

1.  Save the file. The **Fabric extension** will push the updated definition directly to the workspace.
1.  Open the report in the Fabric workspace to confirm the changes are reflected.
   
   ![microsoftFabric-report-after-edit](resources/img/microsoftFabric-report-after-edit.png)
   
> [!TIP]
> Editing PBIR through the **Fabric extension** allows you to make quick, targeted changes to a report in a workspace **without needing Power BI Desktop or a local PBIP copy**. Combined with TMDL editing, this means you can make end-to-end changes to both semantic models and reports directly in a workspace from **Visual Studio Code**.

## ✅ Wrap-up

You've now:

* Understood PBIP files and its folder structure.
* Set up Git source control, tracked changes, committed, and restored to a previous version.
* Published from both **Power BI Desktop** and **Fabric VS Code extension**, understanding the key differences.
* Connected a report to a remote semantic model using multiple `definition.pbir` files.
* Edited and reused TMDL files for semantic model development.
* Edited PBIR files, mapped report objects to PBIR files and batch-changed a report using a script.
* Edited TMDL and PBIR files directly on a workspace using the **Fabric extension**.

## Useful links

* [Power BI Project docs](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-overview)
* [TMDL language docs](https://learn.microsoft.com/en-us/analysis-services/tmdl/tmdl-overview?view=sql-analysis-services-2025)
* [Visual Studio Code basics](https://code.visualstudio.com/docs/introvideos/basics)
* [Git explained in 100s](https://www.youtube.com/watch?v=hwP7WQkmECE&t=9s)
* [Git Will Finally Make Sense After This](https://www.youtube.com/watch?si=h_hAniLBVfO05X7A&v=Ala6PHlYjmw&feature=youtu.be)
* [Introduction to Git in Visual Studio Code](https://code.visualstudio.com/docs/sourcecontrol/overview)
* [Git cheat-sheet](https://git-scm.com/cheat-sheet)
* [Microsoft Fabric extension for VS Code](https://marketplace.visualstudio.com/items?itemName=fabric.vscode-fabric)
