---
title: '#PSTip List all WMI namespaces on a system'
author: Ravikanth C
type: post
date: 2013-10-18T18:00:19+00:00
url: /2013/10/18/pstip-list-all-wmi-namespaces-on-a-system/
categories:
  - WMI
  - Tips and Tricks
tags:
  - Tips and Tricks
  - WMI

---
**Note**: This tip requires PowerShell 2.0 or above.

In WMI, a [namespace][1] is a collection of classes. There are many WMI namespaces on a system and each namespace might contain more namespaces. Each WMI namespace is an instance of [__NAMESPACE][2] system class. So, we can use _Get-WmiObject_ cmdlet to list all WMI namespaces on a system.

<pre class="brush: powershell; title: ; notranslate" title="">Get-WmiObject -Class __NAMESPACE | select Name
</pre>

But since _Get-WmiObject_ defaults to root\cimv2 namespace, we get namespaces under root\cimv2 namespace only.

Here is a simple function that helps you get all WMI namespaces on a system in a recursive manner.

```
Function Get-WmiNamespace {
    Param (
        $Namespace='root'
    )
    Get-WmiObject -Namespace $Namespace -Class __NAMESPACE | ForEach-Object {
            ($ns = '{0}\{1}' -f $_.__NAMESPACE,$_.Name)
            Get-WmiNamespace $ns
    }
}
```




[1]: http://msdn.microsoft.com/en-us/library/aa390820(v=vs.85).aspx#wmi.gloss_namespace
[2]: http://msdn.microsoft.com/en-us/library/aa394657(v=vs.85).aspx

