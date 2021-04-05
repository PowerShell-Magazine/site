---
title: PowerShell DSC for Linux released!
author: Bartek Bielawski
type: post
date: 2015-05-14T22:29:50+00:00
url: /2015/05/14/powershell-dsc-for-linux-released/
views:
  - 9940
post_views_count:
  - 1934
categories:
  - PowerShell DSC
  - Linux
tags:
  - PowerShell DSC
  - Linux

---
The day that people who love both PowerShell DSC and xplat solutions where looking for had finally arrived. Last week the first version of PowerShell DSC for Linux was released. The information we have so far is based either on the <a href="https://github.com/MSFTOSSMgmt/WPSDSCLinux/releases/tag/V1.0.0-320" target="_blank">release description</a> that can be found in the official PowerShell DSC for Linux repository on GitHub or information shared in the [Release Notes][1] published to Microsoft Download Center.

The most important changes:

  * support for Pull mode is added, including support for partial configurations
  * number of resources available got doubled – currently there are 10, including nxArchive, nxEnvironment, nxFile, nxFileLine, nxGroup, nxPackage, nxScript, nxService, nxSshAuthorizedKeys, and nxUser
  * new version is available in package form, both as rpm and as deb

CTP had several issues (you could read about these issues in the [series about using DSC for Linux in PowerShell Magazine][2]) but most of them are fixed in this release:

  * unlike CTP, this release works fine with WMF 4.0 and WMF 5.0, including April preview of latter (no need to clean up your MOF after it’s compiled)
  * problem with configuration drift not being fixed is resolved
  * schema for nxFile resource in nx module matches the one on the Linux box

If you prefer to use packages when installing software on Linux system you may also want to visit <a href="https://collaboration.opengroup.org/omi/documents.php?action=show&dcat=&gdid=32721" target="_blank">Open Group page</a>, where you will find a package for OMI. That is the main change in OMI 1.0.8.1. If you just want to play with PowerShell DSC for Linux without investing too much time in getting compiler and libraries to work, that may be the best option for you. You can expect more details about PowerShell DSC for Linux from us soon, once we spent enough time <del>breaking</del> testing it!

[1]: http://www.microsoft.com/en-us/download/details.aspx?id=46919
[2]: /2015/02/23/working-with-powershell-dsc-for-linux-part-1/

