---
title: 'Weekly Module Spotlight: ImportExcel'
author: Ravikanth C
type: post
date: 2019-09-09T04:00:30+00:00
url: /2019/09/09/weekly-module-spotlight-importexcel/
post_views_count:
  - 9857
views:
  - 9379
categories:
  - Module Spotlight
  - ImportExcel
tags:
  - Modules

---
In my previous job, I worked at a customer site that had multiple Windows and Unix Servers. One of the every day tasks was to report the disk space utilization from these servers. When I joined the customer site, engineers there used to collect the statistics manually and create an Excel spreadsheet manually. This was not just time consuming but boring as well. This task is something that needs to be automated. Period.

So, I went on to create a rather long VBScript that uses WMI for collecting disk usage statistics from Windows servers and uses Excel COM object to generate spreadsheets that contain the reports. It certainly made my job easy. I just had to run this sitting at my local workstation and within a few seconds I would have the Excel report that I can mail to the IT manager.

But, those of you who worked on Excel COM objects and PowerShell would know that it is not the best thing. Working with VBScript is a pain. When that combined with COM object, the pain of writing and testing a script increases a few folds.

You would be glad to hear that you don&#8217;t have to do that anymore if you know PowerShell even a little bit. Thanks to [ImportExcel](https://github.com/dfinke/ImportExcel).

ImportExcel allows you to read and write Excel files without installing Microsoft Excel on your system. With this module, there is no need to bother with the cumbersome Excel COM-object. With ImportExcel, creating Tables, Pivot Tables, Charts and much more has becomes a lot easier.

Before you try any of the following examples, install ImportExcel module from the [PowerShell Gallery](https://www.powershellgallery.com/packages/ImportExcel)

Here is the simple first example for you!

```powershell
Get-Process | Select-Object Company, Name, Handles | Export-Excel
```

This a command exports the values of selected properties from the process object and opens an Excel spreadsheet automatically.

Here is another example from [ImportExcel GitHub repository](https://github.com/dfinke/ImportExcel/blob/master/Examples/Charts/ChartAndTrendlines.ps1) that generates charts.

```powershell
$xlfile = "$env:TEMP\trendLine.xlsx"
Remove-Item $xlfile -ErrorAction SilentlyContinue

$data = ConvertFrom-Csv @"
Region,Item,TotalSold
West,screws,60
South,lemon,48
South,apple,71
East,screwdriver,70
East,kiwi,32
West,screwdriver,1
South,melon,21
East,apple,79
South,apple,68
South,avocado,73
"@

$cd = New-ExcelChartDefinition -XRange Region -YRange TotalSold -ChartType ColumnClustered -ChartTrendLine Linear
$data | Export-Excel $xlfile -ExcelChartDefinition $cd -AutoNameRange -Show
```

Finally, here is something I showed at the PowerShell Conference Europe 2019. This uses the speaker and session data JSON and generates a spreadsheet.

```powershell
if (-not (Get-Module -ListAvailable -Name ImportExcel -ErrorAction SilentlyContinue))
{
    Install-Module -Name ImportExcel -Force
}

$speakersJson = 'https://raw.githubusercontent.com/psconfeu/2019/master/data/speakers.json'
$sessionsJson = 'https://raw.githubusercontent.com/psconfeu/2019/master/sessions.json'

$speakers = ConvertFrom-Json (Invoke-WebRequest -UseBasicParsing -Uri $speakersJson).content
$sessions = ConvertFrom-Json (Invoke-WebRequest -UseBasicParsing -Uri $sessionsJson).content
```

### All Sessions Sheet

```powershell
$sessions | Select-Object Name, Starts, Ends, Track, Speaker | 
            Export-Excel -Path .\psconfeu2019.xlsx -WorksheetName 'All Tracks' `
            -Title 'PowerShell Conference EU 2019 - Sessions' `
            -TitleBold -TitleFillPattern DarkDown -TitleSize 20 `
            -TableStyle Medium6 -BoldTopRow
```

### Track sheets
```powershell
foreach ($i in 1..3)
{
    $trackSessions = $sessions.Where({$_.Track -eq "Track $i"})
    $trackSessions | Select-Object Name, Starts, Ends, Speaker |
        Export-Excel -Path .\psconfeu2019.xlsx -WorksheetName "Track $i" `
        -Title 'PowerShell Conference EU 2019 - Track $i' `
        -TitleBold -TitleFillPattern DarkDown -TitleSize 20 `
        -TableStyle Medium6 -BoldTopRow        
}
```

### Add Speakers sheet
```powershell
$speakers | Export-Excel -Path .\psconfeu2019.xlsx -WorksheetName 'Speakers' `
    -Title 'PowerShell Conference EU 2019 - Speakers' `
    -TitleBold -TitleFillPattern DarkDown -TitleSize 20 `
    -TableStyle Medium6 -BoldTopRow
```

### Add chart for speaker country number

```powershell
$chartDef = New-ExcelChart -Title 'PowerShell Conference EU 2019 - Speakers' `
                    -ChartType ColumnClustered `
                    -XRange Name -YRange Count `
                    -Width 800 -NoLegend -Column 3 

$speakers | Group-Object -Property Country | Select-Object Name, Count |  Sort-Object -Property Count -Descending |
    Export-Excel -path .\psconfeu2019.xlsx -AutoSize -AutoNameRange -ExcelChartDefinition $chartDef -WorksheetName SpeakerCountryChart -Show
```

There are many other ways you can use this module in creating report dashboards. The [GitHub repository contains several examples](https://github.com/dfinke/ImportExcel/tree/master/Examples) that you can use as a starting point.