---
lab:
    topic: Azure Storage
    title: 'Create Blob storage resources with the .NET client library'
    description: 'Learn how to use the Azure Storage .NET client library to create containers, upload and list blobs, and delete containers.'
---

# Create Blob storage resources with the .NET client library

In this exercise, you create an Azure Storage account and build a .NET console application using the Azure Storage Blob client library to create containers, upload files to blob storage, list blobs, and download files. You learn how to authenticate with Azure, perform blob storage operations programmatically, and verify results in the Azure portal.

Tasks performed in this exercise:

* Prepare the Azure resources
* Create a console app to create and download data
* Run the app and verify results
* Clean up resources

This exercise takes approximately **30** minutes to complete.

## Create an Azure Storage account

In this section of the exercise you create the needed resources in Azure with the Azure CLI.

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new cloud shell in the Azure portal, selecting a ***Bash*** environment. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal. If you are prompted to select a storage account to persist your files, select **No storage account required**, your subscription, and then select **Apply**.

    > **Note**: If you have previously created a cloud shell that uses a *PowerShell* environment, switch it to ***Bash***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

1. Create a resource group for the resources needed for this exercise. Replace **myResourceGroup** with a name you want to use for the resource group. You can replace **eastus2** with a region near you if needed. If you already have a resource group you want to use, proceed to the next step.

    ```
    az group create --location eastus2 --name myResourceGroup
    ```

1. Many of the commands require unique names and use the same parameters. Creating some variables will reduce the changes needed to the commands that create resources. Run the following commands to create the needed variables. Replace **myResourceGroup** with the name you're using for this exercise.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    accountName=storageacct$RANDOM
    ```

1. Run the following commands to create the Azure Storage account, each account name must be unique. The first command creates a variable with a unique name for your storage account. Record the name of your account from the output of the **echo** command. 

    ```
    az storage account create --name $accountName \
        --resource-group $resourceGroup \
        --location $location \
        --sku Standard_LRS 
    
    echo $accountName
    ```

### Assign a role to your Microsoft Entra user name

To allow your app to create resources and items, assign your Microsoft Entra user to the **Storage Blob Data Owner** role. Perform the following steps in the cloud shell.

>**Tip:** Resize the cloud shell to display more information, and code, by dragging the top border. You can also use the minimize and maximize buttons to switch between the cloud shell and the main portal interface.

1. Run the following command to retrieve the **userPrincipalName** from your account. This represents who the role will be assigned to.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Run the following command to retrieve the resource ID of the storage account. The resource ID sets the scope for the role assignment to a specific namespace.

    ```
    resourceID=$(az storage account show --name $accountName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. Run the following command to create and assign the **Storage Blob Data Owner** role. This role gives you the permissions to manage containers and items.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Blob Data Owner" \
        --scope $resourceID
    ```

## Create a .NET console app to create containers and items

Now that the needed resources are deployed to Azure the next step is to set up the console application. The following steps are performed in the cloud shell.

1. Run the following commands to create a directory to contain the project and change into the project directory.

    ```
    mkdir azstor
    cd azstor
    ```

1. Create the .NET console application.

    ```
    dotnet new console
    ```

1. Run the following commands to add the required packages in the application.

    ```
    dotnet add package Azure.Storage.Blobs
    dotnet add package Azure.Identity
    ```

1. Run the following command to create a **data** folder in your project. 

    ```
    mkdir data
    ```

Now it's time to add the code for the project.

### Add the starter code for the project

1. Run the following command in the cloud shell to begin editing the application.

    ```
    code Program.cs
    ```

1. Replace any existing contents with the following code. Be sure to review the comments in the code.

    ```csharp
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Identity;
    
    Console.WriteLine("Azure Blob Storage exercise\n");
    
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Run the examples asynchronously, wait for the results before proceeding
    await ProcessAsync();
    
    Console.WriteLine("\nPress enter to exit the sample application.");
    Console.ReadLine();
    
    async Task ProcessAsync()
    {
        // CREATE A BLOB STORAGE CLIENT
        
    
    
        // CREATE A CONTAINER
        
    
    
        // CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE
        
    
    
        // UPLOAD THE FILE TO BLOB STORAGE
        
    
    
        // LIST BLOBS IN THE CONTAINER
        
    
    
        // DOWNLOAD THE BLOB TO A LOCAL FILE
        
    
    }
    ```

1. Press **ctrl+s** to save your changes, and continue to the next step.


## Add code to complete the project

Throughout the rest of the exercise you add code in specified areas to create the full application. 

1. Locate the **// CREATE A BLOB STORAGE CLIENT** comment, then add the following code directly beneath the comment. The **BlobServiceClient** acts as the primary entry point for managing containers and blobs in a storage account. The client uses the *DefaultAzureCredential* for authentication. Be sure to replace **YOUR_ACCOUNT_NAME** with the name you recorded earlier.

    ```csharp
    // Create a credential using DefaultAzureCredential with configured options
    string accountName = "YOUR_ACCOUNT_NAME"; // Replace with your storage account name
    
    // Use the DefaultAzureCredential with the options configured at the top of the program
    DefaultAzureCredential credential = new DefaultAzureCredential(options);
    
    // Create the BlobServiceClient using the endpoint and DefaultAzureCredential
    string blobServiceEndpoint = $"https://{accountName}.blob.core.windows.net";
    BlobServiceClient blobServiceClient = new BlobServiceClient(new Uri(blobServiceEndpoint), credential);
    ```

1. Press **ctrl+s** to save your changes, and continue to the next step.

1. Locate the **// CREATE A CONTAINER** comment, then add the following code directly beneath the comment. Creating a container includes creating an instance of the **BlobServiceClient** class, and then calling the **CreateBlobContainerAsync** method to create the container in your storage account. A GUID value is appended to the container name to ensure that it's unique. The **CreateBlobContainerAsync** method fails if the container already exists.

    ```csharp
    // Create a unique name for the container
    string containerName = "wtblob" + Guid.NewGuid().ToString();
    
    // Create the container and return a container client object
    Console.WriteLine("Creating container: " + containerName);
    BlobContainerClient containerClient = 
        await blobServiceClient.CreateBlobContainerAsync(containerName);
    
    // Check if the container was created successfully
    if (containerClient != null)
    {
        Console.WriteLine("Container created successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("Failed to create the container, exiting program.");
        return;
    }
    ```

1. Press **ctrl+s** to save your changes, and continue to the next step.

1. Find the **// CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE** comment, then add the following code directly beneath the comment. This creates a file in the data directory that is uploaded to the container.

    ```csharp
    // Create a local file in the ./data/ directory for uploading and downloading
    Console.WriteLine("Creating a local file for upload to Blob storage...");
    string localPath = "./data/";
    string fileName = "wtfile" + Guid.NewGuid().ToString() + ".txt";
    string localFilePath = Path.Combine(localPath, fileName);
    
    // Write text to the file
    await File.WriteAllTextAsync(localFilePath, "Hello, World!");
    Console.WriteLine("Local file created, press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Press **ctrl+s** to save your changes, and continue to the next step.

1. Locate the **// UPLOAD THE FILE TO BLOB STORAGE** comment, then add the following code directly beneath the comment. The code gets a reference to a **BlobClient** object by calling the **GetBlobClient** method on the container created in the previous section. It then uploads a generated local file using the **UploadAsync** method. This method creates the blob if it doesn't already exist, and overwrites it if it does.

    ```csharp
    // Get a reference to the blob and upload the file
    BlobClient blobClient = containerClient.GetBlobClient(fileName);
    
    Console.WriteLine("Uploading to Blob storage as blob:\n\t {0}", blobClient.Uri);
    
    // Open the file and upload its data
    using (FileStream uploadFileStream = File.OpenRead(localFilePath))
    {
        await blobClient.UploadAsync(uploadFileStream);
        uploadFileStream.Close();
    }
    
    // Verify if the file was uploaded successfully
    bool blobExists = await blobClient.ExistsAsync();
    if (blobExists)
    {
        Console.WriteLine("File uploaded successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("File upload failed, exiting program..");
        return;
    }
    ```

1. Press **ctrl+s** to save your changes, and continue to the next step.

1. Locate the **// LIST BLOBS IN THE CONTAINER** comment, then add the following code directly beneath the comment. You list the blobs in the container with the **GetBlobsAsync** method. In this case, only one blob was added to the container, so the listing operation returns just that one blob. 

    ```csharp
    Console.WriteLine("Listing blobs in container...");
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        Console.WriteLine("\t" + blobItem.Name);
    }
    
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Press **ctrl+s** to save your changes, and continue to the next step.

1. Locate the **// DOWNLOAD THE BLOB TO A LOCAL FILE** comment, then add the following code directly beneath the comment. The code uses the **DownloadAsync** method to download the blob created previously to your local file system. The example code adds a suffix of "DOWNLOADED" to the blob name so that you can see both files in local file system. 

    ```csharp
    // Adds the string "DOWNLOADED" before the .txt extension so it doesn't 
    // overwrite the original file
    
    string downloadFilePath = localFilePath.Replace(".txt", "DOWNLOADED.txt");
    
    Console.WriteLine("Downloading blob to: {0}", downloadFilePath);
    
    // Download the blob's contents and save it to a file
    BlobDownloadInfo download = await blobClient.DownloadAsync();
    
    using (FileStream downloadFileStream = File.OpenWrite(downloadFilePath))
    {
        await download.Content.CopyToAsync(downloadFileStream);
    }
    
    Console.WriteLine("Blob downloaded successfully to: {0}", downloadFilePath);
    ```

1. Press **ctrl+s** to save the file, then **ctrl+q** to exit the editor.

## Sign into Azure and run the app

1. In the cloud shell command-line pane, enter the following command to sign into Azure.

    ```
    az login
    ```

    **<font color="red">You must sign into Azure - even though the cloud shell session is already authenticated.</font>**

    > **Note**: In most scenarios, just using *az login* will be sufficient. However, if you have subscriptions in multiple tenants, you may need to specify the tenant by using the *--tenant* parameter. See [Sign into Azure interactively using Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) for details.

1. Run the following command to start the console app. The app will pause many times during execution waiting for you to press any key to continue. This gives you an opportunity to view the messages in the Azure portal.

    ```
    dotnet run
    ```

1. In the Azure portal, navigate to the Azure Storage account you created. 

1. Expand **> Data storage** in the left navigation and select **Containers**.

1. Select the container the application created and you can view the blob that was uploaded.

1. Run the two commands below to change into the **data** directory and list the files that were uploaded and downloaded.

    ```
    cd data
    ls
    ```

## Clean up resources

Now that you finished the exercise, you should delete the cloud resources you created to avoid unnecessary resource usage.

1. In your browser navigate to the Azure portal [https://portal.azure.com](https://portal.azure.com); signing in with your Azure credentials if prompted.
1. Navigate to the resource group you created and view the contents of the resources used in this exercise.
1. On the toolbar, select **Delete resource group**.
1. Enter the resource group name and confirm that you want to delete it.

> **CAUTION:** Deleting a resource group deletes all resources contained within it. If you chose an existing resource group for this exercise, any existing resources outside the scope of this exercise will also be deleted.

