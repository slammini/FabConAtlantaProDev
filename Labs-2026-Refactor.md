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
  - Edit semantic model using TMDL (reuse and adapt)
  - Reuse semantic model objects using TMDL files (reuse and adapt)
  - Edit TMDL using Fabric Extension (new)
  - Edit Model using TMDL view
- Report as Code with PBIR
  - Edit Report using PBIR (reuse and adapt)
  - Edit with script
  - Edit PBIR using Fabric Extension (new)

## Lab 2 - CI/CD 

- Adapt the [projects-deploy-fabric-cicd](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-deploy-fabric-cicd) to a lab
- Include the following exercises:
  - Run fabric-cicd locally
  - Show some configurations such as deploy only reports or semantic models
  - Call out parametrization
  - Co-Development explanation with branching
  - Configure and run fabric-cicd in GitHub as automation
    - Setup service principal
    - Show Branch > PR > Deploy flow
  
## Lab 3 - Power BI & AI

- Edit TMDL and PBIR files with AI
  - Edit TMDL with Auto Comple
  - Set descriptions on TMDL files: [demo](https://www.linkedin.com/posts/ruiromano_worksmart-tmdl-tmdlview-activity-7295798659290423297-jswy?utm_source=share&utm_medium=member_desktop&rcm=ACoAAALWDywB9c6Gn0_KgodALqsO-wFYG9PvaOk)
    - Demonstrate how to configure skills and how they benefit the experience
  - Align report visuals with PBIR: [demo](https://www.linkedin.com/feed/update/urn:li:activity:7407397642026139650?updateEntityUrn=urn%3Ali%3Afs_feedUpdate%3A%28V2%2Curn%3Ali%3Aactivity%3A7407397642026139650%29&lipi=urn%3Ali%3Apage%3Ad_flagship3_profile_view_base%3B9VNdtwF5TIWZ0oHdcNcd5w%3D%3D)
  - Build a wireframe from a PBIR report for analysis
- Modeling with Power BI Modeling MCP
  - Set descriptions
  - Troubleshoot measure performance with a loop: analyze DAX > fix DAX > ensure data is the same > ensure performance is real with trace
  - Refactor model following team best practices
  - Create new model from sample data




