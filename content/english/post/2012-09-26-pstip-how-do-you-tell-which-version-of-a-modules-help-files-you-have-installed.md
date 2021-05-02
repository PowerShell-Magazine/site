---
title: '#PSTip How do you tell which version of a module’s Help files you have installed?'
author: Aleksandar Nikolic
type: post
date: 2012-09-26T18:00:35+00:00
url: /2012/09/26/pstip-how-do-you-tell-which-version-of-a-modules-help-files-you-have-installed/
views:
  - 30431
post_views_count:
  - 965
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Note: This tip requires PowerShell 3.0.

Before you start using PowerShell 3.0, learn about the Updatable Help feature first. The <a title="about_Updatable_Help" href="http://technet.microsoft.com/en-us/library/hh847735.aspx" target="_blank">about_Updatable_Help</a> help topic is a good place to start. Also, the Hey, Scripting Guy! blog has published two blog posts written by <a title="Follow June Blender on Twitter" href="http://twitter.com/juneb_get_help" target="_blank">June Blender</a>, a senior programming writer on the Windows PowerShell team&#8211;<a title="Understanding and Using Updatable PowerShell Help" href="http://blogs.technet.com/b/heyscriptingguy/archive/2012/08/31/understanding-and-using-updatable-powershell-help.aspx" target="_blank">Understanding and Using Updatable PowerShell Help</a> and <a title="Where is PowerShell Updatable Help for Windows Modules?" href="http://blogs.technet.com/b/heyscriptingguy/archive/2012/09/24/where-is-powershell-updatable-help-for-windows-modules.aspx" target="_blank">Where is PowerShell Updatable Help for Windows Modules?</a>. June is responsible for most of the PowerShell help topics, thus there is no better person to talk about these subjects.

Tomorrow night (9/27), the PowerScripting Podcast&#8217;s dynamic duo (Hal and Jon) will be speaking to June about Updatable Help and other exciting new features of windows PowerShell 3.0. Be sure to join them live tomorrow at 9:30 PM EDT at <a title="PowerScripting Podcast Live" href="http://live.powerscripting.net/" target="_blank">live.powerscripting.net</a>!

Today&#8217;s tip is inspired by June&#8217;s script from one of the mentioned blog posts and will show you a different approach to find versions of modules&#8217; Help files that you have installed on your machine:

<pre class="brush: powershell; title: ; notranslate" title="">dir $pshome\*helpinfo.xml -Recurse | foreach {
    $modulename = ($_.Name -split '_')[0]
    $UICultures = ((Get-Content $_)).HelpInfo.SupportedUICultures.UICulture
    $UICultures | foreach {
        [pscustomobject]@{
            ModuleName = $modulename
            Version = $_.UICultureVersion
            UICulture = $_.UICultureName
        }
    }
}
</pre>