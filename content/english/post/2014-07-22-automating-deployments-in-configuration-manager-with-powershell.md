---
title: Automating deployments in Configuration Manager with PowerShell
author: Deepak Dhami
type: post
date: 2014-07-22T16:00:00+00:00
url: /2014/07/22/automating-deployments-in-configuration-manager-with-powershell/
categories:
  - How To
  - SCCM
tags:
  - How To
  - SCCM
---
### Introduction

First of all, thanks to PowerShell magazine for giving me this great honour to post an article here.

There have been a few articles about Configuration Manager in PowerShell Magazine&#8211;one by PowerShell MVP Trevor Sullivan is especially worth [reading][1].

One of the main responsibilities of being a ConfigMgr administrator is to perform application deployments. After you have all the applications packaged and setup in the ConfigMgr Software Library, you would probably want to deploy them to the end users’ machines.

In this post, I’ll be focusing only on the deployments targeting to the devices using the query-based Collection Membership rules, as they are easier to maintain than Direct Membership Query Rule when you are deploying applications to a large number of devices.

### Background on the Manual Process

In order to deploy applications to devices, one has to create logical groupings of resources called Collections and deploy applications to them. Every Collection has rules that are used to configure the members of the Collection. You’ll have more details [here][2].

A Query Rule uses a WQL query statement which defines the membership criteria for the Collection. A simple query that we use looks like below:

<pre class="brush: sql; title: ; notranslate" title="">select * from SMS_R_System where SMS_R_System.NetbiosName in ("null","Computer1","Computer2")
</pre>

In order to add a new machine name to the Query Rule, one has to use the ConfigMgr Console. See below clip:

{{< youtube 93Yoa3dHuZE >}}

### Meet “PowerShell”, the automation engine

Before we proceed any further, I would like to point out that one can go in two ways to automate Configuration Manager using PowerShell:

  1. Through SMS Provider
  2. Using the ConfigurationManager PowerShell module (since CM 2012 SP1)

For this post I’ll be concentrating mainly on the SCCM cmdlets , but this can very well be done leveraging the SMS Provider with PowerShell (WMI/CIM cmdlets). For more info on this visit  ECM MVP Kaido’s site [cm12sdk.net][3]. There are few things which you need to know before you start using the Configuration Manager PowerShell module. Read my post on &#8220;[Getting Started][4]&#8220;.

Once you have the CM Module loaded and your current location set to the CMSite provider, you can run the cmdlets.

**Note:** You need to explicitly load the module as it doesn’t reside in the $env:PSModulePath and if your current location is not to the CMSite PSDrive then cmdlets throw an error.

The easiest way to get these pre-requisites out of hand is to use the ConfigMgr console to open the PowerShell host. Click on the top left corner then “**Connect via Windows PowerShell**”.

![](/images/17.png)

This pops up a PowerShell console with your current location set to the three letter site code e.g DEX for my lab (the same site your CM console is connected to).

Now let&#8217;s use the _Get-Command_ cmdlet to discover the cmdlets that work with the Query Membership Rule, Please note the three letter site code e.g DEX in the PS prompt showing my current location in the CMSite PSDrive:

<pre class="brush: powershell; title: ; notranslate" title="">PS DEX:\&gt; Get-Command -Noun *QueryMembershipRule -Module ConfigurationManager
CommandType     Name                                              ModuleName
-----------     ----                                               ----------
Cmdlet         Add-CMDeviceCollectionQueryMembershipRule         ConfigurationManager
Cmdlet         Add-CMUserCollectionQueryMembershipRule            ConfigurationManager
Cmdlet         Get-CMDeviceCollectionQueryMembershipRule         ConfigurationManager
Cmdlet         Get-CMUserCollectionQueryMembershipRule           ConfigurationManager
Cmdlet         Remove-CMDeviceCollectionQueryMembershipRule       ConfigurationManager
Cmdlet         Remove-CMUserCollectionQueryMembershipRule         ConfigurationManager
</pre>

Let&#8217;s get to task of adding a machine name to the Query Membership Rule for the Collection &#8220;Quest AD&#8221; (same collection in the video above).

First let&#8217;s get the Query Membership Rule and then we will modify it, remove the old one, and save the new one back.

<pre class="brush: powershell; title: ; notranslate" title="">$Oldquery= Get-CMDeviceCollectionQueryMembershipRule -CollectionName "Quest AD" -RuleName "Quest AD QueryRule" –Verbose
</pre>

Once you have the Query Rule, you have to manipulate the QueryExpression property a bit to add the machine name at the end in the proper WQL format (proper quotes and comma).

Note the -Verbose switch with the CM cmdlets gives the WQL the cmdlet uses. It&#8217;s a good idea to use it.

<pre class="brush: powershell; title: ; notranslate" title="">$Oldquery.QueryExpression = $Oldquery.QueryExpression.TrimEnd(")") + ',"DexSCCM")'
</pre>

Now we have the QueryExpression ready, time to remove the old one from the collection, and save the new one.

<pre class="brush: powershell; title: ; notranslate" title="">Remove-CMDeviceCollectionQueryMembershipRule -CollectionName "Quest AD" -RuleName "Quest AD QueryRule" -Verbose -Force
Add-CMDeviceCollectionQueryMembershipRule -CollectionName "Quest AD" -RuleName $Oldquery.RuleName -QueryExpression $Oldquery.QueryExpression -Verbose
</pre>

One can easily put the appending machine name logic in a function to perform this operation.

Below is a short video showing the above in action:

{{< youtube vTuV9chOTf8 >}}

### Conclusion

The above post just gives you a hint of how PowerShell can be used in Configuration Manager. Based on your process requirement you can automate the manual repetitive tasks, but before you go on the automation spree first understand the ins and outs of the process. Do a lot of testing and always ask the awesome PowerShell community for feedback.

Feel free to contact me on [Twitter][5] / [Facebook][6] (DexterPOSH is my handle in case you were wondering).

[1]: /2012/04/16/automating-configuration-manager-with-windows-powershell/
[2]: http://technet.microsoft.com/en-in/library/gg712295.aspx
[3]: http://cm12sdk.net/
[4]: http://dexterposh.blogspot.com/2014/06/powershell-sccm-2012-getting-started.html
[5]: https://twitter.com/DexterPOSH
[6]: https://www.facebook.com/DexterPOSH