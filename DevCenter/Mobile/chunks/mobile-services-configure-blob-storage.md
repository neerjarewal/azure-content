

A new insert script is registered that generates an SAS when a new Todo item is inserted.

0. If you haven't yet created your storage account, see [How To Create a Storage Account].

1. In the Management Portal, click **Storage**, click the storage account, then click **Manage Keys**. 

  ![][0]

2. Make a note of the **Storage Account Name** and **Access Key**.

   ![][1]

3. In your mobile service, click the **Configure** tab, scroll down to **App settings** and enter a **Name** and **Value** pair for each of the following that you obtained from the storage account, then click **Save**.

	+ `STORAGE_ACCOUNT_NAME`
	+ `STORAGE_ACCOUNT_ACCESS_KEY`

	![10][]

	The storage account access key is stored encrypted in app settings. You can access this key from any server script at runtime. For more information, see [App settings].

4. Click the **Data** tab and then click the **TodoItem** table. 

   ![][3]

5.  In **todoitem**, click the **Script** tab and select **Insert**, replace the insert function with the following code, then click **Save**:

		var azure = require('azure');
		var qs = require('querystring');
		var appSettings = require('mobileservice-config').appSettings;
		
		function insert(item, user, request) {
		    // Get storage account settings from app settings. 
		    var accountName = appSettings.STORAGE_ACCOUNT_NAME;
		    var accountKey = appSettings.STORAGE_ACCOUNT_ACCESS_KEY;
		    var host = accountName + '.blob.core.windows.net';
		
		    if ((typeof item.containerName !== "undefined") && (
		    item.containerName !== null)) {
		        // Set the BLOB store container name on the item, which must be lowercase.
		        item.containerName = item.containerName.toLowerCase();
		
		        // If it does not already exist, create the container 
		        // with public read access for blobs.        
		        var blobService = azure.createBlobService(accountName, accountKey, host);
		        blobService.createContainerIfNotExists(item.containerName, {
		            publicAccessLevel: 'blob'
		        }, function(error) {
		            if (!error) {
		
		                // Provide write access to the container for the next 5 mins.        
		                var sharedAccessPolicy = {
		                    AccessPolicy: {
		                        Permissions: azure.Constants.BlobConstants.SharedAccessPermissions.WRITE,
		                        Expiry: new Date(new Date().getTime() + 5 * 60 * 1000)
		                    }
		                };
		
		                // Generate the upload URL with SAS for the new image.
		                var sasQueryUrl = 
		                blobService.generateSharedAccessSignature(item.containerName, 
		                item.resourceName, sharedAccessPolicy);
		
		                // Set the query string.
		                item.sasQueryString = qs.stringify(sasQueryUrl.queryString);
		
		                // Set the full path on the new new item, 
		                // which is used for data binding on the client. 
		                item.imageUri = sasQueryUrl.baseUrl + sasQueryUrl.path;
		
		            } else {
		                console.error(error);
		            }
		            request.execute();
		        });
		    } else {
		        request.execute();
		    }
		}

 	![][4]

   This replaces the function that is invoked when an insert occurs in the TodoItem table with a new script. This new script generates a new SAS for the insert, which is valid for 5 minutes, and assigns the value of the generated SAS to the `sasQueryString` property of the returned item. The `imageUri` property is also set to the resource path of the new BLOB to enable image display during binding in the client UI.

Next, you will update the quickstart app to add image upload functionality by using the SAS generated on insert.

 
<!-- Anchors. -->

<!-- Images. -->
[0]: ../Media/mobile-blob-storage-account.png
[1]: ../Media/mobile-blob-storage-account-keys.png
[2]: ../Media/mobile-add-storage-nuget-package-dotnet.png
[3]: ../Media/mobile-portal-data-tables.png
[4]: ../Media/mobile-insert-script-blob.png
[5]: ../Media/mobile-app-manifest-camera.png
[6]: ../Media/mobile-quickstart-blob-appbar.png
[7]: ../Media/mobile-quickstart-blob-camera.png
[8]: ../Media/mobile-quickstart-blob-appbar2.png
[9]: ../Media/mobile-quickstart-blob-ie.png
[10]: ../Media/mobile-blob-storage-app-settings.png

<!-- URLs. -->
[How To Create a Storage Account]: /en-us/manage/services/storage/how-to-create-a-storage-account
[App settings]: http://msdn.microsoft.com/en-us/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7