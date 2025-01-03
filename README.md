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


### What we need to do here is we will have our .NET code that will be build and published in the build pipeline and that pipeline will create an artifact.


----
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
----


### Steps to create the Release Pipeline:
---
1. Go to release section
2. Click on Create Release
3. 
---