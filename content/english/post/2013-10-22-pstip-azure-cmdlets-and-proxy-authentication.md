---
title: '#PSTip Azure cmdlets and proxy authentication'
author: Ravikanth C
type: post
date: 2013-10-22T18:00:32+00:00
url: /2013/10/22/pstip-azure-cmdlets-and-proxy-authentication/
categories:
  - Azure
  - Tips and Tricks
tags:
  - Azure
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

A few days ago, I was working on setting up [Kemp Azure VLM][1]. I will talk about this more in a detailed post later.

So, as part of this process, I had to upload the [Kemp Azure VLM VHD][2] to my Azure storage using Azure PowerShell cmdlets. However, my home proxy was not letting me do that and there was no way to provide proxy credentials when using Azure cmdlets. So, essentially, things like _Add-AzureVHD_ will fail with a 407 error.

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; Add-AzureVhd -LocalFilePath C:\Scripts\LoadMaster-VLM-7.0-3-Azure.vhd -Destination "http://mystorage.blob.core.window
s.net/kemp/kempvlm.vhd"
Add-AzureVhd : "An exception occurred when calling the ServiceManagement API. HTTP Status Code: 407. Service
Management Error Code: &lt;NONE&gt;. Message: &lt;NONE&gt;. Operation Tracking ID: &lt;NONE&gt;."
At line:1 char:1
+ Add-AzureVhd -LocalFilePath C:\Scripts\LoadMaster-VLM-7.0-3-Azure.vhd -Destination ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 + CategoryInfo : CloseError: (:) [Add-AzureVhd], ServiceManagementClientException
 + FullyQualifiedErrorId : Microsoft.WindowsAzure.Management.ServiceManagement.StorageServices.AddAzureVhdCommand
</pre>

That is not a very friendly message but there is an easy way to resolve the problem with a little help of .NET:

<pre class="brush: powershell; title: ; notranslate" title="">[System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials
</pre>

The [_DefaultCredentials_][3] property of [_System.Net.CredentialCache_][4] represents the current security context in which the application is running. For a client-side application, these are usually the Windows credentials (user name, password, and domain) of the user running the application.

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; Add-AzureVhd -LocalFilePath C:\Scripts\LoadMaster-VLM-7.0-3-Azure.vhd -Destination "http://mystorage.
blob.core.windows.net/kemp/kemp.vhd" -Verbose
MD5 hash is being calculated for the file  C:\Scripts\LoadMaster-VLM-7.0-3-Azure.vhd.
MD5 hash calculation is completed.
Elapsed time for the operation: 00:01:40
Creating new page blob of size 34359738880...
Upload failed with exceptions:
Elapsed time for upload: 00:06:50
</pre>

[1]: http://kemptechnologies.com/en/news/kemp-announces-first-full-featured-layer-7-capable-application-delivery-controller-windows
[2]: http://kemptechnologies.com/load-balancer-for-azure
[3]: http://msdn.microsoft.com/en-us/library/system.net.credentialcache.defaultcredentials(v=vs.110).aspx
[4]: http://msdn.microsoft.com/en-us/library/System.Net.CredentialCache(v=vs.110).aspx