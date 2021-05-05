---
title: Parsing Failover Cluster Validation Report in PowerShell
author: Ravikanth C
type: trending
date: 2020-01-30T13:22:02+00:00
url: /2020/01/30/parsing-failover-cluster-validation-report-in-powershell/
featured_image: /wp-content/uploads/2012/01/logo-250x250.png
is_featured:
  - 'true'
post_views_count:
  - 2306
views:
  - 2169
categories:
  - FailoverCluster
tags:
  - Scripts

---
If you have ever worked with the Test-Cluster command in the failover clustering module, you will know that this command generates an HTML report. This is visually good, for like IT managers, but not very appealing to people are automating infrastructure build process. There is no way this command provides any passthru type of functionality through which it returns the result object that can be easily parsed or used in PowerShell.

{{< figure src="/images/failover1.png" >}} {{< load-photoswipe >}}

As a part of some larger automation that I was building, I needed to parse the validation result into a PowerShell object that can be used later in the orchestration. Parsing HTML isn&#8217;t what I needed but a little of digging gave some clues about the XML report that gets generated when we run this command.

Behind the scenes, the Test-Cluster command generates an XML every time it was run. This XML gets stored at C:\Windows\Temp. Looking at the XML you can easily notice that the schema was really designed to generate the HTML easily. So, it took a few minutes to understand how the tests are categorized and come up with the below script.

```powershell
[CmdletBinding()]
param
(
    [Parameter(Mandatory = $true)]
    [String]
    $ValidationXmlPath
)


$xml = (Get-Content -Path $ValidationXmlPath)
$channels = $xml.Report.Channel.Channel

$validationResultArray = New-Object -TypeName System.Collections.ArrayList

foreach ($channel in $channels)
{
    if ($channel.Type -eq 'Summary')
    {
        $channelSummaryHash = [PSCustomObject]@{}
        $summaryArray = New-Object -TypeName System.Collections.ArrayList

        $channelId = $channel.id
        $channelName = $channel.ChannelName.'#cdata-section'        
        
        foreach ($summaryChannel in $channels.Where({$_.SummaryChannel.Value.'#cdata-section' -eq $channelId}))
        {
            $channelTitle = $summaryChannel.Title.Value.'#cdata-section'
            $channelResult = $summaryChannel.Result.Value.'#cdata-section'
            $channelMessage = $summaryChannel.Message.'#cdata-section'
    
            $summaryHash = [PSCustomObject] @{
                Title = $channelTitle
                Result = $channelResult
                Message = $channelMessage
            }
    
            $null = $summaryArray.Add($summaryHash)
        }
    
        $channelSummaryHash | Add-Member -MemberType NoteProperty -Name Category -Value $channelName
        $channelSummaryHash | Add-Member -MemberType NoteProperty -Name Results -Value $summaryArray
    
        $null = $validationResultArray.Add($channelSummaryHash)
    }

}

return $validationResultArray
```

The input to the above script is the XML file that gets generated at C:\Windows\Temp. Once you run the script, you should see the output similar to what is shown below.

{{< figure src="/images/failover2.png" >}} 

I have only added the property values that I really need in my scripts but you can look at the XML and then easily modify the above script to add other details as you need. 

Comment on this [Gist](https://gist.github.com/rchaganti/b4fc88f1615c6810b2c46b647ae4fe96) if you have any suggestions or have you version of the script.