---
title: 'PowerShell DSC for Linux: Installation'
author: Bartek Bielawski
type: post
date: 2015-07-06T16:00:44+00:00
url: /2015/07/06/powershell-dsc-for-linux-installation/
views:
  - 11268
post_views_count:
  - 1980
categories:
  - PowerShell DSC
  - Linux
tags:
  - PowerShell DSC
  - Linux
---
First version of PowerShell DSC for Linux is released. We <a href="/2015/05/14/powershell-dsc-for-linux-released/" target="_blank">announced that</a> and promised to come back to that subject. It&#8217;s time to fulfil that promise. In this article, first part of update series, we will focus on new features and bug fixes related to installation.

The first good news is that now both OMI and PowerShell DSC for Linux can be installed on the systems that use _deb_ or _rpm_ packages. This removes requirement of installing development packages previously necessary to compile both products on every node that we want deploy DSC to. Package for OMI can be found at the <a href="https://collaboration.opengroup.org/omi/documents.php?action=show&#038;dcat=&#038;gdid=32721" target="_blank">Open Group page</a> and packages for DSC are distributed in <a href="http://www.microsoft.com/en-us/download/details.aspx?id=46919" target="_blank">MSI file available on Microsoft Download Center</a>. If we do not own any local package repository, we can install these packages using command for local installation, after we manually download the packages to the Linux box.

Here is an example for distributions that are using _yum_ as a package manager:

```bash
yum -y localinstall packages/omiserver-1.0.8.ssl_100.x64.rpm
yum -y localinstall package/dsc-1.0.0-320.ssl_100.x64.rpm
```


The only change we have to perform is to open ports on firewall to allow inbound WS-Man traffic. For example, when using CentOS 7 with firewalld we would need to call _firewall-cmd_:

```shell
firewall-cmd --add-port=5986/tcp --permanent
firewall-cmd --reload
```


We have to also enable the OMI server service; even though it is started after installation, it won&#8217;t run automatically after reboot:

```shell
[root@PSMag ~]# systemctl status omiserver
omiserver.service - OMI CIM Server
   Loaded: loaded (/usr/lib/systemd/system/omiserver.service; disabled)
   Active: inactive (dead)
```


The method used to enable the service also depends on distribution/service&#8217;s controller. In case of systemd used by CentOS 7, we would have to use systemctl enable/start commands to get OMI server up and running:

```shell
systemctl enable omiserver
systemctl start omiservers
```


I would also suggest to get proper certificate for SSL connections. More details on how to achieve that can be found in one of the <a href="/2015/03/23/omi-with-wsman-over-https-done-right/" target="_blank">previous OMI articles</a> in the PowerShell Magazine.

The difference between installing OMI/DSC using packages is that the path to OMI is different than the one we got by default when we installed OMI from the sources. In the current version of the package _OMI_HOME_ points to _/opt/omi_.

Installing OMI from sources is still an option. Unfortunately, installation process in this scenario hasn&#8217;t improved much. We still have to make sure that we have sources for both OMI and DSC structured in a fashion that _make_ scripts from DSC expect. There is also one extra requirement for DSC&#8211;curl libraries. For example, if we would like to install DSC on a system running CentOS, we would have to request libcurl-devel package:

```shell
yum -y install libcurl-devel
```


Another thing added in this version is the configuration script for DSC. We need to make sure it has proper rights before we can call it (we will use _chmod_ for that). The list of parameters available for configure is limited, but we need to run it first before we can install DSC:

```shell
chmod 755 configure
./configure
make
make reg
```


In either case, the major problem we observed around installation in CTP version was fixed in this release&#8211;ConsistencyInvoker is not only copied to the right location during _make reg_ (you can read about the problem <a href="/2015/02/23/working-with-powershell-dsc-for-linux-part-1/" target="_blank">here</a>), it also works as expected and will fix configuration drift if we decide to do so in Local Configuration Manager setting.

There are a few important changes in the way DSC providers are structured in version 1.0. Previously all Python scripts used by providers were not designed for any specific Python version. That has changed and if we want to make sure any update to Python script works for any version of Python installed on the box, we have to modify 3 scripts per resource. There are three options present:

  * Python 2.4-2.5
  * Python 2.6-2.7
  * Python 3

Design is also more modular: a lot of code was moved to the scripts shared between the resources. I believe that is the first step to make it possible to build custom resources with only Get/Test/Set implemented in Python, without a need to implement part of the code that translates MOF document to the actual parameters of the resources. For now, we are limited to resources delivered by the team. Another good news is that the number of resources doubled. We will look at new resources offered in this release in the second part of this series.