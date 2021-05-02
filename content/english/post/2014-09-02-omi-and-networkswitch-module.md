---
title: OMI and NetworkSwitch module
author: Bartek Bielawski
type: post
date: 2014-09-02T16:00:12+00:00
url: /2014/09/02/omi-and-networkswitch-module/
categories:
  - OMI
tags:
  - OMI

---
One of the first tasks I used PowerShell for was related to Windows Management Instrumentation (WMI). I had heard about WMI before I started my adventure with PowerShell, but never tried it myself. Due to the ease of using WMI in PowerShell, I could get answers from any Windows box to the questions I never dared to ask before. Then a few years passed&#8230;

In 2012 <a href="http://blogs.technet.com/b/windowsserver/archive/2012/06/28/open-management-infrastructure.aspx" target="_blank">Microsoft announced Open Management Infrastructure (OMI)</a>, an open source implementation of WMI that could run on almost any device.

When I&#8217;ve started playing with OMI I decided to use Linux as my guinea pig, because it was easier to get a Linux box up and running at relatively low cost. I just had to spin up an extra VM and install Linux on it. Then download and install the OMI binaries and last but not least &#8211; create an OMI provider that was doing something useful. You can find all details about this journey <a href="http://becomelotr.wordpress.com/2013/06/20/managing-linux-via-omi-roadmap/" target="_blank">on my blog</a>.

One thing I haven&#8217;t considered back then was using virtualization to &#8216;mimic&#8217; network gears that I could not get otherwise. But when Microsoft shipped a preview of <a href="http://blogs.technet.com/b/windowsserver/archive/2014/04/03/windows-management-framework-v5-preview.aspx" target="_blank">PowerShell 5.0 with a NetworkSwitch module</a>, I started looking for a way to play with this module without getting network team to borrow me one of their &#8216;toys&#8217;. And that&#8217;s how vEOS came about: it&#8217;s a virtual platform provided by Arista that behaves like one of Arista own devices. It has beta support for Hyper-V (my virtualization platform of choice since I migrated to Windows 8). In this article I would like to describe my path from idea to the point when I could use NetworkSwitch module to manage my own (though virtual) Arista switch.

### Building a demo machine

The first thing you have to do is to [download][1] a boot image and a VMDK file (requires login). The next thing I did was converting the VMDK file to a format that my Hyper-V host could understand: VHD. There are several ways to do it, and I decided to use a tool provided by Microsoft:

<a href="http://msdn.microsoft.com/en-us/library/hh967435.aspx" target="_blank">Microsoft Virtual Machine Converter Solution Accelerator</a>. This tool comes with two command line utilities. One I&#8217;ve used, _MVDC.exe_, simply converts one virtual disk format to the other:

```
New-Alias -Name MVDC -Value 'C:\Program Files (x86)\Microsoft Virtual Machine Converter Solution Accelerator\MVDC.exe'
Push-Location E:\VMs\VHDx
MVDC .\vEOS-4.13.7M.vmdk vEOS.vhd
```


Once the file is converted we can move on to creating and configuring the virtual machine. We have to remember that vEOS can only be used with legacy network interfaces and that it supports up to four of them. We also have to make sure that the VM will boot from the image provided by Arista:

```
New-VM -Name EOS -BootDevice CD -VHDPath .\vEOS.vhd -MemoryStartupBytes 1GB -Generation 1

# Need legacy adapters...
Get-VMNetworkAdapter -VMName EOS | Remove-VMNetworkAdapter
$Adapters = @{
    VMName = 'EOS'
    SwitchName = 'Internal'
    IsLegacy = $true
}
Add-VMNetworkAdapter @Adapters -Name Mgmt
Add-VMNetworkAdapter @Adapters -Name Eth1
Add-VMNetworkAdapter @Adapters -Name Eth2
Add-VMNetworkAdapter @Adapters -Name Eth3

Set-VMDvdDrive -VMName EOS -Path E:\VMs\ISO\Aboot-veos-2.0.8.iso
Start-VM EOS
```

And that&#8217;s it from the Hyper-V perspective. We now move on to our switch where we have to configure a management interface (so that we can connect using TCP/IP) and enable/configure OMI.

### Switch configuration

The initial login doesn&#8217;t require any password. We specify the default username (admin) and once we are in&#8211;we turn on privileged commands using the _enable_ command. To configure management interface we need to get to the correct context and run _ip_:

```
configure
interface management 1
ip address 192.168.200.66/24
exit
```


In order to enable and configure OMI we need to browse to a different context and enable it. We will also change the HTTP/HTTPS ports to the ones that we know and love. Mind that even though when you check the configuration ports they show up as 7778 and 7779, OMI listens on 5985/5986 anyway, so we are doing it to avoid confusion:

```
management cim-provider
no shutdown
http 5985
https 5986
exit
```


Another change is required&#8211;because we can&#8217;t use the CIM cmdlets with OMI if the password is blank we need to configure a password for the root account:

```
aaa root secret P@$$w0rd
```


And the final step&#8211;we need to modify the access list to allow WSMAN over HTTP and HTTPS traffic:

```
ip access-list OMI
10 permit tcp 192.168.200.0/24 any eq 5985 5986

# ... other permits that we want... e.g.
20 permit tcp any any eq ssh
exit

control-plane
ip access-group OMI in
exit
```

Our switch is now configured and it is ready to welcome our first CIM session.

### Managing the switch using CIM cmdlets

We can connect to the switch using both HTTP and HTTPS, depending on our needs. If we want to use SSL we will have to disable any checks, because the certificate on our switch is not trusted on our Windows machine:

```
$rootPassword = ConvertTo-SecureString -AsPlainText -Force -String 'P@$$w0rd'
$switchParams = @{
    ComputerName = '192.168.200.66'
    Credential = New-Object pscredential -ArgumentList root, $rootPassword
    Authentication = 'Basic'
    SessionOption = New-CimSessionOption -UseSsl -SkipCACheck -SkipCNCheck -SkipRevocationCheck
}
$cimSwitch = New-CimSession @switchParams
```


Our CIM session is created, so now we can start managing the switch with CIM cmdlets. Unfortunately, we won&#8217;t get any discoverability features that we get with WMI:

<pre class="brush: powershell; title: ; notranslate" title="">Get-CimClass -CimSession $cimSwitch -ClassName CIM_*
The WinRM client received an HTTP server error status (500), but the remote service did not include any other information about the cause of the failure. (raised by: Get-CimClass)
</pre>

If we are not running PowerShell 5.0 than the best option is to look for documentation of OMI/CIM provider implemented on Arista switches, or make some intelligent guesses:

```
Get-CimClass -CimSession $cimSwitch -ClassName Arista_EthernetPort
   NameSpace: root/cimv2

CimClassName                        CimClassMethods      CimClassProperties
------------                        ---------------      ------------------
Arista_EthernetPort                 {RequestStateChan... {InstanceID, Caption, Description, ElementName...}
```

We&#8217;ve found the class, time to take a look at its instances:

```
Get-CimInstance -CimSession $cimSwitch -ClassName Arista_EthernetPort |
    Format-Table -AutoSize DeviceId, OperationalStatus, EnabledState

DeviceId    OperationalStatus EnabledState
--------    ----------------- ------------
Ethernet1   {2}                          2
Ethernet2   {2}                          2
Ethernet3   {2}                          2
Management1 {0}                          2
```

### Managing the switch using the NetworkSwitch module

PowerShell 5.0 preview comes with a NetworkSwitch module that takes away necessity of knowing class names. Instead, we can just load the module, redirect any command in it to our switch, and take advantage of work done by Microsoft:

```
Import-Module -Name NetworkSwitch
$PSDefaultParameterValues.'*-NetworkSwitch*:CimSession' = $cimSwitch

Get-NetworkSwitchEthernetPort -DeviceId Management1 |
    Set-NetworkSwitchPortProperty -Property @{
        Description = 'This is just a test'
    }
```

And sure enough: our changes are reflected on the remote end:

![](/images/Arista-Interface-Confiured-With-OMI-And-NetworkSwitch-PowerShell-Module.png)

There are many other elements we can manage using this module:

```
Get-Command -Module NetworkSwitch |
    Group-Object Noun |
    Format-Table Name, @{
        Name = 'Verbs'
        Expression = {
            $_.Group.Verb -join ', '
        }
    } -AutoSize

Name                               Verbs
----                               -----
NetworkSwitchEthernetPort          Disable, Enable, Get
NetworkSwitchFeature               Disable, Enable, Get
NetworkSwitchVlan                  Disable, Enable, Get, New, Remove
NetworkSwitchGlobalData            Get
NetworkSwitchEthernetPortIPAddress Remove, Set
NetworkSwitchConfiguration         Restore, Save
NetworkSwitchPortMode              Set
NetworkSwitchPortProperty          Set
NetworkSwitchVlanProperty          Set
```

As you can see we can configure IP addresses on ports, enable or disable switch features, add and configure VLANS and more. All of this is possible with an OMI provider on the remote end and the NetworkSwitch module on the local computer.

[1]: https://www.arista.com/en/support/software-download