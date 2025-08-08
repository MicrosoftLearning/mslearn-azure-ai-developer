---
lab:
    topic: Azure container services
    title: 'Deploy a container to Azure Container Apps with the Azure CLI'
    description: 'Learn how to use Azure CLI commands to create a secure Azure Container Apps environment, and deploy a container.'
---

# Deploy a container to Azure Container Apps with the Azure CLI

In this exercise, you deploy a containerized application to Azure Container Apps using Azure CLI. You learn how to create a container app environment, deploy your container, and verify that your application is running in Azure.

Tasks performed in this exercise:

* Create resources in Azure
* Create an Azure Container Apps environment
* Deploy a container app to the environment

This exercise takes approximately **15** minutes to complete.

## Create a resource group and prepare the Azure environment

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. Create a resource group for the resources needed for this exercise. Replace **myResourceGroup** with a name you want to use for the resource group. You can replace **eastus** with a region near you if needed. If you already have a resource group you want to use, proceed to the next step.

    ```azurecli
    az group create --location eastus --name myResourceGroup
    ```

1. Run the following command to ensure you have the latest version of the Azure Container Apps extension for the CLI is installed.

    ```azurecli
    az extension add --name containerapp --upgrade
    ```

### Register namespaces

There are two namespaces that need to be registered for Azure Container Apps, and you ensure they are registered in the following steps. Each registration can take a few minutes to complete if they aren't already configured in your subscription. 

1. Register the **Microsoft.App** namespace. 

    ```bash
    az provider register --namespace Microsoft.App
    ```

1. Register the **Microsoft.OperationalInsights** provider for the Azure Monitor Log Analytics workspace if you haven't used it before.

    ```bash
    az provider register --namespace Microsoft.OperationalInsights
    ```

## Create an Azure Container Apps environment

An environment in Azure Container Apps creates a secure boundary around a group of container apps. Container Apps deployed to the same environment are deployed in the same virtual network and write logs to the same Log Analytics workspace.

1. Create an environment with the **az containerapp env create** command. Replace **myResourceGroup** and **myLocation** with the values you used earlier. It takes a few minutes for the operation to complete.

    ```bash
    az containerapp env create \
        --name my-container-env \
        --resource-group myResourceGroup \
        --location myLocation
    ```

## Deploy a container app to the environment

After the container app environment finishes deploying, you can deploy a container image to your environment.

1. Deploy a sample app container image with the **containerapp create** command. Replace **myResourceGroup** with the value you used earlier.

    ```bash
    az containerapp create \
        --name my-container-app \
        --resource-group myResourceGroup \
        --environment my-container-env \
        --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
        --target-port 80 \
        --ingress 'external' \
        --query properties.configuration.ingress.fqdn
    ```

    By setting **--ingress** to **external**, you make the container app available to public requests. The command returns a link to access your app.

    ```
    Container app created. Access your app at <url>
    ```

To verify the deployment, select the URL returned by the **az containerapp create** command to verify the container app is running.

## Clean up resources

Now that you finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

1. Navigate to the resource group you created and view the contents of the resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.

> **CAUTION:** Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.
