---
title: '#PSTip Listing all blobs in a public Azure storage container'
author: Ravikanth C
type: post
date: 2014-04-10T18:00:53+00:00
url: /2014/04/10/pstip-listing-all-blobs-in-a-public-azure-storage-container/
categories:
  - Azure
  - Tips and Tricks
tags:
  - Azure
  - Tips and Tricks

---
In the earlier article, I showed you how to [deploy a Azure VM from VM depot][1]. The VM depot has a public storage container where all the VHDs are stored as blobs. I was looking for a way to get a list of all blobs in a public container and had no luck with the Azure PowerShell cmdlets.

I had approached [Gaurav][2]&#8211;good friend and an Azure MVP&#8211;and he showed me a way to do this using [Azure Blob Service API][3]. This is a simple and neat trick. The following code snippet shows how this can be done. For this example, we will use the VM depot community blob store URL (https://vmdepoteastus.blob.core.windows.net/linux-community-store/).

```
$uri = "https://vmdepoteastus.blob.core.windows.net/linux-community-store/?comp=list&restype=container"
$wc = New-Object System.Net.WebClient
$xml = (($wc.DownloadString($uri))).EnumerationResults

$xml.Blobs.Blob | ForEach-Object {
    New-Object -TypeName PSObject -Property @{
        BlobName = $_.Name
        BlobUrl = $_.Url
        BlobType = $_.Properties.BlobType
    }
}
```

![](/images/15.png)


I tried using _Invoke-RestMethod_ and _Invoke-WebRequest_. Both are returning invalid XML and therefore I am forced to use .NET WebClient. This is a simple way to explore what is there in an Azure storage container.

[1]: /2014/03/20/deploying-azure-virtual-machines-from-vm-depot-using-powershell/
[2]: http://gauravmantri.com/
[3]: https://myaccount.blob.core.windows.net/?comp=list