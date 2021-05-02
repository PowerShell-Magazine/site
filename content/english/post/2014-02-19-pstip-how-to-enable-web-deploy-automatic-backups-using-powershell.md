---
title: '#PSTip How to enable Web Deploy automatic backups using PowerShell'
author: Shay Levy
type: post
date: 2014-02-19T19:00:31+00:00
url: /2014/02/19/pstip-how-to-enable-web-deploy-automatic-backups-using-powershell/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

<span style="line-height: 1.5em;">Web Deploy v3 introduced an automatic server-side backup feature for IIS 7 and above. </span>When automatic backups are configured on the server and a user publishes to his site using Web Deploy, it will first take a backup of the live site and store it on the server before committing any changes to the site.

There is no UI to enable this feature. To enable automatic Web Deploy backups, run the following command:

<pre class="brush: powershell; title: ; notranslate" title="">Import-Module WebAdministration
Set-WebConfiguration -PSPath MACHINE/WEBROOT/APPHOST -Filter system.webServer/wdeploy/backup -Value @{turnedOn=$true; enabled=$true}
</pre>

The default location of backups on the server is set to _&#8220;{sitePathParent}&#123;siteName}_snapshots&#8221;_, where _&#8220;{sitePathParent}&#8221;_ and _&#8220;{siteName}&#8221;_ are path replacement variables for which are determined at run-time. _sitePathParent_ is the physical file path of the parent of your sites content and _siteName_ is the name of your site. In the case of the &#8220;Default Web Site&#8221; website, the location will resolve to _&#8220;C:\inetpub\Default Web Site_snapshots&#8221;._

Here&#8217;s a sample output of that location. Backups are saved as Zip files:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-ChildItem "C:\inetpub\Default Web Site_snapshots" -Name
msdeploy_2014_01_22_10_57_58.zip
msdeploy_2014_01_23_10_27_25.zip
msdeploy_2014_01_28_13_40_26.zip
msdeploy_2014_02_10_18_21_11.zip
</pre>

By default, only the last 4 backups are stored on the server in the above location. When the maximum number of backups has been created, the oldest backup will be deleted. To change the value, run:

<pre class="brush: powershell; title: ; notranslate" title="">Set-WebConfiguration -PSPath MACHINE/WEBROOT/APPHOST -Filter system.webServer/wdeploy/backup -Value @{numberOfBackups=6}
</pre>

Note that Web Deploy doesn&#8217;t ship with IIS and requires a [separate installation][1].

[1]: http://www.iis.net/downloads/microsoft/web-deploy