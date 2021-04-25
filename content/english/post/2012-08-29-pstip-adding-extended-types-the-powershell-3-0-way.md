---
title: '#PSTip Adding extended types – the PowerShell 3.0 way!'
author: Ravikanth C
type: post
date: 2012-08-29T18:00:07+00:00
url: /2012/08/29/pstip-adding-extended-types-the-powershell-3-0-way/
views:
  - 8265
post_views_count:
  - 1442
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
PowerShell 3.0 introduced important changes to the Update-TypeData cmdlet which includes specifying the [dynamic type data][1]. Using these new features of [Update-TypeData][2] cmdlet, we can add extended types without using the types.ps1xml files.

For example, let us say that we want to add the age of a file as a property to the output of Get-ChildItem cmdlet.

This can be achieved using:

<pre class="brush: powershell; title: ; notranslate" title="">Update-TypeData -TypeName System.IO.FileInfo -MemberName FileAge -MemberType ScriptProperty -Value { ((get-date) - ($this.creationtime)).days }
</pre>
What we are doing in the above command is very straightforward. For every System.IO.FileInfo type of object, we instruct PowerShell to add a ScriptProperty called FileAge and assign the days since the file was created as its value.

![](/images/tip7-file.png)

In PowerShell 2.0, this would require creating a <a href="http://technet.microsoft.com/en-us/library/hh847881.aspx">types.ps1xml</a> file and then using Update-TypeData cmdlet. PowerShell 3.0 made it very simple and efficient to use. Make a note, however, the Update-TypeData does not update the format information for the specified types. It only applies to the current session and won&#8217;t be available once you close the session.


[1]: http://technet.microsoft.com/en-us/library/hh857339.aspx#BKMK_DynamicTypes
[2]: http://technet.microsoft.com/en-us/library/hh849908.aspx