---
title: '#PSTip Comparing DateTime with a given precision'
author: Jakub Jareš
type: post
date: 2014-02-18T19:00:46+00:00
url: /2014/02/18/pstip-comparing-datetime-with-a-given-precision/
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
**Note**: This tip requires PowerShell 2.0 or above.

Comparing two _System.DateTime_ objects in PowerShell is really easy. All you need to do is:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $date1 = $date2 = Get-Date
PS&gt; $date1 -eq $date2
True
</pre>

When the two objects are compared their _Ticks_ properties are in fact compared:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $date1.Ticks -eq $date2.Ticks
True
</pre>

Ticks are the most precise way to represent a _DateTime;_ there are 10,000 ticks in one millisecond. That is 10 million ticks in one second. In my current project I needed the comparisons to be a little less precise and consider _DateTime_ objects equal even if they were a few milliseconds apart. To do that I decided to trim the DateTime objects by removing any excess milliseconds and ticks.

Doing that in PowerShell 3.0 is pretty easy:

```
PS> $date = Get-Date
PS> $date.Ticks
635276593924675678

PS> Get-Date -Date ($date) -Millisecond 0  | Select -ExpandProperty Ticks
635276593260000000
```

This will take the current date and set its Millisecond and any extra Ticks to 0.

In PowerShell 2.0 the conversion is a little bit more challenging because there is no _Millisecond_ parameter there. And you can&#8217;t simply use the _AddMilliseconds_ method to remove the extra milliseconds, because that won&#8217;t affect the extra ticks:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $date.AddMilliseconds(- $date.Millisecond) | Select -ExpandProperty Ticks
635276593920005678
</pre>

Another workaround has to be used:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; [datetime]($Date.Ticks - $date.Ticks % 10000000) | Select -ExpandProperty Ticks
635276593260000000
</pre>

I needed to make the script compatible with PowerShell 2.0 and I also wanted it to be readable. So I ended up writing function to do it, and more:


    function Trim-Date {
        param (
            [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
            [DateTime]$Date,
    
            [ValidateSet('Tick','Millisecond','Second','Minute','Hour','Day')]
            [String]$Precision = 'Second'
        )
    
        process {
    
            if ($Precision -eq 'Tick')
            {
                return $Date
            }
    
            $TickCount = Switch ($Precision)
            {
                'Millisecond' { 10000; break }
                'Second' { ( New-TimeSpan -Seconds 1 ).ticks; break }
                'Minute' { ( New-TimeSpan -Minutes 1 ).ticks; break }
                'Hour' { ( New-TimeSpan -Hours 1 ).ticks; break }
                'Day' { ( New-TimeSpan -Days 1 ).ticks; break }
            }
    
            $Result = $Date.Ticks - ( $Date.Ticks % $TickCount )
            [DateTime]$Result
        }
    }
This function takes two parameters: Date and Precision. The Date is the DateTime object to be trimmed and the Precision specifies what is the biggest unit to be kept. If, for example, _Hour_ precision is specified, minutes, seconds, millisecond and extra ticks are set to 0. In the following example the default Second precision is used so just milliseconds and any extra ticks are set to zero.

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; $date | Trim-Date -Precision Second  | Select -ExpandProperty Ticks
635276593260000000
</pre>

The function also turned out to be great for checking if two timestamps were created during the same day:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; ([datetime]"2013/12/10 12:14:36" | Trim-Date -Precision Day) -eq ([datetime]"2013/12/10 18:10:21" | Trim-Date -Precision Day)
True
</pre>

or creating precise plans for the next day:

<pre class="brush: powershell; title: ; notranslate" title="">PS&gt; (Get-Date -Hour 11).AddDays(1) | Trim-Date -Precision Hour
Tuesday, February 11, 2014 11:00:00 AM
</pre>