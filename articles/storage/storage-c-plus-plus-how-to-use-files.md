<properties
	pageTitle="How to use File Storage from C++ | Microsoft Azure"
	description="Store file data in the cloud with Azure File storage."
	services="storage"
	documentationCenter=".net"
	authors="seguler"
	manager="jahogg"
	editor="tysonn" />

<tags ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/10/2016"
	ms.author="seguler;robinsh" />

# How to use File Storage from C++

[AZURE.INCLUDE [storage-selector-file-include](../../includes/storage-selector-file-include.md)]
<br/>
[AZURE.INCLUDE [storage-try-azure-tools-files](../../includes/storage-try-azure-tools-files.md)]
[AZURE.INCLUDE [storage-file-overview-include](../../includes/storage-file-overview-include.md)]

## About this tutorial

In this tutorial, you’ll learn how to perform basic operations on the Microsoft Azure File storage service. Through samples written in C++, you’ll learn how to create shares and directories, upload, list, and delete files. If you are new to Microsoft Azure’s File Storage service, going through the concepts in the sections that follow will be very helpful in understanding the samples.

[AZURE.INCLUDE [storage-file-concepts-include](../../includes/storage-file-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## Create a C++ application

To build the samples, you will need to install the Azure Storage Client Library 2.4.0 for C++. You should also have created an Azure storage account.

To install the Azure Storage Client 2.4.0 for C++, you can use one of the following methods:

-   **Linux:** Follow the instructions given in the [Azure Storage Client Library for C++ README](https://github.com/Azure/azure-storage-cpp/blob/master/README.md) page.

-   **Windows:** In Visual Studio, click **Tools &gt; NuGet Package Manager &gt; Package Manager Console**. Type the following command into the [NuGet Package Manager console](http://docs.nuget.org/docs/start-here/using-the-package-manager-console) and press **ENTER**.

	Install-Package wastorage

## Set up your application to use File storage

Add the following include statements to the top of the C++ file where you want to use the Azure storage APIs to access files:

	#include "was/storage_account.h"
	#include "was/file.h"

## Set up an Azure storage connection string

To use File storage, you need to connect to your Azure storage account. The first step would be to configure a connection string which we’ll use to connect to your storage account. Let’s define a static variable to do that.

	// Define the connection-string with your values.
	const utility::string_t 
	storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));

## Connecting to an Azure storage account

You can use the **cloud_storage_account** class to represent your Storage Account information. To retrieve your storage account information from the storage connection string, you can use the **parse** method.

	// Retrieve storage account from connection string.	
	azure::storage::cloud_storage_account storage_account = 
	  azure::storage::cloud_storage_account::parse(storage_connection_string);

## How to: Create a Share

All files and directories in File storage reside in a container called a **Share**. Your storage account can have as many shares as your account capacity allows. To obtain access to a share and its contents, you need to use a File storage client.

	// Create the file storage client.
	azure::storage::cloud_file_client file_client = 
	  storage_account.create_cloud_file_client();

Using the File storage client, you can then obtain a reference to a share.

	// Get a reference to the file share
	azure::storage::cloud_file_share share = 
      file_client.get_share_reference(_XPLATSTR("my-sample-share"));

To create the share, use the **create_if_not_exists** method of the **cloud_file_share** object.

	if (share.create_if_not_exists()) {	
		std::wcout << U("New share created") << std::endl;	
	}

At this point, **share** holds a reference to a share named **my-sample-share**.

## How to: Upload a file

At the very least, an Azure File Storage Share contains a root directory where files can reside. In this section, you'll learn how to upload a file from local storage onto the root directory of a share.

The first step in uploading a file is to obtain a reference to the directory where it should reside. You do this by calling the **get_root_directory_reference** method of the share object.

	//Get a reference to the root directory for the share.
	azure::storage::cloud_file_directory root_dir = share.get_root_directory_reference();

Now that you have a reference to the root directory of the share, you can upload a file onto it. This example uploads from a file, from text, and from a stream.

	// Upload a file from a stream.
	concurrency::streams::istream input_stream = 
      concurrency::streams::file_stream<uint8_t>::open_istream(_XPLATSTR("DataFile.txt")).get();

	azure::storage::cloud_file file1 = 
	  root_dir.get_file_reference(_XPLATSTR("my-sample-file-1"));
	file1.upload_from_stream(input_stream);

	// Upload some files from text.
	azure::storage::cloud_file file2 = 
	  root_dir.get_file_reference(_XPLATSTR("my-sample-file-2"));
	file2.upload_text(_XPLATSTR("more text"));

	// Upload a file from a file.
	azure::storage::cloud_file file4 = 
	  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));
	file4.upload_from_file(_XPLATSTR("DataFile.txt"));	

## How to: Create a Directory

You can also organize storage by putting files inside subdirectories instead of having all of them in the root directory. The Azure file storage service allows you to create as much directories as your account will allow. The code below will create a directory named **my-sample-directory** under the root directory as well as a subdirectory named **my-sample-subdirectory**.
	
	// Retrieve a reference to a directory
	azure::storage::cloud_file_directory directory = share.get_directory_reference(_XPLATSTR("my-sample-directory"));
	
	// Return value is true if the share did not exist and was successfully created.
	directory.create_if_not_exists();
	
	// Create a subdirectory.
	azure::storage::cloud_file_directory subdirectory = 
	  directory.get_subdirectory_reference(_XPLATSTR("my-sample-subdirectory"));
	subdirectory.create_if_not_exists();

## How to: List files and directories in a share

Obtaining a list of files and directories within a share is easily done by calling **list_files_and_directories** on a **cloud_file_directory** reference. To access the rich set of properties and methods for a returned **list_file_and_directory_item**, you must call the **list_file_and_directory_item.as_file** method to get a **cloud_file** object, or the **list_file_and_directory_item.as_directory** method to get a **cloud_file_directory** object.

The following code demonstrates how to retrieve and output the URI of each item in the root directory of the share.

	//Get a reference to the root directory for the share.
	azure::storage::cloud_file_directory root_dir = 
	  share.get_root_directory_reference();
	
	// Output URI of each item.
	azure::storage::list_file_and_diretory_result_iterator end_of_results;
	
	for (auto it = directory.list_files_and_directories(); it != end_of_results; ++it)
	{
		if(it->is_directory())
		{
			ucout << "Directory: " << it->as_directory().uri().primary_uri().to_string() << std::endl;
		}
		else if (it->is_file())
		{
			ucout << "File: " << it->as_file().uri().primary_uri().to_string() << std::endl;
		}		
	}
	

## How to: Download a file

To download files, first retrieve a file reference and then call the **download_to_stream** method to transfer the file contents to a stream object which you can then persist to a local file. Alternatively, you can use the **download_to_file** method to download the contents of a file to a local file. You can use the **download_text** method to download the contents of a file as a text string.

The following example uses the **download_to_stream** and **download_text** methods to demonstrate downloading the files which were created in previous sections.

	// Download as text
	azure::storage::cloud_file text_file = 
	  root_dir.get_file_reference(_XPLATSTR("my-sample-file-2"));
	utility::string_t text = text_file.download_text();
	ucout << "File Text: " << text << std::endl;
	
	// Download as a stream.
	azure::storage::cloud_file stream_file = 
      root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));
	
	concurrency::streams::container_buffer<std::vector<uint8_t>> buffer;
	concurrency::streams::ostream output_stream(buffer);
	stream_file.download_to_stream(output_stream);
	std::ofstream outfile("DownloadFile.txt", std::ofstream::binary);
	std::vector<unsigned char>& data = buffer.collection();
	outfile.write((char *)&data[0], buffer.size());
	outfile.close();

## How to: Delete a file

Another common file storage operation is file deletion. The following code deletes a file named my-sample-file-3 stored under the root directory.

	// Get a reference to the root directory for the share.	
	azure::storage::cloud_file_share share = 
	  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
	
	azure::storage::cloud_file_directory root_dir = 
	  share.get_root_directory_reference();
	
	azure::storage::cloud_file file = 
	  root_dir.get_file_reference(_XPLATSTR("my-sample-file-3"));
	
	file.delete_file_if_exists();

## How to: Delete a directory

Deleting a directory is a simple task, although it should be noted that you cannot delete a directory that still contains files or other directories.
	
	// Get a reference to the share.
	azure::storage::cloud_file_share share = 
	  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
	
	// Get a reference to the directory.
	azure::storage::cloud_file_directory directory = 
	  share.get_directory_reference(_XPLATSTR("my-sample-directory"));
	
	// Get a reference to the subdirectory you want to delete.
	azure::storage::cloud_file_directory sub_directory =
	  directory.get_subdirectory_reference(_XPLATSTR("my-sample-subdirectory"));
	
	// Delete the subdirectory and the sample directory.
	sub_directory.delete_directory_if_exists();
	
	directory.delete_directory_if_exists();

## How to: Delete a Share

Deleting a share is done by calling the **delete_if_exists** method on a cloud_file_share object. Here's sample code that does that.
	
	// Get a reference to the share.
	azure::storage::cloud_file_share share = 
	  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
	
	// delete the share if exists
	share.delete_share_if_exists();

## Set the maximum size for a file share

You can set the quota (or maximum size) for a file share, in gigabytes. You can also check to see how much data is currently stored on the share.

By setting the quota for a share, you can limit the total size of the files stored on the share. If the total size of files on the share exceeds the quota set on the share, then clients will be unable to increase the size of existing files or create new files, unless those files are empty.

The example below shows how to check the current usage for a share and how to set the quota for the share.

	// Parse the connection string for the storage account.
	azure::storage::cloud_storage_account storage_account = 
	  azure::storage::cloud_storage_account::parse(storage_connection_string);
	
	// Create the file client.
	azure::storage::cloud_file_client file_client = 
	  storage_account.create_cloud_file_client();
	
	// Get a reference to the share.
	azure::storage::cloud_file_share share = 
	  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
	if (share.exists())
	{
		std::cout << "Current share usage: " << share.download_share_usage() << "/" << share.properties().quota();
	
		//This line sets the quota to 2560GB
		share.resize(2560);
	
		std::cout << "Quota increased: " << share.download_share_usage() << "/" << share.properties().quota();
	
	}

## Generate a shared access signature for a file or file share

You can generate a shared access signature (SAS) for a file share or for an individual file. You can also create a shared access policy on a file share to manage shared access signatures. Creating a shared access policy is recommended, as it provides a means of revoking the SAS if it should be compromised.

The following example creates a shared access policy on a share, and then uses that policy to provide the constraints for a SAS on a file in the share.

	// Parse the connection string for the storage account.
	azure::storage::cloud_storage_account storage_account = 
	  azure::storage::cloud_storage_account::parse(storage_connection_string);
	
	// Create the file client and get a reference to the share
	azure::storage::cloud_file_client file_client = 
	  storage_account.create_cloud_file_client();
	
	azure::storage::cloud_file_share share = 
	  file_client.get_share_reference(_XPLATSTR("my-sample-share"));
	
	if (share.exists())
	{
		// Create and assign a policy
		utility::string_t policy_name = _XPLATSTR("sampleSharePolicy");

		azure::storage::file_shared_access_policy sharedPolicy = 
		  azure::storage::file_shared_access_policy();

		//set permissions to expire in 90 minutes
		sharedPolicy.set_expiry(utility::datetime::utc_now() + 
	 	  utility::datetime::from_minutes(90));

		//give read and write permissions
		sharedPolicy.set_permissions(azure::storage::file_shared_access_policy::permissions::write | azure::storage::file_shared_access_policy::permissions::read);

		//set permissions for the share
		azure::storage::file_share_permissions permissions;	

		//retrieve the current list of shared access policies
		azure::storage::shared_access_policies<azure::storage::file_shared_access_policy> policies;
		
		//add the new shared policy
		policies.insert(std::make_pair(policy_name, sharedPolicy));

		//save the updated policy list
		permissions.set_policies(policies);
		share.upload_permissions(permissions);

		//Retrieve the root directory and file references
		azure::storage::cloud_file_directory root_dir = 
	  	  share.get_root_directory_reference();
		azure::storage::cloud_file file = 
		  root_dir.get_file_reference(_XPLATSTR("my-sample-file-1"));

		// Generate a SAS for a file in the share 
		//  and associate this access policy with it.		
		utility::string_t sas_token = file.get_shared_access_signature(sharedPolicy);
		
		// Create a new CloudFile object from the SAS, and write some text to the file.		
		azure::storage::cloud_file file_with_sas(azure::storage::storage_credentials(sas_token).transform_uri(file.uri().primary_uri()));
		utility::string_t text = _XPLATSTR("My sample content");		
		file_with_sas.upload_text(text);		
		
		//Download and print URL with SAS.
		utility::string_t downloaded_text = file_with_sas.download_text();		
		ucout << downloaded_text << std::endl;		
		ucout << azure::storage::storage_credentials(sas_token).transform_uri(file.uri().primary_uri()).to_string() << std::endl;
	
	}

For more information about creating and using shared access signatures, see [Using Shared Access Signatures (SAS)](storage-dotnet-shared-access-signature-part-1.md).

## Next Steps

To learn more about Azure Storage, explore these resources:

-   [Storage Client Library for C++](https://github.com/Azure/azure-storage-cpp)

-   [Azure Storage Explorer](http://go.microsoft.com/fwlink/?LinkID=822673&clcid=0x409)

-   [Azure Storage Documentation](https://azure.microsoft.com/documentation/services/storage/)
