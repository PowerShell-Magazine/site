---
title: '#PSTip How to remove the “Folders” group from my computer in Windows 8'
author: Shay Levy
type: post
date: 2014-03-24T18:00:08+00:00
url: /2014/03/24/pstip-how-to-remove-the-folders-group-from-my-computer-in-windows-8/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
When you double click the _&#8220;This PC&#8221;_ desktop icon in Windows 8 you get the familiar _&#8220;My Computer&#8221;_ window which includes several groups such as _Devices and drives_,_ Folders_, _Network locations_ (in case you have mapped drives), etc.

![](/images/win8thispc.png)

<span style="line-height: 1.5em;">Personally I don&#8217;t like the </span><em style="line-height: 1.5em;">Folders</em><span style="line-height: 1.5em;"> group. I can collapse it and hide its content but I prefer not seeing it at all as I can get to each item it has via the left-hand side explorer tree. Each folder in the Folders group is represented by a Registry key under the <em>HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace</em> registry path.</span>

![](/images/win8folders.png)

As you can see the keys do not have a readable name, instead GUIDs are in use. Here&#8217;s how they map to folder names:

<pre class="brush: plain; title: ; notranslate" title="">Desktop Folder   – B4BFCC3A-DB2C-424C-B029-7FE99A87C641
Documents Folder – A8CDFF1C-4878-43be-B5FD-F8091C1C60D0
Downloads Folder – 374DE290-123F-4565-9164-39C4925E467B
Music Folder     – 1CF1260C-4DD0-4ebb-811F-33C572699FDE
Pictures Folder  – 3ADD1653-EB32-4cb0-BBD7-DFA0ABB5ACCA
Videos Folder    – A0953C92-50DC-43bf-BE83-3742FED03C9C
</pre>

In addition, there&#8217;s another key under that path, _DelegateFolders_. To remove a specific folder, remove its Registry key. In my case I wanted to remove the whole group. So, PowerShell to the rescue (you might want to back up the _NameSpace_ key before you modify it).

```
$path = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace'

Get-ChildItem $path |
Where-Object { $_.PSChildName -as [guid] } |
Remove-Item
```

In the above example, I list all keys under the _NameSpace_ key and exclude the _DelegateFolders_ key by filtering all key names that can cast to a _GUID_, then I remove them.

When I reopen the _This PC_ window, they are all gone 🙂