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

### Steps to create the Release Pipeline for the deployment on the WebApp same way you can do for the FuncionApp:
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

### We also need to select the appropriate runner in the Release pipeline.
---
1. Go to Release from the left-side panel.
2. Click on the release.
3. Click on Edit option > Edit PipeClick online
4. Click on Stage 1 Job, 1 Task
5. Select "Run On Agent"
6. Display name: Run On Agent
7. Agent Pool: Default
8. Parallelism: None
9. Timeout: 0
10. Job cancel timeout: 1
11. Artifact download: Select the appropriate one.
12. Run this job: Only when all previous jobs have succeeded.
---

### Now, it is time for running the build pipeline.
---
1. Go to pipelines
2. Select the build pipeline
3. Click on Run Pipeline
---

### Now, it is time for running the release pipeline.
---
1. Go to Releases
2. Select the release
3. On stages: Redeploy
4. Click on Logs (Besides Redeploy)
---

### The logs will show that the content is deployed on the webpage and it will also show you the URL.

### Below is the code for Release Pipeline for the deployment on the FuncitonApp same way you can do for the WebApp:

```
stages:
- stage: __default
  jobs:
  - job: Job
    pool:
      name: 'Default'
    steps:

    - task: DownloadBuildArtifacts@0
      displayName: 'Download Build Artifacts'
      inputs:
        buildType: 'specific'
        project: 'DotNet_Pipeline'
        pipeline: 'Build-faraz9993.netapp'
        buildVersionToDownload: 'latest'
        downloadPath: '$(Pipeline.Workspace)'
        artifactName: 'netapplication'

    - task: AzureFunctionApp@1
      displayName: 'Release Live discounts Func'
      inputs:
        azureSubscription: faraz-svc
        appType: functionApp
        appName: 'FarazFunctionApp'
        package: '$(Pipeline.Workspace)/netapplication/*.zip'
        deploymentMethod: 'runFromPackage'

    - task: AzureFunctionApp@2
      displayName: 'Release Func'
      inputs:
        connectedServiceNameARM: faraz-svc
        appType: functionApp
        appName: 'FarazFunctionApp'
        package: '$(Pipeline.Workspace)/netapplication/*.zip'
        deploymentMethod: 'runFromPackage'

    - task: AzureWebApp@1
      displayName: 'Deploy to Azure Web App'
      inputs:
        azureSubscription: faraz-svc
        appName: 'FarazWebApp'
        appType: webApp
        package: '$(Pipeline.Workspace)/netapplication/*.zip'
        deploymentMethod: 'auto'
```