---
title: '#PSTip Get all writeable properties of a WMI class'
author: Ravikanth C
type: post
date: 2012-09-06T18:00:11+00:00
url: /2012/09/06/pstip-get-all-writeable-properties-of-a-wmi-class/
views:
  - 6234
post_views_count:
  - 1079
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Not all WMI class properties can be modified. Many of these properties are read-only. Take a look at the properties and [Win32_ComputerSystem][1], for example.

![](/images/tip7-wmi.png)


Now, how do we retrieve properties of a WMI class that are read/write capable? This can be achieved by looking at the property [qualifiers][2]. Let us see how:

<pre class="brush: powershell; title: ; notranslate" title="">$class = [wmiclass]'Win32_ComputerSystem'
$class.Properties | ForEach-Object {
      foreach ($qualifier in $_.Qualifiers) {
           if ($qualifier.Name -eq "Write") {
                $_.Name
           }
      }
}
</pre>

Now, this can be easily extended to verify and list other qualifying properties such as data type, etc.

[1]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa394102(v=vs.85).aspx
[2]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa394571(v=vs.85).aspx