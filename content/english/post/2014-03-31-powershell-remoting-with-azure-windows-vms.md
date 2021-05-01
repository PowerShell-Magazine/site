---
title: PowerShell Remoting with Azure Windows VMs
author: Ravikanth C
type: post
date: 2014-03-31T16:00:07+00:00
url: /2014/03/31/powershell-remoting-with-azure-windows-vms/
categories:
  - How To
  - Azure
tags:
  - Azure
  - How To

---
These days, for all my demos at various events, I have been using [Azure VM][1]s extensively. If you have worked on creating these VMs using [Azure PowerShell cmdlets][2], you will notice that they create a PowerShell remoting endpoint for a VM. However, this endpoint requires SSL and therefore certificates.

Let us see this step by step.

### Creating VM

I assume that you already have a cloud service. We will create a VM from the latest Windows Server 2012 R2 image and add it to the existing service.

```
Get-AzureVMImage |
Where-Object { $_.Label -like "*SERVER 2012 R2*" } |
Sort -Property PublishedDate -Descending |
Select -Property ImageName,Label
```

The above code snippet gives us the Windows Server 2012 R2 OS images.

![](/images/azurevm1.png)

```
$VMName = "WSR2-3"
$UserName = "Ravikanth"
$Password = "Y0urPassW0Rd!"
$VMLocation = "East US"
$VMImage = "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201403.01-en.us-127GB.vhd"
$ServiceName = "WinVMs"
New-AzureVMConfig -Name $VMName -InstanceSize "Small" -ImageName $VMImage |
Add-AzureProvisioningConfig -Windows -AdminUsername $UserName -Password $Password |
New-AzureVM -ServiceName $ServiceName -Location $VMLocation -WaitForBoot
```

In the above code snippet, we first get the default subscription and set the storage account to the default subscription. If you have multiple subscriptions, you need to set the right subscription storage account here. Also, ensure that <em>$VMLocation</em> is set to the same location as the storage account.

### Listing Endpoints

We can see the default endpoints created for this VM using the <em>Get-AzureEndpoint</em> cmdlet.

```
Get-AzureVM -ServiceName "WinVMs" -Name "WSR2-3" |
Get-AzureEndpoint |
Select Name, LocalPort, Port
```

![](/images/azurevm2.png)

As you see, the public port (Port=64434 in the above example) for PowerShell remoting is not the default remoting port. Also, as you might be aware, the DNS name for any VM is DNS name of the cloud service. So, we can use that DNS name and public port to connect to the remoting endpoint in the VM.

```
Enter-PSSession -ConnectionUri https://winvms.cloudapp.net:64434 -Credential (Get-Credential -UserName Ravikanth -Message "Password")
```

This results in an error because the certificate from the cloud service isn&#8217;t trusted on the local system. We need to install that certificate.

### Installing Cloud Service Certificate

```
$WinRMCertificateThumbprint = (Get-AzureVM -ServiceName "WinVMs" -Name "WSR2-3" | Select-Object -ExpandProperty VM).DefaultWinRMCertificateThumbprint
(Get-AzureCertificate -ServiceName "WinVMs" -Thumbprint $WinRMCertificateThumbprint -ThumbprintAlgorithm SHA1).Data | Out-File "${env:TEMP}\CloudService.tmp"

$X509Object = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 "$env:TEMP\CloudService.tmp"
$X509Store = New-Object System.Security.Cryptography.X509Certificates.X509Store "Root", "LocalMachine"
$X509Store.Open([System.Security.Cryptography.X509Certificates.OpenFlags]::ReadWrite)
$X509Store.Add($X509Object)
$X509Store.Close()

Remove-Item "$env:TEMP\CloudService.tmp"
```

The above code snippet downloads the certificate from the cloud service and installs it on the local system. Once this step is complete, we can use any of the remoting cmdlets to remote into the Azure VM.


[1]: http://www.windowsazure.com/en-us/solutions/infrastructure/
[2]: http://msdn.microsoft.com/en-us/library/windowsazure/jj156055.aspx