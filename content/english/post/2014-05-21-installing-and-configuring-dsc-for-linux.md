---
title: Installing and configuring DSC for Linux
author: Ravikanth C
type: post
date: 2014-05-21T16:00:29+00:00
url: /2014/05/21/installing-and-configuring-dsc-for-linux/
categories:
  - Linux
  - Azure
  - PowerShell DSC
tags:
  - Azure
  - PowerShell DSC
  - Linux

---
**[Update 1: 5/23/2014]** Added pre-requisite install commands for Oracle Linux

I am sure you are aware of the Microsoft&#8217;s [announcement of the CTP release of DSC for Linux][1]. This is an exciting announcement and I wanted to explore more on this. There is a very useful [step-by-step guide by Kristopher Bash][2] on the Microsoft Building Clouds blog on how to set up DSC for Linux. These steps assume that you have a Linux VM that is either Fedora/RHEL/Cent OS. It will be clear if you follow the comments on that post that the steps needed some updates and there is different set of commands you need to run for the other Linux distributions such as Ubuntu and Debian. In this post, I will talk about these differences and how to setup DSC for Linux in Azure Linux VMs. Thanks to <a href="http://operatingquadrant.com/">Kristopher Bash</a> for the start! 

If you also plan to use Azure VMs for this purpose, don&#8217;t miss the Azure VM configuration requirements towards the end of this article.

This post assumes that you have the necessary Azure Linux VMs. To follow the steps in this article, you&#8217;ll need either CentOS or Ubuntu or Oracle Linux VMs. I deployed these Linux VMs from the Azure VM gallery. Also, this article assumes that the systems or VMs where you are deploying DSC for Linux have Internet connectivity. There are a bunch of pre-requisites you need to install before installing OMI Server or DSC LCM. This is where the first difference is.

Note: For all steps listed below, you need root access. You can use _sudo -s_ to elevate the privileges.

### Installing Pre-requisites in CentOS

We use the Yum repository to install the required pre-requisites. I will not go into the details of each but it should be clear from the package names. In this following set of commands, I have added _-y_ switch to avoid the prompt.

<pre class="brush: bash; title: ; notranslate" title="">yum -y groupinstall 'Development Tools'
yum -y install pam-devel
yum -y install openssl-devel
yum -y install python
yum -y install python-devel
</pre>

### Installing Pre-requisites in Ubuntu

The default package manager in Ubuntu is apt-get. So, we use apt-get to install the pre-requisites required for OMI and DSC. The package names here will be slightly different. Once again, I used _-y_ switch to avoid any confirmation prompts.

<pre class="brush: bash; title: ; notranslate" title="">apt-get -y install build-essential
apt-get -y install pkg-config
apt-get -y install python
apt-get -y install python-dev
apt-get -y install libpam-dev
apt-get -y install libssl-dev
</pre>

### Installing Pre-requisites in Oracle Linux

Thanks again to [Kris][3]. We use the Yum repository to install the required pre-requisites which is similar to Cent OS. However, we need to install wget and kernel-headers packages on Oracle Linux.

<pre class="brush: bash; title: ; notranslate" title="">yum -y groupinstall 'Development Tools'
yum -y install pam-devel
yum -y install openssl-devel
yum -y install python
yum -y install python-devel
yum -y install wget
yum -y install kernel-headers
</pre>

### Downloading and Building OMI Server

Next, we need to download and install OMI Server. How this is done is common across various versions of Linux. The OMI build can be downloaded from Open Group site. The current version of OMI server is 1.0.8.

```
mkdir /root/downloads
cd /root/downloads


wget https://collaboration.opengroup.org/omi/documents/30532/omi-1.0.8.tar.gz
tar -xvf omi-1.0.8.tar.gz

cd omi-1.0.8
./configure | tee /tmp/omi-configure.txt
make | tee /tmp/omi-make.txt
make install | tee /tmp/omi-make-install.txt
```

The _tee_ command with _make_ and _make install_ commands will help us capture the output from the make process for any analysis later. The above set of commands build and install the OMI server.

### Downloading and Building DSC LCM

The Linux DSC build is available on [Github][4]. This can be downloaded to the Linux system and configured. The following steps are common across Linux versions.

```
cd /root/downloads

wget https://github.com/MSFTOSSMgmt/WPSDSCLinux/releases/download/v1.0.0-CTP/PSDSCLinux.tar.gz
tar -xvf PSDSCLinux.tar.gz
cd dsc/
mv * /root/downloads/

cd /root/downloads
make | tee /tmp/dsc-make.txt
make reg | tee /tmp/dsc-make-reg.txt
```

The _make_ command for building DSC will throw a lot of warning messages on Ubuntu. It is not completely clear about the impact of those warning messages. However, since DSC LCM works as expected, I have put the same steps for both versions of Linux I verified.

We now have the OMI Server and DSC LCM built. We can start the OMI Server so that the access to DSC LCM is enabled.

```
OMI_HOME=/opt/omi-1.0.8
/opt/omi-1.0.8/bin/omiserver -d
```


The above commands set the _$OMI_HOME_ variable in the shell and start the OMI Server. At this moment, you should be able to create a CIM session to connect to these Linux systems.

### Things to note before you proceed with DSC

By default, the OMI server listens at port number 5985/5986. You can change this by editing the omiserver.conf at /opt/omi-1.0.8/etc/. For now, let us move on with the default port configurations. Also, at this moment, you cannot use the Azure user account created during VM provisioning for enacting DSC configuration. It results in permission issues.

If you are also using Azure VMs, you need to first enable these endpoints.

#### Configuring Azure VM endpoints for DSC

Azure Linux VMs, by default, are not configured to provide access to ports 5895 and 5896. So, we need to add this endpoint first. We can use the Azure PowerShell cmdlets to do this.

```
Get-AzureVM -ServiceName "DSCDemo" -Name "DSCLinux" |
Add-AzureEndpoint -Name "DSCLinux-HTTPS" -Protocol TCP -LocalPort 5896 -PublicPort 5896 -Verbose |
Update-AzureVM -Verbose

Get-AzureVM -ServiceName "DSCDemo" -Name "DSCLinux-1" |
Add-AzureEndpoint -Name "DSCLinux-HTTPS" -Protocol TCP -LocalPort 54321 -PublicPort 5896 -Verbose |
Update-AzureVM -Verbose
```

#### Resetting root password

You can skip this step if you are using non-Azure VMs or already have root access to your Linux systems for pushing DSC configuration. The Linux Azure VMs, by default, do not provide the root user password. So, therefore you need to reset the root account password. This can be done using the _passwd_ command. On the CentOS VM, you need to first reset security context of /etc/shadow file.

<pre class="brush: bash; title: ; notranslate" title="">chcon -t shadow_t /etc/shadow
passwd root
</pre>

This is it. We are all set to connect to the Azure Linux VMs and push DSC configuration.

The built-in DSC resources in DSC Linux are different from what we have on Windows systems. So, we first need to copy the DSC Linux resources to the Windows systems from which you will be connecting to Linux DSC. These resources are available in the GitHub repository at <https://github.com/MSFTOSSMgmt/WPSDSCLinux/releases/download/v1.0.0-CTP/nx-PSModule.zip>. You need to extract the contents of this archive to %SystemRoot%\system32\WindowsPowerShell\v1.0\Modules.

### Configuration Document for Linux Systems

For the purpose of demonstration, we will use a simple nxFile resource and create a file called dsctest in /tmp directory of Linux VMs.

<pre class="brush: powershell; title: ; notranslate" title="">Configuration MyDSCDemo
{
    Import-DSCResource -Module nx
    Node "DSCDemo.cloudapp.net"{
        nxFile myTestFile
        {
            Ensure = "Present"
            Type = "File"
            DestinationPath = "/tmp/dsctest"
            Contents="This is my DSC Test!"
        }
    }
}
</pre>

You can simply load this configuration into memory and create the MOF file by executing the configuration command _MyDSCDemo_.

### Creating CIM sessions

Before enacting the configuration, we need to create a CIM session to the Linux system.

```
$username = "azureuser"
$hostname = "dscdemo.cloudapp.net"
$port = 54321

$Cred = Get-Credential -UserName root -Message "Enter root password"
$CimOptions = New-CimSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck -UseSsl
$cimSession = New-CimSession -Credential $Cred -ComputerName $hostname -Port $port -Authentication Basic -SessionOption $CimOptions
```

We now have a CIM Session and a MOF created for the configuration. We can now use the _Start-DscConfiguration_ cmdlet to enact this.

<pre class="brush: powershell; title: ; notranslate" title="">Start-DscConfiguration -Wait -Verbose -CimSession $cimSession -Path .\MyDSCDemo
</pre>

You can check if the configuration is applied or not by using the _Get-DscConfiguration_ cmdlet.

![](/images/azure.png)

On the Linux VM, you can find the DSC logs at _$OMI_HOME/var/log/_.

The Building Clouds blog that has the step by step guidance also talks about creating a init script to start OMI Server every time you start the Linux system. However, the way it is written, it only works on CentOS and Fedora. I am looking for another version of it for Debian/Ubuntu systems. I will update this post when I have some solid solution for that.

Next up, we will see how to create custom DSC Linux resources. There is a lot to discuss. Watch this space.

[1]: /2014/05/19/microsoft-announced-the-ctp-release-of-windows-powershell-desired-state-configuration-for-linux/
[2]: http://blogs.technet.com/b/privatecloud/archive/2014/05/19/powershell-dsc-for-linux-step-by-step.aspx
[3]: http://operatingquadrant.com/
[4]: https://github.com/MSFTOSSMgmt/WPSDSCLinux/releases