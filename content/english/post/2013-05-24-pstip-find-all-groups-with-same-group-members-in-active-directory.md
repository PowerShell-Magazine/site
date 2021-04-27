---
title: '#PSTip Find all groups with same group members in Active Directory'
author: Jaap Brasser
type: post
date: 2013-05-24T18:00:00+00:00
url: /2013/05/24/pstip-find-all-groups-with-same-group-members-in-active-directory/
categories:
  - Active Directory
  - Tips and Tricks
tags:
  - Active Directory
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Today I was asked if there was a way to find out which groups have the same group members. This is possible by parsing the output of a DirectoryServices.DirectorySearcher or [adsisearcher] class. The following example groups the results and sorts by the number of groups that have the same group membership:


    $Searcher = [adsisearcher]'(member=*)'
    $Searcher.PageSize = 500
    $Searcher.FindAll() |  ForEach-Object {
        New-Object -TypeName PSCustomObject -Property @{
            DistinguishedName = $_.Properties.distinguishedname[0]
            Member = $_.Properties.member -join ';'
        }
    } | Group-Object -Property member |
    Where-Object {$_.Count -gt 1} |
    Sort-Object -Property Count -Descending

The output looks similar to this:

<pre class="brush: powershell; title: ; notranslate" title="">Count Name                      Group
----- ----                      -----
   15 CN=Domain Users,CN=Use... {@{distinguishedname=CN=test123...
   13 CN=Domain Users,CN=Use... {@{distinguishedname=CN=test456...
To get the group names and the members, the output from the Group-Object cmdlet should be expanded by utilizing Select-Object –ExpandProperty. This output will be piped to Export-Csv which will generate a report containing all groups in Active Directory that have exactly the same members:


    $Searcher = [adsisearcher]'(member=*)'
    $Searcher.PageSize = 500
    $Searcher.FindAll() | ForEach-Object {
        New-Object -TypeName PSCustomObject -Property @{
            DistinguishedName = $_.Properties.distinguishedname[0]
            Member = $_.Properties.member -join ';'
        }
    } | Group-Object -Property member | Where-Object {$_.Count -gt 1} |
    Sort-Object -Property Count -Descending |
    Select-Object -ExpandProperty Group |
    Export-Csv -Path GroupWithIdenticalMembership.csv -NoTypeInformation

The output of this command is as follows:

![](/images/Jaap_AD1.png)