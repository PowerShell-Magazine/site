---
title: Creating and configuring a SharePoint 2013 search service application
author: Ravikanth C
type: post
date: 2013-04-29T16:37:24+00:00
url: /2013/04/29/creating-and-configuring-a-sharepoint-2013-search-service-application/
categories:
  - How To
  - SharePoint
tags:
  - How To
  - SharePoint

---
The [Search Service Application][1] (SSA) in SharePoint provides the content crawl and search functionality. In SharePoint 2010, the Central Administration (CA) site has the option to configure and customize the SSA. This gave the novice and _Graphical Interface Administrators_ (GIA)&#8211;this is what I call people who don&#8217;t use PowerShell!&#8211;an option to add or remove search components such as crawlers, index and query roles, and create mirror copies of search index, etc.

Things have [changed quite a bit in SharePoint 2013][2]. Now, there are more components in the SharePoint SSA and to a GIA&#8217;s nightmare, there is no way to edit the search topology in Central Administration. We can create a new search service instance but we cannot configure the same.

There are several steps involved in deploying SharePoint Search Service Application.

  * <span style="line-height: 13px;">Creating a SharePoint Search Service Application</span>
  * Creating a SSA Proxy
  * Creating an administration component
  * Adding content processing, web analytics, crawler, and query components
  * Adding index component and search index replicas

In this article, I will show you how to create and configure the SharePoint 2013 SSA using PowerShell and customize search topology.

Let us start with an example. Assume that we have a SharePoint 2013 farm with multiple servers hosting different roles. In this farm, we have two web front-end servers, two SQL database servers, and two application servers on which we want to configure all our SharePoint search service components.

For the sake of brevity, the following diagram shows only the SharePoint application servers on which we want to configure the search instance.

![](/images/ssa.png)

So, as we see above, we will configure the search administration component on &#8216;APP Server 01&#8217; and rest all components on both servers for high availability. Also, look at the way index is shown. We will configure two index partitions and ensure both servers have a replica of the index partition.

Note: The following commands need to be run on a server running SharePoint software and you need to load the SharePoint PowerShell snap-in or run these commands at the SharePoint Management Shell. You can load the SharePoint PowerShell snap-in using the following command:

<pre class="brush: powershell; title: ; notranslate" title="">Add-PSSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue
</pre>

In this example, we will be running all the commands on &#8216;APP Server 01&#8217;.

The first step in the search service configuration process is to create the search service application. We need a application pool for the search service and we will call it &#8216;SharePoint_SearchApp&#8217;. Also, we need an account for the search service application pool. We will use a domain user account (in my test domain) called SPSearchPool for this purpose. The following code shows how to create a search service application.

```
$App1 = "APP-Server-01"
$APP2 = "APP-Server-02"
$SearchAppPoolName = "SharePoint_SearchApp"
$SearchAppPoolAccountName = "TestDomain\SPSearchPool"
$SearchServiceName = "SharePoint_Search_Service"
$SearchServiceProxyName = "SharePoint_Search_Proxy"
$DatabaseName = "SharePoint_Search_AdminDB"

#Create a Search Service Application Pool
$spAppPool = New-SPServiceApplicationPool -Name $SearchAppPoolName -Account $SearchAppPoolAccountName -Verbose

#Start Search Service Instance on all Application Servers
Start-SPEnterpriseSearchServiceInstance $App1 -ErrorAction SilentlyContinue
Start-SPEnterpriseSearchServiceInstance $App2 -ErrorAction SilentlyContinue
Start-SPEnterpriseSearchQueryAndSiteSettingsServiceInstance $App1 -ErrorAction SilentlyContinue
Start-SPEnterpriseSearchQueryAndSiteSettingsServiceInstance $App2 -ErrorAction SilentlyContinue

#Create Search Service Application
$ServiceApplication = New-SPEnterpriseSearchServiceApplication -Partitioned -Name $SearchServiceName -ApplicationPool $spAppPool.Name -DatabaseName $DatabaseName

#Create Search Service Proxy
New-SPEnterpriseSearchServiceApplicationProxy -Partitioned -Name $SearchServiceProxyName -SearchApplication $ServiceApplication
```

We have just created the search service application. Now, we need to configure different search components as described above and then finalize the search topology. Let&#8217;s start with creation of the new search topology. For this, we first need to clone the existing active search topology:

```
$clone = $ServiceApplication.ActiveTopology.Clone()
$App1SSI = Get-SPEnterpriseSearchServiceInstance -Identity $app1
$App2SSI = Get-SPEnterpriseSearchServiceInstance -Identity $app2
```


Once we have the cloned topology, we can start creating the search components.

```
#We need only one admin component
New-SPEnterpriseSearchAdminComponent –SearchTopology $clone -SearchServiceInstance $App1SSI

#We need two content processing components for HA
New-SPEnterpriseSearchContentProcessingComponent –SearchTopology $clone -SearchServiceInstance $App1SSI
New-SPEnterpriseSearchContentProcessingComponent –SearchTopology $clone -SearchServiceInstance $App2SSI

#We need two analytics processing components for HA
New-SPEnterpriseSearchAnalyticsProcessingComponent –SearchTopology $clone -SearchServiceInstance $App1SSI
New-SPEnterpriseSearchAnalyticsProcessingComponent –SearchTopology $clone -SearchServiceInstance $App2SSI

#We need two crawl components for HA
New-SPEnterpriseSearchCrawlComponent –SearchTopology $clone -SearchServiceInstance $App1SSI
New-SPEnterpriseSearchCrawlComponent –SearchTopology $clone -SearchServiceInstance $App2SSI

#We need two query processing components for HA
New-SPEnterpriseSearchQueryProcessingComponent –SearchTopology $clone -SearchServiceInstance $App1SSI
New-SPEnterpriseSearchQueryProcessingComponent –SearchTopology $clone -SearchServiceInstance $App2SSI
```

We created all the search components as per the diagram shown above except the index partitions. As a best practice, we want to place the search index primary copy and replica at different locations on the application servers. The following commands define the locations for the primary and replica copies and then create the index components as required.

```
#Set the primary and replica index location; ensure these drives and folders exist on application servers
$PrimaryIndexLocation = "E:\Data"
$ReplicaIndexLocation = "F:\Data"

#We need two index partitions and replicas for each partition. Follow the sequence.
New-SPEnterpriseSearchIndexComponent –SearchTopology $clone -SearchServiceInstance $App1SSI -RootDirectory $PrimaryIndexLocation -IndexPartition 0
New-SPEnterpriseSearchIndexComponent –SearchTopology $clone -SearchServiceInstance $App2SSI -RootDirectory $ReplicaIndexLocation -IndexPartition 0
New-SPEnterpriseSearchIndexComponent –SearchTopology $clone -SearchServiceInstance $App2SSI -RootDirectory $PrimaryIndexLocation -IndexPartition 1
New-SPEnterpriseSearchIndexComponent –SearchTopology $clone -SearchServiceInstance $App1SSI -RootDirectory $ReplicaIndexLocation -IndexPartition 1
```

Finally, we activate the cloned topology to bring the changes into effect.

<pre class="brush: powershell; title: ; notranslate" title="">$clone.Activate()
</pre>

This will take a while to finalize the changes. Once the re-configuration of search topology is complete, we can verify the same by running the following commands.

<pre class="brush: powershell; title: ; notranslate" title="">$ssa = Get-SPEnterpriseSearchServiceApplication
Get-SPEnterpriseSearchTopology -Active -SearchApplication $ssa
</pre>
Once the configuration is complete, if you check the search topology on CA site, you should see something similar to what is shown below.

![](/images/searchCA.png)

In this article, we looked at only a two-server search topology. But, in real-world, there may be a bigger implementation. However, the steps mentioned in this article can easily be extended to support any size of the farm.

[1]: http://technet.microsoft.com/en-in/library/ee792877.aspx
[2]: http://technet.microsoft.com/en-in/library/jj219705.aspx