---
title: '#PSTip Preview PowerShell script files in Windows Explorer Preview Pane'
author: Shay Levy
type: post
date: 2012-09-10T18:00:28+00:00
url: /2012/09/10/pstip-preview-powershell-script-files-in-windows-explorer-preview-pane/
views:
  - 14174
post_views_count:
  - 3032
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks
---
The Windows Explorer Preview Pane can display the contents of text files, graphics, and other file types without having to use external software.

![](/images/noPreview.png)

To add new file types to preview, such as PowerShell scripts, run the following command:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Set-ItemProperty Registry::HKEY_CLASSES_ROOT\.ps1 -Name PerceivedType -Value text
</pre>

Here&#8217;s how it looks in regedit.exe after the value has been added.

![](/images/ps1.png)

Now navigate to your scripts folder and select a script file. If the preview pane is open and you don&#8217;t see the script content, turn preview off and then on (you can use the ALT+P shortcut twice) to force Explorer to read the Registry value. Now you should see its content displayed:

![](/images/previewAfter.png)

To enable preview for multiple extensions:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; Get-Item Registry::HKEY_CLASSES_ROOT\* -Include .ps1,.psm1,.psd1 | Set-ItemProperty -Name PerceivedType -Value text
</pre>