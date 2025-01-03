# Deployment of .NET application on Azure Function app and Web app.

### We have our code pushed in GitHub repo. The code is fetched from GitHub repo to the Azure DevOps CI/CD pipeline and deployed on Azure resources such as WebApp and FunctionApp.

### We need to create resources such as WebApp and FuunctionApp on Microsoft Azure.

### We need to have several files of the .NET code to be pushed in the GitHub repo. However. before pushing the code to the repo make sure .NET application is running successfully in local.

----
1. Program.cs
2. Startup.cs
3. HelloWorld.csproj
4. Azure_Devops_.NET.sln
----

### Once, you will have running code available in the repo, it is time to create FunctionApp or WebApp as per the need. 

### Next step is to create Azure CI/CD pipeline. We will have two pipelines:
----
1. Publish
2. Release
----

### However, before that we need to have a runner. We will create a self-hosted runner. Below is the link of the video that will help to create the self-hosted runner either in the Local Machine or the VM.

```
https://youtu.be/O-HQmWniyyY?si=ZIOb4PiHuyj2qBMZ
```

### Steps to create the Publish Pipeline:
---
1. Login to the Azure DevOps
2. Go to pipelines
3. Click on "New Pipeline"
4. Select GitHub
5. Select the appropriate repo
6. Select "Existing Azure Pipelines YAML file"
7. Select "Starter Pipeline"
8. Save and Run
---

### You can rename the pipeline by clicking on the tree dots in the pipeline section.


### What we need to do here is we will have our .NET code that will get build and published in the build pipeline and that pipeline will create an artifact. The same artifact will then be used by another Release pipeline. Below is our code for the build pipeline. 


```
trigger:
  branches:
    include:
    - main
stages:
- stage: __default
  jobs:
  - job: Job
    pool:
      name: 'default'
    steps:
    - task: CmdLine@2
      displayName: 'Restore dependencies'
      inputs:
        script: |
          dotnet restore HelloWorldApp.csproj
    - task: CmdLine@2
      displayName: 'Build project'
      inputs:
        script: |
          dotnet build HelloWorldApp.csproj --configuration Release
    - task: CmdLine@2
      displayName: 'Publish project'
      inputs:
        script: |
          dotnet publish HelloWorldApp.csproj --configuration Release --output $(Build.ArtifactStagingDirectory)
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: $(Build.ArtifactStagingDirectory)
        artifactName: 'drop'
        publishLocation: 'Container'
```


### Steps to create the Release Pipeline:
---
1. Go to Release section
2. Click on Create Release
3. Stages for a trigger change from automated to manual: Stage 1
4. Select the appropriate Artifact that is being created form build pipeline.
5. Create
6. Now, click on the Release.
7. Click on Edit option > Edit PipeClick online
8. Click on Stage 1 Job, 1 Task
9. Deploy Azure App Service
10. Display name: Deploy Azure App Service
11. Connection Type: Azure Resource Manager
12. Azure Subscription: faraz-svc
13. App Service Type: Web App on Linux
14. App Service Name: FarazWebApp
15. Package or Folder: "$(System.DefaultWorkingDirectory)/_New_Build.Azure_Devops_.NET/drop"
16. Startup command: dotnet HelloWorldApp.dll 
---