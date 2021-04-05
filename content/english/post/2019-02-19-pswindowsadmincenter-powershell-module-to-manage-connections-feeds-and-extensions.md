---
title: 'PSWindowsAdminCenter: PowerShell Module to Manage Connections, Feeds, and Extensions'
author: Ravikanth C
type: post
date: 2019-02-19T14:42:53+00:00
url: /2019/02/19/pswindowsadmincenter-powershell-module-to-manage-connections-feeds-and-extensions/
post_views_count:
  - 7253
views:
  - 8024
categories:
  - Module Spotlight
  - PSWindowsAdminCenter
tags:
  - Modules

---
I had published a PowerShell DSC resource module, last month, called [WindowsAdminCenterDsc](https://www.powershellmagazine.com/2019/01/31/dsc-resource-module-to-install-and-configure-windows-admin-center/) that uses the PowerShell module that was made available with Windows Admin Center version 1812. This module makes use of the REST API that comes with Windows Admin Center to manage connections, feeds, and extensions. 

I had immediately verified that the API was available in version 1809.5 as well. So, I wanted to build another PowerShell module that has similar and/or more features than the current module that ships with version 1812. Also, the goal was to ensure that I can use this module in my build process to add the newly deployed servers and clusters to Windows Admin Center in an automated manner.

> Note: This module works with Windows Admin Center 1809.5 and above.


This module can be installed from PowerShell Gallery:

```powershell
Install-Module -Name PSWindowsAdminCenter
```

This project is available in [my GitHub repository](https://github.com/rchaganti/PSWindowsAdminCenter/). I have a few TODOs:

  * Add Export option to _Get-WacConnection_ command so that you can export the connections details to a CSV file.
  * Add Import option to _Add-WacConnection_ command so that you can import all connections from a CSV file.
  * Update _WindowsAdminCenterDsc_ module to use the _PSWindowsAdminCenter_ instead of the module that ships with WAC.

If you see any issues or would like to see new features, feel free to [create an issue](https://github.com/rchaganti/PSWindowsAdminCenter/issues/new).