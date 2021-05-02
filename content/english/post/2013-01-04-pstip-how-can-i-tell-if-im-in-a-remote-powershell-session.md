---
title: '#PSTip How can I tell if I’m in a remote PowerShell session?'
author: Shay Levy
type: post
date: 2013-01-04T19:00:50+00:00
url: /2013/01/04/pstip-how-can-i-tell-if-im-in-a-remote-powershell-session/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 3.0 or above.

There are times where you&#8217;ll want to determine if the session your code is running in is a remoting session. For a quick test you can check the value of _$Host.Name_. If it&#8217;s _ServerRemoteHost_, you are in a remote session (works in v2.0). For a more robust solution you can use the existence of the _$PSSenderInfo_ variable (exist in v2.0). From [about\_Automatic\_Variables][1]:

```
$PSSenderInfo
   Contains information about the user who started the PSSession,
   including  the user identity and the time zone of the originating
   computer. This variable is available only in PSSessions.

   The $PSSenderInfo variable includes a user-configurable property,
   ApplicationArguments, which, by default, contains only the
   $PSVersionTable from the originating session. To add data to the
   ApplicationArguments property, use the ApplicationArguments parameter
   of the New-PSSessionOption cmdlet.
```

Here we connect to a remote server and we&#8217;re also adding out our own custom value to the $PSSenderInfo variable.

We&#8217;re using the new inline syntax, available in PowerShell 3.0, to create a new session object, which is passing a new value (in the form of a hash table) that will be stored in the $PSSenderInfo variable. Once we connect to the remote computer, we can check its value. In addition to the default content of $PSSenderInfo, we can also find our custom value:

```
PS> Enter-PSSession Thor -SessionOption @{ApplicationArguments=@{MyKey="MyValue"}}
[Thor]: PS C:\Users\psmag&gt; $PSSenderInfo.ApplicationArguments

Name                           Value
----                           -----
PSVersionTable                 {CLRVersion, WSManStackVersion, PSVersion, BuildVersion...}
MyKey                          MyValue

[Thor]: PS C:\Users\psmag&gt; $PSSenderInfo.ApplicationArguments.MyKey
MyValue
```

[1]: http://technet.microsoft.com/en-us/library/hh847768.aspx