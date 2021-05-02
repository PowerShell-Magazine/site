---
title: '#PSTip Enumerating mapped network drives using WScript.Network'
author: Jaap Brasser
type: post
date: 2014-07-23T18:00:24+00:00
url: /2014-07-23-pstip-enumerating-mapped-network-drives-using-wscript-network/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
In the &#8216;[Create mapped network drive using WScript.Network][1]&#8216; tip, it was shown how a mapped drive can be mounted. The _WScript.Network_ object also contains the _EnumNetworkDrives_ method which enumerates all locally mapped network drives:

```
(New-Object -ComObject WScript.Network).EnumNetworkDrives()
```


The output of this method is unfortunately a bit disappointing. It returns a COM object containing an array of strings:

![](/images/18.png)

To make this information more useful the following function can be used:

<pre class="brush: powershell; title: ; notranslate" title="">Function Get-MappedDrive {
    (New-Object -ComObject WScript.Network).EnumNetworkDrives() | ForEach-Object -Begin {
        $CreateObject = $false
    } -Process {
       if ($CreateObject) {
           $HashProps.NetworkPath = $_
           New-Object -TypeName PSCustomObject -Property $HashProps
           $CreateObject = $false
       } else {
           $HashProps = @{
               LocalPath = $_
           }
           $CreateObject =$true
       }
   }
}
</pre>

This function gathers the output of the _EnumNetworkDrives_ method and organizes this into PowerShell objects:

![](/images/124.png)

This function is also available in the TechNet Script Gallery available [here][2].

[1]: /2014/07/21/pstip-create-mapped-network-drive-using-wscript-network/
[2]: http://gallery.technet.microsoft.com/Get-MappedDrive-Get-list-f454b048