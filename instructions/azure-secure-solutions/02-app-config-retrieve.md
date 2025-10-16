---
lab:
    topic: Secure solutions in Azure
    title: "Retrieve configuration settings from Azure App Configuration"
    description: "Learn how to create an Azure App Configuration resource, and set configuration information with Azure CLI. Then, use the **ConfigurationBuilder** to retrieve settings for you application."
---

# Retrieve configuration settings from Azure App Configuration

In this exercise, you create an Azure App Configuration resource, store configuration settings using Azure CLI, and build a .NET console application that uses the **ConfigurationBuilder** to retrieve configuration values. You learn how to organize settings with hierarchical keys and authenticate your application to access cloud-based configuration data.

Tasks performed in this exercise:

* Create an Azure App Configuration resource
* Store connection string configuration information
* Create a .NET console app to retrieve the configuration information
* Clean up resources

This exercise takes approximately **15** minutes to complete.

## Create an Azure App Configuration resource and add configuration information

In this section of the exercise you create the needed resources in Azure with the Azure CLI.

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal. If you are prompted to select a storage account to persist your files, select **No storage account required**, your subscription, and then select **Apply**.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

1. Create a resource group for the resources needed for this exercise. If you already have a resource group you want to use, proceed to the next step. Replace **myResourceGroup** with a name you want to use for the resource group. You can replace **eastus** with a region near you if needed.

    ```
    az group create --name myResourceGroup --location eastus
    ```

1. Many of the commands require unique names and use the same parameters. Creating some variables will reduce the changes needed to the commands that create resources. Run the following commands to create the needed variables. Replace **myResourceGroup** with the name you're using for this exercise. If you changed the location in the previous step, make the same change in the **location** variable.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    appConfigName=appconfigname$RANDOM
    ```

1. Run the following command to get the name of App Configuration resource. Record the name, you need it later in the exercise.

    ```
    echo $appConfigName
    ```

1. Run the following command to ensure the **Microsoft.AppConfiguration** provider is registered for your subscription.

    ```
    az provider register --namespace Microsoft.AppConfiguration
    ```

1. It can take a few minutes for the registration to complete. Run the following command to check the status of the registration. Proceed to the next step when the results return **Registered**.

    ```
    az provider show --namespace Microsoft.AppConfiguration --query "registrationState"
    ```

1. Run the following command to create an Azure App Configuration resource. This can take a few minutes to run.

    ```
    az appconfig create --location $location \
        --name $appConfigName \
        --resource-group $resourceGroup \
        --sku Free
    ```

    >**Tip:** If there is an issue creating the AppConfig Resource due to quota restrictions using the **Free** SKU value, please use **Developer** instead.
    

### Assign a role to your Microsoft Entra user name

To retrieve configuration information, you need to assign your Microsoft Entra user to the **App Configuration Data Reader** role. 

1. Run the following command to retrieve the **userPrincipalName** from your account. This represents who the role will be assigned to.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Run the following command to retrieve the resource ID of your App Configuration service. The resource ID sets the scope for the role assignment.

    ```
    resourceID=$(az appconfig show --resource-group $resourceGroup \
        --name $appConfigName --query id --output tsv)
    ```

1. Run the following command to create and assign the **App Configuration Data Reader** role.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "App Configuration Data Reader" \
        --scope $resourceID
    ```

Next, add a placeholder connection string to App Configuration.

### Add configuration information with Azure CLI

In Azure App Configuration, a key like **Dev:conStr** is a hierarchical, or namespaced key. The colon (:) acts as a delimiter that creates a logical hierarchy, where:

* **Dev** represents the namespace or environment prefix (indicating this configuration is for the Development environment)
* **conStr** represents the configuration name

This hierarchical structure allows you to organize configuration settings by environment, feature, or application component, making it easier to manage and retrieve related settings.

Run the following command to store the placeholder connection string. 

```
az appconfig kv set --name $appConfigName \
    --key Dev:conStr \
    --value connectionString \
    --yes
```

This command returns some JSON. The last line contains the value in plain text. 

```json
"value": "connectionString"
```

## Create a .NET console app to retrieve configuration information

Now that the needed resources are deployed to Azure the next step is to set up the console application. The following steps are performed in the cloud shell.

>**Tip:** Resize the cloud shell to display more information, and code, by dragging the top border. You can also use the minimize and maximize buttons to switch between the cloud shell and the main portal interface.

1. Run the following commands to create a directory to contain the project and change into the project directory.

    ```
    mkdir appconfig
    cd appconfig
    ```

1. Create the .NET console application.

    ```
    dotnet new console
    ```

1. Run the following commands to add the **Azure.Identity** and **Microsoft.Extensions.Configuration.AzureAppConfiguration** packages to the project.

    ```
    dotnet add package Azure.Identity
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

### Add the code for the project

1. Run the following command in the cloud shell to begin editing the application.

    ```
    code Program.cs
    ```

1. Replace any existing contents with the following code. Be sure to replace **YOUR_APP_CONFIGURATION_NAME** with the name you recorded earlier, and read through the comments in the code.

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    using Azure.Identity;
    
    // Set the Azure App Configuration endpoint, replace YOUR_APP_CONFIGURATION_NAME
    // with the name of your actual App Configuration service
    
    string endpoint = "https://YOUR_APP_CONFIGURATION_NAME.azconfig.io"; 
    
    // Configure which authentication methods to use
    // DefaultAzureCredential tries multiple auth methods automatically
    DefaultAzureCredentialOptions credentialOptions = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a configuration builder to combine multiple config sources
    var builder = new ConfigurationBuilder();
    
    // Add Azure App Configuration as a source
    // This connects to Azure and loads configuration values
    builder.AddAzureAppConfiguration(options =>
    {
        
        options.Connect(new Uri(endpoint), new DefaultAzureCredential(credentialOptions));
    });
    
    // Build the final configuration object
    try
    {
        var config = builder.Build();
        
        // Retrieve a configuration value by key name
        Console.WriteLine(config["Dev:conStr"]);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error connecting to Azure App Configuration: {ex.Message}");
    }
    ```

1. Press **ctrl+s** to save the file, then **ctrl+q** to exit the editor.

## Sign into Azure and run the app

1. In the cloud shell, enter the following command to sign into Azure.

    ```
    az login
    ```

    **<font color="red">You must sign into Azure - even though the cloud shell session is already authenticated.</font>**

    > **Note**: In most scenarios, just using *az login* will be sufficient. However, if you have subscriptions in multiple tenants, you may need to specify the tenant by using the *--tenant* parameter. See [Sign into Azure interactively using Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) for details.

1. Run the following command to start the console app. The app will display the **connectionString** value you assigned to the **Dev:conStr** setting earlier in the exercise.

    ```
    dotnet run
    ```

    The app will display the **connectionString** value you assigned to the **Dev:conStr** setting earlier in the exercise.

## Clean up resources

Now that you finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.
1. Navigate to the resource group you created and view the contents of the resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.

> **CAUTION:** Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.
