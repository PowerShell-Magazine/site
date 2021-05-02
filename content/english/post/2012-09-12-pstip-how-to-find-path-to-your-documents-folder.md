---
title: '#PSTip How to find path to your Documents folder'
author: Aleksandar Nikolic
type: post
date: 2012-09-12T22:21:10+00:00
url: /2012/09/12/pstip-how-to-find-path-to-your-documents-folder/
views:
  - 11615
post_views_count:
  - 3796
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
It&#8217;s pretty easy to find path to your Documents folder with a little help from .NET Framework&#8217;s Environment class:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Environment]::GetFolderPath('MyDocuments')
C:\Users\Administrator\Documents
</pre>

You can accomplish the same using the identifier name Personal:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [Environment]::GetFolderPath('Personal')
C:\Users\Administrator\Documents
</pre>

How can you find all valid names? Specify a wrong name, and PowerShell will help you 🙂

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [environment]::GetFolderPath('wrongvalue')
Cannot convert argument "folder", with value: "wrongvalue", for "GetFolderPath" to
 type "System.Environment+SpecialFolder": "Cannot convert value "wrongvalue" to type
 "System.Environment+SpecialFolder". Error: "Unable to match the identifier name wrongvalue
 to a valid enumerator name. Specify one of the following enumerator names and try again:
 Desktop, Programs, Personal, MyDocuments, Favorites, Startup, Recent, SendTo, StartMenu,
 MyMusic, MyVideos, DesktopDirectory, MyComputer, NetworkShortcuts, Fonts, Templates,
 CommonStartMenu, CommonPrograms, CommonStartup, CommonDesktopDirectory, ApplicationData,
 PrinterShortcuts, LocalApplicationData, InternetCache, Cookies, History, CommonApplicationData,
 Windows, System, ProgramFiles, MyPictures, UserProfile, SystemX86, ProgramFilesX86, CommonProgramFiles,
 CommonProgramFilesX86, CommonTemplates, CommonDocuments, CommonAdminTools, AdminTools, CommonMusic,
 CommonPictures, CommonVideos, Resources, LocalizedResources, CommonOemLinks, CDBurning""
</pre>