---
title: '#PSTip Get requests that are currently executing in a worker process'
author: Shay Levy
type: post
date: 2014-03-18T18:00:18+00:00
url: /2014/03/18/pstip-get-requests-that-are-currently-executing-in-a-worker-process/
categories:
  - Tips and Tricks
  - WMI
tags:
  - Tips and Tricks
  - WMI
---
**Note**: This tip requires PowerShell 2.0 or above.

IIS lets you view all worker processes of a server when you double click the<em> Worker Process</em> tile in IIS manager (inetmgr.exe).

![](/images/wp1.png)

![](/images/wp2.png)

If you double click a process you will get the requests that are currently running inside that worker process. The information gives you a good view of the current requests but this is only good for the server you&#8217;re currently working. Using PowerShell you can get the information from multiple servers and view all requests in one view.

Using WMI we can get the worker processes of a server by getting the instances of the _WorkerProcess_ class.

```
PS> Get-WmiObject WorkerProcess -Namespace root\WebAdministration -ComputerName IIS1
__GENUS          : 2
__CLASS          : WorkerProcess
__SUPERCLASS     : Object
__DYNASTY        : Object
__RELPATH        : WorkerProcess.ProcessId=4252
__PROPERTY_COUNT : 3
__DERIVATION     : {Object}
__SERVER         : IIS1
__NAMESPACE      : root\WebAdministration
__PATH           : \\IIS1\root\WebAdministration:WorkerProcess.ProcessId=4252
AppPoolName      : SharePoint - mysite
Guid             : 0fb641ca-b913-4628-99ff-6ffd13ee4b52
ProcessId        : 4252
PSComputerName   : IIS1
__GENUS          : 2
__CLASS          : WorkerProcess
__SUPERCLASS     : Object
__DYNASTY        : Object
__RELPATH        : WorkerProcess.ProcessId=8512
__PROPERTY_COUNT : 3
__DERIVATION     : {Object}
__SERVER         : IIS1
__NAMESPACE      : root\WebAdministration
__PATH           : \\IIS1\root\WebAdministration:WorkerProcess.ProcessId=8512
AppPoolName      : DefaultAppPool
Guid             : 0237506e-4924-45a8-b334-284202d83d8d
ProcessId        : 8512
PSComputerName   : IIS1
```

If you pipe the result to the _Get-Member_ cmdlet you will find a method called _GetExecutingRequests_.

```
PS> Get-WmiObject WorkerProcess -Namespace root\WebAdministration -ComputerName IIS1

   TypeName: System.Management.ManagementObject#root\WebAdministration\WorkerProcess
Name                 MemberType    Definition
----                 ----------    ----------
PSComputerName       AliasProperty PSComputerName = __SERVER
GetExecutingRequests Method        System.Management.ManagementBaseObject GetExecutingRequests()
GetState             Method        System.Management.ManagementBaseObject GetState()
AppPoolName          Property      string AppPoolName {get;set;}
Guid                 Property      string Guid {get;set;}
(...)
```

The _GetExecutingRequests_ method lets you see the requests that were executing at the time that the method was run. Requests execute fast so you might get empty results when you execute the method. To restrict the result to a specific pool name, use the _-Filter_ parameter.

```
Get-WmiObject WorkerProcess -Namespace root\WebAdministration -ComputerName IIS1 -Filter "AppPoolName='SharePoint - mysite'" |
Invoke-WmiMethod -Name GetExecutingRequests
__GENUS          : 2
__CLASS          : __PARAMETERS
__SUPERCLASS     :
__DYNASTY        : __PARAMETERS
__RELPATH        :
__PROPERTY_COUNT : 1
__DERIVATION     : {}
__SERVER         :
__NAMESPACE      :
__PATH           :
OutputElement    : {}
PSComputerName   :

__GENUS          : 2
__CLASS          : __PARAMETERS
__SUPERCLASS     :
__DYNASTY        : __PARAMETERS
__RELPATH        :
__PROPERTY_COUNT : 1
__DERIVATION     : {}
__SERVER         :
__NAMESPACE      :
__PATH           :
OutputElement    : {mysite, mysite, mysite}
PSComputerName   :
```

The resultant objects you want to take a look at are the ones that have a value in the _OutputElement_ property. The following is the result of the above highlighted _OutputElement_ property:

```
__GENUS          : 2
__CLASS          : HttpRequest
__SUPERCLASS     : Object
__DYNASTY        : Object
__RELPATH        :
__PROPERTY_COUNT : 14
__DERIVATION     : {Object}
__SERVER         :
__NAMESPACE      :
__PATH           :
ClientIPAddress  : 10.10.10.10
ConnectionId     : cf0000006001800f
CurrentModule    : ManagedPipelineHandler
GUID             :
HostName         : mysite
LocalIPAddress   : 11.11.11.11
LocalPort        : 80
PipelineState    : 128
SiteId           : 1202063871
TimeElapsed      : 1544
TimeInModule     : 1529
TimeInState      : 1529
Url              : /Pages/Default.aspx
Verb             : GET
PSComputerName   :
(...)
```


Using the following snippet you can create a custom object that captures the server name, application pool name and ID, and all relevant properties of the _OutputElement_ object.

```
$wp = Get-WmiObject WorkerProcess -Namespace root\WebAdministration -ComputerName IIS1,IIS2,IIS3

foreach($w in $wp)
{
    $w | Invoke-WmiMethod -Name GetExecutingRequests |
    Select-Object -ExpandProperty OutputElement | Foreach-Object{
        [PSCustomObject]@{
            AppPoolName     = $w.AppPoolName
            ProcessId       = $w.ProcessId
            ClientIPAddress = $_.ClientIPAddress
            CurrentModule   = $_.CurrentModule
            HostName        = $_.HostName
            LocalIPAddress  = $_.LocalIPAddress
            LocalPort       = $_.LocalPort
            SiteId          = $_.SiteId
            Url             = $_.Url
            Verb            = $_.Verb
            PSComputerName  = $w.__SERVER
        }
    }
}

AppPoolName     : SharePoint - mysite
ProcessId       : 5080
ClientIPAddress : 10.10.10.10
CurrentModule   : IIS Web Core
HostName        : mysite
LocalIPAddress  : 11.11.11.11
LocalPort       : 80
SiteId          : 1202063871
Url             : /Pages/_layouts/IMAGES/MasterPages/Dropdown.png
Verb            : GET
PSComputerName  : IIS1
```

