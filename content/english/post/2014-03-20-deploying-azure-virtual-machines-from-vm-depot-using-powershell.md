---
title: Deploying Azure Virtual Machines from VM Depot using PowerShell
author: Ravikanth C
type: post
date: 2014-03-20T18:36:57+00:00
url: /2014/03/20/deploying-azure-virtual-machines-from-vm-depot-using-powershell/
categories:
  - How To
  - Azure
  - Kemp
tags:
  - Azure
  - Kemp
  - How To
---
[VM Depot][1] is a community-driven catalog of open source virtual machine images. Azure management portal can be used to copy these VM images into &#8220;My images&#8221; of your Azure subscription. Once we have the community image in &#8220;My images&#8221;, we can create a VM from that. This is a multi-step process. There is no PowerShell equivalent of doing this. We can, however, use [Azure CLI][2] to achieve this in a single command. I will show you how this can be done using PowerShell. For this example, I will deploy Kemp&#8217;s Virtual LoadMaster from VM Depot.

Before we go to the PowerShell method, let us look at how this is done using Azure CLI. The deployment script for the Azure CLI can be obtained from the VM Depot portal.

![](/images/vmimage.png)

I wanted to understand how Azure CLI can achieve what is not available in Azure PowerShell. So, from what I found, these are the steps involved.

  1. Get the VM image name. In this example, &#8220;vmdepot-23920-1-1&#8221; is the VM image name.
  2. Retrieve the VHD using the VM image name. Azure CLI uses the OData service provided by VM Depot.
  3. Copy the VHD blob from community images container obtained in step 2 to your storage account.
  4. Create a VM image from the copied VHD blob.
  5. Create a new VM from the image created in step 4.

We seriously need PowerShell here! 🙂

Step 1 is already known. We can just grab the VM image name from the deployment script given by VM Depot. Let us go to step 2.

### Retrieve VHD blob path from VM image name

For this part, I reverse engineered what Azure CLI was doing. This is when I came to know that VM Depot has a OData service. We need to use the ResolveUid endpoint and pass the VM image name as the uid.

<pre class="brush: powershell; title: ; notranslate" title="">(Invoke-RestMethod -Uri 'http://vmdepot.msopentech.com/OData.svc/ResolveUid?uid=%27vmdepot-23920-1-1%27').ResolveUid.Element.BlobUrl
</pre>

### Copying the VM VHD blob to Azure storage account

This is easy. We can use the Azure cmdlets to achieve this. You need a storage account, access key, and an existing container. You can look at the Azure management portal for this information. The following code snippet helps you copy the VHD blob to your storage account.

```
$BlobUri = "http://vmdepoteastus.blob.core.windows.net/linux-community-store/community-34189-c64af7cb-cfec-4c4f-b893-99fd4debdf3a-1.vhd"

Select-AzureSubscription "your-subscription-name"
$StorageAccount = "your-storageaccount-name"
$StorageKey = "your-storageaccount-key"

$DestinationContext = New-AzureStorageContext –StorageAccountName $StorageAccount -StorageAccountKey $StorageKey
$ContainerName = "your-container-name"

$StorageBlob = Start-AzureStorageBlobCopy -SrcUri $BlobUri -DestContainer $ContainerName -DestBlob "kemp1.vhd" -DestContext $DestinationContext
$CopyStatus = $StorageBlob | Get-AzureStorageBlobCopyState

While ($CopyStatus.Status -eq 'Pending') {
   $CopyStatus = ($StorageBlob | Get-AzureStorageBlobCopyState)
   Write-Progress -Id 1 -Activity "Copying VHD BLOB to Azure Storage" -Status ("{0} bytes of {1} completed" -f $CopyStatus.BytesCopied, $CopyStatus.TotalBytes)
   Start-Sleep -Seconds 5
}

"Blob copy completed with status: $($copyStatus.Status)"
```

Once the copy process completes successfully, you can see the VHD added to the container in your storage account.

![](/images/vhd.png)

Creating VM image from the VHD

Once again, this is a simple task with the help of Azure PowerShell cmdlets. We need the location of the VHD we copied in the previous step. We can use the storage context created earlier to create this Uri.

<pre class="brush: powershell; title: ; notranslate" title="">$ImageLocation = "$($DestinationContext.BlobEndPoint)$($ContainerName)/kemp1.vhd"
</pre>

We can use the _Add-AzureVMImage_ cmdlet to add this VHD as a VM image.

<pre class="brush: powershell; title: ; notranslate" title="">Add-AzureVMImage -ImageName "Kemp" -MediaLocation $ImageLocation -OS Linux -Verbose
</pre>

This usually takes a little while to show up in images. You can use the _Get-AzureVMImage_ cmdlet to verify this.

<pre class="brush: powershell; title: ; notranslate" title="">Get-AzureVMImage -ImageName "Kemp"
</pre>

### Create a new VM from the image

We can use the _New-AzureVM_ cmdlet to deploy the Kemp VLM.

<pre class="brush: powershell; title: ; notranslate" title="">$KempImage = Get-AzureVMImage -ImageName Kemp
$KempVLM = New-AzureVMConfig -Name "Kemp-VLM-1" -InstanceSize Small -ImageName $KempImage.ImageName | Add-AzureProvisioningConfig -Linux -LinuxUser "Ravikanth" -Password "Pass@w0Rd" -NoSSHEndPoint -Verbose
New-AzureVM -ServiceName "ravikemp" -VMs $KempVLM -WaitForBoot
</pre>

This is it. A virtual machine has been deployed using a VM Depot image. You can use this procedure to deploy any VM Depot image.

While we are talking about Kemp VLM, let us see how we can complete the process of configuring Kemp VLM. We need to create the endpoints for accessing the VLM locally and externally.

<span style="line-height: 1.5em;">The <em>Add-AzureEndpoint</em> cmdlet can be used to add the endpoints to the Azure VM.</span>

<pre class="brush: powershell; title: ; notranslate" title="">Get-AzureVM -ServiceName "RaviKemp" -Name "Kemp-VLM-1" |
Add-AzureEndpoint -Name "Port22" -Protocol TCP -LocalPort 22 -PublicPort 22 -Verbose |
Add-AzureEndpoint -Name "Port8443" -Protocol TCP -LocalPort 8443 -PublicPort 8443 -Verbose |
Update-AzureVM -Verbose
</pre>

<p style="text-align: left;">
  Once we have the endpoints created, we can access the Kemp Web User Interface to license the VLM. You will require a Kemp ID to complete the licensing process.
</p>

<pre class="brush: powershell; title: ; notranslate" title="">Start-Process "https://VMPublicVIPAddress:8443/"
</pre>

<p style="text-align: left;">
  In the upcoming posts, we will see using <a href="http://forums.kemptechnologies.com/index.php?p=/categories/powershell-for-loadmaster">Kemp PowerShell module</a> to manage a Kemp VLM.
</p>

[1]: http://vmdepot.msopentech.com/List/Index?search=&sort=Date&page=3
[2]: http://www.windowsazure.com/en-us/documentation/articles/xplat-cli/