---
title: '#PSTip Using XPath in PowerShell, Part 3'
author: Bartek Bielawski
type: post
date: 2014-06-26T18:00:07+00:00
url: /2014/06/26/pstip-using-xpath-in-powershell-part-3/
categories:
  - Tips and Tricks
  - XML
tags:
  - Tips and Tricks
  - XML

---
Filtering data with XPath works very well even if we need more complex filters that require information from different levels in the XML document. Perfect example of such document is the result of nmap stored in the XML format:

<pre class="brush: powershell; title: ; notranslate" title="">nmap.exe 192.168.200.0/24 -oX testLocal.xml
</pre>

**Note**: You can find the code and input file <a href="https://gist.github.com/bielawb/8739f9afbc5b7e347a0c" target="_blank">here</a>.

If we want to retrieve all IPv4 addresses from the hosts that are currently up we can ask for addresses (using //host/address path) and filter them on the information both on current level (to make sure we get correct address type) and value of attribute on node that has the same parent as node we want to retrieve (&#8216;state&#8217; on &#8216;status&#8217; node):

```
$addressUp = @'
    //host/address[
        @addrtype = 'ipv4' and
        ../status/@state = 'up'
    ]
'@ 

$nodeList = Select-Xml -Path .\testLocal.xml -XPath $addressUp |
            ForEach-Object { $_.Node }

$ldapPort = @'
    /host/ports/port[
        @portid = '389' and
        state/@state = 'open'
    ]
'@

foreach ($node in $nodeList) {
    $noteproperties = [ordered]@{
        Address = $node.addr
        HostName = $node.ParentNode.hostnames.hostname.name
    }

    $partialXml = [XML]('<host>{0}</host>' -f $node.ParentNode.InnerXml)

    Select-Xml -Xml $partialXml -XPath $ldapPort  | 
        Select-Object -ExpandProperty Node | 
        Add-Member -NotePropertyMembers $noteproperties -PassThru
}

Address  : 192.168.200.1
HostName : DC
protocol : tcp
portid   : 389
state    : state
service  : service
```

Because &#8216;Node&#8217; property will not contain markup for opening/closing node, we need to add it to get valid XML. Results are not very surprising&#8211;it’s not uncommon for domain controllers to have port 389 open.