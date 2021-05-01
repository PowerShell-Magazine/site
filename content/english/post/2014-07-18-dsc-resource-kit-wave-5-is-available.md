---
title: DSC Resource Kit Wave 5 is available
author: Ravikanth C
type: post
date: 2014-07-18T07:38:49+00:00
url: /2014/07/18/dsc-resource-kit-wave-5-is-available/
categories:
  - News
  - PowerShell DSC
tags:
  - PowerShell DSC
  - News

---
Windows PowerShell team released DSC Resource Kit Wave 5.

This wave has added the following

- **xWordPress**: This module contains two resources, **xIISWordPressSite** and **xWordPressSite**, as well as example configurations under Samples, which show end-to-end deployment of the common WordPress site.
- **xPhp**: This DSC resource allows you to set up PHP in IIS. This is used in the xWordPress examples.
- **xMySql**: This module includes 5 resources that allow you to set up a MySQL Server, Database, User, and create a Grant for the user. This is used in the WordPress examples.
- **xPsDesiredStateConfiguration** now includes **xWindowsOptionalFeature**. This resource allows configuring Windows Optional Features for Windows client SKUs.
- **xWebAdministration** has added xIisModule, which enables registration of modules (such as FastCgiModules) with IIS.
- **xWindowsUpdate**: This module actually went live just after Wave 4, so it missed the announcement of the Reskit. It contains the xHotfix resource, which handles installation of a Windows update (or a hotfix) from a given path (file path or a URI).

There are a total 77 resources in the DSC resource kit. Some of these new resources require Windows Management Framework (WMF) 5.0 Experimental Release July 2014.

> This release of WMF 5.0 is an experimental release made available only for the DSC resources. The other functionality which was a part of WMF 5.0 May 2014 release may not have been updated and still in the early stages. DO NOT consider this as a WMF 5.0 Preview release.

Windows Management Framework (WMF) 5.0 Experimental Release July 2014 is available only for Windows Server 2012 R2 and Windows 8.1 operating systems.

Go ahead and explore the new resources! 

