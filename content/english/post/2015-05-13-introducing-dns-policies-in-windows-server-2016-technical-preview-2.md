---
title: Introducing DNS Policies in Windows Server 2016 Technical Preview 2
author: Jan Egil Ring
type: post
date: 2015-05-13T16:00:25+00:00
url: /2015/05/13/introducing-dns-policies-in-windows-server-2016-technical-preview-2/
views:
  - 17763
post_views_count:
  - 7954
categories:
  - News
tags:
  - News

---
In this article, we will have a look at DNS Policies &#8211; a new feature in the DNS Server role introduced in Windows Server 2016 Technical Preview 2 (TP2):

_You can configure DNS policies to specify how a DNS server responds to DNS queries. DNS responses can be based on client IP address (location), time of the day, and several other parameters. DNS policies enable location-aware DNS, traffic management, load balancing, split-brain DNS, and other scenarios._

In the Technical Preview, DNS Policies can be managed through PowerShell only. Whether the feature will be manageable from the DNS Server MMC-console or any other GUI tools is currently unknown, but as Jeffrey Snover [stated on Twitter][1] recently, PowerShell management is created before layering a GUI later on for most new things in Windows Server:

![](/images/dns1.png)

Before we can demonstrate the new feature, we must first install the DNS Server role and create a zone on a computer running Windows Server 2016 TP2:

```powershell
# Install DNS Server
Install-WindowsFeature -Name DNS

# List all DNS policy cmdlets
Get-Command -Module DnsServer -Name *policy*

# Update help for the DnsServer module; the help files already have a lot of content and examples
Update-Help -Module DnsServer

# Create a test zone
Add-DnsServerPrimaryZone -Name powershell.no -ZoneFile powershell.no.dns
```

Next we create traffic management policies:

```powershell
# The first two commands create client subnets by using the Add-DnsServerClientSubnet cmdlet. The client subnets are for clients in Oslo and clients in Trondheim.
Add-DnsServerClientSubnet -Name OsloSubnet -IPv4Subnet "10.0.1.0/22" -PassThru
Add-DnsServerClientSubnet -Name TrondheimSubnet -IPv4Subnet "10.0.2.0/24" -PassThru
```


![](/images/dns2.png)

```powershell
# The next two commands create zone scopes for Oslo and Trondheim by using the Add-DnsServerZoneScope cmdlet.

Add-DnsServerZoneScope -ZoneName powershell.no -Name "OsloZoneScope" -PassThru
Add-DnsServerZoneScope -ZoneName powershell.no -Name "TrondheimZoneScope" -PassThru
```


![](/images/dns3.png)

```powershell
# The next two commands add resource records for the zone powershell.no by using the Add-DnsServerResourceRecord cmdlet.
# The name for both records is the same, staging, but the two records point to different addresses. The records also have different scopes.
Add-DnsServerResourceRecord -ZoneName powershell.no -A -Name staging -IPv4Address "10.0.1.10" -ZoneScope "OsloZoneScope" -PassThru
Add-DnsServerResourceRecord -ZoneName powershell.no -A -Name staging -IPv4Address "10.0.2.10" -ZoneScope "TrondheimZoneScope" -PassThru
```


![](/images/dns4.png)

```powershell
# The final two commands create two policies. The policies allow queries for members of different subnets.
# The policies differ in scope, so that some clients receive one response to a query, while other clients receive a different response to the same query.
Add-DnsServerQueryResolutionPolicy -Name "OsloPolicy" -Action ALLOW -ClientSubnet "eq,OsloSubnet" -ZoneScope "OsloZoneScope,1" -ZoneName powershell.no -PassThru
Add-DnsServerQueryResolutionPolicy -Name "TrondheimPolicy" -Action ALLOW -ClientSubnet "eq,TrondheimSubnet" -ZoneScope "TrondheimZoneScope,1" -ZoneName powershell.no -PassThru
```


![](/images/dns5.png)

Now we are ready to test if the new policies work as desired. First, we must configure the computers used for testing to use the Windows Server 2016 TP2 DNS Server for DNS queries:

```powershell
Set-DnsClientServerAddress -InterfaceAlias Ethernet -ServerAddresses 10.0.1.200
```


And then, we’ll use Resolve-DnsName from a computer in the Oslo subnet (10.0.1.0):

![](/images/dns6.png)

Next, we\`re doing the same DNS query from a computer in the Trondheim subnet (10.0.2.0):

![](/images/dns7.png)

As we can see, the expected IP addresses are returned in both scenarios.

Let’s also test another available scenario&#8211;Time of the day.

First, we modify the OsloPolicy to apply in a restricted time:

```powershell
Set-DnsServerQueryResolutionPolicy -Name "OsloPolicy" -ZoneName powershell.no -TimeOfDay "EQ,22:10-23:00" -PassThru
```


Then we create a function for testing purposes, which will clear the DNS client cache, output the current time and perform a DNS query.


```powershell
function Test-DnsName {
	param(
		[string]$Name
	)
    Clear-DnsClientCache

    Get-Date
    Resolve-DnsName -Name $Name
}
Test-DnsName -Name staging.powershell.no
```

When running Test-DnsName before the allowed time of day we get an error stating that the DNS name does not exist, while after the allowed time of day the query returns the desired result:

![](/images/dns8.png)

This concludes our introduction to DNS Policies in Windows Server 2016, a feature that will be very useful in real world scenarios when it becomes available for production use.

**Resources**

[What&#8217;s New in DNS Server in Windows Server Technical Preview][2] (TechNet)

[Domain Name System (DNS) Server Cmdlets][3] (MSDN)

[1]: https://twitter.com/jsnover/status/596695949306515456
[2]: https://technet.microsoft.com/en-us/library/dn765484.aspx
[3]: https://msdn.microsoft.com/en-us/library/jj649850.aspx