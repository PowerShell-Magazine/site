---
title: Copy DSC resources to Azure VMs using Custom Script Extension
author: Ravikanth C
type: post
date: 2014-05-02T17:30:43+00:00
url: /2014/05/02/copy-dsc-resources-to-azure-vms-using-custom-script-extension/
categories:
  - PowerShell DSC
  - Azure
tags:
  - PowerShell DSC
  - Azure

---
In an earlier article, I showed you [how to use the Custom Script Extension][1]. The examples that I had there were too trivial and were intended to explain how the Custom Script Extension works. In this article, I will show one of the use cases for the Custom Script Extension. I have been using this to deploy my custom DSC resources to my Azure virtual machines.

For the purpose of storing all the DSC resources, I created a container in my existing Azure storage account and made that a public container. So, you can use most of the content and scripts from this post to deploy the DSC resources to your Azure VMs. Without delay, let us see how this is done.

### Copying files/folders to an Azure container

First of all, I will show you the code that is used to copy the DSC resources to a storage container. You can certainly use one of the Azure storage explorers out there to achieve this. But, hey, this is PowerShell Magazine! 🙂

```
$StorageAccount = 'psmag'
$StorageKey = 'storagekey'
$StorageContainer = 'scripts'
$LocalFolderPath = 'C:\DSCResources'

$StorageContext = New-AzureStorageContext -StorageAccountName $StorageAccount -StorageAccountKey $StorageKey
$AzureBlobContainer = Get-AzureStorageContainer -Name $StorageContainer -Context $StorageContext

foreach ($file in (Get-ChildItem -Recurse $LocalFolderPath -File)) {
    Set-AzureStorageBlobContent -File $file.FullName -Blob (Split-Path -Path $file.FullName -NoQualifier) -Context $StorageContext -Container $StorageContainer -Force
}
```

Within Azure storage, there is no concept called subfolders. There is a storage account and a single container. For using the above code, you need to replace the values of _$StorageAccount, $StorageContainer_, and _$StorageKey_ variables to your own. Also, you need to extract the [recent wave of DSC resources from PowerShell team][2] to C:&#46; This is how that looks on my system.

![](/images/cse1.png)

Like I&#8217;d mentioned, there is no concept of subfolders in Azure containers. So, when we need to upload a complete folder structure to an Azure container, we need to use a technique that uses the folder and subfolder names as part of the file name. For example, if there is a file at C:\DSCResources\xActiveDirectory\DSCResources\MSFT\_xADUser\MSFT\_xADUser.psm1 and we need to upload this to an Azure container, we simply name the blob file name as /DSCResources/xActiveDirectory/DSCResources/MSFT\_xADUser/MSFT\_xADUser.psm1. In the above code, we just iterate through the local folder and convert each file name to the format we need for the Azure blob. This is what the _Split-Path_ cmdlet is achieving inside the _foreach_ loop.

### Reading the contents of a public Azure container

In an earlier article, I showed you how we can use the Azure blob services API to [read the contents of a public Azure container][3]. If you look into the contents of an Azure storage container either in the management portal or using the script I&#8217;d posted in the [earlier article][3], you will notice that each blob inside the container has a blob URL and a name.

<pre class="brush: powershell; title: ; notranslate" title="">$uri = "http://psmag.blob.core.windows.net/scripts/?comp=list&restype=container"
$wc = New-Object System.Net.WebClient
$xml = (($wc.DownloadString($uri))).EnumerationResults
$scripts = $xml.Blobs.Blob | ForEach-Object {
	if ($_.Name.StartsWith('/DSCResources')) {
		New-Object -TypeName PSObject -Property @{
			BlobName = $_.Name.Remove(0,'/DSCResources/'.Length)
			BlobUrl = $_.Url
		}
	}
}
</pre>
As you see above, I am only interested in files that are copied as a part of DSC resources. I have so many other scripts in that container but it simply does not make sense to copy them to the modules folder. So, I am filtering the blob names for those that start with /DSCResources. 

![](/images/2-1024x401.png)

Once we have the name of file blob in the container, we need to convert it to the format the file system will understand. In this use case, we are looking at deploying DSC custom resource modules. There are different places you can copy the DSC resources to. In this example we will copy the DSC resources to C:\Program Files\WindowsPowerShell\Modules folder. So, when copying to this folder, we don&#8217;t need the top level DSC resources folder. So, in the above script, I am removing /DSCResources/ from the blob file name. The final output can be seen in the above screenshot.

### Downloading the files and copying them to module path

This is the final step. We have to invoke this action inside the Azure VM. And, the Azure Custom Script Extension is the method to do that. In an earlier article, I showed you how to use this extension. So, without going into the details, I will show you here how this extension can be used to deploy the custom DSC resources to an Azure VM. In the previous section, I had already showed you how to read the contents of a public Azure storage container and how to prepare the blob names so that we can use that to create the necessary folder structure. Now, we will see how the code for that can be modified to download the scripts and copy them to the file system inside the Azure VM.

```
param (
    $BlobUri = "http://psmag.blob.core.windows.net/scripts",
    $DSCResourcePath = "${env:ProgramFiles}\WindowsPowerShell\Modules"
)

$Uri = "${BlobUri}/?comp=list&restype=container"
$WebClient = New-Object System.Net.WebClient
$BlobXml = (($WebClient.DownloadString($uri))).EnumerationResults

$BlobXml.Blobs.Blob | ForEach-Object {
    if ($_.Name.StartsWith('/DSCResources')) {
        $FileRelativePath = $_.Name.Remove(0,'/DSCResources/'.Length)
        $FileFullPath = Join-Path -Path $DSCResourcePath -ChildPath $FileRelativePath
        if (-not (Test-Path (Split-Path $FileFullPath -Parent))) {
            New-Item -Path (Split-Path $FileFullPath -Parent) -ItemType Directory -ErrorAction Stop | Out-Null
        }
        $WebClient.DownloadFile($_.Url, $FileFullPath)
        Write-Host "Downloaded ${FileFullPath}"
    }
}
```

I copied the code shown above into a PowerShell script and called it _CopyDSCResources.ps1_. This is also available in my public Azure storage container. The _New-Item_ cmdlet used within the _foreach_ loop creates the necessary folder structure to copy the file. So, at the end of this script execution, we will have all the DSC resources copied to the Azure VM at the specified module location.

<pre class="brush: powershell; title: ; notranslate" title="">$Vm = Get-AzureVM -ServiceName "DSCDemo" -Name "DSCPull"
Set-AzureVMCustomScriptExtension -FileUri http://psmag.blob.core.windows.net/scripts/CopyDSCResources.ps1 -Run CopyDSCResources.ps1 -VM $Vm | Update-AzureVM -Verbose
</pre>

This is it. You can use the last code snippet to copy the DSC resources from my public Azure container to your Azure VMs if you have enabled the Azure Custom Script Extension.

[1]: /2014/04/30/understanding-azure-custom-script-extension
[2]: http://gallery.technet.microsoft.com/scriptcenter/DSC-Resource-Kit-All-c449312d
[3]: /2014/04/10/pstip-listing-all-blobs-in-a-public-azure-storage-container/