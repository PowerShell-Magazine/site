---
title: Managing PCIe SSD devices in VMware ESXi by using PowerCLI cmdlets
author: Krishna Prasad K
type: post
date: 2014-08-19T16:00:53+00:00
url: /2014/08/19/managing-pcie-ssd-devices-in-vmware-esxi-by-using-powercli-cmdlets/
categories:
  - Dell
  - VMware
tags:
  - Dell
  - VMware

---
PCIe Solid State Disk (SSD) devices provides higher IOPS and great sequential read/write speeds. These devices need to be configured before they can be used for Virtual Machine (VM) storage. In this article, we will see how this can be done using PowerCLI cmdlets.

Typically, there are currently three use cases for PCIe SSDs in VMware ESXi.

  1. Configuring PCIe SSD as VMFS datastore
  2. Configuring PCIe SSD as Host Swap Cache
  3. Creating vFlash resource out of the PCIe SSD

Let us dive into the PowerCLI commands that are needed for automating each of these use cases.

**Note**: The content of this article is a part of a technical paper published by Dell Hypervisor Engineering team. This paper can be [downloaded from Dell Tech Center][1].

### Configuring PCIe SSD as VMFS datastore

The first step in automation using PowerCLI is to connect to the ESXi server. This is done using the _Connect-ViServer_ cmdlet.

```
Connect-VIServer dhcp-10-10-5-111.helab.bdc
```

![](/images/Connect-VIServer.png)

Once the connection is established, we can use the _Get-ScsiLun_ cmdlet to see all available LUNs on the system.

![](/images/Get-ScsiLun.png)

The highlighted device name in the above screen shot shows the PCIe SSD device. We can grab this object alone using the _-CanonicalName_ parameter of the _Get-ScsiLun_ cmdlet.

```
$PCIeSSD = Get-ScsiLun -CanonicalName t10*
```


Once we have this object, we can use the New-DataStore cmdlet to add the PCIe SSD to the VMFS datastore.

![](/images/New-DataStore.png)

The _–Path_ parameter accepts the full path of the device and _–FileSystemVersion_ designates the VMFS filesystem version in which the device has to be formatted. With VMFS filesystem created on the PCIe SSD device, it’s now possible to host the virtual machines on this datastore.

### Configuring PCIe SSD as host swap cache

In this use case, let us see how we can configure a portion of the PCIe SSD as host swap cache in ESXi. What you see below is a script that is slightly modified from the original version posted by [Joe Keegan][2].

```
# Here PCIeSSD is the VMFS datastore created in earlier step
$DataStore = Get-Datastore -Name "PCIeSSD"
$HostCacheConfigurationSpec = New-Object VMware.Vim.HostCacheConfigurationSpec
$HostCacheConfigurationSpec.datastore = New-Object VMware.Vim.ManagedObjectReference
$HostCacheConfigurationSpec.datastore.type = "Datastore"
$HostCacheConfigurationSpec.datastore.Value = ($DataStore.id).substring(10, 35)
```


The _Value_ field for datastore object should be the ID of the VMFS datastore we created in the earlier step.

The size of the host swap cache need to be specified in MB and should be less than or equal to the free space available in the PCIe SSD data store.

<pre class="brush: powershell; title: ; notranslate" title="">$HostCacheConfigurationSpec.swapSize = 2048
</pre>

Once we have the required configuration, we can execute the [_ConfigureHostCache_Task_][3] method. For this, we first need to derive the Host Cache Configuration Manager ID. This can be done using the following snippet of code.

```
$VMHost = Get-VMHost -Name Server01
$HostCacheConfigurationManager_ID  = $VMHost.ExtensionData.ConfigManager
$HostCacheConfigurationManager = Get-View -Id $HostCacheConfigurationManager_ID
$HostCacheConfigurationManager.ConfigureHostCache_Task($HostCacheConfigurationSpec) | Out-Null
```


This completes the configuration of a portion of the PCIe SSD as the host swap cache.

### PCIe SSD as a vFlash Device

For configuring PCIe SSD as a vFlash device, we will need the vSphere Flash Read Cache (vFRC) cmdlets.

We can enable the vFRC cmdlets by installing the [VMware.VimAutomation.Extensions][4]. Once these extensions are installed, the vFRC cmdlets can be imported by running the following command.

<pre class="brush: powershell; title: ; notranslate" title="">Import-Module VMware.VimAutomation.Extensions
</pre>

The following snippet retrieves the SSD device object. This can later be used to add the SSD as a vFlash resource.

```
$VMHost1 = Get-VMHost 10.10.5.111
$ssd = $VMHost1 | Get-VMHostDisk | Where { $_.CanonicalName -like "t10*" }
```

The _Set-VMHostVFlashConfiguration_ cmdlet can be used to complete the configuration of the SSD as a vFlash resource.

```
$VMHost1 | Get-VMHostVFlashConfiguration | Set-VMHostVFlashConfiguration -AddDevice $ssd
```

![](/images/Set-VFlash.png)

The PCIe SSD is now added as a vFlash resource. We can set up reservation for HostSwapCache to 20GB by using the following command:

```
Set-VMHostVFlashConfiguration -VFlashConfiguration t10* -SwapCacheReservationGB 20
```


Once the vFlash resource and HostSwapCache are set up, vFRC can be enabled for VMs. Make a note that in order to leverage vFRC, the VM hardware needs to be upgraded to version 10 (VMX-10).

```
$VM = Get-VM "rhel6.3"
$disk = Get-HardDisk -VM $VM
$conf = Get-HardDiskVFlashConfiguration -HardDisk $disk
Set-HardDiskVFlashConfiguration -VFlashConfiguration $conf -CacheSizeGB 10
```

![](/images/SetvFRC.png)

For more detailed information on these use cases, refer to the Dell technical paper [at http://en.community.dell.com/techcenter/extras/m/white_papers/20439058.aspx][1]. This white paper was authored by Dell team members Karan Singh Gandhi, Krishna Prasad, and Avinash Bendigeri.

**_Karan Singh Gandhi_** works at Dell Inc. Bangalore in virtualization domain. Love working on PowerShell scripting and like to contribute more In this domain.

_**Avinash Bendigeri**_ works at Dell Inc. Bangalore in virtualization domain. Expertise in virtualization domain and python scripting. Recently started with PowerShell scripting. Would like to contribute more and more.

[1]: http://en.community.dell.com/techcenter/extras/m/white_papers/20439058.aspx
[2]: http://infrastructureadventures.com/tag/esxi/
[3]: http://pubs.vmware.com/vsphere-55/index.jsp#com.vmware.wssdk.apiref.doc/vim.host.CacheConfigurationManager.html
[4]: http://labs.vmware.com/flings/powercli-extensions