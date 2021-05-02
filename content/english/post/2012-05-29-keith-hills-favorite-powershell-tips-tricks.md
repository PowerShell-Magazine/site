---
title: 'Keith Hill’s Favorite PowerShell Tips & Tricks'
author: Keith Hill
type: post
date: 2012-05-29T18:00:56+00:00
url: /2012/05/29/keith-hills-favorite-powershell-tips-tricks/
views:
  - 15791
post_views_count:
  - 3492
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
### ScriptBlock Binding to Pipeline Bound Parameters

I see many PowerShell snippets on Stack Overflow that look similar to one below:

<pre>Get-ChildItem *.txt | Foreach {
   Rename-Item -Path $_ -NewName "$($_.basename).bak"
}</pre>

The use of the ForEach-Object cmdlet above is unnecessary because Rename-Item accepts pipeline input for both the Path and NewName parameters.   You can see this in the following example.

<pre>PS&gt; $obj = New-Object PSObject –Property `
           @{Path='C:\Users\Keith\foo.txt';NewName='bar.txt'}
PS&gt; $obj | Rename-Item –WhatIf
What if: Performing operation "Rename File" on Target
"Item: C:\Users\Keith\foo.txt Destination: C:\Users\Keith\bar.txt".</pre>

You may be thinking that while this may be an interesting academic exercise, how is it any better than the original version that uses ForEach-Object?  Well, it is not better at this point but PowerShell has one more trick up its sleeve to help us achieve a more efficient expression of this rename operation.  The trick is that PowerShell will accept a snippet of script called a script block for any parameter that is pipeline bound.  You can see if a parameter is pipeline bound via Get-Help e.g.:

<pre>PS&gt; Get-Help Rename-Item
...
-LiteralPath
    ...
    <strong>Accept pipeline input? true (ByPropertyName)</strong>

-Path
    ...
    <strong>Accept pipeline input? true (ByValue, ByPropertyName)</strong>

-NewName
    ...
    Position?                    2
    <strong>Accept pipeline input? true (ByPropertyName)</strong></pre>

This information tells us that the LiteralPath, Path, and NewName parameters accept pipeline input.   The pipeline output of Get-ChildItem is bound to the LiteralPath parameter of the Rename-Item cmdlet.  We can use the script block binding trick to specify the NewName.  The updated version of this one-liner is shown below.

<pre>PS&gt; Get-ChildItem *.txt | Rename-Item -NewName {"$($_.BaseName).bak"}</pre>

In this version the script block {&#8220;$($\_.BaseName).bak&#8221;} is evaluated and used as the argument to the NewName parameter.   Because the script block is used to produce an argument for a pipeline bound parameter, it is legal to reference pipeline input via the $\_ variable.

### Using PowerShell v3’s Out-GridView for Multiple-Selection

This tip is pretty simple but occasionally very useful.  The updated Out-GridView cmdlet in PowerShell v3 supports the PassThru parameter.  In addition, Out-GridView supports multiple-selection of the items passed into as well as the ability to cancel an operation.  This can be very handy for cases where, for example, you may want to select from a list of processes to stop.:

<pre>PS&gt; Get-Process devenv |
    Select Name,Id,MainWindowTitle |
    Out-GridView -PassThru | Stop-Process</pre>

This command displays the Out-GridView dialog as shown below.  I can see which instance of Visual Studio I want to kill based on the MainWindowTitle property.  I can select one or more devenv processes.  If I press OK then the processes I selected will be stopped.  If I press the Cancel button on the Out-GridView dialog, the pipeline is stopped and no processes are stopped.

![](/images/KeithTricks1.png)

### Using the PowerShell Community Extension’s Show-Tree Command to Explore Providers

This tip requires the free [PowerShell Community Extensions][1] (PSCX) module.  PSCX is a set of general purpose PowerShell commands.  One of the commands it provides is Show-Tree which is very useful for exploring PowerShell drives like:

  * WSMan:\
  * Cert:\
  * HKLM:\
  * IIS:\ (if you have imported the WebAdministration module)

Normally, if you want to explore a drive you would use Windows Explorer. Unfortunately Windows Explorer has no visibility into PowerShell drives except for those that are based on the file system.  Equally unfortunate is that drives like WSMan: and IIS: hide a lot of functionality i.e. the contained functionality is not very discoverable.  This is where the Show-Tree comes in very handy.  It can display information in a PowerShell drive very much like dostree displays file system structure in a console.  For example, here is sample output from running Show-Tree on the IIS:\ drive:

<pre style="line-height: 16px;">PS&gt; Show-Tree IIS:\ -Depth 3
IIS:\
├──AppPools
│  ├──ASP.NET v4.0
│  │  └──WorkerProcesses
│  ├──ASP.NET v4.0 Classic
│  │  └──WorkerProcesses
│  ├──Classic .NET AppPool
│  │  └──WorkerProcesses
│  └──DefaultAppPool
│     └──WorkerProcesses
├──Sites
│  └──Default Web Site
│     ├──aspnet_client
│     └──Blog
└──SslBindings</pre>

In general, PowerShell drives have items that can be either Container or Leaf items.  What you see above are container items only.  There can also be ItemProperties which are usually where the action is when it comes to changing settings on a PowerShell provider-based drive.  For example, here is listing of item properties for the DefaultAppPool:

<pre style="line-height: 16px;">PS&gt; Show-Tree IIS:\AppPools\DefaultAppPool -ShowProperty
IIS:\AppPools\DefaultAppPool
├──Property: applicationPoolSid = S-1-5-82-3006700770-424185619-1745488364-7...
├──Property: Attributes = Microsoft.IIs.PowerShell.Framework.ConfigurationAt...
├──Property: autoStart = True
├──Property: ChildElements = Microsoft.IIs.PowerShell.Framework.Configuratio...
├──Property: CLRConfigFile =
├──Property: cpu = Microsoft.IIs.PowerShell.Framework.ConfigurationElement
├──Property: ElementTagName = add
├──Property: enable32BitAppOnWin64 = False
├──Property: enableConfigurationOverride = True
├──Property: failure = Microsoft.IIs.PowerShell.Framework.ConfigurationElement
├──Property: ItemXPath = /system.applicationHost/applicationPools/add[@name=...
├──<strong>Property: managedPipelineMode = Integrated</strong>
├──Property: managedRuntimeLoader = webengine4.dll
├──<strong>Property: managedRuntimeVersion = v2.0</strong>
├──Property: Methods = Microsoft.IIs.PowerShell.Framework.ConfigurationMetho...
├──Property: passAnonymousToken = True
├──Property: processModel = Microsoft.IIs.PowerShell.Framework.Configuration...
├──Property: queueLength = 1000
├──Property: recycling = Microsoft.IIs.PowerShell.Framework.ConfigurationEle...
├──Property: Schema = Microsoft.IIs.PowerShell.Framework.ConfigurationElemen...
├──Property: startMode = OnDemand
├──Property: state = Started
├──Property: workerProcesses = Microsoft.IIs.PowerShell.Framework.Configurat...
└──WorkerProcesses
...</pre>

This view of the IIS:\ drive reveals much more information such as which _Managed PipelineMode_ is in use by the app pool as well as which version of the .NET runtime it uses.  With this information it becomes much easier to figure how to change these settings:

<pre>PS&gt; Set-ItemProperty IIS:\AppPools\DefaultAppPool managedRuntimeVersion v4.0</pre>

Discovering these properties and their locations is more than half the battle to figuring out how to change their values.

[1]: http://pscx.codeplex.com/