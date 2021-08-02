# Use an ARM template to deploy a Web app to Azure

Get started with Azure Resource Manager templates (ARM templates) by deploying a Web app with SQL database. ARM templates give you a way to save your configuration in code. Using an ARM template is an example of infrastructure as code and a good DevOps practice.

An ARM template is a JavaScript Object Notation (JSON) file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment.

## Basic Template Structure:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "functions": [],
  "variables": {},
  "resources": [],
  "outputs": {}
}
```

## SQL Database Example ARM Template:

```json
{
  "$schema": "https://schema.management.  azure.com/schemas/2019-04-01/ deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "serverName": {
      "type": "string",
      "defaultValue": "[uniqueString  ('sql', resourceGroup().id)]",
      "metadata": {
        "description": "The name of the   SQL logical server."
      }
    },
    "sqlDBName": {
      "type": "string",
      "defaultValue": "SampleDB",
      "metadata": {
        "description": "The name of the   SQL Database."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().  location]",
      "metadata": {
        "description": "Location for all  resources."
      }
    },
    "administratorLogin": {
      "type": "string",
      "metadata": {
        "description": "The administrator   username of the SQL logical   server."
      }
    },
    "administratorLoginPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The administrator   password of the SQL logical   server."
      }
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2020-02-02-preview",
      "name": "[parameters('serverName')] ",
      "location": "[parameters('location')  ]",
      "properties": {
        "administratorLogin": "[parameters  ('administratorLogin')]",
        "administratorLoginPassword": " [parameters  ('administratorLoginPassword')]"
      },
      "resources": [
        {
          "type": "databases",
          "apiVersion": "2020-08-01-preview",
          "name": "[parameters  ('sqlDBName')]",
          "location": "[parameters  ('location')]",
          "sku": {
            "name": "Standard",
            "tier": "Standard"
          },
          "dependsOn": [
            "[resourceId('Microsoft.Sql/  servers', concat(parameters ('serverName')))]"
          ]
        }
      ]
    }
  ]
}
```

## Azure Resource Manager Layer

![armmanagementlayer](./Images/ARMmanagementlayer.png)

DevOps Use the REST APIs to create resources lets take a quick look at the REST Endpoints for [Create or Update a Group Scope](https://docs.microsoft.com/en-us/rest/api/resources/deployments/create-or-update-at-management-group-scope).

## Terminology

If you're new to Azure Resource Manager, there are some terms you might not be familiar with.

- **resource** - A manageable item that is available through Azure. Virtual machines, storage accounts, web apps, databases, and virtual networks are examples of resources. Resource groups, subscriptions, management groups, and tags are also examples of resources.
- **resource group** - A container that holds related resources for an Azure solution. The resource group includes those resources that you want to manage as a group. You decide which resources belong in a resource group based on what makes the most sense for your organization. [See Resource groups](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview#resource-groups).
- **resource provider** - A service that supplies Azure resources. For example, a common resource provider is Microsoft.Compute, which supplies the virtual machine resource. Microsoft.Storage is another common resource provider. See [Resource providers and types](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types).
- **Resource Manager template** - A JavaScript Object Notation (JSON) file that defines one or more resources to deploy to a resource group, subscription, management group, or tenant. The template can be used to deploy the resources consistently and repeatedly. See [Template deployment overview](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview).
- **declarative syntax** - Syntax that lets you state "Here is what I intend to create" without having to write the sequence of programming commands to create it. The Resource Manager template is an example of declarative syntax. In the file, you define the properties for the infrastructure to deploy to Azure. See [Template deployment overview](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview).

ARM Templates docs this is good place to understand templates.

Prerequisites
Before you begin, you need:

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- An active Azure DevOps organization. [Sign up for Azure Pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started/pipelines-sign-up?view=azure-devops).
  Get the code

- Create a resource connection in Azure DevOps Project.

> Note: Nice to have tool `Azure CLI` installed on your local dev environment [Azure CLI download and install](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

- Get your subscription Id use `az account show` or navigate to the Subscriptions in the Azure Portal.

## For this repository on GitHub:

From Git Hub fork the below git hub repository.

> If you need guidance on forking a repo [click here](https://docs.github.com/en/get-started/quickstart/fork-a-repo).

    ```
        https://github.com/Azure/   azure-quickstart-templates  /tree/master/quickstarts/

    ```

# Create your pipeline and deploy your template

1. Sign in to your Azure DevOps organization and navigate to your project. Create a project if you do not already have one.

2. Go to Pipelines, and then select Create Pipeline.

3. Select GitHub as the location of your source code.

> Note
> You may be redirected to GitHub to sign in. If so, enter your GitHub credentials.

4. When the list of repositories appears, select yourname/azure-quickstart-templates/.

> Note
> You may be redirected to GitHub to install the Azure Pipelines app. If so, select Approve and install.

5. When the Configure tab appears, select Starter pipeline.

6. Replace the content of your pipeline with this code:

   ```yml
   trigger:
     - none

   pool:
   vmImage: 'ubuntu-latest'
   ```

7. Create three variables: `skuName`, `skuCapacity`, `location` and `sqlAdministratorLogin` . `administratorLoginPassword` needs to be a secret variable.

   - Select Variables.
   - Use the + sign to add three variables. When you create `administratorLoginPassword`, select Keep this value secret.
   - Click Save when you're done.

     | Variable                      | Value      | Secret? |
     | ----------------------------- | ---------- | ------- |
     | skuName                       | S1         | No      |
     | skuCapacity                   | 1          | No      |
     | location                      | East US    | No      |
     | sqlAdministratorLogin         | admin      | No      |
     | sqlAdministratorLoginPassword | Fqdn:5362! | Yes     |

8. Map the secret variable `$(adminPass)` so that it is available in your Azure Resource Group Deployment task. At the top of your YAML file, map `$(adminPass)` to `$(ARM_PASS)`.

   ```yml
   variables:
     ARM_PASS: $(adminPass)

   trigger:
     - none

   pool:
     vmImage: 'ubuntu-latest'
   ```

9. Add the Copy Files task to the YAML file. You will use the web-app-sql-database project. For more information, see [Provision a web app with a SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.web/web-app-sql-database) repo for more details.

   ```yml
   variables:
   ARM_PASS: $(adminPass)

   trigger:
     - none

   pool:
   vmImage: 'ubuntu-latest'

   steps:
     - task: CopyFiles@2
       inputs:
       SourceFolder: 'quickstarts/microsoft.web/web-app-sql-database'
       Contents: '**'
       TargetFolder: '$(Build.ArtifactStagingDirectory)'
   ```

10. Add and configure the Azure Resource Group Deployment task.

    The task references both the artifact you built with the Copy Files task and your pipeline variables. Set these values when configuring your task.

- **Deployment scope (deploymentScope)**: Set the deployment scope to Resource Group. You can target your deployment to a management group, an Azure subscription, or a resource group.
- **Azure Resource Manager connection (azureResourceManagerConnection)**: Select your Azure Resource Manager service connection. To configure new service connection, select the Azure subscription from the list and click Authorize. See Connect to Microsoft Azure for more details
- **Subscription (subscriptionId)**: Select the subscription where the deployment should go.
- **Action (action)**: Set to Create or update resource group to create a new resource group or to update an existing one.
  Resource group: Set toARMPipelinesLAMP-rg to name your new resource group. If this is an existing resource group, it will be updated.
- **Location(location)**: Location for deploying the resource group. Set to your closest location (for example, West US). If the resource group already exists in your subscription, this value will be ignored.
- **Template location (templateLocation)**: Set to Linked artifact. This is location of your template and the parameters files.
- **Template (cmsFile)**: Set to $(Build.ArtifactStagingDirectory)/azuredeploy.json. This is the path to the ARM template.
- **Template parameters (cmsParametersFile)**: Set to $(Build.ArtifactStagingDirectory)/azuredeploy.parameters.json. This is the path to the parameters file for your ARM template.
- **Override** template parameters (overrideParameters): Set to -siteName $(skuName) -administratorLogin $(adminUser) -administratorLoginPassword $(administratorLoginPassword) to use the variables you created earlier. These values will replace the parameters set in your template parameters file.
- **Deployment mode (deploymentMode)**: The way resources should be deployed. Set to **Incremental**. **Incremental** keeps resources that are not in the ARM template and is faster than **Complete**. Validate mode lets you find problems with the template before deploying.

  ```yml
  variables:
    ARM_PASS: $(adminPass)

  trigger:
    - master

  pool:
    vmImage: ubuntu-latest

  steps:
    - task: CopyFiles@2
      inputs:
        SourceFolder: 'quickstarts/microsoft.web/web-app-sql-database'
        Contents: '**'
        TargetFolder: '$(Build.ArtifactStagingDirectory)'

    - task: AzureResourceManagerTemplateDeployment@3
      inputs:
        deploymentScope: 'Resource Group'
        azureResourceManagerConnection: '<your-resource-manager-connection>'
        subscriptionId: '<your-subscription-Id>'
        action: 'Create Or Update Resource Group'
        resourceGroupName: 'ARMPipelinesLAMP-rg'
        location: '<your-closest-location>'
        templateLocation: 'Linked artifact'
        csmFile: '$(Build.ArtifactStagingDirectory)/azuredeploy.json'
        csmParametersFile: '$(Build.ArtifactStagingDirectory)/azuredeploy.parameters.json'
        overrideParameters: '-skuName $(skuName) -skuCapacity $(skuCapacity) -sqlAdministratorLogin  $(administratorLogin) -sqlAdministratorLoginPassword  $(ARM_PASS)'
        deploymentMode: 'Incremental'
  ```

11. Click Save and run to deploy your template. The pipeline job will be launched and after few minutes, depending on your agent, the job status should indicate Success.

Ensure all indention is correct or file will fail to parse.

## Clean Up

You can now clean up the resource you created in the lab by running the `az group delete` command.

```CLI
  az group delete --resource-group ARMPipelinesLAMP-rg

```

# More Resources

## Learn

A good resource for learning more and getting some good practice in is Microsoft Learn.

### Deploy and manage resources in Azure by using JSON ARM templates

[Learn to Deploy ARM Templates learning path](https://docs.microsoft.com/en-us/learn/paths/deploy-manage-resource-manager-templates/)

## Arm Template Documentation

[ARM Template documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/)
