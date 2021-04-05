---
title: PowerShell DSC Resource Module to Install and Configure Windows Admin Center
author: Ravikanth C
type: post
date: 2019-01-31T17:00:15+00:00
url: /2019/01/31/dsc-resource-module-to-install-and-configure-windows-admin-center/
post_views_count:
  - 7091
views:
  - 6796
categories:
  - Module Spotlight
  - WindowsAdminCenterDSC
tags:
  - Modules

---
[Windows Admin Center](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/understand/windows-admin-center) (WAC) is the new web-based management application for managing Windows Servers, Failover Clusters, Hyper-Converged Infrastructure (HCI) clusters, and Windows 10 PCs. This is a free application and can be installed on Windows Server 2016, Windows Server 2019, or Windows 10. Unlike System Center Operations Manager (SCOM), WAC does not store any monitoring data locally and therefore it is near real-time only.

Ever since WAC was released, one thought I had was to automatically onboard the servers and clusters that I want to manage within WAC right after their deployment is complete. There was no API that was available to do this earlier. 

With the release of the WAC version 1812 (insider preview), there are a couple of PowerShell modules that are bundled with WAC. This internally uses the REST API and wraps around the same for a few management tasks.

When I saw this, I immediately explored a design to implement DSC resources for WAC install/uninstall and configuration. And, the result is here: https://github.com/rchaganti/WindowsAdminCenterDsc

> This works only with Windows Admin Center 1812 insider preview and above.


This REST API is available in 1809.5 as well and I am working on creating a PowerShell to wrap that API as a set of cmdlets. I will update this DSC resource module as well without breaking the current DSC resource design.

This module contains a set of resources to install Windows Admin Center (WAC) and configure WAC feeds, extensions, and connections. For complete documentation, see https://windowsadmincenterdsc.readthedocs.io/en/latest/.
