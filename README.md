# azure-devops-ai

This repo contains code and pipeline definition for a machine learning project demonstrating how to automate the end to end ML/AI project. The build pipelines include DevOps tasks for data sanity test, unit test, model training on different compute targets, model version management, model evaluation/model selection, model deployment as realtime web service, staged deployment to QA/prod, integration testing and functional testing.

## Prerequisite
- Active Azure subscription
- Azure DevOps project
- Minimum contributor access to Azure subscription

## Getting Started:

### Update Pipeline Config:

#### Build Pipeline
1. In **Azure Pipeline**, create new **Build** pipeline from this Github repo
2. Choose to create new build pipeline from [azure-pipelines.yml](/azure-pipelines.yml)

#### Release Pipeline
1. In **Azure Pipeline**, create new **Release** pipeline
2. For release artifact, choose latest artifact from the build pipeline
3. Add stage `QA - Deploy on ACI` with the following configurations

```
pool:
  vmImage: 'ubuntu-latest'

variables:
  azureSubscriptionName: '<>'
  azureSubscriptionId: '<>'

steps:
- task: UsePythonVersion@0
  displayName: 'Use Python 3.6'
  inputs:
    versionSpec: 3.6

- task: Bash@3
  displayName: 'Install Requirements'
  inputs:
    targetType: filePath
    filePath: './$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai/environment_setup/install_requirements.sh'
    workingDirectory: '$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai/environment_setup'

- task: AzureCLI@1
  displayName: '50. Deploy Webservice on ACI'
  inputs:
    azureSubscription: '$(azureSubscriptionName) ($(azureSubscriptionId))'
    scriptLocation: inlineScript
    inlineScript: 'python ./aml_service/50-deployOnAci.py'
    workingDirectory: '$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai'

- task: AzureCLI@1
  displayName: '60. Test ACI Webservice'
  inputs:
    azureSubscription: '$(azureSubscriptionName) ($(azureSubscriptionId))'
    scriptLocation: inlineScript
    inlineScript: 'python ./aml_service/60-AciWebserviceTest.py'
    workingDirectory: '$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai'
```

4. Add stage `Prod - Deploy on AKS` with the following configurations

```
pool:
  vmImage: 'ubuntu-latest'

variables:
  azureSubscriptionName: '<>'
  azureSubscriptionId: '<>'

steps:
- task: UsePythonVersion@0
  displayName: 'Use Python 3.6'
  inputs:
    versionSpec: 3.6

- task: Bash@3
  displayName: 'Install Requirements'
  inputs:
    targetType: filePath
    filePath: './$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai/environment_setup/install_requirements.sh'
    workingDirectory: '$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai/environment_setup'

- task: AzureCLI@1
  displayName: '51. Deploy on AKS'
  inputs:
    azureSubscription: '$(azureSubscriptionName) ($(azureSubscriptionId))'
    scriptLocation: inlineScript
    inlineScript: 'python ./aml_service/51-deployOnAks.py'
    workingDirectory: '$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai'

- task: AzureCLI@1
  displayName: '61. Test AKS Webservice'
  inputs:
    azureSubscription: '$(azureSubscriptionName) ($(azureSubscriptionId))'
    scriptLocation: inlineScript
    inlineScript: 'python ./aml_service/61-AksWebserviceTest.py'
    workingDirectory: '$(System.DefaultWorkingDirectory)/_devops-for-ai-CI/devops-for-ai'
```

### Update Repo config:
1. Go to the **Repos** on the newly created Azure DevOps project
2. Open the config file [/aml_config/config.json](/aml_config/config.json) and edit it
3. Put your Azure subscription ID in place of <>
4. Change resource group and AML workspace name if you want
5. Put the location where you want to deploy your Azure ML service workspace
6. Save the changes and commit these changes to master branch
7. The commit will trigger the build pipeline to run deploying AML end to end solution
8. Go to **Pipelines -> Builds** (on Azure DevOps) to see the pipeline run

## Steps Performed in the Build Pipeline:

1. Prepare the python environment
2. Get or Create the workspace
3. Submit Training job on the remote DSVM / Local Python Env
4. Register model to workspace
5. Create Docker Image for Scoring Webservice
6. Copy and Publish the Artifacts to Release Pipeline

## Steps Performed in the Release Pipeline
In Release pipeline we deploy the image created from the build pipeline to Azure Container Instance and Azure Kubernetes Services

### Deploy on ACI - QA Stage
1. Prepare the python environment
2. Create ACI and Deploy webservice image created in Build Pipeline
3. Test the scoring image

### Deploy on AKS - PreProd/Prod Stage
1. Prepare the python environment
2. Deploy on AKS
    - Create AKS and create a new webservice on AKS with the scoring docker image

        OR

    - Get the existing AKS and update the webservice with new image created in Build Pipeline
3. Test the scoring image

### Repo Details

You can find the details of the code ans scripts in the repository [here](/docs/code_description.md)