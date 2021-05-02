---
title: Connecting to Hyper-V virtual machines with PowerShell
author: Shay Levy
type: post
date: 2012-10-11T18:00:30+00:00
url: /2012/10/11/connecting-to-hyper-v-virtual-machines-with-powershell/
categories:
  - Hyper-V
tags:
  - Hyper-V

---
I&#8217;ve been using Windows 8 since its pre-release versions and one of the features I like most in it is &#8211; Hyper-V. Until Windows 8, Hyper-V was available only as a Server technology. Starting with Windows 8, it is now available out of the box!

In a nutshell, Hyper-V lets you run more than one 32-bit or 64-bit operating system at the same time on the same computer. In Windows 8, this technology is now built into the non-server version of Windows. Client Hyper-V provides the same virtualization capabilities as Hyper-V in Windows Server 2012.

For us, IT Pros, having a virtualized environment nowadays is almost a necessity as many of us need to run multiple operating systems, and maintain multiple test environments.  Hyper-V is an optional feature. You must first enable it.

**Note**: Client Hyper-V is supported only on 64-bit versions of Windows 8, Pro or Enterprise, having at least 4 GB of RAM,  and requires modern Intel and AMD microprocessors that include Second Level Address Translation (SLAT) technologies.

### Enable Hyper-V using the UI

On the Control Panel, click Programs, and then click Programs and Features, click &#8216;Turn Windows features on or off&#8217;, tick the Hyper-V checkbox , click OK, and then click Close.

![](/images/Hyper-V_Add.png)

#### Enable Client Hyper-V using Windows PowerShell

Open Windows PowerShell and type the following command (which is a part of the Dism module):

<pre class="brush: powershell; title: ; notranslate" title="">Enable-WindowsOptionalFeature –FeatureName Microsoft-Hyper-V -All
</pre>

To complete installation you need to restart your computer. After restarting the computer, you can use Hyper-V Manager or Windows PowerShell to create and manage virtual machines. You can also use Virtual Machine Connection (vmc) to connect to virtual machines locally or remotely.

### Connecting to Virtual Machines

You can connect to your VMs by using the Virtual Machine Connection utility. Notice the warning at the bottom of the dialog. I launched it without administrative privileges and as a result I won&#8217;t be able to connect to VMs. Make sure you right click it and open it as an administrator.

![](/images/Hyper-V_VMC.png)

That warning is also valid when you use the Hyper-V cmdlets, and the error you get back is misleading; it doesn&#8217;t mention required permissions whatsoever. For example, if you run the Get-VM from a non-elevated console: 

```
PS> Get-VM -VMName DC12

Get-VM :The parameter is not valid. Hyper-V was unable to find a virtual machine with name DC12.
At line:1 char:1

+ Get-VM -VMName DC12
+ CategoryInfo          : InvalidArgument: (DC12:String) [Get-VM], VirtualizationInvalidArgumentException
+ FullyQualifiedErrorId : InvalidParameter,Microsoft.HyperV.PowerShell.Commands.GetVMCommand
```

Alternatively, you can use the Hyper-V Manager. Just right click a VM and then &#8216;Connect&#8217;:

![](/images/Hyper-V_Manager.png)

### The Hyper-V module for Windows PowerShell

The Hyper-V module for Windows PowerShell includes more than 160 cmdlets to manage Hyper-V virtual machines. The cmdlets  provide an easy way to automate Hyper-V management tasks. Here&#8217;s a partial list of the module commands:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Command -Module Hyper-V
CommandType     Name                                               ModuleName
-----------     ----                                               ----------
Cmdlet          Add-VMDvdDrive                                     hyper-v
Cmdlet          Add-VMFibreChannelHba                              hyper-v
Cmdlet          Add-VMHardDiskDrive                                hyper-v
Cmdlet          Add-VMMigrationNetwork                             hyper-v
Cmdlet          Add-VMNetworkAdapter                               hyper-v
Cmdlet          Add-VMNetworkAdapterAcl                            hyper-v
Cmdlet          Add-VMRemoteFx3dVideoAdapter                       hyper-v
Cmdlet          Add-VMScsiController                               hyper-v
Cmdlet          Add-VMStoragePath                                  hyper-v
Cmdlet          Add-VMSwitch                                       hyper-v
Cmdlet          Add-VMSwitchExtensionPortFeature                   hyper-v
Cmdlet          Add-VMSwitchExtensionSwitchFeature                 hyper-v
Cmdlet          Checkpoint-VM                                      hyper-v
Cmdlet          Compare-VM                                         hyper-v
Cmdlet          Complete-VMFailover                                hyper-v
Cmdlet          Connect-VMNetworkAdapter                           hyper-v
Cmdlet          Connect-VMSan                                      hyper-v
Cmdlet          Convert-VHD                                        hyper-v
Cmdlet          Disable-VMEventing                                 hyper-v
(...)
</pre>

The online reference of the Hyper-V cmdlets in Windows PowerShell can be found here: http://technet.microsoft.com/library/hh848559.aspx

One of the first things I&#8217;ve tried when working with Hyper-V was to connect to a VM via PowerShell but I couldn&#8217;t find a Connect-VM cmdlet!

Under the hood, Hyper-V Manager is using the vmc utility (vmconnect.exe) to connect to a VM.

![](/images/Hyper-V_VMCHelp.png)

Based on the parameters of vmc, I created the [Connect-VM][1] advanced function. With Connect-VM, you can connect to a VM, locally or on a remote machine  and even start it, if it isn&#8217;t already running. You can pipe VMs you get with Get-VM to it or just specify the VM name(s) or GUIDs. And with a few modifications you can also extend it to launch Remote Desktop instead of Hyper-V.

    #requires -Version 3.0
    function Connect-VM
    {
      [CmdletBinding(DefaultParameterSetName='name')]
    
      param(
        [Parameter(ParameterSetName='name')]
        [Alias('cn')]
        [System.String[]]$ComputerName=$env:COMPUTERNAME,
    [Parameter(Position=0,
        Mandatory,ValueFromPipelineByPropertyName,
        ValueFromPipeline,ParameterSetName='name')]
    [Alias('VMName')]
    [System.String]$Name,
    
    [Parameter(Position=0,
        Mandatory,ValueFromPipelineByPropertyName,
        ValueFromPipeline,ParameterSetName='id')]
    [Alias('VMId','Guid')]
    [System.Guid]$Id,
    
    [Parameter(Position=0,Mandatory,
        ValueFromPipeline,ParameterSetName='inputObject')]
    [Microsoft.HyperV.PowerShell.VirtualMachine]$InputObject,
    
    [switch]$StartVM
      )
    
      begin
      {
        Write-Verbose "Initializing InstanceCount, InstanceCount = 0"
        $InstanceCount=0
      }
    
      process
      {
        try
        {
          foreach($computer in $ComputerName)
          {
            Write-Verbose "ParameterSetName is '$($PSCmdlet.ParameterSetName)'"
    if($PSCmdlet.ParameterSetName -eq 'name')
        {
              # Get the VM by Id if Name can convert to a guid
              if($Name -as [guid])
              {
    		Write-Verbose "Incoming value can cast to guid"
    		$vm = Get-VM -Id $Name -ErrorAction SilentlyContinue
              }
              else
              {
    		$vm = Get-VM -Name $Name -ErrorAction SilentlyContinue
              }
        }
        elseif($PSCmdlet.ParameterSetName -eq 'id')
        {
              $vm = Get-VM -Id $Id -ErrorAction SilentlyContinue
        }
        else
        {
          $vm = $InputObject
        }
    
        if($vm)
        {
          Write-Verbose "Executing 'vmconnect.exe $computer $($vm.Name) -G $($vm.Id) -C $InstanceCount'"
          vmconnect.exe $computer $vm.Name -G $vm.Id -C $InstanceCount
        }
        else
        {
          Write-Verbose "Cannot find vm: '$Name'"
        }
    
        if($StartVM -and $vm)
        {
          if($vm.State -eq 'off')
          {
            Write-Verbose "StartVM was specified and VM state is 'off'. Starting VM '$($vm.Name)'"
            Start-VM -VM $vm
          }
          else
          {
            Write-Verbose "Starting VM '$($vm.Name)'. Skipping, VM is not not in 'off' state."
          }
        }
    
        $InstanceCount+=1
        Write-Verbose "InstanceCount = $InstanceCount"
      }
    }
    catch
    {
      Write-Error $_
    }
    }
    }
### Usage examples

```
# Connect to VM and start it
PS> Connect-VM -VMName DC12 -StartVM
```

![](/images/Hyper-V_ConnectVM1.png)

2. Connecting to multiple VMs, when multiple connections are made the windows are cascaded. Without it they would sit on top of each other at the same position making it hard to switch from one to another.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-VM | Connect-VM
</pre>

![](/images/Hyper-V_ConnectVM2.png)

3. Connect using a VM&#8217;s Guid

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Connect-VM -Id 34d22d46-15c7-4226-9fa2-04f6c90a5a9a -StartVM
</pre>

![](/images/Hyper-V_ConnectVM3.png)

You can download the function [HERE][1]

[1]: /images/Connect-VM.zip