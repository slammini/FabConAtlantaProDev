## Lab 1 - PBIP Fundamentals

- Save a PBIX to PBIP Format - OK
- Explore PBIP files and folders - OK
- Git Basics & Source Control - OK  
- Publish Power BI Project to workspace
  - From Desktop - OK
  - From Fabric VS Code extension - OK
  - Call out differences, first publish everything and second provides granular control on what gets published but no data is pushed
  - Connect a report to a semantic model in service by creating a new definition-live.pbir and keep it side by side with bypath definition.pbir
- Semantic Modeling as Code with TMDL
  - Edit semantic model using TMDL - OK
  - Reuse semantic model objects using TMDL files - OK
  - Edit TMDL using Fabric Extension - OK
  - Edit Model using TMDL view - NOK
- Report as Code with PBIR
  - Edit Report using PBIR - OK
  - Edit with script - OK
  - Edit PBIR using Fabric Extension - OK

## Lab 2 - Power BI and CI/CD

- ~Adapt the [projects-deploy-fabric-cicd](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-deploy-fabric-cicd) to a lab
  - Specific repo for the E2E demo: https://github.com/RuiRomano/workshops-cicd-demo
- Exercises:
  - Setup fabric-cicd locally 
    - Install python package (Lab 1 already ensures python installation)
  - Prepare repo for CICD from Github
    - Create a new repo in GitHub
    - Clone
  - Save the PBIX as PBIP into the src/ folder of the cloned repo    
  - Create the `/scripts/deploy.py` targeting the src/ folder
  - Run fabric-cicd locally - Goal is to show deployment using code
    - Show some configurations such as deploy only reports or semantic models
    - Call out parametrization
    - Deploy to a different workspace
  - Deploy from GitHub
    - Prep metadata files and scripts
      - Configure `.github/workflows`; 
      - Configure `scripts/.bpa/`
    - Commit and sync
    - Configure SPN in GitHub: Use approach from docs with Azure Identity env variables
      - Prepare list for users
    - Simulate development flow: New BRanch; Change some TMDL / Report > PR > Validate > Deployment to workspace      
      
## Lab 3 - Power BI Development with AI

- Prepare AI environment with VS Code
  - .github/skills; Install MCP Extension
  - Download skills from repo: https://github.com/RuiRomano/powerbi-agentic-plugins
- Copilot against code files directly
  - Editor Auto Complete when creating a new measure. 
  - Align report visuals using PBIR skill: [demo](https://www.linkedin.com/feed/update/urn:li:activity:7407397642026139650?updateEntityUrn=urn%3Ali%3Afs_feedUpdate%3A%28V2%2Curn%3Ali%3Aactivity%3A7407397642026139650%29&lipi=urn%3Ali%3Apage%3Ad_flagship3_profile_view_base%3B9VNdtwF5TIWZ0oHdcNcd5w%3D%3D)
  - Build a repor wireframe for documentation purposes?
- Modeling with Copilot and Modeling MCP
  - Pick one of the simple demos from the demo video: https://www.linkedin.com/feed/update/urn:li:activity:7396586403863887874/
  - Agentic development by enforcing team rules in a markdown against a semantic model. Run BPA analysis and Fix the errors autonomously
  - Troubleshoot measure performance with a loop: analyze DAX > fix DAX > ensure data is the same > ensure performance is real with trace  
- E2E agentic development with GitHub Copilot CLI (test with basic models)
  - Using agentic plugins: https://github.com/RuiRomano/powerbi-agentic-plugins  
  - Prompt to create a new semantic model and report from scratch with a prompt like:
      ```
      Using powerbi-semantic-model and powerbi-pbir skills.
      Create a new semantic model based on the CSV files located in `https://github.com/RuiRomano/powerbi-agentic-plugins/tree/main/assets/sample-data` use a http connector against the HTTP files. Apply standard modeling best practices throughout (e.g., proper relationships, naming conventions, data types, and star schema design). 
      In the end, create a sample report on top of the semantic model.
      ```




