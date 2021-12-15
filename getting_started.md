## 1. Setting up Azure Key Vault

This repo requires a number of secrets to create various artefacts in Azure, Azure DevOps and GitHub. In order to keep the secrets secure, an Azure Key Vault with the following secrets needs to be created:

| Secret Name            | Required Value           | Short description                                                                                                           |
| ------------------------ | ------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| AZURE-DEVOPS-EXT-GITHUB-PAT | [GitHub Personal Access token]       | GitHub [Personal Access Token](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token) of an account that has enough rights to create new repos in the GitHub org                                 |
| AZURE-RM-SERVICE-PRINCIPAL-ID-DEV                 | [Application (client) ID]                 | Client ID of the [Service Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#register-an-application). This service principal should have enough rights to create a new resource group and provision new Azure Services                           |
| AZURE-RM-SERVICE-PRINCIPAL-KEY-DEV           | [Client Secret]                  | [Client Secret](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-a-client-secret) of the service principal                                                                                                  |
| AZURE-RM-SUBSCRIPTION-ID-DEV           | [Subscription Id]              | Azure Subscription Id                                                                                                     |
| AZURE-RM-SUBSCRIPTION-NAME-DEV  | [Subscription name] | Azure Subscription name |
| AZURE-RM-TENANT-ID | [Subscription Tenand Id]  | [Tenant Id](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-how-to-find-tenant) of your Azure subscription               |

After adding all the settings, your Key Vault Secrets should look like this:
![image](https://user-images.githubusercontent.com/525867/135234077-af139012-55fc-4bc7-83a3-9aff2d727478.png)

NOTE: If your subscription name has a space in it, put quotes `""` around your subscription name for the AZURE-RM-SUBSCRIPTION-NAME-DEV secret. Else, the pipeline will fail.

## 2. GitHub Setup

### 2.1 GitHub Repos

Create the following repos in your GitHub organization via "Use this template" capability in GitHub (preferably use the same repo names):

1. [Bootstrap](https://github.com/ai-factory-azure/bootstrap)
2. [MLOps templates](https://github.com/ai-factory-azure/templates-mlops)
3. [Code template](https://github.com/ai-factory-azure/project-template-file)

In the `project-template-file` repo, replace the string `ai-factory-azure` with the name of your GitHub org (or username) in the following files:
* `controller/devops-pipelines/deploy-model-training-pipeline.yml`
* `controller/devops-pipelines/deploy-model-batch-scoring.yml`
* `controller/devops-pipelines/deploy-model-to-aks.yml`

### 2.1 GitHub Settings

Make the `project-template-file` repo a template repository by ticking the "Template Repository" option in the repos' settings:

![image](https://user-images.githubusercontent.com/26466075/132330317-528a2e71-7371-46c8-870e-10349a9fefcc.png)

## 3. Setting up Azure DevOps

You'll use Azure DevOps for running the multi-stage pipeline with build, model training, and scoring service release stages. If you don't already have an Azure DevOps organization, create one by following the instructions at [Quickstart: Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

If you already have an Azure DevOps organization, or once you've created an organisation, create a new project called `ai-factory-bootstrap` (or something similar) using the guide at [Create a project in Azure DevOps and TFS](https://docs.microsoft.com/en-us/azure/devops/organizations/projects/create-project?view=azure-devops).

## 4. Set up for the bootstrap Pipeline

Create a variable group named **`bootstrap-variables-kv`** (under `Pipelines` --> `Library`). The YAML pipeline definitions in this repository refer to this variable group by name.

The "Link secrets from an Azure Key Vault as variables" button should be set and the variable group should be linked to the Azure Key Vault created in step 1. Once it it has been linked, all the secrets created above should be added as variables in this variable group.

Go to the Azure DevOps organization's settings and grant "Create new projects" permission to the build service for the new project. The name of this account should be "<azure devops project name> Build Service"
  
![image](https://user-images.githubusercontent.com/26466075/132328747-02b0011f-aaa6-4a85-b51c-8e9feeed14d4.png)

![image](https://user-images.githubusercontent.com/26466075/132328799-55d9cbb4-520f-4890-9753-240fbae6a5d1.png)

## 5. Create a new Pipeline in Azure DevOps

Create a new pipeline in Azure DevOps as per the steps below:

1. Pipelines -> New Pipeline -> GitHub (YAML)
1. Choose the `bootstrap` repo you created in step 2 (If your repo is not visible in the default view, then click on the dropdown under Select a repository and choose "All repositories". If it is still not visible, then click on they hyperlink in the message "If you can't find a repository, make sure you provide access." at the bottom of the page and provide access to all repos your GitHub organization via this option)
1. Choose "Existing Azure Pipelines YAML File" option and then select the file `/azuredevops/pipelines/azure-devops-iac.yml`
1. Save the pipeline (do not run it yet)

## 6. Update parameters for the pipeline

Go to the file named `/azuredevops/pipelines/project-variables.yml` in the `bootstrap` repo in the GitHub update the variable values as per your requirements. This file defines how your first ML project should look like.

## 7. Pipeline Run

Once you commit the `project-variables.yml` file, it should run the pipeline we created in step 5. Please note that the first run of the pipeline will request for permission to access the Azure Subscription in order to access the Key Vault. 

Some notes:

* It is hard to get all the permissions right in the first go, so it'll be very surprising if the pipeline succeeds in the first go.
* If not, then please look for the errors and fix them. The pipeline will eventually run once everything is in place.
* There's another catch, even if the pipeline is successful, the steps that create the GitHub repos return an error in spite of them running successfully. This seems like a GitHub CLI issue and we haven't investigated enough on this. Having said that, the pipeline will run fine in spite of those errors. 
