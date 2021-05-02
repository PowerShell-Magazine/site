---
title: '#PSTip For which system modules have you downloaded updatable help files?'
author: Aleksandar Nikolic
type: post
date: 2012-09-25T18:07:12+00:00
url: /2012/09/25/for-which-system-modules-have-you-downloaded-updatable-help-files/
views:
  - 4998
post_views_count:
  - 1011
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When you run the new Update-Help cmdlet in PowerShell 3.0, it downloads and installs Help files, but you also get HelpInfo XML files. You can find them in $pshome and its subfolders. They are always named in the same way&#8211;ModuleName\_GUID\_HelpInfo.xml.

In the following command you will iterate through the $pshome folder and use that consistent naming scheme of HelpInfo files to find all system modules for which you have downloaded Help files:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; dir $pshome\*HelpInfo.xml -Recurse | foreach { ($_.name -split '_')[0] }
</pre>