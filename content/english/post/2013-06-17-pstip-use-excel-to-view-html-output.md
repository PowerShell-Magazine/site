---
title: '#PSTip Use Excel to View HTML Output'
author: Josh Miller
type: post
date: 2013-06-17T18:00:00+00:00
url: /2013/06/17/pstip-use-excel-to-view-html-output/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
Excel can be used to view HTML.  If you add some CSS styles to the result of _ConvertTo-HTML_ you can add much more rich output to Excel then you would get with a simple _Export-Csv_

```
$HTMLFile = Join-Path $Home "Processes.html"
$HTML = Get-Process | Select-Object CPU, ID, ProcessName | ConvertTo-HTML
 
# Reference for color names http://www.w3schools.com/cssref/css_colornames.asp
$HTML = $HTML -replace '^[<]tr[>][<]td[>][<][/]td[>]','<tr style="color:red" ><td></td>'
 
# Highlight anything that has Chrome or Google In the Name
$HTML = $HTML -replace '[<]td(?<T>[>]((chrome)|(Google[^<]*))[<][/]td[>])','<td style="background:blue;color:Yellow" ${T}'
$HTML | Out-File $HTMLFile
 
#Find a good version of Excel.exe
$Excel = Resolve-Path "C:\Program Files*\Microsoft Office\Office*\EXCEL.EXE" | 
            Select-Object -First 1 -ExpandProperty Path  
 
& $Excel $HTMLFile 
```

![](/images/excelcolor.png)