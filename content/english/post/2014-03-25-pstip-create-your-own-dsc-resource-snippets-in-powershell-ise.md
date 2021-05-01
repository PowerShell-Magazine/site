---
title: '#PSTip Create your own DSC resource snippets in PowerShell ISE'
author: Aleksandar Nikolic
type: post
date: 2014-03-25T20:04:04+00:00
url: /2014/03/25/pstip-create-your-own-dsc-resource-snippets-in-powershell-ise/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 4.0 or above.

PowerShell ISE 4.0 comes with just two DSC-related snippets (DSC Configuration (simple) and DSC Resource Provider (simple)). (Be aware that DSC Configuration (simple) snippet has a bug and uses the Requires property instead of new DependsOn property.).

What about DSC resources? How can we get info about their properties and valid values? You can take the cursor to the place where you have the name of the resource, press _CTRL+Space_, and a resource syntax will pop up:

![](/images/service_syntax.png)

But, wouldn&#8217;t it be nice if we could get DSC snippets with a syntax for all our DSC resources? Luckily, the _Get-DscResource_ cmdlet has the _-Syntax_ parameter that we can use:

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; Get-DscResource -Name service -Syntax
Service [string] #ResourceName
{
    Name = [string]
    [ BuiltInAccount = [string] { LocalService | LocalSystem | NetworkService }  ]
    [ Credential = [PSCredential] ]
    [ DependsOn = [string[]] ]
    [ StartupType = [string] { Automatic | Disabled | Manual }  ]
    [ State = [string] { Running | Stopped }  ]
}
</pre>

The easiest way to create DSC snippets is to enumerate all DSC resources, get the syntax using the _Get-DscResource_ cmdlet, and pass it to _New-IseSnippet_ cmdlet. Let&#8217;s wrap all that in a simple function called _New-DscSnippet_:

    function New-DscSnippet {
       $resources = Get-DscResource | select -ExpandProperty name
    
       foreach ($resource in $resources) {
       $text = Get-DscResource -Name $resource -Syntax
       New-ISESnippet -Text $text -Title "DSC $resource Resource" -Description "Full DSC $resource resource syntax" -Force
       }
    }
Get-DscResource is a very slow cmdlet and this function will need quite some time to create all DSC snippets, but at the end you will get very usable code snippets to use from now on.

![](/images/DSC_Resources.gif)