---
title: How to use WDS to PXE Boot a Nano Server VHD with PowerShell
author: Emin Atac
type: post
date: 2015-06-29T18:58:10+00:00
url: /2015/06/29/how-to-use-wds-to-pxe-boot-a-nano-server-vhd-with-powershell-2/
views:
  - 21308
post_views_count:
  - 2642
categories:
  - How To
  - PowerShell DSC
tags:
  - How To
  - PowerShell DSC

---
The goal of this article is to provide a more detailed step by step way of achieving what was recently presented in [How to use WDS to PXE Boot a Nano Server VHD][1] post on Nano Server team blog.

Let’s have a look at the requirements to go through these steps. I&#8217;ll actually assume that:

  * You should have already installed Hyper-V and have enough compute and storage resources available on your host.
  * You&#8217;ve downloaded an [ISO image of the Windows Server 2016 Technical Preview 2][2] (build 10074 from May 2015).

My test laptop is a Windows 10 Enterprise insider preview installed (build 10130).

Let&#8217;s quickly summarize what steps are required to be able to deploy a Nano.vhd from WDS after a PXE boot:

  1. Create an Internal Hyper-V switch so that computers attached to it won&#8217;t impact your environment but can communicate with each other.
  2. Provision a Domain Controller virtual machine from the ISO file
  3. Configure the DC by: 
      * Setting a static IP Address and assign it to the adapter bound to the Internal Hyper-V switch
      * Installing the required Windows features and roles: Active Directory Domain Services, DHCP and DNS Server, and Windows Deployment Services (a.k.a. WDS)
      * Configuring these features to be able to PXE boot from the boot.wim file available on the original ISO file and delivering the prepared Nano.vhd file as an image to be installed
  4. Provision a new empty VM, PXE boot and install the Nano.vhd

#### **Step 1**

Prepare the Internal switch on Hyper-V and assign it a static IP address.

```powershell
# Create an internal switch on Hyper-V
($VMswitch = New-VMSwitch -Name "Internal-Test" -SwitchType Internal)
```

![](/images/wds1.jpg)

```powershell
# Set a static IP address on Hyper-V switch
Get-NetAdapter |
   Where Name -eq "vEthernet ($($VMswitch.Name))" |
   Where InterfaceDescription -match "Hyper-V\sVirtual\sEthernet Adapter" |
   New-NetIPAddress -IPAddress 10.0.0.1 -PrefixLength 24
```


![](/images/wds2.jpg)

I&#8217;ve downloaded the required ISO into the Downloads folder of the logged on user. I&#8217;ll store it as a variable to be able to use it later on.

```powershell
# Set the path to ISO
$iso  = $ExecutionContext.SessionState.Path.
GetUnresolvedProviderPathFromPSPath('~/downloads/en_windows_server_technical_preview_2_x64_dvd_6687981.iso')
```


NB: The path is expanded and there&#8217;s no check whether the file exists or not.

Let&#8217;s make sure that the integrity of the file is fine.

```powershell
# Integrity check of ISO file
if (
 (Get-FileHash -Path $ISO -Algorithm SHA256 -ErrorAction SilentlyContinue).Hash -eq
 'D8D841393F661E30D448D2E6CBCEE20A94D9A57A94695B64EE76CA6B0910F849'
){
 Write-Information -Msg "Got the correct Technical Preview 2 ISO file" -InfA 2
} else {
 Write-Warning -Message "Don't have the correct ISO of the Technical Preview 2"
 break
}
```


The _Msg_ parameter name is the alias for _MessageData_ and _InfA_ is the alias for _InformationAction_.

![](/images/wds3.jpg)

Let&#8217;s also prepare a Nano.vhd file that will be copied to the DC1 disk and proposed as an installation image by the PXE server.

To create this Nano.vhd file, you can either follow the [Getting Started with Nano Server][3] guide, or you can use the new [PowerShell Script to build your Nano Server Image][4]. I&#8217;ll use the latter.

That said, first download the required scripts:

```powershell
# Download scripts requirements

@(
 @{
 URI = 'http://blogs.technet.com/cfs-filesystemfile.ashx/__key/telligent-evolution-components-attachments/01-10474-00-00-03-65-09-88/NanoServer.ps1';
 SHA1 = '27C00A02B49F3565783051B95D82498F17F74D57' ;
 },
 @{
 URI = 'https://gallery.technet.microsoft.com/scriptcenter/Convert-WindowsImageps1-0fe23a8f/file/59237/7/Convert-WindowsImage.ps1';
 SHA1 = '4B91A8ED09BD1E9DB5C63C8F63BB2BA83567917C' ;
 }
) | ForEach-Object -Process {
     $f = ([system.uri]$($_.URI)).Segments[-1] ;
     $o = (Join-Path ~/Downloads -ChildPath $f) ;
     if(-not((Get-FileHash -Path $o -Algorithm SHA1 -ErrorAction SilentlyContinue).Hash -eq $_.SHA1)) {
         try {
            $null = Invoke-WebRequest -Uri $($_.URI) -OutFile $o -ErrorAction Stop
            Unblock-File $o -ErrorAction Stop
            Write-Information -Msg "Successfully downloaded the correct version of $f file" -InfA 2
         } catch {
            Write-Warning -Message "There was a problem downloading the $f file"
         }
     } else {
         Write-Information -Msg "Successfully found the correct version of $f file" -InfA 2
     }
}


# Dot-sourcing functions inside a script
. ~/Downloads/Convert-WindowsImage.ps1
# Fix hard-coded path in the script
(Get-Content -Path ~/Downloads/NanoServer.ps1 -ReadCount 1 -ErrorAction Stop) -replace
[regex]::Escape('. .\Convert-WindowsImage.ps1'),"" |
Set-Content -Path ~/Downloads/NanoServer.ps1 -Encoding UTF8 -ErrorAction Stop

# Load the modified version
. ~/Downloads/NanoServer.ps1
```

![](/images/wds4.jpg)

**Step 2**

To provision the Domain Controller, I&#8217;ll use 3 techniques: the post-installation script setupcomplete.cmd that runs at the end of the specialize phase, the unattend.xml file, and PowerShell Desired Configuration to achieve the equivalent of a DCPromo. The main idea here is to move all artifacts (DSC modules, boot.wim, the prepared nano.vhd file…) required for configuring both the Domain Controller and WDS (Windows Deployment Services) into the VHD of the domain controller.

```powershell
# Mount ISO
Mount-DiskImage -ImagePath $iso -StorageType ISO -Access ReadOnly -PassThru
$dl = (Get-DiskImage -ImagePath $iso | Get-Volume).DriveLetter

# Define VM Name
$VM = "DC1-test"

# Set parent VHD
$ServerVHD = (Join-Path -Path ((Get-VMHost).VirtualHardDiskPath) -ChildPath "$VM.vhd")

# Create parent VHD
# Convert the WIM file to a VHD using the loaded Convert-WindowsImage function

if (-not(Test-Path -Path $ServerVHD -PathType Leaf)) {
    Convert-WindowsImage -Sourcepath  "$($dl):\sources\install.wim" `
    -VHD $ServerVHD `
    -VHDformat VHD -Edition "Windows Server 2012 R2 SERVERSTANDARD" `
    -VHDPartitionStyle MBR -Verbose:$true
}
Write-Information -Msg "Created parent VHD: size = $('{0:N2} GB' -f ((Get-Item $ServerVHD).Length/1GB))" -InfA 2
```

![](/images/wds6.jpg)

```powershell
# Create child VHD
$cvp = (Join-Path -Path ((Get-VMHost).VirtualHardDiskPath) -ChildPath "$VM-child.vhd")
$childVHD = New-VHD -Path $cvp -ParentPath $ServerVHD -Differencing

# Create a VM Gen 1
New-VM -Name $VM -MemoryStartupBytes 2048MB -NoVHD -SwitchName Internal-Test -Generation 1

# Attach disk
Get-VM $VM | Add-VMHardDiskDrive -Path $childVHD.Path

# Increase processor count for DC
Get-VM $VM | Set-VMProcessor -Count 2

# Mount the VHD
$cm = Mount-VHD -Path $childVHD.Path -Passthru
$cml = (Get-Disk $cm.DiskNumber | Get-Partition | Where DriveLetter | Select -First 1).DriveLetter

# Prepare a Nano VHD with the new script
$bdir = Join-Path (Split-Path $iso -Parent) -ChildPath "Base"
if (-not(Test-Path -Path $bdir -PathType Container)) {
    mkdir $bdir
}

$admincred = Get-Credential -Message 'Admin password of your Nano image' -UserName 'Administrator'
$nnHT = @{
   ComputerName = 'Nano-PXE' ;
   MediaPath = "$($dl):\" ;
   BasePath = $bdir ;              # The location for the copy of the source media
   TargetPath = "$bdir\Target" ;   # The location of the final, modified image
   Language = 'en-US' ;            # The language locale of the packages
   GuestDrivers = $true ;          # Add the Guest Drivers package (enables integration of Nano Server with Hyper-V when running as a guest).
   EnableIPDisplayOnBoot = $true ; # Configures the image to show the output of 'ipconfig' on every boot
   AdministratorPassword =  $admincred.Password ;
}

New-NanoServerImage @nnHT
```

![](/images/wds5.jpg)

```powershell
# Setupcomplete.cmd file

$s = @'
@echo off
:: Define a static IP for the DC
netsh int ip set address name="Ethernet" source=static address=10.0.0.10/24 gateway=10.0.0.1
:: Configure the DNS client
netsh dns set dnsservers name="Ethernet" source=static address=10.0.0.10 validate=no
'@

mkdir "$($cml):\Windows\Setup\Scripts"
$s | Out-File -FilePath  "$($cml):\Windows\Setup\Scripts\setupcomplete.cmd" -Encoding ASCII

# Unattend.xml

$unattendDC1 = @'
<xml version="1.0" encoding="utf-8">
<unattend xmlns="urn:schemas-microsoft-com:unattend">
<settings pass="oobeSystem">
<component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<UserAccounts>
<AdministratorPassword>
<Value>UABAAHMAcwB3ADAAcgBkAEEAZABtAGkAbgBpAHMAdAByAGEAdABvAHIAUABhAHMAcwB3AG8AcgBkAA==</Value>
<PlainText&gt;false&lt;/PlainText>
</AdministratorPassword>
</UserAccounts>
<RegisteredOwner>Tuva user</RegisteredOwner>
<RegisteredOrganization&gt;NanoRocks&lt;/RegisteredOrganization>
</component>
<component name="Microsoft-Windows-International-Core" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<SystemLocale&gt;en-US&lt;/SystemLocale>
<InputLocale&gt;0409:0000040c&lt;/InputLocale>
<UILanguage&gt;en-US&lt;/UILanguage>
<UserLocale&gt;en-US&lt;/UserLocale>
</component>
</settings>
<settings pass="specialize">
<component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<ComputerName&gt;DC1-test&lt;/ComputerName>
</component>
<component name="Microsoft-Windows-DNS-Client" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<DNSSuffixSearchOrder>
<DomainName wcm:action="add" wcm:keyValue="1"&gt;10.0.0.10&lt;/DomainName>
</DNSSuffixSearchOrder>
</component>
<component name="Microsoft-Windows-UnattendedJoin" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<Identification>
<JoinWorkgroup>test.local&lt;/JoinWorkgroup>
</Identification>
</component>
</settings>
<cpi:offlineImage cpi:source="wim:c:/iso.6687981/sources/install.wim#Windows Server 2012 R2 SERVERSTANDARDCORE" xmlns:cpi="urn:schemas-microsoft-com:cpi" />
</unattend>
'@

$unattendDC1 | Out-File -FilePath "$($cml):\Unattend.xml" -Encoding UTF8

# Get required DSC resource

if (-not (Get-Module -ListAvailable -Name xActiveDirectory)) {
    Find-Module -Name xActiveDirectory -Repository PSGallery | Install-Module -Verbose
}

# Define environment
$ConfigData = @{
   AllNodes = @(
     @{
        NodeName = 'localhost';
        PSDscAllowPlainTextPassword = $true;
        RequiredFeatures = @(
          @{ Name = 'DHCP'}
          @{ Name = 'DNS'}
          @{ Name = 'WDS'}
          @{ Name = 'RSAT-DHCP'}
          @{ Name = 'RSAT-DNS-Server'}
          @{ Name = 'WDS-AdminPack'}
       )

   DCAdminPassword = New-Object pscredential -ArgumentList 'nanorocks\administrator',
   (ConvertTo-SecureString -String 'P@ssw0rd' -Force -AsPlainText)
   SafeAdminPassword = New-Object pscredential -ArgumentList 'Password Only',
   (ConvertTo-SecureString -String 'Azerty@123' -Force -AsPlainText)
}

  )
}

# DSC config

Configuration DCConfig {
     Param()
     Import-DscResource -ModuleName xActiveDirectory
     Node localhost {
        LocalConfigurationManager {
           RebootNodeIfNeeded = $true;
        }
	WindowsFeature ADDS {
       Name = 'AD-Domain-Services';
       Ensure = 'Present';
    }

foreach ($f in $Node.RequiredFeatures)
{
    WindowsFeature $f.Name
    {
         Name = $f.Name ;
         Ensure = 'Present';
    }
 }

 xADDomain DSDC1 {
    DomainName = 'nanorocks.local';
    DomainAdministratorCredential = $Node.DCAdminPassword
    SafemodeAdministratorPassword = $Node.SafeAdminPassword
    DependsOn = '[WindowsFeature]ADDS';
 }

}
}

# Compile config into MOF file

if (-not(Test-Path -Path ~/Documents/DSC) ){ mkdir ~/Documents/DSC }
DCConfig -outputPath ~/Documents/DSC -ConfigurationData $ConfigData
```

![](/images/wds7.jpg)

```powershell
# Copy DSC resource
$cHT = @{
    Path = 'C:\Program Files\WindowsPowerShell\Modules\xActiveDirectory';
    Destination = "$($cml):\Program Files\WindowsPowerShell\Modules\xActiveDirectory"
}

Copy-Item @cHT -Recurse -Force
# Copy DSC config
Copy-Item -Path ~/documents/DSC/*.mof -Destination "$($cml):\Users\Public\Documents"
# Copy original boot image from ISO
Copy-Item -Path "$($dl):\Sources\boot.wim" -Destination "$($cml):\Users\Public\Documents"
# Copy prepared Nano.vhd
Copy-Item -Path "$bdir\Target\*.VHD" -Destination "$($cml):\Users\Public\Documents"
# Unmount ISO file
Get-DiskImage -ImagePath $iso | Dismount-DiskImage
# Unmount VHD
Dismount-VHD -Path $childVHD.Path
Start-Vm -VMName $vm
```

After a few minutes, the operating system of the Domain Controller is ready.

![](/images/wds8.jpg)

#### **Step 3**

Let’s promote it as DC with the DSC (Desired State Configuration)

```powershell
# DCPromo over PowerShell Direct
Invoke-Command -VMName $VM -Credential (Get-Credential 'test.local\administrator') -ScriptBlock {
    Set-DscLocalConfigurationManager C:\Users\Public\Documents
    Start-DscConfiguration C:\Users\Public\Documents -Verbose -Wait
    exit
}
```


![](/images/wds9.jpg)

![](/images/wds10.jpg)

![](/images/wds11.jpg)

![](/images/wds12.jpg)

As I’m on a Windows 10 Hyper-V, I can leverage PowerShell Direct recently introduced, so that I don’t rely on the network stack.

![](/images/wds13.jpg)

Once the DC has rebooted, I can start configuring the features I provisioned:

```powershell
# Post-install
Invoke-Command -VMName $VM -Credential (Get-Credential 'nanorocks\administrator') -ScriptBlock {
    # DHCP configuration

    # Authorize

    if (-not(Get-DhcpServerInDC | Where DnsName -eq "$($env:computername).$($env:USERDNSDOMAIN)")) {
        Add-DhcpServerInDC
    } else {
        Get-DhcpServerInDC
    }

    # Scope
    Add-DhcpServerv4Scope -StartRange 10.0.0.20 -EndRange 10.0.0.100 -Name "Nano scope" -State Active -SubnetMask 255.255.255.0

    # Activate (done with Add-DhcpServerv4Scope -State param
    # WDS
    mkdir C:\RemoteInstall
    wdsutil /verbose /progress /initialize-server /RemInst:c:\RemoteInstall # /Authorize
    wdsutil /start-server
    wdsutil /verbose /progress /set-server /AnswerClients:ALL
    Import-WdsBootImage -Path C:\Users\Public\Documents\boot.wim
    dir C:\Users\Public\Documents\*.vhd | Import-WdsInstallImage
}
```

#### **Step 4**

Let&#8217;s now create a new VM. To be able to boot over PXE on a Generation 1 virtual machine its network adapter should be a legacy network card.

```powershell
# Create test VM Generation 1 and add legacy network card for PXE boot
$testVM = 'Nano-test-pxe'
New-VHD -Path (Join-Path -Path ((Get-VMHost).VirtualHardDiskPath) -ChildPath "$($testVM).vhdx") -Dynamic -SizeBytes 127GB
New-VM -VMName $testVM -Generation 1 -MemoryStartupBytes 1024MB -NoVHD -SwitchName Internal-test |
Remove-VMNetworkAdapter
Get-VM -VMName $testVM |
Add-VMNetworkAdapter -IsLegacy:$true -SwitchName 'Internal-test'
Get-VM -VMName $testVM |
Add-VMHardDiskDrive -Path  (Join-Path -Path ((Get-VMHost).VirtualHardDiskPath) -ChildPath "$($testVM).vhdx")
Start-VM -VMName $testVM
```

![](/images/wds14.jpg) 

![](/images/wds15.jpg)

#### **Step 5**

![](/images/wds16.jpg)

Press F12 to PXE boot.

![](/images/wds17.jpg)

![](/images/wds18.jpg)

![](/images/wds19.jpg)

![](/images/wds20.jpg)

![](/images/wds21.jpg)

![](/images/wds22.jpg)

![](/images/wds23.jpg)

![](/images/wds24.jpg)

![](/images/wds25.jpg)

[1]: http://blogs.technet.com/b/nanoserver/archive/2015/06/03/how-to-use-wds-to-pxe-boot-a-nano-server-vhd.aspx
[2]: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview?WT.mc_id=Blog_WS_Announce_TTD
[3]: https://msdn.microsoft.com/en-us/library/mt126167.aspx?f=255&MSPPError=-2147217396
[4]: http://blogs.technet.com/b/nanoserver/archive/2015/06/16/powershell-script-to-build-your-nano-server-image.aspx